# PayFlow AWS Highly Available Web Infrastructure

## Project Overview

This project demonstrates the design and deployment of a **highly available, scalable, and secure web application infrastructure** on Amazon Web Services (AWS).

---

## Project Architecture
<img width="1536" height="1024" alt="Architectural Diagram 1" src="https://github.com/user-attachments/assets/89651126-848f-42c2-834e-f780002517fc" />
---
<img width="1408" height="768" alt="Architectural Diagram 2" src="https://github.com/user-attachments/assets/aa572f75-9671-40c6-848e-cfe2f860acd1" />

---

## Architecture Summary

- Custom VPC with public and private subnets across multiple Availability Zones
- Internet Gateway for public access
- NAT Gateway for private subnet outbound internet access
- Application Load Balancer for traffic distribution
- Auto Scaling Group for elasticity
- EC2 instances running a simple Apache web application

### Tech Stack

- Amazon VPC
- EC2 (Amazon Linux)
- Application Load Balancer (ALB)
- Auto Scaling Groups
- NAT Gateway / Internet Gateway
- IAM & Security Groups
- Apache Web Server (User Data bootstrapping)

---

## Skills Demonstrated

- AWS Cloud Architecture Design  
- Networking (VPC, Subnets, Routing)  
- Load Balancing & Auto Scaling  
- Cloud Security (IAM, Security Groups)  
- Linux Server Configuration  
- High Availability System Design  
- Production-grade infrastructure thinking  

## Step-by-Step Implementation

---

## 1. Create VPC

- Navigate to **VPC → Create VPC**
- Configuration:
  - Name: `payflow-vpc`
  - IPv4 CIDR: `10.0.0.0/16`
<img width="973" height="431" alt="image" src="https://github.com/user-attachments/assets/636a2fc9-6737-422e-9aaa-18bc73fc1c6b" />

---


## 2. Create Subnets

### Public Subnets (Load Balancer Layer)

**Subnet 1**
- Name: `public-subnet-1`
- CIDR: `10.0.1.0/24`
- AZ: `us-east-1a`

**Subnet 2**
- Name: `public-subnet-2`
- CIDR: `10.0.2.0/24`
- AZ: `us-east-1b`

---

### Private Subnets (EC2 Instances)

**Subnet 3**
- Name: `private-subnet-1`
- CIDR: `10.0.3.0/24`
- AZ: `us-east-1a`

**Subnet 4**
- Name: `private-subnet-2`
- CIDR: `10.0.4.0/24`
- AZ: `us-east-1b`
<img width="975" height="293" alt="image" src="https://github.com/user-attachments/assets/a2f76525-362d-4f7d-8019-6134b03fc6ae" />

---

## 3. Create Internet Gateway

- Name: `payflow-igw`
- Attach it to `payflow-vpc`
<img width="975" height="181" alt="image" src="https://github.com/user-attachments/assets/67bd1169-0f89-4279-a13a-65f4495fdda2" />

---

## 4. Create Route Tables

### Public Route Table

- Name: `public-rt`
- Route:
  - Destination: `0.0.0.0/0`
  - Target: Internet Gateway
- Associate with:
  - `public-subnet-1`
  - `public-subnet-2`
<img width="975" height="510" alt="image" src="https://github.com/user-attachments/assets/174f8775-8222-4414-a9e9-629d8f6aee75" />

---

### Private Route Table

- Name: `private-rt`
- Associate with:
  - `private-subnet-1`
  - `private-subnet-2`
<img width="975" height="242" alt="image" src="https://github.com/user-attachments/assets/1316a0b6-1e77-4f25-86de-06f3a2fac437" />

---

## 5. Create NAT Gateway

- Place NAT Gateway in: `public-subnet-1`
- Allocate an Elastic IP
<img width="975" height="532" alt="image" src="https://github.com/user-attachments/assets/f25bd9c3-1f45-42ce-afc9-4b31655703cb" />

### Update Private Route Table

- Add route:
  - Destination: `0.0.0.0/0`
  - Target: NAT Gateway
<img width="973" height="232" alt="image" src="https://github.com/user-attachments/assets/f258ff4e-c751-4557-b8b6-45518c41c664" />

---

## 6. Security Groups

### ALB Security Group

- Allow:
  - HTTP (Port 80) from `0.0.0.0/0`

---

### EC2 Security Group

- Allow:
  - HTTP (Port 80) ONLY from ALB Security Group
<img width="974" height="224" alt="image" src="https://github.com/user-attachments/assets/36548efb-dbe1-4a06-9b58-53ff32ebb48a" />

