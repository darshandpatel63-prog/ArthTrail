# ArthTrail Vault Lock — Design

Status: design document only. Do not treat it as implemented until code and tests exist.

## Goal
Require a separate user-created ArthTrail secret before the local finance vault can be opened. It is independent from Firebase login. The secret and key material must never go to a server, analytics service, ad SDK or log.

## User flow
1. After first account sign-in (or local-only mode if added later), show a clear explanation that vault data is encrypted locally.
2. User creates an app-specific passphrase or strong PIN and confirms it. Recommend a longer passphrase; never encourage a 4-digit PIN for a high-value finance vault.
3. Explain honestly: Firebase password reset will not reset this lock. Without the vault secret or an independently stored recovery key, encrypted data may be permanently lost.
4. Offer optional device biometrics after the secret is set up and key wrapping has passed tests.
5. Lock after the app is backgrounded or inactive, after the configured timeout, and after process restart.
6. On each unlock, verify the user locally, unwrap/use the vault key through the platform-protected key path, and only then open the database.

## Native Android design
- Build the Android app with Flutter native builds. No Capacitor, no Web App and no WebView wrapper.
- Use a reviewed SQLCipher-compatible SQLite package in a native Flutter build. Verify the database cannot be read from copied raw files without the key.
- Generate a cryptographically random vault key. Protect/wrap it with a key derived from the user secret using a reviewed KDF and a unique random salt. Do not use the short PIN directly as a database encryption key.
- Keep the unwrapping key material in Android Keystore-backed storage where supported. Require user authentication for key use where the platform configuration supports it.
- Biometric unlock must use Android's system prompt / supported local-auth API. Biometrics do not mean fingerprint data is read or stored by ArthTrail.
- Apply progressive delays after repeated failures. Do not add an insecure 'reset lock' shortcut that decrypts the old vault from Firebase.

## Other native platforms
- The first implementation targets Android. Later native targets must use the operating system's appropriate secure key store and platform authentication APIs.
- Do not claim identical hardware-backed guarantees across platforms until each native target has been tested.
- A browser application is not in scope and must not be built as a fallback.
## True passkeys vs local app lock
A FIDO/WebAuthn passkey is a public-key authentication credential stored by a credential provider and validated with a server challenge. A user-typed app PIN/passphrase is not a FIDO passkey. ArthTrail's immediate requirement is the separate local Vault Lock. True platform-passkey sign-in may be added later only after a native-only provider/device flow has been verified.

## Recovery and backup
- Backup has its own password/key; do not silently reuse the local app secret.
- Include the encrypted vault/archive only after authentication-tag and schema validation.
- If user changes their Vault Lock while the vault is unlocked, re-wrap the data key using a new salt/KDF output and verify the new lock before retiring the previous wrapped key.
- Do not put the Vault Lock secret, derived key or local authentication session into backup archives.

## Required tests
- Correct passphrase succeeds; wrong passphrase never opens the database.
- Empty, weak, repeated and malformed secrets are handled safely.
- Rate limiting works across repeated attempts and cannot be reset simply by navigating between screens.
- Lock triggers on background, inactivity and restart.
- Biometrics unavailable, cancelled, failed or invalidated falls back safely.
- Firebase password reset does not open the vault.
- Raw database copied to another device is unreadable without valid authorized key material.
- App logs/network requests contain no Vault Lock secret or key.
- Backup recovery and Vault Lock rotation do not corrupt the only copy of the vault.