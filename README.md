# DevOps & Cloud Technical Challenge – Automation Engineer

This repository implements a secure, scalable, and automated Continuous Delivery (CD) lifecycle on AWS using **Terraform** and **GitHub Actions**.

The solution deploys a containerized web application (Nginx) on **Amazon ECS Fargate**, exposed via an **Application Load Balancer (ALB)**, and guarantees that **all outbound traffic to the internet originates from a static public IP address** using a **NAT Gateway with an Elastic IP**.

---

## 1. Overview

**Key objectives achieved:**

- Infrastructure defined entirely as code (Terraform)
- Secure CI/CD pipeline using GitHub Actions
- Authentication from GitHub to AWS via **OIDC** (no long-lived credentials)
- Application deployed in **private subnets**
- Deterministic outbound IP using **NAT Gateway + Elastic IP**
- Preview (plan) stage for pull requests
- Automated deployment on merge to `main`

---

## 2. Architecture Overview

### Technologies Used

- **Cloud Provider:** AWS
- **IaC:** Terraform
- **CI/CD:** GitHub Actions
- **Compute:** Amazon ECS Fargate
- **Registry:** Amazon ECR
- **Networking:** VPC, ALB, NAT Gateway, Elastic IP
- **Security:** IAM OIDC, Least Privilege

---

## 3. Repository Structure

```text
.
├── app/
│   ├── Dockerfile
│   └── index.html
├── infra/
│   ├── main.tf
│   ├── providers.tf
│   ├── vpc.tf
│   ├── security.tf
│   ├── iam.tf
│   ├── ecs_iam.tf
│   ├── ecs.tf
│   ├── alb.tf
│   └── outputs.tf
├── .github/workflows/
│   └── deploy.yml
└── README.md
```

---

## 4. Architecture Diagrams

### 4.1 Control Plane – CI/CD Flow

```text
Developer
   |
   | git push / pull request
   v
GitHub Repository
   |
   v
GitHub Actions
   |
   | OIDC (sts:AssumeRoleWithWebIdentity)
   v
AWS IAM Role (Least Privilege)
   |
   v
Terraform → AWS APIs
```

### 4.2 Data Plane – Application Traffic Flow

```text
Internet
   |
   v
Application Load Balancer (Public Subnet)
   |
   v
ECS Fargate Tasks (Private Subnets)
   |
   v
NAT Gateway (Public Subnet)
   |
Elastic IP (Static)
   |
   v
Internet
```

---

## 5. CI/CD Pipeline

### Stages

1. **Validation**
   - Terraform format and validation checks
2. **Build**
   - Docker image build
   - Push image to Amazon ECR
3. **Preview**
   - Terraform plan posted as Pull Request comment
4. **Deploy**
   - Automatic deployment on merge to `main`

---

## 6. How to Replicate the Deployment

### Prerequisites

- AWS Account
- Terraform >= 1.5
- GitHub repository
- IAM Role configured for GitHub OIDC

### Steps

1. Clone the repository
2. Configure GitHub OIDC IAM role
3. Push a Pull Request to preview the plan
4. Merge to `main` to deploy automatically

---

## 7. Follow-up Questions

### Performance
Docker build times can be optimized using layer caching, multi-stage builds, and remote cache in Amazon ECR.

### Reliability
Terraform is idempotent. If deployment fails, the state reflects the last successful resources and subsequent applies reconcile the environment.

### Observability
Application health is verified using ALB health checks and ECS service events. Rollbacks can be automated using ECS deployment circuit breakers.

### Cost Optimization
For low-frequency outbound traffic, NAT Gateway can be replaced with scheduled Lambda executions or VPC Endpoints where applicable.

---

## Conclusion

This project demonstrates production-ready AWS infrastructure with secure CI/CD, strong network isolation, and predictable outbound connectivity using modern DevOps best practices.
