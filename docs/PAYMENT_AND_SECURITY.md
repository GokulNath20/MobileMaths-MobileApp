# Payment Gateway Integration & Device Security

## 💳 HitPay Payment Architecture

MobileMaths integrates **HitPay Payment Gateway** to support Southeast Asian regional payment channels (PayNow QR, FPX, Credit/Debit Cards, e-Wallets).

### Payment Lifecycle Flow

```mermaid
sequenceDiagram
    autonumber
    actor User as Student / Parent
    participant App as Mobile App (React Native)
    participant CloudFn as Firebase Cloud Functions
    participant Firestore as Cloud Firestore
    participant HitPay as HitPay Gateway

    User->>App: Selects Subscription Plan
    App->>CloudFn: createHitpayPayment({ uid, planId, amount })
    CloudFn->>HitPay: POST /v1/payment-requests
    HitPay-->>CloudFn: Returns checkout URL & payment_request_id
    CloudFn->>Firestore: Writes pending transaction with 15-min expiry
    CloudFn-->>App: Returns checkout URL
    App->>User: Opens in-app WebView for Checkout
    
    alt User pays successfully via PayNow / Card
        User->>HitPay: Completes Payment
        HitPay->>CloudFn: POST /webhookHitpay (Signature verified)
        CloudFn->>Firestore: Updates transaction status: 'completed'
        CloudFn->>Firestore: Sets users/{uid}.subscription = true
        Firestore-->>App: Real-time listener detects subscription active
        App->>User: Unlocks full access & navigates to Home
    else User abandons checkout / Closes WebView
        User->>App: Dismisses WebView
        App->>App: Starts 10-Minute Abandoned Payment Watcher
        Note over App,Firestore: If no completed webhook within 10 mins, marks transaction 'expired'
    end
```

---

## 🛡 Abandoned Checkout Handling (QR & PayNow)

Dynamic QR payments (such as PayNow or DuitNow) cannot be programmatically canceled via API once generated. MobileMaths implements a dual-layer resolution strategy:

1. **Pending Status Initialization**:
   - Each transaction begins in `pending` state with a `paymentWindowExpiresAt` timestamp (now + 15 minutes).
2. **Client-Side 10-Minute Watcher**:
   - When the user exits the checkout WebView or foregrounds the app, a temporary 10-minute Firestore listener monitors the transaction.
   - If no `payment_request.completed` webhook arrives before the timeout, the mobile app marks the record as `status: 'expired'`, `failureReason: 'abandoned'`.
3. **Late-Payment Grace Mechanism**:
   - If the student pays in their banking application after the mobile watcher expires the transaction, the Cloud Function webhook handler still accepts and processes `payment_request.completed` webhooks, activating the subscription and updating the status to `completed`.

---

## 🔐 Webhook Authentication & Security

```javascript
// Verification Mechanism in Firebase Cloud Functions
const crypto = require('crypto');

function verifyHitpaySignature(payload, signatureHeader, webhookSalt) {
  const hmac = crypto.createHmac('sha256', webhookSalt);
  hmac.update(payload);
  const calculatedSignature = hmac.digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(calculatedSignature, 'utf8'),
    Buffer.from(signatureHeader, 'utf8')
  );
}
```

- **Per-Webhook Salt**: Uses the unique per-webhook endpoint salt rather than the standard API key salt to validate the `Hitpay-Signature` header.
- **Timing Safe Comparison**: Prevents timing attacks during HMAC signature verification.
- **Secret Manager**: Production credentials stored securely via Firebase Functions Secrets (`firebase functions:secrets:set`).

---

## 🔒 Hardware Device Binding

To prevent account and subscription sharing across multiple devices:

1. **Hardware Fingerprinting**:
   - Extracts unique device identifier using `react-native-device-info` (`getUniqueId()`, `getDeviceId()`, `getSystemName()`).
2. **Subscription Verification Gate**:
   - When a verified subscriber signs in, the device fingerprint is checked against `users/{uid}.deviceInfo`.
   - If a mismatch is detected, the app restricts access and informs the user that their active subscription is bound to their primary registered device.
3. **Device Migration Handshake**:
   - Allows users to re-bind their subscription to a new device through authenticated security confirmation.

---

## 🛡 Firebase App Check & Network Protection

- **Integrity Attestation**:
  - **Android**: Google Play Integrity API.
  - **iOS**: Apple DeviceCheck / App Attest.
- **Backend Enforcement**:
  - All Cloud Firestore read/write rules and Cloud Function callable endpoints verify App Check tokens before processing payloads.
