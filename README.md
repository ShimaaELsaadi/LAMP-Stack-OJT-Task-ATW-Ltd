# LAMP Stack Project: Setup and Deployment Guide
![LAMP Stack Architecture](images/lamp.png)


## Overview
This document details the steps to set up a LAMP stack on a Linux server, create a simple PHP-based website with database functionality, and make it publicly accessible via AWS.

---

## Description

[LAMP]is a powerful bash script for the installation of Apache + PHP + MySQL/MariaDB and so on. You can install Apache + PHP + MySQL/MariaDB in an very easy way, just need to choose what you want to install before installation.

---
## Prerequisites
- AWS Account
- Basic knowledge of Linux and Apache
- SSH client for connecting to the server

---

## Steps

## **Task 1: Install Required Packages**

### **Local Machine**
1. **Update Package List:**
   ```bash
   sudo apt-get update
   ```
   ![apt-get update result](images/apt-update.png)

2. **Install Apache, MySQL, and PHP:**
   ```bash
   sudo apt-get install apache2 mysql-server php libapache2-mod-php php-mysql -y
   ```
   ![Install Apache, MySQL, and PHP](images/install.png)
3. **Verify Installation:**
   - Text:
     ```bash
     sudo systemctl status apache2
     ```
     ![Text](images/testApache.png)
   - Check MySQL status:
     ```bash
     sudo systemctl status mysql
     ```
     ![Text](images/testMySQL.png)

