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

 ![](screenshots/diagram.png)

 ➡️ Final Result
A fully functional near real-time stock analytics pipeline built using AWS services, featuring:

1. Event-driven architecture with Amazon Kinesis for real-time data ingestion
2. Lambda-based anomaly detection and stock trend evaluation
3. Low-latency storage in DynamoDB for fast access to processed data
4. Historical data archiving in Amazon S3 and querying via Athena
5. Real-time alerts via Amazon SNS (Email/SMS) for significant stock movements
6. Secure and cost-optimized design using IAM and serverless technologies

This project implements a near real-time data analytics pipeline rather than a fully real-time system. The stock data is streamed, processed by AWS Lambda, and stored in DynamoDB with a 30-second delay. The primary goal of this guide is to provide a hands-on learning experience on creating a Data Analytics pipeline while keeping AWS costs low.

# Setting Up Data Streaming with Amazon Kinesis

# STEPS TO BE PERFORMED: 
1. Create a Kinesis Data Stream

2. Set Up Your Local Python Environment

3. Write the Python Script to Stream Stock Data

4. Run the Script and Verify Data Streaming

1. Create a Kinesis Data Stream
In this step, we set up an Amazon Kinesis Data Stream to ingest real-time stock data from a stock market API like Yahoo Finance. 
Amazon Kinesis Data Streams is a managed service that collects and processes real-time data streams. Kinesis enables us to capture and process large amounts of streaming data with minimal latency.

a. Log in to the AWS Management Console.

b. Search for Kinesis in the AWS search bar and open it:

![](screenshots/kinesis.png)

c. Click Create data stream.

![](screenshots/datastream.png)

Configure the Stream

Stream name → `stock-market-stream`

Data Stream Capacity Mode → Select On-demand (this is free-tier friendly).

![](screenshots/configurestream.png)

Retention Period → Keep the default 24 hours.

Click Create data stream.

Your Kinesis Data Stream is now ready!

![](screenshots/readystream.png)

2. Set Up Your Local Python Environment
We need a local Python environment to fetch stock data from Yahoo Finance and send it to Kinesis. This step ensures that all required dependencies are installed and AWS authentication is configured.

1. Install Python (if not already installed)

Open a code editor like VSCode and ensure you have Python 3.8+ installed by running the below command in the terminal.

python --version

If not installed, download it from python.org.

2. Install Required Python Libraries

In the code editor, open the terminal and run:

pip install boto3 yfinance
![](screenshots/installboto.png)

`boto3`: AWS SDK for Python (to interact with AWS services).
`yfinance`: Fetch latest stock prices from yfinance

3. Configure AWS Credentials (if not done already)

Since we're running the script locally, AWS needs permission to send data to Kinesis.

Install AWS CLI (if not installed):

Download from AWS CLI and install it.

Configure AWS CLI (if not done previously):

Run the following command and enter your AWS credentials.

aws configure
![](screenshots/awsconfigure.png)

AWS Access Key ID → Enter your key
AWS Secret Access Key → Enter your secret
Default region name → Use the region where you created the Kinesis stream (e.g., `us-east-1`).
Default output format → Leave empty (press Enter).
Why do we need this? AWS CLI stores your credentials so boto3 can automatically authenticate your requests.

Note: If AWS CLI is not configured, refer to Section 1.1 of Project 1
3. Write the Python Script to Stream Stock Data
Now that Kinesis is ready and our local environment is set up, we need a Python script to fetch stock data and continuously send it to Kinesis.

1. Create a New Python File

Open any code editor (VS Code, PyCharm, or Notepad++).
Create a new file and save it as `stream_stock_data.py`.

2. Copy and Paste the Script Below

import boto3
import json
import time
import yfinance as yf
​
# AWS Kinesis Configuration
kinesis_client = boto3.client('kinesis', region_name='us-east-1')
STREAM_NAME = "<YOUR_DATA_STREAM_NAME>"  # Replace with your actual stream name
STOCK_SYMBOL = "AAPL"
DELAY_TIME = 30  # Time delay in seconds
​
# Function to fetch stock data
def get_stock_data(symbol):
    try:
        stock = yf.Ticker(symbol)
        data = stock.history(period="2d")  # Fetch last 2 days to get previous close
​
        if len(data) < 2:
            raise ValueError("Insufficient data to fetch previous close.")
