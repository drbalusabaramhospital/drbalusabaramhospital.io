# First-Time Configuration Guide

Sabaram Hospital Patient Queue Management System

Complete this guide after finishing INSTALL.md. Do not open the app with patients until every step is done.

---

## 1. Overview — What Needs to Be Done Before Going Live

| Task | Where | Required |
|---|---|---|
| Create admin user in Firebase Auth | Firebase console | Yes |
| Create `users` document for admin | Firestore console | Yes |
| Sign in and open admin.html | Browser | Yes |
| Set clinic name and self-service URL | Admin UI | Yes |
| Add at least one doctor | Admin UI | Yes |
| Create staff logins (reception, doctors) | Firebase console + Admin UI | Yes |
| Configure UPI payment (if collecting fees) | Admin UI | Recommended |
| Configure voice announcements | Admin UI | Optional |
| Set up queue board on waiting-room screen | Browser / TV | Recommended |
| Set up self-service kiosk | Browser / tablet | Optional |
| Test a full patient registration cycle | Reception UI | Yes — before going live |

---

## 2. Creating the First Admin User

### Step 1 — Add user in Firebase Auth

1. Open https://console.firebase.google.com → your project.
2. Click **Build → Authentication → Users**.
3. Click **Add user**.
4. Enter the admin's email address and a password.
5. Click **Add user**.
6. The user appears in the list. Click the row to see the **User UID** (a long alphanumeric string). **Copy it.**

### Step 2 — Create the Firestore document

1. Click **Build → Firestore Database → Data**.
2. Click **+ Start collection**.
3. Collection ID: `users` → click **Next**.
4. Document ID: paste the UID you copied (do not use Auto-ID).
5. Add fields:

```
displayName  (string)  →  "Admin User"   (or the person's actual name)
role         (string)  →  "admin"
email        (string)  →  the email address used above
```

6. Click **Save**.

> **Note:** The `role` field must be exactly `"admin"` (lowercase). Any other value will not redirect to `admin.html` on sign-in.

---

## 3. Signing In as Admin for the First Time

1. Open your deployed URL (e.g. `https://drbalusabaramhospital.github.io/clinic/`).
2. Enter the admin email and password.
3. Click **Sign In**.
4. You are redirected to `admin.html`.

If you are redirected to `reception.html` instead, the `users` document was not found or `role` is not `"admin"`. Check Firestore.

---

## 4. Admin Setup Walkthrough

All configuration is done from `admin.html`. Work through each section in order.

### 4a. Clinic Name and Self-Service URL

1. In Admin UI, find the **Clinic Settings** section.
2. Set **Clinic Name** (e.g. `Sabaram Hospital`) — this appears on printouts and the queue board.
3. Set **Self-Service URL** — the full URL of your `self-service.html` page. Example:
   ```
   https://drbalusabaramhospital.github.io/clinic/self-service.html
   ```
   This URL is encoded into the QR code on patient cards so patients can self-book on return visits.
4. Click **Save**.

### 4b. UPI Payment Configuration

Required if the reception desk collects fees digitally.

| Field | Description | Example |
|---|---|---|
| UPI VPA | Your UPI Virtual Payment Address | `sabaram@upi` |
| Merchant Name | Name shown on the patient's UPI app | `Sabaram Hospital` |
| Merchant Code | Optional category code from your bank | `5099` |

> **Note:** The app generates a UPI deep-link/QR for each payment. No payment gateway integration is needed — the patient scans and pays from their UPI app.

### 4c. Adding the First Doctor

1. In Admin UI, go to **Doctors**.
2. Click **Add Doctor**.
3. Fill in all fields:

| Field | Description | Example |
|---|---|---|
| Name | Full name with title | `Dr. Balu Sabaram` |
| Specialization | Shown on queue board | `General Medicine` |
| Consultation Fee | In rupees | `300` |
| Avg. Consultation Time (minutes) | Used to estimate wait times | `8` |
| Room | Shown on queue board and voice announcements | `Room 1` |
| Status | Set to `active` to accept patients | `active` |

4. Click **Save**. The doctor appears in the list and is immediately available in reception.

### 4d. Linking a Doctor's Firebase Auth Account

Each doctor who logs in needs both a Firebase Auth account and a `users` document that links to their doctor profile.

**Step 1 — Create Firebase Auth account for the doctor**

1. Firebase console → Authentication → Add user.
2. Enter the doctor's email and a password.
3. Copy the UID.

**Step 2 — Get the doctor's Firestore document ID**

1. Firebase console → Firestore → `doctors` collection.
2. Find the doctor you added via Admin UI.
3. Copy the **Document ID** shown at the top of that document (e.g. `abc123xyz`).

**Step 3 — Create the `users` document**

1. Firestore → `users` collection → Add document.
2. Document ID = the doctor's Firebase Auth UID.
3. Fields:

```
displayName  (string)  →  "Dr. Balu Sabaram"
role         (string)  →  "doctor"
doctorId     (string)  →  "abc123xyz"   ← the doctors collection document ID
email        (string)  →  "doctor@sabaram.com"
```

4. Save.

> **Note:** The `doctorId` field must exactly match a document ID in the `doctors` collection. If it does not, the doctor's queue page will be empty.

### 4e. Linking a Reception Staff Account

