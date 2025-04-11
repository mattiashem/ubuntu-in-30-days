# How to Install and Set Up Ubuntu Server

## Table of Contents
- [Introduction](#introduction)
- [Cattle vs Pets](#cattle-vs-pets)
- [Using SSH to Connect to Your Server](#using-ssh-to-connect-to-your-server)
- [Installing Ubuntu Server with USB](#installing-ubuntu-server-with-usb)
- [Using Ubuntu in Virtual Machine](#using-ubuntu-in-virtual-machine)
- [Ubuntu Server in Google Cloud and Hetzner](#ubuntu-server-in-google-cloud-and-hetzner)
- [Large Ubuntu Installations with MAAS](#large-ubuntu-installations-with-maas)

---

## Introduction
Ubuntu can be installed and run on cloud providers, regular computers, servers, and small microcomputers like Raspberry Pi. The installation approach varies based on where you're installing—whether it's a single server or several hundreds.

In this chapter, we will cover various installation methods including:
- **Cattle vs Pets**: How to treat your servers in different environments.
- **SSH Keys**: Secure method to connect to your server.
- **Ubuntu Server Installations**: From USB, Virtual Machine (VM), Google Cloud, and Hetzner.
- **Large Installations**: Using MAAS to manage multiple servers.

---

## Cattle vs Pets

When managing servers, the terms **Cattle** and **Pets** are used to describe different server management philosophies.

### Pets:
- You treat servers as individual units, tending to them regularly with updates and software installations.
- They are kept around for long periods and require frequent individual attention.

### Cattle:
- Servers are treated as disposable and are rebuilt rather than maintained over time.
- When an update or change is needed, you destroy and recreate the server, which can be automated for scalability.

In production environments, it's recommended to treat your servers like **cattle** for easier management and scaling.

---

## Using SSH to Connect to Your Server

SSH (Secure Shell) is the standard way to securely access and manage Ubuntu servers.

### Steps to Create SSH Keys:
1. **Generate the SSH Key Pair:**
   ```bash
   ssh-keygen
   ```

   This will generate a private and a public key. By default, these are stored in the `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub` files.

2. **Get the Public Key:**
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

   Copy this key to add to your server (e.g., cloud provider like GitHub or your VM).

3. **Copy Your Public Key to the Server:**
   ```bash
   ssh-copy-id mahe@192.168.122.27
   ```

   This step allows you to log in without a password, using SSH keys instead.

4. **Log in Using SSH:**
   ```bash
   ssh mahe@192.168.122.27
   ```

---

## Installing Ubuntu Server with USB

### Steps:
1. **Download the Ubuntu Server Image** from the official [Ubuntu website](https://ubuntu.com/download/server).
2. **Burn the Image to a USB Stick** using tools like **Balena Etcher**.
3. **Boot from USB**: Start your computer and select the USB device to boot from.
4. Follow the installation guide to set up Ubuntu Server.

   - Choose between a **Regular** or **Minimal** installation:
     - **Regular**: Full installation, includes office tools and other software.
     - **Minimal**: Smaller, suitable for server use like Docker.

5. **Disk Setup**: Choose default disk configuration or customize partitions if required.
6. **Network Setup**: Connect the server to a network during installation.
7. **Install OpenSSH**: Select the option to install OpenSSH to allow remote access once installation is complete.

   Example:
   ```bash
   Figure 7.3: Select to install OpenSSH server.
   ```

8. **Package Selection**: Choose additional packages during installation, if needed.

---

## Using Ubuntu in Virtual Machine

Ubuntu Server can also be installed in a Virtual Machine (VM) using libvirt or tools like **KVM**. This method allows you to clone servers and create new ones easily.

### Steps:
1. **Create Ubuntu Server in VM**: Follow the standard Ubuntu installation process within the VM.
2. **Clone the Server**: After installation, you can clone the server to create new instances.
   Example:
   ```bash
   Figure 7.5: Cloning Ubuntu Server.
   ```

3. **Deploy Clones**: Use the newly cloned servers for testing, staging, or scaling up your environment.
   Example:
   ```bash
   Figure 7.6: Cloned Ubuntu server.
   ```

---

## Ubuntu Server in Google Cloud and Hetzner

Cloud providers like **Google Cloud** and **Hetzner** make it easy to set up Ubuntu servers. These platforms provide templates for Ubuntu server images that you can launch in just a few clicks.

### Steps:
1. **Create an Instance in Google Cloud**: Select Ubuntu as the OS and choose the server size.
2. **Access via SSH**: Use SSH keys for secure access.
3. **Configure Network and Storage**: Set up networking and additional disks based on your needs.

---

## Large Ubuntu Installations with MAAS

For managing large-scale installations, **MAAS** (Metal as a Service) is a tool that helps manage multiple Ubuntu servers efficiently.

### Steps:
1. **Install MAAS**: Set up MAAS on a server and configure it to manage your hardware.
2. **Provision Servers**: Use MAAS to provision Ubuntu on multiple machines, automating the process of installation and configuration.

---

# How to Set Up Ubuntu Server in Google Cloud and Hetzner

This guide will walk you through the process of setting up Ubuntu servers on **Google Cloud** and **Hetzner**, as well as connecting to them using SSH. The instructions are aimed at users setting up simple servers for personal or small business use.

## Table of Contents
- [Hetzner Cloud Setup](#hetzner-cloud-setup)
- [Google Cloud Setup](#google-cloud-setup)
- [SSH Access to Cloud Servers](#ssh-access-to-cloud-servers)
- [Additional Information](#additional-information)

---

## Hetzner Cloud Setup

### 1. Log in to Hetzner Cloud Control Panel

Once logged into Hetzner's cloud control panel, you can begin creating your server.

- **Step 1**: Log in to Hetzner cloud control panel.
- **Step 2**: Select **Linux** as the server type (see the example in Figure 7.7).
- **Step 3**: Choose the configuration options for your server (e.g., CPU, memory, disk size).

---

### 2. Add SSH Key to Server

You will need to add your SSH key to the server for secure access.

- **Step 1**: In the server creation process, there will be an option to add an SSH key.
- **Step 2**: Upload or paste your SSH key into the panel (see Figure 7.8).

---

### 3. Server Deployed and Ready

Once the server is deployed, you will see information about your server, including the external IP address, which you'll need to connect to it via SSH.

- **Step 1**: Access the **server settings** (Figure 7.9).
- **Step 2**: Look for the **external IP** and **SSH details**.

---

### 4. SSH into Hetzner Server

To connect to your Hetzner server, run the following command from your local machine:

```bash
ssh root@37.27.8.127
```

This will log you into the server's **Ubuntu 22.04 LTS** instance.

---

## Google Cloud Setup

### 1. Create Ubuntu Server in Google Cloud

In Google Cloud, the process is slightly more complex due to the variety of services available.

- **Step 1**: Navigate to **Compute Engine**.
- **Step 2**: Click **Create** to set up a new instance.
- **Step 3**: Select **Ubuntu** as the operating system by changing the boot disk and choosing the Ubuntu version you need (Figure 7.10).

---

### 2. Connect to Your Google Cloud Server

After the server is created, you can find the connection settings in the Google Cloud console.

- **Step 1**: Open the settings for your server.
- **Step 2**: You will find options to connect via **SSH**.

---

### 3. Add SSH Key to Google Cloud Server

To access the server securely, you need to add your SSH key:

- **Step 1**: Navigate to the **SSH keys section** in the Google Cloud console.
- **Step 2**: Add your **public SSH key** (see Figure 7.12).

---

### 4. SSH into Google Cloud Server

From your local computer, use the following command to log in:

```bash
ssh USERNAME@EXTERNAL_IP
```

Replace `USERNAME` with the username you specified and `EXTERNAL_IP` with the IP of your server.

---

## SSH Access to Cloud Servers

Once the server is deployed on either **Google Cloud** or **Hetzner**, you can access it using SSH in the same way as you would with a local Ubuntu server.

1. **Obtain the external IP address** from the cloud provider's console.
2. **Use SSH** to log in by running:
   ```bash
   ssh USERNAME@EXTERNAL_IP
   ```

Ensure that your SSH key is added during the server creation process so you don’t have to authenticate using a password.

---

## Additional Information

### Best Practices for Production Servers

While the above steps show how to set up a basic Ubuntu server, **production environments** require more careful configuration. Be sure to follow your cloud provider's guides to:
- **Set up firewalls** and security groups.
- **Configure backups and monitoring**.
- **Implement best practices** for server hardening.

### Large Installations with MAAS

For managing a large number of Ubuntu servers, consider using **MAAS (Metal as a Service)**, a tool designed for easy provisioning and management of physical servers. MAAS is ideal for setting up servers in data centers or on a larger scale.

---

# How to Provision a VM with MAAS

This guide will walk you through the steps to provision a Virtual Machine (VM) using **MAAS** (Metal as a Service), install Ubuntu on the VM, and set up networking for a KVM-based deployment. We will also cover steps for creating and managing machines, configuring the MAAS network, and PXE-booting the server.

## Table of Contents
- [Install MAAS on Ubuntu Server](#install-maas-on-ubuntu-server)
- [Initialize MAAS](#initialize-maas)
- [Create an Admin User](#create-an-admin-user)
- [Set Up Network for MAAS](#set-up-network-for-maas)
- [Provision a VM and PXE Boot](#provision-a-vm-and-pxe-boot)
- [Control VM Power](#control-vm-power)
- [Conclusion](#conclusion)

---

## Install MAAS on Ubuntu Server

### 1. Install MAAS

Start by installing MAAS on a running **Ubuntu server**. You can install it using **snap**:

```bash
sudo snap install --channel=3.3 maas
```

This will install MAAS on the server. 

---

## Initialize MAAS

### 2. Initialize MAAS with Help Option

Run the following command to get more information about initializing MAAS:

```bash
maas init --help
```

This command will display available options for running MAAS in different modes such as `region`, `rack`, or `region+rack`.

### 3. Install Demo Database for MAAS

To set up a demo database for testing:

```bash
sudo snap install maas-test-db
```

Then, initialize MAAS with the demo database:

```bash
sudo maas init region+rack --database-uri maas-test-db:///
```

---

## Create an Admin User

### 4. Create Admin User

Once MAAS is initialized, create an admin user with the following command:

```bash
sudo maas createadmin
```

You will be prompted for:
- **Username**
- **Password**
- **Email**
- Optionally, SSH key import (use `n` for no)

---

## Set Up Network for MAAS

### 5. Configure Network for KVM

To configure the network for your KVM setup:
1. Go to **Virtual Manager** and create a new network called `provision`.
2. The network should have an IP range like `10.33.33.0` and the DHCP server disabled.

### 6. Add Network Device to VM

After creating the network, add it to your VM:
1. Add a new device to the VM.
2. Save the configuration.
3. Power cycle the VM for the new settings to take effect.

---

### 7. Set Static IP for Network Device

After the VM reboots, set a static IP for the new network interface using `netplan`:

```bash
ip a
```

From the output, identify the network interface name (e.g., `enp7s0`) and edit the network configuration:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Update the configuration:

```yaml
network:
  ethernets:
    enp1s0:
      dhcp4: true
    enp7s0:
      dhcp4: no
      addresses: [10.33.33.2/24]
  version: 2
```

Apply the changes:

```bash
sudo netplan apply
```

### 8. Verify Network Configuration

Verify the new IP configuration:

```bash
ip a
```

---

## Provision a VM and PXE Boot

### 9. Create a New VM in KVM

Create a new VM in KVM and configure it to use the `provision` network.

### 10. Update Boot Settings to PXE

Make sure the VM is set to **boot from network** to enable PXE booting.

### 11. Register the VM in MAAS

1. Log in to the MAAS dashboard.
2. Register the VM by adding its **MAC address**.

---

### 12. Boot and Install Server via PXE

After registering the VM, boot it up and follow the console log.

- The installation will start.
- When the installation is complete, the VM will power off and its status will change to **Ready**.

---

## Control VM Power

### 13. Connect MAAS to KVM Host

To control the power state of the VM from MAAS:
- Connect MAAS to the KVM host.
- MAAS can now manage the power and provisioning of the VM.

---

## Conclusion

You’ve successfully learned how to:

- Install MAAS on an Ubuntu server.
- Configure networking for MAAS and a KVM-based VM.
- PXE boot a VM and use MAAS for provisioning.

This process will help you deploy and manage Ubuntu servers using MAAS, making it easier to handle large-scale installations.

For further details, check out the [MAAS documentation](https://maas.io/).

