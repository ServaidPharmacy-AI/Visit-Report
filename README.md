# Servaid Field Visits — Android app

This wraps the Servaid Field Visits web app in a native Android shell and produces an
installable APK. The whole app lives in `app/src/main/assets/index.html`; the Java shell
adds the things a browser cannot do on its own:

- **Saves PDFs, CSVs and backups into `Downloads/Servaid`** and opens the PDF straight
  after saving, instead of dropping a file into browser storage.
- **Camera on the photo fields** — tapping "Take or choose photo" offers the camera and
  the gallery side by side.
- **GPS check-in** on visit start.
- Runs offline. There is no server and no network call anywhere in the app.

One detail worth knowing if you ever modify the shell: the app is served over
`https://appassets.androidplatform.net` through `WebViewAssetLoader`, not from a
`file://` URL. Chromium blocks IndexedDB and geolocation on `file://` origins, and every
visit, photo and signature is stored in IndexedDB — loading it as a file would silently
lose all data on close.

---

## Build it — pick one

### 1. GitHub Actions (no software to install)

1. Create a new repository on GitHub, private is fine.
2. Upload this whole folder to it, keeping the structure.
3. Open the **Actions** tab. The `Build APK` workflow runs on push, or press
   **Run workflow**.
4. When it finishes (about three minutes), open the run and download the
   **servaid-field-apk** artifact. Unzip it — the `.apk` is inside.

### 2. Android Studio

Open this folder with **File → Open**, let it sync, then
**Build → Build Bundle(s) / APK(s) → Build APK(s)**.
The result lands in `app/build/outputs/apk/debug/`.

### 3. Command line

Needs JDK 17 and the Android SDK (`ANDROID_HOME` set, platform 34 installed):

```
gradle assembleDebug
```

---

## Install it on a phone

1. Put the `.apk` on the phone — WhatsApp it, email it, or copy over USB.
2. Tap it. Android will ask permission to install from this source; allow it.
3. Grant location and camera when the app first asks.

The build is signed with the standard Android debug key. That is fine for installing
directly on your own team's phones. It is **not** accepted by the Play Store, and phones
treat a debug-signed app as an unknown source.

## Updating the app later

Replace `app/src/main/assets/index.html` with the newer file, bump `versionCode` and
`versionName` in `app/build.gradle`, and rebuild. Installing the new APK over the old one
keeps existing data as long as the signing key has not changed. **Uninstalling deletes
every visit and photo**, so run *Back up all data* from the app menu first.

## A real signing key (optional)

Debug-signed builds expire and cannot be updated with a differently-signed build. For
anything long-lived, generate a key once:

```
keytool -genkey -v -keystore servaid.jks -keyalg RSA -keysize 2048 \
        -validity 10000 -alias servaid
```

Keep `servaid.jks` somewhere safe — lose it and you can never update the installed app.
Then add to `app/build.gradle` inside `android { }`:

```gradle
signingConfigs {
    release {
        storeFile file("../servaid.jks")
        storePassword System.getenv("KS_PASS")
        keyAlias "servaid"
        keyPassword System.getenv("KEY_PASS")
    }
}
```

and point `buildTypes.release.signingConfig` at `signingConfigs.release`.

---

## What is in here

```
app/src/main/assets/index.html       the entire app — this is the file you update
app/src/main/java/.../MainActivity.java   WebView shell, camera picker, file saving
app/src/main/AndroidManifest.xml     permissions and the launcher entry
app/src/main/res/                    icon, theme, FileProvider paths
.github/workflows/build-apk.yml      the GitHub Actions build
```

Package name `pk.servaid.field`, minimum Android 7.0, target Android 14.
