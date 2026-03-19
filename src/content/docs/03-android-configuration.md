---
title: "Permissions and Configuration"
description: "Add internet permissions to AndroidManifest.xml, create local.properties with the backend URL, and verify BuildConfig is generated correctly."
step: 3
---

Before writing any Kotlin code, you need to do two things:

1. Tell Android the app needs internet access (without this, all network calls silently fail)
2. Give the app the URL of your backend in a way that's not hardcoded in source code

---

## Step 1: Add internet permission to `AndroidManifest.xml`

Open `app/src/main/AndroidManifest.xml` and add the `INTERNET` permission inside the `<manifest>` tag, before the `<application>` block:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Verify">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/Theme.Verify">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

> Without `android.permission.INTERNET`, Android blocks all outbound network requests. The app will appear to work but all HTTP calls will silently fail with a connection error.

### Cleartext traffic (HTTP)

If your backend is served over plain `http://` (not `https://`), Android will block it on newer API levels. For local development (e.g. ngrok with HTTPS, or Codespaces with a public forwarded port), this usually isn't needed — but if you're testing against a plain HTTP address, add this attribute to the `<application>` tag:

```xml
<application
    android:usesCleartextTraffic="true"
    ...>
```

---

## Step 2: Create `local.properties`

Android Studio automatically creates `local.properties` in the project root (the same directory that contains `app/`). Open it (or create it if it doesn't exist) and add:

```
BACKEND_URL=https://your-codespace-name-3000.app.github.dev
PHONE_NUMBER=+34600000000
```

- **`BACKEND_URL`** — the publicly accessible URL of your backend (the forwarded Codespaces URL from step 08, without a trailing slash)
- **`PHONE_NUMBER`** — a real phone number in E.164 format used as the default in the app's phone input field

> `local.properties` is listed in `.gitignore` by default — it's machine-specific and should never be committed.

---

## Step 3: Verify `BuildConfig` is generated

The `build.gradle.kts` you wrote in the previous step exposes these values as typed constants:

```kotlin
BuildConfig.BACKEND_URL   // "https://your-backend-url"
BuildConfig.PHONE_NUMBER  // "+34600000000"
```

To confirm they're generated correctly, build the project:

**Build → Make Project**

If the build succeeds, the `BuildConfig` class was generated. If you see an error like:

```
Execution failed for task ':app:generateDebugBuildConfig'.
local.properties is missing BACKEND_URL
```

It means `local.properties` is missing or the variable name has a typo. Double-check the file exists in the root of the Android project (same level as `app/`) and that the key is exactly `BACKEND_URL`.

---

## What you've done so far

```
app/
├── src/main/
│   ├── AndroidManifest.xml    ← INTERNET permission added
│   └── kotlin/...
│       └── MainActivity.kt
└── build.gradle.kts           ← dependencies + BuildConfig
local.properties               ← BACKEND_URL + PHONE_NUMBER (not in git)
```

---

## Checkpoint

- [ ] `AndroidManifest.xml` has `<uses-permission android:name="android.permission.INTERNET" />`
- [ ] `local.properties` exists at the project root with `BACKEND_URL` and `PHONE_NUMBER`
- [ ] **Build → Make Project** succeeds with no errors about missing properties
