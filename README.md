## 🚀 Deploy a MERN Stack Application with Docker, Jenkins & Kubernetes
[mern‑stack‑in‑k8s](https://github.com/PrabhaAjit/mern-stack-in-k8s) is a full‑stack MERN app for managing employee info. It uses MongoDB for the backend and React for the frontend UI.
## A) 📦 Prerequisites
Make sure you have the following are installed and configured:
- Docker  
- Docker Compose  
- Kubernetes cluster (e.g., KIND)  
- kubectl  
- Jenkins  
- Git  
- GitHub and Docker Hub accounts  
- MERN stack code (cloned from GitHub)  
Refer to the installation guide below in Section E.

---
## B)🐳 DOCKERIZE THE MERN STACK
**1. Project Setup**

Create a folder mern-stack in locally and clone the MERN stack repo from Github. 
```bash
mkdir mern-stack  && cd mern-stack
git init 
git clone https://github.com/doananhtingithub40102/mern-app.git
```
Create necessary files for auto-deployment within appropriate folders: 
![image](https://github.com/user-attachments/assets/aae9c49f-92c4-46e9-ada7-0bb4ff393bd8)

**2. Set up Dockerfile**

Each service (client,server) should have its own Dockerfile and .dockerignore. This Dockerfile is referred in docker-compose.yaml

**3. Set Up Docker Compose**

Docker Compose manages the multiple containers (MongoDB, Node.js, React). Create a docker-compose.yaml file at the root of your project to define and manage these services.

**4. Testing Locally with Docker**

Build & Run MERN stack containers: Navigate to your project's root directory and execute
```bash
docker login
docker-compose up –build
```
This command builds the images and starts all services.
Open the links in browser and verify both frontend UI and backend apps are running.  
Client: `http://localhost:3000`    Server: `http://localhost:5000`

---

**5. Push images to DockerHub**

Add below fields in docker-compose.yaml to name images. You have to specify your dockerbub username, server/client image names and tags. Rebuild the docker images.
```yaml
services:
  server:
    build: ./server
    image: <DockerHub-Username>/<Server-Image-Name>:<Tag>
  client:
    build: ./client
    image: <DockerHub-Username>/<-ClientImage-Name>:<Tag>
```
Once docker containers are up and running we can push them into docker registry.
```bash
docker push <DockerHub-Username>/<-ClientImage-Name>:<Tag>
docker push <DockerHub-Username>/<Server-Image-Name>:<Tag>
```
---
## C)☸️ DEPLOY WITH KUBERNETES  
**MERN on Kubernetes: Deployment Flow**  
**1.	Containerization**  
Each MERN component—MongoDB, backend (Express/Node), and frontend (React w/ Nginx)—is packaged into its own Docker image.  

**2.	Deployments**  
This is where you define the pod template, which includes all the details about the pods that get created and managed by the Deployment. It defines the desired replica counts for each component, continuously ensuring pods are always running and recovering them if they fail.  

**3.	Services**  
Pair each deployment with a Service (ClusterIP for internal traffic, NodePort/LoadBalancer for external access). Services provide stable networking, keeping communication reliable despite pod volatility.  

**4.	Configuration & Secrets**  
Inject runtime configuration and sensitive data using ConfigMaps and Secrets, keeping credentials out of container images.  

**5.	Scaling & Rolling Updates**  
Scale via the replicas field or kubectl scale; Kubernetes orchestrates rolling updates, replacing pods one-by-one to avoid downtime.  

**6.	Persistent Storage (PV & PVC)**  
For MongoDB’s stateful data, define a PersistentVolume (PV) and PersistentVolumeClaim (PVC), then mount the PVC (e.g. at /data/db) in the database pod—ensuring data survives pod restarts. 

**7.	Ingress**  
Add an Ingress resource in front of your frontend (and optionally backend) Service. Ingress acts as an HTTP(S) entrypoint, enabling a single external endpoint for route-based traffic, SSL termination, and load balancing.  

**8.	Local Testing with KIND**  
Use KIND (Kubernetes-in-Docker) to simulate a local cluster: build images, apply manifests, and inspect pods and services via kubectl get. Ideal for development workflows.  

You manage your MERN Kubernetes deployment using .yaml manifests. Here are the primary resource types used:  
| Manifest Type             | Purpose |
|--------------------------|---------|
| Deployments              | Define pod templates, replicas, container images, volume mounts. |
| Services                 | Expose Deployments internally (ClusterIP) or externally (NodePort/LoadBalancer). |
| ConfigMaps & Secrets     | Store non-sensitive config and sensitive credentials for pods. |
| PersistentVolumes & PVCs | Provision and claim stable storage for MongoDB's data. |
| Ingress                  | Manage external HTTP(S) access using routing rules, SSL termination, and load balancing via a single entry point. |

**Apply Kubernetes Manifests:**
```bash
kubectl apply -f client.yaml
kubectl apply -f server.yaml
kubectl apply -f mongodb.yaml
kubectl apply -f ingress.yaml
```
Verify the status of pods and services. 
```bash
kubectl get pods
kubcetl get services
kubectl get deploy
```
For trouble shooting
```bash
# Check pod logs
kubectl logs <pod-name> -f
# Debug services
kubectl describe svc <service-name>
```
![image](https://github.com/user-attachments/assets/768bcbc4-1210-4c2d-992d-2df8234e8d1d)
```bash
kubectl port-forward service/client-service 8086:80
```

Verify `http://localhost:8086/` and web page will open with the list of employees as given below.

![image](https://github.com/user-attachments/assets/0a0766c8-c1b5-43e8-93eb-a13fbf27cbae)

---

## D)🤖 JENKINS AUTOMATION

The entire process of pulling source code from Git, building a Docker image, and deploying it to a Kubernetes cluster can be fully automated using Jenkins. Follow the steps below:

**1.  Create a Jenkinsfile**  
      Define a pipeline script in a file named Jenkinsfile at the root of your project. This script should include all stages:  
      •	Cloning the source code from the Git repository  
      •	Building and pushing the Docker image  
      •	Deploying the application to Kubernetes using kubectl  
      •	Removes local Docker images and cleans up workspace.  
      
**2.  Create a Jenkins Pipeline Job**  
      •	Open your Jenkins dashboard.  
      •	Create a new Pipeline job.  
      •	Configure the job to pull your project repository (GitHub/GitLab/Bitbucket).  
      •	Set the pipeline definition to "Pipeline script from SCM" and specify the location of the Jenkinsfile.  
      
**3. Run the Pipeline**  
      Trigger the pipeline job manually or configure webhooks for automated builds. Jenkins will:  
      •	Pull the latest code from the Git repository  
      •	Build the Docker image and push it to a container registry  
      •	Apply Kubernetes manifests to deploy or update your application   

**Steps for Jenkins Pipeline Setup**

**1)	Install required plugins**
      1.	Docker Pipeline
      2.	Kubernetes CLI
      3.	Credentials Binding               
      4.	Git  

**2)	Create Pipeline Job**  
      Navigate to dashboard New Item. Enter job name (eg. ‘mern-stack-deploy’) and job type as Pipeline and click OK.
      ![image](https://github.com/user-attachments/assets/d2c26bed-e967-42a6-8aa4-94f0fc3fc783)      

**3)	Set SCM**  
      Dashboard->mern-stack-deploy(select your pipeline job)->Click Configure
      General-> Pipeline Defnition->Select Pipeline script from SCM and mention your repository URL
      ![image](https://github.com/user-attachments/assets/26ce869a-0689-4121-89c6-dc979192297d)

**4)	Set Jenkinsfile**  
      A Jenkinsfile with the pipeline script is selected. It is kept in the project root folder in github.

**5)	Create Credentials**  
      Dashboard->Manage Jenkins->Credentials->globals->Add credentials   
      **1.	 Docker Hub: For pulling docker images from dockerhub.**    
             Kind- username and password  
             Username : <dockerhub username>   
             Password : <dockerhub password>   
             ID: dockerhubcredentials  
      **2.	Kubectl credentials – For applying kubectl commands in Jenkins pipeline script**    
            Kind- secret file  
            File : <attach kube config file from ~/.kube folder>  
            ID : kubeconfig-credentials-id  
      ![image](https://github.com/user-attachments/assets/5bfea327-8965-4429-b898-9ba6533e907f)

**6)	Build Project**  
      Build the project by clicking the Build Now button.      
      ![image](https://github.com/user-attachments/assets/688ad4c7-6dba-45ce-9f30-09600f5a3b6a)

**7) When jenkins build is successful, verify the pods are running successfully.**

```bash
kubectl port-forward service/client-service 8086:80
```
Verify `http://localhost:8086/` and the web page will open with the list of employee.

---

## E)🛠️ INSTALLATION GUIDE

**[Docker](https://docs.docker.com/engine/install/)**
```bash
sudo apt update   
sudo apt install docker.io	
sudo systemctl enable docker 
sudo systemctl status docker
```
```bash
#Ubuntu user need docker access permission
sudo usermod -aG docker $USER
```
```bash
#Add Jenkins user to the Docker group. Jenkins user need permission for building images in pipeline
sudo usermod -aG docker jenkins
```
---

**[Docker Compose](https://docs.docker.com/compose/install/)**
```bash
sudo apt update
sudo apt install docker-compose
docker-compose version
```
---

**[KIND](https://kind.sigs.k8s.io/)**  
Multi-Node-Cluster Installation for KIND  
Create a kind-multi-node.yaml file. 
```yaml
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
```
```bash
kind create cluster --name=multi-node-cluster --config=kind-multi-node.yaml
```

![image](https://github.com/user-attachments/assets/1fdba79b-451c-47ad-ad1e-6129c156f2ab)
```bash
kind get clusters
```
---

**[Jenkins](https://www.jenkins.io/doc/book/installing/linux/)**   
Java is required to run Jenkins. Run below command to install Java
```bash
sudo apt update
sudo apt install openjdk-17-jre-headless
```
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install Jenkins
```
```bash
#You can enable the Jenkins service to start at boot with the command:
sudo systemctl enable jenkins
```
```bash
#You can start the Jenkins service with the command:
sudo systemctl start jenkins
```
```bash
#You can check the status of the Jenkins service using the command:
sudo systemctl status Jenkins
```
![image](https://github.com/user-attachments/assets/8861a338-d93a-4e4f-82f3-c2c389bbed51)  
If status is active, the highlighted value in above image is the initial password.

![image](https://github.com/user-attachments/assets/dfba5657-5870-44ea-8752-3628f93e5326)  
Install Plugins  
![image](https://github.com/user-attachments/assets/8cf6afca-cd77-4588-b000-2fbaee22d0cd)  
Dashboard->Mange Jenkins->Plugins  
Ensure  plugins mentioned below are installed:
1.	Docker Pipeline
2.	Kubernetes CLI  
![image](https://github.com/user-attachments/assets/fd467ea4-1921-4671-a6a8-012bb9b0cb69)  

---

**[kubectl](https://kubernetes.io/docs/tasks/tools/)**  
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```
---
MongoDB Atlas is a cloud db. You can pass mongodb connection string in application and verify the database connection instead of creating a pod for database.
![image](https://github.com/user-attachments/assets/08b89d59-2602-411c-8cae-3a9e5e29c683)

----

## E) REFERENCES

**MERN stack application**  
[https://www.mongodb.com/resources/languages/mern-stack-tutorial](https://www.mongodb.com/resources/languages/mern-stack-tutorial)  
[https://dev.to/yash_patil16/building-a-robust-cicd-pipeline-using-jenkins-for-a-mern-stack-application-592c?utm_source=chatgpt.com](https://dev.to/yash_patil16/building-a-robust-cicd-pipeline-using-jenkins-for-a-mern-stack-application-592c?utm_source=chatgpt.com)  
[https://github.com/YashPatil1609/MERN-CI-CD-Jenkins/blob/main/docker-compose.yml](https://github.com/YashPatil1609/MERN-CI-CD-Jenkins/blob/main/docker-compose.yml)  

**MongoDB**  
[https://www.w3schools.com/mongodb/](https://www.w3schools.com/mongodb/)

---
