---
title: "Push Docker Image to Amazon Elastic Container Registry (ECR)"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: " <b> 3.4 </b> "
---

#### Push Docker Image to Amazon Elastic Container Registry (ECR) as a Private Image Repository

1. Access the [ECR service management interface](https://ap-southeast-1.console.aws.amazon.com/ecr/home)
- Choose **Get started**
![ECR](/images/3.createekscluster/001-createecr.png)

2. In the ECR repository creation page:

- Under **Visibility settings**, select **Private**
- In **Repository name**, choose **front-end**
![ECR](/images/3.createekscluster/002-createecr.png)
- Choose **Create repository**
![ECR](/images/3.createekscluster/003-createecr.png)

3. After successfully creating the ECR repository, access the [Endpoint service management interface](https://ap-southeast-1.console.aws.amazon.com/vpc/home?region=ap-southeast-1#Endpoints:)
- Select the **Endpoints** with a **service name** ending in **.ecr.api**
- Choose the **Security Groups** tab
- Select the security group
![ECR](/images/3.createekscluster/004-createecr.png)

4. On the Security Groups page:
- Choose the **Inbound rules** tab
- Select **Edit inbound rules**
![ECR](/images/3.createekscluster/005-createecr.png)

5. On the Edit inbound rules page:
- Choose **Add rule**
- Set **Type** to **All traffic**
- Set **Source** to **aws-cloud9-eks-bastion**
- Choose **Save rules**
![ECR](/images/3.createekscluster/006-createecr.png)

Once you have successfully added the inbound rules, you can log in to ECR using the AWS CLI.

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
After a successful push, you can view the images in the [ECR console](https://ap-southeast-1.console.aws.amazon.com/ecr/repositories?region=ap-southeast-1).

![ECR](/images/3.createekscluster/010-createecr.png)

#### Prepare the deployment manifest
First, export your account ID to an environment variable.
```
export ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

```
![ECR](/images/3.createekscluster/012-createecr.png)

Create a deploy.yaml file.
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
#### Expose the Application to the Internet
To achieve this, we need to install the aws-load-balancer controller, which will help the Kubernetes cluster manage the lifecycle of the load balancer through the Ingress API. For more details, you can read the [official documentation](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/). Since we used `eksctl` to provide the necessary service account resources and IAM policies, we can simply install the controller using Helm.
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

Export public subnets

```
# export public subnets
export PUBLIC_SUBNETS_ID_A=$(aws ec2 describe-subnets --filters "Name=tag:Name,Values=k8s-ap-southeast-1a-public-subnet" | jq -r .Subnets[].SubnetId)
export PUBLIC_SUBNETS_ID_B=$(aws ec2 describe-subnets --filters "Name=tag:Name,Values=k8s-ap-southeast-1b-public-subnet" |

 jq -r .Subnets[].SubnetId)
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

Check if the Ingress (ALB) is running.
```
kubectl get ing -n front-end

```
![ECR](/images/3.createekscluster/019-createecr.png)

```
echo "http://$(kubectl get ing -n front-end --output=json | jq -r .items[].status.loadBalancer.ingress[].hostname)"
```
![ECR](/images/3.createekscluster/020-createecr.png)
![ECR](/images/3.createekscluster/021-createecr.png)