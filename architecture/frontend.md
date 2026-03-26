# Frontend Architecture

## Tech Stack

| Technology | Version / Detail |
|---|---|
| React | 19 |
| Vite | 7 |
| Node.js | v24.14.0 LTS (managed with fnm) |
| Tailwind CSS | v4 (`@tailwindcss/vite` plugin) |
| UI components | shadcn/ui (manual) |
| Routing | React Router DOM v7 |
| Linting | ESLint 9 + Prettier |

## Project Structure

```
src/
├── App.jsx                  # Root: providers + routes
├── index.css                # Tailwind imports + dark mode
├── main.jsx                 # Entry point
├── components/
│   ├── ui/                  # shadcn/ui primitives
│   ├── layout/              # Header, AppLayout, MobileUserBar
│   ├── auth/                # GuestRoute, ProtectedRoute
│   ├── addresses/           # AddressForm, AddressList
│   ├── catalog/             # ProductCard, ProductDetailView
│   ├── checkout/            # PaymentMethodSelector
│   ├── loyalty/             # PointsBadge, CouponInput, RedeemPoints
│   └── panel/               # ActiveOrder, OrderHistory, Points, Account
├── context/                 # Each context = 2 files
│   ├── *-context.js         #   createContext() export
│   └── *Context.jsx         #   Provider with logic
├── hooks/                   # useAuth, useTheme, useAddresses, useCart, useLoyalty
├── pages/                   # One page per route
│   └── admin/               # Admin panel pages
├── services/
│   ├── api.js               # Axios-like wrapper with JWT
│   └── handlers.js          # All API call functions
├── mocks/                   # Legacy mock data (reference only)
├── lib/utils.js             # cn() helper
└── utils/                   # Generic utilities
```

## Provider Architecture

Provider order in `App.jsx` (context dependencies flow top → down):

```
ThemeProvider
  → AuthProvider
    → AddressProvider
      → CartProvider
        → LoyaltyProvider
          → BrowserRouter (routes)
```

`LoyaltyProvider` depends on `useAuth` to determine user eligibility.

## Code Conventions

### Files and structure
- **Path alias**: `@/` → `./src` (configured in `vite.config.js`)
- **Contexts**: split into 2 files to avoid ESLint react-refresh errors
  - `.js` file exports `createContext()`
  - `.jsx` file exports the Provider component
- **Hooks**: one file per context (`use{Domain}.js`)
- **No setState inside useEffect** — use derived state or `useState(initializer)`

### Style and UI
- **Mobile first**: customer-facing views target mobile
- **Desktop first**: admin panel targets desktop
- **Dark mode**: class-based (`dark:` variants), persisted in localStorage, detects system preference
- **Optional fields**: labeled "(opcional)"; required fields have no asterisk
- **Tailwind breakpoints**: xs (default) → sm (640px) → md (768px) → lg (1024px)

## Backend Connection

All API calls go through `src/services/api.js` → `src/services/handlers.js`.

- **Base URL**: `VITE_API_URL` env var (default: `http://localhost:8000/api`)
- **Auth**: `Authorization: Bearer {token}` header added automatically from localStorage
- **Response unwrapping**: automatically extracts `{ data: ... }` from backend responses
- The old mock layer (`src/mocks/`) is kept as reference but not imported

## Current State (v0.22.1)

### Implemented
- Auth: login, registration, guest mode, Google mock, password recovery
- Dark mode with system preference detection
- Address CRUD with coverage validation (Haversine)
- Ice cream catalog: 16 flavors, format selection, extras
- Cart with format, flavors, extras, comment
- Checkout: delivery/pickup, address selector, delivery cost by zones
- Loyalty: points (1 peso = 1 point), redemption, discount coupons
- Payment methods: Mercado Pago, bank transfer, card, cash on delivery
- Order confirmation with payment status and points earned
- User panel: active order tracking, order history, points, account
- Admin panel: orders board, products CRUD, flavor lists CRUD, combos, counter orders
- Backend integration: all API calls via `src/services/handlers.js`
