# Movie Picture Pipeline – CI/CD with GitHub Actions, Docker & Kubernetes

Automated CI/CD pipelines for a two-part web app (React (JavaScript) frontend + Python/Flask backend).
Every change is linted, tested, built into a Docker image tagged with the git commit SHA,
pushed to Amazon ECR and deployed to an AWS EKS (Kubernetes) cluster.

<!-- TODO: add a screenshot of a green GitHub Actions run here -->

## What I built

Four GitHub Actions workflows in [`.github/workflows/`](.github/workflows):

| Workflow | Runs on | What it does |
|---|---|---|
| `frontend-ci.yaml` | Pull requests to `main` (frontend changes) + manual | ESLint and Jest tests in parallel → Docker build only if both pass |
| `backend-ci.yaml` | Pull requests to `main` + manual | flake8 lint and pytest in parallel (with pip caching) → Docker build only if both pass |
| `frontend-cd.yaml` | Push to `main` (frontend changes) + manual | Lint + test → build image tagged with git SHA → push to ECR → deploy to EKS with kustomize |
| `backend-cd.yaml` | Push to `main` (backend changes) + manual | Lint + test → build image tagged with git SHA → push to ECR → deploy to EKS with kustomize |

Key points:
- **Fail fast:** the build and deploy jobs only run when lint and tests pass.
- **Traceable releases:** each image is tagged with the commit SHA, so you can see exactly which
  commit is running in the cluster.
- **No secrets in code:** AWS credentials come from GitHub Secrets; account ID and API URL from
  GitHub Variables.

## Tech
GitHub Actions · Docker · Kubernetes (AWS EKS) · kustomize · Amazon ECR · Terraform · Python/Flask ·
pytest · flake8 · React · Jest · ESLint

## Repository layout
```
.github/workflows/   CI/CD pipelines (my work)
starter/frontend/    React app + Dockerfile + k8s manifests
starter/backend/     Flask API + Dockerfile + k8s manifests
setup/terraform/     AWS infrastructure (EKS cluster, ECR repos, IAM user)
```

## Run it yourself
1. Create the AWS infrastructure: `cd setup/terraform && terraform apply`
2. Give the GitHub Actions IAM user access to the cluster: `cd setup && ./init.sh`
3. In the GitHub repo settings, add:
   - **Secrets:** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
   - **Variables:** `AWS_ACCOUNT_ID`, `BACKEND_API_URL` (the backend's load-balancer URL)
4. Push to `main` or start a workflow manually from the **Actions** tab.
5. Remember to run `terraform destroy` afterwards – the AWS resources cost money.

Full original task description: [`docs/original-project-instructions.md`](docs/original-project-instructions.md)

## Credits
Starter application and infrastructure code provided by Udacity (see [LICENSE.md](LICENSE.md)).
CI/CD workflows written by me (Krish Lamba).
