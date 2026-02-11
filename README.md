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
