# 🛡️ VaultGuard — Privacy-First Android AppLock

VaultGuard is a premium, offline-first application locking and privacy suite for Android. Built with a modern, modular Clean Architecture and Jetpack Compose UI, VaultGuard allows users to protect apps behind multiple auth methods, setup geofenced safe zones, vault private notifications, capture intruder selfies, and even break addictive habits with a built-in Doomscroll Guard.

---

## 🚀 Key Features

### 🔒 Core Security & App Interception
* **⚡ Blazingly Fast Interception**: Foreground applications are intercepted and overlayed in **under 150ms** using a highly optimized background `AccessibilityService`.
* **🧠 In-Memory Caching**: Implements a reactive, thread-safe memory cache of settings, configurations, and rules, reducing active should-lock evaluation times to **under 1ms** (pure in-memory logic).
* **⏱️ Dynamic Time-Sync PINs**: Hardens security against shoulder surfing and screen recording by syncing your master PIN to the current system clock (e.g. `14:35` -> `1435`).
  * *Time + Offset:* Shift the PIN minutes forward/backward (e.g. time + 10 mins).
  * *Time Reversed:* Reverse clock digits for offset syncing (e.g. `14:35` -> `5341`).
  * *Clock Formats:* Choose 12-hour or 24-hour clock formats (e.g., noon formats to `1200` or `0100`). Acceptance fallback evaluates both to prevent lockouts.
* **📬 Master PIN Recovery Options**: Securely regain access if you forget your master PIN.
  * *Recovery Key:* Standard offline backup key generated during setup.
  * *Recovery Email:* Enter a pre-configured recovery email in the tabbed Forgot PIN prompt to reset your master credentials instantly.
* **📍 Smart Lock (Secure Zones)**: Automatically bypasses lock overlay screens when connected to a trusted Wi-Fi network or inside defined GPS coordinates. Requests permissions dynamically in a modern two-step runtime flow (Foreground -> Background location).
* **⏱️ Flexible Lock Timeouts**: Configure session lock rules (Immediate, 15 seconds, 1 minute, etc.). Smart re-evaluation on screen off and screen on ensures apps are securely re-locked when the timeout expires.
* **🎭 Multi-Profile & Decoy Mode**: Setup up to 5 profiles. Supports a silent "Duress PIN" that switches the system to a pre-defined decoy profile to hide sensitive settings.
* **⛔ Advanced Self-Protection**: Prevents users from disabling admin rights or uninstalling/force-stopping the app without entering the correct master PIN.
* **🌐 Offline & Private**: 100% offline with zero network permissions requested.

### 🧠 Mindfulness & Doomscroll Interrupters
* **🛑 Doomscroll Guard**: Breaks addictive scrolling habits. Configure screen time limits for individual locked apps (e.g., 10 minutes for Instagram). Once reached, the app is locked, forcing a focus-restoring exercise to unlock:
  * **💨 Breathing Exercise**: Guided deep breathing cycle with animated circle scaling.
  * **✍️ Focus Typing**: Type mindful affirmations or sentences to regain access.
  * **🧮 Math Puzzle**: Solve rapid mental arithmetic to bypass the lock screen.

### 📸 Proactive Surveillance
* **📸 Silent Intruder Selfie**: Captures a snapshot of the person attempting access using the front camera after a pre-configured number of failed authentication attempts, saving it locally in a secure image vault.

### 🎨 Visual & Theme Customization
* **🎨 Curated Lock Screen Themes**: Move away from boring black lock overlays. Choose from four premium visual effects:
  * **Ambient Shader**: The default ultra-smooth, color-shifting ambient shader background.
  * **Glassmorphism**: Sleek frosted dark glass with animated, floating blue and pink canvas-drawn orbs.
  * **Neon Glow**: High-contrast rotating gradient pink-purple-cyan-orange glow.
  * **Meme Mode**: Display sarcastic, humorous warning card memes to deter snoopers.

---

## 📂 Architectural Layout

VaultGuard is structured into highly cohesive, decoupled modules conforming to **Clean Architecture** principles:

