# ERT300 String Tension Converter

A mobile web app that converts ERT300 Dynamic Tension (DT) readings to kg/lbs, and tracks string tension history across multiple rackets. Works as a home screen app on iPhone via Safari.

---

## Features

- DT → kg/lbs conversion calibrated directly from ERT300 disc values (DT 26–53)
- Supports MID (83–94 in²), MID+ (95–105 in²), and OVER (106–115 in²) head sizes
- Playing style indicator (Lo / Mid / Hi / Hi+) with out-of-range warnings
- Racket database with string tension history and % tension loss tracking
- **Guest mode** — data saved locally in the browser (no login required)
- **Cloud sync** — sign in with Google to sync rackets across all devices via Firebase

---

## Deploying to GitHub Pages

### Step 1 — Create a GitHub repository

1. Go to [github.com](https://github.com) and sign in
2. Click **+** → **New repository**
3. Name it `ert300` (or anything you like)
4. Set visibility to **Public** (required for free GitHub Pages)
5. Click **Create repository**

### Step 2 — Upload the file

1. On the repository page click **Add file → Upload files**
2. Drag in `index.html` (and `README.md` if you want)
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

The app will open full-screen like a native app. Rackets are saved locally in Safari's storage.

---

## Setting up Firebase (cloud sync across devices)

Firebase is optional. Without it the app works fine with local storage. With it, your rackets sync instantly across all your devices when signed in with Google.

### Step 1 — Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project**
3. Give it a name (e.g. `ert300`)
4. You can disable Google Analytics if you don't need it
5. Click **Create project**

### Step 2 — Register a web app

1. On the project overview page click the **`</>`** (Web) icon
2. Give it a nickname (e.g. `ert300-web`)
3. Click **Register app**
4. You'll see a `firebaseConfig` object — copy these values, you'll need them in Step 6

### Step 3 — Enable Google Sign-in

1. In the left sidebar go to **Build → Authentication**
2. Click **Get started** if prompted
3. Click the **Sign-in method** tab
4. Click **Google** → toggle **Enable** → click **Save**

### Step 4 — Create a Firestore database

1. In the left sidebar go to **Build → Firestore Database**
2. Click **Create database**
3. Choose **Start in production mode**
4. Select a region (e.g. `us-east1`) → click **Done**

### Step 5 — Set Firestore security rules

1. In Firestore, click the **Rules** tab
2. Replace all existing content with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/rackets/{racketId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

3. Click **Publish**

These rules ensure each user can only access their own racket data.

### Step 6 — Add your GitHub Pages domain to authorized domains

1. In **Authentication**, click the **Settings** tab
2. Scroll down to **Authorized domains**
3. Click **Add domain**
4. Enter `yourusername.github.io` (your actual GitHub username)
5. Click **Add**

### Step 7 — Add Firebase config to index.html

1. Open `index.html` in a text editor
2. Find the `FIREBASE_CONFIG` block near the bottom of the file:

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

3. Replace each `"YOUR_..."` value with the corresponding value from your Firebase project
4. Save the file

### Step 8 — Commit and deploy

1. Go to your GitHub repository
2. Click on `index.html` → click the pencil (edit) icon
3. Paste the updated file contents → click **Commit changes**
4. GitHub Pages will redeploy automatically in ~1 minute

### Step 9 — Test sign-in

1. Open your app URL in Safari
2. Tap **Sign in with Google**
3. Complete Google sign-in
4. The banner in My Rackets should turn green and show **Synced**

---

## Note on the Firebase API key

The Firebase API key is visible in the HTML source. This is expected and safe — Firebase API keys identify your project but do not grant access to your data. Your data is protected by the Firestore security rules set up in Step 5. GitHub may flag it as a "secret scanning alert" — this can be safely dismissed as a false positive.

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
- Cloud Firestore (database)
- localStorage (offline/guest fallback)
