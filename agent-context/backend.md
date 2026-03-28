# CLAUDE.md — AI Agent Context

Instructions and context for any agent working on this project.

## Project

**Ainara Helados — Backend API** for a delivery/pickup ordering system. Python backend using AWS SAM (Serverless Application Model) with Lambda functions and DynamoDB. Serves the React frontend at `pedidos-front/`.

## Tech Stack

| Technology | Version / Detail |
|---|---|
| Python | 3.13 (pinned in `.python-version`) |
| AWS SAM | CLI for local dev + deployment |
| Runtime | AWS Lambda (Python 3.13) |
| API | API Gateway HTTP API (v2) |
| Database | DynamoDB (single-table design) |
| Auth | JWT (PyJWT) + bcrypt for passwords |
| IaC | CloudFormation via SAM `template.yaml` |
| Testing | pytest + moto (DynamoDB mocking) |
| Developer OS | Windows (PowerShell in VS Code) |

## Project Structure

```
pedidos-backend/
├── template.yaml              # SAM: API Gateway + Lambdas + DynamoDB + Layer
├── samconfig.toml             # SAM CLI deployment config
├── docker-compose.yml         # DynamoDB Local for dev
├── requirements-dev.txt       # Local dev / test dependencies
├── .python-version            # Python version pinning
├── layers/
│   └── common/                # Lambda Layer — shared code
│       ├── requirements.txt   # Layer runtime dependencies (PyJWT, bcrypt)
│       └── common/
│           ├── __init__.py
│           ├── db.py          # DynamoDB helpers (get, put, query, delete)
│           ├── auth.py        # JWT create/verify, password hash/check, decorators
│           ├── responses.py   # Standardized HTTP JSON responses
│           ├── exceptions.py  # Custom exceptions
│           ├── logging.py     # @log_handler decorator — structured request/response logging
│           └── geo.py         # Haversine formula, delivery zone logic
├── functions/                 # Lambda handlers (one per domain)
│   ├── auth/app.py
│   ├── catalog/app.py
│   ├── addresses/app.py
│   ├── orders/app.py
│   ├── payments/app.py
│   ├── loyalty/app.py
│   ├── coupons/app.py
│   ├── delivery/app.py
│   ├── admin_orders/app.py
│   ├── admin_products/app.py
│   ├── admin_flavors/app.py
│   └── admin_users/app.py
├── seeds/
│   └── seed.py                # Populate DynamoDB with initial data
└── tests/
    ├── conftest.py            # Shared fixtures (mock DynamoDB, token helpers)
    └── test_*.py              # Test files by domain
```

## Code Conventions

### Files and structure
- **One Lambda per domain**: each handler routes internally by HTTP method + path
- **Common layer**: shared code in `layers/common/common/`, importable as `from common.db import ...`
- **Handler pattern**: each `app.py` exports `lambda_handler(event, context)` with a router dict

### Naming
- **Functions/variables**: `snake_case`
- **Constants**: `UPPER_SNAKE_CASE`
- **DynamoDB keys**: `PascalCase` for PK/SK prefixes (`USER#`, `ORDER#`, `PRODUCT#`)

### DynamoDB Single-Table Design
- **Table**: `PedidosTable`
- **Keys**: `PK` (partition), `SK` (sort) — both strings
- **GSI1**: `GSI1PK`, `GSI1SK` — for cross-entity queries (user orders, product by category, etc.)
- All entities share the same table with prefixed keys

### API
- **Base path**: `/api/`
- **Responses**: always JSON with `{ "data": ..., "message": ... }` structure
- **Errors**: `{ "error": "message" }` with appropriate HTTP status
- **Error messages**: in **Spanish (Argentina)** — these are shown directly to the user

### Style
- No classes for handlers — plain functions
- Minimal dependencies — stdlib where possible, boto3 is available in Lambda runtime
- Input validation at handler level before DB operations
- All timestamps in ISO 8601 UTC

### Git and versioning
- **Commit messages**: English, conventional commits format (`feat:`, `fix:`, `refactor:`, `docs:`)
- **Branch**: `master`, direct push to `origin/master`
- **Changelog**: in `CHANGELOG.md`, ordered newest to oldest
- **Versions**: semver — major features = minor bump (v0.X.0), fixes = patch (v0.X.Y)
- **Workflow**: change → update CHANGELOG → `git add -A` → `git commit` → `git push origin master`

