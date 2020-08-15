---
description: I may save you many headaches
tags: python mac mssql 
---

# 1 - Get your configuration ready 

Check that you have FreeTDS installed 

```
brew info freetds
```

Verify the configuration and add the server you want to connect to :

```
vim /usr/local/etc/odbcinst.ini
 
"""
[FreeTDS]
Description=FreeTDS Driver for Linux & MSSQL
Driver=/usr/local/lib/libtdsodbc.so
Setup=/usr/local/lib/libtdsodbc.so
UsageCount=1
"""
 
vim /Users/admor/.odbc.ini
 
"""
[MYMSSQL]
Description         = Test to SQLServer
Driver              = FreeTDS
Server              = 1.2.3.4
Port                = 1033
UID                 = #########
PWD                 = #########
"""
```

# 2 - Connect using python

```python
import pyodbc
conn = pyodbc.connect('DSN=MYMSSQL;UID=???;PWD=???')
crsr = conn.cursor()
crsr.execute("SELECT @@VERSION")
#<pyodbc.Cursor object at 0x10b6162b0>
```
