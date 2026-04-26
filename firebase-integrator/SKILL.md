---
name: firebase-integrator
description: >
  Add Firebase to Rockson's standalone HTML/CSS/JS web apps — Firestore, Storage, Hosting,
  and Auth. Generates boilerplate, config, security rules, and CLI deploy commands.
  Use this skill whenever the user wants to add Firebase, connect a database, add file upload,
  deploy a web app, add login/auth, or says anything like "add Firebase", "connect Firestore",
  "deploy to Firebase", "add login", "add file upload to the app", "host the app".
---

# Firebase Integrator

Rockson builds standalone HTML/CSS/JS web apps (no build tools, no npm). Firebase is added via CDN script tags. He uses Firebase CLI in the terminal for deploy and project setup. He has deployed via terminal before so CLI commands are familiar.

**Default context:** Small internal team of ~8 people. Apps are mostly for himself and the team.
**Client-sharing context:** When sharing with clients, Auth is required (Google + email/password).

Never ask about Firebase internals, SDK versions, or security theory — just ask what's needed and generate the right config and rules.

---

## Step 1: Ask three questions

Before generating anything, ask:

1. **Which services does this app need?**
   - Firestore (database)
   - Storage (file / image upload)
   - Hosting (deploy to Firebase)
   - Auth (login) — needed?

2. **Is this for internal team use, or will clients log in?**
   - Internal → public rules, no auth required
   - Client-facing → auth required (Google + email/password login)

3. **Does the app already have a Firebase project set up?** (yes / no / not sure)
   - If no or unsure: walk through Firebase CLI setup first

---

## Firebase CLI Setup (run once per project)

If no Firebase project exists yet, guide through this in order:

```bash
# 1. Install Firebase CLI (run once ever on this machine)
npm install -g firebase-tools

# 2. Log in
firebase login

# 3. Go to your project folder
cd /path/to/your/app

# 4. Initialize Firebase (interactive — select services when prompted)
firebase init

# When prompted:
# - Select: Firestore, Hosting, Storage (and Functions only if needed)
# - Use an existing project OR create a new one at console.firebase.google.com
# - Hosting public directory: . (a single dot — the app root)
# - Single-page app: No (unless the app uses JS routing)
```

After init, these files are created — do not delete them:
- `firebase.json` — deploy config
- `.firebaserc` — project ID
- `firestore.rules` — security rules
- `storage.rules` — storage security rules
- `firestore.indexes.json` — Firestore indexes

---

## Firebase SDK (CDN — no npm needed)

Add to `<head>` in `index.html` — include only what the app uses:

```html
<!-- Firebase App (always required) -->
<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.0/firebase-app.js";
  import { getFirestore } from "https://www.gstatic.com/firebasejs/11.0.0/firebase-firestore.js";
  import { getStorage } from "https://www.gstatic.com/firebasejs/11.0.0/firebase-storage.js";
  import { getAuth } from "https://www.gstatic.com/firebasejs/11.0.0/firebase-auth.js";

  // Firebase config — get this from Firebase Console → Project Settings → Your Apps
  const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT.appspot.com",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
  };

  const app = initializeApp(firebaseConfig);
  const db = getFirestore(app);
  const storage = getStorage(app);
  const auth = getAuth(app);

  // Make available globally
  window.db = db;
  window.storage = storage;
  window.auth = auth;
</script>
```

Tell Rockson: "Replace the values in `firebaseConfig` with the ones from Firebase Console → Project Settings → Your apps → SDK setup and configuration."

---

## Firestore Patterns

### Read a collection

```js
import { collection, getDocs, query, orderBy } from "https://www.gstatic.com/firebasejs/11.0.0/firebase-firestore.js";

async function loadItems() {
  const q = query(collection(db, "items"), orderBy("createdAt", "desc"));
  const snapshot = await getDocs(q);
  snapshot.forEach(doc => {
    console.log(doc.id, doc.data());
  });
}
```

### Add a document

```js
import { collection, addDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.0.0/firebase-firestore.js";

async function addItem(data) {
  await addDoc(collection(db, "items"), {
    ...data,
    createdAt: serverTimestamp(),
  });
}
```

### Update a document

```js
import { doc, updateDoc } from "https://www.gstatic.com/firebasejs/11.0.0/firebase-firestore.js";

async function updateItem(id, changes) {
  await updateDoc(doc(db, "items", id), changes);
}
```

### Delete a document

```js
import { doc, deleteDoc } from "https://www.gstatic.com/firebasejs/11.0.0/firebase-firestore.js";

async function deleteItem(id) {
  await deleteDoc(doc(db, "items", id));
}
```

