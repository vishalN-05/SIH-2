# QR Verification & Tamper-Evident Evidence Architecture

## 1. Executive Summary
The **Digital Companion for Field Drug Testing** generates tamper-evident QR verification tokens for physical evidence bags, officer notebooks, and forensic chain-of-custody transfer slips. The system guarantees complete zero-leakage privacy while providing instant cryptographic tamper detection.

---

## 2. Minimal Zero-Leakage Payload Specification

To eliminate privacy and operational risks associated with barcode leakage, QR codes encode exclusively three minimal fields in standard JSON:

```json
{
  "v": "1.0",
  "id": "DFT-20260930-982145A1",
  "h": "75fb2480c5e0c3737441670aa603862c725708d0d9cefa368b8f7903a19baeb5"
}
```

### 2.1 Payload Fields
| Key | Field | Type | Description |
|---|---|---|---|
| `v` | Version | String | Schema specification version (`"1.0"`) |
| `id` | Test Identifier | String | Canonical field test identifier (e.g. `DFT-20260930-...`) |
| `h` | Record Hash | String (64 hex) | Deterministic SHA-256 seal computed over immutable record facts |

### 2.2 Strict Information Exclusion Guarantees
The QR code generator **strictly prohibits** encoding any of the following sensitive fields:
- ❌ **NO GPS Coordinates**: Latitude and longitude are never embedded in the QR matrix.
- ❌ **NO Authentication Secrets**: Passwords, API keys, and session JWTs are strictly excluded.
- ❌ **NO Private Storage URLs**: Supabase image URLs and internal file paths are omitted.
- ❌ **NO Personal Identifying Information**: Officer badge numbers and names are not stored in the QR payload.

This ensures that even if an evidence tag or paper label is photographed by unauthorized parties, zero operational or location data is compromised.

---

## 3. Cryptographic Verification & Tamper Detection

### 3.1 Verification Flow
When an evidence QR code is scanned in the Web Portal or on an authorized Companion terminal:
1. **JSON Parsing**: The raw QR content is parsed to extract `id` and `h`.
2. **Record Retrieval**: The local Room database or Supabase post-sync registry is queried for the test record matching `id`.
3. **Canonical Reconstruction**: The verifier reconstructs the canonical string using the immutable facts stored in the authoritative database:
   ```
   canonical = testId | kitId | result | formattedConfidence | timestamp | safeImageHash
   ```
4. **Digest Computation**: The verifier computes the SHA-256 digest using standard Web Crypto or JVM `MessageDigest`.
5. **Comparison**:
   - If `computedHash.toLowerCase() == h.toLowerCase()` &rarr; **`VALID`** (Cryptographic Record Authenticated).
   - If `computedHash != h` &rarr; **`INVALID`** (TAMPER DETECTED: Record facts have been modified since generation).

### 3.2 Unit Test Verification
The Android project includes exhaustive automated test coverage in `RecordIntegrityAndQrTest.kt`:
- Deterministic hash consistency across invocations.
- Tamper detection when result is altered (e.g., `NEGATIVE` to `POSITIVE`).
- Tamper detection when model confidence is modified.
- Strict payload privacy checks verifying zero presence of `latitude`, `longitude`, `password`, or `token`.

---

## 4. Mandatory Presumptive Result Notice

Whenever a QR code is scanned or rendered for verification, the interface must prominently display the statutory disclaimer:

> **"Presumptive field-test result — laboratory confirmation required."**

This ensures that receiving forensic intake officers and legal counsel are immediately notified of the presumptive nature of the field test.
