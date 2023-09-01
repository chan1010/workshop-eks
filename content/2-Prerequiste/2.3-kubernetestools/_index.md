---
title: "Install Tools"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b>2.3 </b>"
---

1. Copy and paste the following command into the **Terminal of the Cloud9 Workspace** to install **eksctl**.
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv -v /tmp/eksctl /usr/local/bin
```
![KUBERNETESTOOL](/images/2.prerequisite/001-installkubernetestool.png)

- Enable bash-completion for the **eksctl** tool by running the following command.
```
eksctl completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
```

2. Copy and paste the following command into the **Terminal of the Cloud9 Workspace** to install **kubectl**.
```
sudo curl --silent --location -o /usr/local/bin/kubectl \
   https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl

kubectl version --short --client
```
![KUBERNETESTOOL](/images/2.prerequisite/002-installkubernetestool.png)

- Enable auto-completion for the **kubectl** tool by running the following command.
```
kubectl completion bash >>  ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
```

3. Copy and paste the following command into the **Terminal of the Cloud9 Workspace** to update **awscli**.
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
![KUBERNETESTOOL](/images/2.prerequisite/003-installkubernetestool.png)

4. Copy and paste the following command into the **Terminal of the Cloud9 Workspace** to install **jq**.
```
sudo yum install -y jq
```
![KUBERNETESTOOL](/images/2.prerequisite/004-installkubernetestool.png)

5. Next, we will configure the aws cli to use the current Region.

```
export AWS_DEFAULT_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
export AZS=($(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text --region $AWS_DEFAULT_REGION))
```

We should set the aws cli configuration with your current region as default.

```
export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
export AZS=($(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text --region $AWS_REGION))
```

Check if AWS_REGION is set to the desired region.
```
test -n "$AWS_REGION" && echo AWS_REGION is "$AWS_REGION" || echo AWS_REGION is not set
```

Save them in bash_profile.
```
echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
echo "export AZS=(${AZS[@]})" | tee -a ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region
```

Authenticate the IAM role.
```
aws sts get-caller-identity --query Arn | grep Admin-Role -q && echo "IAM role valid" || echo "IAM role NOT valid"
```

If the IAM role is not valid, DO NOT CONTINUE. Go back and verify the steps on this page.