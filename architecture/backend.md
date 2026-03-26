# Backend Architecture

## Tech Stack

| Technology | Version / Detail |
|---|---|
| Python | 3.13 |
| AWS SAM | CLI for local dev + deployment |
| Runtime | AWS Lambda (Python 3.13) |
| API | API Gateway HTTP API (v2) |
| Database | DynamoDB (single-table design) |
| Auth | JWT (PyJWT) + bcrypt |
| IaC | CloudFormation via SAM `template.yaml` |
| Testing | pytest + moto (DynamoDB mocking) |

## Project Structure

```
pedidos-backend/
├── template.yaml              # SAM: API Gateway + Lambdas + DynamoDB + Layer
├── samconfig.toml             # SAM CLI deployment config (multi-env)
├── docker-compose.yml         # DynamoDB Local for dev
├── requirements-dev.txt       # Local dev / test dependencies
├── layers/
│   └── common/                # Lambda Layer — shared code
│       ├── requirements.txt   # Layer runtime deps (PyJWT, bcrypt)
│       └── common/
│           ├── db.py          # DynamoDB helpers
│           ├── auth.py        # JWT + password + decorators
│           ├── responses.py   # HTTP JSON responses
│           ├── exceptions.py  # Custom exceptions
│           └── geo.py         # Haversine + delivery zones
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
    ├── conftest.py            # Shared fixtures
    └── test_*.py              # Tests by domain (125 tests)
```

## Code Conventions

- **One Lambda per domain**: each handler routes internally by HTTP method + path
- **Common layer**: `from common.db import ...` — shared across all functions
- **Handler pattern**: `lambda_handler(event, context)` with router dict
- **Naming**: `snake_case` (functions/vars), `UPPER_SNAKE_CASE` (constants), `PascalCase` (DynamoDB key prefixes)
- **API responses**: `{ "data": ..., "message": ... }` or `{ "error": "message" }`
- **Error messages**: Spanish (Argentina) — shown directly to end users
- **Timestamps**: ISO 8601 UTC

## DynamoDB Single-Table Design

Table: `PedidosTable-{env}` with PK/SK + GSI1 (GSI1PK/GSI1SK).

### Access Patterns

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
| `JWT_EXPIRATION_HOURS` | Token expiry | `24` |
| `DYNAMODB_ENDPOINT` | DynamoDB endpoint (local) | `http://localhost:8100` |
| `CORS_ORIGIN` | Allowed CORS origin | `http://localhost:5173` |
