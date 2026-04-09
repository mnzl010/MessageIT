# MessageIt 💬🔒

> A secure Android messaging application built with Kotlin, Firebase, and modern Android architecture — developed as a university cybersecurity assignment.

![Android](https://img.shields.io/badge/Android-API%2026%2B-brightgreen) ![Kotlin](https://img.shields.io/badge/Kotlin-1.9-purple) ![Firebase](https://img.shields.io/badge/Firebase-Auth%20%2B%20Firestore-orange) ![Architecture](https://img.shields.io/badge/Architecture-MVVM-blue) ![Encryption](https://img.shields.io/badge/Encryption-AES--256--GCM-red)

---

## What is MessageIt?

MessageIt is a fully functional Android messaging app where security is not an afterthought — it is the foundation. Every design decision, from how passwords are stored to how messages travel between users, was made with a specific threat in mind.

The app demonstrates real-world implementation of cybersecurity principles including end-to-end encrypted messaging, secure authentication, brute-force protection, and encrypted local storage — all built from scratch in Kotlin.


---

## Features

### Core Features
- **User Registration and Login** — email/password with full validation
- **Real-time Messaging** — one-to-one encrypted chat using Firestore listeners
- **Post Feed** — create text and image posts, like and react
- **Photos Section** — image-only feed with double-tap to like, long-press for emoji reactions
- **Friend Search** — search users by username to start conversations
- **Profile Management** — profile picture, display name, security alerts, preferences
- **Dark Mode** — system-wide instant toggle from the profile screen

### Security Features
- **Password Strength Validator** — algorithmic scoring system with live feedback
- **CAPTCHA Verification** — arithmetic challenge on login and registration
- **Rate Limiting** — account lockout after 5 failed attempts for 30 minutes
- **Email Verification Gate** — unverified accounts cannot log in, ever
- **End-to-End Encryption** — AES-256-GCM on every message
- **Encrypted Local Storage** — EncryptedSharedPreferences with hardware-backed keys
- **Input Sanitization** — all user input cleaned before touching the backend
- **Biometric Authentication** — fingerprint / face unlock for returning users
- **Google Sign-In** — OAuth 2.0 with cryptographic nonce via Credential Manager
- **Firestore Security Rules** — server-side access control enforced by Google
- **Back Stack Protection** — prevents navigation bypass after logout

---

## Security Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   DEVICE LEVEL                          │
│  allowBackup=false  │  back stack cleared  │  biometric │
├─────────────────────────────────────────────────────────┤
│                   INPUT LEVEL                           │
│  InputSanitizer  │  PasswordValidator  │  CaptchaHelper │
├─────────────────────────────────────────────────────────┤
│                AUTHENTICATION LEVEL                     │
│  Firebase bcrypt  │  Email verification  │  Rate limit  │
│  Google OAuth 2.0 │  Credential Manager  │  Nonce check │
├─────────────────────────────────────────────────────────┤
│                   DATA LEVEL                            │
│  AES-256-GCM E2E  │  EncryptedSharedPrefs │  Firestore  │
│  encryption       │  AES-256-GCM keys     │  rules      │
└─────────────────────────────────────────────────────────┘
```

### Password Validation

Passwords are scored using a 7-rule algorithm (0–100 points):

| Rule | Points |
|------|--------|
| Minimum 8 characters | 20 |
| 12+ characters bonus | 10 |
| Contains lowercase letter | 15 |
| Contains uppercase letter | 15 |
| Contains digit | 15 |
| Contains special character | 20 |
| No common patterns (password, 123456, qwerty...) | +5 / −20 |

Scores below 40 → **Weak** (blocked). 40–70 → **Medium**. Above 70 → **Strong**.

### End-to-End Encryption

Every message is encrypted on the sender's device before it reaches Firestore. Firebase cannot read message content.

```
Sender device                 Firebase              Recipient device
─────────────                 ────────              ────────────────
plaintext "Hello"             stores only           ciphertext → decrypt
     ↓                        ciphertext                  ↓
AES-256-GCM encrypt    →    + IV (stored)    →    AES-256-GCM decrypt
     ↓                                                     ↓
ciphertext "x7kP9..."                             plaintext "Hello"
```

- **Algorithm:** AES-256-GCM (confidentiality + integrity)
- **IV:** Fresh 12-byte random value per message — same message encrypts differently every time
- **Key derivation:** Deterministic from chat ID — both participants derive the same key independently
- **Integrity:** GCM authentication tag detects any message tampering in transit

---

## Tech Stack

| Category | Technology |
|----------|-----------|
| Language | Kotlin 1.9 |
| Architecture | MVVM (Model-View-ViewModel) |
| Authentication | Firebase Authentication |
| Database | Cloud Firestore |
| Image Storage | Cloudinary (free tier) |
| Image Loading | Glide 4.16 |
| Local Encryption | EncryptedSharedPreferences (AES-256-GCM) |
| Message Encryption | javax.crypto — AES/GCM/NoPadding |
| Biometric | AndroidX Biometric API (Class 3 BIOMETRIC_STRONG) |
| Google Sign-In | Credential Manager + GetGoogleIdOption |
| Navigation | Navigation Component |
| UI | Material Design 3, ViewBinding |
| Minimum SDK | API 26 (Android 8.0) |
| Target SDK | API 34 (Android 14) |

---

## Project Structure

```
app/src/main/java/com/messageit/
│
├── activities/
│   ├── SplashActivity.kt          # Entry point, session check, back-stack protection
│   ├── AuthActivity.kt            # Container for login/register flow
│   └── MainActivity.kt            # Container for main app with bottom nav
│
├── fragments/
│   ├── auth/
│   │   ├── LoginFragment.kt       # Login UI, biometric card, Google button
│   │   ├── RegisterFragment.kt    # Registration with live strength bar
│   │   ├── ForgotPasswordFragment.kt
│   │   └── VerifyEmailFragment.kt
│   ├── home/
│   │   ├── HomeFragment.kt        # Post feed
│   │   ├── MessagesFragment.kt    # Conversations list
│   │   └── PhotosFragment.kt      # Image feed
│   ├── chat/
│   │   └── ChatFragment.kt        # Real-time encrypted chat
│   └── profile/
│       └── ProfileFragment.kt     # Settings, security alerts
│
├── viewmodels/
│   ├── AuthViewModel.kt           # All auth logic, state management
│   ├── HomeViewModel.kt
│   ├── ChatViewModel.kt           # Message listener lifecycle
│   ├── MessagesViewModel.kt
│   ├── PhotosViewModel.kt
│   └── ProfileViewModel.kt
│
├── repositories/
│   ├── AuthRepository.kt          # Firebase Auth calls
│   ├── UserRepository.kt          # Firestore user documents
│   ├── MessageRepository.kt       # Encrypted send/receive
│   └── PostRepository.kt          # Posts with sanitization
│
├── models/
│   ├── User.kt
│   ├── Message.kt                 # Contains encryptedContent, iv fields
│   ├── Post.kt
│   ├── Chat.kt
│   └── SecurityEvent.kt
│
├── adapters/
│   ├── MessageAdapter.kt          # Two ViewTypes: sent / received
│   ├── PostAdapter.kt
│   ├── PhotoAdapter.kt            # Double-tap like, long-press reactions
│   ├── ChatAdapter.kt
│   └── UserSearchAdapter.kt
│
└── utils/
    ├── PasswordValidator.kt       # 7-rule scoring algorithm
    ├── CaptchaHelper.kt           # Math challenge generation/validation
    ├── RateLimiter.kt             # 5 attempts → 30min lockout
    ├── InputSanitizer.kt          # XSS and injection prevention
    ├── EncryptionHelper.kt        # AES-256-GCM E2E encryption
    ├── BiometricHelper.kt         # Fingerprint/face authentication
    ├── GoogleAuthHelper.kt        # OAuth 2.0 with nonce
    ├── PasskeyHelper.kt           # FIDO2/WebAuthn passkey support
    ├── SessionManager.kt          # EncryptedSharedPreferences wrapper
    ├── CloudinaryHelper.kt        # Image upload to Cloudinary
    └── Constants.kt               # Security thresholds and config
```

---

## Getting Started

### Prerequisites

- Android Studio Ladybug (2024.2) or newer
- JDK 17 (bundled with Android Studio)
- Android SDK API 34
- A device or emulator running Android 8.0 (API 26) or higher
- Internet connection for Firebase and Gradle

---

### Installation — Run the APK directly

The fastest way — no Android Studio setup needed.

**Download the APK:**

👉 **[Download app-debug.apk](YOUR_GITHUB_RELEASE_OR_RAW_LINK_HERE)**

**Install on a physical device:**
1. Transfer the APK to your Android device
2. Go to **Settings → Security → Install unknown apps** → enable
3. Tap the APK file → **Install**
4. Open **MessageIt** from your app drawer

**Install on an emulator:**
1. Start any Android Studio emulator (API 26+)
2. Drag and drop the APK file onto the emulator window
3. Open **MessageIt** from the app drawer

---

### Installation — Build from Source

**Step 1 — Clone the repository**

```bash
git clone https://github.com/YOUR_USERNAME/MessageIt.git
cd MessageIt
```

**Step 2 — Open in Android Studio**

File → Open → select the cloned `MessageIt` folder → OK

Wait for Gradle sync to complete.

**Step 3 — Configure local.properties**

Create a file called `local.properties` in the project root (if it doesn't exist) and add:

```properties
sdk.dir=/path/to/your/Android/Sdk
CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
CLOUDINARY_UPLOAD_PRESET=messageit_uploads
GOOGLE_WEB_CLIENT_ID=your_web_client_id_here
```

> `google-services.json` is already included in `app/` — Firebase is pre-configured.

**Step 4 — Sync and Run**

Click **Sync Now** when prompted, then press **Run** (Shift+F10).

---

### Cloudinary Setup (for image uploads)

1. Create a free account at [cloudinary.com](https://cloudinary.com)
2. From the dashboard, copy your **Cloud Name**
3. Go to **Settings → Upload → Upload presets → Add upload preset**
4. Set **Signing mode** to **Unsigned**, name it `messageit_uploads`
5. Add both values to `local.properties`

---

### Firebase Setup (if using your own project)

1. Create a project at [console.firebase.google.com](https://console.firebase.google.com)
2. Enable **Authentication** → Email/Password and Google
3. Enable **Firestore Database** in test mode
4. Register your Android app with package name `com.messageit`
5. Add your **SHA-1 fingerprint** (required for Google Sign-In)
6. Download `google-services.json` → place in `app/` folder

**Get your SHA-1:**
```bash
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```

**Firestore Security Rules:**
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
      match /securityEvents/{eventId} {
        allow read, write: if request.auth != null && request.auth.uid == userId;
      }
    }
    match /posts/{postId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && request.resource.data.authorId == request.auth.uid;
      allow update: if request.auth != null;
      allow delete: if request.auth != null && resource.data.authorId == request.auth.uid;
    }
    match /chats/{chatId} {
      allow read: if request.auth != null && request.auth.uid in resource.data.participantIds;
      allow create: if request.auth != null;
      allow update: if request.auth != null && request.auth.uid in resource.data.participantIds;
      match /messages/{messageId} {
        allow read: if request.auth != null;
        allow create: if request.auth != null && request.resource.data.senderId == request.auth.uid;
      }
    }
  }
}
```

---

## Creating an Account

1. Open the app and tap **Register**
2. Enter a username, email, and a strong password (watch the strength bar)
3. Solve the CAPTCHA
4. Tap **Create Account**
5. Check your inbox for a verification email from Firebase
6. Click the verification link
7. Return to the app and sign in

> The app will refuse login for any account that has not verified their email. This is intentional.

---

## Security Design Decisions

### Why MVVM for security?

Separating the UI from business logic means security checks live in the ViewModel and Repository layers — not in the Fragment. Even if UI code is modified, the validation still runs underneath. The Fragment cannot bypass the ViewModel.

### Why EncryptedSharedPreferences?

Standard SharedPreferences stores data as plain XML at `/data/data/com.messageit/shared_prefs/`. On a rooted device, any app can read this. EncryptedSharedPreferences encrypts both keys and values using AES-256-GCM. The master key is stored in the Android Keystore — a hardware-backed secure enclave — and never enters application memory.

### Why AES-256-GCM for messages?

GCM mode provides both confidentiality (content is hidden) and integrity (tampering is detected). The authentication tag generated during encryption is verified during decryption — a modified ciphertext fails decryption entirely. Combined with a fresh IV per message, this prevents both content exposure and pattern analysis.

### Why not just use Firebase's built-in security?

Firebase provides server-side security. MessageIt adds client-side security on top. Even if Firebase were compromised — message content would still be unreadable because it's encrypted before it leaves the device. Defence in depth means no single point of failure.

---

## Known Limitations

| Limitation | Context |
|-----------|---------|
| CAPTCHA is arithmetic-based | Advanced bots could solve it. Production would use reCAPTCHA v3. Acknowledged in technical report. |
| Rate limiting is client-side | Can be reset by clearing app data. Server-side enforcement requires Cloud Functions. |
| Biometric gates session, not Firebase | If session expires, biometric alone cannot re-authenticate. User must use password. |
| E2EE key derivation is simplified | Production would use Diffie-Hellman key exchange (Signal Protocol / X3DH). |

---

## Acknowledgements

Built as part of a university cybersecurity assignment. All security implementations are for educational purposes demonstrating core cryptographic and authentication principles.

Libraries used: Firebase Android SDK, Glide, AndroidX Biometric, AndroidX Credentials, Google Identity, OkHttp, Material Design Components.

---

## License

This project is submitted for academic assessment and is not licensed for commercial use.

---

*Made by Manjil Bhattarai*
