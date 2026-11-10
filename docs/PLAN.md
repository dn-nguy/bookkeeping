# Kiln Studio Bookkeeping Agent: Build Plan (DRAFT)

Prepared Nov 10, 2026. Replaces Collective's bookkeeping. Payroll moves to Gusto direct; tax strategy and filing move to a separate tax person (Lori or other).

## 1. Goal
- An agent that keeps Kiln Studio's books with no human bookkeeper: pulls every Chase transaction, categorizes it, matches receipts from Gmail, books payroll, reimbursements and distributions, and closes the month.
- Dawn spends about 15 minutes a month approving flagged items and signing off the close.
- The tax person gets a read-only login plus standard exports (P&L, balance sheet, general ledger, trial balance, 1099 summary).
- Running cost under $100/month.
- Live for Jan 1, 2027.

## 2. What we know (from interview and gbrain)
| Area | Fact |
|---|---|
| Entity | Kiln Studio LLC, taxed as S corp. Cash basis (as Collective books it). |
| Accounts | Chase only: 1 business checking, 2 business credit cards. No personal accounts used. |
| Revenue in | ACH direct to Chase, or Square then payout to Chase. Invoices tracked in Airtable CRM Invoice table. |
| Payroll | $40K officer salary, moving to Gusto direct. Officer health insurance paid by the S corp. Solo 401(k) deferrals. |
| Contractors | Paid via Upwork or ACH. Kyle needs a 2026 1099-NEC. |
| Reimbursements | Accountable plan: home office, phone/internet, mileage. Today Dawn submits in Collective, then transfers the amount from business to personal. |
| Distributions | Ad hoc transfers to personal. |
| Volume | About 40 transactions/month. |
| Receipts | Mostly in Gmail (dawn@kilnstudio.io). |
| 2026 size | About $261K revenue, about $85K expenses (gbrain pnl-2026). |

## 3. Key decision: ledger backend
| | A. Custom ledger | B. QuickBooks Online + agent (chosen) |
|---|---|---|
| What it is | Double-entry ledger in Supabase, web app on Vercel, agent does the work | QBO is the ledger; agent works through the QBO API |
| Bank feed | Plaid (or SimpleFIN as backup) | QBO's built-in Chase feed (free) |
| Tax person access | Read-only login to our app plus Excel exports | Free QBO accountant login, the format most preparers know |
| Monthly cost | About $30 to $60 | About $45 to $60 (QBO Simple Start about $40, verify, Intuit raised prices Aug 2026) |
| Build effort | Larger (ledger, reports, portal) | Smaller (agent only) |
| Main risk | We own correctness; a preparer may charge more if unfamiliar | Lock-in, price hikes, less control over the agent's workflow |

**Decision (Dawn, Nov 10): B. QuickBooks Online is the ledger to start.** The agent does the work through the QBO API. A custom ledger stays an option later; QBO exports keep that door open.

**Why this works:**
- QBO handles double-entry, the balance sheet (Schedule L) and closed-period locks.
- Tax person gets a free QBO accountant login, the format most preparers already use.
- QBO's built-in Chase feed replaces Plaid, so one less service to run.

## 4. Architecture (QuickBooks backend)
- **Ledger:** QuickBooks Online (Simple Start or Essentials; pick in Phase 0 based on price and whether we need bill tracking). Chase checking and both cards connected through QBO bank feeds.
- **Data sources:** QBO API (bank feed transactions, chart of accounts, reports), Square API (payouts and fees), Gusto (direct QBO integration for payroll entries), Gmail (receipts), Airtable (invoices, contractors).
- **Agent:** Claude API on a daily schedule (Vercel cron). Small model for categorizing; larger model only for unusual items. Low-confidence items go to a review queue instead of posting.
- **App:** Small Vercel web app (or a Slack/email digest) for the review queue and close checklist. Reports come straight from QBO.
- **Agent state:** Supabase (free tier) for rules, review queue, receipt index and audit log of every change the agent makes in QBO.
- **Security:** QBO OAuth tokens stored encrypted, MFA on QBO, tax person on the accountant seat.

## 5. What the agent does
| Job | How |
|---|---|
| Pull transactions | Daily from the QBO bank feed "For review" list. Card payments matched as transfers, not expenses. |
| Categorize | Rules first (e.g. Anthropic = Software), then AI for the rest. Learns from your corrections. Low confidence goes to review. |
| Revenue | Matches each deposit to an Airtable invoice and client. Square payouts split into gross revenue minus Square fees. |
| Receipts | Scans Gmail for receipts and invoices, attaches the PDF to the QBO transaction, matches by amount, date and vendor. Flags any expense over $75 with no receipt. |
| Payroll | Gusto's QBO integration posts each run (wages, employer taxes, health insurance, 401(k)). Agent checks it against the Chase debit. |
| Reimbursements | You log a reimbursement (home office monthly, phone, mileage) in the app. Agent books the payable and matches your transfer to personal. |
| Distributions | Any other transfer to personal is proposed as a distribution for you to confirm. Tracks YTD distributions vs salary. |
| Contractors | Tags contractor payments by person. Builds the 1099 list at year end. |
| Month-end close | Reconciles each account in QBO to the Chase statement balance, lists open items, drafts the P&L, sets the QBO closing date after your sign-off. |

