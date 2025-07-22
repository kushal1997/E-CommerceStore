# Jenkins Setup on AWS EC2 with Pipeline Deployment

## Overview

This document explains the end-to-end setup of Jenkins on an AWS EC2 instance from scratch, including the issues encountered and their solutions. It also covers Docker, AWS CLI, and kubectl integration to deploy a MERN application to an EKS cluster.

---

## 1. EC2 Instance Setup

- **AMI Used:** Ubuntu 22.04 LTS
- **Instance Type:** t2.medium
- **Security Group Rules:**
  - Port 22: SSH (your IP)
  - Port 8080: Jenkins (0.0.0.0/0 for public access)
  - Port 80: HTTP (optional)

### Commands:

```bash
ssh -i <key.pem> ubuntu@<ec2-public-ip>
```

---

## 2. Jenkins Installation

### Steps:

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y
```

### Start Jenkins:

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

## 3. Initial Jenkins Setup

- Access via: `http://<ec2-ip>:8080`
- Retrieve password:
  ```bash
  sudo cat /var/lib/jenkins/secrets/initialAdminPassword
  ```
- Install suggested plugins
- Create admin user

---

## 4. Issues and Fixes

### ❌ Error: `docker: not found`

- **Cause**: Docker not installed
- **Solution**:
  ```bash
  sudo apt install ca-certificates curl gnupg lsb-release -y
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt update
  sudo apt install docker-ce docker-ce-cli containerd.io -y
  sudo usermod -aG docker jenkins
  sudo systemctl restart docker jenkins
  ```

### ❌ Error: `aws: not found`

- **Solution**:
  ```bash
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  sudo apt install unzip -y
  unzip awscliv2.zip
  sudo ./aws/install
  ```

  <br>

- Add to Jenkins PATH:
  ```bash
  sudo nano /etc/default/jenkins
  # Add:
  PATH="/usr/local/bin:/usr/bin:/bin"
  sudo systemctl restart jenkins
  ```

### ❌ Error: `kubectl: not found`

- **Solution**:
  ```bash
  LATEST_VERSION=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
  curl -LO https://dl.k8s.io/release/${LATEST_VERSION}/bin/linux/amd64/kubectl
  chmod +x kubectl
  sudo mv kubectl /usr/local/bin/
  ```

---

## 5. EKS Access Setup

### Create kubeconfig for Jenkins

```bash
aws eks update-kubeconfig --region ap-northeast-3 --name <cluster-name>

sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
```

---

## Conclusion

This document captures the full setup process of Jenkins on an EC2 instance including Docker, AWS CLI, and kubectl configuration. It also documents common issues faced during installation and pipeline deployment along with their fixes.

