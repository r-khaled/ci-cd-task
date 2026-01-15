# End-to-End DevSecOps Cloud-Native Application Deployment on AWS Kubernetes

## Project Overview

This project demonstrates a production-grade DevSecOps implementation for deploying a containerized Java Spring Boot application on Amazon EKS (Elastic Kubernetes Service). The solution implements a complete CI/CD pipeline with integrated security controls, infrastructure-as-code provisioning, and GitOps-based continuous delivery.

The project addresses the entire deployment lifecycle, from source code management through production deployment, with emphasis on security, automation, and operational excellence.

## Architecture Overview
![maven-app-digram](https://github.com/user-attachments/assets/48180d8f-4d74-4770-a94a-d7f7045ef3a6)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         GitHub Repository                               │
│  (Source Code, Dockerfile, Helm Charts, GitHub Actions Workflows)       │
└─────────────────────┬───────────────────────────────────────────────────┘
                      │
                      ▼
        ┌─────────────────────────────────┐
        │   GitHub Actions CI Pipeline    │
        │  (Build, Test, Scan, Push)      │
        └──────────────┬──────────────────┘
                       │
        ┌──────────────┴──────────────┐
        ▼                             ▼
   ┌─────────────┐           ┌──────────────────┐
   │  SonarQube  │           │   Docker Hub     │
   │  (SAST)     │           │   (Registry)     │
   └─────────────┘           └────────┬─────────┘
                                      │
                                      ▼
                            ┌──────────────────┐
                            │  Trivy Scanner   │
                            │  (DAST)          │
                            └────────┬─────────┘
                                     │
        ┌────────────────────────────┴────────────────────┐
        │                                                  │
        ▼                                                  ▼
┌──────────────────────┐                        ┌──────────────────┐
│   Terraform (IaC)    │                        │   Argo CD        │
│  - AWS IAM           │                        │  (GitOps)        │
│  - EKS Cluster       │                        │  - Continuous    │
│  - Networking        │                        │    Deployment    │
└──────────────────────┘                        └────────┬─────────┘
        │                                                  │
        │                              ┌───────────────────┘
        │                              │
        ▼                              ▼
┌─────────────────────────────────────────────────────────────┐
│            AWS EKS Cluster (Production)                     │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Kubernetes Namespaces & Deployments              │    │
│  │  ┌──────────────────────────────────────────────┐ │    │
│  │  │  Spring Boot Application Pod                 │ │    │
│  │  │  - Helm Deployed                             │ │    │
│  │  │  - Service (ClusterIP/LoadBalancer)          │ │    │
│  │  │  - Replicas & Auto-scaling                   │ │    │
│  │  └──────────────────────────────────────────────┘ │    │
│  │  - ConfigMaps & Secrets (K8s native)             │    │
│  │  - Health Checks & Resource Limits               │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## CI/CD Pipeline Explanation

The GitHub Actions-based CI/CD pipeline automates the entire deployment workflow with integrated security checkpoints:

### Pipeline Stages

**1. Code Checkout & Build**
- Repository code is cloned
- Maven dependency resolution and compilation
- Application JAR artifact generation
- Unit and integration test execution

**2. Static Application Security Testing (SAST)**
- SonarQube performs comprehensive code analysis
- Identifies code vulnerabilities, code smells, and quality issues
- Enforces quality gates before artifact creation
- Generates security reports for code review

**3. Docker Image Build & Push**
- Multi-stage Dockerfile optimizes image layers
- Application JAR is containerized
- Image is tagged with commit SHA and version
- Image is pushed to Docker Hub registry
- Images are scanned for vulnerabilities before deployment

**4. Container Vulnerability Scanning (DAST)**
- Trivy performs dynamic container image scanning
- Identifies CVEs in application and base image dependencies
- Generates SBOM (Software Bill of Materials)
- Blocks deployment if critical vulnerabilities are detected

**5. Kubernetes Deployment via Helm**
- Helm charts template Kubernetes manifests
- Environment-specific values override defaults
- Deployment applied to AWS EKS cluster
- Rolling updates ensure zero-downtime deployments

**6. GitOps Continuous Delivery via Argo CD**
- Argo CD monitors Git repository for changes
- Automatically synchronizes Kubernetes cluster state with Git
- Enables declarative deployment management
- Provides audit trail for all infrastructure changes

## Infrastructure & Terraform Overview

Infrastructure provisioning is fully codified using Terraform, ensuring reproducibility and version control:

### AWS IAM & Security

- IAM roles for EKS service account permissions
- OIDC provider integration for pod identity management
- Least privilege principle enforced through granular policies
- Audit logging enabled for compliance tracking

### Amazon EKS Cluster

- Managed Kubernetes control plane by AWS
- Worker node groups with auto-scaling capability
- Private subnets for enhanced security
- VPC CNI plugin for pod networking
- Integration with AWS CloudWatch for monitoring

### Kubernetes Networking

- Service mesh-ready architecture (prepared for Istio/Cilium)
- Network policies enforcing segmentation
- LoadBalancer or Ingress-based external exposure
- DNS resolution via CoreDNS
- CNI-based pod-to-pod communication

## Kubernetes & Helm Deployment

### Helm Chart Structure

```
helm/app-backend/
├── Chart.yaml              # Chart metadata and versioning
├── values.yaml             # Default configuration values
├── values-staging.yaml     # Staging environment overrides
├── values-prod.yaml        # Production environment overrides
├── templates/
│   ├── deployment.yaml     # Kubernetes Deployment manifest
│   ├── service.yaml        # Kubernetes Service manifest
│   ├── _helpers.tpl        # Template helper functions
│   ├── configmap.yaml      # Application configuration
│   ├── secret.yaml         # Sensitive data management
│   ├── ingress.yaml        # HTTP routing rules
│   └── NOTES.txt           # Post-deployment instructions
└── charts/                 # Dependency charts directory
```

### Configuration Management

- **values.yaml**: Defines default replica count, resource limits, image pull policy
- **Environment-Specific Overrides**: Separate values files for staging and production
- **Secrets Management**: Kubernetes Secrets for credentials, API keys, certificates
- **ConfigMaps**: Non-sensitive configuration data for application properties

### Deployment Strategy

- **Rolling Updates**: Gradual pod replacement with zero downtime
- **Readiness Probes**: Health checks before routing traffic
- **Liveness Probes**: Automatic pod restart on failure
- **Resource Requests/Limits**: CPU and memory guarantees and constraints
- **Image Pull Strategy**: IfNotPresent for registry efficiency

## Security & DevSecOps Practices

### Shift-Left Security Approach

Security is integrated from the earliest stages of development rather than retrofitted at deployment:

- **Pre-commit Hooks**: Static analysis on developer machines
- **Early Scanning**: Code analysis before merge requests
- **Pipeline Gates**: Security checks blocking non-compliant deployments

### Static Application Security Testing (SAST)

**SonarQube Integration**
- Scans Java source code for vulnerabilities and anti-patterns
- Detects SQL injection, XSS, insecure deserialization risks
- Enforces coding standards and maintainability metrics
- Provides remediation guidance for developers

### Dynamic Application Security Testing (DAST)

**Trivy Container Scanning**
- Scans Docker images for known CVEs
- Checks application dependencies in JAR files
- Identifies vulnerable transitive dependencies
- Generates actionable remediation reports

### Secrets Management

- GitHub Secrets: Secure storage for credentials and tokens
- Kubernetes Secrets: Encrypted at-rest secrets management
- No hardcoded credentials in source code or container images
- Automatic secret rotation policies implemented

### Kubernetes Security Best Practices

- Network Policies: Restrict pod-to-pod communication
- Pod Security Standards: Enforce container runtime policies
- RBAC (Role-Based Access Control): Fine-grained permission management
- Service Account Token Projection: Workload identity for AWS integration
- Resource Quotas: Prevent resource exhaustion attacks

## Technologies & Tools

### Application Development
- **Java 11+**: Primary application language
- **Spring Boot 2.x/3.x**: Web framework and runtime
- **Maven**: Build automation and dependency management
- **JUnit/Mockito**: Unit testing frameworks

### Containerization
- **Docker**: Container image format and runtime
- **Docker Hub**: Public container registry
- **Multi-stage Builds**: Optimized image sizes

### CI/CD & Automation
- **GitHub Actions**: Workflow orchestration and automation
- **SonarQube**: Static code analysis and quality gates
- **Trivy**: Container vulnerability scanning

### Infrastructure & Orchestration
- **Terraform**: Infrastructure as Code for AWS resources
- **AWS EKS**: Managed Kubernetes service
- **Helm**: Kubernetes package manager and templating

### GitOps & Deployment
- **Argo CD**: Declarative continuous deployment
- **Git**: Source of truth for infrastructure and application state
- **Kustomize**: Configuration management (alternative/complement to Helm)

### Monitoring & Observability
- **Prometheus**: Metrics collection and alerting
- **Grafana**: Metrics visualization dashboards
- **ELK Stack**: Centralized logging (Elasticsearch, Logstash, Kibana)

## Quick Start: Running After Clone

### Prerequisites for Local/Development Setup

- Java 11+ installed on your machine
- Maven 3.6+ for building the application
- Docker Desktop installed and running
- kubectl configured to access an EKS cluster
- helm 3.x installed for chart management
- Git configured with SSH or HTTPS credentials

### Getting Started (Local Development)

1. **Clone the Repository**
   ```bash
   git clone https://github.com/r-khaled/ci-cd-task.git
   cd ci-cd-task
   ```

2. **Build the Spring Boot Application**
   ```bash
   cd spring-boot-app
   mvn clean package
   ```

3. **Run the Application Locally**
   ```bash
   java -jar target/spring-boot-web.jar
   ```
   The application will be available at `http://localhost:8080`

4. **Build Docker Image Locally**
   ```bash
   docker build -t app-backend:latest .
   ```

5. **Deploy to Local Kubernetes (Minikube or Kind)**
   ```bash
   cd ../helm/app-backend
   helm install app-backend . -f values.yaml
   kubectl port-forward svc/app-backend 8080:80
   ```

6. **Verify Deployment**
   ```bash
   kubectl get pods
   kubectl logs -l app.kubernetes.io/name=app-backend
   kubectl get svc
   ```

### Running the GitHub Actions CI/CD Pipeline

To trigger the CI/CD pipeline:

1. Push code to the main branch:
   ```bash
   git add .
   git commit -m "feat: update application"
   git push origin main
   ```

2. Monitor pipeline execution in GitHub:
   - Navigate to Actions tab in your repository
   - View real-time build logs and test results
   - Confirm SonarQube analysis and Trivy scan results

3. Upon successful completion:
   - Docker image is pushed to Docker Hub
   - Helm deployment updates the EKS cluster
   - Argo CD synchronizes the desired state

---

## How to Deploy to Production

### Prerequisites for Production Deployment

- AWS Account with EKS cluster provisioned via Terraform
- Docker Hub account with push credentials
- GitHub Secrets configured (DOCKER_USERNAME, DOCKER_PASSWORD, etc.)
- SonarQube server accessible from GitHub Actions runners
- Argo CD installed and configured on EKS cluster

### Deployment Process (High-Level)

1. **Push Code to Main Branch**
   - Developer commits and pushes changes to GitHub
   - GitHub Actions workflow triggers automatically

2. **Automated Quality Checks**
   - Maven builds the application
   - SonarQube performs static analysis
   - Tests execute and coverage is validated

3. **Containerization**
   - Docker image is built from Dockerfile
   - Image is tagged with commit SHA and version
   - Image is pushed to Docker Hub registry

4. **Security Scanning**
   - Trivy scans image for vulnerabilities
   - SBOM is generated for compliance
   - Critical vulnerabilities block progression (optional gating)

5. **Kubernetes Deployment**
   - Helm renders Kubernetes manifests from charts
   - Kubectl applies manifests to EKS cluster
   - Rolling update strategy ensures availability

6. **GitOps Synchronization**
   - Argo CD detects manifest changes in Git
   - Cluster state automatically synchronizes with Git
   - Deployment status is monitored continuously

7. **Validation**
   - Health checks verify pod readiness
   - Service endpoints become available
   - Logs confirm successful startup

### Rollback Strategy

- Git-based rollback: Revert commit and push to trigger redeploy
- Helm rollback: Revert to previous Helm release version
- Argo CD synchronization: Maintains desired state declaratively

## Future Enhancements

### Advanced Security
- Container runtime security with Falco
- Kubernetes API audit logging and analysis
- Zero-trust networking with Cilium
- Secrets rotation automation with HashiCorp Vault
- SLSA framework compliance for supply chain security

### Observability & Performance
- Distributed tracing with Jaeger or Zipkin
- Custom metrics and business KPIs
- Service mesh implementation (Istio/Linkerd)
- Performance testing automation
- Chaos engineering for resilience validation

### DevOps Maturity
- Multi-environment promotion pipeline (dev → staging → prod)
- Feature flags integration for controlled rollouts
- Cost optimization through resource right-sizing
- Multi-region deployment for high availability
- Disaster recovery and backup automation

### Compliance & Governance
- Automated compliance scanning (CIS benchmarks)
- Policy-as-Code enforcement (OPA/Gatekeeper)
- Audit logging centralization
- FIPS compliance for sensitive workloads
- Data residency and encryption requirements

## Repository Structure

```
ci-cd-task/
├── spring-boot-app/          # Application source code
│   ├── src/                  # Java source and resources
│   ├── pom.xml              # Maven configuration
│   ├── Dockerfile           # Container image definition
│   └── JenkinsFile          # Legacy CI definition
├── k8s/                      # Kubernetes manifests (declarative)
│   ├── deployment.yml       # Pod and replica definitions
│   └── service.yml          # Service networking
├── helm/                     # Helm charts (templated)
│   └── app-backend/         # Application Helm chart
│       ├── Chart.yaml       # Chart metadata
│       ├── values.yaml      # Default values
│       └── templates/       # K8s manifest templates
├── .github/
│   └── workflows/           # GitHub Actions CI/CD pipelines
│       └── main.yml         # Primary workflow definition
├── terraform/               # Infrastructure as Code (if provisioned)
│   ├── main.tf             # AWS EKS and networking
│   ├── variables.tf        # Input variables
│   └── outputs.tf          # Exported values
└── README.md               # This file
```

## Conclusion

This project represents a comprehensive DevSecOps implementation demonstrating enterprise-grade practices for secure, automated, and scalable application deployment on Kubernetes. By combining infrastructure-as-code, containerization, automated security scanning, and GitOps principles, the solution delivers reliable, auditable, and maintainable deployments suitable for production environments.

The architecture is designed to be extensible, allowing integration of additional tools and practices as organizational needs evolve while maintaining security and operational excellence.

