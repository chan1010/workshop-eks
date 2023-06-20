---
title: "Đẩy hình ảnh Docker vào Amazon Elastic Container Registry(ECR)"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: " <b> 3.4 </b> "
---

#### Đẩy hình ảnh Docker vào Amazon Elastic Container Registry(ECR) dưới dạng kho lưu trữ hình ảnh riêng tư

1. Truy cập [giao diện quản trị dịch vụ ECR](https://ap-southeast-1.console.aws.amazon.com/ecr/home)
- Chọn **Get started**
![NODEECR](/images/3.createekscluster/001-createecr.png)

2. Trong trang tạo repository ECR

- Tại mục **Visibility settings** chọn **Private**
- Tại **Repository name** chọn **front-end**
![ECR](/images/3.createekscluster/002-createecr.png)
- Chọn **Create repository**
![ECR](/images/3.createekscluster/003-createecr.png)

3. Sau khi hoàn thành tạo repository ECR truy cập [giao diện quản trị dịch vụ Endpoints](https://ap-southeast-1.console.aws.amazon.com/vpc/home?region=ap-southeast-1#Endpoints:)
- chọn **Endpoints** có **service name** đuối **.ecr.api**
- Chọn tab **Security Groups**
- Chọn security groups
![ECR](/images/3.createekscluster/004-createecr.png)
4. Tại trang Security Groups
- Chọn tab **inbound rules**
- Chọn Edit **inbound rules**
![ECR](/images/3.createekscluster/005-createecr.png)
5. Tại trang edit inbound rules
- Chọn **Add rule**
- Chọn **Type** là **All traffic**
- Chọn **source** là **aws-cloud9-eks-bastion**
- Chọn **Save rules**
![ECR](/images/3.createekscluster/006-createecr.png)

Sau khi thêm inbound rules thành công, chúng ta sẽ có thể đăng nhập vào ECR bằng cách sử dụng aws cli.

```
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin $(aws sts get-caller-identity --query Account --output text
).dkr.ecr.ap-southeast-1.amazonaws.com

```
![ECR](/images/3.createekscluster/007-createecr.png)

6. Tag the Docker image
```
docker tag front-end:latest <REPLACE YOUR ACCOUNT ID HERE>.dkr.ecr.ap-southeast-1.amazonaws.com/front-end:latest
```
![ECR](/images/3.createekscluster/008-createecr.png)
7. Push to the ECR repository.
```
docker push <REPLACE YOUR ACCOUNT ID HERE>.dkr.ecr.ap-southeast-1.amazonaws.com/front-end:latest
```
![ECR](/images/3.createekscluster/009-createecr.png)
Sau khi đẩy thành công, chúng ta sẽ có thể xem hình ảnh qua bảng điều khiển [ECR](https://ap-southeast-1.console.aws.amazon.com/ecr/repositories?region=ap-southeast-1).

![ECR](/images/3.createekscluster/010-createecr.png)

#### Prepare the deployment manifest
Trước tiên xuất id tài khoản sang biến môi trường.
```
export ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

```
![ECR](/images/3.createekscluster/012-createecr.png)

Tạo file depoy.yaml
```
cat > /home/ec2-user/environment/app/deploy.yaml <<EOF
apiVersion: v1
kind: Namespace # create the namespace for this application
metadata:
  name: front-end
---
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: front-end-deployment
  namespace: front-end
spec:
  selector:
    matchLabels:
      app: front-end
  template:
    metadata:
      labels:
        app: front-end
    spec:
      containers:
      - name: front-end
        image: ${ACCOUNT_ID}.dkr.ecr.ap-southeast-1.amazonaws.com/front-end:latest # specify your ECR repository
        ports:
        - containerPort: 3000 
        resources:
            limits:
              cpu: 500m
            requests:
              cpu: 250m
---
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: front-end-hpa
  namespace: front-end
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: front-end-deployment
  minReplicas: 1 
  maxReplicas: 100
  targetCPUUtilizationPercentage: 10 # define the replicas range and the scaling policy for the deployment
---
apiVersion: v1
kind: Service
metadata:
  name: front-end-service
  namespace: front-end
  labels:
    app: front-end
spec:
  selector:
    app: front-end
  ports:
    - protocol: TCP
      port: 3000 
      targetPort: 3000
  type: NodePort # expose the service as NodePort type so that ALB can use it later.
EOF
```
![ECR](/images/3.createekscluster/013-createecr.png)

```
kubectl apply -f deploy.yaml 
```
![ECR](/images/3.createekscluster/014-createecr.png)
#### Đưa ứng dụng lên internet
Để đạt được điều này, chúng ta cần cài đặt bộ điều khiển aws-load-balancer, bộ điều khiển sẽ giúp cụm k8s quản lý vòng đời của bộ cân bằng tải thông qua API của Ingress. Để khám phá thêm, hãy đọc [tài liệu] chính thức(https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/). Bởi vì chúng tôi đã sử dụng eksctl` để cung cấp các tài nguyên tài khoản dịch vụ và chính sách IAM cần thiết, chúng tôi có thể chỉ cần cài đặt bộ điều khiển bằng cách sử dụng Helm.
Add the EKS chart repo to helm

```
helm repo add eks https://aws.github.io/eks-charts
```
Install the TargetGroupBinding CRDs if upgrading the chart via helm upgrade
```
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"
```

Install the helm chart if using IAM roles for service accounts. NOTE you need to specify both of the chart values serviceAccount.create=false and serviceAccount.name=aws-load-balancer-controller
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=web-host-on-eks --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```

![ECR](/images/3.createekscluster/015-createecr.png)

Xuất bản các mạng con công khai

```
# export public subnets
export PUBLIC_SUBNETS_ID_A=$(aws ec2 describe-subnets --filters "Name=tag:Name,Values=k8s-ap-southeast-1a-public-subnet" | jq -r .Subnets[].SubnetId)
export PUBLIC_SUBNETS_ID_B=$(aws ec2 describe-subnets --filters "Name=tag:Name,Values=k8s-ap-southeast-1b-public-subnet" | jq -r .Subnets[].SubnetId)
export PUBLIC_SUBNETS_ID_C=$(aws ec2 describe-subnets --filters "Name=tag:Name,Values=k8s-ap-southeast-1c-public-subnet" | jq -r .Subnets[].SubnetId)

```
![ECR](/images/3.createekscluster/016-createecr.png)

```
cat > /home/ec2-user/environment/app/ingress.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: front-end
  name: ingress-front-end
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip # using IP routing policy of ALB
    alb.ingress.kubernetes.io/subnets: $PUBLIC_SUBNETS_ID_A, $PUBLIC_SUBNETS_ID_B, $PUBLIC_SUBNETS_ID_C # specifying the public subnets id
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: front-end-service # refer to the service defined in deploy.yaml
                port:
                  number: 3000
EOF
```
![ECR](/images/3.createekscluster/017-createecr.png)

```
kubectl apply -f ingress.yaml 
```
![ECR](/images/3.createekscluster/018-createecr.png)

Kiểm tra xem Ingress(ALB) có đang chạy không.
```
kubectl get ing -n front-end

```
![ECR](/images/3.createekscluster/019-createecr.png)

```
echo "http://$(kubectl get ing -n front-end --output=json | jq -r .items[].status.loadBalancer.ingress[].hostname)"
```
![ECR](/images/3.createekscluster/020-createecr.png)
![ECR](/images/3.createekscluster/021-createecr.png)