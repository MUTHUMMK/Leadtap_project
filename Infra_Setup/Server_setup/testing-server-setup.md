## Testing Server Setup (DevSecOps Infrastructure)

This document describes the setup of the Security Testing Server (t3.medium) used for:
-  SAST (SonarQube + Sonar Scanner)
-  DAST (OWASP ZAP)
-  Security analysis in CI/CD pipeline

---

### 1️ Server Requirements:

### EC2 Instance
- Instance Type: t3.medium
- OS: Ubuntu 22.04
- Ports Open:
  - 9000 (SonarQube)
  - 22 (SSH)
  - 80/443 (Optional)
  - 8080 (OWASP ZAP)

---

### 2️ Install SonarQube (Docker):

```bash
docker run -d   -p 9000:9000   --name sonarqube   sonarqube:9.9-community
```

### Access
http://<server-ip>:9000

### Default Login
- admin / admin

---

### 3️ Install Sonar Scanner CLI:

```bash
sudo wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-8.0.1.6346-linux-x64.zip

unzip sonar-scanner-cli-8.0.1.6346-linux-x64.zip

mv sonar-scanner-8.0.1.6346-linux-x64/* /opt/sonar-scanner
```

---

###  Set PATH

```bash
echo 'export PATH=$PATH:/opt/sonar-scanner/bin' >> ~/.bashrc
source ~/.bashrc
```

---

###  Verify

```bash
sonar-scanner -v
```

---

### 4️ SonarQube Project Config:

Create file:

```bash
nano sonar-project.properties
```

```properties
sonar.projectKey=nextjs-app
sonar.projectName=Security Scan
sonar.sources=.
sonar.exclusions=node_modules/**,.next/**,out/**,coverage/**
sonar.sourceEncoding=UTF-8
```

---

### Run Scan

```bash
sonar-scanner   -Dsonar.host.url=http://server_ip:9000   -Dsonar.login=<YOUR_SONAR_TOKEN>
```

---

### 5️ OWASP ZAP Scan:

```bash
sudo docker run --rm   --network host   -v "$(pwd):/zap/wrk"   ghcr.io/zaproxy/zaproxy:stable   zap-baseline.py   -t https://YourDomain/api/sendgrid   -r zap-report.html
```

---

### 6️ Purpose:

- SAST → SonarQube
- SCA → npm audit
- DAST → OWASP ZAP
- CI/CD Security validation

---

## Pipeline Flow

GitHub Push → Testing Server → SAST → SCA → DAST → Report
