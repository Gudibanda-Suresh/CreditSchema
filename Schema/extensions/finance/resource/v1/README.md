# CreditResourceAttributes — v1 Property Table

**Pack ID:** `ion-finance-credit-resource-v1`  
**Attaches to:** `Resource.resourceAttributes`  
**ION IRI:** `https://schema.ion.id/finance/credit-resource/v1#CreditResourceAttributes`  
**Beckn flow:** `on_discover` → `message.catalog.resources[].resourceAttributes`

---

## Required Properties

| Property | Type | Description |
|----------|------|-------------|
| `productIdentity` | `ProductIdentity` | Product name, type, institution, channels |
| `principalRange` | `PrincipalRange` | Loan amount range (min/max) |
| `tenor` | `Tenor` | Tenor range (min/max/available) |
| `costOfFund` | `CostOfFundMechanism` | Rate/margin/profit-share, method, benchmark |
| `repaymentSchedule` | `RepaymentSchedule` | Frequency, payment channels, grace period |
| `borrowerCriteria` | `BorrowerCriteria` | Age, income, employment, region, DSR |
| `regulatoryDeclaration` | `RegulatoryDeclaration` | OJK licences, POJK 6/2022 disclosures |
| `requiredDocuments` | `RequiredDocument[]` | Application document checklist |

## Optional Properties

| Property | Type | Condition |
|----------|------|-----------|
| `feeStructure` | `FeeStructure` | All products |
| `collateral` | `Collateral` | Secured products |
| `islamicFinance` | `IslamicFinanceStructure` | `financeType = ISLAMIC` |
| `governmentProgram` | `GovernmentProgram` | KUR, FLPP, UMi, BPUM |
| `paylaterService` | `PaylaterService` | `productType = PAYLATER` |

---

## Block 1 — ProductIdentity

| Field | Type | Required | Values |
|-------|------|----------|--------|
| `productName` | string | Yes | Official product name (matches RIPLAY header) |
| `institutionName` | string | Yes | Full legal entity name as per OJK registration |
| `productType` | enum | Yes | KPR, KPA, KKB, KTA, KUR_MIKRO, FLPP, PAYLATER, ... |
| `financeType` | enum | Yes | `CONVENTIONAL` \| `ISLAMIC` |
| `currency` | enum | Yes | `IDR` (default) \| USD \| SGD \| EUR |
| `institutionType` | enum | Yes | BANK_UMUM, BPR, BPRS, P2P_LENDING, ... |
| `bankCapitalTier` | enum | No | BUKU_1 – BUKU_4 (BANK_UMUM only) |
| `digitalOnly` | boolean | No | `false` (default) |
| `distributionChannels` | enum[] | No | BRANCH_OFFICE, MOBILE_BANKING, DIRECT_API, ... |

## Block 2 — PrincipalRange

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `minimum` | MonetaryAmount | Yes | `{value, currency}` |
| `maximum` | MonetaryAmount | Yes | `{value, currency}` |
| `maximumKUROfficial` | number | No | KUR plafond cap (IDR) |
| `denomination` | number | No | Amount increment (e.g. 500,000) |

## Block 3 — Tenor

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `unit` | enum | Yes | DAYS \| MONTHS \| YEARS |
| `minimum` | integer | Yes | Min tenure value |
| `maximum` | integer | Yes | Max tenure value |
| `availableTenors` | integer[] | No | Fixed options (e.g. [3, 6, 12]) |

## Block 4 — CostOfFundMechanism

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `yieldType` | enum | Yes | FIXED \| FLOATING \| COMBINED \| MARGIN \| NISBAH \| UJRAH \| ZERO_COST |
| `calculationMethod` | enum | No | ANNUITY \| FLAT \| EFFECTIVE \| DECLINING_BALANCE |
| `rateMinPercent` | number | No | Min rate % p.a. |
| `rateMaxPercent` | number | No | Max rate % p.a. |
| `rateReference` | enum | No | BI_RATE \| SBDK \| JIBOR (for FLOATING) |
| `spreadPercent` | number | No | Spread above benchmark |
| `fixedPeriodYears` | integer | No | Fixed period for COMBINED products |
| `governmentSubsidyPercent` | number | No | Subsidy % (KUR) |
| `effectiveRatePercent` | number | No | Effective borrower rate after subsidy |

## Block 5 — RepaymentSchedule

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `frequency` | enum | Yes | DAILY \| WEEKLY \| MONTHLY \| QUARTERLY \| BULLET \| SEASONAL |
| `paymentChannels` | enum[] | Yes | AUTODEBIT, QRIS, BIFAST, VIRTUAL_ACCOUNT, ... |
| `gracePeriodDays` | integer | No | Default 0 |
| `earlySettlementAllowed` | boolean | No | Default `true` |
| `restructuringAllowed` | boolean | No | Default `true` |

