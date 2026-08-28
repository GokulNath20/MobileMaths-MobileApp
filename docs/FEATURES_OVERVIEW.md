# Features Overview & User Flows

This document details the functional specifications and workflows built into **MobileMaths**.

---

## 🔐 1. Authentication & Onboarding Flow

- **Secure Login**:
  - Email and password validation with case normalization.
  - "Remember Me" persistent token caching.
  - Graceful handling of invalid credentials and network timeouts.
- **Robust Sign Up**:
  - Multi-field input validation (Username, Full Name, Phone, Email, Password, Confirm Password).
  - Strict password policy enforcement (8+ characters, minimum 1 uppercase letter, 1 symbol).
  - Mandatory Terms of Service and Privacy Policy checkbox consent.
- **Email Verification & Resend Throttling**:
  - Intercepts unverified accounts upon login and routes them to a dedicated verification screen.
  - Intelligent resend cooldown timer (60-second lockout) persisted across app foreground/background cycles.
  - Deep-link listeners for automatic session resumption upon email link verification.
- **Password Reset**:
  - Secure dispatch of reset instructions via Firebase Auth.
  - In-app confirmation dialogs and navigation reset.

---

## 🏠 2. Student Dashboard (`HomeScreen`)

- **XP & Level Progression**:
  - Visual circular/linear progress bar displaying accumulated XP and current student mastery level.
  - Dynamic XP formulas rewarding speed drill accuracy.
- **Quick Action Feature Cards**:
  - **AI Detect Assessment**: Direct launch into diagnostic evaluation.
  - **Speed Math Arena**: Instant entry into high-tempo timed math drills.
  - **Personalized E-Learning**: Direct access to recommended video modules.
- **Real-Time Notification Bar**:
  - Bell icon with reactive unread badge count updated via Zustand selector.
- **Avatar Preview Modal**:
  - Tap-to-expand avatar preview powered by locally primed image cache.
- **Pull-To-Refresh**:
  - Re-syncs Firestore user profiles, subscription states, and remote stats seamlessly.

---

## 🎯 3. AI Diagnostic Assessment (`AIDetectScreen`)

- **24-Question Structured Assessment**:
  - Divided into 4 distinct question sets designed to isolate algebraic arithmetic rules:
    - **Type A**: Standard subtraction (\(5 - 2\), \(13 - 8\), \(16 - 23\))
    - **Type B**: Negative minuend (\(-5 - 2\), \(-13 - 8\), \(-16 - 23\))
    - **Type C**: Negative subtrahend (\(5 - (-2)\), \(13 - (-8)\), \(16 - (-23)\))
    - **Type D**: Double negative operations (\(-5 - (-2)\), \(-13 - (-8)\), \(-16 - (-23)\))
- **Live Input Validation**:
  - Supports negative numbers, clears placeholder states, and prevents invalid submissions.
- **Diagnostic Result Computation**:
  - If a student makes even one mistake in a type group, that category is marked as needing remediation.
  - Generates comprehensive result summaries (e.g., *"Requires practice in Type B and Type D"*).
  - Persists completed assessment records with ISO timestamps to Firestore.
  - Emits in-app activity notifications.

---

## 📚 4. Personalized E-Learning & Video Player

- **Remediation Mapping**:
  - The E-Learning hub dynamically highlights which specific video modules are recommended based on the user's latest diagnostic findings.
- **Local Video Storage & Caching**:
  - Integrates `react-native-fs` to download lessons from Firebase Cloud Storage to the device sandbox.
  - Video progress tracking and auto-resumption from last-watched timestamps.
- **Custom Video Player**:
  - Powered by `react-native-video`.
  - Fullscreen toggle with orientation lock/unlock handler.
  - Custom playback controls (scrubber, play/pause, forward/rewind, time indicators).

---

## ⚡ 5. Speed Math Arena (`GameScreen`)

- **60-Second Timed Math Challenge**:
  - Real-time countdown timer with visual progress indicator.
  - Dynamically generated random integer arithmetic problems.
- **High-Throughput Input**:
  - Auto-submits answers when input matches expected length to optimize user flow.
  - Tactile color-coded answer chips for recent submissions.
- **XP & Streak Reward Engine**:
  - Multipliers based on accuracy streaks.
  - Updates total accumulated XP and calculates level thresholds.
- **Session Archiving**:
  - Compiles full session question arrays (question, user answer, correct answer, correctness boolean) and saves to Cloud Firestore under `questionArray`.

---

## 👤 6. Profile, Activity History & Settings

- **Student Performance Insights**:
  - Summary cards showing total games played, total diagnostic tests completed, and accuracy rates.
- **Recent Activity Feed**:
  - Timeline of completed speed math drills and diagnostic assessments with drill-down question views.
- **Edit Profile & Avatar Upload**:
  - Image picker integration (`react-native-image-picker`) with local image optimization and Firebase Storage upload.
- **Account Security**:
  - In-app password change modal with credential re-authentication.
  - Account deletion modal with safety confirmations.

---

## 🔔 7. Notifications Hub (`NotificationsScreen`)

- **Filterable Notification Feed**:
  - Category tabs: **All**, **System**, **Learning**, **Subscription**.
- **Status Management**:
  - Mark individual notifications as read.
  - "Mark all as read" batch update.
  - Clear notification history.
- **Interactive Routing**:
  - Tapping a notification navigates directly to the relevant destination (e.g. diagnostic results, E-Learning module, subscription checkout).
