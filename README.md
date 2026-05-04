# Angular 21 — Email Sign Up with Verification, Authentication & Forgot Password
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

A full-featured Angular 21 authentication boilerplate with JWT-based login, email verification, forgot/reset password flow, role-based access control, and an admin panel. Uses a **fake backend interceptor** for development so no real API server is required.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Fake Backend](#fake-backend)
- [Authentication Flow](#authentication-flow)
- [Role-Based Access](#role-based-access)
- [Available Routes](#available-routes)
- [Key Components & Services](#key-components--services)
- [Known Behaviors](#known-behaviors)

---

## Features

- User registration with email verification
- JWT authentication with auto-refresh (token refreshed 1 minute before expiry)
- Forgot password & reset password via tokenized link
- Role-based route protection (`User` / `Admin`)
- Admin panel to create, edit, and delete accounts
- User profile view and update (including password change and account deletion)
- Fake backend interceptor — no real server needed during development
- Alert/notification system with auto-close and fade support

---

## Tech Stack

| Technology | Version |
|---|---|
| Angular | 21.2.7 |
| TypeScript | ~5.9.2 |
| RxJS | ~7.8.0 |
| Bootstrap | 5.2.3 (CDN) |
| Zone.js | ~0.16.1 |

---

## Project Structure

```
src/
├── app/
│   ├── _components/        # Shared components (Alert)
│   ├── _helpers/           # Guards, interceptors, validators, fake backend
│   ├── _models/            # TypeScript models (Account, Alert, Role)
│   ├── _services/          # AccountService, AlertService
│   ├── account/            # Login, Register, Verify Email, Forgot/Reset Password
│   ├── admin/              # Admin layout, overview, account management
│   │   └── accounts/       # List, Add, Edit accounts (admin only)
│   ├── home/               # Home page (authenticated users)
│   ├── profile/            # View and update own profile
│   ├── app.component.*     # Root component & main nav
│   ├── app.module.ts       # Root module
│   └── app-routing.module.ts
├── environments/
│   ├── environment.ts      # Development config
│   └── environment.prod.ts # Production config
└── styles.css
```

---

## Getting Started

### Prerequisites

- Node.js (v18 or higher recommended)
- npm v11+

### Installation

```bash
# Clone the repository
git clone https://github.com/Zagreeed/Galan-Lab7-INTPROG
cd Galan-Lab7-INTPROG

# Install dependencies
npm install
```

### Running the App

```bash
npm start
```

The app will be available at `http://localhost:4200`.

### Building for Production

```bash
npm run build
```

---

## Fake Backend

This project uses an **HTTP interceptor** (`fake-backend.ts`) that simulates a real REST API entirely in the browser using `localStorage`. No backend server is required during development.

To enable it, make sure the following line is **uncommented** in `src/app/app.module.ts`:

```typescript
providers: [
    // ...
    fakeBackendProvider  // ← must be uncommented to use fake backend
]
```

### How it works

- All registered accounts are persisted in `localStorage` under the key `angular-15-signup-verification-boilerplate-accounts`
- JWT tokens are fake but structurally valid (base64-encoded payload with expiry)
- Refresh tokens are stored as browser cookies (`fakeRefreshToken`)
- "Emails" (verification links, reset links) are displayed as in-app alert notifications instead of being sent to a real inbox

### Simulated API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/accounts/authenticate` | Login |
| POST | `/accounts/refresh-token` | Refresh JWT |
| POST | `/accounts/revoke-token` | Logout |
| POST | `/accounts/register` | Register new account |
| POST | `/accounts/verify-email` | Verify email token |
| POST | `/accounts/forgot-password` | Request password reset |
| POST | `/accounts/validate-reset-token` | Validate reset token |
| POST | `/accounts/reset-password` | Reset password |
| GET | `/accounts` | Get all accounts (Admin only) |
| GET | `/accounts/:id` | Get account by ID |
| POST | `/accounts` | Create account (Admin only) |
| PUT | `/accounts/:id` | Update account |
| DELETE | `/accounts/:id` | Delete account |

---

## Authentication Flow

### Registration
1. User fills out the registration form
2. A verification link is displayed as an in-app alert (simulating an email)
3. User clicks the link → navigated to `/account/verify-email?token=...`
4. Token is validated and the account is marked as verified
5. User is redirected to the login page

### Login
1. User submits email and password
2. A JWT token (15-minute expiry) and refresh token (7-day cookie) are issued
3. A background timer auto-refreshes the JWT 1 minute before it expires
4. On 401/403 responses, the user is automatically logged out

### Forgot Password
1. User submits their registered email
2. A password reset link is displayed as an in-app alert (simulating an email)
3. User clicks the link → navigated to `/account/reset-password?token=...`
4. Token is validated server-side before the form is shown
5. User sets a new password and is redirected to login

> **Security note:** If the submitted email is not registered, the app still shows the same success message ("Please check your email..."). This is intentional it prevents **email enumeration attacks** where a malicious user could probe which emails have accounts.

---

## Role-Based Access

The app supports two roles defined in `src/app/_models/role.ts`:

| Role | Access |
|---|---|
| `User` | Home, own Profile (view, update, delete) |
| `Admin` | Everything above + Admin panel (manage all accounts) |

- The **first registered account** is automatically assigned the `Admin` role
- All subsequent registrations are assigned the `User` role
- Admin-only routes are protected by `AuthGuard` with a `data: { roles: [Role.Admin] }` check

---

## Available Routes

| Route | Access | Description |
|---|---|---|
| `/` | Authenticated | Home page |
| `/account/login` | Public | Login form |
| `/account/register` | Public | Registration form |
| `/account/verify-email` | Public | Email verification handler |
| `/account/forgot-password` | Public | Request password reset |
| `/account/reset-password` | Public | Reset password form |
| `/profile` | Authenticated | View profile |
| `/profile/update` | Authenticated | Update profile / change password / delete account |
| `/admin` | Admin only | Admin overview |
| `/admin/accounts` | Admin only | List all accounts |
| `/admin/accounts/add` | Admin only | Create a new account |
| `/admin/accounts/edit/:id` | Admin only | Edit an existing account |

---

## Key Components & Services

### Services

**`AccountService`** — Handles all authentication and account operations. Exposes an `account` Observable that emits the current logged-in account (or `null`).

**`AlertService`** — Global notification system. Supports `success`, `error`, `info`, and `warn` types with optional auto-close and keep-after-route-change behavior.

### Helpers

**`AuthGuard`** — Protects routes from unauthenticated or unauthorized access. Redirects to `/account/login` if not logged in, or to `/` if the role doesn't match.

**`JwtInterceptor`** — Automatically attaches the `Authorization: Bearer <token>` header to all requests going to the configured API URL.

**`ErrorInterceptor`** — Catches 401/403 responses and auto-logs out the current user.

**`MustMatch`** — Custom reactive form validator that ensures two fields (e.g. password & confirm password) have the same value.

**`appInitializer`** — Runs before the app bootstraps to attempt a token refresh, restoring the session if a valid refresh token cookie exists.

---

## Known Behaviors

- **Fake email alerts auto-close after 3 seconds by default.** The verification and reset link alerts are configured with `{ autoClose: false }` so they stay visible until dismissed.
- **Clearing `localStorage`** will remove all registered accounts. Use your browser's DevTools → Application → Local Storage to manage this during testing.
- **Clearing cookies** will invalidate all refresh tokens, effectively logging out all sessions.
- **The fake backend adds a 500ms delay** to all responses to simulate real network latency.