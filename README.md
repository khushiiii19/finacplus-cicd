# FinacPlus CI/CD Pipeline

A CI/CD pipeline implementation using GitHub, Jenkins, Docker, Docker Hub, and Kubernetes.

The pipeline automatically validates application code, runs automated application tests, builds a Docker image, pushes the image to Docker Hub, deploys the new image to Kubernetes, verifies the deployment, and attempts an automatic rollback if a Kubernetes rollout fails.

---

## 📦 Assignment Deliverables

### 1. Jenkins Pipeline

* `Jenkinsfile` contains the Groovy-based CI/CD pipeline with build, test, and Kubernetes deployment stages.
* Includes source checkout, application validation, automated pytest tests, Docker build, Docker Hub push, Kubernetes deployment, rollout verification, and deployment-aware rollback.

### 2. Setup Documentation

* GitHub repository and webhook configuration
* Jenkins configuration
* Docker Hub credential setup
* Kubernetes cluster and deployment configuration

### 3. Test Cases & Validation

* Python syntax validation
* Automated application endpoint tests using pytest
* Docker image build validation
* Docker Hub push validation
* Kubernetes deployment validation
* Rollout and pod status verification
* Application `/` and `/health` endpoint verification

### 4. Monitoring & Logging Recommendations

* Jenkins build and console logs
* Kubernetes pod and deployment status
* Application health checks
* Recommended monitoring and logging practices

---

## 📦 Assignment Deliverables

### 1. Jenkins Pipeline
- `Jenkinsfile` contains the Groovy-based CI/CD pipeline with build and Kubernetes deployment stages.
- Includes source checkout, application validation, Docker build, Docker Hub push, Kubernetes deployment, and rollout verification.

### 2. Setup Documentation
- GitHub repository and webhook configuration
- Jenkins configuration
- Docker Hub credential setup
- Kubernetes cluster and deployment configuration

### 3. Test Cases & Validation
- Application code validation
- Docker image build validation
- Docker Hub push validation
- Kubernetes deployment validation
- Rollout and pod status verification
- Application `/health` endpoint verification

### 4. Monitoring & Logging Recommendations
- Jenkins build and console logs
- Kubernetes pod and deployment status
- Application health checks
- Recommended monitoring and logging practices

---

## 1. Project Overview

This project implements an automated CI/CD workflow for a containerized Python Flask application.

The pipeline is triggered whenever changes are pushed to the `main` branch of the GitHub repository.

### Pipeline Flow

```text
Developer
    |
    | git push
    v
GitHub Repository
    |
    | GitHub Webhook
    v
Jenkins
    |
    +--> Checkout source code
    |
    +--> Run application validation
    |
    +--> Run automated pytest tests
    |
    +--> Build Docker image
    |
    +--> Push image to Docker Hub
    |
    +--> Deploy image to Kubernetes
    |
    +--> Verify rollout and pod status
    |
    +--> Roll back if Kubernetes rollout fails
    |
    v
Kubernetes Cluster
    |
    v
Running Application
```

---

## 2. Objectives

The main objectives of this project are:

* Automate application builds when code is committed to Git.
* Validate application source code before building the Docker image.
* Run automated application tests before deployment.
* Build and package the application as a Docker image.
* Store the Docker image in Docker Hub.
* Automatically deploy successful builds to Kubernetes.
* Verify that the Kubernetes rollout completes successfully.
* Automatically attempt a rollback when a Kubernetes deployment rollout fails.
* Make the pipeline reusable for different applications and Kubernetes environments.
* Handle failures clearly and prevent deployment when earlier stages fail.
* Follow basic security practices by storing credentials in Jenkins rather than hardcoding them.

---

## 3. Technology Stack

| Technology     | Purpose                           |
| -------------- | --------------------------------- |
| GitHub         | Source code repository            |
| Jenkins        | CI/CD automation                  |
| Groovy         | Jenkins pipeline scripting        |
| Python / Flask | Sample application                |
| pytest         | Automated application testing     |
| Docker         | Application containerization      |
| Docker Hub     | Container image registry          |
| Kubernetes     | Container orchestration           |
| kubectl        | Kubernetes command-line interface |
| GitHub Webhook | Automatic Jenkins trigger         |

