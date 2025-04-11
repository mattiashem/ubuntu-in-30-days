Here’s the requested content converted into a Markdown "How-to" guide:

```markdown
# Running Virtualization Server Environment

## Introduction

The ability to run multiple services such as different web servers, databases, or several databases for different purposes is a common use case when working with Ubuntu servers. To achieve this and isolate each service, we turn to virtualization. Virtualization allows us to transform a single Ubuntu server into multiple virtual servers, each with its own isolated environment. 

This guide will cover the installation and management of Virtual Machines (VMs) on your Ubuntu server. We will also explore containers, and how tools like Podman can achieve similar results for running services in isolated environments.

## Structure

In this guide, we will cover the following topics:

- Installing KVM on your Ubuntu server
- Connecting from the desktop using KVM GUI
- Installing the KVM web interface
- Creating a VM server
- Controlling your VM using the `virsh` command
- Setting up a dedicated VM with Linux
- Introduction to Containers
- Podman's features
- Setting up and monitoring with Grafana and Prometheus
- Reading logs with Loki
- Container-based monitoring clients

## Objectives

By the end of this guide, you will know how to:

1. Set up an Ubuntu server as a virtual host.
2. Create new virtual machines and set up services like web servers or databases.
3. Manage your virtual servers by cloning, backing them up, and configuring networks.
4. Set up monitoring with tools like Grafana, Prometheus, and Loki.
5. Use Podman to manage containers as an alternative to Docker.

---

## Installing KVM on Your Ubuntu Server

To start using KVM (Kernel-based Virtual Machine) on your Ubuntu server, ensure that you have SSH access to the server. We'll install KVM and then verify that we can run virtual machines (VMs).

### Step 1: Install KVM

To install KVM on Ubuntu, run the following commands:

```bash
sudo apt -y install bridge-utils cpu-checker libvirt-clients libvirt-daemon qemu qemu-kvm
```

### Step 2: Verify KVM Installation

After installation, run the following command to verify that KVM acceleration is supported:

```bash
kvm-ok
```

You should see:

```bash
INFO: /dev/kvm exists
KVM acceleration can be used
```

---

## Connecting from the Desktop Using KVM GUI

You can manage your KVM virtual machines from your desktop using a GUI tool like `Virtual Machine Manager`.

### Step 1: Add SSH Key for Access

First, add your SSH key to the server for passwordless access. Then, add your user to the `libvirt` group to ensure you can connect to `libvirtd`.

### Step 2: Set Up KVM Connection on Desktop

1. Open the **Virtual Machine Manager** on your desktop.
2. Add a new connection by providing your username and host information (IP address of the server).
3. No password is needed when using SSH keys for authentication.

Once connected, you can manage your virtual servers from your desktop.

---

## Installing the KVM Web Interface

You can manage your KVM virtual machines via a web interface using the **Cockpit** tool.

### Step 1: Install Cockpit

Install the Cockpit plugin for KVM management by running the following command:

```bash
sudo apt install cockpit-machines
```

### Step 2: Access Cockpit Web Interface

Log in to your cockpit console by opening the web browser and accessing `http://<your-server-ip>:9090`. You should see your virtual machines running.

Both **Virt Manager** and **Cockpit** can be used interchangeably for managing virtual machines, either locally or remotely.

---

## Creating a VM Server

To create a virtual machine (VM) on your Ubuntu server, you can use the terminal.

### Step 1: Run the `virt-install` Command

Use the following command to create a new VM server:

```bash
sudo virt-install --name ubuntu-guest --os-variant ubuntu20.04 --vcpus 2 --ram 2048 --location http://ftp.ubuntu.com/ubuntu/dists/focal/main/installer-amd64/ --network bridge=virbr0,model=virtio --graphics none --extra-args='console=ttyS0,115200n8 serial' --disk size=5
```

This command will start the installation of Ubuntu on a new virtual machine.

### Step 2: Follow the Installation Process

The installation process will proceed in the terminal. You can monitor the installation progress using Virt Manager or Cockpit.

