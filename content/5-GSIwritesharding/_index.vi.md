---
title : "Global Secondary Index Write Sharding"
date :  "`r Sys.Date()`" 
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Tổng quan 

Khóa chính của bảng Cơ sở hoặc bảng Chỉ mục GSI trong DynamoDB bao gồm partition key và sort key (tùy chọn).
Cách bạn thiết kế các khóa cho bảng Chỉ mục đóng vai trò cực kỳ quan trọng đối với cấu trúc và hiệu suất sử dụng của cơ sở dữ liệu mà bạn xây dựng.
Giá trị của Partition key sẽ giúp xác định cách chia CSDL ban đầu thành các phân vùng logic, nơi mà dữ liệu hay các item được lưu trữ.
Việc chọn giá trị partition key đúng giúp cho workload của database được phân tán đồng đều trên tất cả các phân vùng trong bảng Cơ sở hoặc bảng Chỉ mục GSI.

Trong bài thực hành này, chúng ta sẽ học cách xây dựng bảng Chỉ mục GSI Write Sharding, giúp tăng hiệu suất bằng cách truy vấn các item trải trên nhiều phân vùng logic khác nhau một cách có chọn lọc.
Quay trở lại ví dụ về nhật ký truy cập máy chủ từ Bài thực hành số 1, dựa trên nhật ký truy cập dịch vụ Apache. Lần này, ta sẽ truy vấn các item có mã phản hồi 4xx.
Lưu ý rằng các item có mã phản hồi 4xx chiếm tỷ lệ rất nhỏ và không được phân bố một cách đồng đều theo mã phản hồi trong bảng dữ liệu.

Biểu đồ sau đây cho thấy sự phân bố của các bản ghi nhật ký dựa theo mã phản hồi trong file mẫu **logfile_medium1.csv**.

![DynamoDB](/images/1-Introduce/0007-Diagram-DynamoDB-Tables.png)

Chúng ta sẽ tạo một bảng Chỉ mục GSI write sharding trên một bảng để ngẫu nhiên hóa các lần ghi giá trị partition key lên phân vùng logic, việc này giúp tăng thông lượng đọc/ghi của ứng dụng.
Để bắt đầu, ta tạo một số ngẫu nhiên từ một tập hợp cố định (ví dụ: 1 đến 10) và sử dụng số này làm partition key cho bảng Chỉ mục GSI. Giá trị partition key ngẫu nhiên mà được ghi vào các phân vùng của bảng Chỉ mục được trải đều trên tất cả các giá trị partition key thuộc tập hợp đã xác định ở trên (từ 1->10) và nó độc lập với tất cả các thuộc tính còn lại. Điều này đem tới khả năng xử lý song song tốt hơn và thông lượng tổng thể cao hơn.

Tóm lại trong bài thực hành này, chúng ta sẽ tạo ra một bảng Chỉ mục mới sử dụng giá trị Ngẫu nhiên làm Partition key, và khóa tổng hợp responsecode#date#hourofday làm Sort key. Bảng logfile_scan mà chúng ta đã tạo ở bước chuẩn bị đã có sẵn 2 thuộc tính này. Cụ thể đoạn mã khởi tạo chúng là:

```
SHARDS = 10
newitem['GSI_1_PK'] = "shard#{}".format((newitem['requestid'] % SHARDS) + 1)
newitem['GSI_1_SK'] = row[7] + "#" + row[2] + "#" + row[3]
```

1. Xem thông tin của bảng Chỉ mục

- Bảng Chỉ mục GSI đã được tạo ở phần thiết lập của workshop. Bạn có thể xem mô tả của bảng Chỉ mục bằng lệnh bên dưới

```
aws dynamodb describe-table --table-name logfile_scan --query "Table.GlobalSecondaryIndexes"
```

**Nội dung mô tả tương tự như sau:**

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
- DynamoDB ItemCount trong ví dụ này có giá trị bằng không.
- DynamoDB tính toán tổng số item được nhìn thấy bởi API này nhiều lần trong ngày và giá trị này thực tế có thể khác với kết quả hiển thị trong ví dụ.

![Global Secondary Index Write Sharding](/images/5-GSIwritesharding/0001-gsiwritesharing.png)

2. **Truy vấn bảng Chỉ mục GSI Write Sharding**

- Để lấy được tất cả bản ghi có mã phản hồi 404, bạn cần truy vấn trên toàn bộ phân vùng chỉ mục phụ toàn cục sử dụng sort key.

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

Chạy script sau để lấy các items từ bảng Chỉ mục GSI Write Sharding chỉ với partition key và mã phản hồi

```
python query_responsecode.py logfile_scan 404
```

Lệnh sẽ truy vấn trên bảng logfile_scan để tìm kiếm những item có sort keys=404 sử dụng tham số begins_with.
Một truy vấn chạy trên từng đoạn của bảng chỉ mục GSI và kết quả đẩy về cho client. Đầu ra của script tương tự như bên dưới

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

3. Bạn có thể truy vấn trên cùng bảng chỉ mục, nhưng bổ sung thêm ngày tháng cho lệnh chạy script. Cụ thể, thực hiện truy vấn bảng **logfile_scan** để tìm các item có sort key bắt là **404#2017-07-21** sử dụng tham số **begins_with**.

```
python query_responsecode.py logfile_scan 404 --date 2017-07-21
```
Kết quả lệnh chạy tương tự bên dưới

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

#### Kết luận 

Trong bài thực hành này, chúng ta đã sử dụng bảng Chỉ mục GSI write sharding để truy xuất nhanh chóng các kết quả đã được sắp xếp, sử dụng các khóa tổng hợp sẽ được đề cập sâu hơn trong bài thực hành số 6.
Bảng Chỉ mục GSI write sharding trong ví dụ trên sử dụng một loạt các khóa partition key có giá trị từ 0 đến 9, nhưng trong ứng dụng của bạn, bạn có thể chọn bất kỳ phạm vi nào.
Đồng thời, bạn có thể thêm nhiều phân đoạn hơn khi số lượng item được lập chỉ mục tăng lên. Trong mỗi phân đoạn, dữ liệu được sắp xếp dựa theo sort key. Điều này cho phép chúng ta truy xuất nhật ký truy cập máy chủ đã được sắp xếp theo **#response code** và **#date**, ví dụ: **404#2017-07-21**.