---

## 4. Repository Structure

```text
finacplus-cicd/
│
├── app/
│   ├── app.py
│   └── requirements.txt
│
├── tests/
│   └── test_app.py
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
├── Dockerfile
├── Jenkinsfile
├── requirements-test.txt
├── README.md
└── .gitignore
```

### File Description

* `app/app.py` - Flask application.
* `app/requirements.txt` - Runtime Python dependencies.
* `tests/test_app.py` - Automated pytest tests for application endpoints.
* `requirements-test.txt` - CI/test dependencies.
* `Dockerfile` - Instructions for building the application container.
* `k8s/deployment.yaml` - Kubernetes Deployment configuration.
* `k8s/service.yaml` - Kubernetes Service configuration.
* `Jenkinsfile` - Complete CI/CD pipeline written in Groovy.
* `README.md` - Project documentation.
* `.gitignore` - Prevents unnecessary and sensitive local files from being committed.

---

## 5. Application

The project uses a simple Flask application with two endpoints.

### Application Endpoint

```text
/
```

Returns:

```text
FinacPlus CI/CD Pipeline is working!
```

### Health Endpoint

```text
/health
```

Returns:

```text
healthy
```

The `/health` endpoint is also used by Kubernetes readiness and liveness probes.

---

## 6. Docker Configuration

The application is packaged using the following Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app/requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .

RUN useradd --create-home appuser

USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
```

The Docker image contains:

* Python 3.12 runtime
* Flask dependency
* Application source code
* Application startup command

### Container Security

The application runs as a non-root user:

```dockerfile
RUN useradd --create-home appuser
USER appuser
```

Running the application as a dedicated non-root user reduces unnecessary container privileges.

The pipeline tags each Docker image using the Jenkins build number.

For example:

```text
khushiiii19/finacplus-cicd:17
```

This provides a unique image version for each pipeline execution.

---

## 7. Kubernetes Configuration

The application is deployed to Kubernetes using a Deployment and a Service.

### Deployment

The Kubernetes Deployment:

* Runs 2 replicas.
* Uses the Docker image generated by the pipeline.
* Exposes container port `5000`.
* Uses readiness and liveness probes.
* Supports rolling updates when a new image is deployed.

The Deployment is named:

```text
finacplus-app
```

### Service

The application is exposed through a Kubernetes NodePort Service.

The Service is named:

```text
finacplus-service
```

The Service configuration uses:

```text
Port: 5000
Target Port: 5000
Node Port: 30080
```

The application can be accessed locally through:

```text
http://localhost:30080
```

The health endpoint can be checked through:

```text
http://localhost:30080/health
```

---

## 8. Jenkins Pipeline

The CI/CD pipeline is defined in the `Jenkinsfile`.

The pipeline contains the following stages:

```text
Checkout
    |
    v
Test
    |
    v
Docker Build
    |
    v
Docker Push
    |
    v
Kubernetes Deploy
    |
    v