---

## Controlling Your VM Using the `virsh` Command

The `virsh` command can be used to manage virtual machines from the command line.

### Step 1: Clone a VM

You can clone your VM into an image using `virsh`. This image can then be moved to another server or used as a backup.

Example command to create an image:

```bash
virsh dumpxml ubuntu-guest > ubuntu-guest.xml
```

### Step 2: Use Shared Storage

You can configure an NFS server to share a disk image between multiple Ubuntu servers. This allows you to store and boot your VM from a shared disk image on any of the connected servers.

---

## Dedicated VM Linux Version

For larger deployments, you may want to consider using specialized tools like **Proxmox** or **OpenStack**. These tools provide more features for running multiple VMs and managing shared storage.

- **Proxmox**: An open-source virtualization platform that supports multiple VMs and shared storage.
- **OpenStack**: A cloud computing platform for managing large VM deployments.

---

## Containers

While KVM is great for running virtual machines, you can also run services in containers. **Podman** is a container tool that works similarly to Docker but without requiring a daemon.

### Step 1: Install Podman

To install Podman on Ubuntu, run the following command:

```bash
sudo apt-get install podman
```

### Step 2: Verify Podman Installation

After installation, you can verify if Podman is installed correctly by running:

```bash
podman --version
```

---

## Podman’s Features

Podman is a powerful container tool with several key features:

- Supports both OCI and Docker images.
- Full network integration with CNI network plugins.
- Allows combining multiple containers into pods, similar to Kubernetes.
- Does not require a daemon, providing better security and resource management.
- Offers a REST API for integration with other tools.

---

## Podman Error with CNI Plugin

If you encounter a network error with Podman, it might be due to a conflict with the CNI plugin. To resolve this, update the CNI version in the network configuration file.

### Step 1: Edit CNI Network File

```bash
sudo nano /etc/cni/net.d/87-podman-bridge.conflist
```

Verify and update the `cniVersion` to `"0.4.0"`.

---

Here's a step-by-step "How-To" guide based on the provided information, formatted in Markdown:

---

# Setup and Monitoring with Grafana and Prometheus

In this guide, we will set up a monitoring stack using **Grafana**, **Prometheus**, and **Node Exporter** with **Loki** for logs using **Podman**. 

We will walk through the process of setting up the monitoring stack, adding services, and configuring data sources for Grafana.

## Prerequisites

- **Podman** or **Docker** installed on your server
- Basic familiarity with YAML and Docker Compose

---

### 1. Set Up Monitoring Folder and Docker Compose File

Create a folder called `/opt/monitoring` and inside that folder, create a `docker-compose.yaml` file.

```bash
mkdir -p /opt/monitoring
cd /opt/monitoring
```

Create the initial `docker-compose.yaml` with just **Grafana** service:

```yaml
version: "3"
services:
  grafana:
    environment:
      - GF_PATHS_PROVISIONING=/etc/grafana/provisioning
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    image: grafana/grafana:latest
    volumes:
      - ./grafana:/var/lib/grafana
    ports:
      - "3000:3000"
```

---

### 2. Start Grafana Service

To start the Grafana service, use the following command:

```bash
podman-compose up
```

If you encounter any errors such as "container name already in use," you can remove the existing container and start again:

```bash
podman start -a monitoring_grafana_1
```

Access the Grafana GUI at: `http://<SERVER_IP>:3000`

---

### 3. Add Prometheus to Your Stack

Create a new folder for Prometheus and add the configuration file `prometheus.yml`.

```bash
mkdir /opt/monitoring/prom
```

Create the `prom/prometheus.yml` configuration file with the following content:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    monitor: 'my-project'

rule_files:
  - 'alert.rules'

alerting:
  alertmanagers:
    - scheme: http
      static_configs:
        - targets:
            - "alertmanager:9093"

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 15s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'cadvisor'
    scrape_interval: 15s
    static_configs:
      - targets: ['cadvisor:8080']

  - job_name: 'node-exporter'
    scrape_interval: 15s
    static_configs:
      - targets: ['node-exporter:9100']
