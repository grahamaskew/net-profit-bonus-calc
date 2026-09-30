# Net Profit Bonus Calculator (Firebase edition)

**Status (2026-09-30): set up.** Firebase project `nop-bonus-calc-gmvz3` is live and its config is in `index.html`. The site is hosted on GitHub Pages at <https://grahamaskew.github.io/net-profit-bonus-calc/>. Redeploy rules and the auth provider with `firebase deploy --only firestore:rules,auth`. The steps below are for recreating it from scratch.

A single static page (`index.html`) with:

- Email + password sign-in — each user sees only their own calculator.
- Autosave (~1s after each edit) to a private Firestore document at `calcs/{uid}`,
  enforced by `firestore.rules`.
- A new user always starts from a blank calculator.

## 1. Create the Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) →
   **Add project** → name it (e.g. "nop-bonus-calc") → finish the wizard
   (Google Analytics is optional).
2. **Build → Authentication → Get started → Sign-in method → Email/Password → Enable.**
3. **Build → Firestore Database → Create database** → **production mode** → pick a
   region close to you.
4. **Project settings** (gear icon) → **General** → **Your apps** → `</>` (web) icon →
   register an app (any nickname, no Firebase Hosting needed) → copy the
   `firebaseConfig` object.
5. In `index.html`, find `FIREBASE_CONFIG` at the top of the `<script type="module">`
   block and paste your six values over the `REPLACE_WITH_...` placeholders.

The config is not a secret — it ships in client code by design. The security rules
are what protect each user's data.

## 2. Apply the security rules

**Build → Firestore Database → Rules** → paste in `firestore.rules` → **Publish**.

Only the signed-in owner can read or write `calcs/{their uid}`. Everything else is denied.

## 3. Authorize your domain

**Authentication → Settings → Authorized domains → Add domain** → add your Vercel
domain (e.g. `nop-bonus-calc.vercel.app`). `localhost` is there by default.

## 4. Deploy on Vercel

1. Push this folder to GitHub.
2. Vercel → **Add New… → Project** → import the repo.
3. Framework preset: **Other** (no build step). Deploy.

## Local preview

```
python3 -m http.server 8000
```

Open `http://localhost:8000/`. Sign-in works once steps 1–2 are done.

## Who can see the data

- Each user: only their own calculator.
- Firebase project owners: everything, via the Firebase console.
- Exported PDFs are ordinary files — whoever has one can read it.
