# ArthTrail — Product Specification

**Status:** Draft 2.0 — expanded specification only; application code is not implemented yet.
**Repository:** https://github.com/darshandpatel63-prog/ArthTrail
**Initial cash budget:** ₹0 target. Use local storage, free tooling and open source; never enable a billable service without the owner's explicit decision.
**Hard requirements:** No Capacitor, no Web App, no Flutter Web, and no WebView wrapper. Android is a real installable Flutter native app; additional native app targets follow after Android is stable.

## 1. Product mission
ArthTrail is a privacy-first, local-first financial manager for people, families, farmers, freelancers, shops and businesses. It will track household expenses, multiple workspaces, company books, bank-statement imports, cash, budgets, invoices, loans, assets, charts and later on-device intelligence.

## 2. Non-negotiable principles
1. Transactions, statements, notes, receipts, balances, reports and charts are stored and processed locally by default.
2. Firebase Authentication is for identity/login/verification only. Do not store financial records in Firestore, Realtime Database, Cloud Storage, Analytics or crash logs.
3. No bank APIs in the first version. The user downloads a statement and imports a local file. Parsing, field mapping, normalization and duplicate detection run on-device; uncertain rows require user review.
4. No third-party AI API receives financial records. AI is a final phase and must be on-device or an explicit, separately reviewed opt-in that clearly contradicts the strict local-only promise. Default: disabled.
5. Every export, Drive backup and sharing operation is a visible user-triggered exception to local-only storage. Backups are encrypted before they leave the device; transfer to a second device necessarily sends data to that device.
6. The app has a separate ArthTrail Vault Lock, independent of Firebase login. Never send or log its passphrase/PIN or encryption keys.
7. Security features must be tested; do not claim 'unhackable' or production-grade security without an independent review.
8. No paid database, server, AI API or relay is required by the initial version. ₹0 is the build target, not a promise that every future feature will always be free.

## 3. Platform and repository architecture
- Android first: Flutter/Dart, with native Android capabilities for local encryption, Keystore-backed key protection, biometrics and lifecycle-aware locking.
- Other devices later: evaluate native Flutter targets for iOS, Windows, macOS and Linux. Each target must pass platform-specific security and build tests before release.
- No browser application, PWA, Flutter Web output, website wrapper or WebView shell.
- Local data: encrypted SQLite/SQLCipher through a reviewed Flutter/native package; verify actual encryption in a native build.
- Key protection: Android Keystore-backed storage where available; other native platforms use the platform's appropriate secure key store.
- Shared code: Dart domain packages for money-safe calculations, ledger rules, validation, statement parsing, report calculation and translations.
- Firebase Authentication: account identity and email verification only; financial records never go to Firestore, Realtime Database, Cloud Storage or Analytics by default.
- No cloud database of financial transactions in the first release.

Suggested structure:
app/ — Flutter app, Android first and native platform folders only
packages/arthtrail_domain/ — money, ledger, validation and calculations
packages/arthtrail_storage/ — local encrypted storage and migrations
packages/arthtrail_import_engine/ — local statement parsing and duplicate detection
packages/arthtrail_backup/ — encrypted archive and restore
packages/arthtrail_security/ — vault-key boundary and security abstractions
packages/arthtrail_reports/ — chart and report calculations
docs/ — product spec, threat model, schema, sharing and test plans
## 4. Authentication and the app's own passkey
Separate the following concepts:
- Account authentication: Firebase login, email verification, reset/recovery and optional Google sign-in.
- ArthTrail Vault Lock: a user-created app-specific passphrase or strong PIN required to open the local financial vault. It must work independently of Firebase and must not be sent to any server.
- Biometrics: optional convenience unlock using Android biometric authentication/Keystore. Biometrics do not replace the encrypted vault key design.
- WebAuthn/FIDO passkey: a cryptographic login credential is not the same as a user-typed app PIN. Treat true passkey-based account login as a later feature after provider/server compatibility is verified.

