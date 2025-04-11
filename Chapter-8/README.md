# Commands for Monitoring a Linux Server

## Top Command

The `top` command is used to print out the current state of your server. It displays resource usage and identifies which processes are consuming the most resources.

```bash
top
```

## Netstat Command

The `netstat` command shows the active connections on your server. You can use various arguments with it, and the following command is one of the common usages:

```bash
netstat -anp
```

## LSOF Command

The `lsof` command shows the files that are currently being used on the system.

```bash
lsof
```

## DU Command

The `du` command helps you find the size of folders. It’s useful when your server runs out of disk space.

```bash
du -h --max-depth=1
```

## Install Cockpit Command

Cockpit is a web-based tool for monitoring and administrating an Ubuntu server. Install it using the following command:

```bash
sudo apt install -t ${VERSION_CODENAME}-backports cockpit
```

## Access Cockpit WebGUI

After installation, access the Cockpit WebGUI by navigating to:

```
https://IP:9090
```

## Install Grafana Commands

Grafana is a tool used for visualizing data. To install Grafana on your Ubuntu server, run the following commands:

```bash
# Adding Grafana key
sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key

# Adding Grafana repositories
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# Updating the repositories
apt update

# Install Grafana
apt install grafana
```

## Restart Grafana Server

If Grafana is already installed, restart the service using the following command:

```bash
systemctl restart grafana-server
```

## Install Prometheus Commands

Prometheus is a tool to collect and store metrics. To install Prometheus on your server, follow these commands:

```bash
# Adding user and group for Prometheus
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus

# Creating necessary directories
sudo mkdir /var/lib/prometheus
sudo mkdir /etc/prometheus/
sudo mkdir /etc/prometheus/rules
sudo mkdir /etc/prometheus/rules.d
sudo mkdir /etc/prometheus/files_sd

# Download and install Prometheus
cd /tmp
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -qi -
tar xvf prometheus*.tar.gz
cd prometheus*/
sudo mv prometheus promtool /usr/local/bin/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo mv consoles/ console_libraries/ /etc/prometheus/
```

## Create Prometheus Systemd Service File

Create a systemd service file for Prometheus:

```bash
sudo tee /etc/systemd/system/prometheus.service<<EOF
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

## Set Permissions for Prometheus

Set the correct permissions for the Prometheus directories:

```bash
for i in rules rules.d files_sd; do sudo chown -R prometheus:prometheus /etc/prometheus/${i}; done
for i in rules rules.d files_sd; do sudo chmod -R 775 /etc/prometheus/${i}; done
sudo chown -R prometheus:prometheus /var/lib/prometheus/
```

## Enable and Start Prometheus

Enable and start the Prometheus service:

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

## Stop Cockpit Service (Optional)

If Prometheus is running on the same port as Cockpit (9090), stop the Cockpit service with the following commands:

```bash
systemctl stop cockpit.socket
systemctl disable cockpit.socket
```

## Install Node Exporter Commands

Node Exporter is a tool used to expose Linux server metrics to Prometheus. To install Node Exporter, run the following commands:

```bash
# Download and unpack Node Exporter
https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
tar zxvf node_exporter-1.6.0.linux-amd64.tar.gz

# Copy Node Exporter to /usr/local/bin
cd node_exporter-1.6.0.linux-amd64
sudo cp node_exporter /usr/local/bin

# Create a user for Node Exporter and set permissions
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

## Create Node Exporter Systemd Service File

Create a systemd service file for Node Exporter:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Add the following content:

```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

## Enable and Start Node Exporter

Enable and start the Node Exporter service:

```bash
systemctl daemon-reload
systemctl start node_exporter
systemctl enable node_exporter
```

## Test Node Exporter

Test if Node Exporter is exposing data:

```bash
curl -v http://127.0.0.1:9100/metrics
```

## Modify Prometheus Config for Node Exporter

To scrape data from the Node Exporter, modify the Prometheus config:

```bash
vi /etc/prometheus/prometheus.yml

# Add the following scraper configuration
- job_name: "g1"
  static_configs:
    - targets: ["localhost:9100"]
```

## Restart Prometheus

Restart the Prometheus service to apply changes:

```bash
systemctl restart prometheus
```

## Install Filebeat for Log Collection

To collect logs with Filebeat and send them to Elasticsearch, run the following command:

```bash
filebeat setup -e
```

## Start Filebeat

Start the Filebeat service:

```bash
systemctl restart filebeat
``` 

## 1. Filebeat & Kibana Setup

### Create a new view in Kibana:
1. Go to `Stack Management | Data View` in Kibana.
2. Create a new view.

### Search for logs in Kibana:
- Search for `error` in your stack.

## 2. Install and Configure Fail2Ban

### Install Fail2Ban:
```bash
apt install fail2ban
```

### Enable and start Fail2Ban:
```bash
systemctl enable fail2ban
systemctl start fail2ban
```

### Configure Fail2Ban:
1. Copy the `jail.conf` to `jail.local`:
   ```bash
   cp jail.conf jail.local
   ```
2. Edit the `backend` in `jail.local` to use `systemd`:
   ```bash
   backend = systemd
   ```

### Restart Fail2Ban and tail the logs:
```bash
systemctl restart fail2ban
tail -f /var/log/fail2ban.log
```

### Log in using a wrong password (client machine) to test blocking:
- After a failed login, the client's IP will be blocked.

## 3. Setting Up OSSEC (Host Intrusion Detection System)

### Install necessary dependencies for OSSEC:
```bash
apt-get install build-essential make zlib1g-dev libpcre2-dev libevent-dev libssl-dev libsystemd-dev
```

### Download and install OSSEC:
```bash
cd /opt
mkdir ossec
cd ossec
wget https://github.com/ossec/ossec-hids/archive/3.7.0.tar.gz
tar -zxvf 3.7.0.tar.gz
cd ossec-hids-3.7.0/
./install.sh
```

### Select "local" for the installation type and follow the prompts to configure OSSEC.

### Install OSSEC using apt (for agent or server):
- To install the server:
  ```bash
  sudo apt-get install ossec-hids-server
  ```
- To install the agent:
  ```bash
  #sudo apt-get install ossec-hids-agent
  ```

### Restart the OSSEC server:
```bash
root@g1:/var/ossec/bin# ./ossec-control restart
```

### List all connected agents:
```bash
root@g1:/var/ossec/bin# ./agent_control -l
```

## 4. Send OSSEC Logs to Elasticsearch using Filebeat

### Set OSSEC server to log in JSON format:
1. In `/var/ossec/etc/ossec.conf`, add the following:
   ```xml
   <global>
     <jsonout_output>yes</jsonout_output>
   </global>
   ```

### Restart OSSEC server:
```bash
# Restart OSSEC to activate JSON logging.
```

### Configure Filebeat to send OSSEC logs to Elasticsearch:
1. Edit Filebeat config file and add:
   ```yaml
   input_type: log
   paths:
     - /var/ossec/logs/alerts/alerts.json
   json.keys_under_root: true
   fields: {log_type: osseclogs}
   ```
```

This markdown structure organizes all the commands and instructions as per your original text for easier readability and use.

