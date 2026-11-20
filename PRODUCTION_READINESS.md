# ScamShield AI production-readiness report

## Scope

This report records the upgrades completed after the audit baseline
`96d7837` / `audit-baseline-20261120`.

## Phase 1 execution

The following checks were run on 2026-11-20 before device testing:

- Expo configuration validation: passed.
- Mobile TypeScript validation: passed.
- Website production build: passed.
- Backend regression tests: 6 passed.
- EAS CLI availability: confirmed (`27.11.0`).
- Git checkpoint: `phase1-predevice-20261120`.

Physical Android execution could not be performed in this environment:

- `adb` is unavailable.
- No Android device is connected.
- No APK has been installed on a physical device.
- Startup, memory, battery, cold model-load, inference, and airplane-mode
  measurements are therefore **not measured**.

The real-device test procedure remains in the Run instructions section. These
results must be collected on the target phone before device behavior can be
marked complete.

## Upgraded successfully

- Bundled the trained English TF-IDF/Logistic Regression parameters into the
  Android app at `mobile/assets/message_model.mobile.json`.
- Added a model manifest with version and SHA-256 values.
- Added canonical model integrity verification before mobile inference.
- Added offline on-device message inference when the backend is unavailable.
- Preserved local URL structural analysis when offline.
- Added model source and inference timing to mobile results.
- Replaced the training input with the UCI SMS Spam Collection.
- Added dataset attribution and CC BY 4.0 licensing documentation.
- Added duplicate removal and stratified train/validation/test evaluation.
- Added evaluation metrics, confusion matrix, and test-error output.
- Added SecureStore-backed biometric app lock.
- Hid mobile message previews by default.
- Preserved local history deletion, export/import, QR confirmation, and manual
  scanning.
- Documented supported Android SMS and call-screening boundaries.
- Added a stable Android package identifier and EAS preview/production build
  profiles for APK/AAB generation.
- Encrypted mobile data exports with a device-kept key while retaining
  backwards-compatible import of older local JSON exports.
- Added application-level encryption for mobile SQLite scan inputs and
  stored result payloads. Encryption keys are protected in Expo SecureStore;
  on Android this uses device-supported secure storage backed by Android
  security facilities where available.
- Added a secure-delete SQLite pragma and migration for legacy plaintext
  previews to encrypted ciphertext fields.
- Added individual and complete scan-history deletion.
- QR URLs are analyzed locally without automatic opening; URL opening requires
  a separate confirmation.
- Camera permission denial and permanent denial now show manual URL fallback.
- Added OP-SQLite and SQLCipher configuration for whole-database encryption.
- Added SecureStore-protected SQLCipher key generation and missing-key recovery
  messaging.
- Added safe migration from `scamshield-local.db` to
  `scamshield-history.db`.
- Added explicit reset for corrupted or missing encrypted databases.

## Model evaluation

Dataset:

- UCI SMS Spam Collection
- 5,574 source messages
- 5,159 unique records after normalized duplicate removal
- Legitimate: 4,517
- Likely scam: 642
- License: CC BY 4.0

Split:

- Training: 2,579
- Validation: 1,290
- Test: 1,290

Held-out test results:

| Class | Precision | Recall | F1 |
| --- | ---: | ---: | ---: |
| Legitimate | 0.9911 | 0.9885 | 0.9898 |
| Likely scam | 0.9207 | 0.9379 | 0.9292 |
| Macro average | 0.9559 | 0.9632 | 0.9595 |

Overall test accuracy: **0.9822**.

Confusion matrix, ordered as `Legitimate`, `Likely scam`:

```text
[[1116, 13],
 [  10, 151]]
```

These results are an English SMS research benchmark, not production accuracy.
The dataset does not represent every scam type, language, region, or current
attack pattern.

Full metrics and test errors are in
`backend/artifacts/evaluation_metrics.json`.

## Offline testing

Verified:

- Bundled model canonical SHA-256 matched the mobile manifest.
- Real offline inference calculation produced a model classification.
- Bundled artifact size: 2,309,747 bytes.
- Desktop bundle load measurement: 12.039 ms.
- Desktop inference measurement: 3.017 ms.
- Desktop heap increase during the probe: approximately 3.7 MB.
- Backend health and message/URL API smoke tests passed.
- Mobile TypeScript validation passed.
- Website production build passed.
- Mobile network state is displayed and local-only/private-API routing is
  implemented.
