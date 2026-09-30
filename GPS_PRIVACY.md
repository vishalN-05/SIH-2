# GPS Privacy & Location Governance Architecture

## 1. Executive Summary
Location metadata associated with field drug testing contains sensitive operational intelligence and officer movement data. The **Digital Companion for Field Drug Testing** enforces a dual-tier location privacy model that balances forensic chain-of-custody requirements against officer safety, data minimization, and ethical law enforcement standards.

---

## 2. Dual-Tier Location Privacy Model

The platform enforces strict separation between operational field access and administrative forensic audits:

| Dimension | Field Officer Role (`OFFICER`) | Forensic Audit Role (`ADMIN`) |
|---|---|---|
| **Exact Coordinates** | **REDACTED** (`[REDACTED — Admin Access Required]`) | **VISIBLE** (e.g. `28.6139° N, 77.2090° E`) |
| **Jurisdictional District** | Visible (e.g. `Central Delhi`) | Visible (e.g. `Central Delhi`) |
| **Mapping Action** | Disabled | Direct satellite link (Google Maps / OpenStreetMap) |
| **QR Code Token** | Excluded | Excluded |
| **Purpose** | Field screening verification | Forensic court submission & chain-of-custody audit |

---

## 3. Android Location Capture Pipeline

### 3.1 Permissions & Privacy Declarations
The Android application requests standard Android location permissions in `AndroidManifest.xml`:
- `android.permission.ACCESS_FINE_LOCATION`
- `android.permission.ACCESS_COARSE_LOCATION`

### 3.2 Pure Android LocationProvider (Zero Play Services Dependency)
- The implementation in `AndroidLocationProvider.kt` relies directly on Android's native `android.location.LocationManager`.
- **Zero Google Play Services dependency**: Enables completely autonomous offline operation in tactical environments, remote field areas, and custom secure government Android ROMs without Google Play Services.

### 3.3 Graceful Degradation & Non-Blocking Workflow
Location acquisition is designed with fail-safe defaults:
1. **Permission Denied**: If an officer declines location permissions, the app logs the refusal without interrupting or blocking the field test workflow.
2. **GPS Unavailable / Timeout**: If satellite or network location fix is unavailable (e.g., inside basements or rural dead zones), `latitude` and `longitude` are recorded as `null`.
3. **Record Integrity Intact**: The cryptographic record seal is computed whether GPS coordinates are present or absent, ensuring evidence capture is never halted.

---

## 4. Web Dashboard & Aggregate Activity Privacy

### 4.1 District-Level Jurisdictional Aggregation
- The Web Dashboard aggregates field screening activity strictly at the administrative district or city municipal level (e.g., *Central Delhi*, *Gautam Buddha Nagar*, *Mumbai Suburban*).
- No exact coordinates or pinpoint markers are exposed on dashboard overview maps.

### 4.2 Ethical Compliance (Zero "Drug Hotspot" Labeling)
- The system **strictly avoids discriminatory or sensationalized labels** such as *"drug hotspots"*, *"crime clusters"*, or *"high-risk zones"*.
- All data visualizations and tables are neutrally labeled as:
  > **"Field Activity by Administrative Jurisdiction"**
- This prevents community stigmatization, aligns with constitutional privacy protections, and ensures ethical public sector data governance.

---

## 5. QR Code Exclusion Guarantee
Under no circumstances are latitude, longitude, altitude, or GPS provider strings encoded into evidence QR codes. This prevents eavesdropping or unauthorized location extraction if a physical evidence label is scanned by an unauthorized third party.
