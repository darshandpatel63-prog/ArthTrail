# ArthTrail — Full Product & Engineering Blueprint

**Project status:** Planning/specification only; source code and features listed here are not yet implemented or verified.
**Product:** ArthTrail
**Repository:** https://github.com/darshandpatel63-prog/ArthTrail
**Budget target:** ₹0 at the beginning and as the standing design goal.
**Distribution model:** Public GitHub repository for the project's CI/build workflow; proprietary software, not open source.
**Platform rule:** Native/installable applications only. Android first. No Web App, no Flutter Web, no website wrapper, no Capacitor and no WebView shell.

## 1. Executive vision
ArthTrail is a privacy-first, local-first financial management application for an individual, family, farmer, freelancer, shop, startup and company. It aims to bring household spending, income, bank-statement imports, cash, budgets, multiple businesses, invoices, loans, assets, investments, reporting and later on-device AI into one app.

The first release must be useful without paying for a server, using a bank API, depending on AI, or uploading transaction data. It should run as an installable Android app first. Other native app targets may be added after the core Android vault is stable and automated tests pass.

## 2. Product principles
1. Privacy first: financial data is stored and processed on the user's device by default.
2. Local first: the app remains useful offline; an account login is not required for basic local ledger calculations once the local vault is set up, subject to the final onboarding design.
3. User ownership: users can export, back up, restore and delete their own data without a proprietary cloud being required.
4. Deterministic accounting: calculations, balances, imports and charts use tested code, not AI guesses.
5. Secure by design: Vault Lock, encryption, backup integrity, workspace isolation and permissions are designed before real data is admitted.
6. Explicit sharing: backup, Google Drive upload and device-to-device sharing are visible user-approved actions. The user is told when data leaves a device.
7. No bank API initially: users download statements from their bank and import files locally.
8. AI is last: no third-party AI API receives financial information. AI remains disabled until a viable on-device design is tested.
9. ₹0-first: choose open-source dependencies and free tools when secure and suitable. Never activate a paid plan, bigger runner, hosted server or paid provider without explicit approval.
10. No false promises: security status, tests, support for file formats and backup success must be reported honestly.

## 3. Scope and exclusions
### In scope
- Personal and family finance.
- Multiple independent workspaces for personal, family, farm, freelance, business, company and projects.
- Income, expenses, transfers, refunds, balances, budgets, recurring items and bills.
- Business contacts, invoices, payments, receivables, payables and double-entry accounting foundations.
- Farming costs/revenue, project costing, loans, assets, liabilities and manually entered investment records.
- Local bank-statement import, transaction categorization, duplicate detection, preview and undo.
- Charts, reports, CSV/report export, encrypted backup and restore.
- User-managed themes, text size, accessibility and multiple languages.
- Firebase Authentication for account identity and email verification only, after local-only behavior is designed.
- QR-based permissioned device/workspace sharing in later milestones.
- Non-personalized/contextual ads and optional premium conveniences after a useful and secure core exists.
- On-device AI insights as a late phase only if it can keep records local.

### Not in the first release
- No direct bank connection, payment initiation, UPI access or automatic bank login.
- No hosted financial database and no background upload of financial records.
- No Web App/PWA/browser finance client, no Flutter Web build and no web-based fallback.
- No Capacitor or WebView wrapper.
- No AI API upload of statements, transactions, balances, notes or reports.
- No guaranteed live share-price feeds or automatic valuations.
- No claim of tax, GST, payroll or statutory accounting compliance until independently checked against applicable rules.
- No automatic money transfers or financial actions.

## 4. App targets and platform strategy
### First target — Android
Use Flutter and Dart to build a genuine installable native Android application. Use native Android capabilities for secure key storage, biometrics, app lifecycle locking, file selection and share sheets where needed. Build signed/debug APKs using GitHub Actions. Keep signing secrets out of the repository and CI logs.

### Later targets — native clients only
After Android is stable, evaluate native Flutter targets for iOS, Windows, macOS and Linux according to actual platform support, device testing and available CI runners. These remain installable apps; the project will not create a browser-based finance app. Some hardware-bound security features behave differently between operating systems, so cross-platform support must pass platform-specific tests.

### Architecture
- Flutter presentation layer: screens, widgets, accessibility, themes and localization.
- Domain layer: money types, accounting rules, validation, permissions, report calculations and business logic.
- Local data layer: encrypted SQLite database with versioned migrations.
- Secure key layer: native platform key store integration and authenticated access.
- Import engine: deterministic parsers and field mapping, all local.
- Backup engine: versioned archive format, authenticated encryption, validation and restore transaction.
- Sharing layer: local transfer first, then explicitly approved secure peer/device connections.
- Authentication adapter: Firebase Authentication only where account sign-in is necessary; no transaction documents in Firebase.
- Testing: pure Dart unit tests, parser/property tests, database integration tests, widget tests, static analysis, dependency checks and build CI.

Suggested repository structure:

