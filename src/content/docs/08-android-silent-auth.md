---
title: "Silent Authentication"
description: "Add the Vonage Client SDK, initialise it in onCreate, implement the checkSilentAuth function, and update the start verification flow to attempt Silent Auth before falling back to SMS."
step: 8
---

The SMS flow is working. Now you'll add the primary path: **Silent Authentication**.

Silent Auth works by making an HTTP request to `check_url` over the device's **mobile data connection**. The Vonage, in coordination with the mobile carrier, verifies that the request came from the expected SIM, without the user typing anything.

The Vonage Client SDK handles the cellular routing for you. Without it, the request might go over Wi-Fi, which would break the carrier-level verification.

---

## Why the Vonage Client SDK?

You might wonder: why not just use OkHttp to call `check_url`? Two reasons:

1. **Carrier routing** — the SDK forces the request over mobile data (not Wi-Fi), which is required for the carrier-level identity check
2. **Redirect handling** — the check flow involves multiple redirects that need to stay on the cellular network. The SDK handles this correctly.

---

## Step 1: Verify the Vonage SDK dependency

Open `app/build.gradle.kts` and confirm the SDK is listed in `dependencies`:

```kotlin
implementation("com.vonage:client-library:1.0.1")
```

You already added this in step 10. If it's there, no action needed.

---

## Step 2: Initialise the SDK in `onCreate`

Open `MainActivity.kt` and update the `onCreate` method to initialise the SDK:

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Required: initialise the Vonage SDK before making any cellular requests
        VGCellularRequestClient.initializeSdk(this.applicationContext)

        setContent { VerifyApp() }
    }
}
```

Also add the SDK imports at the top of the file, with the other imports:

```kotlin
import com.vonage.clientlibrary.VGCellularRequestClient
import com.vonage.clientlibrary.VGCellularRequestParameters
```

---

## Step 3: Add the `checkSilentAuth` function

Add this suspend function to the bottom of `MainActivity.kt`, alongside the other network functions:

```kotlin
/**
 * Makes a cellular GET request to check_url using the Vonage SDK.
 * The SDK routes the request over mobile data so the carrier can verify the SIM.
 * Returns the code returned by the Vonage servers, which is then checked via /check-code.
 */
private suspend fun checkSilentAuth(url: String): String = withContext(Dispatchers.IO) {
    val params = VGCellularRequestParameters(
        url = url,
        headers = mapOf(),
        queryParameters = mapOf(),
        maxRedirectCount = 10
    )

    val response = VGCellularRequestClient.getInstance()
        .startCellularGetRequest(params, false)

    val httpStatus = response.optInt("http_status", -1)
    val sdkError = response.optString("error", "")

    if (sdkError.isNotEmpty()) {
        throw IOException("Silent Auth SDK error: $sdkError")
    }

    if (httpStatus !in 200..299) {
        val rawBody = response.optString("response_raw_body", "")
        throw IOException("Silent Auth failed: HTTP $httpStatus - ${rawBody.take(200)}")
    }

    val bodyJsonObj = response.optJSONObject("response_body")
    val code = bodyJsonObj?.optString("code", null)

    if (code.isNullOrBlank()) {
        throw IOException("Silent Auth response missing 'code'")
    }

    code
}
```

---

## Step 4: Update the "Start verification" button

Now update the `onClick` handler in the `EnterPhone`/`Error` button to actually attempt Silent Auth when `check_url` is present:

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

                    if (!checkUrl.isNullOrBlank()) {
                        // check_url present — attempt Silent Authentication
                        statusMessage = "Attempting Silent Authentication..."
                        try {
                            val codeFromSa = checkSilentAuth(checkUrl)
                            val result = submitCode(requestId, codeFromSa)

                            if (result.verified) {
                                uiState = VerifyUiState.Verified("Silent Authentication")
                                statusMessage = "Verified via Silent Authentication"
                            } else {
                                // Code was returned but backend didn't accept it — fallback to SMS
                                uiState = VerifyUiState.EnterSms(requestId)
                                statusMessage = "Silent Authentication didn't complete. Please enter the SMS code."
                            }
                        } catch (e: Exception) {
                            // Silent Auth failed (network issue, no mobile data, unsupported carrier, etc.)
                            // Fall back to SMS immediately
                            statusMessage = "Silent Authentication failed. Please enter the SMS code."
                            try {
                                requestNextWorkflow(requestId)
                            } catch (fallbackError: Exception) {
                                // Non-fatal — Vonage will fall back automatically after timeout
                            }
                            uiState = VerifyUiState.EnterSms(requestId)
                        }
                    } else {
                        // No check_url — Silent Auth not available for this number/network
                        uiState = VerifyUiState.EnterSms(requestId)
                        statusMessage = "Silent Authentication is not available. Please enter the SMS code."
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

### The fallback logic explained

```
start verification
    ↓
check_url present?
    NO  → show SMS input immediately
    YES → attempt Silent Auth
              ↓
          SDK succeeds + code returned?
              YES → POST /check-code → verified? → Ok or show SMS input
              NO  → POST /next (best-effort) → show SMS input
```

The `/next` call is wrapped in its own `try/catch` because it's a "nice to have" — if it fails, the user experience is slightly worse (they wait ~20 seconds for Vonage to time out automatically) but the verification still completes.

---

## Build and run

Build and run the app on a **real device with mobile data enabled**.

Test both paths:

**Silent Auth path** (device on mobile data, supported carrier):
1. Tap **Start verification**
2. If Silent Auth succeeds, you should see "Verified via Silent Authentication" — no code entry needed

**SMS fallback path** (device on Wi-Fi only, or Silent Auth not supported):
1. Tap **Start verification**
2. The status message says "Silent Authentication failed. Please enter the SMS code."
3. Enter the code from the SMS → "Verified via SMS"

> **Tip**: To force the SMS path even on mobile data, turn on Airplane mode then re-enable only Wi-Fi — this disables mobile data without losing network connectivity.

---

## Checkpoint

- [ ] `VGCellularRequestClient.initializeSdk(...)` called in `onCreate`
- [ ] `checkSilentAuth` function added
- [ ] "Start verification" button attempts Silent Auth when `check_url` is present
- [ ] Falls back to SMS correctly when Silent Auth fails
- [ ] Silent Auth verified end-to-end on a real device with mobile data
