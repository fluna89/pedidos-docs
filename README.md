# Ainara Helados — Documentation

Central documentation for the Ainara Helados ordering system.

## Repositories

| Repo | Description |
|------|-------------|
| [pedidos-front](https://github.com/fluna89/pedidos-front) | React 19 + Vite 7 frontend (customer app + admin panel) |
| [pedidos-backend](https://github.com/fluna89/pedidos-backend) | Python 3.13 + AWS SAM backend (Lambda + DynamoDB) |
| pedidos-docs (this repo) | Architecture, infrastructure, setup guides, and changelogs |

## Documentation Index

### Architecture
- [Overview](architecture/overview.md) — System architecture, components, and data flow
- [Backend](architecture/backend.md) — DynamoDB single-table design, access patterns, Lambda structure
- [Frontend](architecture/frontend.md) — React provider architecture, conventions, project structure

### Infrastructure
- [Environments](infrastructure/environments.md) — dev / staging / prod strategy
- [Deploy](infrastructure/deploy.md) — SAM deploy, sam sync, rollback procedures
- [AWS Resources](infrastructure/aws-resources.md) — What the CloudFormation stack creates

### Setup
- [Backend Setup](setup/backend.md) — Python, SAM CLI, DynamoDB Local, testing
- [Frontend Setup](setup/frontend.md) — Node.js (fnm), npm, Vite dev server

### Requirements
- [Functional Requirements](requirements/functional.md) — User-facing feature specifications
- [Admin Roadmap](requirements/roadmap.md) — Admin panel phases and planned features

### Changelogs
- [Frontend Changelog](changelog/frontend.md) — Version history for pedidos-front
- [Backend Changelog](changelog/backend.md) — Version history for pedidos-backend
