---
title: "Cleanup"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

In this section, we will perform the following steps to clean up the resources we created in this hands-on lab.

1. Access the [CloudFormation service management interface](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home).
   - Select all the **stacks** one by one to delete.
   - Choose **Delete**.
   ![CLEAN](/images/4.clean/001-clean.png)

   - After selecting **Delete**, the stacks will be initiated for deletion.
   ![CLEAN](/images/4.clean/002-clean.png)

2. Access the [ECR service management interface](https://ap-southeast-1.console.aws.amazon.com/ecr/repositories).
   - Select the **front-end** repository.
   - Choose **Delete**.
   ![CLEAN](/images/4.clean/003-clean.png)

   - After the confirmation screen appears:
     - Enter `delete`.
     - Choose **Delete**.
   ![CLEAN](/images/4.clean/004-clean.png)

3. Access the [EC2 service management interface](https://ap-southeast-1.console.aws.amazon.com/ec2) to delete the Load Balancer.
   - Choose the **Load Balancers** tab.
   - Select **k8s-frontdend**.
   - Choose **Actions**.
   - Choose **Delete load balancer**.
   ![CLEAN](/images/4.clean/007-clean.png)

   - On the delete confirmation page:
     - Enter `confirm` in the confirmation box.
     - Choose **Delete**.
   ![CLEAN](/images/4.clean/008-clean.png)

4. Access the [VPC service management interface](https://ap-southeast-1.console.aws.amazon.com/vpc/home) to delete the NAT gateway.
   - Choose the **NAT gateways** tab.
   - Select the NAT gateway with the name **nat**.
   - Choose **Actions**.
   - Choose **Delete NAT Gateway**.
   ![CLEAN](/images/4.clean/005-clean.png)

   - On the delete confirmation page:
     - Enter `delete` in the confirmation box.
     - Choose **Delete** to confirm the deletion.
   ![CLEAN](/images/4.clean/006-clean.png)

5. Access the [VPC service management interface](https://ap-southeast-1.console.aws.amazon.com/vpc/home) to delete the VPC.
   - Choose the **Your VPCs** tab.
   - Select the VPC with the name **k8s-vpc**.
   - Choose **Actions**.
   - Choose **Delete VPC**.
   ![CLEAN](/images/4.clean/009-clean.png)

   - On the delete confirmation page:
     - Enter `delete` in the confirmation box.
     - Choose **Delete**.
   ![CLEAN](/images/4.clean/010-clean.png)
