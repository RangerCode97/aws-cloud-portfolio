# aws-cloud-portfolio

This repository contains a production-style AWS environment deployed using Infrastructure as Code (IaC).

The goal of this project is to demonstrate how to design, deploy, and operate **secure private workloads**
in AWS using real-world architectural and security patterns.

The environment mirrors production backend design principles:
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

Core components:

- VPC (`10.0.0.0/16`)
- Two Availability Zones
- Two public subnets (one per AZ)
- Two private subnets (one per AZ)
- Internet Gateway
- NAT Gateway (single AZ)
- Bastion host (one per AZ)
- Private Auto Scaling Group spanning both private subnets
- Dedicated IAM role (SSM + CloudWatch only)

---

## Architecture

### Network Design

- Public subnets are internet-facing and associated with a public route table.
- Private subnets are isolated and route outbound traffic through a NAT Gateway.
- Separate route tables enforce public/private segmentation.
- No private workload has a public IP address.

---

## Traffic Flow

### Inbound Internet Traffic (Public Tier)

Internet  
→ Internet Gateway  
→ Public Route Table (`0.0.0.0/0 → IGW`)  
→ Public Subnets  
→ Bastion Hosts  

Management access is performed via AWS Systems Manager (SSM), not exposed SSH.

---

### Outbound Internet Traffic (Private Tier)

Private ASG Instances  
→ Private Route Table (`0.0.0.0/0 → NAT Gateway`)  
→ NAT Gateway  
→ Internet Gateway  
→ Internet  

Private instances:

- Have no public IPs
- Cannot receive inbound internet traffic
- Can initiate outbound connections for updates and SSM communication

**Note:**  
Egress currently depends on a single NAT Gateway.  
Full production-grade high availability would require one NAT Gateway per AZ.

---

## Compute Layer

- Bastion hosts deployed in each public subnet
- Private workloads deployed via Auto Scaling Group across both private subnets
- Desired capacity maintains multi-AZ redundancy
- If one AZ fails, workload capacity remains available in the remaining AZ

---

## IAM & Least Privilege Model

All EC2 instances use a dedicated IAM role with only:

- `AmazonSSMManagedInstanceCore`
- CloudWatch permissions for logging/metrics

No static credentials are stored on instances.

This ensures:

- Secure Systems Manager access
- Centralized logging
- Minimal permission surface area
- Reduced blast radius

---

## Security Model

### Network Controls

- Private subnets have no direct route to the Internet Gateway
- Outbound-only internet via NAT
- No inbound SSH exposure
- Security groups restrict access to only required sources

### Identity Controls

- IAM roles attached to instances
- No embedded access keys
- Least privilege enforced at role level

### Logging & Observability

- CloudTrail enabled for API auditing
- VPC Flow Logs capture network metadata
- CloudWatch alarms monitor instance health and CPU utilization

### Data Protection

- EBS volumes encrypted at rest
- Encrypted storage enables safe redeployment

---

## Security Principles Applied

- Defense in depth
- Least privilege IAM
- Public/private network segmentation
- No direct management port exposure
- Centralized control-plane access via SSM
- Encrypted storage and audit logging

---

## Deployment

All infrastructure is deployed using CloudFormation templates located in the `infra/` directory.

To deploy:

```bash
aws cloudformation deploy \
  --template-file main.yaml \
  --stack-name secure-multi-az-environment \
  --capabilities CAPABILITY_NAMED_IAM


