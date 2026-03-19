---
title: "Build the Verification Screen"
description: "Implement the full VerificationScreen composable with phone input, SMS input, buttons for each state, and a status message."
step: 6
---

# Build the Verification Screen

Now you'll build the full UI for the verification flow. The screen renders differently depending on the current `VerifyUiState` — this is Compose's core idea: **UI = function(state)**.

At the end of this step the screen will be fully functional visually, but buttons won't do anything yet — you'll wire them to the backend in the next step.

---

## Replace `VerificationScreen` in `MainActivity.kt`

Open `MainActivity.kt` and replace the empty `VerificationScreen()` function with the full implementation below.

Add this right after `VerifyApp()`:

```kotlin
@Composable
fun VerificationScreen() {
    var phone by remember { mutableStateOf(DEFAULT_PHONE) }
    var smsCode by remember { mutableStateOf("") }

    var uiState by remember { mutableStateOf<VerifyUiState>(VerifyUiState.EnterPhone) }
    var statusMessage by remember { mutableStateOf("") }

    val scope = rememberCoroutineScope()

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = "Verify your phone",
            style = MaterialTheme.typography.headlineSmall
        )

        Spacer(modifier = Modifier.height(12.dp))

        // Phone number input
        OutlinedTextField(
            value = phone,
            onValueChange = { phone = it },
            enabled = uiState !is VerifyUiState.Loading,
            label = { Text("Phone number (E.164)") },
            placeholder = { Text("+34600111222") },
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Phone),
            modifier = Modifier.fillMaxWidth()
        )

        // SMS code input — only visible in EnterSms state
        val requestIdForSms = (uiState as? VerifyUiState.EnterSms)?.requestId
        if (requestIdForSms != null) {
            Spacer(modifier = Modifier.height(12.dp))
            OutlinedTextField(
                value = smsCode,
                onValueChange = { smsCode = it },
                enabled = uiState !is VerifyUiState.Loading,
                label = { Text("SMS code") },
                keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number),
                modifier = Modifier.fillMaxWidth()
            )
        }

        Spacer(modifier = Modifier.height(16.dp))

        // Button / indicator changes based on current state
        when (uiState) {
            is VerifyUiState.Loading -> {
                CircularProgressIndicator()
            }

            is VerifyUiState.EnterPhone,
            is VerifyUiState.Error -> {
                Button(
                    modifier = Modifier.fillMaxWidth(),
                    onClick = {
                        // TODO: wire to backend in next step
                    }
                ) {
                    Text("Start verification")
                }
            }

            is VerifyUiState.EnterSms -> {
                Button(
                    modifier = Modifier.fillMaxWidth(),
                    enabled = smsCode.isNotBlank(),
                    onClick = {
                        // TODO: wire to backend in next step
                    }
                ) {
                    Text("Submit code")
                }
            }

            is VerifyUiState.Verified -> {
                val method = (uiState as VerifyUiState.Verified).method
                Text("Success! Verified using $method.")
                Spacer(modifier = Modifier.height(12.dp))
                Button(
                    modifier = Modifier.fillMaxWidth(),
                    onClick = {
                        smsCode = ""
                        statusMessage = ""
                        uiState = VerifyUiState.EnterPhone
                    }
                ) {
                    Text("Verify another number")
                }
            }
        }

        Spacer(modifier = Modifier.height(12.dp))

        if (statusMessage.isNotBlank()) {
            Text(statusMessage)
        }
    }
}
```

---

## What you just built

### Local state

```kotlin
var phone by remember { mutableStateOf(DEFAULT_PHONE) }
var smsCode by remember { mutableStateOf("") }
var uiState by remember { mutableStateOf<VerifyUiState>(VerifyUiState.EnterPhone) }
var statusMessage by remember { mutableStateOf("") }
```

`remember { mutableStateOf(...) }` creates state that survives recompositions. When any of these values change, Compose re-renders the affected parts of the UI.

### Conditional SMS input

```kotlin
val requestIdForSms = (uiState as? VerifyUiState.EnterSms)?.requestId
if (requestIdForSms != null) { ... }
```

The SMS code field only appears when `uiState` is `EnterSms`. The safe cast (`as?`) returns `null` for all other states, so the `if` block is skipped. This `requestId` will be passed to `/check-code` when the user submits the code.

### Disabled inputs during loading

```kotlin
enabled = uiState !is VerifyUiState.Loading
```

Both text fields are disabled while an async operation is in progress, preventing the user from modifying input mid-request.

### `when (uiState)` — the heart of the screen

The `when` block is exhaustive — it covers every possible state. Compose only renders the matching branch, so:
- In `Loading` → spinner
- In `EnterPhone` or `Error` → "Start verification" button
- In `EnterSms` → "Submit code" button (disabled until the user types something)
- In `Verified` → success message + "Verify another number" reset button

---

## Build and run

Click **Build → Make Project**, then run the app. You should see:

- A "Verify your phone" title
- A pre-filled phone number field (from `DEFAULT_PHONE` in `local.properties`)
- A "Start verification" button

Tapping the button does nothing yet — that's expected. In the next step you'll wire it to the backend.

---

## Checkpoint

- [ ] `VerificationScreen` implemented with all four state branches
- [ ] Phone input visible; SMS input only visible when state is `EnterSms`
- [ ] Inputs disabled during `Loading`
- [ ] Build succeeds and app shows the phone entry screen
