# YALA Android Signing Inventory

| Field | Value |
|-------|--------|
| **Date** | 2026-07-24 |
| **Scope** | YALA Rider, YALA Driver, YALA Delivery AAB signing |
| **Source** | `keytool -list -v` on configured keystores + `signing/` artifacts |
| **Build script** | `scripts/build-release-aabs.ps1` |

---

## 1. Configured release keystores

These are the keystores wired through `signing/credentials.env` into each app’s `build.gradle` release signing config.

### YALA Rider

| Field | Value |
|-------|--------|
| **File** | `C:\Users\Housseinou\Projects\Django\taxi-booking\signing\yala-rider-upload.jks` |
| **Alias** | `yala-rider-upload` |
| **Owner (CN)** | `CN=Yala Rider Upload, OU=Mobile, O=Yala Taxi MR, L=Nouakchott, ST=Nouakchott, C=MR` |
| **SHA-1** | `82:67:AF:26:56:1B:BF:8A:00:94:85:61:21:C1:E8:84:6B:22:F7:4E` |
| **SHA-256** | `AC:A1:F0:65:F6:A3:AB:35:7E:F4:6E:E2:92:71:32:A9:1F:2C:D6:16:6A:48:A5:D1:43:BA:B6:7F:46:7C:B7:16` |

### YALA Driver

| Field | Value |
|-------|--------|
| **File** | `C:\Users\Housseinou\Projects\Django\taxi-booking\signing\yala-driver-upload.jks` |
| **Alias** | `yala-driver-upload` |
| **Owner (CN)** | `CN=Yala Driver Upload, OU=Mobile, O=Yala Taxi MR, L=Nouakchott, ST=Nouakchott, C=MR` |
| **SHA-1** | `18:AB:BF:3F:AD:8B:95:B3:4A:0A:96:4D:2D:F3:1D:D7:31:4A:94:1E` |
| **SHA-256** | `7F:E1:F0:AB:9C:88:A8:F1:F5:7D:E2:6D:76:4B:39:0F:80:88:0B:7E:3E:F5:47:88:78:03:F2:1A:CE:85:34:0F` |

### YALA Delivery

| Field | Value |
|-------|--------|
| **File** | `C:\Users\Housseinou\Projects\Django\taxi-booking\yala-delivery-upload-key.jks` |
| **Alias** | `yala_delivery_upload` |
| **Owner (CN)** | `CN=Yala Delivery, OU=Mobile, O=Yala Technologies, L=Nouakchott, ST=Nouakchott, C=MR` |
| **SHA-1** | `63:3C:BA:83:A8:7E:FA:D4:8C:B6:AC:A3:EB:AD:65:7F:13:7A:4E:D9` |
| **SHA-256** | `33:53:7E:F5:2C:A8:BD:3E:44:90:F0:5D:E6:FD:67:F6:BA:8A:66:A7:9A:05:5F:8F:AC:36:EF:E1:31:4B:28:4E` |

---

## 2. Play Console expected upload certificate

`upload_certificate.pem` (repo root) is the public upload key Play Console currently expects for Rider and Driver.

| Field | Value |
|-------|--------|
| **File** | `C:\Users\Housseinou\Projects\Django\taxi-booking\upload_certificate.pem` |
| **Owner** | `CN=Yala Technologies, OU=Mobile, O=Yala Technologies, L=Nouakchott, ST=Nouakchott, C=MR` |
| **SHA-1** | `92:B7:04:8F:ED:04:24:89:52:F5:EC:56:7D:89:6B:AE:23:AC:C6:38` |
| **SHA-256** | `2C:24:6D:5F:F2:FC:21:A1:8D:43:9F:7F:AA:93:47:62:1F:F5:DE:0A:BE:50:46:EA:BE:62:94:24:6C:28:56:B1` |

**Note:** Delivery Play Console expected upload key should be cross-checked against `signing/yala-delivery-original-upload.pem`. The current Delivery keystore (`yala-delivery-upload-key.jks`) has SHA-1 `63:3C:BA:83...` and matches the on-disk original delivery upload certificate.

---

## 3. Match matrix

| App | Current configured upload key SHA-1 | Play Console expected SHA-1 | Match |
|-----|------------------------------------:|------------------------------:|:-----:|
| YALA Rider | `82:67:AF:26...` | `92:B7:04:8F...` | **NO** |
| YALA Driver | `18:AB:BF:3F...` | `92:B7:04:8F...` | **NO** |
| YALA Delivery | `63:3C:BA:83...` | `63:3C:BA:83...` *(assumes original delivery cert)* | **YES** *(if Play Console still has the original delivery upload cert)* |

### Legacy / locked keystores

Root-level keystores (`yala-upload-key.jks`, `yala-release.jks`, `yala-release.keystore`) could not be unlocked with the project-known password. The original private upload key for Rider/Driver matching `92:B7:04:8F...` is therefore not usable for builds from this repo.

---

## 4. Blockers for Play upload

### P0 — Rider and Driver upload key mismatch

Rider and Driver AABs built with the configured `signing/yala-*-upload.jks` keys will be rejected by Play Console because the upload key SHA-1 does not match the currently registered `upload_certificate.pem` (`92:B7:04:8F...`).

### P1 — Delivery key path confirmation

Delivery is using `yala-delivery-upload-key.jks` at the repo root. Verify that this is the path configured in `signing/credentials.env` (`YALA_DELIVERY_KEYSTORE`) and that Play Console still expects the `63:3C:BA:83...` certificate (`signing/yala-delivery-original-upload.pem`). If the Console was rotated to a different cert, Delivery will also be rejected.

---

## 5. Recommended next actions

1. **Rider / Driver** — Reset the upload key in Google Play Console for both apps to the new configured keys:
   - Extract PEM certificates:
     ```powershell
     keytool -export -rfc -keystore signing\yala-rider-upload.jks -alias yala-rider-upload -file yala-rider-upload-new.pem
     keytool -export -rfc -keystore signing\yala-driver-upload.jks -alias yala-driver-upload -file yala-driver-upload-new.pem
     ```
   - In Play Console: **App → Setup → App integrity → Reset upload key** and upload each new `.pem`.

2. **Delivery** — Confirm the expected upload certificate. If it is still the original `63:3C:BA:83...` cert, make sure `signing/credentials.env` points to `yala-delivery-upload-key.jks` (root) so the AAB is signed with the matching key. If Play Console was reset to a newer cert, follow the same PEM reset flow as Rider/Driver.

3. **Do not delete locked legacy keystores** (`yala-upload-key.jks`, `yala-release.jks`, `yala-release.keystore`) until the new keys are accepted and at least one release is live.

4. **Rebuild RC1** after `signing/credentials.env` and/or Play Console are aligned, bump version codes (Rider 22→23, Driver 42→43, Delivery 9→10), verify `jarsigner`, then upload to Internal Testing.

---

*End of Android Signing Inventory*
