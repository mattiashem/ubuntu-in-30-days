# How to Set Up and Use Virtualization with KVM on Ubuntu

## Introduction
Virtualization allows multiple operating systems (OS) to run simultaneously on a single physical machine. It is widely used in cloud computing, testing, and development environments. In this guide, we will learn how to set up KVM, create virtual machines (VMs), and configure them for various use cases. We will also explore networking options, snapshots, hardware passthrough, and alternative virtualization tools like Vagrant.

## Chapter Contents
- [Overview of Virtualization in Ubuntu](#overview-of-virtualization-in-ubuntu)
- [Installing KVM Virtualization](#installing-kvm-virtualization)
- [Creating a Bridge Network](#creating-a-bridge-network)
- [Installing Your First Virtual Machine (VM)](#installing-your-first-virtual-machine-vm)
- [Configuring VM Settings](#configuring-vm-settings)
- [Using Snapshots](#using-snapshots)
- [Accessing Your VM](#accessing-your-vm)
- [Passing Through Hardware Devices](#passing-through-hardware-devices)
- [Using Other Virtualization Tools](#using-other-virtualization-tools)
- [Building and Running a Vagrant Box Inside KVM](#building-and-running-a-vagrant-box-inside-kvm)
- [Converting Virtual Machine Images](#converting-virtual-machine-images)
- [Conclusion](#conclusion)

---

## Overview of Virtualization in Ubuntu

Virtualization enables the creation of virtual environments that allow one physical machine to run multiple OS instances. It's ideal for testing, development, and resource management. For example, you can use it to:
- Run multiple versions of Ubuntu or other operating systems.
- Test a server setup before deploying it to production.
- Provide isolated environments for clients or different services.

Before enabling full virtualization, ensure your CPU supports it by running:

```bash
sudo kvm-ok
```

This command checks if KVM acceleration is available. If supported, you can proceed to install KVM and set up virtual machines.

---

## Installing KVM Virtualization

KVM (Kernel-based Virtual Machine) is the default virtualization engine for Linux. Here's how to install KVM on Ubuntu:

1. Install KVM and associated tools:

    ```bash
    sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
    ```

2. Add your user to the `libvirt` group to grant necessary permissions:

    ```bash
    sudo adduser ‘username’ libvirt
    ```

3. Verify hardware virtualization support:

    ```bash
    sudo apt install cpu-checker
    sudo kvm-ok
    ```

If the output is “KVM acceleration can be used,” your system is ready for virtualization. Otherwise, performance may be slower.

---

## Creating a Bridge Network

A bridge network allows your virtual machine to access the same network as the host system. Here’s how to set it up:

1. Check the current network configuration:

    ```bash
    nmcli con show
    ```

2. Create a bridge network:

    ```bash
    sudo nmcli con add ifname br0 type bridge con-name br0
    sudo nmcli con add type bridge-slave ifname ens3 master br0
    sudo nmcli con mod br0 bridge.stp no
    sudo nmcli con down ens3
    sudo nmcli con up br0
    ```

Make sure to replace `ens3` with the correct interface name on your machine (e.g., `enp45s0`).

---

## Installing Your First Virtual Machine (VM)

Now that KVM is set up and the bridge network is ready, you can create your first virtual machine.

1. Download the Ubuntu ISO from [Ubuntu's download page](https://ubuntu.com/download/desktop).
2. Launch `virt-manager` (KVM's GUI tool) and click "Add Virtual Machine."
3. Follow the prompts to set up the VM, ensuring to select the bridge network created earlier.
4. Once the VM is installed, boot it up, and log in to your new Ubuntu virtual machine.

---

## Configuring VM Settings

You can customize the settings for your VM, such as adding additional network interfaces, storage devices, or adjusting CPU and memory. Here’s how:

1. Open the VM settings from `virt-manager` by selecting the VM and clicking the settings icon (blue circle with 'i').
2. You can modify various parameters, such as:
   - Add additional disks or network devices.
   - Adjust CPU, memory, and other hardware settings.

Changes that affect hardware usually require the VM to be stopped and started.

---

## Using Snapshots

Snapshots capture the current state of your VM, allowing you to restore it later. Here's how to create and manage snapshots:

1. In `virt-manager`, go to your VM’s settings and click on the "Snapshots" tab.
2. Create a snapshot, name it something like "Stable", to save a clean state of your VM.
3. You can create custom snapshots by installing applications or making system changes, then taking a snapshot to preserve that state.

---

## Accessing Your VM

You can access your VM in several ways:

1. **SSH Access**: Install `openssh-server` inside the VM and use SSH from the host machine.
2. **Graphical Access**: Use KVM’s built-in graphical interface or configure VNC for desktop sharing.

Example: To access your VM via SSH:

```bash
ssh user@192.168.1.x  # Replace with your VM's IP address
```

---

## Passing Through Hardware Devices

You can pass hardware devices such as USB peripherals or GPUs to your VM. This is useful for running applications that require direct hardware access, such as gaming on a virtualized Windows machine.

To configure hardware passthrough:
1. Open the VM settings in `virt-manager`.
2. Under "Add Hardware," select "PCI Host Device" for GPUs or "USB Host Device" for USB devices.

---

## Using Other Virtualization Tools

While KVM is the default on Ubuntu, there are other virtualization tools available:

- **VirtualBox**: A cross-platform virtualization tool that works on multiple OSes. You can download it from [VirtualBox’s official page](https://www.virtualbox.org/).
- **VMware Player**: Another popular tool for running virtual machines. Visit [VMware’s official website](https://www.vmware.com/se/products/workstation-player.html) to download it.

If you have KVM running, note that it may conflict with other virtualization tools (e.g., VirtualBox might throw an error about "AMD-V" extensions).

---

## Building and Running a Vagrant Box Inside KVM

[Vagrant](https://www.vagrantup.com/) is a tool that helps you create and share virtual environments. You can build a "box" (VM) and upload it to Vagrant’s cloud for others to download and use. Here's how to set it up:

1. Install Vagrant:

    ```bash
    sudo apt install vagrant
    ```

2. Create a Vagrant project and add a box:

    ```bash
    mkdir vagrant
    cd vagrant
    vagrant box add hashicorp/bionic64
    vagrant init hashicorp/bionic64
    vagrant up
    ```

---

## Converting Virtual Machine Images

Different virtualization tools use different image formats. Luckily, you can convert between formats. For example, to convert a KVM image to VirtualBox format:

1. Locate the KVM image (usually in `/var/lib/libvirt/images`).
2. Convert the image to VMDK format (used by VirtualBox):

    ```bash
    qemu-img convert -p -f qcow2 -O vmdk ubuntu22.04-2.qcow2 ubuntu22.04.vmdk
    ```

To convert back to the QCOW2 format (KVM):

```bash
qemu-img convert -f vmdk -O qcow2 ubuntu22.04.vmdk ubuntu22.04.qcow2
```

---

## Conclusion

In this guide, we’ve learned how to set up KVM virtualization on Ubuntu, create and manage virtual machines, configure networking, use snapshots, and pass hardware devices to VMs. Additionally, we explored Vagrant for sharing VM images and converting VM images between different formats. With this knowledge, you can leverage virtualization for various purposes, from testing to running isolated environments for different clients or services.

In the next chapter, we’ll explore running Kubernetes and Docker on virtual machines.