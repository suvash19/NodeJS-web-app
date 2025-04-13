# Create a simple nodeJs application and deploy it onto a docker container.

Create a working directory
```bash 
mkdir <working_directory_name>
```
Running this command in working directory will initialize your project
```bash 
npm init
```
This will create a package.json file in the folder, that file contains app dependency packages.

Replace the following code of package.json
```bash 
  // package.json

  {
    "name": "docker_web_app",
    "version": "1.0.0",
    "description": "Node.js deploy on Docker container",
    "author": "cmuth001@odu.edu",
    "main": "server.js",
    "scripts": {
      "start": "node server.js"
    },
    "dependencies": {
      "express": "^4.16.1"
    }
  }
```
Running this command will install all the dependencies from package.json
```bash 
npm install
```
Lets create a server.js file that defines a web-app using an Express framework.
```bash 
   // server.js
   'use strict';
   var express = require('express');
   var app = express();
   app.get('/', function (req, res) {
     res.send('Hello World!');
   });
   app.listen(3000, function () {
     console.log('Example app listening on port 3000!');
   });
```
Lets test the application, run the below command
node server.js

If you followed the above steps on your system, you will see the same output as below image: http://localhost:3000/

Node.js

Now node.js app is running successfully.

Lets try running the same node.js application running on the docker container. To run the application on the docker conatiner we need a docker image.

First, we will create a docker image for the application.

Create a Dockerfile
```bash 
touch Dockerfile
```
Dockerfile should look like this
FROM node:10
## Create app directory
WORKDIR /usr/app

## Install app dependencies
## A wildcard is used to ensure both package.json AND package-lock.json are copied
## where available (npm@5+)
COPY package*.json ./

RUN npm install
## If you are building your code for production
## RUN npm ci --only=production

## Bundle app source
```bash 
COPY . .
EXPOSE 3000
CMD [ "node", "server.js" ]
```
Create .dockerignore file with following content
node_modules
npm-debug.log
This will prevent from copying onto docker image.

Building Docker image

docker build -t node-web-app .

Node.js

Check the Docker images

docker images

Node.js

Run the docker image
```bash 
docker run -p 49160:3000 -d node-web-app
```
Get the container id
```bash 
docker ps
```

# Installing Docker
Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

Use these commands and you can also check screenshots for reference:
```bash 
sudo apt update
```

Next, install a few prerequisite packages which let apt use packages over HTTPS:
```bash 
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

Then add the GPG key for the official Docker repository to your system:
```bash 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add the Docker repository to APT sources:
```bash 
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

This will also update our package database with the Docker packages from the newly added repo.

Make sure you are about to install from the Docker repo instead of the default Ubuntu repo:
```bash 
apt-cache policy docker-ce
```

Notice that docker-ce is not installed, but the candidate for installation is from the Docker.

Finally, install Docker:
```bash 
sudo apt install docker-ce
```

Check the status of docker:
```bash 
sudo systemctl status docker
```
![image](https://github.com/user-attachments/assets/d3b3fa18-f6e6-4069-a543-d1f48991c2cc)

# Installing Docker-Compose
```bash 
sudo apt-get update -y
sudo apt-get install docker-compose -y
```
Check your docker-compose version after its installed:
```bash 
sudo docker-compose --version
```
# Bitbucket Pipelines CI/CD for Docker Hub Deployment 

## Step 1: Prepare Your GitHub Repository

GitHub Repo: suvash19/docker-node-sample-app

Contains:

index.js (Node.js app)

Dockerfile

## Step 2: Create a Bitbucket Repository

Go to bitbucket.org

Create a new repository (you can leave it empty)

This Bitbucket repo will host the bitbucket-pipelines.yml file

## Step 3: Add Bitbucket Environment Variables
Go to: Bitbucket Repo → Repository Settings → Repository Variables

Add the following secured variables:
Name	                   Value
DOCKER_HUB_USERNAME	     Your Docker Hub username
DOCKER_HUB_PASSWORD	     Docker Hub password or access token

Note: Access Tokens are safer than passwords.

![image](https://github.com/user-attachments/assets/d17fcc5a-ed48-464f-9046-44f13fef9db0)


## Step 4: Create bitbucket-pipelines.yml
```bash 
image: docker:20.10.16

