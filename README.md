# Stock-Market-Real-Time-Data-Analytics-Pipeline-on-AWS
This project builds a real-time stock market data analytics pipeline using AWS, leveraging event-driven architecture and serverless technologies. The architecture ingests, processes, stores, and analyzes stock market data in real-time while minimizing costs.

# Key tasks include:

Streaming real-time stock data from sources like yfinance using Amazon Kinesis Data Streams.
Processing data and detecting anomalies with AWS Lambda.
Storing processed stock data in Amazon DynamoDB for low-latency querying.
Storing raw stock data in Amazon S3 for long-term analytics.
Querying historical data using Amazon Athena.
Sending real-time stock trend alerts using AWS Lambda & Amazon SNS (Email/SMS).

In this lab, you'll learn how to integrate various AWS services to create a streamlined CI/CD workflow, reducing manual intervention and enabling faster, reliable application deployment.



# Steps to be performed 👩‍💻
We'll go through the following steps in the next few lessons.

1. Setting Up Data Streaming with Amazon Kinesis

2. Processing Data with AWS Lambda

3. Query Historical Stock Data using Amazon Athena

4. Stock Trend Alerts using SNS


 # Services Used 🛠
 Amazon Kinesis Data Streams – Ingests stock data in real-time.
 AWS Lambda – Processes stock data and detects stock trends.
 Amazon DynamoDB – Stores structured stock data for quick lookups.
 Amazon S3 – Stores raw stock data for historical analysis.
 Amazon Athena – Queries stock data directly from S3.
 Amazon SNS – Sends stock trend alerts via Email/SMS.
 IAM Roles & Policies – Manages permissions securely.

 # Estimated Time & Cost ⚙️
This project is estimated to take about 2-3 hours
Cost: ~$1 to ~$2


➡️ Diagram
This is the architectural diagram for the project:
