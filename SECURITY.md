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
- A true platform/FIDO passkey is a cryptographic authentication credential, not the same as a typed local app-lock code. Do not call the PIN a FIDO passkey. Implement true passkey sign-in separately only after the server/provider path is verified.

## 4. Data encryption and key management
### Android
- Use an encrypted SQLite database through a reviewed SQLCipher-compatible Flutter/native package. Test encryption end-to-end in an actual native Android build.
- Generate a random high-entropy database/vault key. Do not derive the database key directly from a short PIN.
- Protect key wrapping with Android Keystore-backed facilities/secure storage; where supported, require local device authentication to use/unlock the key.
- Keep keys out of AsyncStorage, plain SQLite, logs, GitHub, build logs, app constants and screenshots.
- On logout, lock and clear transient in-memory secrets; never silently delete the local vault unless the user confirms.

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
- Build-configuration checks that no Web App/Flutter Web output or WebView wrapper is introduced.
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
- Flutter platform support: https://docs.flutter.dev/reference/supported-platforms
- Flutter Android integration: https://docs.flutter.dev/platform-integration/android
- Android Biometrics: https://developer.android.com/identity/sign-in/biometric-auth
- Android passkeys: https://developer.android.com/identity/passkeys

## 13. Additional controls for AI, rules, interests, chat and offline sync

### AI subsystem design (not bundled until Ritav.ai is ready)
- Create an isolated provider interface, typed tool registry, workspace-scoped context builder, local conversation memory, permissions, policy gates and disabled UI state. Keep real model initialization behind a hard production-off switch.
- No third-party AI endpoint receives account transactions, imported statements, reports, balances, invoices, interest agreements or chats.
- AI tools are allowlisted and call validated domain services; never expose SQL/database handles, shell execution or unrestricted file/network access.
- Every output is untrusted. Calculated facts come from deterministic tools and include workspace, date range, currency and source records.
- Mutations become a preview/draft, then require the user's current permission and confirmation. An AI/chat request never bypasses authorization.
- Treat imported descriptions, PDFs, messages, vendor names and attachments as untrusted prompt-injection surfaces.
- Tests must prove the disabled provider performs zero initialization and zero network calls, and that cross-workspace queries and unapproved writes fail closed.

### Interest calculations and reminders
- Interest rate basis must be explicit (annual/monthly/weekly/daily/custom), as must simple/compound method, compounding frequency, day-count basis and payment-allocation rule.
- Store formula/version and dated inputs. Do not silently add late fees, convert a weekly rate to an annual rate, or rewrite historical repayment events.
- Keep partial, early, late, over-, under- and backdated payments as separate auditable events.
- Notification quick-actions require idempotent occurrence IDs; repeated taps and sync replay must not duplicate an entry.
- Lock-screen messages should avoid balances, debt amounts, account names or detailed transaction descriptions unless the user explicitly opts into that exposure.
- User-confirmed SMS/WhatsApp drafts are the default. Automated external messaging is disabled until a provider, consent, cost and applicable policy review is complete.

### Device host, peer sync and chat
- Every local edit is persisted in the local vault first. Sync is an explicit encrypted outbound queue scoped to permitted workspaces.
- Use authenticated peer identities, per-device revocation, cryptographic integrity, replay/duplicate protection, bounded queues, versioned schemas and idempotent event processing.
- Never use last-write-wins as the financial ledger truth. Posted entries are append-only; corrections use reversal/adjustment; ambiguous simultaneous edits require review.
- A relay forwards ciphertext only and must not possess end-to-end decryption keys. Minimize routing metadata and document what a server host can see.
- The app must show pending/synced/conflict status. A disconnected peer cannot be assumed updated; never claim instant sync when no reachable peer/host exists.
- QR invitations are short-lived, high entropy and limited-use; QR content must not contain financial data, passphrases, vault keys or long-lived device secrets.
- Revocation prevents future access/sync where peers reconnect but cannot guarantee erasing a copy that an authorized device already downloaded.
- Chat attachments and inline finance tools are workspace-scoped. Do not expose a report just because someone is in the same chat; evaluate the member's finance permissions independently.
- Self-hosted mode requires authenticated clients, secure transport, patching, backups, rate limits, monitoring and clear key-visibility disclosures.

### Recurrence engine, calculator and external contacts
- Recurring rules must be previewed and explicitly activated. No unanswered prompt is proof that a product was received or a payment was made.
- Use stable occurrence IDs and atomically link each confirmed occurrence to at most one transaction.
- Unit prices and quantities must be normalized safely; reject ambiguous currency/unit conversions instead of guessing.
- The Quick Calculator is an in-app widget, not a global Accessibility Service or permission to draw over other apps. Calculation does not commit financial data without an explicit user action.
- For external message drafts, verify recipient number, template and due item in a preview. Do not programmatically send SMS or read messages by default. Avoid SMS permissions unless a separate Android distribution/policy review establishes eligibility.
- Contact phone numbers, message drafts and assigned reminders are private financial-adjacent data; encrypt locally and keep them out of logs/analytics.

### CA / enterprise workspaces
- Client, company, home, farm and project spaces are isolated. CA membership must be explicitly granted per client and must not imply access to another client workspace.
- Enforce module/record/action permissions at the domain layer, including exports, loans, interest terms, payroll-sensitive data, journal posting, period re-opening and member administration.
- Approval separation and audit history must be enforceable in logic, not only represented visually.
- Enterprise server mode remains optional and must not silently migrate local vault data to a remote server.

## 14. Added security acceptance tests
- AI-off test proves zero provider initialization, zero context egress and no remote-model package/service used.
- Tool authorization for each role/workspace/module; prompt injection and malformed provider output fail closed.
- Rate basis, day-count convention, compounding, rounding and payment-allocation tests use known independent calculation vectors.
- Repeated notification actions and peer replays never create duplicate transactions.
- Offline change logs, tampered messages, replayed QR tokens, expired/revoked device and wrong workspace requests are rejected.
- Conflict test demonstrates that simultaneous changes to posted financial facts never silently overwrite one another.
- Chat recipient cannot obtain a financial report through message context or tool calls without the relevant workspace/report permission.
- Reminder lock-screen privacy, app-lock queueing and external-message confirmation are tested.
- Self-hosted configuration and host mode cannot accept anonymous write operations.
- All CI tests use synthetic financial information. Physical-only behavior is pending until it is tested on actual hardware.
