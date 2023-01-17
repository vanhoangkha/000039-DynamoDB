---
title : "Amazon DynamoDB Streams and AWS Lambda"
date : "`r Sys.Date()`"
weight : 10
chapter : false
pre : " <b> 10. </b> "
---

The combination of DynamoDB Stream with AWS Lambda creates many powerful new architectural patterns. In this exercise, you will copy items from a DynamoDB table to another table using DynamoDB Stream and AWS Lambda. DynamoDB Stream captures a sequence of item-level changes and reorders it chronologically and stores it in a 24-hour log. Any application can access and view this log data with actual modification time. DynamoDB stream is used in the following cases:

A game with a distributed database topology, hosted across many different regions. Each region is always synchronized with changes occurring in remote regions. (In fact, DynamoDB's global data synchronization mechanism is based on DynamoDB Stream)

The client adds data to the DynamoDB table. This event will call AWS Lambda to copy data to a separate DynamoDB table for persistent data storage

We will reuse the logfile table we created in exercise #1 to enable DynamoDB Stream for it. Whenever there is a change in the logfile table, the change information is immediately put into the stream. Next we will attach a Lambda function to the stream with the purpose of querying the changes on the logfile table, and then recording those updates to the newly created table named logfile_replica. The following diagram shows the steps for this exercise:


![DynamoDB](/images/1-Introduce/0008-Diagram-DynamoDB-Tables.png)

1. Create a Copy Table

The Replica table named **logfile_replica** will have the same properties as the original


```
aws dynamodb create-table --table-name logfile_replica \
--attribute-definitions AttributeName=PK,AttributeType=S AttributeName=GSI_1_PK,AttributeType=S \
--key-schema AttributeName=PK,KeyType=HASH --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=5 \
--tags Key=workshop-design-patterns,Value=targeted-for-cleanup \
--global-secondary-indexes "IndexName=GSI_1,KeySchema=[{AttributeName=GSI_1_PK,KeyType=HASH}],\

Projection={ProjectionType=INCLUDE,NonKeyAttributes=['bytessent', 'requestid', 'host']},\
ProvisionedThroughput={ReadCapacityUnits=10,WriteCapacityUnits=5}"
```

The table is also linked to the Global Secondary Index. The detailed information is listed as follows:

- Key schema: HASH (partition key)

- Table read capacity units (RCUs) = 10

- Table write capacity units (WCUs) = 5

- Global secondary index: GSI_1 (10 RCUs, 5 WCUs) - Allows querying based on the host's IP address.

| Attribute Name (Type)  | Special Attribute?  |Attribute Use Case   |  Sample Attribute Value |
|---|---|---|---|
| PK (STRING)  | 	Partition key  | Holds the request id | request#104009 | 
|GSI_1_PK (STRING)  |	GSI 1 partition key   |  	Host |  	host#66.249.67.3 | 

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/0001-dynamodbstreamandlambda.png)

2. Run the following command, wait until the status of the table changes to ACTIVE

```
aws dynamodb wait table-exists --table-name logfile_replica
```

![Amazon DynamoDB Streams and AWS Lambda](/images/9-DynamoDBStreamsandLambda/0002-dynamodbstreamandlambda.png)

3. Create **logfile_replica** table successfully


![Amazon DynamoDB Streams and AWS Lambda](/images/9-DynamoDBStreamsandLambda/0003-dynamodbstreamandlambda.png)

4. Check the IAM Policy assigned to AWS Lambda

In the setup we prepared an IAM Role named DDBReplicationRole that was assigned to the AWS Lambda function. This IAM Role grants the AWS Lambda function the necessary permissions to perform data replication. Specifically, those permissions include:

AWS Lambda function has the ability to call DynamoDB Stream and get record changes from Stream


```
{
    "Action": [
        "dynamodb:DescribeStream",
        "dynamodb:GetRecords",
        "dynamodb:GetShardIterator",
        "dynamodb:ListStreams"
    ],
    "Resource": [
        "*"
    ],
    "Effect": "Allow"
}
```
AWS Lambda function can add or remove items from logfile_replica table

```
{
    "Action": [
        "dynamodb:DeleteItem",
        "dynamodb:PutItem"
    ],
    "Resources": [
        "*"
    ],
    "Effect": "Allow"
}
```

5. Create Lambda function

The AWS Lambda function will be tied to the DynamoDB Stream of the logfile table to replicate the original table's item add or delete action to the logfile_replica replica table. The content of the Lambda function is in the ddbreplica_lambda.py file, which you can view with the vim or less command.
Compress the content of the script and then upload it to the AWS Lambda service to generate the Lambda function.

```
zip ddbreplica_lambda.zip ddbreplica_lambda.py lab_config.py
```

![Amazon DynamoDB Streams and AWS Lambda](/images/9-DynamoDBStreamsandLambda/0004-dynamodbstreamandlambda.png)

6. Get the ARN information of the IAM Role code to be assigned to the Lambda function.

```
cat ~/workshop/ddb-replication-role-arn.txt
```

The RNA information should look something like this:

```
arn:aws:iam::<ACCOUNTID>:role/XXXXX-DDBReplicationRole-XXXXXXXXXXX
```

