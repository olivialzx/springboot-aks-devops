# Spring PetClinic CI/CD on Azure Kubernetes Service

An end-to-end **DevOps CI/CD project** that automates the build, testing, code-quality analysis, containerization, security scanning, and deployment of a Java Spring PetClinic application to **Azure Kubernetes Service (AKS)**.

The project uses **GitHub, Jenkins, Maven, SonarQube, Docker, Trivy, Azure Container Registry (ACR), Kubernetes, AKS, and the Brevo API** to implement an automated application delivery workflow.

## Project Overview

The goal of this project is to demonstrate a complete CI/CD workflow for deploying a containerized Java application to Azure.

The Jenkins pipeline automates the process from source code checkout through application deployment and rollout verification:


GitHub
   |
   v
Jenkins
   |
   +--> Maven Validate
   |
   +--> Maven Test
   |
   +--> Maven Compile
   |
   +--> SonarQube Analysis
   |
   +--> Maven Package
   |
   +--> Docker Build
   |
   +--> Trivy Security Scan
   |
   +--> Push to Azure Container Registry
   |
   +--> Create ACR Pull Secret
   |
   +--> Deploy to AKS
   |
   +--> Verify Kubernetes Rollout
   |
   +--> Brevo Success/Failure Notification
```

 What I Implemented

This project demonstrates practical work across the application delivery lifecycle:

* Created and maintained a dedicated GitHub repository for the project.
* Configured Jenkins as the CI/CD automation server.
* Implemented Maven validation, compilation, testing, and packaging.
* Integrated SonarQube/SonarCloud for static code-quality analysis.
* Created a Docker image for the Spring PetClinic application.
* Integrated Trivy to scan the container image for HIGH and CRITICAL vulnerabilities.
* Configured Azure Container Registry for private Docker image storage.
* Configured Jenkins credentials for secure ACR authentication.
* Created Kubernetes Deployment and Service manifests.
* Deployed the application to Azure Kubernetes Service.
* Configured two Kubernetes application replicas.
* Configured an ACR image-pull secret for private registry authentication.
* Exposed the application through a Kubernetes LoadBalancer Service.
* Added Kubernetes rollout verification to the CI/CD pipeline.
* Integrated the Brevo API for automated pipeline success/failure notifications.
* Troubleshot Docker, Jenkins, ACR, Kubernetes, AKS, authentication, and deployment issues.

Architecture


                         GitHub
                            |
                            | Source Code
                            v
                         Jenkins
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
        Maven          SonarQube          Docker Build
     Build & Test      Code Quality            |
                                                v
                                             Trivy
                                        Security Scan
                                                |
                                                v
                                  Azure Container Registry
                                                |
                                                | Docker Image
                                                v
                                      Azure Kubernetes Service
                                                |
                                      +---------+---------+
                                      |                   |
                                      v                   v
                                   Pod 1               Pod 2
                                      \                   /
                                       \                 /
                                        v               v
                                     Kubernetes Service
                                      LoadBalancer :80
                                             |
                                             v
                                        Application
                                             |
                                             v
                                       Brevo Notification
```

 Technology Stack

| Technology               | Purpose                             |
| ------------------------ | ----------------------------------- |
| Java 17                  | Application runtime                 |
| Spring Framework         | Application framework               |
| Maven                    | Build, test, and package management |
| GitHub                   | Source control                      |
| Jenkins                  | CI/CD automation                    |
| SonarQube / SonarCloud   | Static code-quality analysis        |
| Docker                   | Application containerization        |
| Trivy                    | Container vulnerability scanning    |
| Azure Container Registry | Private Docker image registry       |
| Kubernetes               | Container orchestration             |
| Azure Kubernetes Service | Managed Kubernetes platform         |
| kubectl                  | Kubernetes administration           |
| Brevo API                | CI/CD email notifications           |

Repository Structure

```text
.
├── src/
│   ├── main/
│   └── test/
├── Dockerfile
├── Jenkinsfile
├── deployment.yaml
├── service.yaml
└── pom.xml
```

 Important Files

| File              | Purpose                                        |
| ----------------- | ---------------------------------------------- |
| `Jenkinsfile`     | Defines the complete CI/CD pipeline            |
| `Dockerfile`      | Builds the application container image         |
| `deployment.yaml` | Defines the Kubernetes application deployment  |
| `service.yaml`    | Exposes the application through a LoadBalancer |
| `pom.xml`         | Maven project configuration and dependencies   |
| `src/`            | Application source code and tests              |

 CI/CD Pipeline

 1. Source Code Checkout

Jenkins checks out the `main` branch from this GitHub repository:


