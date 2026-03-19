---
title: "Connect to the Backend"
description: "Implement the three backend API calls as suspend functions and wire them to the Start verification and Submit code buttons."
step: 7
---

The UI is in place. Now you'll add the three backend calls — `startVerification`, `submitCode`, and `requestNextWorkflow` — and wire them to the buttons.

At the end of this step the app will support the full **SMS flow**: start → receive code by SMS → submit → verified. Silent Authentication comes in the final step.

---

## Why coroutines?

Android's UI runs on the **main thread**. Network calls cannot run on the main thread — they would block the UI, making the app freeze and eventually crash with an `NetworkOnMainThreadException`.

Kotlin coroutines let you write sequential-looking async code that runs on a background thread. The key pieces:

- `scope.launch { ... }` — starts a coroutine from the UI layer (inside a Composable)
- `withContext(Dispatchers.IO) { ... }` — switches to a background thread for the network call
- When the network call finishes, execution returns to the main thread automatically so you can update `uiState`

---

## Step 1: Add the three suspend functions

Add these functions at the **bottom** of `MainActivity.kt`, after the closing brace of `VerificationScreen`:

```kotlin
/**
 * POST /verification
 * Asks the backend to start a verification for the given phone number.
 * Returns (request_id, check_url?) — check_url is null when Silent Auth isn't available.
 */
private suspend fun startVerification(phone: String): Pair<String, String?> = withContext(Dispatchers.IO) {
    val client = OkHttpClient()

    val json = Gson().toJson(mapOf("phone" to phone))
    val requestBody = json.toRequestBody("application/json".toMediaType())

    val request = Request.Builder()
        .url("$BACKEND_URL/verification")
        .post(requestBody)
        .build()

    val response = client.newCall(request).execute()
    if (!response.isSuccessful) {
        val errorBody = response.body?.string() ?: "Unknown error"
        throw IOException("Start verification failed: HTTP ${response.code} - $errorBody")
    }

    val body = response.body?.string() ?: throw IOException("Empty response body")
    val jsonBody = Gson().fromJson(body, JsonObject::class.java)

    val requestId = jsonBody.get("request_id")?.asString
        ?: throw IOException("Missing request_id in response")
    val checkUrl = jsonBody.get("check_url")?.asString // may be null

    Pair(requestId, checkUrl)
}

/**
 * POST /check-code
 * Submits the code (from SMS or Silent Auth) to the backend for validation.
 * Returns CheckCodeResponse with verified=true if the code is correct.
 */
private data class CheckCodeResponse(val verified: Boolean, val status: String?)

private suspend fun submitCode(requestId: String, code: String): CheckCodeResponse = withContext(Dispatchers.IO) {
    val client = OkHttpClient()

    val json = Gson().toJson(mapOf("request_id" to requestId, "code" to code))
    val requestBody = json.toRequestBody("application/json".toMediaType())

    val request = Request.Builder()
        .url("$BACKEND_URL/check-code")
        .post(requestBody)
        .build()

    val response = client.newCall(request).execute()
    if (!response.isSuccessful) {
        val errorBody = response.body?.string() ?: "Unknown error"
        throw IOException("Check code failed: HTTP ${response.code} - $errorBody")
    }

    val body = response.body?.string() ?: throw IOException("Empty response body")
    val jsonBody = Gson().fromJson(body, JsonObject::class.java)

    CheckCodeResponse(
        verified = jsonBody.get("verified")?.asBoolean ?: false,
        status = jsonBody.get("status")?.asString
    )
}

/**
 * POST /next
 * Asks the backend to skip Silent Auth and trigger the SMS fallback immediately.
 * Non-fatal if it fails — Vonage will fall back automatically after a timeout.
 */
private suspend fun requestNextWorkflow(requestId: String): Unit = withContext(Dispatchers.IO) {
    val client = OkHttpClient()

    val json = Gson().toJson(mapOf("requestId" to requestId))
    val requestBody = json.toRequestBody("application/json".toMediaType())

    val request = Request.Builder()
        .url("$BACKEND_URL/next")
        .post(requestBody)
        .build()

    val response = client.newCall(request).execute()
    if (!response.isSuccessful) {
        val errorBody = response.body?.string() ?: "Unknown error"
        throw IOException("Next workflow failed: HTTP ${response.code} - $errorBody")
    }
}
```