Vault design requirements:
- Generate a random vault data key; do not use a short PIN directly as the database encryption key.
- Derive a wrapping key from the user secret using a reviewed password KDF with a per-vault random salt and strong work factor; prefer Argon2id where the chosen audited implementation can be safely shipped, otherwise document and test the fallback.
- Keep Android keys in Android Keystore-backed secure storage; require local authentication for unwrapping/using the key where supported.
- Rate-limit unlock attempts with increasing delays, avoid leaking whether a passphrase is correct, lock on background/inactivity and require the lock after process restart.
- Offer biometric unlock only after a secure local secret is configured. App uninstall/device loss can make local data unrecoverable; explain this before setup.
- Password reset for Firebase must not silently decrypt the vault. A separate encrypted backup password/recovery key is required to restore a backup.

## 5. Main feature scope
### Personal and family
- Income, expenses, transfers, refunds, adjustments, custom categories, tags and recurring items.
- Household budgets, member contributions, shared expenses, settlements, bill reminders, loans and EMI.
- Cash in hand and manually maintained account balances.

### Multi-business and company finance
- Multiple independent workspaces per user: personal, family, farm, freelance, business/company and projects.
- Customers, suppliers, invoices, invoice lines, payments, payables/receivables and expense records.
- Business summaries, cash flow, budget vs actual and profit/loss based on well-defined accounting rules.
- Double-entry bookkeeping for formal company transactions; every posted journal balances.
- Do not advertise GST, payroll, statutory accounting or tax compliance until rules are verified and independently tested.

### Agriculture, freelancer, assets and investment
- Crop/season, farm input, labour, equipment and crop income tracking.
- Freelance clients, projects, time/cost notes, invoices and receivables.
- Assets and dated user-supplied valuations; loans and liabilities; manually entered investment lots.
- Net worth = recorded assets minus recorded liabilities. Do not claim live market prices without a trusted, authorized data source.

### Statement import, export and backup
- Import CSV first, then XLSX/ODS, OFX/QFX and supported text-based PDFs after testing. Do not promise every bank format.
- Detect file type from content where possible, parse locally, support column mapping, date/amount locale preview, duplicate detection, row errors and import undo.
- Never upload a statement for conversion/OCR. If a format cannot be parsed reliably, ask the user to map fields or use manual entry.
- Export CSV and readable reports. Backup the local vault as an encrypted, versioned archive.
- Restore into a temporary database, validate the archive and authentication tag, preview what will change, then commit atomically or roll back.
- Never back up Firebase sessions, passwords, app lock secrets, refresh tokens, API keys or OS permissions.

### Graphs and reports
- Income vs expense, cash-flow trend, expenses by category, budget vs actual, business comparisons, profit/loss, debt trend, receivables/payables aging, asset/liability/net-worth, crop/project profitability and reconciliation difference.
- Each chart has a data-table alternative, date-range/workspace/currency filter, accessible labels and a legend. Color must not be the only way to distinguish categories.
- Clearly separate actual values from estimates/forecasting.

### Settings and languages
- Languages at launch: English, Gujarati and Hindi. Design translation files so Marathi, Bengali, Tamil, Telugu and Spanish can be added later.
- Appearance: System, Light, Dark; accent palettes such as Teal, Indigo, Emerald, Amber, Rose and high contrast.
- Text size: Small, Default, Large, Extra Large, controlled within ArthTrail. Do not require users to change phone-wide accessibility settings.
- Preferences: currency, date/time and number formats, first day of week, app lock, privacy, local files, backup destination, devices/sharing, data export and delete.

## 6. Logical database schema
Use UUID primary keys, foreign keys, migration versions and explicit timestamps. Use integer minor currency units or a tested decimal type—never binary floating point for money.

### Profile, devices and workspace
- local_profile(id, firebase_uid nullable, email, display_name, language, created_at, updated_at)
- devices(id, profile_id, display_name, platform, public_key, trusted_at, last_seen_at, revoked_at)
- workspaces(id, name, workspace_type, owner_profile_id, base_currency, timezone, created_at, archived_at)
- workspace_members(id, workspace_id, profile_id, role, permission_set_id, status, invited_at, accepted_at, revoked_at)
- permission_sets(id, workspace_id, name, can_view_transactions, can_create_transactions, can_edit_own, can_edit_all, can_reconcile, can_manage_invoices, can_view_reports, can_export, can_manage_members, can_delete_workspace)
- sharing_invites(id, workspace_id, created_by, role, permission_set_id, token_hash, created_at, expires_at, max_uses, uses_count, revoked_at)

