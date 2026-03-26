# Environments — dev / staging / prod

## Strategy

Three isolated CloudFormation stacks from the same `template.yaml`, differentiated by the `Environment` parameter:

```
pedidos-backend-dev      → Environment=dev
pedidos-backend-staging  → Environment=staging
pedidos-backend-prod     → Environment=prod
```

Each stack creates its own API Gateway, DynamoDB table, Lambda functions, and CloudWatch alarm.

## Environment Comparison

| Aspect | dev | staging | prod |
|--------|-----|---------|------|
| **Stack name** | pedidos-backend-dev | pedidos-backend-staging | pedidos-backend-prod |
| **DynamoDB table** | PedidosTable-dev | PedidosTable-staging | PedidosTable-prod |
| **Deploy method** | `sam sync --watch` | `sam deploy --config-env staging` | `sam deploy --config-env prod` |
| **Confirm changeset** | No | Yes | Yes |
| **Canary deployment** | Disabled | Disabled | Enabled (10% → 5min → 100%) |
| **Auto-rollback** | No | No | Yes (on Lambda errors) |
| **Purpose** | Rapid iteration | Pre-release validation | Live traffic |

## samconfig.toml

```toml
[default.deploy.parameters]
stack_name = "pedidos-backend-dev"
resolve_s3 = true
s3_prefix = "pedidos-backend"
region = "us-east-1"
confirm_changeset = false
capabilities = "CAPABILITY_IAM"
parameter_overrides = "Environment=\"dev\""

[default.sync.parameters]
stack_name = "pedidos-backend-dev"
watch = true
region = "us-east-1"

[staging.deploy.parameters]
stack_name = "pedidos-backend-staging"
resolve_s3 = true
s3_prefix = "pedidos-backend-staging"
region = "us-east-1"
confirm_changeset = true
capabilities = "CAPABILITY_IAM"
parameter_overrides = "Environment=\"staging\""

[prod.deploy.parameters]
stack_name = "pedidos-backend-prod"
resolve_s3 = true
s3_prefix = "pedidos-backend-prod"
region = "us-east-1"
confirm_changeset = true
capabilities = "CAPABILITY_IAM"
parameter_overrides = "Environment=\"prod\""
```

## Staging as a Prod Copy

Staging uses the same template and infrastructure as prod, creating a separate `PedidosTable-staging`. To populate it with production-like data:

1. **From seed**: `python seeds/seed.py` (targeting staging endpoint)
2. **From prod** (future): a script that copies data from `PedidosTable-prod` → `PedidosTable-staging`

## Workflow

```
┌─────────┐   sam sync     ┌─────────┐
│  Local   │ ──--watch──→   │   DEV   │   ~5s deploys
└─────────┘                 └─────────┘
                                 │
                            sam deploy
                            --config-env staging
                                 │
                            ┌─────────┐
                            │ STAGING │   validate release
                            └─────────┘
                                 │
                            sam deploy
                            --config-env prod
                                 │
                            ┌─────────┐
                            │  PROD   │   canary + auto-rollback
                            └─────────┘
```
