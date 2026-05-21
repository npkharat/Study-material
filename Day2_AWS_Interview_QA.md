# AWS


## Table of Contents
1. [Core AWS Concepts (Q1–Q20)](#1-core-aws-concepts)
2. [EC2 & Auto Scaling (Q21–Q35)](#2-ec2--auto-scaling)
3. [S3 & Storage (Q36–Q48)](#3-s3--storage)
4. [Networking — VPC, ELB, Route53 (Q49–Q65)](#4-networking--vpc-elb-route53)
5. [IAM & Security (Q66–Q78)](#5-iam--security)
6. [CI/CD & DevOps Services (Q79–Q92)](#6-cicd--devops-services)
7. [Advanced & Real-World Scenarios (Q93–Q110)](#7-advanced--real-world-scenarios)

---

## 1. Core AWS Concepts

**Q1. What is AWS?**
> AWS (Amazon Web Services) is a cloud platform by Amazon. It provides on-demand computing resources like servers, storage, databases, networking, and more — you pay only for what you use.

---

**Q2. What are the 3 cloud service models?**
> - **IaaS (Infrastructure as a Service):** You manage OS, apps. AWS manages hardware. Example: EC2
> - **PaaS (Platform as a Service):** You manage only your app/code. Example: Elastic Beanstalk
> - **SaaS (Software as a Service):** Everything managed for you. Example: Gmail, Salesforce

---

**Q3. What is a Region and Availability Zone in AWS?**
> - **Region:** A geographic location (e.g., `us-east-1` = North Virginia, `ap-south-1` = Mumbai)
> - **Availability Zone (AZ):** One or more data centers inside a Region. Each Region has 2–6 AZs.
> - **Why it matters:** Deploy across multiple AZs for high availability. If one AZ goes down, others keep running.

---

**Q4. What is the AWS Shared Responsibility Model?**
> - **AWS is responsible for:** Security OF the cloud (hardware, data centers, network infrastructure)
> - **You are responsible for:** Security IN the cloud (your OS patches, firewall rules, IAM, data encryption)
> - Simple: AWS secures the building, you secure what's inside.

---

**Q5. What is an AMI?**
> AMI (Amazon Machine Image) is a template used to launch EC2 instances. It includes:
> - Operating system (Ubuntu, Amazon Linux, Windows)
> - Pre-installed software
> - Storage configuration
> Think of it like a "snapshot" or "golden image" of a server.

---

**Q6. What is the difference between vertical and horizontal scaling?**
> - **Vertical Scaling (Scale Up):** Make the server bigger (more CPU, RAM). Has limits. Example: t2.micro → t2.xlarge
> - **Horizontal Scaling (Scale Out):** Add more servers. No limit. Example: 1 EC2 → 10 EC2 instances
> AWS prefers horizontal scaling — use Auto Scaling + Load Balancer.

---

**Q7. What is High Availability (HA) in AWS?**
> HA means your application keeps running even if something fails. Achieved by:
> - Deploying across multiple AZs
> - Using Load Balancers
> - Using Auto Scaling
> - Using managed services (RDS Multi-AZ, S3, etc.)

---

**Q8. What is the difference between Fault Tolerance and High Availability?**
> - **High Availability:** System stays available with minimal downtime (maybe brief interruption)
> - **Fault Tolerance:** System continues WITHOUT any interruption even during failures (zero downtime)
> Fault Tolerance is harder and more expensive. Example: Multi-AZ RDS = HA. Active-Active multi-region = Fault Tolerant.

---

**Q9. What is Elasticity in AWS?**
> Elasticity means automatically scaling resources UP when demand increases and DOWN when demand decreases — so you pay only for what you use. Example: Auto Scaling Group adding EC2s during traffic spikes.

---

**Q10. What are AWS Global Services vs Regional Services?**
> - **Global Services:** IAM, Route 53, CloudFront, WAF — not tied to a region
> - **Regional Services:** EC2, RDS, S3 (buckets are regional), Lambda — exist within a specific region

---

**Q11. What is AWS CLI and how do you use it?**
> AWS CLI is a command-line tool to interact with AWS services from your terminal.
```bash
# Configure credentials
aws configure
# Enter: Access Key, Secret Key, Region, Output format

# Common commands
aws s3 ls                          # list S3 buckets
aws ec2 describe-instances         # list EC2 instances
aws s3 cp file.txt s3://my-bucket/ # upload file to S3
aws ec2 start-instances --instance-ids i-1234567890
```

---

**Q12. What is AWS CloudFormation?**
> CloudFormation is AWS's Infrastructure as Code (IaC) tool. You write templates in JSON or YAML, and AWS creates all the resources automatically.
```yaml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0abcdef1234567890
      InstanceType: t2.micro
```

---

**Q13. What is Terraform vs CloudFormation?**
> | Feature | Terraform | CloudFormation |
> |---|---|---|
> | By | HashiCorp | AWS |
> | Language | HCL | JSON/YAML |
> | Multi-cloud | ✅ Yes | ❌ AWS only |
> | State management | Local/remote state file | AWS managed |
> | Community | Very large | AWS-focused |

---

**Q14. What is AWS CDK?**
> AWS CDK (Cloud Development Kit) lets you define AWS infrastructure using real programming languages like Python, TypeScript, or Java — instead of YAML/JSON. It generates CloudFormation under the hood.

---

**Q15. What is AWS Pricing model?**
> - **On-Demand:** Pay per hour/second. No commitment. Most expensive.
> - **Reserved Instances:** 1 or 3 year commitment. Up to 72% cheaper.
> - **Spot Instances:** Use spare AWS capacity. Up to 90% cheaper. Can be terminated anytime.
> - **Savings Plans:** Flexible pricing commitment (like Reserved but more flexible)

---

**Q16. What is a Spot Instance and when do you use it?**
> Spot Instances use unused AWS capacity at up to 90% discount. AWS can terminate them with 2-minute notice.
> Use for: Batch processing, CI/CD build agents, Big Data jobs, non-critical workloads.
> Don't use for: Databases, production web servers, anything that can't tolerate interruption.

---

**Q17. What is AWS Free Tier?**
> AWS offers free usage for 12 months after signup for services like:
> - EC2: 750 hours/month (t2.micro or t3.micro)
> - S3: 5 GB storage
> - RDS: 750 hours/month (db.t2.micro)
> - Lambda: 1 million requests/month (always free)

---

**Q18. What are AWS Tags and why are they important?**
> Tags are key-value labels you attach to AWS resources. Example: `Environment=Production`, `Team=DevOps`, `Project=MyApp`
> Why important:
> - Cost allocation (see which team spends what)
> - Resource management (find all production resources)
> - Automation (Auto Scaling, backup policies based on tags)

---

**Q19. What is AWS Organizations?**
> AWS Organizations lets you manage multiple AWS accounts centrally. You can:
> - Group accounts into Organizational Units (OUs)
> - Apply Service Control Policies (SCPs) across accounts
> - Consolidated billing (one bill for all accounts)
> - Common pattern: Separate accounts for Dev, Staging, Production

---

**Q20. What is the difference between AWS Console, CLI, and SDK?**
> - **Console:** Web browser UI — good for learning, manual tasks
> - **CLI:** Command line — good for scripts, automation
> - **SDK:** Code libraries (Python boto3, Java, Node.js) — good for applications
> In DevOps, you mostly use CLI and SDK for automation.

---

## 2. EC2 & Auto Scaling

**Q21. What is EC2?**
> EC2 (Elastic Compute Cloud) is AWS's virtual server service. You can launch Linux or Windows servers in minutes, choose CPU/RAM/storage, and pay by the hour or second.

---

**Q22. What are EC2 Instance Types?**
> | Family | Purpose | Examples |
> |---|---|---|
> | General Purpose | Balanced CPU/RAM | t3, m5 |
> | Compute Optimized | High CPU | c5, c6g |
> | Memory Optimized | High RAM | r5, x1 |
> | Storage Optimized | High disk I/O | i3, d2 |
> | GPU | ML, graphics | p3, g4 |

---

**Q23. What is a Security Group in EC2?**
> A Security Group is a virtual firewall for your EC2 instance. It controls:
> - **Inbound rules:** What traffic can come IN (e.g., allow port 80 from anywhere, port 22 only from your IP)
> - **Outbound rules:** What traffic can go OUT (default: allow all)
> Security Groups are **stateful** — if you allow inbound, the response automatically comes back.

---

**Q24. What is the difference between Security Group and NACL?**
> | Feature | Security Group | Network ACL (NACL) |
> |---|---|---|
> | Level | Instance level | Subnet level |
> | Stateful/Stateless | Stateful | Stateless |
> | Rules | Allow only | Allow and Deny |
> | Order | All rules checked | Rules checked in order |
> | Default | All outbound allowed | All traffic allowed |

---

**Q25. What is a Key Pair in EC2?**
> A Key Pair is used to SSH into an EC2 instance. AWS stores the public key on the instance, you keep the private key (`.pem` file).
```bash
ssh -i "my-key.pem" ec2-user@ec2-54-123-456-789.compute.amazonaws.com
```
> If you lose the private key, you cannot SSH in — there's no password recovery.

---

**Q26. What is User Data in EC2?**
> User Data is a script that runs automatically when an EC2 instance starts for the first time. Used for bootstrapping:
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from $(hostname)</h1>" > /var/www/html/index.html
```

---

**Q27. What is an Elastic IP?**
> A regular EC2 public IP changes every time you stop/start the instance. An Elastic IP is a **static public IP** that stays the same even after restart.
> - Free while attached to a running instance
> - Costs money if not attached (wasted resource)

---

**Q28. What is EBS?**
> EBS (Elastic Block Store) is a persistent storage volume attached to EC2 — like a hard drive for your server.
> - **gp3/gp2:** General purpose SSD (most common)
> - **io1/io2:** High performance SSD (databases)
> - **st1:** Throughput HDD (big data, logs)
> - **sc1:** Cold HDD (infrequent access, cheapest)
> EBS data persists even if the EC2 instance is stopped.

---

**Q29. What is an EBS Snapshot?**
> A Snapshot is a backup of an EBS volume stored in S3. You can:
> - Restore a volume from a snapshot
> - Create an AMI from a snapshot
> - Copy snapshots across regions for disaster recovery
> Snapshots are incremental — only changes since last snapshot are saved.

---

**Q30. What is EC2 Instance Store?**
> Instance Store is temporary storage physically attached to the host machine. It's faster than EBS but:
> - **Data is lost** when instance stops, terminates, or fails
> - Good for: temporary data, cache, scratch space
> - Not good for: anything you need to keep

---

**Q31. What is Auto Scaling Group (ASG)?**
> ASG automatically adds or removes EC2 instances based on demand or schedules.
> - **Min capacity:** Minimum instances always running
> - **Max capacity:** Maximum instances allowed
> - **Desired capacity:** Target number of instances
> It works with a Load Balancer to distribute traffic across instances.

---

**Q32. What are Auto Scaling policies?**
> - **Target Tracking:** Maintain a metric at a target value. Example: Keep CPU at 50%
> - **Step Scaling:** Scale by specific amounts based on alarm thresholds
> - **Simple Scaling:** Scale based on a single CloudWatch alarm
> - **Scheduled Scaling:** Scale at specific times (e.g., add servers every Monday 9AM)
> - **Predictive Scaling:** ML-based, predicts future load and scales in advance

---

**Q33. What is a Launch Template vs Launch Configuration?**
> - **Launch Configuration:** Old way. Can't be modified after creation.
> - **Launch Template:** New way. Can be versioned and modified. Supports Spot + On-Demand mix.
> Always use Launch Templates now.

---

**Q34. What is EC2 placement groups?**
> Controls how EC2 instances are physically placed:
> - **Cluster:** All instances on same rack — lowest latency, high throughput (HPC use cases)
> - **Spread:** Each instance on separate rack — max availability (up to 7 per AZ)
> - **Partition:** Groups of instances on separate racks — large distributed systems (Kafka, Cassandra)

---

**Q35. How do you connect to an EC2 instance without SSH key?**
> Use **AWS Systems Manager Session Manager**:
> - No need to open port 22
> - No SSH key needed
> - Access directly from AWS Console or CLI
> - Audit trail of all commands in CloudTrail
```bash
aws ssm start-session --target i-1234567890abcdef0
```

---

## 3. S3 & Storage

**Q36. What is S3?**
> S3 (Simple Storage Service) is AWS's object storage. You store files (objects) in buckets.
> - Unlimited storage
> - 99.999999999% (11 nines) durability
> - Objects can be up to 5TB each
> - Globally unique bucket names

---

**Q37. What are S3 Storage Classes?**
> | Class | Use Case | Cost |
> |---|---|---|
> | S3 Standard | Frequent access | High |
> | S3 Standard-IA | Infrequent access | Medium |
> | S3 One Zone-IA | Infrequent, one AZ | Lower |
> | S3 Glacier Instant | Archive, ms retrieval | Low |
> | S3 Glacier Flexible | Archive, minutes/hours retrieval | Very Low |
> | S3 Glacier Deep Archive | Long-term archive, 12hr retrieval | Cheapest |
> | S3 Intelligent-Tiering | Auto-moves between tiers | Auto |

---

**Q38. What is an S3 Bucket Policy?**
> A JSON policy that controls who can access a bucket and what actions they can do.
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
```
> This makes the bucket publicly readable (for static websites).

---

**Q39. What is S3 Versioning?**
> Versioning keeps multiple versions of an object. When you overwrite or delete a file, old versions are preserved.
> - Protects against accidental deletion
> - Once enabled, can only be suspended (not disabled)
> - Increases storage cost

---

**Q40. What is S3 Lifecycle Policy?**
> Automatically moves or deletes objects based on age. Example:
> - After 30 days → move to Standard-IA
> - After 90 days → move to Glacier
> - After 365 days → delete
> Great for cost optimization — automatically archive old logs.

---

**Q41. What is S3 Cross-Region Replication (CRR)?**
> Automatically replicates objects from one S3 bucket to another bucket in a different region.
> Use cases:
> - Disaster recovery (backup in another region)
> - Lower latency for users in another region
> - Compliance (data must be in specific region)

---

**Q42. What is S3 Transfer Acceleration?**
> Uses AWS CloudFront edge locations to speed up uploads to S3. When you upload, data goes to the nearest CloudFront edge, then travels AWS's fast private network to S3. Up to 300x faster for distant uploads.

---

**Q43. What is a Pre-signed URL in S3?**
> A pre-signed URL gives temporary access to a private S3 object without making it public.
```bash
aws s3 presign s3://my-bucket/myfile.pdf --expires-in 3600
# Returns a URL valid for 1 hour (3600 seconds)
```
> Use case: Let users download a private file for 1 hour without giving them AWS credentials.

---

**Q44. How do you host a static website on S3?**
> 1. Create S3 bucket (name = domain name)
> 2. Enable "Static Website Hosting" in bucket properties
> 3. Upload `index.html` and other files
> 4. Set bucket policy to allow public read
> 5. Use the S3 website endpoint URL
> 6. (Optional) Add CloudFront in front for HTTPS and caching

---

**Q45. What is EFS?**
> EFS (Elastic File System) is a managed NFS file system that can be mounted on multiple EC2 instances simultaneously. Unlike EBS (one instance at a time), EFS is shared storage.
> Use case: Web servers that need to share files, shared home directories.

---

**Q46. What is the difference between S3, EBS, and EFS?**
> | Feature | S3 | EBS | EFS |
> |---|---|---|---|
> | Type | Object storage | Block storage | File storage |
> | Access | HTTP API | One EC2 instance | Multiple EC2 instances |
> | Use case | Files, backups, static sites | OS disk, databases | Shared files |
> | Protocol | REST API | iSCSI | NFS |

---

**Q47. What is AWS Glacier?**
> Glacier is long-term archive storage — extremely cheap but slow to retrieve (minutes to hours). Use for data you must keep but rarely access (compliance records, old backups).

---

**Q48. What is S3 Object Lock?**
> Prevents objects from being deleted or overwritten for a specified time (WORM — Write Once Read Many). Required for compliance regulations like SEC 17a-4, HIPAA.

---

## 4. Networking — VPC, ELB, Route53

**Q49. What is a VPC?**
> VPC (Virtual Private Cloud) is your own private network inside AWS. It's isolated from other customers. You define:
> - IP address range (CIDR block, e.g., `10.0.0.0/16`)
> - Subnets, route tables, gateways
> - Security rules
> Every AWS account gets a default VPC in each region.

---

**Q50. What is the difference between Public and Private Subnet?**
> - **Public Subnet:** Has a route to the Internet Gateway. Resources here can communicate with internet. Used for: Load Balancers, Bastion hosts, NAT Gateways
> - **Private Subnet:** No direct internet route. Resources here can't be reached from internet. Used for: Application servers, Databases

---

**Q51. What is an Internet Gateway?**
> An Internet Gateway (IGW) is attached to a VPC to allow communication between resources in the VPC and the internet. Without an IGW, nothing in your VPC can reach the internet.

---

**Q52. What is a NAT Gateway?**
> NAT Gateway allows instances in **private subnets** to connect to the internet (for updates, downloads) WITHOUT being reachable from the internet.
> - Place NAT Gateway in public subnet
> - Private subnet route table points to NAT Gateway
> - NAT Gateway costs money per hour + per GB processed

---

**Q53. What is a Bastion Host?**
> A Bastion Host (Jump Server) is a special EC2 instance in the public subnet that you SSH into first, then from there SSH into private instances. It's the only entry point for SSH access.
```
Your PC → SSH → Bastion (public) → SSH → App Server (private)
```
> Modern alternative: Use AWS Systems Manager Session Manager (no bastion needed).

---

**Q54. What is VPC Peering?**
> VPC Peering connects two VPCs so they can communicate privately using private IPs. The traffic doesn't go over the internet.
> Limitations:
> - Not transitive (A↔B, B↔C doesn't mean A↔C)
> - Can peer across accounts and regions
> - IP ranges cannot overlap

---

**Q55. What is AWS Transit Gateway?**
> Transit Gateway is a hub that connects multiple VPCs and on-premises networks. Instead of creating mesh peering (N×N connections), everything connects to one Transit Gateway (star topology).
> Use when you have many VPCs that need to communicate.

---

**Q56. What is a Route Table in VPC?**
> A Route Table contains rules (routes) that determine where network traffic goes.
```
Destination       Target
10.0.0.0/16      local           ← traffic within VPC
0.0.0.0/0        igw-xxxx        ← internet traffic → Internet Gateway
```
> Each subnet is associated with exactly one route table.

---

**Q57. What are the types of Load Balancers in AWS?**
> - **ALB (Application Load Balancer):** Layer 7. Routes based on URL path, host headers, HTTP methods. Best for web apps, microservices.
> - **NLB (Network Load Balancer):** Layer 4. Ultra-low latency, handles millions of requests/sec. Best for TCP/UDP, gaming, IoT.
> - **GLB (Gateway Load Balancer):** For third-party network appliances (firewalls, IDS).
> - **CLB (Classic Load Balancer):** Old, legacy. Don't use for new projects.

---

**Q58. What is ALB path-based and host-based routing?**
```
Path-based routing:
myapp.com/api    → API servers target group
myapp.com/web    → Web servers target group
myapp.com/admin  → Admin servers target group

Host-based routing:
api.myapp.com    → API servers
www.myapp.com    → Web servers
admin.myapp.com  → Admin servers
```

---

**Q59. What is a Target Group in Load Balancer?**
> A Target Group is a group of EC2 instances, containers, or Lambda functions that the Load Balancer routes traffic to. It also runs health checks on targets and stops sending traffic to unhealthy ones.

---

**Q60. What is Route 53?**
> Route 53 is AWS's DNS service. It translates domain names to IP addresses. Features:
> - Domain registration
> - DNS routing
> - Health checks
> - Traffic routing policies

---

**Q61. What are Route 53 Routing Policies?**
> - **Simple:** One record, one IP. Basic DNS.
> - **Weighted:** Split traffic by percentage. Example: 80% → v1, 20% → v2 (for canary releases)
> - **Latency:** Route to region with lowest latency for the user
> - **Failover:** Active-Passive. If primary fails, route to secondary
> - **Geolocation:** Route based on user's country/continent
> - **Geoproximity:** Route based on geographic distance (with bias)
> - **Multi-Value:** Return multiple IPs (basic load balancing)

---

**Q62. What is CloudFront?**
> CloudFront is AWS's CDN (Content Delivery Network). It caches content at edge locations worldwide so users get it from a nearby location — faster response.
> Use for: Static websites, videos, API acceleration, DDoS protection.

---

**Q63. What is the difference between CloudFront and S3 Transfer Acceleration?**
> - **CloudFront:** Caches content at edge locations for users to download faster (good for read-heavy content)
> - **S3 Transfer Acceleration:** Speeds up UPLOADS to S3 (good for write-heavy uploads)

---

**Q64. What is VPC Flow Logs?**
> VPC Flow Logs captures information about IP traffic going to/from network interfaces in your VPC. Useful for:
> - Troubleshooting connectivity issues
> - Security analysis (detect unusual traffic)
> - Compliance auditing
> Logs are stored in CloudWatch Logs or S3.

---

**Q65. What is AWS Direct Connect?**
> Direct Connect is a dedicated private network connection from your on-premises data center to AWS — not over the public internet. Benefits:
> - More reliable (no internet congestion)
> - Lower latency
> - Higher bandwidth
> - More consistent performance
> - Good for hybrid cloud setups

---

## 5. IAM & Security

**Q66. What is IAM?**
> IAM (Identity and Access Management) controls who can access what in AWS. You can create:
> - **Users:** Individual people
> - **Groups:** Collection of users with same permissions
> - **Roles:** Permissions for AWS services or external identities
> - **Policies:** JSON documents defining permissions

---

**Q67. What is an IAM Policy?**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:PutObject"],
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
```
> A policy says: **Effect** (Allow/Deny), **Action** (what operation), **Resource** (on what).

---

**Q68. What is the Principle of Least Privilege in AWS?**
> Give users and services ONLY the permissions they need — nothing more. Example:
> - An app that only reads from S3 should only have `s3:GetObject` permission, not `s3:*`
> - A Lambda that writes to DynamoDB should only have `dynamodb:PutItem`, not `dynamodb:*`

---

**Q69. What is an IAM Role and when do you use it?**
> A Role is like a temporary identity that AWS services can assume. Unlike users, roles have no passwords or access keys.
> Use cases:
> - EC2 instance needs to access S3 → attach an IAM Role to EC2
> - Lambda needs to write to DynamoDB → attach an IAM Role to Lambda
> - Cross-account access → assume a role in another account
> **Never** store AWS credentials on EC2. Use IAM Roles instead!

---

**Q70. What is the difference between IAM User and IAM Role?**
> | Feature | IAM User | IAM Role |
> |---|---|---|
> | For | People | Services / Applications |
> | Credentials | Username/password + access keys | Temporary credentials |
> | Expiry | Permanent | Temporary (15 min to 12 hours) |
> | Use case | Developer login | EC2 accessing S3 |

---

**Q71. What is AWS STS?**
> STS (Security Token Service) issues temporary security credentials. When a role is assumed, STS provides:
> - Access Key ID
> - Secret Access Key
> - Session Token
> - Expiration time
> These expire automatically — much safer than permanent credentials.

---

**Q72. What is MFA in AWS?**
> MFA (Multi-Factor Authentication) adds a second layer of security. Even if someone knows your password, they need the MFA device (phone, hardware key) to log in.
> Best practices:
> - Enable MFA on root account always
> - Enable MFA for all admin users
> - Require MFA for sensitive API calls (delete operations)

---

**Q73. What is AWS KMS?**
> KMS (Key Management Service) manages encryption keys. You create Customer Master Keys (CMKs) and use them to encrypt/decrypt data in S3, EBS, RDS, etc.
> - AWS managed keys: Free, AWS manages rotation
> - Customer managed keys: $1/month per key, you control rotation and policies

---

**Q74. What is AWS Secrets Manager?**
> Secrets Manager securely stores secrets like database passwords, API keys. Features:
> - Automatic rotation of secrets (e.g., rotate DB password every 30 days)
> - Applications retrieve secrets at runtime (no hardcoding)
> - Audit access via CloudTrail
```python
import boto3
client = boto3.client('secretsmanager')
secret = client.get_secret_value(SecretId='prod/myapp/db-password')
```

---

**Q75. What is the difference between Secrets Manager and SSM Parameter Store?**
> | Feature | Secrets Manager | SSM Parameter Store |
> |---|---|---|
> | Cost | $0.40/secret/month | Free (Standard), Paid (Advanced) |
> | Auto rotation | ✅ Built-in | ❌ Custom Lambda needed |
> | Secret size | Up to 65KB | Up to 8KB |
> | Best for | Passwords, API keys | Config values, feature flags |

---

**Q76. What is AWS WAF?**
> WAF (Web Application Firewall) protects web apps from common attacks:
> - SQL Injection
> - Cross-Site Scripting (XSS)
> - DDoS (rate limiting)
> - IP blocking
> Works with CloudFront, ALB, API Gateway.

---

**Q77. What is AWS Shield?**
> Shield protects against DDoS attacks:
> - **Shield Standard:** Free. Protects against common DDoS attacks.
> - **Shield Advanced:** $3000/month. Advanced protection, 24/7 DDoS response team, cost protection.

---

**Q78. What is AWS CloudTrail?**
> CloudTrail records ALL API calls made in your AWS account — who did what, when, from where. Think of it as the audit log for AWS.
> - Every `aws s3 cp`, every IAM change, every EC2 launch
> - Stored in S3
> - Essential for security and compliance

---

## 6. CI/CD & DevOps Services

**Q79. What is AWS CodePipeline?**
> CodePipeline is AWS's fully managed CI/CD service. It automates the release process from source code to production. You define stages: Source → Build → Test → Deploy.

---

**Q80. What is AWS CodeBuild?**
> CodeBuild is a fully managed build service. It compiles code, runs tests, and produces deployable artifacts. Defined in `buildspec.yml`:
```yaml
version: 0.2
phases:
  install:
    commands:
      - npm install
  build:
    commands:
      - npm run build
      - npm test
artifacts:
  files:
    - '**/*'
  base-directory: dist
```

---

**Q81. What is AWS CodeDeploy?**
> CodeDeploy automates code deployment to EC2, Lambda, or on-premises servers. Deployment strategies:
> - **In-place:** Update existing instances one by one
> - **Blue/Green:** Launch new instances, switch traffic, terminate old ones
> Defined in `appspec.yml`.

---

**Q82. What is AWS CodeCommit?**
> CodeCommit is AWS's managed Git repository service (like GitHub but on AWS). Integrates natively with other AWS services. Note: AWS announced EOL for CodeCommit in 2024 — most teams use GitHub/GitLab instead now.

---

**Q83. What is Elastic Beanstalk?**
> Elastic Beanstalk is a PaaS service. You just upload your application code and AWS handles everything: EC2, Load Balancer, Auto Scaling, monitoring. Good for developers who don't want to manage infrastructure.
> Supports: Java, Python, Node.js, Ruby, PHP, .NET, Docker.

---

**Q84. What is AWS Lambda?**
> Lambda is serverless compute. You write a function, upload it, and AWS runs it when triggered — no server management needed. You pay only when it runs (per request + per 100ms execution).
```python
def lambda_handler(event, context):
    print(f"Received event: {event}")
    return {
        'statusCode': 200,
        'body': 'Hello from Lambda!'
    }
```

---

**Q85. What can trigger a Lambda function?**
> - API Gateway (REST/HTTP requests)
> - S3 events (file upload)
> - DynamoDB Streams
> - SQS (message queue)
> - CloudWatch Events / EventBridge (scheduled)
> - SNS (notifications)
> - ALB (HTTP requests)
> - Cognito (auth events)

---

**Q86. What is API Gateway?**
> API Gateway creates, publishes, and manages REST, HTTP, or WebSocket APIs. It's the front door for Lambda functions. Features:
> - Authentication (IAM, Cognito, API Keys)
> - Rate limiting and throttling
> - Caching
> - Request/response transformation
> - CORS support

---

**Q87. What is Amazon ECS?**
> ECS (Elastic Container Service) is AWS's managed Docker container service. You define tasks (containers) and services (how many containers to run). Two launch types:
> - **EC2 launch type:** You manage the EC2 instances (servers)
> - **Fargate launch type:** Serverless — AWS manages the servers

---

**Q88. What is AWS Fargate?**
> Fargate is serverless compute for containers. You define CPU and memory for your containers, and AWS handles the underlying infrastructure. No EC2 management needed.
> Use when: You want containers without managing servers.

---

**Q89. What is Amazon EKS?**
> EKS (Elastic Kubernetes Service) is AWS's managed Kubernetes service. AWS manages the Kubernetes control plane (master nodes). You manage worker nodes (or use Fargate).
> Use when: You need full Kubernetes with AWS integration.

---

**Q90. What is Amazon ECR?**
> ECR (Elastic Container Registry) is AWS's private Docker image registry (like Docker Hub but private in AWS). Push/pull Docker images securely.
```bash
# Login to ECR
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin 123456789.dkr.ecr.ap-south-1.amazonaws.com

# Push image
docker tag myapp:latest 123456789.dkr.ecr.ap-south-1.amazonaws.com/myapp:latest
docker push 123456789.dkr.ecr.ap-south-1.amazonaws.com/myapp:latest
```

---

**Q91. What is CloudWatch?**
> CloudWatch is AWS's monitoring and observability service:
> - **Metrics:** CPU, memory, disk, network (for EC2, RDS, Lambda, etc.)
> - **Logs:** Collect and store log files from EC2, Lambda, etc.
> - **Alarms:** Alert (email, SMS, auto-scale) when metric crosses a threshold
> - **Dashboards:** Visual graphs of your metrics
> - **Events/EventBridge:** React to events in your AWS account

---

**Q92. What is AWS SNS vs SQS?**
> - **SNS (Simple Notification Service):** Pub/Sub messaging. One message → many subscribers (email, SMS, Lambda, SQS). Push-based.
> - **SQS (Simple Queue Service):** Message queue. Producer puts messages in queue, consumer pulls them. Decouples services.
> Common pattern: SNS → SQS → Lambda (fan-out to multiple queues)

---

## 7. Advanced & Real-World Scenarios

**Q93. Design a highly available 3-tier architecture on AWS.**
```
Internet
    ↓
Route 53 (DNS)
    ↓
CloudFront (CDN + WAF)
    ↓
ALB (Load Balancer) — Multi-AZ
    ↓
EC2 Auto Scaling Group (Web/App Tier) — Private Subnet, Multi-AZ
    ↓
RDS MySQL Multi-AZ (Database Tier) — Private Subnet
    +
ElastiCache Redis (Caching Layer)
    +
S3 (Static Assets)
```

---

**Q94. How do you implement Blue-Green deployment on AWS?**
> **Option 1 — Route 53 weighted routing:**
> 1. Blue environment running with 100% traffic
> 2. Deploy new version to Green environment
> 3. Test Green environment
> 4. Shift Route 53 weight: Blue 90%, Green 10%
> 5. Gradually shift: 50/50, then 0/100
> 6. Terminate Blue
>
> **Option 2 — ALB Target Groups:**
> 1. Create new target group (Green) with new instances
> 2. ALB listener rule: send % traffic to Green
> 3. Shift gradually, then delete Blue target group

---

**Q95. How do you reduce AWS costs in a DevOps setup?**
> 1. Use Spot Instances for CI/CD agents (save 70-90%)
> 2. Right-size EC2 instances (use AWS Compute Optimizer)
> 3. Use Reserved Instances for stable workloads
> 4. Schedule non-prod environments to shut down nights/weekends
> 5. Use S3 Lifecycle policies to archive old data
> 6. Delete unattached EBS volumes and unused Elastic IPs
> 7. Use CloudFront to reduce data transfer costs
> 8. Enable AWS Cost Anomaly Detection for alerts

---

**Q96. How do you implement disaster recovery on AWS?**
> DR strategies (from least to most expensive):
> - **Backup & Restore (RTO hours):** S3 backups, restore manually
> - **Pilot Light (RTO minutes):** Core services running in DR region, scale up when needed
> - **Warm Standby (RTO seconds):** Scaled-down version always running in DR region
> - **Multi-Site Active-Active (RTO ~0):** Full production in multiple regions simultaneously

---

**Q97. How do you secure an EC2 instance in production?**
> 1. Use IAM Role (not access keys) for AWS API access
> 2. Keep security group rules minimal (no 0.0.0.0/0 for SSH)
> 3. Use Systems Manager Session Manager instead of SSH
> 4. Enable automatic OS patching (AWS Patch Manager)
> 5. Encrypt EBS volumes with KMS
> 6. Enable CloudTrail and VPC Flow Logs
> 7. Use private subnet — no public IP
> 8. Regular security scans with Amazon Inspector

---

**Q98. What is AWS Well-Architected Framework?**
> Six pillars for building good systems on AWS:
> 1. **Operational Excellence** — Run and monitor systems, improve processes
> 2. **Security** — Protect data and systems
> 3. **Reliability** — Recover from failures, scale dynamically
> 4. **Performance Efficiency** — Use resources efficiently
> 5. **Cost Optimization** — Avoid unnecessary costs
> 6. **Sustainability** — Minimize environmental impact

---

**Q99. What is Infrastructure as Code (IaC) and how does AWS support it?**
> IaC means defining infrastructure in code files (version controlled, repeatable).
> AWS tools:
> - **CloudFormation** — AWS native, YAML/JSON
> - **CDK** — Use Python/TypeScript/Java code
> - **Terraform** — Multi-cloud, very popular
> Benefits: Consistent environments, peer review, audit trail, quick disaster recovery.

---

**Q100. How do you implement auto-healing EC2 instances?**
> 1. **Auto Scaling Group:** If health check fails, ASG terminates and replaces the instance
> 2. **EC2 Auto Recovery:** CloudWatch alarm detects system failure, automatically recovers instance on new hardware
> 3. **ELB Health Checks:** Load balancer stops sending traffic to unhealthy instances

---

**Q101. What is Amazon RDS and its features?**
> RDS (Relational Database Service) is managed SQL database service. Supports MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora.
> Features:
> - **Multi-AZ:** Standby replica in another AZ, automatic failover
> - **Read Replicas:** Up to 5 read replicas for read scaling
> - **Automated Backups:** Daily snapshots, up to 35 days retention
> - **Encryption:** At-rest with KMS, in-transit with SSL

---

**Q102. What is Amazon Aurora?**
> Aurora is AWS's cloud-native database, compatible with MySQL and PostgreSQL but:
> - 5x faster than MySQL
> - 3x faster than PostgreSQL
> - Storage auto-scales up to 128 TB
> - 6 copies of data across 3 AZs (highly durable)
> - Aurora Serverless: Auto-scales compute up/down

---

**Q103. What is ElastiCache and when do you use it?**
> ElastiCache is managed in-memory caching (Redis or Memcached). Use to:
> - Cache frequent database queries (reduce DB load)
> - Store user sessions
> - Leaderboards, real-time analytics
> - Reduces response time from milliseconds to microseconds

---

**Q104. How do you implement a serverless API on AWS?**
```
Client Request
    ↓
API Gateway (HTTPS endpoint)
    ↓
Lambda Function (business logic)
    ↓
DynamoDB (database)
    +
S3 (file storage)
    +
SNS/SQS (async processing)
```
> No servers to manage. Pay only when API is called.

---

**Q105. What is AWS EventBridge?**
> EventBridge is a serverless event bus. It routes events between AWS services, your apps, and SaaS applications. Example:
> - EC2 instance terminates → EventBridge → Lambda → notify Slack
> - S3 file uploaded → EventBridge → Step Functions → process file
> - Schedule: Run Lambda every day at 8AM (like cron)

---

**Q106. What is AWS Step Functions?**
> Step Functions orchestrates multi-step workflows as state machines. Each step can be a Lambda, ECS task, or AWS service call. If a step fails, it retries or goes to an error handler. Visual workflow editor in AWS Console.

---

**Q107. What is Amazon DynamoDB?**
> DynamoDB is AWS's fully managed NoSQL database. Features:
> - Single-digit millisecond performance at any scale
> - Serverless — no capacity planning
> - Auto-scaling
> - Multi-region replication (Global Tables)
> - On-demand or provisioned capacity mode
> Best for: User profiles, shopping carts, gaming leaderboards, IoT data.

---

**Q108. How do you monitor a production application on AWS?**
> Complete observability stack:
> 1. **Metrics:** CloudWatch Metrics + custom metrics from app
> 2. **Logs:** CloudWatch Logs / OpenSearch (Elasticsearch)
> 3. **Tracing:** AWS X-Ray (trace requests across services)
> 4. **Alarms:** CloudWatch Alarms → SNS → Email/Slack/PagerDuty
> 5. **Dashboards:** CloudWatch Dashboards or Grafana
> 6. **Synthetic Monitoring:** CloudWatch Synthetics (canary scripts)

---

**Q109. What is AWS X-Ray?**
> X-Ray is distributed tracing for your application. It shows you the full path of a request across microservices — how long each step took, where errors occurred. Essential for debugging microservices performance issues.

---

**Q110. Real-world scenario: How do you deploy a containerized microservice to AWS?**
```
Developer pushes code to GitHub
    ↓
GitHub Actions / Jenkins triggers pipeline
    ↓
CodeBuild builds Docker image
    ↓
Image pushed to ECR (private registry)
    ↓
ECS/EKS deployment updated (rolling update)
    ↓
ALB health checks verify new containers
    ↓
Old containers gracefully terminated
    ↓
CloudWatch monitors metrics
    ↓
X-Ray traces requests
    ↓
CloudWatch Alarm → SNS → Slack if errors spike
```

---
