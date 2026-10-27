# AWS Event-Driven Data Pipeline

## Project Overview
This project implements an event-driven, serverless data pipeline on AWS to process nested JSON order transaction data and make it available for SQL-based analytics.

When a new JSON file arrives in Amazon S3, an event notification triggers an AWS Lambda function. The function reads the input, flattens nested customer and product information, converts the result into Parquet format, and writes the processed data to a separate S3 location.

An AWS Glue Crawler then discovers the schema and updates the AWS Glue Data Catalog, allowing Amazon Athena to query the processed data.

## Architecture
Amazon S3 (Raw JSON) → S3 Event Notification → AWS Lambda (Python/Pandas) → Parquet Output in S3 → AWS Glue Crawler → AWS Glue Data Catalog → Amazon Athena

## AWS Services and Technologies
- **Amazon S3:** Stores the raw JSON input and processed Parquet output.
- **AWS Lambda:** Executes Python processing automatically when an eligible S3 event occurs.
- **AWS SDK for Python (Boto3):** Reads objects from S3 and starts the Glue Crawler.
- **Pandas:** Converts flattened order-product records into a tabular DataFrame.
- **Parquet:** Provides a columnar format suited to analytical queries.
- **AWS Glue Crawler:** Discovers the schema of the processed data.
- **AWS Glue Data Catalog:** Stores table and schema metadata.
- **Amazon Athena:** Enables SQL-based analytics on data stored in S3.

## Processing Workflow
1. A nested JSON transaction file arrives in the configured S3 input location.
2. An S3 event notification triggers the Lambda function.
3. Lambda reads and parses the JSON file using Boto3 and Python's JSON library.
4. The transformation logic flattens nested customer details and product arrays into tabular records.
5. Pandas creates a DataFrame, which is converted into Parquet using an in-memory binary buffer.
6. The Parquet output is uploaded to a separate S3 prefix.
7. Lambda starts the configured AWS Glue Crawler.
8. The crawler discovers the data schema and updates the Glue Data Catalog.
9. Amazon Athena queries the catalogued Parquet data using SQL.

## Key Design Decisions
- **Event-driven execution:** Removes the need to manually trigger each file-processing run.
- **Serverless compute:** Lambda avoids the need to manage a continuously running EC2 server for this workload.
- **IAM-based authentication:** The Lambda execution role allows AWS API access without embedding long-lived AWS access keys in the code.
- **Flattened data:** Converts nested JSON into a tabular structure suitable for SQL analytics.
- **Parquet storage:** Supports column-oriented analytical workloads.
- **Metadata discovery:** Glue Crawler and Data Catalog make the processed schema available to Athena.
- **Separation of raw and processed data:** Keeps the source JSON distinct from the transformed output.

## Operational Considerations
- Lambda's memory and execution limits must be considered for larger input files.
- S3 event notifications can be delivered more than once, so duplicate processing should be considered.
- The S3 trigger should be configured to avoid retriggering on the Parquet output prefix.
- The Glue Crawler must be available to run; repeated or overlapping invocations require handling.
- Required Python dependencies, including Pandas and a Parquet engine, must be available to the Lambda runtime.
- IAM permissions should follow least-privilege principles.

## Repository Contents
- `lambda_function.py` — Lambda processing and orchestration logic.
- `orders_etl.json` — sample nested order transaction input.

## Security
No AWS access keys or secret access keys should be committed to the repository. AWS service access is intended to use the Lambda execution role.

## Note
This repository documents the pipeline design and processing logic. Resource names, IAM permissions, S3 event configuration, Glue Crawler configuration, and Athena setup may need to be configured in your own AWS account.