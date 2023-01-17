---
title : "Create CloudFormation Stack"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

{{% notice warning %}}

Throughout the exercise, we will work with DynamoDB tables that can cost tens or hundreds of dollars per day. Be sure to delete the CloudFormation stack and all DynamoDB tables with the DynamoDB console right after finishing the exercise.

{{% /notice %}}

1. Go to [AWS Management Console](https://aws.amazon.com/console/)

- Find **CloudFormation**
- Select **CloudFormation**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0001-createcloudformationstack.png)

2. In the **AWS CloudFormation** interface

- Select **Create stack**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0002-createcloudformationstack.png)

- Download file **lab.yaml** [here](https://s3.amazonaws.com/amazon-dynamodb-labs.com/assets/lab.yaml)

- Alternatively, to initialize CloudFormation template in region **US East 1** to deploy necessary resources by clicking [Launch Stack](https://us-east-1.console.aws.amazon .com/cloudformation/home?region=us-east-1#/stacks/create/template?stackName=amazon-dynamodb-labs&templateURL=https://s3.amazonaws.com/amazon-dynamodb-labs.com/assets/ lab.yaml)

- Or you can download [CloudFormation template YAML](https://s3.amazonaws.com/amazon-dynamodb-labs.com/assets/lab.yaml) and create it yourself.

3. In the **Create stack** interface

- Select **Template is ready**
- Select **Upload a template file**
- Select **Choose file**
- Select **lab.yaml**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0003-createcloudformationstack.png)

4. In the **Create stack** interface

- **Stack name**, enter ```amazon-dynamodb-labs```
- **InstanceType**, enter ```t3.xlarge```
- **VPCSelection**, select **CreateNewVPC**
- **WorkshopCodeURL**, keep default value
- Select **Next**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0004-createcloudformationstack.png)

5. Select **Next**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0005-createcloudformationstack.png)

6. In the **Create stack** interface

- Select **I acknowledge that AWS CloudFormation might create IAM resources**
- Select **Create stack**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0006-createcloudformationstack.png)

7. After about 5 minutes, create the stack successfully. The Stack will create **an EC2 Instance** with an associated **Role**. It also creates additional Roles for the **AWS Lambda function** that will be used throughout this exercise.

- Select the stack just created
- Select **Events**
- See the stack creation process

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0007-createcloudformationstack.png)

8. In the **amazon-dynamodb-labs** interface

- Select **Outputs**
- See the output value as the ec2 instance used for the lab.

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0008-createcloudformationstack.png)

9. Check ec2 instance initialize successfully

- Go to [AWS Management Console](https://aws.amazon.com/console/)
- Find **EC2**
- Select **EC2**

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/0009-createcloudformationstack.png)

10. In the **EC2** interface

- Select **Instances**
- Select **instance** just created
- View instance details

![Create CloudFormation Stack](/images/2-Prerequiste/2.1-CreateCloudFormationStack/00010-createcloudformationstack.png)