# Real-Time Data Streaming using Apache Nifi, AWS, Snowpipe, Stream & Task

This repository contains the code and configuration for a robust data ingestion and processing pipeline utilizing Apache NiFi, Docker, AWS EC2 and S3 services, and Snowflake. The pipeline is designed to simulate data generation, orchestrate data flows, and enable incremental data loading into Snowflake for analytics purposes.

##  Overview
The pipeline integrates several technologies:

- Apache NiFi: A data integration tool to manage, automate, and monitor data flows.
- Docker: To containerize and manage the Apache NiFi and Jupyter notebook environments.
- AWS EC2: Hosting the Docker containers.
- AWS S3: Object storage service used as an intermediate data store.
- Snowflake: A cloud data platform for large-scale data warehousing and analytics.

## Architecture

![Architecture](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Real%20Time%20Data%20Streaming.jpg)

The architecture diagram illustrates the flow of data from generation to final storage. It involves generating test data with Python (Faker library), processing and transferring the data through Apache NiFi, storing it temporarily in AWS S3, and finally loading it into Snowflake using Snowpipe.

## Setup Instructions
### Prerequisites
- AWS & Snowflake account.
- Experinece with Python and SQL 
- Knowledge about CI/CD
- Basic understanding of ETL processes

## Configuration
- EC2 Setup: Launch a t2.xlarge EC2 instance and configure security groups to allow HTTP and SSH access. Attach the created key pair for SSH access.
    - Upload the docker-compose.yml  file on the ec2 instance <br />

![Ec2 Setup](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/Ec2connect.png)

- Docker Setup: Utilize a pre-defined Docker image to deploy Apache NiFi and Jupyter on the EC2 instance. Run the following commands to start services:
  - docker-compose up # Starts the containers and installs dependencies
  - docker ps # Lists running containers
 
Once the depencies are installed, check the list of running container

![ps](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/ps.png)

- Data Generation: Create test data using the Python library Faker in a Jupyter notebook. (access on - PublicIPAddress:4888)

![Python](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/Python%20code%20for%20data%20generation.png)

- Apache NiFi Flow: Set up NiFi to List, Fetch, and Upload data to the S3 bucket. NiFi picks up the data created by the Python script.
![Apache Nifi](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/nififlow.png)

- After the Nifi Flow, S3 bucket is upload with datasets. 

![s3](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/S3%20OBJECTS.png)

- Snowflake Setup : Create three tables
  - Staging Table: Stores raw data without any transformations.
  - Actual Table: Implements SCD Type 1, overwriting data directly.
  - Historical Table: Implements SCD Type 2, tracking each row's updates.

![Table](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/TBALE%20CREATION.png)

- Snowpipe Configuration: Configure Snowpipe to load data into the staging table whenever files are dropped into the S3 bucket. Set up event notifications in S3 to trigger Snowpipe.
    - Under properties create event notification -> give prefix as streamdata/ (Folder name in the bucket) -> Enter Arn for SQS queue Snowpipe created successfully. 

- Stream & Task : Snowflake stream, and a Snowflake task in Snowflake. Use the Snowflake MERGE command for incremental loads and handle updates to the dataset.
- Create a Snowflake stored procedure using JavaScript to automate the merging process. Schedule a task to run every minute.

```SQL

CREATE OR REPLACE TASK tsk_scd_raw 
WAREHOUSE = COMPUTE_WH 
SCHEDULE = '1 minute'
ERROR_ON_NONDETERMINISTIC_MERGE=FALSE 
AS CALL pdr_scd_demo();

SHOW TASKS;

ALTER TASK tsk_scd_raw RESUME;

```



