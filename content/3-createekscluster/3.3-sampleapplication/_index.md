---
title: "Deploy Application"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: " <b> 3.3 </b> "
---

In this section, we will build a sample application (Node.js), deploy it into the EKS environment, and expose the application to the internet through an Application Load Balancer.

#### Install Node.js

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

#### Create a Sample Application

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

View the running application:
- Choose **Preview**
- Choose **Preview Running Application**

![NODE](/images/3.createekscluster/003-createapp.png)

Application Interface:
![NODE](/images/3.createekscluster/004-createapp.png)

#### Package the Application with Docker
- Create a Dockerfile 

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

Use `docker run` to start the application and use `-d` to run it in the background.
```
docker run -d -p 8080:3000 front-end 
```
Then, you can use the `docker ps` command to list running processes and use the "Preview Running Application" menu to access the application on Cloud9.
```
docker ps
```
![NODE](/images/3.createekscluster/008-createapp.png)