​
        stock_data = {
            "symbol": symbol,
            "open": round(data.iloc[-1]["Open"], 2),
            "high": round(data.iloc[-1]["High"], 2),
            "low": round(data.iloc[-1]["Low"], 2),
            "price": round(data.iloc[-1]["Close"], 2),
            "previous_close": round(data.iloc[-2]["Close"], 2),
            "change": round(data.iloc[-1]["Close"] - data.iloc[-2]["Close"], 2),
            "change_percent": round(((data.iloc[-1]["Close"] - data.iloc[-2]["Close"]) / data.iloc[-2]["Close"]) * 100, 2),
            "volume": int(data.iloc[-1]["Volume"]),
            "timestamp": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime())
        }
        return stock_data
    except Exception as e:
        print(f"Error fetching stock data: {e}")
        return None
​
# Function to stream data into Kinesis
def send_to_kinesis():
    while True:
        try:
            stock_data = get_stock_data(STOCK_SYMBOL)
            if stock_data is None:
                print("Skipping this iteration due to API error.")
                time.sleep(DELAY_TIME)
                continue
​
            print(f"Sending: {stock_data}")
​
            # Send to Kinesis
            response = kinesis_client.put_record(
                StreamName=STREAM_NAME,
                Data=json.dumps(stock_data),
                PartitionKey=STOCK_SYMBOL
            )
​
            # Debugging Response
            if response["ResponseMetadata"]["HTTPStatusCode"] == 200:
                print(f"Kinesis Response: {response}")
            else:
                print(f"Error sending to Kinesis: {response}")
​
            time.sleep(DELAY_TIME)  # Send data every 30 seconds
​
        except Exception as e:
            print(f"Error: {e}")
            time.sleep(DELAY_TIME)
​
# Run the streaming function
send_to_kinesis()
​
Remember to replace <YOUR_DATA_STREAM_NAME> with the actual name of your Kinesis Data Stream. 

What This Code Does

1. Fetches Stock Data from Yahoo Finance

Retrieves the latest Open, High, Low, Close, Volume, and Previous Close for `AAPL`.
Calculates Change and Change Percentage based on previous close.

2. Formats the Data into a JSON Object

Converts stock data into a structured dictionary with a timestamp.

3. Streams Data to AWS Kinesis

Sends the JSON-encoded stock data to an AWS Kinesis Data Stream every 30 seconds.
Uses AAPL as the `PartitionKey` for better ordering.

4. Handles Errors & Retries

If API fails or data is missing, the script waits and retries instead of crashing.

5. Ensures Debugging & Logging

Prints each sent record and Kinesis response to the terminal for monitoring.
AAPL is the stock ticker symbol for Apple Inc. So basically here, we are fetching real time Apple Stock Data.
Stored Data Format in Kinesis

Each record in Kinesis will be a JSON object like this:

{
  "symbol": "AAPL",
  "open": 211.25,
  "high": 213.95,
  "low": 209.58,
  "price": 213.49,
  "previous_close": 209.68,
  "change": 3.81,
  "change_percent": 1.82,
  "volume": 60107582,
  "timestamp": "2025-03-16T09:05:09Z"
}


4. Run the Script and Verify Data Streaming
1. Run the Python Script

Open the terminal and run:

python stream_stock_data.py


Expected Output: Every 30 seconds, you should see something like:

Sending: {'symbol': 'AAPL', 'open': 211.25, 'high': 213.95, 'low': 209.58, 'price': 213.49, 'previous_close': 209.68, 'change': 3.81, 'change_percent': 1.82, 'volume': 60060200, 'timestamp': '2025-03-16T09:25:45Z'}