options:
  docker: true

pipelines:
  default:
    - step:
        name: Build, Push & Deploy
        services:
          - docker
        caches:
          - docker
        script:
          - apk add --no-cache git openssh-client curl

          # Clone GitHub repo securely using environment variables
          - git clone https://suvash19:Git-Token@github.com/suvash19/NodeJS-web-app.git
          - cd NodeJS-web-app

          # Tag image with commit hash
          - export IMAGE_TAG=$BITBUCKET_COMMIT

          # Docker login
          - echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin

          # Build and push Docker image
          - docker build -t $DOCKERHUB_USERNAME/docker-node-sample-app:$IMAGE_TAG .
          - docker push $DOCKERHUB_USERNAME/docker-node-sample-app:$IMAGE_TAG

          # Decode SSH private key
          - echo "$AWS_SSH_KEY_B64" | base64 -d > test.pem
          - chmod 600 test.pem

          # SSH into EC2 and deploy Docker container
          - |
            ssh -o StrictHostKeyChecking=no -i test.pem ubuntu@18.212.54.143 << EOF 
              docker pull $DOCKERHUB_USERNAME/docker-node-sample-app:$IMAGE_TAG
              docker stop app || true
              docker rm app || true
              docker run -d --name app -p 80:8080 $DOCKERHUB_USERNAME/docker-node-sample-app:$IMAGE_TAG
            EOF

          # Clean up key
          - rm test.pem
```
## Step 6: Trigger the Pipeline

To test your pipeline:

Commit and push the bitbucket-pipelines.yml to your Bitbucket repo

## Step 7: Expected Output

![image](https://github.com/user-attachments/assets/2b4a4aaa-e3dc-45cc-81f8-c031e18f8c39)



# Setting up prometheus and grafana

After installing docker-compose we can easily setup prometheus and grafana using docker-compose.yml file. While we are doing this let’s try to monitor our host machine (server) as well. We need node_exporter for this.

Lets configure docker-compose.yml and start up our containers
```bash 
sudo mkdir prometheus
cd prometheus
sudo nano docker-compose.yml
```
```bash 
version: '3.3'
volumes:
  prometheus-data:
    driver: local
  grafana-data:
    driver: local
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./config:/etc/prometheus/
      - prometheus-data:/prometheus
    networks:
      - prometheus-network
    ports:
      - "9090:9090"
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    volumes:
      - grafana-data:/var/lib/grafana
    networks:
      - prometheus-network
    ports:
      - "3000:3000"
  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter
    command:
      - '--path.rootfs=/host'
    pid: host
    ports:
      - "9100:9100"
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'
    networks:
      - prometheus-network
networks:
  prometheus-network:
    driver: bridge
```
Also, we need a config file for Prometheus. So let’s create it.
```bash 
mkdir config
sudo nano config/prometheus.yml
```

# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  
  # scrape_timeout is set to the global default (10s).
  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'codelab-monitor'
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['prometheus:9090']
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']
    
Now lets start our containers:
```bash 
docker-compose up --build –d
```
and check the status:
```bash 
docker-compose ps
```
![image](https://github.com/user-attachments/assets/43cb65a4-7575-449b-8f95-536666f28c75)

Looks like everything is running fine. Now go to http://yourserverip:9090/ to access Prometheus:

Go to Status->targets.

![image](https://github.com/user-attachments/assets/a3e7f1b8-29d5-45e3-a2ab-4fc20f97efcc)


Here we can see that both our targets prometheus and node_exporter is up.

It means that prometheus can scrape data from these endpoints.

However, we won’t be discussing about prometheus for now. We will be using grafana to display the metrics that is scraped by prometheus.

Let’s login to our grafana server.

Goto http://yourserverip:3000/


Default username and password is admin.

After logging in click on Add your first data source:


Select Prometheus:


Set your data source name and enter url for making connection to prometheus.

In our case set this url:

http://prometheus:9090

This above url works only because grafana and prometheus are in same network in docker.



Click on Save and test.

Now let’s setup our dashboard for host monitoring.

![image](https://github.com/user-attachments/assets/a4cc7052-efc1-4f5a-a25c-edb95a2e5a94)


