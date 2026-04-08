# CLAUDE.md — AI Agent Context

Instructions and context for any agent working on this project.

## Project

**Ainara Helados** — Front-end for a delivery/pickup ordering system. Currently themed as an artisanal ice cream shop, but the architecture is generic and could be adapted to any food business (restaurant, bakery, etc.). The backend will be Python in a separate repository; for now all backend logic is mocked.

## Tech Stack

| Technology | Version / Detail |
|---|---|
| React | 19 |
| Vite | 7 |
| Node.js | v24.14.0 LTS (pinned in `.node-version`, managed with fnm) |
| Package manager | npm |
| Tailwind CSS | v4 (`@tailwindcss/vite` plugin) |
| UI components | shadcn/ui (manually created in `src/components/ui/`) |
| Routing | React Router DOM v7 |
| Linting | ESLint 9 + Prettier |
| Developer OS | Windows (PowerShell in VS Code) |

## Project Structure

```
src/
├── App.jsx                  # Root: providers + routes
├── index.css                # Tailwind imports + dark mode variant
├── main.jsx                 # Entry point
├── components/
│   ├── ui/                  # shadcn/ui: button, input, label, card, textarea
│   ├── layout/              # Header, AppLayout, MobileUserBar
│   ├── auth/                # GuestRoute, ProtectedRoute
│   ├── addresses/           # AddressForm, AddressList
│   ├── catalog/             # ProductCard
│   ├── checkout/            # PaymentMethodSelector
│   ├── loyalty/             # PointsBadge, CouponInput, RedeemPoints
│   └── panel/               # ActiveOrderSection, OrderHistorySection, PointsSection, AccountSection
├── context/                 # Each context uses 2 files:
│   ├── auth-context.js      #   - *-context.js → createContext()
│   ├── AuthContext.jsx       #   - *Context.jsx → Provider with logic
│   ├── theme-context.js
│   ├── ThemeContext.jsx
│   ├── address-context.js
│   ├── AddressContext.jsx
│   ├── cart-context.js
│   ├── CartContext.jsx
│   ├── loyalty-context.js
│   └── LoyaltyContext.jsx
├── hooks/                   # useAuth, useTheme, useAddresses, useCart, useLoyalty
├── mocks/
│   ├── data.js              # Mock data: flavors, menu, categories, points, coupons, addresses, zones, payment methods
│   └── handlers.js          # Async functions with simulated delay (replace with real API)
├── pages/                   # One page per route
├── services/
│   └── api.js               # Prepared for real backend connection
├── lib/
│   └── utils.js             # cn() helper (clsx + tailwind-merge)
└── utils/                   # Generic utilities
```

## Code Conventions

### Files and structure
- **Path alias**: `@/` points to `./src` (configured in `vite.config.js`)
- **Contexts**: split into 2 files to avoid ESLint react-refresh errors. The `.js` file exports `createContext()`, the `.jsx` file exports the Provider
- **Hooks**: one file per context in `src/hooks/`, named `use{Domain}.js`
- **No setState inside useEffect** — use derived state or `useState(initializer)`. ESLint flags this as an error

### Style and UI
- **Mobile first**: client-facing design targets mobile. Desktop layout reserved for future admin panel
- **Dark mode**: class-based (`dark:` variants). Toggle in header, persisted in localStorage, detects system preference
- **Optional fields**: labeled "(opcional)". Required fields have no asterisk
- **Tailwind breakpoints**: xs (default) → sm (640px) → md (768px) → lg (1024px)

### Git and versioning
- **Commit messages**: English, conventional commits format (`feat:`, `fix:`, `style:`, `docs:`, `refactor:`)
- **Branch**: `master`, direct push to `origin/master`
- **Changelog**: in `CHANGELOG.md`, ordered newest to oldest. Roadmap at the bottom. **Every version bump MUST include its changelog entry before committing**
- **Versions**: semver — major features are minor bumps (v0.X.0), fixes/improvements are patch (v0.X.Y)
- **Workflow**: after each feature, improvement or fix → update CHANGELOG → update roadmap (mark completed items) → `git add -A` → `git commit` → `git push origin master`. Every change gets committed and pushed immediately with a new version

### Conversation
- The user speaks **Spanish**
- Commit messages are in **English**
- Code (variables, technical comments) in **English**
- UI labels and text in **Spanish** (Argentina)

## Provider Architecture

Provider order in `App.jsx` matters (context dependencies):

```
ThemeProvider → AuthProvider → AddressProvider → CartProvider → LoyaltyProvider → BrowserRouter
```

`LoyaltyProvider` depends on `useAuth` to determine user eligibility.

## Backend Connection

The app connects to `pedidos-backend` (Python / AWS SAM) via `src/services/api.js`. All handlers live in `src/services/handlers.js` and call the REST API.

**Configuration**: set `VITE_API_URL` in `.env` (default: `http://localhost:8000/api`). The backend runs with `sam local start-api --port 8000`.

The old mock layer (`src/mocks/`) is kept as reference but is no longer imported.

### Test Data
- **Mock user**: `juan@test.com` / `1234` (id: 1)
- **Coupons**: `HELADOGRATIS` ($3,500 fixed), `VERANO20` (20%), `AINARA10` (10%), `EXPIRADO` (expired)
- **Mock user points**: ~16,800 available
- **3 mock addresses**: Casa (CABA, in coverage), Trabajo (CABA, in coverage), Casa de mamá (Pilar, out of coverage)
- **Coverage zone**: 5 km from CABA center (-34.6037, -58.3816)
- **Delivery zones**: Cercana ≤1.5km ($500), Media ≤3km ($800), Lejana ≤5km ($1,200)

## Current State (v0.20.0)

### Implemented
- Auth: login, registration, guest mode, Google mock, password recovery
- Dark mode with system preference detection
- Address CRUD with coverage validation (Haversine)
- Ice cream catalog: 16 flavors, format selection, extras
- Cart with format, flavors, extras, comment
- Checkout: delivery/pickup, address selector, delivery cost by zones
- Loyalty program: points (1 peso = 1 point), redemption, discount coupons
- Responsive header with MobileUserBar for small screens
- Payment methods: Mercado Pago, bank transfer, card, cash on delivery
- Order confirmation page with payment status and points earned
- User panel: active order tracking, order history, points balance, account management
- Admin panel: orders board, products CRUD, flavor lists CRUD, combos, counter orders
- **Backend integration**: all API calls go to pedidos-backend via `src/services/handlers.js`

### Pending (roadmap)
- **Future**: real-time notifications, testing, CI/CD

## Useful Commands

```bash
# Development
npm run dev              # Start dev server (Vite)

# Lint
npx eslint src/          # Check for lint errors

# Expose with ngrok (previously configured in vite.config.js)
ngrok http 5173
```

## Important Notes

1. Backend is in `pedidos-backend` repo — run with `sam local start-api --port 8000` + DynamoDB Local on port 8100
2. Points are only earned by registered users (not guests)
3. The `minOrder` field on coupons was left intentionally for future admin panel configuration
4. Guest address geolocation is a placeholder — uses fixed coordinates for now
5. `server.allowedHosts` in vite.config.js includes `*.ngrok-free.app` for remote testing
6. Full requirements specification is in `docs/requirements.md`
