# Wanderlust Deployment on Kubernetes

### In this project, we will learn about how to deploy wanderlust application on Kubernetes.

### Pre-requisites to implement this project:
-  Create 2 AWS EC2 instance (Ubuntu) with instance type t2.medium and root volume 29GB.
   ![ec2-server](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0010.jpg)
  
-  Setup <a href="https://github.com/priyannkasantoki1/Reactjs"><u> Kubeadm </a></u>

#
## Steps for Kubernetes deployment:

1) Become root user :
```bash
sudo su
```

#
2) Clone code from remote repository (GitHub) :
```bash
git clone -b devops https://github.com/priyannkasantoki1/Reactjs.git
```

#
3) Verify nodes are in ready state or not :
```bash
kubectl get nodes
```
![Alt text](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0007.jpg)

#
4) Create kubernetes namespace :
```bash
kubectl create namespace wanderlust
kubectl get namespace wanderlust
```
![Namespace](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/WhatsApp%20Image%202025-03-31%20at%2019.22.05_1d193a43.jpg)

#
5) Update kubernetes config context : 
```bash
kubectl config set-context --current --namespace wanderlust
```
![Update context](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0009.jpg).
#
6) Enable DNS resolution on kubernetes cluster :

- Check coredns pod in kube-system namespace and you will find <i> Both coredns pods are running on master node </i>

```bash
kubectl get pods -n kube-system -o wide | grep -i core
```
![Alt text](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/WhatsApp%20Image%202025-03-31%20at%2019.22.05_dc2bcdf8.jpg)

- Above step will run coredns pod on worker node as well for DNS resolution

```bash
kubectl edit deploy coredns -n kube-system -o yaml
```
<i> Make replica count from 2 to 4 </i>

![replica 4](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0006.jpg)

#
7) Navigate to frontend directory :
```bash
cd frontend
```

#
8) Edit .env.docker file and change the public IP Address with your worker node public IP :
```bash
vi .env.docker
```
![IP](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/WhatsApp%20Image%202025-03-31%20at%2019.24.25_f3748092.jpg)

#
9) Build frontend docker image : 
```bash
docker build -t priyankapatel06/fornted-wanderlust:latest .

```
![Dockerfile frontend](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0027.jpg)

#
10) Navigate to backend directory :
```bash
cd ../backend/
```

#
11) Open .env.docker file and edit below variables : 

    - MONGODB_URI: \<your-mongodb-servicename>
    - REDIS_URL: \<your-redis-servicename>
    - FRONTEND_URL: \<your-workernode-publicIP>

> Note: To get service names, check <u>mongodb.yaml, redis.yaml</u>

![Backend env file](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0008.jpg)

#
12) Build backend docker image : 
```bash
docker build -t priyankapatel06/backend-wanderlust:latest .

```
![Backend dockerfile](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0025.jpg)

#
13) Check docker images:
```bash
docker images
```
![docker images](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0015.jpg)

#
14) Login to DockerHub and push image to DockerHub
```bash
docker login
```
![docker login](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0017.jpg)

```bash
docker push priyankapatel06/frontend-wanderlust:latest
docker push priyankapatel06/backend-wanderlust:latest
```

#
15) Once, Image is pushed to DockerHub, navigate to kubernetes directory
```bash
cd ../kubernetes
```

#
16) Apply manifests file the below order:

    - Create persistent volume :
    ```bash
    kubectl apply -f persistentVolume.yaml
    kubectl get pv
    ```
    ![Peristent volume](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/WhatsApp%20Image%202025-03-31%20at%2019.41.38_007b8eac.jpg)

    - Create persistent volume Claim :
    ```bash
    kubectl apply -f persistentVolumeClaim.yaml
    kubectl get pvc
    ```
    ![Peristent volume Claim](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0020.jpg)

    - Create MongoDB deployment and service :
    ```bash
    kubectl apply -f mongodb.yaml
    kubectl get service mongo-service
    ```
    ![MongoDb](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0018.jpg)

    - Create Redis deployment and service :
    > Note: Wait for 3-4 mins to get mongodb, redis pods and service should be up, otherwise backend-service will not connect.
    ```bash
    kubectl apply -f redis.yaml
    kubectl get service redis-service
    ```
    ![Redis](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0014.jpg)

    - Create Backend deployment and service :
    ```bash
    kubectl apply -f backend.yaml
    kubectl get service backend-service
    ```
    ![Backend](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0024.jpg)

    - Create Frontend deployment and service :
    ```bash
    kubectl apply -f frontend.yaml
     kubectl get service frontend-service
    ```
    ![Frontend](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0023.jpg)

#
17)  Check all deployments and services :
```bash 
kubectl get all
```
![all deployments and services](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0022.jpg)

18) Check logs for all the pods :
> Note: This is mandatory to ensure all pods and services are connected or not, if not then recreate deployments
```bash
kubectl logs <pod-name>
```

20) Navigate to chrome and access your application at 31000 port :
```bash
http://<your-workernode-publicip>:31000/
```
![App](https://github.com/priyannkasantoki1/Reactjs/blob/main/project-image/IMG-20250331-WA0019.jpg)

#
