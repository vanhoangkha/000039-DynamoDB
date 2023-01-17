---
title : "Composite Keys"
date : "`r Sys.Date()`"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

#### Overview

Selecting the attribute as the Sort key plays an extremely important role because it can greatly improve the speed of item selection when performing a query. Suppose you want to find employees by geographic location (state and city) and by work location, the corresponding attributes are state, city, and dept; you can create a composite key of all 3 properties to allow search by location/dept. In DynamoDB, you can query a table using a combination of the partition key and the sort key. In this case, your query requires more than 2 properties, so you have to create a composite key that allows you to query using more than 2 properties.

We have created the Employee table and loaded the sample data. In this exercise, the state property in the Base table will be prefixed with # to become #state - acting as a partition key with the name GSI_3_PK for the new index table. The city & dept attributes are concatenated together to become the composite attribute city#dept - acting as a sort key with the name GSI_3_SK for the new index.


Attribute Name (Type)  | 	Special Attribute?	  | Attribute Use Case  |  Sample Attribute Value |
|---|---|---|---|
|  GSI_3_PK (STRING)	 | GSI_3 partition key  |  The state of the employee | 	state#WA  |
| GSI_3_SK (STRING)	GSI_  | GSI_3 sort key	  |  The city and department of the employee, concatenated | Seattle#Development  | 

1. Create a new GSI Index Table

Run the following command:

```
aws dynamodb update-table --table-name employees \
--attribute-definitions AttributeName=GSI_3_PK,AttributeType=S AttributeName=GSI_3_SK,AttributeType=S \
--global-secondary-index-updates file://gsi_city_dept.json
```


![Composite Keys](/images/7-CompositeKeys/0001-compositekeys.png)

2. Check the status of the command to create a new sub-index table

```
aws dynamodb describe-table --table-name employees --query "Table.GlobalSecondaryIndexes[].IndexStatus"
```
After the table creation state becomes ACTIVE, then you can start the query with a new secondary index table using the new partition key and sort key. With the sort key, you can use the begin_with expression to query properties that start with a certain string or value. As a result, you will be able to list all the employees in a city or a certain department in the city.

![Composite Keys](/images/7-CompositeKeys/0002-compositekeys.png)

3. In **DynamoDB** interface, **GSI_3** has been created successfully

![Composite Keys](/images/7-CompositeKeys/0003-compositekeys.png)

4. Query employee information based on the Partition key of the new secondary index

If the search is only related to the state property, then the query will only use the partition key and not the soft key. However, if the content requires city or dept information, the query will use the GSI_3_SK key with the value of the city_dept attribute to find the corresponding information. The image below shows the search content including city and department.

- In the **employees** table, select **Query**
- Select **GSI_3**
- **Partition key**, enter ```state#AZ```
- For **Sortkey** using **Begin with**, enter the value ```Phoenix#Develop```
- Select **Sort descending**
- Select **Run**

![Composite Keys](/images/7-CompositeKeys/0004-compositekeys.png)

5. Query results

![Composite Keys](/images/7-CompositeKeys/0005-compositekeys.png)

6. We can also execute this query with a Python script, specifically in the query_city_dept.py file. The code shows that the script takes 2 input parameters value1 and value2, and the query will search on the new secondary index table GSI_3.


```
if value2 == "-":
  ke = Key('GSI_3_PK').eq("state#{}".format(value1))
else:
  ke = Key('GSI_3_PK').eq("state#{}".format(value1)) & Key('GSI_3_SK').begins_with(value2)

response = table.query(
  IndexName='GSI_3',
  KeyConditionExpression=ke
  )
```
Chạy lệnh python như bên dưới để tìm kiếm tất cả các nhân viên ở bang Texas

```
python query_city_dept.py employees TX
```

Kết quả hiển thị

```
List of employees . State: TX
    Name: Bree Gershom. City: Austin. Dept: Development
    Name: Lida Flescher. City: Austin. Dept: Development
    Name: Tristam Mole. City: Austin. Dept: Development
    Name: Malinde Spellman. City: Austin. Dept: Development
    Name: Giovanni Goutcher. City: Austin. Dept: Development
  ...
  Name: Cullie Sheehy. City: San Antonio. Dept: Support
  Name: Ari Wilstead. City: San Antonio. Dept: Support
  Name: Odella Kringe. City: San Antonio. Dept: Support
Total of employees: 197. Execution time: 0.238062143326 seconds
```

![Composite Keys](/images/7-CompositeKeys/0006-compositekeys.png)

7. Find all employees in a city

Run below command to list all employees in a city

```
python query_city_dept.py employees TX --citydept Dallas
```

The results displayed are as follows:

```
List of employees . State: TX
    Name: Grayce Duligal. City: Dallas. Dept: Development
    Name: Jere Vaughn. City: Dallas. Dept: Development
    Name: Valeria Gilliatt. City: Dallas. Dept: Development
  ...
  Name: Brittani Hunn. City: Dallas. Dept: Support
    Name: Oby Peniello. City: Dallas. Dept: Support
Total of employees: 47. Execution time: 0.21702003479 seconds
```

![Composite Keys](/images/7-CompositeKeys/0007-compositekeys.png)

8. Search for all employees of a certain department in a city

Run the following command to list all employees in the Operations department located in Dallas, Texas

```
python query_city_dept.py employees TX --citydept 'Dallas#Op'
```

The following results


```
List of employees . State: TX
    Name: Brady Marvel. City: Dallas. Dept: Operation
    Name: Emmye Fletcher. City: Dallas. Dept: Operation
    Name: Audra Leahey. City: Dallas. Dept: Operation
    Name: Waneta Parminter. City: Dallas. Dept: Operation
    Name: Lizbeth Proudler. City: Dallas. Dept: Operation
    Name: Arlan Cummings. City: Dallas. Dept: Operation
    Name: Bone Ruggs. City: Dallas. Dept: Operation
    Name: Karlis Prisk. City: Dallas. Dept: Operation
    Name: Marve Bignold. City: Dallas. Dept: Operation
Total of employees: 9. Execution time: 0.174154996872 seconds
```

![Composite Keys](/images/7-CompositeKeys/0008-compositekeys.png)