- QR URL extraction does not open links automatically and uses the existing
  local URL analyzer.

Not yet verified:

- Performance on a physical Android device.
- Android airplane-mode test on physical hardware.
- Native development-build startup.
- Memory behavior on low-end Android devices.

Desktop measurements must not be presented as Android performance claims.

## Android permissions

Currently requested or configured:

- Camera permission only for user-initiated QR scanning.
- SecureStore and local SQLite capabilities.

Not requested:

- `READ_SMS`
- `RECEIVE_SMS`
- Call recording permissions
- Call-log permissions
- `CallScreeningService`

Direct SMS monitoring is intentionally not implemented. Android default-handler
and Play policy restrictions make silent SMS monitoring inappropriate for this
application. Call screening would require a native development build,
`CallScreeningService`, the user-selected call-screening role, and explicit
system authorization.

## Privacy verification

- No external AI provider is used.
- Backend URL analysis never visits submitted URLs.
- Backend history hides content previews by default.
- Mobile history remains in local SQLite.
- Mobile message previews are hidden by default.
- Optional biometric app lock uses Expo SecureStore and device authentication.
- QR destinations require user confirmation before opening.
- No private message content is intentionally written to application logs.

Remaining privacy limitations:

- This implementation encrypts individual scan inputs and result payloads.
  It is not a whole-database SQLCipher or other page-encrypted SQLite
  implementation; SQLite metadata and non-sensitive fields remain visible.
- Database encryption has not been verified after physical-device restart,
  key loss, or corrupted-database recovery.
- Encrypted exports are device-keyed and cannot be restored on a different
  device without a deliberate key-transfer design.
- The backend development API uses HTTP unless deployed behind TLS.
- Authentication remains local UI/access protection rather than account
  authentication.

## Known incomplete items

- Whole-database SQLCipher encryption is implemented in source and native
  configuration, but its Android native build and device restart/migration
  behavior are not yet verified. Native Gradle build attempts were blocked by
  an environment JVM out-of-memory crash.
- ONNX Runtime or TensorFlow Lite native acceleration is not yet integrated.
  The current offline path is a bundled JavaScript implementation of the
  trained baseline.
- Hindi and Telugu model training is not complete.
- A fail-closed Hindi/Telugu dataset validation and training pipeline is now
  prepared under `backend/multilingual`. No unverified or synthetic dataset is
  included, and both languages remain unavailable until separately evaluated
  artifacts are approved.
- Native SMS share intents are not implemented.
- Native call screening is not implemented.
- No production identity/authentication service exists.
- No physical-device performance certification exists.
- No production APK has been generated in this environment.

The EAS configuration is now prepared, but generating and installing the
artifact still requires an authenticated Expo account and an Android build
environment or EAS service. Physical-device performance, airplane-mode
verification, and native Android service behavior remain unverified here.

## Run instructions

### Website

```powershell
cd "C:\Users\Laxmi prasad\OneDrive\Desktop\security_system\website"
npm install
npm run dev
```

Open `http://127.0.0.1:5173`.

### Backend

```powershell
cd "C:\Users\Laxmi prasad\OneDrive\Desktop\security_system\backend"
python -m pip install -r requirements.txt
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

### Android application

```powershell
cd "C:\Users\Laxmi prasad\OneDrive\Desktop\security_system\mobile"
npm install
npm start
```

For a physical device, set `EXPO_PUBLIC_API_URL` to the computer's trusted LAN
address. Offline message and URL scans do not require the backend.

## APK generation

Native capabilities require an Expo development build:

```powershell
cd "C:\Users\Laxmi prasad\OneDrive\Desktop\security_system\mobile"
npx expo install expo-dev-client
npx expo run:android
```

For a distributable preview APK, configure and review an EAS profile:

```powershell
npx eas-cli build:configure
npx eas-cli build --platform android --profile preview
```

No APK build was claimed or uploaded during this phase.
