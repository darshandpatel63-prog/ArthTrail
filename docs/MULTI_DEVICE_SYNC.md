# ArthTrail — Multi-Device Sync, Device-as-Server and Workspace Chat

**Status:** architecture only. Peer sync, host mode and chat are not implemented.

## 1. Modes the user can choose
1. **Local-only (default):** everything stays on one device; no remote account or sync required for core local bookkeeping.
2. **Direct local transfer:** export/import an encrypted pack between trusted devices; useful when no always-on host exists.
3. **Trusted device host:** a phone, tablet or computer can be selected as the user's own server/host. It serves only approved workspaces while awake/reachable, after explicit local authentication and approval.
4. **Local network peer sync:** trusted devices on the same Wi-Fi/LAN exchange encrypted change sets directly.
5. **Internet peer/mesh mode:** devices can exchange changes across networks when a reachable peer/host or suitable direct connection exists. Direct connection may fail behind carrier NAT/firewalls; a relay/private-network service might be required. Do not claim a guaranteed zero-cost global mesh.
6. **Self-hosted server:** a technically capable user or business can install the server on their own computer/NAS/VM and keep authoritative shared workspace data on their own infrastructure. Installation, secure updates, backups, ports, certificates and uptime become the owner's responsibility.

Settings must explain the location of data for each mode and show a clear badge: `This device only`, `Shared with named devices`, or `Stored on your self-hosted server`. Never silently change modes.

## 2. Device-as-server reality
A phone can act as a server only while it is running and reachable. If all devices are offline, no new update can be delivered to another device until connectivity returns. If a user's phone has no internet, that phone can still record changes locally; those changes queue and are sent when it reconnects to an online reachable peer/host. Instant remote updates are best effort, not guaranteed under Android battery, network, NAT, firewall and device-sleep restrictions.

If another device is online but cannot reach the offline phone or any copy of its queued changes, it cannot learn about those changes yet. A relay or self-hosted always-on server solves this only by adding a reachable service/device. The app must display `Pending to sync` rather than falsely showing a change as synced.

## 3. Data architecture
- Every change is first committed locally and then placed in an encrypted local outbox.
- Outbox operation IDs are globally unique and idempotent. Store originating device ID, device sequence number, workspace ID, schema version, entity type, operation type, timestamps and cryptographic integrity metadata.
- Only approved workspace data is included in a sync package. Device auth, vault keys, app passwords and hidden personal workspaces never sync through another workspace's membership.
- Sync packets use end-to-end authenticated encryption between approved devices. A relay may forward ciphertext but must not be handed decryption keys. Minimize routing metadata as well.
- Peer identity is verified during pairing and each device has independently revocable credentials.
- When a peer reconnects, exchange versioned change cursors, missing operation IDs and acknowledgements; retry with bounded exponential backoff.
- Importing a repeated operation must not duplicate a transaction, payment, chat message or milk-delivery confirmation.

## 4. Conflict resolution for financial records
Do not apply generic last-write-wins to money or posted accounting records.
- Posted journal/transaction history is append-only. Corrections use reversal/adjustment events with author and reason.
- Each operation has a stable ID and entity version; handlers are idempotent.
- Conflicting drafts, recurring-rule edits, contact info and non-financial preferences can use a documented field-level merge or present a conflict UI.
- If two offline devices change the same invoice/payment allocation or interest agreement, mark a conflict and require explicit resolution instead of guessing.
- Account balance and reports are recalculated from accepted ledger events; don't sync cached balances as the source of truth.
- Time stamps alone do not decide financial truth. Clock drift, backdated entries and late sync must be handled.
- Reconciliation screen shows local/synced/pending/conflict states, last successful peer contact and operation counts.

## 5. Pairing by in-app QR scanner
1. Owner chooses workspace, recipient, device role, exact permissions, expiry and whether the invite allows sync, chat, exports or attachments.
2. Create a high-entropy, short-lived invite; QR contains only pairing metadata and a temporary token, never financial data or a long-lived private key.
3. Receiver scans only within ArthTrail, sees the workspace and permissions in plain language and approves the request.
4. Both devices authenticate each other and show a key-fingerprint/safety phrase for confirmation where practical.
5. Owner sees the pending device and approves/rejects. Pairing becomes trusted only after authenticated key exchange.
6. Access can be revoked by workspace or device. Expired, consumed and revoked invites fail closed.