```
app/          — Jetpack Compose UI Screens, ViewModels, and App Entry Point
auth/         — Hashing, Lockout State Manager, and Verification Use Cases
locker/       — AccessibilityService, Window Overlay UI, and Timeout Evaluation engine
profiles/     — Profile models and storage configuration
data/         — Room Database implementation, DataStore repository bindings
domain/       — Use Case interfaces, domain repositories, and core Models
common/       — Thread-safe cryptography tools, security constants, and results wrappers
```

---

## 🔒 Advanced Self-Protection Architecture

To prevent unauthorized uninstallation, data clearing, or deactivation of VaultGuard, the app utilizes a custom **Multi-Layered Protection System** designed to counter platform-level bypasses:

1. **Active Settings Ejection (Layer 1)**:
   * Android OS enforces a platform security policy (`mForceHideNonSystemOverlayWindow=true`) on the Settings app to prevent tapjacking. This makes standard alert windows invisible on Settings screens.
   * To counter this, `AppLockService` actively detects when an unauthorized user attempts to open Settings -> App Info or Device Admin screens for VaultGuard.
   * It immediately sends a global `GLOBAL_ACTION_BACK` to eject the user from the Settings app, waits for a `300ms` transition delay, and then launches our `MainActivity` PIN verification screen to cover the screen.
2. **Deactivation Request Warning (Layer 2)**:
   * If the deactivation intent is requested, `VaultGuardAdminReceiver.onDisableRequested` intercepts the removal attempt and returns a custom warning message to the OS prompt: *"Deactivating VaultGuard administrator requires PIN verification. Please verify your PIN in the app first."*
3. **Re-activation Recovery (Layer 3)**:
   * In case of unauthorized deactivation (bypassing callbacks), `VaultGuardAdminReceiver.onDisabled` detects the state and immediately launches the system-level Device Admin activation screen (`ACTION_ADD_DEVICE_ADMIN`) to force re-activation.

### Authorized Bypass
Authorized admins can easily configure settings or uninstall the app by entering their PIN in the app. This opens a secure **15-second grace window** during which all Settings actions and deactivation attempts are permitted.

---

## 🛡️ Security & Cryptography

* **PIN Storage**: Credentials are never written in plaintext. PINs are salted with a cryptographically secure 16-byte salt (`SecureRandom`) and hashed using `Argon2id` (falling back to OWASP-compliant `PBKDF2-HMAC-SHA256` where unavailable).
* **Biometrics**: Integrates with hardware-backed Android Keystore signatures (`BiometricPrompt` Class 3 validation).
* **Database Security**: Local profiles, configuration, and audit logs are encrypted.

---

## 🛠️ Build & Setup

### Prerequisites
* Android Studio Koala / Ladybug or newer.
* Android SDK 26 (Android 8.0) minimum, targeting SDK 35 (Android 15 / API 35).

### Compilation & Running
1. Clone the repository and open in Android Studio.
2. Build the project debug APK:
   ```bash
   ./gradlew assembleDebug
   ```
3. Run unit and integration tests:
   ```bash
   ./gradlew test
   ```
4. Deploy the debug build directly to an emulator or physical device.

---

## 📋 Permissions Required
To function reliably, VaultGuard requires the following critical Android permissions:
1. **Accessibility Service**: Allows foreground package detection and active ejection.
2. **Display Over Other Apps (`SYSTEM_ALERT_WINDOW`)**: Required to render the lock overlay in a secure window.
3. **Device Admin**: Active permission to prevent unauthorized uninstallation of VaultGuard.
4. **Biometric Authentication (`USE_BIOMETRIC`)**: Required to query and authenticate via hardware biometric sensors.
5. **Location Access (`ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION`)**: Required to scan for trusted Wi-Fi SSIDs and verify GPS geofences for Smart Lock.
6. **Background Location (`ACCESS_BACKGROUND_LOCATION`)**: Required to check safe zones when the app monitors foreground app switches in the background.
7. **Wi-Fi State (`ACCESS_WIFI_STATE`, `CHANGE_WIFI_STATE`)**: Required to read the connected network SSID for Trusted Wi-Fi lock bypass.
8. **Notification Access (`BIND_NOTIFICATION_LISTENER_SERVICE`)**: Required to intercept and vault notifications for locked applications.
9. **Camera (`android.permission.CAMERA`)**: Required to capture front photos of intruders on failed authentication attempts.