```text
ArthTrail/
  README.md
  LICENSE
  SECURITY.md
  blueprint.md
  docs/
    PRODUCT_SPEC.md
    HANDOVER.md
    APP_LOCK_DESIGN.md
    THREAT_MODEL.md
    DATA_DICTIONARY.md
    IMPORT_FORMATS.md
    BACKUP_FORMAT.md
    SHARING_PROTOCOL.md
    RELEASE_CHECKLIST.md
  app/                         # Flutter native app; Android first
    lib/
      app/                     # app bootstrap and navigation
      core/                    # errors, logging policy, utilities
      design_system/           # colors, typography, spacing, components
      features/
        auth/
        vault_lock/
        dashboard/
        workspaces/
        transactions/
        accounts/
        budgets/
        imports/
        reports/
        business/
        invoices/
        loans_assets/
        backups/
        sharing/
        settings/
      l10n/                    # English, Gujarati, Hindi
    test/
    integration_test/
    android/
    ios/                       # enable after Android foundation
    windows/                   # later native target if selected
    macos/                     # later native target if selected
    linux/                     # later native target if selected
  packages/
    arthtrail_domain/
    arthtrail_storage/
    arthtrail_import_engine/
    arthtrail_backup/
    arthtrail_security/
    arthtrail_reports/
  test_fixtures/               # synthetic data only
  .github/
    workflows/
      ci.yml
      android-build.yml
      dependency-review.yml
```

Do not create a `web/` target. If a framework-generated folder is created accidentally, exclude it from release/build workflows and remove it from the product plan. Do not use GitHub Pages to publish a working web app.

## 5. Repository visibility, license and CI
The repository is public to support public-repository GitHub Actions runs. Standard GitHub-hosted runners are currently free and unlimited for public repositories when using the standard runner types, but GitHub still has workflow duration, queue, concurrency, API, artifact/storage and acceptable-use limits. Larger runners and external services can cost money. Re-check official policies before depending on them.

Public does not mean open source. The root `LICENSE` must reserve rights and must not grant a general permission to use, copy, modify, redistribute or commercially exploit ArthTrail. Third-party dependencies remain subject to their own licenses, which must be preserved in notices. A restrictive license is a legal notice, not a technical method for hiding public source code: anyone may be able to view the public source and GitHub's terms govern platform viewing/forking. Do not store secrets in public files.

CI must contain no private secrets for unit tests and should run automatically on pushes and pull requests. Never run untrusted pull-request code with privileged write tokens or private secrets. Use minimal permissions, pin or review third-party Actions, set timeouts, limit artifact retention and avoid infinite trigger loops.

### CI responsibilities
- Format check and `flutter analyze`.
- Unit and property tests for money/accounting logic.
- Import-engine fixture tests and fuzz/negative cases where feasible.
- Database migration, workspace isolation and transaction consistency tests.
- Vault Lock and backup cryptographic test vectors using test keys only.
- Restore failure/tamper tests and duplicate import tests.
- UI widget/accessibility smoke tests using synthetic records.
- Dependency and license inventory; flag unexpected network/telemetry dependencies.
- Android debug APK build; optionally other native target builds when supported.
- Produce artifacts only for maintainers, with no financial data, service-account files or signing secrets.
- CI reports the actual pass/fail state; never report a test as passed if the workflow did not execute it.

Physical testing still requires a real device. Emulators and CI can test much of the application logic, but cannot prove real biometric behavior, OEM-specific background behavior, hardware key protection, camera QR scanning or all file-provider behavior on every device.

## 6. User types and workspaces
### User profiles
A profile identifies the person who uses the app. Account identity is separate from ownership of a local vault. A user may have a local vault before/without finishing account setup only if the final authentication flow permits it.

### Workspace model
Every set of financial records belongs to a workspace. Example types: personal, family, farm, freelance, sole business, company, joint project. Workspace boundaries are mandatory, not just a UI filter.

### Roles
- Owner: full workspace control, role management, export and deletion.
- Admin: broad workspace operations except transfer of ownership unless specifically authorized.
- Manager: only capabilities selected by the owner.
- Accountant: view/enter/reconcile finance according to granted permissions.
- Member: limited entry and own records where configured.
- Viewer: read-only access.

All permissions default to deny. The UI hiding a button is not a security boundary; domain and storage methods must enforce the current workspace and role. Personal workspace access must not be granted when a person is invited to a company workspace.

## 7. Core finance features
### Transactions
Support income, expense, transfer, refund, adjustment, opening balance and reversal. Each row should include date, amount, currency, account, category, notes, counterparty/reference where relevant, creation/edit history and source (manual/import). Avoid destructive editing of posted accounting history; use an adjustment or reversal where appropriate.

### Money representation
Never use binary floating-point for monetary calculations. Use integer minor units when currency has a fixed minor unit, or a tested decimal library for special currencies/precision. Store currency code with every amount. Handle rounding, locale, sign, negative values and exchange rates explicitly.

### Transfers
A transfer between two accounts is not income or expense. It must have linked outgoing/incoming entries and preserve the ledger total. Fees are separate expense lines. Test account-to-account, cash-to-bank and foreign-currency transfers.

### Double-entry accounting
Formal business workspaces use journal entries with debit and credit lines. A posted journal must balance under documented currency/base-currency rules. Posted entries are not silently overwritten; correct them with reversals/adjustments. Financial statements use the chart of accounts and documented report rules.

### Business
Support contacts, invoices, line items, due dates, partial payments, credit notes/refunds, accounts receivable/payable aging, expenses and business profitability. GST, TDS, payroll, tax returns and statutory compliance remain out of scope until each is separately specified and reviewed for India and any future jurisdiction.

### Agriculture and projects
Allow farm workspaces to track season/crop, land or field label, seeds, fertilizer, irrigation, fuel, equipment, labor, sales and crop-specific results. Project/freelance workspaces track clients, invoices, direct costs and project profit. These are user-maintained accounting records, not official certifications.