https://github.com/olivialzx/springboot-aks-devops.git

2. Maven Validation

The pipeline validates the Maven project:


mvn validate


3. Automated Testing

The application tests are executed using:


mvn test


The project previously completed:


62 tests
0 failures
0 errors
0 skipped


 4. Maven Compilation

The Java application is compiled using:


mvn compile


 5. SonarQube Analysis

Jenkins runs SonarQube analysis to evaluate code quality and identify potential issues such as bugs, vulnerabilities, and code smells.

The project is configured with its own SonarQube/SonarCloud organization and project configuration.

6. Maven Packaging

The application is packaged using:


mvn package


This produces:


target/petclinic.war


 7. Docker Image Build

The application is containerized using Docker.

The Dockerfile uses Jetty with Java 17:

```dockerfile
FROM jetty:11.0-jdk17
COPY target/petclinic.war /var/lib/jetty/webapps/ROOT.war
EXPOSE 8080
```

The resulting image is:

```text
oliviacontainerreg.azurecr.io/springbootjavaapp:latest
```

8. Trivy Security Scan

The Docker image is scanned using Trivy for HIGH and CRITICAL vulnerabilities.

The pipeline generates:

```text
trivy-image-report.txt
```

This provides a security check before the image is pushed to the container registry.

9. Push to Azure Container Registry

Jenkins securely authenticates with Azure Container Registry and pushes the Docker image.

Registry:

```text
oliviacontainerreg.azurecr.io
```

Image:

```text
oliviacontainerreg.azurecr.io/springbootjavaapp:latest
```

 10. Kubernetes ACR Pull Secret

Because the Docker image is stored in a private ACR, Kubernetes needs authentication to pull it.

The pipeline creates or updates:

```text
acr-secret
```

The Kubernetes Deployment references this secret:

```yaml
imagePullSecrets:
  - name: acr-secret
```

This allows AKS to pull the private image from Azure Container Registry.

 11. Deployment to AKS

Jenkins deploys the Kubernetes resources:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

The Deployment runs two replicas:

```yaml
replicas: 2
```

The application container listens on:

```text
8080
```

 12. Kubernetes Service

The application is exposed through a Kubernetes `LoadBalancer` Service.

```text
External Port: 80
Target Port: 8080
Service Type: LoadBalancer
```

Traffic flows as:

```text
Internet
   |
   v
Azure Load Balancer
   |
   v
Kubernetes Service
   |
   +------> Pod 1
   |
   +------> Pod 2
```

13. Deployment Rollout Verification

Jenkins verifies that the Kubernetes deployment successfully rolls out:

```bash
kubectl rollout status deployment/petclinic --timeout=180s
```

This ensures the pipeline does not report deployment success until Kubernetes confirms that the application has successfully rolled out.

 14. Brevo Notifications

The Jenkins pipeline integrates with the Brevo API.

A success notification is sent when the pipeline and deployment complete successfully.

A failure notification is sent when a pipeline or deployment stage fails.

The Brevo API key is stored securely in Jenkins credentials and is not hardcoded in the repository.

 Azure Resources

The project uses the following Azure components:

```text
Azure Container Registry
    Name: oliviacontainerreg
    Login Server: oliviacontainerreg.azurecr.io

Azure Kubernetes Service
    Cluster: demoaks

Kubernetes Deployment
    Name: petclinic

Kubernetes Service
    Name: petclinic-service
```

Jenkins Configuration

The following Jenkins tools and credentials are used by the pipeline:

| Jenkins Configuration  | Name / Credential ID | Purpose                       |
| ---------------------- | -------------------- | ----------------------------- |
| Maven Tool             | `maven`              | Maven build                   |
| SonarQube Installation | `sonar-server`       | Code-quality analysis         |
| Username/Password      | `acr-creds`          | ACR authentication            |
| Secret File            | `kubeconfig`         | AKS/Kubernetes authentication |
| Secret Text            | `brevo-api-key`      | Brevo API authentication      |

The Jenkins environment variables include:

```text
ACR_SERVER=oliviacontainerreg.azurecr.io
IMAGE_NAME=springbootjavaapp
IMAGE_TAG=latest
K8S_NAMESPACE=default
DEPLOYMENT_NAME=petclinic
```

Sensitive credentials are stored in Jenkins rather than committed to GitHub.

 Kubernetes Configuration

 Deployment

The Kubernetes Deployment:

* Runs two application replicas.
* Uses the private ACR image.
* Uses `acr-secret` for image authentication.
* Exposes container port 8080.
* Uses the `app: petclinic` label.

Service

The Kubernetes Service:

