---
title: "End-to-End Testing"
description: "Run through all three verification scenarios, use the common issues table to diagnose problems, and explore what to build next."
step: 9
---

The backend and app are built. This final step walks you through three test scenarios, explains how to diagnose the most common failures, and points to where you can take this further.

---

## Prerequisites checklist

Before starting, confirm everything is in place:

### Backend (Codespaces)
- [ ] `nodemon server.js` is running in a terminal
- [ ] Port 3000 is forwarded with **Public** visibility in the Codespaces Ports panel
- [ ] The public URL (e.g. `https://your-codespace-3000.app.github.dev`) is set in `local.properties` as `BACKEND_URL`
- [ ] `.env` contains valid `VONAGE_APP_ID` and `VONAGE_PRIVATE_KEY` values
- [ ] The callback URL in the Vonage Dashboard is set to `https://your-codespace-url/callback`

### Android App
- [ ] The app is installed on a real Android device (API 24+)
- [ ] The device has an active SIM with mobile data
- [ ] `local.properties` has the correct `BACKEND_URL` and `PHONE_NUMBER`

---

## Backend smoke test

Before running the full flow in the app, confirm the backend is reachable from outside Codespaces:

```bash
curl https://your-codespace-3000.app.github.dev/health
```

Expected:

```json
{ "status": "ok" }
```

If this fails, the port isn't forwarded publicly. Check the Ports tab in Codespaces.

---

## Scenario 1: Silent Authentication — successful path

**Device requirement**: Real Android device with mobile data enabled (Wi-Fi can also be on as the SDK forces the cellular route).

**Steps**:
1. Open the app
2. Confirm the phone number field shows your number from `local.properties`
3. Tap **Start verification**
4. Watch the status message. It should show "Attempting Silent Authentication..."
5. After a few seconds (2–5 seconds typically), the screen shows "Verified via Silent Authentication"

**No SMS should arrive.** The verification completed without user input.

**Backend logs** (in the nodemon terminal) should show:

```
Received verification request for: +34600111222
Vonage newRequest result: { requestId: '...', checkUrl: 'https://...' }
Checking code for request: ...
Vonage checkCode result: completed
Callback received: { request_id: '...', status: 'completed' }
```

---

## Scenario 2: SMS Fallback — automatic

**Trigger**: Device does not support Silent Auth (e.g. carrier not enrolled in Vonage network registry).

In this case, the backend returns `check_url: null` and the app skips directly to the SMS input.

**Steps**:
1. Tap **Start verification**
2. The status message should immediately say "Silent Authentication is not available. Please enter the SMS code."
3. You receive an SMS with a 4–6 digit code
4. Enter the code and tap **Submit code**
5. The screen shows "Verified via SMS"

---

## Scenario 3: Forced SMS Fallback

**Trigger**: Silent Auth is available (`check_url` returned) but the attempt fails (e.g. Wi-Fi only, SDK error, timeout).

To test this manually, disable mobile data on your device while keeping Wi-Fi on before tapping **Start verification**.

**Steps**:
1. Turn off mobile data (keep Wi-Fi on)
2. Tap **Start verification**
3. The app attempts Silent Auth over mobile data, but mobile data is off, so the SDK fails
4. The status message says "Silent Authentication failed. Please enter the SMS code."
5. You receive an SMS with the code
6. Enter the code and tap **Submit code** → "Verified via SMS"

---

## Common issues

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| `curl /health` returns connection refused | Port not forwarded / server not running | Check nodemon is running; set port 3000 to Public in Codespaces Ports panel |
| `start verification failed: HTTP 401` | Invalid credentials | Double-check `VONAGE_APP_ID` and `private.key` in `.env` |
| `start verification failed: HTTP 422` | Phone number format | Use E.164 format: `+34600111222` (include country code, no spaces) |
| `check_url` is always `null` | Carrier not in Vonage network registry | Normal for many carriers — SMS fallback is the expected path |
| Silent Auth always fails | Mobile data off or emulator | Test on a real device with mobile data. Emulators cannot do Silent Auth. |
| SMS code never arrives | Rate limiting or wrong phone number | Wait 5 minutes and try again; confirm the number format is correct |
| App crashes on `startCellularGetRequest` | SDK not initialised | Make sure `VGCellularRequestClient.initializeSdk(this.applicationContext)` is called in `onCreate` |
| Backend logs show no callback received | Callback URL not configured or not public | Set the callback URL in Vonage Dashboard; confirm the Codespaces port is Public |
| Callback received but status is `expired` | Verification timed out (10 min limit) | Start a fresh verification request |

---

## Debugging tips

### Check the server logs

The `nodemon` terminal shows every incoming request and Vonage SDK response. When something goes wrong, it's usually logged there:

```
Callback received: { request_id: '...', status: 'expired' }
```

### Android Logcat

In Android Studio, open **Logcat** (bottom panel) with your device selected. Filter by your package name (`com.vonage.verify2.test`) to see any exceptions thrown by the networking code.

---

## What's next

You've built a complete 2FA system with Silent Authentication and SMS fallback. Congratulations! Here are some directions to explore:

### Production hardening

- **Persist the verification store** — replace the in-memory `Map` with Redis or a database. Right now, restarting the server loses all in-progress verifications.
- **Add rate limiting** — prevent brute-force code guessing by limiting `/check-code` attempts per `request_id`
- **Clean up expired entries** — add a `setInterval` that removes store entries older than 10 minutes

### Vonage Verify features

- **Voice fallback** — add `{ channel: "voice", to: phone }` as a third workflow step for users who can't receive SMS
- **Custom expiry** — set `codeExpiry` on the `newRequest` call (max 15 minutes)
- **Fraud check** — enable Vonage's [SIM swap](https://developer.vonage.com/en/identity-insights/overview) before starting the verification

### Infrastructure

- **Deploy to production** — replace Codespaces with a persistent Node.js host
- **Secure the backend** — add authentication to your API endpoints so only your Android app can call them