### Finance and ledger
- ledger_accounts(id, workspace_id, code, name, account_type, subtype, currency_code, parent_id, is_system, is_active)
- categories(id, workspace_id, name, kind, icon, color_token, parent_id, is_system)
- counterparties(id, workspace_id, kind, display_name, contact_note, is_active)
- transactions(id, workspace_id, transaction_date, created_at, description, transaction_kind, source_kind, counterparty_id, reference_number, notes, created_by, import_batch_id, status, reversal_of_id, idempotency_key)
- transaction_lines(id, transaction_id, ledger_account_id, debit_minor, credit_minor, currency_code, base_amount_minor, exchange_rate, memo). For posted journal entries, debits must equal credits under the defined currency rules.
- financial_accounts(id, workspace_id, ledger_account_id, display_name, account_kind, currency_code, opening_balance_minor, opening_balance_date, is_active, last_four)
- transaction_attachments(id, transaction_id, document_id, attachment_kind)
- recurring_rules(id, workspace_id, template, frequency, next_run_date, end_date, enabled)

### Other business entities
- budgets(id, workspace_id, name, period_start, period_end, currency_code, total_minor, created_by)
- budget_lines(id, budget_id, category_id, limit_minor, warning_percent)
- bills(id, workspace_id, counterparty_id, title, due_date, amount_minor, currency_code, status, recurring_rule_id)
- invoices(id, workspace_id, invoice_number, kind, counterparty_id, issue_date, due_date, currency_code, subtotal_minor, tax_minor, total_minor, paid_minor, status)
- invoice_lines(id, invoice_id, description, quantity_decimal, unit_price_minor, tax_rate_basis_points, line_total_minor, ledger_account_id)
- loan_accounts(id, workspace_id, name, principal_minor, currency_code, annual_rate_basis_points, start_date, term_months, emi_minor, next_due_date, linked_ledger_account_id)
- assets(id, workspace_id, name, asset_type, purchase_date, purchase_cost_minor, currency_code, linked_ledger_account_id, notes)
- asset_valuations(id, asset_id, valuation_date, value_minor, currency_code, source_note)
- investment_lots(id, workspace_id, asset_name, asset_class, quantity_decimal, purchase_price_minor, purchase_date, currency_code, account_id, notes)
- project_records(id, workspace_id, project_type, name, start_date, end_date, validated_metadata)

### Import, backup and audit
- import_batches(id, workspace_id, source_filename, detected_format, file_sha256, started_at, completed_at, row_count, imported_count, skipped_count, error_count, status)
- import_staging_rows(id, import_batch_id, source_row_number, raw_fields, normalized_fields, dedupe_key, confidence_flags, status, transaction_id)
- documents(id, workspace_id, local_relative_path, mime_type, size_bytes, sha256, created_at, retention_status)
- backup_runs(id, profile_id, destination_kind, created_at, schema_version, encrypted, status, file_size_bytes, checksum, last_error_code)
- audit_events(id, workspace_id, actor_profile_id, device_id, action, entity_type, entity_id, occurred_at, before_hash, after_hash, reason)
- app_preferences(profile_id, language, theme_mode, accent_theme, font_choice, text_scale, default_currency, date_format, backup_preferences, privacy_preferences)

Store schema migrations as versioned scripts. Use typed fields and constraints for money-critical data; do not hide the entire schema in arbitrary JSON.

## 7. Import algorithm — deterministic, no AI
1. Validate file size, type and contents locally.
2. Detect format by signature/content; choose a parser from an allowlist.
3. Parse into a staging table without modifying the ledger.
4. Normalize dates, currencies, debit/credit direction, references and descriptions with a clear user-visible mapping screen.
5. Compute a duplicate fingerprint from account, date, amount, normalized description and reference when available.
6. Preview every row, warn on ambiguity and show skipped/error counts.
7. Require user confirmation, commit as one atomic batch and retain an undoable local import log.
8. Test repeated imports, malformed files, huge files, bad encodings, incorrect date locales and decimal precision.

## 8. Backup and Google Drive
- A local file backup must include only user-selected app data: workspace data, transactions, categories, settings and any selected attachments.
- Encrypt on the device before writing/uploading. Use an authenticated encryption format (for example AES-256-GCM) and a robust KDF for the separate backup password.
- Wrong password, modified/corrupt archive or wrong account must fail closed without partially replacing current data.
- Google Drive backup is optional, user-authorized and uploaded only after encryption. Use a visible folder if the user wants to manage files manually; a hidden appDataFolder is not visible in Drive UI.
- Daily backup is best-effort: the OS may defer work while the app/device is unavailable. Show last successful backup, retry and stale-backup warnings.
- Never include the vault lock secret, app login password, OAuth token, API key or active session.

