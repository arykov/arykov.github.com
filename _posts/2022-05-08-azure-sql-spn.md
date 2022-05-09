---
layout: post
title: "Connecting to Azure SQL"
description: ""
category:
tags: [microsoft, sql, java, python, azure, pyodbc, msal, jdbc, odbc, development, linux]
---

Expected to be trivial experience of connecting to Azure SQL in Python and a lesser extent Java using SPN took more effort than anticipated. 

Recent version of MS SQL ODBC driver(Python needs for connectivity) and JDBC driver(Java) support two ways of doing it:
1. Using access token that you need to acquire separately
2. Providing SPN credentials to the driver and letting it worry about the token(ActiveDirectoryServicePrincipal). This feature has been supported since ODBC driver 17.7 and JDBC 9.2

Using Java was by far the simpler of the two. All you need is a Microsoft JDBC driver and msal library dependencies. Have a look [here](https://github.com/arykov/azure-sql-spn-samples/tree/main/java-sample).

Python presented more of challenge. You need unixODBC, MS SQL ODBC drivers as well as python packages specified in requirements.txt in my examples. Working samples are here for [python 3](https://github.com/arykov/azure-sql-spn-samples/tree/main/python3-sample) and here for [python 2](https://github.com/arykov/azure-sql-spn-samples/tree/main/python2-sample). I only implemented ActiveDirectoryServicePrincipal approach in Python 2, but if you are really stuck with this long EOLed version, consider going through [this](https://github.com/mkleehammer/pyodbc/issues/228)

Few things that took some time
- When using MSAL(at least versions in my samples), use scope https://database.windows.net/.default. Not sure what the history is, but a few variations are mentioned in various articles and even Microsoft's own documentation.
- Some versions of unixODBC(2.3.1 has the affliction ) appear to have memory management issues. Upgrading to 2.3.7 did the trick for me. To find out the version you are dealing with, use odbcinst -j command. Newer version that I used in RedHat came from Microsoft's repo. It is mentioned in [MS ODBC driver install instructions](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server)
- To install MS SQL ODBC follow [these instructions](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server)
- pyodbc contains native components and requires gcc, c++, python dev as well as unixODBC dev. Conda offers [precompiled package](https://anaconda.org/anaconda/pyodbc)
- If you are in an environment that has TLS introspection in place(some enterprises do that), you might have to install their CA certificate in the trust stores for things to work. For java you will need to import it into your default truststore(<JAVA_HOME>/jre/lib/security/cacerts) or create a custom one. In case of python, you might have to muck with both system(depends on distro RedHat 7 /etc/ssl/certs/ca-bundle.crt) and python truststore(<PYTHON_HOME>/lib/python<version>/site-packages/certifi/cacert.pem) for things to work. This is due to MSAL for python piggybacking on python http stack and unixODBC on the system one.