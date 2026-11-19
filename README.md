## Complete CI/CD Pipeline with EKS and AWS ECR

This project was a part of the TWN DevOps Bootcamp.

## Technologies used:
Kubernetes, Jenkins, AWS EKS, AWS ECR, Java, Maven, Linux, Docker, Git

## Project Description:
- Create private AWS ECR Docker repository
- Adjust Jenkinsfile to build and push Docker Image to AWS ECR
- Integrate deploying to K8s cluster in the CI/CD pipeline from AWS ECR private registry
- So the complete CI/CD project we build has the following configuration:
  
  a) CI step: Increment version
  
  b) CI step: Build artifact for Java Maven application

  c) CI step: Build and push Docker image to AWS ECR

  d) CD step: Deploy new application version to EKS cluster

  e) CD step: Commit the version update

### 1. Add a Kubernetes Deploy Stage

1. **Create K8s manifests** (`deployment.yaml`, `service.yaml`) under a new `kubernetes/` folder.
   - Use placeholders like `${IMAGE_NAME}` and `${APP_NAME}` for the image tag and application name.
2. **Define environment variables** in the Jenkinsfile (e.g., `IMAGE_NAME`, `APP_NAME`).
   - Set them in an `environment` block or per‑stage block.
3. **Install `envsubst`** in the Jenkins agent container (via `apt-get install gettext`).
   - Use it to replace the placeholders in the YAML files at runtime.
4. **Apply the rendered manifests**:
   ```groovy
   sh "envsubst < kubernetes/deployment.yaml | kubectl apply -f -"
   sh "envsubst < kubernetes/service.yaml    | kubectl apply -f -"
   ```
5. **Create a Docker‑registry secret** (once) so the cluster can pull private images:
   ```bash
   kubectl create secret docker-registry my-registry-key \
       --docker-server=docker.io \
       --docker-username=<user> \
       --docker-password=<pass>
   ```
   - Reference the secret in the pod spec (`imagePullSecrets`).

### 2. Switch from Docker Hub to AWS ECR

1. **Create an ECR repository** (e.g., `java-maven-app`) in the AWS console.
2. **Generate ECR credentials**:
   ```bash
   aws ecr get-login-password --region <region>
   ```
   - Username is always `AWS`; password is the output above.
3. **Add these credentials to Jenkins** (type “Username with password”, ID `ecr‑creds`).
4. **Create a matching Kubernetes secret** for ECR:
   ```bash
   kubectl create secret docker-registry ecr-registry-key \
       --docker-server=<account-id>.dkr.ecr.<region>.amazonaws.com \
       --docker-username=AWS \
       --docker-password=$(aws ecr get-login-password)
   ```
5. **Update the Jenkinsfile**:
   - Add `DOCKER_REPO` (full ECR URI) and `DOCKER_REPO_SERVER` (ECR server part) as environment
     variables.
   - Use the new credentials ID (`ecr‑creds`) for Docker login.
   - Tag and push the image to `${DOCKER_REPO}:${IMAGE_NAME}`.
   - Pass `DOCKER_REPO_SERVER` as the third argument to `docker login`.
6. **Reference the ECR secret** (`ecr-registry-key`) in the deployment YAML (`imagePullSecrets`).

### 3. Run the Pipeline

- Commit changes, trigger the Jenkins job.
- Pipeline builds the JAR, creates a Docker image, pushes it to ECR, substitutes variables in the K8s
  manifests, applies them, and the new pod runs the image pulled from ECR.

**Result:** A fully automated CI/CD flow that builds, versions, pushes Docker images to AWS ECR, and
deploys them to an EKS‑hosted Kubernetes cluster, with all image/tag values injected dynamically via
environment variables.

---

### What did I build?

I created a **complete end‑to‑end CI/CD pipeline** that:

1. **Compiles a Java‑Maven application** and packages it as a Docker image.
2. **Version‑increments** the image on every run, ensuring traceable releases.
3. **Pushes the image to a container registry** – first to Docker Hub, then migrated to **AWS Elastic
   Container Registry (ECR)**.
4. **Deploys the image to a Kubernetes cluster** (EKS) using dynamically‑generated Kubernetes
   manifests (Deployment & Service).
5. **Handles private‑registry authentication** by creating Docker‑registry secrets both in Jenkins
   (for `docker login`) and in the Kubernetes cluster (for `imagePullSecrets`).
6. **Uses environment‑variable substitution (`envsubst`)** to inject the current image tag and
   application name into the YAML files at runtime, keeping the manifests reusable and source‑control
   friendly.

All pipeline logic lives in a single **Jenkinsfile**, making the process transparent, reproducible,
and easy to extend (e.g., moving code to a shared library).

