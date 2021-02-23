+++
title = "Advanced Design Patterns for Amazon DynamoDB"
date = 2020
weight = 1
chapter = true
+++

# Một số Mẫu Kiến trúc Nâng cao Sử dụng Dịch vụ Amazon DynamoDB 

Trong workshop này, chúng ta sẽ cùng nhau tìm hiểu một số mấu kiến trúc nâng cao sử dụng dịch vụ Amazon DynamoDB và các phương pháp hay nhất để xây dựng các ứng dụng có khả năng mở rộng mạnh mẽ mà đã được tối ưu hóa về hiệu suất và chi phí.  

Để có thể dễ dàng thực hành theo workshop, chúng tôi đã chuẩn bị các tập lệnh Python để giúp dựng sẵn các mẫu kiến trúc sử dụng trong workshop. Và cho tới phần cuối của workshop, bạn sẽ có đủ kiến thức để xây dựng và giám sát các ứng dụng sử dụng dịch vụ DynamoDB mà có thể phát triển đến bất kỳ kích thước và quy mô nào.  

Workshop bao gồm những nội dung chính sau đây:
#### [Setup](http://localhost:1313/0-setup/)
Thiết lập môi trường thực hành và kết nối với EC2 Instance mà sẽ được sử dụng trong bài thực hành.
    
#### [Lab 1 - DynamoDB Capacity Units và Partitioning](http://localhost:1313/1-dynamodb-capacity-units-and-partitioning/)
Tìm hiểu về dung lượng được cung cấp.
    
#### [Lab 2 - Các kiểu Quét dữ liệu bảng: Tuần tự (Sequential) và Song song (Parallel)](http://localhost:1313/2-sequential-and-parallel-table-scans/)
Tìm hiểu sự khác biệt giữa quét tuần tự và quét song song.
    
#### [Lab 3 - Global Secondary Index Write Sharding](http://localhost:1313/3-global-secondary-index-write-sharding/)
Truy vấn với chỉ mục thứ cấp toàn cục được phân đoạn giúp cho việc đọc dữ liệu trở nên nhanh chóng hơn do chúng đã được sắp xếp theo mã trạng thái và ngày tháng.
    
#### [Lab 4 - Global Secondary Index cục Key Overloading](http://localhost:1313/4-global-secondary-index-key-overloading/)
Tìm hiểu cách duy trì khả năng truy vấn trên nhiều thuộc tính khi bạn có một bảng nhiều thực thể.

#### [Lab 5 - Sparse Global Secondary Indexes](http://localhost:1313/5-sparse-global-secondary-indexes/)
Tìm hiểu cách cắt giảm các tài nguyên đáp ứng cho việc truy vấn trên các thuộc tính không phổ biến.
	
#### [Lab 6 - Composite Keys](http://localhost:1313/6-composite-keys/)
Tìm hiểu cách kết hợp hai thuộc tính thành một để tận dụng những ưu điểm của sort key trong DynamoDB.
	
#### [Lab 7 - Adjacency Lists](http://localhost:1313/7-adjacency-lists/)
Tìm hiểu cách lưu trữ nhiều loại thực thể trong một bảng DynamoDB.
    
#### [Lab 8 - Amazon DynamoDB Streams và AWS Lambda](http://localhost:1313/8-amazon-dynamodb-streams-and-aws-lambda/)
Tìm hiểu cách xử lý các hạng mục trong DynamoDB với AWS Lambda khi xuất hiện các trigger vô hạn.
	
### Đối tượng mục tiêu
Workshop phù hợp với đối tượng là những nhà phát triển ứng dụng, kĩ sư phần mềm, và các nhà quản trị cơ sở dữ liệu mà có tham gia vào công việc thiết kế và bảo trì các ứng dụng sử dụng dịch vụ DynamoDB.

### Yêu cầu
#### Kiến thức cơ bản về các dịch vụ AWS  

Cùng với những dịch vụ khác, workshop sẽ hướng dẫn bạn cách sử dụng Trình Quản lý Phiên của AWS Systems Manager và AWS Lambda


#### Hiểu biết cơ bản về DynamoDB

Nếu bạn chưa từng làm việc với DynamoDB thì xin hãy tham khảo tài liệu [Amazon DynamoDB là gì?](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html) trước khi bắt đầu workshop

### Tham khảo

[1] Tài liệu tham khảo chính [Advanced Design Patterns for Amazon DynamoDB](https://amazon-dynamodb-labs.com/design-patterns.html)  
[2] Tài liệu tham khảo [Core Components of Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html#HowItWorks.CoreComponents.PrimaryKey)  
[3] Tài liệu tham khảo [Database sharding là gì?](https://viblo.asia/p/database-sharding-la-gi-Az45boQVKxY)
[4] Video giới thiệu các mẫu kiến trúc nâng cao sử dụng DynamoDB {{< youtube 6yqfmXiZTlM >}}

