# Backend Setup

## Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| [Python](https://www.python.org/downloads/) | 3.13 | Lambda runtime |
| [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html) | Latest | Build + deploy + local dev |
| [Docker](https://www.docker.com/products/docker-desktop/) | Latest | DynamoDB Local + `sam local` |
| [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) | v2 | Deploy + cloud operations |

## Local Development

```bash
# 1. Create virtual environment
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux/Mac

# 2. Install dev dependencies
pip install -r requirements-dev.txt

# 3. Start DynamoDB Local (Docker must be running)
docker compose up -d

# 4. Populate with test data
python seeds/seed.py

# 5. Build the SAM project
sam build

# 6. Start the local API
sam local start-api --port 8000 --host 0.0.0.0 --env-vars env.json --warm-containers eager
```

The API will be available at `http://localhost:8000/api/`.

> **Note**: `env.json` contains local environment variables (DynamoDB endpoint, JWT secret, etc.) and is excluded from the repo via `.gitignore`.

## Restarting SAM Local

Required after changing `template.yaml`, `env.json`, layer dependencies (`requirements.txt`), or any handler/layer code.

```bash
# 1. Kill the running SAM process (Ctrl+C in its terminal)

# 2. Rebuild (recompiles functions + layer with dependencies)
sam build

# 3. Start the local API
sam local start-api -t .aws-sam\build\template.yaml -n env.json --warm-containers EAGER --port 3000
```

### When is a rebuild + restart needed?

| Change | Rebuild + restart required |
|--------|:---:|
| Handler code (`app.py`) | Yes |
| `template.yaml` (routes, env vars, policies) | Yes |
| `env.json` (local env vars) | Yes |
| Layer `requirements.txt` (new dependency) | Yes |
| Layer code (`common/*.py`) | Yes |

> `--warm-containers EAGER` pre-loads containers for faster responses but does **not** hot-reload code. Any code change requires a full rebuild + restart.

## Test Data

| Credential | Value |
|------------|-------|
| Customer | `juan@test.com` / `1234` |
| Admin | `admin@ainara.com` / `admin` |
| Coupons | `HELADOGRATIS`, `VERANO20`, `AINARA10`, `EXPIRADO` |

## Running Tests

```bash
python -m pytest tests/ -v
```

125 tests covering all 12 handlers with moto-mocked DynamoDB.

## Deploy

See [infrastructure/deploy.md](../infrastructure/deploy.md) for full deploy and rollback procedures.

```bash
# Quick reference
sam build && sam deploy                          # → dev
sam build && sam deploy --config-env staging      # → staging
sam build && sam deploy --config-env prod         # → prod
sam sync --watch                                  # → dev (fast iteration)
```

## Important Notes

- `boto3` is included in the Lambda runtime — don't add it to layer requirements
- The common layer is built by SAM and mounted at `/opt/python/` in Lambda
- `sam local start-api` uses Docker to emulate Lambda containers
- DynamoDB Local runs on port **8100** (not 8000) to avoid conflict with the API
- All handler functions must be named `lambda_handler` to match the SAM template
