# AWS Highly Available VPC Architecture Project

## Overview

This project demonstrates the design and deployment of a highly available, secure, and auto scalable AWS infrastructure using core AWS networking and compute services.

The architecture follows enterprise-level cloud best practices and was implemented to provide:

- High Availability
- Scalability
- Secure Network Segmentation
- Bastion-Based Secure Access
- Centralized Monitoring & Logging
- Automated Infrastructure Provisioning
- Load Balancing
- Private EC2 Deployment
- Session Manager Secure Access

---

# Architecture Design

## Infrastructure Architecture
<img width="1672" height="941" alt="VPC-Architecture" src="https://github.com/user-attachments/assets/8b6f1398-d941-4f3b-b7a0-a1f232dbcb9f" />


<img width="1364" height="689" alt="website" src="https://github.com/user-attachments/assets/e07f82da-2544-40de-8116-a392a3d5216e" />

---

# AWS Services Used

- Amazon EC2
- Amazon VPC
- Internet Gateway (IGW)
- NAT Gateway
- Transit Gateway
- Auto Scaling Group (ASG)
- Launch Template
- Network Load Balancer (ALB)
- IAM Roles & Policies
- AWS Systems Manager (SSM)
- Amazon S3
- Amazon CloudWatch
- VPC Flow Logs

---

# Project Architecture

## VPC Design

Two isolated VPCs were created:

| VPC Name | CIDR Block | Purpose |
|---|---|---|
| Bastion VPC | `192.168.0.0/16` | Secure administrative access |
| Application VPC | `172.32.0.0/16` | Hosting private application servers |

---

## Subnet Architecture

The infrastructure was deployed across multiple Availability Zones for high availability.

### Public Subnets

Used for:

- Bastion Host
- Application Load Balancer
- NAT Gateway

### Private Subnets

Used for:

- Application EC2 Instances
- Auto Scaling Group Instances

---

# Networking Components

## Internet Gateway (IGW)

Each VPC is connected to an Internet Gateway to allow internet connectivity for public resources.

---

## NAT Gateway

A NAT Gateway was deployed in the public subnet to provide outbound internet access for private instances securely without exposing them to inbound internet traffic.

---

## Transit Gateway

AWS Transit Gateway was configured to establish secure communication between:

- Bastion VPC
- Application VPC

This enabled private administrative access from the Bastion Host to the application servers.

---

# Security Configuration

## Bastion Host Security Group

Configured to allow:

| Protocol | Port | Source |
|---|---|---|
| SSH | 22 | My Public IP |

---

## Application Security Group

Configured to allow:

| Protocol | Port | Source |
|---|---|---|
| SSH | 22 | Bastion Security Group |
| HTTP | 80 | Load Balancer Security Group |

---

# IAM Roles & Policies

IAM Roles were configured following the Principle of Least Privilege.

Attached permissions include:

- AmazonSSMManagedInstanceCore
- Custom S3 Read Access Policy
- CloudWatch Agent permissions

Instead of using full administrative access, custom IAM policies were implemented for enhanced security.

---

# Monitoring & Logging

## CloudWatch Log Groups

Centralized CloudWatch Log Groups were configured for:

- VPC Flow Logs
- Network Traffic Monitoring
- Instance Logs

---

## VPC Flow Logs

Enabled on both VPCs to monitor:

- Accepted Traffic
- Rejected Traffic
- Source/Destination IPs
- Network Activity

---

# Compute Layer

## Golden AMI

A reusable Golden AMI was created containing:

- Apache HTTP Server (`httpd`)
- AWS CLI
- SSM Agent
- Application Dependencies
- Preconfigured Environment

This Golden AMI is used by the Auto Scaling Group instances.

---

## Launch Template

The Launch Template includes:

| Configuration | Value |
|---|---|
| AMI | Golden AMI |
| Instance Type | `t3.micro` |
| IAM Role | Attached |
| Security Group | Configured |
| Key Pair | Configured |
| User Data | Automated |

