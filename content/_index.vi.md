---
title : "Amazon DynamoDB nâng cao"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---

# Amazon DynamoDB nâng cao

#### Tổng quan

Trong workshop này, chúng ta sẽ cùng nhau tìm hiểu một số mấu kiến trúc nâng cao sử dụng dịch vụ **Amazon DynamoDB** và các phương pháp hay nhất để xây dựng các ứng dụng có khả năng mở rộng mạnh mẽ mà đã được tối ưu hóa về hiệu suất và chi phí.

Để có thể dễ dàng thực hành theo workshop, chúng tôi đã chuẩn bị các tập lệnh Python để giúp dựng sẵn các mẫu kiến trúc sử dụng trong workshop. Và cho tới phần cuối của workshop, bạn sẽ có đủ kiến thức để xây dựng và giám sát các ứng dụng sử dụng dịch vụ **DynamoDB** mà có thể phát triển đến bất kỳ kích thước và quy mô nào.

![DynamoDB](/images/1-Introduce/0001-Diagram-DynamoDB-Tables.png)

#### Nội dung

1. [Giới thiệu](1-introduce)
2. [Các bước chuẩn bị](2-prerequiste)
3. [DynamoDB Capacity Units và Partitioning](3-dynamodbcapacityunits)
4. [Các kiểu Quét dữ liệu bảng: Tuần tự (Sequential) và Song song (Parallel)](4-scan)
5. [Global Secondary Index Write Sharding](5-gsiwritesharding)
6. [Global Secondary Index cục Key Overloading](6-gsikeyoverloading)
7. [Sparse Global Secondary Indexes](7-sparsegsi)
8. [Composite Keys](8-compositekeys)
9. [Adjacency Lists](9-adjacencylists)
10. [Amazon DynamoDB Streams và AWS Lambda](10-dynamodbstreamsandlambda)
11. [Dọn dẹp tài nguyên](11-cleanup)

#### Đối tượng mục tiêu
Workshop phù hợp với đối tượng là những nhà phát triển ứng dụng, kĩ sư phần mềm, và các nhà quản trị cơ sở dữ liệu mà có tham gia vào công việc thiết kế và bảo trì các ứng dụng sử dụng dịch vụ DynamoDB.

#### Yêu cầu
- Kiến thức cơ bản về các dịch vụ AWS
- Cùng với những dịch vụ khác, workshop sẽ hướng dẫn bạn cách sử dụng Trình Quản lý Phiên của AWS Systems Manager và AWS Lambda
- Hiểu biết cơ bản về DynamoDB
- Nếu bạn chưa từng làm việc với DynamoDB thì xin hãy tham khảo tài liệu Amazon DynamoDB là gì? trước khi bắt đầu workshop

#### Tham khảo

1. [Tài liệu tham khảo chính Advanced Design Patterns for Amazon DynamoDB](https://amazon-dynamodb-labs.com/design-patterns.html)
2. [Tài liệu tham khảo Core Components of Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html#HowItWorks.CoreComponents.PrimaryKey)
3. [Tài liệu tham khảo Database sharding là gì?](https://viblo.asia/p/database-sharding-la-gi-Az45boQVKxY)
4. [Video giới thiệu các mẫu kiến trúc nâng cao sử dụng DynamoDB](https://www.youtube.com/watch?v=6yqfmXiZTlM)
