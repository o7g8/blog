+++
title = "Installation of Oracle ODBC driver in Linux"
date = 2018-11-29T18:57:19Z
draft = false
tags = ["Linux", "Oracle", "ODBC", "DB"]
categories = []
+++

# Making Oracle ODBC driver working with Dyalog APL/Linux

There are two alternative ODBC implementations available on Linux:

* [unixODBC](http://www.unixodbc.org/)

* [iODBC](http://www.iodbc.org)

You can read about differences between the projects in the article [What are the functional differences between iODBC and unixODBC?](https://stackoverflow.com/questions/7548825/what-are-the-functional-differences-between-iodbc-and-unixodbc)

Which of them to choose? At the time of writing, the unixODBC is more actively developed than iODBC and unixODBC looks more popular as well.

My goal is to to get working [Dyalog APL SQAPL](http://docs.dyalog.com/17.0/SQL%20Interface%20Guide.pdf) on Linux with Oracle database. The SQAPL supports unixODBC, therefore I'm firm with my decision :)

## Choice of Oracle ODBC drivers, available on Linux

There is number of Oracle ODBC driver implementations:

* [Oracle Instant Client ODBC driver](https://www.oracle.com/technetwork/database/database-technologies/instant-client/overview/index.html): free. The ODBC driver is distributed by Oracle itself.

* [Simba Oracle ODBC Driver](https://www.simba.com/drivers/oracle-odbc-jdbc/): paid, offers 20 days trial, [documentation](https://www.simba.com/products/Oracle/doc/ODBC_InstallGuide/linux/content/odbc/or/configuring/odbcini.htm)

* [Devart ODBC Driver for Oracle](https://www.devart.com/odbc/oracle/download.html): paid, offers 30 days trial.

* [Easysoft Oracle® ODBC Driver](https://www.easysoft.com/products/data_access/odbc_oracle_driver/index.html#section=tab-1): paid, offers 14 days trial, [documentation](https://www.easysoft.com/products/data_access/odbc_oracle_driver/getting_started.html).

* [Progress DataDirect Oracle ODBC driver](https://www.progress.com/odbc/oracle-database): paid. Dyalog recommends the driver for SQAPL Interface. Funny enough Oracle itself recommends the driver in their own product [Oracle® Fusion Middleware Metadata Repository Builder](https://docs.oracle.com/cd/E25178_01/fusionapps.1111/e20836/deploy_rpd.htm).

I choose the free Oracle Instant Client.

## Installation of Oracle client

Here is the table with compatible versions of unixODBC Driver Manager and Instant Client:

Instant Client version | unixODBC version | O/S with the required unixODBC version
--- | --- | ---
Instant Client 12.2 | 2.3.4 | Ubuntu 18.04
Instant Client 12.1 | 2.3.1 | Ubuntu 16.04
Instant Client 11g | 2.2.11, 2.2.14 | Ubuntu 14.04

The guidelines below are the "shortest path to success" which are compiled from articles which I have found. Some of the articles you may find in the `References` section. 

I use Ubuntu 16.04 and therefore I download following packages of Instant Client 12.1 from [Instant Client Downloads for Linux x86-64 (64-bit)](https://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html):

* oracle-instantclient12.1-basiclite-12.1.0.2.0-1.x86_64.rpm - Oracle client libraries

* oracle-instantclient12.1-odbc-12.1.0.2.0-1.x86_64.rpm - the ODBC driver

* oracle-instantclient12.1-sqlplus-12.1.0.2.0-1.x86_64.rpm - SQL*Plus, it is needed only for troubleshooting and therefore the package be skipped

Install the downloaded packages and their dependencies:

```bash
$ sudo apt install alien libaio1
$ sudo alien -i oracle-instantclient12.1-*
```

Make the Oracle libraries available in `LD_LIBRARY_PATH` and initialize system-wide `ORACLE_HOME`:

```bash
$ echo /usr/lib/oracle/12.1/client64/lib/ | sudo tee  /etc/ld.so.conf.d/oracle.conf && sudo chmod o+r /etc/ld.so.conf.d/oracle.conf
$ sudo ldconfig
$ echo 'export ORACLE_HOME=/usr/lib/oracle/12.1/client64' | sudo tee /etc/profile.d/oracle.sh && sudo chmod o+r /etc/profile.d/oracle.sh
$ . /etc/profile.d/oracle.sh
```

Ensure all library references are resolved for SQL*Plus:

```bash
$ ldd `which sqlplus64`
        linux-vdso.so.1 =>  (0x00007ffc77593000)
        libsqlplus.so => /usr/lib/oracle/12.1/client64/lib/libsqlplus.so (0x00007f3cf0112000)
        libclntsh.so.12.1 => /usr/lib/oracle/12.1/client64/lib/libclntsh.so.12.1 (0x00007f3ced155000)
        libclntshcore.so.12.1 => /usr/lib/oracle/12.1/client64/lib/libclntshcore.so.12.1 (0x00007f3cecbe3000)
        libmql1.so => /usr/lib/oracle/12.1/client64/lib/libmql1.so (0x00007f3cec96d000)
        libipc1.so => /usr/lib/oracle/12.1/client64/lib/libipc1.so (0x00007f3cec5ef000)
        libnnz12.so => /usr/lib/oracle/12.1/client64/lib/libnnz12.so (0x00007f3cebee5000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f3cebce1000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f3ceb9d8000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f3ceb7bb000)
        libnsl.so.1 => /lib/x86_64-linux-gnu/libnsl.so.1 (0x00007f3ceb5a2000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007f3ceb39a000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3ceafd0000)
        libons.so => /usr/lib/oracle/12.1/client64/lib/libons.so (0x00007f3cead8b000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f3cf040a000)
        libaio.so.1 => /lib/x86_64-linux-gnu/libaio.so.1 (0x00007f3ceab89000)
```

Test your database connection using SQL*Plus. The successful session should look like this:

```bash
$ sqlplus64 username/password@//dbhost:1521/SID

SQL*Plus: Release 12.1.0.2.0 Production on Mon Nov 26 08:31:29 2018

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

Last Successful login time: Sun Nov 25 2018 19:40:35 -08:00

Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> SELECT COUNT(*) FROM DUAL;

  COUNT(*)
----------
         1
```

If you don't plan to use SQL*Plus in the future, you may safely remove package `oracle-instantclient12.1-sqlplus` without affecting your ODBC setup.

## Installation of unixODBC

Install unixODBC and ensure all library dependencies of the Oracle ODBC driver are resolved:

```bash
$ sudo apt install unixodbc
$ ldd /usr/lib/oracle/12.1/client64/lib/libsqora.so.12.1
        linux-vdso.so.1 =>  (0x00007fff5d39b000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fc959b08000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fc9597ff000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fc9595e2000)
        libnsl.so.1 => /lib/x86_64-linux-gnu/libnsl.so.1 (0x00007fc9593c9000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007fc9591c1000)
        libclntsh.so.12.1 => /usr/lib/oracle/12.1/client64/lib/libclntsh.so.12.1 (0x00007fc956204000)
        libodbcinst.so.2 => /usr/lib/x86_64-linux-gnu/libodbcinst.so.2 (0x00007fc955ff2000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fc955c28000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fc95a17d000)
        libmql1.so => /usr/lib/oracle/12.1/client64/lib/libmql1.so (0x00007fc9559b2000)
        libipc1.so => /usr/lib/oracle/12.1/client64/lib/libipc1.so (0x00007fc955634000)
        libnnz12.so => /usr/lib/oracle/12.1/client64/lib/libnnz12.so (0x00007fc954f2a000)
        libons.so => /usr/lib/oracle/12.1/client64/lib/libons.so (0x00007fc954ce5000)
        libaio.so.1 => /lib/x86_64-linux-gnu/libaio.so.1 (0x00007fc954ae3000)
        libclntshcore.so.12.1 => /usr/lib/oracle/12.1/client64/lib/libclntshcore.so.12.1 (0x00007fc954571000)
        libltdl.so.7 => /usr/lib/x86_64-linux-gnu/libltdl.so.7 (0x00007fc954367000)
```

Initialize `TNS_ADMIN` and populate `tnsnames.ora`.

```bash
$ echo 'export TNS_ADMIN=$ORACLE_HOME/network/admin' | sudo tee -a /etc/profile.d/oracle.sh
$ . /etc/profile.d/oracle.sh

$ sudo mkdir -p $ORACLE_HOME/network/admin
$ cat | sudo tee $ORACLE_HOME/network/admin/tnsnames.ora
# paste tnsnames content here and finish with Ctrl-D
```

Look where unixODBC expects to find configuration files:

```bash
$ odbcinst -j
unixODBC 2.3.1
DRIVERS............: /etc/odbcinst.ini
SYSTEM DATA SOURCES: /etc/odbc.ini
FILE DATA SOURCES..: /etc/ODBCDataSources
USER DATA SOURCES..: /home/vagrant/.odbc.ini
SQLULEN Size.......: 8
SQLLEN Size........: 8
SQLSETPOSIROW Size.: 8
```

Configure `/etc/odbcinst.ini` as:

```ini
[Oracle 12c ODBC driver]
Description     = Oracle ODBC driver for Oracle 12c
Driver          = /usr/lib/oracle/12.1/client64/lib/libsqora.so.12.1
Setup           =
FileUsage       =
CPTimeout       =
CPReuse         =
```

Configure `/etc/odbc.ini` as:

```ini
[OracleODBC-12c]
Application Attributes = T
Attributes = W
BatchAutocommitMode = IfAllSuccessful
BindAsFLOAT = F
CloseCursor = F
DisableDPM = F
DisableMTS = T
Driver = Oracle 12c ODBC driver
DSN = OracleODBC-12c
EXECSchemaOpt =
EXECSyntax = T
Failover = T
FailoverDelay = 10
FailoverRetryCount = 10
FetchBufferSize = 64000
ForceWCHAR = F
Lobs = T
Longs = T
MaxLargeData = 0
MetadataIdDefault = F
QueryTimeout = T
ResultSets = T
ServerName = <!!your TNS name goes here!!>
SQLGetData extensions = F
Translation DLL =
Translation Option = 0
DisableRULEHint = T
UserID =
StatementCache=F
CacheBufferSize=20
UseOCIDescribeAny=F
SQLTranslateErrors=F
MaxTokenSize=8192
AggregateSQLType=FLOAT
```

Test the ODBC connection with unixODBC client:

```bash
$ isql "OracleODBC-12c" <dbuser> <dbpassword> -v
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL> SELECT COUNT(*) FROM DUAL;
+-----------------------------------------+
| COUNT(*)                                |
+-----------------------------------------+
| 1                                       |
+-----------------------------------------+
SQLRowCount returns -1
1 rows fetched
SQL>
```

If your output looks like this, then the ODBC part is done.

In my next article I describe how to make the Dyalog APL work with the unixODBC.

## References

* /usr/share/oracle/12.1/client64/ODBC_IC_Readme_Unix.html

* [Oracle Instant Client](https://help.ubuntu.com/community/Oracle%20Instant%20Client)

* [Installing Oracle SQL*Plus client on Ubuntu](http://webikon.com/cases/installing-oracle-sql-plus-client-on-ubuntu)

* [Install Oracle Instant Client on Ubuntu (Linux)](http://mritjudge.com/linux/install-oracle-instant-client-on-ubuntu-linux/)

* [Oracle ODBC FAQ](http://www.orafaq.com/wiki/ODBC_FAQ)

* [Using the Oracle ODBC Driver](http://www.dba86.com/docs/oracle/12.2/ADFNS/odbc-driver.htm), see also:
  * [Oracle Instant Client ODBC Installation Notes](https://www.oracle.com/technetwork/database/features/oci/odbc-ic-releasenotes-094306.html)
  * [Certification Matrix for Oracle ODBC Driver on UNIX Platforms](https://web.archive.org/web/20181126140812/http://www.dba86.com/docs/oracle/12.2/ODBCR/index.html#GUID-421E4E54-5F99-4403-A878-B1A44601806B)
  * [Post-installation Task](https://web.archive.org/web/20181126140812/http://www.dba86.com/docs/oracle/12.2/ODBCR/index.html#GUID-47268A17-F2CB-4615-BEEE-E6A6B8C1F422)

* [Oracle Database - ODBC Driver Installation on Linux](https://gerardnico.com/db/oracle/odbc_linux)

* [How do I setup Oracle ODBC drivers on RHEL 6/Linux](https://stackoverflow.com/questions/13922415/how-do-i-setup-oracle-odbc-drivers-on-rhel-6-linux)

* [[zLinux] RHEL and SUSE: Configuring an Oracle datasource to use ODBC](https://www.ibm.com/support/knowledgecenter/en/SS7UH9_6.4.2/ncm/wip/confg/task/ncm_config_configureoracleforodbc.html)

* [To install the Oracle Instant Client](http://docs.adaptivecomputing.com/9-1-0/MWS/Content/topics/moabWorkloadManager/topics/databases/installingTheOracleInstantClient.htm)

* [To connect to an Oracle database with an ODBC driver](http://docs.adaptivecomputing.com/9-1-0/MWS/Content/topics/moabWorkloadManager/topics/databases/oracle.html)