---

# EC2 User Data Automation

The User Data script automatically configures instances during launch.

## User Data Responsibilities

- Install required packages
- Configure Apache Web Server
- Pull website files from S3
- Start and enable services
- Deploy static website files automatically
  

### Infrastructure Automation (User Data Script)
When instances are launched by the Auto Scaling Group, they are automatically bootstrapped using the following Bash script:

## 📜 Infrastructure Automation (User Data Script)

When EC2 instances are launched dynamically by the Auto Scaling Group, they automatically fetch the deployment package and bootstrap the web server environment using the following initialization script:

<pre><code>#!/bin/bash
# Update system packages
yum update -y

# Install Apache Web Server and AWS CLI
yum install -y httpd aws-cli

# Start and enable Apache service
systemctl start httpd
systemctl enable httpd

# Deploy static web assets from S3 securely
cd /var/www/html
aws s3 cp s3://app-config-s3-bucket-khaled/AWS-VPC-Architecture/html-web-app/ /var/www/html/ --recursive

# Restart Apache to apply changes
systemctl restart httpd</code></pre>

---

## ⚙️ Core Components & Configuration

### 🚀 Auto Scaling Group (ASG)
The ASG is configured to handle dynamic traffic loads and ensure the application remains self-healing.

| Setting | Value |
| :--- | :--- |
| **Minimum Capacity** | 2 |
| **Desired Capacity** | 2 |
| **Maximum Capacity** | 4 |

**Key Features Deployed:**
* **Multi-AZ Deployment:** Instances are distributed across separate Availability Zones for blast-radius isolation.
* **Automatic Instance Replacement:** Failed or unhealthy instances are automatically terminated and replaced.
* **Elastic Scaling:** Scales capacity up or down based on demand.

### 🔀 Load Balancing (ALB)
A public-facing **Application Load Balancer (ALB)** is deployed across public subnets to manage incoming traffic.
* **Traffic Distribution:** Evenly distributes user requests to healthy backend instances.
* **Health Checks:** Continuously monitors instance health and stops routing traffic to unhealthy ones.
* **Target Group Automation:** ASG instances automatically register/deregister themselves within the Target Group.

### 📦 Storage & Content Delivery
* **Amazon S3 Integration:** Used as a secure, centralized repository to store static website files, HTML/CSS/JS assets, and configuration files.
* **Automated Pulls:** Instances leverage IAM roles to securely fetch deployment files at launch time without hardcoded credentials.

---

## 🛡️ Security & Operations Validation

The following validation steps were successfully completed during testing:

* **[✓] Secure Bastion Host Access:** Successfully established secure SSH tunnels to private EC2 instances via a dedicated Bastion Host in the public subnet.
* **[✓] AWS Systems Manager (SSM):** Validated secure, passwordless terminal access to instances using **SSM Session Manager**, eliminating the need to open inbound SSH ports (Port 22).
* **[✓] Web App Public Validation:** Confirmed the web application is publicly accessible and responsive via the **Load Balancer DNS endpoint**.
* **[✓] Golden AMI & Launch Template:** Verified that new instances spin up seamlessly with pre-configured requirements.

---

## 💡 Challenges Solved During Implementation

* **Private Subnet Internet Access:** Configured **NAT Gateways** properly within public subnets to allow private instances to download updates and pull S3 assets.
* **IAM Least Privilege Access:** Debugged and resolved SSM Agent permission issues and S3 bucket policies by attaching proper IAM Roles to the EC2 Instance Profiles.
* **Multi-VPC Routing:** Designed and implemented cross-VPC communication efficiently using **AWS Transit Gateway**.

---

## 🔗 Project Repository
* **GitHub Repository:** [Khalednabeel01/AWS-VPC-Architecture](https://github.com/Khalednabeel01/AWS-VPC-Architecture)
