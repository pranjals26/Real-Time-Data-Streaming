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

