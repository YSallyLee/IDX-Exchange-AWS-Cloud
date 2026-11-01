# IDX Exchange AWS Cloud Engineering Internship

This repository documents my learning and hands-on labs during the
IDX Exchange AWS Cloud Engineering Internship.

---

## Week 01 — Cloud Fundamentals & Account Setup



For week 1, we focus on foundational AWS account security and the critical distinction between the root user and IAM users. The root user is the parent account created when the account was created, and has unrestricted power over the entire AWS account, including closing the account, deleting all data, and modifying credit card payment information. Therefore, we need to lock the MFA and put it in a safe, never use it for routine operations. The routine task must be operated by an IAM user, and needs to follow the principle of least privilege: grant only the necessary permissions, and the access key can be rotated or revoked at any time to reduce the disaster in the event of a leak.

Under the Shared Responsibility Model, Amazon famously states that "AWS secures OF the cloud, you secure IN the cloud." Their responsibility covers physical and underlying defenses, physical data centers, hardware facilities, and virtualization layers. And for the customer side, including  operating system updates, security groups, identity and access control (IAM), and data encryption. This is achieved through the zero-spend budget alert set this week. All of these reinforces the fundamental requirement for cloud architects to strike a balance between system security and cost control.


## Week 02 — IAM & Security Foundations