## 9. QR sharing and remote access
- QR contains a short-lived, high-entropy invite token or connection info only—never financial data, password or long-lived key.
- Owner selects workspace, role, exact permissions, expiry and maximum uses. Receiver signs in and confirms permissions; owner may need to approve.
- Store invitation token hash, expire/revoke it and revoke access for a device/member.
- V1: local-only. Next: encrypted file transfer. Then same-Wi-Fi/LAN host while owner device is awake. Arbitrary internet remote access requires a reachable host or a relay/private network; do not promise it is always free.
- Sharing to another device is a deliberate exception where financial data leaves the owner's device and must be shown clearly to the owner.

## 10. UI screen list
1. Welcome/privacy explanation and language selection
2. Firebase sign-in: email/password, verification, password reset and optional Google sign-in
3. Create Vault Lock secret; confirm; optional biometrics; explain recovery risk
4. Home dashboard and workspace switcher
5. Transactions: list, search, filters, add/edit, transfer, refund and reversal
6. Accounts and reconciliation
7. Categories, budgets and recurring items
8. Local statement import wizard, preview, errors, import history and undo
9. Reports and graphs with tables, workspace/date/currency filters
10. Business workspace, customers/suppliers, invoices, payments and reports
11. Loans, bills, assets, investments and net worth
12. Backup/export/restore and optional Drive authorization
13. QR invitation, scan, pending access, permissions, trusted devices and revoke
14. Settings: language, themes, text size, security lock, privacy, currency, backup and export/delete
15. Help, privacy policy, terms, financial disclaimer
16. AI assistant only in a later phase, disabled until an audited local model approach is available.

## 11. Ads and premium — after the secure core
- Free: core finance, local import/export, graphs, manual records and encrypted backup file.
- Optional banner ads only on non-critical screens, never in transaction entry/import/confirmation or next to destructive actions.
- Interstitials only at natural transitions, conservatively limited (for example at most one per 3 hours), never at app launch or during a financial task. Follow current Google ad policies.
- Premium candidates: extra report templates, advanced comparisons, custom exports/branding, more themes and convenience tools. Do not paywall access to a user's own data or basic backup/restore.
- Monetization must not change privacy rules. Ad requests must never contain balances, categories, transaction descriptions, account identifiers or imported-file content.
- Revenue is not guaranteed. Store registration, provider quotas or hosted relay may introduce costs later; require explicit approval before enabling billable services.

## 12. Milestones and acceptance tests
### M0 — specification and UI prototype
Set up repo, theme tokens, navigation and fake demo data. No real financial records, backend database or paid feature. English/Gujarati/Hindi, light/dark, and several accent colors work.

### M1 — local finance vault
Build the Android local store, workspace separation, accounts, categories, manual income/expense/transfer and tested currency-safe calculations. No Web App target is planned. Verify the selected SQLCipher-compatible Flutter/native package in an actual native Android build.
Acceptance: data survives restart; transfer is not counted as income/expense; ₹0.10 + ₹0.20 = ₹0.30 exactly; cross-workspace tests pass; test data can be reset.

### M2 — app lock and storage protection
Implement a user-created vault secret, lock on background/inactivity, native biometric option and secure key management. Confirm wrong secret never opens the vault. Test brute-force delays and restore/re-key behavior.

### M3 — charts and statement imports
CSV first, safe preview, staging, duplicate detection and atomic import/undo; chart totals match ledger totals. Repeat import must not double-count.

### M4 — encrypted backup/restore
Versioned authenticated encrypted archive, backup password, restore preview and rollback. Tampered file and wrong password must fail safely. Drive support only after OAuth scopes and consent are reviewed.

### M5 — sharing and business modules
Workspace permission tests, invoices, loans, assets, QR invite, local network host, revocation. Remote relay stays off until threat model and cost/privacy review.

### M6 — release hardening, ads/premium, then AI
Run security review, import fuzz tests, backup restore tests and real-device tests. Add ads/premium only after policy/privacy review. AI stays off until on-device processing and privacy tests are verified.

