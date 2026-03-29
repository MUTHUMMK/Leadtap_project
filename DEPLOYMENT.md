# DevSecOps Architecture & CI/CD Workflow

Dev → Code → CI
               ↓
      SAST (block if fail)
               ↓
       Deploy → Staging
               ↓
           DAST (ZAP)
               ↓
            Approval
               ↓
            Production


## Overview

The application is deployed using a two-server architecture to separate production workload from security testing and CI/CD operations. This design follows DevSecOps principles by integrating security checks into the deployment pipeline.

---

## Infrastructure Setup

![Testing Server Setup](<./Documents/testing-server-setup.md>)

### 1. Security & CI/CD Server (t3.medium)

This server is dedicated to security testing and CI/CD execution.

#### Configuration:
- Instance Type: t3.medium
- OS: Ubuntu LTS
- Installed Tools:
  - SonarQube (running via Docker)
  - Sonar Scanner
  - OWASP ZAP (running via Docker)
  - Docker
  - Git
  - GitHub Actions Self-Hosted Runner

#### Purpose:
- Perform Static Application Security Testing (SAST)
- Perform Dynamic Application Security Testing (DAST)
- Execute CI/CD pipelines
- Generate security reports

#### Reasoning:
- Requires higher CPU and memory for scanning tools
- Isolated from production for security and performance
- Acts as centralized security and automation server

---

### 2. Production Server (t3.micro) 
![Production Server Setup](<./Documents/linux-deploy-setup.md>)

This server hosts the live application.

#### Configuration:
- Instance Type: t3.micro
- OS: Ubuntu LTS
- Installed Software:
  - Node.js (via NVM)
  - PM2 (process manager)
  - Nginx (reverse proxy)
  - Git

#### Security Hardening:
- Root login disabled
- SSH key-based authentication enabled
- Password authentication disabled
- UFW firewall configured (only required ports open)
- HTTPS enabled using Let's Encrypt
- Application runs as non-root user

#### Purpose:
- Host the Next.js application
- Serve traffic via HTTPS (xxx.com domain)

#### Reasoning:
- Lightweight instance for cost optimization
- Minimal software → reduced attack surface
- Dedicated to production traffic only

---

## CI/CD Pipeline Workflow

# Create the CI/CD Pipeline workflow 
![CI/CD Pipeline Setup](<./Documents/ci_cd_pipeline.md>)

A secure CI/CD pipeline is implemented using GitHub Actions with a self-hosted runner on the testing server.

### Pipeline Flow:

1. Developer pushes code to GitHub repository
2. GitHub Actions workflow is triggered
3. Pipeline runs on self-hosted runner (t3.medium)

### Step 1: Build Stage
- Install dependencies
- Build the Next.js application

### Step 2: SAST (Static Analysis)
- Run Sonar Scanner
- Analyze source code using SonarQube

### Step 3: Quality Gate Check
- If vulnerabilities are detected:
  - Pipeline fails
  - Deployment is blocked
- If passed:
  - Proceed to deployment

### Step 4: SCA (Software Composition Analysis)

- Run dependency vulnerability scan using:
  - `npm audit` 
- Checks performed:
  - Vulnerable npm packages
  - Known CVEs in dependencies
  - Outdated librariess

### Step 5: Deployment
- Application is deployed to production server (t3.micro) via SSH
- Commands executed:
  - Pull latest code
  - Install dependencies
  - Build application
  - Restart using PM2

### Step 5: DAST (Dynamic Analysis)
- OWASP ZAP is executed against live domain (https://yourdomain.com)
- Performs baseline security scan

### Step 7: Report Generation
- ZAP generates HTML report
- Report naming format:
  zap-report-<commit-id>.html 
- Stored on testing server for analysis

### Step 8: Pipeline Validation
- If vulnerabilities are detected:
  - Pipeline is marked as failed
- Ensures continuous security validation

---

## Security Strategy

### Shift Left Security
- SonarQube used before deployment
- Prevents vulnerable code from reaching production

### Shift Right Security
- OWASP ZAP used after deployment
- Identifies runtime vulnerabilities

### Isolation
- Security tools run on separate server
- Production environment remains clean and secure

### Least Privilege Principle
- Application does not run as root
- Restricted SSH access

### Defense in Depth
- Firewall (UFW)
- HTTPS (SSL/TLS)
- Reverse proxy (Nginx)
- Access control

---

## Trade-offs

- DAST performed on production instead of staging due to limited infrastructure
- ZAP baseline scan used for faster execution instead of full scan
- Resource constraints on free-tier instances

---

## Conclusion

This architecture provides:
- Secure and automated deployment
- Integrated SAS, SCA, DAST security checks
- Separation of concerns between environments
- Scalable and maintainable DevSecOps workflow

The pipeline ensures that no vulnerable code is deployed without validation, and continuous security testing is enforced after deployment.