Here's the extracted content formatted into markdown:

```markdown
# Chapter 9: Setup Advanced Network, Firewall, and VPN Servers

## Introduction
Ubuntu server can be used as a regular firewall. Additionally, in some cases, it performs better than any shell firewall. During this chapter, we will use Ubuntu as the primary firewall to control what traffic can come in and out of the network. We will set up our own DHCP server and control what IP clients will get and where they will go to get out on the internet. With a DNS server, we will set up and point domain names to the IP we want. Ubuntu also has the tools to set up secure tunnels between servers and clients to create a private network over the internet.

## Structure
In this chapter, we will cover the following topics:

- Using your Ubuntu as main firewall
- Network Clients with DHCP and DNS
- Securing communications
- VPN troubleshooting

## Objectives
In this chapter, we will learn how to set up our Ubuntu Server as a firewall and control what traffic can come in and out of our network. We will set up basic network tools like DHCP and DNS to have a complete network that clients can use on both servers and desktops. We will also learn how we can set up secure tunnels between our server and clients to make our own private network.

---

## Using Your Ubuntu as Main Firewall

During the ten years of working at the company Fareoffice, we used a Linux firewall with iptables as our main firewall for all our traffic to our high-loaded sites. This gave us the ability to run a simple server with Linux to control our network, and when we needed more tools or services, we could easily integrate them into our Linux server. Now, let us set up our Ubuntu server to have a public IP and then route traffic to our private network.

For this, we need two network devices, one for our external traffic and one for our private network.

On our server, we have one NIC connected using the IP `192.168.122.27`, shown by running the command `ip -a` below:

```
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:65:75:89 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.27/24 metric 100 brd 192.168.122.255 scope global dynamic enp1s0
       valid_lft 3291sec preferred_lft 3291sec
    inet6 fe80::5054:ff:fe65:7589/64 scope link
       valid_lft forever preferred_lft forever
```

Here, we will make a new virtual NIC with a new IP range to use for our network.

### Ubuntu Virtual NIC and VLAN

In Ubuntu, if we only have one network card on our server, we can add virtual NICs on that network card and clone it to new.

When we do an alias network card, it is a raw clone of the network card.

#### VLAN

VLAN is a network protocol that many switches support. Here, we can create new layers inside the network. Only the network card in the same layer can communicate. One VLAN number is 10. Then, only the network card connected to VLAN number 10 can communicate.

### Setup Network for Routing

On our server, we have two physical network cards. Let's create one more so we have three networks: one public network where we get an IP from our internet provider, and two private networks.

Now, we have three networks we can use on our setup. For setting up the network, we use the tool **netplan**. Then, we edit the file (be aware the file can have different names on different installations, so look in the netplan folder):

```
/etc/netplan/generated_name_of_your_installation (My name was /etc/netplan/00-installer-config.yaml)
```

Example network config:

```yaml
# This is the network config written by 'subiquity'

network:
  ethernets:
    enp1s0:
      dhcp4: true
    enp7s0:
      dhcp4: no
      addresses: [10.33.33.2/24]
  vlans:
    vlan.10:
      id: 10
      link: enp7s0
      addresses: [10.33.34.2/24]
  version: 2
```

Apply changes:

```
root@hrb-demo:/home/mahe# netplan apply
```

---

## Controlling Traffic with Iptables

Iptables is a tool we use to control traffic coming in and out of our server. There are three basic chains: **input**, **forward**, and **output**.

To set up our Ubuntu server as a basic router and route traffic from our private network to our public network, we need to set our server to forward packets between our network cards. This is done by running the following code:

```
echo "1" > /proc/sys/net/ipv4/ip_forward
```

We can make a simple script that sets up our router. If you search on the internet, there are many firewall scripts for Ubuntu that setup the same configuration. Here's a simple example:

### Keeping You Safe

Before you start editing firewall rules, set up a backup path to get access. It can be as simple as a screen and keyboard or a virtual console.

If you do not have any way of setting up a screen, a crontab rule that clears the iptables rules can be added. The command: `iptables -F` will clear all the rules, and adding that to a crontab will save you if you lock yourself out.

Add the following to `/etc/crontab`:

```
00 1 * * * root iptables -F
```

### Firewall Script

Create a new file called `fw.sh` and add the following commands in that script:

```bash
iptables -t nat -A POSTROUTING -s 10.33.33.0/24 ! -d 10.33.33.0/24 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 10.33.34.0/24 ! -d 10.33.34.0/24 -j MASQUERADE
iptables -A FORWARD -d 10.33.33.0/24 -o enp7s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -d 10.33.34.0/24 -o vlan.11@enp7s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -s 10.33.33.0/24 -i enp7s0 -j ACCEPT
iptables -A FORWARD -s 10.33.34.0/24 -i vlan.11@enp7s0 -j ACCEPT
```

### Default Input Rules

Set the default rule for the chains:

```bash
iptables -P INPUT DROP
iptables -P OUTPUT DROP
```

Now you should be able to set your Ubuntu server or desktop as a router and send traffic from a private network to a public network.

---

## Network Clients with DHCP and DNS

Now that we have our router set up, we can configure clients to use the network. We need two services: a **DHCPD server** to assign IP addresses to clients and a **DNS server** to resolve domain names.

We will use **dnsmasq**, which can work as both a DHCPD and DNS server.

### Installing dnsmasq

Install `dnsmasq` by running:

```
apt install dnsmasq
```

### Configuring dnsmasq

Open the configuration file:

```
vi /etc/dnsmasq.conf
```

Example configuration:

```bash
dhcp-range=10.33.33.10,10.33.34.200,255.255.255.0,12h
dhcp-host=3C:98:72:F9:14:D8,server,10.33.33.12,infinite
dhcp-option=3,10.33.33.2
dhcp-option=6,10.33.33.2
```

### Disabling Ubuntu's Default DNS Server

To disable Ubuntu's own DNS server and restart `dnsmasq`, run the following:

```bash
# Disable Ubuntu's own DNS server
systemctl stop systemd-resolved
systemctl disable systemd-resolved
systemctl mask systemd-resolved

