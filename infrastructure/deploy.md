# Deploy & Rollback

## Prerequisites

```bash
# AWS CLI configured with credentials
aws configure
# → Access Key, Secret Key, Region: us-east-1

# JWT secret in SSM Parameter Store
aws ssm put-parameter --name "/pedidos/jwt-secret" --type SecureString --value "YOUR_SECRET"
```

## Deploy Commands

### Development — Fast Iteration

```bash
# One-time build + watch for changes (~5s per code change)
sam sync --stack-name pedidos-backend-dev --watch

# Or traditional deploy (1-3 min)
sam build && sam deploy
```

`sam sync --watch` monitors file changes and:
- **Code changes** (e.g., edit `app.py`): uploads directly to Lambda via `UpdateFunctionCode` API (~5s, bypasses CloudFormation)
- **Infrastructure changes** (e.g., add a route): triggers a full CloudFormation deploy (~1-3 min)

### Staging — Pre-release Validation

```bash
sam build
sam deploy --config-env staging
# → Reviews changeset → Confirm (y/n) → Deploy
```

### Production — Safe Deploy with Canary

```bash
sam build
sam deploy --config-env prod
# → Reviews changeset → Confirm (y/n) → Canary deployment
```

Production deploys use `Canary10Percent5Minutes`:
1. 10% of traffic goes to the new version
2. Waits 5 minutes
3. If no alarms trigger → shifts 100% to new version
4. If CloudWatch alarm fires (≥5 Lambda errors/min) → **automatic rollback**

## Rollback Procedures

### Automatic (prod only)
The `ApiErrorAlarm` CloudWatch alarm monitors Lambda errors across all functions. During a canary deployment, if ≥5 errors occur in 1 minute, CodeDeploy automatically rolls back to the previous Lambda version.

### Manual — CloudFormation

```bash
# Roll back the entire stack to the previous deployment
aws cloudformation rollback-stack --stack-name pedidos-backend-prod
```

### Manual — Lambda Alias (instant)

Each Lambda has a `live` alias pointing to an immutable version. To roll back a specific function:

```bash
# List recent versions
aws lambda list-versions-by-function --function-name pedidos-backend-prod-AuthFunction

# Point alias to previous version
aws lambda update-alias \
  --function-name pedidos-backend-prod-AuthFunction \
  --name live \
  --function-version PREVIOUS_VERSION_NUMBER
```

This is instant (no CloudFormation involved).

## First Deploy (New Environment)

```bash
# Build
sam build

# Interactive guided deploy (creates S3 bucket, sets parameters)
sam deploy --guided

# Or use samconfig.toml defaults
sam deploy                          # → dev
sam deploy --config-env staging     # → staging
sam deploy --config-env prod        # → prod
```

## sam sync vs sam deploy

| Aspect | `sam sync` | `sam deploy` |
|--------|-----------|-------------|
| Speed (code change) | ~5 seconds | 1-3 minutes |
| Speed (infra change) | 1-3 minutes | 1-3 minutes |
| Uses CloudFormation | Only for infra | Always |
| Changeset review | No | Yes (if `confirm_changeset = true`) |
| Best for | Development | Staging / Production |
| Watch mode | `--watch` flag | N/A |

## Useful Commands

```bash
# Check stack status
aws cloudformation describe-stacks --stack-name pedidos-backend-dev

# Tail Lambda logs
sam logs -n AuthFunction --stack-name pedidos-backend-dev --tail

# View stack outputs (API URL, table name)
sam list stack-outputs --stack-name pedidos-backend-dev

# Delete a stack entirely
sam delete --stack-name pedidos-backend-dev
```
