# AWS VPC Free Tier Learning Guide

A comprehensive 7-day self-paced learning guide for building AWS VPC infrastructure using **ONLY** AWS Free Tier resources. Master networking fundamentals, security configurations, and progress toward a basic landing zone architecture—all without spending a penny!

## 🎯 Learning Objectives

By the end of this 7-day journey, you will:

- ✅ Understand AWS networking fundamentals and VPC concepts
- ✅ Design and implement custom VPC architectures
- ✅ Configure security groups and Network ACLs for defense in depth
- ✅ Set up NAT instances and bastion hosts for secure connectivity
- ✅ Implement VPC monitoring and troubleshooting
- ✅ Build a basic AWS landing zone architecture
- ✅ Know how to stay within AWS Free Tier limits while learning

## 👥 Target Audience

This guide is perfect for:
- **Complete beginners** to AWS networking with zero prior experience
- **Students and professionals** with zero budget for cloud learning
- **Career changers** wanting hands-on AWS experience
- **Anyone** wanting to understand VPC architecture without financial risk

### Prerequisites
- ✅ Basic computer literacy and internet access
- ✅ Basic understanding of networking concepts (helpful but not required)
- ✅ Willingness to learn and experiment
- ❌ **NO AWS experience required**
- ❌ **NO credit card charges** (Free Tier only)

## 🆓 AWS Free Tier Overview

### What's Included Forever Free:
- VPC and networking components (subnets, route tables, security groups)
- 750 hours of t2.micro EC2 instances per month
- 15 GB of bandwidth out per month
- 5 Elastic IPs per region
- VPC Flow Logs to S3 (first 5GB free)

### ⚠️ What to Avoid:
- NAT Gateways ($45+/month) - we'll use NAT instances instead
- Load Balancers ($18+/month) - not needed for learning
- Large instance types beyond t2.micro
- Data transfer over 15GB/month

## 📅 7-Day Learning Schedule

| Day | Topic | Time | Key Skills |
|-----|-------|------|------------|
| **[Day 1](Day1.md)** | AWS Fundamentals & VPC Basics | 2-3 hours | Account setup, basic VPC creation, billing alerts |
| **[Day 2](Day2.md)** | Custom VPC & Subnet Design | 2-3 hours | CIDR design, public/private subnets, route tables |
| **[Day 3](Day3.md)** | Security Groups Deep Dive | 2-3 hours | Stateful firewalls, layered security, rule testing |
| **[Day 4](Day4.md)** | Network ACLs & Defense in Depth | 2-3 hours | Stateless firewalls, rule evaluation, combined security |
| **[Day 5](Day5.md)** | NAT & Private Connectivity | 3-4 hours | NAT instances, bastion hosts, SSH tunneling |
| **[Day 6](Day6.md)** | Advanced Networking & Monitoring | 2-3 hours | VPC Flow Logs, monitoring, endpoints, troubleshooting |
| **[Day 7](Day7.md)** | Landing Zone Architecture | 3-4 hours | Multi-VPC design, hub-and-spoke, documentation |

**Total Time Investment:** 16-22 hours over 7 days

## 🚀 Getting Started

