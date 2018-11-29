+++
title = "Accessing Oracle DB from Dyalog APL in Linux using SQAPL"
date = 2018-11-29T19:06:24Z
draft = false
tags = ["Linux", "Oracle", "ODBC", "DB", "APL", "SQAPL"]
categories = []
+++

Install Oracle ODBC driver as it's described in [Installation of Oracle ODBC driver in Linux](https://o7g8.github.io/posts/linux-oracle-odbc/).

Make Dyalog APL aware of ODBC and Oracle configurations:

```bash
echo 'export ODBCINI=/etc/odbc.ini' >> ~/.dyalog/dyalog.config
echo 'export TNS_ADMIN='$TNS_ADMIN >> ~/.dyalog/dyalog.config
```

Start an APL session and test your DB access:

```apl
      )load sqapl
/opt/mdyalog/17.0/64/unicode/ws/sqapl.dws saved Mon Jun 11 05:36:59 2018
      ⍝ ensure the necessary environment variables are initialized
      GetEnvironment 'TNS_ADMIN'
/usr/lib/oracle/12.1/client64/network/admin
      GetEnvironment 'ODBCINI'
/etc/odbc.ini

      ⍝ look into available DSNs, connect to a DSN and fetch some data
      SQA.Init''
0  SQAPL loaded from: cxdya63u64v.so Using default translation no aplunicd.ini present
      SQA.DSN''
0   OracleODBC-12c  Oracle 12c ODBC driver
      SQA.Connect 'c1' 'OracleODBC-12c' 'password' 'user'
0
      SQA.Do 'c1' 'select count(*) from dual'
0  c1.s1   1       6
      SQA.Do'c1' 'SELECT TO_CHAR (SYSDATE, ''MM-DD-YYYY HH24 :MI :SS'') "NOW" FROM DUAL;'
0  c1.s1    11-27-2018 17 :10 :00        6
      SQA.Close 'c1'
0
```

## References

* [Dyalog SQL Interface Guide](http://docs.dyalog.com/17.0/SQL%20Interface%20Guide.pdf)

* [Installation of Oracle ODBC driver in Linux](https://o7g8.github.io/posts/linux-oracle-odbc/)