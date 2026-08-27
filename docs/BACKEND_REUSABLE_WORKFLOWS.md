# Reusable Backend Workflows

This repository contains reusable GitHub Actions workflows that backend services can call via `workflow_call`.

The default behavior and example wiring are based on the current `recipe-management-ai-service` pipeline on `main`. Other services can override inputs where they differ.

## Available workflows

- `.github/workflows/backend-java-ci.yml`
  - Builds Java 21 Spring Boot services
  - Configures Maven for GitHub Packages
  - Runs tests, verification, packaging
  - Uses Git SHA (`short-sha`) for build versioning
  - Optionally runs SonarCloud
  - Uploads build artifacts on `main`
- `.github/workflows/backend-java-cloud-run-cd.yml`
  - Builds and pushes Docker images to Artifact Registry
  - Deploys to Cloud Run using Git SHA tag
  - Optionally runs OWASP ZAP and post-deployment smoke tests

## Expected files in a service repo

The reusable workflows assume the consuming service repository contains:

- `pom.xml`
- `Dockerfile`
- `test-deployment.sh` for post-deployment smoke tests

## Example consumer workflow

A backend service can create a thin wrapper workflow like this:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  packages: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}

jobs:
  ci:
    uses: theandiman/recipe-management/.github/workflows/backend-java-ci.yml@main
    with:
      run_sonar: false
    secrets: inherit

  cd:
    needs: ci
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    uses: theandiman/recipe-management/.github/workflows/backend-java-cloud-run-cd.yml@main
    with:
      service_name: recipe-ai-service
      artifact_registry_repository: recipe-ai
      deployment_test_script: test-deployment.sh
      run_security_scan: true
      run_post_deploy_tests: true
    secrets: inherit
```

## Required secrets in consuming repos

- `GCP_SA_KEY` or WIF credentials (`GCP_WORKLOAD_IDENTITY_PROVIDER`, `GCP_DEPLOY_SA`)
- `GCP_PROJECT_ID`
- `SONAR_TOKEN` when `run_sonar: true`

The workflows use the built-in `GITHUB_TOKEN` for GitHub Packages access.
