---
title: "Check Your Setup"
description: "Build and run a minimal Jetpack Compose app to confirm the Android environment is working correctly before writing the real implementation."
step: 4
---

Before building the verification screen, let's confirm your Android environment is correctly wired up: Gradle builds, the emulator or device runs the app, and Compose renders UI.

This is a quick smoke test — if it works, you can be confident the problems you'll encounter in later steps are logic issues, not environment issues.

---

## Replace `MainActivity.kt` with a minimal counter app

In Android Studio, open:

```
app/src/main/kotlin/com/vonage/verify/app/MainActivity.kt
```

Replace its entire contents with:

```kotlin
package com.vonage.verify.app

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MinimalApp()
        }
    }
}

@Composable
fun MinimalApp() {
    MaterialTheme {
        Surface(modifier = Modifier.fillMaxSize()) {
            CounterScreen()
        }
    }
}

@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = "Android setup successful!",
            style = MaterialTheme.typography.headlineSmall
        )
        Spacer(modifier = Modifier.height(16.dp))
        Text(text = "Button clicked $count times")
        Spacer(modifier = Modifier.height(16.dp))
        Button(onClick = { count++ }) {
            Text("Click me")
        }
    }
}
```

This is a temporary implementation — you'll replace it with the real verification screen in the following steps.

---

## Build and run

1. Connect your Android device via USB (with USB debugging enabled), or start an Android emulator.
2. Click the **Run** button (green triangle) in Android Studio, or press `Shift+F10`.
3. Select your device from the deployment target dialog.
4. Wait for the app to install and launch.

You should see a screen with:

- "Android setup successful!"
- A counter starting at 0
- A "Click me" button

Tap the button a few times to confirm UI interactions work.

---

## What to do if it fails

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Gradle sync fails | Version mismatch | Check AGP and Kotlin plugin versions; try File → Invalidate Caches |
| App installs but crashes immediately | Missing dependency or runtime error | Check Logcat for a stack trace |
| Device not detected | USB debugging off or driver issue | Enable USB debugging in Developer Options; try a different cable |
| Build succeeds but screen is blank | Theme or Compose version issue | Check Logcat for compose-related errors |

---

## Checkpoint

- [ ] `MainActivity.kt` replaced with the counter app code
- [ ] Build succeeds (`BUILD SUCCESSFUL`)
- [ ] App runs on device or emulator showing "Android setup successful!"
- [ ] Tapping the button increments the counter

Once this works, move on — you'll replace this screen with the real verification UI.
