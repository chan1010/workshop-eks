---
title: "Tạo Cloud9 workspace"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: " <b> 2.2 </b> "
---

### Tạo Cloud9 workspace

1. Truy cập [giao diện quản trị dịch vụ Cloud9](https://ap-southeast-1.console.aws.amazon.com/cloud9control/home)

- Chọn **Create environment**.

![CLOUD9](/images/2.prerequisite/001-createcloud9.png)

2. Tại trang **Create environment**.
- Tại mục **Name** nhập `web-host-on-eks-workshop`

![CLOUD9](/images/2.prerequisite/002-createcloud9.png)

- Để các tham số và cấu hình như mặc định.
- Kéo xuống dưới và chọn **Create**

![CLOUD9](/images/2.prerequisite/003-createcloud9.png)

3. Giao diện sau khi tạo môi trường thành công
- Chọn **Open** để mở môi trường vừa tạo

![CLOUD9](/images/2.prerequisite/004-createcloud9.png)

4. Dao diện sau khi mở môi trường **web-host-on-eks-workshop**

![CLOUD9](/images/2.prerequisite/005-createcloud9.png)


### Gán IAM role vào cloud9

1. Trong giao diện AWS Cloud9
- Chọn biểu tượng **R**
- Chọn **Manage EC2 Intance**

![CLOUD9](/images/2.prerequisite/006-createcloud9.png)

2. Trong giao diện **EC2**

- chọn **Intance**
- Chọn **aws-cloud9-web-host-on-eks-workshop**
- Chọn **Actions**
- CHọn **Security**
- Chọn **Modify IAM role**

![CLOUD9](/images/2.prerequisite/007-createcloud9.png)

3. Trong giao diện sửa IAM role
- Tại mục **IAM role** chọ **Admin-Role**
- Chọn **Update IAM role**

![CLOUD9](/images/2.prerequisite/008-createcloud9.png)

### Cập nhật cấu hình Cloud9

{{%notice tip%}}
Cloud9 sẽ quản lý thông tin chứng thực IAM một cách tự động. Cấu hình mặc định này hiện tại không tương thích với chứng thực EKS thông qua IAM, chúng ta sẽ cần phải vô hiệu hóa tính năng này và sử dụng IAM Role.
{{%/notice%}}

1. Trong giao diện AWS Cloud9
- Chọn **AWS Cloud9**
- Chọn **Preferences**

![CLOUD9](/images/2.prerequisite/009-createcloud9.png)

2. Trong giao diện AWS Cloud9
- Chọn **AWS Settings**
- Bỏ chọn **AWS managed temporary credentials**

![CLOUD9](/images/2.prerequisite/010-createcloud9.png)

3. Để đảm bảo các thông tin chứng thực tạm không được lưu lại trong Cloud9, chúng ta sẽ xóa tất cả những thông tin chứng thực đang tồn tại bằng câu lệnh dưới đây.

```
aws cloud9 update-environment  --environment-id $C9_PID --managed-credentials-action DISABLE
rm -vf ${HOME}/.aws/credentials
```
![CLOUD9](/images/2.prerequisite/011-createcloud9.png)

