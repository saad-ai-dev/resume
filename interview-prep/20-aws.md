# 20. AWS CLOUD SERVICES

## 20.1 Services You've Used

### EC2 (Elastic Compute Cloud)
```
Virtual servers in the cloud.

Instance Types:
- t3.micro  → 2 vCPU, 1 GB RAM (dev/staging)
- t3.medium → 2 vCPU, 4 GB RAM (small production)
- c5.xlarge → 4 vCPU, 8 GB RAM (compute-intensive)

Key Concepts:
- AMI (Amazon Machine Image) → server template
- Security Groups → virtual firewall (inbound/outbound rules)
- Key Pairs → SSH access
- Elastic IP → static public IP
```

### S3 (Simple Storage Service)
```
Object storage for files, backups, static assets.

Key Concepts:
- Bucket → container for objects
- Object → file + metadata
- Pre-signed URLs → temporary secure file access
- Storage classes → Standard, IA (Infrequent Access), Glacier (archive)
- Lifecycle policies → auto-move old data to cheaper storage

Use Cases (Your Experience):
- Data backup and archival
- Disaster recovery
- Static asset hosting
- User file uploads
```

### Lambda (Serverless Functions)
```
Run code without managing servers. Pay per execution.

Triggers: API Gateway, S3, SQS, CloudWatch Events
Limits: 15 min timeout, 10 GB memory, 50 MB deploy package

Your Usage:
- Serverless sentiment analysis
- Review data processing
```

### VPC (Virtual Private Cloud)
```
Isolated network environment in AWS.

Components:
- Subnets → public (internet-facing) and private (internal only)
- Internet Gateway → connects VPC to internet
- NAT Gateway → lets private subnets access internet (outbound only)
- Route Tables → rules for traffic routing
- Network ACLs → stateless firewall at subnet level
- Security Groups → stateful firewall at instance level

Typical Architecture:
┌────────────────── VPC (10.0.0.0/16) ──────────────────┐
│  ┌─── Public Subnet ───┐  ┌─── Private Subnet ───┐    │
│  │  Load Balancer       │  │  App Servers (EC2)   │    │
│  │  NAT Gateway         │  │  Database (RDS)      │    │
│  │  Bastion Host        │  │  Redis (ElastiCache) │    │
│  └──────────────────────┘  └──────────────────────┘    │
│           ↕ Internet Gateway                            │
└─────────────────────────────────────────────────────────┘
```

### ELB (Elastic Load Balancer)
```
Distributes traffic across multiple targets.

Types:
- ALB (Application LB) → HTTP/HTTPS, path-based routing
- NLB (Network LB) → TCP/UDP, ultra-low latency
- CLB (Classic LB) → Legacy

Health Checks:
- Sends requests to /health endpoint
- Removes unhealthy instances from rotation
- Re-adds when healthy again
```

### CodePipeline + CodeDeploy (CI/CD)
```
Pipeline: Source → Build → Test → Deploy

Source:  GitHub/CodeCommit (trigger on push)
Build:   CodeBuild (npm install, npm test, npm build)
Deploy:  CodeDeploy (rolling, blue/green, all-at-once)

Deployment Strategies:
- All-at-once: Fast but risky (downtime)
- Rolling: Deploy to subset at a time
- Blue/Green: Two environments, switch traffic
- Canary: Deploy to 10% → monitor → deploy to 100%
```

## 20.2 Common AWS Interview Questions

**Q: How would you design a highly available architecture?**
- Multi-AZ deployment (instances in multiple Availability Zones)
- Auto Scaling Group (scale based on CPU/memory/requests)
- Load Balancer distributing traffic
- RDS Multi-AZ for database failover
- S3 for static assets (11 nines durability)

**Q: How do you secure an AWS infrastructure?**
- VPC with private subnets for app/DB
- Security Groups (least privilege)
- IAM roles (not access keys) for service-to-service
- Encryption at rest (KMS) and in transit (TLS)
- CloudTrail for audit logging