# Restart dnsmasq
systemctl restart dnsmasq
```

If there's an error, fix `resolve.conf`:

```bash
rm /etc/resolv.conf
vi /etc/resolv.conf
```

Then manually set the DNS:

```bash
nameserver 1.1.1.1
```

### DNS Settings

To add a custom DNS record, open `/etc/hosts` and add your entry:

```bash
10.100.0.40 sidero-cp
```

To test DNS resolution:

```bash
dig google.com @127.0.0.1
```

This verifies that your server is responding to DNS queries.

---

# Securing Communications

This section will look at how we can secure communications between two Ubuntu Linux systems. When we set up the VPN, we will have one server and one or more clients connecting to that server. The communications between the client and the server are secure, and we can set it up so that all client traffic passes through the server. You can also use VPN to connect to many different locations over the public internet.

We will be exploring two solutions: **OpenVPN** and **WireGuard VPN**. These two work in different ways and can be used on different platforms. Here, we will connect a server and clients. Depending on your needs, pick the VPN that best suits your use case.

## OpenVPN

OpenVPN is the oldest of the two solutions and can be found as a standard tool on both servers and desktops. In the previous chapter, we learned how to set up an OpenVPN client to connect to a server. Now, we will set up an OpenVPN server and connect the client to that server. The server needs to be accessible over the internet, but for testing, you can run this setup on a local network as well.

OpenVPN is found in the standard Ubuntu repository and can be installed by running the following command:

```bash
sudo apt install openvpn easy-rsa
```

When setting up OpenVPN server and client, we use **TLS certificates** to verify the client and server. For that, we need to create certificates that we can use. To make this process easier, we will use the tool **easy-rsa** to generate the certificates.

### Setting Up the Server

1. **Create a folder for Easy-RSA and link necessary files:**

   ```bash
   mkdir /etc/pki/easy-rsa
   cd /etc/pki/easy-rsa
   ln -s /usr/share/easy-rsa/* .
   ```

2. **Create the `vars` file for the server certificate:**

   ```bash
   vi /etc/pki/easy-rsa/vars
   ```

   Add the following content:

   ```bash
   set_var EASYRSA_ALGO "ec"
   set_var EASYRSA_DIGEST "sha512"
   ```

3. **Initialize the Public Key Infrastructure (PKI):**

   ```bash
   ./easyrsa init-pki
   ```

4. **Build the Certificate Authority (CA):**

   ```bash
   ./easyrsa build-ca
   ```

   This will generate the CA certificate and prompt you to set a passphrase. Remember this passphrase.

5. **Generate the server certificate request:**

   ```bash
   ./easyrsa gen-req server nopass
   ```

   You will be prompted for a common name for your server. For example, `vpn.server.robots.beer`.

6. **Sign the server certificate with the CA:**

   ```bash
   ./easyrsa sign-req server server
   ```

7. **Generate the shared secret key:**

   ```bash
   openvpn --genkey --secret ta.key
   ```

### Configuring OpenVPN Server

1. **Copy the default configuration file:**

   ```bash
   sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/
   ```

2. **Update the server configuration file (`server.conf`):**

   Modify the file to point to the correct certificate locations:

   ```bash
   vi /etc/openvpn/server/server.conf
   ```

   Modify the following lines:

   ```bash
   ca /etc/pki/easy-rsa/pki/ca.crt
   cert /etc/pki/easy-rsa/pki/issued/server.crt
   key /etc/pki/easy-rsa/pki/private/server.key
   dh /etc/pki/easy-rsa/dh2048.pem
   tls-auth /etc/pki/easy-rsa/ta.key 0
   ```

3. **Start the OpenVPN server:**

   ```bash
   sudo openvpn --config /etc/openvpn/server/server.conf
   ```

   You should see the message `Initialization Sequence Completed`, which means the server is ready to accept connections.

---

## Connecting the Client

1. **Generate the client certificate request:**

   On the OpenVPN server, generate a client certificate request:

   ```bash
   ./easyrsa gen-req client1 nopass
   ```

2. **Sign the client certificate with the CA:**

   ```bash
   ./easyrsa sign-req client client1
   ```

3. **Transfer the certificates to the client:**

   On the client machine, install OpenVPN and copy the necessary certificate files (e.g., `ca.crt`, `client1.crt`, `client1.key`, and `ta.key`) from the server.

4. **Configure the client configuration file (`client.conf`):**

   Copy the example client configuration:

   ```bash
   cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/client/
   ```

   Edit the configuration file as needed:

   ```bash
   vi /etc/openvpn/client/client.conf
   ```

   Update the file with the correct paths to the certificates:

   ```bash
   remote vpn.server.robots.beer 1194
   ca /etc/openvpn/tls/ca.crt
   cert /etc/openvpn/tls/client1.crt
   key /etc/openvpn/tls/client1.key
   tls-auth /etc/openvpn/tls/ta.key 1
   ```

5. **Start the OpenVPN client:**

   ```bash
   sudo openvpn --config /etc/openvpn/client/client.conf
   ```

   If successful, the VPN connection will be established. You should see output showing the connection initialization.

6. **Verify the VPN connection:**

   You can verify the new network interface and IP address by running:

   ```bash
   ip a
   ```

   You should see the `tun0` interface with an IP in the `10.8.0.0/24` range.

7. **Test the connection:**

   From the client, ping the OpenVPN server's IP:

   ```bash
   ping 10.8.0.1
   ```

   If the ping is successful, the connection is established and secure.




Here’s the content you provided converted into Markdown format:

```markdown
# Wireguard VPN

Wireguard is a more modern VPN and somewhat easier to set up. You will understand it better in this section. Let’s start with installing Wireguard on our server.

## Install Wireguard

To install Wireguard on your server, run the following command:

```bash
sudo apt install wireguard
```

## Generate Private and Public Keys

Once Wireguard is installed, we need to create a private and a public key. Wireguard has some tools to help with this process. First, create the private key. Then, generate the public key from the private key.

```bash
wg genkey >> private.key
chmod go= private.key
cat private.key | wg pubkey | tee public.key
```

Now verify the keys with:

```bash
ls -l
```

You should see:

```bash
-rw------- 1 root root 45 Sep 14 13:52 private.key
-rw-r--r-- 1 root root 45 Sep 14 13:53 public.key
```

## Server Configuration

Now that we have our server keys, we can configure the server. Below is an example Wireguard configuration for a server using the range `10.13.13.0`:

```ini
[Interface]
Address = 10.13.13.1
ListenPort = 51820
PrivateKey = sMeT7YVzcjA
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE
```

### Adding the Private Key

Add the private key from the files you created earlier. Make sure the network range `10.13.13.0` is not already in use.

## Create Client's Peer

In Wireguard, clients are called *peers*. First, create the private key and then the public key, as we did for the server.

```bash
wg genkey | tee /etc/wireguard/private.key
cat private.key
chmod go= /etc/wireguard/private.key
cat /etc/wireguard/private.key | wg pubkey | tee /etc/wireguard/public.key
```

After creating the private and public keys, create a config file for the client. Add the following configuration in the file `wg0.conf` located in `/etc/wireguard/`:

```ini
[Interface]
Address = 10.13.13.11
PrivateKey = <PrivateKey>
ListenPort = 51820
DNS = 10.13.13.1

[Peer]
PublicKey = <PublicKey>
PresharedKey = <PresharedKey>
Endpoint = <IP_OR_ADDRESS_TO_YOUR_SERVER>:51820
AllowedIPs = 10.13.13.0/24
```

### Generate Preshared Key

You can generate the preshared key for the client with the following command:

```bash
wg genpsk > preshared.key
```

## Add Peer Configuration to Server

Now, add the peer configuration to your Wireguard server's `wg0.conf` file. Include the following:

```ini
[Peer]
# hrb
PublicKey = XEA2rYM6x4JKGFLVLWJkM2Pj4/p8ASo5ekREUV2e11M=
PresharedKey = fM7XOI0imNNEIUE6N/FpJN4Hb+tGlfw7a25tWIkxELw=
AllowedIPs = 10.13.13.11/32
```

### Verify Configurations

Make sure the following parameters match for both the server and client:

- IP Address
- PublicKey
- PresharedKey

## Restart Wireguard

To apply the configuration changes, restart your Wireguard server and client. If your config file is `wg0.conf`, you can run the following commands:

### Stop the Wireguard Interface

```bash
wg-quick down wg0
```

You should see output like:

```bash
[#] ip link delete dev wg0
[#] iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE
```

### Start the Wireguard Interface

```bash
wg-quick up wg0
```

The output will show the setup process:

```bash
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.13.13.1 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] ip -4 route add 10.13.13.6/32 dev wg0
[#] ip -4 route add 10.13.13.5/32 dev wg0
[#] ip -4 route add 10.13.13.4/32 dev wg0
[#] ip -4 route add 10.13.13.3/32 dev wg0
[#] ip -4 route add 10.13.13.2/32 dev wg0
[#] ip -4 route add 10.13.13.11/32 dev wg0
[#] iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE
```

That's it! Wireguard should now be configured and running.
```

This markdown structure includes all the steps, commands, and relevant explanations from your original text.