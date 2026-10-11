# ArthTrail AI System — Architecture and Security Design

**Status:** architecture/design specification only. No AI model is connected and no AI-powered financial response is currently implemented.
**Decision:** Build the complete AI subsystem, UI, permissions, local finance-tool integration and tests as an isolated module, but keep the runtime disabled until Ritav.ai is ready and has been reviewed. Do not connect a third-party model/API in the meantime.

## 1. Product goal
ArthTrail must contain an AI architecture that can later connect to Ritav.ai without redesigning accounting, security, chat UI or permissions. The AI subsystem may be built and tested now with deterministic local tools and a mock provider used only for development/testing. It must not pretend that mock responses are real AI.

## 2. Runtime switches
- `ARTH_TRAIL_AI_UI_ENABLED`: controls whether the AI screen/entry point is visible. Initially false in production builds.
- `ARTH_TRAIL_AI_RUNTIME_ENABLED`: hard kill switch. Initially false. If false, no model is initialized and no prompts/financial context are passed to any provider.
- `ARTH_TRAIL_AI_PROVIDER`: enum with `disabled`, `mock_test`, and future `ritav_local` / `ritav_approved_host`. Production must reject `mock_test` as a real assistant.
- `ARTH_TRAIL_AI_TOOL_ACCESS`: separately controls approved local finance tools and starts read-only.
- These switches must be centrally validated by secure configuration, not changed by a remote feature flag that could silently upload financial records.

## 3. Suggested package structure
```text
packages/arthtrail_ai/
  lib/src/domain/                 # request, response, source, confidence, permissions
  lib/src/provider/               # AiProvider interface and mock test provider
  lib/src/ritav/                  # isolated future Ritav adapter; no live endpoint yet
  lib/src/context/                # scoped context builder and minimization
  lib/src/tools/                  # typed allowlist of local finance tools
  lib/src/policy/                 # authorization, confirmation and safety rules
  lib/src/memory/                 # opt-in local-only memories by workspace
  lib/src/audit/                  # metadata-only audit events
  lib/src/ui/                     # chat panel, provenance, privacy status
  test/                           # fake providers, tool permissions and privacy tests
```

Do not add a network AI SDK, model API key, npm package that forwards financial records, telemetry dependency or hosted vector database now.

## 4. Provider interface
Define a model-neutral `AiProvider` interface in Dart. Its public contract should be similar to:
```dart
abstract interface class AiProvider {
  Future<AiCapabilityReport> capabilities();
  Stream<AiEvent> generate(AiRequest request);
  Future<void> cancel(String requestId);
  Future<void> dispose();
}
```
The exact types must be versioned and tested when implemented. `AiRequest` may contain a user question, a minimal context summary, a locale and typed tool definitions. It must not expose direct database handles, file paths, encryption keys, arbitrary SQL, shell/process execution or unrestricted network tools.

### Future Ritav connection
- Implement `RitavProvider` only after Ritav.ai has a stable documented local/native interface and security review.
- Prefer on-device or device-local inference. A locally hosted Ritav endpoint must bind only to a restricted interface and require authenticated pairing; never assume a LAN endpoint is private.
- If Ritav runs on a separate trusted device, use the approved ArthTrail device-sharing protocol and an explicitly enabled permission. Show which data/context is being shared.
- If Ritav requires a remote service that receives financial context, it violates the default local-only promise; keep the provider disabled unless the owner creates a clearly separate opt-in design and obtains informed user consent.
- Do not hard-code Ritav URLs, API keys or credentials in the app. Do not connect it just because a package exists; verify identity, authentication, encryption, model capabilities and data retention first.

## 5. Local finance tool registry
The assistant uses an allowlisted registry of typed functions, never direct SQL or arbitrary code. Initial tool groups may include:
- Read-only summary: balance by account/workspace/date, expenses by category, income vs expense, budgets vs actual, invoice status, overdue items, loan/interest calculations and report summaries.
- Safe calculation: EMI estimate, simple/compound interest, date/period calculations and tax arithmetic only for explicitly configured rules.
- Draft operations: prepare a transaction, invoice, reminder or journal entry as a preview; do not post it automatically.
- High-risk operations such as edit/delete/export/share/permission changes are unavailable to an AI agent unless a separate capability is expressly granted and the user confirms each action.

