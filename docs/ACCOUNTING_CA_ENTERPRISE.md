# ArthTrail — Business Accounting, Chartered Accountants and Enterprise Scale

**Status:** product requirements. Statutory, GST and tax behavior must be validated separately before being presented as compliant.

## 1. Design from a sole trader to multi-entity enterprise
One account should be able to create separate, isolated workspaces for home, farm, freelance work, sole proprietorship, partnership, company, branch, client, project or investment. The same app navigation can offer simple and advanced modes without forcing every household user to see a corporate ledger.

Required levels:
- **Simple:** home expenses, receipts, money owed, recurring purchases and budgets.
- **Micro/small business:** sales, purchase, stock, contacts, bills, invoices, receivables/payables, cash flow and profit summaries.
- **Professional accounting:** chart of accounts, journal vouchers, general ledger, subledgers, trial balance, reconciliation, period close and financial statements.
- **Multi-entity:** subsidiaries, branches, departments, cost centres, projects, currencies, approval workflows and controlled consolidation.
- **Enterprise-ready pathway:** clearly defined data partitioning, tenant/entity isolation, versioned ledger events, backup/restore, audit history, import/export and optional server deployment.

A single offline Android database cannot honestly promise production-grade availability or performance for a trillion-rupee company or billions of entries. The product model should support complex workspaces and a migration path to an organization's own server or approved future infrastructure; enterprise deployment requires load tests, operational controls, support and possibly paid hardware/services.

## 2. Professional accounting modules
- Custom chart of accounts with parent/child accounts and account types.
- Journal vouchers: general, sales, purchase, receipt, payment, contra, credit note, debit note, adjustment, opening and reversal.
- General ledger and subledgers for customers, suppliers, bank/cash, tax ledgers, loans, inventory and fixed assets.
- Trial balance with debits/credits and difference diagnostics.
- Profit & Loss/Income Statement, Balance Sheet and Cash Flow reports for selected periods and entities.
- Bank reconciliation against imported statements; match/partial-match/unmatched and explanation history.
- Accounts receivable/payable aging, customer/supplier statements, due dates, payment receipts and credit management.
- Accrual/prepaid expenses, depreciation and fixed asset register using configured policies.
- Inventory/stock valuation, stock movement, reorder, batches/expiry where relevant; manufacturing/BOM only in a later module.
- Budgets, actual-vs-budget, forecasts with assumptions, cost centers, project costing and branch-level comparisons.
- Fiscal year setup, opening/closing balances, period close/lock, controlled reopening and audit log.
- Attachments/evidence per voucher with local encryption; reports can reference source documents.
- Multi-currency support: transaction currency, functional/base currency, exchange rate source/date, rounding and realized/unrealized FX handling under the selected accounting policy.
- Consolidation and intercompany entries in an enterprise phase; eliminate intercompany balances under explicit rules.

## 3. India-focused workflow — configurable, not falsely certified
Potential later modules include GST tax fields/reports, input/output tax reconciliation, TDS records, payroll/employee costs, invoice numbering, e-invoice/e-way bill integrations and return-preparation exports. Each must be designed against current official requirements and effective dates; laws/rates change. Do not imply official GST portal integration, tax filing, legal compliance or a Chartered Accountant's sign-off until required integrations, audit and professional review exist.

At first, make tax fields/rates explicit user configuration and label outputs `management report / draft for review`, not `filed return` or `certified accounts`. Support country/jurisdiction per workspace as a future schema concept, but do not ship all countries' tax rules as if universal.

## 4. Chartered Accountant (CA) workspace
CA users should be able to manage multiple clients without mixing the ledgers:
- Invite a CA into only the named client workspace with read-only, accountant, reviewer or preparer role.
- Configure an engagement period and access expiry; client can revoke access.
- Import a client's statement or trial balance into a staging area and map accounts without silently overwriting books.
- Ask for documents using a checklist, mark received/missing, attach evidence locally or in an explicitly selected shared location.
- Review exceptions: unreconciled bank rows, unbalanced journals, duplicate invoices, missing references, overdue items and unusual changes.
- Propose adjusting journals as drafts; the client owner or authorized accountant approves posting according to role policy.
- Review/annotate reports and leave comments tied to a specific period, voucher or report.
- Export audit-ready ledgers/trial balances/financial reports and source-document index, with user-selected columns and a clear export warning.
- Track review state, engagement history, reviewer identity and accepted/rejected adjustments.
- Separate client data by workspace; a CA's membership in Client A gives no default access to Client B.