### Assets, loans and investments
Track loans, EMI schedule estimates, bills, property and equipment costs, dated user-entered valuations, liabilities and investment lots. Label estimated values and assumptions clearly. Net worth equals recorded asset values minus recorded liabilities; do not imply real-time market values without a verified provider.

## 8. Data model and storage contract
Use UUID identifiers, foreign keys, constraints, migration versioning, created/updated timestamps and an explicit soft-delete/archive policy. Keep amounts typed and constrained. Use JSON only for genuinely extensible metadata, not as a substitute for fields needed by accounting constraints.

### Identity, workspaces and devices
- `local_profiles`: local profile UUID, optional Firebase UID, display name, email hint, locale, timestamps.
- `workspaces`: UUID, name, type, owner profile, base currency, timezone, create/archive dates.
- `workspace_members`: workspace, profile/device identity, role, status, permission-set reference, invitation/acceptance/revocation dates.
- `permission_sets`: named capabilities such as view, create, edit own, edit all, reconcile, manage invoices, report access, export, manage members, delete workspace.
- `trusted_devices`: device ID, workspace/profile relation as needed, public key, platform, trusted/revoked status and last-seen metadata.
- `sharing_invites`: workspace, inviter, role/permissions, token hash, creation/expiry, use limit and revocation.

### Ledger
- `ledger_accounts`: workspace, unique account code, name/type/subtype, parent, currency and active flag.
- `financial_accounts`: cash/bank/card/manual account details and ledger link; do not store bank login credentials or PINs.
- `categories`: workspace, income/expense/transfer classification, parent, icon and theme color token.
- `counterparties`: customer/supplier/person/vendor display data and contact notes.
- `transactions`: workspace, date, description, kind, source, status, counterparty/reference, import batch, creator and idempotency key.
- `transaction_lines`: transaction, ledger account, debit/credit, currency, base amount, exchange-rate metadata and line memo.
- `transfers`: linked accounts and paired transaction IDs; enforce all-or-nothing creation.
- `recurring_rules`: template, frequency, next due date, end date and enabled status. Never create a payment to a bank automatically.
- `audit_events`: actor, workspace, action, entity reference, timestamp and non-sensitive change metadata; do not copy transaction descriptions or secrets into logs.

### Budgets and bills
- `budgets`: workspace, name, period, currency and status.
- `budget_lines`: budget, category and limit.
- `bills`: workspace, counterparty, due date, amount, currency and status.
- `invoices` / `invoice_lines`: invoice kind/number, counterparty, date, currency, line totals, taxes as user-entered/configured fields, payments and remaining balance.
- `invoice_payments`: invoice, transaction reference, amount, date and method label; avoid double-counting payment as a second expense where the invoice already records the expense/revenue.

### Loans and assets
- `loans`: workspace, principal, currency, interest-rate convention, start date, term, scheduled EMI and linked liability account.
- `assets`: workspace, name/type, purchase date/cost, currency and linked account.
- `asset_valuations`: asset, valuation date, amount, source note and estimate label.
- `investment_lots`: instrument label, class, quantity, purchase price/date, currency and linked account.
- `projects`: workspace, type, name, dates and a validated extension schema for farm/freelance details.

### Import, files and backup
- `import_batches`: workspace, source name, detected format, hash, timestamps, row counts, error counts and status.
- `import_staging_rows`: batch, row number, raw/normalized local fields, dedupe fingerprint, warnings and chosen disposition.
- `documents`: local file reference, content type, size, hash, retention state and workspace. No remote upload by default.
- `backup_history`: target type, creation time, schema version, encrypted status, result, size and safe error code.
- `app_preferences`: language, theme, text scale, currency/date preferences, lock/privacy settings and backup settings.

Schema implementation must be reviewed against these logical entities before migrations are committed. No real user data belongs in test fixtures.

## 9. Statement import algorithm (no AI)
1. Accept a file from the system file picker; show its name and size before parsing.
2. Process on-device only. Validate file type/content signature, limits and safe parser availability.
3. Detect supported format and parse to a staging area. Do not write straight into the main ledger.
4. Let the user map date, description, debit, credit, amount, currency, balance and reference columns when needed.
5. Normalize date/number locale and sign conventions with explicit preview. Never infer unclear debit/credit meaning silently.
6. Produce a deterministic duplicate fingerprint using account, date, amount, reference and normalized description where available.
7. Show imported, skipped, duplicate and failed rows. Require confirmation for ambiguous rows.
8. Commit all approved rows as one transaction/batch; if any integrity requirement fails, roll back.
9. Preserve import history and provide a safe undo/reversal flow.

### Supported formats — gradual rollout
- First: CSV with preview and user column mapping.
- Next: XLSX/ODS and OFX/QFX after tests across fixtures.
- Later: text-based PDFs only when extraction is reliable locally. Scanned PDFs/images can be deferred; never call a cloud OCR service by default.
- Do not promise universal compatibility. If the file is unsupported, show instructions for export to CSV from the bank's own app/site.
- Test thousands of synthetic rows, wrong encodings, locale edge cases, duplicate imports, malicious formulas, corrupted files and size limits.

## 10. Reports, graphs and dashboard
Core charts: income vs expense, balance trend, category spending, budget vs actual, business comparison, profit/loss, cash flow, overdue invoices, debt trend, asset/liability/net worth, crop/project costing and reconciliation difference.

Every report must identify its workspace, date range, currency and whether values are actual or estimated. Provide a data table/list alternative to each chart. Color alone must not convey meaning; use labels, patterns/icons where relevant and accessible contrast. Allow filters without leaking data across workspaces.

