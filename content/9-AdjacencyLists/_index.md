---
title : "Adjacency List"
date : "`r Sys.Date()`"
weight : 9
chapter : false
pre : " <b> 9. </b> "
---

#### Overview

When entities in an application have a many-to-many relationship, it is easy to model the relationship to create a contiguous list between them. In this model, all top-level entities (equivalent to nodes in the graph model) act as partition keys. Every connection with other entities is treated as an item in a partition by assigning the value of the sort key to the ID of the target entity (target node).

The exercise uses the InvoiceAndBills table to illustrate this design pattern. According to the scenario, Customer has many Invoices, and a 1-to-many connection is created between Customer and Invoice ID. An Invoice contains many Bills, and a Bill can be split and split into multiple Invoices, forming a many-to-many relationship between Bill ID and Invoice ID. The Partition key property can now be Invoice ID, Bill ID, or Customer ID.

You need to model a table to allow execution of the following types of queries:

Use Invoice ID to get Invoice details, Customer information and Bill details respectively related to Invoice.

Get all Invoice IDs of a specific customer

Use Bill ID to get Bill details, Customer information and respective Invoice details related to Bill.

The InvoiceAndBills table includes the following information:

Key schema: HASH, RANGE (partition and sort key)

Table read capacity units (RCUs) = 100

Table write capacity units (WCUS) = 100

Global secondary index (GSI): GSI_1 (100 RCUs, 100 WCUs) - Allows looking up related entities.

| Attribute Name (Type)  | Special Attribute?  |Attribute Use Case   |  Sample Attribute Value |
|---|---|---|---|
| PK (STRING)  | 	Partition key  | Holds the ID of the entity, either a bill, invoice, or customer  |  B#3392 or I#506 or C#1317 | 
| SK (STRING)  |Sort key, GSI_1 partition key   |  Holds the related ID: either a bill, invoice, or customer |  	I#1721 or C#506 or I#1317 | 

#### Practice

1. Create and load sample data for **InvoiceandBilling** table

Run the following command to create the InvoiceAndBilling table and the GSI_1 Secondary Index table with the partition key as the SK attribute of the Base table

```
aws dynamodb create-table --table-name InvoiceAndBills \
--attribute-definitions AttributeName=PK,AttributeType=S AttributeName=SK,AttributeType=S \
--key-schema AttributeName=PK,KeyType=HASH AttributeName=SK,KeyType=RANGE \
--provisioned-throughput ReadCapacityUnits=100,WriteCapacityUnits=100 \
--tags Key=workshop-design-patterns,Value=targeted-for-cleanup \
--global-secondary-indexes "IndexName=GSI_1,\

KeySchema=[{AttributeName=SK,KeyType=HASH}],\
Projection={ProjectionType=ALL},\
ProvisionedThroughput={ReadCapacityUnits=100,WriteCapacityUnits=100}"
```

![Adjacecy List](/images/8-AdjacencyLists/0001-adjacencylists.png)

2. Wait until the board goes into ACTIVE state

```
aws dynamodb wait table-exists --table-name InvoiceAndBills
```

![Adjacecy List](/images/8-AdjacencyLists/0002-adjacencylists.png)

3. In the **DyanmoDB** interface

- Select **Table**
- The **InvoiceAndBills** table has been created and the status has changed to **Active**

![Adjacecy List](/images/8-AdjacencyLists/0003-adjacencylists.png)

4. Then load the sample data into the table

```
python load_invoice.py InvoiceAndBills ./data/invoice-data.csv
```

![Adjacecy List](/images/8-AdjacencyLists/0004-adjacencylists.png)

5. Check out the new table on DynamoDB Console

- Select **Table**
- Select table **InvoiceAndBills**
- Select **Explore items**
- Select **Scan**
- Select **GSI_1**
- Select **Run**

![Adjacecy List](/images/8-AdjacencyLists/0005-adjacencylists.png)

6. Data scan results according to **GSI_1**

![Adjacecy List](/images/8-AdjacencyLists/0006-adjacencylists.png)

7. Invoice details query

Run the following script to query Invoice details


```
python query_invoiceandbilling.py InvoiceAndBills 'I#1420'
```

Output result


```
=========================================================
 Invoice ID:I#1420, BillID:B#2485, BillAmount:$135,986.00 , BillBalance:$28,322,352.00


 Invoice ID:I#1420, BillID:B#2823, BillAmount:$592,769.00 , BillBalance:$8,382,270.00


 Invoice ID:I#1420, Customer ID:C#1420


 Invoice ID:I#1420, InvoiceStatus:Cancelled, InvoiceBalance:$28,458,338.00 , InvoiceDate:10/31/17, InvoiceDueDate:11/20/17

 =========================================================
```

Review content of Invoice details, customer details, and Bill details. Notice how the results show the relationship between Invoice ID and Customer ID and Bill ID.

![Adjacecy List](/images/8-AdjacencyLists/0007-adjacencylists.png)

8. Query customer and bill details using Index

Make a query using the Customer ID, and then review the customer's details, along with a list of related invoices.

```
python query_index_invoiceandbilling.py InvoiceAndBills 'C#1249'
```

Output result

```

 =========================================================
 Invoice ID: I#661, Customer ID: C#1249



 Invoice ID: I#1249, Customer ID: C#1249

 =========================================================
```

![Adjacecy List](/images/8-AdjacencyLists/0008-adjacencylists.png)

9. Next, execute a query using Bill ID. Review the results to see the link between Bill ID and related Invoices.

```
python query_index_invoiceandbilling.py InvoiceAndBills 'B#3392'
```
Output result


```
=========================================================
 Invoice ID: I#506, Bill ID: B#3392, BillAmount: $383,572.00 , BillBalance: $5,345,699.00



 Invoice ID: I#1721, Bill ID: B#3392, BillAmount: $401,844.00 , BillBalance: $25,408,787.00



 Invoice ID: I#390, Bill ID: B#3392, BillAmount: $581,765.00 , BillBalance: $11,588,362.00

 =========================================================
```

![Adjacecy List](/images/8-AdjacencyLists/0009-adjacencylists.png)