* Uses type `LoadBalancer`.
* Exposes port 80.
* Routes traffic to port 8080.
* Selects the PetClinic pods.

 Local Development

Prerequisites

For local development:

* JDK 17 or later
* Maven
* Docker, if running the containerized application

 Clone the Repository

```bash
git clone https://github.com/olivialzx/springboot-aks-devops.git
cd springboot-aks-devops
```

RUN TESTS
```bash
mvn clean test
```

 Build the Application

```bash
mvn clean package
```

The generated WAR file will be:

```text
target/petclinic.war
```

 Run with Docker

Build the image:

```bash
docker build -t springbootjavaapp:latest .
```

Run the container:

```bash
docker run --rm -p 8080:8080 springbootjavaapp:latest
```

The application can then be accessed at:

```text
http://localhost:8080
```

 Deploy to Kubernetes Manually

If deploying outside Jenkins, create an ACR pull secret first:

```bash
kubectl create secret docker-registry acr-secret \
  --docker-server=<your-registry>.azurecr.io \
  --docker-username=<registry-username> \
  --docker-password=<registry-password> \
  --namespace default
```

Then apply the Kubernetes manifests:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Verify the deployment:

```bash
kubectl rollout status deployment/petclinic --timeout=180s
kubectl get pods
kubectl get service petclinic-service
```

Once Azure assigns an external IP to the LoadBalancer Service, the application can be accessed through that address.

 Useful Kubernetes Commands

View the application resources:

```bash
kubectl get all -l app=petclinic
```

View application logs:

```bash
kubectl logs -l app=petclinic --tail=100
```

Watch the LoadBalancer external IP:

```bash
kubectl get service petclinic-service --watch
```

Inspect the Deployment:

```bash
kubectl describe deployment petclinic
```

Inspect the pods:

```bash
kubectl describe pods -l app=petclinic
```

Check rollout status:

```bash
kubectl rollout status deployment/petclinic
```

Troubleshooting Experience

During implementation, several real DevOps and deployment issues were identified and resolved.

 Dockerfile / Build Context

The Jenkins workspace and Docker build context had to be configured correctly so Docker could locate the Dockerfile and the Maven-generated WAR file.
 Jenkins Docker Permissions

Jenkins required permission to communicate with the Docker daemon so that it could build and push container images.

 Azure Container Registry Authentication

Jenkins was configured with ACR credentials to authenticate and push the private Docker image.
Kubernetes ImagePullBackOff

The AKS pods initially experienced image-pull authentication problems.

This was resolved by configuring the Kubernetes `acr-secret` and referencing it through `imagePullSecrets`.

 AKS / ACR Permissions

The AKS identity was checked for the required ACR pull permission.
 Jenkins Credentials

Jenkins credentials were configured for:

* Azure Container Registry
* AKS kubeconfig
* Brevo API

Deployment Verification

The pipeline was configured to verify the Kubernetes rollout rather than treating `kubectl apply` alone as proof that the application was successfully running.

Security Practices

The project incorporates several security practices:

* Trivy container vulnerability scanning.
* SonarQube code-quality analysis.
* Private Azure Container Registry.
* Kubernetes image-pull authentication.
* Jenkins credential management.
* No Brevo API key stored in source code.
* No registry passwords stored directly in the Kubernetes manifest.
* ACR pull permissions for AKS.

 Project Outcome

The completed pipeline automates the application delivery lifecycle:

```text
Source Code
     ↓
Maven Build & Test
     ↓
SonarQube Analysis
     ↓
Maven Package
     ↓
Docker Image
     ↓
Trivy Security Scan
     ↓
Azure Container Registry
     ↓
AKS / Kubernetes
     ↓
2 Application Replicas
     ↓
LoadBalancer Service
     ↓
Rollout Verification
     ↓
Brevo Notification
```

The result is an automated CI/CD workflow that takes the application from source control to a running containerized workload on Azure Kubernetes Service.

 Key DevOps Skills Demonstrated

This project demonstrates practical experience with:

* CI/CD pipeline development
* Jenkins Pipeline as Code
* Git and GitHub
* Maven build automation
* Automated testing
* SonarQube integration
* Docker containerization
* Container security scanning with Trivy
* Azure Container Registry
* Kubernetes Deployments
* Kubernetes Services
* Kubernetes Secrets
* Azure Kubernetes Service
* `kubectl` administration
* Container registry authentication
* AKS/ACR integration
* CI/CD troubleshooting
* API-based CI/CD notifications

 Acknowledgement

The application is based on the Spring Framework PetClinic sample application and is used in this repository to demonstrate an end-to-end DevOps and CI/CD deployment workflow on Microsoft Azure.
