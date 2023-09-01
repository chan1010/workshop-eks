---
title: "Create Cloud9 Workspace"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b>2.2 </b>"
---

### Create Cloud9 Workspace

1. Access the [AWS Cloud9 service management console](https://ap-southeast-1.console.aws.amazon.com/cloud9control/home).

- Select **Create environment**.

![CLOUD9](/images/2.prerequisite/001-createcloud9.png)

2. On the **Create environment** page.

- Under **Name**, enter `web-host-on-eks-workshop`.

![CLOUD9](/images/2.prerequisite/002-createcloud9.png)

- Leave the parameters and configurations as default.
- Scroll down and select **Create**.

![CLOUD9](/images/2.prerequisite/003-createcloud9.png)

3. Interface after successfully creating the environment.
- Select **Open** to open the newly created environment.

![CLOUD9](/images/2.prerequisite/004-createcloud9.png)

4. Interface after opening the **web-host-on-eks-workshop** environment.

![CLOUD9](/images/2.prerequisite/005-createcloud9.png)

### Assign IAM Role to Cloud9

1. In the AWS Cloud9 interface.
- Click on the **R** icon.
- Select **Manage EC2 Instance**.

![CLOUD9](/images/2.prerequisite/006-createcloud9.png)

2. In the **EC2** interface.

- Select **Instances**.
- Choose **aws-cloud9-web-host-on-eks-workshop**.
- Select **Actions**.
- Choose **Security**.
- Choose **Modify IAM role**.

![CLOUD9](/images/2.prerequisite/007-createcloud9.png)

3. In the IAM role edit interface.

- Under **IAM role**, select **Admin-Role**.
- Choose **Update IAM role**.

![CLOUD9](/images/2.prerequisite/008-createcloud9.png)

### Update Cloud9 Configuration

{{%notice tip%}}
Cloud9 will automatically manage IAM credentials information. The default configuration is currently not compatible with EKS IAM credentials; we need to disable this feature and use IAM Role.
{{%/notice%}}

1. In the AWS Cloud9 interface.
- Click **AWS Cloud9**.
- Choose **Preferences**.

![CLOUD9](/images/2.prerequisite/009-createcloud9.png)

2. In the AWS Cloud9 interface.

- Choose **AWS Settings**.
- Uncheck **AWS managed temporary credentials**.

![CLOUD9](/images/2.prerequisite/010-createcloud9.png)

3. To ensure that temporary credentials information is not saved in Cloud9, we will delete all existing credentials information with the following command.

```
aws cloud9 update-environment  --environment-id $C9_PID --managed-credentials-action DISABLE
rm -vf ${HOME}/.aws/credentials
```

![CLOUD9](/images/2.prerequisite/011-createcloud9.png)