# EdWise School Finance Verification — Demo Guide

**Application:** EdWise Vendor Certification Portal  
**Module:** School Finance Verification  
**Standard:** Ed-Fi ODS 2026 · Indiana DOE  
**Version:** v1.0.0

---

## What Is This Application?

The EdWise School Finance Verification Portal is a web-based tool that allows Indiana DOE vendors to **automatically validate their school finance data** before official submission. The app connects to the Ed-Fi ODS API in real time, pulls vendor-submitted financial records, and runs comprehensive automated checks across 10 validation sections — all in one click.

---

## Demo Scenario

> **Vendor:** Concord Community Schools  
> **Account ID:** `S-1394-25110-940-5170-51`  
> **Education Org ID:** `1094950000`  
> **Fiscal Year:** `2025`  
> **Approved Budget:** `$150,000`

This vendor has submitted data for all five finance entities. We will run the full certification validation and walk through each result.

---

## Step-by-Step Walkthrough

---

### STEP 1 — Enter Account Lookup Parameters

**What to do:**  
On the top of the page, fill in the account details.

| Field | Example Value |
|-------|--------------|
| Account ID | `S-1394-25110-940-5170-51` |
| EdOrg ID | `1094950000` |
| Fiscal Year | `2025` |
| Approved Budget (optional) | `150000` |

**What happens:**  
All five finance entity tables (below) automatically sync the Account ID. The API endpoint URLs also update in the background.

> 💡 **Tip:** You can add multiple records by clicking **"+ Add New Record"** — useful when validating multiple accounts in a single run.

---

### STEP 2 — Review Vendor Sample Data

**What to do:**  
Review the editable data tables for all five finance entities:

| Tab | Entity | Key Fields |
|-----|--------|-----------|
| 📋 LocalAccount | Account dimension codes | FunctionCode=25110, FundCode=1394, ObjectCode=940 |
| 📊 LocalActual | Actual transaction amount | Amount=10,125, AsOfDate=2024-10-06 |
| 🖥️ LocalCapitalizedEquipment | Equipment purchase | PaymentAmount=99,645, PerUnitCost=11,603, CapThreshold=5,000 |
| 🤝 LocalSubaward | Subaward expenditure | ExpenditureAmount=24,937, First50k=16,528, Excess50k=8,409 |
| 🏖️ LocalUnusedLeavePayment | Leave payout | Direct=9,213, Indirect=8,162, PaymentDate=2024-09-03 |

**What happens:**  
These tables show exactly what the vendor has submitted. You can also manually edit values to test specific scenarios.

---

### STEP 3 — Click "Run Certification Validation"

**What to do:**  
Click the blue **"▶ Run Certification Validation"** button.

**What happens behind the scenes:**
1. The app authenticates to the Ed-Fi ODS API (OAuth2 Bearer Token)
2. Fetches live data for all 5 resources for each account
3. Runs all 10 validation sections
4. Renders results on the page

> ⏱️ Typical run time: 15–30 seconds for one account (depends on API response speed)

---

## Results Walkthrough

---

### Result 1 — Vendor-Submitted Data (API Response)

**What it shows:**  
Raw records returned from the ODS API across all 5 finance tables.

**Example — LocalActual record returned:**

| AccountIdentifier | FiscalYear | AsOfDate | Amount | FinancialCollectionDescriptor |
|------------------|-----------|---------|--------|-------------------------------|
| S-1394-25110-940-5170-51 | 2025 | 2024-10-06 | 10125 | 1 |

**Status Indicators:**
- No color = record found successfully ✅
- 🔴 Red row = `NOT FOUND` — vendor did not post this record
- 🟡 Yellow row = `EMPTY RESPONSE` — API returned 200 but 0 records (Account ID may be wrong)

---

### Result 2 — Field-Level Validation

**What it checks:**  
Every individual field is validated for format, required value, and API code lookup.

**Example — LocalAccount validation:**

