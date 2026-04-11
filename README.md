# ERT300 String Tension Converter

A mobile web app for tennis players and stringers. Converts ERT300 Dynamic Tension (DT) readings to kg/lbs, tracks string tension over time per racket, and maintains a historical log of stringing jobs. Works as a home screen app on iPhone via Safari.

---

## Features

- **DT → kg/lbs conversion** calibrated directly from ERT300 disc values (DT 26–53)
- Supports MID (83–94 in²), MID+ (95–105 in²), and OVER (106–115 in²) head sizes
- Playing style indicator (Lo / Mid / Hi / Hi+) with out-of-range warnings
- **My Rackets** — equipment registry for your physical frames with DT tracking and % tension loss
- **Stringing Jobs** — historical log of every stringing job, linked to rackets in your registry
- **Single source of truth** — stringing jobs drive the baseline DT in My Rackets automatically; no double entry
- **DT vs tension chart** — scatter plot with trend line to predict DT targets for future stringing jobs
- **Guest mode** — all data saved locally in the browser (no login required)
- **Cloud sync** — sign in with Google to sync all data across devices via Firebase

---

## How the data model works

**My Rackets** is your equipment registry. Each entry represents a physical racket frame you own (e.g. R1, R2). You define the name, racket model, and head size. No stringing data is entered here.

**Stringing Jobs** is where all stringing data lives — racket model, string, machine tension, baseline DT, date, and notes. Each job can be assigned to a racket in your registry via a dropdown. When assigned:
- The racket in My Rackets automatically shows the most recent assigned job as its current baseline
- The machine tension, string, and date on the racket card come from the job
- Editing the job updates the racket card automatically

**Follow-up DT readings** are added directly on the racket card in My Rackets. The baseline (from the stringing job) is read-only and shown with a "baseline · from job" badge. Follow-up readings can be edited or deleted.

**When a new stringing job is assigned to a racket**, the racket's follow-up reading history is cleared automatically since those readings belonged to the previous string.

**Unassigned jobs** (e.g. strung for someone else) are logged normally and appear in the chart but are not linked to any racket in your registry.

---

## Deploying to GitHub Pages

### Step 1 — Create a GitHub repository

1. Go to [github.com](https://github.com) and sign in
2. Click **+** → **New repository**
3. Name it `ert300` (or anything you like)
4. Set visibility to **Public** (required for free GitHub Pages)
5. Click **Create repository**

### Step 2 — Upload the files

1. On the repository page click **Add file → Upload files**
2. Drag in `index.html` and `README.md`
3. Click **Commit changes**

### Step 3 — Enable GitHub Pages

1. Go to **Settings** → **Pages** (left sidebar)
2. Under **Source** select **Deploy from a branch**
3. Set branch to `main`, folder to `/ (root)`
4. Click **Save**
5. After ~1 minute your app is live at `https://yourusername.github.io/ert300`

### Step 4 — Add to iPhone home screen

1. Open the URL in **Safari** on your iPhone
2. Tap the **Share** button (box with arrow)
3. Tap **Add to Home Screen**
4. Name it `ERT300` → tap **Add**

The app opens full-screen like a native app. Data is saved locally in Safari's storage until you sign in with Google.

---

## Setting up Firebase (cloud sync across devices)

Firebase is optional. Without it the app works fine with local storage. With it, all data syncs instantly across all your devices when signed in with Google.

### Step 1 — Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project**, give it a name (e.g. `ert300`)
3. Click **Create project**

### Step 2 — Register a web app

1. On the project overview page click the **`</>`** (Web) icon
2. Give it a nickname (e.g. `ert300-web`) → click **Register app**
3. Copy the `firebaseConfig` values — you'll need them in Step 7

### Step 3 — Enable Google Sign-in

1. Left sidebar → **Build → Authentication** → **Get started**
2. **Sign-in method** tab → **Google** → toggle **Enable** → **Save**

### Step 4 — Create a Firestore database

1. Left sidebar → **Build → Firestore Database** → **Create database**
2. Choose **Start in production mode** → select a region → **Done**

### Step 5 — Set Firestore security rules

1. In Firestore, click the **Rules** tab
2. Replace all existing content with:

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

3. Click **Publish**

The `{collection}` wildcard covers `rackets`, `jobs`, and any future collections with a single rule. Each user can only access their own data.

### Step 6 — Add your GitHub Pages domain

1. **Authentication** → **Settings** tab → **Authorized domains**
2. Click **Add domain** → enter `yourusername.github.io` → **Add**

### Step 7 — Add Firebase config to index.html

1. Open `index.html` in a text editor
2. Find the `FIREBASE_CONFIG` block and replace each `"YOUR_..."` value:

```javascript
const FIREBASE_CONFIG = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT_ID.firebaseapp.com",
  projectId:         "YOUR_PROJECT_ID",
  storageBucket:     "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId:             "YOUR_APP_ID",
  measurementId:     "YOUR_MEASUREMENT_ID"   // optional
};
```

3. Save and commit the file to GitHub — the app redeploys automatically

### Step 8 — Test sign-in

1. Open the app URL in Safari
2. Tap **Sign in with Google** and complete sign-in
3. The banner in My Rackets should turn green and show **Synced**

---

## Note on the Firebase API key

The Firebase API key is visible in the HTML source. This is expected and safe — it identifies your project but does not grant access to data. Your data is protected by the Firestore security rules. GitHub may flag it as a "secret scanning alert" — this can be safely dismissed as a false positive.

---

## DT conversion reference

Conversion table calibrated from ERT300 disc values. Valid range is DT 26–53.

| DT range | Playing style |
|----------|--------------|
| Below 28 | Out of range |
| 28–34    | Lo — defensive baseline play |
| 35–41    | Mid — dynamic and offensive |
| 42–46    | Hi — fast and aggressive |
| 47–56    | Hi+ — very fast and aggressive |
| Above 56 | Out of range |

---

## Tech stack

- Vanilla HTML/CSS/JavaScript — no build tools required
- Firebase Authentication (Google Sign-in)
- Cloud Firestore (rackets + jobs collections)
- localStorage (offline/guest fallback)
