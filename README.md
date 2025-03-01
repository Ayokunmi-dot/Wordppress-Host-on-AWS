# WordPress on AWS

## Overview
This project demonstrates how to host a WordPress website on AWS using a combination of AWS services and DevOps principles. The infrastructure has been designed for high availability, fault tolerance, and security.

## Architecture
The project employs the following AWS resources:

1. **Virtual Private Cloud (VPC)**: Configured with both public and private subnets across two Availability Zones.
2. **Internet Gateway**: Enables connectivity between VPC instances and the wider Internet.
3. **Security Groups**: Implemented as a firewall mechanism.
4. **Multi-AZ Deployment**: Ensures high availability and fault tolerance.
5. **Public Subnets**: Used for NAT Gateway and Application Load Balancer.
6. **EC2 Instance Connect Endpoint**: Provides secure connections to both public and private subnets.
7. **Private Subnets**: Houses web servers (EC2 instances) for enhanced security.
8. **NAT Gateway**: Allows instances in private subnets to access the Internet.
9. **EC2 Instances**: Hosts the WordPress website.
10. **Application Load Balancer (ALB) & Target Group**: Distributes traffic across EC2 instances.
11. **Auto Scaling Group (ASG)**: Manages EC2 instances for scalability and fault tolerance.
12. **GitHub**: Used for storing scripts and managing version control.
13. **AWS Certificate Manager**: Secures communications.
14. **AWS Simple Notification Service (SNS)**: Sends notifications related to Auto Scaling activities.
15. **Route 53**: Handles domain registration and DNS configuration.
16. **Elastic File System (EFS)**: Used for shared file storage.
17. **Relational Database Service (RDS)**: Manages the MySQL database.

## Deployment Steps
### 1. Setup VPC and Networking
- Create a VPC with public and private subnets.
- Attach an Internet Gateway.
- Configure Security Groups and NAT Gateway.
- Enable EC2 Instance Connect Endpoint.

### 2. Configure EC2 Instances for WordPress
- Deploy EC2 instances in private subnets.
- Install Apache, PHP, and MySQL.
- Configure an Auto Scaling Group.
- Mount EFS for shared storage.

### 3. Install and Configure WordPress
Run the following script on the EC2 instance:
```bash
sudo su
sudo yum update -y
sudo mkdir -p /var/www/html
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server
sudo systemctl start mysqld
sudo systemctl enable mysqld
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
sudo chown apache:apache -R /var/www/html
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo service httpd restart
```

### 4. Configure Auto Scaling Group
The Auto Scaling Group is defined to ensure fault tolerance and scalability.

#### Launch Template Script:
```bash
#!/bin/bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server
sudo systemctl start mysqld
sudo systemctl enable mysqld
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a
sudo chown apache:apache -R /var/www/html
sudo service httpd restart
```
![Architecture Diagram](Wordpress_Host_on_AWS_Architecture.png)

```



## Deployment Verification
1. Ensure EC2 instances are running and accessible.
2. Verify Auto Scaling Group behavior.
3. Confirm WordPress is accessible via the domain name.
4. Check SNS notifications and CloudWatch logs.

## Security Measures
- Used Security Groups for access control.
- Hosted web servers in private subnets.
- Enforced HTTPS using AWS Certificate Manager.
- Utilized IAM roles for EC2 permissions.

## Conclusion
This AWS-based WordPress deployment ensures scalability, high availability, and security, leveraging AWS best practices and automation techniques.

