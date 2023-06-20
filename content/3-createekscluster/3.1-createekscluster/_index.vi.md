---
title : "Tạo EKS Cluster"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

1. Chuẩn bị tệp cấu hình EKS cluster
Trước tiên, chúng ta cần lấy đầu ra của các tài nguyên VPC để có thể triển khai cụm EKS cho cấu trúc liên kết VPC vừa tạo.
- Copy và Paste đoạn lệnh dưới đây vào **Terminal của Cloud9 Workspace**
```
# export VPC
export VPC_ID=$(terraform output -json | jq -r .vpc_id.value)
# export private subnets
export PRIVATE_SUBNETS_ID_A=$(terraform output -json | jq -r .private_subnets_id.value[0])
export PRIVATE_SUBNETS_ID_B=$(terraform output -json | jq -r .private_subnets_id.value[1])
export PRIVATE_SUBNETS_ID_C=$(terraform output -json | jq -r .private_subnets_id.value[2])
# export public subnets
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

2. Tạo tệp cấu hình Cụm EKS
- Copy và Paste đoạn lệnh dưới đây vào **Terminal của Cloud9 Workspace**
```
cat > /home/ec2-user/environment/eks-cluster.yaml <<EOF
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: web-host-on-eks
  region: ${AWS_REGION}
  version: "1.25"
privateCluster: # Allows configuring a fully-private cluster in which no node has outbound internet access, and private access to AWS services is enabled via VPC endpoints
  enabled: true
  additionalEndpointServices: # specifies additional endpoint services that must be enabled for private access. Valid entries are: "cloudformation", "autoscaling", "logs".
  - cloudformation
  - logs
  - autoscaling
vpc:
  id: "${VPC_ID}" # Specify the VPC_ID to the eksctl command
  subnets: # Creating the EKS master nodes to a completely private environment
    private:
      private-ap-east-1a: 
        id: "${PRIVATE_SUBNETS_ID_A}"
      private-ap-east-1b:
        id: "${PRIVATE_SUBNETS_ID_B}"
      private-ap-east-1c:
        id: "${PRIVATE_SUBNETS_ID_C}"
managedNodeGroups: # Create a managed node group in private subnets
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
      autoScaler: true # enables IAM policy for cluster-autoscaler
      albIngress: true 
      cloudWatch: true 
  # securityGroups:
  #    attachIDs: ["sg-1", "sg-2"]
  ssh:
      allow: true
      publicKeyPath: ~/.ssh/id_rsa.pub
      # new feature for restricting SSH access to certain AWS security group IDs
  subnets:
    - private-ap-east-1a
    - private-ap-east-1b
    - private-ap-east-1c
cloudWatch:
  clusterLogging:
    # enable specific types of cluster control plane logs
    enableTypes: ["all"]
    # all supported types: "api", "audit", "authenticator", "controllerManager", "scheduler"
    # supported special values: "*" and "all"
addons: # explore more on doc about EKS addons: https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html
- name: vpc-cni # no version is specified so it deploys the default version
  attachPolicyARNs:
    - arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
- name: coredns
  version: latest # auto discovers the latest available
- name: kube-proxy
  version: latest
iam:
  withOIDC: true # Enable OIDC identity provider for plugins, explore more on doc: https://docs.aws.amazon.com/eks/latest/userguide/authenticate-oidc-identity-provider.html
  serviceAccounts: # create k8s service accounts(https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/) and associate with IAM policy, see more on: https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html
  - metadata:
      name: aws-load-balancer-controller # create the needed service account for aws-load-balancer-controller while provisioning ALB/ELB by k8s ingress api
      namespace: kube-system
    wellKnownPolicies:
      awsLoadBalancerController: true
  - metadata:
      name: cluster-autoscaler # create the CA needed service account and its IAM policy
      namespace: kube-system
      labels: {aws-usage: "cluster-ops"}
    wellKnownPolicies:
      autoScaler: true
EOF
```

3. Tạo khóa ssh mặc định trong môi trường Cloud9

```
ssh-keygen
```
Sau khi chạy lệnh vui lòng nhấn **ENTER** để giữ tất cả các giá trị đầu vào mặc định

![CREATECLUSTER](/images/3.createekscluster/003-createcluster.png)

4. Tạo EKS Cluster bằng cách sử dụng tệp cấu hình yaml chúng ta vừa tạo
```
eksctl create cluster -f /home/ec2-user/environment/eks-cluster.yaml
```

- Kết quả chạy sẽ như bên dưới

![CREATECLUSTER](/images/3.createekscluster/004-createcluster.png)

Trong thời gian chờ đợi, bạn có thể điều hướng đến Bảng điều khiển AWS và kiểm tra trang [dịch vụ CloudFormation](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home), bạn sẽ có thể kiểm tra trạng thái của ngăn xếp CloudFormation đang diễn ra.

![CREATECLUSTER](/images/3.createekscluster/005-createcluster.png)

{{%notice note%}}
Mất khoảng 45 phút để hoàn thành quá trình khởi tạo EKS Cluster
{{%/notice%}}

- Sau khi tạo thành công, bạn sẽ có thể thấy các ngăn xếp khác được tạo thành công sau đó.

![CREATECLUSTER](/images/3.createekscluster/006-createcluster.png)

5. Truy cập [bảng điều kiển EKS](https://ap-southeast-1.console.aws.amazon.com/eks/home) để kiểm tra các cụm vừa tạo
![CREATECLUSTER](/images/3.createekscluster/007-createcluster.png)