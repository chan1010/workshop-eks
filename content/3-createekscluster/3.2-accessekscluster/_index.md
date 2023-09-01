---
title: "Access EKS Cluster"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: " <b> 3.2 </b> "
---

Since we have created a private cluster and its API endpoint can only be accessed within the VPC, we will need to create a Cloud9 environment inside the k8s VPC.

### Create Cloud9 Inside the k8s VPC

1. Access the [Cloud9 service management interface](https://ap-southeast-1.console.aws.amazon.com/cloud9control/home)

- Choose **Create environment**.

![CLOUD9](/images/2.prerequisite/001-createcloud9.png)

2. On the **Create environment** page.

- In the **Name** section, enter `eks-bastion`

![CLOUD9](/images/3.createekscluster/002-createcloud9.png)

3. In the Network settings

- In the **Amazon Virtual Private Cloud (VPC)** choose **k8s-vpc**
- In the **Subnet** choose **k8s-ap-southeast-1a-public-subnet**
 ![CLOUD9](/images/3.createekscluster/003-createcloud9.png)
- Click **Create** to create the environment.
 ![CLOUD9](/images/3.createekscluster/004-createcloud9.png)

5. Interface after successfully creating the environment

- Click **Open** to open the newly created environment.

![CLOUD9](/images/3.createekscluster/005-createcloud9.png)

### Assign IAM Role to Cloud9

1. In the AWS Cloud9 interface

- Click the icon **R**
- Choose **Manage EC2 Intance**

![CLOUD9](/images/3.createekscluster/006-createcloud9.png)

2. In the **EC2** interface

- Select **Intance**
- Choose **aws-cloud9-eks-bastion**
- Click **Actions**
- Select **Security**
- Choose **Modify IAM role**

![CLOUD9](/images/3.createekscluster/007-createcloud9.png)

3. In the IAM role editing interface

- In the **IAM role**, select **Admin-Role**
- Click **Update IAM role**

![CLOUD9](/images/3.createekscluster/008-createcloud9.png)

### Update Cloud9 Configuration

{{%notice tip%}}
Cloud9 will automatically manage IAM credentials information. The current default configuration is not compatible with EKS credentials via IAM, so we need to disable this feature and use an IAM Role.
{{%/notice%}}

1. In the AWS Cloud9 interface,

- Click **AWS Cloud9**
- Select **Preferences**

![CLOUD9](/images/2.prerequisite/009-createcloud9.png)

2. In the AWS Cloud9 interface

- Choose **AWS Settings**
- Uncheck **AWS managed temporary credentials**

![CLOUD9](/images/3.createekscluster/010-createcloud9.png)

3. To ensure that temporary credentials information is not stored in Cloud9, we will delete all existing credential information with the following command.

```
rm -vf ${HOME}/.aws/credentials
```

![CLOUD9](/images/3.createekscluster/011-createcloud9.png)

### Update Kube Configuration File Information

In the Cloud9 environment hosted on **web-host-on-eks-workshop**, we will copy the kubeconfig file to the **eks-bastion** environment that you have just created. The **kubeconfig** file is located at **~/.kube/config**, and eksctl will automatically update this file after creation by default.

1. Copy the contents of the eks **web-host-on-eks-workshop** configuration file.

```
cat ~/.kube/config
```

![CLOUD9](/images/3.createekscluster/012-createcloud9.png)

2. Paste the configuration file contents into the **eks-bastion** environment.

```
mkdir ~/.kube
cat > ~/.kube/config << EOF
 <PASTE YOUR KUBE CONFIG FILE HERE>
EOF
chmod 600 ~/.kube/config
```

![CLOUD9](/images/3.createekscluster/013-createcloud9.png)

### Install kubectl

- Copy and paste the following command into the Terminal of Cloud9 Workspace **eks-bastion** to install kubectl.

```
sudo curl --silent --location -o /usr/local/bin/kubectl \
  https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl

kubectl version --short --client
```

![CLOUD9](/images/3.createekscluster/014-createcloud9.png)

- Activate the auto-completion feature for the kubectl tool by running the following command:

```
kubectl completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
```

![CLOUD9](/images/3.createekscluster/015-createcloud9.png)

#### Copy and paste the following command into the Terminal of Cloud9 Workspace to update awscli

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Install inbound rule security group inbound rule in the EKS cluster

1. Access the [EKS service management interface](https://ap-southeast-1.console.aws.amazon.com/eks/home).

- Choose **web-host-on-eks**.

![CLOUD9](/images/3.createekscluster/001-updateeks.png) 

2. In the **web-host-on-eks** cluster,

- Select the **Networking** tab.
- Select **Additional security groups**.

![CLOUD9](/images/3.createekscluster/002-updateeks.png)

3. On the security Groups page,

- Select the **Inbound rules** tab.
- Click **Edit inbound rules**.

![CLOUD9](/images/3.createekscluster/003-updateeks.png)

4. On the **Edit inbound rules** page,

- Click **Add rule**.
- For **Type**, select **All TCP**.
- For **Source**, select **aws-cloud9-eks-bastion**.
- Click **Save rules**.

![CLOUD9](/images/3.createekscluster/004-updateeks.png)

5. After successfully updating, access the **eks-bastion** environment and access the EKS cluster using kubectl.

```
kubectl get nodes
```

![CLOUD9](/images/3.createekscluster/001-accessclustereks.png)

```
kubectl get pod -A
```

![CLOUD9](/images/3.createekscluster/002-accessclustereks.png)