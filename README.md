# Azure DMS: Oracle to Azure PostgreSQL

## Prerequisites:

1.  Have credentials for a single-tenant Oracle Database (10g, 11c, 12c)
2.  Have credentials for single-tenant Azure PostgreSQL Database
3.  Have these ports open for both databases/Vm’s: 443, 53, 9354, 445, 12000
4.  Have a network drive set up for “File Share” which contains the OCI Driver for Azure DMS to use, driver download: [https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html#ic_winx64_inst](https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html#ic_winx64_inst)
    

### Step 1: Setup Tables for data sync

1.  Go to your PostgreSQL Database and create a table with the exact same name as the table you want to data sync with (from Oracle Database)
2.  Ensure that column datatypes from the PostgreSQL table are properly mapped to the Oracle Database table.
3.  In your Oracle Database; change the ‘UNITS’ to be ‘CHAR’ and not ‘BYTE’ for character datatypes in your desired table for DMS. (When DMS is active and the character outputs to PostgreSQL are incorrect, change this setting)

![](/Images/1.png)


#### Step 2: Enable Archive Redo Logs (Required by Azure DMS to capture data change)

1. Sign into your oracle database using sqlplus: sqlplus (user)/(password) as sysdba
2.  SHUTDOWN IMMEDIATE;
3.  STARTUP MOUNT;
4.  ALTER DATABASE ARCHIVELOG;
5.  ALTER DATABASE OPEN;
6.  SELECT log_mode FROM v$database;
7.  This should return ‘ARCHIVELOG’ if done correctly
8.  ALTER DATABASE ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY, UNIQUE) COLUMNS;
9.  ALTER TABLE [TABLENAME] ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
10.  ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
11.  ALTER TABLE xxx ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
12.  SELECT supplemental_log_data_min FROM v$database;
13.  This should return ‘YES’ if done correctly

##### Step 3: Connect Oracle Database to Azure DMS 

1. Create new Azure Migration Service resource 

![](/Images/2.png)

2. Enter in your credentials for the resource and make sure to choose the “Premium SKU” as this is required for continuous data migration. 

![](/Images/3.png)

3. Create new migration project 

![](/Images/4.png)

4. Create the migration project, making sure to once again use the “Premium SKU” 

![](/Images/5.png)

5. Create a “New Activity” in the new migration project

![](/Images/6.png)

6. Connect to your Oracle Database

![](/Images/7.png)

7. Connect OCI Driver from File Share Folder 

* You will need the file share link, username (which has all permissions for the drive), and the password for the username 
* If ‘readonly, archive’ error: right click the driver zip file->uncheck ‘read-only’ 

![](/Images/8.png)

8. Connect to the Azure PostgreSQL Database 

![](/Images/9.png)

9. Select the Schemas you want to use for sync 

* If nothing shows up this means you did not properly set up the names, datatypes, etc. For the Oracle and PostgreSQL tables. 

![](/Images/10.png)

10. Check the summary if all settings/configurations are correct then click "Run Migration"

![](/Images/11.png)

11. Then check the summary to see if all settings are correct and then run the activity. Once this is done you can check on the current activities in the migration.

![](/Images/12.png)
