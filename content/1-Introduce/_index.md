---
title : "Introduction"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---
### Introduction
The topic "Deploying Web Applications on AWS Elastic Kubernetes Service (EKS)" helps provide a better understanding of deploying web applications on AWS Elastic Kubernetes Service (EKS).

Through this workshop, it also imparts fundamental knowledge about Kubernetes and EKS, helping users gain a deeper understanding of important concepts such as Pods, Services, Deployments, Nodes, and Clusters. With this topic, users can learn and apply skills for deploying web applications on EKS, enabling them to master and develop skills for working on modern cloud computing platforms.

### Amazon Elastic Kubernetes Service (EKS)
Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service provided by Amazon Web Services (AWS). It allows users to easily deploy, manage, and scale containerized applications in cloud computing environments. EKS simplifies complex Kubernetes management tasks, including deployment, configuration, and cluster upgrades, allowing users to focus more on application development rather than infrastructure management. With EKS, users can leverage AWS tools and services such as Amazon EC2, Elastic Load Balancing, Amazon RDS, AWS Fargate, and Amazon ECR to deploy, manage, and scale their applications on Kubernetes.

### Pods
Pods in EKS are used to package and deploy containerized applications, including one or more containers running on the same node and capable of sharing resources and networking with each other.

### Services
Services are used to route traffic to Pods within a cluster, enabling applications to be accessed and used from outside the cluster. A Service in EKS is a Kubernetes resource that routes requests to Pods in one or more Deployments, ensuring the seamless and high availability of applications. When a Service is deployed on EKS, it has a unique IP address and DNS name that users can use to access their applications.

### Deployments
Deployments define Pods and ReplicaSets (comprising Pods that make up a replica of an application) and make managing application versions and updates easier. They also ensure that deployed applications are available and running on Kubernetes, simplifying application management and efficiency.

### Nodes
Nodes are physical or virtual machines running Kubernetes, where Pods are deployed and run. Each Node can host multiple Pods, and Pods can run on different Nodes. Nodes are managed by Kubernetes and can be automatically scaled up or down based on application needs. Nodes can also be monitored and managed to ensure they remain stable and available for running applications.

### Clusters
Clusters are groups of Nodes managed by Kubernetes, providing an environment for running containerized applications and managing them across multiple computers. Clusters offer a mechanism for managing and deploying applications in a consistent environment, allowing applications to run and be managed on multiple Nodes easily. Clusters also provide security, monitoring, and resource management features for applications running on them.

### AWS Cloud9
AWS Cloud9 is an integrated cloud-based development and deployment service by Amazon Web Services (AWS). It offers a complete cloud-based development environment that allows users to develop, edit, and execute source code from anywhere via a web browser.

Cloud9 provides a feature-rich Integrated Development Environment (IDE), including code editors, debugging, preview, and integration with other development tools. It supports multiple popular programming languages and provides utilities to enhance application development productivity.

One of the advantages of Cloud9 is its multi-user collaboration and environment sharing capabilities. Users can invite colleagues or team members into their development environment to work together, discuss, and share source code. This fosters collaboration, significantly reduces the time and effort required for information exchange, and facilitates development environment setup.

Cloud9 integrates strongly with other AWS services, allowing access to and management of cloud resources such as EC2 instances, S3 storage systems, and Lambda functions. It also integrates with third-party development tools and version control services like Git to support source code and version management.

With AWS Cloud9, users can harness the power and flexibility of the cloud to develop applications efficiently, save time, and increase development productivity.

### Terraform
Terraform is an Infrastructure as Code (IaC) tool used for infrastructure provisioning, configuration, and management. It provides a simple language and concepts for creating, configuring, and managing cloud infrastructure and systems through source code.