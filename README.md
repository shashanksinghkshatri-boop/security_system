# ScamShield AI

ScamShield AI is a privacy-first scam detection prototype for suspicious messages, URLs, and QR destinations.

## Current functional status

| Capability | Status |
| --- | --- |
| Website message scanning | Functional through local FastAPI |
| Website URL scanning | Functional through local FastAPI |
| Message model | English UCI-based development baseline |
| URL heuristics | Functional; no automatic URL visits |
| Risk score and explanations | Functional |
| Website scan history | Functional through local SQLite backend |
| Mobile local history | Functional through Expo SQLite |
| Mobile history encryption | Sensitive inputs and result payloads encrypted with a SecureStore-protected key |
| Mobile secure preference | Functional through Expo SecureStore |
| Mobile data export/import | Functional with device-keyed encrypted exports |
| Mobile QR scanning | Functional with camera permission and confirmation before opening |
| Offline URL fallback | Functional with local structural heuristics |
| Offline message inference | Functional with bundled, hash-verified TF-IDF/Logistic Regression parameters |
| Hindi/Telugu model inference | Detected and marked unsupported; model training is incomplete |
| Direct SMS monitoring | Not implemented intentionally |
| Share-to-scan | Requires native Android intent integration |
| Call screening | Requires native `CallScreeningService` and development build |
| Authentication | UI prototype only |
| Cloud synchronization | Not implemented |

Hindi and Telugu pipeline preparation is available under `backend/multilingual`.
No Hindi or Telugu dataset is included and no accuracy is claimed. A verified
dataset must include source, license, permitted-use, non-synthetic, and SHA-256
metadata before `backend/train_multilingual.py` will accept it. Until models
are trained and evaluated, those languages remain unavailable rather than being
classified by the English model.

## Run the website

```powershell
cd "C:\Users\Laxmi prasad\OneDrive\Desktop\security_system\website"
npm install
npm run dev
```

Open `http://127.0.0.1:5173`.

## Run the backend

Use the project dependencies in a supported Python environment:

```powershell
cd "C:\Users\Laxmi prasad\OneDrive\Desktop\security_system\backend"
python -m pip install -r requirements.txt
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Verify:

```powershell
curl.exe http://127.0.0.1:8000/api/health
```

The backend defaults to hiding message and URL previews in history. Configure local development with `backend/.env.example`.

## Run the Expo app

```powershell
cd "C:\Users\Laxmi prasad\OneDrive\Desktop\security_system\mobile"
npm install
npm start
```

Expo Go can display the UI and QR flow where supported. Native camera, SQLite, SecureStore, and future SMS/call capabilities should be tested with an Expo development build.

## Physical Android device

The phone cannot use `127.0.0.1` to reach a backend running on the computer. Set the computer's LAN address before starting Expo:

```powershell
$env:EXPO_PUBLIC_API_URL="http://192.168.1.25:8000"
npm start
```

Replace the address with the computer's local IPv4 address. The phone and computer must be on the same trusted network, and the backend must bind to the LAN interface if required:

```powershell
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Do not expose the development API to the public internet.

## Development build and APK

Native integrations are not guaranteed in Expo Go. Use a development build for camera, SQLite, SecureStore, and future Android services:

```powershell
npx expo install expo-dev-client
npx expo run:android
```

For a distributable APK, configure an Expo application identity and use an approved EAS build profile:

```powershell
npx eas-cli build:configure
npx eas-cli build --platform android --profile preview
```

The app has a stable Android package identifier and a checked-in `mobile/eas.json`
preview profile that produces an installable APK. Authenticate with an Expo
account before running the build and review the generated configuration. No
private scan contents should be uploaded as build inputs.

## Offline testing

1. Start the mobile app with the backend available.
2. Scan a URL and confirm the result is saved to local history.
3. Stop the backend or disable network access.
4. Scan a URL again and confirm the local structural fallback is clearly labeled.
5. Scan a message and confirm the bundled model runs locally and shows the model version and inference time.
6. Open Settings and export/import local data.
7. Delete local history and confirm it is empty after restarting the app.

## Tests

Backend regression tests:

```powershell
cd "C:\Users\Laxmi prasad\OneDrive\Desktop\security_system\backend"
python -m unittest discover -s tests -v
```

Prepare a verified language model after adding its approved dataset:

```powershell
python train_multilingual.py --language hindi --dataset-dir data\multilingual\hindi
python train_multilingual.py --language telugu --dataset-dir data\multilingual\telugu
```

These commands fail closed when metadata, licensing evidence, checksums, or
language-specific records are missing.

Mobile type-check:

```powershell
cd "C:\Users\Laxmi prasad\OneDrive\Desktop\security_system\mobile"
npx tsc --noEmit
```

## Privacy boundaries

- Private data is not sent to external AI providers.
- Backend history hides input previews by default.
- Mobile history is stored locally.
- Mobile exports are encrypted with a device-kept key and are intended for
  import on the same device.
- QR links are never opened automatically.
- SMS monitoring and call recording are not implemented.
- Detection is an indicator, not proof that content is safe or malicious.
- The bundled mobile message model is an English development baseline and is verified against a packaged SHA-256 manifest before inference.