### Step 1: AWS Account Setup
1. **Create AWS Account:** Go to [aws.amazon.com](https://aws.amazon.com) and sign up
2. **Verify Free Tier Status:** Check your account is eligible for 12 months of Free Tier
3. **Set Up Billing Alerts:** Critical for staying within free limits

### Step 2: Environment Preparation
1. **Install AWS CLI:** [Download instructions](https://aws.amazon.com/cli/)
2. **Choose Your Region:** We recommend `us-east-1` (N. Virginia) for maximum Free Tier compatibility
3. **Bookmark AWS Console:** You'll be using it extensively

### Step 3: Safety Measures
1. **Set Billing Alerts:** $1, $5, and $10 thresholds
2. **Create IAM User:** Never use root account for daily tasks
3. **Tag All Resources:** Use consistent tagging for cost tracking

## 💰 Cost Prevention Strategy

### Daily Cleanup Protocol
⚠️ **CRITICAL:** Clean up resources at the end of each day to avoid charges!

Each daily guide includes specific cleanup instructions, but here's the general order:
1. Terminate EC2 instances
2. Release Elastic IPs
3. Delete NAT instances
4. Remove VPC components (in reverse order of creation)

### Emergency Cleanup
If you're ever unsure about costs:
```bash
# Quick resource check
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]'
aws ec2 describe-addresses --query 'Addresses[*].[PublicIp,InstanceId,AllocationId]'
```

### Cost Monitoring Tools
- **AWS Budgets:** Set up $10 monthly budget with alerts
- **Billing Dashboard:** Check daily during learning period
- **Cost Explorer:** Review usage patterns weekly

## 📚 Daily Learning Structure

Each day follows this proven learning structure:

### 🎯 Learning Objectives
Clear, measurable goals for what you'll accomplish

### ⏱️ Time Estimate
Realistic time commitment (2-4 hours per day)

### 🔧 Hands-On Labs
Step-by-step practical exercises with real AWS resources

### ✅ Verification Steps
Commands and checks to confirm everything works correctly

### 🐛 Troubleshooting
Common issues and their solutions

### 🧹 Resource Cleanup
Mandatory cleanup to stay within Free Tier

### 📖 Additional Resources
Links to official docs, videos, and community resources

### 🧠 Knowledge Check
Quick quiz to reinforce learning

## 🔧 Required Tools

### AWS Console
- Primary interface for learning
- Visual feedback for configurations
- Easy resource management

### AWS CLI
- Command-line interface for automation
- Verification commands
- Script execution

### Text Editor
- Any editor for taking notes
- Optional: VS Code with AWS extensions

### Terminal/Command Prompt
- For running AWS CLI commands
- SSH connectivity to instances

## 📋 Progress Tracking

Track your progress with this checklist:

### Week 1 Progress
- [ ] Day 1: Completed AWS setup and first VPC
- [ ] Day 2: Built custom VPC with public/private subnets
- [ ] Day 3: Mastered security group configurations
- [ ] Day 4: Implemented Network ACLs
- [ ] Day 5: Set up NAT instance and bastion host
- [ ] Day 6: Configured monitoring and troubleshooting
- [ ] Day 7: Designed basic landing zone architecture

### Skills Acquired
- [ ] VPC architecture design
- [ ] Security group management
- [ ] Network troubleshooting
- [ ] Cost optimization awareness
- [ ] AWS CLI proficiency
- [ ] Infrastructure documentation

## 🆘 Getting Help

### Free Resources
- **AWS Documentation:** [docs.aws.amazon.com](https://docs.aws.amazon.com)
- **AWS Forums:** Community support for Free Tier users
- **Stack Overflow:** Tagged questions about AWS VPC
- **YouTube:** AWS official channel tutorials

### Community Support
- **Reddit:** r/aws, r/AWSCertifications
- **Discord:** AWS Community servers
- **LinkedIn:** AWS User Groups

### Troubleshooting Strategy
1. **Check AWS Service Health:** [status.aws.amazon.com](https://status.aws.amazon.com)
2. **Review CloudTrail Logs:** Understand what actions were taken
3. **Validate Free Tier Usage:** Ensure you haven't exceeded limits
4. **Search Error Messages:** Often others have solved similar issues

## 🎓 What's Next?

After completing this guide, you'll be ready for:

### Immediate Next Steps
- **AWS Solutions Architect Associate:** Industry-recognized certification
- **Advanced VPC Features:** Transit Gateways, VPC Endpoints, Direct Connect
- **Infrastructure as Code:** Terraform or CloudFormation
- **Advanced Security:** AWS Config, GuardDuty, Security Hub

### Career Paths
- **Cloud Solutions Architect**
- **DevOps Engineer**
- **Cloud Security Specialist**
- **Site Reliability Engineer**

### Advanced Learning
- **Multi-Account Strategy:** AWS Organizations and Control Tower
- **Hybrid Networking:** VPN and Direct Connect
- **Container Networking:** EKS and Fargate
- **Serverless Architectures:** Lambda and API Gateway

## ⚖️ Terms and Disclaimer

### AWS Free Tier Limitations
- Free Tier is valid for 12 months from account creation
- Usage limits reset monthly
- Some services have permanent free tiers
- Always verify current Free Tier terms on AWS website

### Learning Disclaimer
- This guide focuses on learning, not production deployments
- Always follow AWS best practices for production workloads
- Test configurations may not be suitable for production use
- Clean up resources to avoid unexpected charges

---

## 🚀 Ready to Start?

**[Begin Day 1: AWS Fundamentals & VPC Basics →](Day1.md)**

---

*Made with ❤️ for the AWS learning community. This guide is open source and contributions are welcome!*

## 📞 Support This Project

If this guide helps you land your first cloud job or advance your career:
- ⭐ Star this repository
- 🐛 Report issues or improvements
- 📢 Share with others learning AWS
- 💡 Contribute additional content

**Happy Learning! 🎉**