Charts must be calculated from the ledger/report engine and tested against known fixtures. Do not use AI-generated numbers. Exported reports should allow the user to choose which fields are included and should warn before sharing financial content.

## 11. Settings, languages and design system
### Languages
Launch languages: English, Gujarati and Hindi. Structure all UI strings in localization files and store the language preference locally. Future candidates: Marathi, Bengali, Tamil, Telugu, Spanish and other languages after full translation review. Never hard-code user-facing copy into widgets.

### Appearance
- Theme: System, Light and Dark.
- Accent palette choices: Teal, Indigo, Emerald, Amber, Rose and a high-contrast mode.
- Text size: Small, Default, Large and Extra Large; respect OS accessibility settings.
- Design tokens for spacing, typography, radius, elevation, chart palettes and semantic colors.
- Support screen readers, large text, keyboard navigation where applicable, sensible tap targets, focus order, reduced motion and readable error messages.
- Polished dashboard with clean information hierarchy; avoid making ads visually similar to controls or confirmations.

### Settings screens
Account; Vault Lock; biometric preference; auto-lock timer; local storage status; device list; language; theme; text scale; currency; date/number format; backup/restore; import/export; workspace access; privacy; ad preferences; premium; help; legal pages; and permanent data deletion.

## 12. Vault Lock, passphrase and biometrics
Account login and vault unlocking are separate controls. Firebase password reset must never silently unlock the encrypted local vault.

- Require a user-created ArthTrail Vault Lock before entering real financial records.
- Prefer a strong passphrase; offer a strong PIN only with progressive failure delays and a secure key design. Do not use the PIN directly as the database key.
- Generate a random vault encryption key and protect it through a reviewed key-wrapping/password-KDF design. Use Android Keystore-backed facilities where supported.
- Use encrypted SQLite/SQLCipher and verify the native build configuration; do not claim encryption merely because the database has a password field.
- Optional biometrics must go through the operating system. ArthTrail must never collect fingerprint or face templates.
- Lock after background/inactivity and app process restart. Clear secrets from memory when practical and do not log any secret or key.
- Explain recovery limitations. The backup password/recovery key is separate; if the secret and all recovery materials are lost, the vault may be unrecoverable.
- A true FIDO/WebAuthn passkey is distinct from the user-created local PIN/passphrase. Do not label a local PIN a FIDO passkey.

Required tests include wrong/correct secret behavior, repeated attempts, background lock, process restart, biometric cancellation/failure, key invalidation, backup/restore, local key rotation and proof that copied raw database files are unreadable without authorized key material.

## 13. Backup and restore
### Local backup file
- User selects create backup and provides a backup password distinct from account login; make password strength and recovery limits clear.
- Serialize only selected data and attachments into a versioned archive.
- Encrypt locally with authenticated encryption and a reviewed password KDF using random salts/nonces.
- Include format version, schema version and integrity metadata, but never include the vault secret, account password, active sessions, OAuth tokens or private service keys.
- Allow user to save/download/share the resulting file using OS-native file APIs.

### Restore
- Select file, validate size/type/version, request password, verify authenticated integrity, decrypt into temporary storage and validate relationships.
- Preview overwrite/merge behavior, counts and conflicts.
- Require confirmation, back up current state first when possible, then atomically commit; otherwise roll back completely.
- Wrong password/tampered file/corrupt archive must fail closed and leave existing data unchanged.

### Google Drive
Drive backup is optional and explicitly authorized by the user. Encrypt before upload. Use a user-visible Drive folder when the user wants to locate/download files easily. Background daily backup is best-effort because Android may defer work; show the last successful backup and stale-backup warning. Never promise backup occurs at an exact time if the OS has deferred it.

## 14. Device sharing and QR workflow
The owner can create a QR invite for a specific workspace and selected permissions. A QR may contain only a short-lived opaque invite/connection token and the minimum connection metadata—never financial records, a password, a permanent secret or a vault key.

### Permission flow
1. Owner selects workspace and exactly what the manager can do (view, add transactions, edit own entries, edit all entries, reconcile, view reports, export, manage invoices, manage members).
2. Owner sets expiration and optional use count.
3. App creates a short-lived cryptographically random invite; store a token hash when server-side storage exists, or keep state on the host for local mode.
4. Manager scans only through ArthTrail's in-app scanner, reviews workspace/permissions, authenticates, and accepts.
5. Owner confirms the connected device/person when the risk warrants it.
6. Both sides show current access status; owner can revoke a device/session immediately when connected.

### Connection reality
- Phase 1: encrypted local file export/import.
- Phase 2: direct transfer on the same local network, with authentication and a visible host session.
- Phase 3: host device reachable over the internet using a secure direct connection or a reviewed private-network/relay design.
- If the owner phone is offline, powered off, or unreachable, it cannot act as a live server. QR code alone does not create an internet path.
- A truly remote connection may require a reachable device, relay or provider. Do not promise it will always work at ₹0. Never enable a paid relay without owner approval.
- Cross-device data transmission is an explicit exception to the local-only rule; explain what is shared, with whom and how to revoke it.

## 15. Firebase and account boundaries
Use Firebase Authentication for email/password plus email verification and/or approved Google sign-in. Keep authentication metadata separate from financial storage.

