## DigiWorks Studio – Unified AWS Infrastructure Stack

This CloudFormation template builds a complete, production-ready AWS environment for DigiWorks Studio.
It includes networking, compute, database, storage, backup, and monitoring layers — all automated through a single stack.

## 1. Parameters

Inputs used to customize the stack:

ProjectName – Prefix for resource naming.

VpcCidr – Main VPC IP range.

PublicSubnet1Cidr – CIDR block for the public subnet.

PrivateSubnet1Cidr / PrivateSubnet2Cidr – CIDR ranges for private subnets across multiple AZs.

## 2. VPC
DigiWorksVPC

Creates the isolated AWS network where all systems run.

## 3. Internet Gateway
InternetGateway + AttachGateway

Provides internet access for public subnets.

Must be attached to the VPC.

## 4. Subnets

PublicSubnet1 → For load balancers, NAT gateway, or bastion hosts.

PrivateSubnet1 & PrivateSubnet2 → For EC2 application servers and databases (no direct internet exposure).

## 5. NAT Gateway & Elastic IP

NatEIP → Static public IP for the NAT.

NatGateway → Allows private instances to access the internet (security updates, package installs).

## 6. Route Tables
PublicRouteTable

Sends outbound traffic to the Internet Gateway.

PrivateRouteTable

Sends outbound traffic to the NAT Gateway.

Subnet Route Associations

Connects each subnet to the correct route table.

## 7. Security Groups
WebServerSG

Allows HTTP (80) and HTTPS (443) for public access.

RDSSecurityGroup

Only allows database traffic (3306) from the WebServerSG.

BastionSG

Allows SSH access from admin IP only.

## 8. IAM Roles
WebServerInstanceRole & InstanceProfile

Permissions for EC2/Beanstalk instances to:

Access S3

Access ECR

Send logs to CloudWatch

Work with Elastic Beanstalk systems

## 9. Elastic Beanstalk Application Layer
DigiWorksApp

Defines the application container for Beanstalk.

DigiWorksEnvironment

Configures the environment with:

Load balancer in public subnet

EC2 application instances in private subnets

Auto-scaling settings

CPU-based scaling rules

ALB listener on port 80

Uses Amazon Linux 2023 with Node.js 22 platform.

## 10. RDS Subnet Group
RDSSubnetGroup

Defines which private subnets the database can use.

## 11. MySQL RDS Database
DigiWorksRDS

Creates a secure MySQL 8.0 DB with:

Private-only access

db.t3.micro instance

20GB storage

Daily snapshots

No public connection

## 12. S3 Bucket
DigiWorksS3Bucket

Stores application assets with:

Private access

Lifecycle rule → Move to Glacier after 30 days

Versioning disabled

## 13. AWS Backup
BackupVault

Where backups are stored.

BackupPlan

Daily backup schedule (3AM).

BackupRole

IAM role for backup operations.

BackupSelection

Selects the RDS instance for daily backup.

## 14. CloudWatch Alarm
DigiWorksCPUAlarm

Triggers when EC2 CPU usage is above 70%.

## Summary

This stack deploys a fully automated AWS infrastructure, including:

Networking & Security

VPC, subnets, NAT, routing

Secure security groups

Compute

Elastic Beanstalk environment

Auto-scaling EC2 instances

Load balancer

Database

Private MySQL RDS instance

Daily automated backups

Storage

S3 bucket + Glacier lifecycle rules

Monitoring

CloudWatch CPU alarm

Backup

Backup vault + plan + role
