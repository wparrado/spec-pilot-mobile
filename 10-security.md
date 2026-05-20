# 10 — Security

## OWASP Mobile Top 10 — Required Checks

| # | Category | Required action |
|---|----------|----------------|
| M1 | Improper Credential Usage | All secrets in Secure Storage (Keychain/Keystore). No hardcoded API keys. |
| M2 | Inadequate Supply Chain Security | All dependencies pinned to exact versions. `cargo audit` / `npm audit` / `pod-checksum` in CI. |
| M3 | Insecure Authentication / Authorization | Enforce PKCE for OAuth flows. Validate tokens server-side. |
| M4 | Insufficient Input/Output Validation | All Value Objects validate through `create()` → `Result`. No raw user input ever reaches domain. |
| M5 | Insecure Communication | Certificate pinning on all authenticated API endpoints. TLS 1.2+ enforced. |
| M6 | Inadequate Privacy Controls | No PII in logs, crash reports, or analytics. User IDs hashed. |
| M7 | Insufficient Binary Protections | Enable App Attest (iOS) / Play Integrity (Android) for high-value operations. |
| M8 | Security Misconfiguration | No debug flags in production build. All logging disabled in release. `INSTA_REQUIRE_SNAPSHOTS=1` in CI. |
| M9 | Insecure Data Storage | SQLite/CoreData/Room databases encrypted at rest if they contain sensitive data. No cache files with PII. |
| M10 | Insufficient Cryptography | Use platform-native crypto APIs. No custom crypto. AES-256-GCM for at-rest encryption. |

---

## Certificate Pinning

```swift
// Swift — URLSession delegate
class PinningDelegate: NSObject, URLSessionDelegate {
  private let pinnedSHA256 = "your-certificate-sha256-base64"

  func urlSession(_ session: URLSession, didReceive challenge: URLAuthenticationChallenge,
                  completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
    guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
          let serverTrust = challenge.protectionSpace.serverTrust,
          let certificate = SecTrustGetCertificateAtIndex(serverTrust, 0) else {
      completionHandler(.cancelAuthenticationChallenge, nil)
      return
    }
    let serverData = SecCertificateCopyData(certificate) as Data
    let serverHash = serverData.sha256Base64()
    if serverHash == pinnedSHA256 {
      completionHandler(.useCredential, URLCredential(trust: serverTrust))
    } else {
      completionHandler(.cancelAuthenticationChallenge, nil)
    }
  }
}
```

---

## Secure Storage Rules

1. **Keychain (iOS)**: Use `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`.
2. **Keystore (Android)**: Use `EncryptedSharedPreferences` with `MasterKey.Builder()`.
3. **Flutter**: Use `flutter_secure_storage` which wraps both platforms.
4. **React Native**: Use `react-native-keychain`.
5. Never store tokens, passwords, or keys in:
   - `UserDefaults` / `SharedPreferences`
   - SQLite / CoreData / Room (unless encrypted at rest AND the key is in Keychain/Keystore)
   - Log files
   - Clipboard (clear on background)

---

## Session Security

- Access tokens: short-lived (≤15 min). Stored in Secure Storage.
- Refresh tokens: long-lived. Stored in Secure Storage. Rotated on every refresh.
- On 401 with invalid refresh token: emit `AppEvent.SessionExpired` → navigate to login.
- On app background (after N minutes): require re-authentication for sensitive operations.

---

## Privacy & PII

- Crash reports: user identifier = hashed UUID (not email, not phone).
- Analytics events: contain only non-identifiable data (screen name, action name, no user data).
- On logout: clear all tokens from Secure Storage + clear sensitive DB records.
