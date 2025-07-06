## Deploy a MERN (MongoDB, Express.js, React, Node.js) stack application using Docker, Jenkins and Kubernetes
[mern-stack-in-k8s] (https://github.com/PrabhaAjit/mern-stack-in-k8s) is a full-stack MERN application that manages the basic information of employees. MongoDB database is used as backend db for displaying employee info in frontend UI React.

**Prerequisites to deploy project locally**
Docker 
Docker Compose
Kubernetes cluster (KIND)
kubectl 
Jenkins 
Git 
Gihub, Dockerhub accounts
MERN stack code cloned from github
Refer installation guide below in D section.

## A) DOCKERISE THE MERN STACK
**1.	Set Up Project Structure**
Create a folder mern-stack in local laptop and clone the MERN stack project from github
```bash
cd mern-stack
git init 
git clone https://github.com/doananhtingithub40102/mern-app.git

Create the files which are required for automatic deployment of MERN application as shown below in respective folders. 
![image](https://github.com/user-attachments/assets/9153dfba-4739-4f18-8440-8d89bd092f58) 

**2. Set up Dockerfile**
Each service has its own Dockerfile. Create a Dockerfile and .dockerignore files inside server and client folders. Dockerfile is referred in docker-compose.yaml

**3. Set Up Docker Compose**
Docker Compose manages the multiple containers (MongoDB, Node.js, React). Create a docker-compose.yml file at the root of your project to define and manage your services.

**4. Testing Locally with Docker**
Build & Run MERN stack containers: Navigate to your project's root directory and execute:
```bash
docker login
docker-compose up –build

This command builds the images and starts all services.
Open the app in the browser and verify the below links and confirm both frontend UI and backend app is running.
Client: http://localhost:3000
Server: http://localhost:5000

**5. Push images to DockerHub or any registry**
Add below fields in docker-compose.yaml to name images and rebuild  docker images.
services:
  server:
    build: ./server
    image: prabhaajit/mern-server:latest
  client:
    build: ./client
    image: prabhaajit/mern-client:latest
Once docker images is verified we can push them into docker registry.
```bash
docker push prabhaajit/mern-server:latest
docker push prabhaajit/mern-client:latest

## B) DEPLOY WITH KUBERNETES
Kubernetecs use the dockerised images of our frontend and backend from dockerhub
and use them for pod creation. client.yaml file has definition for pod deployment and its services for frontend application. server.yaml file has definition for pod deployment and its services for backend application. mongodb.yaml has definition for pod creation for database. Run below commands to create all resources and their services. 
```bash
kubectl apply -f client.yaml
kubectl apply -f server.yaml
kubectl apply -f mongodb.yaml
kubectl apply -f ingress.yaml

Verify the status of pods and services.
```bash
kubectl get pods
kubcetl get services
kubectl get deploy

![image](https://github.com/user-attachments/assets/d0a254cc-506a-4fcd-8023-33576df3e266)

```bash
kubectl port-forward service/client-service 8086:80

And verify http://localhost:8086/


![my picture](https://doananhtingithub40102.github.io/MyData/mern/mypicture.png)

![image](https://github.com/user-attachments/assets/635a9031-5788-40f5-87ae-535f90264dcb)

## C) JENKINS AUTOMATION
Entire process of pulling source code from Git, docker image creation, and deploying in Kubernetes can be automated using Jenkins. Create a Jenkinsfile with the automated script
and place it in project root directory. Create a pipeline job in Jenkins and attach the Jenkinsfile. Build Jenkins job and the entire process is automated and pods will be deployed.

**Jenkins Pipeline**
A Jenkinsfile defines stages:
•	Checkout: Pulls code from GitHub.
•	Build Images: Uses docker-compose to build React and Node.js Docker images.
•	Push to DockerHub: Authenticates using Jenkins credentials and pushes images to DockerHub.
•	Deploy to Kubernetes: Uses a kubeconfig credential to apply Kubernetes manifests. Variables are injected using envsubst.
•	Cleanup: Removes local Docker images and cleans up workspace.

**1)	Ensure these Jenkins plugins are installed:**
1.	Docker Pipeline
2.	Kubernetes CLI
3.	Credentials Binding                  
4.	Git

**2)	Click on New Item. Enter job name as ‘mern-stack-deploy’ and job type as Pipeline and click OK.**
![image](https://github.com/user-attachments/assets/dbfa1b17-d91e-4404-a024-4f592d613b64)

3)	Dashboard->mern-stack-deploy(select your pipeline job)->Click Configure
General-> Pipeline Defnition->Select Pipeline script from SCM and mention your repository URL

![image](https://github.com/user-attachments/assets/0d467619-b957-40c7-9208-c731d5aea5d1)

4)	A Jenkinsfile with the pipeline script is selected. It is kept in the project root folder in github.

5)	Dashboard->Manage Jenkins->Credentials->globals->Add credentials
  	Create 2 credentials
    1.	 Docker Hub: For pulling docker images from dockerhub.
         Kind- username and password
         Username : <docker hub username>
         Password : <docker hub password>
         ID: dockerhubcredentials
    2.	Kubectl credentials – For applying kubectl commands in Jenkins pipeline script
        Kind- secret file
        File : <attach kube config file from ~/.kube folder>
        ID : kubeconfig-credentials-id