```

Next, update the `docker-compose.yaml` to add the **Prometheus** service:

```yaml
version: "3"
services:
  grafana:
    environment:
      - GF_PATHS_PROVISIONING=/etc/grafana/provisioning
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    image: grafana/grafana:latest
    volumes:
      - ./grafana:/var/lib/grafana
    ports:
      - "3000:3000"

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prom/:/etc/prometheus/
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    ports:
      - "9090:9090"
```

Start the services using:

```bash
podman-compose up
```

Once the services are up, access Prometheus at `http://<SERVER_IP>:9090`.

---

### 4. Add Node Exporter

To collect metrics from the host system, add **Node Exporter** to the `docker-compose.yaml` file.

```yaml
  node-exporter:
    image: prom/node-exporter:v1.6.1
    container_name: nodeexporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    expose:
      - "9100"
```

Update your `docker-compose.yaml` to include **Node Exporter**, and start the stack again:

```bash
podman-compose up
```

After starting the services, open Grafana and set up **Prometheus** as the data source. Then, import a dashboard with the ID `1860` to visualize your system metrics.

---

### 5. Add Loki for Log Collection

**Loki** is used for log aggregation, and **Promtail** is the agent that pushes logs to Loki.

#### 5.1 Create Folder for Loki

Create a folder for Loki and set the appropriate permissions:

```bash
mkdir /opt/monitoring/loki
chmod 777 /opt/monitoring/loki
```

#### 5.2 Update `docker-compose.yaml` to Include Loki and Promtail

Add the **Loki** and **Promtail** services to the `docker-compose.yaml` file:

```yaml
  loki:
    image: grafana/loki:2.9.1
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    volumes:
      - ./loki:/loki:Z

  promtail:
    image: grafana/promtail:2.9.1
    volumes:
      - /var/log:/var/log
    command: -config.file=/etc/promtail/config.yml
```

#### 5.3 Start the Full Stack

Run the following command to start all services:

```bash
podman-compose up
```

#### 5.4 Configure Loki in Grafana

- In Grafana, navigate to **Data Sources** and add **Loki** at the address `http://loki:3100`.
- Save and test the connection.
- Go to **Explorer** in Grafana and select **Loki** as the data source to view logs.

---

### 6. Monitor Logs and Metrics

At this point, you have successfully set up a monitoring stack with **Grafana**, **Prometheus**, **Node Exporter**, and **Loki**. You can now:

- View metrics from your server in Grafana.
- Monitor logs using Loki and Promtail in Grafana's **Explorer** view.

---

### 7. Extend Monitoring to Additional Servers

To extend monitoring to other servers on your network:

1. Ensure **Podman** or **Docker** is running on the target server.
2. Use the same **Prometheus**, **Node Exporter**, and **Loki** configuration to collect metrics and logs from additional servers.

---
# How to Set Up Journal Logs and Container-Based Monitoring Clients with Loki, Promtail, and Prometheus

## Journal Logs Collection

Ubuntu uses a tool called `journal` to handle logs, but the default configuration in Promtail does not collect logs from the journal. To get all system logs, we need to create a custom Promtail configuration file.

### Step 1: Create Promtail Configuration for Journal Logs

1. **Create a folder for Promtail**:
   ```bash
   root@pihole:/opt/monitoring# mkdir promtail
   ```

2. **Create a file called `config.yml`** in the `promtail` folder and add the following content:
   ```yaml
   server:
     http_listen_port: 9080
     grpc_listen_port: 0

   positions:
     filename: /tmp/positions.yaml

   clients:
     - url: http://loki:3100/loki/api/v1/push

   scrape_configs:
     - job_name: system
       static_configs:
         - targets:
             - localhost
           labels:
             job: varlogs
             __path__: /var/log/*log

     - job_name: journal
       journal:
         json: false
         max_age: 12h
         path: /var/log/journal
         matches: _TRANSPORT=kernel
         labels:
           job: systemd-journal
       relabel_configs:
         - source_labels: ['__journal__systemd_unit']
           target_label: 'unit'
   ```

