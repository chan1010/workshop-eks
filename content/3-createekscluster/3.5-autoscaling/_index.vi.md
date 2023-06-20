---
title: "Auto Scaling"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: " <b> 3.5 </b> "
---

Trong phần này, chúng ta sẽ khám phá hành vi mở rộng quy mô trên EKS cho ứng dụng đã triển khai của chúng ta.

Mở 4 terminal khác nhau và coppy mỗi câu lệnh sau vào mỗi terminal
```
watch kubectl get pod -n front-end
```

```
watch kubectl get hpa -n front-end
```
```
watch kubectl get node

```
Và sắp xếp các cửa sổ đầu cuối vào chế độ xem lưới để xem kết quả kiểm tra tải sắp tới.

![autoscaling](/images/3.createekscluster/001-autoscaling.png)

#### Load Testing
```
ab -c 500 -n 30000 http://$(kubectl get ing -n front-end --output=json | jq -r .items[].status.loadBalancer.ingress[].hostname)/
```
Chúng ta sẽ thấy HPA giúp tự động thay đổi quy mô ứng dụng như mong đợi, tuy nhiên, vẫn còn rất nhiều nhóm ở trạng thái đang chờ xử lý, điều này là do giới hạn tài nguyên, vì các nhóm tiếp tục mở rộng theo chiều ngang, nhưng các nút EC2 bên dưới không tự động thay đổi quy mô . Để giải quyết vấn đề này, chúng ta sẽ cài đặt Cluster Autocaler và nó sẽ giúp các nút EC2 của cụm được tự động chia tỷ lệ.
![autoscaling](/images/3.createekscluster/002-autoscaling.png)
![autoscaling](/images/3.createekscluster/007-autoscaling.png)
#### Install the cluster autoscaler

```
kubectl scale deployment/front-end-deployment -n front-end --replicas 1
```
```
cat > /home/ec2-user/environment/eks-ca.yaml <<EOF
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["events", "endpoints"]
    verbs: ["create", "patch"]
  - apiGroups: [""]
    resources: ["pods/eviction"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["endpoints"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["watch", "list", "get", "update"]
  - apiGroups: [""]
    resources:
      - "pods"
      - "services"
      - "replicationcontrollers"
      - "persistentvolumeclaims"
      - "persistentvolumes"
    verbs: ["watch", "list", "get"]
  - apiGroups: ["extensions"]
    resources: ["replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["watch", "list"]
  - apiGroups: ["apps"]
    resources: ["statefulsets", "replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "csinodes"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["batch", "extensions"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["create"]
  - apiGroups: ["coordination.k8s.io"]
    resourceNames: ["cluster-autoscaler"]
    resources: ["leases"]
    verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create","list","watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
    verbs: ["delete", "get", "update", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        cluster-autoscaler.kubernetes.io/safe-to-evict: 'false'
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8085'
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
        - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.20.0 #image needs to match to your k8s version, see more on https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler
          name: cluster-autoscaler
          resources:
            limits:
              cpu: 100m
              memory: 300Mi
            requests:
              cpu: 100m
              memory: 300Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/web-host-on-eks
            - --balance-similar-node-groups
            - --skip-nodes-with-system-pods=false
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt #/etc/ssl/certs/ca-bundle.crt for Amazon Linux Worker Nodes
              readOnly: true
          imagePullPolicy: "Always"
      volumes:
        - name: ssl-certs
          hostPath:
            path: "/etc/ssl/certs/ca-bundle.crt"

EOF

```

![autoscaling](/images/3.createekscluster/003-autoscaling.png)

Áp dụng tệp yaml cho cụm.
```
kubectl apply -f eks-ca.yaml
```
![autoscaling](/images/3.createekscluster/004-autoscaling.png)
Kiểm tra xem CA đã hoạt động chưa
```
kubectl get deployment -n kube-system
```
![autoscaling](/images/3.createekscluster/005-autoscaling.png)

Check log CA logs

```
kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler
```
Sau khi cài đặt Cluster Autoscaler, chúng ta có thể thử chạy lại lệnh ab để xem liệu nhóm nút EKS có được mở rộng lại như mong đợi hay không.

```
ab -c 500 -n 30000 http://$(kubectl get ing -n front-end --output=json | jq -r .items[].status.loadBalancer.ingress[].hostname)/
```
![autoscaling](/images/3.createekscluster/008-autoscaling.png)
```
kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler
```