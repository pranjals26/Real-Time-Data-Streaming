# Real-Time-Data-Streaming
Real-Time Data Streaming using Apache Nifi, AWS, Snowpipe, Stream & Task


## Project Overview 

![cloud Architecture](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Real%20Time%20Data%20Streaming.jpg) 

Handling real-time data using Snowflake, 
Deploy Apache Nifi on the EC2 instance 
We use docker file and install all the dependecies for nifi
Create Docker image


To generate realtime data we use the Python library Faker, The generated is updated in the S3 BUCKET by nifi

Once the data is uploaded in te S3 we willl configure the snow pipe
trigger the snow pipe and load the data in staging area(Storing the raw data) 

create snowfalke strwam and snowflake task 

We create three tables 
Staging table for raw data 
Table 2 - Actual 
Table 3 Historical 

Apache Nifi - data integration tool 

Dockerfile , Compose, Image, container, 


Tasks and Stream 
task - Schedule SQL Statement 

1) We use EC2 machine and deploy apache nifi on it, we use docker to deploy Apache and Jupyter on Ec2
  - I have used t2.xlarge instance for the project, Created a key pair 
2) we use a docker image to download all the dependencies (Docker image is tri defined template) 
    Docker Compose up #download all the dependencies. 
    docker ps #lists the containers that are running on your host

3)We generate the test data using the Python library Faker.


4)We create a Nifi flow to List, Fetch and Upload data to S3 bucket, Nifi pickup ups the data created by the python code. 

5) We configure the snow pipe, Snowpipe whenever we have the file in the s3 bucket. The snow pipe  triggers the staging area 
for snowpipe - go to s3 bucket, Stream data 
under properties create event notification -> give prefix as streamdata/ -> Enter Arn for SQS queue Snowpipe created successfully. 

Create a Staging table in the snowflake. 
Stating area is basically storing the data as it is, not changing anything.

once the snowpipe is created, run the python code run, the pipeline will start where dat goes from nifi to s3 and from s3 to the staging table.

6) create a snowflake stream( track changes in the table) and Snowflaketask (automation of SQL queries) 

used merge snowflake command to (Incremental load) if there are no updates and modification we don’t update the dataset 

We created a stored procedure in the snowflake using JavaScript to automate the process of merging data from customer_raw file to customer. 

Further we create task to automate the code in every one minute

create or replace task tsk_scd_raw warehouse = COMPUTE_WH schedule = '1 minute'
ERROR_ON_NONDETERMINISTIC_MERGE=FALSE
as
call pdr_scd_demo();
show tasks;
alter task tsk_scd_raw resume; --suspend
show tasks;


7) for the project we create three tables - Stating table, an Actual and historical table
Used CDC (Change Data Concept) and SDC(Slowly changing data ) 
The actual Table is SDC1( replacing the data as it is- on top of it)
historical Data - SDC2 (monitoring each and every row getting updated)


About the tools 
Apache Nifi - Data Integration tool,provides web based interface for designing, controlling and monitoring data flows. 




