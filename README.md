# Secure AWS VPC Infrastructure Deployment - Capaciti Project

**Project Title:** Secure AWS VPC Infrastructure Deployment

**Objective**  
Design and implement a secure, tiered Virtual Private Cloud (VPC) that allows resources in the private subnet to access the internet for updates (via NAT Gateway) while remaining inaccessible to direct inbound traffic from the public internet.

**Technical Requirements**

- **VPC Configuration**:
  - Created a VPC with CIDR block: **10.0.0.0/16**
  - Named: `vpc-capaciti-project`

- **Subnet Design**:
  - Public Subnet: `capaciti-project-public-subnet` – CIDR **10.0.25.0/24**
  - Private Subnet: `capaciti-project-private-subnet` – CIDR **10.0.50.0/24**
  - Both subnets are in the same Availability Zone

- **Connectivity & Gateways**:
  - Internet Gateway: `capaciti-project-igw` – Attached to VPC for public access
  - NAT Gateway: Deployed in the public subnet with Elastic IP allocation

- **Routing & Security**:
  - **Public Route Table** (`Public RT`): 0.0.0.0/0 → Internet Gateway
  - **Private Route Table** (`Private RT`): 0.0.0.0/0 → NAT Gateway
  - Network ACL (NACL): `capaciti-project-nacl` applied to both subnets
    - Inbound: Allow TCP 1024–65535 (ephemeral ports) from 0.0.0.0/0 for return traffic + Deny all else
    - Outbound: Allow all traffic to 0.0.0.0/0

This setup ensures:
- Private subnet resources have outbound internet access for updates.
- No direct inbound traffic from the public internet reaches the private subnet.

## Architecture Diagram

The following diagram illustrates the tiered VPC architecture with NAT Gateway in the public subnet serving outbound traffic for the private subnet EC2 instance via dedicated route tables.

Diagram matching the implemented setup:
<img width="931" height="852" alt="project-vpc drawio" src="https://github.com/user-attachments/assets/b82370b9-9726-440a-a627-cf014d43efd9" />




**Custom Diagram Notes**:
- VPC: 10.0.0.0/16
- Public Subnet (10.0.25.0/24): Contains NAT Gateway + route to IGW
- Private Subnet (10.0.50.0/24): Contains test EC2 instance + route to NAT
- Outbound flow: Private → Private RT → NAT → IGW → Internet (outbound only)

## Infrastructure Proof (Screenshots)

1. **VPC CIDR Block**  
   <img width="1586" height="548" alt="vpc-CIDR Block" src="https://github.com/user-attachments/assets/e41392d4-270d-4a74-a76a-0f84340db0b5" />


2. **Route Table Associations**  
 <img width="1597" height="788" alt="public-rt-assossiation" src="https://github.com/user-attachments/assets/d494de2f-2b77-4d51-8bd1-9f3738d803be" />

  <img width="1600" height="746" alt="private-rt-association" src="https://github.com/user-attachments/assets/f5b277ce-da9a-4c48-815c-b5d8f963f6e5" />


3. **NAT Gateway & Internet Gateway Status**  
  <img width="1595" height="770" alt="NAT-Status" src="https://github.com/user-attachments/assets/6b2cf8ae-71a8-4789-90ce-941ecd67095d" />

  <img width="1595" height="754" alt="nat-gateway" src="https://github.com/user-attachments/assets/899864f2-b728-497a-97fb-15cd415cde95" />


## Connectivity Test

Verified the secure design with test EC2 instances:

**Private Subnet Test**:
- Instance launched in `capaciti-project-private-subnet` (no public IP)
- Outbound access via NAT:  
 
  curl ifconfig.me    # Returns NAT Gateway's Elastic IP
  ping google.com     # Success

  ## Connectivity Test – Public Subnet Focus
 
 **public subnet** (direct internet access via IGW):

- Instance: `test-public-ec2` (or similar)
- Subnet: `capaciti-project-public-subnet` (10.0.25.0/24)
- Auto-assign Public IP: Enabled (public IPv4 address assigned)
- Security Group: SSH (port 22) allowed from my IP

**Test Results**:
- Direct SSH connection successful
 