## Block 6 — FeeStructure

| Field | Type | Notes |
|-------|------|-------|
| `provisionFeePercent` | number | % of principal |
| `administrationFeeIDR` | number | Flat IDR amount |
| `appraisalFeeIDR` | number | KJPP appraisal |
| `notaryFeeIDR` | number | Estimate |
| `mandatoryInsurance` | boolean | Default `false` |
| `mandatoryInsuranceTypes` | enum[] | LIFE_INSURANCE, FIRE_INSURANCE, ... |
| `latePenaltyPercentPerDay` | number | % per day overdue |
| `stampDutyIDR` | integer | Default 10,000 (UU No. 10/2020) |
| `maxTotalCostPercent` | number | APR equivalent cap (POJK 6/2022) |

## Block 7 — Collateral

| Field | Type | Notes |
|-------|------|-------|
| `required` | boolean | **Required field** |
| `collateralTypes` | enum[] | PROPERTY, MOTOR_VEHICLE_CAR, GOLD, UNSECURED, ... |
| `maxLTVPercent` | number | BI LTV limits apply |
| `appraisalRequired` | boolean | KJPP appraisal needed |
| `guarantorInstitution` | enum | JAMKRINDO \| ASKRINDO \| JAMKRIDA |

## Block 8 — BorrowerCriteria

| Field | Type | Notes |
|-------|------|-------|
| `applicantTypes` | enum[] | **Required.** INDIVIDUAL, MICRO_ENTERPRISE, ... |
| `minimumAge` | integer | Min 17 |
| `maximumAgeAtMaturity` | integer | Max 80 |
| `minimumMonthlyIncomeIDR` | number | Gross monthly income in IDR |
| `minimumSLIKGrade` | integer | 1–5 (1 = Lancar/Current) |
| `employmentTypes` | enum[] | CIVIL_SERVANT, PERMANENT_EMPLOYEE, ... |
| `serviceRegions` | enum[] | ISO 3166-2:ID codes or NATIONAL |
| `maxDSRPercent` | number | OJK guidance cap: 70% |

## Block 9 — RegulatoryDeclaration

| Field | Type | Notes |
|-------|------|-------|
| `licenses` | License[] | **Required** (min 1). OJK/BI/DSN-MUI records |
| `disclosures` | Disclosure[] | RIPLAY, T&C, PRIVACY_POLICY, ... |
| `complaintService` | ComplaintService | POJK 6/2022 mandatory |
| `privacyPolicyUrl` | URI | UU PDP No. 27/2022 |

## Block 10 — RequiredDocument

| Field | Type | Notes |
|-------|------|-------|
| `documentType` | enum | **Required.** KTP, NPWP, SHM, BPKB, SLIK_CONSENT, ... |
| `mandatory` | boolean | **Required** |
| `issuer` | string | e.g. DUKCAPIL, DJP, BPN |
| `digitalVerification` | boolean | e-KYC/OCR support |

## Block 11 — IslamicFinanceStructure _(when financeType = ISLAMIC)_

| Field | Type | Notes |
|-------|------|-------|
| `contractType` | enum | **Required.** MURABAHAH, MUSHARAKAH_MUTANAQISAH, MUDHARABAH, ... |
| `dsnMuiFatwaNumber` | string | **Required.** DSN-MUI fatwa reference |
| `marginPercentPerYear` | number | Murabahah margin rate |
| `bankProfitSharePercent` | number | Mudharabah/Musharakah — bank share % |
| `customerProfitSharePercent` | number | Customer profit share % |

## Block 12 — GovernmentProgram _(KUR, FLPP, UMi, BPUM)_

| Field | Type | Notes |
|-------|------|-------|
| `programType` | enum | **Required.** KUR_SUPER_MIKRO, KUR_MIKRO, FLPP, UMI, ... |
| `subsidyPercent` | number | Gov. subsidy % p.a. to bank |
| `effectiveRatePercent` | number | Borrower rate after subsidy |
| `officialGuarantorInstitution` | enum | JAMKRINDO \| ASKRINDO |
| `specialRequirements` | string[] | Eligibility conditions |

## Block 13 — PaylaterService _(when productType = PAYLATER)_

| Field | Type | Notes |
|-------|------|-------|
| `maxCreditLimitIDR` | number | Per-customer credit limit |
| `billingCycle` | enum | FIXED_INSTALLMENT \| MONTHLY_BILLING |
| `availableInstallmentTenors` | integer[] | e.g. [3, 6, 12, 24] |
| `merchantCategories` | enum[] | E_COMMERCE, FASHION_RETAIL, ALL_MERCHANTS, ... |