Deployment Verification
```

### Stage 1 — Checkout

Jenkins checks out the source code from the configured GitHub repository.

```groovy
checkout scm
```

---

### Stage 2 — Test

The pipeline performs application validation before building the Docker image.

First, the Python version is checked:

```bash
python3 --version
```

Python syntax is validated using:

```bash
python3 -m py_compile app/app.py
```

The pipeline then installs the test dependencies:

```bash
python3 -m pip install --user -r requirements-test.txt
```

Automated application tests are executed using:

```bash
python3 -m pytest tests/ -v
```

The tests validate:

* `/` endpoint response
* `/health` endpoint response
* HTTP status codes
* Expected response bodies

If any test fails, the pipeline stops and the Docker build and deployment stages are not executed.

---

### Stage 3 — Docker Build

A Docker image is built using the current Jenkins build number as the image tag.

Example:

```text
khushiiii19/finacplus-cicd:17
```

Using the Jenkins build number provides a unique version for each pipeline execution.

---

### Stage 4 — Docker Push

The generated image is pushed to Docker Hub.

Jenkins retrieves Docker Hub credentials from its credential store using `withCredentials`.

The credentials are not hardcoded in the Jenkinsfile.

The image is then pushed to the configured Docker Hub repository.

---

### Stage 5 — Kubernetes Deploy

The pipeline updates the Kubernetes Deployment with the newly built image using:

```bash
kubectl set image
```

The pipeline then waits for the rollout to complete using:

```bash
kubectl rollout status
```

The rollout has a timeout to prevent the pipeline from waiting indefinitely.

The deployment is only marked as completed after Kubernetes reports that the rollout has successfully finished.

---

### Stage 6 — Deployment Verification

The pipeline verifies:

* Kubernetes Deployment status
* Kubernetes Pod status

This provides an additional verification step after deployment.

---

## 9. Jenkins Pipeline Parameters

The Jenkinsfile uses parameters to make the pipeline reusable.

The configurable parameters are:

| Parameter        | Purpose                    |
| ---------------- | -------------------------- |
| `IMAGE_REPO`     | Docker image repository    |
| `K8S_DEPLOYMENT` | Kubernetes Deployment name |
| `K8S_CONTAINER`  | Kubernetes container name  |
| `K8S_NAMESPACE`  | Kubernetes namespace       |
| `K8S_CONTEXT`    | Kubernetes context         |

For the current project, the default values target the local Docker Desktop Kubernetes cluster.

This allows the pipeline structure to be adapted to another application or Kubernetes environment by changing parameters rather than rewriting the pipeline.

---

## 10. Jenkins Setup

Jenkins is configured as a Docker container with access to:

* Docker
* Kubernetes
* The project's Kubernetes kubeconfig

The Jenkins container uses the Docker socket to build and push Docker images.

The Kubernetes kubeconfig is mounted into the Jenkins container as a read-only file.

The current Kubernetes configuration uses:

```text
Kubernetes context: docker-desktop
Namespace: default
```

---

## 11. Docker Hub Credentials

Docker Hub authentication is handled using a Jenkins credential.

Credential ID:

```text
dockerhub-creds
```

The Jenkinsfile accesses the credential using Jenkins' `withCredentials` mechanism.

No Docker Hub password or access token is stored in the Git repository.

A Docker Hub Personal Access Token is used for CI/CD authentication.

---

## 12. GitHub Webhook

GitHub is configured to trigger Jenkins automatically when code is pushed to the repository.

The Jenkins job has the following trigger enabled:

```text
GitHub hook trigger for GITScm polling
```

Because Jenkins is running locally, an ngrok tunnel is used to expose the Jenkins webhook endpoint to GitHub during development.

The webhook flow is:

```text
GitHub Push
    |
    v
GitHub Webhook
    |
    v
ngrok
    |
    v
Jenkins
    |
    v
Pipeline Execution
```

This removes the need to manually start a Jenkins build after every Git push.

---

## 13. CI/CD Trigger Flow

When a developer pushes a change:

```text
git push
    |
    v
GitHub
    |
    v
GitHub Webhook
    |
    v
Jenkins
    |
    v
Checkout
    |
    v
Test
    |
    v
Docker Build
    |
    v
Docker Push
    |
    v
Kubernetes Deployment
    |
    v
Rollout Verification
```

If a required stage fails, subsequent stages are not executed.

For example, if automated tests fail:

```text
Checkout       ✓
Test           ✗
Docker Build   Not executed
Docker Push    Not executed
Kubernetes     Not executed
```

---

## 14. Validation and Test Cases

The project includes both automated application tests and end-to-end CI/CD validation.

### Automated Application Tests

The automated pytest suite contains:

| Test               | Expected Result                           |
| ------------------ | ----------------------------------------- |
| `/` endpoint       | HTTP 200 and expected application message |
| `/health` endpoint | HTTP 200 and `healthy` response           |

The tests can be executed locally using:

```bash
python3 -m pytest tests/ -v
```

A successful local test execution produced:

```text
tests/test_app.py::test_home PASSED
tests/test_app.py::test_health PASSED

