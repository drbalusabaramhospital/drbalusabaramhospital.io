# Sabaram Hospital — Patient Queue Management System

A web-based patient queue management system for Sabaram Hospital. Handles walk-in and pre-booked patients, real-time queue board, consultation workflow, prescription capture, billing, and backup/recovery — all without a server or build step.

**Live site:** https://drbalusabaramhospital.github.io/clinic/  
**Stack:** Vanilla JS ES modules · Firebase Firestore + Auth · GitHub Pages

---

## Documentation

| Document | Description |
|---|---|
| [docs/INSTALL.md](docs/INSTALL.md) | Full installation and deployment guide — Firebase setup, indexes, GitHub Pages, first admin account |
| [docs/SETUP.md](docs/SETUP.md) | First-time configuration — clinic settings, adding doctors, linking staff logins, go-live checklist |
| [docs/FIRESTORE_SCHEMA.md](docs/FIRESTORE_SCHEMA.md) | Firestore collections, field reference, indexes, and security rules |

---

## Pages

| Page | Login required | Role | Purpose |
|---|---|---|---|
| `index.html` | — | All | Sign in — redirects to correct page by role |
| `admin.html` | Yes | Admin | Configure doctors, staff logins, voice, branding, UPI |
| `reception.html` | Yes | Reception | Register patients, issue tokens, vitals, billing, Rx |
| `doctor.html` | Yes | Doctor | Live queue, call / consult / complete, patient history |
| `board.html` | **No** | Public | Waiting-room TV display — queue, ETAs, voice announcements |
| `self-service.html` | **No** | Public | Patient self-booking kiosk via QR code |
| `patients.html` | Yes | All staff | Patient directory — search, edit, print card |
| `billing.html` | Yes | All staff | Fee reports — date range, doctor, and patient filters |
| `backup.html` | Yes | Admin | Export / restore all clinic data as JSON |
| `patient-card.html` | Yes | All staff | Printable patient card with QR code |

---

## Quick Start

See **[docs/INSTALL.md](docs/INSTALL.md)** for the full guide. Short version:

1. Fork this repo to your GitHub account
2. Create a Firebase project, enable Firestore and Email/Password Auth
3. Replace the config values in `js/firebase-config.js` with your project's values
4. Paste `firestore.rules` into Firebase Console → Firestore → Rules and publish
5. Create the required Firestore indexes (see [docs/INSTALL.md](docs/INSTALL.md) § 7)
6. Enable GitHub Pages (Settings → Pages → Source: GitHub Actions)
7. Push to `main` — GitHub Actions deploys automatically
8. Create your first admin user in Firebase Auth, then manually add their `users` document in Firestore
9. Sign in at your Pages URL and complete setup via Admin Setup

---

## Running Locally

No build step needed. Serve the folder with any static file server:

```bash
# Using npx serve (recommended)
npx serve sabaramhospital

# Or Python
python3 -m http.server 8080 --directory sabaramhospital
```

Then open `http://localhost:8080`. Opening HTML files directly via `file://` will not work because ES modules are blocked by the browser in that context.

> **Note:** When running locally, the app connects to the real Firebase project. Use test accounts — do not use real patient data for local development.

---

## Key Features

- **Multi-doctor queue** — patients select preferred doctor(s), system calculates ETA per doctor
- **ACT (Average Consultation Time)** — rolling 25-visit average with Tukey IQR outlier removal, auto-adjusts ETAs as the day progresses
- **Fixed-slot booking** — reception or self-service can book a specific time slot; queue board shows urgency badges
- **Voice announcements** — multi-language TTS on the queue board; configurable templates with `{token}`, `{room}`, `{doctor}` variables
- **Prescription capture** — camera or file upload, JPEG compression, multi-page per visit
- **UPI QR payment** — full-screen QR overlay for cashless fee collection
- **Backup & restore** — full JSON export/import with pre-restore safety snapshot
- **PWA** — installable on Android/iOS, service worker caching

---

## Firebase Security Rules

Rules are in [`firestore.rules`](firestore.rules). Key principle:

- `doctors`, `visits`, `clinicSettings`, `counters` — **public read** (queue board needs this without login)
- `visits` — **public create** allowed (self-service kiosk submits token bookings without login)
- `counters` — **public write** allowed (token-number transaction must run from self-service kiosk)
- `patients`, `users`, `backups` — **auth required** for all access
- All **update / delete** operations require authentication

Paste the contents of `firestore.rules` into Firebase Console → Firestore Database → Rules and click **Publish**.

---

## Repository Structure

```
sabaramhospital/
├── index.html, admin.html, reception.html, ...  ← App pages
├── css/styles.css                                ← Single stylesheet
├── js/
│   ├── firebase-config.js     ← Firebase init + re-exports
│   ├── queue-logic.js         ← ACT, ETA, Patient ID (shared)
│   ├── qrcode-lib.js          ← Self-contained QR encoder (no CDN)
│   ├── barcode-scanner.js     ← Camera QR/barcode via BarcodeDetector API
│   └── theme.js               ← 5-theme switcher
├── assets/                    ← Icons, manifest images
├── firestore.rules            ← Firestore security rules
├── manifest.json              ← PWA manifest
├── sw.js                      ← Service worker (network-first cache)
├── docs/
│   ├── INSTALL.md             ← Installation guide
│   ├── SETUP.md               ← First-time configuration guide
│   └── FIRESTORE_SCHEMA.md   ← Firestore data schema reference
└── .github/workflows/
    └── deploy.yml             ← GitHub Actions → GitHub Pages
```
