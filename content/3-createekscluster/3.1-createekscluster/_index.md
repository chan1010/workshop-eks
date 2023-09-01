---
title: "Create EKS Cluster"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b>3.1 </b>"
---

1. Prepare the EKS cluster configuration file
First, we need to retrieve the output of VPC resources to deploy an EKS cluster for the newly created VPC structure.
- Copy and paste the following commands into the **Cloud9 Workspace Terminal**:

```
# Export VPC
export VPC_ID=$(terraform output -json | jq -r .vpc_id.value)
# Export private subnets
export PRIVATE_SUBNETS_ID_A=$(terraform output -json | jq -r .private_subnets_id.value[0])
export PRIVATE_SUBNETS_ID_B=$(terraform output -json | jq -r .private_subnets_id.value[1])
export PRIVATE_SUBNETS_ID_C=$(terraform output -json | jq -r .private_subnets_id.value[2])
# Export public subnets
export PUBLIC_SUBNETS_ID_A=$(terraform output -json | jq -r .public_subnets_id.value[0])
export PUBLIC_SUBNETS_ID_B=$(terraform output -json | jq -r .public_subnets_id.value[1])
export PUBLIC_SUBNETS_ID_C=$(terraform output -json | jq -r .public_subnets_id.value[2])
```

![CREATECLUSTER](/images/3.createekscluster/001-createcluster.png)

```
echo "VPC_ID=$VPC_ID, \
PRIVATE_SUBNETS_ID_A=$PRIVATE_SUBNETS_ID_A, \
PRIVATE_SUBNETS_ID_B=$PRIVATE_SUBNETS_ID_B, \
PRIVATE_SUBNETS_ID_C=$PRIVATE_SUBNETS_ID_C, \
PUBLIC_SUBNETS_ID_A=$PUBLIC_SUBNETS_ID_A, \
PUBLIC_SUBNETS_ID_B=$PUBLIC_SUBNETS_ID_B, \
PUBLIC_SUBNETS_ID_C=$PUBLIC_SUBNETS_ID_C"
```

![CREATECLUSTER](/images/3.createekscluster/002-createcluster.png)

2. Create the EKS Cluster configuration file
- Copy and paste the following commands into the **Cloud9 Workspace Terminal**:

```
cat > /home/ec2-user/environment/eks-cluster.yaml <<EOF
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: web-host-on-eks
  region: ${AWS_REGION}
  version: "1.25"
privateCluster:
  enabled: true
  additionalEndpointServices:
  - cloudformation
  - logs
  - autoscaling
vpc:
  id: "${VPC_ID}"
  subnets:
    private:
      private-ap-east-1a:
        id: "${PRIVATE_SUBNETS_ID_A}"
      private-ap-east-1b:
        id: "${PRIVATE_SUBNETS_ID_B}"
      private-ap-east-1c:
        id: "${PRIVATE_SUBNETS_ID_C}"
managedNodeGroups:
- name: managed
  labels:
    role: worker
  instanceType: t3.small
  minSize: 3
  desiredCapacity: 3
  maxSize: 10
  privateNetworking: true
  volumeSize: 50
  volumeType: gp2
  volumeEncrypted: true
  iam:
    withAddonPolicies:
      autoScaler: true
      albIngress: true
      cloudWatch: true
  ssh:
    allow: true
    publicKeyPath: ~/.ssh/id_rsa.pub
  subnets:
    - private-ap-east-1a
    - private-ap-east-1b
    - private-ap-east-1c
cloudWatch:
  clusterLogging:
    enableTypes: ["all"]
addons:
- name: vpc-cni
  attachPolicyARNs:
    - arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
- name: coredns
  version: latest
- name: kube-proxy
  version: latest
iam:
  withOIDC: true
  serviceAccounts:
  - metadata:
      name: aws-load-balancer-controller
      namespace: kube-system
    wellKnownPolicies:
      awsLoadBalancerController: true
  - metadata:
      name: cluster-autoscaler
      namespace: kube-system
      labels: {aws-usage: "cluster-ops"}
    wellKnownPolicies:
      autoScaler: true
EOF
```

3. Create the default SSH key in the Cloud9 environment

```
ssh-keygen
```

After running the command, please press **ENTER** to keep all default input values.

![CREATECLUSTER](/images/3.createekscluster/003-createcluster.png)

4. Create the EKS Cluster using the yaml configuration file we just created

```
eksctl create cluster -f /home/ec2-user/environment/eks-cluster.yaml
```

- The result of running the command will look like below

![CREATECLUSTER](/images/3.createekscluster/004-createcluster.png)

During the waiting time, you can navigate to the AWS Console and check the [CloudFormation service](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home) page to monitor the status of the ongoing CloudFormation stack.

![CREATECLUSTER](/images/3.createekscluster/005-createcluster.png)

{{%notice note%}}
It takes about 45 minutes to complete the EKS Cluster creation process.
{{%/notice%}}

- After successful creation, you will see other stacks created successfully afterward.

![CREATECLUSTER](/images/3.createekscluster/006-createcluster.png)

5. Access the [EKS console](https://ap-southeast-1.console.aws.amazon.com/eks/home) to check the newly created clusters.

![CREATECLUSTER](/images/3.createekscluster/007-createcluster.png)