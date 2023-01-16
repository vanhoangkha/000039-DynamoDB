---
title : "Composite Keys"
date :  "`r Sys.Date()`" 
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

#### Tổng quan

Việc lựa chọn thuộc tính làm Sort key đóng vai trò vô cùng quan trọng bởi nó có thể giúp cải thiện đáng kể tốc độ lựa chọn item khi thực hiện truy vấn. Giả sử bạn muốn tìm các nhân viên dựa theo vị trí địa lý (state và city) và theo nơi công tác, các thuộc tính tương ứng là state, city, và dept; bạn có thể tạo một khóa tổng hợp của cả 3 thuộc tính để cho phép tìm kiếm theo location/dept. Trong DynamoDB, bạn có thể truy vấn bảng sử dụng khóa kết hợp giữa partition key và sort key. Còn trong trường hợp này, yêu cầu truy vấn của bạn nhiều hơn là 2 thuộc tính, do vậy bạn phải tạo một khóa tổng hợp cho phép bạn truy vấn sử dụng nhiều hơn 2 thuộc tính.

Chúng ta đã tạo bảng Employee và nạp vào những dữ liệu mẫu. Trong bài thực hành này, thuộc tính state trong bảng Cơ sở sẽ được thêm tiền tố # để thành #state - đóng vai trò làm partition key với tên GSI_3_PK cho bảng chỉ mục mới. Các thuộc tính city & dept được ghép lại với nhau trở thành thuộc tính tổng hợp city#dept - đóng vai trò làm sort key với tên GSI_3_SK cho bảng chỉ mục mới.


Attribute Name (Type)  | 	Special Attribute?	  | Attribute Use Case  |  Sample Attribute Value |
|---|---|---|---|
|  GSI_3_PK (STRING)	 | GSI_3 partition key  |  The state of the employee | 	state#WA  |
| GSI_3_SK (STRING)	GSI_  | GSI_3 sort key	  |  The city and department of the employee, concatenated | Seattle#Development  | 

1. Tạo Bảng Chỉ mục GSI mới

Thực hiện chạy lệnh sau:

```
aws dynamodb update-table --table-name employees \
--attribute-definitions AttributeName=GSI_3_PK,AttributeType=S AttributeName=GSI_3_SK,AttributeType=S \
--global-secondary-index-updates file://gsi_city_dept.json
```


![Composite Keys](/images/7-CompositeKeys/0001-compositekeys.png)

2. Kiểm tra trạng thái lệnh tạo bảng chỉ mục phụ mới

```
aws dynamodb describe-table --table-name employees --query "Table.GlobalSecondaryIndexes[].IndexStatus"
```
Sau khi trạng thái tạo bảng trở thành ACTIVE, thì bạn có thể bắt đầu truy vấn với bảng chỉ mục phụ mới sử dụng partition key và sort key mới. Với sort key, bạn có thể dùng thêm biểu thức begin_with để truy vấn những thuộc tính bắt đầu với chuỗi nào đó hoặc giá trị nào đó. Kết quả là bạn sẽ liệt kê ra được tất cả các nhân viên trong một thành phố hoặc một phòng ban nào đó trong thành phố.

![Composite Keys](/images/7-CompositeKeys/0002-compositekeys.png)

3. Trong giao diện **DynamoDB**, **GSI_3** đã tạo thành công

![Composite Keys](/images/7-CompositeKeys/0003-compositekeys.png)

4. Truy vấn thông tin nhân viên dựa vào Partition key của bảng chỉ mục phụ mới

Nếu nội dung tìm kiếm chỉ liên quan tới thuộc tính state, thì truy vấn sẽ chỉ sử dụng partition key mà không dùng tới soft key. Tuy nhiên nếu nội dung có yêu cầu thông tin city hoặc dept, truy vấn sẽ sử dụng tới khóa GSI_3_SK với giá trị là thuộc tính city_dept để tìm kiếm thông tin tương ứng. Hình ảnh bên dưới thể hiện nội dung tim kiếm có bao gồm city và department.

- Trong bảng **employees**, chọn **Query**
- Chọn **GSI_3**
- **Partition key**, nhập ```state#AZ```
- Đối với **Sortkey** sử dụng **Begin with**, nhập giá trị ```Phoenix#Develop```
- Chọn **Sort descending**
- Chọn **Run**

![Composite Keys](/images/7-CompositeKeys/0004-compositekeys.png)

5. Kết quả truy vấn

![Composite Keys](/images/7-CompositeKeys/0005-compositekeys.png)

6. Chúng ta cũng có thể thực hiện truy vấn này bằng một tập lệnh Python, cụ thể trong file query_city_dept.py. Đoạn mã cho biết tập lệnh gồm 2 tham số đầu vào là value1 và value2, và truy vấn sẽ tìm kiếm trên bảng chỉ mục phụ mới GSI_3.

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

7. Tìm tất cả nhân viên trong một thành phố

Chạy lệnh bên dưới để liệt kê tất cả nhân viên trong một thành phố

```
python query_city_dept.py employees TX --citydept Dallas
```

Kết quả hiện thị như sau:

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

8.  Tìm kiếm tất cả nhân viên thuộc một phòng ban nào đó trong một thành phố

Chạy lệnh sau để liệt kê tất cả các nhân viên thuộc phòng Operation nằm ở thành phố Dallas, Texas

```
python query_city_dept.py employees TX --citydept 'Dallas#Op'
```

Kết quả như sau

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