Already downloaded data cannot be guaranteed erased from a previously authorized/untrusted device after revocation. Revoke future access, stop new sync, invalidate credentials and clearly explain residual local copies.

## 6. Permission granularity
Permissions apply independently to each workspace type: home/family, a named company, farm, project, client or other user-created workspace.

Within a workspace, the owner can switch modules and actions independently: dashboard, account balances, transaction list/details, add entries, edit own entries, edit all, delete/reverse, budgets, invoices, loans/interest, salary/payroll, assets, reports, export/backup, chat, attachments, manage members and administer settings.

Add record-level or field-level restrictions for high-sensitivity areas such as salary, owner drawings, personal loans and partner capital. Default deny. Permission is checked at the data/domain function, not just by hiding controls. All permission changes are audit logged.

## 7. Chat and collaboration
The app may later include private direct and workspace/group chats for owner, CEO, manager, employee, accountant and suppliers/partners where invited.
- Chats are scoped to a named workspace and membership. Direct messages do not imply access to company financial data.
- Use end-to-end encryption between trusted clients if implemented. Messages and attachments stay locally encrypted at rest. Do not store plain message content in relay logs.
- Sync message envelopes across allowed devices; offline outgoing messages remain queued with visible delivery state.
- Attach a transaction/invoice/report only through a preview and explicit confirmation, with the exact fields shown. Do not automatically expose financial tools or reports to every chat member.
- Chat retention, message deletion, device revocation, export, file size/virus scanning (without cloud content upload by default) and legal retention need separate policies before release.

### Finance tools inside chat
Use the same tool registry described in `AI_SYSTEM_DESIGN.md`, not a second finance engine. A user could type a command such as `show overdue invoices for Workshop A` or request a draft expense. The command engine only offers actions that the current sender already has permission to use.
- Read-only reports can return a locally calculated summary if the user has access.
- Posting a transaction, recording payment, changing interest terms, exporting records or inviting a user needs an action preview and confirmation.
- Other recipients see only data they are entitled to see; the sender cannot use chat as a permission bypass.
- When the future Ritav AI is connected, it may help interpret a request, but deterministic tools and role checks remain authoritative.

## 8. Own server option
The user can select a server they own. Initial server mode should be optional and documented, never mandatory for basic local finance.

Server responsibilities if implemented:
- Authenticate each device independently; TLS plus a well-reviewed end-to-end encrypted payload format.
- Enforce workspace membership/permissions server-side before storing/forwarding data.
- No shared master token embedded in the app. Per-device credentials can be revoked.
- Encrypted backups, restore drills, patching, secure port exposure, rate limits, audit events and availability monitoring.
- Document who controls encryption keys and whether server admins can read financial data. Prefer client-side encryption where the server must not read values.
- Provide database migrations and export/exit path so the customer is not locked in.
- Optional servers have uptime and maintenance costs. If a company chooses to run one, those are its responsibility; remote access is not guaranteed free forever.

## 9. Sync security threat tests
- Pairing code guessed, reused, expired, replayed or intercepted.
- Unauthorized workspace/member/device attempts, after role downgrade and after revocation.
- Queue tampering, duplicate operation delivery, dropped acknowledgements and out-of-order messages.
- Two offline devices record the same invoice/payment; no silent double count.
- Clock drift, backdated entries, timezone changes, reboot and storage-full cases.
- Host disconnects mid-transfer, resumes later, enters sleep or changes network.
- Packet capture confirms finance content is not visible to a relay/unauthorized peer.
- Chat recipient not permitted to see attached report or invoice.
- No financial data is broadcast on LAN discovery, QR content or logs.
- Offline, local-only use still works when every network permission is denied.

## 10. Rollout order
Phase 1: local-only outbox/event model and export/import between devices.
Phase 2: same-network trusted direct sync and pair/revoke UI.
Phase 3: encrypted peer sync over internet only where reachability and security pass tests.
Phase 4: optional own-server/self-hosted sync.
Phase 5: end-to-end encrypted chat and shared-workspace attachments.
Do not make public release depend on phases 3–5 or on a paid relay.

## 11. Platform references
- Android persistent work and retry: https://developer.android.com/develop/background-work/background-tasks/persistent
- Android background restrictions: https://developer.android.com/develop/background-work/background-tasks/bg-work-restrictions
- Android Keystore: https://developer.android.com/privacy-and-security/keystore