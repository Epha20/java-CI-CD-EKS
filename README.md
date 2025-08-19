# java-CI-CD-EKS


This project implements a secure and automated CI/CD pipeline for a Java-app

- **GitLab CI** for orchestrating code quality checks and triggering Jenkins
- **Jenkins** for build, scan, image handling, and deployment
- **Maven** to build the Java application
- **SonarQube** for code quality analysis
- **Trivy** for container vulnerability scanning
- **Docker** to containerize the app
- **Harbor** to store Docker images
- **Vault** to inject secrets during deployment
- **AWS EKS** as the Kubernetes hosting platform

## 🔄 Pipeline Stages

### 1. GitLab CI

- Runs `sonar-scanner-cli` to analyze code quality
- Triggers Jenkins job via the default API

### 2. Jenkins Pipeline

1. **Checkout**: Pull latest code
2. **Build and Analyze**: Run `mvn clean package` and analyzes with `sonarqube`
3. **Docker Build**: Create container image
4. **Trivy Scan**: Scan image for CVEs
5. **Push to Harbor**: Save image to private registry
6. **Vault Secrets**: Fetch DB password from Vault
7. **Deploy to EKS**: Apply Kubernetes YAMLs using `kubectl`



## 🔐 Secrets

Secrets like database passwords are securely fetched from **Vault** and injected into Kubernetes manifests dynamically.

## 📦 Image Registry

All images are pushed to **Harbor** for secure storage and management.
