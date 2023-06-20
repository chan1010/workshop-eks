---
title: "Triển khai ứng dụng"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: " <b> 3.3 </b> "
---

Trong phần này, chúng ta sẽ xây dựng một ứng dụng mẫu (NodeJS), triển khai ứng dụng đó vào môi trường EKS và đưa ứng dụng đó ra internet thông qua Cân bằng tải ứng dụng.

#### Cài đặt NodeJS

```
sudo yum -y update
```
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```
```
. ~/.bashrc
```
```
nvm install 16
```
![NODE](/images/3.createekscluster/001-installnode.png)

#### Tạo một ứng dụng mẫu

```
mkdir app && cd app
```
```
npx express-generator
```
![NODE](/images/3.createekscluster/001-createapp.png)

```

npm install && npm start
```
![NODE](/images/3.createekscluster/002-createapp.png)

Xem giao diện ứng dụng chạy
- Chọn **Preview**
- Chọn **Preview Running Application**

![NODE](/images/3.createekscluster/003-createapp.png)
Giao diện ứng dụng
![NODE](/images/3.createekscluster/004-createapp.png)

#### Đóng gói ứng dụng docker
- Tạo Dockerfile 

```
cat > /home/ec2-user/environment/app/Dockerfile <<EOF
FROM node:14

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 3000
CMD [ "node", "./bin/www" ]
EOF
```
![NODE](/images/3.createekscluster/005-createapp.png)
- Build the Image
```
docker build -t front-end .
```
![NODE](/images/3.createekscluster/006-createapp.png)
```
 docker images
```
![NODE](/images/3.createekscluster/007-createapp.png)
Sử dụng docker run để khởi động ứng dụng và sử dụng -d để chạy nó ở chế độ nền.
```
docker run -d -p 8080:3000 front-end 
```
Sau đó, bạn sẽ có thể sử dụng lệnh docker ps để liệt kê tiến trình đang chạy và sử dụng menu "Xem trước ứng dụng đang chạy" để truy cập ứng dụng trên Cloud9.
```
docker ps
```
![NODE](/images/3.createekscluster/008-createapp.png)

