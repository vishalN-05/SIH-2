# Digital Companion for Field Drug Testing

> **SIH 2026 Prototype & Evaluation System**  
> **Package:** `com.sih.drugtestcompanion`  
> **Mandatory Statutory Notice:** *"Presumptive field-test result — laboratory confirmation required."*

---

## 1. Project Overview

The **Digital Companion for Field Drug Testing** is an offline-first, mobile and web evidentiary ecosystem engineered for law enforcement field officers and forensic administrators. It standardizes, digitizes, and cryptographically secures field colorimetric drug assays (e.g., Marquis, Cobalt Thiocyanate / Scott, Duquenois-Levine).

The system replaces manual, subjective color interpretation with controlled on-device capture, automatic image quality verification, reference card color calibration, local machine learning evaluation, SHA-256 tamper-proof sealing, zero-leakage QR verification, and automated offline-to-online synchronization to an administrative Supabase portal.

---

## 2. Problem Statement

Field colorimetric spot testing conducted by law enforcement faces critical challenges:
1. **Subjective Interpretation:** Variable ambient lighting, glare, and officer color perception lead to false assumptions or disputed results.
2. **Chain-of-Custody Vulnerabilities:** Manual logs can be contested in judicial proceedings without tamper-evident cryptographic seals.
3. **Connectivity Deficits:** Remote border checkpoints and narcotics search zones often lack reliable cellular coverage.
4. **Data Privacy Risks:** Traditional cloud services risk exposing sensitive officer GPS coordinates and raw narcotics evidence to third-party APIs.

---

## 3. Main Features

- **Offline-First Architecture:** Full capture, image analysis, on-device evaluation, and record sealing work seamlessly without internet connectivity.
- **On-Device Preprocessing & Quality Gate:** Real-time Laplacian sharpness/blur detection, luminance/exposure verification, and quadrilateral reference card detection via OpenCV.
- **Colour Calibration & Tensor Normalization:** Reference card-guided Gray World color balance normalization and $224 \times 224$ float32 tensor preprocessing matching Python deep learning pipelines.
- **Fail-Safe On-Device ML:** On-device TensorFlow Lite runner. When a validated ML model is absent or capture quality is inadequate, the system deterministically outputs `INCONCLUSIVE` (confidence `0.0%`) to prevent fake positives or negatives.
- **Cryptographic Evidentiary Seals:** Every test record generates a SHA-256 image digest bound into a deterministic SHA-256 record hash (`testId|testKitId|result|confidence|timestamp|imageHash`).
- **Zero-Leakage QR Verification:** Encodes strictly version `v`, test identifier `id`, and cryptographic seal `h`. Never leaks GPS coordinates, auth tokens, passwords, or image URLs.
- **Role-Governed GPS Privacy:** Officers see redacting status to safeguard operational privacy; administrators access exact coordinates for forensic custody audits.
- **Idempotent Cloud Synchronization:** Automatic queuing in Room database (`PENDING_SYNC`); synchronizes to Supabase with image uploads when connectivity returns, preventing duplicate rows via unique `test_id`.
- **Multilingual Support:** Fully localized in **English**, **தமிழ் (Tamil)**, and **हिन्दी (Hindi)** with real-time UI switching.
- **Web Administrative Dashboard:** Secure React/TypeScript portal with KPI metrics, jurisdictional analytics, audit logs, and instant QR verification.

---

## 4. Technology Stack

### Android Application
- **Language & Framework:** Kotlin 2.2.10, Jetpack Compose (Material 3), Jetpack Lifecycle
- **Architecture:** Clean Architecture + MVVM + Flow/Coroutines
- **Local Persistence:** Room Database 2.6.1 with KSP code generation
- **Computer Vision:** OpenCV Android SDK 4.5.3.0
- **On-Device ML:** TensorFlow Lite 2.16.1
- **Camera:** CameraX 1.3.4 (Lifecycle, View, Camera2)
- **Barcodes:** ZXing 3.5.3

### Web Dashboard
- **Framework:** React 18, TypeScript, Vite
- **Styling:** Modular CSS with Material 3 Law-Enforcement Dark Palette
- **Icons:** Lucide React
- **QR Engine:** node-qrcode

### Backend & Cloud
- **Platform:** Supabase (PostgreSQL, Row-Level Security, Storage Buckets)
- **Authentication:** Role-Based Access Control (`ADMIN`, `OFFICER`)
- **Key Safety:** Client uses `SUPABASE_ANON_KEY` strictly; `service_role` key is strictly barred from client code.

---

## 5. System Architecture

