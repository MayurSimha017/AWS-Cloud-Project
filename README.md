# AWS CloudOps Project – Infrastructure Setup & Deployment

A hands-on AWS CloudOps project focused on designing and provisioning a custom VPC, configuring public and private subnets, implementing security-group-based access, and setting up Amazon EC2 and Amazon RDS (MySQL).

The project demonstrates practical AWS networking, compute, database, and infrastructure-management concepts using the AWS Management Console.

> **Project status:** Core VPC, networking, security groups, EC2 and RDS infrastructure provisioned. Application deployment, load balancer deployment and final cleanup are separate pending tasks.

## 1. Project Objectives

- Design a custom AWS VPC with a structured subnet layout.
- Configure public and private route tables.
- Attach an Internet Gateway for public-subnet connectivity.
- Implement network access controls using security groups.
- Provision an EC2 instance running Amazon Linux 2023.
- Provision a private Amazon RDS MySQL database.
- Understand private application-to-database communication.
- Document the infrastructure and validate AWS resource configuration.

## 2. AWS Services & Technologies

| Service | Purpose |
|---|---|
| Amazon VPC | Isolated virtual network |
| Amazon EC2 | Compute instance |
| Amazon RDS | Managed MySQL database |
| Internet Gateway | Internet connectivity for public subnets |
| Route Tables | Traffic routing |
| Security Groups | Instance and database access control |
| EBS | EC2 persistent block storage |
| EC2 Instance Connect | Browser-based instance access |

**Region:** Asia Pacific (Mumbai) – `ap-south-1`

## 3. Network Architecture

### VPC Configuration

| Property | Configuration |
|---|---|
| VPC Name | `cloudops-lab-vpc` |
| IPv4 CIDR | `10.0.0.0/16` |
| Region | `ap-south-1` |
| Internet Gateway | `cloudops-lab-igw` |

### Subnet Design

The VPC contains six subnets across two Availability Zones.

| Subnet | CIDR | Availability Zone | Purpose |
|---|---|---|---|
| public-subnet-a | `10.0.1.0/24` | ap-south-1a | EC2 instance |
| public-subnet-b | `10.0.2.0/24` | ap-south-1c | Reserved |
| app-private-subnet-a | `10.0.11.0/24` | ap-south-1a | Reserved |
| app-private-subnet-b | `10.0.12.0/24` | ap-south-1c | Reserved |
| db-private-subnet-a | `10.0.21.0/24` | ap-south-1a | RDS subnet group |
| db-private-subnet-b | `10.0.22.0/24` | ap-south-1c | RDS subnet group |

The EC2 instance is deployed in `public-subnet-a`. The application-private subnets are reserved for a possible future private application tier.

### Route Table Configuration

**Public Route Table – `public-rt`**
- `10.0.0.0/16` → Local
- `0.0.0.0/0` → Internet Gateway
- Associated with both public subnets.

**Private Application Route Table – `private-app-rt`**
- `10.0.0.0/16` → Local
- Associated with both application-private subnets.

**Private Database Route Table – `private-db-rt`**
- `10.0.0.0/16` → Local
- Associated with both database-private subnets.

No NAT Gateway was provisioned. The private route tables therefore do not provide direct outbound internet access.

## 4. Security Group Configuration

Security groups were configured to control access between infrastructure components.

| Security Group | Inbound Rule | Source |
|---|---|---|
| `app-sg` | TCP 5000 | `alb-sg` |
| `app-sg` | TCP 22 (SSH) | AWS EC2 Instance Connect managed prefix list |
| `db-sg` | TCP 3306 (MySQL) | `app-sg` |
| `alb-sg` | TCP 80 (HTTP) | `0.0.0.0/0` |
| `alb-sg` | TCP 443 (HTTPS) | `0.0.0.0/0` |

The security groups establish the intended access-control relationship between the application and database tiers.

**Note:** The ALB security group has been created, but an Application Load Balancer has not been provisioned. Consequently, the TCP 5000 rule is preparatory and does not represent active ALB traffic.

## 5. Amazon EC2 – Compute Infrastructure

An EC2 instance was provisioned with the following configuration:

| Property | Configuration |
|---|---|
| Instance Name | `cloudops-flask-server` |
| Instance Type | `t3.micro` |
| Operating System | Amazon Linux 2023 |
| VPC | `cloudops-lab-vpc` |
| Subnet | `public-subnet-a` |
| Security Group | `app-sg` |
| Root Storage | 8 GiB encrypted EBS volume |

EC2 Instance Connect was configured using the AWS-managed prefix list for SSH access.

