# fullstack-bloggin-app-CICD-pipeline

Here's a detailed description for the **Blogging App CI/CD Project**. This document is designed to be clear and instructional for other developers or DevOps engineers who want to replicate this setup.



This project demonstrates a complete CI/CD pipeline for deploying a blogging application using **Jenkins**, **Docker**, **SonarQube**, **Nexus**, **Trivy**, **Terraform**, and **Kubernetes (EKS)**. Monitoring is handled via **Prometheus** and **Grafana**, and a custom domain is managed through **GoDaddy**.

> ⚠️ **IMPORTANT:** Ensure that you do not store sensitive tokens or credentials in public repositories. Remove tokens and secrets before pushing to GitHub.

---

## 🧱 Project Overview

The pipeline is triggered when the source code is pushed to GitHub. Jenkins executes multiple automated stages including:

1. Code compilation using Maven
2. Unit testing
3. Static analysis via SonarQube
4. Vulnerability scanning using Trivy
5. Artifact deployment to Nexus
6. Docker image creation and scan
7. Deployment to AWS EKS via Kubernetes
8. Domain mapping via GoDaddy
9. Monitoring via Prometheus & Grafana

---

## 🛠️ Tools & Technologies

- **Jenkins** (CI Orchestration)
- **GitHub** (Source Code Repository)
- **SonarQube** (Static Code Analysis)
- **Trivy** (Vulnerability Scanner)
- **Maven** (Build Tool)
- **Nexus** (Artifact Repository)
- **Docker** (Containerization)
- **Terraform** (Infrastructure as Code)
- **AWS EKS** (Kubernetes on AWS)
- **GoDaddy** (Domain Management)
- **Prometheus + Grafana** (Monitoring)
- **Email Notifications** (SMTP configuration on Jenkins)

---

## 📁 Folder Structure

```

terraform-configuration/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
k8s/
│   ├── svc.yml
│   ├── deployment.yaml
.gitignore
Jenkinsfile
README.md

````

---

## 🚀 CI/CD Pipeline Stages

### ✅ 1. GitHub Integration

- Push your source code to a GitHub repository.
- Configure Jenkins to poll the repository or set up a webhook.

### 🧾 2. Jenkins Configuration

- Install Jenkins on a dedicated EC2 instance.
- Install required plugins:
  - Docker, Kubernetes, Maven, SonarQube, Trivy, Config File Provider, etc.
- Configure Jenkins to discard old builds to save disk space.

### ⚙️ 3. Pipeline Definition (Jenkinsfile)

- Define tools block: `jdk 'jdk17'`, `maven 'maven3'`
- Git checkout stage using credentials.
- Compilation: `mvn compile`
- Unit Testing: `mvn test`

### 🔍 4. Vulnerability Scan (Trivy FS)

```bash
trivy fs --format table -o fs.html .
````

### 📊 5. SonarQube Code Analysis

```bash
export SCANNER_HOME=tool 'sonar-scanner'
$SCANNER_HOME/bin/sonar-scanner \
  -Dsonar.projectName=Blogging-app \
  -Dsonar.projectKey=Blogging-app \
  -Dsonar.java.binaries=target
```

* Add SonarQube server configuration in Jenkins
* Create credentials using your SonarQube token

### 📦 6. Maven Build and Deploy to Nexus

* Use `mvn package` to build the project.
* Configure `pom.xml` with Nexus distribution URLs.
* Set Nexus credentials in Jenkins as secret text.

### 🐳 7. Docker Build and Push

* Build Docker image from the project.
* Scan image with Trivy.
* Push to a private Docker registry using `withDockerRegistry`.

```groovy
withDockerRegistry([credentialsId: 'docker-creds', url: 'https://index.docker.io/v1/']) {
  sh 'docker build -t myblogapp .'
  sh 'docker push myblogapp'
}
```

---

## ☁️ Infrastructure Setup with Terraform

1. Launch a new EC2 instance to run Terraform.
2. Install Terraform:

   ```bash
   sudo snap install terraform --classic
   ```
3. Install AWS CLI and configure credentials.
4. Create `main.tf`, `variables.tf`, `outputs.tf`
5. Initialize and apply Terraform:

   ```bash
   terraform init
   terraform plan
   terraform apply
   ```

---

## ☸️ Kubernetes Setup

### Step-by-step:

* Install `kubectl`

* Update kubeconfig to connect to EKS:

  ```bash
  aws eks update-kubeconfig --region us-east-1 --name devopsshack-cluster
  ```

* Create Namespace:

  ```bash
  kubectl create ns webapps
  ```

* Create ServiceAccount:

  ```yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: jenkins
    namespace: webapps
  ```

* Apply via:

  ```bash
  kubectl apply -f svc.yml
  ```

### Docker Secret for Private Registry

```bash
kubectl create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=<your-username> \
  --docker-password='<your-password>' \
  --namespace=webapps
```

### K8s Deployment (Pipeline Stage)

```groovy
sh 'kubectl apply -f deployment.yaml'
sleep 20
sh 'kubectl get pods'
sh 'kubectl get svc'
```

---

## 📧 Email Notifications

* Configure SMTP under Jenkins → Manage Jenkins → Configure System
* Use Google App Passwords for SMTP Auth
* Add pipeline stage for email alerts based on build result

---

## 🌐 Domain Setup (GoDaddy)

* Create A record pointing your domain to the LoadBalancer IP exposed by the Kubernetes service.
* Wait for DNS propagation (\~5-10 minutes)

---

## 📈 Monitoring with Prometheus & Grafana

* Deploy Prometheus and Grafana to the Kubernetes cluster.
* Expose Grafana via LoadBalancer or Ingress.
* Configure Prometheus as a data source in Grafana.
* Set alerts based on pod health, memory usage, etc.

---

## 🔐 Credentials Management

Use Jenkins credentials store for:

* GitHub token (Personal Access Token)
* SonarQube token (secret text)
* Nexus credentials (username/password)
* Docker Hub credentials (username/password)
* Kubernetes token (service account secret)

---

## 🧽 Cleanup

* Set up `discard old builds` in Jenkins
* Schedule cleanup policies in Nexus & Docker Registry
* Revoke any unused tokens and rotate secrets regularly

---

## 📝 License

MIT License - see [LICENSE](LICENSE) file for details.

---

## 👥 Contributors

* [Your Name](https://github.com/yourusername)
* Open to contributions! Fork this repo and open a PR.

---

> **DISCLAIMER:** Do not commit any real API tokens or passwords to your public repository. All secrets above are illustrative.


