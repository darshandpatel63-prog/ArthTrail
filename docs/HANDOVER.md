# ArthTrail — Handover

## Current state
- Repository: https://github.com/darshandpatel63-prog/ArthTrail
- Default branch: main
- Working branch: feature/secure-foundation
- main was empty at the start. A minimal README was added to main, then a feature branch was created.
- Product specification: docs/PRODUCT_SPEC.md
- Security requirements: SECURITY.md
- No application source code, Firebase project configuration, Android APK, Web build or automated test has been implemented yet.

## Decisions that must not be changed without approval
- Product name: ArthTrail.
- Initial cash budget target: ₹0.
- Do not use Capacitor. Do not wrap the website in WebView. Android must be a native-capable React Native/Expo app.
- Financial records stay local by default. No financial records in Firebase or external AI APIs.
- Firebase Authentication is for account identity/email verification only.
- Add a separate user-created ArthTrail Vault Lock (passphrase/strong PIN) plus optional device biometrics; never send the vault secret to a server.
- AI is a later phase, not a dependency for the first release.
- User-requested backup/Drive and device sharing are explicit exceptions to local-only data; encrypt before upload/transfer and clearly disclose the destination.
- Prioritize security and deterministic money calculation over feature count.

## Next milestone
1. Add monorepo skeleton with apps/web, apps/mobile, packages/domain, packages/import-engine, packages/backup-format and tests.
2. Make a polished UI prototype with synthetic data only, English/Gujarati/Hindi, Light/Dark and multiple accent palettes.
3. Implement shared money types and unit tests before connecting any persistent store.
4. Implement encrypted local storage on Android (SQLCipher native build) and browser IndexedDB encryption, documenting the difference in threat model.
5. Add the Vault Lock flow and test it before importing any real financial information.

## Definition of done for the first code milestone
- Android app builds without Capacitor and without a WebView wrapper.
- Web app works separately.
- No real bank files or credentials in the repository.
- Basic workspace, account and transaction model compiles with automated tests.
- Transfers do not inflate income/expense; decimal amounts use exact arithmetic.
- Test data is visibly fake and can be reset.
- No external AI API, remote finance database, ads or remote relay is enabled.

## Working rules
- Use small commits and feature branches; do not add unreviewed code straight to main.
- Keep this file updated at the end of a meaningful milestone.
- Do not claim a build/test ran unless it actually ran.
- Before adding dependencies, review current official documentation, licence, security posture and network/telemetry behavior.