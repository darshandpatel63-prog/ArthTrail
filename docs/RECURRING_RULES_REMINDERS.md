# ArthTrail — Recurring Rules, Household Routines, Interest and Reminders

**Status:** planned deterministic rules engine; no AI required.

## 1. Purpose
Users should not have to re-enter a repeated item every day. The app will use clear rules and templates to prepare a transaction, ask for confirmation at the chosen time, allow changes for that occurrence and keep a complete record of what was actually received or paid.

## 2. General recurring-rule engine
Each rule has:
- Name and purpose (for example `Evening milk`, `Office tea`, `Workshop rent`, `Monthly installment`, `Customer interest due`).
- Workspace and optional location/branch, counterparty/vendor, product/service, category and account.
- Frequency: once, multiple times per day, daily, selected weekdays, weekly, monthly, yearly, selected dates, every N days/weeks/months, or a custom interval.
- Start/end date, time zone, reminder lead time, skip days/holidays, pause/resume, and an optional end condition.
- Default quantity, unit, rate/price method, tax/discount fields where required, currency and rounding rules.
- Who is reminded and who may confirm/change/cancel the occurrence.
- Per-occurrence actions: `Yes / As planned`, `Change quantity/amount`, `Different item/vendor`, `Skip today`, `Not received/paid`, and `Record later`.
- A history of generated occurrences and actual user confirmations. A rule must not itself silently claim something was received or paid.

Rules and calculations are deterministic. AI may later help the user draft a rule, but the user sees and confirms every field before the rule becomes active.

## 3. Household example: milk, produce and daily supplies
### Milk subscriptions
A user can configure several vendors/items separately—for example Vendor A, 1 litre at a selected morning time; Vendor B, 500 ml in the evening; and a third flexible purchase from a shop. More than one entry per day is supported.

When a reminder arrives, the notification may offer `Yes, as planned`, `Change`, `Skip`, and `Not received`. `Yes` creates a local proposed/confirmed occurrence using the configured quantity and price, without requiring the user to navigate to the app. The app must store the event idempotently and avoid duplicate records if the notification is tapped twice.

### Variable price and quantity
- Fixed price per unit: e.g. litre, 500 ml, kg, gram, piece, bundle, packet or service-hour.
- Actual price per purchase: price changes day-by-day or shop-by-shop.
- Quantity × unit price, discounts, delivery charge, taxes where configured, and explicit rounding.
- Support weighted produce: quantity and actual rate can both vary; allow multiple vegetables from multiple sellers in one day and different categories/accounts.
- `Different item/vendor` starts a quick custom entry; no assumption that all milk/vegetables share one price.
- If the user says `Skip` or has not answered, do not book a purchase. Show pending/missed reminders separately. Users may enter the real event later with its actual date and mark it as backfilled.

### Notification action security
- Quick action may store a small local event while the app is in the background, using the native notification action/background entry point. Implement and test this on real Android builds; operating-system/vendor limits apply.
- Default lock-screen notification text should avoid showing sensitive balances or personal debt details.
- User chooses whether a quick `Yes` action can work while the vault is locked. Safer default: queue a minimal pending confirmation and require vault unlock before committing financial data, unless the user explicitly enables a tested secure quick-entry mode.
- Local reminder timing is best-effort; battery restrictions, Doze, reboot and notification permission can defer or suppress notifications. Provide a reminder health screen and in-app fallback list.

## 4. Other templates for home, office and business
Built-in templates can include:
- Rent, electricity, gas, phone/internet, school/college fees, groceries, subscriptions and regular deliveries.
- Office meals/tea, cleaning supplies, transport, rent, equipment service, software subscriptions and vendor bills.
- Daily/weekly sales, raw materials, stock reorder, production consumption, labor, delivery, fuel and service jobs.
- Customer/supplier invoice due date, partial payment, installment, loan EMI, salary, contractor invoice and tax-filing prep date.
- Project milestones, crop/season expenses, seed/fertilizer orders and equipment maintenance.
- Insurance, property tax, annual renewals and contract expiry.

Templates are examples, not pre-filled factual obligations. The user must configure amounts, dates, recipients and recurrence.

## 5. Interest and lending/borrowing ledger
Create a dedicated loan/advance/receivable record whenever money is borrowed or lent. At minimum record:
- Direction: `I lent`, `I borrowed`, `they owe me`, `I owe them`.
- Person/company, contact reference, workspace and responsible people.
- Principal, currency, agreement date, due/maturity date, repayment schedule and notes/document evidence.
- Rate value and explicit basis: annual %, monthly %, weekly %, daily % or another agreed period; do not infer the basis from a bare percentage.
- Method: simple interest or compound interest; compounding frequency and day-count convention where applicable.
- Whether days use actual/365, actual/366, 30/360 or another agreement-specific convention.
- Payment allocation rule, chosen explicitly: interest first, principal first or manual allocation. Do not silently decide when principal and interest are paid.
- Grace period, late-fee rule only if explicitly agreed, rounding rule and optional reminder schedule.
- Schedule: one-off due date, weekly, fortnightly, monthly, quarterly, yearly, custom interval, flexible/early repayment, or manually entered actual payment dates.

### Formula policy
For simple annual interest under an actual-days/365 convention, the estimate can be `principal × annual_rate × actual_days / 365`. Other conventions require a different documented formula. Compound interest applies only when the contract/rule explicitly selects it, and the compounding frequency is recorded. A stated 2% weekly must never be interpreted as 2% annual.

