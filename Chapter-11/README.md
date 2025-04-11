```markdown
# How to Setup Webserver & Deploy Web Apps

## Introduction

Running web applications is a standard method of delivering online services today, such as blogs, online shops, and social media platforms. Being able to host and run web applications on Ubuntu servers is a crucial skill. This guide walks you through installing and setting up Apache and Nginx web servers, deploying simple web applications, and working with databases like MariaDB and MongoDB.

## Structure

- [Web Servers](#web-servers)
  - [Apache Installation & Setup](#apache-installation--setup)
  - [Nginx Installation & Setup](#nginx-installation--setup)
- [Databases](#databases)
  - [MariaDB Installation & Setup](#mariadb-installation--setup)
  - [MariaDB Commands](#mariadb-commands)

---

## Web Servers

### Apache Installation & Setup

1. **Install Apache 2:**

   ```bash
   sudo apt install apache2
   ```

2. **View Apache Configuration Files:**

   Apache creates its configuration files in `/etc/apache2`. To see the file structure, run:

   ```bash
   ls -l /etc/apache2
   ```

3. **Understanding Apache's Folder Structure:**
   - `apache2.conf`: The main configuration file.
   - `mods-available`: Folder containing available modules (e.g., PHP).
   - `sites-available`: Folder where site configuration files are stored.

4. **Create Content for Web Server:**
   - Create a folder in `/var/www/html` for your web app:

     ```bash
     mkdir /var/www/html/myapp
     ```

   - Create a simple `index.html` file:

     ```bash
     cat > /var/www/html/myapp/index.html <<EOL
     <html>
         <title>Test</title>
         <h2>Hello</h2>
     </html>
     EOL
     ```

5. **Configure Apache to Serve Your App:**

   Edit `000-default.conf` to point to your new folder:

   ```bash
   nano /etc/apache2/sites-enabled/000-default.conf
   ```

   Update the `DocumentRoot` line:

   ```apache
   DocumentRoot /var/www/html/myapp
   ```

6. **Restart Apache:**

   ```bash
   systemctl restart apache2
   ```

7. **Check Apache Logs:**
   - To monitor access logs:

     ```bash
     tail -f /var/log/apache2/access.log
     ```

### Nginx Installation & Setup

1. **Stop Apache Server:**
   You can't run both Apache and Nginx on the same port. Stop Apache:

   ```bash
   systemctl stop apache2
   ```

2. **Install Nginx:**

   ```bash
   sudo apt install nginx
   ```

3. **Configure Nginx to Serve Your App:**

   Edit the `default` configuration in `/etc/nginx/sites-enabled`:

   ```bash
   nano /etc/nginx/sites-enabled/default
   ```

   Set the root path:

   ```nginx
   root /var/www/html/myapp;
   ```

4. **Restart Nginx:**

   ```bash
   systemctl restart nginx
   ```

5. **Verify Your Setup:**
   Open your browser and go to `http://<server-ip>`. You should see the "Hello" message served by Nginx.

---

## Databases

### MariaDB Installation & Setup

1. **Install MariaDB:**

   ```bash
   sudo apt install mariadb-server mariadb-client -y
   ```

2. **Secure MariaDB Installation:**

   Run the following to secure the installation:

   ```bash
   sudo mysql_secure_installation
   ```

   - Answer "Y" to all questions, except "Switch to unix_socket authentication", which you should answer "N".

3. **Restart MariaDB:**

   ```bash
   systemctl restart mariadb
   ```

4. **Login to MariaDB:**

   ```bash
   mysql -u root -p
   ```

   Enter your password when prompted.

### MariaDB Commands

- **Show Databases:**

   ```sql
   SHOW DATABASES;
   ```

- **Create a Database:**

   ```sql
   CREATE DATABASE mattias;
   ```

- **Drop a Database:**

   ```sql
   DROP DATABASE mattias;
   ```

- **Show Databases Again to Confirm:**

   ```sql
   SHOW DATABASES;
   ```

---

# How to Set Up MongoDB and Web Applications

This guide will walk you through the process of installing MongoDB, setting up web applications, and managing databases, including WordPress and Observium. We'll also cover database backups, creating users, and configuring virtual hosts for Apache.

---