Forbidden in Firebase services by default: transactions, balances, imported statement rows, receipt contents, invoices, reports, budget details, backup passwords, vault keys and finance-related analytics. Do not add Firestore, Realtime Database or Cloud Storage for finance data unless a future owner-approved spec explicitly changes the privacy promise.

Firebase or another provider can change free-tier limits. ₹0 is the target. Disable billing and avoid paid products unless separately approved. Use synthetic accounts and test environments; never include private keys or service-account JSON in GitHub.

## 16. AI — final stage only
AI is not needed to calculate balances, import bank files, categorize fixed rules or draw graphs. These are deterministic features.

Possible late-phase features: asking questions about local reports, explaining spending patterns, suggesting budgets and summarizing trends. The preferred design is on-device inference, and the model must not transmit financial records. If a practical on-device model cannot run on supported devices, keep AI unavailable rather than silently using a third-party API. If an opt-in remote AI mode is ever considered, it changes the local-only promise and needs a separate user-approved privacy design.

AI responses must query approved report functions, cite date range/workspace/data sources, distinguish fact from estimate, and never claim guaranteed future results. AI cannot execute payments, delete data or post accounting entries without explicit preview and confirmation.

## 17. Advertising and premium plan
Monetization follows core functionality and a privacy review.

### Free
- Manual transactions, workspaces, core reports/charts, local statement import/export and encrypted local backup.
- Banner ads only on non-critical screens and never near save/delete/restore buttons.
- Interstitial ads only at natural transitions; never at app launch, during data entry, statement import, backup restore, a financial confirmation or while a sensitive screen is active. Use a conservative frequency cap and re-check current ad network policies.

### Premium candidates
- Advanced comparison reports and customizable report templates.
- Branded PDF/report exports for small businesses.
- Additional theme/font customizations and convenience dashboards.
- Ad-free mode.
- Optional team convenience features only after secure sharing is working.

Never paywall access to a user's own data, basic export, deletion or essential data recovery. Ad providers must not receive financial records, transaction descriptions, balances, categories, workspace names or imported-file contents. Do not add contextual or personalized ad SDKs until their network behavior and consent obligations have been reviewed. Revenue is not guaranteed.

## 18. Data privacy, logs and dependencies
- Create an explicit list of allowed network destinations.
- Authentication connections may carry only authentication data required for sign-in.
- Drive connections happen only after user consent, and only encrypted backups are uploaded.
- Sharing connections only operate for an approved session and permission set.
- No analytics of transactions or sensitive financial activity.
- Logs may contain safe diagnostic codes and app version, not amounts, account identifiers, statement rows, notes, vault keys or passwords.
- Use minimal dependencies, pinned lockfiles, license inventory, dependency audits, and review of packages that have network, file, cryptographic, OCR or analytics permissions.
- Do not commit real bank data, secrets, keystores, signing files, OAuth tokens or test exports containing real personal information.

## 19. Testing strategy — automated first, device tests explicitly separate
All tests that can be meaningfully automated should run in GitHub Actions using synthetic fixture data. Physical-only tests must be listed as not yet verified rather than simulated as passed.

### Static checks
- Dart formatter check.
- `flutter analyze` with serious warnings treated as failures.
- Dependency and license inventory; security audit where tools support the dependency ecosystem.
- Secret scanning and repository checks for accidental private files.
- Verify the workflow does not build Flutter Web and the manifest contains no Capacitor/WebView wrappers.

### Domain/unit tests
- Exact addition/subtraction, rounding, currencies, negative amounts and date boundaries.
- Transfer pair consistency; transfers do not inflate income/expense.
- Double-entry debit/credit balancing and reversal behavior.
- Workspace filtering and permissions.
- Budget aggregation and report totals against known fixtures.

### Import tests
- Synthetic statements in each supported format.
- Duplicate re-import, malformed data, ambiguous date/amount columns and decimal separators.
- Untrusted cells, formula injection, huge files, corrupted archives and size limits.
- No change to the main ledger before user confirmation; failed import leaves no partial rows.

### Local storage and security tests
- Database migrations from each supported prior schema.
- Raw database file inspection with test content to confirm it is unreadable without the correct key.
- Vault Lock wrong-secret, repeated-attempt, timeout, process restart, lifecycle and key invalidation behavior.
- Workspace isolation and role enforcement at the data/domain layer.
- Network tests confirm creating transactions/importing statements sends no financial data to remote endpoints.

### Backup/sharing tests
- Correct/wrong password, tampered archive, corrupted/truncated archive and unsupported schema version.
- Restore preview and all-or-nothing rollback.
- Drive upload uses encrypted archives only.
- Expired, replayed and revoked QR invites; permission changes; owner disconnect; unauthorized workspace attempts.
- Host stop/revoke behavior and session expiry.

### UI/build tests
- Widget tests for core workflows, validation, error states and accessibility labels.
- App navigation and empty/loading/error states.
- Android debug APK build in CI.
- Add native target builds as those platforms are enabled.
- Use golden tests for design where stable, but don't use screenshots alone to claim functionality.

### Physical tests that still require actual devices
- Real Android biometric prompt and key invalidation behavior.
- Camera QR scanning, file picker/provider behavior and share sheet on representative OEMs.
- Background scheduling after battery optimization and reboot.
- Storage pressure, app upgrade, device loss/reinstall and actual backup restoration on a separate device.
- Verify Android package and install/launch on real hardware.

## 20. GitHub Actions design
Use workflows for non-physical tests and builds. Start with pull-request/push CI, then add artifact build workflows. Keep triggers scoped to relevant files and avoid infinite workflow triggers.

