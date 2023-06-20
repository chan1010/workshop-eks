---
title : "Dọn dẹp tài nguyên"
date :  "`r Sys.Date()`" 
weight : 4 
chapter : false
pre : " <b> 4. </b> "
---

Chúng ta sẽ tiến hành các bước sau để xóa các tài nguyên chúng ta đã tạo trong bài thực hành này.

1. Truy cập [giao diện quản trị dịch vụ cloudformation](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home)
- Lần lượt chọn tất cả các **stack** để xóa.
- Chọn **Delete**.
![CLEAR](/images/4.clean/001-clean.png)

- Sau quá trình chọn **Delete** các **stack** sẽ được khởi tạo xóa

![CLEAR](/images/4.clean/002-clean.png)
2. Truy cập [giao diện quản trị dịch vụ ECR](https://ap-southeast-1.console.aws.amazon.com/ecr/repositories)
- Chọn repository **front-end**
- Chọn **Delete**
![CLEAR](/images/4.clean/003-clean.png)
Sau khi màn hình xác nhận xuất hiện:
- Nhập `delete`
- Chọn **Delete**
![CLEAR](/images/4.clean/004-clean.png)
3. Truy cập [giao diện quản trị dịch vụ EC2](https://ap-southeast-1.console.aws.amazon.com/ec2) xóa Load Balancer
- Chọn tab **Load Balancers**
- Chọn **k8s-frontdend**
- Chọn **Actions**
- Chọn **Delete load balancer**
![CLEAR](/images/4.clean/007-clean.png)
Tại trang xác nhận delete
- Nhập `confirm` vào ô xác nhận
- Chọn **Delete**
![CLEAR](/images/4.clean/008-clean.png)
4. Truy cập [giao diện quản trị dịch vụ VPC](https://ap-southeast-1.console.aws.amazon.com/vpc/home) để xóa NAT gateway
- Chọn tab **NAT gateways**
- Chọn NAT gateway có tên **nat**
- Chọn **Actions**
- Chọn **Delete NAT Gateway**
![CLEAR](/images/4.clean/005-clean.png)
Tại trang xác nhận xóa
- Nhập `delete` vào ô xác nhận
- Chọn **Delete** để xác nhận xóa
![CLEAR](/images/4.clean/006-clean.png)
5. Truy cập [giao diện quản trị dịch vụ VPC](https://ap-southeast-1.console.aws.amazon.com/vpc/home) để xóa VPC
- Chọn tab **your VPCs**
- Chọn vpc name **k8s-vpc**
- chọn **Actions**
- chọn **Delete VPC**
![CLEAR](/images/4.clean/009-clean.png)
Tại trang xác nhận xóa VPC
- Nhập `delete` vào ô xác nhận
- Chọn **Delete**
![CLEAR](/images/4.clean/010-clean.png)