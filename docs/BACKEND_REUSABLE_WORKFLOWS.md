# Reusable Backend Workflows

This repository contains reusable GitHub Actions workflows that backend services can call via `workflow_call`.

The default behavior and example wiring are based on the current `recipe-management-ai-service` pipeline on `main`. Other services can override inputs where they differ.

## Available workflows

- `.github/workflows/backend-java-ci.yml`
  - Builds Java 21 Spring Boot services
  - Configures Maven for GitHub Packages
  - Runs tests, verification, packaging
  - Optionally runs SonarCloud
  - Uploads build artifacts on `main`
- `.github/workflows/backend-java-cloud-run-cd.yml`
  - Builds and pushes Docker images to Artifact Registry
  - Deploys to Cloud Run
  - Optionally runs OWASP ZAP and post-deployment smoke tests
  - Optionally bumps the next development version when `version.sh` is present

## Expected files in a service repo

The reusable workflows assume the consuming service repository contains:

- `pom.xml`
- `Dockerfile`
- `test-deployment.sh` for post-deployment smoke tests
- `version.sh` when using `version-script` versioning

If `version.sh` is absent, the CI workflow automatically falls back to short-SHA versioning, regardless of the `version_strategy` input.

Use `version_strategy: short-sha` only when you want to force short-SHA versioning even when a `version.sh` script is present. This mirrors the current `ai-service` behavior on `main`: script-based versioning when `version.sh` exists, with automatic short-SHA fallback otherwise.

## Example consumer workflow

A backend service can create a thin wrapper workflow like this. This example intentionally matches the current `recipe-management-ai-service` setup on `main`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: write
  packages: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}

jobs:
  ci:
    uses: theandiman/recipe-management/.github/workflows/backend-java-ci.yml@v1
    with:
      version_strategy: version-script
      run_sonar: false
    secrets: inherit

  cd:
    needs: ci
    uses: theandiman/recipe-management/.github/workflows/backend-java-cloud-run-cd.yml@v1
    with:
      service_name: recipe-ai-service
      artifact_registry_repository: recipe-ai
      deployment_test_script: test-deployment.sh
      run_security_scan: true
      run_post_deploy_tests: true
      run_version_bump: true
    secrets: inherit
```

## Service-specific inputs

### AI service style wiring

```yaml
with:
  service_name: recipe-ai-service
  artifact_registry_repository: recipe-ai
  version_strategy: version-script
```

### Storage service style wiring

This is a deliberate extension from the `ai-service` baseline, adding SonarCloud inputs used by storage service:

```yaml
with:
  service_name: recipe-storage-service
  artifact_registry_repository: recipe-storage
  version_strategy: version-script
  run_sonar: true
  sonar_project_key: theandiman_recipe-management-service
```

## Required secrets in consuming repos

- `GCP_SA_KEY` — raw service account key JSON (not base64-encoded)
- `GCP_PROJECT_ID`
- `SONAR_TOKEN` when `run_sonar: true`
- `GIT_PAT` — PAT/fine-grained token required by the reusable CD workflow so version bump commits can be pushed back to protected `main`

The workflows still use the built-in `GITHUB_TOKEN` for GitHub Packages access, but the reusable CD workflow assumes consuming repositories provide `GIT_PAT` for the version bump checkout/push.