## 6. Phases (each needs your approval to start)
| # | Phase | Output | Timing |
|---|---|---|---|
| 0 | Foundations | QBO account opened; chart of accounts set for S corp; Chase feeds connected; Collective exports pulled; repo set up | Week of Nov 16 |
| 1 | QBO connection | QBO API app, OAuth, read/write test in sandbox, agent audit log | Late Nov |
| 2 | Data feeds | Square payouts, Gusto to QBO integration, Airtable invoice matching | Early Dec |
| 3 | Categorize + review queue | Rules, AI categorizer, review UI or digest | Mid Dec |
| 4 | Receipts | Gmail scan, attach to QBO, missing-receipt flags | Mid Dec |
| 5 | Reimbursements, distributions, contractors | S corp flows above | Late Dec |
| 6 | Close + tax person access | Reconciliation checklist, closing date, accountant seat invite | Late Dec |
| 7 | Parallel run | Run December 2026 in both systems; compare to Collective | Jan 1 to 15 |
| 8 | Nice to haves | Quarterly tax reminders, cash forecast, gbrain autofill | Jan to Feb |

Build approach: I build each phase in Claude Code, you approve before and after. Needs a GitHub repo (you create it; I attach it).

## 7. Cutover plan
- **Jan 1, 2027:** New system is the book of record. Gusto direct payroll starts (clean quarter and year).
- **Keep Collective through mid-January** so it closes December 2026. Then export the final 2026 reports (P&L, balance sheet, general ledger, payroll summaries, 2026 W-2 and 941/940 status) and cancel.
- **Opening balances** in QBO on Jan 1, 2027 come from Collective's Dec 31, 2026 balance sheet.
- **2026 return:** your new tax person files the 2026 1120-S and 1040 from Collective's final exports. Confirm with them before canceling.
- **Kyle's 1099-NEC:** due Jan 31, 2027. Ask Collective if they file it; if not, use Track1099 or Gusto. Going forward, pay contractors through Gusto so 1099s are automatic.
- **Payroll year-end:** confirm who files the Q4 2026 941 and 2026 W-2/W-3 (Collective's Gusto setup or you).

## 8. Nice to haves (Phase 8)
- **Quarterly tax reminders:** Federal only (Nevada has no income tax). Agent sends reminders before Apr 15, Jun 15, Sep 15 and Jan 15 with YTD profit and a safe-harbor estimate. Your tax person sets the actual amount; the option of paying through extra Gusto withholding is noted.
- **Cash forecast:** Pulls unpaid and projected invoices from Airtable plus the expense run rate. Builds on your existing 2027 revenue model.
- **gbrain autofill:** After each close, updates the P&L page (kiln-studio/finance/pnl-2026, then a 2027 page) with a timeline entry.

## 9. Monthly running cost (estimates; verify in Phase 0)
| Item | Cost/mo |
|---|---|
| QuickBooks Online Simple Start or Essentials | about $40 to $75 |
| Vercel (Hobby is non-commercial; Pro if we need the web app) | $0 to $20 |
| Supabase free tier (agent state only) | $0 |
| Claude API at about 40 txns/month | about $5 to $15 |
| **Total** | **about $45 to $100** (stay on Simple Start and a digest instead of a web app to keep it near $50) |

## 10. Assumptions (please confirm)
1. Cash basis accounting continues (no accrual).
2. Only Chase accounts are business accounts; Square is a pass-through to Chase.
3. Gusto direct payroll is live for the first 2027 pay run.
4. Your tax person uses the free QBO accountant seat.
5. Receipts come to dawn@kilnstudio.io; anything in another inbox gets forwarded there.
6. No sales tax obligations (services only).

## 11. Risks
- **Accuracy:** a wrong category flows straight to the tax return. Mitigation: review queue, monthly reconciliation to Chase statements, tax person review at year end.
- **Bank feed breaks** (Chase reconnects): agent alerts you; CSV upload into QBO as backup.
- **QBO price increases** could push past $100/mo. Revisit the custom ledger if that happens.
- **Your time:** if the review queue grows past 15 minutes a month, we tighten rules.

## 12. Your next steps
1. Create a GitHub repo (e.g. kiln-books) and tell me; I attach it and start Phase 0.
2. Open a QBO account (Simple Start to start) and connect the 3 Chase accounts, or I walk you through it.
3. Pull Collective exports when ready: 2026 YTD general ledger, balance sheet, payroll summary.