### Conversation
- The user speaks **Spanish**
- Commit messages in **English**
- Code (variables, technical comments) in **English**
- API error messages in **Spanish** (Argentina) — shown to end users

## DynamoDB Access Patterns

| # | Pattern | PK | SK | Index |
|---|---------|----|----|-------|
| 1 | User by ID | `USER#{id}` | `PROFILE` | Table |
| 2 | User by email | `EMAIL#{email}` | `USER` | GSI1 |
| 3 | User addresses | `USER#{id}` | `ADDR#{addrId}` | Table |
| 4 | All products | `PRODUCTS` | `PROD#{id}` | Table |
| 5 | Products by category | `CAT#{catId}` | `PROD#{id}` | GSI1 |
| 6 | Single product | `PRODUCT#{id}` | `PRODUCT` | Table |
| 7 | All categories | `CATEGORIES` | `CAT#{id}` | Table |
| 8 | Flavor source meta | `FLVSRC#{srcId}` | `META` | Table |
| 9 | Flavors in source | `FLVSRC#{srcId}` | `FLV#{flvId}` | Table |
| 10 | All flavor sources | `FLAVOR_SOURCES` | `FLVSRC#{id}` | Table |
| 11 | Order by ID | `ORDER#{id}` | `ORDER` | Table |
| 12 | Orders by user | `USER#{userId}` | `ORD#{createdAt}` | Table |
| 13 | All orders (admin) | `ORDERS` | `{createdAt}#{id}` | GSI1 |
| 14 | Points by user | `USER#{userId}` | `PTS#{ptsId}` | Table |
| 15 | Coupon by code | `COUPON_CODE#{code}` | `COUPON` | Table |
| 16 | Payment methods | `CONFIG` | `PAY#{methodId}` | Table |
| 17 | Delivery config | `CONFIG` | `DELIVERY` | Table |
| 18 | Store config | `CONFIG` | `STORE` | Table |

## Environment Variables

| Variable | Description | Default (local) |
|----------|-------------|-----------------|
| `TABLE_NAME` | DynamoDB table name | `PedidosTable` |
| `JWT_SECRET` | Secret for signing JWTs | `dev-secret-change-in-prod` |
| `JWT_EXPIRATION_HOURS` | Token expiry in hours | `24` |
| `DYNAMODB_ENDPOINT` | DynamoDB endpoint (local dev) | `http://localhost:8100` |
| `CORS_ORIGIN` | Allowed CORS origin | `http://localhost:5173` |

## Useful Commands

```bash
# Local development
sam build                           # Build all functions + layer
sam local start-api --warm-containers eager   # Start local API (hot reload)

# DynamoDB Local
docker compose up -d                # Start DynamoDB Local on port 8100
python seeds/seed.py                # Populate with test data

# Testing
python -m pytest tests/ -v          # Run all tests

# Deploy
sam deploy --guided                 # First deploy (interactive)
sam deploy                          # Subsequent deploys

# Logs
sam logs -n AuthFunction --tail     # Tail logs for a function
```

## Important Notes

1. `boto3` is included in the Lambda runtime — no need to add it to layer requirements
2. The common layer is built by SAM and mounted at `/opt/python/` in Lambda
3. For local dev, `sam local start-api` uses Docker to emulate Lambda
4. DynamoDB Local runs on port 8100 (not 8000) to avoid conflict with the API
5. The frontend expects the API at `http://localhost:8000/api` — SAM local runs on port 3000, so a proxy or env var change is needed
6. All handler functions must be named `lambda_handler` to match the SAM template

## Infrastructure Change Protocol

**Any change to deployed infrastructure MUST follow these steps:**

1. **Make the change** in `template.yaml` or `samconfig.toml`
2. **Validate**: `sam validate --lint`
3. **Deploy**: `sam build && sam deploy`
4. **Document**: update `pedidos-docs/infrastructure/` (aws-resources.md, deploy.md, or environments.md as needed)
5. **Commit all repos affected** (backend, docs, frontend) with a clear message: `infra: describe what changed`
6. **Push**: `git push origin master` for every repo that changed

Never leave deployed infrastructure undocumented. If a new resource, output, parameter, or environment value is created, it must be reflected in `pedidos-docs/infrastructure/` before the work is considered done.