### Step 2: Update Promtail in Docker Compose

Update the `promtail` service in your `docker-compose.yaml` file to use the newly created configuration file:

```yaml
promtail:
  image: grafana/promtail:2.9.1
  volumes:
    - ./promtail/:/etc/promtail/
    - /var/log:/var/log
  command: -config.file=/etc/promtail/config.yml
```

### Step 3: Restart Monitoring Stack

Once the `promtail` config is updated, restart your monitoring stack:

```bash
root@pihole:/opt/monitoring# podman-compose up  # or docker-compose up
```

Verify that logs from the journal are being collected by visiting the Grafana interface and checking if logs are available from the journal source.

---

## Container-Based Monitoring Clients

We will now set up monitoring clients that send logs and metrics to the base monitoring server.

### Step 1: Create Docker Compose File for Monitoring Client

On the client machine (for example, `g1`), create the following `docker-compose.yaml` to run Promtail for logs and Node Exporter for metrics.

1. **Promtail Configuration**:
   - Create a `config.yml` file for `promtail`:

   ```yaml
   server:
     http_listen_port: 9080
     grpc_listen_port: 0

   positions:
     filename: /tmp/positions.yaml

   clients:
     - url: http://192.168.1.4:3100/loki/api/v1/push  # IP of your base monitoring server

   scrape_configs:
     - job_name: system
       static_configs:
         - targets:
             - localhost
           labels:
             job: varlogs
             __path__: /var/log/*log

     - job_name: journal
       journal:
         json: false
         max_age: 12h
         path: /var/log/journal
         matches: _TRANSPORT=kernel
         labels:
           job: systemd-journal
       relabel_configs:
         - source_labels: ['__journal__systemd_unit']
           target_label: 'unit'
   ```

2. **Update the Node Exporter Configuration**:
   Add the following service in the `docker-compose.yaml` to start Node Exporter:

   ```yaml
   node-exporter:
     image: prom/node-exporter:v1.6.1
     container_name: nodeexporter
     volumes:
       - /proc:/host/proc:ro
       - /sys:/host/sys:ro
       - /:/rootfs:ro
     command:
       - '--path.procfs=/host/proc'
       - '--path.rootfs=/rootfs'
       - '--path.sysfs=/host/sys'
       - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
     expose:
       - 9100
   ```

### Step 2: Update Prometheus Configuration

On the base monitoring server, update the Prometheus configuration to scrape metrics from your client nodes.

1. **Add Node Exporter Target**:
   Edit the `prometheus.yml` file to include the IPs of your client nodes (e.g., `g1`):

   ```yaml
   scrape_configs:
     - job_name: 'node-exporter'
       scrape_interval: 15s
       static_configs:
         - targets: ['node-exporter:9100', '192.168.1.11:9100', '10.0.0.17:9100']  # Add clients' IP addresses here
   ```

### Step 3: Start the Client Monitoring Stack

Now, start the client monitoring stack using Podman or Docker Compose:

```bash
root@g1:/opt/monitoring# podman-compose up  # or docker-compose up
```

---

## Viewing Logs and Metrics in Grafana

Once everything is up and running, you should be able to view logs and metrics in Grafana:

### Adding Loki as a Data Source in Grafana

1. Go to **Configuration** > **Data Sources** in Grafana.
2. Add **Loki** as a data source by specifying the URL `http://loki:3100`.
3. Save and test the connection.

### Viewing Logs in the Explorer View

1. Go to **Explore** in Grafana.
2. Select **Loki** as the data source and filter logs by job or unit to view logs.

### Viewing Metrics with Node Exporter Dashboard

1. Go to **Dashboards** in Grafana.
2. Import the **Node Exporter** dashboard (ID 1860).
3. View the metrics from your client server(s) in Grafana.

---




