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
2) we use a docker image to download all the depencies (Docker image is tri defined template) 

3)We generate the test data use python library faker.
4)We create a Nifi flow to List, Fetch and Upload data to S3 bucket, nifi pickup ups the data created by the python code. 


5) We configure the snow pipe, Snowpipe whenever we have file available in s3 bucket, we trigger the snowpipe and trigger the staging area
Create a Staging table in the snowflake. 
Stating area is basically storing the data as it is, not changing anything.

6) create a snowflake stream( track changes in the table) and Snowflaketask (automation of SQL queries) 

7) for the project we create three tables - Stating table, an Actual and historical table
Used CDC (Change Data Concept) and SDC(Slowly changing data ) 
The actual Table is SDC1( replacing the data as it is- on top of it)
historical Data - SDC2 (monitoring each and every row getting updated)


About the tools 
Apache Nifi - Data Integration tool,provides web based interface for designing, controlling and monitoring data flows. 