Reception staff need a Firebase Auth account and a `users` document with `role: "reception"`.

**Step 1 — Create Firebase Auth account** (same as above — Firebase console → Authentication → Add user).

**Step 2 — Create the `users` document**

Document ID = the staff member's Firebase Auth UID. Fields:

```
displayName  (string)  →  "Reception Staff Name"
role         (string)  →  "reception"
email        (string)  →  "reception@sabaram.com"
```

No `doctorId` field needed for reception.

### 4f. Voice Announcements

Voice announcements are spoken through the queue board page (`board.html`) when a doctor calls the next patient.

1. In Admin UI, go to **Voice Settings**.
2. Toggle **Enable Voice** on or off.
3. Set **Repeat count** (1–3 times per announcement).
4. Set **Interval** (seconds between repeats; `0` = no auto-repeat after the first).
5. Configure languages. Each language entry has:
   - **Language code** (e.g. `en-IN` for English, `ta-IN` for Tamil, `hi-IN` for Hindi)
   - **Template** — the announcement text with placeholders:
     - `{token}` — the token number
     - `{room}` — the doctor's room
     - `{name}` — the patient's name
     - `{doctor}` — the doctor's name
   - **Enabled** toggle

Example templates:

```
English:  Token number {token}, please proceed to {room}.
Tamil:    டோக்கன் எண் {token}, {room} க்கு வரவும்.
Hindi:    टोकन नंबर {token}, कृपया {room} में आएं।
```

6. Click **Save**.

> **Note:** Voice uses the browser's Web Speech API. The queue board must be open in Chrome/Edge on the display screen for voice to work. It will not speak if the browser tab is hidden or the system volume is muted.

---

## 5. Which Collections Are Created Automatically vs Manually

| Collection | How it is created | Action needed |
|---|---|---|
| `users` | **Manually** in Firestore console | Create one document per staff member |
| `doctors` | Via Admin UI | No console action needed after setup |
| `clinicSettings` | Via Admin UI (on first save) | No console action needed |
| `visits` | Automatically when reception registers a visit | None |
| `patients` | Automatically when reception registers a patient | None |
| `counters` | Automatically on first token/patient registration | None |
| `backups` | Via Backup page | None |

---

## 6. Setting Up the Queue Board on a Waiting-Room Screen

The queue board (`board.html`) requires no login and is designed for a TV or large monitor.

1. On the display device, open Chrome or Edge.
2. Navigate to:
   ```
   https://drbalusabaramhospital.github.io/clinic/board.html
   ```
3. Press **F11** (or use Chrome's **Kiosk mode**) to go full screen.
4. The page auto-refreshes via Firestore real-time listeners — no manual refresh needed.
5. Voice announcements are spoken from this page when a doctor calls a patient.

**Kiosk mode (prevent staff from accidentally closing the tab):**

```
chrome.exe --kiosk https://drbalusabaramhospital.github.io/clinic/board.html
```

Or on a Raspberry Pi / dedicated display device, set the browser to auto-launch in kiosk mode on startup.

---

## 7. Setting Up the Self-Service Kiosk

The self-service kiosk (`self-service.html`) requires no login. Place a tablet at the front desk or waiting area.

1. Navigate to `self-service.html` on the tablet.
2. Patients enter their mobile number to check in or register.
3. Returning patients are recognized by mobile number.
4. The kiosk generates a token and shows it on screen.

**Lock the tablet to the kiosk page:**

- Android: Use **Guided Access** or a kiosk app (e.g. SureLock).
- iPad: Use **Guided Access** (Settings → Accessibility → Guided Access).
- Windows tablet: Use **Assigned Access** (Settings → Accounts → Family & other users → Set up a kiosk).

> **Note:** The self-service URL set in Clinic Settings (step 4a) must match the actual URL of your deployed `self-service.html`. Patient QR codes on printed cards encode this URL.

---

## 8. Backup Schedule Recommendation

The app includes a Backup page (`backup.html`) that exports Firestore data.

| Frequency | When |
|---|---|
| Daily | End of clinic day — export visits and patients |
| Weekly | Full export of all collections |
| Before any code update | Full backup before deploying changes |

Backups are stored as documents in the `backups` Firestore collection and can also be downloaded as JSON files.

---

## 9. Checklist — Ready to Go Live

Work through this list before registering the first real patient.

- [ ] Firebase project created and Firestore enabled
- [ ] Security rules published
- [ ] All 5 Firestore indexes created and status shows **Enabled**
- [ ] `js/firebase-config.js` updated with correct project values
- [ ] GitHub Pages deployed — site URL opens correctly
- [ ] Admin user created in Firebase Auth
- [ ] Admin `users` document created in Firestore with `role: "admin"`
- [ ] Signed in as admin — redirected to `admin.html`
- [ ] Clinic name saved in Admin UI
- [ ] Self-service URL saved in Admin UI
- [ ] At least one doctor added
- [ ] At least one reception staff account created and linked
- [ ] Queue board (`board.html`) opens and displays correctly on waiting-room screen
- [ ] Test: Reception registers a test patient → token appears on queue board
- [ ] Test: Doctor calls the token → token moves to "called" on board
- [ ] Voice announcement works (if enabled) on queue board screen
- [ ] UPI settings saved (if collecting fees)
- [ ] First backup taken