Security rules for CI:
- Public repository source is visible: never place secrets or real financial records in CI fixtures.
- Use least-privilege `permissions` such as read-only contents unless a job specifically needs more.
- Do not expose secrets to pull-request code; avoid privileged `pull_request_target` execution for untrusted code.
- Pin third-party GitHub Actions to reviewed versions/commit SHAs where practical.
- Set job timeouts, concurrency cancellation where safe, artifact retention and dependency caches responsibly.
- Use standard public GitHub-hosted runners for the ₹0 target; do not select larger runners without explicit cost approval.
- Verify each workflow's actual run result before claiming tests passed.

## 21. Milestones
### M0 — documentation and CI guardrails
Complete repository docs, proprietary license, native-only platform rule, threat model and the first GitHub Actions CI workflow. No real user data. Exit: documentation CI passes and no Web App/Capacitor commitment remains.

### M1 — Flutter Android skeleton and design system
Create the Flutter app under `app/`, native Android target only, navigation shell, design tokens, English/Gujarati/Hindi localization scaffolding, light/dark themes, fake demo data and basic CI. Exit: static analysis, widget tests and Android debug build pass.

### M2 — local vault and app lock
Implement encrypted SQLite/SQLCipher with a native build, key-management design, separate Vault Lock, optional biometrics and timeout behavior. Exit: vault lock/storage tests pass; physical biometric tests clearly marked separately.

### M3 — finance core
Implement workspace separation, accounts, income, expenses, transfers, budgets and exact reports. Exit: unit/property tests and data-isolation tests pass.

### M4 — import/export
CSV importer with staging, mapping, duplicate detection, preview and undo; export selected records. Add formats only after fixtures and tests. Exit: parser regression suite passes.

### M5 — encrypted backup
Versioned encrypted local backup, restore validation, restore preview, rollback and optional authorized Drive upload. Exit: tamper/wrong-password/restore tests pass.

### M6 — business finance
Invoices, payments, receivables/payables, loans, assets, business dashboards and double-entry foundations. Exit: ledger invariants and report tests pass.

### M7 — device sharing
QR invite, explicit roles, local direct transfer, revocation and later secure remote host design if a feasible zero-cost solution exists. Exit: abuse/replay/expiry and workspace authorization tests pass.

### M8 — release hardening and monetization
Security review, dependency review, privacy/legal pages, ad policy check, premium settings, release signing and beta feedback. Exit: no critical security defect open and backup/restore verified.

### M9 — optional on-device AI
Prototype local inference only after core reliability and device resource testing. Default AI off until it proves it can keep finance data local.

## 22. Release acceptance checklist
- App installs as a native Android app; no Capacitor and no WebView wrapper.
- No Web App/Flutter Web build target in CI.
- App-specific Vault Lock is implemented and tested; passphrase and keys are never sent or logged.
- Local finance database encryption and key lifecycle have been verified in a native build.
- Import is local, deterministic, staged and reversible.
- Exact-money and ledger consistency tests pass.
- Cross-workspace access attempts fail closed.
- Backup encryption, corruption handling and restore rollback tests pass.
- No financial data sent to Firebase, ad networks, analytics or AI services.
- Privacy, terms, disclaimer and clear data-loss/recovery warnings are accessible in the app.
- Build logs and artifacts contain no private data or credentials.
- Automated CI results are reviewed; real-device test limitations remain explicit.

## 23. Open decisions to resolve during implementation
- Exact native SQLite/SQLCipher package and compatibility with Flutter/native builds.
- Exact key-wrapping/KDF implementation after security and platform review.
- Whether local-only account mode is available or authentication is required before vault creation.
- Which bank statement formats are reliably supported by fixtures.
- Which other native platforms to add after Android.
- Whether a no-cost remote sharing method meets security and reliability needs; otherwise keep remote sharing deferred.
- Whether a fully on-device AI model is feasible on supported devices.
- Exact monetization and ad frequency after policy and privacy checks.

## 24. Official references
- GitHub Actions billing: https://docs.github.com/en/billing/concepts/product-billing/github-actions
- GitHub Actions limits: https://docs.github.com/en/actions/reference/limits
- GitHub-hosted runners: https://docs.github.com/en/actions/reference/runners/github-hosted-runners
- Flutter platform support: https://docs.flutter.dev/reference/supported-platforms
- Android Keystore: https://developer.android.com/privacy-and-security/keystore
- OWASP MASVS: https://mas.owasp.org/MASVS/
- Firebase Authentication: https://firebase.google.com/docs/auth
- Google Drive app data: https://developers.google.com/workspace/drive/api/guides/appdata

---
This blueprint is the planned target, not a claim that implementation, security testing, legal review or release approval has already occurred.

---

# ArthTrail V2 — Additive Product Expansion (2026-10-11)

This addendum extends the existing blueprint without removing earlier features. Clarification: specify the complete AI architecture, UI, policies, tools and tests now, but do not add or bundle the AI module/screen in the current ArthTrail app. Implement and integrate the separate AI subsystem only after Ritav.ai is actually ready and reviewed. No model/provider is connected now.

## 25. AI subsystem fully specified now; Ritav integration later

Follow docs/AI_SYSTEM_DESIGN.md as the complete design contract. It specifies an isolated AI package, model-neutral provider interface, capability/status model, workspace-scoped context builder, allowlisted finance tool registry, policy/confirmation gates, local conversation/memory store, audit metadata and polished chat UX. Do not add the package, route or AI UI to the current app bundle.

