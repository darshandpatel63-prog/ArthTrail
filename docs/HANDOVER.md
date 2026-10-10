# ArthTrail — Handover

## Current state
- Repository: https://github.com/darshandpatel63-prog/ArthTrail
- Default branch: main
- Working branch: feature/secure-foundation
- Current pull request is documentation-only: https://github.com/darshandpatel63-prog/ArthTrail/pull/1
- Full blueprint: blueprint.md
- Product spec: docs/PRODUCT_SPEC.md
- Proprietary license: LICENSE
- Security requirements: SECURITY.md
- Vault Lock design: docs/APP_LOCK_DESIGN.md
- No app source code, Firebase project configuration, APK, test suite or security control has been implemented or verified yet.

## Decisions that must not change without approval
- Product name: ArthTrail.
- Initial spending target: ₹0.
- The GitHub repository stays public for standard GitHub Actions builds, but ArthTrail remains proprietary and is not open source.
- Native installable apps only. Android first; other native targets may follow.
- No Web App, no Flutter Web, no Capacitor and no WebView wrapper.
- Use Flutter/Dart unless a documented technical blocker justifies a reviewed change.
- Financial data stays local by default. Firebase Authentication is for identity/email verification only.
- No finance records to third-party AI APIs; AI is a late, on-device-only phase unless the privacy decision is explicitly changed.
- Build a separate user-created ArthTrail Vault Lock/passphrase or strong PIN plus optional device biometrics. Never send the lock secret to a server.
- Backups and sharing are explicit exceptions; encrypt before upload/transfer and show the destination.
- Prioritize security and exact financial arithmetic over feature count.

## Next milestone
1. Add the minimal Flutter native project under app/ and enable only Android initially.
2. Add CI in GitHub Actions for format check, static analysis, unit tests and Android debug APK build.
3. Create polished UI scaffolding using synthetic demo data, English/Gujarati/Hindi, System/Light/Dark modes, accent palettes and accessible text sizes.
4. Add Dart domain types and tests for money-safe transactions before persistent financial data is allowed.
5. Implement and test the Vault Lock and encrypted local database before importing real statements.

## Definition of done for the first code milestone
- Flutter Android app builds without Capacitor or WebView.
- No Web App/Flutter Web target is built.
- CI runs on GitHub and its actual result is reviewed.
- No real bank data, user finance records, signing keys or credentials appear in the repo or artifacts.
- Exact-money arithmetic and transfer accounting tests pass.
- Demo data is synthetic and can be reset.
- No remote finance database, AI API, ads or relay is enabled.

## Testing policy
Use GitHub Actions for every test that can be reliably automated: formatting, analysis, unit/property tests, database and import tests, backup corruption tests, permission tests, dependency/license checks and native build jobs. Real-device biometrics, OEM background behavior, camera scanning and device-specific file providers must be marked as pending until they are tested physically.

## Working rules
- Use small commits and feature branches; do not make unreviewed changes straight to main.
- Keep this file updated after each meaningful milestone.
- Never state that a test/build passed unless its result is visible and reviewed.
- Review dependency code/license and network/telemetry behavior before adding packages.
- Never add a paid service or enable billing without the owner's explicit approval.