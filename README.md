#Configure a user and database for Asterisk in MySQL
```shell
mysql -u root -p
```
```sql
CREATE USER 'asterisk'@'%' IDENTIFIED BY 'Password@123';
CREATE DATABASE asterisk;
GRANT ALL PRIVILEGES ON asterisk.* TO 'asterisk'@'%';
exit
```
```shell
mysql -u asterisk -p asterisk
exit
```

### Install ODBC and the MySQL ODBC connector
#### Install the latest unixODBC and GNU Libtool Dynamic Module Loader packages
```shell
sudo yum install unixODBC unixODBC-devel libtool-ltdl libtool-ltdl-devel
```
#### Install the latest MySQL ODBC connector
```shell
sudo yum install mysql-connector-odbc
```
#### Create CDR table in asterisk database
```shell
mysql -p
```

```sql
CREATE DATABASE asterisk;

CREATE TABLE cdr ( 
        calldate TIMESTAMP NOT NULL default CURRENT_TIMESTAMP, 
        clid varchar(80) NOT NULL default '', 
        src varchar(80) NOT NULL default '', 
        dst varchar(80) NOT NULL default '', 
        dcontext varchar(80) NOT NULL default '', 
        channel varchar(80) NOT NULL default '', 
        dstchannel varchar(80) NOT NULL default '', 
        lastapp varchar(80) NOT NULL default '', 
        lastdata varchar(80) NOT NULL default '', 
        duration int(11) NOT NULL default '0', 
        billsec int(11) NOT NULL default '0', 
        disposition varchar(45) NOT NULL default '', 
        amaflags int(11) NOT NULL default '0', 
        accountcode varchar(20) NOT NULL default '', 
        uniqueid varchar(32) NOT NULL default '', 
        userfield varchar(255) NOT NULL default '' ,
        peeraccount varchar(20) NOT NULL default '',
        sequence int(11) NOT NULL default '0'
);
```
```shell
vi /etc/odbc.ini
```
```
[asterisk-connector]
Description = MySQL connection to 'asterisk' database
Driver = MySQL
Database = asterisk
User = root
Password = Password@123
Server = localhost
Port = 3306
Socket = /var/lib/mysql/mysql.sock
```

#### Change Description, Driver if necessory from `vi /etc/odbcinst.ini`
```shell
vi /etc/odbc.ini
```
```
[asterisk-connector]
Description = MySQL ODBC 8.0 ANSI Driver
Driver = MySQL ODBC 8.0 ANSI Driver
Database = asterisk
User = root
Password = Password@123
Server = localhost
Port = 3306
Socket = /var/lib/mysql/mysql.sock
```
```shell
isql -v asterisk-connector asterisk replace_with_strong_password
```
## Configure res_odbc.conf
```shell
vi /etc/asterisk/res_odbc.conf
```
```
[asterisk]
enabled => yes
dsn => your-configured-dsn-name
username => your-database-username
password => insecurepassword
pre-connect => yes
```
```shell
vi /etc/odbc.ini
```
```
[asterisk-connector]
Description = MySQL ODBC 8.0 ANSI Driver
Driver = MySQL ODBC 8.0 ANSI Driver
Database = asterisk
User = root
Password = Sachin@123
Server = localhost
Port = 3306
Socket = /var/lib/mysql/mysql.sock
```
```shell
vi /etc/asterisk/cdr_adaptive_odbc.conf
```

#### from vi /etc/odbc.ini
> [asterisk-connector]

#### from vi /etc/asterisk/res_odbc.conf
> connection=asterisk

#### table name
> table=cdr


#### Then start up Asterisk and assuming res_odbc loads properly on the CLI you can use odbc show to verify a DSN is configured and shows up:
```shell
asterisk -vvvvvvr
```
```shell
localhost*CLI> odbc show
```

> ODBC DSN Settings
> -----------------
> \
>   Name:   asterisk\
>   DSN:    asterisk-connector\
>     Number of active connections: 1 (out of 1)\
>     Logging: Disabled


```shell
localhost*CLI> cdr show status
```

> Call Detail Record (CDR) settings
> ----------------------------------\
>   Logging:                    Enabled\
>   Mode:                       Simple\
>   Log unanswered calls:       Yes\
>   Log congestion:             Yes\
> 
> \* Registered Backends\
>   -------------------\
>    Adaptive ODBC\
>    cdr_manager (suspended)\
>    cdr-custom\
>    csv


## make a call
#### you can check the cdr in csv log at
```shell
tail /var/log/asterisk/cdr-csv/Master.csv
```
#### and in mysql > asterisk > cdr
```shell
mysql -p
```
```sql
USE asterisk;
SELECT * FROM cdr;
```
