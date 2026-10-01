# API Schema Registry

## 1. Executive Summary
This repository defines the architecture and governance for a centralized API Schema Registry managing OpenAPI Specification (OAS) 3.0.1 files across multiple microservices.

To eliminate maintenance overhead and file duplication, the registry utilizes a flat directory-per-service strategy. Version history and immutability are managed natively via Git tagging and automated CI/CD pipeline validation rather than physical version folders.

---

## 2. Repository Structure
The schema registry is hosted in a single, centralized Git repository. Each microservice is granted an isolated directory to ensure clear ownership, path-based CI/CD triggers, and room for local extension.

### Directory Layout
```text
├── .github/              # CI/CD workflows and validation hooks
├── common/               # Shared domain schemas and error payloads
│   ├── errors.yaml
│   └── pagination.yaml
├── services/             # Microservice isolated directories
│   ├── identity-service/
│   │   └── openapi.yaml  # Single source of truth for Identity
│   ├── order-service/
│   │   └── openapi.yaml  # Single source of truth for Orders
│   └── payment-service/
│       └── openapi.yaml  # Single source of truth for Payments
└── README.md
```

### Design Rules
* **Single Source of Truth:** Exactly one `openapi.yaml` file exists per microservice on the `main` branch.
* **No File Duplication:** Storing duplicate files for major versions or tracking aliases is strictly prohibited.
* **Component Reusability:** Common domain models must reside in the `/common` root directory.
* **Relative Referencing:** Reference shared objects using standard OpenAPI relative file pointers:

```yaml
$ref: '../../common/errors.yaml#/components/schemas/NotFoundError'
```

---

## 3. Versioning Strategy
API versions are decoupled from the physical file paths and managed entirely via Git metadata and Semantic Versioning (SemVer 1.0.0).

### Git Tagging Schema
Every production-ready contract deployment must be marked with a unique Git tag scoped to that specific microservice:

```text
<service-name>/v<major>.<minor>.<patch>
```

* *Example:* `identity-service/v1.2.0`
* *Example:* `order-service/v2.0.1`

### API Lifecycle & Evolution
To avoid maintaining multiple active major versions of a service simultaneously, developers must prioritize the Robustness Principle (Postel's Law) to evolve schemas seamlessly:

* **Additions Only:** New endpoints, optional request properties, and response properties can be added freely.
* **No Breaking Mutations:** Renaming fields, deleting fields, or changing fields from optional to required requires a major version bump and coordinated team deprecation notices.
* End