In the week2 json file, the custom IAM policy conforming to the Least Privilege principle was established and validated. This policy strictly restricts the test user `s3-test-user` to only perform object-level read and upload operations `s3:GetObject and s3:PutObject` under the specified path my-training-bucket-sally/*, while deliberately excluding global storage enumeration permissions `s3:ListAllMyBuckets` and storage-level list permissions `s3:ListBucket`. 

When executing `aws s3 ls --profile s3 test` on the terminal triggers an AccessDenied error, it fully verifies the default denial logic in the AWS policy evaluation mechanism, proving that the user is precisely restricted to the minimum authorized scope and cannot arbitrarily snoop on or laterally access other resources in the account. 

Furthermore, during the troubleshooting process in Debug Lab 2.3, I also gained a deeper understanding of the key differences between object-level ARNs (ending with `/*`) and storage-level ARNs (without `/*`). If a storage-level path is assigned to an object operation (such as s3:PutObject), it will be blocked due to resource mismatch. This demonstrates the importance of accurately defining cloud permission boundaries for system security and problem diagnosis.

---


## Week 3 Compute: EC2 Deployment, EBS Snapshots & Security Triage

For Week 3, I successfully deployed PropertyLite on an EC2 instance using a User Data bootstrap script and verified both the `/health` and properties API endpoints. 

I also practiced point-in-time backups by creating an EBS root volume snapshot. During the debug lab, I triaged an SSH connection timeout caused by client-side IP drift, identifying that Security Group 'My IP' is a static snapshot rather than a dynamic lookup.

[View Week 03 Runbook](./week-03/runbook.md)


---



## Week 4 - AWS Storage & Database Architectural Decision Guide

For Week 4, I the focus was on AWS Storage and Databases. Here’s the step by step  

* S3 & Lifecycle Management: Created an S3 bucket and configured a 30-day lifecycle rule to transition objects to Glacier Flexible Retrieval.   

* RDS & Data Governance: Provisioned an RDS MySQL instance and connected via EC2. I imported the ~600MB `rets_property` dataset and executed the data-governance workflow: dropping 17 sensitive PII columns and remapping MLS IDs to synthetic IDs `(R1xxxxx)` using `ROW_NUMBER()`.   

* DynamoDB & Deliverable: Created the `Properties` DynamoDB table wrote the cleaned synthetic data `(R10001)` for Key-Value query verification. This was mainly to simulate the low-latency scenario of checking details via property ID under high front-end traffic. It also pre-verified the data format for Week 7's Serverless architecture. 


* Debug (from step 46): 
In this week, the original debugging was an RDS security group network timeout, but my network connection worked smoothly during setup, and I didn't encounter any Security Group blocking issues. However, when executing Lab 4.2 data governance, I encountered another very real underlying database error during primary key conversion, because the original table's id field had an `AUTO_INCREMENT` attribute, MySQL restrictions prevented the direct removal of the old primary key, resulting in `ERROR 1075 (42000): Incorrect table definition; there can be only one auto column and it must be defined as a key`. I first removed the auto-increment attribute using `MODIFY id INT NOT NULL`, and then successfully bound the de-identified `L_ListingID` as the new primary key.

[View Week 04 decision-guide](./week-04/decision-guide.md)



## Week 5 - Networking: Custom VPC Deep Dive & Bastion Host Pattern

The focus of this week is to build a secure multi-layer VPC from scratch, and use Bastion Host (bastion host/springboard host) with SSH Agent Forwarding to securely connect to the private host isolated in the backend.


```text
                  Internet (Public Users / Laptop)
                                 │
                   [Internet Gateway (IGW)]
                                 │
  ┌──────────────────────────────┴──────────────────────────────┐
  │  VPC: 10.0.0.0/16                                           │
  │                                                             │
  │  【Public Subnet】                                          │
  │   - Bastion Host / Web Server (Internet Ingress/Egress)      │
  │   - [NAT Gateway] (Outbound Egress Only) ───────┐           │
  │                                                 │           │
  │                                                 ▼           │
  │  【Private Subnet】                                         │
  │   - Backend Node / RDS Database (No Direct Internet Ingress)│
  │                                                             │
  └─────────────────────────────────────────────────────────────┘
```


[View Week 05 VPC Architecture Resource Map](./week-05/vpc-diagram.png)



## Week 06 — Elastic Load Balancing & Auto Scaling

### 1. ASG behind an ALB

* **Application**: PropertyLite Flask API running on port `8080` (deployed via Launch Template User Data).
* **Network & Compute**: 
  * An **Application Load Balancer (ALB)** spanning 2 Public Subnets.
  * An **Auto Scaling Group (ASG)** managing EC2 instances across 2 Private Subnets with outbound NAT Gateway connectivity.
* **Scaling Strategy**: Target Tracking Scaling Policy targeting Average CPU Utilization.


### 2. Prove self-healing and scaling

#### A. Self-Healing Verification
* **Action**: Manually terminated a running backend EC2 instance via the console.
* **Observation**: The ASG detected the unhealthy/missing instance and automatically launched a replacement instance within 1–2 minutes, maintaining the desired capacity.

#### B. Auto-Scaling & Scale-Out Event
* **Alarm Trigger Time (Timestamp 1)**: `2026-10-31 19:34:14 UTC` (CloudWatch AlarmHigh entered `ALARM` state triggered by the scaling policy).
* **Scale-Out Action Time (Timestamp 2)**: `2026-10-31 19:34:22 UTC` (ASG increased desired capacity from 2 to 4 and initiated provisioning of new instances).
* **Timestamp Gap**: **8 seconds** between the CloudWatch alarm trigger and the ASG launching the additional instances.


### 3. Debug

During ALB testing, I frequently encountered `504 Gateway Timeout` errors initially, and both registered targets in the Target Group displayed `Unhealthy`. After investigation, I found that this corresponded to the scenario in Debug Lab 6.3: the security group of the backend instance was not correctly allowing traffic from ALB. I went to the security group used by the EC2 instance (`launch-wizard-1`), clicked **Edit inbound rules** in **Inbound rules**, manually added a rule with **Custom TCP Port 8080** and Source set to **Anywhere-IPv4 (0.0.0.0/0)**, and saved it. After waiting about 2 minutes for the health check to re-polle, the instance status in the Target Group successfully turned green to `Healthy`, and API calls returned to normal.


### 4. Clean up

* ASG-> property-api-asg
* ALB-> property-alb
* Target Group-> property-api-tg
* Launch Template-> property-api-template
* NAT Gateway
