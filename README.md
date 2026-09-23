# AttendQR

# AttendQR

Simple QR-based attendance for universities.

Professors start a timed attendance session and display a QR code in class.
Students, already identified through their account, scan the code and are
marked present automatically — no re-typing name or student number every
session. Built as an **offline-first Progressive Web App**: once a student
has loaded the app while online, it keeps working (reading cached data,
queueing writes) if connectivity briefly drops.

## Current implementation status

**Step 1 of 14 — complete.** Project scaffold, Firebase configuration, and
PWA foundation only.

Implemented:
- Vite + React project structure (`components/`, `pages/`, `services/`,
  `hooks/`, `utils/`)
- Firebase app/Auth/Firestore initialization, isolated in `src/firebase.js`
- Firestore offline persistence (IndexedDB-backed local cache, multi-tab)
- `vite-plugin-pwa` configured to cache the app shell for offline reloads
- Environment-variable-based Firebase config (`.env.example` provided)
- A status shell page showing live Firebase / PWA / online-offline state
- Graceful "Firebase not configured" warning instead of a crash when env
  vars are missing

**Not implemented yet** (later steps): login/registration UI, class
creation, class enrollment, attendance sessions, QR generation/scanning,
the real-time attendance dashboard, history, export, and the strict
Firestore Security Rules. See **Roadmap** below.

## Technology stack

- React 18 + Vite 5 (JavaScript, not TypeScript)
- React Router 6
- Firebase Authentication
- Cloud Firestore (with offline persistence)
- Firebase Hosting (for later deployment step)
- `vite-plugin-pwa` (Workbox-based service worker + manifest)
- No custom backend server — Firestore + Security Rules is the backend

## Installation

```bash
npm install
```

## Firebase setup

1. Go to the [Firebase Console](https://console.firebase.google.com/) and
   create a project (or use an existing one).
2. In the project, add a **Web app** (Project settings → General → Your
   apps → `</>`). Copy the config values it shows you.
3. Enable **Authentication** in the Firebase console (you can enable the
   Email/Password provider now; it will be wired up in Step 2).
4. Enable **Cloud Firestore** (Build → Firestore Database → Create
   database). Start in test mode for local development — Step 12 will
   replace this with strict security rules before anything real goes into
   it.
5. (Optional, for later steps) Enable **Firebase Hosting**.

## Environment variables

Copy the example file and fill in the values from step 2 above:

```bash
cp .env.example .env
```

```
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_STORAGE_BUCKET=...
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...
```

`.env` is git-ignored and must never be committed. `.env.example` contains
no real credentials and is safe to commit.

If any of `VITE_FIREBASE_API_KEY`, `VITE_FIREBASE_AUTH_DOMAIN`, or
`VITE_FIREBASE_PROJECT_ID` are missing, the app will still start and will
show a clear on-screen warning instead of crashing.

## Development

```bash
npm run dev
```

Opens the app at `http://localhost:5173`. The Firebase/PWA/online status
panel on the landing page reflects the live state of each system.

## Production build

```bash
npm run build
npm run preview   # serve the production build locally to test the PWA
```

The service worker and offline caching **only behave like production in
the `preview` build**, not in `npm run dev` (though `devOptions.enabled` in
`vite.config.js` does register a dev-mode service worker so you can test
early).

## Offline / PWA explanation

- **App shell caching**: `vite-plugin-pwa` pre-caches the built HTML/CSS/JS
  so the app can be reopened even with no network connection.
- **Firestore offline persistence**: `src/firebase.js` initializes
  Firestore with `persistentLocalCache` (IndexedDB-backed, shared across
  tabs). Reads are served from the local cache when offline, and writes
  are queued locally and automatically synced when the network returns.
- **Honesty about limits**: this does **not** mean two devices can talk to
  each other with zero network. A student scanning a QR code while
  completely offline can have their attendance write *queued* on their
  device — it only reaches the professor's session once that student's
  device regains connectivity and Firestore syncs the pending write. There
  is no way to reach the Firestore server with no network at all.
- **Online/offline indicator**: uses the browser's real `navigator.onLine`
  value plus the `online`/`offline` window events (`src/hooks/useOnlineStatus.js`) — not a simulated or hardcoded state.

## Architecture notes

- `src/firebase.js` is the **only** file that reads Firebase env vars or
  calls `initializeApp` / `getAuth` / `getFirestore`. Nothing else should
  configure Firebase directly.
- `src/services/` will hold all Firestore read/write logic as features are
  added (e.g. `services/classes.js`, `services/attendance.js`), kept
  separate from React components.
- `src/hooks/` holds reusable stateful logic (e.g. `useOnlineStatus`).
- `src/components/` holds presentational/reusable UI; `src/pages/` holds
  route-level screens.
- Firebase Authentication is the source of identity. Firestore **Security
  Rules**, not client-side code, will be the actual authorization boundary
  once written in Step 12 — nothing sent from the browser (including a
  claimed "role") is trusted until rules enforce it server-side.

## Roadmap

1. ~~Project setup + Firebase configuration + PWA foundation~~ ✅ (this step)
2. Authentication + user profiles
3. Professor class creation and management
4. Student enrollment / class membership
5. Professor attendance session creation
6. Temporary QR code generation
7. Student QR scanning and automatic identity recognition
8. Attendance submission + Firestore offline queue
9. Real-time professor attendance dashboard
10. Attendance history and filtering
11. CSV/Excel export
12. Strict Firestore Security Rules and security audit
13. Production PWA + Firebase Hosting deployment
14. UI polish, responsive mobile design, error handling, loading states, accessibility
