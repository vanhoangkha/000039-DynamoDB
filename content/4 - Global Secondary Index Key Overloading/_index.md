+++
title = "Global Secondary Index Key Overloading"
date = 2020
weight = 5
chapter = true
pre = "<b>5. </b>"
+++

# Bài Thực Hành 4: Global Secondary Index Key Overloading

Bạn có thể tạo 20 bảng chỉ mục GSI cho bảng DynamoDB tính đến thời điểm chúng tôi xây dựng workshop này.  
Tuy nhiên, đôi khi ứng dụng của chúng ta có thể cần hỗ trợ nhiều mẫu truy cập hơn thế và vượt quá giới hạn hiện tại của bảng chỉ mục GSI trên mỗi bảng.  
Từ đó sinh ra mẫu thiết kế bảng chỉ mục GSI Key Overloading bằng việc chọn lựa và sử dụng lại một thuộc tính (tiêu đề cột) trong số các item và lưu trữ giá trị của thuộc tính đó tùy vào ngữ cảnh của loại thuộc tính.  

Khi bạn tạo bảng Chỉ mục GSI trên thuộc tính đó, chính là bạn đang lập chỉ mục cho nhiều mẫu truy cập, mỗi mẫu cho một loại item khác nhau — và chỉ sử dụng 1 bảng Chỉ mục GSI.
Ví dụ với bảng **Employees**, một nhân viên có thể chứa các item loại metadata (để biết chi tiết về nhân viên), employee-title (tất cả các chức danh công việc mà nhân viên đó đã đảm nhiệm) hoặc employee-location (tất cả các tòa nhà văn phòng và địa điểm mà nhân viên đó đã làm việc).

Các mẫu truy cập cần thiết cho kịch bản này gồm: 
- Truy vấn tất cả nhân viên của một bảng 

- Truy vấn tất cả nhân viên với một chức danh cụ thể hiện tại
- Truy vấn tất cả nhân viên từng có một chức danh cụ thể 
- Truy vấn tất cả nhân viên theo tên 

