
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

🔄 3. Update Packages
sudo su
sudo apt update -y

