# Publishing Flare-Hydra to the Google Play Store

## 1. Developer account
Create one at https://play.google.com/console — one-time $25 fee, plus
identity verification (may ask for a government ID).

## 2. The closed-testing gate (read this first)

New personal developer accounts (anyone who registered after Nov 13, 2023)
**must** run a closed test with at least 12 opted-in testers, active
continuously for 14 days, before Google grants "production access" — the
ability to publish publicly. There's no way to skip this for a new account.
Plan for roughly 2+ weeks between "app is ready" and "app is actually live."

Practical tips:
- Recruit real people (friends, classmates, TCS colleagues) — Google tracks
  actual engagement, not just installs.
- Ship at least one update during the 14 days (even a tiny tweak) — Google
  reportedly treats a completely static test as a sign you're just running
  out the clock.
- After 12+ testers have been opted in for 14 continuous days, a "Production
  access" option appears on your Play Console dashboard with a short
  questionnaire about your app and testing process.

## 3. Generate your signing key (do this locally, keep it forever)

```bash
keytool -genkey -v -keystore flare-hydra-release.jks -alias flarehydra -keyalg RSA -keysize 2048 -validity 10000
```

Keep the resulting `.jks` file and the passwords you set somewhere safe and
backed up. This key is what proves every future update is really from you —
losing it means you can never update this app listing again.

## 4. Add your signing secrets to GitHub

In your repo: **Settings → Secrets and variables → Actions → New repository
secret**. Add these three:

| Secret name | Value |
|---|---|
| `KEYSTORE_BASE64` | Output of `base64 -w0 flare-hydra-release.jks` (base64-encoded keystore file) |
| `KEYSTORE_PASSWORD` | The keystore password you chose |
| `KEY_ALIAS` | The alias you chose (e.g. `flarehydra`) |

None of these ever appear in your code or commit history — GitHub Actions
injects them at build time only.

## 5. Build the signed release bundle

Push a tag to trigger the release workflow:

```bash
git tag v1.0.0
git push origin v1.0.0
```

This runs `.github/workflows/build-android-release.yml`, which builds and
signs `app-release-signed.aab`. Download it from the Actions run's
**Artifacts** section, same as the debug APK before.

## 6. Store listing requirements

- **Privacy policy URL — mandatory**, since Flare-Hydra collects email and
  password. A single hosted page (even a simple GitHub Pages page) describing
  what you collect and how it's stored is enough to satisfy this.
- App icon, at least 2 phone screenshots, short description (80 chars) and
  full description (4000 chars max).
- Content rating questionnaire (a water tracker will rate very low/general).
- **Data safety form** — declare that you collect email + password, and how
  it's stored (on-device only right now, since there's no backend yet).

## 7. Target API level

As of Aug 31, 2026, Play Console requires new apps/updates to target Android
16 (API level 36) or higher. Check `android/variables.gradle` after running
`npx cap add android` and bump `targetSdkVersion`/`compileSdkVersion` if the
Capacitor-generated default is lower.

## 8. Upload and release

Upload `app-release-signed.aab` to a **Closed testing** track first (this is
what satisfies the requirement in step 2). Once you've met the 12
testers/14 days threshold and applied for production access, promote the
release to **Production** from Play Console.
