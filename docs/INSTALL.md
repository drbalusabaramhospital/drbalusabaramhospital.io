# Installation & Deployment Guide

Sabaram Hospital Patient Queue Management System

---

## 1. Prerequisites

No server, no Node.js, no build tools required. Everything runs in the browser.

| Requirement | Notes |
|---|---|
| Chrome or Edge (latest) | Firefox works but voice announcements may differ |
| GitHub account | Free — for hosting the code |
| Firebase account | Free tier (Spark plan) is sufficient for most clinics |
| A domain / URL | Provided automatically by GitHub Pages |

---

## 2. Fork or Clone the Repository

**Option A — Fork (recommended for your own clinic)**

1. Go to https://github.com/drbalusabaramhospital/clinic
2. Click **Fork** (top-right).
3. Choose your GitHub account as the owner.
4. Keep the repository name or rename it (e.g. `clinic`).

**Option B — Clone locally**

```bash
git clone https://github.com/drbalusabaramhospital/clinic.git
cd clinic
```

> **Note:** The app is pure static HTML/JS — there is no `npm install` or build step. You can open files directly or deploy them as-is.

---

## 3. Create a Firebase Project

1. Go to https://console.firebase.google.com and sign in.
2. Click **Add project**.
3. Enter a project name (e.g. `sabaramhospital-67090`).
4. Disable Google Analytics if you do not need it, then click **Create project**.
5. Wait for provisioning, then click **Continue**.

### 3a. Enable Firestore

1. In the Firebase console, click **Build → Firestore Database**.
2. Click **Create database**.
3. Choose **Start in production mode** (you will paste the correct rules in step 6).
4. Select the region closest to your clinic (e.g. `asia-south1` for India).
5. Click **Enable**.

### 3b. Enable Authentication

1. Click **Build → Authentication**.
2. Click **Get started**.
3. Under **Sign-in method**, click **Email/Password**.
4. Toggle **Email/Password** to **Enabled**.
5. Click **Save**.

---

## 4. Update `js/firebase-config.js`

After creating your Firebase project, get your web app config:

1. In Firebase console, click the **gear icon → Project settings**.
2. Scroll to **Your apps** and click **Add app → Web** (`</>`).
3. Register the app (name it anything, e.g. `clinic-web`).
4. Copy the `firebaseConfig` object shown.

Open `js/firebase-config.js` in your repo and replace the values:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.firebasestorage.app",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

> **Note:** These values are safe to commit to a public GitHub repo. Firebase Security Rules (not the API key) control who can read or write data.

---

## 5. Enable Email/Password Authentication

Already covered in step 3b. Confirm the provider shows **Enabled** in Firebase console → Authentication → Sign-in method.

---

## 6. Paste Firestore Security Rules

1. In Firebase console, go to **Firestore Database → Rules**.
2. Replace everything in the editor with the rules below.
3. Click **Publish**.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /doctors/{id} {
      allow read: if true;
      allow write: if request.auth != null;
    }
    match /visits/{id} {
      allow read: if true;
      allow create: if true;           // self-service kiosk creates visits without login
      allow update, delete: if request.auth != null;
    }
    match /clinicSettings/{id} {
      allow read: if true;
      allow write: if request.auth != null;
    }
    match /counters/{id} {
      allow read: if true;
      allow write: if true;            // token counter transaction must work from self-service kiosk
    }
    match /patients/{id} {
      allow read, write: if request.auth != null;
    }
    match /users/{id} {
      allow read, write: if request.auth != null;
    }
    match /backups/{id} {
      allow read, write: if request.auth != null;
    }
  }
}
```

> **Note:** `doctors`, `visits`, `clinicSettings`, and `counters` allow public read so the queue board works without login. `visits` also allows public `create` and `counters` allows public `write` so the self-service kiosk can issue tokens without login. Staff operations (update, delete, patient access) always require authentication.

---

## 7. Create Required Firestore Indexes

The app uses compound queries that Firestore cannot serve without explicit indexes. Create each one manually.

**How to create an index:**
1. Go to Firebase console → **Firestore Database → Indexes**.
2. Click **Add index**.
3. Enter the collection name and fields exactly as listed below.
4. Set field order (Ascending/Descending) as specified.
5. Click **Create**.

| # | Collection | Field 1 | Field 2 | Field 3 |
|---|---|---|---|---|
| 1 | `visits` | `doctorId` ASC | `status` ASC | `consultEndAt` DESC |
| 2 | `visits` | `visitDate` ASC | `status` ASC | — |
| 3 | `visits` | `visitDate` ASC | `tokenNumber` ASC | — |
| 4 | `visits` | `patientId` ASC | `issuedAt` DESC | — |
| 5 | `visits` | `visitDate` ASC | `feeCollected` ASC | — |

> **Note:** Index creation takes 1–5 minutes. The app will show Firestore errors for queries involving these fields until indexes are ready. You can also create indexes by clicking the direct link that appears in browser console errors.

---

## 8. Enable GitHub Pages

1. In your GitHub repo, go to **Settings → Pages**.
2. Under **Source**, select **GitHub Actions**.
3. Click **Save**.

The repo already contains `.github/workflows/deploy.yml`, which deploys the site on every push to `main`.

---

## 9. Deploy — Push to Main

After updating `js/firebase-config.js` with your project values, commit and push:

```bash
git add js/firebase-config.js
git commit -m "Configure Firebase project"
git push origin main
```

GitHub Actions will run automatically. Monitor progress under the **Actions** tab in your repo. Deployment typically takes under 2 minutes.

Your site will be live at:

```
https://<your-github-username>.github.io/<repo-name>/
```

---

## 10. Create the First Admin User

This must be done manually — there is no self-registration UI.

### Step 1 — Create the Auth account

1. Go to Firebase console → **Authentication → Users**.
2. Click **Add user**.
3. Enter an email and a strong password for the admin.
4. Click **Add user**.
5. Copy the **User UID** shown in the user list (it looks like `abc123XYZ...`).

### Step 2 — Create the Firestore user document

1. Go to Firebase console → **Firestore Database → Data**.
2. Click **Start collection**, enter collection ID: `users`, click **Next**.
3. In the **Document ID** field, paste the UID you copied.
4. Add the following fields:

| Field | Type | Value |
|---|---|---|
| `displayName` | string | e.g. `Admin User` |
| `role` | string | `admin` |
| `email` | string | the email you entered |

5. Click **Save**.

### Step 3 — Sign in

Go to your deployed URL (or `index.html` locally), sign in with the admin credentials. You will be redirected to `admin.html`.

---

## 11. Verify the Deployment

Check each of these URLs after deployment:

| URL | Expected result |
|---|---|
| `https://<your-site>/` | Sign-in page loads |
| `https://<your-site>/board.html` | Queue board loads (no login) |
| `https://<your-site>/self-service.html` | Self-service kiosk loads (no login) |
| Firebase console → Firestore → Rules | Rules show `Published` status |
| Firebase console → Authentication → Users | Admin user appears |
| GitHub repo → Actions tab | Latest workflow run shows green checkmark |

> **Note:** If `board.html` shows a Firebase error, check that the Firestore indexes are finished building and the security rules are published correctly.
