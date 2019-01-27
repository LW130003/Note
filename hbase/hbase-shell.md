# HBase Shell

source: https://www.tutorialspoint.com/hbase/hbase_shell.htm

HBase contains a shell using which you can communicate with HBase. HBase uses the Hadoop File System to store its data. It will have a master server and region servers. THe data storage will be in the form of regions (tables). These regions will be split up and stored in region servers. The master server manages these region servers and all these tasks take place on HDFS. 

## General Commands
- **status** - Provides the status of HBase, for example, the number of servers.
- **version** - Provides the version of HBase being used.
- **table_help** - Provides help for table-reference commands.
- **whoami** - Provides information about the user.

## Data Definition Language
These are the commands that operate on the tables in HBase.
- **create** - Creates a table.
- **list** - Lists all the tables in HBase
- **disable** - Disables a table.
- **is_disabled** - Verifies whether a table is disabled.
- **enable** - Enables a table.
- **is_enabled** - Verifies whether a table is enabled.
- **describe** - Provides the description of a table.
- **alter** - Alters a table.
- **exists** - Verifies whether a table exists.
- **drop** - Drops a table from HBase.
- **drop_all** - Drops the tables matching the 'regex' given in the command.
- **Java Admin API** - prior to all the above commands, Java provides an Admin API to achieve DDL functionalities through programming. Under **org.apache.hadoop.hbase.client** package, HBaseAdmin and HTableDescriptor are the two important classes in this package that provide DDL functionalities.

## Data Manipulation Language
- **put** - Puts a cell value at a specified column in a specified row in a particular table.
- **get** - Fetches the contents of row or a cell.
- **delete** - Deletes a cell value in a table.
- **deleteall** - Deletes all the cells in a given row.
- **scan** - Scans and returns the table data.
- **count** - Counts and returns the number of rows in a table.
- **truncate** - Disables, drops, and recreates a specified table.
- **Java client API** - Prior to all the above commands, Java provides a client API to achieve DML functionalities, CRUD (Create Retrieve Update Delete) operations and more through programming, under org.apache.hadoop.hbase.client package. HTable Put and Get are the important classes in this package.

## Starting HBase Shell
To access the HBase Shell, 
1. You have to navigate to the HBase Home folder
```bash
cd /usr/localhost/
cd hbase
```
2. Start the HBase Interactive Shell using **"hbase shell"** command
```bash
./bin/hbase shell
```

To exit the interactive shell command at any moment, type exit or use <ctrl + c>. Check the shell functioning before proceeding further. Use the **list** command for this purpose. **List** is a command used to get the list of all the tables in HBase. First of all, verify the installation and the configuration of HBase in your system using this command as shown below

```bash
hbase(main):001:0> list
```

When you type this command, it gives you the following output.
```bash
hbase(main):001:0> list
TABLE
```
