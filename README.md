# Github-Actions

[![DevSecOps Pipeline](https://github.com/Praveen48589/Github-Actions/actions/workflows/devsecops-pipeline.yml/badge.svg)](https://github.com/Praveen48589/Github-Actions/actions/workflows/devsecops-pipeline.yml)
[![Portfolio Deploy](https://github.com/Praveen48589/Github-Actions/actions/workflows/portfolio-deploy.yml/badge.svg)](https://github.com/Praveen48589/Github-Actions/actions/workflows/portfolio-deploy.yml)

A hands-on DevOps project that demonstrates how to build, scan, containerize, publish, and deploy a small Flask application with Docker and GitHub Actions.

The repository also includes a static portfolio page that can be deployed to GitHub Pages with a custom domain.

## What This Project Shows

- Flask application served with Gunicorn inside a Docker container
- Health endpoint for deployment checks and uptime monitoring
- Docker image build and publish flow through Docker Hub
- Reusable GitHub Actions workflows for code quality, security, image scanning, and deployment
- DevSecOps pipeline that gates deployment behind linting, SAST, secret scanning, dependency auditing, Dockerfile linting, and Trivy image scanning
- EC2-style remote server deployment over SSH using Docker Compose
- Optional GitHub Pages deployment for the static portfolio page

## Project Structure

```text
.
|-- app.py
|-- templates/
|   `-- index.html
|-- Dockerfile
|-- docker-compose.yml
|-- requirements.txt
|-- index.html
|-- CNAME
|-- README.md
`-- .github/
    `-- workflows/
        |-- devsecops-pipeline.yml
        |-- code-quality.yml
        |-- secrets-scan.yml
        |-- dependency-scan.yml
        |-- docker-lint.yml
        |-- docker-build-push.yml
        |-- image-scan.yml
        |-- deploy-to-server.yml
        |-- portfolio-deploy.yml
        |-- python-matrix.yml
        |-- cicd.yml
        |-- hello.yml
        `-- deploy-app.yml
```

## Application Overview

The Flask app is intentionally small so the CI/CD system stays easy to understand.

| Route | Purpose |
| --- | --- |
| `/` | Renders `templates/index.html`, the deployed CI/CD success page. |
| `/health` | Returns a plain-text health response: `Server is up and running`. |

Runtime stack:

- Python
- Flask
- Gunicorn
- Docker

Quality and security tools:

- Flake8 for linting
- Bandit for Python SAST
- pip-audit for dependency vulnerability checks
- Gitleaks for secret scanning
- Hadolint for Dockerfile linting
- Trivy for container image vulnerability scanning

## Quick Start

### 1. Clone The Repository

```bash
git clone https://github.com/Praveen48589/Github-Actions.git
cd Github-Actions
```

### 2. Create A Virtual Environment

```bash
python -m venv .venv
```

Activate it:

```bash
# Windows PowerShell
.\.venv\Scripts\Activate.ps1

# macOS/Linux
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Run The Flask App Locally

```bash
python -m flask --app app run --debug
```

Open:

```text
http://127.0.0.1:5000
```

Check health:

```bash
curl http://127.0.0.1:5000/health
```

## Run With Docker

Build the image:

```bash
docker build -t github-actions-app .
```

Run the container:

```bash
docker run --rm -p 8080:80 github-actions-app
```

Open:

```text
http://localhost:8080
```

Health check:

```bash
curl http://localhost:8080/health
```

## Docker Compose Deployment

The included `docker-compose.yml` is designed for a deployment server that pulls a published image from Docker Hub:

```yaml
services:
  web:
    image: ${DOCKERHUB_USERNAME}/github-actions-app:${DOCKER_TAG}
    ports:
      - "80:80"
```

Required environment variables:

| Variable | Description |
| --- | --- |
| `DOCKERHUB_USERNAME` | Docker Hub username or organization. |
| `DOCKER_TAG` | Image tag to deploy. In the pipeline this is the GitHub commit SHA. |

Example:

```bash
export DOCKERHUB_USERNAME=your-dockerhub-username
export DOCKER_TAG=your-image-tag
docker compose up -d
```

## CI/CD Pipeline

The main production workflow is:

```text
.github/workflows/devsecops-pipeline.yml
```

It runs on every push to `main` and chains reusable workflows in this order:

1. `code-quality.yml`
   - Installs dependencies
   - Runs Flake8 against `app.py`
   - Runs Bandit against `app.py`
   - Tests against Python `3.11`, `3.12`, and `3.13`

2. `secrets-scan.yml`
   - Runs Gitleaks across the full Git history

3. `dependency-scan.yml`
   - Installs `pip-audit`
   - Audits dependencies from `requirements.txt`

4. `docker-lint.yml`
   - Runs Hadolint against the Dockerfile

5. `docker-build-push.yml`
   - Logs in to Docker Hub
   - Builds the Docker image
   - Pushes the image as:

```text
${DOCKERHUB_USERNAME}/github-actions-app:${GITHUB_SHA}
```

6. `image-scan.yml`
   - Logs in to Docker Hub
   - Scans the pushed image with Trivy
   - Fails the pipeline on `CRITICAL` vulnerabilities

7. `deploy-to-server.yml`
   - Connects to the remote server over SSH
   - Installs Docker and Docker Compose if needed
   - Copies `docker-compose.yml` to `~/devops`
   - Pulls the newly built image
   - Recreates the running container

## Required GitHub Configuration

Add these in the repository settings before running the full pipeline.

### Repository Variables

| Name | Used For |
| --- | --- |
| `DOCKERHUB_USERNAME` | Docker image namespace and Docker Hub login username. |

### Repository Secrets

| Name | Used For |
| --- | --- |
| `DOCKERHUB_TOKEN` | Docker Hub authentication token. |
| `EC2_SSH_HOST` | Public IP or DNS name of the deployment server. |
| `EC2_SSH_USERNAME` | SSH user on the deployment server. |
| `EC2_SSH_PRIVATE_KEY` | Private SSH key with access to the deployment server. |

`GITHUB_TOKEN` is provided automatically by GitHub Actions and is used by the Gitleaks workflow.

## Deployment Flow

After a successful push to `main`, the pipeline:

1. Validates Python code quality and security.
2. Checks for leaked secrets and vulnerable Python dependencies.
3. Validates the Dockerfile.
4. Builds and pushes a Docker image to Docker Hub.
5. Scans the pushed image for critical vulnerabilities.
6. SSHs into the server.
7. Pulls the new image by commit SHA.
8. Recreates the container on port `80`.

The deployed app should then be available at:

```text
http://<server-ip-or-domain>
```

Health endpoint:

```text
http://<server-ip-or-domain>/health
```

## Static Portfolio Page

This repository also contains a root-level `index.html` file for a personal portfolio site.

The portfolio deployment workflow is:

```text
.github/workflows/portfolio-deploy.yml
```

It can be run manually with `workflow_dispatch` and publishes the repository content to GitHub Pages.

The custom domain is configured through:

```text
CNAME
```

Current domain:

```text
praveentomar.shop
```

Important distinction:

- `templates/index.html` is used by the Flask app.
- `index.html` at the repository root is used by GitHub Pages.

## Additional Workflows

| Workflow | Purpose |
| --- | --- |
| `hello.yml` | Manual learning workflow that prints basic runner output. |
| `cicd.yml` | Manual demo workflow showing code, build, test, and conditional prod deploy stages. |
| `python-matrix.yml` | Manual Python lint matrix across Python `3.9` through `3.13`. |
| `deploy-app.yml` | Older/experimental self-hosted deployment workflow. The main deployment path is `deploy-to-server.yml`. |

## Useful Commands

Run lint locally:

```bash
flake8 app.py
```

Run Bandit locally:

```bash
bandit -r app.py
```

Run dependency audit locally:

```bash
pip install pip-audit
pip-audit -r requirements.txt
```

Build and run locally with Docker:

```bash
docker build -t github-actions-app .
docker run --rm -p 8080:80 github-actions-app
```

## Troubleshooting

### Docker Compose Cannot Find The Image

Confirm that:

- `DOCKERHUB_USERNAME` is set correctly.
- `DOCKER_TAG` matches an image tag that exists in Docker Hub.
- The Docker Hub token has permission to pull the image.

### Deployment Runs But The Site Is Not Reachable

Check that:

- Port `80` is open in the server firewall or cloud security group.
- Docker is running on the server.
- The container is running with `docker ps`.
- The application responds to `/health`.

### Trivy Blocks The Pipeline

The image scanner fails on critical vulnerabilities. Rebuild with patched base images or updated dependencies, then push again.

### GitHub Pages Shows The Portfolio Instead Of The Flask App

That is expected. GitHub Pages serves the root `index.html`; the Flask app serves `templates/index.html` from the Docker container.

## Roadmap Ideas

- Add automated Flask route tests with `pytest`.
- Add a Docker healthcheck for `/health`.
- Pin dependency versions for repeatable builds.
- Add release tags alongside commit-SHA Docker tags.
- Add deployment smoke tests after `docker compose up`.

## Author

Praveen Tomar

- GitHub: [Praveen48589](https://github.com/Praveen48589)
- Portfolio domain: [praveentomar.shop](https://praveentomar.shop)
