# Flare-Hydra — Android app via Capacitor

This folder wraps the Flare-Hydra web app (in `www/index.html`) into a native
Android shell using Capacitor. Your existing HTML/CSS/JS stays as-is; this
just adds the native container, app icon, and real background notifications.

The app's notification code already checks for Capacitor at runtime, so
`www/index.html` works both as a plain browser page (falls back to browser
notifications) and inside the native app (uses real scheduled local
notifications that fire even if the app is closed).

## Getting the APK without installing Android Studio

If you don't want to set up Android Studio locally, this project includes a
GitHub Actions workflow (`.github/workflows/build-android.yml`) that builds
the debug APK on GitHub's own servers, which come with the Android SDK
preinstalled — no local setup needed.

1. Create a free GitHub account if you don't have one, and a new repository.
2. Push this whole project folder to that repository:
   ```bash
   cd flare-hydra-app
   git init
   git add .
   git commit -m "Flare-Hydra Android project"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<your-repo>.git
   git push -u origin main
   ```
3. On GitHub, open your repo → the **Actions** tab. You'll see "Build Android
   APK" running automatically (it also runs on every future push, or you can
   trigger it manually from that tab).
4. When it finishes (green check), click into the run → scroll to
   **Artifacts** → download `flare-hydra-debug-apk`. It's a zip containing
   `app-debug.apk`.
5. Transfer that `.apk` to your Android phone (email it to yourself, Google
   Drive, USB cable, whatever's easiest) and open it there. Android will ask
   to confirm installing from that source the first time — allow it, and the
   app installs like any other.

This gives you a real, installable APK with zero Android Studio install on
your machine. The tradeoff: no emulator/live debugging — for that you'd still
want Android Studio locally eventually, but for "just get it on my phone and
try it," this is the fastest path.

## What you need installed first (for the local Android Studio route)

- **Node.js** (LTS) — https://nodejs.org
- **Android Studio** — https://developer.android.com/studio (this also installs
  the Android SDK you need)
- A **JDK** (Android Studio bundles one; you don't need to install this separately
  in recent versions)

## One-time setup

Open a terminal in this folder and run:

```bash
npm install
npx cap add android
npx cap sync
```

`npx cap add android` creates an `android/` folder — a real native Android
Studio project. `npx cap sync` copies `www/` into it and wires up the plugins.

## Running it

```bash
npx cap open android
```

This opens the project in Android Studio. Let Gradle finish syncing (first
time can take a few minutes), then hit the green ▶ Run button to install it
on an emulator or a plugged-in Android phone with USB debugging enabled.

Whenever you change `www/index.html`, re-run `npx cap sync` and re-run in
Android Studio to see the update.

## App icon

Right-click the `res` folder in Android Studio → **New → Image Asset**. Feed
it a square PNG or SVG (a simple water-drop mark works well — you can reuse
the teal drop shape from the login screen) and it will generate all the
required icon sizes automatically.

## Notifications — a couple of real-world notes

- Android 13+ requires the user to grant a notification permission at
  runtime. The in-app "Browser notifications" toggle already triggers this
  via the Local Notifications plugin — no extra code needed.
- The app schedules a rolling batch of reminder notifications ahead of time
  (so they fire even if the app is closed), and refreshes that batch each
  time the app is opened. Some Android phones aggressively battery-optimize
  background apps — if reminders feel unreliable on a particular device,
  excluding the app from battery optimization in Android's settings helps.
  A production app usually pairs this with server-side push (Firebase Cloud
  Messaging) for guaranteed delivery regardless of battery settings — that's
  a backend addition, not something Capacitor alone solves.

## Publishing to the Play Store (when you're ready)

1. Create a Google Play Console developer account (one-time $25 fee).
2. In Android Studio: **Build → Generate Signed Bundle / APK → Android App
   Bundle**, and create a signing key when prompted (keep this key safe —
   you'll need the same one for every future update).
3. In Play Console, create a new app, fill in the store listing (screenshots,
   description, content rating questionnaire).
4. **Privacy policy is required** since this app collects email/password —
   you'll need a hosted privacy policy page and its URL before Google will
   publish the app.
5. Upload the `.aab` file, submit for review.

## Going further

Right now accounts/passwords/logs are still stored only on the device
(`window.storage`, Capacitor's Preferences-backed storage under the hood).
For real accounts, real password security, and real email-based OTP, that
storage layer needs to be replaced with calls to an actual backend — see the
Spring Boot + MySQL plan we discussed for the next step.
