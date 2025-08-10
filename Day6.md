# Day 6: Advanced Networking & Monitoring

**Welcome to Day 6!** Today you'll master advanced VPC features and monitoring capabilities. You'll configure VPC Flow Logs, set up CloudWatch monitoring, implement VPC Endpoints for cost savings, and learn powerful network troubleshooting techniques that will make you a VPC expert.

## 📋 Table of Contents
- [Learning Objectives](#-learning-objectives)
- [Prerequisites Review](#-prerequisites-review)
- [VPC Flow Logs Deep Dive](#-vpc-flow-logs-deep-dive)
- [CloudWatch Network Monitoring](#-cloudwatch-network-monitoring)
- [VPC Endpoints for Cost Optimization](#-vpc-endpoints-for-cost-optimization)
- [DNS and DHCP Configuration](#-dns-and-dhcp-configuration)
- [Hands-On Lab: Monitoring Setup](#-hands-on-lab-monitoring-setup)
- [Network Troubleshooting Tools](#-network-troubleshooting-tools)
- [Performance Analysis](#-performance-analysis)
- [Verification Steps](#-verification-steps)
- [Troubleshooting](#-troubleshooting)
- [Clean Up Resources](#-clean-up-resources)
- [Knowledge Check](#-knowledge-check)
- [Additional Resources](#-additional-resources)
- [Next Steps](#-next-steps)

## 🎯 Learning Objectives

By the end of Day 6, you will:
- [ ] Configure VPC Flow Logs for traffic analysis
- [ ] Set up CloudWatch monitoring for network metrics
- [ ] Implement VPC Endpoints to reduce data transfer costs
- [ ] Configure DNS and DHCP options for optimal performance
- [ ] Analyze network traffic patterns and security events
- [ ] Use AWS network troubleshooting tools effectively
- [ ] Optimize network performance and costs
- [ ] Create comprehensive monitoring dashboards

## ⏱️ Time Required: 2-3 hours

## 📚 Prerequisites Review

Before starting Day 6, ensure you have:
- [ ] Completed Days 1-5 successfully
- [ ] Understanding of VPC components and routing
- [ ] Knowledge of Security Groups and NACLs
- [ ] Experience with CloudWatch basics
- [ ] Previous day's resources cleaned up
- [ ] S3 bucket for Flow Logs (we'll create one)

### Quick Environment Check

```bash
# Verify AWS CLI and check current region
aws configure get region
aws sts get-caller-identity

# Check for any existing VPCs from previous days
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=Day*" \
    --query 'Vpcs[*].[VpcId,Tags[?Key==`Name`].Value|[0]]' --output table
```

## 📊 VPC Flow Logs Deep Dive

### What are VPC Flow Logs?

**VPC Flow Logs** capture information about IP traffic going to and from network interfaces in your VPC. They provide visibility into network traffic patterns, security analysis, and troubleshooting capabilities.

### Flow Log Data Fields

| Field | Description | Example |
|-------|-------------|---------|
| **account-id** | AWS account ID | 123456789012 |
| **interface-id** | Network interface ID | eni-1235b8ca123456789 |
| **srcaddr** | Source IP address | 10.0.1.5 |
| **dstaddr** | Destination IP address | 10.0.2.10 |
| **srcport** | Source port | 49152 |
| **dstport** | Destination port | 80 |
| **protocol** | Protocol number | 6 (TCP), 17 (UDP) |
| **packets** | Number of packets | 25 |
| **bytes** | Number of bytes | 1500 |
| **start** | Start time | 1418530010 |
| **end** | End time | 1418530070 |
| **action** | ACCEPT or REJECT | ACCEPT |

### Flow Log Destinations

| Destination | Cost | Benefits | Use Case |
|-------------|------|----------|---------|
| **S3** | Storage costs only | Long-term retention, analysis | ✅ **Free Tier friendly** |
| **CloudWatch Logs** | $0.50/GB ingested | Real-time analysis, alarms | Limited for Free Tier |
| **Kinesis Data Firehose** | Processing costs | Real-time streaming | Advanced use cases |

For our Free Tier learning, we'll use **S3 destination**!

### Traffic Analysis Use Cases

- 🔍 **Security Analysis:** Identify unusual traffic patterns
- 🐛 **Troubleshooting:** Diagnose connectivity issues
- 📊 **Performance:** Analyze bandwidth usage
- 💰 **Cost Optimization:** Identify data transfer patterns
- 🛡️ **Compliance:** Meet audit requirements

## 📈 CloudWatch Network Monitoring

### Key Network Metrics

| Metric | Description | Threshold |
|--------|-------------|-----------|
| **NetworkIn** | Bytes received | Monitor spikes |
| **NetworkOut** | Bytes sent | Watch for unusual patterns |
| **NetworkPacketsIn** | Packets received | Baseline for normal traffic |
| **NetworkPacketsOut** | Packets sent | Detect packet floods |

### Custom Metrics for VPC

- **Connection Attempts:** Failed SSH/HTTP connections
- **Data Transfer Costs:** Track bandwidth usage
- **Security Events:** Blocked traffic in NACLs
- **Performance:** Latency and packet loss

### Alarms and Notifications

Set up alarms for:
- Unusual traffic spikes
- Security events (rejected connections)
- Cost thresholds (data transfer)
- Performance degradation

## 🔗 VPC Endpoints for Cost Optimization

### What are VPC Endpoints?

**VPC Endpoints** allow private connectivity to AWS services without internet gateway, NAT instance, or VPN connection. Traffic stays within AWS network, reducing costs and improving security.

### Types of VPC Endpoints

| Type | Services | Cost Model | Use Case |
|------|---------|------------|----------|
| **Gateway Endpoints** | S3, DynamoDB | ✅ **Free** | Bulk data access |
| **Interface Endpoints** | Most AWS services | $0.01/hour + data | Real-time access |

### Cost Benefits

**Without VPC Endpoint:**
```
Private Instance → NAT Instance → Internet → S3
Cost: Data transfer charges + NAT instance costs
```

**With VPC Endpoint:**
```
Private Instance → VPC Endpoint → S3
Cost: Free for Gateway Endpoints!
```

### Supported Services

**Gateway Endpoints (Free):**
- Amazon S3
- DynamoDB

**Interface Endpoints ($):**
- EC2, ECS, Lambda
- Systems Manager
- CloudWatch, CloudTrail
- And 100+ other services

## 🌐 DNS and DHCP Configuration

### VPC DNS Resolution

AWS VPC provides DNS resolution with:
- **DNS hostnames:** Enable for public IP DNS names
- **DNS resolution:** Enable for DNS queries
- **DNS server:** 169.254.169.253 (VPC+2 address)

### DHCP Options Sets

Configure network settings for EC2 instances:
- **Domain name:** Custom domain for instances
- **Domain name servers:** Custom DNS servers
- **NTP servers:** Network Time Protocol servers
- **NetBIOS servers:** Windows networking

### Route 53 Integration

**Private Hosted Zones** for internal DNS:
- Custom domains for internal services
- Service discovery for microservices
- Health checks and failover

## 🔬 Hands-On Lab: Monitoring Setup

### Step 1: Create Lab VPC

```bash
# Create VPC for Day 6 monitoring lab
export VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Day6-Monitoring-VPC}]' \
    --query 'Vpc.VpcId' --output text)

# Enable DNS hostnames and resolution
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-support

# Create subnets
export PUBLIC_SUBNET=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Day6-Public-Subnet}]' \
    --query 'Subnet.SubnetId' --output text)

export PRIVATE_SUBNET=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.11.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Day6-Private-Subnet}]' \
    --query 'Subnet.SubnetId' --output text)

# Configure Internet Gateway and routing
export IGW_ID=$(aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Day6-IGW}]' \
    --query 'InternetGateway.InternetGatewayId' --output text)

aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID

echo "VPC ID: $VPC_ID"
echo "Public Subnet: $PUBLIC_SUBNET"
echo "Private Subnet: $PRIVATE_SUBNET"
```

### Step 2: Create S3 Bucket for Flow Logs

⚠️ **Free Tier Note:** S3 storage is free for first 5GB per month.

```bash
# Create unique S3 bucket name
export BUCKET_NAME="day6-vpc-flow-logs-$(date +%s)"

# Create S3 bucket
aws s3 mb s3://$BUCKET_NAME --region us-east-1

# Create bucket policy for VPC Flow Logs
cat > /tmp/flow-logs-policy.json << EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AWSLogDeliveryWrite",
            "Effect": "Allow",
            "Principal": {
                "Service": "delivery.logs.amazonaws.com"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::$BUCKET_NAME/*"
        },
        {
            "Sid": "AWSLogDeliveryAclCheck",
            "Effect": "Allow",
            "Principal": {
                "Service": "delivery.logs.amazonaws.com"
            },
            "Action": "s3:GetBucketAcl",
            "Resource": "arn:aws:s3:::$BUCKET_NAME"
        }
    ]
}
EOF

# Apply bucket policy
aws s3api put-bucket-policy --bucket $BUCKET_NAME --policy file:///tmp/flow-logs-policy.json

echo "S3 bucket created: $BUCKET_NAME"
```

### Step 3: Create VPC Flow Logs

```bash
# Create Flow Logs for the entire VPC
export FLOW_LOG_ID=$(aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids $VPC_ID \
    --traffic-type ALL \
    --log-destination-type s3 \
    --log-destination "arn:aws:s3:::$BUCKET_NAME" \
    --tag-specifications 'ResourceType=vpc-flow-log,Tags=[{Key=Name,Value=Day6-VPC-Flow-Logs}]' \
    --query 'FlowLogIds[0]' --output text)

echo "Flow Log ID: $FLOW_LOG_ID"

# Verify Flow Log creation
aws ec2 describe-flow-logs --flow-log-ids $FLOW_LOG_ID --output table
```

### Step 4: Create VPC Endpoint for S3

```bash
# Get route table ID (we'll create one)
export ROUTE_TABLE_ID=$(aws ec2 create-route-table \
    --vpc-id $VPC_ID \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Day6-RT}]' \
    --query 'RouteTable.RouteTableId' --output text)

# Create S3 VPC Endpoint (Gateway type - FREE!)
export VPC_ENDPOINT_ID=$(aws ec2 create-vpc-endpoint \
    --vpc-id $VPC_ID \
    --service-name com.amazonaws.us-east-1.s3 \
    --route-table-ids $ROUTE_TABLE_ID \
    --policy-document '{"Statement":[{"Effect":"Allow","Principal":"*","Action":"s3:*","Resource":"*"}]}' \
    --query 'VpcEndpoint.VpcEndpointId' --output text)

echo "VPC Endpoint ID: $VPC_ENDPOINT_ID"

# Verify endpoint creation
aws ec2 describe-vpc-endpoints --vpc-endpoint-ids $VPC_ENDPOINT_ID --output table
```

### Step 5: Launch Test Instances

```bash
# Create security group for testing
export TEST_SG=$(aws ec2 create-security-group \
    --group-name Day6-Test-SG \
    --description "Security group for Day 6 monitoring tests" \
    --vpc-id $VPC_ID \
    --query 'GroupId' --output text)

# Allow SSH and HTTP
aws ec2 authorize-security-group-ingress --group-id $TEST_SG --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $TEST_SG --protocol tcp --port 80 --cidr 0.0.0.0/0

# Launch test instance in public subnet
export TEST_INSTANCE_ID=$(aws ec2 run-instances \
    --image-id ami-0c02fb55956c7d316 \
    --instance-type t2.micro \
    --key-name Day6-Test-Key \
    --security-group-ids $TEST_SG \
    --subnet-id $PUBLIC_SUBNET \
    --associate-public-ip-address \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Day6-Test-Instance}]' \
    --user-data '#!/bin/bash
yum update -y
yum install -y httpd awscli
systemctl start httpd
systemctl enable httpd
echo "<h1>Day 6 Monitoring Test</h1>" > /var/www/html/index.html

# Generate some S3 traffic for Flow Logs
aws s3 ls s3://'$BUCKET_NAME' > /tmp/s3-test.log
echo "S3 test completed" >> /var/log/monitoring-test.log' \
    --query 'Instances[0].InstanceId' --output text)

echo "Test Instance ID: $TEST_INSTANCE_ID"
```

### Step 6: Configure CloudWatch Monitoring

```bash
# Create CloudWatch alarm for network traffic
aws cloudwatch put-metric-alarm \
    --alarm-name "Day6-High-Network-Out" \
    --alarm-description "Alert when network out exceeds threshold" \
    --metric-name NetworkOut \
    --namespace AWS/EC2 \
    --statistic Sum \
    --period 300 \
    --threshold 1000000000 \
    --comparison-operator GreaterThanThreshold \
    --dimensions Name=InstanceId,Value=$TEST_INSTANCE_ID \
    --evaluation-periods 2

echo "CloudWatch alarm created for network monitoring"

# Create custom metric for connection tracking
aws logs create-log-group --log-group-name "/aws/vpc/day6-connections" || true
```

## 🛠️ Network Troubleshooting Tools

### 1. VPC Reachability Analyzer

**Purpose:** Analyze network paths between source and destination

```bash
# Create reachability analysis (requires instances to be running)
aws ec2 wait instance-running --instance-ids $TEST_INSTANCE_ID

# Get network interface ID
export ENI_ID=$(aws ec2 describe-instances --instance-ids $TEST_INSTANCE_ID \
    --query 'Reservations[0].Instances[0].NetworkInterfaces[0].NetworkInterfaceId' --output text)

echo "Network Interface ID: $ENI_ID"
echo "Use this ENI ID in VPC Reachability Analyzer in the console"
```

### 2. VPC Flow Logs Analysis

```bash
# Create analysis script for Flow Logs
cat > /tmp/analyze-flow-logs.py << 'EOF'
#!/usr/bin/env python3
import boto3
import gzip
from collections import defaultdict

def analyze_flow_logs(bucket_name):
    s3 = boto3.client('s3')
    
    # List Flow Log files
    response = s3.list_objects_v2(Bucket=bucket_name)
    
    if 'Contents' not in response:
        print("No Flow Log files found yet. They may take a few minutes to appear.")
        return
    
    traffic_stats = defaultdict(int)
    
    for obj in response['Contents']:
        if obj['Key'].endswith('.gz'):
            # Download and analyze file
            s3.download_file(bucket_name, obj['Key'], '/tmp/flow_log.gz')
            
            with gzip.open('/tmp/flow_log.gz', 'rt') as f:
                for line in f:
                    fields = line.strip().split(' ')
                    if len(fields) >= 13:
                        action = fields[12]
                        protocol = fields[7]
                        traffic_stats[f"{action}_{protocol}"] += 1
    
    print("Traffic Analysis:")
    for key, count in traffic_stats.items():
        print(f"  {key}: {count}")

if __name__ == "__main__":
    import sys
    if len(sys.argv) > 1:
        analyze_flow_logs(sys.argv[1])
    else:
        print("Usage: python3 analyze-flow-logs.py <bucket-name>")
EOF

chmod +x /tmp/analyze-flow-logs.py
```

### 3. Network Performance Testing

```bash
# Create network test script
cat > /tmp/network-test.sh << 'EOF'
#!/bin/bash

echo "=== Network Performance Test ==="

# Get instance public IP
INSTANCE_IP=$(aws ec2 describe-instances --instance-ids $TEST_INSTANCE_ID \
    --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

echo "Testing connectivity to: $INSTANCE_IP"

# Ping test
echo "1. Ping Test:"
ping -c 5 $INSTANCE_IP

# HTTP response time test
echo "2. HTTP Response Time:"
curl -w "@-" -o /dev/null -s "http://$INSTANCE_IP" << 'CURL_FORMAT'
     time_namelookup:  %{time_namelookup}\n
        time_connect:  %{time_connect}\n
     time_appconnect:  %{time_appconnect}\n
    time_pretransfer:  %{time_pretransfer}\n
       time_redirect:  %{time_redirect}\n
  time_starttransfer:  %{time_starttransfer}\n
                     ----------\n
          time_total:  %{time_total}\n
CURL_FORMAT

# Traceroute
echo "3. Traceroute:"
traceroute $INSTANCE_IP 2>/dev/null || echo "Traceroute not available"

echo "Network test completed!"
EOF

chmod +x /tmp/network-test.sh
```

### 4. Security Analysis Tools

```bash
# Create security analysis script
cat > /tmp/security-analysis.sh << 'EOF'
#!/bin/bash

echo "=== VPC Security Analysis ==="

# Check for overly permissive security groups
echo "1. Security Groups allowing SSH from anywhere:"
aws ec2 describe-security-groups \
    --query 'SecurityGroups[?IpPermissions[?IpProtocol==`tcp` && FromPort==`22` && IpRanges[?CidrIp==`0.0.0.0/0`]]].[GroupName,GroupId]' \
    --output table

# Check for default security groups in use
echo "2. Instances using default security groups:"
aws ec2 describe-instances \
    --query 'Reservations[*].Instances[?SecurityGroups[?GroupName==`default`]].[InstanceId,Tags[?Key==`Name`].Value|[0]]' \
    --output table

# Check NACLs for any deny rules
echo "3. Network ACLs with deny rules:"
aws ec2 describe-network-acls \
    --query 'NetworkAcls[?Entries[?RuleAction==`deny`]].[NetworkAclId,Tags[?Key==`Name`].Value|[0]]' \
    --output table

echo "Security analysis completed!"
EOF

chmod +x /tmp/security-analysis.sh
```

## 📊 Performance Analysis

### Step 1: Generate Test Traffic

```bash
# Wait for instance to be ready
aws ec2 wait instance-running --instance-ids $TEST_INSTANCE_ID

# Get instance IP
INSTANCE_IP=$(aws ec2 describe-instances --instance-ids $TEST_INSTANCE_ID \
    --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

echo "Instance IP: $INSTANCE_IP"

# Generate HTTP traffic for analysis
for i in {1..50}; do
    curl -s http://$INSTANCE_IP > /dev/null
    echo "Request $i completed"
    sleep 1
done

echo "Traffic generation completed. Flow Logs will appear in S3 within 5-10 minutes."
```

### Step 2: Monitor CloudWatch Metrics

```bash
# Get current network metrics
aws cloudwatch get-metric-statistics \
    --namespace AWS/EC2 \
    --metric-name NetworkOut \
    --dimensions Name=InstanceId,Value=$TEST_INSTANCE_ID \
    --start-time $(date -u -d '10 minutes ago' +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period 300 \
    --statistics Sum \
    --output table

echo "Check CloudWatch console for detailed metrics visualization"
```

### Step 3: Analyze VPC Endpoint Usage

```bash
# Test S3 access through VPC endpoint
ssh -i ~/.ssh/Day6-Test-Key.pem ec2-user@$INSTANCE_IP << 'EOF'
# This traffic should go through VPC endpoint (no NAT charges)
aws s3 ls s3://day6-vpc-flow-logs-* 
echo "S3 access through VPC endpoint completed"
EOF

# Check VPC endpoint metrics
aws ec2 describe-vpc-endpoints --vpc-endpoint-ids $VPC_ENDPOINT_ID \
    --query 'VpcEndpoints[0].[VpcEndpointId,State,ServiceName,CreationTimestamp]' \
    --output table
```

## ✅ Verification Steps

### Step 1: Flow Logs Validation

```bash
echo "=== VPC Flow Logs Verification ==="

# Check Flow Log status
aws ec2 describe-flow-logs --flow-log-ids $FLOW_LOG_ID \
    --query 'FlowLogs[0].[FlowLogId,FlowLogStatus,DeliverLogsStatus]' \
    --output table

# Wait a few minutes, then check S3 for log files
sleep 60
aws s3 ls s3://$BUCKET_NAME/ --recursive

echo "Flow Logs should appear in S3 within 5-10 minutes of traffic generation"
```

### Step 2: VPC Endpoint Verification

```bash
echo "=== VPC Endpoint Verification ==="

# Check endpoint configuration
aws ec2 describe-vpc-endpoints --vpc-endpoint-ids $VPC_ENDPOINT_ID \
    --query 'VpcEndpoints[0].[VpcEndpointId,State,ServiceName,RouteTableIds]' \
    --output table

# Verify route table has endpoint route
aws ec2 describe-route-tables --route-table-ids $ROUTE_TABLE_ID \
    --query 'RouteTables[0].Routes[?GatewayId==`'$VPC_ENDPOINT_ID'`]' \
    --output table
```

### Step 3: Monitoring Setup Verification

```bash
echo "=== Monitoring Setup Verification ==="

# Check CloudWatch alarm
aws cloudwatch describe-alarms --alarm-names "Day6-High-Network-Out" \
    --query 'MetricAlarms[0].[AlarmName,StateValue,StateReason]' \
    --output table

# Verify log group
aws logs describe-log-groups --log-group-name-prefix "/aws/vpc/day6" \
    --query 'logGroups[*].[logGroupName,storedBytes]' \
    --output table
```

### Step 4: Network Performance Check

```bash
echo "=== Network Performance Verification ==="

# Run network tests
/tmp/network-test.sh

# Run security analysis
/tmp/security-analysis.sh

echo "All verification steps completed!"
```

## 🐛 Troubleshooting

### Issue: Flow Logs Not Appearing in S3

**Symptoms:** S3 bucket remains empty after generating traffic

**Diagnosis Steps:**
1. **Check Flow Log status:**
```bash
aws ec2 describe-flow-logs --flow-log-ids $FLOW_LOG_ID \
    --query 'FlowLogs[0].DeliverLogsErrorMessage' --output text
```

2. **Verify S3 bucket policy:**
```bash
aws s3api get-bucket-policy --bucket $BUCKET_NAME
```

3. **Check IAM permissions:**
```bash
aws iam get-role --role-name flowlogsRole 2>/dev/null || echo "Flow logs role may be missing"
```

**Solutions:**
- Wait 10-15 minutes (Flow Logs have delay)
- Verify bucket policy allows delivery.logs.amazonaws.com
- Ensure bucket is in same region as VPC
- Check that traffic is actually flowing through VPC

### Issue: VPC Endpoint Not Working

**Symptoms:** S3 traffic still going through internet gateway

**Diagnosis:**
```bash
# Check endpoint policy
aws ec2 describe-vpc-endpoints --vpc-endpoint-ids $VPC_ENDPOINT_ID \
    --query 'VpcEndpoints[0].PolicyDocument' --output text

# Verify route table association
aws ec2 describe-route-tables --route-table-ids $ROUTE_TABLE_ID \
    --query 'RouteTables[0].Routes' --output table
```

**Solutions:**
- Ensure route table is associated with correct subnets
- Check endpoint policy allows required S3 actions
- Verify DNS resolution is enabled in VPC
- Test with `nslookup s3.amazonaws.com` from instance

### Issue: CloudWatch Metrics Missing

**Symptoms:** No network metrics appearing in CloudWatch

**Diagnosis:**
```bash
# Check instance monitoring
aws ec2 describe-instances --instance-ids $TEST_INSTANCE_ID \
    --query 'Reservations[0].Instances[0].Monitoring.State' --output text

# Verify CloudWatch agent (if needed)
ssh -i ~/.ssh/Day6-Test-Key.pem ec2-user@$INSTANCE_IP \
    "ps aux | grep cloudwatch"
```

**Solutions:**
- Wait 5-15 minutes for metrics to appear
- Verify instance is generating network traffic
- Check that detailed monitoring is enabled if needed
- Ensure instance has CloudWatch permissions

### Issue: Network Performance Problems

**Symptoms:** High latency or packet loss

**Common Causes:**
- Security group or NACL blocking traffic
- Instance type limitations (t2.micro has limited network)
- Network congestion or AWS service issues

**Diagnosis Tools:**
```bash
# Check for packet loss
ping -c 100 $INSTANCE_IP | grep "packet loss"

# Analyze MTU issues
ping -M do -s 1472 $INSTANCE_IP

# Check AWS service health
curl -s https://status.aws.amazon.com/
```

## 🧹 Clean Up Resources

⚠️ **Important:** Clean up monitoring resources to avoid ongoing charges.

### Step 1: Delete CloudWatch Resources

```bash
# Delete CloudWatch alarm
aws cloudwatch delete-alarms --alarm-names "Day6-High-Network-Out"

# Delete log group
aws logs delete-log-group --log-group-name "/aws/vpc/day6-connections" || true

echo "CloudWatch resources deleted"
```

### Step 2: Delete VPC Flow Logs

```bash
# Delete Flow Logs
aws ec2 delete-flow-logs --flow-log-ids $FLOW_LOG_ID

echo "VPC Flow Logs deleted"
```

### Step 3: Clean Up S3 Bucket

```bash
# Empty S3 bucket
aws s3 rm s3://$BUCKET_NAME --recursive

# Delete S3 bucket
aws s3 rb s3://$BUCKET_NAME

echo "S3 bucket and Flow Log data deleted"
```

### Step 4: Delete VPC Resources

```bash
# Terminate instance
aws ec2 terminate-instances --instance-ids $TEST_INSTANCE_ID

# Wait for termination
aws ec2 wait instance-terminated --instance-ids $TEST_INSTANCE_ID

# Delete VPC endpoint
aws ec2 delete-vpc-endpoint --vpc-endpoint-ids $VPC_ENDPOINT_ID

# Delete security group
aws ec2 delete-security-group --group-id $TEST_SG

# Delete route table
aws ec2 delete-route-table --route-table-id $ROUTE_TABLE_ID

# Delete IGW
aws ec2 detach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID

# Delete subnets
aws ec2 delete-subnet --subnet-id $PUBLIC_SUBNET
aws ec2 delete-subnet --subnet-id $PRIVATE_SUBNET

# Delete VPC
aws ec2 delete-vpc --vpc-id $VPC_ID

echo "Day 6 cleanup completed!"
```

### Verification

```bash
# Verify cleanup
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=Day6-*" --output table
aws s3 ls | grep day6-vpc-flow-logs || echo "No S3 buckets found"
aws cloudwatch describe-alarms --alarm-names "Day6-High-Network-Out" 2>/dev/null || echo "Alarm deleted"

echo "All Day 6 resources have been cleaned up!"
```

## 🧠 Knowledge Check

Test your advanced networking knowledge:

1. **What's the difference between Gateway and Interface VPC Endpoints?**
   <details>
   <summary>Answer</summary>
   Gateway Endpoints (S3, DynamoDB) are free and use route table entries. Interface Endpoints cost $0.01/hour and use ENIs with private IPs for other AWS services.
   </details>

2. **Why are VPC Flow Logs useful for security analysis?**
   <details>
   <summary>Answer</summary>
   Flow Logs show all network traffic including rejected connections, source/destination IPs, and ports. This helps identify attack patterns, unauthorized access attempts, and unusual traffic flows.
   </details>

3. **How can VPC Endpoints reduce costs?**
   <details>
   <summary>Answer</summary>
   VPC Endpoints eliminate data transfer charges for AWS service access from private subnets by keeping traffic within AWS network instead of routing through NAT Gateway/Instance and internet.
   </details>

4. **What CloudWatch metrics would indicate a potential DDoS attack?**
   <details>
   <summary>Answer</summary>
   Sudden spikes in NetworkPacketsIn, high NetworkIn bytes, increased CPU utilization, and many connections from diverse source IPs in VPC Flow Logs.
   </details>

5. **How long does it typically take for VPC Flow Logs to appear in S3?**
   <details>
   <summary>Answer</summary>
   VPC Flow Logs typically appear in S3 within 5-15 minutes after traffic occurs, depending on volume and aggregation intervals.
   </details>

## 📚 Additional Resources

### AWS Documentation
- [VPC Flow Logs User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [VPC Endpoints Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html)
- [CloudWatch VPC Metrics](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/viewing_metrics_with_cloudwatch.html)

### Monitoring and Analysis Tools
- [VPC Flow Logs Analysis](https://aws.amazon.com/blogs/networking-and-content-delivery/analyze-vpc-flow-logs-with-amazon-elasticsearch-service/)
- [Network Performance Monitoring](https://docs.aws.amazon.com/vpc/latest/userguide/monitoring-overview.html)
- [AWS X-Ray for Application Tracing](https://aws.amazon.com/xray/)

### Security and Compliance
- [VPC Security Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)
- [Network Monitoring for Compliance](https://aws.amazon.com/compliance/shared-responsibility-model/)
- [GuardDuty VPC Flow Logs](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_data-sources.html)

### Cost Optimization
- [VPC Cost Optimization](https://aws.amazon.com/blogs/networking-and-content-delivery/optimize-network-performance-and-cost-with-vpc-endpoints/)
- [Data Transfer Cost Analysis](https://aws.amazon.com/blogs/architecture/overview-of-data-transfer-costs-for-common-architectures/)

## 🎯 Day 6 Summary

Outstanding achievement! You've completed Day 6 and have:

✅ **Mastered VPC Flow Logs** for comprehensive traffic analysis  
✅ **Configured CloudWatch monitoring** for network performance tracking  
✅ **Implemented VPC Endpoints** for cost-effective AWS service access  
✅ **Set up advanced monitoring** with alarms and custom metrics  
✅ **Built network troubleshooting** capabilities and analysis tools  
✅ **Optimized costs** with Gateway Endpoints and monitoring strategies  

### Key Concepts Mastered:
- VPC Flow Logs configuration and analysis
- CloudWatch network monitoring and alerting
- VPC Endpoints for cost optimization
- Network troubleshooting methodologies
- Performance analysis and optimization

### Skills Developed:
- Traffic pattern analysis and security monitoring
- Cost optimization through VPC Endpoints
- CloudWatch alarm and dashboard creation
- Network performance testing and analysis
- Automated monitoring and alerting setup

### Monitoring Architecture Built:
- Comprehensive VPC Flow Logs to S3
- Real-time CloudWatch monitoring and alarms
- Cost-saving VPC Endpoints for AWS services
- Network performance testing suite
- Security analysis and compliance tools

### Cost Optimization Achieved:
- **Free VPC Flow Logs** to S3 (vs paid CloudWatch Logs)
- **Free Gateway Endpoints** for S3/DynamoDB access
- **Eliminated data transfer charges** for AWS service access
- **Proactive monitoring** to prevent cost overruns

## 🚀 Next Steps

**[Continue to Day 7: Landing Zone Architecture Assembly →](Day7.md)**

Tomorrow you'll:
- 🏗️ Design a complete multi-VPC landing zone architecture
- 🌐 Implement hub-and-spoke network topology
- 📋 Create comprehensive documentation and diagrams
- 🔧 Build Infrastructure as Code templates (optional)
- 🎯 Plan next steps for continued AWS learning

**You're now a VPC monitoring and optimization expert!** The skills you've learned today are essential for managing production AWS environments efficiently and cost-effectively. 🎉

---

**Questions about monitoring or VPC optimization?** Check the [troubleshooting section](#-troubleshooting) or explore the [additional resources](#-additional-resources).

**Pro Insight:** The monitoring and cost optimization techniques you learned today are critical for production environments. Many companies spend thousands of dollars monthly on unnecessary data transfer and inefficient monitoring - you now know how to avoid these costs!