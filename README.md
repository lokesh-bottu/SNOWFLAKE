# SNOWFLAKE

# Loading Data
1) Bulk Loading:

    Bulk loading is ideal for handling large volumes of data and involves utilizing warehouses and loading data from stages. The most common methods include:

    Copy Command: A widely used method for bulk loading data into Snowflake from external sources.
    Key Features:

    Uses Warehouses: Bulk loading operations utilize warehouses for processing power.
    Loading from Stages: Data is loaded into Snowflake from staging areas.
    Transformations Possible: Transformations can be applied during the bulk loading process.

2) Continuous Loading:

    Designed for loading small amounts of data continuously, this feature automatically loads data as it becomes available in designated stages, providing real-time access to the latest data.

    Key Features:

    Automatic Loading: Data is loaded into Snowflake automatically as it's added to designated staging areas.
    Latest Results for Analysis: Continuous loading ensures access to the most up-to-date data for analysis.
    Snowpipe (Serverless Feature): A serverless feature enabling continuous data loading without additional infrastructure management.

## Stages

- External Cloud provider

- External stages are data base objects that are created in database.
- stages are locations where you can store and manage files, such as CSV or JSON files, that you want to load into or unload from the Snowflake database.
```html
Create or replace stage db.schema.stage_name
url = "s3://bucketsnowflake3"
credentials = (aws_key_id = 'key_id' aws_secret_key = 'secret_key')

```
If we dont give the credentials then it is publicly accessible.
## How to load the data from Stage?

```html
copy into <db.schema.tablename>
    from <@stagename>
    file_format = (type = "csv" field_delimiter = "," skip_header = 1)
```

# Snow pipe
- Snowpipe automatically loads the file when the new  file is added to the bucket.
- So when we want the data to be loaded immediately from the specified bucket then we use snow pipe.
- This uses server less feature it doesnt need any warehouses.
- We can set an event notification when the new file is added.

Steps to create a snowpipe
1) Create a stage
- This contains the connection to the external stage that we want to copy the data from.
2) create a snow pipe

3) Set up the event notification

```html
create or replace pipe db.schema.pipe_name
auto_ingest = True
as 
copy into db.schema.table_name
from db.schema.stage_name

after this in the s3 bucket in properties give the 
suffix and prefix
select the event type 
channel_code of the pipe
```

# Time Travel
- We can use Time Travel to restore data that we have updated or deleted accidentally.
- With this we can travel back in time and see the data exactly before we have updated the data at a particular time.
- We can do time travel for 90 days.

```html
select * from <table_name> at (offset => -60*1)
Here we indicate the time in seconds

select * from <table_name> before (timestamp => <time_stamp>::timestamp)
we do time travel using timestamp 


select * from <table_name> before(statement=>'Query_id')
Using the query Id we can do time travel
```


# Fail Safe
- We use the fail safe in order to protect our data just like the time travel.

- In case of our data is lost or corrupted then we use file safe.
- In case our timetravel period is over i.e 90 days then the fail safe period starts. This lasts for 7 days.But this can only be done for the permanent tables.
- Like time travel we cant go back using queries we need to contact snowflake and we also need to pay for the storage.

# Table Types

- We have 3 types of tables.
    ### 1) Permanent Tables
    - This is created using the create table command.
    - Only the permanent tables falls in the fail safe.

    ### 2) Transient Tables
    - This is created using the below syntax
    - create transient table table_name
    - In this the time travel period is 1 day and fail safe is not allowed.
    ### 3) Temporary Tables
    - This contains all the properties of transient table.
    - But this exists only till the session ends.
    - Once the session is closed we wont be able to see the data
    - Max time of the temp tables is 1 day.
    - Another user will not be able to see the data.


# Zero copy cloning
- This allows us to copy of databases, schemas, tables etc
- with just 1 single command we can create a clone of original object.
- We will have a copy of source object, but here it will refer to the metadata of the original table.
- We can clone complete table without the extra storage.
- This is completely independent of source object and if changes are made to the cloned that will be not reflected in the source object.

``` HTML 
CREATE TABLE <TABLENAME>
    CLONE <SOURCE_OBJECTNAME>
```


# Scheduling the Tasks
- Task is an object that stores a certain sql statement which will be executed in a particular time which we mention.

- We can also create stand alone tasks with just 1 sql statement we can create dependencies in such a way a particular task will be executed 1 after other.
- Single parent task can have multiple child tasks but not vice versa.

- Below is the syntax to create a task

```html
create or replace task <task_name>
    warehouse = <warehouse_name>
    schedule = '1 minute'
    as 
    insert into <table_name> values();
```
Here the task will be executed for every 1 minute.


# Data Sharing

- If we want to share the data then it is very complex to take a physical copy of that.
- Using snowflake we can share the data without even actually creating the copy of data and even for the non snowflake users.
- Snowflake can share the data always upto date and immediately upto date.
- The owner can have the complete control over the shared data and the user needs to use his own compute resources and he will only have read privileges over the shared data.
- Using reader account the non snowflake users can access the data.


# Streams
- Streams are objects that records the data if there are any changes and what changes are made to the specific table.
- We use streams when we want to copy the data to the other table with the changes.
- We create an object on specific table then the operations like insert,update and delete are captured.
- The data we have in the object is actually retrieved from the source table only so this uses less extra space.
- so the momene when we insert the records that are present in the object to the other table the object gets freed up.

Below is the syntax to create a stream


```html
create stream <stream_name>
    on table <table_name>
```



# Dynamic Data Masking

- We use it for security purposes.
- If we have a table which needs to be protected then we can use dynamic masking for that also
like if we want to hide the data and we can cover the data with the mask as a result the data would be invisible.
- We can specify partular columns and customize which needs to be masked.

Below is the syntax to create a mask

```html

create or replace masking policy phone
as (val varchar) return varchar -> 
case when current_role() in ('role1','role2') then val 
else '####"
end;
```



