# ArthTrail — Full Product & Engineering Blueprint

**Document status:** Working blueprint v1.0. This is a plan, not evidence that features are implemented or security-tested.
**Product:** ArthTrail
**Repository:** https://github.com/darshandpatel63-prog/ArthTrail
**Initial cash budget:** ₹0 target
**Distribution/source policy:** Proprietary, not open source. The public GitHub repository is used for workflow/CI access and transparent development; public visibility does not prevent viewing or forking on GitHub.
**Non-negotiable:** No Capacitor. No web app. Build native-capable apps only.

---

## 1. Product vision

ArthTrail is one private financial workspace for an individual, family, farmer, freelancer, shop, startup or company. It combines household spending, several businesses, income and expense records, accounts, loans, bills, invoices, assets, investments, statement imports, reports and (later) private AI analysis.

The product should make three promises:
1. **Understandable:** users can enter or import their own data, see how totals were calculated, and correct mistakes.
2. **Private by default:** financial data is stored and processed on the user's devices. It is not sent to Firebase, analytics services, ad providers, AI APIs or package maintainers.
3. **Useful without a paid server:** core accounting, charts, import/export and encrypted local backup should work offline after installation. Paid or hosted add-ons must not be prerequisites for core use.

## 2. Platform decision: native apps only

### Approved direction
Use **Flutter** as the initial cross-platform app framework because it can target Android, iOS, Windows, macOS and Linux from one shared app codebase. Generate only the native targets required; do not enable/build Flutter Web.

- Android: first-class release target and first device for development.
- iOS: planned native target; signing and release require Apple developer tooling/accounts and real-device testing before public distribution.
- Windows, macOS and Linux: native desktop clients planned after the Android local vault is stable.
- Web App/PWA/website client: **not in scope**. Do not create `apps/web`, do not publish a browser version and do not add a WebView shell.
- Capacitor: **prohibited**.

Use Flutter's native application build system, not a website-to-app converter. If a dependency forces a web app or embeds a remote website as the primary interface, reject it.

### Native project structure
Suggested structure (adjust only when implementation requires it):

```text
ArthTrail/
  android/                   # Flutter-generated native Android project
  ios/                       # native iOS target (planned)
  windows/                   # native Windows desktop target (planned)
  macos/                     # native macOS desktop target (planned)
  linux/                     # native Linux desktop target (planned)
  lib/
    app/                     # app shell, navigation, lifecycle/lock handling
    core/
      crypto/                # reviewed wrappers around vetted crypto APIs
      database/              # migrations, encrypted local persistence
      money/                 # exact-money types and ledger calculations
      auth/                  # Firebase identity adapter (no finance data)
      permissions/           # workspace/role authorization
      backup/                # archive, encrypt/decrypt, validation, restore
      import/                # local statement parsers and staging
      sharing/               # trusted devices, invites, transfer protocols
      localization/           # English/Gujarati/Hindi and later languages
    features/
      onboarding/
      vault_lock/
      dashboard/
      workspaces/
      transactions/
      accounts/
      budgets/
      businesses/
      invoices/
      loans_bills/
      assets_investments/
      reports/
      import_export/
      backup_restore/
      device_sharing/
      settings/
      premium_ads/
      ai_assistant/           # disabled until privacy criteria are passed
  assets/
  test/
    unit/
    widget/
    integration/
    security/
  docs/
  .github/workflows/
  BLUEPRINT.md
  SECURITY.md
  LICENSE
  README.md
```

Use one primary database per local vault/profile. Treat UI as a view of domain services, not as the place where money rules are calculated.

## 3. Architecture

### Local-first data flow

```text
Native UI (Android / iOS / Windows / macOS / Linux)
          |
          v
Domain services (money, ledger, permissions, parsing, reports)
          |
          v
Encrypted local database + encrypted local attachment folder
          |
          +--> Local charts/reports
          +--> User-selected encrypted backup file
          +--> Optional encrypted Google Drive backup
          +--> Explicitly authorized device-to-device transfer

Firebase Authentication: identity and email verification only
Remote financial database: none in the first release
Third-party AI API: none
```

