# ArthTrail

**ArthTrail** is a proprietary, privacy-first financial management app for Android first, with native apps planned for other devices. It is not an open-source project.

## Non-negotiable rules

- **No Capacitor. No Web App. No WebView wrapper.** Build native-capable applications only. Initial target: Android; other native platforms follow after the core vault is stable.
- The repository is public to support the GitHub Actions build workflow. Public visibility does not make the project open source or grant a general license to reuse/distribute the app; see `LICENSE`.
- GitHub currently states that standard GitHub-hosted runners are free and unlimited for public repositories. Workflow duration, queue, concurrency, API, artifact/storage and acceptable-use limits still apply; larger runners and external services may cost money. Re-check the official policy because it can change.
- Financial records, statements, invoices, balances, reports and receipts remain on the user's device by default.
- Firebase Authentication is for identity and email verification only; do not store finance data in Firebase services.
- No third-party AI API receives financial data. AI is a later phase and stays disabled until a local/on-device design and tests are approved.
- ArthTrail has its own user-created Vault Lock/passphrase, separate from account login; the secret is never sent to a server or logged.
- Backups and device sharing are explicit user-approved exceptions: encrypt locally before any upload/transfer and clearly display the destination.
- Never claim security features have passed until tests actually run and logs/results are reviewed.

## Current documentation

- `blueprint.md` — full product and engineering blueprint.
- `LICENSE` — proprietary software license draft; third-party dependencies retain their own licenses.
- `SECURITY.md` — security policy and threat model.
- `docs/PRODUCT_SPEC.md` — features, schema, UX and milestones.
- `docs/APP_LOCK_DESIGN.md` — local Vault Lock design.
- `docs/HANDOVER.md` — current status and next step.

## Platform direction

Flutter native app targets are planned so one codebase can serve Android and later iOS/Windows/macOS/Linux. Do not enable Flutter Web. No browser financial app is planned. The first development milestone is a native Android project and automated CI checks.

## Status

Documentation and design only at this stage. No app screens, encrypted vault, Firebase setup, APK or security test has yet been implemented or verified.

## Important limitations

₹0 is the initial spending target, not a guarantee that every future service is free forever. Core local finance should not require a paid service. Remote connectivity, app store accounts, commercial distribution, and third-party providers can introduce costs.

Do not commit API keys, Firebase service-account files, OAuth secrets, signing keys, real bank statements, financial records, or any private user data.