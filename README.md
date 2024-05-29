![Alt text](/Host_a_WordPress_Website_on_AWS.png)


# Hosting a WordPress Website on AWS

This repository contains the resources and scripts used to deploy a WordPress website on Amazon Web Services (AWS). The project leverages various AWS services to ensure high availability, scalability, and security for the WordPress application.

## Architecture Overview

The WordPress website is hosted on EC2 instances within a highly available and secure architecture that includes:

- A Virtual Private Cloud (VPC) with public and private subnets across two Availability Zones (AZs) for fault tolerance and high availability.
- An Internet Gateway to allow communication between instances in the VPC and the internet.
- Security Groups acting as a virtual firewall to control inbound and outbound traffic.
- Public Subnets used for the NAT Gateway and Application Load Balancer, facilitating external access and load balancing.
- Private Subnets for web servers to enhance security.
- EC2 Instance Connect Endpoint for secure SSH access.
- An Application Load Balancer with a target group to distribute incoming web traffic across multiple EC2 instances.
- An Auto Scaling Group to automatically adjust the number of EC2 instances based on traffic, ensuring scalability and resilience.
- Amazon RDS for a managed relational database service.
- Amazon EFS for a scalable, elastic file storage system.
- AWS Certificate Manager for managing SSL/TLS certificates.
- AWS Simple Notification Service (SNS) for notifications related to the Auto Scaling Group activities.
- Amazon Route 53 for domain name registration and DNS management.

## Deployment Scripts

### WordPress Installation Script

This script is used for the initial setup of the WordPress application on an EC2 instance. It includes steps for installing Apache, PHP, MySQL, and mounting the Amazon EFS to the instance.

```bash
# Create to root user
sudo su

# Update the software packages on the EC2 instance 
sudo yum update -y

# Create an html directory 
sudo mkdir -p /var/www/html

# Environment variable
EFS_DNS_NAME=fs-05d8fb1f43c48e885.efs.us-east-2.amazonaws.com

# Mount the EFS to the html directory 
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install the Apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 8 along with several necessary extensions for WordPress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install the MySQL version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 

# Install the MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
sudo chown apache:apache -R /var/www/html 

# Download WordPress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Edit the wp-config.php file
sudo vi /var/www/html/wp-config.php

# Restart the web server
sudo service httpd restart
```

### Auto Scaling Group Launch Template Script

This script is included in the launch template for the Auto Scaling Group, ensuring that new instances are configured correctly with the necessary software and settings.

```bash
#!/bin/bash
# Update the software packages on the EC2 instance 
sudo yum update -y

# Install the Apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 8 along with several necessary extensions for WordPress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install the MySQL version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 

# Install the MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Environment variable
EFS_DNS_NAME=fs-05d8fb1f43c48e885.efs.us-east-2.amazonaws.com

# Mount the EFS to the html directory 
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set permissions
sudo chown apache:apache -R /var/www/html

# Restart the web server
sudo service httpd restart


## Conclusion

Deploying a WordPress website on AWS using the provided architecture and scripts ensures a robust, scalable, and secure solution for hosting your application. By leveraging AWS services like EC2, RDS, EFS, and Auto Scaling, you can achieve high availability and fault tolerance, while the use of security groups, VPC configurations, and AWS Certificate Manager enhances the overall security of your setup. This approach not only meets the demands of fluctuating web traffic but also simplifies management and maintenance, allowing you to focus on content creation and user engagement. Happy hosting!
