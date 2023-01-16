---
title : "Amazon DynamoDB Streams và AWS Lambda"
date :  "`r Sys.Date()`" 
weight : 10
chapter : false
pre : " <b> 10. </b> "
---

Sư kết hợp giữa DynamoDB Stream với AWS Lambda tạo ra nhiều mẫu kiến trúc mới vô củng mạnh mẽ. Trong bài thực hành này, bạn sẽ sao chép các item từ bảng DynamoDB sang một bảng khác sử dụng DynamoDB Stream và AWS Lambda. DynamoDB Stream chụp lại một chuỗi sự thay đổi ở mức item và xếp lại theo thứ tự thời gian rồi lưu vào một nhật ký có thời hạn lưu trữ là 24h. Mọi ứng dụng đều có thể truy cập và xem dữ liệu nhật kí này với thời gian chỉnh sửa so với thực tế là gân thực. DynamoDB stream được sử dụng trong các trường hợp sau đây:

Một trò chơi có cấu trúc liên kết cơ sở dữ liệu phân tán, lưu trữ trên nhiều region khác nhau. Mỗi region luôn được đồng bộ những sự thay đổi xảy ra ở những region ở xa. (Thực tế cơ chế đồng bộ dữ liệu toàn cầu của DynamoDB là dựa trên DynamoDB Stream)

Khách hàng thêm dữ liệu vào bảng DynamoDB. Sự kiện này sẽ gọi tới AWS Lambda để sao chép dữ liệu tới một bảng DynamoDB riêng biệt để lưu trữ dữ liệu lâu dài

Chúng ta sẽ sử dụng lại bảng logfile mà chúng ta đã tạo ở bài thực hành số 1 để bật DynamoDB Stream cho nó. Bất cứ khi nào có sự thay đổi trên bảng logfile, ngay lập tức thông tin thay đổi sẽ được đưa vào stream. Tiếp theo ta sẽ gắn một Lambda function vào stream với mục đích truy vấn sự thay đổi trên bảng logfile, rồi ghi lại những cập nhật đó vào bảng được tạo mới có tên là logfile_replica. Biểu đồ sau thể hiện các bước thực hiện của bài thực hành này:


![DynamoDB](/images/1-Introduce/0008-Diagram-DynamoDB-Tables.png)

1. Tạo Bảng Sao chép

Bảng Sao chép có tên **logfile_replica** sẽ có các thuộc tính tương tự với bản gốc

```
aws dynamodb create-table --table-name logfile_replica \
--attribute-definitions AttributeName=PK,AttributeType=S AttributeName=GSI_1_PK,AttributeType=S \
--key-schema AttributeName=PK,KeyType=HASH --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=5 \
--tags Key=workshop-design-patterns,Value=targeted-for-cleanup \
--global-secondary-indexes "IndexName=GSI_1,KeySchema=[{AttributeName=GSI_1_PK,KeyType=HASH}],\

Projection={ProjectionType=INCLUDE,NonKeyAttributes=['bytessent', 'requestid', 'host']},\
ProvisionedThroughput={ReadCapacityUnits=10,WriteCapacityUnits=5}"
```

Bảng cũng được liên kết với Chỉ mục phụ toàn cục. Thông tin chi tiết được thống kê lại như sau:

- Key schema: HASH (partition key)

- Table read capacity units (RCUs) = 10

- Table write capacity units (WCUs) = 5

- Global secondary index: GSI_1 (10 RCUs, 5 WCUs) - Cho phép truy vấn dựa theo địa chỉ IP của máy host.

| Attribute Name (Type)  | Special Attribute?  |Attribute Use Case   |  Sample Attribute Value |
|---|---|---|---|
| PK (STRING)  | 	Partition key  | Holds the request id | request#104009 | 
|GSI_1_PK (STRING)  |	GSI 1 partition key   |  	Host |  	host#66.249.67.3 | 

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/0001-dynamodbstreamandlambda.png)

2. Chạy lệnh sau, đợi cho tới khi trạng thái của bảng chuyển thành ACTIVE

```
aws dynamodb wait table-exists --table-name logfile_replica
```

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/0002-dynamodbstreamandlambda.png)

3. Tạo bảng **logfile_replica** thành công


![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/0003-dynamodbstreamandlambda.png)

4. Kiểm tra lại IAM Policy gán cho AWS Lambda

Ở phần thiết lập chúng ta đã chuẩn bị trước một IAM Role có tên DDBReplicationRole mà đã được gán cho AWS Lambda function. IAM Role này cấp cho AWS Lambda function những quyền hạn cần thiết để thực hiện sao chép dữ liệu. Cụ thể các permission đó bao gồm:

