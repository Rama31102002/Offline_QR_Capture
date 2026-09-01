# Offline QR Capture PWA

A fully local, offline-first Progressive Web App for mobile field capture. After the app is loaded and installed, normal operation does not require a backend, PHP, MySQL, cloud account, remote API, or CDN.

## What it does

- Local username/password login using PBKDF2 password verifiers in IndexedDB.
- Initial local Administrator setup.
- Admin can create users, reset passwords, activate/deactivate accounts and view all local records.
- Rear-camera image capture.
- Captured photograph remains visible until the user presses **Retry** or **OK**. Tapping the photograph itself does not reopen the camera.
- Offline detection of exactly two QR codes from the captured image using the browser's built-in `BarcodeDetector` QR detector.
- GPS latitude, longitude and accuracy capture.
- Automatic user ID, date, time, timezone and UUID stamping.
- Compressed JPEG images stored locally as `Blob` values in IndexedDB.
- User-separated record lists, gallery/list view, QR/date search and filtering.
- ZIP backup/export using a locally bundled copy of JSZip.
- Full Admin backup import/restore with duplicate UUID protection.
- Installable PWA shell cached by a service worker.

## Important compatibility note: QR detection

This build intentionally has **zero runtime network dependencies**. It uses the browser's native `BarcodeDetector` API for QR recognition. On supported browsers this can detect multiple QR codes in a single captured image while completely offline. Current Chromium-based Android browsers are the recommended target.

Before field rollout, test the exact phone/browser model you will use by turning on airplane mode and capturing a photograph that contains two QR codes. If the browser reports that `BarcodeDetector` or QR detection is unsupported, use a supported Chromium-based browser/device or replace `assets/qr-reader.js` with a locally bundled JavaScript QR decoder that supports multi-code detection.

## Installation on normal hosting

The PWA must initially be served from **HTTPS** so camera, geolocation and service workers are available.

1. Upload the whole `Offline_QR_Capture` folder to your hosting, for example:

   `public_html/Offline_QR_Capture/`

2. Open:

   `https://YOUR-DOMAIN/Offline_QR_Capture/`

3. Complete **Initial Setup** and create the first Administrator.

4. Use the browser's **Install App** / **Add to Home Screen** option.

5. Open the installed app once and verify the dashboard loads.

6. Turn off Wi-Fi and mobile data and repeat the full workflow.

No database import is required. No `config.php` is required.

## First use

When the local IndexedDB contains no users, the app shows **Initial Setup**. Create the first Administrator with:

- Administrator User ID
- Name
- Username
- Password (minimum 8 characters)

The Administrator can then open **Admin** and create normal users.

## Where data is stored

All application data is stored in the browser/PWA's IndexedDB database:

`OfflineQRCaptureDB`

Stores:

- `users`
- `captures`
- `settings`

Captured images are stored as JPEG `Blob`s in the `captures` store. They are not placed in `localStorage`.

## Device-local security

Because this is a completely offline application, the phone/device is the main security boundary. Passwords are not stored as plaintext; the app stores a PBKDF2-derived verifier and a random salt. However, a technically skilled person with complete access to the device/browser profile may be able to modify local browser data.

Use:

- phone screen lock
- device encryption
- restricted physical access
- regular local backups

## Backup is essential

There is no server copy. If the phone is lost, reset, damaged, or browser storage is cleared, unexported records can be lost.

Users can choose **Export My Backup**. Administrators can choose **Export New Records**.

Full backups contain `backup.json` plus images under `images/`. Administrator incremental exports can also contain local user password verifiers, so store backup ZIP files securely.

Admin can use ****. Existing record UUIDs are skipped instead of duplicated.

## GPS

GPS is requested only while completing a scan. The default is **GPS Required = Yes**. Admin can make GPS optional under Admin → Settings.

GPS does not require mobile data to calculate coordinates on devices with usable satellite/location services, but real-world location acquisition speed and accuracy can vary by device/environment.

## Camera flow

1. New Scan
2. Camera opens
3. Capture
4. Camera stops
5. Static image preview
6. Retry or OK
7. OK → detect exactly two QR codes
8. Review decoded QR values
9. Continue → GPS
10. Save to IndexedDB

The image preview has no click/touch handler that reopens the camera.

## QR rules

A normal record is saved only after exactly two unique QR values are detected. Zero, one, or more than two detections cause the app to ask for a retake.

## Storage

The dashboard shows approximate browser storage usage when `navigator.storage.estimate()` is supported. Browser storage quotas vary by device and operating system. Export backups regularly, especially before device maintenance or browser data cleanup.

## PWA updates

The service worker uses a versioned cache named `offline-qr-pwa-v1`. When changing deployed files, increment the cache version in `service-worker.js` so clients receive the updated application shell. Updating the cache does not delete IndexedDB records.

## Fully offline acceptance test

After installation:

1. Create a test user while online or during local setup.
2. Install the PWA.
3. Turn on airplane mode.
4. Open the installed PWA.
5. Login.
6. New Scan.
7. Capture a photo containing exactly two QR codes.
8. Verify preview remains visible.
9. Verify tapping the preview does nothing.
10. Press OK and verify both QR values are detected.
11. Continue and capture GPS.
12. Save.
13. Close the PWA.
14. Reopen it while still offline.
15. Login and confirm the record and image still exist.
16. Export a backup ZIP.

## Included third-party library

`assets/jszip.min.js` is JSZip 3.10.1, bundled locally for offline ZIP creation/restoration. JSZip is distributed under its upstream open-source license. No third-party library is loaded from a CDN at runtime.