### Core principles
- Local database is the source of truth by default. Network loss must not prevent manual entry, calculations or reading existing records.
- No transaction description, amount, statement row, receipt OCR, account balance, report or financial chart data may be sent to a cloud backend.
- Firebase stores authentication information required for login only. Do not use Firestore, Realtime Database, Cloud Storage, Analytics or Crashlytics to hold/log financial data.
- Backups and sharing are exceptions only after a clear user action. Encrypt locally before data leaves the device.
- A copy of the encrypted file uploaded to Drive is not plain-text finance data, but the user must understand that a third-party provider stores the encrypted object and metadata.
- A transfer to another device means data leaves the source device; make destination, workspace and permissions explicit before sending.
- npm/other dependency installation does not grant those packages the right to receive user data. Review dependencies, disable unneeded telemetry and block unexpected network destinations.

### ₹0 build target
Use open-source build tools, a public repository with GitHub Actions standard runners, local processing and free tiers only where appropriate. Do not enable paid services automatically. A ₹0 target cannot guarantee that app-store accounts, all future hosting, domain names, remote relays, email delivery, provider quotas or commercial distribution are free forever.

## 4. Users and workspace model

One signed-in profile may create independent workspaces such as:
- Personal
- Family / household
- Farm / agriculture
- Freelance / creator
- Shop / small business
- Startup / company
- Project / rental property

A user may have many workspaces; a company can invite other app users into one specific workspace. Never merge private and company records merely because they share an email address.

Initial roles:
- Owner: full control of the workspace, membership and recovery/export policy.
- Manager: permissions selected by owner; may be allowed to enter transactions/invoices and view reports without permission to export, delete or manage members.
- Accountant: accounting-entry/reconciliation role where enabled.
- Viewer: read-only, with reports/transactions limited to the assigned workspace.
- Custom role: later, built from individual permissions.

Authorization must be checked in the domain/data layer, not just by hiding UI buttons.

## 5. Feature map

### 5.1 Personal and household finance
- Income and expenses, categories/subcategories, tags, notes, payees and sources.
- Cash in hand, manually maintained accounts, transfers, refunds, reversals and recurring items.
- Household budget by month/category, budget-vs-actual, saving targets and upcoming bills.
- Family member contributions/shared costs, split expenses and settle-up calculations.
- Loan/EMI tracking and money owed to/from others.

### 5.2 Multi-business finance
- Many independent workspaces and companies.
- Customers, suppliers, invoices, invoice lines, payments, receivables and payables.
- Business income/expenses, cash flow, budget comparison and profit/loss reports.
- Double-entry ledger for posted company transactions; every posted journal must balance.
- Inventory/stock and employee cost tracking after core accounting is reliable.
- Tax, payroll and statutory features must not be marketed as compliant until jurisdiction rules and tests have been reviewed by a qualified professional.

### 5.3 Agriculture
- Farm/season/crop grouping, seed, fertilizer, pesticide, labour, irrigation, equipment and transport costs.
- Crop-sale receipts, supplier debt, seasonal profitability and farm-loan tracking.
- All prices/quantities are user-entered; no claims about market prices without a trusted and authorized source.

### 5.4 Freelancer and project management
- Clients, jobs/projects, expenses per project, time/cost notes, invoice/payment tracking and due dates.
- Separate personal and business cash flow.

### 5.5 Assets, loans and investments
- User-maintained assets (house, land, vehicle, equipment), loans and liabilities.
- Investments recorded with manually entered purchase details and dated valuations.
- Net worth = recorded asset values minus recorded liabilities.
- Clearly label stale/manual valuations; do not imply live market quotes.