## 13. Free-build reality
The ₹0 target is most plausible for local-first features, open-source dependencies and free development tools. It cannot guarantee free domains, app-store registration, OAuth verification, remote relay, hosted AI/OCR or provider quotas forever. Do not make the core app depend on those services.

## 14. Official technical references to re-check
- Flutter platform support: https://docs.flutter.dev/reference/supported-platforms
- Flutter Android integration: https://docs.flutter.dev/platform-integration/android
- Android Keystore: https://developer.android.com/privacy-and-security/keystore
- Android Keystore: https://developer.android.com/privacy-and-security/keystore
- Firebase Authentication: https://firebase.google.com/docs/auth/
- Google Drive appDataFolder (hidden): https://developers.google.com/workspace/drive/api/guides/appdata
- Android passkeys: https://developer.android.com/identity/passkeys

---

# Product Specification Addendum — V2 requirements (2026-10-11)

This addendum is additive: it retains all previous scope and adds the user's latest requirements. For the detailed contract, refer to the linked design documents.

## A. AI system architecture is part of the app; live AI is deferred

Specify the full AI subsystem, screen/UI, provider abstraction, permission engine, local context builder, tool registry, local conversation store, audit hooks and tests now. Do not add the AI screen/module to the current app navigation or bundle. Implement and integrate it only when Ritav.ai is actually ready and security-reviewed.

- No third-party AI provider, AI API key, cloud model, hosted vector DB or finance-data API upload now.
- Use a mock provider only inside tests for the future module; never present mock answers as real AI.
- The current app must have no AI screen/route/runtime. The separate future module has a runtime-off default and an honest not-connected state after integration.
- Future Ritav adapter must be isolated and versioned; verify authentication, capabilities, local/remote execution, privacy, prompt-injection resistance and permissions before connecting.
- AI reports must use deterministic local report/calculation tools. AI-generated write actions are drafts; the user previews and approves them.
- No arbitrary SQL, shell, payment execution, permission changes or silent transaction posting from an agent.
- Source: docs/AI_SYSTEM_DESIGN.md.

## B. Deterministic recurring-rules engine

Add configurable household, office and business routines without using AI for the accounting calculation. Milk is one example; the engine must support other repeated goods, services and obligations.

- Multiple vendors/items and multiple daily entries per rule family.
- Different units, price per unit, actual quantity, discounts/configured tax, different prices by vendor/date, and custom one-off changes.
- Notification actions: Yes/as planned, Change, Different item/vendor, Skip, Not received and Record later.
- Stable occurrence IDs to prevent double-entry on repeated notification taps, import or sync.
- An unanswered reminder is not a purchase/payment. A skipped reminder produces no financial transaction.
- Additional routine templates: rent/utilities, groceries, office supplies, raw materials, inventory, service, staff costs, invoices, project/farm costs, installments, insurance and renewal dates.
- Source: docs/RECURRING_RULES_REMINDERS.md.

## C. Interest, lending/borrowing and actual repayments

Each agreement stores parties, direction, principal/currency, start and maturity dates, rate plus explicit basis (annual/monthly/weekly/daily/custom), simple/compound method, compounding interval, day-count convention, repayment schedule and payment allocation policy.

Support partial, early, late, missed, over-, under- and backdated payment records. Interest is calculated by a deterministic, versioned formula and shown with a breakdown. Never guess the time basis of a percentage, silently apply a penalty, or overwrite the agreement history. Users can choose who receives which reminders and pause disputed obligations.

## D. Messages and payment reminders

Create reminders for the owner, selected team members or a configured counterparty. Support several lead times, selected local times, quiet hours, snooze and bounded overdue reminders.

**No third-party messaging API.** Use the user's own phone/SIM and installed SMS/WhatsApp app; ArthTrail's server does not send messages and does not incur a per-message API fee. At reminder time, a local notification opens the native SMS compose screen or a WhatsApp draft with recipient/text prefilled. The user reviews and taps Send; recipients see the user's normal number/account. Any carrier/SMS charges are governed by the user's own plan. Optional direct SMS using Android-native capabilities may be explored only as opt-in after OS permission and current distribution-policy review; if unavailable, fall back to compose-and-confirm. Never bypass OS permission or promise background auto-send on all devices. Never disclose debt details on a lock screen by default.

