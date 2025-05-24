# Jenkins Setup on AWS EC2 Ubuntu 24.04 Instance

This guide walks through the steps to set up Jenkins on an AWS EC2 Ubuntu 24.04 instance.

---

## 📌 1. Launch an EC2 Instance

- **AMI**: Ubuntu 24.04 LTS
- **Instance Type**: `t2.large`
- **Storage**: 30 GB (minimum)
- **Security Group**: Allow port `8080` (for Jenkins), `22` (for SSH)

---

## 🔗 2. Connect to the Instance

Use the following command:

```bash
ssh -i "your-key.pem" ubuntu@<your-ec2-public-ip>