Provide an interest breakdown: opening principal, principal repaid, interest accrued, interest paid, interest outstanding, fees (separately labelled), next due date and how each figure was computed. This is an accounting calculation based on user-entered terms, not a legal opinion about enforceability.

### Real-world repayment problems
- Partial payment: record the actual amount/date and allocate according to the saved agreement or explicit user choice.
- Early payment: recalculate future interest only under the chosen agreement convention; retain the pre-change and post-change schedules.
- Late payment or missed payment: mark overdue; do not silently add a penalty or compound interest unless the configured agreement calls for it.
- Forgot to record an earlier payment: allow a backdated payment with a correction reason and audit trail. Recalculate outstanding amounts from recorded events.
- Overpayment: show the surplus as unallocated credit until user chooses how to handle it.
- One person has several loans/advances: keep agreements separate while offering a consolidated contact summary.
- Multiple responsible people: control who sees a reminder, who is allowed to record payment and who can approve adjustments.
- Payment dispute: mark disputed and freeze automated reminders/actions until reviewed; preserve all prior records.

## 6. Reminders and follow-up messages
Reminder types: upcoming installment, interest payment/collection, rent/bill, receivable, payable, overdue amount, contract renewal, recurring purchase, approval request, reconciliation and user-defined event.

Every reminder supports multiple lead times (for example 7 days, 1 day and at due time), preferred local time, repeat interval after overdue (with caps), quiet hours, escalation person, snooze, dismissal reason and delivery status.

### Message channels
- In-app notification and local Android notification: first choice and designed to work without cloud service.
- SMS: the first version opens the phone's default messaging app with recipient and text prefilled, and the user taps Send. Do not read SMS or send silently. Direct programmatic SMS can trigger sensitive-permission/Google Play default-handler restrictions; do not request those permissions for a simple finance tracker without a reviewed policy path.
- WhatsApp: open a WhatsApp conversation/deep link with a user-approved prefilled message, where supported; the user reviews and taps Send.
- Automated WhatsApp Business Platform messaging is a separate integration, with account/provider setup, user consent/template requirements and potentially per-message charges. Keep disabled under the ₹0 launch target unless the owner explicitly approves fees and compliance work.
- External messages can reveal debt or financial details to someone else. Show recipient name/number, message preview and confirm before sending. Never send a reminder to the wrong person because a contact changed.
- Scheduled external auto-send is not promised in the free version. Use scheduled local reminders that require user approval to open/send the message.

## 7. Scheduler architecture
- Store recurrence rules and generated occurrence IDs locally.
- Generate upcoming occurrences deterministically over a bounded horizon, using stable IDs to avoid duplicates across reboot/restore/sync.
- Use native local notification scheduling for reminders; use Android background scheduling for maintenance/retries where appropriate, without assuming an exact execution time.
- Track states such as upcoming, notified, confirmed, adjusted, skipped, missed, backfilled, paid, partially paid, overdue, disputed and cancelled.
- Sync rule updates and occurrence events using stable operation IDs; avoid generating separate duplicate charges on different devices.
- Provide a calendar/list view, filters by workspace/person/vendor and a dashboard for today's confirmations and overdue items.

## 8. Data model extensions
- recurring_rule: workspace, rule_type, name, schedule, timezone, start/end, enabled, owner, role policy, template JSON/version.
- recurring_occurrence: rule_id, occurrence_id, planned_at, status, source_device_id, confirmed_at, confirmed_by, actual quantity/amount, created_transaction_id, skipped/missed reason.
- catalog_item: workspace, name, unit, default category, default price method, tax settings.
- vendor_price_history: item, vendor, effective date, unit price, currency, source/confirmation.
- loan_agreement: parties, direction, principal, rate/basis, simple/compound, day-count, compounding interval, payment allocation rule, schedule and agreement version.
- loan_payment: agreement, received/paid date, actual amount, principal allocation, interest allocation, fee allocation, backdated flag and correction reason.
- interest_calculation_snapshot: agreement version, calculation window, principal base, days, formula/convention, computed amount, engine version and audit hash.
- reminder_rule, reminder_delivery, message_draft and reminder_assignment with recipient/channel/preferences and delivery status.

Sensitive message bodies and financial amounts should not be copied into general notification logs. Support export, backup and migration for all of these records.

## 9. Required tests
- Milk reminder with multiple daily vendors/times/quantities, yes/change/skip/no-response and duplicate taps.
- A vegetable price changes by vendor/date; correct quantity × price and totals.
- User backfills a missed delivery without duplicating it.
- Annual/monthly/weekly rates are never confused; simple/compound results match known calculation vectors.
- Partial, early, late, over- and unallocated payments; leap year and daylight-saving/time-zone date boundaries.
- User chooses who gets which reminders; revoked users don't receive future locally-generated share payloads.
- Message preview, wrong-number protection, quiet hours and no external send until the intended user approves.
- Offline scheduler, reboot, notification permission denied, OEM restrictions, app locked and device restored from backup.

## 10. Official technical references
- Android persistent/background work: https://developer.android.com/develop/background-work/background-tasks/persistent
- Android background restrictions: https://developer.android.com/develop/background-work/background-tasks/bg-work-restrictions
- Google Play SMS/Call Log policy: https://support.google.com/googleplay/android-developer/answer/10208820
- Keep external messaging adapters disabled until current provider policies and prices are re-checked.