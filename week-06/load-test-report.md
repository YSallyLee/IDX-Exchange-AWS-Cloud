# Week 06 — Load Test & Auto Scaling Report


## 1. Architecture Overview
* **Application**: PropertyLite running on port 8080.
* **Network**: Application Load Balancer in 2 Public Subnets forwarding traffic to an Auto Scaling Group in 2 Private Subnets.
* **Scaling Policy**: Target Tracking Scaling Policy targeting Average CPU Utilization at 50% (tested via synthetic load threshold).

## 2. Debug: 504 Gateway Timeout

Issue faced: A 504 Gateway Timeout occurred when sending a request, and both machines in the Target Group displayed a red "Unhealthy" status.

Troubleshooting: Checking the security group (launch-wizard-1) of the backend EC2 revealed that Port 8080 traffic from ALB was not being allowed; manually adding a Custom TCP Port 8080 rule restored the health status to green "Healthy".

## 3. Test Execution & Timestamp Gap
* **CloudWatch Alarm Triggered**: `2026-10-31 19:34:14 UTC` (`AlarmHigh` -> `ALARM`).
* **ASG Scale-Out Action**: `2026-10-31 19:34:22 UTC` (ASG received the instruction to expand the capacity from 2 to 4 units and start the machines.).
* **Timestamp Gap**: **8 seconds** between the CloudWatch alarm trigger and the ASG launching replacement/additional instances.

## 3. Key Takeaways
1. **Self-Healing Verification**: Terminating unhealthy instances causes the ASG to automatically replace them within 1–2 minutes, maintaining desired availability.
2. **Elastic Scaling**: When load breaches the target threshold, the system scales out horizontally from 2 to 4 instances automatically without manual intervention.