### 5.6 Import from bank statement files — no direct bank connection
- The user downloads their bank statement from the bank and opens it in ArthTrail.
- Parsers run locally; no online converter, backend OCR or AI API is used.
- Start with CSV and carefully tested statement-like spreadsheets. Add OFX/QFX and text-based PDF only after parser tests. Image-only/scanned PDFs may require local OCR or manual entry; do not upload the file.
- Identify the format by content/signature when practical, then offer column/date/amount mapping.
- Show a preview, uncertain rows, debit/credit direction, duplicates, import errors and final totals before the user confirms.
- Stage rows first; commit atomically; support undo and a local import history.
- Do not claim support for every bank/file type until representative test fixtures pass.

### 5.7 Import/export and reports
- CSV export for transactions, accounts, categories, invoices and selected reports.
- Human-readable reports such as PDF only when supported by local generators and properly tested.
- Every graph gets its date/workspace/currency filters, legend, accessible text and a table alternative.
- Graph types: income vs expense, category share, monthly trend, cash flow, budget vs actual, business comparison, profit/loss, receivables aging, debt/EMI trend, net worth, farm/project cost and reconciliation difference.
- Use distinct, configurable colors, but never rely on color alone; include labels/patterns/table values and accessible contrast.

### 5.8 Backup and recovery
- Manual backup to a local file at any time.
- Encrypted backup before writing to disk, removable storage or Google Drive.
- Optional daily backup schedule with last-success time, retry and stale-backup warning. Android OS may defer background work; never promise an exact minute or guaranteed daily run.
- Restore preview and validation before modifying live data.
- Backup settings, selected workspace data, categories, ledger, transactions and user-selected attachments. Do not back up Firebase tokens, app-lock secrets, passkeys, private keys or OS permissions.
- Backups need a separate strong password or recovery key. Firebase password reset is not a way to decrypt an encrypted local vault.

### 5.9 Device sharing and remote access
- QR code contains a short-lived invitation or connection token only; no financial values or passwords.
- Owner selects workspace, role, granular permissions, expiry and allowed uses; receiver confirms identity and permissions.
- The host device must be unlocked/authorized, powered on and reachable for live host mode.
- V1: export/import encrypted data and same-LAN transfer. A later peer connection may use authenticated encryption and device keys.
- Remote use across unrelated networks needs a reachable host, VPN/private network, or relay. A public relay means network infrastructure exists and may cost money; do not promise an always-free global remote connection.
- Show active peers, workspace being shared and a visible Stop Sharing button. Support revocation.

### 5.10 AI assistant — gated, final phase
- AI is deliberately not part of the first core release even though the blueprint reserves its screens/contracts.
- First choice: a local/on-device model if it fits supported devices and passes privacy/performance tests. If local AI is not viable, the default remains AI disabled; ask the user before ever offering an external provider.
- Do not send finance records to a third-party model, SDK, npm package, analytics or server.
- All totals and balances come from deterministic tested queries; AI only explains the results and must identify periods/workspaces used.
- The assistant must not execute a transfer, payment, deletion or transaction change without an explicit confirmation screen.
- Mark estimates as estimates and show the transaction range/source behind an answer.

## 6. Authentication and private Vault Lock

### Account access
- Firebase Authentication for account identity, email/password, email verification and optional Google login where the platform's flow is supported.
- Keep billing disabled unless the owner explicitly approves a paid service.
- Financial data stays local even after account login. Account identity alone does not grant access to an encrypted vault.

### ArthTrail Vault Lock
- User creates an app-specific strong passphrase or PIN; recommend a long passphrase for the primary secret.
- App lock is distinct from Firebase password and true FIDO/WebAuthn passkeys. Do not call a typed PIN a FIDO passkey.
- Require lock after background/inactivity and after process restart; support optional platform biometrics only through native APIs.
- Generate a random vault key. Do not use a short PIN directly as an encryption key. Use a reviewed password KDF with random salt and a platform-protected key-wrapping design.
- On Android, prefer a vetted SQLCipher integration for SQLite and use Android Keystore-backed key protection where feasible.
- Web-specific key design is out of scope because there is no web app.
- Limit repeated attempts with increasing delays. Never transmit, log or back up the app-lock secret or decrypted keys.
- Explain that losing both the secret and recovery material may permanently lose access to encrypted local data.

