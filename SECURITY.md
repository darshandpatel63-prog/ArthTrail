# ArthTrail Security Policy and Threat Model

Security status: specification only. This document is not proof that the app is implemented securely or has passed a penetration test.

## 1. Security promise
ArthTrail must keep financial records on the user's device by default. There are no silent financial-data uploads. Data can leave the device only after a clear user action, for example encrypted backup to a chosen destination or an explicitly approved transfer/share to another device. Firebase is limited to account authentication and verification; no finance records are stored there.

## 2. Threat model
Protect against:
- A person casually opening an unattended/borrowed unlocked phone.
- Another signed-in user or workspace member trying to view or modify records outside their permissions.
- Theft of a device or copied local files/backup archives.
- Malformed or malicious statement imports, oversized files, parser abuse and duplicate transactions.
- Replayed, guessed, copied or expired QR invitations.
- A compromised dependency or unexpected network request trying to exfiltrate statement content.
- Leaked GitHub source/configuration, API keys, logs or crash reports.
- Accidental loss during backup restore, database migration or app update.

Residual risks to explain honestly:
- A rooted/compromised device, malware running with accessibility or screen-capture privileges, or a malicious OS can observe data while the vault is open.
- A Web App can be exposed by same-origin XSS or a compromised dependency; browser-only storage does not provide the same hardware-backed isolation as a native mobile key store.
- If users deliberately share or upload a backup, the recipient/selected provider receives the data or encrypted file respectively.
- Losing the vault secret and all backup/recovery material may make data permanently unrecoverable.

## 3. Separate authentication layers
### Account authentication
Use Firebase Authentication for sign-in identity, verified email, Google sign-in where configured and account recovery. Firebase must not be used as the financial database. Do not put sensitive finance fields into user display names, auth claims, analytics events or URL parameters.

### ArthTrail Vault Lock
- Create an app-specific passphrase or strong PIN that is different from the Firebase password.
- Require lock setup before allowing real financial records to be entered.
- Lock when app is backgrounded or inactive and after a configurable short timeout; require unlock after app process restart.
- Use constant-time verification where applicable; progressive delay and rate limiting after failures; no detailed failure reason that leaks secret state.
- Never log, transmit, store in plaintext or back up the vault secret.
- Provide biometric unlock only as a convenience secured by platform APIs and a proper wrapped-key design. If biometrics change or the OS invalidates the key, fall back to the local secret/recovery flow.
- A true WebAuthn/FIDO passkey is a cryptographic account-authentication credential, not the same as a typed local app-lock code. Do not call the PIN a FIDO passkey. Implement true passkey sign-in separately only after the server/provider path is verified.

## 4. Data encryption and key management
### Android
- Use an encrypted SQLite database through SQLCipher in a native build. Expo Go does not prove SQLCipher support; use a development/release build and test encryption end-to-end.
- Generate a random high-entropy database/vault key. Do not derive the database key directly from a short PIN.
- Protect key wrapping with Android Keystore-backed facilities/secure storage; where supported, require local device authentication to use/unlock the key.
- Keep keys out of AsyncStorage, plain SQLite, logs, GitHub, build logs, app constants and screenshots.
- On logout, lock and clear transient in-memory secrets; never silently delete the local vault unless the user confirms.

### Web
- Store data in IndexedDB and encrypt sensitive payloads with Web Crypto. Keep cryptographic keys non-exportable where appropriate and design passphrase-derived key handling carefully.
- Use a strict Content Security Policy, no remote third-party scripts in the finance origin, no eval/unsafe-eval, pinned/reviewed dependencies, output escaping and tests for XSS.
- Explain that same-origin script compromise could still access a vault while unlocked; browser security is not identical to a hardware-backed native key.

### Backup encryption
- Encrypt locally before file write or Drive upload with authenticated encryption (e.g. AES-256-GCM) and a reviewed password KDF with a unique random salt.
- Never upload backup password/derived key, or place them in the same archive.
- Verify authentication tag, version, checksum, size limits and schema before restore.
- Restore to temporary state; only replace/merge after validation and explicit confirmation. Roll back fully on failure.

## 5. Network and data-exfiltration policy
- Financial database rows, balances, statement contents, descriptions, receipt text and report data may not be sent to Firebase, Analytics, crash reporting, AI APIs or npm package services.
- Package installation downloads public code during development, but the app must not upload user financial records to package maintainers. Review package code, telemetry and network behavior.
- Keep dependency count low. Pin versions/lockfile, audit dependencies/licenses and review any package with network, analytics, OCR, file-system or cryptographic access.
- Maintain an explicit network destination inventory. Authentication endpoints are allowed only for auth; Google Drive network requests are allowed only after the user explicitly enables backup; sharing connections are allowed only for a confirmed invitation/session.
- Add tests that intercept/record app network requests while creating transactions and importing sample statements; assert no financial content or derived sensitive values appear in request URLs, headers, analytics or bodies.
- Avoid third-party analytics in the finance app. Do not put balances, category names, descriptions or file names in crash logs.
- App advertising must be isolated from finance screens. Never send financial data, workspace IDs, balance, descriptions or imported-file content to ad SDKs.

