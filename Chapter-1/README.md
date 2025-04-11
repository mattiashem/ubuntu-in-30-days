# How to Guide: Getting Familiar with Ubuntu Ecosystem

## Introduction

In this chapter, we will be introduced to Linux and Ubuntu. We will start from the history of Linux, its evolution, and then dive into Ubuntu, including its releases, versions, and differences from other Linux distributions.

### Objectives:
- Learn how Linux started.
- Understand the Linux stack and how it works.
- Recognize the differences between Linux distributions and how Ubuntu is versioned and released.

---

## Linux History

Linux was created by Linus Torvalds, a computer science student at the University of Helsinki, in 1991 at the age of 21. Initially called *freax*, the project was renamed to Linux by an administrator at FUNET, where it was first hosted.

- **Tux**: The penguin mascot was chosen after Linus was bitten by a small penguin during a visit to the National Zoo and Aquarium in Canberra.

You can follow the Linux kernel's development on GitHub: [Linux on GitHub](https://github.com/torvalds/linux).

---

## Linux Stack

The Linux stack is composed of the following components:
- **Kernel**: The Linux kernel is the core component, handling hardware and system resources.
- **Operating System**: Various OS, like Ubuntu and RedHat, are built on top of the Linux kernel.
- **Window Manager**: For a desktop experience, a window manager is required, such as **i3** or **GNOME** in Ubuntu.

---

## Usage and Stats of Linux

Linux is widely used across industries:
- **Supercomputers**: Linux powers most supercomputers, including NASA servers.
- **IoT Devices**: The dominant OS for IoT devices.
- **Cloud**: Linux powers 90% of cloud systems, including Kubernetes clusters.
- **Web Servers**: 96.3% of the top 1 million web servers use Linux.
- **Popular Linux Distros**: Ubuntu leads with 32.8% usage, followed by Debian (14.4%) and CentOS (10.8%).

---

## Ubuntu History

Ubuntu is based on Debian, developed by Canonical Ltd. The first release, Ubuntu 4.10, was launched in October 2004. Ubuntu is designed to be user-friendly and is available in multiple versions:
- **Ubuntu Desktop**: For home and office computers with a GUI.
- **Ubuntu Server**: For cloud and physical servers.
- **Ubuntu Core**: For IoT devices.

---

## Ubuntu Releases

Ubuntu releases occur twice a year in April (xx.04) and October (xx.10). Each release is named by its year and month. **LTS (Long-Term Support)** releases are supported for 10 years, making them ideal for long-term setups.

- To get more information about Ubuntu's release cycle: [Ubuntu Release Cycle](https://ubuntu.com/about/release-cycle)

---

## Ubuntu Versions

Various versions of Ubuntu exist for specific use cases:
1. **Linux Mint**: A user-friendly Ubuntu-based OS using the Xfce window manager. Ideal for new users.  
   [Mint Linux Website](https://linuxmint.com/)
2. **Pop!_OS**: Developed by System76, designed for easy navigation and workflow optimization.  
   [Pop!_OS Website](https://pop.system76.com/)
3. **LXLE**: A lightweight Ubuntu-based OS designed for low-resource systems.  
   [LXLE Website](https://lxle.net/)

---

## Other Linux Distros

In addition to Ubuntu, several other Linux distributions serve specific use cases:
- **Debian**: The base for Ubuntu.
- **RedHat Enterprise Linux**: A stable Linux distro used by many enterprises.
- **Fedora**: A cutting-edge Linux distro, next in line after RedHat.
- **Alpine Linux**: A minimal Linux designed for small Docker images.
  [Alpine Linux Website](https://www.alpinelinux.org/)
- **Flatcar & CoreOS**: Specialized for running containers and Kubernetes.
- **AWS Bottlerocket**: AWS's own Linux distribution designed for container workloads.
- **Talos Linux**: A Kubernetes-only Linux distribution.

To learn more about these distros:
- [Fedora](https://getfedora.org/en/coreos?stream=stable)
- [Flatcar Linux](https://flatcar-linux.org/)
- [Talos Linux](https://www.talos.dev/)
- [Bottlerocket](https://aws.amazon.com/bottlerocket/)

---

## Managing Packages in Linux

The installation of packages differs based on the Linux distribution. Here's how you install Apache on different distros:

1. **Ubuntu/Debian** (using `apt`):
   ```bash
   sudo apt-get install apache2
   ```

2. **RedHat/CentOS** (using `yum`):
   ```bash
   sudo yum install httpd
   ```

3. **Alpine** (using `apk`):
   ```bash
   sudo apk add apache2
   ```

While the package names may be the same, the package manager commands vary.

---

## Create GitHub and Blogger Accounts

As part of this book, you will be using Git for version control and saving your work:
1. **GitHub**: A platform for storing your code repositories and configurations.  
   [GitHub](https://github.com)

2. **Blogger**: A free blog platform to record and share your progress and findings.  
   [Blogger](https://www.blogger.com)

You can also clone the book's GitHub repo to access the code examples:
- [Book GitHub Repo](https://github.com/bpbpublications/Ubuntu-Linux-in-30-days)

---

## Conclusion

Linux, started as a free operating system by Linus Torvalds, has become a widely used and versatile platform. Ubuntu is one of the most famous distributions, and this book will guide you through using it for various applications and projects.

In the next chapter, we will learn how to **install Ubuntu** and get started using it on your desktop.

---

## References:
- [Fedora CoreOS](https://getfedora.org/en/coreos?stream=stable)
- [Flatcar Linux](https://flatcar-linux.org/)
- [Talos Linux](https://www.talos.dev/)
- [AWS Bottlerocket](https://aws.amazon.com/bottlerocket/)