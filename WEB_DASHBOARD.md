# Web Admin & Evidence Verification Dashboard

## 1. Executive Summary
The **Web Admin & Evidence Verification Dashboard** (`D:\SIH\web`) provides law enforcement supervisors, forensic laboratory technicians, and evidence intake personnel with centralized oversight over field drug screenings. It features real-time cryptographic validation via standard Web Crypto, role-based GPS disclosure, and district-level jurisdictional activity reporting.

---

## 2. Technology Stack

- **Framework**: React 18 with TypeScript 5
- **Build Tool**: Vite 5
- **Backend / Sync**: Supabase JS Client (`@supabase/supabase-js`)
- **Cryptography**: Native Browser Web Crypto API (`crypto.subtle.digest('SHA-256', ...)`)
- **QR Generation**: `qrcode` library
- **Icons**: `lucide-react`
- **Routing**: `react-router-dom` v6

---

## 3. Dual-Role Access Control (RBAC)

The dashboard provides role-based permission boundaries with a quick demonstration switcher:

### 3.1 Administrator (`ADMIN`)
- Full access to all jurisdictional test records.
- **Exact GPS Coordinates Unlocked**: Decimal latitude and longitude coordinates are visible (`28.6139° N, 77.2090° E`) with satellite map links for forensic chain-of-custody documentation.
- Live cryptographic integrity verification across all submitted test records.

### 3.2 Field Officer (`OFFICER`)
- Filtered access to assigned officer field screenings.
- **Exact GPS Coordinates Redacted**: Coordinates are masked as `[REDACTED — Admin Access Required for Chain-of-Custody Audit]` to safeguard officer movements and privacy.
- Displays administrative district/jurisdiction overview.

### 3.3 Quick Role Switcher
- Evaluators can toggle between **Administrator** and **Field Officer** mode with one click in the navigation header to verify role boundary enforcement.

---

## 4. Application Routes & Key Features

| Route | View Component | Description |
|---|---|---|
| `/login` | `LoginPage.tsx` | Law enforcement authentication with one-click Administrator/Officer demo buttons and statutory notice. |
| `/dashboard` | `DashboardPage.tsx` | KPI metric cards (Total, Positive, Negative, Inconclusive, Sync Health, Record Integrity), presumptive results breakdown, and safe district activity summaries. |
| `/tests` | `TestsPage.tsx` | Comprehensive registry with search by Test ID/District, filters by presumptive result and sync state, and GPS privacy indicators. |
| `/test/:id` | `TestDetailPage.tsx` | Full chain-of-custody record audit, live Web Crypto SHA-256 verification button, role-based GPS disclosure, and generated QR token. |
| `/verification` | `VerificationPage.tsx` | QR payload decoder and validator with one-click **Load Valid Sample** and **Load Tampered Sample** demo buttons, prominent tamper warning alerts, and statutory disclaimers. |

---

## 5. Development & Production Build Instructions

### 5.1 Prerequisites
- Node.js &ge; 18.x (Node v24.14.0 verified)
- npm &ge; 9.x (npm 11.9.0 verified)

### 5.2 Running the Development Server
```bash
cd D:\SIH\web
npm run dev
```
The application will launch locally at `http://localhost:5173`.

### 5.3 Compiling Production Assets
```bash
cd D:\SIH\web
npm run build
```
Builds optimized production assets to `D:\SIH\web\dist` with zero TypeScript errors.

### 5.4 Environment Configuration (Optional)
Create a `.env` file in `D:\SIH\web` to link to a live Supabase instance:
```ini
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOi... (publishable anon key ONLY)
```
*Note: If no `.env` is configured, the dashboard operates seamlessly using embedded deterministic seed records, fully supporting offline demonstrations.*
