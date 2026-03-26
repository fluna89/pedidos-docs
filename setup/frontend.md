# Frontend Setup (Windows)

## Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| **fnm** | 1.x | Node.js version manager |
| **Node.js** | 24.x LTS | JavaScript runtime |
| **npm** | 11.x | Package manager |
| **Git** | 2.x | Version control |

## 1. Install fnm (Fast Node Manager)

```powershell
winget install Schniz.fnm
```

## 2. Configure Shell

### Option A: CMD

Navigate to the project and run:

```cmd
cd C:\repos\pedidos\pedidos-front
init.cmd
```

Run this once per CMD session.

### Option B: PowerShell (recommended)

Add to your PowerShell profile (`notepad $PROFILE`):

```powershell
fnm env --use-on-cd --shell powershell | Out-String | Invoke-Expression
```

This auto-activates the correct Node.js version when you `cd` into a project with `.node-version`.

> If PowerShell blocks scripts, run once:
> ```powershell
> Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
> ```

## 3. Install Node.js

```bash
fnm install   # Downloads the version from .node-version
fnm use       # Activates it

# Verify
node --version  # Should match .node-version
npm --version
```

## 4. Install Dependencies

```bash
npm install
```

## 5. Available Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start dev server (Vite) at `localhost:5173` with hot reload |
| `npm run build` | Production build to `dist/` |
| `npm run preview` | Serve `dist/` locally to test build |
| `npm run lint` | Check code with ESLint |
| `npm run lint:fix` | Auto-fix ESLint issues |
| `npm run format` | Format with Prettier |
| `npm run format:check` | Check formatting without modifying |

## 6. Environment Variables

Create `.env` in the project root:

```env
VITE_API_URL=http://localhost:8000/api
VITE_STRICT_MODE=true
```

## 7. Backend Connection

The frontend expects the backend API at `VITE_API_URL` (default: `http://localhost:8000/api`).

To run the full stack locally:
1. Start DynamoDB Local: `docker compose up -d` (in pedidos-backend)
2. Seed data: `python seeds/seed.py` (in pedidos-backend)
3. Start backend: `sam local start-api --port 8000 --host 0.0.0.0 --env-vars env.json --warm-containers eager` (in pedidos-backend)
4. Start frontend: `npm run dev` (in pedidos-front)

## 8. Recommended VS Code Extensions

- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

Configure VS Code to format on save for best results.
