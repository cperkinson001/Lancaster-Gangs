# Gang Intelligence Database

A mobile-friendly record app for gang unit field work, shared live across every signed-in detective's device. Free to host: the app itself lives on GitHub Pages, and the shared data lives in a free Firebase project (Google Cloud).

**Records include:** full legal name, monikers/AKAs, photos, race, sex, gang affiliation, tattoos (location + description), weapons information, home address, vehicles (plate/make/model/color), police contact history, and free-text notes. Everything is searchable and filterable by any of those fields, plus a combined name/moniker search box.

**How sharing works:** there is no self-service sign-up. An admin (probably you) creates one Firebase account per detective. Everyone signs into the same app with their own email/password, and every record any of them adds, edits, or deletes shows up on everyone else's screen within a second or two — no manual export/import needed day-to-day.

---

## Part 1 — Create the shared database (Firebase)

This takes about 10 minutes, once, and is free for a small team's usage.

1. Go to **console.firebase.google.com** and sign in with any Google account (a personal or department Google account both work).
2. Click **Add project**, give it a name (e.g. `gang-intel-db`), and finish the wizard (you can decline Google Analytics — not needed).
3. In the left sidebar, click **Build → Authentication → Get started**. Under "Sign-in method," enable **Email/Password**, then Save.
4. In the left sidebar, click **Build → Firestore Database → Create database**. Choose a region close to you, and start in **production mode**. Click Enable.
5. Once created, go to the **Rules** tab of Firestore and replace the contents with:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /records/{recordId} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```

   Click **Publish**. This means: only someone signed in can read or write records — nobody else, no exceptions. (Optional tightening: you can restrict further to a specific email domain by changing the condition to `request.auth != null && request.auth.token.email.matches('.*@yourdepartment[.]gov')`.)

6. Go to **Project settings** (gear icon, top left) → scroll to "Your apps" → click the **</>** (Web) icon → give the app any nickname → **Register app**. Firebase will show you a `firebaseConfig` object that looks like:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "gang-intel-db.firebaseapp.com",
     projectId: "gang-intel-db",
     storageBucket: "gang-intel-db.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```

   Copy that whole block.

7. Open `index.html` in a text editor (or GitHub's own file editor) and find the `firebaseConfig` placeholder near the top of the `<script type="module">` section. Replace it with the block you just copied. Save.

   > This config identifies *which* Firebase project to talk to — it isn't a secret password. Real access control comes from the Firestore rule in step 5 plus the accounts you create in step 8. It's normal for this to be visible in a public web app's source code.

8. Add your detectives: **Authentication → Users → Add user**. Enter each detective's email and a temporary password, one at a time. Give each detective their email + temp password through whatever channel your department already uses for credentials.

That's it — the backend is live.

## Part 2 — Put the app on GitHub Pages

1. Create a new **repository** on GitHub (Public is fine — the app has no secrets in it; see note in step 7 above).
2. Upload these files to the root of the repo:
   - `index.html` (with your `firebaseConfig` pasted in)
   - `manifest.json`
   - `service-worker.js`
   - `icon-192.png`
   - `icon-512.png`
3. Go to **Settings → Pages**. Under "Build and deployment," set **Source** to "Deploy from a branch," pick your default branch (usually `main`) and folder `/ (root)`, then **Save**.
4. GitHub will give you a URL like `https://yourusername.github.io/your-repo-name/`. It can take a minute or two to go live.

## Part 3 — Add it to your phone's home screen

**iPhone (Safari):**
1. Open the GitHub Pages URL in Safari.
2. Tap the **Share** icon (square with an arrow) in the toolbar.
3. Tap **Add to Home Screen**, then **Add**.

**Android (Chrome):**
1. Open the URL in Chrome.
2. Tap the **⋮** menu.
3. Tap **Add to Home screen** (or you may see an automatic **Install app** prompt), then confirm.

Each detective does this once, on their own phone, and signs in with the credentials you gave them.

## Using the app

- **Sign in** with the email/password your admin created. There's no "forgot password" flow in-app — the admin resets it from the Firebase Authentication console.
- **New record** (top right) opens a form for name, AKAs, photos, race, sex, gang, home address, vehicles, tattoos, weapons, and contact history entries. Tattoos, weapons, vehicles, and contacts each support multiple entries — use "+ Add" to add more, and the × to remove one.
- Tap any card in the list to view the full record, then **Edit record** or **Delete** from there. Each record shows who added and who last updated it.
- The left-hand panel (on phones, tap the ☰ icon top-left) filters by race, sex, gang affiliation, tattoo keyword, weapon keyword, address, and vehicle, and the search box at the top matches name or any moniker.
- Photos are automatically resized/compressed on upload; a single record has a hard size ceiling (Firestore's ~1MB-per-record limit), so a record with many large photos may occasionally need one removed to save — the app will tell you if that happens.
- A small dot next to your name in the top bar turns grey if the app loses its live connection; it will keep working offline and re-sync automatically once you're back online.

## Backups

The **⋮** menu still has:
- **Export all records (.json)** — downloads everything currently loaded as one file. Worth doing periodically as an offline archive, independent of Firebase.
- **Import from backup file** — adds records from a file into the shared database (merge), or wipes and replaces the whole shared database with the file's contents (replace — requires typing DELETE to confirm, since it affects everyone).
- **Erase all shared records** — wipes the database for every detective. Requires typing DELETE to confirm.

## Before relying on this operationally

This is a real, working shared tool, but a few things are worth doing before treating it as your department's system of record:

- **Check with your department's records/IT and legal unit** before storing gang-intelligence data (names, addresses, vehicles, tattoos, gang affiliation) in a third-party cloud service. Many agencies have specific rules for gang intelligence files (entry criteria, review/expiration periods, subject notification, CJIS or state-RMS requirements) and for where such data is allowed to be hosted. Firebase/Google Cloud has its own compliance certifications, but whether those satisfy your agency's specific policy is something your IT/legal unit needs to confirm — this app doesn't decide that for you.
- **Limit who gets an account** to detectives who need it, and remove accounts promptly (Authentication → Users → delete) when someone transfers or leaves.
- **Set a device passcode / biometric lock** on every phone that has this installed — signing out of the app doesn't protect data already cached on the device.
- If you outgrow the free Firebase tier (very unlikely for a small unit — the free plan covers roughly 50,000 reads and 20,000 writes per day), Firebase will prompt you to add a billing account; usage at this app's scale should stay comfortably free.

## Files in this repo

| File | Purpose |
|---|---|
| `index.html` | The entire app — UI, styling, and logic, including the Firebase config you paste in |
| `manifest.json` | Lets phones install it as a home-screen app |
| `service-worker.js` | Minimal offline caching for the app shell so it still opens without signal |
| `icon-192.png` / `icon-512.png` | App icons used on the home screen |