![Amazon DynamoDB Streams and AWS Lambda](/images/9-DynamoDBStreamsandLambda/0005-dynamodbstreamandlambda.png)

7. Copy the ARN obtained above, replace YOUR_ARN_HERE and run the command to create Lambda function

```
aws lambda create-function \
--function-name ddbreplica_lambda --zip-file fileb://ddbreplica_lambda.zip \
--handler ddbreplica_lambda.lambda_handler --timeout 60 --runtime python3.7 \
--description "Sample lambda function for dynamodb streams" \
--role YOUR_ARN_HERE
```

![Amazon DynamoDB Streams and AWS Lambda](/images/9-DynamoDBStreamsandLambda/0006-dynamodbstreamandlambda.png)

8. AWS Lambda Function has been initialized successfully

![Amazon DynamoDB Streams and AWS Lambda](/images/9-DynamoDBStreamsandLambda/0007-dynamodbstreamandlambda.png)

9. Enable DynamoDB Stream

Once streaming is enabled, we can choose whether DynamoDB will copy new items, or old items, or all, or just adjust the partition key and sort key.

To enable DynamoDB Stream for the logfile table with NEW_IMAGE option (ie all new items)

```
aws dynamodb update-table --table-name 'logfile' --stream-specification StreamEnabled=true,StreamViewType=NEW_IMAGE
```

![Amazon DynamoDB Streams and AWS Lambda](/images/9-DynamoDBStreamsandLambda/0008-dynamodbstreamandlambda.png)

10. Get DynamoDB Stream Information

```
aws dynamodb describe-table --table-name 'logfile' --query 'Table.LatestStreamArn' --output text
```

The output information is similar to the following

```
arn:aws:dynamodb:<REGION>:<ACCOUNTID>:table/logfile/stream/2018-10-27T02:15:46.245
```

![Amazon DynamoDB Streams and AWS Lambda](/images/9-DynamoDBStreamsandLambda/0009-dynamodbstreamandlambda.png)

11. Connect Stream source with Lambda function

So the original table has DynamoDB Stream enabled and the Lambda function has also been created. Next step, we will connect Stream with Lambda function using following command

```
aws lambda create-event-source-mapping \
--function-name ddbreplica_lambda --enabled --batch-size 100 --starting-position TRIM_HORIZON \
--event-source-arn YOUR_STREAM_ARN_HERE
```

Expected result will be similar below


```
{
    "UUID": "0dcede66-709c-4073-a628-724d01b92095",
    "BatchSize": 100,
    "MaximumBatchingWindowInSeconds": 0,
    "ParallelizationFactor": 1,
    "EventSourceArn": "arn:aws:dynamodb:<REGION>:<ACCOUNTID>:table/logfile/stream/2021-12-31T00:00:00.000",
    "FunctionArn": "arn:aws:lambda:<REGION>:<ACCOUNTID>:function:ddbreplica_lambda",
    "LastModified": 1663286115.972,
    "LastProcessingResult": "No records processed",
    "State": "Creating",
    "StateTransitionReason": "User action",
    "DestinationConfig": {
        "OnFailure": {}
    },
    "MaximumRecordAgeInSeconds": -1,
    "BisectBatchOnFunctionError": false,
    "MaximumRetryAttempts": -1
}
```

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/00010-dynamodbstreamandlambda.png)

12. Push data to logfile table and check copy progress to logfile_replica table

Run the python command below to load more items into the logfile table. The new records will be injected into the DynamoDB Stream, leading to the activation of the Lambda function to write the new item information to the logfile_replica table

```
python load_logfile.py logfile ./data/logfile_stream.csv
```

The result is similar to the following

```
RowCount: 2000, Total seconds: 15.808809518814087

```

![Amazon DynamoDB Streams and AWS Lambda](/images/9-DynamoDBStreamsandLambda/00011-dynamodbstreamandlambda.png)

13. To check whether the copy to the logfile_replica table is successful, perform a scan of the logfile_replica table data.

```
aws dynamodb scan --table-name 'logfile_replica' --max-items 2 --output text
```

The results are similar to the following:


```
None    723     723
BYTESSENT       2969
DATE    2009-07-21
HOST    64.233.172.17
HOUROFDAY       8
METHOD  GET
REQUESTID       4666
RESPONSECODE    200
TIMEZONE        GMT-0700
URL     /gwidgets/alexa.xml
USERAGENT       Mozilla/5.0 (compatible) Feedfetcher-Google; (+http://www.google.com/feedfetcher.html)
BYTESSENT       1160
DATE    2009-07-21
HOST    64.233.172.17
HOUROFDAY       6
METHOD  GET
REQUESTID       4119
RESPONSECODE    200
TIMEZONE        GMT-0700
URL     /gadgets/adpowers/AlexaRank/ALL_ALL.xml
USERAGENT       Mozilla/5.0 (compatible) Feedfetcher-Google; (+http://www.google.com/feedfetcher.html)
NEXTTOKEN       eyJFeGNsdXNpdmVTdGFydEtleSI6IG51bGwsICJib3RvX3RydW5jYXRlX2Ftb3VudCI6IDJ9
```

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/00012-dynamodbstreamandlambda.png)
