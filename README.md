🚀 DevSecOps CI/CD Pipeline (SAST + DAST + SonarQube + OWASP ZAP)
📌 Overview

This project implements a complete DevSecOps CI/CD pipeline using GitHub Actions, SonarQube, and OWASP ZAP.

It automates:
![alt text](<mermaid-diagram.png>)

🧑‍💻 Code push from developer
🧪 Static Application Security Testing (SAST)
🔍 SonarQube Quality Gate validation
🚀 Automated deployment via SSH
🛡️ Dynamic Application Security Testing (DAST)
📊 Security report generation
🏗️ Architecture
⚙️ Tech Stack
GitHub Actions (CI/CD)
SonarQube (SAST)
OWASP ZAP (DAST)
Node.js / npm
PM2 Process Manager
Docker (optional)
AWS EC2 (Deployment servers)
SSH Key Authentication
🔐 Pipeline Stages
1️⃣ SAST - SonarQube Scan
Static code analysis
Detects vulnerabilities, bugs, code smells
Blocks pipeline if Quality Gate fails
2️⃣ Build Stage
npm install --no-audit --no-fund
npm run build
3️⃣ Deploy Stage (SSH)
git pull origin main
npm install
npm run build
pm2 restart all
Secure SSH key-based authentication
Runs on production EC2 server
4️⃣ DAST - OWASP ZAP Scan
Runs on deployed application
Detects runtime vulnerabilities:
SQL Injection
XSS
Security misconfigurations
5️⃣ Security Report

Generated artifacts:

zap-report.html
security-report.md

Stored in test server for auditing.

🔄 CI/CD Flow
Developer Push
      ↓
GitHub Actions Trigger
      ↓
SonarQube SAST Scan
      ↓
Quality Gate Check
      ↓ (PASS ONLY)
Deployment via SSH
      ↓
OWASP ZAP DAST Scan
      ↓
Security Report Storage
      ↓
Pipeline Success
🔐 Security Features
✔ SSH key authentication
✔ SonarQube Quality Gate enforcement
✔ SAST + DAST layered security
✔ No direct production access
✔ Automated vulnerability detection
⚙️ GitHub Actions Example
- name: SonarQube Scan
  run: sonar-scanner

- name: Quality Gate Check
  uses: sonarsource/sonarqube-quality-gate-action

- name: Deploy via SSH
  uses: appleboy/ssh-action@v0.1.10

- name: OWASP ZAP Scan
  run: |
    docker run --rm -v $(pwd):/zap/wrk \
    ghcr.io/zaproxy/zaproxy:stable \
    zap-baseline.py -t http://<app-url> -r zap-report.html
📊 Output Artifacts
SonarQube Dashboard Report
ZAP Security Report (HTML)
Markdown Security Summary
Deployment logs
🚀 Benefits
Fully automated CI/CD pipeline
Shift-left security approach
Early vulnerability detection
Production-ready DevSecOps workflow
Interview & enterprise-grade architecture
📌 Future Improvements
Kubernetes deployment (EKS)
ArgoCD GitOps integration
Slack / Email alerts
Auto rollback on failure
Centralized security dashboard
