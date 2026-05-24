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
<img width="1666" height="944" alt="VPC-Architecture" src="https://github.com/user-attachments/assets/13f8ceff-a7c6-491f-8527-c42be67dbcd8" />
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

### **Example:**

```bash
#!/bin/bash

yum update -y
yum install -y httpd aws-cli

systemctl start httpd
systemctl enable httpd

cd /var/www/html

sudo aws s3 cp s3://app-config-s3-bucket-khaled/AWS-VPC-Architecture/html-web-app/ /var/www/html/ --recursive

systemctl restart httpd

## **Auto Scaling Group (ASG)**

The Auto Scaling Group was configured to ensure high availability and scalability.

Setting	Value
Minimum Capacity	2
Desired Capacity	2
Maximum Capacity	4
Features
Multi-AZ Deployment
Automatic Instance Replacement
High Availability
Fault Tolerance
Elastic Scaling
Load Balancer
Aplication Load Balancer (ALB)

A public-facing Network Load Balancer was deployed across public subnets.

# **Responsibilities include:**

Traffic Distribution
High Availability
Health Checks
Public Access to Application
Amazon S3 Integration

# **An S3 bucket was used to store:**

Static Website Files
HTML/CSS/JS Assets
Application Deployment Files
Configuration Files

Website files were automatically pulled to EC2 instances using User Data scripts.


The following validation steps were successfully completed:

Bastion Host Access
Successfully connected to private EC2 instances through Bastion Host using SSH.
Session Manager Validation
Successfully connected to EC2 instances using AWS Systems Manager Session Manager.
Website Validation
Successfully accessed the web application publicly using:
Load Balancer ALB
Auto Scaling Validation
Auto Scaling instances launched successfully using the Golden AMI and Launch Template.
Instances automatically registered with the Target Group.
Key Learning Outcomes

This project strengthened practical experience in:

AWS Networking
Cloud Security
Infrastructure Automation
Auto Scaling
High Availability Design
Load Balancing
IAM & Least Privilege Access
Route53 DNS Management
Cloud Monitoring
Secure Bastion Architecture
AWS Systems Manager
DevOps Infrastructure Design
Challenges Solved During Implementation
Configuring private subnet internet access using NAT Gateway
Fixing SSM Agent IAM permission issues
Configuring Auto Scaling instances with IAM Roles
Deploying static website files from S3 automatically
Configuring public Load Balancer subnets correctly
Registering instances automatically inside Target Groups
Multi-VPC communication using Transit Gateway
Project Repository

# **GitHub Repository:**

https://github.com/Khalednabeel01/AWS-VPC-Architecture
