# Architecture Overview

## System Architecture

The Recipe Management Platform is built using a microservices architecture deployed on Google Cloud Platform (GCP), with a modern web frontend hosted on Firebase.

```mermaid
graph TB
    subgraph "Client Layer"
        WebUI[Web Application<br/>React + TypeScript]
    end
    
    subgraph "Authentication"
        Firebase[Firebase Auth<br/>JWT Tokens]
    end
    
    subgraph "API Gateway / CDN"
        CloudCDN[Cloud CDN<br/>Firebase Hosting]
    end
    
    subgraph "Backend Services - Cloud Run"
        AIService[AI Service<br/>Java + Spring Boot<br/>Recipe Generation]
        StorageService[Storage Service<br/>Java + Spring Boot<br/>Recipe CRUD]
    end
    
    subgraph "AI/ML Services"
        Gemini[Google Gemini<br/>Recipe Generation]
        Imagen[Google Imagen<br/>Image Generation]
    end
    
    subgraph "Data Layer"
        Firestore[(Firestore<br/>Recipe Data)]
        CloudStorage[Cloud Storage<br/>Recipe Images]
    end
    
    subgraph "Shared Components"
        SharedLib[Shared Library<br/>TypeScript + Java<br/>Common Models]
    end
    
    WebUI --> CloudCDN
    WebUI --> Firebase
    CloudCDN --> WebUI
    
    WebUI --> AIService
    WebUI --> StorageService
    
    AIService --> Gemini
    AIService --> Imagen
    AIService --> SharedLib
    
    StorageService --> Firestore
    StorageService --> CloudStorage
    StorageService --> SharedLib
    
    Firebase -.->|Verify JWT| StorageService
    Firebase -.->|Verify JWT| AIService
    
    style WebUI fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
    style AIService fill:#E8F5E9,stroke:#388E3C,stroke-width:2px
    style StorageService fill:#E8F5E9,stroke:#388E3C,stroke-width:2px
    style Firestore fill:#FFF3E0,stroke:#F57C00,stroke-width:2px
    style Firebase fill:#FFF3E0,stroke:#F57C00,stroke-width:2px
    style Gemini fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
    style Imagen fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
```

## Service Details

### Frontend Application

