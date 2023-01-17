---
title : "Sparse Global Secondary Indexes"
date : "`r Sys.Date()`"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

#### Overview

The GSI Sparse Index table is used to locate items that contain uncommon attributes - that is, attributes that do not appear in all items in the table. Querying items with uncommon attributes can be more efficient since the number of items in the Index table is significantly less than the number of items in the Base table. In addition, the fewer attributes that project from the base table to the index table, the less read and write space is consumed by the index table.

#### Practice

1. Add a new Global Secondary Index to the **Emloyees** table

The new GSI Index table will use the is_manager attribute as the partition key because not all employees are managers and this attribute does not appear on all items. This key is stored under an attribute named GSI_2_PK. The employee's title is stored as the soft key GSI_2_SK.

```
aws dynamodb update-table --table-name employees \
--attribute-definitions AttributeName=GSI_2_PK,AttributeType=S AttributeName=GSI_2_SK,AttributeType=S \
--global-secondary-index-updates file://gsi_manager.json
```


![Composite Keys](/images/6-SparseGSI/0001-sparsegsi.png)

2. The process of creating a new index for the table will take about 5 minutes. Check the status with the command below. If the IndexStatus value is equal to CREATING then you will have to keep waiting until it changes to ACTIVE.

```
aws dynamodb describe-table --table-name employees --query "Table.GlobalSecondaryIndexes[].IndexStatus"
```

![Composite Keys](/images/6-SparseGSI/0002-sparsegsi.png)

3. In the **DynamoDB** interface

- Select **Table**
- Select the table **employees**
- Select **Indexes**
- **GSI_2** has completed initialization and **Active** status

![Composite Keys](/images/6-SparseGSI/0003-sparsegsi.png)

4. Scan the table for Manager employees - Don't use Discrete Indexes

Sparse index will help to shred your data into smaller pieces with corresponding indexes for more efficient Data Scraping or Query API. Instead of scanning the entire data in a DynamoDB Base table, you can create an index that holds only a portion of the information for easy querying and searching.

To better understand how to split the Base table into smaller tables with Sparse Index, you can refer to the article [Take Advantage of Sparse Indexes](https://docs.aws.amazon.com/amazondynamodb/latest) /developerguide/bp-indexes-general-sparse-indexes.html)

First, we will scan the data table to find all managers without using the index table.

```
python scan_for_managers.py employees 100
```

In which, employees is the table name, 100 is the number of items per scan. The result of the scan operation is similar to the following:

```
Managers count: 84. # of records scanned: 4000. Execution time: 0.596132993698 seconds
```

![Composite Keys](/images/6-SparseGSI/0004-sparsegsi.png)

5. Scan the table for employees who are Managers - Use Discrete Indexes

Use the Sparse index table created in step 1 to perform a data scan of the Employees table. The code that uses the Sparse index described in the script that runs the scan command is as follows:

```
response = table.scan(
  Limit=pageSize,
  IndexName='GSI_2'
)
```

Executing the data scan command

```
python scan_for_managers_gsi.py employees 100
```
In which, employees is the table name, 100 is the number of items of each scan. The results show that the number of items scanned and the execution time is much smaller than the case without using Sparse index.

```
Number of managers: 84. # of records scanned: 84. Execution time: 0.287754058838 seconds
```

![Composite Keys](/images/6-SparseGSI/0005-sparsegsi.png)