```
[ Field Officer ]
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│ Android Field Companion (com.sih.drugtestcompanion)          │
│                                                             │
│  1. CameraX Viewfinder & Capture Reticle                   │
│  2. OpenCV Image Quality Analysis (Blur/Illumination)       │
│  3. Reference Card Quadrilateral Polygon Detection          │
│  4. Illuminant Balance Calibration (Gray World)             │
│  5. 224x224 RGB Tensor Normalization                        │
│  6. On-Device TFLite Classifier (Fail-safe Inconclusive)    │
│  7. SHA-256 Image & Deterministic Record Hashing            │
│  8. Local Room Storage (PENDING_SYNC)                       │
│  9. Zero-Leakage QR Generator (v, id, h)                    │
└──────────────────────────────┬──────────────────────────────┘
                               │
            (When Network Available - SyncManager)
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ Supabase Cloud Platform                                     │
│                                                             │
│  - PostgreSQL with Row-Level Security (RLS)                 │
│  - Private Storage Bucket: test-images (Officer Upload)     │
│  - Idempotent Ingestion by UNIQUE test_id                   │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ Web Forensic & Audit Portal (React + Vite)                  │
│                                                             │
│  - Presumptive Screenings & Operational KPI Dashboard       │
│  - Administrator GPS Chain-of-Custody Satellite Mapping     │
│  - QR Cryptographic Evidence Verification Scanner           │
│  - Bi-lingual Interface (English & தமிழ்)                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Project Setup & Installation

### Prerequisites
- **Android Studio** Ladybug / Jellyfish or later
- **JDK:** OpenJDK 17 or 21 (JDK 25 toolchain fallback supported)
- **Android SDK:** Compile SDK 35, Min SDK 26
- **Node.js:** v18.0.0 or later (for Web Dashboard)
- **Gradle:** 9.6.0 (wrapper included)

### Configuration (`local.properties`)
Create `local.properties` in `D:\SIH` if not present:
```properties
sdk.dir=C:\\Users\\<USER>\\AppData\\Local\\Android\\Sdk
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```
*(Never place a `service_role` key in `local.properties` or source code).*

---

## 7. How to Run the Project

### A. Run Android Unit Tests & Build APK
```powershell
cd D:\SIH

# Execute all 64 automated unit tests
.\gradlew.bat testDebugUnitTest

# Assemble debug APK
.\gradlew.bat assembleDebug
```
The compiled APK will be at:
`app\build\outputs\apk\debug\app-debug.apk`

### B. Run Web Dashboard
```powershell
cd D:\SIH\web

# Install dependencies
npm install

# Start Vite development server
npm run dev

# Or build production static assets
npm run build
```
Dashboard runs at `http://localhost:3000`.

---

## 8. Complete Demo Flow

1. **Authentication:**
   - Launch app &rarr; Sign in as Field Officer (`IND-OFFICER-442`).
2. **Select Test Kit:**
   - Tap **New Test** &rarr; Select approved assay protocol (e.g. *Rapid Marquis Field Assay*).
3. **Live Camera Capture:**
   - Frame the reaction chamber with the reference color card visible inside the guide reticle.
   - Tap **Capture**.
4. **Pre-Analysis Verification:**
   - Automatic OpenCV quality analysis verifies sharpness (Laplacian variance), illumination, and reference card alignment.
   - Tap **Continue**.
5. **On-Device Evaluation & Sealing:**
   - Feature extraction and on-device model execution evaluate the reaction.
   - If real validated weights are absent, system deterministically returns **INCONCLUSIVE** (`0.0%` confidence).
   - Generates SHA-256 image digest and deterministic record seal.
   - Tap **Save Test Record** (persisted locally to Room with `PENDING_SYNC`).
6. **Offline-to-Online Sync:**
   - Tap **Sync Now** or enable network; record uploads to Supabase and marks `SYNCED`.
7. **History & QR Inspection:**
   - Open **Test History** &rarr; Select test &rarr; Tap **View Verification QR**.
   - Review zero-leakage payload.
8. **Web Portal Audit:**
   - Open Web Dashboard &rarr; View synchronized test records and audit the cryptographic seal in the QR Verification Portal.

---

## 9. Current Limitations

- **Model Weights Availability:** In compliance with SIH guidelines, fake chemical colors and mock positive results have been eliminated. In the absence of an accredited laboratory-validated `.tflite` model file, all evaluations safely output `INCONCLUSIVE`.
- **OpenCV Native Binaries on Emulators:** Ensure ARM64 or x86_64 target architecture matches the hardware for OpenCV native library loading.
