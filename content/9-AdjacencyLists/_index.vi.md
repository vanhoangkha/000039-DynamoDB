---
title : "Adjacency List"
date :  "`r Sys.Date()`" 
weight : 9
chapter : false
pre : " <b> 9. </b> "
---

#### Tổng quan

Khi các thực thể trong một ứng dụng có mối quan hệ nhiều-nhiều với nhau, thì sẽ rất dễ khi mô hình hóa quan hệ để tạo ra một danh sách liền kề giữa chúng. Trong mô hình này, tất cả các thực thể ở cấp cao nhất (tương đương với node trong mô hình đồ thị) đóng vai trò là partition key. Mọi kết nối với các thực thể khác được coi như một item trong 1 phân vùng bằng cách gán giá trị của sort key với ID của thực thể mục tiêu (target node)

Bài thực hành sử dụng bảng InvoiceAndBills để minh họa về mẫu thiết kế này. Theo kịch bản, Customer có nhiều Invoice, và kết nối 1-nhiều được tạo ra giữa Customer với Invoice ID. Một Invoice thì chứa nhiều Bill, và 1 Bill có thể bị tách và chia thành nhiều Invoice, tạo thành một quan hệ nhiều-nhiều giữa Bill ID và Invoice ID. Thuộc tính Partition key lúc này có thể là Invoice ID, Bill ID hoặc Customer ID.

Bạn cần mô hình hóa một bảng để cho phép thực thi các kiểu truy vấn như sau:

Sử dụng Invoice ID để lấy thông tin chi tiết của Invoice, thông tin Customer và thông tin chi tiết của Bill tương ứng có liên quan tới Invoice.

Lấy tất cả Invoice ID của một khách hàng cụ thể

Sử dụng Bill ID để lấy thông tin chi tiết của Bill, thông tin Customer và thông tin chi tiết của Invoice tương ứng có liên quan tới Bill.

Bảng InvoiceAndBills bao gồm những thông tin sau:

Key schema: HASH, RANGE (partition và sort key)

Table read capacity units (RCUs) = 100

Table write capacity units (WCUS) = 100

Global secondary index (GSI): GSI_1 (100 RCUs, 100 WCUs) - Cho phép tra cứu những thực thể liên quan với nhau.

| Attribute Name (Type)  | Special Attribute?  |Attribute Use Case   |  Sample Attribute Value |
|---|---|---|---|
| PK (STRING)  | 	Partition key  | Holds the ID of the entity, either a bill, invoice, or customer  |  B#3392 or I#506 or C#1317 | 
| SK (STRING)  |Sort key, GSI_1 partition key   |  Holds the related ID: either a bill, invoice, or customer |  	I#1721 or C#506 or I#1317 | 

#### Thực hành

1. Tạo và nạp dữ liệu mẫu cho bảng **InvoiceandBilling**

Chạy lệnh sau để tạo bảng InvoiceAndBilling và bảng Chỉ mục phụ GSI_1 với partition key có giá trị là thuộc tính SK của bảng Cơ sở

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

2. Đợi cho tới khi bảng chuyển sang trạng thái ACTIVE

```
aws dynamodb wait table-exists --table-name InvoiceAndBills
```

![Adjacecy List](/images/8-AdjacencyLists/0002-adjacencylists.png)

3. Trong giao diện **DyanmoDB**

- Chọn **Table**
- Bảng **InvoiceAndBills** đã được tạo và trang thái chuyển sang **Active**

![Adjacecy List](/images/8-AdjacencyLists/0003-adjacencylists.png)

4. Rồi thực hiện load dữ liệu mẫu vào bảng

```
python load_invoice.py InvoiceAndBills ./data/invoice-data.csv
```

![Adjacecy List](/images/8-AdjacencyLists/0004-adjacencylists.png)

5. Kiểm tra bảng mới trên DynamoDB Console

- Chọn **Table**
- Chọn bảng **InvoiceAndBills**
- Chọn **Explore items**
- Chọn **Scan**
- Chọn **GSI_1**
- Chọn **Run**

![Adjacecy List](/images/8-AdjacencyLists/0005-adjacencylists.png)

6. Kết quả scan dữ liệu theo **GSI_1**

![Adjacecy List](/images/8-AdjacencyLists/0006-adjacencylists.png)

7. Truy vấn thông tin chi tiết của Invoice

Chạy tập lệnh sau đây để truy vấn thông tin chi tiết của Invoice


```
python query_invoiceandbilling.py InvoiceAndBills 'I#1420'
```

Kết quả đầu ra

```
=========================================================
 Invoice ID:I#1420, BillID:B#2485, BillAmount:$135,986.00 , BillBalance:$28,322,352.00


 Invoice ID:I#1420, BillID:B#2823, BillAmount:$592,769.00 , BillBalance:$8,382,270.00


 Invoice ID:I#1420, Customer ID:C#1420


 Invoice ID:I#1420, InvoiceStatus:Cancelled, InvoiceBalance:$28,458,338.00 , InvoiceDate:10/31/17, InvoiceDueDate:11/20/17

 =========================================================
```
Duyệt lại nội dung thông tin chi tiết Invoice, thông tin chi tiết customer, và thông tin chi tiết Bill. Chú ý cách kết quả hiển thị mối quan hệ giữa Invoice ID và Customer ID và Bill ID.

![Adjacecy List](/images/8-AdjacencyLists/0007-adjacencylists.png)

8. Truy vấn thông tin chi tiết customer và bill sử dụng Chỉ mục

Thực hiện truy vấn sử dụng Customer ID, rồi duyệt lại thông tin chi tiết của customer, cùng với danh sách các invoice liên quan.

```
python query_index_invoiceandbilling.py InvoiceAndBills 'C#1249'
```

Kết quả đầu ra

```

 =========================================================
 Invoice ID: I#661, Customer ID: C#1249



 Invoice ID: I#1249, Customer ID: C#1249

 =========================================================
```

![Adjacecy List](/images/8-AdjacencyLists/0008-adjacencylists.png)

9. Tiếp theo, thực hiện truy vấn sử dụng Bill ID. Duyệt lại kết quả để thấy được sự liên kết giữa Bill ID và các Invoice liên quan.

```
python query_index_invoiceandbilling.py InvoiceAndBills 'B#3392'
```
Kết quả đầu ra

```
=========================================================
 Invoice ID: I#506, Bill ID: B#3392, BillAmount: $383,572.00 , BillBalance: $5,345,699.00



 Invoice ID: I#1721, Bill ID: B#3392, BillAmount: $401,844.00 , BillBalance: $25,408,787.00



 Invoice ID: I#390, Bill ID: B#3392, BillAmount: $581,765.00 , BillBalance: $11,588,362.00

 =========================================================
```

![Adjacecy List](/images/8-AdjacencyLists/0009-adjacencylists.png)