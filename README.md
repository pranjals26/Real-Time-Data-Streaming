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
    - Allow ports 4000 - 38888
    - Connect to ec2 via ssh
    - Upload the docker-compose.yml  file on the ec2 instance <br />

```
# connect to EC2
ssh -i snowflake-project.pem ec2-user@ec2-54-203-235-65.us-west-2.compute.amazonaws.com

# Copy files to EC2
scp -r -i snowflake-project.pem docker-exp ec2-user@ec2-13-232-189-6.ap-south-1.compute.amazonaws.com:/home/ec2-user/docker_exp


- Commands to install Docker
sudo yum update -y
sudo yum install docker
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo gpasswd -a $USER docker
newgrp docker
sudo yum install python-pip
sudo pip install docker-compose

```


![Ec2 Setup](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/Ec2connect.png)

- Docker Setup: Utilize a pre-defined Docker image to deploy Apache NiFi and Jupyter on the EC2 instance. Run the following commands to start services:
```
#Start Docker: 
sudo systemctl start docker
#Stop Docker:
sudo systemctl stop docker

# Starts the containers and installs dependencies
docker-compose up

# Lists running containers
docker ps 
 ```
Once the dependencies are installed, check the list of running container

![ps](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/ps.png)

- Data Generation: Create test data using the Python library Faker in a Jupyter notebook. [Jupyter Lab at http://ip_address:4888/lab?] (use faker.ipynb)

![Python](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/Python%20code%20for%20data%20generation.png)

- Apache NiFi Flow: Set up NiFi to List, Fetch, and Upload data to the S3 bucket. NiFi picks up the data created by the Python script. [http://ip_address:2080/nifi/]
![Apache Nifi](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/nififlow.png)

- After the Nifi Flow, the S3 bucket is uploaded with datasets. 

![s3](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/S3%20OBJECTS.png)

- Snowflake Setup: Create three tables (use Table Creation.sql)
  - Staging Table: Stores raw data without any transformations.
  - Actual Table: Implements SCD Type 1, overwriting data directly.
  - Historical Table: Implements SCD Type 2, tracking each row's updates.

![Table](https://github.com/pranjals26/Real-Time-Data-Streaming/blob/main/Workflow/TBALE%20CREATION.png)

- Snowpipe Configuration: Configure Snowpipe to load data into the staging table whenever files are dropped into the S3 bucket. Set up event notifications in S3 to trigger Snowpipe.
    - Under properties create event notification -> give prefix as streamdata/ (Folder name in the bucket) -> Enter Arn for SQS queue Snowpipe created successfully. 
```SQL
// Creating external stage (create your own bucket)
CREATE OR REPLACE STAGE SCD_DEMO.SCD2.customer_ext_stage
    url='s3://dw-snowflake-course-darshil/stream_data'
    credentials=(aws_key_id='' aws_secret_key='');
   

CREATE OR REPLACE FILE FORMAT SCD_DEMO.SCD2.CSV
TYPE = CSV,
FIELD_DELIMITER = ","
SKIP_HEADER = 1;

SHOW STAGES;
LIST @customer_ext_stage;


CREATE OR REPLACE PIPE customer_s3_pipe
  auto_ingest = true
  AS
  COPY INTO customer_raw
  FROM @customer_ext_stage
  FILE_FORMAT = CSV
  ;

show pipes;
select SYSTEM$PIPE_STATUS('customer_s3_pipe');

SELECT count(*) FROM customer_raw limit 10;

TRUNCATE  customer_raw;
```

- Create a Snowflake stored procedure using JavaScript to automate the merging process. Schedule a task to run every minute.

```sql
CREATE OR REPLACE PROCEDURE pdr_scd_demo()
returns string not null
language javascript
as
    $$
      var cmd = `
                 merge into customer c 
                 using customer_raw cr
                    on  c.customer_id = cr.customer_id
                 when matched and c.customer_id <> cr.customer_id or
                                  c.first_name  <> cr.first_name  or
                                  c.last_name   <> cr.last_name   or
                                  c.email       <> cr.email       or
                                  c.street      <> cr.street      or
                                  c.city        <> cr.city        or
                                  c.state       <> cr.state       or
                                  c.country     <> cr.country then update
                     set c.customer_id = cr.customer_id
                         ,c.first_name  = cr.first_name 
                         ,c.last_name   = cr.last_name  
                         ,c.email       = cr.email      
                         ,c.street      = cr.street     
                         ,c.city        = cr.city       
                         ,c.state       = cr.state      
                         ,c.country     = cr.country  
                         ,update_timestamp = current_timestamp()
                 when not matched then insert
                            (c.customer_id,c.first_name,c.last_name,c.email,c.street,c.city,c.state,c.country)
                     values (cr.customer_id,cr.first_name,cr.last_name,cr.email,cr.street,cr.city,cr.state,cr.country);
      `
      var cmd1 = "truncate table SCD_DEMO.SCD2.customer_raw;"
      var sql = snowflake.createStatement({sqlText: cmd});
      var sql1 = snowflake.createStatement({sqlText: cmd1});
      var result = sql.execute();
      var result1 = sql1.execute();
    return cmd+'\n'+cmd1;
    $$;
call pdr_scd_demo();

```



- Stream & Task: Snowflake stream and a Snowflake task in Snowflake. Use the Snowflake MERGE command for incremental loads and handle updates to the dataset.

```sql

#Set up TASKADMIN role
use role securityadmin;
create or replace role taskadmin;
#Set the active role to ACCOUNTADMIN before granting the EXECUTE TASK privilege to TASKADMIN
use role accountadmin;
grant execute task on account to role taskadmin;

```

## Usage
To initiate the pipeline, run the Python code to generate data. The NiFi flow will begin processing this data, transferring it to S3, and from there, Snowpipe will load it into the staging table in Snowflake. The Snowflake stream and tasks will manage the incremental loading and merging of this data into the final tables for analysis.

## Contributing
Feel free to fork this repository and submit pull requests to contribute to the project. For major changes, please open an issue first to discuss what you would like to change.





