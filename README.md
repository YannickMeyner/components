# GitLab CI/CD Components for ACR-based Workflows

This repository contains reusable GitLab CI/CD components designed to automate the container build, push, security scanning, Software Bill of Materials (SBOM) generation, and release management processes with **Azure Container Registry (ACR)** integration.

## Available Components

| Component | Description |
|:----------|:------------|
| `acr-build-component.yml` | Build a Docker image from a specified directory and save it as a `.tar` archive. |
| `acr-push-component.yml` | Push the built Docker image to Azure Container Registry. |
| `auto-release-notes-component.yml` | Automatically generate and publish release notes based on Git tags. |
| `generate-sbom-component.yml` | Generate SBOM (Software Bill of Materials) for the Docker image and project directory. |
| `scan-cve-component.yml` | Scan Docker images for known vulnerabilities (CVEs) and generate reports. |

---

## Usage

Each component is a modular GitLab CI/CD job that can be included into your pipelines.

Example inclusion in your `.gitlab-ci.yml`:

```yaml
include:
  - component: gitlab.fhnw.ch/dps/module/fs25/fs25-group13/fs13-git-components/acr-build-component@main
    inputs:
      acr_login_server: "$ACR_LOGIN_SERVER"
      acr_username: "$ACR_USERNAME"
      acr_password: "$ACR_PASSWORD"
      project_name: "$CI_PROJECT_NAME"
      image_tag: "$CI_COMMIT_SHORT_SHA"
      directory: "."
```

---

##  Component Details

### 1. `acr-build-component.yml`

Builds a Docker image and saves it as a `.tar` file for later use.

**Inputs:**
- `acr_login_server` - ACR login server URL
- `acr_username` - ACR username
- `acr_password` - ACR password
- `project_name` - Project name used for image naming
- `image_tag` - Image tag
- `directory` (optional) - Directory with the Dockerfile (default: `.`)
- `build_stage` (optional) - Pipeline stage for the build (default: `build`)

**Artifacts:**
- Saved Docker image tar file

---

### 2. `acr-push-component.yml`

Pushes a previously built image to the Azure Container Registry.

**Inputs:**
- Same as `acr-build-component`
- `push_stage` (optional) - Stage for push job (default: `deploy`)

**Rules:**
- Only runs on the `main` branch or when a Git tag is pushed.

---

### 3. `auto-release-notes-component.yml`

Generates a GitLab release based on Git commit history.

**Inputs:**
- `release_stage` (optional) - Stage for the release (default: `release`)
- `draft_mode` (optional) - Create as draft or publish immediately (default: `true`)

**Notes:**
- Requires the `GITLAB_API_TOKEN` environment variable with `api` scope.

---

### 4. `generate-sbom-component.yml`

Creates SBOM files for both the Docker image and the project directory using **Anchore Syft**.

**Inputs:**
- `acr_login_server`
- `project_name`
- `image_tag`
- `directory` (optional) - Source code directory (default: `.`)
- `sbom_stage` (optional) - Stage for SBOM generation (default: `test`)

**Artifacts:**
- `sbom-project.json`
- `sbom-image.json`

---

### 5. `scan-cve-component.yml`

Scans the built image for vulnerabilities using **Anchore Grype**.

**Inputs:**
- `acr_login_server`
- `project_name`
- `image_tag`
- `cve_stage` (optional) - Stage for CVE scanning (default: `test`)

**Artifacts:**
- `cve-report.json`

---

## Security Notes
- Credentials (`acr_password`, `GITLAB_API_TOKEN`, etc.) must be stored securely as GitLab CI/CD variables.
- The `docker:dind` service is required for components dealing with Docker operations.
