# SIH 2026 Presentation & Demonstration Checklist

**Project:** Digital Companion for Field Drug Testing  
**Package:** `com.sih.drugtestcompanion`  
**Evaluation Standard:** Smart India Hackathon (SIH 2026) Presumptive Drug Testing Prototype  

Use this checklist during live evaluator demos to systematically demonstrate end-to-end functionality, security controls, and forensic reliability.

---

## 1. Android Application & Launch
- [x] **App Installs & Boots Cleanly:** Application initializes `AppDatabase` and `LocaleHelper` without startup crashes.
- [x] **Officer Authentication:** Login screen allows officer credentials entry and quick-role demonstration (`ADMIN` / `OFFICER`).
- [x] **Language Switching:** In **Settings**, change language between **English**, **தமிழ் (Tamil)**, and **हिन्दी (Hindi)**. UI re-renders immediately with all technical identifiers (`Test ID`, `SHA-256`, `GPS`) preserved.
- [x] **Mandatory Statutory Notice:** Clear banner: *"Presumptive field-test result — laboratory confirmation required"* is visible on all relevant screens.

---

## 2. Test Kit Selection & Camera
- [x] **New Test Initiation:** Tap **New Test** &rarr; Select approved kit (e.g. *Rapid Marquis Field Assay*).
- [x] **Safety Checklist & Warnings:** Clear operational checklist displayed before camera access.
- [x] **Camera Permission Handling:** Runtime camera permission prompt appears; handles permission denial gracefully with an informative explanatory state.
- [x] **CameraX Live Preview:** Framing guides and reference card bounding reticle overlay the live feed.
- [x] **Capture Feedback:** Capturing displays a progress indicator and freezes the captured frame for immediate review.

---

## 3. Pre-Analysis Verification (Quality & Reference Card)
- [x] **Sharpness & Blur Analysis:** OpenCV Laplacian variance evaluates focus. (Simulating a blurred image displays an actionable warning).
- [x] **Lighting & Exposure Gate:** Luminance checks flag underexposed or severely glared frames.
- [x] **Reference Colour Card Detection:** Quadrilateral contour detection locates calibration card boundary.
- [x] **Gated Continue Button:** **Continue** button remains disabled unless image quality is acceptable and the reference card is detected.

---

## 4. On-Device ML Analysis & Fail-Safe Demo Mode
- [x] **Colour Calibration:** Normalizes illuminant cast using reference card gray-world scaling.
- [x] **Tensor Normalization:** Resizes and extracts $224 \times 224$ float32 tensor matching standard Python pipelines.
- [x] **On-Device TFLite Execution:** Runs inference strictly on background coroutine (`Dispatchers.Default`), keeping UI fluid.
- [x] **Safety Fallback Verification:** With no unvalidated or fake model present, result reliably evaluates to **INCONCLUSIVE** with confidence `0.0%`.
- [x] **No Fake Results:** Verifiably demonstrates that the application never hallucinates positive or negative drug results.

---

## 5. Result Staging, Sealing & Room Storage
- [x] **Comprehensive Result Display:** Screen displays Test ID, Test Kit, Presumptive Result, Model Confidence, Model Version, Calibration Status, Image Quality, Timestamp, Sync Status, and Integrity Status.
- [x] **Cryptographic SHA-256 Digest:** Generates SHA-256 hash of the captured image file.
- [x] **Deterministic Record Seal:** Binds immutable fields (`testId|kitId|result|confidence|timestamp|imageHash`) into a 64-character SHA-256 record hash.
- [x] **Offline Room Persistence:** Tap **Save Test Record**; record is written to local SQLite Room database with status `PENDING_SYNC`.
- [x] **Offline Stability:** App operates completely offline without network exceptions or crashes.

---

## 6. Offline-to-Online Cloud Synchronization
- [x] **Pending Sync Counter:** Home Dashboard and History screen display pending upload counter.
- [x] **Background Connectivity Listener:** `ConnectivityNetworkMonitor` detects restoration of internet connection.
- [x] **Sync Execution:** Tap **Sync Now** or reconnect internet; record uploads to Supabase and updates local status to `SYNCED`.
- [x] **Idempotent Ingestion:** Re-running sync checks unique `test_id` to guarantee zero duplicate records in Supabase.
- [x] **Failed Sync Recovery:** If server is unreachable, local records remain safely preserved with status `SYNC_FAILED` for future retry.

---

## 7. History, Custody Audit & QR Verification
- [x] **History Listing:** Lists all local and synced screenings with search by Test ID and filter chips for results and sync states.
- [x] **Empty State Handling:** Clear, friendly empty-state illustration when no records match filter criteria.
- [x] **Test Details Screen:** Shows comprehensive forensic custody trail.
- [x] **GPS Coordinate Privacy:** Officer view displays redacted coordinates (`[Redacted]`); administrator mode unlocks exact coordinates for forensic audit.
- [x] **Zero-Leakage QR Dialog:** Generates verifiable QR code containing strictly version `v`, `id`, and hash `h`. (Verify that no GPS coordinates, auth tokens, passwords, or image URLs are encoded).
- [x] **Tamper Detection Test:** Modifying any factual field in a test record immediately flips the cryptographic integrity status to `INVALID`.

---

## 8. Web Administrative Dashboard
- [x] **Web Build & Startup:** Runs at `http://localhost:3000` via Vite.
- [x] **Role-Based Demo Access:** Quick-switch between `ADMINISTRATOR` (full audit & GPS) and `FIELD OFFICER` (redacted coordinates).
- [x] **Operational KPI Overview:** Total screenings, Presumptive Positives, Presumptive Negatives, Inconclusive tests, and Sync counts.
- [x] **Jurisdictional Aggregation:** District-level safe summaries without leaking sensitive officer whereabouts.
- [x] **QR Verification Portal:** Paste or load QR payloads to validate authenticity against deterministic SHA-256 seals. Demonstrates instant detection of tampered tokens.
- [x] **Language Toggle:** Supports English and தமிழ் on the web portal.

---

## 9. Security Verification Summary
- [x] **No service_role Key:** Verified 0 occurrences of Supabase `service_role` keys in client source code or build configs.
- [x] **Row-Level Security (RLS):** Enabled on all Supabase tables (`test_records`, `officers`, `devices`, `test_kits`).
- [x] **Image Storage Privacy:** Reaction photos are restricted to authenticated uploads in private storage buckets.
- [x] **No Secret Logging:** Logcat never outputs API keys, auth tokens, or passwords.