Kinesis Response: {'ShardId': 'shardId-000000000002', 'SequenceNumber': '49661462778447467935367742360732390732264283159369089058', 'Response

![](screenshots/runscript.png)

2. Verify Data in Kinesis

Open AWS Console → Kinesis → stock-market-stream.

Click Monitoring.

Look at Incoming Records:

![](screenshots/incomingrecords.png)

If you are interested, you can also check the incoming shards from the Data Viewer tab. See the shard name in the terminal where your records are getting stored and select Trim Horizon as your Starting position. 

![](screenshots/trimhorizon.png)

REMEMBER TO STOP YOUR PYTHON EXECUTION USING A KEYBOARD INTERRUPT- `CTRL+C` IN THE TERMINAL WHEN YOU DON’T NEED TO STREAM THE DATA.

If the script continues running, it will keep sending records to AWS Kinesis every 30 seconds indefinitely, which may lead to charges if the usage exceeds the Free Tier limits.
Congratulations! You have successfully streamed real-time stock data into Amazon Kinesis!

Processing Data with AWS Lambda
STEPS TO BE PERFORMED: 
Create a DynamoDB Table for storing Processed Stock Data.
Create S3 for storing Raw Stock Data.
Configure AWS Lambda in the AWS Console.
Test the Integration.
1. Create a DynamoDB Table for storing Processed Stock Data
In this step, we will process incoming stock data using AWS Lambda and store structured records in Amazon DynamoDB for real-time querying.

Why Do We Need a DynamoDB Table?

A NoSQL database like DynamoDB is ideal for handling real-time stock data due to its:

Fast read/write operations – Ensuring low-latency querying.
Flexible schema – Enabling easy adjustments to stock data fields.
Scalability – Handling high-volume stock transactions efficiently.

Instead of storing raw stock prices, AWS Lambda will process data before saving it to DynamoDB.
This approach allows us to:

1. Structure the data for fast retrieval.

2.Compute stock metrics like price changes and moving averages.

3. Detect anomalies in stock movements.


What Will Be Stored in DynamoDB?

The Lambda function will process the following key stock data fields before storing them in DynamoDB:
![](screenshots/datafield.png)

1. Create the DynamoDB Table in AWS Console

Open the AWS Console → Navigate to DynamoDB.

![](screenshots/dynamodb.png)

Click Create Table.
![](screenshots/newtable.png)

Table Name: `stock-market-data`.

Partition Key: `symbol` (String).

Sort Key: `timestamp` (String).

![](screenshots/tabledetails.png)

Keep the rest of the configurations as default.
Click Create Table.
2. Create S3 for storing Raw Stock Data
In this step, we will store raw stock data in Amazon S3 before processing it with AWS Lambda.



Why Use S3 for Raw Stock Data?

Long-term storage → Store raw stock data for historical analysis.
Batch processing → Useful for training machine learning models on stock trends.
Flexible querying → Query historical data using Amazon Athena.

1. Create an S3 Bucket

Open AWS Console → Navigate to S3.
Click Create bucket.
Bucket Name: Enter a unique name, e.g., `stock-market-data-bucket-33454`.
Region: Choose the same region as your Lambda function.
Keep default settings (Block Public Access enabled).
Click Create bucket.
3. Configure AWS Lambda in the AWS Console
Why Do We Need Lambda for Processing?



Instead of simply storing raw stock prices, we use AWS Lambda to:

Structure the data → Store processed data in DynamoDB for fast retrieval.
Compute stock metrics → Track price changes, percentage movements, and moving averages.
Detect anomalies → Flag stocks with sudden spikes or drops.
Raw data → Store the raw data in S3 bucket.

1. Create a New IAM Role for Lambda


Before creating the Lambda function, set up an IAM role with the necessary permissions:

Open AWS Console → Navigate to IAM.

Click Roles → Create Role.

Trusted Entity Type: Select AWS Service.

Use Case: Choose Lambda → Click Next.

![](screenshots/trustedentity.png)

Attach Policies: Add the following managed policies:
'AmazonKinesisFullAccess` (Read from Kinesis).
`AmazonDynamoDBFullAccess` (Write to DynamoDB).
`AWSLambdaBasicExecutionRole` (CloudWatch logging).`
AmazonS3FullAccess` (Write to S3).
Click Next → Name the role `Lambda_Kinesis_DynamoDB_Role` → Create Role.

![](screenshots/lambdarole.png)

2. Create a New Lambda Function

Open AWS Console → Navigate to Lambda.

![](screenshots/lambdafunction.png)

Click Create Function → Select Author from Scratch.
Function Name: ProcessStockData.
Runtime: Python 3.13.
Execution Role:  Select Use an existing role.
Choose `Lambda_Kinesis_DynamoDB_Role` (created in the previous step).

![](screenshots/lambdakinesisrole.png)

Click Create Function.

3. Add Kinesis as a Trigger

In the Function Overview tab → Click Add Trigger.

![](screenshots/kinesistrigger.png)

Select Kinesis as the source → Choose the stock-market-stream.

![](screenshots/triggerconfiguration.png)

Add a Batch size of 2. 

![](screenshots/batchsize.png)

Understand Batch size:

The batch size in Kinesis determines how many records must be collected before triggering the Lambda function.

Since, we are sending records every 30 seconds into Kinesis, the Lambda will wait for 2 records(1 min) to be triggered.

If you want to play around with the time for generating every record, make sure you take in account the Batch size too.

For eg: If the record generation happens every 2 seconds, increase the batch size to 10-50

Click Add.
4. Deploy the Lambda Code

Copy the Lambda function code from below:
import json
import boto3
import base64
from decimal import Decimal
​
# Initialize AWS Clients
dynamodb = boto3.resource("dynamodb")
s3 = boto3.client("s3")
​
# Resource Names
DYNAMO_TABLE = "stock-market-data"
S3_BUCKET = "stock-market-data-bucket-33454"
​
# Table reference
table = dynamodb.Table(DYNAMO_TABLE)
​
def lambda_handler(event, context):
    for record in event['Records']:
        try:
            # Decode base64 Kinesis data
            raw_data = base64.b64decode(record["kinesis"]["data"]).decode("utf-8")
            payload = json.loads(raw_data)
            print(f"Processing record: {payload}")
​
            # Store raw data in S3
            try:
                s3_key = f"raw-data/{payload['symbol']}/{payload['timestamp'].replace(':', '-')}.json"
                s3.put_object(
                    Bucket=S3_BUCKET,
                    Key=s3_key,
                    Body=json.dumps(payload),
                    ContentType='application/json'
                )
                print(f"Raw data saved to S3: {s3_key}")
            except Exception as s3_error:
                print(f"Failed to save raw data to S3: {s3_error}")
​
            # Compute stock metrics
            price_change = round(payload["price"] - payload["previous_close"], 2)
            price_change_percent = round((price_change / payload["previous_close"]) * 100, 2)
            is_anomaly = "Yes" if abs(price_change_percent) > 5 else "No"
            moving_average = (payload["open"] + payload["high"] + payload["low"] + payload["price"]) / 4
​
            # Structured data for DynamoDB
            processed_data = {
                "symbol": payload["symbol"],
                "timestamp": payload["timestamp"],
                "open": Decimal(str(payload["open"])),
                "high": Decimal(str(payload["high"])),
                "low": Decimal(str(payload["low"])),
                "price": Decimal(str(payload["price"])),
                "previous_close": Decimal(str(payload["previous_close"])),
                "change": Decimal(str(price_change)),
                "change_percent": Decimal(str(price_change_percent)),
                "volume": int(payload["volume"]),
                "moving_average": Decimal(str(moving_average)),
                "anomaly": is_anomaly
            }
​
            # Store in DynamoDB
            table.put_item(Item=processed_data)
            print(f"Stored in DynamoDB: {processed_data}")
​
        except Exception as e:
            print(f"Error processing record: {e}")
​
    return {"statusCode": 200, "body": "Processing Complete"}

    Paste it into the AWS Lambda Code Editor.
    Click Deploy.
4. Test the Integration

1. Verify Data in DynamoDB

Open AWS Console → Navigate to DynamoDB.
Click on stock-market-data → Explore Table Items.
You should see processed stock data records appearing in real-time.

![](screenshots/dynamodbstock.png)

2. Verify Data in S3

Open AWS Console → Navigate to S3.
You should see the raw stock data records appearing in real-time.

![](screenshots/s3data.png)

REMEMBER TO START THE PYTHON SCRIPT BEFORE TESTING THE INTEGRATION AND STOPPING IT AFTER STORING AROUND 15-20 RECORDS IN THE DYNAMO DB TABLE. 

Now we have created a DynamoDB Table, developed an AWS Lambda function to process stock data from Kinesis, and validated the integration.


In the next steps, we will query the historical data in S3 using Amazon Athena.

# Query Historical Stock Data using Amazon Athena

# STEPS TO BE PERFORMED: 
Create a Glue Catalog Table for Athena.
Create an S3 Bucket to store Query Results.
Query Data Using Athena.


1. Create a Glue Catalog Table for Athena
Now that raw stock data is stored in Amazon S3, we will use Amazon Athena to query and analyze it efficiently.


Why Use Amazon Athena?

Serverless SQL Queries: Query S3 data without setting up a database.
Cost-Effective: Pay only for the data scanned.
Scalable: Handles large datasets with ease.
Amazon Athena requires a Glue Data Catalog to define the schema of S3 data.

1. Open AWS Glue Console

Navigate to AWS Console → Open AWS Glue.

![](screenshots/awsglue.png)

Click Data Catalog → Databases.

![](screenshots/database.png)

Click Create Database.
Database Name: `stock_data_db`
Click Create.

![](screenshots/database2.png)

