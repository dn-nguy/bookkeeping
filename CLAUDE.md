# Kiln Books

Bookkeeping agent for Kiln Studio LLC (S corp, cash basis). Replaces Collective starting Jan 1, 2027.

- Plan and decisions: `docs/PLAN.md`
- Ledger: QuickBooks Online (decided Nov 10, 2026). The agent works through the QBO API; it never edits closed periods.
- Built in phases. Each phase starts only with Dawn's approval and ships as its own PR.
- Writes to QBO, Gmail, Airtable or any external system need Dawn's approval until the review queue is live.
- No secrets in the repo. Tokens live in environment variables or an encrypted store.
- Style for docs and messages: concise bullets, no em dashes.
