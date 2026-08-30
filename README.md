# Movie Picture Pipeline

## Project Overview

This project implements automated Continuous Integration and Continuous Deployment pipelines for a movie catalog application. The application consists of:

- A React and TypeScript frontend.
- A Python Flask backend API.
- Docker images stored in Amazon Elastic Container Registry.
- Kubernetes workloads deployed to Amazon Elastic Kubernetes Service.
- Infrastructure provisioned with Terraform.
- GitHub Actions workflows for linting, testing, building, publishing, and deploying both applications.

The deployed frontend retrieves the movie list from the backend `/movies` endpoint.

## Architecture

The deployment uses the following components:

- GitHub Actions for CI/CD automation.
- Amazon ECR repositories named `frontend` and `backend`.
- Amazon EKS cluster named `cluster`.
- Kubernetes Deployments and LoadBalancer Services.
- Kustomize for applying the image tagged with the current Git commit SHA.
- Terraform 1.3.9 with AWS provider 5.98.0.
- Amazon Linux 2023 EKS worker nodes running Kubernetes 1.34.

## Continuous Integration

The repository contains two CI workflows:

- `.github/workflows/frontend-ci.yml`
- `.github/workflows/backend-ci.yml`

Both workflows:

- Run on pull requests targeting `main`.
- Run only when the corresponding application changes.
- Support manual execution with `workflow_dispatch`.
- Execute linting and testing in parallel.
- Build the Docker image only after linting and testing succeed.

The frontend Docker build uses the `REACT_APP_MOVIE_API_URL` build argument.

## Continuous Deployment

The repository contains two CD workflows:

- `.github/workflows/frontend-cd.yml`
- `.github/workflows/backend-cd.yml`

Both workflows:

- Run on pushes to `main` when the corresponding application changes.
- Support manual execution.
- Run validation jobs before building.
- Tag Docker images with the GitHub commit SHA.
- Authenticate with Amazon ECR.
- Push the tagged image to ECR.
- Configure access to the EKS cluster.
- Update the Kubernetes image with Kustomize.
- Apply the generated Kubernetes manifests.
- Report deployment failures through GitHub Actions annotations.

## GitHub Configuration

The CD workflows require these repository secrets:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

The frontend workflow also uses this repository variable:

- `REACT_APP_MOVIE_API_URL`

The variable must contain the externally accessible backend URL without a trailing slash.

Sensitive credentials and Terraform state files must never be committed to Git.

## Local Validation

### Frontend

```bash
cd starter/frontend
npm ci
npm run lint
CI=true npm test -- --watchAll=false
docker build \
  --build-arg REACT_APP_MOVIE_API_URL=http://localhost:5000 \
  --tag mp-frontend:latest \
  .
```

### Backend

```bash
cd starter/backend
python -m pip install pipenv
pipenv sync --dev
pipenv run lint
pipenv run test
docker build --tag mp-backend:latest .
```

## Infrastructure Deployment

Terraform 1.3.9 is required.

```bash
terraform -chdir=setup/terraform init
terraform -chdir=setup/terraform validate
terraform -chdir=setup/terraform plan
terraform -chdir=setup/terraform apply
```

Terraform provisions the VPC, EKS cluster, managed node group, ECR repositories, and IAM resources.

## Deployment Verification

```bash
kubectl get nodes
kubectl get pods
kubectl get deployments
kubectl get services -o wide
```

Verify the backend:

```bash
curl http://BACKEND_LOAD_BALANCER/movies
```

Open the frontend LoadBalancer address and confirm that these movies appear:

- Top Gun: Maverick
- Sonic the Hedgehog
- A Quiet Place

## Deployment Evidence

Evidence is stored in the `screenshots` directory:

- Pull-request CI checks.
- Backend CD success.
- Frontend CD success.
- Running frontend application.

## Releasing Changes

For a new release, modify the frontend or backend, open a pull request against `main`, wait for CI to pass, and merge it. The corresponding CD workflow builds, publishes, and deploys the SHA-tagged image.

## Cleanup

After collecting the evidence and submitting the project:

```bash
terraform -chdir=setup/terraform destroy
```