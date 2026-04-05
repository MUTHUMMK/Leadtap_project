## DevSecOps CI/CD Pipeline (SAST + DAST + SonarQube + OWASP ZAP)

### Overview

This project implements a complete DevSecOps CI/CD pipeline using GitHub Actions, SonarQube, and OWASP ZAP.

###  Architecture

![Click ==>> Create Testing & Deployment Server ](<DEPLOYMENT.md>)

GitHub Actions CI/CD
        |
        |—— SAST → SonarQube (code scan)
        |
        |—— SCA → npm audit
        |
        |—— Build Next.js app
        |
        |—— Deploy to server
        |
        |—— DAST → OWASP ZAP scan (live URL)

#### It automates:
```
 Code push from developer
 Static Application Security Testing (SAST)
 SonarQube Quality Gate validation
 Software Composition Analysis (SCA) for dependency vulnerability scanning
 Automated deployment via SSH
 Dynamic Application Security Testing (DAST)
 Security report generation
 ```



### Tech Stack

- GitHub Actions
- SonarQube
- OWASP ZAP
- Node.js / npm
- PM2
- AWS EC2
- SSH Key Authentication

## Pipeline Stages:

### SAST
SonarScanner runs static analysis and blocks pipeline on failure.

### SCA
SCA scans project dependencies (npm packages) to identify known vulnerabilities in third-party libraries.

### Build & Deploy
git pull origin main
npm install
npm run build
pm2 restart all

### DAST
OWASP ZAP scans deployed application for runtime vulnerabilities.

### Output
- SonarQube Report
- ZAP HTML Report

### Benefits
- Automated security pipeline
- Shift-left security
- Production-ready DevSecOps flow
