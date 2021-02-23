+++
title = "Setup"
date = 2020
weight = 1
chapter = true
pre = "<b>1. </b>"
+++
# Thiết lập môi trường

### 1. Khởi tạo CloudFormation Stack

{{% notice warning %}}
Xuyên suốt bài thực hành, chúng ta sẽ làm việc với các bảng DynamoDB với chi phí có thể lên tới hàng chục hoặc hàng trăm $ mỗi ngày.
Phải chắc chắn bạn đã xóa CloudFormation stack và toàn bộ các bảng DynamoDB với DynamoDB console ngay sau khi kết thúc bài thực hành.
{{% /notice %}}

a. Khởi tạo CloudFormation template tại region **US East 1** để triển khai các tài nguyên cần thiết bằng cách bấm vào [Launch Stack](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=amazon-dynamodb-labs&templateURL=https://s3.amazonaws.com/amazon-dynamodb-labs.com/assets/lab.yaml) hoặc bạn có thể tải về [CloudFormation template YAML](https://s3.amazonaws.com/amazon-dynamodb-labs.com/assets/lab.yaml) rồi tự khởi tạo.  

b. Nhấp vào **Next** ở hộp thoại đầu tiên.
![Security Hub](../../../images/1.png?width=90pc)   

c. Trong phần **Parameters**, tùy chọn mặc định cho **VPCSelection** là ***CreateNewVPC***. Lựa chọn này giúp bạn dễ dàng thực hiện theo bài hướng dẫn tuy nhiên nếu bạn muốn sử dụng VPC mặc định của mình, hãy thay đổi tùy chọn này thành ***Default***.  

d. Giữ nguyên thông số **WorkshopCodeURL** và nhấp vào **Next**
![Security Hub](../../../images/2.png?width=90pc)    

e. Tiếp tục giữ nguyên mọi giá trị mặc định của trang cấu hình tiếp theo rồi đi tới trang **Review**. Kéo xuống dưới và nhấp chọn **I acknowledge that AWS CloudFormation might create IAM resources** rồi bấm **Create stack**.
![Security Hub](../../../images/3.png?width=90pc)   

f. Stack sẽ tạo ra một EC2 Instance với Role đi kèm. Ngoài ra nó còn tạo thêm Role cho AWS Lambda function sẽ được sử dụng xuyên suốt bài thực hành này. 

### 2. Truy cập EC2 Instance qua AWS Systems Manager Console

a. Truy cập dịch vụ [AWS Systems Manager](https://console.aws.amazon.com/systems-manager/home?region=us-east-1), rồi chọn **Session Manager** ở cột điều hướng bên trái màn hình.  

b. Bấm **Start session** để bắt đầu một phiên shell mới, rồi chọn đúng EC2 Instance mà được tạo ra bởi CloudFormation stack. Thường tên EC2 Instance sẽ bắt đầu với ***mod-xxxx…***  

c. Ở phiên shell mới, chuyển sang thao tác bằng tài khoản ec2-user sử dụng lệnh
```python
sudo su - ec2-user
```  

d. Chạy lệnh ***shopt login_shell*** để đảm bảo kết quả trả về là ***login_shell on***. Sau đó chuyển sang thao tác trên thư mục của workshop bằng lệnh ***cd ~/workshop***
```python
sh-4.2$ sudo su - ec2-user
[ec2-user@ip-172-31-24-0 ~]$ #Verify login_shell is 'on'
[ec2-user@ip-172-31-24-0 ~]$ shopt login_shell
login_shell     on
[ec2-user@ip-172-31-24-0 ~]$ #Change into the workshop directory
[ec2-user@ip-172-31-24-0 ~]$ cd ~/workshop
```

### 3. Kiểm tra cài đặt Python và AWS CLI

a. Trên EC2 Instance chạy lệnh kiểm tra phiên bản cài đặt Python
```python
python --version
Python 3.6.12
```  

b. Kiểm tra phiên bản cài đặt AWS CLI
```python
aws --version
aws-cli/1.18.139 Python/3.6.12 Linux/4.14.193-113.317.amzn1.x86_64 botocore/1.17.62
```  
*Note: Để thực hành workshop không xảy ra lỗi, phải đảm bảo rằng phiên bản AWS CLI là 1.18.139 và python là 3.6.12*


### 4. Kiểm tra cài đặt boto3

Boto3 là một SDK của AWS phát triển riêng cho Python, nó cho phép các nhà phát triển Python xây dựng các ứng dụng dựa trên các dịch vụ AWS.
Trong cửa sổ EC2 shell, chạy lệnh python để bắt đầu trình tương tác với python rồi dán các mã lệnh sau vào cửa sổ: 
```python
# Run this code:
import boto3
ddb = boto3.client('dynamodb')
ddb.describe_limits()
```  
Kết quả hiển thị phải tương tự như bên dưới
```python
{u'TableMaxWriteCapacityUnits': 40000, u'TableMaxReadCapacityUnits': 40000, u'AccountMaxReadCapacityUnits': 80000, 'ResponseMetadata': {'RetryAttempts': 0, 'HTTPStatusCode': 200, 'RequestId': 'BFMGAS4P48I3DJTP5NU22QRDDJVV4KQNSO5AEMVJF66Q9ASUAAJG', 'HTTPHeaders': {'x-amzn-requestid': 'BFMGAS4P48I3DJTP5NU22QRDDJVV4KQNSO5AEMVJF66Q9ASUAAJG', 'content-length': '143', 'server': 'Server', 'connection': 'keep-alive', 'x-amz-crc32': '3062975651', 'date': 'Tue, 31 Dec 2020 00:00:00 GMT', 'content-type': 'application/x-amz-json-1.0'}}, u'AccountMaxWriteCapacityUnits': 80000}
```
Kết thúc kiểm tra, thực hiện tắt trình tương tác với python 
```python
quit()
```

### 5. Kiểm tra nội dung thư mục workshop
Trên EC2 instance, truy cập thư mục chứa workshop rồi chạy lệnh ls:
```python
cd /home/ec2-user/workshop
ls -l .
```
Các nội dung phải có trong thư mục workshop bao gồm  
**Python code:**  
*ddbreplica_lambda.py  
load_employees.py  
load_invoice.py  
load_logfile_parallel.py  
load_logfile.py  
lab_config.py  
query_city_dept.py  
query_employees.py  
query_index_invoiceandbilling.py  
query_invoiceandbilling.py  
query_responsecode.py  
scan_for_managers_gsi.py  
scan_for_managers.py  
scan_logfile_parallel.py  
scan_logfile_simple.py*

**JSON:**  
*gsi_city_dept.json  
gsi_manager.json  
iam-role-policy.json  
iam-trust-relationship.json*

**Text:**  
*ddb-replication-role-arn.txt*

Chạy lệnh ls để kiểm tra danh sách các dữ liệu mẫu: 
```python
ls -l ./data
employees.csv
invoice-data2.csv
invoice-data.csv
logfile_medium1.csv
logfile_medium2.csv
logfile_small1.csv
logfile_stream.csv
```

### 6. Kiểm tra Định dạng và Nội dung các file chứa Dữ liệu

Chúng ta sẽ làm việc với rất nhiều dữ liệu khác nhau xuyên suốt bài thực hành, đó là:
- Server Logs data

- Employees data
- Invoices and Bills data

Cấu trúc dữ liệu của file Server Logs gồm:
- requestid (number)

- host (string)
- date (string)
- hourofday (number)
- timezone (string)
- method (string)
- url (string)
- responsecode (number)
- bytessent (number)
- useragent (string)

Để xem một bản ghi mẫu trong file, dùng lệnh:
```python
head -n1 ./data/logfile_small1.csv
1,66.249.67.3,2017-07-20,20,GMT-0700,GET,"/gallery/main.php?g2_controller=exif.SwitchDetailMode&g2_mode=detailed&g2_return=%2Fgallery%2Fmain.php%3Fg2_itemId%3D15741&g2_returnName=photo",302,5,"Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"
```

Tương tự, cấu trúc của file Employees gồm: 
- employeeid (number)

- name (string)
- title (string)
- dept (string)
- city (string)
- state (string)
- dob (string)
- hire-date (string)
- previous title (string)
- previous title end date (string)
- is a manager (string), *1 cho nhân viên là Quản lý employees, và non-existent cho các loại nhân viên còn lại*

Để xem một bản ghi mẫu trong file, dùng lệnh:
```python
head -n1 ./data/employees.csv
1,Onfroi Greeno,Systems Administrator,Operation,Portland,OR,1992-03-31,2014-10-24,Application Support Analyst,2014-04-12
```

### 7. Nạp trước các Item cho bài tập Quét dữ liệu bảng

*Reminder: Tất cả các lệnh hướng dẫn phải được thực thi trong cửa sổ shell của EC2 Instance, không thao tác trên trên máy local.*

Trong bài thực hành **Lab #2** chúng ta sẽ thảo luận về Quét dữ liệu bảng và những phương pháp hay nhất. 
Tại bước này, chúng ta sẽ nạp trước 1 triệu items để chuẩn bị cho bài thực hành Quét dữ liệu bảng.
Chạy lệnh sau đây để bắt đầu việc tạo bảng: 
```python
aws dynamodb create-table --table-name logfile_scan \
--attribute-definitions AttributeName=PK,AttributeType=S AttributeName=GSI_1_PK,AttributeType=S AttributeName=GSI_1_SK,AttributeType=S \
--key-schema AttributeName=PK,KeyType=HASH \
--provisioned-throughput ReadCapacityUnits=5000,WriteCapacityUnits=5000 \
--tags Key=workshop-design-patterns,Value=targeted-for-cleanup \
--global-secondary-indexes "IndexName=GSI_1,\
KeySchema=[{AttributeName=GSI_1_PK,KeyType=HASH},{AttributeName=GSI_1_SK,KeyType=RANGE}],\
Projection={ProjectionType=KEYS_ONLY},\
ProvisionedThroughput={ReadCapacityUnits=3000,WriteCapacityUnits=5000}"
```

Kết quả lệnh tạo ra một bảng mới có tên **logfile_scan** và một GSI, cụ thể: 
- Key schema: HASH

- Table RCU = 5000
- Table WCU = 5000
- GSI(s): GSI_1 (3000 RCU, 5000 WCU) - cho phép quét dữ liệu nhật ký truy cập theo kiểu tuần tự hoặc song song. Sắp xếp theo status code và timestamp.
		
| Tên Thuộc tính (Loại) |      Mô tả     |                                  Trường hợp sử dụng                                  | Ví dụ Giá trị Thuộc tính  |   |
|:---------------------:|:--------------:|:------------------------------------------------------------------------------------:|:-------------------------:|---|
| PK (STRING)           | Hash key       | Thông tin request id phục vụ công tác kiểm tra nhật ký truy cập                      | request#104009            |   |
| GSI_1_PK (STRING)     | GSI 1 hash key | Là một shard key, với các giá trị từ 0-N, phục vụ công tác tìm kiếm bản ghi nhật ký  | shard#3                   |   |
| GSI_1_SK (STRING)     | GSI 1 sort key | Sắp xếp các bản ghi nhật ký theo thứ bậc, từ status code -> date -> hour             | 200#2019-09-21#01         |   |

Chạy lệnh sau để đợi cho đến khi trạng thái bảng trở thành Active:
```python
aws dynamodb wait table-exists --table-name logfile_scan
```

Chạy lệnh nạp 1,000,000 bản ghi từ file **Server logs** vào bảng **logfile_scan**.
```python
nohup python load_logfile_parallel.py logfile_scan &
disown
```
Tùy chọn ***nohup*** được sử dụng để chạy ngầm các tiến trình, còn ***disown*** cho phép dữ liệu vẫn sẽ tiếp tục được nạp trong trường hợp bạn vừa bị mất kết nối.

*Note: Lệnh nạp dữ liệu tạo ra một tiến trình chạy ngầm và mất khoảng 10 phút để hoàn thành việc này.*

Chạy lệnh ***pgrep -l python*** để kiểm tra dữ liệu vẫn được nạp vào bảng. 
```python
pgrep -l python
3257 python
```

Và bạn đã hoàn thành quá trình Thiết lập và sẵn sàng cho những bài thực hành rất thú vị ở phía trước!