### **AWS EC2 Instance**
1. **Launch an EC2 Instance:**
   - Go to the [AWS Management Console](https://aws.amazon.com/console/).
   - Sign in with your AWS account credentials.
   - In the AWS Management Console, locate the **Services** dropdown.
   - Click on **EC2** under the Compute section.
    ![Text](images/EC2seviceAWS.png)
   - In the EC2 Dashboard, click on the **Launch Instances** button.
    ![Text](images/LUNCHEC2.png)
   - Select an Ubuntu AMI 
    ![Text](images/EC2AMI.png)
   - Choose an instance type.
    ![Text](images/EC2TYPE.png)
   - Configure security groups to allow SSH (port 22).
   - Select an existing key pair or create a new one:
        If creating a new key pair, download the `.pem` file and keep it secure.
    ![Text](images/SSHKEY.png)

2. **SSH into the Instance:**
   ```bash
   ssh -i lampkey.pem ubuntu@3.142.252.60
   ```

3. **Install Apache, MySQL, and PHP:**
   Follow the same steps as in the Local Machine section.

---

## **Task 2: Configure Apache**

1. **Ensure Apache Serves Files from `/var/www/html/`:**
   - This is the default configuration for Apache.
       ```bash
         ls /var/www/html/
        ```
    ![Text](images/apachconfj.png)
2. **Access the Website Locally:**
   - Open a browser and navigate to:
     - Visit:
     ```
     http://localhost/
     ```
     (Local Machine) 

### **AWS EC2 Instance**
1. **Access the Website Externally:**
   - Obtain the public IP from the EC2 dashboard.
   - In your browser, visit:
     ```
     http://<public-ip>/
     ```
    ![Text](images/defultapache.png)
---

## **Task 3: Create a Simple Website**

1. **Replace `index.html` with `index.php`:**
    - Remove the existing index.html file:
        ```bash
         sudo rm /var/www/html/index.html
        ```
   - create a new index.php file:
        ```bash
         sudo nano /var/www/html/index.php
        ```

   - Add the following PHP code to the file
     <?php
        echo "<h1>Hello, World!</h1>";
     ?>
   - Save the file and exit the editor (in Nano, press CTRL + X, then Y, and then Enter)
   - Restart Apache
        ```bash
        sudo systemctl restart apache2
        ```
2. **Verify PHP Functionality:**
   - Visit:
     ```
     http://localhost/
     ```
     (Local Machine) or
     ```
     http://3.142.252.60/
     ```
     (AWS EC2)
    ![Text](images/hellophp.png) 

---

## **Task 4: Configure MySQL**

### **Secure MySQL Installation**
1. **Run the Secure Installation Script:**
   ```bash
   sudo mysql_secure_installation
   ```

2. **Follow the Prompts:**
   - Set a strong root password.
   - Remove anonymous users.
   - Disallow root login remotely.
    ![Text](images/setupMySQL.png) 
   - Remove test databases.
   - Reload privilege tables.
    ![Text](images/setupMySQL2.png) 

### **Create a Database and User**
1. **Log into MySQL:**
   ```bash
   sudo mysql -u root -p
   ```

2. **Run the Following Commands:**
   ```sql
   CREATE DATABASE web_db;
   CREATE USER 'web_user'@'localhost' IDENTIFIED BY 'StrongPassword@123';
   GRANT ALL PRIVILEGES ON web_db.* TO 'web_user'@'localhost';
   FLUSH PRIVILEGES;
   exit;
   ```
   ![Text](images/SQLedit.png)
3. **Verification:**
    - To verify that your new database and user were created successfully, you can log back into MySQL and run the following commands:
    ```sql
    SHOW DATABASES;
    SELECT User, Host FROM mysql.user;
   ```  
   ![Text](images/dbsqltest.png)

---

## **Task 5: Modify the Website to Use the Database**

1. **Edit `index.php`:**
   ```bash
   sudo nano /var/www/html/index.php
   ```

2. **Replace with the Following Code:**
   ```php
   <?php
        // Database configuration
        $servername = "localhost";
        $username = "web_user"; // MySQL username
        $password = "StrongPassword@123"; // MySQL password
        $dbname = "web_db"; // database name

        // Create connection
        $conn = new mysqli($servername, $username, $password, $dbname);

        // Check connection
        if ($conn->connect_error) {
            die("Connection failed: " . $conn->connect_error);
        }

        // Create a simple table if it doesn't exist
        $sql = "CREATE TABLE IF NOT EXISTS visits (
            id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
            ip_address VARCHAR(45) NOT NULL,
            visit_time DATETIME DEFAULT CURRENT_TIMESTAMP
        )";

        if ($conn->query($sql) === TRUE) {
            // Insert visitor's IP address into the table
            $visitor_ip = $_SERVER['REMOTE_ADDR'];
            $insert_sql = "INSERT INTO visits (ip_address) VALUES ('$visitor_ip')";
            $conn->query($insert_sql);
        }

        // Get the current time
        $current_time = date('Y-m-d H:i:s');

        // Display the message
        echo "<h1>Hello, World!</h1>";
        echo "<p>Your IP address is: " . $visitor_ip . "</p>";
        echo "<p>The current time is: " . $current_time . "</p>";

        // Close connection
        $conn->close();
        ?>

   ```

3. **Test the Updated Website:**
   - Visit:
     ```
     http://localhost/
     ```
     (Local Machine) 
     ```
     or 
    http://3.142.252.60/  
    ```
     (AWS EC2)
     ```


## **Task 6: Testing Locally**
1. **Local Machine:**
   - Open a browser and navigate to:
     ```
     http://localhost/
     ```
    ![Text](images/weblocal.png)


2. **AWS EC2 Instance:**
   - Access the public IP in a browser:
     ```
    http://3.142.252.60/
     ```
    ![Text](images/WEBAWS.png)

3. **Verification**
   - After testing, you can log into MySQL and check if the visits table has been populated with your IP address:
    ```sql
    USE web_db;
    SELECT * FROM visits;
    ```
    (Local Machine)
    ![Text](images/tablelocal.png)

    (AWS EC2)
    ![Text](images/tableaws.png)
     ```
---
## **Task 7: Make the Website Publicly Accessible**

1. **Configure Security Groups:**
   - In AWS, edit the security group attached to the instance:
     - Add a rule for HTTP (port 80) with source `0.0.0.0/0`.
     - Add a rule for HTTPS (port 443).
        ![Text](images/NETRULES.png)

2. **Obtain the Public IP:**
   - From the AWS EC2 dashboard, locate the public IPv4 address.

3. **Test Accessibility:**
   - Open a browser and navigate to:
     ```
     http://3.142.252.60/
     ```

4. **Troubleshooting:**
   - Ensure Apache is running:
     ```bash
     sudo systemctl status apache2
     ```
   - Verify firewall settings:
     ```bash
     sudo ufw allow 80
     sudo ufw allow 443
     ```
# Notes
- Remember to secure your AWS instance by limiting IP ranges for SSH and other services.
- Always use strong passwords for database users.

---

## References
- [LAMP Stack Documentation](https://www.linux.com/what-is-lamp-stack)
- [AWS EC2 User Guide](https://docs.aws.amazon.com/ec2/index.html)

---

If you encounter any issues, feel free to reach out for support!
---

# Git & GitHub

## Steps

### 1. Initialize Git Locally
1. Open your terminal and navigate to your project directory:
   ```bash
   cd /home/shaimaa/LAMP Stack-(OJT)Task-ATW Ltd
   ```
   ![Text](images/cd.png)
2. Initialize Git in the project directory:
   ```bash
   git init
   ```
   ![Text](images/gitinit.png)
---

### 2. Create a `.gitignore` File
1. Create a `.gitignore` file using a text editor:
   ```bash
   nano .gitignore
   ```
2. Add entries to exclude sensitive files and unnecessary files. The content:
   ```
    # Ignore sensitive files
    *.env
    config.php
    db_credentials.php
    .gitignore
    # Ignore log files
    *.log
    lamp.log
    upgrade_apache.log
    upgrade_db.log
    upgrade_php.log
    upgrade_phpmyadmin.log
    upgrade_adminer.log
    # Ignore system files
    .DS_Store
    .vscode/
    pmaversion.txt
    software/
    mysql_bkup/
    mariadb_bkup/
   ```
3. Save and exit the file (`CTRL + O`, `Enter`, `CTRL + X`).

---

### 3. Commit Documentation & Source Code
1. Add your `README.md` file to the project directory.
2. Stage all files for the initial commit:
   ```bash
   git add .
   ```
3. Commit the files with a descriptive message:
   ```bash
   git commit -m "Initial commit: Add documentation and website files"
   ```
   ![Text](images/gitcommit.png)
---

### 4. Create and Push to a GitHub Repository
1. **Create a New Repository on GitHub**:
   - Log in to GitHub.
   - Click on **New Repository**.
   ![Text](images/newrepo.png)
   - Fill in the repository details:
     - **Name**:`LAMP Stack - (OJT)Task- ATW Ltd`.
     - **Visibility**: Public.
      ![Text](images/repoconfig.png)
   - Click **Create Repository**.
   ![Text](images/repo.png)


2. **Add the GitHub Repository as a Remote**:
   ```bash
   git remote add origin <your-github-repo-url>
   ```
   Replace `<your-github-repo-url>` with the repository URL, e.g., `https://github.com/username/LAMP-Stack-Setup.git`.

3. **Push to GitHub**:
   ```bash
   git push -u origin main
   ```

---

## Verification
- Confirm the repository is visible on your GitHub account.
- Ensure sensitive files listed in `.gitignore` are excluded.

---

# Networking Basics

<p align="center">
  <img src="https://static.itez.com/itez-content/iAd6BhFDC12Kn6YjJHByglje0c9669XOskKQX3zp.png" alt="networks">
</p>

## Description
in this part we address some networking basics including IP address, MAC address and other essential parts of any network.

## Table of Contents
- [IP address](#IP-address)
    - [What's an IP address?](##What's-an-IP-address?)
    - [Types of IP Addresses](##Types-of-IP-Addresses)
    - [Purpose of an IP Address](##Purpose-of-an-IP-Address)
- [MAC address](#MAC-address)
    - [What's MAC address?](#What's-MAC-address?)
    - [Characteristics of MAC Addresses](#Characteristics-of-MAC-Addresses)
    - [Differences Between MAC and IP Addresses](#Differences-Between-MAC-and-IP-Addresses)
- [Switches, Routers, and Routing Protocol](#Switches,-Routers,-and-Routing-Protocol)
    - [Switches](#Switches)
    - [Routers](#Routers)
    - [Routing protocols](#Routing-protocols)
- [Connect to Cloud based Linux Instance](#Connect-to-Cloud-based-Linux-Instance)

# IP address
## What's an IP address?
**IP Address** stands for **Internet Protocol** Address. It is a unique numerical label assigned to each device connected to a computer network that uses the Internet Protocol for communication. The IP address serves two primary functions:
1. Identification: It identifies the host or network interface on the network.

2. Location Addressing: It indicates where the device is located within the network, allowing data to be routed correctly.

**IP Addresses** is assigned to identify each device as it's included in the packets of data to identify and manipulate packets inside the network.
the following figure dipicts how IP addresses is assigned and used:

<p align="center">
  <img src="https://www.ipxo.com/app/uploads/2024/03/blog-image.png" alt="IP address">
</p>


## Types of IP Addresses
1. IPv4: The most widely used version, consisting of four octets (e.g., 192.168.1.1). It allows for approximately 4.3 billion unique addresses.
<p align="center">
  <img src="https://hw-images.hostwinds.com/strapi-images/ipv4_address_format_01_17ec1ede2f.webp" alt="IPv4">
</p>

2. IPv6: Introduced to address the limitations of IPv4, it uses a longer format with eight groups of hexadecimal numbers (e.g., 2001:0db8:85a3:0000:0000:8a2e:0370:7334), allowing for an immense number of unique addresses.
<p align="center">
  <img src="https://www.cloudns.net/blog/wp-content/uploads/2023/11/IPv6.png" alt="IPv6">
</p>

## Purpose of an IP Address
1. Routing: IP addresses enable routers to forward data packets to the correct destination. Each packet contains the sender's and receiver's IP addresses, ensuring that data reaches its intended target.

<p align="center">
  <img src="https://www.ipxo.com/app/uploads/2022/04/Network-Routing5.png" alt="routing">
</p>

2. Network Interface Identification: Devices on a local network (such as computers, printers, and smartphones) are identified by their IP addresses, allowing for communication and resource sharing.

3. Geolocation: IP addresses can sometimes provide information about the geographical location of a device, which can be useful for various applications, including content delivery and security measures.

<p align="center">
  <img src="https://cdn.prod.website-files.com/5e6a84f09b62b91e6157c149/5e6a8dd88f50f50f9ef987f1_5e37b64f7c077b4c62ef76a2_shutterstock_1363873922.jpeg" alt="GEOLOCATION">
</p>

# MAC address
## What's MAC address?
A **MAC address**, or **Media Access Control address**, is a unique identifier assigned to every network interface card (NIC) on a device. Unlike the **IP address**, which operates at the network layer, the MAC address exists at the data link layer, governing the direct communication between networked entities.

<p align="center">
  <img src="https://cdn.kastatic.org/ka-perseus-images/6a0cd3a5b7e709c2f637c959ba98705ad21e4e3c.svg" alt="networks-layers">
</p>


## Characteristics of MAC Addresses
1. Permanence: Unlike IP addresses, which can change based on network configuration, MAC addresses are usually hard-coded into the NIC by the manufacturer. This makes them permanent identifiers for the hardware.

2. Format: A MAC address consists of 48 bits, typically represented in hexadecimal format. The first half of the address identifies the manufacturer (Organizationally Unique Identifier, or OUI), while the second half identifies the specific device. It is a series of six pairs of hexadecimal digits, separated by colons or hyphens. For example:  00:1A:2B:3C:4D:5E

<p align="center">
  <img src="https://media.fs.com/images/ckfinder/ftp_images/tutorial/mac-addresse-numbers.jpg" alt="mac-address">
</p>

## Differences Between MAC and IP Addresses

<p align="center">
  <img src="https://gnstechnologies.in/img/dyn/blog-images-exploring-the-differ.-mac-addresses_18.webp" alt="MAC-address-vs-IP-address">
</p>

1. Layer of Operation: MAC addresses operate at the data link layer (Layer 2) of the OSI model, while IP addresses function at the network layer (Layer 3).
2. Scope: MAC addresses are used for local network communication, whereas IP addresses are used for routing across networks and the internet.
3. Changeability: MAC addresses are generally fixed and do not change, whereas IP addresses can be dynamically assigned or changed based on network conditions.

# Switches, Routers, and Routing Protocol
## Switches
<p align="center">
  <img src="https://www.dlink.com/us/en/-/media/product-pages/dms/108/dms-108-a1-right-side.png" alt="Switches">
</p>
A switch is a networking device that connects devices within a local area network (LAN). It operates at the data link layer (Layer 2) of the OSI model.

### Purpose

1. Data Forwarding: Switches receive incoming data packets and forward them to the appropriate destination device within the same network. They use MAC addresses to identify devices and determine where to send the data.
2. Segmentation: By connecting multiple devices, switches create separate collision domains, reducing the chances of data collisions and improving overall network efficiency.
3. Broadcast Domain: Switches operate within a single broadcast domain, meaning all devices connected to a switch can receive broadcast messages.

<p align="center">
  <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRGHwu3r9575bOZiB3vCYrvDlwXpIhW_fmhmw&s" alt="Switches">
</p>

### Types of Switches

1. Unmanaged Switches: Basic switches that plug and play without configuration.
2. Managed Switches: Offer advanced features such as VLAN support, traffic monitoring, and configuration options for network management.

    - Key differences between maneged and unmanaged switches.
<p align="center">
  <img src="https://www.techtarget.com/rms/onlineimages/networking-managed_vs_unmanaged_switch_mobile.png" alt="Routers vs Switches">
</p>

## Routers

<p align="center">
  <img src="https://www.lifewire.com/thmb/6MyHXATC81C0nxzHIjB-XqBnxrg=/1500x0/filters:no_upscale():max_bytes(150000):strip_icc()/netgear-router-gaming-5b2c15f6ff1b780037823ccd.jpg" alt="Router">
</p>

A router is a networking device that routes data between different networks. It operates at the network layer (Layer 3) of the OSI model.

### Purpose

1. Inter-Network Communication: Routers connect multiple networks (e.g., connecting a home network to the internet) and facilitate communication between them.
2. IP Addressing: Routers use IP addresses to determine the best path for data packets to reach their destination across different networks.
3. Traffic Management: Routers can manage traffic using protocols like RIP (Routing Information Protocol) or OSPF (Open Shortest Path First) to ensure efficient data transmission.
<p align="center">
  <img src="https://radhikaclasses.com/wp-content/uploads/2020/08/router-gif.gif" alt="Router">
</p>


### Types of Routers

1. Static Routers: Manually configured with fixed routing paths.
2. Dynamic Routers: Automatically update their routing tables based on network changes using dynamic routing protocols.
## Key Differences
Here are some key differences between a router and a switch.
<p align="center">
  <img src="https://planetechusa.com/wp-content/uploads/2021/04/switch-vs-router-chart.png" alt="Routers vs Switches">
</p>

## Routing protocols
Routing protocols are an essential part of computer networking, as they enable devices to communicate and exchange data across different networks. Here are some of the commonly used routing protocols:

### RIP (Routing Information Protocol)
-  distance-vector routing protocol that determines the best path to a destination based on the number of hops (routers) between the source and the destination.
-  Uses a maximum hop count of 15, which means that it is not suitable for large networks.
-  simple and easy-to-configure protocol, making it suitable for small- to medium-sized networks.
<p align="center">
  <img src="https://media.fs.com/images/community/upload/kindEditor/202104/26/what-is-rip-in-networking-1619422898-4RqAymItOt.png" alt="RIP">
</p>

### OSPF (Open Shortest Path First)
- link-state routing protocol that determines the best path to a destination based on the cost of the link.
- uses the Dijkstra algorithm to calculate the shortest path between the source and the destination.
- more complex than RIP, but it is more scalable and can handle larger networks more efficiently.
- supports features such as load balancing, authentication, and fast convergence.
<p align="center">
  <img src="https://resource.fs.com/mall/generalImg/J58VbUKOAop0OdxstwQcapamnVe.png" alt="OSPF">
</p>

### EIGRP (Enhanced Interior Gateway Routing Protocol)
- hybrid routing protocol that combines the features of both distance-vector and link-state protocols.
- uses the Diffusing Update Algorithm (DUAL) to determine the best path to a destination, taking into account factors such as bandwidth, delay, reliability, and load.
- proprietary to Cisco and is often used in Cisco-based networks.
- more complex than RIP but simpler than OSPF, making it a good choice for medium-sized networks.


### BGP (Border Gateway Protocol)
- path-vector routing protocol that is used to exchange routing information between autonomous systems (AS) on the internet.
- the primary routing protocol used on the internet and is responsible for the global routing table.
- complex protocol that is used to manage the large-scale routing of the internet, and it is often used by internet service providers (ISPs) and large organizations.
<p align="center">
  <img src="https://www.techtarget.com/rms/onlineImages/BGP_backbone.png" alt="BGP">
</p>


# Connect to Cloud based Linux Instance
To connect to a cloud-based Linux instance from a remote machine using SSH, follow these steps:

### Obtain the cloud instance's public IP address or hostname:
This information is typically provided by the cloud service provider when you create the instance.
### Ensure you have a valid SSH key pair:
If you don't have an SSH key pair, you can generate one using a tool like ssh-keygen on your local machine.
The public key should be added to the cloud instance's authorized_keys file.
### Configure your local SSH client:
Open your terminal or command prompt on your local machine.
Ensure that the SSH client is installed. Most modern operating systems have an SSH client built-in (e.g., OpenSSH on Linux/macOS, PuTTY on Windows).
### Connect to the cloud instance using SSH:
In the terminal, use the following command to connect to the cloud instance:

    ssh -i <path_to_private_key_file> <username>@<cloud_instance_ip_or_hostname>
- Replace <path_to_private_key_file> with the path to your private SSH key file.
- Replace <username> with the appropriate username for the cloud instance, typically ec2-user for Amazon EC2 instances or ubuntu for Ubuntu-based instances.
- Replace <cloud_instance_ip_or_hostname> with the public IP address or hostname of the cloud instance.
### Authenticate using your SSH key:
- If the connection is successful, you will be prompted to authenticate using your SSH private key.
- Provide the passphrase for the private key, if you set one during the key generation process.
### Verify the connection:
Once authenticated, you should be connected to the cloud-based Linux instance, and you can interact with it using the command line.