## 6. Local statement-import safety
- Process files entirely locally. Never submit statements to a web converter or AI OCR endpoint.
- Check file extension, MIME type, signatures, file size, archive expansion limits, row/column count, text length and parser errors.
- Use vetted parsers and fixed format allowlists. Treat every imported cell as untrusted data and prevent spreadsheet-formula injection in CSV exports.
- Parse into staging only; show field mapping and preview. Any ambiguous debit/credit direction or date convention requires confirmation.
- Deduplicate conservatively and make imports reversible. Never silently auto-post malformed or uncertain rows.

## 7. Workspaces, roles and QR invitations
- Every data query/action carries the current workspace scope and checks permission at the domain/data layer. Hiding a button is not authorization.
- Default-deny permissions. A manager has only the explicitly assigned capabilities; membership in one company does not grant access to another company or the owner's personal workspace.
- Use random, short-lived, single-use or limited-use invitation tokens. The QR contains only token/connection metadata; store only a token hash where the flow allows it.
- Confirm exact workspace, member identity, role and permissions on both sides. Log member/device additions, revocation, exports and sensitive changes without logging financial content.
- The owner must be able to revoke a device/member. Expired/revoked tokens must fail closed.
- Host mode visibly shows active sharing, connected devices, last activity and a Stop Sharing control.
- Remote mode must not be released until TLS/secure channel, peer identity, authorization, replay protection, endpoint restrictions, rate limiting, network exposure and cost/privacy consequences are tested.

## 8. Firebase boundary
Allowed: sign-in identity, verified-email status and account recovery/auth metadata necessary for Firebase Authentication.
Forbidden: transactions, balances, notes, imported statement rows, receipts, invoices, financial reports and backup passwords in Firestore/Realtime Database/Cloud Storage or analytics.
Review Firebase project settings, app-check/rules if any Firebase data service is later introduced, email-enumeration protection, allowed domains and provider settings before release. No billable service should be enabled without explicit owner approval.

## 9. Dependency and repository hygiene
- Do not use Capacitor or a WebView website wrapper.
- Never commit real service credentials, private keys, signing keys, sample real bank statements or real user records.
- Use public synthetic test data only in the repository.
- Keep secrets in local environment/CI secret stores only when a service truly requires them; never expose secrets to client bundles.
- Require lockfile and dependency diff review for PRs. Run static analysis, tests, dependency audit and build checks.
- Avoid requesting Android permissions that aren't needed. Explain each permission and test that denying it does not expose data.

## 10. Tests required before public release
- Cross-workspace and cross-user isolation tests for view/create/edit/export/delete.
- App lock: correct secret, wrong secret, failure rate limit, background lock, biometric cancellation/failure, process restart, key invalidation and recovery behavior.
- Storage: inspect copied app files and confirm database records are unreadable without authorized key material.
- Web XSS/CSP tests and no-third-party-script checks.
- Import: malformed/oversized files, duplicate imports, bad encodings, date/number locales, formula injection and atomic rollback.
- Backup: correct/wrong password, tampered archive, wrong schema version, partial/corrupt archive, low storage and rollback.
- Sharing: expired/replayed/guessed/revoked token, unauthorized workspace, changed role, host disconnect and reconnection.
- Network: prove that import and chart flows do not send financial data. Review all app network destinations.
- Accessibility and user-facing privacy comprehension testing.
- Independent security review before claiming strong production security.

## 11. Incident handling
If a suspected leak or unauthorized access is found: disable the affected integration/release path, preserve non-sensitive diagnostic evidence, notify affected users where appropriate, rotate/revoke exposed credentials/tokens, publish a clear fix and add a regression test. Never collect users' finance files as a default debugging measure.

## 12. Security references
- OWASP MASVS: https://mas.owasp.org/MASVS/
- Android Keystore: https://developer.android.com/privacy-and-security/keystore
- Expo SQLite / SQLCipher: https://docs.expo.dev/versions/latest/sdk/sqlite/
- Expo SecureStore: https://docs.expo.dev/versions/latest/sdk/securestore/
- Expo LocalAuthentication: https://docs.expo.dev/versions/latest/sdk/local-authentication/
- Android passkeys: https://developer.android.com/identity/passkeys