- When the Ritav integration milestone begins, use deterministic local tools and a mock provider for that module's tests only. Never present mock output as real AI.
- AI production runtime/provider stays disabled by default. Do not add a third-party AI SDK, remote AI key, hosted vector database or finance-data upload now.
- The app must honestly show “AI not connected yet”; it must not invent answers.
- AI tools call existing report/accounting services, never arbitrary SQL. Read-only tools may summarize authorized data; changes become drafts requiring review and confirmation.
- Reserve a versioned future Ritav adapter. When Ritav.ai is ready, verify its actual local/native interface and security before connecting. A remote service receiving finance context would change the privacy promise and requires a separate explicit opt-in decision.
- AI honors Vault Lock, workspace/module/record permissions, chat membership, localization, audit policy and retention settings.
- Required tests: runtime disabled means no model initialization/provider calls/network requests; mock AI cannot post transactions; scope violations fail closed; hostile imported descriptions cannot override system policy; invalid output is rejected.

## 26. Deterministic recurring rules for household, office and business

See docs/RECURRING_RULES_REMINDERS.md. Build a visible, deterministic rules engine—not AI guessing—for recurring purchases, events, reminders and obligations.

### Household routines
Support milk, vegetables, grocery deliveries, water cans, newspaper, tiffin, rent, utilities, school fees, subscriptions, cleaning and other repeated items. Each rule allows several vendors and multiple occurrences per day with different times, quantities, rates and payment methods.

Example: Vendor A supplies 1 litre of milk in the morning at one configured rate; Vendor B supplies 500 ml in the evening at another rate. Each is an independent rule. The notification offers Yes, as planned; Change quantity/amount; Different item/vendor; Skip; and Not received. A one-tap action records at most one occurrence using a unique ID. A skipped or unanswered reminder is not evidence of a purchase. Backfilled records preserve actual dates and correction history.

### Variable prices and units
Support units including kg, grams, litres, 500 ml, pieces, packs, hours, service units and custom units. A product may have a default price method, but actual vendor/quantity/rate can change for each transaction. Support several vegetable items/vendors in one day, discounts and configured tax/rounding. The deterministic engine calculates quantity × rate and the resulting total; it must not assume all vendors or all days share one price.

### Office and business templates
Provide configurable templates for office meals/tea, vendor bills, raw materials, stock replenishment, rent, staff/contractor costs, equipment servicing, recurring sales, projects, farm inputs, customer receivables, supplier payables, EMI, insurance, renewals and tax-preparation dates. Templates are starting points, not assumed obligations.

## 27. Lending, interest, repayments and reminders

A dedicated agreement model must record direction (I lent/I borrowed), parties, principal, currency, agreement/start and due dates, rate value and explicit basis (annual/monthly/weekly/daily/custom), simple or compound method, compounding frequency, day-count convention, payment schedule, payment allocation rule and any explicitly agreed grace/fee rules.

Never infer that a bare 2% means annual or monthly. Store the exact formula/convention and calculation-engine version. For simple annual interest using actual days/365, the estimate is principal × annual rate × actual days / 365. Other day-count methods need their own documented formula; compound interest applies only when explicitly selected. Weekly 2% must never become annual 2%.

Record each actual partial/early/late/backdated/over/under-payment as its own event. Support forgotten payments entered later with the true date and correction reason; recalculate outstanding values from the event ledger. Show principal, accrued interest, paid interest, outstanding interest, fees and next due date separately. Don't silently add late penalties or compound interest. Let users choose allocation among principal, interest and fees. Support disputes and pause follow-ups if needed.

Users select who receives which reminders, several lead times, preferred local times, quiet hours, snooze, escalation and bounded overdue repeats. See docs/RECURRING_RULES_REMINDERS.md.

## 28. Built-in calculator and accessibility

Add an in-app floating/collapsible ArthTrail Quick Calculator across ordinary app pages by default. It is an in-app widget—not an Android Accessibility Service and not an always-on-top overlay requiring permission. Users may hide it in Settings; it must not cover Save/Delete/Confirm controls or keyboard content.

Calculator and math tools: arithmetic, percentages, ratio/proportion, quantity × unit price, unit conversion, discounts/markup, user-configured tax arithmetic, bill splitting, profit margin, break-even estimate, EMI estimates, simple/compound interest, date/period differences, outstanding balances and rounding. Provide a labelled floating bubble plus a toolbar/accessibility action. Keep inputs and results screen-reader/keyboard accessible and respect text scaling. Users explicitly tap to copy a result into a form; calculation alone never posts a transaction. Show formulas/assumptions for interest and EMI.

Essential accessibility is enabled by default: screen-reader labels, semantic controls, sufficient contrast, large touch targets, clear validation, and OS text scaling. Settings can alter text size, accent palette, reduce-motion preference and the optional calculator bubble. Do not enable every optional feature by default and do not request the OS's global Accessibility Service permission for an in-app calculator.

## 29. Professional accounting, CA workflows and enterprise roadmap

See docs/ACCOUNTING_CA_ENTERPRISE.md. Retain a Simple Mode for households and add Advanced Accounting Mode for business and CA users. Plan for a configurable chart of accounts, journal vouchers, general ledger/subledgers, balanced double-entry posting, trial balance, profit/loss, balance sheet, cash flow, bank reconciliation, receivable/payable aging, invoices/credit notes/partial payments, inventory, fixed assets/depreciation policies, budgets/forecasts, cost centres, projects, branches, multi-currency, fiscal-year close/lock, reversals, approvals, audit trail and document evidence.

