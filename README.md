# Sample Data Pipeline

Data Pipeline code using Kafka, Spark and Airflow tried on Mac(local setup)

## Table of Contents
- [Pre-requisite](#pre-requisite)
- [Sample 1: Airflow to trigger Spark Job](#sample-1-airflow-to-trigger-spark-job)
- [Sample 2: Airflow to trigger Spark Job with AWS S3](#sample-2-airflow-to-trigger-spark-job-with-aws-s3)
- [References](#references)

## Pre-requisite

### Kafka setup
- Install Kafka (used Kafka 2.6.0 for this sample)
- Start Zookeeper and kafka server
- Create a topic named "patient-data"
```bash
kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic patient-data
```

### Spark setup
- Install Spark (used 3.0.1 for this sample)

### AWS S3 setup
- AWS account and S3 bucket
- Access Key and Secret key for the AWS account

### Airflow setup
- Install Airflow

## Sample 1: Airflow to trigger Spark Job

### 1. Ingest data to kafka topic
```bash
kafka-console-producer --broker-list localhost:9092 --topic patient-data
> This is a test kafka message
```

### 2. Change Spark job and test
- Change Spark master settings in fetch_data.py based on Spark installation
- Start spark:
```bash
./start-all.sh
```
- Run spark job(fetch_data.py) manually:
```bash
spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.0.1 fetch_data.py
```
- Should display count of message in Kafka topic

### 3. Use Apache Airflow to automate scheduling
- Update spark_dag.py based on file location
- Place spark_dag.py in "dags" folder under Airflow home directory
- Start Airflow:
```bash
airflow initdb
airflow webserver
```
- Start Airflow scheduler:
```bash
airflow scheduler
```
- Go to Airflow UI and trigger "spark_job_dag"

## Sample 2: Airflow to trigger Spark Job with AWS S3

### 1. Ingest data to kafka topic
```bash
kafka-console-producer --broker-list localhost:9092 --topic patient-data
> 10 20 30 40
```

### 2. Change Spark job and test
- Change data_pipeline.py to include correct ACCESS_KEY, SECRET_KEY, S3_BUCKET
- Start spark:
```bash
./start-all.sh
```
- Run spark job(data_pipeline.py) manually:
```bash
/usr/local/Cellar/apache-spark/3.0.1/libexec/bin/spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.0.1,com.amazonaws:aws-java-sdk:1.11.633,org.apache.hadoop:hadoop-aws:3.2.0 /Users/philips/Documents/Study_code/Spark/spark-samples/spark_kafka_s3.py
```
Note: Command may change based on folder structure, also aws-java-sdk & hadoop-aws versions may be different
- If successful, it will save parquet data in S3 bucket

### 3. Use Apache Airflow to automate scheduling
- Update save_data_S3.py based on file location
- Place save_data_S3.py in "dags" folder under Airflow home directory
- Start Airflow:
```bash
airflow initdb
airflow webserver
```
- Start Airflow scheduler:
```bash
airflow scheduler
```
- Go to Airflow UI and trigger "S3_data_save_dag"

## References
- [Spark Structured Streaming Kafka Integration](https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html)
