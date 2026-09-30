# Security Architecture & Data Protection

## 1. Executive Summary
The **Digital Companion for Field Drug Testing** is an evidence-grade field screening and audit platform built for law enforcement officers and forensic intake personnel. The platform enforces end-to-end cryptographic record sealing, zero-leakage QR evidence verification, strict role-based access control (RBAC), and offline-first data synchronization.

---

## 2. Cryptographic Record Integrity

### 2.1 Deterministic Record Hashing
Every completed presumptive field test is sealed using a canonical, deterministic SHA-256 digest. This binds the immutable facts of the test into a tamper-evident seal before local persistence or cloud transmission.

The canonical string is constructed using pipe delimiters (`|`) in fixed order:
```
canonical = testId | kitId | result | formattedConfidence | timestamp | safeImageHash
```

| Parameter | Type / Format | Example | Description |
|---|---|---|---|
| `testId` | String | `DFT-20260930-982145A1` | Unique field test identifier |
| `kitId` | String (UUID / ID) | `c0000000-0000-0000-0000-000000000001` | Reagent kit identifier |
| `result` | Enum String | `POSITIVE`, `NEGATIVE`, `INCONCLUSIVE` | Presumptive classification |
| `confidence` | Float (4 decimals) | `0.9650` | Formatted to 4 decimal places (`%.4f`) |
| `timestamp` | Epoch Milliseconds | `1727696520000` | Epoch timestamp in milliseconds |
| `imageHash` | String (SHA-256) | `3a7bd3e2...4f1b` | Hash of captured evidence photo |

### 2.2 Image SHA-256 Binding
- Upon camera capture via CameraX, the raw JPEG bytes are digested using SHA-256 before saving to local encrypted/app-private storage.
- The `imageHash` is directly embedded into the canonical record string. If the evidence photo is modified, replaced, or degraded after capture, the record hash verification fails immediately.

### 2.3 Integrity Lifecycle States
- **`VALID`**: Computed SHA-256 digest matches the embedded/transmitted record seal.
- **`INVALID`**: Mismatch detected between record facts and sealed hash (evidence of tampering).
- **`NOT_VERIFIED`**: Record seal has not yet been computed or verified.

### 2.4 Edge-Only Processing (Zero External AI APIs)
- All image quality analysis (Laplacian blur variance, brightness, sharpness) and reference card color calibration run on-device using OpenCV.
- All ML inference runs locally via TFLite/embedded classifier models.
- **Zero raw evidence images or suspect sample features are ever transmitted to external AI APIs or third-party web services.**

---

## 3. Database & Cloud Security Architecture

### 3.1 Key Management & Secret Protection
- **Client Android Application**: Uses exclusively the public Supabase `anon` / publishable key loaded via `local.properties` &rarr; `BuildConfig`.
- **Web Dashboard**: Reads client-side configuration exclusively from `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`.
- **Prohibition of `service_role`**: The high-privilege `service_role` key is **strictly prohibited** in the Android app, client web bundle, Git repository, and Logcat outputs.

### 3.2 Row Level Security (RLS)
The Supabase PostgreSQL database enforces strict Row Level Security policies:
1. `officers` table: Authenticated users can view their own profile; admins can query personnel.
2. `devices` table: Enforces registered hardware terminal bindings.
3. `test_records` table:
   - Field Officers can insert new tests and query only records matching their `officer_id`.
   - Administrators possess query rights across all jurisdictional records for chain-of-custody audits.
4. `test_audit_logs` table: Append-only ledger recording record modifications, sync events, and verifications.

### 3.3 Evidence Storage Protection
- Evidence images are stored in a private Supabase Storage bucket (`test-images`).
- Public bucket access is disabled.
- Image retrieval requires authenticated session tokens or short-lived signed URLs.

---

## 4. Statutory Presumptive Result Guardrails

Field testing produces presumptive indicators intended solely for rapid investigative screening and laboratory referral.

### Mandatory System-Wide Notice
> **"Presumptive field-test result — laboratory confirmation required."**

- This statutory disclaimer is permanently displayed across:
  - Android Result Screen
  - Android Test Detail & History Screens
  - Generated QR Verification Modals
  - Web Admin Dashboard KPI Banners
  - Web Evidence Registry and Detail Views
  - QR Public Verification Portal
