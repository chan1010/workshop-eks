---
title : "Giới thiệu"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---
### Giới thiệu
Đề tài "Triển khai ứng dụng Web trên AWS Elastic Kubernetes Service (EKS)" giúp hiểu rõ hơn về việc triển khai ứng dụng web trên AWS Elastic Kubernetes Service (EKS)..

Qua đó workshop cũng cung cấp các kiến thức cơ bản về Kubernetes và EKS, giúp người dùng hiểu rõ hơn về các khái niệm quan trọng như Pods, Services, Deployments, Nodes và Clusters. Với đề tài này, người dùng có thể học tập và áp dụng các kỹ năng về triển khai ứng dụng web trên EKS, giúp họ nắm vững và phát triển kỹ năng làm việc trên các nền tảng điện toán đám mây hiện đại.

### Amazon Elastic Kubernetes Service (EKS)
Amazon Elastic Kubernetes Service (EKS) là một dịch vụ quản lý Kubernetes được cung cấp bởi Amazon Web Services (AWS). Cho phép người dùng dễ dàng triển khai, quản lý và mở rộng các ứng dụng container trên các môi trường điện toán đám mây. EKS giúp giảm thiểu những bước phức tạp trong việc quản lý Kubernetes, như việc triển khai, cấu hình và nâng cấp các cluster, giúp cho người dùng tập trung vào việc phát triển ứng dụng hơn là quản lý hạ tầng. Với EKS, người dùng có thể sử dụng các công cụ và dịch vụ của AWS như Amazon EC2, Elastic Load Balancing, Amazon RDS, AWS Fargate và Amazon ECR để triển khai, quản lý và mở rộng các ứng dụng của mình trên Kubernetes.

### Pods
Pods trong EKS được sử dụng để đóng gói và triển khai các ứng dụng container, bao gồm một hoặc nhiều container chạy trên cùng một node và có thể chia sẻ tài nguyên và mạng với nhau.

### Services
Services được sử dụng để định tuyến traffic đến các Pod trong một cluster, giúp cho các ứng dụng có thể được truy cập và sử dụng từ bên ngoài cluster.
Một Service trong EKS là một tài nguyên trên Kubernetes, nó định tuyến các yêu cầu tới các Pod trong một hoặc nhiều Deployment, đồng thời đảm bảo sự động nhất và sẵn sàng cao của ứng dụng. Khi một Service được triển khai trên EKS, nó sẽ có một địa chỉ IP và một tên DNS duy nhất để truy cập vào các Pod của nó, người dùng có thể sử dụng địa chỉ IP hoặc tên DNS để truy cập vào ứng dụng của mình.

### Deployments
Deployment định nghĩa các Pod và ReplicaSet (bao gồm các Pod tạo thành một bản sao của ứng dụng) và giúp cho việc quản lý các phiên bản và cập nhật của ứng dụng trở nên dễ dàng hơn. Nó cũng đảm bảo rằng các ứng dụng được triển khai có sẵn và đang chạy trên Kubernetes, giúp cho việc quản lý ứng dụng trở nên đơn giản và hiệu quả hơn.

### Nodes
Là các máy tính vật lý hoặc máy ảo được chạy Kubernetes, Nodes là nơi mà các Pod được triển khai và chạy. Mỗi Node có thể chứa nhiều Pod, và các Pod có thể chạy trên các Node khác nhau. Các Node được quản lý bởi Kubernetes và có thể được tự động mở rộng hoặc thu hẹp dựa trên nhu cầu của ứng dụng. Các Node cũng có thể được quản lý và giám sát để đảm bảo rằng chúng luôn ổn định và có sẵn để chạy các ứng dụng.

### Clusters
Là một nhóm các Nodes được quản lý bởi Kubernetes, Cluster là một môi trường chạy các ứng dụng container và quản lý chúng trên nhiều máy tính. Cluster cung cấp một cơ chế để quản lý và triển khai các ứng dụng trên một môi trường đồng nhất, cho phép các ứng dụng có thể chạy và được quản lý trên nhiều Nodes một cách dễ dàng. Cluster cũng cung cấp các tính năng bảo mật, giám sát và quản lý tài nguyên cho các ứng dụng chạy trên nó.

### AWS Cloud 9
AWS Cloud9 là một dịch vụ phát triển và triển khai mã nguồn tích hợp trên đám mây của Amazon Web Services (AWS). Nó cung cấp một môi trường phát triển đám mây hoàn chỉnh, cho phép người dùng phát triển, chỉnh sửa và thực thi mã nguồn từ bất kỳ đâu thông qua trình duyệt web.

Cloud9 cung cấp một Integrated Development Environment (IDE) đầy đủ tính năng, bao gồm trình soạn thảo mã, gỡ lỗi, xem trước, và tích hợp với các công cụ phát triển khác. Nó hỗ trợ nhiều ngôn ngữ lập trình phổ biến và cung cấp các tiện ích giúp tăng năng suất phát triển ứng dụng.

Một trong những ưu điểm của Cloud9 là khả năng làm việc đa người dùng và chia sẻ môi trường phát triển. Người dùng có thể mời đồng nghiệp hoặc đồng đội của mình vào môi trường phát triển của mình để cùng làm việc, thảo luận và chia sẻ mã nguồn. Điều này giúp tăng tính hợp tác và giảm đáng kể thời gian và công sức trong việc trao đổi thông tin và cài đặt môi trường phát triển.

Cloud9 tích hợp một cách mạnh mẽ với các dịch vụ AWS khác, cho phép truy cập và quản lý tài nguyên đám mây như máy chủ EC2, hệ thống lưu trữ S3 và hàm Lambda. Nó cũng tích hợp với các công cụ phát triển bên thứ ba và dịch vụ quản lý phiên bản như Git để hỗ trợ quản lý mã nguồn và phiên bản.

Với AWS Cloud9, người dùng có thể tận dụng sức mạnh và linh hoạt của đám mây để phát triển ứng dụng một cách hiệu quả, tiết kiệm thời gian và tăng năng suất phát triển.

### Terraform
Terraform là một công cụ mã hóa cơ sở hạ tầng (Infrastructure as Code - IaC). Nó cung cấp một ngôn ngữ đơn giản và khái niệm để tạo, cấu hình và quản lý hạ tầng đám mây và hệ thống thông qua mã nguồn.