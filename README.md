# SSB: SQL Storage Benchmark

## Getting started
1. Make sure your system meets all the requirements listed in the Requirements section below
2. If on Windows, install python 3.8.6, which can be downloaded [here](https://www.python.org/downloads/windows/).
3. Clone SSB repository or download its files
4. Install all SSB dependencies:
    ```
    py -m pip install -r requirements.txt
    ```
Ubuntu 20.04 doesn't like the above command and the requirements file is different, so use:


    pip install -r requirements_python3.txt

### Example - Populating a MSSQL database
Creating a 200GB MSSQL database with 10 (ten) 20GB tables with randomized data.
```
py setupssb.py -s mssql -d 200GB -ut 10 -D ssbtest -H 10.1.1.1 -l sa -p mysapassword -o 17 -P 1433
```
Ubuntu 20.04 doesn't seem to like the above command, so use this instead:
```
python3 setup_python3_ssb.py -s mssql -d 200GB -ut 10 -D ssbtest -H 10.1.1.1 -l sa -p mysapassword -o 17 -P 1433
```

### Example - Driving a workload using SSB
Starting a 80% SELECT workload for 30 seconds using 1 thread per user table.
```
py startssb.py -s mssql -D ssbtest -u 10 -r 80 -H hostname-or-IP -l sa -p Netapp123! -o 17 -S 30 -mu 10 -rs 70 -ru 70 -t 1
```
Ubuntu 20.04 doesn't seem to like the above command, so use this instead:
```
python3 startssb.py -s mssql -D ssbtest -u 10 -r 80 -H hostname-or-IP -l sa -p Netapp123! -o 17 -S 30 -mu 10 -rs 70 -ru 70 -t 1
```

## What is SSB?

SSB is an Open Source Benchmark tool written in Python. It is designed to generate a ‘real-world’ workload that emulates database interaction in such a way to measure the performance of the storage subsystem. The intent of SSB is to allow organizations and individuals to measure the performance of their storage subsystem under the stress of a SQL database workload.


SSB is designed to generate a constant SQL workload to the storage subsystem by using data that is randomized to meet “real-world” compression and deduplication efficiencies. This helps to ensure realistic results and limit data catchability. SSB allows the user to modify different read/write and random/sequential mixes. SSB currently uses an 8KB block size but may support other block sizes in the future.


SSB differentiates itself from other SQL benchmarks by focusing on measuring the performance of the storage subsystem. Many other benchmarks are designed to test the database’s engine like joins and aggregation (GROUP BY, ORDER BY, among others) and not its storage engine specifically. These benchmarks attempt to emulate warehouse or financial market OLTP workloads that involve user think time, cache and stored procedures and do not send a constant IO stream to the storage subsystem. As a result, this requires a substantial amount of database servers and computing resources to drive the storage subsystem to its peak performance.


SSB currently supports MySQL and Microsoft SQL Server. It will support other RDBMS in future releases as needed. 


Since SSB is written in Python, the operating system is transparent regardless of the database being tested. It can run on Linux, and with the appropriate Microsoft tools for Unix, can both build the database and drive the workload to a database server over the network, regardless of what that database server is running on.  


For more performant storage subsystems, SSB requires additional compute resources. You will need to experiment to determine the required compute resources for your environment. However, SSB can be run from a dedicated server without significant impact to the network or CPU on the client running SSB. Therefore, a single master client can drive several separate database servers. Similarly, more than one SSB client can also be used to drive the workload; however, in this case, it will be necessary to synchronize the SSB workloads either manually or through automation as this is currently beyond the scope of SSB functionality.  


The resulting storage performance data is not collected by SSB. It is instead collected by traditional performance tools such as iostat, nfsiostat or Windows Performance Monitor (PerfMon). Future versions may support single and aggregated performance data collection across database servers.


## Requirements

* Windows or Linux Operating System with Python 3 (tested with 3.8.6)
* Microsoft SQL Server 2008 and later
* MySQL (MariaDB) v10 and later

SSB leverages other python modules. The required Python modules are listed in the requirements.txt file. All the appropriate modules with versions can be installed using the following command:

On Linux
```
python3 -m pip install -r requirements.txt
```
Ubuntu 20.04 doesn't like the above command and the requirements file is different, so use:
```
pip install -r requirements_python3.txt
```

On Windows:
```
py -m pip install -r requirements.txt
``` 

When using SSB with Microsoft SQL Server, the appropriate Microsoft operating system tools must be installed. Microsoft SQL Server should install these tools by default. 

The Microsoft tools needed by SSB include:
* Microsoft ODBC Driver 17 for SQL Server, which can be downloaded [here](https://docs.microsoft.com/en-us/SQL/connect/odbc/windows/microsoft-odbc-driver-for-SQL-server-on-windows?view=SQL-server-ver15)
* Bulk Copy Program (bcp.exe), which can be downloaded [here](https://docs.microsoft.com/en-us/SQL/tools/bcp-utility?view=SQL-server-ver15#download-the-latest-version-of-bcp-utility)
    * bcp.exe should be in the operating system PATH environment

## Authentication
When running SSB, it is important that the database has been configured appropriately and the login being used for testing has the necessary permissions. When testing MySQL Server, it is important that both the user and the host that SSB is running on have permissions to access the MySQL server and the database being tested.

## Special Considerations
* When building a database in Microsoft SQL Server, bcp.exe by default uses port 1433 unless otherwise specified. Microsoft SQL Server 2008 and later uses Dynamic ports by default. If SSB is not told the correct port number that the Microsoft SQL Server is using, SSB will fail to load the data into the database as bcp.exe will default to port 1433.  The Microsoft Configurator tool can be used to determine the dynamic port number that Microsoft SQL Server is running on.  If SSB will be used to test MySQL, there is no port number requirement, however, SSB in this current version expects the port number that MySQL is listening on to be the default port number of 3306. Support for different port number assignments may be made available in future releases.

* When using the load generating tool, keep in mind that if too many users are attempting to write to the same table at the same time, a [deadlock](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-deadlocks-guide?view=sql-server-ver16) will occur. 

This issue does not occur when doing 100% reads, but using any percentage of writes can hit this error - especially when specifying a larger number of users (via the -u option). Naturally, the larger % of writes, the greater the chance of hitting this condition. You will be less likely to hit this issue if you also specify -mu (max users) as the same or greater value as users (ie, if -u is 100, then set -mu as 100 or more) or to the same or lesser value of the number of tables. (If your database was created with 20 tables, set max users to 20 or less to avoid deadlocks).

* If the number of max users specified exceeds the number of tables created in the database, then errors can also occur (as the tables won't exist of the users to read/write to).

* Before a test run, it may make sense to [drop memory caches](https://unix.stackexchange.com/questions/87908/how-do-you-empty-the-buffers-and-cache-on-a-linux-system) on the client to ensure RAM isn't helping us.

* It may also be beneficial (especially on read-heavy tests) to [modify the readahead caches](https://learn.microsoft.com/en-us/azure/azure-netapp-files/performance-linux-nfs-read-ahead) on the mount.

## Building a database
When building a MySQL database on either Linux or Windows, it is necessary to specify a path for temporary data files to be written. If no path is specified, SSB will use the default working directory as the data files path. SSB uses a built-in MySQL function to read the data files and quickly load the data into the database. There must be enough capacity to hold at least 4 GB of temporary data on the drive pointed to as the temporary data path, regardless of the size of the database specified. SSB does not currently retain the datafiles so the space required will only be temporary.  When building a Microsoft SQL Server database on either Linux or Windows, it is not necessary to specify a path for temporary data files to be written.

## Recommended Settings
### MySQL 
* Currently the default page size for MySQL is 16KB and SSB currently only has support for 16KB page size, although this will likely change in future versions. 

* SSB with MySQL has been tested only with the InnoDB database engine. It is recommended that InnoDB be configured as shown below.

/etc/my.cnf:
```
innodb_file_per_table=on
innodb_flush_log_at_trx_commit=2
innodb_open_files=4096
innodb_page_size=16384
innodb_read_io_threads=64
innodb_write_io_threads=64
innodb_doublewrite=0;
max_connections=1000
innodb_thread_concurrency=128
innodb_max_dirty_pages_pct=0
max_prepared_stmt_count=1048576

```
For a synopsis on what the following recommended settings do, please refer to the [MySQL Documentation](https://dev.mysql.com/doc/).

### Microsoft SQL Server 
* Checkpointing should be set to 32767. This disables checkpointing for performance reasons.
* Database logging mode: Simple
* Database Target Recovery Interval: 0 seconds

Microsoft also publishes a [best practice for SQL performance document](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-performance-best-practices?view=sql-server-ver16).

These recommendations include some settings that leverage the use of the [tuned](https://tuned-project.org/) utility in Linux.

Using the MS SQL profile changes these settings and when tuned is disabled, the changes revert to defaults. Highly recommended.

Network setting changes will vary based on your configuration. (For instance, jumbo frames might not be enabled in your network, NIC drivers may not be able to enable some settings).

## SSB Command Line & Parameters

### setupssb.py | setup_python3_ssb.py
```
[-h Help] (optional): Provides help on the start_SSB.py command

-s {mySQL, msSQL}: Currently only MySQL or Microsoft SQL Server is supported. Future versions will be supported as needed.

-d dbsize: Database size in KB,MB,GB,TB: example 1TB. This parameter takes the specified database size and makes the correct calculations based on the size entered into “Kilobytes(KB)”,”Megabytes(MB)”, “Gigabytes(GB)” or “Terabytes(TB)”. Entering 1024GB is the same as entering 1TB.

-ut user_tables: The Number of user tables to create in the database. There is one user per user table. The number of the user tables in the database will determine the number of 8KB rows per user to create the database of the specified size. 

-D database-name: The name of the test database 

-H host:  Server FQDN or IP or Instance name. The host parameter will take the IP address of a Microsoft SQL Server or MySQL server as xx.xx.xx.xx\instance_name or xx.xx.xx.xx. It will also take the DNS short name. For example, Sqlsrvr\Sql01 or MySql01. It can also take the fully qualified domain name as well.  If you are running SSB from a Linux environment and are accessing a Microsoft SQL Server database, either the name or IP address needs to use “\\” as Linux treats a single “\” as an escape character. For the connection string to be seen correctly from Linux, using “\\” will result in a single “\” in the name. For example, SqlSrvr\Sql01 will result in SSB seeing SqlSrvrSql01 and the connection will fail. Using SqlSrvr\\Sql01 will result in SSB seeing SqlSrvr\Sql01 and will result in a proper connection to the SQL server being tested. 

-l username: The user specified to create the database and load data into that database needs to be a user that has database permissions. The Microsoft SQL Server user must be a native SQL Server user or the sa account and must have full control permissions of the database to be tested. The user for MySQL must be a user that has full permissions to the MySQL server and the database to be tested. The host that SSB is running from must also be able to access the MySQL server remotely. See the MySQL documentation for how to set this up correctly.  

-p password: The database password for the user to access the Microsoft SQL Server or MySQL

-o sql-driver-number (optional): Tells SSB what ODBC driver to use. MySQL does not use this parameter. The Microsoft ODBC drivers for Windows and Linux use the string “ODBC Driver XX for SQL Server”. For example, if sql-driver-number is 17, then SSB will use that number to form the correct string for the Microsoft ODBC driver.  Note that the ODBC driver may need to be downloaded from Microsoft separately.  

-t temp-path (optional): This is the temporary file location where the data files for MySQL will be written. The data files contain the actual data that will be stored in the database. SSB uses the native MySQL function “Load data in-file” to load large amounts of data into the database. If no path is specified, the current working directory is used to hold the temporary data. 4GB of disk space is needed at a minimum. 

-P sql-port (optional): When building a database in Microsoft SQL Server, bcp.exe by default uses port 1433 unless otherwise specified. Microsoft SQL Server uses dynamic ports by default. The default sql-port number is 1433 for SQL Server and 3306 for MySQL. 
```

### startssb.py
```
[-h Help] (optional): Provides help on the start_SSB.py command

-S Secs: The length of the test in seconds

-u Users: The number of users to run in the test. You can enter more users than exist in the database. The effect is the program will cycle through the Users parameter so that each user table is accessed. For example, if you have a 4-user database and you enter “-u 6” for the test, then the first four users will access their respective user tables, the fifth user will access user one table, and the sixth user will access user two table. If you enter “-u 8”, then each user table would have two users accessing their respective table.

-mu max_users_tables: The maximum number of user tables in the database.

[-mr max_rows] rows (optional – default to actual number of rows in each user table): This is the  maximum number of rows in each user table. Entering anything less than the maximum  number of rows per user has the effect of changing the working set size,  with each user table being accessed, but limited to the max number of rows specified.

-r read_percent: The required percentage of SQL Selects in percentage. Entering “80” will be read as 80% SQL Selects and 20% SQL Updates. Entering “100” SQL Selects will result in a 100% read workload. Note that SQL Update commands issue both a read and write.
 
-s {mySQL, msSQL}: Choice between MySQL or MSSQL. Future versions of other Databases will be supported as needed. 
 
-rs rows_select: This parameter increases the rows read in a single query, increasing the read throughput

-ru rows_update: This parameter increases the rows updated in a single query, increasing the write throughput  

-D database_name: The name of the database under test

-H Host: Name of the SQL server. If you are running SSB from a Linux environment and are accessing a Microsoft SQL Server database, either the name or IP address when specifying a SQL Server instance name need to use “\\” as Linux treats a single “\” as an escape character. For the connection string to be seen correctly from Linux, using two “\\” will result in a single “\” in the name. For example: SqlSrvr\Sql01 will result in SSB seeing SqlSrvrSql01 and the connection will fail. Using SqlSrvr\\Sql01 will result in SSB seeing SqlSrvr\Sql01 and assuming the login credentials are correct will result in a proper connection to the SQL server being tested.

-l Username: The SQL login username: At this time, Integrated Windows Authentication is not supported so there should be the sa account or equivalent alternative native SQL account on the Microsoft SQL Server database being used. If you are testing MySQL, the proper changes to the host/user authentication tables need to be made for the connection to succeed.

-p Password: The password for the user with permissions to access the database instance     being tested.

[-o SQL Driver Version] (optional) This parameter is required for Microsoft SQL Server. It is not required for MySQL. The current supported Microsoft ODBC drivers have the string, “ODBC Driver XX for SQL Server” where XX is the current version of the SQL ODBC driver. The -o parameter uses the Version number XX to specify which Microsoft SQL ODBC driver is installed on the SSB client. For example, you have Microsoft ODBC Driver 13 for SQL Server installed, the -o parameter would be -o 13. This option is only valid if the SQL version being specified is “mssql” or Microsoft SQL Server, otherwise, it is not used.
```
