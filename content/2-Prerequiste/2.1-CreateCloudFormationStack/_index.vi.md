---
title : "Tạo CloudFormation Stack"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 2.1 </b> "
---

{{% notice warning %}} 

Xuyên suốt bài thực hành, chúng ta sẽ làm việc với các bảng DynamoDB với chi phí có thể lên tới hàng chục hoặc hàng trăm $ mỗi ngày. Phải chắc chắn bạn đã xóa CloudFormation stack và toàn bộ các bảng DynamoDB với DynamoDB console ngay sau khi kết thúc bài thực hành.

{{% /notice %}}

1. Truy cập [AWS Management Console](https://aws.amazon.com/console/)

- Tìm **CloudFormation**
- Chọn **CloudFormation**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0001-createcloudformationstack.png)

2. Trong giao diện **AWS CloudFormation**

- Chọn **Create stack**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0002-createcloudformationstack.png)

- Tải file **lab.yaml** [tại đây](https://s3.amazonaws.com/amazon-dynamodb-labs.com/assets/lab.yaml)

-  Ngoài ra, để khởi tạo CloudFormation template tại region **US East 1** để triển khai các tài nguyên cần thiết bằng cách bấm vào [Launch Stack](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/template?stackName=amazon-dynamodb-labs&templateURL=https://s3.amazonaws.com/amazon-dynamodb-labs.com/assets/lab.yaml) 

-  Hoặc bạn có thể tải về [CloudFormation template YAML](https://s3.amazonaws.com/amazon-dynamodb-labs.com/assets/lab.yaml) rồi tự khởi tạo.

3. Trong giao diện **Create stack**

- Chọn **Template is ready**
- Chọn **Uplaod a template file**
- Chọn **Choose file**
- Chọn **lab.yaml**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0003-createcloudformationstack.png)

4. Trong giao diện **Create stack**

- **Stack name**, nhập ```amazon-dynamodb-labs```
- **InstanceType**, nhập ```t3.xlarge```
- **VPCSelection**, chọn **CreateNewVPC**
- **WorkshopCodeURL**, giữ giá trị mặc định
- Chọn **Next**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0004-createcloudformationstack.png)

5. Chọn **Next**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0005-createcloudformationstack.png)

6. Trong giao diện **Create stack**

- Chọn **I acknowledge that AWS CloudFormation might create IAM resources**
- Chọn **Create stack**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0006-createcloudformationstack.png)

7. Sau khoảng 5 phút, tạo stack thành công. Stack sẽ tạo ra **một EC2 Instance** với **Role** đi kèm. Ngoài ra nó còn tạo thêm Role cho **AWS Lambda function** sẽ được sử dụng xuyên suốt bài thực hành này.

- Chọn stack vừa tạo
- Chọn **Events**
- Xem quá trình tạo stack

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0007-createcloudformationstack.png)

8. Trong giao diện **amazon-dynamodb-labs**

- Chọn **Outputs**
- Xem giá trị output là ec2 instance sử dụng cho bài lab.

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0008-createcloudformationstack.png)

9. Kiểm tra ec2 instance khởi tạo thành công

- Truy cập [AWS Management Console](https://aws.amazon.com/console/)
- Tìm **EC2**
- Chọn **EC2**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0009-createcloudformationstack.png)

10. Trong giao diện **EC2**

- Chọn **Instances**
- Chọn **instance** vừa tạo
- Xem chi tiết instance

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/00010-createcloudformationstack.png)
