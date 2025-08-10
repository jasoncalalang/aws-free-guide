# Day 1: AWS Fundamentals & VPC Basics

**Welcome to your AWS VPC learning journey!** Today you'll set up your AWS account, understand fundamental concepts, and create your first VPC using the AWS wizard. This foundation will support everything we build in the coming days.

## 📋 Table of Contents
- [Learning Objectives](#-learning-objectives)
- [Prerequisites](#-prerequisites)
- [AWS Account Setup](#-aws-account-setup)
- [Understanding VPC Fundamentals](#-understanding-vpc-fundamentals)
- [Hands-On Lab: Create Your First VPC](#-hands-on-lab-create-your-first-vpc)
- [Free Tier Monitoring Setup](#-free-tier-monitoring-setup)
- [Verification Steps](#-verification-steps)
- [Troubleshooting](#-troubleshooting)
- [Clean Up Resources](#-clean-up-resources)
- [Knowledge Check](#-knowledge-check)
- [Additional Resources](#-additional-resources)
- [Next Steps](#-next-steps)

## 🎯 Learning Objectives

By the end of Day 1, you will:
- [ ] Create and secure an AWS account with Free Tier access
- [ ] Set up IAM user with appropriate VPC permissions
- [ ] Understand AWS regions, availability zones, and VPC concepts
- [ ] Learn CIDR notation and IP address planning
- [ ] Create your first VPC using the AWS VPC Wizard
- [ ] Configure billing alerts to stay within Free Tier
- [ ] Navigate the AWS Console confidently

## ⏱️ Time Required: 2-3 hours

## 📚 Prerequisites
- Computer with internet connection
- Valid email address
- Basic understanding that this is a learning environment
- **NO** prior AWS experience needed!

## 🔐 AWS Account Setup

### Step 1: Create Your AWS Account

⚠️ **Free Tier Alert:** New AWS accounts get 12 months of Free Tier access. This is crucial for our cost-free learning!

1. **Navigate to AWS:** Go to [aws.amazon.com](https://aws.amazon.com)
2. **Click "Create an AWS Account"**
3. **Enter Account Details:**
   ```
   Account Name: YourName-Learning-Account
   Email: your.email@example.com
   Password: [Strong password with 12+ characters]
   ```
4. **Choose Account Type:** Personal (for learning)
5. **Enter Contact Information:** Use your real information
6. **Payment Information:** 
   - ⚠️ **Important:** AWS requires a credit card for identity verification
   - You won't be charged if you stay within Free Tier limits
   - We'll set up billing alerts to prevent charges

### Step 2: Verify Your Account
1. **Phone Verification:** AWS will call or text you
2. **Identity Verification:** May take a few minutes
3. **Support Plan:** Choose "Basic Support" (Free)

### Step 3: Sign In and Initial Setup
1. **Sign in to AWS Console:** [console.aws.amazon.com](https://console.aws.amazon.com)
2. **Select Region:** Choose **US East (N. Virginia) us-east-1**
   - Most Free Tier resources available here
   - Lowest latency for most users
   - Consistent with this guide's examples

## 👤 IAM User Setup

⚠️ **Security Best Practice:** Never use your root account for daily AWS tasks!

### Step 1: Create IAM User
1. **Navigate to IAM:** Search "IAM" in the AWS Console search bar
2. **Click "Users"** in the left sidebar
3. **Click "Create user"**
4. **User Details:**
   ```
   User name: vpc-learner
   AWS access type: 
   ✅ Programmatic access (for CLI)
   ✅ AWS Management Console access
   Console password: Custom password
   ✅ Require password reset: No (uncheck this)
   ```

### Step 2: Set Permissions
1. **Click "Next: Permissions"**
2. **Choose "Attach policies directly"**
3. **Search and select these policies:**
   - `AmazonVPCFullAccess`
   - `AmazonEC2FullAccess`
   - `IAMReadOnlyAccess`
   - `AmazonS3ReadOnlyAccess`
   - `CloudWatchReadOnlyAccess`

⚠️ **Learning Note:** In production, you'd use least-privilege access. For learning, these broad permissions simplify the experience.

### Step 3: Download Credentials
1. **Click "Create user"**
2. **⚠️ CRITICAL:** Download the CSV file with your credentials
3. **Save it securely** - you'll need it for AWS CLI setup

### Step 4: Set Up AWS CLI
```bash
# Install AWS CLI (if not already installed)
# Windows: Download from https://aws.amazon.com/cli/
# macOS: brew install awscli
# Linux: sudo apt-get install awscli

# Configure CLI with your IAM user credentials
aws configure

# Enter the following when prompted:
AWS Access Key ID: [From downloaded CSV]
AWS Secret Access Key: [From downloaded CSV]
Default region name: us-east-1
Default output format: json
```

**Test your setup:**
```bash
aws sts get-caller-identity
```

Expected output:
```json
{
    "UserId": "AIDACKCEVSQ6C2EXAMPLE",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/vpc-learner"
}
```

## 🌐 Understanding VPC Fundamentals

### What is a VPC?
A **Virtual Private Cloud (VPC)** is your own isolated section of AWS cloud where you can launch AWS resources in a virtual network that you define.

Think of it as your own private data center in the cloud, but:
- ✅ No physical hardware to manage
- ✅ Scalable on demand
- ✅ Pay only for what you use
- ✅ Globally distributed

### Key VPC Components

| Component | Purpose | Free Tier Status |
|-----------|---------|------------------|
| **VPC** | Virtual network container | ✅ Always Free |
| **Subnets** | Network segments within VPC | ✅ Always Free |
| **Internet Gateway** | Internet access | ✅ Always Free |
| **Route Tables** | Traffic routing rules | ✅ Always Free |
| **Security Groups** | Instance-level firewall | ✅ Always Free |
| **NACLs** | Subnet-level firewall | ✅ Always Free |
| **NAT Instance** | Private subnet internet access | ✅ Free with t2.micro |
| **NAT Gateway** | Managed NAT service | ❌ $45+/month |

### Understanding CIDR Notation

**CIDR (Classless Inter-Domain Routing)** is how we define IP address ranges.

Examples:
```
10.0.0.0/16    = 65,536 IP addresses (10.0.0.0 to 10.0.255.255)
10.0.1.0/24    = 256 IP addresses (10.0.1.0 to 10.0.1.255)
10.0.1.0/28    = 16 IP addresses (10.0.1.0 to 10.0.1.15)
192.168.1.0/24 = 256 IP addresses (192.168.1.0 to 192.168.1.255)
```

**Quick Reference:**
- `/16` = 65,536 IPs (Class B equivalent)
- `/24` = 256 IPs (Class C equivalent)
- `/28` = 16 IPs (small subnet)

### AWS Regions and Availability Zones

**Region:** Geographic area with multiple data centers
- Example: us-east-1 (N. Virginia)
- Choose based on: latency, compliance, service availability

**Availability Zone (AZ):** Individual data center within a region
- Example: us-east-1a, us-east-1b, us-east-1c
- Each AZ is isolated for fault tolerance
- Connected with high-speed, low-latency networking

## 🔬 Hands-On Lab: Create Your First VPC

### Step 1: Access VPC Console
1. **Sign in** to AWS Console as your IAM user (not root!)
2. **Navigate to VPC:** Search "VPC" in the console search bar
3. **Select "VPC"** from the results

### Step 2: Launch VPC Wizard
1. **Click "Create VPC"**
2. **Choose "VPC and more"** for the guided experience
3. **Configure VPC Settings:**

```
Resources to create: VPC and more
Name tag auto-generation: 
✅ Auto-generate name tags
VPC name: Day1-Learning-VPC

IPv4 CIDR block: 10.0.0.0/16
IPv6 CIDR block: No IPv6 CIDR block
Tenancy: Default

Number of Availability Zones: 2
Number of public subnets: 2
Number of private subnets: 2

NAT gateways: None
VPC endpoints: None
DNS options:
✅ Enable DNS hostnames
✅ Enable DNS resolution
```

⚠️ **Cost Alert:** We selected "None" for NAT gateways because they cost $45+/month. We'll create a NAT instance on Day 5 instead!

### Step 3: Review and Create
1. **Review** your configuration in the preview
2. **Verify** no costly resources are selected
3. **Click "Create VPC"**

The wizard will create:
- 1 VPC with 10.0.0.0/16 CIDR
- 2 Public subnets (10.0.0.0/24, 10.0.1.0/24)
- 2 Private subnets (10.0.128.0/24, 10.0.129.0/24)
- 1 Internet Gateway
- Route tables for public and private subnets
- Security groups and NACLs

### Step 4: Explore Your VPC
1. **Navigate to "Your VPCs"** to see your new VPC
2. **Click on your VPC ID** to see details
3. **Explore the following sections:**
   - Subnets
   - Route Tables
   - Internet Gateways
   - Security Groups
   - Network ACLs

## 💰 Free Tier Monitoring Setup

### Step 1: Create Billing Alerts
1. **Navigate to Billing:** Click your account name → "Billing Dashboard"
2. **Click "Budgets"** in left sidebar
3. **Click "Create budget"**
4. **Choose "Cost budget"**
5. **Configure Budget:**
   ```
   Budget name: Free-Tier-Alert
   Budget amount: $10
   Time unit: Monthly
   Recurring budget
   ```
6. **Set up alerts:**
   ```
   Alert threshold: 80% of budgeted amount ($8)
   Email recipients: [Your email]
   ```

### Step 2: Enable Detailed Billing
1. **Go to Account Settings** in Billing Console
2. **Enable "Receive Billing Alerts"**
3. **Save preferences**

### Step 3: CloudWatch Billing Alarm
```bash
# Create a billing alarm using AWS CLI
aws cloudwatch put-metric-alarm \
    --alarm-name "Free-Tier-Spending-Alert" \
    --alarm-description "Alert when spending exceeds $5" \
    --metric-name EstimatedCharges \
    --namespace AWS/Billing \
    --statistic Maximum \
    --period 86400 \
    --threshold 5.0 \
    --comparison-operator GreaterThanThreshold \
    --dimensions Name=Currency,Value=USD \
    --evaluation-periods 1 \
    --alarm-actions arn:aws:sns:us-east-1:YOUR-ACCOUNT-ID:billing-alerts
```

## ✅ Verification Steps

### Step 1: Verify VPC Creation
```bash
# List your VPCs
aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,CidrBlock,State]' --output table

# Expected output: Your VPC with 10.0.0.0/16 CIDR in 'available' state
```

### Step 2: Check Subnets
```bash
# List subnets in your VPC
aws ec2 describe-subnets --filters "Name=vpc-id,Values=YOUR-VPC-ID" \
    --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone,MapPublicIpOnLaunch]' \
    --output table
```

Expected results:
- 2 public subnets with `MapPublicIpOnLaunch: true`
- 2 private subnets with `MapPublicIpOnLaunch: false`

### Step 3: Verify Internet Gateway
```bash
# Check Internet Gateway attachment
aws ec2 describe-internet-gateways --filters "Name=attachment.vpc-id,Values=YOUR-VPC-ID" \
    --query 'InternetGateways[*].[InternetGatewayId,Attachments[0].State]' \
    --output table
```

### Step 4: Console Verification
1. **VPC Dashboard:** Should show 1 VPC
2. **Subnets:** Should show 4 subnets (2 public, 2 private)
3. **Internet Gateways:** Should show 1 attached gateway
4. **Route Tables:** Should show 2 route tables

## 🐛 Troubleshooting

### Issue: "User not authorized to perform actions"
**Cause:** Missing IAM permissions
**Solution:**
1. Sign in as root user
2. Go to IAM → Users → your user
3. Add the missing policies from Step 2 of IAM setup

### Issue: "Cannot create VPC in this region"
**Cause:** Region doesn't support your requested configuration
**Solution:**
1. Switch to us-east-1 region
2. Try creating VPC again
3. Verify Free Tier availability in your chosen region

### Issue: VPC Wizard fails
**Cause:** Various reasons (limits, permissions, region)
**Solution:**
1. Check if you've hit VPC limits (default: 5 VPCs per region)
2. Verify you're in us-east-1
3. Try refreshing browser and attempting again

### Issue: Billing alerts not working
**Cause:** Billing alerts can take 24 hours to activate
**Solution:**
1. Wait 24 hours after setup
2. Verify email address is correct
3. Check spam folder for AWS notifications

## 🧹 Clean Up Resources

⚠️ **Important:** VPCs themselves are free, but we'll clean up to prepare for Day 2's custom VPC creation.

### Step 1: Delete VPC (End of Day)
```bash
# Get your VPC ID first
aws ec2 describe-vpcs --query 'Vpcs[?IsDefault==`false`].[VpcId,CidrBlock]' --output table

# Delete the VPC (this will delete all associated resources)
aws ec2 delete-vpc --vpc-id YOUR-VPC-ID
```

### Step 2: Verify Cleanup
```bash
# Confirm VPC is deleted
aws ec2 describe-vpcs --query 'Vpcs[?IsDefault==`false`]' --output table

# Should return empty or only show default VPC
```

### Console Cleanup Alternative:
1. **Go to VPC Console**
2. **Select your VPC**
3. **Actions → Delete VPC**
4. **Type "delete" to confirm**

**Why Clean Up:** Tomorrow we'll build a VPC from scratch to understand each component deeply.

## 🧠 Knowledge Check

Test your understanding with these questions:

1. **What CIDR block did we use for our VPC, and how many IP addresses does it provide?**
   <details>
   <summary>Answer</summary>
   10.0.0.0/16 provides 65,536 IP addresses (2^16 - /16 leaves 16 bits for host addresses)
   </details>

2. **What's the difference between a public and private subnet?**
   <details>
   <summary>Answer</summary>
   Public subnets have a route to an Internet Gateway and can assign public IPs. Private subnets don't have direct internet access.
   </details>

3. **Why did we choose "None" for NAT Gateway in the wizard?**
   <details>
   <summary>Answer</summary>
   NAT Gateways cost $45+/month and would exceed our Free Tier budget. We'll use a NAT instance instead on Day 5.
   </details>

4. **What IAM principle did we follow when creating our user?**
   <details>
   <summary>Answer</summary>
   We avoided using the root account for daily operations, which is a security best practice.
   </details>

## 📚 Additional Resources

### AWS Documentation
- [VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)
- [VPC Pricing](https://aws.amazon.com/vpc/pricing/)
- [Free Tier Details](https://aws.amazon.com/free/)

### Networking Fundamentals
- [CIDR Calculator](https://www.ipaddressguide.com/cidr)
- [Subnetting Practice](https://subnettingpractice.com/)
- [TCP/IP Basics](https://www.cloudflare.com/learning/network-layer/what-is-a-protocol/)

### Video Tutorials
- [AWS VPC Beginner Tutorial](https://www.youtube.com/watch?v=fpxDGU2KdkA)
- [CIDR Notation Explained](https://www.youtube.com/watch?v=POPoAjWFkGg)

### Community Resources
- [r/aws Subreddit](https://reddit.com/r/aws)
- [AWS Community Forums](https://forums.aws.amazon.com/)
- [Stack Overflow AWS VPC Tags](https://stackoverflow.com/questions/tagged/amazon-vpc)

## 🎯 Day 1 Summary

Congratulations! You've completed Day 1 and have:

✅ **Set up a secure AWS account** with Free Tier monitoring  
✅ **Created an IAM user** following security best practices  
✅ **Learned VPC fundamentals** including CIDR notation  
✅ **Built your first VPC** using the AWS wizard  
✅ **Configured billing alerts** to prevent unexpected charges  
✅ **Gained familiarity** with the AWS Console and CLI

### Key Concepts Mastered:
- AWS account security with IAM
- VPC components and their relationships
- CIDR notation and IP planning
- AWS regions and availability zones
- Free Tier monitoring and cost control

### Skills Developed:
- AWS Console navigation
- AWS CLI basic usage
- VPC wizard operation
- Billing alert configuration
- Resource cleanup procedures

## 🚀 Next Steps

**[Continue to Day 2: Custom VPC & Subnet Design →](Day2.md)**

Tomorrow you'll:
- 🏗️ Build a VPC from scratch (no wizard!)
- 🎯 Master subnet design and CIDR planning
- 🛣️ Configure routing tables manually
- 🌐 Set up Internet Gateway connectivity
- 🧪 Test network connectivity concepts

**Rest up and get ready to become a VPC architect!** 🎉

---

**Questions or stuck?** Check the [troubleshooting section](#-troubleshooting) or refer to the [additional resources](#-additional-resources).

**Remember:** Learning cloud networking takes time. Don't worry if everything doesn't click immediately - we'll reinforce these concepts throughout the week!