---
title: "Cài đặt công cụ"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: " <b> 2.3 </b> "
---

1. Copy và Paste đoạn lệnh dưới đây vào **Terminal của Cloud9 Workspace** để cài đặt **eksctl**.
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv -v /tmp/eksctl /usr/local/bin

```
![KUBERNETESTOOL](/images/2.prerequisite/001-installkubernetestool.png)
- Kích hoạt tính năng bash-completion cho công cụ **eksctl** bàng chạy lệch dưới đây
```
eksctl completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
```
2. Copy và Paste đoạn lệnh dưới đây vào **Terminal của Cloud9 Workspace** để cài đặt **kubectl**.
```
sudo curl --silent --location -o /usr/local/bin/kubectl \
   https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl

kubectl version --short --client
```
![KUBERNETESTOOL](/images/2.prerequisite/002-installkubernetestool.png)
- Kích hoạt tính năng tự động hoàn tất cho công cụ **kubectl** bằng cách chạy lệnh dưới đây
```
kubectl completion bash >>  ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
```
3. Copy và Paste đoạn lệnh dưới đây vào **Terminal của Cloud9 Workspace** để cập nhật **awscli**
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
![KUBERNETESTOOL](/images/2.prerequisite/003-installkubernetestool.png)
4. Copy và Paste đoạn lệnh dưới đây vào **Terminal của Cloud9 Workspace** cài đặt **jq**
```
sudo yum install -y jq
```
![KUBERNETESTOOL](/images/2.prerequisite/004-installkubernetestool.png)

5. Tiếp theo chúng ta sẽ cấu hình aws cli sử dụng Region hiện tại.

```
export AWS_DEFAULT_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
export AZS=($(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text --region $AWS_DEFAULT_REGION))
```

Chúng ta nên đặt cấu hình aws cli với khu vực hiện tại của bạn làm mặc định.

```
export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
export AZS=($(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text --region $AWS_REGION))
```

Kiểm tra xem AWS_REGION có được đặt thành vùng mong muốn không
```
test -n "$AWS_REGION" && echo AWS_REGION is "$AWS_REGION" || echo AWS_REGION is not set
```
Hãy lưu chúng vào bash_profile

```
echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
echo "export AZS=(${AZS[@]})" | tee -a ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region
```
Xác thực vai trò IAM
```
aws sts get-caller-identity --query Arn | grep Admin-Role -q && echo "IAM role valid" || echo "IAM role NOT valid"
```
Nếu vai trò IAM không hợp lệ, KHÔNG TIẾP TỤC. Quay lại và xác nhận các bước trên trang này.