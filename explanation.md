# 1. CREATING YOLO MICRO-SERVICE USING DOCKER COMPOSE
bThe aim of this project is to containerize the yolo application and run it through docker compose. The image built will then be uploaded to docker hub.
## Prerequisites
- Npm
```bash
$ npm install npm@latest -g

```
- Docker Compose (Installed using _Curl_.)
```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

```
Test the installation.
```bash
$ docker-compose --version
```
- Docker hub Account.
## Installation
- To install MongoDB locally, use the link [MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/).  
Ensure the MongoDB server is running:

```bash
$ sudo service mongod start 
```

- To install [Node](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-18-04) locally follow the link's guidelines.

## Layering the Docker File
- Create a client and backend Docker File.
 ```bash
# Client DockerFile
$ touch DockerFile
```
- Import the node base image.
```bash
FROM node:14-alpine
```
- Install Dependencies using `npm ci` to install the dependencies in production mode.
- Start the node server.
```bash
CMD ["npm", "start"]
```
- Create the `docker-compose.yml` file which contains volumes to allow the container to persist and implemented a network for communication between containers.  
```yml
#docker-compose.yml
version: '3.9'
services:
  mongodb:
    container_name: db_container
    image: mongo:latest
    restart: always
    ports:
      - 2717:27017
    volumes: 
      - mongodb:/data/db

  backend:
    build: ./backend
    image: yolo-backend:v0.01
    ports: 
      - 5000:5000
    environment:
      PORT: 5000
      MONGODB_URI: mongodb://db_container:27017
      DB_NAME: yolomy
    depends_on: 
      - mongodb 
  
  client:
    build: ./client
    image: yolo-client:v0.01
    tty: true
    ports: 
      - 3000:3000
    environment:
      PORT: 3000
    depends_on:
      - backend

volumes:
  mongodb: {}
```
- To dockerize the application both backend and client images run:
```bash
$ sudo docker compose up
```
- Test whether the website is running by clicking on `https://localhost:3000`
### Pushing Docker Images to Docker Hub
- Login to your Docker hub account
```bash
$ sudo docker login
```
- Build and tag the client image
```bash
# Client Image
$ sudo docker build . -t <Dockerhub_Username/image_name:tag_name> 
```
- Push the image to docker hub account
```bash
# Client Image
$ sudo docker push <Dockerhub_Username/image_name:tag_name>
```
  &nbsp;
# 2. CONFIGURATION MANAGEMENT OF THE YOLO PROJECT USING ANSIBLE-PLAYBOOKS
In this project, we will set up an automated Ansible configuration playbook that automates variable implementation, creation of roles, allocates blocks and tags on a Vagrant provisioned server.
## Prerequisites
- Pip
```bash
$ python3 -m pip -V
```
- Ansible
- Ansible play-books
- Vagrant
## Installation
- Ansible 
```bash
$ python3 -m pip install --user ansible
```
Test the installation
```bash
$ ansible --version
```
- Vagrant 
```bash
$ wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```
```bash
$ echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" |
```

```bash
$ sudo tee /etc/apt/sources.list.d/hashicorp.list
```
```bash
$ sudo apt update && sudo apt install vagrant
```
## Configuration Steps
1. Create a **Vagrant File** in the project root directory.
```bash
$ touch Vagrantfile
```
2. Run the `Vagrant up` command to create and configure the virtual machine with the specifications in the **Vagrantfile** created.  
Create the **vagrant playbook** which will contain the project's main play.
```bash
$ touch playbook.yml
```
3. Populate the `playbook.yml` file with information that will allow SSH to configure the server as per the required specifications such as playbook name and tasks to undertake such as _Installing git_ to allow running of the _'git clone'_ command.  
4. Create  a `vars.yml` file specified in the playbook.yml file 
```yml
#vars.yml
#Git
repository: "https://github.com/VelmaGatwiri/yolo"
#branch
yoloVersion: "master"
#repo destination
dest: "/home/velma/yolo" 
```
5. Create **roles** folder that will let us to automatically load related vars, files, tasks etc and reuse the created roles dynamically throughout the project. The roles contained in that folder are:
`docker`, `docker-compose` and `git`
```bash
$ ansible-galaxy init <roleName>
```
6. Edit the roles created in the `main.yml` file with their respective tasks.
```yml
#docker role

---
# tasks file
- name: install prerequisites
  apt:
   name:
     - docker.io
   update_cache: yes
   
  # Install Docker SDK
- name: install python package manager
  apt:
   name: python3-pip
- name: install python sdk
  pip:
    name:
      - docker
      - docker-compose
```
7. To output a valid configuration for an SSH config file to SSH into the running Vagrant machine run:
```bash
$ vagrant ssh-config
```
8. Create the ansible required files:   
    a).`ansible.cfg` - Informs Ansible about our test server.  
    b).`hosts` -  Defines the server on which to access the project
