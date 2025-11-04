# daily-disbursement-collection-summary-sql
SQL query to consolidate and analyze daily disbursement and collection transactions for financial reporting, reconciliation, and cash flow monitoring.
# 💰 Disbursement and Collection Daily Summary — SQL Query

## 📌 Objective

This SQL query consolidates **daily financial transactions** by combining **disbursement** and **collection** data into a single, analytics-ready dataset.  

It retrieves:
- **Disbursements made today** (funds sent out to recipients)
- **Collections received yesterday** (funds received via wallet or bank transfer)

The goal is to provide a **daily financial activity overview**, supporting transaction reconciliation, performance reporting, and operational monitoring.

---

## 🧠 Query Overview

The query performs two major data pulls using a `UNION ALL`:

1. **Disbursement Section**
   - Fetches all disbursements created **today** with a status of `'Success'`.
   - Captures details such as the amount disbursed, recipient wallet, payment gateway, and currency.
   - Labels all entries as `'Disbursement'`.

2. **Collection Section**
   - Fetches all authorized collections created **yesterday**.
   - Filters by valid payment methods: **wallet** or **bank transfer**.
   - Labels all entries as `'Collection'`.

Finally, both sets are **combined and sorted by creation date** (most recent first).

---

## 🧾 Output Fields

| Column | Description |
|---------|--------------|
| `transaction_type` | Indicates whether the record is a *Disbursement* or *Collection* |
| `name` | Unique transaction ID |
| `datasource` | Source system or integration origin |
| `creation` | Timestamp of the transaction |
| `amount` | Transaction amount (in native currency) |
| `disbursement` | Amount disbursed (null for collections) |
| `collection` | Amount collected (null for disbursements) |
| `reference_docname` | Reference document name (transfer or payment reference) |
| `status` | Transaction status (e.g., Success, Authorized) |
| `wallet` | Recipient wallet or account number |
| `payment_reference` | Payment reference code |
| `bank_name` | Payment gateway or bank name |
| `currency` | Currency of the transaction |
| `usd_amount` | Transaction amount converted to USD |

---

## 📊 Reporting Impact

This query enables **daily transaction tracking and financial reconciliation** across multiple payment sources.  

### 🔍 Key Use Cases:
- Monitor **daily cash flow** (inflows vs. outflows)
- Build **disbursement and collection trend dashboards**
- Support **financial reconciliation** and audit reporting
- Track **payment gateway performance** or success rates

### 💡 Business / Analytical Value:
- Improves **visibility into operational cash flow**
- Provides a **reliable dataset for automated daily reports**
- Supports **KPI generation** like:
  - Total Disbursed Today  
  - Total Collected Yesterday  
  - Net Daily Movement (`collection - disbursement`)

---

## 🧩 Full SQL Query

```sql
SELECT
  'Disbursement' AS transaction_type,
  name,
  datasource,
  creation,
  total_disbursed AS amount,
  total_disbursed AS disbursement,
  NULL AS collection,
  transfer_reference AS reference_docname,
  disbursement_status AS status,
  recipient_account_number AS wallet,
  NULL AS payment_reference,
  payment_gateway AS bank_name,
  currency,
  usd_amount
FROM `tabdisbursement` AS t0
WHERE disbursement_status = 'Success'
  AND DATE(creation) = CURRENT_DATE()  -- Disbursement today

UNION ALL

SELECT
  'Collection' AS transaction_type,
  name,
  datasource,
  creation,
  amount AS amount,
  NULL AS disbursement,
  amount AS collection,
  payment_reference AS reference_docname,
  'Authorized' AS status,
  NULL AS wallet,
  payment_reference,
  payment_gateway,
  currency,
  usd_amount
FROM `tabintegration_requests` AS t0
WHERE authorized = 1
  AND DATE(creation) = DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY)  -- Collection yesterday
  AND (
    (payment_gateway LIKE '%wallet%' AND payment_method = 'CHECKOUT PAGE')
    OR payment_method = 'BANK TRANSFER'
  )
ORDER BY creation DESC;


Do the same for this query too
SELECT
  'Disbursement' AS transaction_type,
  name,
  datasource,
  creation,
  total_disbursed AS amount,
  total_disbursed AS disbursement,
  NULL AS collection,
  transfer_reference AS reference_docname,
  disbursement_status AS status,
  recipient_account_number AS wallet,
  NULL AS payment_reference,
  payment_gateway AS bank_name,
  currency,
  usd_amount
FROM `tabdisbursement` AS t0
WHERE disbursement_status = 'Success'
  AND DATE(creation) = CURRENT_DATE()  -- Disbursement today

UNION ALL

SELECT
  'Collection' AS transaction_type,
  name,
  datasource,
  creation,
  amount AS amount,
  NULL AS disbursement,
  amount AS collection,
  payment_reference AS reference_docname,
  'Authorized' AS status,
  NULL AS wallet,
  payment_reference,
  payment_gateway,
  currency,
  usd_amount
FROM `tabintegration_requests` AS t0
WHERE authorized = 1
  AND DATE(creation) = DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY)  -- Collection yesterday
  AND (
    (payment_gateway LIKE '%wallet%' AND payment_method = 'CHECKOUT PAGE')
    OR payment_method = 'BANK TRANSFER'
  )
ORDER BY creation DESC;
