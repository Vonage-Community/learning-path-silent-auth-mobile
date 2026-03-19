---
title: "Add Dependencies"
description: "Add all required libraries to build.gradle.kts and sync Gradle."
step: 2
---

The app needs several libraries to build its UI, make HTTP requests to the backend, and later use the Vonage SDK for Silent Authentication. You'll add them all now so you don't have to interrupt the flow later.

---

## Open `app/build.gradle.kts`

In Android Studio, open the file at:

```
app/build.gradle.kts
```

(Make sure you're opening the **app-level** `build.gradle.kts`, not the project-level one at the root.)

---

## Replace the entire file content

Replace the entire content of `app/build.gradle.kts` with the following:

```kotlin
import java.util.Properties

plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

val localPropsFile = rootProject.file("local.properties")
val props = Properties().apply {
    if (!localPropsFile.exists()) {
        error("Missing local.properties in the project root. Add BACKEND_URL and PHONE_NUMBER there.")
    }
    load(localPropsFile.inputStream())
}

android {
    namespace = "com.vonage.verify.app"
    compileSdk = 36

    defaultConfig {
        applicationId = "com.vonage.verif.app"
        minSdk = 24
        targetSdk = 36
        versionCode = 1
        versionName = "1.0"

        val backendUrl = props.getProperty("BACKEND_URL")
            ?: error("local.properties is missing BACKEND_URL")
        val phoneNumber = props.getProperty("PHONE_NUMBER")
            ?: error("local.properties is missing PHONE_NUMBER")

        buildConfigField("String", "BACKEND_URL", "\"$backendUrl\"")
        buildConfigField("String", "PHONE_NUMBER", "\"$phoneNumber\"")

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        vectorDrawables {
            useSupportLibrary = true
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    buildFeatures {
        compose = true
        buildConfig = true
    }

    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {
    // Compose and UI
    implementation("androidx.activity:activity-compose:1.12.3")
    implementation(platform("androidx.compose:compose-bom:2026.01.01"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.material3:material3")

    // Networking
    implementation("com.squareup.okhttp3:okhttp:5.3.2")
    implementation("com.google.code.gson:gson:2.13.2")

    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.10.2")
}
```

### What each dependency does

| Dependency | Purpose |
|-----------|---------|
| `activity-compose` | Bridges Android Activity lifecycle with Jetpack Compose |
| `compose-bom` | Manages Compose library versions so they're always compatible |
| `compose.ui` | Core Compose UI primitives |
| `material3` | Material Design 3 components (buttons, text fields, etc.) |
| `okhttp3` | HTTP client for calling the backend (more reliable than Android's built-in `HttpURLConnection`) |
| `gson` | Converts Kotlin objects to/from JSON |
| `kotlinx-coroutines-android` | Runs network calls off the main thread; lets you update UI from coroutines safely |

### About `local.properties` and `BuildConfig`

The `build.gradle.kts` above reads `BACKEND_URL` and `PHONE_NUMBER` from a `local.properties` file and exposes them as typed `BuildConfig` constants. This means:

- The backend URL is never hardcoded in your Kotlin source
- `local.properties` is not committed to git (Android Studio creates it automatically and `.gitignore` excludes it)

You'll create `local.properties` in the next step.

---

## Sync Gradle

After saving the file, Android Studio shows a banner at the top:

> "Gradle files have changed since last project sync. A project sync may be necessary."

Click **Sync Now**.

Wait for the sync to complete (progress bar at the bottom). You should see no errors in the Build output.

If sync fails:
- Check your internet connection (Gradle needs to download the libraries)
- Make sure the compileSdk and targetSdk values match the Android SDK you have installed

---

## Checkpoint

- [ ] `app/build.gradle.kts` updated with all dependencies
- [ ] Gradle sync completes without errors
- [ ] No red underlines in the build file
