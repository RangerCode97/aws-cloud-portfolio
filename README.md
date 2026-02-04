# aws-cloud-portfolio

This repository contains a production-style AWS environment deployed using Infrastructure as Code (IaC).  
The goal of this project is to demonstrate how to design, deploy, and operate **secure private workloads**
in AWS using real-world architectural and security patterns.

This environment is intentionally designed to mirror how backend systems are deployed in production:
private by default, tightly controlled administrative access, auditable changes, and clean teardown.

---

## Problem Statement

Design an AWS environment that:

- Keeps workloads fully private and inaccessible from the public internet
- Enforces least-privilege access at the network and IAM layers
- Allows controlled administrative access without exposing management ports
- Supports outbound internet access for patching and updates
- Is fully reproducible and removable using Infrastructure as Code

---

## Solution Overview

This project deploys a **multi-AZ, two-tier AWS architecture** inside a dedicated VPC using CloudFormation.

The architecture consists of:

- A single VPC (10.0.0.0/16)
- Two Availability Zones within the same AWS region
- Public subnets used only for controlled access and egress
- Private subnets hosting backend workloads
- No direct inbound access to private resources
- Centralized logging, monitoring, and encryption

---

## Architecture

### Network Design

The environment includes:

- Two public subnets (one per AZ)
- Two private subnets (one per AZ)
- An Internet Gateway attached to the VPC
- A NAT Gateway to allow outbound-only internet access from private subnets
- Separate route tables for public and private subnets

### Compute

- Bastion access is provided without exposing inbound management ports
- Private EC2 instances are deployed in private subnets
- Private instances are not assigned public IP addresses
- Workloads are designed to support future Auto Scaling integration

---

## Security Model

Security is enforced in multiple layers:

### Network Controls
- Private subnets have no direct route to the internet
- Outbound traffic from private workloads flows through a NAT Gateway
- Security groups restrict traffic to only required sources
- No inbound management ports exposed to the internet

### Identity and Access Management
- EC2 instances use IAM roles instead of static credentials
- IAM permissions are scoped to the minimum required access
- Systems Manager (SSM) is used for administrative access where applicable

### Logging and Monitoring
- CloudTrail is enabled to audit API and configuration changes
- VPC Flow Logs capture network traffic metadata
- CloudWatch alarms monitor instance health and CPU utilization

### Data Protection
- EBS volumes are encrypted at rest
- Encrypted storage allows safe recovery and redeployment of instances

---

## Deployment

All infrastructure is deployed using CloudFormation templates located in the `infra/` directory.

To deploy:

```bash
aws cloudformation deploy \
  --template-file main.yaml \
  --stack-name secure-multi-az-environment \
  --capabilities CAPABILITY_NAMED_IAM