---

## Step 2: Wire the "Start verification" button

In `VerificationScreen`, replace the TODO inside the `EnterPhone`/`Error` button's `onClick`:

```kotlin
is VerifyUiState.EnterPhone,
is VerifyUiState.Error -> {
    Button(
        modifier = Modifier.fillMaxWidth(),
        onClick = {
            scope.launch {
                uiState = VerifyUiState.Loading
                statusMessage = ""

                try {
                    val (requestId, checkUrl) = startVerification(phone)

                    // No check_url means Silent Auth isn't available — go straight to SMS
                    if (checkUrl.isNullOrBlank()) {
                        uiState = VerifyUiState.EnterSms(requestId)
                        statusMessage = "Please enter the SMS code."
                    } else {
                        // check_url present — Silent Auth will be handled in the next step.
                        // For now, immediately fall back to SMS.
                        try {
                            requestNextWorkflow(requestId)
                        } catch (e: Exception) {
                            // Non-fatal — Vonage will fall back automatically
                        }
                        uiState = VerifyUiState.EnterSms(requestId)
                        statusMessage = "Please enter the SMS code."
                    }
                } catch (e: Exception) {
                    uiState = VerifyUiState.Error(e.message ?: "Unknown error")
                    statusMessage = "Unable to start verification: ${e.message}"
                }
            }
        }
    ) {
        Text("Start verification")
    }
}
```

> **Note**: When `check_url` is present, this code currently skips Silent Auth and forces SMS. You'll implement the Silent Auth path in the next step. This lets you test the full SMS flow first.

---

## Step 3: Wire the "Submit code" button

Replace the TODO inside the `EnterSms` button's `onClick`:

```kotlin
is VerifyUiState.EnterSms -> {
    Button(
        modifier = Modifier.fillMaxWidth(),
        enabled = smsCode.isNotBlank(),
        onClick = {
            scope.launch {
                uiState = VerifyUiState.Loading
                statusMessage = ""

                try {
                    val requestId = requestIdForSms
                        ?: throw IOException("Missing request_id")

                    val result = submitCode(requestId, smsCode)

                    if (result.verified) {
                        uiState = VerifyUiState.Verified("SMS")
                        statusMessage = "Verified via SMS"
                    } else {
                        uiState = VerifyUiState.EnterSms(requestId)
                        statusMessage = "Invalid code. Please try again."
                    }
                } catch (e: Exception) {
                    uiState = VerifyUiState.Error(e.message ?: "Unknown error")
                    statusMessage = "Error checking code: ${e.message}"
                }
            }
        }
    ) {
        Text("Submit code")
    }
}
```

---

## Build and test the SMS flow

Build and run the app. Test the full SMS path:

1. Tap **Start verification** — the app shows a loading spinner, then the SMS code field appears
2. Check your phone for the SMS code
3. Type the code into the SMS field and tap **Submit code**
4. You should see "Verified via SMS"

If something goes wrong:
- Check Logcat in Android Studio for the full exception
- Make sure `BACKEND_URL` in `local.properties` points to the correct public URL
- Confirm the backend is running (`nodemon server.js` in the Codespaces terminal)

---

## Checkpoint

- [ ] `startVerification`, `submitCode`, and `requestNextWorkflow` added to `MainActivity.kt`
- [ ] "Start verification" button triggers a verification and transitions to `EnterSms`
- [ ] "Submit code" button submits the code and transitions to `Verified` or shows an error
- [ ] Full SMS flow works end-to-end on a real device