---

## 7. Create Web Application (EC2 Instance)

- Launch EC2 instance in **private subnet**
- Amazon Linux AMI
- Install Apache web server using User Data:

```bash
#!/bin/bash
yum update -y
yum install -y httpd

systemctl enable httpd
systemctl start httpd

echo "PayFlow Server - $(hostname)" > /var/www/html/index.html
```
<img width="975" height="461" alt="image" src="https://github.com/user-attachments/assets/09507b4e-5aeb-4859-b9be-b6da58a52541" />

---

## 8. Create AMI (Golden Image)

- Convert the configured EC2 instance into an **Amazon Machine Image (AMI)**
- This AMI captures:
- OS configuration
- Installed packages (Apache HTTP Server)
- Application setup
- The AMI will be used by the Auto Scaling Group to launch identical instances
<img width="974" height="219" alt="image" src="https://github.com/user-attachments/assets/55843605-dccf-40dc-828c-42a07f1a902b" />

---

## 9. Create Launch Template

The Launch Template defines how new EC2 instances will be created.

**Configuration:**
- **AMI:** Custom Amazon Linux AMI
- **Instance Type:** `t3.micro`
- **Key Pair:** Configured SSH key pair
- **Security Group:**
- Allow HTTP (Port 80)
- **User Data Script:**
- Installs and configures Apache HTTP Server
<img width="974" height="515" alt="image" src="https://github.com/user-attachments/assets/8a53d99d-a7cf-44d5-83ec-6ef9693c48a0" />

---

## 10. Create Target Group

The Target Group is used by the Load Balancer to route traffic.

**Configuration:**
- **Type:** Instances
- **Protocol:** HTTP
- **Port:** 80

### Health Check
- **Path:** `/`
- Ensures only healthy instances receive traffic
- Unhealthy instances are automatically removed from routing
<img width="974" height="493" alt="image" src="https://github.com/user-attachments/assets/16019871-6282-45c7-8c47-df42d7c83e0c" />

---

## 11. Create Application Load Balancer (ALB)

- **Type:** Internet-facing Load Balancer
- **Listener:** HTTP (Port 80)

### Subnets:
- `public-subnet-1`
- `public-subnet-2`

### Attachments:
- Connect ALB to the Target Group
<img width="974" height="534" alt="image" src="https://github.com/user-attachments/assets/365105d9-4da0-43c4-a9b9-89c157d7ef66" />

---

## 12. Create Auto Scaling Group (ASG)

The ASG ensures high availability and elasticity.

**Configuration:**
- **Launch Template:** Previously created template
- **Subnets:**
- `private-subnet-1`
- `private-subnet-2`
- **Attach Target Group**

### Scaling Settings:
- **Minimum Instances:** 2
- **Desired Instances:** 2
- **Maximum Instances:** 4
<img width="975" height="246" alt="image" src="https://github.com/user-attachments/assets/26368f1e-f148-44cb-8fce-c281c5003860" />

---

## 13. Configure Auto Scaling Policies Using Dynamic Policy (Step Scaling)

### Scale Out Policy
- If CPU utilization **> 70%**
- ➜ Add more EC2 instances automatically

### Scale In Policy
- If CPU utilization **< 30%**
- ➜ Remove excess EC2 instances
<img width="974" height="526" alt="image" src="https://github.com/user-attachments/assets/641c0087-66ce-49d0-83b1-e4af01a6d43e" />
<img width="975" height="540" alt="image" src="https://github.com/user-attachments/assets/2e26e45f-e3ac-4721-8862-365e135363c2" />

---

## 14. Testing Load Balancing

To verify the setup:

1. Copy the **ALB DNS name**
2. Paste it into a browser
3. Refresh multiple times

### Expected Result:
- Different EC2 hostnames appear on each refresh
- Traffic is distributed across multiple instances
- Load Balancing is working correctly
<img width="975" height="124" alt="image" src="https://github.com/user-attachments/assets/361065cb-9517-4959-b61e-9aacbc7f7e6b" />
<img width="975" height="123" alt="image" src="https://github.com/user-attachments/assets/98d0f56d-61ac-4414-9c71-1427a1bd94bd" />


## Outcome

A fully scalable, fault-tolerant web application infrastructure capable of automatically handling traffic changes while maintaining high availability.

## Security Highlights
•	EC2 instances deployed in private subnets 
•	No direct public access to backend servers 
•	Controlled traffic via ALB only 
•	Security groups enforcing least privilege access