## Table of Contents
1. [Installing MongoDB](#installing-mongodb)
2. [Creating MongoDB Database and Collection](#creating-mongodb-database-and-collection)
3. [Managing Databases](#managing-databases)
4. [Installing phpMyAdmin](#installing-phpmyadmin)
5. [Deploying Web Applications](#deploying-web-applications)
    - [WordPress](#wordpress)
    - [Observium](#observium)
6. [Setting Up Virtual Hosts for Apache](#setting-up-virtual-hosts-for-apache)
7. [Installing Rocket.Chat](#installing-rocketchat)
8. [Web Performance Optimization](#web-performance-optimization)
9. [Backing Up Databases](#backing-up-databases)
10. [Creating Database Users](#creating-database-users)
11. [Conclusions](#conclusions)

---

## 1. Installing MongoDB

Follow these steps to install MongoDB on your server:

1. Add MongoDB's public GPG key:
    ```bash
    curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
      sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
    ```

2. Add MongoDB's repository:
    ```bash
    echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
    ```

3. Update package list:
    ```bash
    sudo apt-get update
    ```

4. Install MongoDB:
    ```bash
    sudo apt-get install -y mongodb-org
    ```

5. Restart MongoDB service:
    ```bash
    systemctl restart mongod
    ```

6. Connect to MongoDB using `mongosh`:
    ```bash
    mongosh
    ```

---

## 2. Creating MongoDB Database and Collection

After MongoDB is set up, follow these steps to create a database and a collection:

1. Switch to the database you want to create (e.g., `mattias`):
    ```bash
    test> use mattias
    ```

2. Create a new collection:
    ```bash
    mattias> db.createCollection("mattias-collections")
    ```

3. Show all collections:
    ```bash
    mattias> show collections
    ```

---

## 3. Managing Databases

You can manage your MongoDB database via the command line or use a web interface for easier management. phpMyAdmin is a popular choice for SQL-based databases like MariaDB.

---

## 4. Installing phpMyAdmin

To install phpMyAdmin on Ubuntu:

1. Install the required package:
    ```bash
    sudo apt install phpmyadmin php libapache2-mod-php
    ```

2. Follow the on-screen instructions to set it up.

3. Visit `http://Your IP/phpMyAdmin` to log in using the root user and the password you set for the MariaDB server.

---

## 5. Deploying Web Applications

### WordPress

To deploy WordPress on your server:

1. Download the latest WordPress package:
    ```bash
    wget https://wordpress.org/latest.zip
    ```

2. Install `unzip` and extract the contents:
    ```bash
    sudo apt install unzip
    unzip latest.zip
    ```

3. Set the correct permissions:
    ```bash
    sudo chown www-data:www-data -R wordpress/
    ```

4. Set up the database in MariaDB (use phpMyAdmin to create a WordPress database).

5. Visit your server's IP to complete the WordPress installation.

---

### Observium

To install Observium:

1. Download the latest version:
    ```bash
    wget http://www.observium.org/observium-community-latest.tar.gz
    ```

2. Extract the package:
    ```bash
    tar zxvf observium-community-latest.tar.gz
    ```

3. Set permissions:
    ```bash
    sudo chown www-data:www-data -R observium
    ```

4. Configure the database for Observium (create a database in phpMyAdmin or via the command line).

5. Configure and install the database tables:
    ```bash
    ./discovery.php -u
    ```

---

## 6. Setting Up Virtual Hosts for Apache

To configure Apache to serve multiple applications:

1. Create a new virtual host configuration file for Observium:
    ```bash
    sudo vi /etc/apache2/sites-available/observium.conf
    ```

    Add the following content:
    ```apache
    <VirtualHost *:80>
        ServerAlias observium.lan
        ServerAdmin webmaster@localhost
        DocumentRoot /opt/observium/html
        <FilesMatch \.php$>
          SetHandler application/x-httpd-php
        </FilesMatch>
        <Directory /opt/observium/html/>
            DirectoryIndex index.php
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Require all granted
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        ServerSignature On
    </VirtualHost>
    ```

2. Enable the new site and restart Apache:
    ```bash
    sudo ln -s /etc/apache2/sites-available/observium.conf /etc/apache2/sites-enabled/
    sudo systemctl restart apache2
    ```

---

## 7. Installing Rocket.Chat

1. Create a MongoDB user for Rocket.Chat:
    ```bash
    mongosh
    use rocket
    db.createUser({
      user: "rocket",
      pwd: "rocketpass",
      roles: [{ role: "readWrite", db: "rocket" }]
    })
    ```

2. Configure MongoDB to accept connections:
    ```bash
    sudo vi /etc/mongod.conf
    # Update bindIp from 127.0.0.1 to 0.0.0.0
    ```

3. Restart MongoDB:
    ```bash
    sudo systemctl restart mongod
    ```

4. Set up Rocket.Chat with Docker:
    Create a `docker-compose.yml` file with the following content:
    ```yaml
    services:
      rocketchat:
        image: registry.rocket.chat/rocketchat/rocket.chat:latest
        restart: always
        environment:
          MONGO_URL: "mongodb://rocket:rocketpass@192.168.1.11:27017/rocket"
          ROOT_URL: http://192.168.1.11:3000
          PORT: 3000
    ```

5. Start Rocket.Chat:
    ```bash
    docker-compose up
    ```

6. Visit `http://192.168.1.11:3000` to access Rocket.Chat.

---

## 8. Web Performance Optimization

To improve performance, enable `KeepAlive` and compression on Apache:

```apache
LoadModule deflate_module modules/mod_deflate.so
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 15
<IfModule mod_deflate.c>
  AddOutputFilterByType DEFLATE text/html text/css application/javascript
</IfModule>
```

For Nginx, enable gzip compression:

```nginx
gzip on;
gzip_types text/plain text/css application/javascript text/html;
```

---

## 9. Backing Up Databases

### MySQL Backup
```bash
mysqldump -u root -p wordpress > /var/backups/wordpress_backup.sql
```

### MongoDB Backup
```bash
mongodump --uri mongodb://rocket:rocketpass@192.168.1.11:27017/rocket --out /var/backups/mongo_backup
```

---

## 10. Creating Database Users

For better security, create specific users for each application:

### MySQL User Creation
```sql
CREATE USER 'dbuser'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON mydatabase.* TO 'dbuser'@'%';
FLUSH PRIVILEGES;
```

### MongoDB User Creation
```bash
use mydatabase
db.createUser({
  user: "appuser",
  pwd: "password",
  roles: [{ role: "readWrite", db: "mydatabase" }]
})
```

---

# How to Set Up Web Servers and Databases: A Step-by-Step Guide

This guide provides instructions on setting up a web server with databases (MariaDB and MongoDB), installing popular web applications like WordPress and Observium, configuring Apache for virtual hosts, and managing backups.

---

## 1. MongoDB Installation

### Add MongoDB Repository and Key

1. **Add MongoDB repository key**:

   ```bash
   curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
   ```

2. **Add MongoDB repository**:

   ```bash
   echo "deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
   ```

3. **Update your package list**:

   ```bash
   sudo apt-get update
   ```

4. **Install MongoDB**:

   ```bash
   sudo apt-get install -y mongodb-org
   ```

### Restart MongoDB and Connect

1. **Restart MongoDB service**:

   ```bash
   sudo systemctl restart mongod
   ```

2. **Connect to MongoDB using Mongosh**:

   ```bash
   mongosh
   ```

---

## 2. Create a MongoDB Database and Collection

1. **Switch to the `mattias` database**:

   ```bash
   use mattias
   ```

2. **Create a collection `mattias-collections`**:

   ```bash
   db.createCollection("mattias-collections")
   ```

3. **Show collections**:

   ```bash
   show collections
   ```

---

## 3. phpMyAdmin Installation for MariaDB

1. **Install phpMyAdmin**:

   ```bash
   sudo apt install phpmyadmin php libapache2-mod-php
   ```

2. **Verify Apache2 server is running** and access phpMyAdmin at `http://your-ip/phpmyadmin`.

3. **Log in** using the root username and the password set for MariaDB.

---

## 4. Deploying Web Applications

### WordPress Installation

1. **Download the latest WordPress**:

   ```bash
   wget https://wordpress.org/latest.zip
   ```

2. **Install `unzip`** and extract the contents:

   ```bash
   sudo apt install unzip
   unzip latest.zip
   ```

3. **Set permissions**:

   ```bash
   sudo chown www-data:www-data -R wordpress/
   ```

4. **Create a MySQL database for WordPress** using phpMyAdmin.

5. **Visit the setup page** and fill in the database information.

### Observium Installation

1. **Download Observium**:

   ```bash
   wget http://www.observium.org/observium-community-latest.tar.gz
   ```

2. **Extract and set permissions**:

   ```bash
   tar zxvf observium-community-latest.tar.gz
   sudo chown www-data:www-data -R observium
   ```

3. **Configure the database for Observium** by editing the `config.php` file.

4. **Run the discovery script** to set up Observium:

   ```bash
   ./discovery.php -u
   ```

5. **Add a user** to Observium:

   ```bash
   ./adduser.php matte password 1
   ```

---

## 5. Configure Apache Virtual Hosts

### Create a Virtual Host for Observium

1. **Create a new virtual host file** for Observium:

   ```bash
   sudo nano /etc/apache2/sites-available/observium.conf
   ```

2. **Add the following content** to the `observium.conf` file:

   ```apache
   <VirtualHost *:80>
       ServerAlias observium.lan
       ServerAdmin webmaster@localhost
       DocumentRoot /opt/observium/html
       <FilesMatch \.php$>
           SetHandler application/x-httpd-php
       </FilesMatch>
       <Directory /opt/observium/html/>
           DirectoryIndex index.php
           Options Indexes FollowSymLinks MultiViews
           AllowOverride All
           Require all granted
       </Directory>
       ErrorLog  ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
       ServerSignature On
   </VirtualHost>
   ```

3. **Activate the virtual host**:

   ```bash
   sudo ln -s /etc/apache2/sites-available/observium.conf /etc/apache2/sites-enabled/
   ```

4. **Restart Apache**:

   ```bash
   sudo systemctl restart apache2
   ```

5. **Update `/etc/hosts` on your local machine**:

   ```bash
   192.168.1.11 observium.lan
   ```

---

## 6. Docker Compose for Rocket Chat

1. **Create MongoDB user for Rocket Chat**:

   ```bash
   mongosh
   use rocket
   db.createUser({ user: "rocket", pwd: "rocketpass", roles: [{ role: "readWrite", db: "rocket" }] })
   ```

2. **Update MongoDB configuration to allow connections**:

   Edit `/etc/mongod.conf`:

   ```yaml
   net:
     port: 27017
     bindIp: 0.0.0.0
   ```

3. **Create Docker Compose file**:

   Create a `docker-compose.yml` file with the following content:

   ```yaml
   services:
     rocketchat:
       image: registry.rocket.chat/rocketchat/rocket.chat:latest
       restart: always
       environment:
         MONGO_URL: "mongodb://rocket:rocketpass@192.168.1.11:27017/rocket"
         ROOT_URL: http://192.168.1.11:3000
         PORT: 3000
       ports:
         - 3000:3000
   ```

4. **Start Rocket Chat**:

   ```bash
   docker-compose up -d
   ```

   Access Rocket Chat at `http://192.168.1.11:3000`.

---

## 7. Web Server Performance Optimization

### Apache Performance Settings

1. **Enable KeepAlive and Compression** in Apache:

   ```apache
   LoadModule deflate_module modules/mod_deflate.so
   KeepAlive On
   MaxKeepAliveRequests 100
   KeepAliveTimeout 15
   ```

2. **Add compression settings**:

   ```apache
   <IfModule mod_deflate.c>
     AddOutputFilterByType DEFLATE text/html text/css text/javascript
   </IfModule>
   ```

### Nginx Performance Settings

1. **Enable Gzip compression** in Nginx:

   ```nginx
   gzip on;
   gzip_types text/plain text/css application/javascript;
   gzip_min_length 1024;
   ```

---

## 8. Database Backup

1. **Backup MariaDB database**:

   ```bash
   mysqldump -u root -p wordpress > wordpress_backup.sql
   ```

2. **Backup MongoDB database**:

   ```bash
   mongodump mongodb://rocket:rocketpass@192.168.1.11:27017/rocket
   ```

---

## 9. Database User Management

1. **Create a new user in MariaDB**:

   ```sql
   CREATE USER 'dbuser'@'%' IDENTIFIED BY 'password';
   GRANT ALL PRIVILEGES ON database.* TO 'dbuser'@'%';
   FLUSH PRIVILEGES;
   ```

---

## Conclusion

In this guide, we have installed and configured essential services including web servers (Apache), databases (MariaDB and MongoDB), and popular applications like WordPress and Observium. We also configured Apache for multiple virtual hosts, optimized web performance, and set up backup and database user management.

