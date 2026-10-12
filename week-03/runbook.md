# Week 3 Compute: EC2 Deployment, EBS Snapshots & Security Triage


This week’s goal is to deploy the PropertyLite Flask application of Week 0 to the AWS EC2 cloud infrastructure, use User Data to achieve automatic startup configuration, complete disk status snapshots through EBS Snapshot, and implement dynamic IP drift debugging of the network security group (Security Group).


## Lab 3.1: Deploy PropertyLite to EC2 via Bootstrap Script

### 1. Configuration and startup parameters
* **Instance Name**: `property-api-01`
* **AMI**: `Amazon Linux 2023 AMI` (Free Tier eligible)
* **Instance Type**: `t3.micro`
* **Key Pair**: `training-key` (RSA, `.pem` format)
* **Security Group Inbound Rules**:
* `SSH (Port 22)`: restrict source to `My IP`
* `Custom TCP (Port 8080)`: Open source is `Anywhere (0.0.0.0/0)`, corresponding to PropertyLite's API listening port


## Debug Lab 3.3: Triage SSH Connection Timeout & IP Drift Analysis

### 1. Symptom (Symptom)
After switching the network environment, execute SSH connection to the EC2 instance on the local terminal:
```bash
ssh -i ~/.ssh/training-key.pem ec2-user@54.81.94.69