2 passed
```

### CI/CD Validation

The following validation scenarios are covered:

| Test Case                      | Expected Result                         |
| ------------------------------ | --------------------------------------- |
| Python syntax validation       | Application compiles successfully       |
| Automated pytest tests         | All application tests pass              |
| Docker image build             | Docker image created successfully       |
| Docker container startup       | Application starts successfully         |
| Application `/` endpoint       | Returns application message             |
| Application `/health` endpoint | Returns `healthy`                       |
| Kubernetes deployment          | 2 replicas become ready                 |
| Kubernetes service             | Application accessible through NodePort |
| Docker Hub push                | Image successfully uploaded             |
| GitHub webhook                 | Jenkins build triggered automatically   |
| Jenkins pipeline               | All stages complete successfully        |
| Kubernetes rollout             | New image deployed successfully         |
| Deployment verification        | Deployment and pods reported healthy    |

---

## 15. Successful Pipeline Validation

A complete Jenkins pipeline execution successfully performed the following stages:

```text
Checkout
    ✓

Test
    ✓

Docker Build
    ✓

Docker Push
    ✓

Kubernetes Deploy
    ✓

Deployment Verification
    ✓
```

The pipeline generates Docker images using the Jenkins build number.

For example:

```text
khushiiii19/finacplus-cicd:<BUILD_NUMBER>
```

The Kubernetes Deployment is updated to the newly generated image and the rollout is verified using:

```bash
kubectl rollout status deployment/finacplus-app
```

The application can also be verified using:

```bash
curl http://localhost:30080/
```

and:

```bash
curl http://localhost:30080/health
```

The expected responses are:

```text
FinacPlus CI/CD Pipeline is working!
```

and:

```text
healthy
```

---

## 16. Error Handling

The Jenkinsfile includes explicit shell failure handling:

```bash
set -e
```

This causes a shell step to stop when a command fails.

The pipeline also uses Jenkins `post` conditions for successful and failed executions.

The Kubernetes deployment stage waits for rollout completion using:

```bash
kubectl rollout status
```

with a timeout.

This prevents the pipeline from reporting a successful deployment when Kubernetes has not completed the rollout successfully.

### Deployment-Aware Automatic Rollback

The pipeline tracks whether a Kubernetes deployment has started and whether the rollout has completed successfully.

If the Kubernetes deployment starts but the rollout fails, Jenkins attempts to roll back the Deployment:

```bash
kubectl rollout undo deployment/finacplus-app
```

The rollback logic does not run for failures that occur before Kubernetes deployment starts, such as:

* Application test failure
* Docker build failure
* Docker push failure

This avoids performing an unnecessary Kubernetes rollback when Kubernetes has not been modified by the current pipeline execution.

---

## 17. Security Considerations

The following security practices are implemented:

* Docker Hub credentials are stored in Jenkins Credentials.
* Docker credentials are not hardcoded in the Jenkinsfile.
* Kubernetes kubeconfig is mounted into Jenkins as read-only.
* Sensitive local files such as `.env` are excluded using `.gitignore`.
* Docker Hub authentication uses a Personal Access Token instead of a password.
* The application container runs as a non-root user using `appuser`.
* The Docker image uses the `python:3.12-slim` base image.

### Production Considerations

The current implementation is designed for a local assignment environment.

For a production implementation, additional security controls should be considered:

* Use a dedicated Jenkins service account.
* Apply least-privilege RBAC permissions to Kubernetes.
* Avoid running Jenkins as root where possible.
* Avoid exposing the Docker socket directly to Jenkins where possible.
* Use secure secret-management solutions.
* Use HTTPS/TLS for external webhook endpoints.
* Use appropriately scoped credentials.
* Regularly rotate credentials and access tokens.
* Use separate Kubernetes namespaces or clusters for different environments.
* Scan container images for vulnerabilities before deployment.

---

## 18. Scalability and Reusability

The pipeline is designed to support different applications and Kubernetes environments.

Instead of hardcoding application-specific deployment values throughout the pipeline, the Jenkinsfile uses parameters such as:

```text
IMAGE_REPO
K8S_DEPLOYMENT
K8S_CONTAINER
K8S_NAMESPACE
K8S_CONTEXT
```

This allows the same pipeline structure to be reused for:

* Different Docker repositories
* Different Kubernetes Deployments
* Different namespaces
* Different Kubernetes contexts

For larger environments, this approach could be extended using:

* Jenkins Shared Libraries
* Environment-specific configuration
* Separate credentials
* Dedicated Kubernetes namespaces
* Separate development, staging, and production environments

---

## 19. Monitoring and Logging Recommendations

The current assignment focuses on CI/CD automation and deployment verification.

For a production environment, monitoring and logging could be extended using tools such as:

* Prometheus for metrics collection
* Grafana for dashboards
* Loki or an ELK-based stack for centralized logs
* Kubernetes events and container logs for troubleshooting
* Jenkins build history and console logs for CI/CD monitoring

Useful metrics to monitor would include:

* Pipeline success/failure rate
* Pipeline execution duration
* Deployment rollout failures
* Pod restarts
* Application health
* CPU and memory utilization
* Container restart frequency

---

## 20. Troubleshooting

### Jenkins Build Does Not Start

Check:

```text
GitHub webhook configuration
Jenkins webhook trigger
GitHub webhook delivery status
ngrok tunnel status
```

For the local Jenkins environment, the ngrok tunnel must remain active.

---

### Docker Build Fails

Check:

```bash
docker version
docker images
```

Also verify that Jenkins has access to the Docker daemon.

---

### Docker Push Fails

Check:

* Docker Hub username
* Jenkins credential ID
* Docker Hub Personal Access Token
* Token permissions
* Docker repository name

---

### Automated Tests Fail

Run the tests locally:

```bash
python3 -m pytest tests/ -v
```

Check:

* Flask application code
* Test expectations
* Python version
* Test dependencies in `requirements-test.txt`

---

### Kubernetes Deployment Fails

Check:

```bash
kubectl get deployments
kubectl get pods
kubectl describe deployment finacplus-app
```

Also verify the Kubernetes context:

```bash
kubectl config current-context
```

---

### Pod Is Not Running

Check:

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

Common causes include:

* Incorrect image name
* Image pull failure
* Application startup failure
* Incorrect port configuration
* Failed health probes

---

## 21. Rollback

Kubernetes maintains rollout history for Deployments.

To view rollout history:

```bash
kubectl rollout history deployment/finacplus-app
```

The Jenkins pipeline also attempts an automatic rollback when a Kubernetes deployment has started but the rollout does not complete successfully.

For a manual rollback, use:

```bash
kubectl rollout undo deployment/finacplus-app
```

The rollout status can then be checked using:

```bash
kubectl rollout status deployment/finacplus-app
```

---

## 22. Useful Commands

### Check Kubernetes Nodes

```bash
kubectl get nodes
```

### Check Pods

```bash
kubectl get pods
```

### Check Deployment

```bash
kubectl get deployment
```

### Check Service

```bash
kubectl get service
```

### Check Application

```bash
curl http://localhost:30080/
```

### Check Health

```bash
curl http://localhost:30080/health
```

### Run Automated Tests

```bash
python3 -m pytest tests/ -v
```

### View Application Logs

```bash
kubectl logs <pod-name>
```

### Check Rollout Status

```bash
kubectl rollout status deployment/finacplus-app
```

### View Rollout History

```bash
kubectl rollout history deployment/finacplus-app
```

---

## 23. Cleanup

To remove the Kubernetes resources:

```bash
kubectl delete -f k8s/
```

To remove a locally stored Docker image generated by Jenkins:

```bash
docker rmi khushiiii19/finacplus-cicd:<BUILD_NUMBER>
```

For example:

```bash
docker rmi khushiiii19/finacplus-cicd:17
```

Docker Hub images can be managed separately through the Docker Hub repository.

---

## 24. Conclusion

This project demonstrates an automated CI/CD workflow in which a GitHub code change can trigger Jenkins, validate the application, run automated tests, build and publish a Docker image, deploy the image to Kubernetes, verify the resulting deployment, and attempt an automatic rollback if the Kubernetes rollout fails.

The pipeline uses Jenkins Pipeline as Code with Groovy and parameterized deployment configuration so that the approach can be adapted to different repositories and Kubernetes environments.

The implementation also incorporates basic container security by running the application as a non-root user and separates CI test dependencies from the production application dependencies.
