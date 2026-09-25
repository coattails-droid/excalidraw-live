# Excalidraw live collaboration — Firebase setup

Your Excalidraw page (https://coattails-droid.github.io/excalidraw-live/) works
fully on its own right now. This stages the one thing it can't do yet: **Live
collaboration** (share a link, draw together in real time). That needs a small
Firebase backend (a database + file storage) owned by your Google account.

## What's already staged (in `firebase/`)

- `firestore.rules`, `storage.rules` — the access rules the Excalidraw project
  itself publishes: anyone with a room link can read/write that room, nobody
  can list or delete other people's rooms.
- `firebase.json`, `firestore.indexes.json` — deploy config, ready to use.
- `.env.example` — the one config value the app build needs, with placeholders.

Nothing here contains secrets and nothing here costs anything on its own.

## Your one remaining step (~10 minutes, on your side)

1. Go to https://console.firebase.google.com/ and sign in with the Google
   account you want to own this.
2. **Add project** → name it something like `excalidraw-collab` → continue
   (Google Analytics is optional).
3. **Build → Firestore Database → Create database** → start in production
   mode → choose a region → Enable.
4. **Build → Storage → Get started** → production mode → Done.
5. Project **Settings → General → Your apps → Web (`</>`)** → give the app
   any nickname → copy the `firebaseConfig` values.
6. Send me those config values (`apiKey`, `authDomain`, `projectId`,
   `storageBucket`, `messagingSenderId`, `appId`).

## What happens after you send the config

I rebuild the Excalidraw page with collaboration enabled, publish the rules
above to your project, and redeploy. The "Live collaboration" button will
then create shareable rooms. (The Firebase web API key is public by design —
it ships inside the page's code. The rules files above are what actually
protect the data.)

## Which Google account?

Your call. If it helps: you keep your secondary account for shared/app
functionality and your personal account separate — a setup like this fits
naturally under the secondary one, but either works. Just tell me which one
you used.
