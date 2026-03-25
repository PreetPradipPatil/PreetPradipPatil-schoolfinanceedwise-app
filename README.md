# 💰 EdWise Group — School Finance Verification Portal

> **Vendor Certification Portal** · Ed-Fi ODS 2026 · Indiana DOE  
> A Streamlit-based web application that fetches live financial data from the Ed-Fi ODS API and runs **100% automated finance validation** across 10 validation sections.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Setup & Installation](#setup--installation)
- [Configuration](#configuration)
- [How It Works](#how-it-works)
- [Validation Sections](#validation-sections)
- [Finance Resources](#finance-resources)
- [API Endpoints](#api-endpoints)
- [Export & Reporting](#export--reporting)
- [Screenshots](#screenshots)

---

## Overview

The **EdWise School Finance Verification Portal** is a multi-module web application built for **Indiana Department of Education (DOE)** vendors to certify their school finance data submissions against the **Ed-Fi ODS 2026** API. The portal fetches live data, validates it across 10 comprehensive rule sections, and produces a downloadable Excel certification report.

The portal currently supports two modules:
- 🎓 **Student Verification** (separate module)
- 💰 **School Finance Verification** (this module)

---

## Features

- ✅ **Live API Fetch** — Pulls real-time data from Ed-Fi ODS & IDOE endpoints with OAuth2 Bearer token authentication
- ✅ **Multi-Record Support** — Validate multiple Account IDs simultaneously in a single run
- ✅ **100% Validation Coverage** — 10 validation sections covering all finance rules
- ✅ **Field-Level Validation** — Every field validated for format, value, and API lookup
- ✅ **Business Rule Engine** — Cross-field math rules, threshold checks, date sequences
- ✅ **Cross-Table Consistency** — Spending vs. Actual Amount balance checks across all 5 tables
- ✅ **Duplicate Detection** — Within-table and cross-table double-counting detection
- ✅ **Lifecycle Validation** — Enforces Account → Actual → Payment transaction sequence
- ✅ **Budget Allocation Tracking** — Running balance checks after each spending category
- ✅ **Fund Classification** — Capital fund misuse detection
- ✅ **Excel Export** — Full certification report with one sheet per validation section
- ✅ **Professional UI** — Custom-styled Streamlit interface with consistent stat cards

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Frontend / App Framework | [Streamlit](https://streamlit.io/) |
| Language | Python 3.10+ |
| Data Processing | Pandas |
| HTTP Client | Requests |
| Authentication | OAuth2 Client Credentials (Bearer Token) |
| API Standard | Ed-Fi ODS REST API v3 |
| Export | openpyxl (Excel) |
| Styling | Custom CSS + Google Fonts (Plus Jakarta Sans) |

---

## Project Structure

```
edwise-finance/
├── pages/
│   ├── 1_Student_Verification.py     # Student module (separate)
│   └── 2_School_Finance_Verification.py  # This file
├── .streamlit/
│   └── secrets.toml                  # API credentials (not committed)
├── README.md
└── requirements.txt
```

---

## Setup & Installation

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/edwise-finance.git
cd edwise-finance
```

### 2. Install Python Dependencies

```bash
pip install -r requirements.txt
```

**requirements.txt**
```
streamlit>=1.32.0
pandas>=2.0.0
requests>=2.31.0
openpyxl>=3.1.0
```

### 3. Configure Secrets

Create `.streamlit/secrets.toml`:

```toml
[ods_api_finance]
token_url   = "https://your-ods-host/oauth/token"
api_key     = "YOUR_CLIENT_ID"
api_secret  = "YOUR_CLIENT_SECRET"
```

> ⚠️ **Never commit `secrets.toml` to version control.** Add it to `.gitignore`.

### 4. Run the Application

```bash
streamlit run pages/2_School_Finance_Verification.py
```

Or from the multi-page app root:
```bash
streamlit run Home.py
```

---

## Configuration

### API Base URLs

The application connects to two base API namespaces:

| Namespace | Base URL |
|-----------|----------|
| Ed-Fi Standard | `https://<host>/2026/data/v3/ed-fi` |
| IDOE Extensions | `https://<host>/2026/data/v3/idoe` |

### Token Caching

Bearer tokens are cached in Streamlit session state and auto-refreshed when expired, based on the `expires_in` value returned by the OAuth token endpoint.

### Fund Classification Constants

These are configurable in the source file:

```python
CAPITAL_FUND_CODES     = {"4200", "4300", ... "4900"}
PAYROLL_OBJECT_CODES   = {"100", "110", ... "290"}
CAPITAL_FUNCTION_CODES = {"4000", "4100", "4200", "4300"}
```

---

## How It Works

```
Step 1  →  Enter Account ID(s), EdOrg ID, Fiscal Year (+ optional Approved Budget)
Step 2  →  Review / edit vendor sample data in editable tables (5 finance entities)
Step 3  →  Click "Run Certification Validation"
           ↓
           Fetch live data from Ed-Fi ODS for each record × each resource
           ↓
           Run all 10 validation sections
           ↓
Results 1–10 rendered on page
           ↓
Download full Excel certification report
```

---

## Validation Sections

The portal covers **10 validation sections** with 100% rule coverage:

| # | Section | What It Checks |
|---|---------|---------------|
| **1** | Field-Level Validation | Format, required fields, API code lookups for all 5 entities |
| **2** | Business Rule Validation | Core math rules, date sequences, reasonability checks |
| **3** | Cross-Table Consistency | Total spending vs. LocalActual Amount; per-category balance |
| **4** | Budget & Allocation | Actual Amount vs. Approved Budget; running balance after each category |
| **5** | Duplicate Detection | Within-table duplicates; cross-table double-counting |
| **6** | Fund & Classification | Capital fund misuse; ObjectCode alignment; AccountIdentifier structure |
| **7** | Multi-Year & Contract | ExpenditureAmount vs. ContractNumberOfYears; concentration check |
| **8** | Lifecycle & Process | Account → Actual → Payment existence chain |
| **9** | Reporting Consistency | FinancialCollectionDescriptor consistency across all related records |
| **10** | Export | Full Excel report with all sections, fails-only sheets, and summary |

### Key Business Rules

```
PerUnitCost ≤ PaymentAmount
PaymentAmount ≥ CapitalizedThreshold
First50k + Excess50k = ExpenditureAmount
First50k ≤ 50,000
If ExpenditureAmount ≤ 50,000 → First50k = ExpenditureAmount, Excess50k = 0
If ExpenditureAmount > 50,000 → First50k = 50,000, Excess50k = ExpenditureAmount − 50,000
SubawardAmount ≤ ExpenditureAmount
Total Spending (Equip + Subaward + Leave) ≤ LocalActual Amount
Actual Amount ≤ Approved Budget
AcquisitionDate ≤ AsOfDate
PaymentDate ≤ AsOfDate
All transaction dates within FiscalYear window (July 1 → June 30)
```

---

## Finance Resources

The portal validates data across **5 Ed-Fi finance resources**:

| Resource | Key Fields |
|----------|-----------|
| **LocalAccount** | AccountIdentifier, FunctionCode, FundCode, ObjectCode, OperationalUnitCode, SectionCode, SubCategoryCode |
| **LocalActual** | AccountIdentifier, AsOfDate, Amount, FinancialCollectionDescriptor |
| **LocalCapitalizedEquipment** | AsOfDate, AcquisitionDate, PaymentAmount, PerUnitCost, CapitalizedThreshold |
| **LocalSubaward** | ExpenditureAmount, First50k, Excess50k, SubawardAmount, ContractNumberOfYears |
| **LocalUnusedLeavePayment** | DirectUnusedLeavePaymentAmount, IndirectUnusedLeavePaymentAmount, PaymentDate |

---

## API Endpoints

### Data Fetch Endpoints

| Resource | Endpoint |
|----------|----------|
| LocalAccount | `GET /ed-fi/LocalAccounts?accountIdentifier={id}` |
| LocalActual | `GET /ed-fi/localActuals?accountIdentifier={id}` |
| LocalCapitalizedEquipment | `GET /idoe/LocalCapitalizedEquipment?accountIdentifier={id}` |
| LocalSubaward | `GET /idoe/LocalSubawards?accountIdentifier={id}` |
| LocalUnusedLeavePayment | `GET /idoe/LocalUnusedLeavePayments?accountIdentifier={id}` |

### Validation Lookup Endpoints

| Code Type | Endpoint |
|-----------|----------|
| FunctionCode | `GET /ed-fi/functionDimensions?schoolYear=2025&code={code}` |
| FundCode | `GET /ed-fi/fundDimensions?schoolYear=2025&code={code}` |
| ObjectCode | `GET /ed-fi/objectDimensions?schoolYear=2025&code={code}` |
| OperationalUnitCode | `GET /ed-fi/operationalUnitDimensions?schoolYear=2025&code={code}` |
| SectionCode | `GET /idoe/sectionDimensions?schoolYear=2025&code={code}` |
| SubCategoryCode | `GET /idoe/subCategoryDimensions?schoolYear=2025&code={code}` |
| ChartOfAccounts | `GET /ed-fi/chartOfAccounts?fiscalYear=2025&accountIdentifier={id}&educationOrganizationId={id}` |
| FinancialCollectionDescriptor | `GET /ed-fi/financialCollectionDescriptors?codeValue={code}` |

---

## Export & Reporting

Clicking **"Export Full Certification Report"** downloads an Excel workbook containing:

| Sheet Name | Contents |
|-----------|---------|
| `Summary` | Pass/Fail counts per resource and business rule section |
| `Target_<Resource>` | Raw API response data for each finance entity |
| `FieldVal_<Resource>` | Field-level validation results |
| `BizRules_<Resource>` | Business rule results per entity |
| `All_Invalid_Fields` | Combined list of all failed field validations |
| `Failed_BizRules` | Combined list of all failed business rules |
| `S2_CrossTable` | Cross-table spending vs. Actual Amount results |
| `S3_Budget_Allocation` | Budget and allocation balance results |
| `S4_Duplicate_Detection` | Duplicate transaction findings |
| `S6_Fund_Classification` | Fund code classification results |
| `S7_MultiYear_Contract` | Multi-year contract distribution results |
| `S9_Lifecycle_Process` | Transaction lifecycle chain results |
| `S10_Descriptor_Consistency` | FinancialCollectionDescriptor consistency results |

---

## Status Indicators

| Icon | Meaning |
|------|---------|
| ✅ Pass / Valid | Rule satisfied, field is correct |
| ❌ Fail / Invalid | Rule violated, field has an error |
| ⚠️ Flag | Anomaly detected — requires manual review |
| ⏭ Skipped | Cannot evaluate — data not available or not fetched |

---

## Version

| Property | Value |
|----------|-------|
| App Version | v1.0.0 |
| Ed-Fi ODS Version | 2026 |
| School Year | 2025 |
| Target State | Indiana DOE |

---

*Built by EdWise Group — Vendor Certification Portal*