9. Provision the vagrant server by running:
```bash
$ vagrant provision
```
&nbsp;
# 3. DEPLOYING THE DOCKERIZED CONTAINER TO KUBERNETES USING GOOGLE CLOUD
## Prerequisites
- Google Cloud
- Kubernetes
- Docker
## Installation
- Install [Google CLoud CLI](https://cloud.google.com/sdk/docs/install#deb) 
- Install Kubectl
```bash
$ sudo snap install kubectl --classic
```
- Install [gke-gcloud-auth-plugin](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke). This allows connection to the cluster.
---
## Configuration Steps
### Local Project Directory
1. Create the `configmap` file which contains all the external configurations of the application. It is connected to the pod. It contains the service name which acts as the Mongo DB endpoint
```bash
$ touch mongo-config.yml
```
```yml
# mongo-config.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-config
data:
  mongo-url: mongo-service
```
2. Create the `secret` file that stores and encodes credentials such as password Base64 format. 
```bash
$ touch mongo-secret.yml
```
```yml
# mongo-secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
type: Opaque 
data:
  mongo-user: dmVsbWE=
  mongo-password: dmVsbWExMjM=
```
To encode the `mongo-user` and `mongo-password` to base64 run:
```bash
$ echo -n mongouser | base 64
```
3. Create `mongo.yml` file that contains the deployment and service configurations that points the pods on which files to use.
```bash
$ touch mongo.yml
```
4. Create `backend.yml` and `client.yml` files that contain the configurations to be used in creating the pod.
```bash
$ touch client.yml
```
```yml
#client.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: yolo-client-deployment
  labels:
    app: yolo-client
spec:
  replicas: 1
  selector:
    matchLabels:
      app: yolo-client
  template:
    metadata:
      labels:
        app: yolo-client
    spec:
      containers:
      - name: yolo-client
        image: velmagatwiri/yolo-client:v0.03

        ports:
        - containerPort: 3000
        env:
        - name: yolo-backend
          value: http://yolo-backend:5000/
---
apiVersion: v1
kind: Service
metadata:
  name: yolo-client-service
spec:
  type: LoadBalancer
  selector:
    app: yolo-client
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
```
5. Commit the changes and push them to the project's github repository.  
---
### Google Kubernetes Engine
1. Log into to the google account.
2. In the google shell, change directory to project's directory i.e **yolo** pull the project to the cloud shell.
3. Create a Kubernetes Cluster.
```bash
$ gcloud container clusters create-auto <clusterName>\
  --region <regionName>
```
4. Create a manifest directory and build the k8s files to deploy the resource to the cluster
```bash
$ kubectl apply -f mongo-config.yml
$ kubectl apply -f mongo-secret.yml
$ kubectl apply -f mongo.yml
$ kubectl apply -f backend.yml
$ kubectl apply -f client.yml
```
### _**Note**_
- Replace the client external IP address in the client package.json file.
```bash
"proxy": "http://34.121.155.147:5000",
```
- Replace the backend external IP address in the components/ProductControl.s file.
