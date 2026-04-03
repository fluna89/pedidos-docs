# AWS Resources

Two CloudFormation stacks per environment:
- **pedidos-backend-{env}**: API Gateway + Lambda + DynamoDB (defined in `pedidos-backend/template.yaml`)
- **pedidos-frontend-{env}**: S3 + CloudFront (defined in `pedidos-front/template.yaml`)

## Backend Stack

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

### Observability

| Resource | Type | Details |
|----------|------|---------|
| ApiErrorAlarm | `AWS::CloudWatch::Alarm` | Monitors Lambda errors for auto-rollback |
| Structured logging | `common.logging` | `@log_handler` decorator on all Lambdas |

- CloudWatch alarm: `AWS/Lambda` → `Errors` ≥ 5 in 1 min
- Lambda logs: method, path, status code, duration (ms) per request
- Log groups: `/aws/lambda/pedidos-backend-{env}-{FunctionName}-{hash}`

### Backend Outputs

| Output | Value |
|--------|-------|
| ApiUrl | `https://{api-id}.execute-api.us-east-1.amazonaws.com/{env}/api/` |
| TableName | `PedidosTable-{env}` |

## Frontend Stack

Defined in `pedidos-front/template.yaml`. Separate CloudFormation stack.

| Resource | Type | Details |
|----------|------|---------|
| FrontendBucket | `AWS::S3::Bucket` | Static assets for React SPA |
| FrontendBucketPolicy | `AWS::S3::BucketPolicy` | Allows CloudFront OAC access only |
| CloudFrontOAC | `AWS::CloudFront::OriginAccessControl` | Secure S3 origin access (sigv4) |
| CloudFrontDistribution | `AWS::CloudFront::Distribution` | CDN for frontend with HTTPS |

- Bucket name: `pedidos-frontend-{env}-{accountId}`
- Public access: **blocked** — only CloudFront can read via OAC
- CloudFront: redirect HTTP→HTTPS, HTTP/2+3, gzip compression
- Cache policy: `CachingOptimized` (managed policy)
- SPA routing: 403/404 errors → `/index.html` (client-side routing)
- Price class: `PriceClass_100` (North America + Europe — cheapest)

### Frontend Outputs

| Output | Value |
|--------|-------|
| FrontendBucketName | `pedidos-frontend-{env}-{accountId}` |
| CloudFrontUrl | `https://{distribution-id}.cloudfront.net` |
| CloudFrontDistributionId | Distribution ID (for cache invalidation) |

## IAM Users

| User | Access | Policy | Purpose |
|------|--------|--------|---------|
| `deploy-user` | CLI (access keys) | AdministratorAccess | SAM/CloudFormation deploys |
| `pedidos-dev-admin` | Console (password) | AdministratorAccess | AWS web console management |

> **TODO (security):** Apply least privilege to `deploy-user`. Replace `AdministratorAccess` with a scoped policy allowing only: CloudFormation, Lambda, API Gateway, DynamoDB, S3, CloudFront, IAM (pass-role), CloudWatch, SSM, and SAM-related actions. Track in roadmap.

Console login URL: `https://561047280243.signin.aws.amazon.com/console`

## Cost Estimates (dev, low traffic)

| Service | Cost (approx.) |
|---------|----------------|
| API Gateway | ~$0 (1M requests free tier) |
| Lambda | ~$0 (1M requests + 400K GB-s free tier) |
| DynamoDB | ~$0 (25 GB + 25 WCU/RCU free tier, on-demand) |
| CloudWatch | ~$0 (basic metrics free) |
| SSM | $0 (standard parameters free) |
| S3 (deploy artifacts) | ~$0.01/month |
| S3 (frontend bucket) | ~$0.01/month |
| CloudFront | ~$0 (1 TB free tier first 12 months, then ~$0.085/GB) |

Total for dev/staging: effectively **$0/month** under AWS Free Tier.

## Frontend Deploy

```powershell
# Build frontend with production API URL
cd pedidos-front
npm run build

# Upload to S3
aws s3 sync dist s3://pedidos-frontend-{env}-{accountId} --delete --profile pedidos-dev

# Invalidate CloudFront cache
aws cloudfront create-invalidation --distribution-id {DISTRIBUTION_ID} --paths "/*" --profile pedidos-dev
```

The frontend reads `VITE_API_URL` from `.env.production` at build time.

## Viewing Logs

```powershell
# List all Lambda log groups
aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/pedidos-backend-dev" --profile pedidos-dev --region us-east-1 --query "logGroups[].logGroupName" --output table

# Tail logs for a specific function in real time
aws logs tail "/aws/lambda/pedidos-backend-dev-AuthFunction-{hash}" --follow --since 1h --profile pedidos-dev --region us-east-1 --format short
```

Log output format (from `@log_handler` decorator):
```
=> POST /dev/api/auth/login
<= POST /dev/api/auth/login -> 200 (45ms)
!! GET /dev/api/menu -> ERROR (8ms): something went wrong
```