AWS Lambda function có khả năng gọi tới DynamoDB Stream và lấy được thông tin các thay đổi bản ghi từ Stream

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

AWS Lambda function có thể thêm hoặc xóa item khỏi bảng logfile_replica

```
{
    "Action": [
        "dynamodb:DeleteItem",
        "dynamodb:PutItem"
    ],
    "Resource": [
        "*"
    ],
    "Effect": "Allow"
}
```

5. Tạo Lambda function

AWS Lambda function sẽ được gắn với DynamoDB Stream của bảng logfile để sao chép hành động thêm hoặc xóa item của bảng gốc sang bảng sao logfile_replica. Nội dung Lambda function nằm trong file ddbreplica_lambda.py, bạn có thể xem bằng lệnh vim hoặc less.
Thực hiện nén nội dung của tập lệnh rồi upload lên dịch vụ AWS Lambda để tạo ra Lambda function.

```
zip ddbreplica_lambda.zip ddbreplica_lambda.py lab_config.py
```

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/0004-dynamodbstreamandlambda.png)

6. Lấy thông tin ARN của IAM Role mã sẽ được gán cho Lambda function.

```
cat ~/workshop/ddb-replication-role-arn.txt
```

Thông tin ARN sẽ tương tự như sau:

```
arn:aws:iam::<ACCOUNTID>:role/XXXXX-DDBReplicationRole-XXXXXXXXXXX
```

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/0005-dynamodbstreamandlambda.png)

7. Sao chép ARN có được ở trên, thay thế cho YOUR_ARN_HERE rồi chạy lệnh tạo Lambda function

```
aws lambda create-function \
--function-name ddbreplica_lambda --zip-file fileb://ddbreplica_lambda.zip \
--handler ddbreplica_lambda.lambda_handler --timeout 60 --runtime python3.7 \
--description "Sample lambda function for dynamodb streams" \
--role YOUR_ARN_HERE
```

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/0006-dynamodbstreamandlambda.png)

8. AWS Lambda Function đã được khởi tạo thành công

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/0007-dynamodbstreamandlambda.png)

9. Bật DynamoDB Stream

Khi stream đã được bật, chúng ta có thể chọn DynamoDB sẽ sao chép những item mới, hay những item cũ, hay tất cả, hay chỉ điều chỉnh partition key và sort key.

Để bật DynamoDB Stream cho bảng logfile với lựa chọn NEW_IMAGE (nghĩa là toàn bộ item mới)

```
aws dynamodb update-table --table-name 'logfile' --stream-specification StreamEnabled=true,StreamViewType=NEW_IMAGE
```

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/0008-dynamodbstreamandlambda.png)

10. Lấy thông tin của DynamoDB Stream

```
aws dynamodb describe-table --table-name 'logfile' --query 'Table.LatestStreamArn' --output text
```

Thông tin đầu ra tương tự như sau

```
arn:aws:dynamodb:<REGION>:<ACCOUNTID>:table/logfile/stream/2018-10-27T02:15:46.245
```

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/0009-dynamodbstreamandlambda.png)

11. Nối nguồn Stream với Lambda function

Như vậy bảng gốc đã được bật DynamoDB Stream và Lambda function cũng đã được tạo. Bước tiếp theo, ta sẽ nối Stream với Lambda function sử dụng lệnh sau đây

```
aws lambda create-event-source-mapping \
--function-name ddbreplica_lambda --enabled --batch-size 100 --starting-position TRIM_HORIZON \
--event-source-arn YOUR_STREAM_ARN_HERE
```

Kết quả mong đợi sẽ tương tự bên dưới

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

12. Đẩy dữ liệu vào bảng logfile và kiểm tra tiến trình sao chép vào bảng logfile_replica

Chạy lệnh python bên dưới để nạp thêm item vào bảng logfile. Các bản ghi mới sẽ được đưa vào DynamoDB Stream, dẫn tới việc kích hoạt Lambda function để ghi thông tin item mới vào bảng logfile_replica

```
python load_logfile.py logfile ./data/logfile_stream.csv
```

Kết quả tương tự như sau

```
RowCount: 2000, Total seconds: 15.808809518814087

```

![Amazon DynamoDB Streams và AWS Lambda](/images/9-DynamoDBStreamsandLambda/00011-dynamodbstreamandlambda.png)

13. Để kiểm tra tiến trình sao chép vào bảng logfile_replica có thành công hay chưa, thực hiện quét dữ liệu bảng logfile_replica.

```
aws dynamodb scan --table-name 'logfile_replica' --max-items 2 --output text
```

Kết quả tương tự như sau:

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