| Field | Value | Status | Reason |
|-------|-------|--------|--------|
| AccountIdentifier | S-1394-25110-940-5170-51 | ✅ Valid | Matches query param, format valid |
| FiscalYear | 2025 | ✅ Valid | Within range 2000–2100, matches query param |
| FunctionCode | 25110 | ✅ Valid | Code found in FunctionDimensions API (schoolYear=2025) |
| FundCode | 1394 | ✅ Valid | Code found in FundDimensions API |
| ObjectCode | 940 | ✅ Valid | Code found in ObjectDimensions API |
| ChartOfAccountIdentifier | IDOE-COA | ✅ Valid | Found in ChartOfAccounts API |

**Summary Cards:** Each of the 5 entities shows Total Fields / Valid / Invalid counts at a glance.

---

### Result 3 — Business Rule Validation

**What it checks:**  
Core calculation rules, date sequences, and reasonability checks.

**Example — CapitalizedEquipment rules:**

| Rule | Values | Status |
|------|--------|--------|
| PerUnitCost ≤ PaymentAmount | PerUnitCost=11,603 ≤ PaymentAmount=99,645 | ✅ Pass |
| PaymentAmount ≥ CapitalizedThreshold | 99,645 ≥ 5,000 | ✅ Pass |
| AcquisitionDate Within FiscalYear | AcquisitionDate=2024-05-28, FY2025 window: 2024-07-01→2025-06-30 | ❌ Fail |
| AcquisitionDate ≤ AsOfDate | 2024-05-28 ≤ 2024-10-06 | ✅ Pass |
| Implied Quantity (Pay/Unit) | 99,645 / 11,603 = 8.59 ≥ 1 | ✅ Pass |

> 🔴 **Fail Example:** AcquisitionDate `2024-05-28` is BEFORE the FY2025 window start `2024-07-01`. This means the asset was acquired in FY2024 but reported in FY2025 — a fiscal period mismatch.

**Example — Subaward rules:**

| Rule | Values | Status |
|------|--------|--------|
| First50k + Excess50k = ExpenditureAmount | 16,528 + 8,409 = 24,937 ✓ | ✅ Pass |
| First50k ≤ 50,000 | 16,528 ≤ 50,000 ✓ | ✅ Pass |
| ExpenditureAmount ≤ 50k logic | Exp=24,937 ≤ 50k → First50k should = Exp | ❌ Fail |
| SubawardAmount ≤ ExpenditureAmount | 12,111 ≤ 24,937 ✓ | ✅ Pass |

---

### Result 4 — Cross-Table Financial Consistency

**What it checks:**  
Total spending across all categories must not exceed the LocalActual Amount.

**Example calculation:**

| Category | Amount |
|---------|--------|
| LocalActual (Available) | $10,125 |
| Equipment PaymentAmount | $99,645 |
| Subaward ExpenditureAmount | $24,937 |
| Leave (Direct + Indirect) | $17,375 |
| **Total Spending** | **$141,957** |
| **Balance** | **−$131,832 ❌** |

> 🔴 **Result:** Total spending ($141,957) EXCEEDS Actual Amount ($10,125) by $131,832. Balance is NEGATIVE — cross-table spending violation detected.

---

### Result 5 — Budget & Allocation Validations

**What it checks:**  
- Actual Amount must not exceed the approved budget
- Running balance after each allocation category must stay non-negative

**Example:**

| Rule | Values | Status |
|------|--------|--------|
| Actual Amount ≤ Approved Budget | $10,125 ≤ $150,000 | ✅ Pass |
| Balance After Equipment Allocation | $10,125 − $99,645 = −$89,520 | ❌ Fail |

> 🔴 After allocating equipment ($99,645), the running balance goes negative — allocation exceeds available funds.

---

### Result 6 — Duplicate Detection

**What it checks:**  
- Same transaction must not appear more than once in the same table
- Same amount on the same date in multiple tables is flagged as potential double-counting

**Example — No Duplicates Found:**

| Rule | Status | Reason |
|------|--------|--------|
| No Duplicate Transactions | ✅ Pass | No duplicate AccountIdentifier+FiscalYear+AsOfDate+Amount found across all tables |

**Example — Duplicate Found:**

| Rule | Status | Reason |
|------|--------|--------|
| No Duplicate Transactions Within Table | ❌ Fail | DUPLICATE DETECTED in LocalActual — same key appears 2 times (Records 1, 2) |

---

### Result 7 — Fund & Classification Rules