## 7. Database design

Use UUID identifiers, foreign keys, database migrations and explicit timestamps. Money must never use binary floating-point. Use integer minor units when currency minor-unit rules are clear, otherwise a reviewed exact-decimal representation; include ISO currency codes for monetary values.

### Profile, devices and workspace
- `profiles(id, firebase_uid nullable, email, display_name, language, created_at, updated_at)`
- `devices(id, profile_id, display_name, platform, public_key, trusted_at, last_seen_at, revoked_at)`
- `workspaces(id, name, workspace_type, owner_profile_id, base_currency, timezone, created_at, archived_at)`
- `workspace_members(id, workspace_id, profile_id, role, permission_set_id, status, invited_at, accepted_at, revoked_at)`
- `permission_sets(id, workspace_id, name, can_view_transactions, can_create_transactions, can_edit_own, can_edit_all, can_reconcile, can_manage_invoices, can_view_reports, can_export, can_manage_members, can_delete_workspace)`
- `sharing_invites(id, workspace_id, created_by, role, permission_set_id, token_hash, created_at, expires_at, max_uses, uses_count, revoked_at)`

### Accounting
- `ledger_accounts(id, workspace_id, code, name, account_type, subtype, currency_code, parent_id, is_system, is_active)`
- `categories(id, workspace_id, name, kind, icon, color_token, parent_id, is_system)`
- `counterparties(id, workspace_id, kind, display_name, contact_note, is_active)`
- `transactions(id, workspace_id, transaction_date, created_at, description, transaction_kind, source_kind, counterparty_id, reference_number, notes, created_by, import_batch_id, status, reversal_of_id, idempotency_key)`
- `transaction_lines(id, transaction_id, ledger_account_id, debit_minor, credit_minor, currency_code, base_amount_minor, exchange_rate, memo)`
- `financial_accounts(id, workspace_id, ledger_account_id, display_name, account_kind, currency_code, opening_balance_minor, opening_balance_date, is_active, last_four)`
- `transaction_attachments(id, transaction_id, document_id, attachment_kind)`
- `recurring_rules(id, workspace_id, template, frequency, next_run_date, end_date, enabled)`

### Budgets and business
- `budgets(id, workspace_id, name, period_start, period_end, currency_code, total_minor, created_by)`
- `budget_lines(id, budget_id, category_id, limit_minor, warning_percent)`
- `bills(id, workspace_id, counterparty_id, title, due_date, amount_minor, currency_code, status, recurring_rule_id)`
- `invoices(id, workspace_id, invoice_number, kind, counterparty_id, issue_date, due_date, currency_code, subtotal_minor, tax_minor, total_minor, paid_minor, status)`
- `invoice_lines(id, invoice_id, description, quantity_decimal, unit_price_minor, tax_rate_basis_points, line_total_minor, ledger_account_id)`

### Loans, assets and projects
- `loan_accounts(id, workspace_id, name, principal_minor, currency_code, annual_rate_basis_points, start_date, term_months, emi_minor, next_due_date, linked_ledger_account_id)`
- `assets(id, workspace_id, name, asset_type, purchase_date, purchase_cost_minor, currency_code, linked_ledger_account_id, notes)`
- `asset_valuations(id, asset_id, valuation_date, value_minor, currency_code, source_note)`
- `investment_lots(id, workspace_id, asset_name, asset_class, quantity_decimal, purchase_price_minor, purchase_date, currency_code, account_id, notes)`
- `project_records(id, workspace_id, project_type, name, start_date, end_date, validated_metadata)`

