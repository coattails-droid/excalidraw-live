# Excalidraw live collaboration — Firebase setup

Your Excalidraw page (https://coattails-droid.github.io/excalidraw-live/) works
fully on its own. This stages the one thing it can't do yet: **Live
collaboration** (share a link, draw together in real time). That needs a small
Firebase backend (a database + file storage) owned by your Google account.

## Current state (2026-09-25)

- Firebase project **`coattails-workspace`** created, owned by your personal
  Google account.
- Firestore Database and Storage created.
- Web app registered; its `firebaseConfig` is baked into the deployed page
  (rebuilt 2026-09-25 with `VITE_APP_FIREBASE_CONFIG` set).
- What's staged in `firebase/`:
  - `firestore.rules`, `storage.rules` — the access rules the Excalidraw
    project itself publishes: anyone with a room link can read/write that
    room, nobody can list or delete other people's rooms.
  - `firebase.json`, `firestore.indexes.json` — deploy config, ready to use.
  - `.env.example` — the one config value the app build needs, with
    placeholders.

Nothing here contains secrets and nothing here costs anything on its own.

## Your one remaining step (~2 minutes, on your side)

The app is built and deployed, but Firestore and Storage still run their
default locked-down rules, so sharing a room would fail. Publish the staged
rules in the Firebase console:

1. https://console.firebase.google.com/ → open **coattails-workspace**.
2. **Build → Firestore Database → Rules** tab → replace the contents with
   `firebase/firestore.rules` from this repo → **Publish**.
3. **Build → Storage → Rules** tab → replace the contents with
   `firebase/storage.rules` from this repo → **Publish**.

That's it. The "Live collaboration" button will then create shareable rooms.
(The Firebase web API key is public by design — it ships inside the page's
code. The rules files above are what actually protect the data.)

## Two gotchas we hit (2026-09-26, both fixed)

1. **Storage needs the Blaze plan on new projects.** Since September 2024,
   Google requires the pay-as-you-go Blaze plan for Cloud Storage on any
   newly created Firebase project — on Spark the Storage section only offers
   an upgrade. Blaze keeps the same free quotas (5 GB stored, etc.), so a
   personal whiteboard stays at $0; it just needs a billing account with a
   card on file. Recommended: set a $1 budget alert in the
   [Google Cloud billing console](https://console.cloud.google.com/billing)
   → Budgets & alerts, so you'd hear about any unexpected usage first.

2. **Pasted images need CORS on the Storage bucket.** Drawings sync through
   Firestore, but images are downloaded straight from the Storage bucket by
   each participant's browser — Google blocks that cross-origin fetch unless
   the bucket explicitly allows the site. Symptom: drawings sync fine, but a
   pasted image shows in the paster's window and is broken everywhere else.
   Fix from [Cloud Shell](https://shell.cloud.google.com) (no install
   needed):

   ```bash
   cat > cors.json <<'EOF'
   [
     {
       "origin": ["https://coattails-droid.github.io"],
       "method": ["GET"],
       "maxAgeSeconds": 3600
     }
   ]
   EOF
   gsutil cors set cors.json gs://coattails-workspace.firebasestorage.app
   gsutil cors get gs://coattails-workspace.firebasestorage.app  # verify
   ```

   (Replace the origin and bucket name with your own if you fork this.)
