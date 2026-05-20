# Resource Extension — Conceptual Overview

## Purpose

The `resource` extension is the **credit product template** — a lender's publicly-available product definition with eligibility ranges, rate bands, and regulatory declarations. It attaches to `Resource.resourceAttributes` and is returned in `on_discover` and `on_search` responses.

A `CreditProduct` (resource) is intentionally generic. It describes what the lender offers to the market. Per-applicant decisions (exact rate, approved amount) are in `CreditApproval` (offer).

---

## Product Template vs Applicant Offer

| Dimension | `CreditProduct` (resource) | `CreditApproval` (offer) |
|---|---|---|
| Who sees it | All BAP users browsing the catalog | Only the specific applicant |
| Rate | Band: `7.5% – 14%` | Exact: `9.5%` |
| Principal | Range: `IDR 50jt – 2M` | Exact: `IDR 450,000,000` |
| Validity | Permanent while product is active | Time-limited (14–30 days) |
| Beckn object | `Resource` | `Offer` |
| API call | `on_discover`, `on_search` | `on_select` |

---

## Product Type Classification

The `productType` field classifies the credit facility by its regulatory category and use of proceeds:

### Retail Consumer Credit

| `productType` | Indonesian Name | OJK Classification |
|---|---|---|
| `KPR` | Kredit Pemilikan Rumah | Kredit Properti |
| `KPA` | Kredit Pemilikan Apartemen | Kredit Properti |
| `KKB` | Kredit Kendaraan Bermotor | Kredit Konsumtif |
| `KTA` | Kredit Tanpa Agunan | Kredit Konsumtif |
| `CREDIT_CARD` | Kartu Kredit | Kredit Konsumtif (revolving) |
| `PAYLATER` | Pinjaman BNPL | LPBBTI / Fintech |

### Business Credit

| `productType` | Indonesian Name | Use Case |
|---|---|---|
| `KUR_MIKRO` | KUR Mikro | Subsidi: max IDR 100jt |
| `KUR_KECIL` | KUR Kecil | Subsidi: max IDR 500jt |
| `WORKING_CAPITAL` | Kredit Modal Kerja | Business working capital |
| `INVESTMENT_LOAN` | Kredit Investasi | Capital expenditure |
| `P2P_PRODUCTIVE` | Pendanaan P2P Produktif | Fintech lending (productive) |

---

## Cost of Fund Mechanisms

`costOfFund` (`CostOfFundMechanism`) defines how the interest / margin is calculated:

| `yieldType` | Used In | Description |
|---|---|---|
| `FIXED` | KTA, KKB | Rate locked for full tenor |
| `COMBINED` | KPR | Fixed 1–5 years → floating |
| `FLOATING` | Business loans | BI Rate + spread, resets periodically |
| `MARGIN` | Islamic KPR (Murabahah) | Fixed profit margin, not interest |
| `NISBAH` | Islamic savings/investment | Profit-sharing ratio |

`calculationMethod` controls the amortisation:
- `ANNUITY` — constant installment (banks, KPR, KKB)
- `FLAT` — constant principal + constant interest on original balance (multifinance, older KTA)
- `EFFECTIVE` — interest on declining balance each period (revolving, overdraft)

---

## Borrower Criteria

`borrowerCriteria` defines who can apply:

- `applicantTypes` — employment status accepted (EMPLOYEE, SELF_EMPLOYED, PROFESSIONAL, BUSINESS_OWNER, RETIREE)
- `minimumMonthlyIncomeIDR` — minimum gross income threshold
- `minimumSLIKGrade` — minimum OJK SLIK collectibility (`"1"` or `"2"`)
- `serviceRegions` — BPS province codes where product is available (`ID-JK`, `ID-JB`, `ID-JT`, etc.)
- `maxDSRPercent` — maximum debt service ratio (DTI) applied at credit assessment

---

## Islamic Finance Structure

When `productIdentity.financeType = ISLAMIC`, the `islamicFinance` block is required. It carries:

- `akadType` — the Sharia contract type (MURABAHAH, MUSYARAKAH, MUDHARABAH, IJARAH, ISTISHNA, WAKALAH_BIL_UJRAH)
- `dsnMuiCertificate` — DSN-MUI fatwa number certifying the product structure
- Profit-sharing parameters for MUSYARAKAH / MUDHARABAH products

All Islamic products must have a valid DSN-MUI fatwa. The akad type governs the legal structure and must be disclosed in the RIPLAY equivalent (Lembar Informasi Produk Syariah).

---

## Government Programs

When the product is a government-supported program, `governmentProgram` is present:

| `programType` | Program | Key Constraint |
|---|---|---|
| `KUR` | Kredit Usaha Rakyat | Rate capped at 6% p.a.; IJP subsidy via SIKP |
| `FLPP` | Fasilitas Likuiditas Pembiayaan Perumahan | Rate capped at 5% fixed; MBR only |
| `UMI` | Ultra Mikro | Max IDR 20jt; via PIP/BRI/Pegadaian |
| `BPUM` | Bantuan Produktif Usaha Mikro | Hibah (grant); not a loan |

For KUR products, `governmentProgram.subventionPercent` shows how much of the rate is subsidised by APBN. The lender's funding cost is reimbursed by the government up to this percentage.

---

## Regulatory Declaration

Every `CreditProduct` must carry a `regulatoryDeclaration` with:

- `ojkApprovedDate` — date OJK approved this product for sale
- `productRegistrationNumber` — OJK product registration code
- `riskCategory` — risk classification for the product
- `disclosureLanguage` — language of RIPLAY (must include Bahasa Indonesia)

The `requiredDocuments` array lists all documents a borrower must submit. Each document has a `documentCode` (e.g., `KTP`, `NPWP`, `SLIP_GAJI_3_BULAN`) and a `mandatory` flag.

---

## Regulatory References

- **POJK 6/POJK.07/2022** — RIPLAY, product registration, APR disclosure
- **POJK 40/POJK.03/2019** — Credit quality and asset classification
- **POJK 10/2022** — P2P lending product rules
- **POJK 35/2018** — Finance company (multifinance) product rules
- **Permenko 1/2023** — KUR program implementation
- **DSN-MUI Fatwa** — Islamic finance product certification
