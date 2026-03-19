---
title: "UI State Machine"
description: "Define the VerifyUiState sealed class that models every screen the user can see, and set up the VerifyApp composable entry point."
step: 5
---

The verification flow has several distinct "moments" a user can be in: entering their phone number, waiting for verification to happen, entering an SMS code, seeing success, or seeing an error. 

Rather than tracking all of this with scattered booleans, we model it as a **sealed class**, a typed state machine where only one state can be active at a time. This is the core pattern that makes the Compose UI predictable and easy to follow.

---

## Why a sealed class?

Consider the alternative: a handful of booleans.

```kotlin
// Fragile — many combinations are invalid
var isLoading = false
var showSmsInput = false
var isVerified = false
var errorMessage = ""
```

With four booleans you have 16 possible combinations, but only ~5 of them make sense. A sealed class enforces the valid states at compile time.

```kotlin
// Only valid states can exist
sealed class VerifyUiState {
    data object EnterPhone : VerifyUiState()
    data object Loading : VerifyUiState()
    data class EnterSms(val requestId: String) : VerifyUiState()
    data class Verified(val method: String) : VerifyUiState()
    data class Error(val message: String) : VerifyUiState()
}
```

Each state carries only the data it needs:
- `EnterSms` carries the `requestId` — without it, we couldn't submit the code
- `Verified` carries which method succeeded (`"Silent Authentication"` or `"SMS"`)
- `Error` carries the message to display

---

## Replace `MainActivity.kt`

Open `app/src/main/kotlin/com/vonage/verify/app/MainActivity.kt` and replace the entire contents with:

```kotlin
package com.vonage.verify.app

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.unit.dp
import com.google.gson.Gson
import com.google.gson.JsonObject
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.withContext
import okhttp3.MediaType.Companion.toMediaType
import okhttp3.OkHttpClient
import okhttp3.Request
import okhttp3.RequestBody.Companion.toRequestBody
import java.io.IOException

private const val BACKEND_URL = BuildConfig.BACKEND_URL
private const val DEFAULT_PHONE = BuildConfig.PHONE_NUMBER

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent { VerifyApp() }
    }
}

/**
 * All possible UI states for the verification screen.
 * Only one state is active at a time — the UI is a function of this state.
 */
private sealed class VerifyUiState {
    data object EnterPhone : VerifyUiState()
    data object Loading : VerifyUiState()
    data class EnterSms(val requestId: String) : VerifyUiState()
    data class Verified(val method: String) : VerifyUiState()
    data class Error(val message: String) : VerifyUiState()
}

@Composable
fun VerifyApp() {
    MaterialTheme {
        Surface(modifier = Modifier.fillMaxSize()) {
            VerificationScreen()
        }
    }
}

@Composable
fun VerificationScreen() {
    // TODO: implement in the next step
}
```

At this point the project will compile but `VerificationScreen()` is empty. You'll fill it in the next step.

---

## Build the project

Click **Build → Make Project** to confirm there are no compilation errors.

You should see `BUILD SUCCESSFUL`. The app itself will show a blank screen if you run it — that's expected until the next step.

---

## What each state represents

| State | When it's active | Data it holds |
|-------|-----------------|---------------|
| `EnterPhone` | Initial screen — user hasn't started yet | nothing |
| `Loading` | Async operation in progress — disable inputs | nothing |
| `EnterSms` | Silent Auth failed or skipped — user must type the SMS code | `requestId` |
| `Verified` | Verification completed successfully | which method worked |
| `Error` | Something went wrong — show a message, allow retry | error message |

---

## Checkpoint

- [ ] `VerifyUiState` sealed class defined with five states
- [ ] `MainActivity` calls `setContent { VerifyApp() }`
- [ ] `VerifyApp` and `VerificationScreen` composables defined
- [ ] **Build → Make Project** succeeds
