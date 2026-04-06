---
name: maton-qbo
description: "Manage QuickBooks Online bookkeeping via Maton API gateway — query transactions, categorize expenses, review uncategorized items, check account balances, and run weekly transaction reviews. Use when user asks about bookkeeping, expenses, QBO transactions, categorization, financial review, or anything involving QuickBooks. NOT for invoicing (use QBO UI) or payroll (use Gusto)."
---

# maton-qbo — QuickBooks Online via Maton API

Manage QBO bookkeeping through the Maton API gateway. Query transactions, categorize expenses, review bank feeds, and automate weekly reconciliation.

## Architecture

```
Agent ──HTTP──▶ Maton Gateway ──OAuth──▶ QuickBooks Online API
```

Maton handles OAuth token refresh. You just need an API key.

## Prerequisites

- Maton account with QBO integration connected
- API key stored securely (e.g., macOS Keychain)
- QBO company connected via Maton dashboard

## API Basics

- **Base URL:** `https://gateway.maton.ai/quickbooks`
- **Auth:** `Authorization: Bearer <maton-api-key>`
- **Accept:** `application/json`
- **Path pattern:** `/quickbooks/v3/company/:realmId/{resource}` — Maton auto-replaces `:realmId`

### Query syntax (QBO SQL-like)
```bash
# GET with query parameter
curl -s -H "Authorization: Bearer $KEY" -H "Accept: application/json" \
  "https://gateway.maton.ai/quickbooks/v3/company/:realmId/query?query=SELECT%20*%20FROM%20Purchase%20WHERE%20TxnDate%20%3E%3D%20'2026-03-30'%20ORDERBY%20TxnDate%20DESC"
```

### Common queries
```sql
-- Recent purchases (last 7 days)
SELECT * FROM Purchase WHERE TxnDate >= '2026-03-30' ORDERBY TxnDate DESC

-- Recent bills
SELECT * FROM Bill WHERE TxnDate >= '2026-03-30' ORDERBY TxnDate DESC

-- All accounts
SELECT * FROM Account

-- Vendor list
SELECT * FROM Vendor

-- Specific transaction by ID
SELECT * FROM Purchase WHERE Id = '123'
```

## Transaction Categorization

QBO transactions have line items with `AccountRef` (expense category). Common pattern:

```json
{
  "Line": [{
    "DetailType": "AccountBasedExpenseLineDetail",
    "AccountBasedExpenseLineDetail": {
      "AccountRef": { "value": "65", "name": "Meals" }
    },
    "Amount": 25.50
  }]
}
```

### Updating a transaction category
```bash
curl -s -X POST \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  "https://gateway.maton.ai/quickbooks/v3/company/:realmId/purchase" \
  -d '{
    "Id": "123",
    "SyncToken": "0",
    "Line": [{
      "DetailType": "AccountBasedExpenseLineDetail",
      "AccountBasedExpenseLineDetail": {
        "AccountRef": { "value": "65" }
      },
      "Amount": 25.50
    }],
    "AccountRef": { "value": "9" }
  }'
```

**Important:** Include `SyncToken` from the original transaction. QBO uses optimistic locking.

## Categorization Rules

Set up vendor-to-category mappings for your business. Example patterns:

| Vendor Pattern | Category | Notes |
|----------------|----------|-------|
| Food delivery / restaurants | Meals | DoorDash, UberEats, Chipotle |
| SaaS subscriptions | Software & apps | Monthly recurring charges |
| Payment processor fees | Bank fees | Stripe, QB Payments |
| Payroll provider | Payroll fees | |
| Gas stations | Vehicle fuel | |
| Utilities | Utilities | Electric, water, gas, internet |
| Personal expenses on business cards | Owner distributions | S-Corp: shareholder distributions |

## Weekly Review Workflow

1. **Sync bank feeds:** Remind user to categorize "For Review" items in QBO UI (API cannot access pending bank feed transactions)
2. **Query recent transactions:**
   ```sql
   SELECT * FROM Purchase WHERE TxnDate >= '<7 days ago>' ORDERBY TxnDate DESC
   ```
3. **Check for uncategorized:** Look for transactions with no `AccountRef` or mapped to "Uncategorized Expense"
4. **Validate categories:** Cross-reference vendor names against categorization rules
5. **Report findings:** Flag miscategorized items, suggest corrections
6. **Fix if authorized:** Update transactions via API with correct categories

## Known Limitations

- **Bank feed "For Review" items** are not accessible via API — user must categorize in QBO UI first
- **Matched transactions** cannot be deleted via API — must unmatch in QBO UI first
- **Attachments/receipts** require separate API calls
- **Rate limits:** Maton may throttle — space out bulk operations
- **SyncToken required** for all updates — always fetch current transaction first

## Account Types

| Type | Examples |
|------|----------|
| Expense | Meals, Software, Utilities, Advertising |
| Equity | Owner Distributions, Contributions |
| Liability | Payroll tax, 401(k) contributions |
| Bank | Checking, savings accounts |
| Credit Card | Business credit cards |

## Tips

- Always fetch a transaction before updating to get the current `SyncToken`
- URL-encode SQL queries in GET requests
- Use `MAXRESULTS` to limit large result sets: `SELECT * FROM Purchase MAXRESULTS 50`
- For date ranges, QBO uses `'YYYY-MM-DD'` format in queries
- Credit card payments between accounts should be recoded to the CC liability account, not expensed
