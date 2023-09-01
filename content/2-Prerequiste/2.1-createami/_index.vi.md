---
title : "Tạo IAM Role"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b>2.1 </b> "
---
## Create Role

1. Access the [IAM service management console](https://us-east-1.console.aws.amazon.com/iamv2/home).

- Select **Role**.
- Choose **Create role**.

![IAM](/images/2.prerequisite/001-createiam.png)

2. On the **Create role** page, under **Select trusted entity**.

- Under **Trusted entity type**, select **AWS service**.
- Under **Use case**, select **EC2**.
- Choose **Next**.

![IAM](/images/2.prerequisite/001-createroleadmin.png)

3. Under **Add permissions**.

- Select **AdministratorAccess**.
- Choose **Next**.

![IAM](/images/2.prerequisite/003-createroleadmin.png)

4. In the **Name, review, and create** section.

- Under **Role name**, enter **Admin-Role**.

![IAM](/images/2.prerequisite/004-createroleadmin.png)

- Review **Add permissions**
- Choose **Create role**.

![IAM](/images/2.prerequisite/005-createroleadmin.png)