### Import, files and logs
- `import_batches(id, workspace_id, source_filename, detected_format, file_sha256, started_at, completed_at, row_count, imported_count, skipped_count, error_count, status)`
- `import_staging_rows(id, import_batch_id, source_row_number, raw_fields, normalized_fields, dedupe_key, confidence_flags, status, transaction_id)`
- `documents(id, workspace_id, local_relative_path, mime_type, size_bytes, sha256, created_at, retention_status)`
- `backup_runs(id, profile_id, destination_kind, created_at, schema_version, encrypted, status, file_size_bytes, checksum, last_error_code)`
- `audit_events(id, workspace_id, actor_profile_id, device_id, action, entity_type, entity_id, occurred_at, before_hash, after_hash, reason)`
- `app_preferences(profile_id, language, theme_mode, accent_theme, font_choice, text_scale, default_currency, date_format, backup_preferences, privacy_preferences)`

All important data is relational and typed; do not hide the entire accounting model inside arbitrary JSON. Every posted double-entry journal must balance. Transfers must not inflate income or expense. Reversals and edits must remain traceable.

## 8. UI/UX and accessibility

### Navigation
- Primary destinations: Overview, Transactions, Accounts, Reports, Workspaces and Settings.
- Contextual sections: Budgets, invoices, loans/bills, assets/investments, imports, backup and sharing.
- Provide one clear workspace switcher and make the current workspace visible before adding or importing data.

### Visual system
- Build a calm, professional financial design with readable cards, clear number hierarchy and no clutter.
- Modes: System, Light and Dark.
- Accent palettes: Teal, Emerald, Indigo, Blue, Amber, Rose and high contrast.
- Typography: System default plus carefully tested font choices; text size Small/Default/Large/Extra Large.
- Languages at launch: English, Gujarati and Hindi. Add Marathi, Bengali, Tamil, Telugu and Spanish via translation files later.
- Support screen readers, large text, dynamic layout, good contrast, reduced animation, minimum touch target sizes and table alternatives to charts.
- Never rely only on color to communicate expense category, debt, warning or status.
- Destructive actions (delete workspace, replace restore, revoke access) require clear confirmation and explain consequences.

## 9. Monetization without breaking privacy

### Free tier
- Manual finance records, workspaces, core graphs, local statement import/export and encrypted local backup.
- Ads, if used, must never transmit financial records, category names, descriptions, account balances or workspace data.
- Keep ads away from transaction entry, import/review, backup/restore, confirmation and other sensitive workflows.
- Use only policy-compliant banner placement; interstitials only at natural transitions and conservatively rate-limited. Do not show at app launch or during a financial task.

### Premium candidates
- Ad removal.
- Advanced report templates, richer comparisons and more export formats.
- Custom labels/themes and branded business reports.
- Additional local automation/reporting and convenience tools.
- Never paywall access to a user's own data, export or essential recovery.

Do not assume ads or premium will generate a guaranteed income. Ads and some store/monetization integrations may require online access, consent, provider accounts and ongoing policy compliance. Do not add an ad SDK until its data collection, permissions, network activity and privacy notices have been reviewed.

## 10. Local statement import algorithm

1. Validate file signature/type, maximum size, row/column count and text size.
2. Parse only supported formats using reviewed local dependencies.
3. Store rows in a staging area—not directly in the ledger.
4. Normalize date, amount, currency, debit/credit direction, reference and description according to an explicit mapping.
5. Create a conservative deduplication fingerprint using account, date, amount, reference and normalized description where available.
6. Preview every row, uncertainty, skipped item and batch total. Let the user map columns and choose the account.
7. Commit only after confirmation, atomically, with batch ID and undo support.
8. Test repeated import, partial corruption, locale variations, negative amounts, refunds, malformed rows and formula-injection payloads in exported CSVs.

