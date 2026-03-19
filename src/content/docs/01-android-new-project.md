---
title: "Create the Project"
description: "Create a new Android project in Android Studio with the correct settings for this tutorial."
step: 1
---

> **Setup reminder**: The tutorial is open in your Codespaces browser. Keep Android Studio open to paste and run the code as you follow each step.

In this step you'll create a new Android project in Android Studio using the settings required for this tutorial.

---

## What you need

- [Android Studio](https://developer.android.com/studio) (stable channel) installed on your laptop
- A real Android device running **Android 7.0 or higher** (API 24+) connected via USB, with USB debugging enabled

> **Why a real device?** Silent Authentication requires a mobile carrier network context that Android emulators cannot provide. You can use an emulator for the SMS fallback path, but Silent Auth will always fail on an emulator — which is fine for development, but you won't see the primary path working.

---

## Overview of the app flow

The app will follow this flow:

1. User enters their phone number.
2. App sends the phone number to the backend (`POST /verification`).
3. If Silent Authentication succeeds, the user is verified.
4. If it fails (or isn’t available), the user is asked for the SMS code.
5. The app sends the code to the backend (`POST /check-code`).
6. The app shows the verification result.

> Remember: The Android app never stores Vonage secrets. It only calls your backend.

---

## Create the project

1. Open Android Studio.
2. Click **New Project**.
3. Select the **Empty Activity** template (this gives you a Jetpack Compose setup).
4. Click **Next** and configure the project:

| Setting | Value |
|---------|-------|
| Name | `Verify2FADemo` |
| Package name | `com.vonage.verify.app` |
| Save location | anywhere on your machine |
| Language | **Kotlin** |
| Minimum SDK | **API 24 (Android 7.0 Nougat)** |

5. Click **Finish**.
6. Wait for the initial Gradle sync to complete. You'll see a progress bar at the bottom of the IDE.

---

## Confirm the project builds

Once Gradle sync finishes, build the project to confirm the base setup is correct:

In the menu bar, click **Build → Make Project** (or press `Ctrl+F9` / `Cmd+F9`).

You should see `BUILD SUCCESSFUL` in the Build output panel at the bottom.

If the build fails at this stage, it's almost always a Gradle or SDK version issue:

- Check that the Android SDK is installed (Android Studio should have prompted you on first run)
- Try **File → Invalidate Caches → Invalidate and Restart**

---

## Your project structure

After the initial sync, the relevant files are:

```
app/
├── src/
│   └── main/
│       ├── AndroidManifest.xml
│       └── kotlin/com/vonage/verify/app/
│           └── MainActivity.kt
└── build.gradle.kts
build.gradle.kts          (project level)
settings.gradle.kts
```

You'll edit `MainActivity.kt`, `AndroidManifest.xml`, and `app/build.gradle.kts` in the coming steps.

---

## Checkpoint

- [ ] Android Studio opened successfully
- [ ] New project created with package name `com.vonage.verify.app` and minimum SDK API 24
- [ ] Gradle sync completed without errors
- [ ] **Build → Make Project** succeeds
