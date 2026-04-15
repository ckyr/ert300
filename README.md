# Tennis Stringing Companion

A responsive web app for tennis players and stringers. Keeps records of your rackets, maintains a historical log of stringing jobs, tracks string tension over time, and converts ERT300 Dynamic Tension (DT) readings to kg/lbs. Works as a home screen app on iPhone via Safari, and as a full responsive web app on tablets and desktops.

---

## Features

- **DT → kg/lbs conversion** calibrated directly from ERT300 disc values (DT 26–53)
- Supports MID (83–94 in²), MID+ (95–105 in²), and OVER (106–115 in²) head sizes
- Playing style indicator (Lo / Mid / Hi / Hi+) with live zone highlight in the converter
- **My Rackets** — equipment registry with DT tracking, % tension loss, colour-coded zones, drag-to-reorder
- **Stringing Jobs** — historical log linked to rackets; single source of truth for all stringing data
- **DT vs tension chart** — scatter plot with trend line, filterable by racket model, exportable as PNG
- **Help tab** — full in-app documentation
- **Settings (⚙)** — full JSON backup/restore, import/export
- **Guest mode** — all data saved locally in the browser (no login required)
- **Cloud sync** — sign in with Google to sync all data across devices via Firebase
- **Fully responsive** — optimised for iPhone, tablet, and desktop

---

## Responsive layout

| Screen width | Layout |
|---|---|
| Below 768px | Mobile — full width, top nav + tab bar, bottom sheet modals |
| 768px – 1289px | Tablet — fluid, sidebar 25% / content 75% |
| 1290px+ | Desktop — fixed 430px sidebar / 860px content, centered |

---

## How the data model works

**My Rackets** is your equipment registry. Each entry represents a physical racket frame (e.g. Racket 1, Racket 2). Define the name, model, and head size. No stringing data is entered here directly.

**Stringing Jobs** is where all stringing data lives. Each job can be assigned to a racket via a dropdown — this auto-fills the model and head size and automatically sets the baseline DT on that racket card. Editing a job updates the racket card in real time.

**Follow-up DT readings** are added on the racket card. The baseline (from the stringing job) is read-only. When a new job is assigned, old follow-up readings are hidden (filtered by date) but not deleted — removing the new job restores the previous state.

**Racket order** is customised by dragging the ⠿ grip handle. Order syncs across devices.

---

## Settings (⚙)

Tap the gear icon next to the sign-in button to access backup and restore options.

### Export data (JSON)
Saves a complete backup of all rackets, stringing jobs, DT history, and sort order to a timestamped JSON file (e.g. `ert300_backup_2026-04-14.json`).

### Import data (JSON) — merge
Restores data from a previously exported JSON file. Merges with existing data — records with matching IDs are updated, new records are added. Existing records not in the backup are left untouched.

> **Note:** If you sign in with Google after importing in guest mode, Firestore cloud data will take precedence over local data. Import is best used in guest mode, or to seed a brand new Google account.

### Restore from JSON — clean
Permanently deletes **all** existing data (locally and from Firestore if signed in), then imports the backup fresh. This is the safest migration path — no conflict possible. **Cannot be undone.** Make sure you have a valid backup before proceeding.

### Delete database
Permanently deletes all rackets, jobs, and DT history — locally and from Firestore if signed in. Useful for clearing test data before real use. **Cannot be undone.**

### Chart export (PNG)
Tap the image icon (🖼) next to the "Show chart" button to save the current chart view as a PNG image. The filename includes the active racket filter if one is set.

---

## Deploying to GitHub Pages

### Step 1 — Create a GitHub repository
1. Go to [github.com](https://github.com) and sign in
2. Click **+** → **New repository**, name it `ert300`, set to **Public**
3. Click **Create repository**

### Step 2 — Upload the files
1. Click **Add file → Upload files**
2. Drag in `index.html` and `README.md` → **Commit changes**

### Step 3 — Enable GitHub Pages
1. **Settings** → **Pages** → Source: **Deploy from a branch**
2. Branch: `main`, folder: `/ (root)` → **Save**
3. After ~1 minute: `https://yourusername.github.io/ert300`

### Step 4 — Add to iPhone home screen
1. Open the URL in **Safari** → tap **Share** → **Add to Home Screen**
2. Name it `ERT300` → tap **Add**

> After updating `index.html` on GitHub, delete and re-add the app from your home screen to pick up changes.

---

## Setting up Firebase (cloud sync)

Firebase is optional. Without it the app uses local browser storage.

### Step 1 — Create a Firebase project
Go to [console.firebase.google.com](https://console.firebase.google.com) → **Add project** → name it `ert300`.

### Step 2 — Register a web app
Project overview → **</>** icon → nickname `ert300-web` → **Register app** → copy the `firebaseConfig` values.

### Step 3 — Enable Google Sign-in
**Build → Authentication** → **Sign-in method** → **Google** → Enable → **Save**.

### Step 4 — Create Firestore database
**Build → Firestore Database** → **Create database** → production mode → choose region → **Done**.

### Step 5 — Set security rules
Firestore → **Rules** tab → replace with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{collection}/{docId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```
→ **Publish**

### Step 6 — Add GitHub Pages domain
**Authentication** → **Settings** → **Authorized domains** → **Add domain** → `yourusername.github.io`

### Step 7 — Add config to index.html
Find the `FIREBASE_CONFIG` block and replace the placeholder values, then commit.

---

## Note on the Firebase API key
The key is visible in the HTML source. This is expected and safe — it identifies your project but does not grant data access. Data is protected by Firestore security rules. GitHub secret scanning alerts for this can be safely dismissed as false positives.

---

## DT conversion reference

| DT range | Zone | Playing style |
|---|---|---|
| Below 28 | — | Out of range |
| 28–34 | Lo | Defensive baseline |
| 35–41 | Mid | Dynamic and offensive |
| 42–46 | Hi | Fast and aggressive |
| 47–56 | Hi+ | Very fast and aggressive |
| Above 56 | — | Out of range |

---

## Tech stack
- Vanilla HTML/CSS/JavaScript — no build tools
- Firebase Authentication (Google Sign-in)
- Cloud Firestore (`rackets` + `jobs` collections)
- localStorage (offline/guest fallback)
