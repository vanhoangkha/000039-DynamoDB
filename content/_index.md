---
title : "Advanced Amazon DynamoDB"
date : "`r Sys.Date()`"
weight : 1
chapter : false
---

# Advanced Amazon DynamoDB

#### Overview

In this workshop, we will learn together some advanced architectures using **Amazon DynamoDB** services and best practices for building powerful, scalable applications that have been performance and cost optimization.

To make it easy to practice following the workshop, we have prepared Python scripts to help build the architectural patterns used in the workshop. And by the end of the workshop, you'll have enough knowledge to build and monitor applications using the **DynamoDB** service that can grow to any size and scale.

![DynamoDB](/images/1-Introduce/0001-Diagram-DynamoDB-Tables.png)

#### Content

1. [Introduction](1-introduce)
2. [Preparation steps](2-prerequiste)
3. [DynamoDB Capacity Units and Partitioning](3-dynamodbcapacityunits)
4. [Table Scan Types: Sequential and Parallel](4-scan)
5. [Global Secondary Index Write Sharding](5-gsiwritesharding)
6. [Global Secondary Index Department Key Overloading](6-gsikeyoverloading)
7. [Sparse Global Secondary Indexes](7-sparsegsi)
8. [Composite Keys](8-compositekeys)
9. [Adjacency Lists](9-adjacencylists)
10. [Amazon DynamoDB Streams and AWS Lambda](10-dynamodbstreamsandlambda)
11. [Resource Cleanup](11-cleanup)

#### Target
The workshop is suitable for application developers, software engineers, and database administrators who are involved in the design and maintenance of applications using DynamoDB services.

#### Request
- Basic knowledge of AWS services
- Along with other services, the workshop will teach you how to use AWS Systems Manager Session Manager and AWS Lambda
- Basic understanding of DynamoDB
- If you have never worked with DynamoDB, please refer to the Amazon DynamoDB documentation. before starting the workshop

#### Reference

1. [Key Reference Advanced Design Patterns for Amazon DynamoDB](https://amazon-dynamodb-labs.com/design-patterns.html)
2. [Core Components of Amazon DynamoDB Reference](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html#HowItWorks.CoreComponents.PrimaryKey)
3. [What is Database sharding reference?](https://viblo.asia/p/database-sharding-la-gi-Az45boQVKxY)
4. [Video introducing advanced architectural patterns using DynamoDB](https://www.youtube.com/watch?v=6yqfmXiZTlM)