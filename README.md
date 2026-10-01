# Configuring High Availability with an Application Load Balancer and Auto Scaling

A hands-on AWS lab where I took a single-instance web app and turned it into a horizontally scaling, load-balanced application across multiple Availability Zones — then proved it worked by stress-testing it until it actually scaled up in real time.

## Scenario
An employee directory application had launched successfully on a single EC2 instance, but a single instance means a single point of failure and a hard ceiling on capacity. My job was to get it running behind a load balancer with auto scaling, so it could handle real demand and survive an instance failure.

## What I did

### 1. Reviewed the existing setup
Confirmed the app was running on a single EC2 instance in one public subnet, and noted its current Availability Zone as a baseline to compare against later.

### 2. Created an Application Load Balancer and target group
Built an ALB spanning both Availability Zones in the VPC, attached to a dedicated `LoadBalancerSG` security group, and created a target group with tuned health check thresholds (2 healthy / 5 unhealthy checks, 30s interval) to control how quickly it reacts to instance health changes.

### 3. Built a launch template
Created a reusable launch template defining the AMI, instance type (`t3.micro`), security group, and IAM instance profile (`EmployeeDirectoryAppRole`) for future instances — plus a user data script that installs Node.js, a stress-testing tool, and the app itself, and starts it automatically on boot.

### 4. Created an Auto Scaling group
Built an Auto Scaling group using that launch template, spanning both public subnets, attached to the load balancer's target group, with a desired/minimum capacity of 2 and a maximum of 4. Configured a target-tracking scaling policy (30% target metric, 300s instance warmup) and an SNS notification so scaling events trigger an email alert.

### 5. Removed the original single point of failure
Once the Auto Scaling group's instances were healthy in the target group, terminated the original manually-launched EC2 instance — confirming the app stayed fully available through the load balancer with zero dependency on that original instance.

### 6. Load tested to trigger real scaling
Used a built-in stress tool to load the CPU on one of the running instances for ~10 minutes, then watched the target group provision additional instances automatically in response — and got the SNS email notification confirming the scaling event.

## Key takeaways
- High availability isn't just about handling more traffic — removing the original single instance and confirming the app survived unscathed is the real proof there's no single point of failure left
- A launch template plus user data script means every new instance bootstraps itself identically, which is what makes automated scaling actually safe to trust
- Watching a stress test trigger a real scaling event, end to end, is a lot more convincing than reading about target-tracking policies in the abstract

## Tools
AWS Application Load Balancer, EC2 Auto Scaling, Launch Templates, Amazon SNS, IAM

---
*Completed as an AWS Cloud Technical Essentials hands-on lab.*