Hình bên dưới mô tả thiết kế của bảng **Employees**.
Thuộc tính **PK(Parition key)** có giá trị là sự kết hợp giữa **employeeID** với tiền tố là chữ **e**. Dấu thăng **(#)** đặt giữa mã định danh loại thực thể **(e)** và ID thật sự của nhân viên **employeeID**.  
Thuộc tính **SK (Soft key)** đóng vai trò là thuộc tính overloaded và giá trị của nó có thể là *current title*, *previous title*, hoặc từ khóa *root* biểu thị đây là một primary item của bảng Employee, item này có giá trị ở những thuộc tính quan trọng nhất 
Thuộc tính **GSI_1_PK** bao gồm chức danh hoặc tên của nhân viên.  
Việc sử dụng lại bảng Chỉ mục GSI cho nhiều loại thực thể như **employees**, **employee locations**, và **employee titles** cho phép chúng ta đơn giản hóa việc quản lý bảng DynamoDB vì chúng ta chỉ cần theo dõi và mất chi phí cho một bảng Chỉ mục phụ GSI thay vì ba chỉ mục riêng biệt.
![Security Hub](../../../images/17.png?width=90pc)  

### Bước 1 - Tạo bảng Employees

Thực hiện chạy lệnh sau để tạo bảng **Employees**
```python
aws dynamodb create-table --table-name employees \
--attribute-definitions AttributeName=PK,AttributeType=S AttributeName=SK,AttributeType=S \
AttributeName=GSI_1_PK,AttributeType=S AttributeName=GSI_1_SK,AttributeType=S \
--key-schema AttributeName=PK,KeyType=HASH AttributeName=SK,KeyType=RANGE \
--provisioned-throughput ReadCapacityUnits=100,WriteCapacityUnits=100 \
--tags Key=workshop-design-patterns,Value=targeted-for-cleanup \
--global-secondary-indexes "IndexName=GSI_1,\
KeySchema=[{AttributeName=GSI_1_PK,KeyType=HASH},{AttributeName=GSI_1_SK,KeyType=RANGE}],\
Projection={ProjectionType=ALL},\
ProvisionedThroughput={ReadCapacityUnits=100,WriteCapacityUnits=100}"
```
Chạy lệnh chờ cho tới khi trạng thái bảng chuyển sang **ACTIVE**
```python
aws dynamodb wait table-exists --table-name employees
```
Bảng Cơ sở có tên là **employees**  
Partition key của bảng là thuộc tính PK và nó chứa giá trị ID của nhân viên.   
Sort key của bảng Cơ sở là thuộc tính SK, với giá trị được chế biến trong script.  
Bảng Chỉ mục có tên là **GSI_1** - là bảng Chỉ mục GSI Overloading được sử dụng ở bước tiếp theo  
Partition key của bảng Chỉ mục GSI là **GSI_1_PK**, và có giá trị bằng với giá trị của thuộc tính **SK** trong bảng Cơ sở.   
Sort key của bảng Chỉ mục GSI là **GSI_1_SK** có giá trị là Tên của nhân viên, thuộc tính **Name**
- Key schema: HASH, RANGE (partition và sort key)

- Table read capacity units (RCUs) = 100
- Table write capacity units (WCUs) = 100
- Global secondary index:  GSI_1 (100 RCUs, 100 WCUs) - Cho phép truy vấn bằng địa chỉ IP của máy host

| Attribute Name (Type) | Special Attribute?  | Attribute Use Case | Sample Attribute Value |   |
|:---------------------:|---------------------|--------------------|------------------------|---|
| PK (STRING)           | Partition Key       | Employee ID        | e#129                  |   |
| SK (STRING)           | Sort key            | Derived value      | root, state#MI         |   |
| GSI_1_PK (STRING)     | GSI_1 partition key | Derived value      | root, state#MI         |   |
| GSI_1_SK (STRING)     | GSI_1 sort key      | Employee name      | Christine Milsted      |   |

### Bước 2 - Nạp dữ liệu vào bảng 

Thực hiện nạp dữ liệu vào bảng bằng lệnh sau
```python
python load_employees.py employees ./data/employees.csv
```
Bản ghi mẫu trong file **employees.csv** có dạng tương tự như sau:
```python
1000,Nanine Denacamp,Programmer Analyst,Development,San Francisco,CA,1981-09-30,2014-06-01,Senior Database Administrator,2014-01-25
```
Khi đẩy dữ liệu vào bảng, một số thuộc tính sẽ được nối với nhau, chẳng hạn như **City_Dept** (Ví dụ: San Francisco: Development), bởi chúng phù hợp với các mẫu truy cập trong lệnh truy vấn.  
Thuộc tính SK cũng là một thuộc tính sẽ được chế biến trong script. Việc ghép được xử lý nhờ các tập lệnh Python, rồi lắp ráp thành một bản ghi hoàn chỉnh và cuối cùng thực thi lệnh **put_item()** để ghi bản ghi vào bảng.

Kết quả của lệnh nạp tương tự như sau: 
```python
python load_employees.py employees ./data/employees.csv
employee count: 100 in 3.7393667697906494
employee count: 200 in 3.7162938117980957
...
employee count: 900 in 3.6725080013275146
employee count: 1000 in 3.6174678802490234
RowCount: 1000, Total seconds: 36.70457601547241
```
Đầu ra cho thấy 1000 bản ghi đã được ghi vào bảng. Truy cập DynamoDB Console, rồi bấm vào xem bảng **employees**. 
![Security Hub](../../../images/18.png?width=90pc)  
Ở phần nội dung Scan, chọn **Index** từ menu thả xuống, rồi bấm **Start Search**
![Security Hub](../../../images/19.png?width=90pc)   
Hình bên dưới cho thấy kết quả của lệnh Quét trên bảng Chỉ mục GSI quá tải ghi. Có nhiều loại thực thể trong bảng kết quả, bao gồm: **root**, **previous tiltle**, và **current title**
![Security Hub](../../../images/20.png?width=90pc)  

### Bước 3 - Truy vấn bảng Employees sử dụng Chỉ mục phụ toàn cục với các thuộc tính overloaded

Thực hiện truy vấn tìm tất cả các nhân viên đang làm tại trụ sở Washington bằng cách chạy script **query_employees.py**, trong đó có chứa đoạn mã để truy vấn bảng sử dụng chỉ mục phụ toàn cục overloaded
```python
python query_employees.py employees state 'WA'
```
Kết quả lệnh truy vấn
```
List of employees with WA in the attribute state:
    Employee name: Alice Beilby - hire date: 2014-12-03
    Employee name: Alla Absalom - hire date: 2015-06-25
    Employee name: Alvan Heliar - hire date: 2016-05-15
    Employee name: Anders Galtone - hire date: 2015-12-22
    Employee name: Ashil Hutchin - hire date: 2015-02-11
  ...
  Employee name: Sula Prattin - hire date: 2014-01-11
    Employee name: Vittoria Edelman - hire date: 2014-10-01
    Employee name: Willie McCuthais - hire date: 2015-05-27
Total of employees: 46. Execution time: 0.13477110862731934 seconds
```
Bạn cũng có thể truy vấn nhân viên dựa theo chức danh của họ, ví dụ 
```python
python query_employees.py employees current_title 'Software Engineer'
```
Kết quả truy vấn 
```python
 List of employees with Software Engineer in the attribute current_title:
    Employee name: Alice Beilby - hire date: 2014-11-03
    Employee name: Anetta Byrne - hire date: 2017-03-15
    Employee name: Ardis Panting - hire date: 2015-08-06
    Employee name: Chris Randals - hire date: 2016-10-27
    Employee name: Constantine Barendtsen - hire date: 2016-06-10
    Employee name: Eudora Janton - hire date: 2015-01-05
    Employee name: Florella Allsep - hire date: 2015-03-31
    Employee name: Horatius Trangmar - hire date: 2013-10-21
    Employee name: Korey Daugherty - hire date: 2016-11-03
    Employee name: Lenka Luquet - hire date: 2014-10-01
    Employee name: Leonora Hyland - hire date: 2016-06-14
    Employee name: Lucretia Ruffell - hire date: 2015-07-04
    Employee name: Malcolm Adiscot - hire date: 2014-04-17
    Employee name: Melodie Sebire - hire date: 2013-08-27
    Employee name: Menard Ogborn - hire date: 2014-06-27
    Employee name: Merwyn Petters - hire date: 2014-06-19
    Employee name: Niels Buston - hire date: 2014-10-30
    Employee name: Noelani Studde - hire date: 2015-03-30
Total of employees: 18. Execution time: 0.11937260627746582 seconds
```
Ngoài ra bạn cũng có thể truy vấn theo tên nhân viên như sau
```python
python query_employees.py employees name 'Dale Marlin'
```
Kết quả truy vấn 
```python
 List of employees with Dale Marlin in the attribute name:
    Employee name: Dale Marlin - hire date: 2014-10-19
Total of employees: 1. Execution time: 0.1274700164794922 seconds
```