**Validation performed:**
- Successfully accessed the instance using EC2 Instance Connect.
- Reached the Amazon Linux 2023 terminal.
- Verified that the instance passed its status checks.

## 6. Amazon RDS – MySQL Database

A managed MySQL database was provisioned using Amazon RDS.

| Property | Configuration |
|---|---|
| DB Identifier | `cloudops-mysql-db` |
| Engine | MySQL |
| Instance Class | `db.t4g.micro` |
| Storage | 20 GiB |
| Availability | Single-AZ |
| Active AZ | `ap-south-1c` |
| Public Access | Disabled |
| DB Subnet Group | `cloudops-db-subnet-group` |
| Security Group | `db-sg` |
| Port | `3306` |

The DB subnet group contains both database-private subnets:
- `db-private-subnet-a` – `10.0.21.0/24`
- `db-private-subnet-b` – `10.0.22.0/24`

The database itself is deployed in `ap-south-1c`. Having two subnets in the DB subnet group does **not** mean that a standby database is running in the second Availability Zone.

The database is configured for private access and is not directly exposed to the internet.

## 7. Intended Application-to-Database Connectivity

The intended communication path is:

1. The application tier runs on EC2.
2. The application connects to Amazon RDS using its private endpoint.
3. The database security group allows MySQL traffic on TCP port `3306` from `app-sg`.
4. The database is not publicly accessible.

The application and database connectivity should be validated from the EC2 environment before describing the application as deployed and operational.

## 8. Troubleshooting & Learning

During infrastructure setup, the project involved working through:

- VPC and subnet configuration.
- Public and private route table associations.
- Internet Gateway attachment and default routing.
- Security-group inbound rules and source restrictions.
- EC2 subnet and security-group selection.
- EC2 Instance Connect access configuration.
- RDS subnet-group and private-access configuration.
- Understanding the difference between subnet-group AZ coverage and an actual Multi-AZ database deployment.

These exercises helped reinforce practical AWS networking and infrastructure troubleshooting.

## 9. Cost Awareness & Resource Cleanup

This project was built as a learning environment with cost awareness in mind.

Important considerations:
- Avoid provisioning unnecessary NAT Gateways or load balancers.
- Monitor EC2 and RDS running time.
- Review EBS volumes, snapshots and other retained resources.
- Check for unused or duplicate EC2 instances.
- Verify that resources are deleted when the lab is no longer required.

**Cleanup status:** Update this section after verifying and completing resource cleanup. Terminating EC2 alone does not automatically remove every associated AWS resource.

## 10. Screenshots & Evidence

The following screenshots provide evidence of the AWS configuration. All screenshots are retained in the repository.

### VPC & Networking

![VPC Overview](screenshots/01-vpc/vpc-overview.png)

![Subnet Overview](screenshots/02-networking/subnet-overview.png)

![Public Route Table](screenshots/02-networking/public-rt.png)

![Private Database Route](screenshots/02-networking/private-db-route.png)

### Security Groups

![Security Group Overview](screenshots/03-security/securitygroup-overview.png)

![Database Security Group](screenshots/03-security/database-security-group.png)

### Amazon RDS

![RDS Setup Confirmation](screenshots/04-database/rds-setup-confirmed.png)

### Amazon EC2

![EC2 Instance Details](screenshots/05-compute/ec2-details.png)

![EC2 Instance Connect](screenshots/05-compute/ec2-instance-connect.png)

## 11. Project Structure

```text
AWS CloudOps Project/
│
├── README.md
│
├── architecture/
│   └── aws-cloudops-architecture.png
│
└── screenshots/
    ├── 01-vpc/
    ├── 02-networking/
    ├── 03-security/
    ├── 04-database/
    ├── 05-compute/
    └── 06-application/
```

## 12. Key Takeaways

Through this project, I gained practical experience with:

- Designing a custom VPC and subnet architecture.
- Configuring public and private routing.
- Understanding Internet Gateway functionality.
- Implementing security-group-based access control.
- Provisioning and accessing Linux-based EC2 instances.
- Configuring private Amazon RDS MySQL infrastructure.
- Understanding Availability Zones and Single-AZ database deployments.
- Troubleshooting common AWS networking and access issues.
- Managing cloud resources with cost awareness.

---

**Project Type:** Hands-on AWS CloudOps / Infrastructure Lab  
**Cloud Provider:** Amazon Web Services (AWS)  
**Region:** ap-south-1 (Mumbai)  
**Status:** Core infrastructure provisioned; further application deployment and cleanup work pending.