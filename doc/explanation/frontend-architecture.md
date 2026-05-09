# Frontend Architecture: Next.js Dashboard

> [!NOTE]
> This document provides a comprehensive technical reference for the DBT Leakage Detection System's frontend application. It covers the full technology stack, all route definitions, page-level content specifications, component architecture, and the security proxy layer (`proxy.ts`) that governs all communication between the browser and the ML Engine.

## Table of Contents
1. [Technology Stack Overview](#1-technology-stack-overview)
2. [Project Structure](#2-project-structure)
3. [Route Architecture](#3-route-architecture)
   - [3.1 Public Routes](#31-public-routes)
   - [3.2 Authenticated Routes (District Finance Officer)](#32-authenticated-routes-district-finance-officer)
   - [3.3 Authenticated Routes (Field Verifier)](#33-authenticated-routes-field-verifier)
   - [3.4 Authenticated Routes (State Administrator)](#34-authenticated-routes-state-administrator)
   - [3.5 API Routes (Internal)](#35-api-routes-internal)
4. [Component Library](#4-component-library)
5. [The Proxy Layer: `proxy.ts`](#5-the-proxy-layer-proxyts)
6. [State Management](#6-state-management)
7. [Animation Architecture (Framer Motion)](#7-animation-architecture-framer-motion)
8. [Theming and Design System](#8-theming-and-design-system)

---

## 1. Technology Stack Overview

The frontend is a **Next.js 14** application using the App Router paradigm. It is purpose-built for a high-trust government environment, emphasizing data integrity, role-based access control, and performance at scale.

### Core Framework

| Technology | Version | Purpose |
|---|---|---|
| **Next.js** | 14.x | Full-stack React framework (App Router) |
| **React** | 18.x | UI component library |
| **TypeScript** | 5.x | Type safety across all files |

### Styling

| Technology | Version | Purpose |
|---|---|---|
| **Tailwind CSS** | 3.x | Utility-first, component-level styling |
| **shadcn/ui** | Latest | Pre-built, accessible component primitives |
| **Lucide React** | Latest | Consistent, scalable icon set |

### Animation

| Technology | Version | Purpose |
|---|---|---|
| **Framer Motion** | 11.x | Declarative, physics-based UI animations |

### Data & Networking

| Technology | Version | Purpose |
|---|---|---|
| **TanStack Query (React Query)** | 5.x | Server-state management, caching, and data fetching |
| **Axios** | 1.x | HTTP client for API calls through the proxy layer |
| **Zod** | 3.x | Runtime schema validation of API response payloads |

### Visualization

| Technology | Version | Purpose |
|---|---|---|
| **Recharts** | 2.x | Composable charting library for dashboards |
| **Mapbox GL JS** | 3.x | Interactive geographic maps for field verifier tracking |

---

## 2. Project Structure

```
frontend/
├── app/                          # Next.js App Router
│   ├── layout.tsx                # Root layout (fonts, providers, navbar)
│   ├── page.tsx                  # Public landing/login page (Route: /)
│   ├── (auth)/                   # Auth route group
│   │   └── login/
│   │       └── page.tsx          # Login page (Route: /login)
│   ├── (dfo)/                    # District Finance Officer route group
│   │   ├── layout.tsx            # DFO-specific sidebar layout
│   │   ├── dashboard/
│   │   │   └── page.tsx          # Main DFO dashboard (Route: /dashboard)
│   │   ├── flags/
│   │   │   ├── page.tsx          # Flagged transactions list (Route: /flags)
│   │   │   └── [id]/
│   │   │       └── page.tsx      # Flag detail view (Route: /flags/[id])
│   │   ├── upload/
│   │   │   └── page.tsx          # Batch CSV upload (Route: /upload)
│   │   └── reports/
│   │       └── page.tsx          # Reports & exports (Route: /reports)
│   ├── (verifier)/               # Field Verifier route group
│   │   ├── layout.tsx            # Verifier-specific mobile layout
│   │   ├── assignments/
│   │   │   └── page.tsx          # Active cases (Route: /assignments)
│   │   └── assignments/[id]/
│   │       └── page.tsx          # Case filing form (Route: /assignments/[id])
│   └── (admin)/                  # State Administrator route group
│       ├── layout.tsx
│       ├── overview/
│       │   └── page.tsx          # Statewide analytics (Route: /overview)
│       ├── schemes/
│       │   └── page.tsx          # Scheme management (Route: /schemes)
│       └── users/
│           └── page.tsx          # User management (Route: /users)
├── components/
│   ├── ui/                       # Primitive shadcn/ui components
│   ├── charts/                   # Recharts-based dashboard widgets
│   ├── tables/                   # Data table components
│   └── layout/                   # Navbar, Sidebar, PageHeader
├── lib/
│   ├── proxy.ts                  # The security proxy layer (see Section 5)
│   ├── auth.ts                   # NextAuth.js configuration
│   └── utils.ts                  # cn() helper and general utilities
├── hooks/
│   ├── useFlags.ts               # TanStack Query hook for flagged transactions
│   ├── useUpload.ts              # File upload mutation hook
│   └── useAnalytics.ts           # Dashboard metrics hook
└── types/
    └── index.ts                  # Global TypeScript interface definitions
```

---

## 3. Route Architecture

The application uses Next.js **Route Groups** (e.g., `(dfo)`, `(verifier)`, `(admin)`) to co-locate role-specific pages and apply separate, nested layouts without affecting the URL path. Access control for each group is enforced at the middleware level (see `proxy.ts`).

---

### 3.1 Public Routes

#### `GET /` — Landing Page (`app/page.tsx`)
The public-facing entry point of the application. This is a non-authenticated page.

**Content:**
- A full-viewport hero section with the DBT Leakage Detection System logo and a brief, one-line value proposition.
- A prominent "Sign In" call-to-action button that routes to `/login`.
- A static "Key Capabilities" section with three cards (Anomaly Detection, Duplicate Identity, Prescriptive AI), each with an icon from Lucide React.
- Subtle entrance animations on scroll powered by Framer Motion's `useInView` hook.

#### `GET /login` — Login Page (`app/(auth)/login/page.tsx`)
The authentication gateway.

**Content:**
- A centered login card with a clean, government-grade aesthetic (no playful branding).
- **Employee ID / Email** input field with Zod-validated format checking.
- **Password** input field with visibility toggle.
- A **"Sign In"** button that triggers a `next-auth` `signIn()` call.
- Role-based redirect: after successful authentication, `next-auth` reads the `role` field from the JWT and redirects to the appropriate default dashboard (`/dashboard`, `/assignments`, or `/overview`).
- On failure, a red alert banner (powered by shadcn/ui `Alert`) is displayed inline without a page reload.

---

### 3.2 Authenticated Routes (District Finance Officer)

These routes are accessible only to users with the `ROLE_DFO` JWT claim. They share a persistent left-sidebar layout defined in `app/(dfo)/layout.tsx`.

#### `GET /dashboard` — Main DFO Dashboard (`app/(dfo)/dashboard/page.tsx`)

The operational command center for the District Finance Officer.

**Content:**
- **KPI Ribbon (Top Row):** Four stat cards showing:
  - Total transactions processed in the current cycle.
  - Total funds disbursed (in ₹ Crores).
  - Active flagged anomalies (clickable, routes to `/flags`).
  - Percentage change vs. the prior payment cycle.
- **Risk Score Distribution Chart (Recharts):** A horizontal bar chart grouping flagged transactions into buckets: `Critical (71-100)`, `Medium (31-70)`, `Low (0-30)`.
- **Leakage Typology Pie Chart (Recharts):** A donut chart showing the breakdown of flag types: `DECEASED_DISBURSEMENT`, `DORMANT_UNDRAWN_FUNDS`, `DUPLICATE_IDENTITY_TRANSLITERATION`, `CROSS_SCHEME_VIOLATION`.
- **Recent Activity Feed:** A scrollable list of the last 10 flagged transactions, each showing beneficiary name, risk score, flag type, and a "Review" button.
- All charts and data fetch from `GET /api/analytics/summary` via the `useAnalytics` TanStack Query hook, with a 60-second `staleTime`.

#### `GET /flags` — Flagged Transactions List (`app/(dfo)/flags/page.tsx`)

The master list of all anomalies detected in the current and previous payment cycles.

**Content:**
- **Filter Bar:** Multi-select dropdowns for `Flag Type`, `Risk Level`, `Scheme`, and `District`. A date range picker for `Transaction Date`.
- **Sortable Data Table:** Built using `@tanstack/react-table` with the following columns:
  - `Beneficiary ID`
  - `Name`
  - `Aadhaar (Masked)` — displays only last 4 digits for privacy
  - `Flag Type` — rendered as a color-coded badge (red for `DECEASED`, orange for `DORMANT`, yellow for `DUPLICATE`)
  - `Risk Score` — rendered as a progress bar
  - `Amount (₹)`
  - `Status` — `OPEN`, `DISPATCHED`, `RESOLVED`, `DISMISSED`
  - `Action` — a "View Details" button routing to `/flags/[id]`
- **Pagination:** Server-side pagination controlled via URL search params (`?page=1&limit=50`).
- **CSV Export:** A button in the top-right that calls `GET /api/flags/export` to download the filtered view as a CSV file.

#### `GET /flags/[id]` — Flag Detail View (`app/(dfo)/flags/[id]/page.tsx`)

A dedicated deep-dive page for a single flagged transaction.

**Content:**
- **Evidence Panel (Left Column):**
  - Full beneficiary profile: Name, Aadhaar, Date of Birth, Address, Scheme, Bank Account.
  - Transaction history table showing all disbursements for this beneficiary.
  - **Anomaly Evidence:** The specific data points that triggered the flag. For a `DECEASED_DISBURSEMENT`, this section shows the `death_date` pulled from the Civil Registry and the `transaction_date` that occurred after it, with the calculated `days_post_mortem`.
- **AI Prescriptive Suggestion Panel (Right Column):**
  - A human-readable, plain-English suggestion generated by the Prescriptive AI (Model D).
  - The `fault` string (e.g., "Aadhaar matched State Death Registry").
  - The `action` string (e.g., "Immediately halt funds. Do not dispatch verifier.").
  - A confidence tag (`HIGH`, `MEDIUM`, `LOW`).
- **Action Buttons:**
  - `Freeze Funds` — calls `POST /api/flags/[id]/freeze`, optimistically updates the UI.
  - `Dispatch Verifier` — opens a modal to select and assign a field verifier from the district roster.
  - `Dismiss Flag` — opens a modal requiring the DFO to enter a reason for dismissal (logged to audit trail).

#### `GET /upload` — Batch CSV Upload (`app/(dfo)/upload/page.tsx`)

The interface for initiating a new payment cycle analysis.

**Content:**
- A full-page drag-and-drop upload zone (using `react-dropzone`) that accepts `.csv` files only, with a max size limit of `50MB`.
- **Pre-flight Schema Validator:** After a file is selected but *before* upload, the frontend reads the CSV headers using a Web Worker and validates them against the expected schema (`beneficiary_id`, `aadhaar`, `name`, `amount`, `withdrawn`, `scheme`, `district`). If any column is missing, an inline error banner appears and the upload button remains disabled.
- **Upload Progress Bar:** A `framer-motion`-animated progress bar that reflects the actual `onUploadProgress` event from the Axios call.
- **Analysis Status Feed:** Once the file is uploaded, the page subscribes to a Server-Sent Events (SSE) stream from `GET /api/upload/status/[jobId]` and renders a real-time feed showing: `File Received`, `Preprocessing...`, `Isolation Forest Running...`, `Fuzzy Matching...`, `Complete — 142 flags detected`.

#### `GET /reports` — Reports & Exports (`app/(dfo)/reports/page.tsx`)

Historical reporting and compliance document generation.

**Content:**
- A list of previously processed payment cycles with their analysis summaries.
- Ability to download a detailed PDF compliance report for any past cycle using a `POST /api/reports/generate` endpoint.
- Trend charts showing how the number of flags has changed over the past 12 months.

---

### 3.3 Authenticated Routes (Field Verifier)

These routes are accessible only to users with the `ROLE_VERIFIER` JWT claim. The layout (`app/(verifier)/layout.tsx`) is optimized for mobile use (larger tap targets, simplified navigation), as verifiers primarily access this in the field on a smartphone.

#### `GET /assignments` — Active Assignments (`app/(verifier)/assignments/page.tsx`)

A mobile-first list of cases assigned to the logged-in verifier.

**Content:**
- A card-based list (not a table) of assigned cases, each showing:
  - Beneficiary name and address.
  - Flag type and risk score.
  - Distance from verifier's current location (using the browser `navigator.geolocation` API).
  - A "Open Case" button routing to `/assignments/[id]`.
- Cases are sorted by proximity by default.

#### `GET /assignments/[id]` — Case Filing Form (`app/(verifier)/assignments/[id]/page.tsx`)

The form a field verifier uses to submit their on-ground findings.

**Content:**
- Full case context displayed at the top (beneficiary details, the AI's prescriptive suggestion).
- **Findings Form:**
  - `Beneficiary Present at Address?` — Yes/No radio.
  - `Physical Condition` — Multi-select checkboxes (Healthy, Elderly, Deceased, Unreachable).
  - `Passbook in Beneficiary's Possession?` — Yes/No radio.
  - `Notes` — A free-text field for additional observations.
- **GPS Stamp:** On form submission, the browser captures the verifier's current coordinates (`navigator.geolocation.getCurrentPosition`) and attaches them to the payload, creating an immutable, GPS-verified audit record.
- **Photo Evidence Upload:** Allows the verifier to upload 1-3 photos directly from their phone camera.
- Form data is submitted via `POST /api/assignments/[id]/submit`.

---

### 3.4 Authenticated Routes (State Administrator)

These routes are accessible only to users with the `ROLE_ADMIN` JWT claim.

#### `GET /overview` — Statewide Analytics (`app/(admin)/overview/page.tsx`)

A high-level command center for the State Administrator.

**Content:**
- **Interactive Mapbox Map:** An interactive map of Gujarat with each district rendered as a choropleth layer. Districts with higher leakage rates are colored in deeper red shades, giving an instant geographic overview of problem areas.
- **Statewide KPIs:** Total funds saved (recovered/frozen), total flags across all districts, top-5 flagged schemes.
- **District League Table:** A sortable table ranking all districts by total flagged amount and anomaly count.

#### `GET /schemes` — Scheme Management (`app/(admin)/schemes/page.tsx`)

A CRUD interface for managing active welfare schemes.

**Content:**
- A list of all registered schemes with their eligibility rules (mutually exclusive schemes are explicitly tagged).
- The ability to add, edit, or deactivate schemes.
- Editing a scheme's exclusivity rules automatically updates the ML engine's `Cross-Scheme Violation` detection logic on the next upload cycle.

#### `GET /users` — User Management (`app/(admin)/users/page.tsx`)

Manage DFO and Field Verifier accounts.

**Content:**
- A table of all system users with their name, role, district, and account status.
- Ability to create, edit, or deactivate user accounts.
- Role assignment (DFO or Verifier) and district mapping.

---

### 3.5 API Routes (Internal)

These are Next.js API Route Handlers (`app/api/`) that act as a Backend-for-Frontend (BFF) layer. They are not called directly by external clients. All communication from browser components goes through these handlers, which then relay requests to the Python ML Engine after validation. The security layer that governs this is `proxy.ts`.

| Method | Route | Purpose |
|---|---|---|
| `GET` | `/api/analytics/summary` | Returns KPIs for the DFO dashboard |
| `GET` | `/api/flags` | Returns paginated list of flags |
| `GET` | `/api/flags/[id]` | Returns a single flag with evidence |
| `POST` | `/api/flags/[id]/freeze` | Freezes funds for a flagged transaction |
| `POST` | `/api/flags/[id]/dispatch` | Assigns a field verifier to a flag |
| `POST` | `/api/flags/[id]/dismiss` | Dismisses a flag with a reason |
| `GET` | `/api/flags/export` | Streams a CSV export of filtered flags |
| `POST` | `/api/upload` | Accepts a CSV file and queues an analysis job |
| `GET` | `/api/upload/status/[jobId]` | SSE stream for real-time analysis progress |
| `POST` | `/api/reports/generate` | Triggers PDF report generation |
| `POST` | `/api/assignments/[id]/submit` | Accepts field verifier case submission |

---

## 5. The Proxy Layer: `proxy.ts`

The `proxy.ts` file (located at `lib/proxy.ts`) is the most critical security module in the frontend codebase. It is **not a standard HTTP proxy**. It is a programmatic function used inside every Next.js API Route Handler to intercept, validate, authorize, and then relay each request before it reaches the Python ML Engine.

No browser component ever calls the Python ML Engine directly. All calls go:

```
Browser Component
    → (TanStack Query / Axios)
    → Next.js API Route Handler (e.g., /api/flags)
    → proxy.ts (validates & transforms)
    → Python Flask ML Engine (http://localhost:5000)
```

### What `proxy.ts` Checks

The `proxy.ts` function receives the incoming `NextRequest` and the current `Session` (from `next-auth`) and performs the following checks in strict sequential order. If any check fails, a standardized error response is returned immediately and the request is never forwarded to the backend.

#### Check 1: Session Validity
```typescript
if (!session || !session.user) {
  return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
}
```
It verifies that a valid, non-expired JWT session exists for the requester. If the session cookie is missing or expired, the request is immediately rejected with a `401 Unauthorized` response. This prevents unauthenticated callers from probing the ML Engine.

#### Check 2: Role-Based Authorization (RBAC)
```typescript
const allowedRoles = options.allowedRoles; // e.g., ['ROLE_DFO', 'ROLE_ADMIN']
if (!allowedRoles.includes(session.user.role)) {
  return NextResponse.json({ error: "Forbidden" }, { status: 403 });
}
```
Each API Route Handler calls `proxy.ts` with a configuration object specifying which roles are permitted. The proxy reads the `role` claim from the verified JWT and checks if the user's role is in the allowed list. A Field Verifier trying to call the `POST /api/flags/[id]/freeze` endpoint (which is DFO-only) will receive a `403 Forbidden` response.

#### Check 3: Request Payload Schema Validation (Zod)
```typescript
const schema = options.bodySchema; // e.g., z.object({ reason: z.string().min(10) })
const parsed = schema.safeParse(await request.json());
if (!parsed.success) {
  return NextResponse.json({ error: parsed.error.flatten() }, { status: 422 });
}
```
If the route accepts a request body, the proxy validates it against a Zod schema defined in the route handler options. This prevents malformed or malicious payloads from being forwarded to the Python backend. For example, the `Dismiss Flag` endpoint requires the `reason` field to be a string of at least 10 characters. If the client sends an empty string, the proxy returns `422 Unprocessable Entity` before the Python engine is ever touched.

#### Check 4: Destination URL Construction & Sanitization
```typescript
const backendUrl = `${process.env.ML_ENGINE_URL}${options.backendPath}`;
```
The proxy constructs the upstream URL to the Flask ML engine using a hardcoded server-side environment variable (`ML_ENGINE_URL`). The backend URL is **never** derived from any user-controlled input (URL params, query strings, or body fields). This prevents Server-Side Request Forgery (SSRF) attacks where a malicious actor might attempt to redirect the server into calling an internal network resource.

#### Check 5: Header Injection (Service-to-Service Authentication)
```typescript
headers: {
  'Content-Type': 'application/json',
  'X-Internal-API-Key': process.env.ML_ENGINE_SECRET_KEY,
  'X-Forwarded-User': session.user.id,
  'X-Forwarded-Role': session.user.role,
}
```
Before relaying the request, the proxy injects critical headers:
- `X-Internal-API-Key`: A shared secret key that the Python Flask engine verifies on every incoming request. This ensures the ML Engine refuses direct calls from anything other than the Next.js proxy, even if an attacker discovers the Flask server's URL and port.
- `X-Forwarded-User` and `X-Forwarded-Role`: The authenticated user's ID and role are forwarded as trusted headers so the Python backend can write them into the audit log without needing its own authentication system.

#### Check 6: Response Validation & Error Normalization
```typescript
const upstreamResponse = await axios.post(backendUrl, parsed.data, { headers });
if (upstreamResponse.status !== 200) {
  // Normalize the error before returning it to the browser
  return NextResponse.json({ error: "ML Engine Error" }, { status: 502 });
}
```
The proxy catches any errors from the ML Engine and normalizes them into a consistent error shape before returning them to the browser. This prevents raw Python stack traces or internal server error messages from leaking to the client. The browser always receives a clean, structured JSON error response.

---

## 6. State Management

The application avoids a global client-side state store (like Redux or Zustand) for server data. Instead, it uses a two-layer approach:

1.  **Server Data (TanStack Query):** All remote data (flags, analytics, schemes) is managed by TanStack Query. It handles caching, background refetching, and loading/error states automatically. Components declare their data needs via custom hooks (e.g., `useFlags`, `useAnalytics`).
2.  **UI State (React `useState` / `useReducer`):** Local, ephemeral UI state (modal open/close, form field values, filter selections) is handled directly with React hooks at the component level.

---

## 7. Animation Architecture (Framer Motion)

Animations are applied selectively to prevent cognitive overload in a high-stress operational tool.

| Animation Type | Usage | Framer Motion API |
|---|---|---|
| Page Transitions | Fade + slide on route change | `AnimatePresence` + `motion.div` |
| KPI Card Entrance | Staggered entrance from bottom | `variants` with `staggerChildren` |
| Flag Badge Pulse | Subtle pulse on `CRITICAL` risk badges | `animate={{ scale: [1, 1.05, 1] }}` |
| Upload Progress | Smooth width animation on progress bar | `motion.div` with `width` animated value |
| Data Table Row | Fade in on new data load | `layout` prop for list reordering |

---

## 8. Theming and Design System

The design system is configured in `tailwind.config.ts` and `app/globals.css`.

- **Typography:** The `Inter` variable font is loaded via `next/font/google`. It is applied as the root font on the `<html>` element.
- **Color Palette:** A custom Tailwind CSS palette is defined using CSS Custom Properties to support a light/dark mode toggle:
  - **Primary Brand:** `--color-primary: 220 80% 50%` (a confident, government-grade blue).
  - **Destructive / Critical Risk:** `--color-destructive: 0 75% 55%` (a clear, authoritative red).
  - **Warning / Medium Risk:** `--color-warning: 38 95% 50%` (an amber/orange).
  - **Surface:** `--color-background` and `--color-card` are separate, allowing cards to visually lift from the page background.
- **Dark Mode:** Implemented via the `class` strategy in Tailwind. A `ThemeProvider` component (wrapping the root layout) reads from `localStorage` and adds `class="dark"` to the `<html>` element.