CA workflow: separately permissioned client workspaces, time-limited invitations, document checklists, review comments, exception reports, draft adjusting journals, owner approval, review status and audit-ready exports. Access to one client must never grant access to another client or the CA's personal workspace.

For larger organizations, design for entity/branch hierarchy, subsidiaries, intercompany entries, consolidation, separation of duties, approval thresholds, read-only auditor role and optional user-owned server. Do not claim a single local database serves trillions in transaction volume. Enterprise scale requires measured performance, server infrastructure, availability, monitoring, recovery drills, security review and potentially non-zero operating costs. GST/TDS/payroll/statutory features are not certified until current rules and qualified professional review are complete.

## 30. Fine-grained sharing permissions

Sharing can be scoped to Home, Family, Farm, Project, Client, Business, Company or another user-created workspace. Inside each workspace, grant view/create/edit-own/edit-all/reverse/approve/reconcile/export/invite/manage/share capabilities separately for dashboards, transactions, recurring items, loans/interest, invoices, budgets, salary-sensitive data, inventory, assets, reports, journal vouchers, attachments, chat, backups and AI tools.

Allow record/field restrictions for salary, owners' drawings, partner capital and personal advances. Enforce permissions in the domain/storage/service layer, not just by hiding UI buttons. Revocation blocks future sync when devices reconnect, but cannot guarantee deletion of a copy already downloaded by a device; explain this to the owner.

## 31. Device peers, server mode, offline sync and chat

See docs/MULTI_DEVICE_SYNC.md. Keep local-only as the default. Add encrypted file transfer, same-network direct sync, optional trusted device-host mode and a future self-hosted-server option. A phone only serves data while running and reachable. If a device is offline, its updates stay in a local encrypted outbox until a trusted reachable peer/host can exchange them. Internet reconnection alone does not guarantee instant delivery if no peer/host is reachable.

Use unique idempotent outbox operations, append-only posted ledger history, reversals for corrections, reviewed conflict handling and recalculation of balances from accepted ledger events. Never silently use last-write-wins for money or posted accounting records.

Future company chat can support direct/group conversations, encrypted attachments, mentions, notifications, delivery state, offline drafts and workspace search. Finance tools inside chat use the same permission system as the app and future Ritav tool registry. A chat member cannot gain data access merely by asking for a report in chat; posting, changing interest, exporting or sharing requires preview and authorization. A user-owned server is optional and must document key ownership, visibility, updates, backups and uptime. Reliable cross-internet peer access may require a reachable host, private network or relay and cannot be promised free forever.

## 32. External reminders and messages

Store contact's selected phone/channel, message template, event condition, due date, sending identity and schedule. On trigger, generate a local notification and message draft in the selected language. First release opens the default SMS app or WhatsApp with text prefilled; the user confirms recipient and taps Send. Do not read SMS or silently send it.

Direct programmatic SMS can be restricted under Google Play policies. Official WhatsApp Business Platform messaging needs separate provider/account configuration, consent/templates and may incur charges. Keep automated external messaging disabled under the ₹0 launch target until expressly approved. Never expose a debt amount on the lock screen by default. Notification actions and background delivery are subject to Android battery/OEM limits, so exact delivery time is not guaranteed.

## 33. Rules engine acceptance requirements

- Every rule is visible, versioned and user-confirmed before activation.
- Every occurrence has a stable ID, planned time, actual values, source rule/version, actor and status.
- No unanswered reminder is treated as proof of purchase/payment.
- No external message or payment is sent without required user action.
- All formulas have deterministic tests and documented rounding/edge cases.
- Importing or syncing the same event twice never doubles a transaction.
- Users can pause, preview, edit, backfill and disable rules.

## 34. Additive automated testing requirements

Implement in GitHub Actions wherever possible, and report actual results rather than assumptions:
- Recurrence, multiple daily milk deliveries, different vendors/prices, yes/change/skip and duplicate notification action tests.
- Variable quantity × unit price, rounding, discounts, configurable tax and date/time boundary tests.
- Known-vector tests for annual/monthly/weekly rates, simple/compound interest, leap years, partial/early/late/backdated/overpayments and payment-allocation rules.
- Permission tests by workspace, module, record and action; home vs company and client vs client isolation.
- Offline outbox, reconnect, replay, out-of-order operations, clock drift, conflicts, revocation and inaccessible host tests.
- Chat attachment and finance-tool permission tests.
- AI disabled/no-network, mock-provider, prompt-injection, invalid output, allowlist, confirmation and cross-workspace isolation tests.
- Calculator, localization, reminder and accessibility semantics tests.
- Keep existing statement-import, backup/restore, ledger consistency and security tests.
- Physical-only tests (OEM background notifications, real biometrics/Keystore, camera QR scanning, real peer networking and cross-device restore) stay marked pending until actual devices are available.

## 35. Added detailed specifications

- docs/AI_SYSTEM_DESIGN.md — AI system architecture/UI/tool permissions, disabled until Ritav.ai is ready.
- docs/RECURRING_RULES_REMINDERS.md — recurring household/business rules, milk/vegetable use cases, interest, repayments and messaging.
- docs/MULTI_DEVICE_SYNC.md — peer/device-host sync, offline queues, QR roles, chat and self-hosted server.
- docs/ACCOUNTING_CA_ENTERPRISE.md — professional accounting, CA workflows and enterprise roadmap.
