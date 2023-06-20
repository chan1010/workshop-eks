---
title: "Truy cập EKS Cluster"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: " <b> 3.2 </b> "
---

Vì chúng ta đã tạo một cụm riêng tư và điểm cuối API của nó chỉ có thể được truy cập trong VPC, vì vậy chúng ta sẽ cần tạo môi trường Cloud9 trong VPC k8s.

### Tạo Cloud9 trong VPC k8s

1. Truy cập [giao diện quản trị dịch vụ Cloud9](https://ap-southeast-1.console.aws.amazon.com/cloud9control/home)

- Chọn **Create environment**.

![CLOUD9](/images/2.prerequisite/001-createcloud9.png)

2. Tại trang **Create environment**.

- Tại mục **Name** nhập `eks-bastion`

![CLOUD9](/images/3.createekscluster/002-createcloud9.png)

3. Tại mục Network settings

- Tại mục **Amazon Virtual Private Cloud (VPC)** chọn **k8s-vpc**
- Tại mục **Subnet** chọn **k8s-ap-southeast-1a-public-subnet**
  ![CLOUD9](/images/3.createekscluster/003-createcloud9.png)
- Chọn **Create** để tạo môi trường
  ![CLOUD9](/images/3.createekscluster/004-createcloud9.png)

5. Giao diện sau khi tạo môi trường thành công

- Chọn **Open** để mở môi trường vừa tạo

![CLOUD9](/images/3.createekscluster/005-createcloud9.png)

### Gán IAM role vào cloud9

1. Trong giao diện AWS Cloud9

- Chọn biểu tượng **R**
- Chọn **Manage EC2 Intance**

![CLOUD9](/images/3.createekscluster/006-createcloud9.png)

2. Trong giao diện **EC2**

- chọn **Intance**
- Chọn **aws-cloud9-eks-bastion**
- Chọn **Actions**
- CHọn **Security**
- Chọn **Modify IAM role**

![CLOUD9](/images/3.createekscluster/007-createcloud9.png)

3. Trong giao diện sửa IAM role

- Tại mục **IAM role** chọ **Admin-Role**
- Chọn **Update IAM role**

![CLOUD9](/images/3.createekscluster/008-createcloud9.png)

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

![CLOUD9](/images/3.createekscluster/010-createcloud9.png)

3. Để đảm bảo các thông tin chứng thực tạm không được lưu lại trong Cloud9, chúng ta sẽ xóa tất cả những thông tin chứng thực đang tồn tại bằng câu lệnh dưới đây.

```
rm -vf ${HOME}/.aws/credentials
```

![CLOUD9](/images/3.createekscluster/011-createcloud9.png)

### Cập nhật thông tin cấu hình file Kube

Trong môi trường Cloud9 lưu trữ trên **web-host-on-eks-workshop**, chúng ta sẽ sao chép tệp kubeconfig vào môi trường **eks-bastion** mà bạn vừa tạo. Tệp **kubeconfig** nằm trong **~/.kube/config** là do eksctl sẽ tự động cập nhật tệp này sau khi tạo theo mặc định.

1. Sao chép nội dung file cấu hình eks **web-host-on-eks-workshop**

```
cat ~/.kube/config
```

![CLOUD9](/images/3.createekscluster/012-createcloud9.png)

2. Paste nội dung file cấu hình sang mỗi trường **eks-bastion**

```
mkdir ~/.kube
cat > ~/.kube/config << EOF
  <PASTE YOUR KUBE CONFIG FILE HERE>
EOF
chmod 600 ~/.kube/config
```

![CLOUD9](/images/3.createekscluster/013-createcloud9.png)

### Cài đặt Install kubectl

- Copy và Paste đoạn lệnh dưới đây vào Terminal của Cloud9 Workspace **eks-bastion** để cài đặt kubectl.

```
sudo curl --silent --location -o /usr/local/bin/kubectl \
   https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl

kubectl version --short --client
```

![CLOUD9](/images/3.createekscluster/014-createcloud9.png)

- Kích hoạt tính năng tự động hoàn tất cho công cụ kubectl bằng cách chạy lệnh dưới đây

```
kubectl completion bash >>  ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion

```

![CLOUD9](/images/3.createekscluster/015-createcloud9.png)
#### Copy và Paste đoạn lệnh dưới đây vào Terminal của Cloud9 Workspace để cập nhật awscli
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
### Cài đặt quy tắc security group inboud rule trong cụm EKS

1. Truy cập [giao diện quản trị dịch vụ EKS](https://ap-southeast-1.console.aws.amazon.com/eks/home)

- Chọn **web-host-on-eks**

![CLOUD9](/images/3.createekscluster/001-updateeks.png) 

2. Tại cluster **web-host-on-eks**

- Chọn tab **Networking**
- Chọn **Additional security groups**

![CLOUD9](/images/3.createekscluster/002-updateeks.png)
3. Tại trang security Groups
- Chọn tab **Inbound rules**
- Chọn **Edit inbound rules**
![CLOUD9](/images/3.createekscluster/003-updateeks.png)

4. Tại trang **Edit inbound rules**
- Chọn **Add rule**
- Tại mục **Type** chọn **All TCP**
- Tại mục **Source** chọn **aws-cloud9-eks-bastion**
- Chọn **Save rules**
![CLOUD9](/images/3.createekscluster/004-updateeks.png)
5. Sau khi cập nhật thành công truy cập môi trường **eks-bastion** truy cập cụm EKS bằng kubectl

```
kubectl get nodes
```
![CLOUD9](/images/3.createekscluster/001-accessclustereks.png)

```
kubectl get pod -A
```
![CLOUD9](/images/3.createekscluster/002-accessclustereks.png)

