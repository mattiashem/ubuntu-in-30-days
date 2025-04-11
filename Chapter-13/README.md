# How to Guide: Task Automations, CI/CD Pipeline & Service Deployment

## Chapter Overview
In this guide, we will automate tasks and services using scripts and tools such as Bash, Ansible, Docker, Terraform, and Kubernetes. This will allow us to run commands in a repeatable, consistent manner, making service deployments and server management easier and more reliable. We will also explore a CI/CD pipeline for building, testing, and deploying applications.

### Table of Contents
1. [Basic Bash](#basic-bash)
2. [Automate Task with Ansible](#automate-task-with-ansible)
3. [Run Host Command from Docker](#run-host-command-from-docker)
4. [Pipeline Step One: Build and Push Docker Images](#pipeline-step-one-build-and-push-docker-images)
5. [Pipeline Step Two: Deploy with Terraform Against Kubernetes](#pipeline-step-two-deploy-with-terraform-against-kubernetes)
6. [CI/CD Pipeline Setup](#cicd-pipeline-setup)
7. [Conclusion](#conclusion)

---

## Basic Bash

### Introduction
Bash scripts allow us to execute commands sequentially, providing a way to automate repetitive tasks. Let's start by writing a basic Bash script to install Docker.

### Step 1: Create a Bash Script to Install Docker
Create a script named `install_docker.sh`:

```bash
#!/bin/bash

echo "Let's get docker"
apt-get update

echo "Installing required packages"
apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release

echo "Getting repo keys"
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "Setting up repo"
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

echo "Installing Docker"
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io
```

### Step 2: Make the Script Executable and Run It
```bash
chmod +x install_docker.sh
./install_docker.sh
```

This will install Docker on your system using the commands defined in the script.

### Step 3: Create a System Status Report Script
For troubleshooting, you can create a script to gather system stats.

```bash
#!/bin/bash
echo "Disk"
df -h
du -h --max-depth=0 /
du -h --max-depth=0 /var

echo "Inode"
for i in `find . -type d`; do echo `ls -a $i | wc -l` $i; done | sort -n

echo "Network"
ip a
ip r
cat /etc/resolv.conf

echo "Memory"
free
```

---

## Automate Task with Ansible

### Introduction
Ansible allows you to automate tasks across multiple servers. It is an agentless tool that uses SSH to connect to remote servers.

### Step 1: Set Up Ansible in Docker
Create a `Dockerfile` to install Ansible in a Docker container:

```dockerfile
FROM ubuntu:latest
RUN apt update
RUN apt install -y software-properties-common
RUN add-apt-repository --yes --update ppa:ansible/ansible
RUN apt install -y ansible
```

### Step 2: Create Docker-Compose for Ansible
Create a `docker-compose.yml` to mount necessary directories:

```yaml
services:
  ansible:
    build: .
    volumes:
      - ./playbooks:/opt/playbooks
      - ./files:/opt/files
      - ./hosts:/etc/ansible/
      - ./ssh:/root/.ssh
    command: tail -f /etc/fstab
```

### Step 3: Define Hosts for Ansible
In the `hosts` folder, create a file named `hosts`:

```yaml
all:
  vars:
    ansible_connection: ssh
    ansible_user: matte
  hosts:
    192.168.122.133:

docker:
  hosts:
    192.168.122.133:
apache:
  hosts:
    192.168.122.133:
```

### Step 4: Create Ansible Playbook to Install Docker
Create a playbook `install-docker.yaml`:

```yaml
- name: Install Docker
  hosts: all
  tasks:
    - name: Create directory
      file:
        path: /opt/files/
        state: directory
      become: yes
    - name: Copy install_docker.sh script
      ansible.builtin.copy:
        src: /opt/files/install_docker.sh
        dest: /opt/files/install_docker.sh
        mode: '0644'
      become: yes
    - name: Upgrade all apt packages
      apt:
        force_apt_get: yes
        upgrade: dist
      become: yes
```

### Step 5: Build Docker Image and Run Ansible Playbook
Build the image and run the Ansible playbook:

```bash
docker compose build
docker compose run ansible /bin/bash
```

Inside the container, run:

```bash
ansible-playbook --ask-pass --ask-become-pass /opt/playbooks/install-docker.yaml
```

---

## Run Host Command from Docker

### Introduction
Sometimes, you may need to run commands on the host from within a Docker container. This is useful for updating configurations or installing temporary tools.

### Step 1: Create Docker File to Update Host Files
Create a `Dockerfile` that allows running commands on the host:

```dockerfile
FROM ubuntu:latest
```

### Step 2: Create Docker-Compose File for Host Update
Create a `docker-compose.yml` to mount `/etc` into the container:

```yaml
services:
  host-update:
    build: .
    volumes:
      - /etc:/mnt/etc
```

### Step 3: Run the Container to Access Host Files
Run the container and access host files:

```bash
docker compose run host-update cat /mnt/etc/passwd
```

This will allow you to interact with host files without installing additional tools on the host.

---

## Pipeline Step One: Build and Push Docker Images

### Introduction
Now, let's create a CI/CD pipeline to build and push Docker images to a Docker registry.

### Step 1: Set Up Docker Hub
Before starting, create an account on Docker Hub for hosting the images.

### Step 2: Create Docker Compose for Nginx Web Server
Create a `docker-compose.yml` for an Nginx web server:

```yaml
services:
  nginx:
    image: nginx:latest
    volumes:
      - ./html:/usr/share/nginx/html
    ports:
      - "80:80"
```

### Step 3: Create HTML File
In the `html` folder, create an `index.html`:

```html
<html>
  <head>
    <title>Home</title>
  </head>
  <body>
    <h1>Home</h1>
    <p>Home page</p>
  </body>
</html>
```

### Step 4: Build and Push Docker Image
Create a script `build_and_push.sh`:

```bash
#!/bin/bash
VERSION=$1
docker build -t mattiashem/ubuntu-static:$VERSION .
docker push mattiashem/ubuntu-static:$VERSION
```

Run the script to build and push the image:

```bash
./build_and_push.sh v1.0
```

---

## Pipeline Step Two: Deploy with Terraform Against Kubernetes

### Step 1: Set Up Terraform with Kubernetes
Create a `Dockerfile` to install both Ansible and Terraform:

```dockerfile
FROM ubuntu:latest
RUN apt update && apt install -y wget unzip software-properties-common gnupg
RUN add-apt-repository --yes --update ppa:ansible/ansible
RUN apt install -y ansible
WORKDIR /opt
RUN wget https://releases.hashicorp.com/terraform/1.6.5/terraform_1.6.5_linux_amd64.zip && \
    unzip terraform_1.6.5_linux_amd64.zip && \
    mv terraform /usr/local/bin/terraform && \
    rm terraform_1.6.5_linux_amd64.zip
RUN terraform --version
```

### Step 2: Create Terraform Configuration
Create `terraform.tf`:

```hcl
terraform {
  backend "local" {
    workspace_dir = "/opt/terraform/state/terraform.tfstate.d"
  }
}
```

Create `deployments.tf` for Kubernetes deployment:

```hcl
resource "kubernetes_deployment" "static" {
  metadata {
    name = "static-data"
    labels = {
      test = "static"
    }
  }
  spec {
    replicas = 3
    selector {
      match_labels = {
        app = "static"
      }
    }
    template {
      metadata {
        labels = {
          app = "static"
        }
      }
      spec {
        container {
          image = "mattiashem/ubuntu-static:$VERSION"
          name  = "static"
          resources {
            limits = {
              cpu    = "0.5"
              memory = "512Mi"
            }
            requests = {
              cpu    = "250m"
              memory = "50Mi"
            }
          }
        }
      }
    }
  }
}
```

### Step 3:

 Initialize Terraform
Run Terraform commands to deploy:

```bash
terraform init
terraform plan
terraform apply
```

---


Here's your content converted into Markdown format:

```markdown
## `terraform apply`

This is the command that will make the change.

```
Plan: 1 to add, 0 to change, 1 to destroy.
```

Do you want to perform these actions?  
Terraform will perform the actions described above.  
Only 'yes' will be accepted to approve.

```
Enter a value: yes
```

```
kubernetes_deployment.static: Destroying... [id=default/static-data] 
kubernetes_deployment.static: Destruction complete after 0s 
kubernetes_deployment.static: Creating... 
kubernetes_deployment.static: Creation complete after 8s [id=default/static-data]
```

```
Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
root@924332efbdd9:/opt/terraform#  
```

And if we login into the cluster, we can see the pods running as follows:

```bash
[core@ubuntu]$ kubectl get pods 
NAME                           READY   STATUS    RESTARTS   AGE 
static-data-555757f6d4-4zpvr   1/1     Running   0          12s 
static-data-555757f6d4-cw7kj   1/1     Running   0          12s 
static-data-555757f6d4-gcz4z   1/1     Running   0          12s
```
```

## CI/CD Pipeline Setup

### Step 1: Create GitHub Actions or Jenkins Pipeline
Use GitHub Actions or Jenkins to create a pipeline that automates the entire process from building Docker images to deploying with Kubernetes.

---

