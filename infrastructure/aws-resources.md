# AWS Resources

Resources created by the `template.yaml` CloudFormation stack for each environment.

## Resource Inventory

### API Gateway

| Resource | Type | Details |
|----------|------|---------|
| PedidosApi | `AWS::Serverless::HttpApi` | HTTP API v2 with CORS, stage = `{Environment}` |

- Routes: 40+ endpoints across 12 Lambda functions
- CORS: all origins (`*`), methods GET/POST/PUT/DELETE/OPTIONS

### DynamoDB

| Resource | Type | Details |
|----------|------|---------|
| PedidosTable | `AWS::DynamoDB::Table` | Single-table design, PAY_PER_REQUEST |

- Table name: `PedidosTable-{env}`
- Keys: `PK` (String), `SK` (String)
- GSI1: `GSI1PK`, `GSI1SK` — for cross-entity queries
- Billing: on-demand (scales to zero, no capacity planning)

### Lambda Functions (12)

All functions share:
- Runtime: Python 3.13
- Memory: 256 MB
- Timeout: 15 seconds
- Layer: CommonLayer
- `AutoPublishAlias: live` (immutable versioning)

| Function | CodeUri | Policy | Routes |
|----------|---------|--------|--------|
| AuthFunction | functions/auth/ | DynamoDBCrud | 6 (login, register, guest, google, recover, profile) |
| CatalogFunction | functions/catalog/ | DynamoDBRead | 6 (menu, item, by-ids, categories, flavors, counter-menu) |
| AddressesFunction | functions/addresses/ | DynamoDBCrud | 4 (list, create, update, delete) |
| OrdersFunction | functions/orders/ | DynamoDBCrud | 3 (create, list, active) |
| PaymentsFunction | functions/payments/ | DynamoDBCrud | 2 (methods, process) |
| LoyaltyFunction | functions/loyalty/ | DynamoDBCrud | 4 (balance, history, redeem, earn) |
| CouponsFunction | functions/coupons/ | DynamoDBRead | 1 (validate) |
| DeliveryFunction | functions/delivery/ | DynamoDBRead | 1 (calc) |
| AdminOrdersFunction | functions/admin_orders/ | DynamoDBCrud | 6 (list, create, advance, revert, cancel, set-status) |
| AdminProductsFunction | functions/admin_products/ | DynamoDBCrud | 7 (list, create, update, delete, toggle, usage, base) |
| AdminFlavorsFunction | functions/admin_flavors/ | DynamoDBCrud | 8 (sources CRUD + flavors CRUD) |
| AdminUsersFunction | functions/admin_users/ | DynamoDBRead | 1 (search) |

### Lambda Layer

| Resource | Type | Details |
|----------|------|---------|
| CommonLayer | `AWS::Serverless::LayerVersion` | Shared Python code (db, auth, responses, geo) |

- Name: `pedidos-common-{env}`
- Built from `layers/common/`
- Runtime dependencies: PyJWT, bcrypt

### CloudWatch

| Resource | Type | Details |
|----------|------|---------|
| ApiErrorAlarm | `AWS::CloudWatch::Alarm` | Monitors Lambda errors for auto-rollback |

- Metric: `AWS/Lambda` → `Errors`
- Threshold: ≥5 errors in 1 minute
- Used by CodeDeploy for canary rollback (prod only)

### Deployment Preferences (prod only)

- **AutoPublishAlias**: `live` — each deploy creates an immutable version
- **Type**: `Canary10Percent5Minutes`
- 10% traffic shifted → 5 min soak → 100% or rollback

### SSM Parameter Store (external)

| Parameter | Type | Description |
|-----------|------|-------------|
| `/pedidos/jwt-secret` | SecureString | JWT signing secret (created manually) |

## Outputs

| Output | Value |
|--------|-------|
| ApiUrl | `https://{api-id}.execute-api.us-east-1.amazonaws.com/{env}/api/` |
| TableName | `PedidosTable-{env}` |

## Cost Estimates (dev, low traffic)

| Service | Cost (approx.) |
|---------|----------------|
| API Gateway | ~$0 (1M requests free tier) |
| Lambda | ~$0 (1M requests + 400K GB-s free tier) |
| DynamoDB | ~$0 (25 GB + 25 WCU/RCU free tier, on-demand) |
| CloudWatch | ~$0 (basic metrics free) |
| SSM | $0 (standard parameters free) |
| S3 (deploy artifacts) | ~$0.01/month |

Total for dev/staging: effectively **$0/month** under AWS Free Tier.
