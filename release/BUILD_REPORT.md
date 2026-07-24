# YALA RC1 Release AAB Build Report

**Date:** 2026-07-24  
**Target:** Google Play Internal Testing (RC1)  
**Stamp:** `20260724-122925`  
**Verdict:** **READY** (with non-blocking config notes below)

## Summary

Signed release AABs for Rider, Driver, and Delivery were produced successfully using local Gradle 8.2.1 (`bundleRelease --offline`) with `GRADLE_USER_HOME=C:\Users\Housseinou\.gradle`. All three AABs passed `jarsigner -verify`.

## Artifacts

| App | Package name | Version | AAB location | Build status | jarsigner |
|-----|--------------|---------|--------------|--------------|-----------|
| Yala Rider | `com.yala.rider.mr` | 1.2.10 (22) | `release\android\yala-rider-1.2.10-22-20260724-122925.aab` | SUCCESS | verified |
| Yala Driver | `com.yala.driver.mr` | 1.2.27 (42) | `release\android\yala-driver-1.2.27-42-20260724-122925.aab` | SUCCESS | verified |
| Yala Delivery | `com.yala.delivery.mr` | 1.0.7 (9) | `release\android\yala-delivery-1.0.7-9-20260724-122925.aab` | SUCCESS | verified |

Absolute paths:

- `C:\Users\Housseinou\Projects\Django\taxi-booking\release\android\yala-rider-1.2.10-22-20260724-122925.aab`
- `C:\Users\Housseinou\Projects\Django\taxi-booking\release\android\yala-driver-1.2.27-42-20260724-122925.aab`
- `C:\Users\Housseinou\Projects\Django\taxi-booking\release\android\yala-delivery-1.0.7-9-20260724-122925.aab`

## Production config check

| Check | Status | Notes |
|-------|--------|-------|
| Rider API | OK | `REACT_APP_API_URL=https://www.yalataxi.live` |
| Driver API | OK | `REACT_APP_API_URL=https://api.yalataxi.live` |
| Delivery API | OK | `REACT_APP_API_URL=https://www.yalataxi.live` |
| `google-services.json` | OK | Present in all three `android/app/` trees (packages include rider/driver/delivery) |
| Background geolocation | OK | `@capacitor-community/background-geolocation@1.2.26` present |
| Maps browser key | NOTE | `frontend/.env.production` still has placeholder `your_restricted_google_maps_browser_key` (web). Native manifests do not embed a Maps meta-data key in the checked AndroidManifest files. |

## Build environment notes (for future agent/CI runs)

1. **Cursor sandbox `GRADLE_USER_HOME`:** Agent shells may set `GRADLE_USER_HOME` to an empty sandbox cache under `%LOCALAPPDATA%\Temp\cursor-sandbox-cache\...\gradle`. That makes `--offline` report “No cached version” even when `~\.gradle\caches` has AGP. Fix: force `GRADLE_USER_HOME=C:\Users\Housseinou\.gradle` before Gradle.
2. **Wrapper SSL / PKIX:** Online fetches to `services.gradle.org` / Maven still fail PKIX on this machine. Prefer local `gradle.bat` + `--offline` with the real Gradle home.
3. **JAVA_HOME:** `C:\Program Files\Android\Android Studio\jbr`
4. **CI:** `CI=false` (CRA treats ESLint warnings as errors when `CI=true`)

## Remaining blockers

| Item | Severity | Impact on Internal Testing upload |
|------|----------|-----------------------------------|
| Maps key placeholder in `.env.production` | Low / config debt | Does not block AAB upload; may affect web Maps if that env is used |
| Online Gradle dependency fetch (PKIX) | Medium (env) | Does not block these offline builds; blocks clean online rebuilds without local cache |

No signing or packaging blockers remain for the three AABs above.

## Play Console upload

| App | Version | Version code | Track | Status | Imported (UTC) |
|-----|---------|--------------|-------|--------|----------------|
| Yala Rider | 1.2.10 | 22 | Internal testing | Active / accessible to testers | 2026-07-24 17:13 |
| Yala Driver | 1.2.27 | 42 | Internal testing | Active / accessible to testers | 2026-07-24 11:11 |
| Yala Delivery | 1.0.7 | 9 | Internal testing | Active / accessible to testers | 2026-07-24 11:09 |

## Final verdict

**PUBLISHED** to Google Play Internal Testing for Rider, Driver, and Delivery.
