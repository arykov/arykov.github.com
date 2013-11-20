---
layout: post
title: "Connection closed: chasing a phantom"
description: ""
category:
tags: [troubleshooting, connection leak, heap dump, WebLogic, MyBatis, Spring, JTA]
---
Recently one of the new applications that has not made it to production yet started suffering from java.sql.SQLException: Connection closed exception. Worse yet was the persistent nature of the error. Once it started occurring it was reappearing until the container restart. 

Connection closed errors do happen - networks are never perfect, databases go down. But descent connection pool implementations tend to have some testing capabilities that remove "bad" connections before giving them to an application. WebLogic's certainly does. And while bug in WebLogic's implementation could not be ruled out it seemed unlikely. 

Failure to release an acquired connection back to the pool by an application seemed more likely. Application code relied on mybatis and mybatis-spring modules for all database activity and seemed to interact with those modules as prescribed. Looking at MyBatis code did not reveal anything obvious. Attempts to create a simple reproducer were unsuccessful.

It was time to try something new. Next time "Connection closed" reared its ugly head I resorted to taking a heap dump in hopes to find references to those closed connections. Turns out that references were held in Threadlocal(no surprise here). References looked like this:

weblogic.jdbc.wrapper.JTAConnection_weblogic_jdbc_wrapper_XAConnection_oracle_jdbc_driver_LogicalConnection @ 0xce64c7c0
'- connection org.mybatis.spring.transaction.SpringManagedTransaction @ 0xce58d9b0                                      
   '- transaction org.apache.ibatis.executor.SimpleExecutor @ 0xce531838                                                
      '- executor org.apache.ibatis.session.defaults.DefaultSqlSession @ 0xce4c6870                                     
         '- sqlSession org.mybatis.spring.SqlSessionHolder @ 0xce469598                                                 
            '- value java.util.HashMap$Entry @ 0xce40a5a8                                                               
               '- [12] java.util.HashMap$Entry[16] @ 0xce39c3a8                                                         
                  '- table java.util.HashMap @ 0xce24b168                                                               
                     '- value java.lang.ThreadLocal$ThreadLocalMap$Entry @ 0xce1e05a8        
                     

Problematic code search narrowed significantly. The org.mybatis.spring.SqlSessionHolder instance was put into ThreadLocal by org.springframework.transaction.support.TransactionSynchronizationManager on the request of org.mybatis.spring.SqlSessionUtils. SqlSessionUtils did seem to make an attempt to clean up but clearly not always successful. Why?

Lack of reproducer required some inventivness. [Small aspect](https://github.com/arykov/weblogic-probes/blob/master/src/main/java/com/ryaltech/weblogic/probe/ExecuteThreadMyBatisLeakDetectAspect.java) came to the rescue. It injected itself into WebLogic code responsible for returning threads back to the execute thread pool and verified whether org.mybatis.spring.SqlSessionHolder remained in thread local after each execution. When the reference was found event was logged and reference was cleaned. The latter part was more of a sympthom supression, rather than a true fix, of course. But one's brain functions best without distractions caused by users' induced excitement.

Produced error messages became good markers to look for in the logs and allowed to significantly reduce search effort. After looking at a few incidents source of the problem became clear. org.mybatis.spring.SqlSessionUtils registers an instance of org.springframework.transaction.support.TransactionSynchronizationAdapter subclass to perform the cleanup. The problem is that clean up was done in afterCompletion, which does not always get executed on the same thread as the related transaction. Fix should be available in mybatis-spring 1.2.2. Snapshot has been published a week or two ago. [Bug 18](https://github.com/mybatis/spring/issues/18) describes the issue and its fix.