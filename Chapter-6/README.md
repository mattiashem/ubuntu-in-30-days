# How to Use Docker and Kubernetes for Local Development

This guide will walk you through setting up Docker on Ubuntu, running your first container, and using Docker Compose for managing multi-container applications. You'll also learn how to get started with Kubernetes for local development.

---

## Table of Contents
1. [Installing Docker on Ubuntu](#installing-docker-on-ubuntu)
2. [Running Your First Docker Container](#running-your-first-docker-container)
3. [Using Docker Compose](#using-docker-compose)
4. [Connecting Services with Docker Compose](#connecting-services-with-docker-compose)
5. [Expanding Docker Compose](#expanding-docker-compose)
6. [Getting Started with Kubernetes](#getting-started-with-kubernetes)

---

## 1. Installing Docker on Ubuntu

### Step 1: Install Dependencies
Before installing Docker, you need to install some dependencies:

```bash
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg
```

### Step 2: Add Docker Repository and Key
Run the following commands to add Docker's official repository and GPG key:

```bash
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Step 3: Install Docker Engine
Now, update your package cache and install Docker:

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Step 4: Add User to Docker Group
To avoid using `sudo` with Docker commands, add your user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

You may need to log out and log back in for this change to take effect.

### Step 5: Verify Installation
Check if Docker is installed correctly:

```bash
docker run hello-world
```

---

## 2. Running Your First Docker Container

Docker allows you to run applications in isolated environments called containers. For example, to run a Minecraft server, use the following command:

```bash
docker run -it -e EULA=true -p 25565:25565 --name mc itzg/minecraft-server
```

This command will start the Minecraft server in a container. To stop the server, press `CTRL+C`.

---

## 3. Using Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. Here’s how you can manage a Minecraft server using a `docker-compose.yaml` file.

### Step 1: Create `docker-compose.yaml`
Create a file named `docker-compose.yaml` with the following content:

```yaml
version: "3"
services:
  mc:
    image: itzg/minecraft-server
    ports:
      - 25565:25565
    environment:
      EULA: "TRUE"
    tty: true
    stdin_open: true
    restart: unless-stopped
    volumes:
      - ./minecraft-data:/data
```

### Step 2: Start the Minecraft Server with Docker Compose

To start the container:

```bash
docker compose up
```

If you want the server to run in the background:

```bash
docker compose up -d
```

To stop the server:

```bash
docker compose stop
```

To view running services:

```bash
docker compose ps
```

---

## 4. Connecting Services with Docker Compose

You can also connect multiple services (like WordPress and MySQL) using Docker Compose.

### Step 1: Create `docker-compose.yaml` for WordPress and MySQL

```yaml
services:
  db:
    image: mariadb:10.6.4-focal
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress
    expose:
      - 3306
      - 33060
    networks:
      - local
  wordpress:
    image: wordpress:latest
    ports:
      - 80:80
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress
    networks:
      - local
volumes:
  db_data:
networks:
  local:
    external: true
```

### Step 2: Start the Stack

Run the stack:

```bash
docker compose up
```

Then, open a web browser and visit `http://localhost` to complete the WordPress installation.

---

## 5. Expanding Docker Compose

You can add more services to your `docker-compose.yaml` file. For example, to add phpMyAdmin for database management, update the file like this:

```yaml
services:
  db:
    image: mariadb:10.6.4-focal
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress
    expose:
      - 3306
      - 33060
    networks:
      - local
  wordpress:
    image: wordpress:latest
    ports:
      - 80:80
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress
    networks:
      - local
  phpmyadmin:
    image: phpmyadmin
    ports:
      - 8080:80
    environment:
      - PMA_ARBITRARY=1
      - PMA_HOST=db
      - PMA_USER=wordpress
      - PMA_PASSWORD=wordpress
    networks:
      - local
volumes:
  db_data:
networks:
  local:
    external: true
```

### Step 3: Start the Stack with phpMyAdmin

Run the updated stack:

```bash
docker compose up
```

Then, access phpMyAdmin by visiting `http://localhost:8080`.

---

## 6. Getting Started with Kubernetes

Once you're comfortable with Docker and Docker Compose, the next step is to deploy applications using Kubernetes.

### Step 1: Install Kubernetes on Ubuntu

To install Kubernetes, follow the official [Kubernetes installation guide for Ubuntu](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/).

### Step 2: Deploy Your First Application on Kubernetes

You can deploy your Docker containers on Kubernetes using the following steps:

1. Install `kubectl` (Kubernetes command-line tool).
2. Create a Kubernetes deployment for your app (e.g., WordPress and MySQL).
3. Use `kubectl` to interact with your Kubernetes cluster.

To deploy a MySQL database on Kubernetes, you can use the following command:

```bash
kubectl create deployment mysql --image=mysql:5.7
```

Then, expose the MySQL service:

```bash
kubectl expose deployment mysql --port=3306 --target-port=3306
```

You can access the service using:

```bash
kubectl get services
```

---
# How to Connect Two Docker Stacks and Deploy Apps with Kubernetes

This guide walks you through connecting two Docker stacks using a shared network and deploying applications with Docker and Kubernetes.

---

## Connecting Two Docker Stacks

When you start a Docker Compose setup, Docker creates a separate network for the containers to ensure isolation. However, in some cases, you may want to connect two Docker stacks (e.g., when developing multiple services that need to interact with each other).

### Steps to Connect Two Docker Stacks

1. **Create a Shared Network**  
   In your Docker Compose files, define a shared network to allow communication between stacks. Here's an example of how to set up a shared network in your `docker-compose.yaml`:

   ```yaml
   networks:
     local:
       external: true
   ```

2. **Create a New Folder for Metabase**  
   In the new folder (e.g., `Metabase`), create a `docker-compose.yaml` file with the following content:

   ```yaml
   version: '3'
   services:
     metabase:
       image: metabase/metabase
       environment:
         - POSTGRES_USER=metabase
         - POSTGRES_DB=metabase
         - POSTGRES_PASSWORD=metabase
       volumes:
         - ./pg:/var/lib/postgresql/data
       networks:
         - local
   networks:
     local:
       external: true
   ```

3. **Keep Track of Container Names and Ports**  
   - **Container Names:** Ensure that the container names are unique across all stacks.
   - **Ports:** Make sure that no two services share the same port when using the same network. For example, set WordPress to use port `8080`.

4. **Start Both Stacks**  
   Run both stacks in two separate terminals:

   ```bash
   docker-compose up
   ```

5. **Access Metabase**  
   Once Metabase is up, visit `http://localhost:3000` to start the installation guide. During the installation, you will be asked to add a database. Choose MySQL and use the credentials from the WordPress installation.

---

## Local Development with Docker

This section covers how to use Docker for local development, particularly with a simple HTML page.

### Steps to Set Up Local Development with Docker

1. **Create a Folder for Your Project**  
   Inside the project folder, create a `docker-compose.yaml` file with the following content:

   ```yaml
   version: '3'
   services:
     dev:
       image: nginx
       volumes:
         - ./code:/usr/share/nginx/html
       ports:
         - 80:80
   ```

2. **Create an HTML File**  
   Inside the project folder, create a folder called `code`. Inside the `code` folder, create an `index.html` file:

   ```html
   <html>
   <head>docker test</head>
   <body><h2>Docker test</h2></body>
   </html>
   ```

3. **Start the Docker Stack**  
   Run the Docker stack with:

   ```bash
   docker-compose up
   ```

---

## About Kubernetes

Kubernetes is a container orchestration platform that automates the deployment, scaling, and management of containerized applications. Kubernetes can be run on nearly any cloud provider and offers APIs for custom extensions and integration with other services.

### Steps to Set Up Kubernetes Locally with Minikube

1. **Install Minikube**  
   First, download and install Minikube on your system:

   ```bash
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   sudo install minikube-linux-amd64 /usr/local/bin/minikube
   ```

2. **Start Minikube**  
   You can start Minikube in different modes (VM or Docker). To use Minikube with Docker, run:

   ```bash
   minikube start
   ```

   To use Minikube with a virtual machine, run:

   ```bash
   minikube start --driver=kvm2
   ```

3. **Install kubectl**  
   Install `kubectl`, the Kubernetes command-line tool:

   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   ```

4. **Verify Kubernetes Nodes**  
   After Minikube is up, check the status of your Kubernetes nodes with:

   ```bash
   kubectl get nodes
   ```

---

## Deploying Apps on Kubernetes

### Deploy MySQL

1. **Create a MySQL Deployment File**  
   Create a `mysql.yaml` file with the following content:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: mysql
     namespace: mysql
   spec:
     ports:
       - port: 3306
     selector:
       app: mysql
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mysql
     namespace: mysql
   spec:
     selector:
       matchLabels:
         app: mysql
     template:
       metadata:
         labels:
           app: mysql
       spec:
         containers:
           - image: mysql:5.7
             name: mysql
             env:
               - name: MYSQL_ROOT_PASSWORD
                 value: "myrootpassword"
               - name: MYSQL_DATABASE
                 value: wordpress
               - name: MYSQL_USER
                 value: wordpress
               - name: MYSQL_PASSWORD
                 value: password
             ports:
               - containerPort: 3306
   ```

2. **Create a Namespace and Apply MySQL Deployment**  
   Run the following commands to create the namespace and deploy MySQL:

   ```bash
   kubectl create namespace mysql
   kubectl apply -f mysql.yaml
   ```

3. **Verify the MySQL Deployment**  
   Check if MySQL is running:

   ```bash
   kubectl get all -n mysql
   ```

### Deploy WordPress

1. **Create a WordPress Deployment File**  
   Create a `wordpress.yaml` file with the following content:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: wordpress
     namespace: wordpress
   spec:
     ports:
       - port: 80
     selector:
       app: wordpress
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: wordpress
     namespace: wordpress
   spec:
     selector:
       matchLabels:
         app: wordpress
     template:
       metadata:
         labels:
           app: wordpress
       spec:
         containers:
           - image: wordpress:6.2.0-apache
             name: wordpress
             env:
               - name: WORDPRESS_DB_HOST
                 value: mysql.mysql.svc
               - name: WORDPRESS_DB_USER
                 value: wordpress
               - name: WORDPRESS_DB_NAME
                 value: wordpress
               - name: WORDPRESS_DB_PASSWORD
                 value: password
             ports:
               - containerPort: 80
   ```

2. **Create a Namespace and Apply WordPress Deployment**  
   Run the following commands to create the namespace and deploy WordPress:

   ```bash
   kubectl create namespace wordpress
   kubectl apply -f wordpress.yaml
   ```

3. **Verify the WordPress Deployment**  
   Check if WordPress is running:

   ```bash
   kubectl get all -n wordpress
   ```

### Access Your WordPress Service

You can access the WordPress service using the following command:

```bash
minikube service wordpress -n wordpress
```

This will open your browser with the WordPress installation page.

---

