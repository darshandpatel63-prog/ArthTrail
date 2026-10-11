# ArthTrail — Handover

## Current project state
- Repository: https://github.com/darshandpatel63-prog/ArthTrail
- Default branch: main
- Working branch: feature/secure-foundation
- Draft PR: https://github.com/darshandpatel63-prog/ArthTrail/pull/1
- Full blueprint: blueprint.md
- Product specification: docs/PRODUCT_SPEC.md
- Security policy: SECURITY.md
- App Lock design: docs/APP_LOCK_DESIGN.md
- AI architecture: docs/AI_SYSTEM_DESIGN.md
- Recurrence/interest/reminder rules: docs/RECURRING_RULES_REMINDERS.md
- Device sync and chat: docs/MULTI_DEVICE_SYNC.md
- CA/enterprise accounting: docs/ACCOUNTING_CA_ENTERPRISE.md
- Proprietary licence: LICENSE
- GitHub Actions workflow: .github/workflows/ci.yml
- Current work includes planning, documentation and documentation CI only. The app source, Vault Lock, encrypted ledger, Firebase configuration, AI runtime, sharing/chat engine and APK are not yet implemented.
- The latest documentation/guardrail CI run passed: https://github.com/darshandpatel63-prog/ArthTrail/actions/runs/38125814505. App code checks/build were skipped because app/pubspec.yaml does not exist yet.

## Decisions that must not change without approval
- Product name is ArthTrail.
- Build-budget target is ₹0; no paid service, relay or provider without explicit approval.
- Repository is public for the workflow; ArthTrail is proprietary, not open source.
- Native/installable app only. Android first, later other native device targets.
- No Web App, no Flutter Web, no Capacitor and no WebView wrapper.
- Flutter/Dart is the planned implementation stack.
- Financial records stay local by default. Firebase Authentication handles identity/verification only.
- Specify the full AI architecture/UI/tool/security plan now, but do not add AI screens or module to the current app. Implement and integrate it only when Ritav.ai is ready and security-reviewed. No outside AI API receives finance data.
- Separate user-created Vault Lock/passphrase or strong PIN plus optional OS biometric; never send the lock secret to a server.
- Statements are downloaded and imported locally; no direct bank API in initial scope.
- Backups, Drive and device sharing are explicit user-approved exceptions; encrypt before upload/transfer.
- Recurring rules, interest, payments and reminders are deterministic algorithms with auditable events; do not rely on AI for accounting arithmetic.

## Feature expansions now in plan
- Complete AI framework and UI scaffold now; later connect the actual Ritav provider behind a hard-off switch.
- Smart recurring household/business entries: multiple vendors/times/quantities/prices, milk/vegetable examples and one-tap notification confirmation where secure.
- Interest agreements with explicit rate basis and simple/compound rules; partial, early, late and backdated payments; person-specific reminders.
- In-app floating quick calculator plus accessible color themes, text styles, screen-reader support and English/Gujarati/Hindi.
- CA mode, professional ledger/reports, client workspace isolation, audit workflows and the pathway to larger enterprise use.
- Granular workspace/module/record/action permissions; peer/device-host offline sync; optional self-hosted server; planned end-to-end encrypted team chat.
- External SMS/WhatsApp sends remain user-confirmed and provider integrations stay disabled until policy and cost review.

## Next development milestone
1. Implement Flutter Android shell, navigation and design system using synthetic data only.
2. Add recurring-rules and calculator UI scaffolding. Keep AI design in its separate spec; do not add AI navigation/module until the Ritav integration milestone.
3. Create the Dart domain layer and tests for exact money, transfers, journal balance, recurring occurrences, interest vectors and workspace permissions.
4. Implement and test Vault Lock plus encrypted local storage before importing any real financial statement.
5. Extend GitHub Actions so all non-physical tests run automatically and the native Android debug APK is built.

## Automated vs physical testing
Use CI for format/static analysis, unit/property tests, import parsers, ledger invariants, reminder scheduling logic, interest formulas, permissions, AI-off/network-egress checks, sync/conflict/replay tests, backup/restore corruption tests, UI widgets, localizations and Android builds. Tests use synthetic data.

Physical-only tests (actual Android Keystore/biometric behavior, OEM notification timing, camera QR scanning, real device-to-device networking, background/reboot behavior and restoring on a different physical device) must remain marked pending until performed on hardware. Never claim they passed based on mocks.

## Working rules
- Use feature branches and small reviewable commits; do not merge unreviewed changes straight to main.
- Keep the main blueprint and relevant specialized docs consistent after every design change.
- Keep secrets, private user data and real bank statements out of GitHub and CI artifacts.
- Report tests by their real status: passed, failed, skipped or pending.
- Avoid adding a dependency until its license, maintenance, security and network/telemetry behavior have been checked.
- Preserve a clear path for users to export and leave any sharing/server mode.