Each tool must declare name, version, input schema, required permission, allowed workspace scope, whether it is read-only, data sensitivity and whether user confirmation is mandatory. The tool wrapper independently validates workspace ID, role, field access, date range and amounts before querying local storage.

## 6. Model output is untrusted
- Treat all model output as suggestions until validated by deterministic domain code.
- The model cannot set ledger balances, invent totals, write arbitrary SQL, execute code or bypass access control.
- Every financial result must be produced by local calculation tools. Show the source workspace, date range, currency and a link/action to inspect the records behind a summary.
- Distinguish measured values, user-entered assumptions, forecasts and model explanations.
- Before posting or sharing data, show a human-readable preview and require the appropriate user permission and confirmation.
- Prompt injection may arrive through transaction descriptions, imported memos, document text, messages or a malicious counterparty name. Treat these as untrusted data, never as developer/system instructions.
- Limit context size and redact irrelevant personal fields. A question about a milk expense must not retrieve payroll, another company's books or unrelated chat history.

## 7. Security and privacy boundaries
- No financial data, prompt, model response containing finance data, imported statement or personal note may go to a third-party AI API by default.
- Never place vault secrets, encryption keys, Firebase tokens or private documents into a prompt.
- Context building is on-device and workspace-scoped. The user should see a privacy indicator such as `Local data only` / `Connected to Ritav on this device` / `Sharing with approved Ritav host`.
- Store conversation history locally, encrypted, and separately per workspace. AI memory is opt-in, editable and deletable; no hidden cross-workspace memory.
- Log only non-sensitive audit metadata (tool name, permission result, timestamp, approval/refusal, request ID). Do not log question text, amounts, account identifiers or full tool payloads by default.
- Provider failure, timeouts, invalid output or disabled runtime must fail closed. Financial features remain usable without AI.
- Add tests proving that AI disabled means provider initialization count is zero and no context leaves the local process.

## 8. AI UX and design
- AI page with a clear disabled/ready state, no fake responses and an explicit explanation that it will connect to Ritav only after activation.
- Suggested question chips: `Summarize this month`, `Explain this report`, `Compare selected workspaces`, `What payments are due?`—disabled until the provider is connected.
- Response cards show answer, source period/workspace, calculation provenance, assumptions and a `View records` action.
- Draft changes appear in a review sheet with `Review`, `Approve`, and `Cancel`; no one-tap uncontrolled posting.
- Give users a `Clear chat`, `Delete local AI memory`, and `Disable AI` option.
- Do not place the AI bubble above emergency actions, lock screens, import confirmation, destructive actions or authentication prompts.
- AI features use the same English, Gujarati and Hindi localization, theme, text scale and screen-reader practices as the app.

## 9. Connection acceptance checklist for the future Ritav milestone
- Ritav interface is documented and versioned, and compatibility tests exist.
- AI disabled in production by default; provider selection cannot be secretly overridden.
- No external requests contain financial context in local mode.
- Every tool is allowlisted and workspace-scoped.
- The model cannot bypass the domain service or post without confirmation.
- Prompt-injection, malformed output, permission bypass, cross-workspace leakage, tool replay and huge-output tests pass.
- UI clearly labels mock/testing vs actual Ritav provider.
- Performance and memory use are tested on supported low- and mid-range phones.
- The privacy description is updated before enabling any remote-context mode.

## 10. Implementation order
1. Create the AI domain types, provider interface, policy and tests without a live model.
2. Build the AI screen and disabled state, then the chat UI and source cards with synthetic fixtures.
3. Connect only deterministic read-only ArthTrail tools and test scope isolation.
4. Add draft-and-confirm actions behind the tool policy; keep AI runtime disabled.
5. When Ritav.ai is ready, review and add the isolated adapter; do not change finance storage to make AI work.
6. Run privacy/network tests and owner approval before enabling the provider in a release.