**Repository**: [recipe-management-frontend](https://github.com/theandiman/recipe-management-frontend)

**Tech Stack**:
- React 19 with TypeScript
- Redux Toolkit for state management
- Tailwind CSS for styling
- Firebase SDK for authentication
- Axios for HTTP requests
- Vite for development and builds

**Key Features**:
- User authentication and profile management
- Recipe browsing and search
- AI-powered recipe generation interface
- Recipe creation and editing
- Public recipe sharing
- Cooking mode with step-by-step instructions
- Responsive design for mobile and desktop

**Deployment**:
- Hosted on Firebase Hosting
- CI/CD via GitHub Actions
- Automatic deployment on merge to main

### AI Service

**Repository**: [recipe-management-ai-service](https://github.com/theandiman/recipe-management-ai-service)

**Tech Stack**:
- Java 21
- Spring Boot 6.2
- Google Vertex AI SDK
- OpenAPI/Swagger documentation
- Maven for dependency management

**API Endpoints**:
- `POST /api/recipes/generate` - Generate recipe from prompt
- `POST /api/recipes/image/generate` - Generate recipe image

**Key Features**:
- Integration with Google Gemini for recipe generation
- Integration with Google Imagen for image generation
- Structured JSON schema for consistent recipe output
- Support for dietary preferences and allergies
- Unit and max time constraints
- Pantry item suggestions

**Deployment**:
- Google Cloud Run (containerized)
- Auto-scaling based on demand
- Cloud Build for CI/CD
- Artifact Registry for container storage

**API Documentation**: https://theandiman.github.io/recipe-management-ai-service/

### Storage Service

**Repository**: [recipe-management-storage-service](https://github.com/theandiman/recipe-management-storage-service)

**Tech Stack**:
- Java 21
- Spring Boot 6.2
- Firebase Admin SDK
- Firestore for data persistence
- OpenAPI/Swagger documentation

**API Endpoints**:
- `GET /api/recipes` - List recipes (with filters)
- `GET /api/recipes/{id}` - Get recipe by ID
- `POST /api/recipes` - Create new recipe
- `PUT /api/recipes/{id}` - Update recipe
- `DELETE /api/recipes/{id}` - Delete recipe
- `PATCH /api/recipes/{id}/sharing` - Update sharing status

**Key Features**:
- Full CRUD operations for recipes
- User ownership and authorization
- Public/private recipe visibility
- Firebase Authentication integration
- Firestore for scalable storage
- Cloud Storage for recipe images

**Deployment**:
- Google Cloud Run (containerized)
- Auto-scaling based on demand
- Cloud Build for CI/CD
- Artifact Registry for container storage

**API Documentation**: https://theandiman.github.io/recipe-management-storage-service/

### Shared Library

**Repository**: [recipe-management-shared](https://github.com/theandiman/recipe-management-shared)

**Purpose**: Provide consistent data models across TypeScript and Java services

**Packages**:
- `@theandiman/recipe-management-shared` (npm - TypeScript)
- `com.recipe:recipe-management-shared` (Maven - Java)

**Key Components**:
- `Recipe` interface/class
- `NutritionalInfo` models
- `RecipeTips` models
- `RecipeSchema` for Gemini AI
- Utility functions for data conversion

**Distribution**:
- Published to GitHub Packages (npm & Maven)
- Versioned releases with automated publishing
- Dual TypeScript/Java builds

### Infrastructure

**Repository**: [recipe-management-infrastructure](https://github.com/theandiman/recipe-management-infrastructure)

**Tech Stack**:
- Terraform for GCP resources
- Pulumi for Firebase configuration
- Cloud Build for CI/CD

**Managed Resources**:
- Cloud Run services and configurations
- Firestore database setup
- Cloud Storage buckets
- IAM roles and service accounts
- VPC and networking
- Firebase projects and apps
- Environment variables and secrets

## Data Flow

### Recipe Generation Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Firebase
    participant AIService
    participant Gemini
    participant Imagen
    participant Storage
    
    User->>Frontend: Enter recipe prompt
    Frontend->>Firebase: Authenticate
    Firebase-->>Frontend: JWT Token
    
    Frontend->>AIService: POST /api/recipes/generate<br/>(with JWT)
    AIService->>Firebase: Verify token
    Firebase-->>AIService: User verified
    
    AIService->>Gemini: Generate recipe with schema
    Gemini-->>AIService: Structured recipe JSON
    
    AIService->>Imagen: Generate recipe image
    Imagen-->>AIService: Image data
    
    AIService-->>Frontend: Complete recipe with image
    
    Frontend->>Storage: POST /api/recipes<br/>(save recipe)
    Storage->>Firestore: Store recipe
    Firestore-->>Storage: Recipe ID
    Storage-->>Frontend: Saved recipe
    
    Frontend->>User: Display recipe
```

### Recipe Retrieval Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Storage
    participant Firestore
    
    User->>Frontend: Browse recipes
    Frontend->>Storage: GET /api/recipes?public=true
    Storage->>Firestore: Query public recipes
    Firestore-->>Storage: Recipe list
    Storage-->>Frontend: Recipe list
    Frontend->>User: Display recipes
    
    User->>Frontend: Click recipe
    Frontend->>Storage: GET /api/recipes/{id}
    Storage->>Firestore: Get recipe by ID
    Firestore-->>Storage: Recipe data
    Storage-->>Frontend: Recipe details
    Frontend->>User: Display full recipe
```

### Recipe Sharing Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Firebase
    participant Storage
    participant Firestore
    
    User->>Frontend: Toggle share button
    Frontend->>Firebase: Verify user
    Firebase-->>Frontend: JWT Token
    
    Frontend->>Storage: PATCH /api/recipes/{id}/sharing<br/>{isPublic: true}
    Storage->>Firebase: Verify token & ownership
    Firebase-->>Storage: Authorized
    
    Storage->>Firestore: Update isPublic field
    Firestore-->>Storage: Updated
    Storage-->>Frontend: Success
    
    Frontend->>User: Show public status
```

## Security Architecture

### Authentication & Authorization

```mermaid
graph LR
    User[User] -->|Login| Firebase[Firebase Auth]
    Firebase -->|JWT Token| Frontend[Frontend App]
    Frontend -->|Bearer Token| Backend[Backend Services]
    Backend -->|Verify Token| Firebase
    Firebase -->|User Info| Backend
    Backend -->|Check Ownership| Firestore[(Firestore)]
```

**Security Layers**:
1. **Firebase Authentication**: User identity management
2. **JWT Tokens**: Stateless authentication for API calls
3. **Backend Verification**: Each service verifies tokens with Firebase
4. **Ownership Checks**: Services verify user owns resources before mutations
5. **CORS Policies**: Restrict API access to authorized origins
6. **Secret Management**: Secrets stored in Cloud Secret Manager

### Security Features

- **Secret Scanning**: Gitleaks pre-commit hooks prevent credential leaks
- **Dependency Scanning**: Renovate monitors for security vulnerabilities
- **Code Quality**: SonarCloud analyzes code for security issues
- **IAM Policies**: Least-privilege access for service accounts
- **Network Security**: Cloud Run with VPC connector for private services
- **Data Encryption**: Firestore encryption at rest and in transit

### Environment Strategy

- **Development**: `recipe-mgmt-dev` (Firebase project)
- **Production**: Future production environment

**Environment Separation**:
- Separate Firebase projects per environment
- Dedicated Cloud Run services
- Environment-specific configuration via Cloud Build substitutions
- Isolated Firestore databases

## Scalability & Performance

### Auto-Scaling

**Cloud Run Services**:
- Automatic scaling from 0 to N instances
- Configurable concurrency (80-100 requests per instance)
- CPU and memory allocation per service
- Startup and request timeout configurations


## Monitoring & Observability

### Logging

- **Structured Logging**: JSON logs with correlation IDs
- **Cloud Logging**: Centralized log aggregation
- **Log Levels**: INFO, WARN, ERROR with appropriate filtering

### Metrics

- **Cloud Monitoring**: Request rates, latencies, errors
- **Custom Metrics**: Recipe generation success rates, image generation times
- **Alerting**: Configured for service degradation and errors

### Tracing

- **Cloud Trace**: Distributed request tracing across services
- **Performance Insights**: Identify bottlenecks in AI generation