## 5. Corporate governance and permissions
Fine-grained role templates: owner, director/CEO, finance manager, accountant, accounts payable, accounts receivable, inventory, HR/payroll (future, restricted), branch manager, auditor/CA, employee and read-only viewer.

Permissions should control not only module visibility but actions and data fields: view balances, view individual transactions, create draft, post, edit own, edit any, reverse, reconcile, approve, export, invite, manage permissions, reopen closed periods and access salary/partner capital/personal loan records.

Separation of duties: optionally require a second approver for large payment records, period reopenings, bank-account changes, journal entries, export of sensitive records and new member invitations. Approval threshold and currency are workspace-configurable. Until actual payment execution exists, these are accounting approval records only; they do not initiate a bank transfer.

## 6. Enterprise audit and controls
- Append-only event history for posted accounting facts; corrections by reversal/adjustment.
- Period locking, close checklist and controlled re-open reason.
- Approval workflows and separation of requester/approver where configured.
- Audit export of who did what and when without adding secrets or unnecessary personal data.
- Data retention and legal hold options only after a reviewed policy is defined.
- Encryption at rest/in transit, tenant isolation tests, user/device offboarding, revocable sessions and backup restore rehearsals.
- Optional read-only audit role; CA/auditor can inspect but cannot silently change evidence.
- Import templates and integrations are versioned and carry source, time and reconciliation status.

## 7. Data and performance architecture
The local-first native app remains the source for personal/offline use. Shared teams use the explicit peer or self-hosted sync design. Keep ledger/domain logic independent from storage backend so a workspace can migrate, after owner consent and verified export, from local-only to a self-hosted store in a later version.

Scale path:
1. Personal/small businesses: one local encrypted database per device and encrypted backups.
2. Teams: multiple replicas with a bounded workspace event log and controlled sync.
3. Larger companies: own server with a transactional SQL database, background job processing, monitoring, per-tenant/entity partitioning, tested restore, key management, secure updates and admin controls.
4. Regulated/global enterprise: reviewed architecture for high availability, disaster recovery, regional requirements, audit retention, concurrency, load, migration and contractual support.

Do not switch users to a cloud database without explicit consent. Server mode must explain whether server operators can see plaintext values. A strictly end-to-end encrypted server may limit server-side reporting; choose and document this trade-off.

## 8. Reports CA and management users expect
- Trial balance; ledger and voucher register.
- P&L, Balance Sheet and Cash Flow.
- Bank reconciliation and unresolved differences.
- Receivables/payables aging and counterparty statement.
- Sales/purchase register and tax summary for configured rules.
- Inventory and stock movement/valuation.
- Fixed assets/depreciation schedules according to configured policy.
- Cost-centre, branch, project and department reports.
- Budget vs actual, cash forecast and scenario assumptions.
- Intercompany balances and consolidated reports when multi-entity is implemented.
- Audit trail, changes after close, missing documents and exception register.

## 9. Required accounting validation
- Double-entry is balanced on each posted journal.
- Every report totals to the underlying ledger and agrees with known synthetic cases.
- Foreign currency is explicit; rates, precision and rounding are deterministic.
- Credit notes, refunds, partial payments, bad-debt adjustments and reversals do not double count.
- Opening/closing balance and period lock/reopen logic is tested.
- Unbalanced, unauthorized or backdated-to-closed-period entries fail or require an authorized override and audit reason.
- User-configured tax rules are versioned by effective date and never assumed global.
- A qualified accountant reviews accounting behavior before claims of professional/statutory compliance.

## 10. Boundaries
ArthTrail is record-keeping and management software, not itself a bank, lending service, tax authority, auditor or substitute for professional advice. Interest projections depend on the terms entered by the user. Do not label calculations legally enforceable or compliant without a suitable review.

## 11. Workflow enhancements informed by accounting products

These augment the earlier accounting requirements:
- Quote/estimate approval and conversion into an invoice, then credit notes, partial payment allocations, customer statements and overdue balance/ageing views.
- Source-document attachments on invoices, purchases, journals, expenses and reconciliation differences.
- Inventory movement history, explicit units, reorder threshold and traceable valuation policy.
- Saved reports and exports scoped to one entity/client; a CA must not receive access to other workspaces by default.
- Multi-currency exchange-rate source/date and deterministic rounding logs; never silently substitute a rate.
- User-facing dashboards may show business KPIs but every value must drill through to the underlying ledger/report and disclose its period and assumptions.
- Tax/GST/e-invoice/return functionality stays labelled as unverified until current official requirements, effective dates, sample vectors, integrations and qualified accountant review are complete.
