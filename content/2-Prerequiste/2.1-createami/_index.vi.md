---
title : "Tạo IAM Role"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b>2.1 </b> "
---

### Tạo Role

1. Truy cập [giao diện quản trị dịch vụ IAM](https://us-east-1.console.aws.amazon.com/iamv2/home)

- Chọn **Role**.
- Chọn **Create role**.

![IAM](/images/2.prerequisite/001-createiam.png)

2. Tại trang **Create role** mục **Select trusted entity**.

- Tại mục **Trusted entity type** chọn **AWS service**.
- Tại mục **Use case** chọn **EC2**.
- Chọn **Next**.

![IAM](/images/2.prerequisite/001-createroleadmin.png)

3. Tại mục **Add permissions**.

- Chọn **AdministratorAccess**.
- Chọn **Next**.

![IAM](/images/2.prerequisite/003-createroleadmin.png)

4. Mục **Name, review and create**.

- Tại mục **Role name** nhập **Admin-Role**.

![IAM](/images/2.prerequisite/004-createroleadmin.png)

- Kiêm tra **Add permissions**
- Chọn **Create role**.

![IAM](/images/2.prerequisite/005-createroleadmin.png)


