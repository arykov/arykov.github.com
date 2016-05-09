---
layout: post
title: "Identity Propogation from Crystal Reports Enterprise to Oracle DB"
description: ""
category:
tags: [reporting, business objects, crystal reports, SAP, java, jdbc]
---

I encountered a curious puzzle while upgrading a simple WebSphere application and its reports running in Business Objects Enterprise. It was designed in a way that all database query results were dependent on Oracle DB connection user identity. 

Historically each application and BOE user had a corresponding Oracle DB user id and functionality was easy to achieve. This approach, however, came with its set of problems. Maintaining a large number of such users and synching passwords is no small feat. Connection pooling is impossible.

Moving away from this on the application side was easy - setClientIdentifier(client) for older versions of Oracle or setClientInfo(”OCSID.CLIENTID", client) starting with Oracle 12c did the trick combined with minor SQL changes. Luck(or someone's wizdom) had it that it was a change to a single stored function.

Now the fun part - reporting.  It turns out that Crystal Reports for Enterprise(part of SAP BOE now) does not have an easy way to call setClientIdentifier on the connection before report is generated unless report is using a Universe. Unfortunately ours did not and there was a sizable number of them.

BOE has an option to associate each user with DB user/password pair and use this combination when creating a connection. That's what we were trying to get away from. But what if we were to create a JDBC driver that pulls connection information from somewhere else(say property file) and uses user id  from BOE to call setClientInfo on created connection?

This approach worked for roughly half of our reports. Changing connection for the rest using crystal reports editor failed. This happened to reports with Oracle specific type conversions. Crystal Reports evidently uses JDBC driver class name to determine DB type and the driver for Oracle has to be oracle.jdbc.OracleDriver. 

To address this I had to do a little load time byte code weaving to intercept calls to oracle.jdbc.driver.OracleDriver.connect and route them to com.ryaltech.jdbc.OracleIdentityDriver.connect for urls starting with jdbc:ryaltech:oraid: and oracle.jdbc.driver.OracleDriver.connect otherwise. 
 
Source code for this driver is available [here](https://github.com/arykov/identityoraclejdbc). 

For this to work you need to do the following:

1.  Create a property file that contains Oracle DB connection information. To do this use [com.ryaltech.jdbc.StoreConnProperties](https://github.com/arykov/identityoraclejdbc/blob/master/src/main/java/com/ryaltech/jdbc/StoreConnProperties.java) utility packaged with the driver. For example: `java -classpath OracleIdentityDriver.jar com.ryaltech.jdbc.StoreConnProperties -url jdbc:oracle:thin:@oraclehost:1521:XE -user dbuser -s /opt/creproperties/mydb.properties -password`
2.  Make sure each user has a db user specified in BOE 
3.  Add <path_to_oraclejdbc>/ojdbc6.jar to the classpath in CRConfig.xml
4.  Set variable BOE_SSL_JVMOPTIONS if not set already and add `-javaagent:<path_to_oracle_identity_driver>/OracleIdentityDriver.jar` to it. Alternatively you could make change to CRConfig.xml similar to this `<JVMMaxHeap>64000000 -javaagent:/home/arykov/identityoraclejdbc/target/OracleIdentityDriver.jar</JVMMaxHeap>` 
5.  Deploy your reports using [Report Deployer](https://github.com/arykov/CrystalReportDeployer). For example: `java -classpath CrystalReportDeployer.jar com.ryaltech.sap.deployment.ReportDeployer -user boeuser -server boeserver:6400 -dbproperties /opt/creproperties/mydb.properties -sourcepath /opt/crystalreportlocation -destinationpath myreports -password`

If you want to switch connection in Crystal Report Studio itself, you can. To achieve that follow steps 1, 3 and 4 above and use jdbc url `jdbc:ryaltech:oraid:@<property_file_location>/<property_file>`. For example `jdbc:ryaltech:oraid:@/opt/creproperties/mydb.properties`