**What it checks:**  
- Capital fund codes (4xxx range) must not be used for leave payments
- Payroll object codes (100–290) must not appear on equipment accounts
- AccountIdentifier must have multi-segment structure

**Example:**

| Rule | Values | Status |
|------|--------|--------|
| Capital Fund Not Used for Leave Payments | FundCode=1394 (non-capital) | ✅ Pass |
| ObjectCode Alignment | ObjectCode=940, No payroll range conflict | ✅ Pass |
| AccountIdentifier Structure | S-1394-25110-940-5170-51 (6 segments) | ✅ Pass |

---

### Result 8 — Multi-Year & Contract Validations

**What it checks:**  
Financial amounts must align with ContractNumberOfYears. Large single-year amounts are flagged.

**Example:**

| Rule | Values | Status |
|------|--------|--------|
| ExpenditureAmount Aligned with ContractYears | $24,937 ÷ 7 years = $3,562/year avg | ✅ Pass |
| No Excessive Single-Year Concentration | $24,937 for 7-year contract — within normal range | ✅ Pass |

---

### Result 9 — Transaction Lifecycle

**What it checks:**  
Enforces the required progression: `LocalAccount → LocalActual → Payment Transactions`

**Example — All layers present:**

| Rule | Layer | Status |
|------|-------|--------|
| LocalAccount Exists (Foundation Layer) | LocalAccount | ✅ Pass |
| LocalActual Exists Before Payment Transactions | LocalActual | ✅ Pass |
| Equipment Payment Has Expenditure Context | LocalCapitalizedEquipment | ✅ Pass |
| Subaward Payment Has Expenditure Context | LocalSubaward | ✅ Pass |
| Leave Payment Has Expenditure Context | LocalUnusedLeavePayment | ✅ Pass |

**Example — Missing LocalActual:**

| Rule | Layer | Status |
|------|-------|--------|
| LocalActual Exists Before Payment Transactions | LocalActual | ❌ Fail |
| | | Payment transactions found for this account but NO LocalActual record exists — lifecycle violation |

---

### Result 10 — Reporting Consistency

**What it checks:**  
FinancialCollectionDescriptor must be the same value across ALL related records for the same account.

**Example — Consistent:**

| Tables Checked | Values | Status |
|----------------|--------|--------|
| LocalActual, LocalCapitalizedEquipment, LocalSubaward, LocalUnusedLeavePayment | All = "1" | ✅ Pass |

**Example — Inconsistent:**

| Tables Checked | Values | Status |
|----------------|--------|--------|
| LocalActual=1, LocalCapitalizedEquipment=2 | Mismatch | ❌ Fail |

---

## Downloading the Report

After validation, click **"📥 Export Full Certification Report"** to download an Excel workbook.

The Excel file contains one sheet per validation section — ready to share with your manager or submit as certification evidence.

**Filename format:** `EdWise_Finance_CertReport_YYYYMMDD_HHMM.xlsx`

---

## Quick Reference — Status Colors

| Color | Meaning |
|-------|---------|
| 🟢 Green row | ✅ Pass / Valid |
| 🔴 Red row | ❌ Fail / Invalid |
| 🟡 Yellow row | ⚠️ Flag — review needed |
| ⚪ Grey row | ⏭ Skipped — data not available |

---

## Common Issues & What They Mean

| Issue | Cause | Resolution |
|-------|-------|-----------|
| 🟡 EMPTY RESPONSE | Account ID not found in ODS | Verify the Account Identifier is correct and data has been posted |
| 🔴 NOT FOUND | HTTP error from API | Check network, API availability, or Account ID format |
| AcquisitionDate outside FiscalYear | Asset acquired in prior fiscal year | Verify the correct FiscalYear is being used for this record |
| First50k + Excess50k ≠ ExpenditureAmount | Subaward math error | Recalculate and resubmit the subaward record |
| Total Spending > Actual Amount | More spending recorded than Actual Amount allows | Review all spending categories against the Actual Amount |
| Lifecycle Fail — No LocalActual | Payment posted without corresponding Actual record | Post the LocalActual record first |

---

*EdWise Group · Vendor Certification Portal · v1.0.0 · Ed-Fi ODS 2026 · Indiana DOE*