AI must not be used to decide how to import money in the core importer. Low-confidence or ambiguous rows require human review.

## 11. Encrypted backup and Google Drive

- Local archive first; use a documented versioned format.
- Encrypt before file write/upload using authenticated encryption (such as AES-256-GCM) and a reviewed password KDF with random salt.
- Backup password/recovery key is separate from Firebase password and is not stored with the archive.
- On restore: check size/version, authenticate/decrypt, validate every record and relation, show a preview, restore to temporary state, then commit atomically. Roll back fully on error.
- The optional Drive integration must use user consent and minimum scopes. A visible Drive folder is required when users expect to browse backup files manually; hidden appDataFolder is not suitable for that UX.
- Daily auto backup is best-effort because Android may defer jobs and the device may be offline, powered off or restricted. Show last successful backup and warn when stale.
- Never include tokens, passkeys, app-lock secrets, database keys, permissions granted by the OS or other credentials in a backup.

## 12. QR and device-to-device security

- Use high-entropy short-lived invite token; show expiry and selected workspace/permissions in the confirmation screen.
- QR contains no financial data, password or long-lived key. Hash tokens at rest where an invite registry is required.
- Authenticate both devices and establish an encrypted peer session using reviewed protocols and per-device keys.
- Protect against replay, expiry, duplicate acceptance, unauthorized workspace, revoked devices and role changes.
- For host mode, the owner device must be awake, unlocked/authorized and reachable. Same-LAN mode is the first live option.
- Arbitrary internet remote access needs an addressable host, VPN/private network or a relay. A relay is not compatible with a literal promise that no infrastructure is used and may not remain free.
- Display active connections and a single obvious Stop Sharing action.

## 13. Test plan — automated first, physical device later

Run all feasible tests through GitHub Actions and local/static analysis before asking the owner to install a build. Do not report tests as passed unless workflow/test output proves that they passed.

### Unit tests
- Money arithmetic and currency conversions (when enabled), decimal rounding and locale parsing.
- Transfer/refund/reversal behavior; balanced journal rules.
- Workspace permission matrix and default-deny checks.
- Budget, category, profit/loss, debt and net-worth calculations.
- Import format detection, field mapping, duplicate fingerprints and error handling.
- Backup format, encryption/authentication, wrong passwords, tamper detection and migrations.

### Widget/UI tests
- Onboarding and Vault Lock flow.
- Workspace switch and current-workspace context.
- Transaction entry and validation states.
- Import preview/uncertain rows/confirm/undo.
- Graph filter controls, language switching, themes and text scaling.
- Permissions and share/revoke screens.

### Security tests
- Cross-workspace read/write/export attempts must fail.
- Wrong Vault Lock secret never opens the database; repeated failures trigger delays.
- Lock after background/inactivity/restart.
- No secrets in logs, fixtures, artifacts, source maps or workflow outputs.
- No financial data in outbound network requests when entering transactions, generating reports or parsing statements.
- Import oversized/malformed files, archive bombs where relevant and CSV formula payloads safely.
- Backup tampering and wrong password fail closed; restore does not partially overwrite the vault.
- QR token expiry/replay/revocation and permission downgrade behavior.
- Dependency audit and secret scanning.

### Build checks through GitHub Actions
- Formatting and static analysis.
- Unit/widget tests.
- Dependency license/security review.
- Build Android debug APK and archive it as a test artifact if workflow quota/policies permit.
- Build native desktop targets on compatible hosted runners when those platform directories exist.
- Do not build or test Web; CI should explicitly verify there is no web target or Capacitor dependency.

### Physical-device-only tests
Automated workflow cannot fully prove fingerprint/face unlock, real network handover, OEM battery/background behavior, camera QR scanning, Android permission prompts or install/upgrade behavior on every device. These require the owner or a physical-device lab after automated tests pass.

## 14. Development milestones and acceptance criteria

