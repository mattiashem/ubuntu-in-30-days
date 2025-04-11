# How to Set Up Firewall, VPN, and WiFi Networks on Ubuntu

This guide provides step-by-step instructions for setting up network services, securing connections with VPNs, and configuring firewalls on your Ubuntu Desktop.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Network Settings (DHCP/Static)](#network-settings-dhcpsstatic)
3. [Connect to VLAN Networks](#connect-to-vlan-networks)
4. [Connect to Wireless Networks](#connect-to-wireless-networks)
5. [Hide Your Computer by Changing MAC Address](#hide-your-computer-by-changing-mac-address)
6. [Secure Your Connections with VPN Service](#secure-your-connections-with-vpn-service)
7. [Protect Your Traffic Using DNS](#protect-your-traffic-using-dns)
8. [Protect Your Computer Using a Firewall](#protect-your-computer-using-a-firewall)
9. [Detect and Stop Computer Viruses](#detect-and-stop-computer-viruses)
10. [Conclusion](#conclusion)

---

## 1. Introduction

Ubuntu offers powerful tools for network management, including Virtual Private Network (VPN) setup, firewall configuration, and Wi-Fi connectivity. This guide will help you configure these services to ensure a secure and efficient network setup on your Ubuntu Desktop.

---

## 2. Network Settings (DHCP/Static)

When connecting your computer to a network, you need an IP address. There are two primary methods for getting an address:

- **DHCP (Dynamic Host Configuration Protocol)**: This allows your computer to automatically get an IP address from the network router or Wi-Fi access point.
- **Static IP**: You can manually configure your IP address.

### To Set a Static IP:

1. Open **Network Settings** on Ubuntu by searching for "Advanced Network Settings."
2. Find your network card and select it.
3. Change the **Method** from `Automatic (DHCP)` to `Manual (Static)`.
4. Set the following parameters:
   - **Address**: Choose an unused IP within your network range (e.g., 192.168.1.49).
   - **Netmask**: Typically `255.255.255.0`.
   - **Gateway (Router)**: If required, enter the IP address of your router (e.g., 192.168.1.1).

---

## 3. Connect to VLAN Networks

Virtual Local Area Networks (VLANs) divide your network into multiple logical segments. If your switch supports VLANs, you can connect your Ubuntu machine to a specific VLAN.

### To Connect to a VLAN:

1. Ensure your switch is VLAN-enabled and configure the VLAN ID.
2. Add a new virtual interface for your network card:
   ```bash
   ip link add link enp1s0 name enp1s0.10 type vlan id 10
   ```
3. Configure this virtual interface with either DHCP or static IP as usual.

---

## 4. Connect to Wireless Networks

Ubuntu makes it easy to connect to Wi-Fi networks.

### To Connect to a Wireless Network:

1. Open **Network Settings** and select **Wi-Fi**.
2. Choose the network you want to connect to.
3. Enter the **password** when prompted.
4. Optionally, configure the network's security settings under the **Security** tab.

---

## 5. Hide Your Computer by Changing MAC Address

Your network device has a unique hardware address (MAC address) that identifies it. Changing this address helps increase privacy and security.

### To Randomize Your MAC Address:

1. Install **macchanger**:
   ```bash
   sudo apt-get install macchanger
   ```
2. Create a script in `/etc/network/if-pre-up.d/` to randomize the MAC address:
   ```bash
   sudo vi /etc/network/if-pre-up.d/macchanger
   ```
3. Add the following content to the script:
   ```bash
   #!/bin/sh
   /usr/bin/macchanger -e "$IFACE"
   ```
4. Make the script executable:
   ```bash
   sudo chmod +x /etc/network/if-pre-up.d/macchanger
   ```
   
---

## 6. Secure Your Connections with VPN Service

VPN (Virtual Private Network) allows you to securely connect to a network over the internet.

### To Set Up OpenVPN or WireGuard:

1. Install OpenVPN and WireGuard:
   ```bash
   sudo apt-get install openvpn wireguard
   ```

#### OpenVPN Setup:

1. Sign up for a VPN provider, like **NordVPN**.
2. Open **Network Settings** and add a new **VPN connection**.
3. Choose **OpenVPN** and configure it using the credentials provided by your VPN service.
4. Test the connection by browsing to [WhatsMyIP](https://www.whatsmyip.org/), which should show a different IP address when the VPN is active.

#### WireGuard Setup:

1. Add a new **WireGuard** connection in **Network Settings**.
2. Enter the configuration provided by your WireGuard VPN provider.

---

## 7. Protect Your Traffic Using DNS

DNS (Domain Name System) is used to resolve domain names like `google.com` into IP addresses. DNS requests are usually unencrypted, but you can secure them using **DNS over HTTPS (DoH)**.

### To Set Up DNS over HTTPS:

1. Install **dnss**:
   ```bash
   sudo apt-get install dnss
   ```
2. Start the DNSS server:
   ```bash
   sudo dnss --enable_dns_to_https --dns_listen_addr=:5553
   ```
3. Verify it works using the `dig` command:
   ```bash
   dig google.com @127.0.0.1 -p5553
   ```

4. Configure your system to use DNS over HTTPS by modifying `/etc/systemd/resolved.conf`:
   ```bash
   DNSStubListener=no
   ```
5. Reboot your system and restart the DNSS server:
   ```bash
   sudo systemctl restart systemd-resolved
   sudo dnss --enable_dns_to_https
   ```

---

## 8. Protect Your Computer Using a Firewall

A firewall controls incoming and outgoing network traffic. You can use **Iptables** to set up your firewall.

### To Lock Down Your Computer:

1. Create a script to lock down the firewall:
   ```bash
   #!/bin/bash
   iptables -F
   iptables -X
   iptables -P INPUT DROP
   iptables -P OUTPUT DROP
   iptables -P FORWARD DROP
   iptables -A INPUT -i lo -j ACCEPT
   iptables -A OUTPUT -o lo -j ACCEPT
   iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
   iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
   iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT
   iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
   ```

2. Run the script with:
   ```bash
   sudo ./lock-down.sh
   ```

---

## 9. Detect and Stop Computer Viruses

Ubuntu is less prone to viruses, but it's still important to use antivirus software, especially when sharing files with other operating systems.

### To Install ClamAV:

1. Install **ClamAV**:
   ```bash
   sudo apt-get install clamav clamtk
   ```

2. Run a quick scan on the **Downloads** folder:
   ```bash
   clamscan ~/Downloads
   ```

3. Update the virus database:
   ```bash
   sudo freshclam
   ```

4. Set up scheduled scans through **ClamAV GUI**.

---

## 10. Conclusion

In this guide, you have learned how to:

- Configure network settings (DHCP/static).
- Connect to VLANs and wireless networks.
- Secure your connections using VPN.
- Protect your traffic using DNS over HTTPS.
- Set up a firewall using Iptables.
- Detect and stop viruses using ClamAV.

You are now ready to use your Ubuntu desktop securely and efficiently in various network environments.