## E. Calculator and app accessibility

Add a floating/collapsible in-app Quick Calculator available on ordinary app pages by default, accessible from a labelled bubble and a consistent accessibility/toolbar action. Include arithmetic, percentage, ratio/proportion, quantity × unit price, unit conversion, discounts/markup, configurable tax arithmetic, bill split, profit margin, break-even estimate, EMI, simple/compound interest, date difference, outstanding balance and rounding. Label assumptions/formulas and estimates. Inputs/results must support screen readers, keyboard/focus navigation where applicable, large touch targets and text scaling controlled within ArthTrail. The bubble must not cover Save/Confirm/Delete or keyboard controls. Results only enter a form after the user explicitly taps to copy; calculations never post silently.

This is not an Android Accessibility Service and needs no global overlay permission. Essential **in-app accessibility** (semantics, screen-reader labels, contrast, large touch targets, clear errors and app-controlled text scaling) is enabled by default. Settings can change ArthTrail's text size, themes, reduced-motion preference and optional calculator bubble; these settings must not modify the phone's system-wide settings. Other optional convenience features are not all switched on by default.

## F. Accounting, Chartered Accountants and large organizations

Simple Mode remains friendly for home/family users. Advanced Accounting Mode targets businesses and CAs: chart of accounts, journal vouchers, balanced double entry, general ledger, trial balance, P&L, balance sheet, cash flow, bank reconciliation, invoices/credit notes/partial payments, receivable/payable aging, inventory, fixed assets, depreciation policy, fiscal periods, locking/reopening, cost centres, branches, projects, multi-currency, approvals, audit history and source documents.

CA features: separate client workspaces, time-limited access, reviewer/preparer roles, document requests, exceptions, draft adjustment journals, owner approvals and audit-ready exports. Never give a CA access to all clients by default. Tax/GST/TDS/payroll/statutory features must be reviewed against current requirements and by qualified professionals before claiming compliance.

Enterprise roadmap should cover subsidiaries, intercompany entries, consolidation, segregation of duties, thresholds, read-only audit role and customer-managed server mode. This is a scalable product direction, not a claim that a single mobile database can process unlimited/trillion-scale workloads. Larger deployments need benchmarks and their own hosting/operations, which may not be free.

- Source: docs/ACCOUNTING_CA_ENTERPRISE.md.

## G. Fine-grained permissions and device collaboration

Sharing is scoped by workspace type—home, family, farm, project, client, company/business or other user-defined type—and then by module, record and action. The owner can independently grant view/create/edit-own/edit-all/reverse/approve/export/manage/share permissions for balances, transactions, recurring items, loans/interest, invoices, reports, inventory, assets, sensitive salary/partner records, chat, attachments, backup and AI tools.

Modes: local-only, encrypted file transfer, same-LAN sync, trusted device host, peer sync when reachable and optional self-hosted server. A disconnected device queues signed/identified operations locally and syncs when it can reach a trusted peer/host. Internet coming back does not guarantee immediate delivery if no peer is reachable. Use idempotent changes, append-only posted ledger events, reversals and explicit conflict handling. Never use generic last-write-wins for financial postings.

Future direct/group chat should use workspace membership, end-to-end encrypted messages/attachments where implemented, offline queues and delivery state. Finance tools inside chat use the same permission engine; chat does not bypass role checks. Source: docs/MULTI_DEVICE_SYNC.md.

## H. Expanded automated tests

In addition to all previous tests, CI should automate:
- Milk/recurring action idempotency, multiple vendors/daily occurrences, changed quantity and backfill.
- Variable unit-price arithmetic, unit conversions, rounding and date/time boundaries.
- Interest vectors for annual/monthly/weekly rates, simple/compound, leap year, partial/early/late/backdated and overpayments.
- Permissions by workspace/module/record/action, including CA client isolation.
- Offline queue, reconnect, replay, conflict, revocation, host loss and chat attachment permission.
- AI runtime disabled/no-network, mock provider labeling, tool allowlist, prompt injection and confirmation gates.
- Calculator widget, notifications, localization and accessibility semantics.
- All existing ledger/import/backup/restore/security tests stay in place.

Physical-only tests (real biometrics/Keystore, OEM background notifications, QR camera and real peer networking) remain pending until actual devices are used; do not simulate them as passed.
