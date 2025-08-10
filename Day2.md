# Day 2: Custom VPC & Subnet Design

**Welcome to Day 2!** Yesterday you learned the basics with the VPC wizard. Today, you'll become a VPC architect by building everything from scratch. You'll master CIDR planning, subnet design, and routing concepts that form the foundation of all AWS networking.

## 📋 Table of Contents
- [Learning Objectives](#-learning-objectives)
- [Prerequisites Review](#-prerequisites-review)
- [CIDR Design Fundamentals](#-cidr-design-fundamentals)
- [Planning Your Custom VPC](#-planning-your-custom-vpc)
- [Hands-On Lab: Build VPC from Scratch](#-hands-on-lab-build-vpc-from-scratch)
- [Routing Configuration](#-routing-configuration)
- [Connectivity Testing](#-connectivity-testing)
- [Verification Steps](#-verification-steps)
- [Troubleshooting](#-troubleshooting)
- [Clean Up Resources](#-clean-up-resources)
- [Knowledge Check](#-knowledge-check)
- [Additional Resources](#-additional-resources)
- [Next Steps](#-next-steps)

## 🎯 Learning Objectives

By the end of Day 2, you will:
- [ ] Design CIDR blocks for scalable VPC architecture
- [ ] Create a custom VPC without using the wizard
- [ ] Build public and private subnets across multiple AZs
- [ ] Configure Internet Gateway and attach to VPC
- [ ] Create and configure custom route tables
- [ ] Understand route table associations and priorities
- [ ] Test basic network connectivity
- [ ] Follow subnet naming and tagging best practices

## ⏱️ Time Required: 2-3 hours

## 📚 Prerequisites Review

Before starting Day 2, ensure you have:
- [ ] Completed Day 1 successfully
- [ ] AWS account with IAM user configured
- [ ] AWS CLI installed and configured
- [ ] Cleaned up Day 1 resources
- [ ] Billing alerts active and monitoring

### Quick Environment Check
```bash
# Verify your AWS CLI is working
aws sts get-caller-identity

# Check current region
aws configure get region

# Verify no existing custom VPCs (should only show default VPC)
aws ec2 describe-vpcs --query 'Vpcs[?IsDefault==`false`]' --output table
```

## 📊 CIDR Design Fundamentals

### Understanding CIDR Planning

**CIDR planning is critical** for scalable, non-overlapping network design. Poor planning leads to:
- ❌ IP address conflicts
- ❌ Inability to connect VPCs
- ❌ Wasted IP address space
- ❌ Complex routing scenarios

### CIDR Best Practices

| Principle | Recommendation | Example |
|-----------|----------------|---------|
| **Future Growth** | Plan for 3-5x current needs | If you need 100 IPs, plan for 500+ |
| **Non-Overlapping** | Use RFC 1918 ranges consistently | 10.x, 172.16-31.x, 192.168.x |
| **Hierarchical** | Use consistent subnet allocation | /16 VPC, /24 subnets |
| **Documentation** | Document all CIDR allocations | Maintain IP address management |

### CIDR Sizing Quick Reference

| CIDR | Usable IPs | Best Use Case |
|------|------------|---------------|
| `/16` | 65,534 | Large VPC (enterprise) |
| `/20` | 4,094 | Medium VPC (department) |
| `/24` | 254 | Standard subnet |
| `/25` | 126 | Small subnet |
| `/26` | 62 | Micro subnet |
| `/28` | 14 | Load balancer subnet |

### AWS Reserved IP Addresses

AWS reserves **5 IP addresses** in every subnet:
```
Example subnet: 10.0.1.0/24 (10.0.1.0 - 10.0.1.255)
Reserved:
- 10.0.1.0   : Network address
- 10.0.1.1   : VPC router
- 10.0.1.2   : DNS server
- 10.0.1.3   : Future use
- 10.0.1.255 : Broadcast address
```

**Usable range:** 10.0.1.4 - 10.0.1.254 (251 addresses)

## 🏗️ Planning Your Custom VPC

### Architecture Overview

Today we'll build a production-ready VPC architecture:

```
Custom VPC: 10.0.0.0/16 (65,536 IPs)
├── us-east-1a
│   ├── Public Subnet:  10.0.1.0/24 (251 usable IPs)
│   └── Private Subnet: 10.0.11.0/24 (251 usable IPs)
├── us-east-1b
│   ├── Public Subnet:  10.0.2.0/24 (251 usable IPs)
│   └── Private Subnet: 10.0.12.0/24 (251 usable IPs)
└── us-east-1c
    ├── Public Subnet:  10.0.3.0/24 (251 usable IPs)
    └── Private Subnet: 10.0.13.0/24 (251 usable IPs)
```

### Subnet Numbering Strategy

**Public Subnets:** 10.0.1-9.x (single digits)
**Private Subnets:** 10.0.11-19.x (teens)
**Database Subnets:** 10.0.21-29.x (twenties) - for future use
**Management Subnets:** 10.0.31-39.x (thirties) - for future use

This leaves room for growth and maintains logical organization.

### Naming Convention

**Resource Naming Pattern:**
```
Environment-Project-Component-AZ
Example: Prod-Learning-Public-1a
```

**Tags to Apply:**
```
Environment: Learning
Project: VPC-Architecture
Purpose: [Public/Private/Database/Management]
AZ: [us-east-1a/1b/1c]
```

## 🔬 Hands-On Lab: Build VPC from Scratch

### Step 1: Create Custom VPC

⚠️ **Cost Check:** VPCs are always free, but verify no NAT Gateways are created by mistake!

```bash
# Create the VPC
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Learning-VPC},{Key=Environment,Value=Learning},{Key=Project,Value=VPC-Architecture}]'
```

**Expected Output:**
```json
{
    "Vpc": {
        "VpcId": "vpc-12345678",
        "State": "pending",
        "CidrBlock": "10.0.0.0/16",
        "DhcpOptionsId": "dopt-12345678",
        "InstanceTenancy": "default",
        "IsDefault": false
    }
}
```

**Save your VPC ID:** `export VPC_ID=vpc-12345678`

### Step 2: Enable DNS Hostnames and Resolution

```bash
# Enable DNS hostnames (required for public IPs to get DNS names)
aws ec2 modify-vpc-attribute \
    --vpc-id $VPC_ID \
    --enable-dns-hostnames

# Enable DNS resolution (enabled by default, but let's be explicit)
aws ec2 modify-vpc-attribute \
    --vpc-id $VPC_ID \
    --enable-dns-support
```

### Step 3: Create Subnets

#### Public Subnets

```bash
# Public Subnet in us-east-1a
aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Learning-Public-1a},{Key=Environment,Value=Learning},{Key=Type,Value=Public},{Key=AZ,Value=us-east-1a}]'

# Public Subnet in us-east-1b
aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.2.0/24 \
    --availability-zone us-east-1b \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Learning-Public-1b},{Key=Environment,Value=Learning},{Key=Type,Value=Public},{Key=AZ,Value=us-east-1b}]'

# Public Subnet in us-east-1c
aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.3.0/24 \
    --availability-zone us-east-1c \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Learning-Public-1c},{Key=Environment,Value=Learning},{Key=Type,Value=Public},{Key=AZ,Value=us-east-1c}]'
```

#### Private Subnets

```bash
# Private Subnet in us-east-1a
aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.11.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Learning-Private-1a},{Key=Environment,Value=Learning},{Key=Type,Value=Private},{Key=AZ,Value=us-east-1a}]'

# Private Subnet in us-east-1b
aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.12.0/24 \
    --availability-zone us-east-1b \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Learning-Private-1b},{Key=Environment,Value=Learning},{Key=Type,Value=Private},{Key=AZ,Value=us-east-1b}]'

# Private Subnet in us-east-1c
aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.13.0/24 \
    --availability-zone us-east-1c \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Learning-Private-1c},{Key=Environment,Value=Learning},{Key=Type,Value=Private},{Key=AZ,Value=us-east-1c}]'
```

### Step 4: Create and Attach Internet Gateway

```bash
# Create Internet Gateway
aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Learning-IGW},{Key=Environment,Value=Learning}]'

# Save IGW ID (replace with your actual ID)
export IGW_ID=igw-12345678

# Attach Internet Gateway to VPC
aws ec2 attach-internet-gateway \
    --internet-gateway-id $IGW_ID \
    --vpc-id $VPC_ID
```

## 🛣️ Routing Configuration

### Understanding Route Tables

**Route Tables** control traffic routing within your VPC:
- Every VPC has a **main route table** (default)
- You can create **custom route tables** for specific subnets
- **Most specific route wins** (longest prefix match)
- **Local routes** are automatically created and cannot be deleted

### Step 1: Create Public Route Table

```bash
# Create custom route table for public subnets
aws ec2 create-route-table \
    --vpc-id $VPC_ID \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Learning-Public-RT},{Key=Environment,Value=Learning},{Key=Type,Value=Public}]'

# Save Route Table ID
export PUBLIC_RT_ID=rtb-12345678

# Add route to Internet Gateway for public access
aws ec2 create-route \
    --route-table-id $PUBLIC_RT_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id $IGW_ID
```

### Step 2: Associate Public Subnets with Public Route Table

```bash
# Get public subnet IDs
PUBLIC_SUBNET_1A=$(aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=$VPC_ID" "Name=tag:Name,Values=Learning-Public-1a" \
    --query 'Subnets[0].SubnetId' --output text)

PUBLIC_SUBNET_1B=$(aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=$VPC_ID" "Name=tag:Name,Values=Learning-Public-1b" \
    --query 'Subnets[0].SubnetId' --output text)

PUBLIC_SUBNET_1C=$(aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=$VPC_ID" "Name=tag:Name,Values=Learning-Public-1c" \
    --query 'Subnets[0].SubnetId' --output text)

# Associate public subnets with public route table
aws ec2 associate-route-table --subnet-id $PUBLIC_SUBNET_1A --route-table-id $PUBLIC_RT_ID
aws ec2 associate-route-table --subnet-id $PUBLIC_SUBNET_1B --route-table-id $PUBLIC_RT_ID
aws ec2 associate-route-table --subnet-id $PUBLIC_SUBNET_1C --route-table-id $PUBLIC_RT_ID
```

### Step 3: Configure Public IP Assignment

```bash
# Enable auto-assign public IP for public subnets
aws ec2 modify-subnet-attribute --subnet-id $PUBLIC_SUBNET_1A --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id $PUBLIC_SUBNET_1B --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id $PUBLIC_SUBNET_1C --map-public-ip-on-launch
```

### Step 4: Verify Route Table Configuration

```bash
# Check public route table routes
aws ec2 describe-route-tables \
    --route-table-ids $PUBLIC_RT_ID \
    --query 'RouteTables[0].Routes' \
    --output table
```

**Expected Routes:**
| Destination | Target | Status |
|------------|--------|--------|
| 10.0.0.0/16 | local | active |
| 0.0.0.0/0 | igw-xxxxx | active |

## 🧪 Connectivity Testing

### Understanding the Test Plan

We'll verify connectivity by:
1. **VPC-level connectivity:** Can resources communicate within VPC?
2. **Internet connectivity:** Can public subnets reach the internet?
3. **Route resolution:** Are routes working correctly?

### Console-Based Verification

1. **Navigate to VPC Console**
2. **Go to "Your VPCs"** and select your Learning-VPC
3. **Check the "Resource Map"** tab to visualize your architecture
4. **Verify Route Tables:**
   - Public route table has route to IGW (0.0.0.0/0 → igw-xxxxx)
   - Main route table only has local route
5. **Check Subnet Associations:**
   - Public subnets are associated with public route table
   - Private subnets use main route table (no internet route)

### CLI-Based Verification

```bash
# Verify VPC CIDR and state
aws ec2 describe-vpcs --vpc-ids $VPC_ID \
    --query 'Vpcs[0].[VpcId,CidrBlock,State]' \
    --output table

# Check all subnets in VPC
aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" \
    --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone,MapPublicIpOnLaunch,Tags[?Key==`Name`].Value|[0]]' \
    --output table

# Verify Internet Gateway attachment
aws ec2 describe-internet-gateways \
    --filters "Name=attachment.vpc-id,Values=$VPC_ID" \
    --query 'InternetGateways[0].[InternetGatewayId,Attachments[0].State]' \
    --output table
```

## ✅ Verification Steps

### Step 1: Architecture Verification

**VPC Structure Check:**
```bash
# Complete VPC summary
echo "=== VPC SUMMARY ==="
echo "VPC ID: $VPC_ID"
echo "VPC CIDR: $(aws ec2 describe-vpcs --vpc-ids $VPC_ID --query 'Vpcs[0].CidrBlock' --output text)"
echo ""

echo "=== SUBNETS ==="
aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" \
    --query 'Subnets[*].[Tags[?Key==`Name`].Value|[0],CidrBlock,AvailabilityZone,MapPublicIpOnLaunch]' \
    --output table

echo "=== ROUTE TABLES ==="
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=$VPC_ID" \
    --query 'RouteTables[*].[RouteTableId,Tags[?Key==`Name`].Value|[0],Associations[0].SubnetId]' \
    --output table
```

### Step 2: Routing Verification

```bash
# Check public route table has internet route
aws ec2 describe-route-tables --route-table-ids $PUBLIC_RT_ID \
    --query 'RouteTables[0].Routes[?DestinationCidrBlock==`0.0.0.0/0`].[DestinationCidrBlock,GatewayId,State]' \
    --output table

# Should show: 0.0.0.0/0 | igw-xxxxx | active
```

### Step 3: DNS and Connectivity Features

```bash
# Verify DNS settings
aws ec2 describe-vpc-attribute --vpc-id $VPC_ID --attribute enableDnsHostnames
aws ec2 describe-vpc-attribute --vpc-id $VPC_ID --attribute enableDnsSupport
```

Both should return `"Value": true`.

### Step 4: Tagging and Organization

```bash
# Verify consistent tagging
aws ec2 describe-tags --filters "Name=resource-id,Values=$VPC_ID" \
    --query 'Tags[*].[Key,Value]' \
    --output table
```

## 🐛 Troubleshooting

### Issue: Subnet Creation Fails
**Error:** `InvalidParameterValue: The CIDR '10.0.1.0/24' conflicts with another subnet`

**Diagnosis:**
```bash
# Check existing subnets in VPC
aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" \
    --query 'Subnets[*].CidrBlock'
```

**Solution:** Choose a different CIDR block that doesn't overlap.

### Issue: Internet Gateway Attachment Fails
**Error:** `Resource.AlreadyAssociated: IGW is already attached to a VPC`

**Diagnosis:**
```bash
# Check IGW current attachments
aws ec2 describe-internet-gateways --internet-gateway-ids $IGW_ID \
    --query 'InternetGateways[0].Attachments'
```

**Solution:** Detach from current VPC first or create a new IGW.

### Issue: Route Creation Fails
**Error:** `InvalidRouteTableID.NotFound`

**Solution:** Verify route table ID and that it exists in the correct region.

### Issue: Public IP Assignment Not Working
**Diagnosis:**
```bash
# Check subnet public IP setting
aws ec2 describe-subnets --subnet-ids $PUBLIC_SUBNET_1A \
    --query 'Subnets[0].MapPublicIpOnLaunch'
```

**Solution:** Enable auto-assign public IP:
```bash
aws ec2 modify-subnet-attribute --subnet-id $PUBLIC_SUBNET_1A --map-public-ip-on-launch
```

### Issue: DNS Resolution Problems
**Diagnosis:** Check VPC DNS attributes
```bash
aws ec2 describe-vpc-attribute --vpc-id $VPC_ID --attribute enableDnsHostnames
aws ec2 describe-vpc-attribute --vpc-id $VPC_ID --attribute enableDnsSupport
```

**Solution:** Enable both DNS features:
```bash
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-support
```

## 🧹 Clean Up Resources

⚠️ **Important:** Clean up in reverse order of creation to avoid dependency issues.

### Cleanup Script

```bash
#!/bin/bash
# Day 2 Cleanup Script

echo "Starting cleanup of Day 2 resources..."

# Get resource IDs
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=tag:Name,Values=Learning-VPC" --query 'Vpcs[0].VpcId' --output text)

if [ "$VPC_ID" != "None" ]; then
    echo "Found VPC: $VPC_ID"
    
    # Get IGW ID
    IGW_ID=$(aws ec2 describe-internet-gateways --filters "Name=attachment.vpc-id,Values=$VPC_ID" --query 'InternetGateways[0].InternetGatewayId' --output text)
    
    # Detach and delete Internet Gateway
    if [ "$IGW_ID" != "None" ]; then
        echo "Detaching Internet Gateway: $IGW_ID"
        aws ec2 detach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
        aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID
    fi
    
    # Delete custom route tables (keep main route table)
    echo "Deleting custom route tables..."
    aws ec2 describe-route-tables --filters "Name=vpc-id,Values=$VPC_ID" --query 'RouteTables[?Associations[0].Main!=`true`].RouteTableId' --output text | xargs -n1 aws ec2 delete-route-table --route-table-id
    
    # Delete subnets
    echo "Deleting subnets..."
    aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" --query 'Subnets[].SubnetId' --output text | xargs -n1 aws ec2 delete-subnet --subnet-id
    
    # Delete VPC
    echo "Deleting VPC: $VPC_ID"
    aws ec2 delete-vpc --vpc-id $VPC_ID
    
    echo "Cleanup completed!"
else
    echo "No Learning VPC found."
fi
```

### Manual Console Cleanup
1. **VPC Console → Your VPCs**
2. **Select Learning-VPC**
3. **Actions → Delete VPC**
4. **Type "delete" to confirm**

**This deletes all associated resources automatically.**

### Verification
```bash
# Verify cleanup
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=Learning-VPC" --output table
# Should return empty
```

## 🧠 Knowledge Check

Test your Day 2 understanding:

1. **How many usable IP addresses are in a /24 subnet, and why?**
   <details>
   <summary>Answer</summary>
   251 usable IPs. A /24 subnet has 256 total IPs (2^8), but AWS reserves 5: network address, VPC router, DNS server, future use, and broadcast address.
   </details>

2. **What's the difference between the main route table and a custom route table?**
   <details>
   <summary>Answer</summary>
   The main route table is created automatically with the VPC and contains only local routes by default. Custom route tables allow you to define specific routing rules for different subnets.
   </details>

3. **Why do we associate public subnets with a custom route table instead of modifying the main route table?**
   <details>
   <summary>Answer</summary>
   Security best practice. The main route table should remain private (local only). New subnets automatically use the main route table, so they'll be private by default.
   </details>

4. **What route makes a subnet "public"?**
   <details>
   <summary>Answer</summary>
   A route with destination 0.0.0.0/0 (all traffic) pointing to an Internet Gateway, plus auto-assign public IP enabled.
   </details>

5. **How would you add a fourth availability zone to this design?**
   <details>
   <summary>Answer</summary>
   Add subnets in us-east-1d: public subnet 10.0.4.0/24 and private subnet 10.0.14.0/24, following the same numbering pattern.
   </details>

## 📚 Additional Resources

### CIDR and Subnetting Tools
- [Visual Subnet Calculator](https://www.davidc.net/sites/default/subnets/subnets.html)
- [CIDR.xyz](https://cidr.xyz/) - Interactive CIDR calculator
- [IP Address Guide](https://www.ipaddressguide.com/cidr)

### AWS Documentation
- [VPC Subnets](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)
- [Route Tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

### Best Practices
- [VPC Design Best Practices](https://aws.amazon.com/blogs/networking-and-content-delivery/vpc-design-best-practices/)
- [IP Address Planning](https://d1.awsstatic.com/whitepapers/building-a-scalable-and-secure-multi-vpc-aws-network-infrastructure.pdf)

### Video Tutorials
- [AWS VPC Deep Dive](https://www.youtube.com/watch?v=fpxDGU2KdkA)
- [Subnetting Made Easy](https://www.youtube.com/watch?v=BWZ-MHIhqjM)

## 🎯 Day 2 Summary

Outstanding work! You've completed Day 2 and have:

✅ **Mastered CIDR planning** for scalable network design  
✅ **Built a custom VPC** from scratch without the wizard  
✅ **Created multi-AZ subnets** following best practices  
✅ **Configured Internet Gateway** and routing manually  
✅ **Set up route tables** with proper associations  
✅ **Implemented naming conventions** and consistent tagging  

### Key Concepts Mastered:
- CIDR notation and subnet sizing
- VPC component relationships
- Route table functionality and associations
- Public vs private subnet configuration
- Internet Gateway connectivity

### Skills Developed:
- Manual VPC creation and configuration
- CIDR planning and IP address management
- Route table creation and management
- Subnet association and configuration
- AWS CLI automation for networking

### Architecture Built:
- Production-ready VPC with 10.0.0.0/16 CIDR
- 6 subnets across 3 availability zones
- Proper public/private subnet separation
- Internet connectivity for public subnets
- Scalable design for future growth

## 🚀 Next Steps

**[Continue to Day 3: Security Groups Deep Dive →](Day3.md)**

Tomorrow you'll:
- 🔒 Master AWS security groups (stateful firewalls)
- 🏗️ Build layered security architectures
- 🎯 Configure web, app, and database tier security
- 🧪 Test security group rules and behavior
- 🔗 Implement security group chaining and references

**You're building real networking skills!** Each day builds upon the previous, and you're well on your way to understanding production AWS architectures. 🎉

---

**Questions or need help?** Review the [troubleshooting section](#-troubleshooting) or explore the [additional resources](#-additional-resources).

**Pro Tip:** The VPC and subnets you built today form the foundation for all future AWS resources. Understanding this architecture deeply will serve you throughout your cloud journey!