# Week 3 Compute: EC2 Deployment, EBS Snapshots & Security Triage


This week's goal was to migrate the PropertyLite Flask API, which we ran locally in Week 0, to the AWS EC2 cloud environment. In addition to verifying cloud hosts and automating boot processes, I also conducted a snapshot exercise on EBS disks and actually encountered a connection timeout failure when switching work networks. I also performed a complete Security Group dynamic IP drift error removal test.

# Lab 3.1: Deploying PropertyLite to EC2 (with User Data Boot Script)

When creating the host, I launched an Amazon Linux 2023 instance named `t3.micro` (named `property-api-01`). Because PropertyLite is designed to listen on port `8080` during Week 0, the Security Group rules must do two things:

1. **SSH (Port 22)**: Set to `My IP`, allowing only my current local machine to connect, preventing the management entry point from being exposed to the public network.

2. **Custom TCP (Port 8080)**: Open to `0.0.0.0/0` (Anywhere), allowing external calls to the API via HTTP.

To ensure the host machine runs automatically upon boot, I didn't manually install the environment. Instead, I combined Week 0's `app.py`, `rets_property_sample.csv`, and installation commands into a single Bash script and directly inserted it into **User Data**.

When EC2 boots for the first time, the system automatically installs Python 3, pip, and Flask 3.0.3 as root, writes the files to `/opt/property-api`, and finally starts Flask in the background using `nohup`.

After the host machine boots up, I directly tested its public IP address (`54.81.94.69`) from my local terminal:

* Performing a health check using `curl http://54.81.94.69:8080/health` successfully returned `{"status":"ok"}`.

* Querying the property `curl http://54.81.94.69:8080/properties/R100234` successfully returned the JSON data for the Sacramento property.

This means the same local application has been seamlessly deployed to a true cloud architecture! *(Screenshot saved in `week-03/curl-response.png`)*



## Lab 3.2: EBS Snapshot Practical Exercise

After the API was successfully launched, I went to the EC2 Volumes page and found the 8 GiB root disk (`vol-09c9ceb0236ee9805`) currently in use by `property-api-01`. I created a snapshot of it using Actions (Snapshot ID: `snap-060778c22ba3fda70`).



## Debug Lab 3.3: SSH Connection Timeout and Dynamic IP Drift Errors

While conducting this experiment, I switched to a different network environment and encountered a classic connection failure:

### 1. Fault Phenomenon
I tried connecting to the host from the terminal:

```bash
ssh -i ~/.ssh/training-key.pem ec2-user@54.81.94.69
```

The command had no response just froze , and after a while it finally show "Operation timed out".



### 2. Check Root Cause

Comparison with the EC2 Security Group (launch-wizard-1) revealed that the whitelist for SSH (Port 22) is still stuck at a static snapshot of the old location, 204.111.44.198/32. Because the network changed, the IP drifted, and the connection packets from the new IP were directly dropped by the firewall, causing Timeout.


### 3. Solutions and Verification


I Go to AWS Console → EC2 → Security Groups, and click on launch-wizard-1. 

Click Edit inbound rules, and reselect My IP (74.96.154.45/32) as the Source for SSH (Port 22) and save. 

Return to the local terminal and enter the command again to verify, the connection and API should immediately return to normal.