![image](https://github.com/user-attachments/assets/720795a5-c90b-451e-ae32-7ad1bb70b52e)

6)	Build the project by clicking the Build Now button.
![image](https://github.com/user-attachments/assets/8562e98a-d1c1-440d-bb0c-6e0cab9262cf)

7)	When jenkins build is successful verify if the pods are running successfully. 
    ```bash    
    kubectl port-forward service/client-service 8086:80

    And verify http://localhost:8086/

## D) INSTALLATION GUIDE
**Docker**
```bash    
sudo apt update   
sudo apt install docker.io	
sudo systemctl enable docker 
sudo systemctl status docker
Ubuntu user need docker access permission
```bash    
sudo usermod -aG docker $USER 
Add Jenkins user to the Docker group. Jenkins user need permission for building images in pipeline
```bash    
sudo usermod -aG docker jenkins

**Docker Compose**
```bash    
sudo apt update
sudo apt install docker-compose
docker-compose version

**KIND for K8S**
Multi-Node-Cluster Installation for KIND
Create a kind-multi-node.yaml file. 
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: kindest/node:v1.33.0
  - role: worker
    image: kindest/node:v1.33.0
  - role: worker
    image: kindest/node:v1.33.0
    extraPortMappings:
    - containerPort: 81
      hostPort: 81
      listenAddress: "0.0.0.0" # Optional, defaults to "0.0.0.0"
      protocol: tcp # Optional, defaults to tcpi
```bash    
kind create cluster --name=multi-node-cluster --config=kind-multi-node.yaml

![image](https://github.com/user-attachments/assets/1fdba79b-451c-47ad-ad1e-6129c156f2ab)
```bash    
kind get clusters

**Jenkins**
Jenkins Install guide:  https://www.jenkins.io/doc/book/installing/linux/
Java is required to run Jenkins. To install Java
```bash    
sudo apt update
sudo apt install openjdk-17-jre-headless

```bash    
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install Jenkins

You can enable the Jenkins service to start at boot with the command:
```bash    
sudo systemctl enable jenkins
You can start the Jenkins service with the command:
```bash    
sudo systemctl start jenkins
You can check the status of the Jenkins service using the command:
```bash    
sudo systemctl status Jenkins
![image](https://github.com/user-attachments/assets/8861a338-d93a-4e4f-82f3-c2c389bbed51)
If status is active, the highlighted value in above image is the initial password.

![image](https://github.com/user-attachments/assets/dfba5657-5870-44ea-8752-3628f93e5326)
Install Plugins
![image](https://github.com/user-attachments/assets/8cf6afca-cd77-4588-b000-2fbaee22d0cd)
Dashboard->Mange Jenkins->Plugins
Ensure these Jenkins plugins are installed:
1.	Docker Pipeline
2.	Kubernetes CLI
![image](https://github.com/user-attachments/assets/fd467ea4-1921-4671-a6a8-012bb9b0cb69)

**kubectl on Jenkins Server**
```bash    
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

MongoDB Atlas is a cloud db. You can pass mongodb connection string in application and verify the database connection instead of creating a pod for database.
![image](https://github.com/user-attachments/assets/08b89d59-2602-411c-8cae-3a9e5e29c683)

## E) REFERENCES
**MERN stack application**
[https://www.mongodb.com/resources/languages/mern-stack-tutorial] (https://www.mongodb.com/resources/languages/mern-stack-tutorial)
[https://dev.to/yash_patil16/building-a-robust-cicd-pipeline-using-jenkins-for-a-mern-stack-application-592c?utm_source=chatgpt.com] (https://dev.to/yash_patil16/building-a-robust-cicd-pipeline-using-jenkins-for-a-mern-stack-application-592c?utm_source=chatgpt.com)
[https://github.com/YashPatil1609/MERN-CI-CD-Jenkins/blob/main/docker-compose.yml] (https://github.com/YashPatil1609/MERN-CI-CD-Jenkins/blob/main/docker-compose.yml)
**MongoDB**
[https://www.w3schools.com/mongodb/] (https://www.w3schools.com/mongodb/)
