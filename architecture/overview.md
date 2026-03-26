# System Architecture — Overview

## High-Level Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENTS                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Customer App │  │ Admin Panel  │  │   (Future)   │      │
│  │   (Mobile)   │  │  (Desktop)   │  │  Mobile App  │      │
│  └──────┬───────┘  └──────┬───────┘  └──────────────┘      │
└─────────┼──────────────────┼────────────────────────────────┘
          │                  │
          ▼                  ▼
┌─────────────────────────────────────────────────────────────┐
│                    pedidos-front                             │
│  React 19 · Vite 7 · Tailwind v4 · shadcn/ui               │
│  SPA served from S3 + CloudFront (future)                   │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS (REST)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   AWS API Gateway                           │
│                   (HTTP API v2)                              │
│  Routes: /api/auth/* · /api/menu/* · /api/orders/* · ...    │
│  CORS · Stages: dev / staging / prod                        │
└────────────────────────┬────────────────────────────────────┘
                         │
          ┌──────────────┼──────────────────┐
          ▼              ▼                  ▼
┌──────────────┐ ┌──────────────┐  ┌──────────────┐
│ AuthFunction │ │CatalogFunc.  │  │ OrdersFunc.  │  ... (12 Lambdas)
│              │ │              │  │              │
│  Python 3.13 │ │  Python 3.13 │  │  Python 3.13 │
└──────┬───────┘ └──────┬───────┘  └──────┬───────┘
       │                │                 │
       └────────────────┼─────────────────┘
                        │
                        ▼
              ┌──────────────────┐
              │  Common Layer    │
              │  (Lambda Layer)  │
              │  db · auth ·     │
              │  responses · geo │
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────┐        ┌──────────────────┐
              │    DynamoDB      │        │  SSM Parameter   │
              │  (Single Table)  │        │  Store           │
              │  PedidosTable    │        │  /pedidos/*      │
              └──────────────────┘        └──────────────────┘
```

## Components

### Frontend (`pedidos-front`)
- **React 19** SPA with client-side routing (React Router v7)
- **Two interfaces**: customer-facing (mobile-first) and admin panel (desktop-first)
- Communicates with backend exclusively via REST API
- State management through React Context (Auth, Cart, Address, Loyalty, Theme)

### API Gateway
- **HTTP API v2** (cheaper and faster than REST API v1)
- Per-environment stages: `dev`, `staging`, `prod`
- CORS configured for frontend origins

### Lambda Functions (12)
One function per domain, each routing internally by HTTP method + path:

| Function | Domain | Access |
|----------|--------|--------|
| AuthFunction | Login, register, guest, Google, recovery, profile | CRUD |
| CatalogFunction | Menu, categories, flavors | Read-only |
| AddressesFunction | Address CRUD | CRUD |
| OrdersFunction | Create, list, active orders | CRUD |
| PaymentsFunction | Payment methods, process payment | CRUD |
| LoyaltyFunction | Points balance, history, redeem, earn | CRUD |
| CouponsFunction | Validate coupon codes | Read-only |
| DeliveryFunction | Calculate delivery cost | Read-only |
| AdminOrdersFunction | Order management (advance, cancel, etc.) | CRUD |
| AdminProductsFunction | Product CRUD, toggle, combos | CRUD |
| AdminFlavorsFunction | Flavor sources and items CRUD | CRUD |
| AdminUsersFunction | User search | Read-only |

### Common Layer
Shared Python code mounted at `/opt/python/` in all Lambdas:
- `db.py` — DynamoDB helpers (get, put, query, delete)
- `auth.py` — JWT creation/verification, password hashing, decorators
- `responses.py` — Standardized HTTP JSON responses
- `exceptions.py` — Custom exceptions
- `geo.py` — Haversine formula, delivery zone logic

### DynamoDB
- **Single-table design** with `PK`/`SK` keys + `GSI1` secondary index
- **PAY_PER_REQUEST** billing (scales automatically, $0 at rest)
- See [backend architecture](backend.md) for full access patterns

### SSM Parameter Store
- Stores secrets (`/pedidos/jwt-secret`) referenced by CloudFormation at deploy time

## Data Flow: Place an Order

```
Customer → POST /api/orders
  → API Gateway → OrdersFunction (Lambda)
    → auth.py: verify JWT token
    → db.py: write ORDER# to DynamoDB
    → db.py: write points transaction
    → responses.py: return { data: order }
  → API Gateway → Customer (200 OK)
```

## Authentication Flow

```
1. POST /api/auth/login  { email, password }
2. Lambda verifies bcrypt hash from DynamoDB
3. Returns JWT (24h expiry) signed with SSM secret
4. Frontend stores token in localStorage
5. All subsequent requests include Authorization: Bearer {token}
```
