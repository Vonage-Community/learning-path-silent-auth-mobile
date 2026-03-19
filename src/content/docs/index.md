---
title: "What We're Building"
description: "A high-level look at the 2FA with Silent Auth — what it does, how it works, and what you'll build across this tutorial."
---

In this tutorial you'll build a complete **two-factor authentication (2FA) system** using [Vonage Verify v2](https://developer.vonage.com/en/verify/overview). It uses **Silent Authentication** as the primary channel — no OTP required when it works — and falls back to **SMS** when it can't.

By the end you'll have:

- A **Node.js backend** that manages the entire verification flow
- An **Android app** (Kotlin + Jetpack Compose) that drives the user experience
- A working end-to-end test confirming both paths

---

### Why two channels?

| Channel | How it works | User experience |
|--------|-------------|-----------------|
| **Silent Authentication** | Vonage checks the phone's mobile network connection — no action from the user | Invisible |
| **SMS OTP** | Vonage sends a one-time code by SMS — user types it in | Familiar fallback |

---

## Design principle: the backend owns everything

The Android app never calls Vonage directly. It never stores API keys. It only talks to **your** backend, which in turn talks to Vonage. This keeps secrets off the device and puts all business logic in one place.

```
Android App  ──→  Your Backend  ──→  Vonage Verify API
                       ↑
                  Vonage Callback (webhook)
```

---

## What you'll need

### Backend (GitHub Codespaces)

This tutorial runs in **GitHub Codespaces**. The backend terminal, nodemon, Node.js, and npm are all pre-configured — no local installation needed.

> **nodemon** is already installed in the Codespaces environment. It watches `server.js` and restarts the server automatically every time you save the file. You'll never need to manually stop and restart the server during this tutorial.

You will need:

- A [Vonage developer account](https://developer.vonage.com/sign-up) (free tier is fine)
- A phone number in **E.164 format** (e.g. `+34600111222`) to perform the verification

### Android app (local machine)

- [Android Studio](https://developer.android.com/studio) (stable channel)
- A real Android device running **Android 7.0+** (API 24 or higher) with a SIM and mobile data

> **Why a real device?** Silent Authentication relies on the mobile carrier network context. An emulator cannot replicate this. You can still use an emulator to test the SMS path, but Silent Auth will always fall back to SMS on an emulator.

---

## Tutorial structure

| # | Page | What you'll do |
|---|------|----------------|
| 01 | New Project | Create project in Android Studio |
| 02 | Dependencies | Add all required libraries |
| 03 | Configuration | Permissions, backend URL |
| 04 | Verify Setup | Minimal counter app smoke test |
| 05 | UI States | State machine for the UI |
| 06 | UI Layout | Full verification screen |
| 07 | Networking | Backend API calls |
| 08 | Silent Auth | Vonage Client SDK integration |
| 09 | End-to-End Testing | Full flow, common issues |