### M0 — repository foundation (current)
- Product/security docs, proprietary license, native-only platform decision and handover updated.
- No application features claimed as complete.

### M1 — Flutter native skeleton
- Flutter project targets Android first; web platform is not enabled. Build config contains no Capacitor or WebView app wrapper.
- Polished shell, navigation, theme tokens and synthetic demo data.
- English/Gujarati/Hindi strings wired in from day one.
- Formatting/static-analysis/test workflow runs green.

### M2 — Vault Lock and encrypted local database
- Device-specific local vault, user secret, secure lock lifecycle and optional native biometric flow.
- SQLCipher-backed SQLite via a reviewed Flutter plugin; verify copied database file cannot reveal records without key material.
- Threat model and recovery UX documented; do not store real financial data before tests pass.

### M3 — core ledger and reports
- Workspaces, accounts, categories, income/expense/transfer, exact-money rules, budgets and graph data.
- Unit tests demonstrate all totals match stored ledger records.

### M4 — statement import/export
- Local CSV first; mapping preview, duplicate checks, staged transactions, atomic commit and undo.
- CSV/export formula-injection tests and representative synthetic statement fixtures.

### M5 — full business/personal modules
- Invoices, receivables, payables, loans, assets/investments, agriculture and project tracking.
- Double-entry journal validation for posted company transactions.

### M6 — encrypted backup/restore and Drive
- Local password-protected backup and validated restore first.
- Optional Drive integration after scope/consent review; background backup only best-effort.

### M7 — trusted-device sharing and native desktop
- Local/LAN encrypted sharing; member roles and revocation tests.
- Build Windows/macOS/Linux native clients in separate CI jobs where supported; no web client.

### M8 — ads/premium, then AI
- Review privacy/policy of any ad/premium SDK; add only after the core vault is stable.
- AI stays disabled until a genuinely local/on-device model is feasible and its no-exfiltration tests pass.

### M9 — release hardening
- Full CI suite, dependency audit, secret scan, recovery drill, accessibility review, security review and release notes.
- Physical device checks remain necessary before public release.

## 15. Repository and branch discipline

- Use `main` only for reviewed stable work; use small feature branches and Draft PRs.
- Keep this blueprint, PRODUCT_SPEC, SECURITY and HANDOVER consistent.
- Update handover at the end of each milestone: changed files, tests actually run, outcomes, known gaps and next step.
- Use synthetic financial data in commits, screenshots and CI artifacts.
- Never commit Firebase service-account JSON, OAuth secrets, Android signing keys, database keys, real statements or users' data.
- Add CI policy checks for forbidden Capacitor/WebView/web targets and accidental secret/data files.

## 16. Source visibility and license

ArthTrail is proprietary and is not an open-source project. A public GitHub repository allows people to view and fork it through GitHub under GitHub's Terms of Service; a custom license does not technically prevent copying or guarantee secrecy. The repository must not contain credentials, signing keys, real user data or confidential production secrets. The custom `LICENSE` limits permission to use, modify and redistribute the project outside rights granted by GitHub/platform terms and applicable law. Third-party dependencies retain their own licenses.

## 17. Official references to re-check before implementation

- Flutter supported platforms: https://docs.flutter.dev/reference/supported-platforms
- Flutter desktop: https://docs.flutter.dev/platform-integration/desktop
- Android Keystore: https://developer.android.com/privacy-and-security/keystore
- Android passkeys: https://developer.android.com/identity/passkeys
- OWASP MASVS: https://mas.owasp.org/MASVS/
- Firebase Authentication: https://firebase.google.com/docs/auth
- Google Drive app data: https://developers.google.com/workspace/drive/api/guides/appdata
- GitHub Actions limits: https://docs.github.com/en/actions/reference/limits
- GitHub license guidance: https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository

---

**Build order is security-first:** do not build every screen before the encrypted vault and permissions work. A beautiful UI over an unprotected financial database is not an acceptable ArthTrail milestone.