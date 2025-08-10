# Day 4: Network ACLs & Defense in Depth

**Welcome to Day 4!** Today you'll master Network Access Control Lists (NACLs), AWS's stateless firewall system. You'll learn how to combine NACLs with Security Groups for bulletproof defense-in-depth protection and understand when to use each security layer effectively.

## 📋 Table of Contents
- [Learning Objectives](#-learning-objectives)
- [Prerequisites Review](#-prerequisites-review)
- [NACLs vs Security Groups](#-nacls-vs-security-groups)
- [NACL Fundamentals](#-nacl-fundamentals)
- [Defense in Depth Strategy](#-defense-in-depth-strategy)
- [Hands-On Lab: Build NACL Architecture](#-hands-on-lab-build-nacl-architecture)
- [Rule Evaluation and Testing](#-rule-evaluation-and-testing)
- [Combined Security Implementation](#-combined-security-implementation)
- [Verification Steps](#-verification-steps)
- [Troubleshooting](#-troubleshooting)
- [Clean Up Resources](#-clean-up-resources)
- [Knowledge Check](#-knowledge-check)
- [Additional Resources](#-additional-resources)
- [Next Steps](#-next-steps)

## 🎯 Learning Objectives

By the end of Day 4, you will:
- [ ] Understand stateless vs stateful firewall differences
- [ ] Master NACL rule numbering and evaluation order
- [ ] Create custom NACLs for different subnet types
- [ ] Implement both allow and deny rules effectively
- [ ] Combine NACLs with Security Groups for layered security
- [ ] Test and troubleshoot NACL configurations
- [ ] Apply defense-in-depth security principles
- [ ] Know when to use NACLs vs Security Groups

## ⏱️ Time Required: 2-3 hours

## 📚 Prerequisites Review

Before starting Day 4, ensure you have:
- [ ] Completed Days 1-3 successfully
- [ ] Understanding of Security Groups from Day 3
- [ ] Knowledge of subnets and routing from Day 2
- [ ] AWS CLI configured and working
- [ ] Previous day's resources cleaned up

### Quick Environment Setup

```bash
# Create VPC for Day 4 testing
export VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Day4-NACL-VPC}]' \
    --query 'Vpc.VpcId' --output text)

# Create subnets for testing
export PUBLIC_SUBNET=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Day4-Public-Subnet}]' \
    --query 'Subnet.SubnetId' --output text)

export PRIVATE_SUBNET=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.11.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Day4-Private-Subnet}]' \
    --query 'Subnet.SubnetId' --output text)

echo "VPC ID: $VPC_ID"
echo "Public Subnet: $PUBLIC_SUBNET"
echo "Private Subnet: $PRIVATE_SUBNET"
```

## ⚔️ NACLs vs Security Groups

### Side-by-Side Comparison

| Feature | Network ACLs | Security Groups |
|---------|--------------|----------------|
| **Level** | Subnet level | Instance level |
| **State** | Stateless | Stateful |
| **Rules** | Allow AND Deny | Allow only |
| **Evaluation** | Order matters | Order doesn't matter |
| **Default** | Allow all | Deny all |
| **Response Traffic** | Must create explicit rules | Automatic |
| **Performance** | Evaluated first | Evaluated second |
| **Use Case** | Subnet-wide protection | Instance-specific protection |

### When to Use Each

**Use NACLs when:**
- 🎯 You need subnet-level protection
- 🚫 You need to explicitly block specific IPs/ports
- 🔒 You want an additional security layer
- 📊 You need to log denied traffic at subnet level
- 🌐 You have compliance requirements for network-level controls

**Use Security Groups when:**
- 🖥️ You need instance-specific rules
- 🔄 You want dynamic security (auto-allow responses)
- 🏗️ You're building application-tier security
- 🔗 You need to reference other security groups
- 📝 You want simpler rule management

### Defense in Depth Philosophy

```
Internet Traffic
       ↓
   [NACL] ← Subnet-level protection (stateless)
       ↓
[Security Group] ← Instance-level protection (stateful)
       ↓
    Instance
```

Both layers work together to provide comprehensive protection!

## 🛡️ NACL Fundamentals

### How NACLs Work

**Network ACLs are stateless firewalls** that control traffic at the subnet boundary. Every packet is evaluated independently, requiring explicit rules for both inbound AND outbound traffic.

Key characteristics:
- 🔢 **Rule Numbers:** Lower numbers have higher priority (100 comes before 200)
- 📊 **First Match Wins:** Evaluation stops at first matching rule
- 🚫 **Implicit Deny:** Traffic not explicitly allowed is denied
- 🔄 **Bidirectional Rules:** Must create both inbound and outbound rules
- 📈 **Subnet Level:** Applies to all instances in the subnet

### NACL Rule Components

| Component | Description | Example |
|-----------|-------------|---------|
| **Rule Number** | Priority (1-32766) | 100, 200, 300 |
| **Type** | Protocol | TCP, UDP, ICMP, ALL |
| **Port Range** | Specific ports | 80, 443, 1024-65535 |
| **Source/Destination** | IP address range | 0.0.0.0/0, 10.0.0.0/16 |
| **Action** | Allow or Deny | ALLOW, DENY |

### Default NACL Behavior

Every VPC comes with a **default NACL** that:
- ✅ Allows ALL inbound traffic
- ✅ Allows ALL outbound traffic
- 🏷️ Is associated with all subnets by default

**Default rules:**
```
INBOUND:
Rule # | Type | Port | Source | Action
100    | ALL  | ALL  | 0.0.0.0/0 | ALLOW
*      | ALL  | ALL  | 0.0.0.0/0 | DENY

OUTBOUND:
Rule # | Type | Port | Dest | Action
100    | ALL  | ALL  | 0.0.0.0/0 | ALLOW
*      | ALL  | ALL  | 0.0.0.0/0 | DENY
```

## 🏗️ Defense in Depth Strategy

### Multi-Layer Security Architecture

Today we'll implement a comprehensive security strategy:

```
Layer 1: NACL (Subnet Level)
├── Public Subnet NACL
│   ├── Allow HTTP/HTTPS from internet
│   ├── Allow SSH from admin networks only
│   └── Deny malicious IP ranges
└── Private Subnet NACL
    ├── Allow traffic from public subnet
    ├── Allow outbound for updates
    └── Deny direct internet access

Layer 2: Security Groups (Instance Level)
├── Web Server Security Group
├── Application Security Group
└── Database Security Group
```

### Security Group + NACL Example

**Scenario:** Web server in public subnet

1. **NACL Rules (Subnet Level):**
   - Allow HTTP (80) from 0.0.0.0/0
   - Allow ephemeral ports (1024-65535) outbound
   - Deny SSH from suspicious countries

2. **Security Group Rules (Instance Level):**
   - Allow HTTP (80) from ALB security group
   - Allow SSH (22) from admin IP only

**Result:** Double protection with different strengths!

## 🔬 Hands-On Lab: Build NACL Architecture

### Step 1: Analyze Default NACL

```bash
# Get default NACL for our VPC
export DEFAULT_NACL=$(aws ec2 describe-network-acls \
    --filters "Name=vpc-id,Values=$VPC_ID" "Name=default,Values=true" \
    --query 'NetworkAcls[0].NetworkAclId' --output text)

echo "Default NACL: $DEFAULT_NACL"

# View default NACL rules
aws ec2 describe-network-acls --network-acl-ids $DEFAULT_NACL \
    --query 'NetworkAcls[0].Entries' --output table
```

### Step 2: Create Custom Public NACL

⚠️ **Free Tier Note:** NACLs are always free - no charges for creating or using them!

```bash
# Create custom NACL for public subnets
export PUBLIC_NACL=$(aws ec2 create-network-acl \
    --vpc-id $VPC_ID \
    --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=Day4-Public-NACL},{Key=Type,Value=Public}]' \
    --query 'NetworkAcl.NetworkAclId' --output text)

echo "Public NACL: $PUBLIC_NACL"
```

### Step 3: Configure Public NACL Inbound Rules

```bash
# Rule 100: Allow HTTP from internet
aws ec2 create-network-acl-entry \
    --network-acl-id $PUBLIC_NACL \
    --rule-number 100 \
    --protocol tcp \
    --port-range From=80,To=80 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow

# Rule 110: Allow HTTPS from internet
aws ec2 create-network-acl-entry \
    --network-acl-id $PUBLIC_NACL \
    --rule-number 110 \
    --protocol tcp \
    --port-range From=443,To=443 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow

# Rule 120: Allow SSH from your IP (replace with your actual IP!)
aws ec2 create-network-acl-entry \
    --network-acl-id $PUBLIC_NACL \
    --rule-number 120 \
    --protocol tcp \
    --port-range From=22,To=22 \
    --cidr-block 203.0.113.0/32 \
    --rule-action allow

# Rule 130: Allow ephemeral ports (for return traffic)
aws ec2 create-network-acl-entry \
    --network-acl-id $PUBLIC_NACL \
    --rule-number 130 \
    --protocol tcp \
    --port-range From=1024,To=65535 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow

# Rule 200: Deny from suspicious IP range (example)
aws ec2 create-network-acl-entry \
    --network-acl-id $PUBLIC_NACL \
    --rule-number 200 \
    --protocol tcp \
    --port-range From=1,To=65535 \
    --cidr-block 198.51.100.0/24 \
    --rule-action deny
```

⚠️ **Important:** Replace `203.0.113.0/32` with your actual public IP address!

### Step 4: Configure Public NACL Outbound Rules

```bash
# Rule 100: Allow HTTP outbound
aws ec2 create-network-acl-entry \
    --network-acl-id $PUBLIC_NACL \
    --rule-number 100 \
    --protocol tcp \
    --port-range From=80,To=80 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow \
    --egress

# Rule 110: Allow HTTPS outbound
aws ec2 create-network-acl-entry \
    --network-acl-id $PUBLIC_NACL \
    --rule-number 110 \
    --protocol tcp \
    --port-range From=443,To=443 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow \
    --egress

# Rule 120: Allow ephemeral ports outbound (for responses)
aws ec2 create-network-acl-entry \
    --network-acl-id $PUBLIC_NACL \
    --rule-number 120 \
    --protocol tcp \
    --port-range From=1024,To=65535 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow \
    --egress

# Rule 130: Allow SSH outbound to private subnet
aws ec2 create-network-acl-entry \
    --network-acl-id $PUBLIC_NACL \
    --rule-number 130 \
    --protocol tcp \
    --port-range From=22,To=22 \
    --cidr-block 10.0.11.0/24 \
    --rule-action allow \
    --egress
```

### Step 5: Create Private NACL

```bash
# Create custom NACL for private subnets
export PRIVATE_NACL=$(aws ec2 create-network-acl \
    --vpc-id $VPC_ID \
    --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=Day4-Private-NACL},{Key=Type,Value=Private}]' \
    --query 'NetworkAcl.NetworkAclId' --output text)

echo "Private NACL: $PRIVATE_NACL"

# Rule 100: Allow SSH from public subnet
aws ec2 create-network-acl-entry \
    --network-acl-id $PRIVATE_NACL \
    --rule-number 100 \
    --protocol tcp \
    --port-range From=22,To=22 \
    --cidr-block 10.0.1.0/24 \
    --rule-action allow

# Rule 110: Allow HTTP from public subnet (for app communication)
aws ec2 create-network-acl-entry \
    --network-acl-id $PRIVATE_NACL \
    --rule-number 110 \
    --protocol tcp \
    --port-range From=8080,To=8080 \
    --cidr-block 10.0.1.0/24 \
    --rule-action allow

# Rule 120: Allow database ports from public subnet
aws ec2 create-network-acl-entry \
    --network-acl-id $PRIVATE_NACL \
    --rule-number 120 \
    --protocol tcp \
    --port-range From=3306,To=3306 \
    --cidr-block 10.0.1.0/24 \
    --rule-action allow

# Rule 130: Allow ephemeral ports (return traffic)
aws ec2 create-network-acl-entry \
    --network-acl-id $PRIVATE_NACL \
    --rule-number 130 \
    --protocol tcp \
    --port-range From=1024,To=65535 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow

# Outbound rules for private NACL
# Rule 100: Allow HTTPS for updates
aws ec2 create-network-acl-entry \
    --network-acl-id $PRIVATE_NACL \
    --rule-number 100 \
    --protocol tcp \
    --port-range From=443,To=443 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow \
    --egress

# Rule 110: Allow ephemeral ports outbound
aws ec2 create-network-acl-entry \
    --network-acl-id $PRIVATE_NACL \
    --rule-number 110 \
    --protocol tcp \
    --port-range From=1024,To=65535 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow \
    --egress

# Rule 120: Allow responses to public subnet
aws ec2 create-network-acl-entry \
    --network-acl-id $PRIVATE_NACL \
    --rule-number 120 \
    --protocol tcp \
    --port-range From=8080,To=8080 \
    --cidr-block 10.0.1.0/24 \
    --rule-action allow \
    --egress
```

### Step 6: Associate NACLs with Subnets

```bash
# Associate public NACL with public subnet
aws ec2 associate-network-acl \
    --network-acl-id $PUBLIC_NACL \
    --subnet-id $PUBLIC_SUBNET

# Associate private NACL with private subnet
aws ec2 associate-network-acl \
    --network-acl-id $PRIVATE_NACL \
    --subnet-id $PRIVATE_SUBNET

echo "NACL associations completed!"
```

## 📊 Rule Evaluation and Testing

### Understanding Rule Evaluation

NACL rules are evaluated in **numerical order** until a match is found:

```
Example Inbound Rules:
100: Allow HTTP from 0.0.0.0/0     ← Checked first
110: Allow HTTPS from 0.0.0.0/0    ← Checked second
200: Deny SSH from 0.0.0.0/0       ← Checked third
*  : Deny ALL                      ← Default deny
```

**Important:** If Rule 100 matches, rules 110, 200, and * are never evaluated!

### Rule Numbering Best Practices

| Range | Purpose | Example |
|-------|---------|---------|
| **100-199** | Common allow rules | HTTP, HTTPS, SSH |
| **200-299** | Specific allow rules | Database ports, custom apps |
| **300-399** | Deny rules | Block suspicious IPs |
| **400-499** | Emergency rules | Temporary blocks |
| **500+** | Future expansion | Reserved for growth |

### Testing NACL Configuration

```bash
# View complete NACL configuration
echo "=== PUBLIC NACL RULES ==="
aws ec2 describe-network-acls --network-acl-ids $PUBLIC_NACL \
    --query 'NetworkAcls[0].Entries[*].[RuleNumber,Protocol,RuleAction,CidrBlock,PortRange.From,PortRange.To,Egress]' \
    --output table

echo "=== PRIVATE NACL RULES ==="
aws ec2 describe-network-acls --network-acl-ids $PRIVATE_NACL \
    --query 'NetworkAcls[0].Entries[*].[RuleNumber,Protocol,RuleAction,CidrBlock,PortRange.From,PortRange.To,Egress]' \
    --output table
```

## 🧪 Combined Security Implementation

### Step 1: Create Security Groups for Testing

```bash
# Create web security group
export WEB_SG=$(aws ec2 create-security-group \
    --group-name Day4-Web-SG \
    --description "Web server security group for NACL testing" \
    --vpc-id $VPC_ID \
    --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=Day4-Web-SG}]' \
    --query 'GroupId' --output text)

# Allow HTTP from anywhere (Security Group layer)
aws ec2 authorize-security-group-ingress \
    --group-id $WEB_SG \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# Allow SSH from your IP (Security Group layer)
aws ec2 authorize-security-group-ingress \
    --group-id $WEB_SG \
    --protocol tcp \
    --port 22 \
    --cidr 203.0.113.0/32

echo "Web Security Group: $WEB_SG"
```

### Step 2: Demonstrate Layered Security

Let's create a test scenario to show how NACLs and Security Groups work together:

```bash
# Create test instance (Free Tier t2.micro)
aws ec2 run-instances \
    --image-id ami-0c02fb55956c7d316 \
    --instance-type t2.micro \
    --key-name Day4-TestKey \
    --security-group-ids $WEB_SG \
    --subnet-id $PUBLIC_SUBNET \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Day4-NACL-Test}]' \
    --user-data '#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>NACL + Security Group Test</h1>" > /var/www/html/index.html
echo "<p>Both layers working together!</p>" >> /var/www/html/index.html'
```

### Step 3: Security Layer Analysis

**Traffic Flow Analysis:**
```
Internet Request (HTTP Port 80)
       ↓
   [Public NACL]
   Rule 100: Allow HTTP from 0.0.0.0/0 ✅
       ↓
   [Web Security Group]
   Rule: Allow HTTP from 0.0.0.0/0 ✅
       ↓
   Web Server Instance ✅

Response Traffic
       ↓
   [Web Security Group]
   Stateful: Auto-allow response ✅
       ↓
   [Public NACL]
   Rule 120: Allow ephemeral ports outbound ✅
       ↓
   Internet ✅
```

### Step 4: Test Deny Rules

```bash
# Add a deny rule to test NACL behavior
aws ec2 create-network-acl-entry \
    --network-acl-id $PUBLIC_NACL \
    --rule-number 90 \
    --protocol tcp \
    --port-range From=80,To=80 \
    --cidr-block 203.0.113.0/32 \
    --rule-action deny

echo "Added deny rule for HTTP from your IP"
echo "This will block YOUR access to the web server while allowing others"
```

**Expected Behavior:**
- Your IP: ❌ Blocked by NACL rule 90 (deny)
- Other IPs: ✅ Allowed by NACL rule 100 (allow)

## ✅ Verification Steps

### Step 1: NACL Configuration Review

```bash
# Complete NACL summary
echo "=== NACL ASSOCIATIONS ==="
aws ec2 describe-network-acls --network-acl-ids $PUBLIC_NACL $PRIVATE_NACL \
    --query 'NetworkAcls[*].[Tags[?Key==`Name`].Value|[0],NetworkAclId,Associations[*].SubnetId]' \
    --output table

# Rule count verification
echo "=== RULE COUNTS ==="
aws ec2 describe-network-acls --network-acl-ids $PUBLIC_NACL \
    --query 'length(NetworkAcls[0].Entries)'
aws ec2 describe-network-acls --network-acl-ids $PRIVATE_NACL \
    --query 'length(NetworkAcls[0].Entries)'
```

### Step 2: Security Layer Testing

```bash
# Get instance public IP for testing
INSTANCE_IP=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=Day4-NACL-Test" "Name=instance-state-name,Values=running" \
    --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

echo "Instance IP: $INSTANCE_IP"

# Test HTTP access (may be blocked if you added deny rule)
curl -m 10 http://$INSTANCE_IP || echo "Access blocked - check NACL deny rule"
```

### Step 3: Rule Order Verification

```bash
# Verify rules are in correct order
aws ec2 describe-network-acls --network-acl-ids $PUBLIC_NACL \
    --query 'NetworkAcls[0].Entries[?Egress==`false`] | sort_by(@, &RuleNumber).[RuleNumber,RuleAction,Protocol,CidrBlock]' \
    --output table
```

Expected order:
1. Rule 90: DENY (if added)
2. Rule 100: ALLOW HTTP
3. Rule 110: ALLOW HTTPS
4. Rule 120: ALLOW SSH
5. Rule 130: ALLOW ephemeral ports

## 🐛 Troubleshooting

### Issue: Instance Can't Access Internet
**Symptoms:** Instance can't download updates or reach external sites

**Diagnosis:**
```bash
# Check NACL outbound rules
aws ec2 describe-network-acls --network-acl-ids $PUBLIC_NACL \
    --query 'NetworkAcls[0].Entries[?Egress==`true`].[RuleNumber,RuleAction,Protocol,PortRange]' \
    --output table
```

**Common Solutions:**
- Add outbound rules for HTTPS (443) and HTTP (80)
- Ensure ephemeral ports (1024-65535) are allowed outbound
- Check that Security Group allows outbound traffic

### Issue: SSH Access Blocked
**Symptoms:** SSH connection times out or is refused

**Diagnosis Steps:**
1. **Check NACL inbound rules:**
```bash
aws ec2 describe-network-acls --network-acl-ids $PUBLIC_NACL \
    --query 'NetworkAcls[0].Entries[?Egress==`false` && PortRange.From==`22`]' \
    --output table
```

2. **Verify your IP address:**
```bash
curl -s https://checkip.amazonaws.com
```

3. **Check rule order for conflicts:**
```bash
aws ec2 describe-network-acls --network-acl-ids $PUBLIC_NACL \
    --query 'NetworkAcls[0].Entries[?Egress==`false`] | sort_by(@, &RuleNumber)' \
    --output table
```

**Solutions:**
- Ensure SSH rule has lower number than any deny rules
- Verify your IP address is correct in the rule
- Check that Security Group also allows SSH

### Issue: NACL Rules Not Working
**Symptoms:** Traffic behavior doesn't match configured rules

**Common Causes:**
1. **Wrong subnet association:**
```bash
aws ec2 describe-network-acls --network-acl-ids $PUBLIC_NACL \
    --query 'NetworkAcls[0].Associations[*].SubnetId' --output table
```

2. **Rule number conflicts:**
```bash
# Check for duplicate rule numbers
aws ec2 describe-network-acls --network-acl-ids $PUBLIC_NACL \
    --query 'NetworkAcls[0].Entries[*].RuleNumber' --output table
```

3. **Missing return traffic rules (stateless behavior)**

### Issue: Can't Delete NACL
**Error:** `DependencyViolation: The network acl is in use`

**Solution:**
```bash
# First, associate subnets back to default NACL
aws ec2 associate-network-acl \
    --network-acl-id $DEFAULT_NACL \
    --subnet-id $PUBLIC_SUBNET

# Then delete custom NACL
aws ec2 delete-network-acl --network-acl-id $PUBLIC_NACL
```

## 🧹 Clean Up Resources

⚠️ **Important:** Clean up in proper order to avoid dependency violations.

### Step 1: Terminate Test Instance

```bash
# Get and terminate instance
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=Day4-NACL-Test" "Name=instance-state-name,Values=running" \
    --query 'Reservations[0].Instances[0].InstanceId' --output text)

if [ "$INSTANCE_ID" != "None" ]; then
    aws ec2 terminate-instances --instance-ids $INSTANCE_ID
    echo "Terminated instance: $INSTANCE_ID"
    
    # Wait for termination
    aws ec2 wait instance-terminated --instance-ids $INSTANCE_ID
fi
```

### Step 2: Reset Subnet Associations

```bash
# Associate subnets back to default NACL before deleting custom NACLs
aws ec2 associate-network-acl \
    --network-acl-id $DEFAULT_NACL \
    --subnet-id $PUBLIC_SUBNET

aws ec2 associate-network-acl \
    --network-acl-id $DEFAULT_NACL \
    --subnet-id $PRIVATE_SUBNET

echo "Subnets reassociated with default NACL"
```

### Step 3: Delete Custom NACLs

```bash
# Delete custom NACLs
aws ec2 delete-network-acl --network-acl-id $PUBLIC_NACL
aws ec2 delete-network-acl --network-acl-id $PRIVATE_NACL

echo "Custom NACLs deleted"
```

### Step 4: Delete Security Groups and VPC

```bash
# Delete security group
aws ec2 delete-security-group --group-id $WEB_SG

# Delete subnets
aws ec2 delete-subnet --subnet-id $PUBLIC_SUBNET
aws ec2 delete-subnet --subnet-id $PRIVATE_SUBNET

# Delete VPC
aws ec2 delete-vpc --vpc-id $VPC_ID

echo "Day 4 cleanup completed!"
```

### Verification

```bash
# Verify cleanup
aws ec2 describe-network-acls --filters "Name=tag:Name,Values=Day4-*" --output table
aws ec2 describe-security-groups --filters "Name=tag:Name,Values=Day4-*" --output table

# Should return empty results
```

## 🧠 Knowledge Check

Test your NACL mastery:

1. **What's the key difference between stateful and stateless firewalls?**
   <details>
   <summary>Answer</summary>
   Stateful firewalls (Security Groups) automatically allow return traffic for established connections. Stateless firewalls (NACLs) evaluate each packet independently and require explicit rules for both directions.
   </details>

2. **If you have NACL rules 100 (ALLOW HTTP) and 90 (DENY HTTP from your IP), which rule takes effect for your traffic?**
   <details>
   <summary>Answer</summary>
   Rule 90 (DENY) takes effect because it has a lower number and is evaluated first. Rule 100 is never reached for your IP address.
   </details>

3. **Why do you need to allow ephemeral ports (1024-65535) in NACL outbound rules?**
   <details>
   <summary>Answer</summary>
   Because NACLs are stateless, return traffic uses ephemeral ports that must be explicitly allowed. Without this rule, responses to requests would be blocked.
   </details>

4. **When would you use a NACL instead of just Security Groups?**
   <details>
   <summary>Answer</summary>
   When you need subnet-level protection, want to explicitly deny traffic, need compliance controls, or want to block traffic before it reaches instances (defense in depth).
   </details>

5. **How do NACLs and Security Groups work together in layered security?**
   <details>
   <summary>Answer</summary>
   NACLs provide the first layer of protection at the subnet level, while Security Groups provide the second layer at the instance level. Traffic must pass both layers to reach an instance.
   </details>

## 📚 Additional Resources

### AWS Documentation
- [Network ACLs User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [Security Groups vs NACLs](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html)
- [VPC Security Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)

### Network Security Concepts
- [Stateful vs Stateless Firewalls](https://www.cisco.com/c/en/us/products/security/firewalls/what-is-a-firewall.html)
- [Defense in Depth Strategy](https://csrc.nist.gov/glossary/term/defense_in_depth)
- [Network Segmentation Best Practices](https://www.sans.org/white-papers/36082/)

### Tools and Utilities
- [VPC Flow Logs for Traffic Analysis](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [AWS Config Rules for Compliance](https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config_use-managed-rules.html)
- [Network Testing Tools](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-troubleshooting.html)

### Video Resources
- [AWS Network Security Deep Dive](https://www.youtube.com/watch?v=hiKPPy584Mg)
- [VPC Security Best Practices](https://www.youtube.com/watch?v=VCsOl_nK8z4)

## 🎯 Day 4 Summary

Fantastic work! You've completed Day 4 and have:

✅ **Mastered Network ACLs** and stateless firewall behavior  
✅ **Understood rule evaluation** and numbering priorities  
✅ **Implemented defense in depth** with layered security  
✅ **Combined NACLs with Security Groups** for comprehensive protection  
✅ **Configured allow and deny rules** effectively  
✅ **Tested and validated** NACL configurations  

### Key Concepts Mastered:
- Stateless vs stateful firewall differences
- NACL rule evaluation order and priorities
- Defense in depth security architecture
- Combined Security Group and NACL strategies
- Rule numbering and best practices

### Skills Developed:
- Custom NACL creation and configuration
- Rule ordering and priority management
- Subnet-level security implementation
- Multi-layer security testing and validation
- Network troubleshooting and diagnosis

### Security Architecture Enhanced:
- Subnet-level protection with custom NACLs
- Instance-level protection with Security Groups
- Proper rule ordering and evaluation
- Allow and deny rule implementation
- Comprehensive defense in depth strategy

## 🚀 Next Steps

**[Continue to Day 5: NAT & Private Connectivity →](Day5.md)**

Tomorrow you'll:
- 🌐 Set up NAT instances for private subnet internet access
- 🏰 Build bastion host architecture for secure access
- 🔗 Configure SSH tunneling and port forwarding
- 💰 Implement cost-effective NAT solutions (Free Tier)
- 🔧 Master private subnet connectivity patterns

**You're becoming a network security expert!** Understanding both stateful and stateless firewalls puts you ahead of many cloud practitioners. 🎉

---

**Questions about NACLs or stateless firewalls?** Review the [troubleshooting section](#-troubleshooting) or explore the [additional resources](#-additional-resources).

**Security Insight:** Defense in depth isn't just about having multiple security layers - it's about having different *types* of security layers that complement each other's strengths and cover each other's weaknesses!