### Real-time listener (live updates)

```js
import { collection, onSnapshot, query, orderBy } from "https://www.gstatic.com/firebasejs/11.0.0/firebase-firestore.js";

const q = query(collection(db, "items"), orderBy("createdAt", "desc"));
const unsubscribe = onSnapshot(q, (snapshot) => {
  snapshot.forEach(doc => {
    // Update UI here — called every time data changes
  });
});

// Stop listening when done:
// unsubscribe();
```

---

## Storage: File / Image Upload

```js
import { ref, uploadBytes, getDownloadURL } from "https://www.gstatic.com/firebasejs/11.0.0/firebase-storage.js";

async function uploadFile(file, folder = "uploads") {
  const fileRef = ref(storage, `${folder}/${Date.now()}_${file.name}`);
  const snapshot = await uploadBytes(fileRef, file);
  const downloadURL = await getDownloadURL(snapshot.ref);
  return downloadURL;  // Save this URL to Firestore to reference the file
}

// Example: hook to a file input
document.getElementById('fileInput').addEventListener('change', async (e) => {
  const file = e.target.files[0];
  if (!file) return;
  const url = await uploadFile(file, "images");
  console.log("Uploaded:", url);
  // Save url to Firestore document as needed
});
```

HTML for file input:
```html
<input type="file" id="fileInput" accept="image/*">
```

---

## Auth: Login

Only add Auth when the app is client-facing. For internal team use, skip Auth entirely.

### Setup: Enable providers in Firebase Console
Go to: **Firebase Console → Authentication → Sign-in method**
- Enable **Google**
- Enable **Email/Password**

### Auth boilerplate

```js
import {
  getAuth,
  signInWithPopup,
  GoogleAuthProvider,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  signOut,
  onAuthStateChanged
} from "https://www.gstatic.com/firebasejs/11.0.0/firebase-auth.js";

const provider = new GoogleAuthProvider();

// Google login
async function loginWithGoogle() {
  await signInWithPopup(auth, provider);
}

// Email/password login
async function loginWithEmail(email, password) {
  await signInWithEmailAndPassword(auth, email, password);
}

// Email/password signup
async function signUpWithEmail(email, password) {
  await createUserWithEmailAndPassword(auth, email, password);
}

// Logout
async function logout() {
  await signOut(auth);
}

// Watch login state — runs on every page load
onAuthStateChanged(auth, (user) => {
  if (user) {
    // User is logged in — show app
    console.log("Logged in as:", user.email);
    document.getElementById('app').style.display = 'block';
    document.getElementById('login-screen').style.display = 'none';
  } else {
    // Not logged in — show login screen
    document.getElementById('app').style.display = 'none';
    document.getElementById('login-screen').style.display = 'block';
  }
});
```

---

## Security Rules

### Firestore Rules

**Internal team use (default) — public read/write:**
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

**Client-facing — authenticated users only:**
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

### Storage Rules

**Internal team use — public:**
```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write: if true;
    }
  }
}
```

**Client-facing — authenticated users only:**
```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

Write the appropriate rules to `firestore.rules` and `storage.rules`. Always tell Rockson which ruleset was used and why.

---

## Deploy to Firebase Hosting

```bash
# Deploy everything (hosting + rules)
firebase deploy

# Deploy hosting only (the web app files)
firebase deploy --only hosting

# Deploy rules only (Firestore + Storage rules)
firebase deploy --only firestore:rules,storage

# Preview before deploying (creates a temporary preview URL)
firebase hosting:channel:deploy preview --expires 1h
```

After deploy, Firebase prints the live URL. Tell Rockson to share that URL with the team or client.

**`firebase.json` config for a single-page app at root:**
```json
{
  "hosting": {
    "public": ".",
    "ignore": [
      "firebase.json",
      ".firebaserc",
      "*.rules",
      "*.md",
      "node_modules"
    ]
  },
  "firestore": {
    "rules": "firestore.rules"
  },
  "storage": {
    "rules": "storage.rules"
  }
}
```

---

## Notes for Claude

- Rockson uses plain HTML/CSS/JS — always use CDN `<script type="module">` imports, never npm packages.
- He knows how to run terminal commands but not Firebase internals — explain rules choices briefly.
- Default to public rules unless the app is client-facing; always state clearly which rules were applied.
- After generating any Firebase code, remind him to replace the `firebaseConfig` values from Firebase Console.
- Auth is only needed for client-facing apps. Never add Auth complexity to internal tools.
- Storage files should use a timestamp prefix (`Date.now()_filename`) to avoid name collisions.
