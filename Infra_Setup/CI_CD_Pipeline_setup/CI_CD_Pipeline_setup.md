## CI/CD Pipeline Setup Guide (GitHub Actions + EC2 + SonarQube)

This document explains how to set up a complete CI/CD pipeline using GitHub Actions, self-hosted runner, SonarQube, and EC2 deployment with SSH.

---

### 1. Prerequisites:

Before starting, ensure you have:

- GitHub repository
- AWS EC2 instances:
  - 1 Testing Server (SonarQube + Scanner + OWASP ZAP)
  - 2 Deployment Server (Application hosting)

---

### 2. Install GitHub Self-Hosted Runner (Testing Server):

### Step 1: Create Runner in GitHub

Go to:
```
GitHub Repository → Settings → Actions → Runners → New self-hosted runner
```

Select:
- OS: Linux
- Architecture: x64

Copy the commands provided by GitHub.

### Step 2: Install Runner on EC2

Run on testing server:

```bash
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64.tar.gz -L https://github.com/actions/runner/releases/latest/download/actions-runner-linux-x64.tar.gz

tar xzf actions-runner-linux-x64.tar.gz
```

### Step 3: Configure Runner

Replace values from GitHub page:

```bash
./config.sh --url https://github.com/your-repo --token YOUR_TOKEN
```

### Step 4: Start Runner

```bash
./run.sh
```

(Optional background)
```bash
./svc.sh install
./svc.sh start
```

---

#### 3. GitHub Secrets Configuration:

Go to:
```
GitHub Repo → Settings → Secrets and variables → Actions → New repository secret
```

Add the following secrets:

| Secret Name | Description |
|-------------|-------------|
| EC2_IP | Deployment server public IP |
| SSH_PRIVATE_KEY | Private key for EC2 SSH access |
| SONAR_HOST | http://sona_qube_server_ip:9000 |
| SONAR_TOKEN | SonarQube authentication token |
| DOMAIN | Application domain (e.g. https://Yourdomain.com) |
| HOST | Deployment username (e.g. ubuntu or devops) |
| PATH | Runner path |
| DEPLOY_PATH | Deployment path |

---

### 4. Example GitHub Actions CI/CD Pipeline:

Create file: 
```
.github/workflows/cicd.yml
```
![CI/CD Pipeline File](<./github-workflow-cicd.yml>)

---

### 5. Deployment Flow:

```
GitHub Push → Runner (Testing Server)
             → npm ci
             → SonarQube Scan
             → Approval/Success
             → SSH Deploy to EC2(Deployment Server)
             → Application Restart
```

---

### 6. Troubleshooting:

### Runner not connected
- Re-run config.sh

### SSH failed
- Check security group port 22
- Validate private key permissions

### Sonar not reachable
- Open port 9000 in EC2 firewall

---

### 7. Conclusion:

This pipeline provides:
- CI (Build + Test)
- SAST (SonarQube)
- CD (EC2 Deployment)
- DAST (OWASP ZAP)

