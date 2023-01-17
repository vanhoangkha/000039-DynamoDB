---
title : "Global Secondary Index Write Sharding"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Overview

The primary key of a Base table or a GSI Index table in DynamoDB includes a partition key and a sort key (optional).
How you design the keys for the Index table is extremely important to the structure and performance of the database you build.
The value of the Partition key determines how to divide the original database into logical partitions where data or items are stored.
Choosing the right partition key value helps the database workload to be evenly distributed across all partitions in the Base table or GSI Index table.

In this exercise, we will learn how to build a GSI Write Sharding Index table, which increases performance by selectively querying items spanning various logical partitions.
Going back to the server access log example from Exercise #1, based on the Apache service access log. This time, we'll be querying for items with a 4xx response code.
Note that items with 4xx response codes account for a very small percentage and are not uniformly distributed according to the response codes in the datasheet.

The following chart shows the distribution of the logs based on the response code in the sample file **logfile_medium1.csv**.

![DynamoDB](/images/1-Introduce/0007-Diagram-DynamoDB-Tables.png)

We will create a GSI write sharding index table on a table to randomize the writes of the partition key value to the logical partition, which increases the read/write throughput of the application.
To start, we generate a random number from a fixed set (e.g. 1 to 10) and use this as the partition key for the GSI Index table. The random partition key value that is written to the partitions of the Index table is spread over all partition key values ​​belonging to the set defined above (from 1 to 10) and it is independent of all remaining attributes. This results in better parallelism and higher overall throughput.

To summarize in this exercise, we will create a new Index table using the value Random as the Partition key, and the composite key responsecode#date#hourofday as the Sort key. The logfile_scan table that we created in the preparation step already has these 2 properties. Specifically, the code that initializes them is:

```
SHARDS = 10
newitem['GSI_1_PK'] = "shard#{}".format((newitem['requestid'] % SHARDS) + 1)
newitem['GSI_1_SK'] = row[7] + "#" + row[2] + "#" + row[3]
```

1. View information from the Index table

- The GSI Index table has been created in the workshop settings. You can see the description of the Index table with the command below

```
aws dynamodb describe-table --table-name logfile_scan --query "Table.GlobalSecondaryIndexes"
```

**Description similar to the following:**

```
{
  "GlobalSecondaryIndexes": [
    {
        "IndexName": "GSI_1",
        "KeySchema": [
            {
                "AttributeName": "GSI_1_PK",
                "KeyType": "HASH"
            },
            {
                "AttributeName": "GSI_1_SK",
                "KeyType": "RANGE"
            }
        ],
        "Projection": {
            "ProjectionType": "KEYS_ONLY"
        },
        "IndexStatus": "ACTIVE",
        "ProvisionedThroughput": {
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 3000,
            "WriteCapacityUnits": 5000
        },
        "IndexSizeBytes": 0,
        "ItemCount": 0,
        "IndexArn": "arn:aws:dynamodb:(region):(accountid):table/logfile_scan/index/GSI_1"
    }
]
}
```
- DynamoDB ItemCount in this example has a value of zero.
- DynamoDB calculates the total number of items seen by this API multiple times a day and the actual value may differ from the result shown in the example.

![Global Secondary Index Write Sharding](/images/5-GSIwritesharding/0001-gsiwritesharing.png)

2. **Query the GSI Write Sharding Index table**

- To get all records with 404 response code, you need to query on the entire global sub-index partition using sort key.

```
  if date == "all":
    ke = Key('GSI_1_PK').eq("shard#{}".format(shardid)) & Key('GSI_1_SK').begins_with(responsecode)
  else:
    ke = Key('GSI_1_PK').eq("shard#{}".format(shardid)) & Key('GSI_1_SK').begins_with(responsecode+"#"+date)

  response = table.query(
    IndexName='GSI_1',
    KeyConditionExpression=ke
    )
```

Run the following script to get items from the GSI Write Sharding Index table with only the partition key and response code

```
python query_responsecode.py logfile_scan 404
```

The command will query the logfile_scan table to find items with sort keys=404 using the begins_with parameter.
A query is run on each segment of the GSI index table and the results are pushed back to the client. The output of the script is similar to below

```
Records with response code 404 in the shardid 0 = 0
Records with response code 404 in the shardid 1 = 1750
Records with response code 404 in the shardid 2 = 2500
Records with response code 404 in the shardid 3 = 1250
Records with response code 404 in the shardid 4 = 1000
Records with response code 404 in the shardid 5 = 1000
Records with response code 404 in the shardid 6 = 1750
Records with response code 404 in the shardid 7 = 1500
Records with response code 404 in the shardid 8 = 3250
Records with response code 404 in the shardid 9 = 2750
Number of records with responsecode 404 is 16750. Query time: 1.5092344284057617 seconds
```

![Global Secondary Index Write Sharding](/images/5-GSIwritesharding/0002-gsiwritesharing.png)

3. You can query on the same index, but add a date to the command to run the script. Specifically, execute a **logfile_scan** table query to find items with a catch key of **404#2017-07-21** using the **begins_with** parameter.

```
python query_responsecode.py logfile_scan 404 --date 2017-07-21
```
The result of running command is similar below

```
Records with response code 404 in the shardid 0 = 0
Records with response code 404 in the shardid 1 = 750
Records with response code 404 in the shardid 2 = 750
Records with response code 404 in the shardid 3 = 250
Records with response code 404 in the shardid 4 = 500
Records with response code 404 in the shardid 5 = 0
Records with response code 404 in the shardid 6 = 250
Records with response code 404 in the shardid 7 = 1000
Records with response code 404 in the shardid 8 = 1000
Records with response code 404 in the shardid 9 = 1000
Number of records with responsecode 404 is 5500. Query time: 1.190359354019165 seconds
```

![Global Secondary Index Write Sharding](/images/5-GSIwritesharding/0003-gsiwritesharing.png)

#### Conclusion

In this exercise, we used the GSI write sharding Index table to quickly retrieve sorted results, using composite keys that will be covered in more depth in exercise 6.
The GSI write sharding table in the above example uses an array of partition keys with values ​​from 0 to 9, but in your application you can choose any range.
Also, you can add more shards as the number of items indexed increases. Within each shard, the data is sorted according to the sort key. This allows us to retrieve server access logs sorted by **#response code** and **#date**, e.g. **404#2017-07-21**.