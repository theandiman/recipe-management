# Recipe Management Platform

A cloud-native recipe management platform built with microservices architecture, featuring AI-powered recipe generation, intelligent image creation, and comprehensive recipe storage and sharing capabilities.

## 🏗️ Architecture

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed architecture overview and diagrams.

## 📦 Repositories

This monorepo organization includes the following services:

- **[recipe-management-frontend](https://github.com/theandiman/recipe-management-frontend)** - React/TypeScript web application
- **[recipe-management-ai-service](https://github.com/theandiman/recipe-management-ai-service)** - AI recipe generation service (Java/Spring Boot)
- **[recipe-management-service](https://github.com/theandiman/recipe-management-service)** - Recipe management service (Java/Spring Boot)
- **[recipe-management-infrastructure](https://github.com/theandiman/recipe-management-infrastructure)** - Infrastructure as Code (Terraform/Pulumi)
- **[recipe-management-shared](https://github.com/theandiman/recipe-management-shared)** - Shared models and types (TypeScript/Java)

## 🚀 Key Features

- **AI Recipe Generation**: Generate recipes using Google's Gemini AI based on ingredients, dietary preferences, and constraints
- **AI Image Generation**: Automatically create recipe images using Imagen 3
- **Recipe Storage**: Store, manage, and share recipes with Firebase integration
- **Public Sharing**: Share recipes with the community with visibility controls
- **Responsive UI**: Modern, mobile-friendly interface built with React and Tailwind CSS
- **Real-time Updates**: Live recipe updates and synchronization
- **Nutritional Information**: Comprehensive nutritional data for all recipes
- **Dietary Preferences**: Support for various dietary restrictions and allergies

## 🛠️ Tech Stack

### Frontend
- React 19 with TypeScript
- Tailwind CSS for styling
- Redux Toolkit for state management
- Firebase Authentication
- Vite for build tooling
- Vitest & Playwright for testing

### Backend Services
- Java 21 with Spring Boot 6
- Google Cloud Run for deployment
- Firestore for data storage
- Google Gemini AI for recipe generation
- Google Imagen for image generation
- OpenAPI/Swagger documentation

### Infrastructure
- Terraform for Google Cloud resources
- Pulumi for Firebase configuration
- GitHub Actions for CI/CD
- Cloud Build for container builds
- Artifact Registry for container images

### Shared Components
- TypeScript/Java dual-language support
- Consistent data models across services
- Maven & npm package distribution via GitHub Packages

## 🔧 Development

### Prerequisites
- Node.js 20+
- Java 21 (Temurin recommended)
- Maven 3.9+
- Firebase CLI
- Google Cloud CLI (gcloud)
- Terraform 1.5+

### Configuration Management

This repo contains a shared Renovate configuration to manage dependencies across all repositories. The base configuration is defined in [`default.json`](./default.json) and extended by each service via their own `renovate.json`.

To use the shared config in a new repo, create a `renovate.json` in its root:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>theandiman/recipe-management"
  ]
}
```

Any repo-specific overrides (e.g. extra package rules or label sets) can be added alongside the `extends` key.

**Renovate Features:**
- Automated dependency updates
- Security vulnerability alerts
- Grouped updates for related packages (Spring, Firebase, React, build tools, testing libraries)
- Auto-merge for minor and patch updates with `platformAutomerge`
- Scheduled updates during off-hours (Europe/London timezone)
- PRs rebased when behind base branch

### Shared GitHub Actions

This repo can also act as the source of reusable GitHub Actions workflows for backend services.

The initial reusable backend workflow design is based on the current `recipe-management-ai-service` pipeline on `main`, with optional inputs for service-specific differences.

- `./.github/workflows/backend-java-ci.yml` provides a reusable Java/Spring Boot CI pipeline
- `./.github/workflows/backend-java-cloud-run-cd.yml` provides a reusable Cloud Run CD pipeline
- `./docs/BACKEND_REUSABLE_WORKFLOWS.md` shows how service repos can wire them in later

## 📚 Documentation

- [Architecture Overview](./ARCHITECTURE.md)
- [Reusable Backend Workflows](./docs/BACKEND_REUSABLE_WORKFLOWS.md)
- [API Documentation - AI Service](https://theandiman.github.io/recipe-management-ai-service/)
- [API Documentation - Recipe Management Service](https://theandiman.github.io/recipe-management-service/)

## 🔐 Security

- Firebase Authentication for user management
- Secret scanning with Gitleaks
- Dependency vulnerability scanning with Renovate
- SonarCloud code quality and security analysis
- Google Cloud IAM for service permissions


## 🔗 Links

- **Dev environment**: https://recipe-mgmt-dev.web.app
- **AI Service API dev environment**: https://recipe-ai-service-341866919859.europe-west2.run.app
- **Storage Service AP dev environmentI**: https://recipe-storage-service-htubs7zkna-nw.a.run.app
