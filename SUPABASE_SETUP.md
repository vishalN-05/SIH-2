# Supabase Database & Storage Setup Guide

**Project:** Digital Companion for Field Drug Testing (Smart India Hackathon)  
**Package:** `com.sih.drugtestcompanion`  
**Migration Script:** [`supabase/migrations/20260929_field_drug_testing_schema.sql`](file:///D:/SIH/supabase/migrations/20260929_field_drug_testing_schema.sql)

---

## 1. How to Run the SQL Migration in Supabase SQL Editor

1. Open your web browser and navigate to the [Supabase Dashboard](https://supabase.com/dashboard).
2. Select your project.
3. In the left navigation sidebar, click on **SQL Editor** (icon `>_`).
4. Click **New query** in the top right.
5. Open the local migration file:
   [`supabase/migrations/20260929_field_drug_testing_schema.sql`](file:///D:/SIH/supabase/migrations/20260929_field_drug_testing_schema.sql)
6. Copy the entire contents of the file and paste them into the SQL Editor.
7. Click the **Run** button (or press `Ctrl+Enter` / `Cmd+Enter`).
8. Verify that the output panel confirms `Success. No rows returned` (or displays successful table creation status).

---

## 2. How to Create & Verify the Private Storage Bucket

The SQL migration automatically configures the storage bucket and its policies. To verify or manually configure it in the Supabase Dashboard:

1. In the Supabase Dashboard sidebar, navigate to **Storage**.
2. Check if a bucket named `test-images` is listed:
   - If present: Click the three dots `...` next to `test-images`, select **Edit bucket**, and ensure **Public bucket** is toggled **OFF** (Strictly Private).
   - If not present: Click **New bucket**, set name to `test-images`, ensure **Public bucket** is **OFF**, and set Allowed MIME types to `image/jpeg, image/png`.
3. Object Path Convention:
   ```
   test-images/{officerId}/{testId}.jpg
   ```
4. Verify Storage Policies under **Storage** > **Policies**:
   - `storage_upload_officer`: Authenticated officers can only upload to their assigned folder path.
   - `storage_read_authorized`: Authenticated officers can read their own images; Admins/Supervisors can read broader records.
   - `storage_delete_authorized`: Deletion restricted to the owning officer or Admin.
   - Unauthenticated (`anon`) access: **Disabled / Denied**.

> [!NOTE]
> Images are never accessible via public permanent URLs. The Android app accesses images using short-lived cryptographically signed URLs generated via `client.storageGetSignedUrl()`.

---

## 3. Required Environment / `local.properties` Values

To configure the Android app to securely connect to your Supabase instance, update [`local.properties`](file:///D:/SIH/local.properties) at the project root:

```properties
## Supabase Configuration (DO NOT COMMIT SECRETS TO VCS)
# Obtain these from: Supabase Dashboard -> Project Settings -> API
SUPABASE_URL=https://<your-project-id>.supabase.co
SUPABASE_ANON_KEY=<your-public-anon-key>
```

> [!CAUTION]
> - Use **ONLY** the public `anon` key.
> - **NEVER** place or bundle the Supabase `service_role` key into the Android application.
> - The Android application strictly detects and rejects any key containing `service_role` at initialization.
> - [`local.properties`](file:///D:/SIH/local.properties) is in [`.gitignore`](file:///D:/SIH/.gitignore) to protect credentials from source control.

---

## 4. How Row Level Security (RLS) Protects Records

Row Level Security is enabled across all sensitive tables: `officers`, `devices`, `test_kits`, `test_records`, and `audit_logs`.

### Security Guarantees:
1. **Unauthenticated (`anon`) Denial:** Direct access to `test_records`, `audit_logs`, `officers`, and `devices` is completely revoked for unauthenticated requests (`REVOKE ALL FROM anon`).
2. **Officer Isolation:** Field officers can only `SELECT`, `INSERT`, and `UPDATE` records matching their authorized officer identity (`officer_id = auth.uid()`).
3. **Supervisor / Admin Broad Access:** Authorized roles (`ADMIN`, `SUPERVISOR`, `LAB_ANALYST`) can query broader records for audit, verification, and analytical oversight.
4. **Append-Only Audit Trail:** `audit_logs` are insert-only for authenticated officers and strictly read-only for Admins. No updates or deletions are permitted.
5. **GPS Coordinates Protection:**
   The database exposes an authorized view `public.authorized_test_records`:
   - Exact latitude and longitude are returned **only** to the creating officer or users with administrative roles.
   - For all other unprivileged queries, coordinates are redacted to `NULL`.

---

## 5. How Android Sync Maps to Supabase

The Android app communicates with Supabase via PostgREST endpoints and the [`SupabaseClient`](file:///D:/SIH/app/src/main/java/com/sih/drugtestcompanion/core/network/SupabaseClient.kt):

| Kotlin Domain / DTO Property | Supabase DB Column | Database Type | Notes |
|---|---|---|---|
| `id` | `id` | `UUID PRIMARY KEY` | Auto-generated via `gen_random_uuid()` |
| `testId` | `test_id` | `TEXT UNIQUE` | Format: `DFT-YYYYMMDD-XXXXXXXX` |
| `officerId` / `operatorId` | `officer_id` | `UUID REFERENCES officers(id)` | Maps friendly tags to officer UUID |
| `deviceId` | `device_id` | `UUID REFERENCES devices(id)` | Maps terminal tag to device UUID |
| `kitId` / `testKitId` | `kit_id` | `UUID REFERENCES test_kits(id)` | Maps kit name/type to test-kit UUID |
| `result` | `result` | `TEXT` | `POSITIVE`, `NEGATIVE`, `INCONCLUSIVE` |
| `confidence` | `confidence` | `DOUBLE PRECISION` | Model prediction confidence (0.0 to 1.0) |
| `timestamp` | `timestamp` | `TIMESTAMPTZ` | Serialized as ISO-8601 string (`Instant`) |
| `latitude` | `latitude` | `DOUBLE PRECISION` | Optional GPS coordinate |
| `longitude` | `longitude` | `DOUBLE PRECISION` | Optional GPS coordinate |
| `imageUrl` | `image_url` | `TEXT` | Path in private `test-images` bucket |
| `imageHash` | `image_hash` | `TEXT` | SHA-256 hash of raw captured image |
| `recordHash` | `record_hash` | `TEXT` | Cryptographic seal of entire test record |
| `signature` | `signature` | `TEXT` | Cryptographic officer digital signature |
| `modelVersion` | `model_version` | `TEXT` | Active ML classifier model version |
| `calibrationStatus` | `calibration_status` | `TEXT` | Reference card calibration state |
| `imageQualityStatus` | `image_quality_status` | `TEXT` | Blur/lighting validation state |
| `syncStatus` | `sync_status` | `TEXT` | `LOCAL_ONLY`, `PENDING_SYNC`, `SYNCED` |
| `createdAt` | `created_at` | `TIMESTAMPTZ` | Auto-generated UTC timestamp |
| `updatedAt` | `updated_at` | `TIMESTAMPTZ` | Managed by trigger `trigger_test_records_updated_at` |

---

## 6. Development Seed Data

The migration includes **only safe development seed entities** to satisfy foreign key constraints:

1. **Development Officer:**
   - **UUID:** `a0000000-0000-0000-0000-000000000001`
   - **Name:** Field Officer Sharma
   - **Employee ID:** `NCB-OFFICER-001`
   - **Role:** `officer`
   - **Status:** `active`

2. **Development Device:**
   - **UUID:** `b0000000-0000-0000-0000-000000000001`
   - **Officer ID:** `a0000000-0000-0000-0000-000000000001`
   - **Device Name:** Field Terminal Alpha
   - **Status:** `active`

3. **Development Test Kit:**
   - **UUID:** `c0000000-0000-0000-0000-000000000001`
   - **Name:** Presumptive Rapid Reagent Screening Kit
   - **Manufacturer:** Standard Field Diagnostics
   - **Test Type:** Colorimetric Field Screening
   - **Model Version:** `v1.0.0`
   - **Status:** `active`

> [!IMPORTANT]
> **Zero fake drug test results or chemistry reactions are created in seed data**, strictly respecting field procedure protocols.
