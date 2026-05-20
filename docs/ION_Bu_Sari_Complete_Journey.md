# Bu Sari's Complete ION Journey
## Apotek Sehat Mandiri — From Onboarding to Reconciliation

**Version:** 0.2-draft  
**Purpose:** End-to-end walkthrough of ION for a real seller — showing how every layer of the hierarchy, every attribute pack, and every protocol endpoint works in practice.

**How to read this document:** This is the technical companion to the **Bu Sari Wizard** document. The Wizard shows what Bu Sari sees and does in plain language. This document shows the exact API calls, JSON payloads, and field-level detail behind each step. Where Indonesian text appears in JSON payloads (as it would in real seller data), an English translation is given immediately below.

**Cross-reference:** Each Phase here maps to the corresponding Problems in the Wizard:

| This document | Wizard |
|---|---|
| Phase 1 — Onboarding | Problems 1, 2, 3 |
| Phase 2 — Discovery | Problem 4 |
| Phase 3 — Transactions A, B, C | Problems 5, 6, 7 |
| Phase 4 — IGM | Problem 8 |
| Phase 5 — Reconciliation | Problem 9 |

---

## The Cast

**Bu Sari** — owner of Apotek Sehat Mandiri, Jl. Kemang Raya No. 47, Jakarta Selatan. One physical store, 3 staff. Currently walk-in only. Her daughter Dinda convinced her to join ION.

**Dinda** — Bu Sari's daughter, handles the tech side.

**Pak Ahmad** — a regular customer. Diabetic, buys metformin monthly. Lives 800m away.

**The seller app** — `sellerapp.apotek-sehat.ion.id` — the BPP (Beckn Provider Platform) Dinda's team builds and operates. *In ION terms: Dinda's team is the Seller App (Role: Seller App). Bu Sari is the Seller (Role: Seller). Two distinct roles — a Seller App can serve many Sellers.*

**The buyer app** — `buyerapp.sehat-kita.ion.id` — the BAP (Beckn Application Platform) Pak Ahmad uses. *Role: Buyer App.*

---

## Items Bu Sari Sells on ION

| # | Item | CRC | Pattern | Variant |
|---|---|---|---|---|
| 1 | Panadol 500mg (4 tabs) | `TRC-health-beauty` | storefront | happy-flow |
| 2 | Amoxicillin 500mg (10 caps) | `TRC-health-beauty` | storefront | delivery-time-kyc |
| 3 | Metformin 500mg (30 tabs) | `TRC-health-beauty` | subscription | delivery-time-kyc |
| 4 | Aqua 600ml mineral water | `TRC-food-bev` | storefront | happy-flow |
| 5 | Optimum Nutrition Whey Protein 2.27kg | `TRC-food-bev` | storefront | happy-flow |

---

# PHASE 1 — ONBOARDING
*Wizard cross-reference: Problems 1, 2, 3 — "Where do I fit?", "How do I register?", "How do I list my products?"*

*ION hierarchy layers engaged: Organisation · Role · Profile · Sector · Segment · CRC · Resource*

## Step 1 — Bu Sari answers "Where do I fit?"

Dinda opens the seller app onboarding wizard. First question:

> *"What kind of business do you run?"*

Bu Sari answers: **Apotek / Pharmacy**

```
Sector:   Trade          (buyer ends up owning the medicine)
Segment:  TRD-04         (Pharmacy & Health Retail)
Role:     Seller
Profile:  seller / trade
KBLI:     47730, 47740   (already on her NIB from store registration)
```

ION validates her NIB against OSS. ✓

---

## Step 2 — Provider attributes published

Dinda fills in the store's operational details. These travel in `Provider.providerAttributes` with `@type: ion:TradeProvider`.

```json
{
  "@context": "https://schema.ion.id/trade/provider/v1/context.jsonld",
  "@type": "ion:TradeProvider",

  "invoicingModel": "CENTRAL",
  "invoicingEntity": "Usaha Dagang Sehat Mandiri",  // Trading Business Sehat Mandiri

  "storeStatus": "OPEN",

  "operatingHours": [
    {
      "type": "ORDER",
      "dayFrom": 1,
      "dayTo": 6,
      "timeFrom": "0800",
      "timeTo": "2100"
    },
    {
      "type": "ORDER",
      "dayFrom": 7,
      "dayTo": 7,
      "timeFrom": "0900",
      "timeTo": "1800"
    }
  ],

  "holidayCalendar": ["2026-08-17", "2026-12-25"],

  "providerCategory": "PHARMACY",

  "nibRegistered": true,

  "kycStatus": "APPROVED",
  "kycLevel": "STANDARD",
  "kycValidUntil": "2028-04-01",

  "categoryLicenses": [
    {
      "licenseType": "PBF_PHARMACY",
      "licenseNumber": "PBF-DKI-2024-004892",
      "validUntil": "2027-06-30",
      "issuingAuthority": "Dinas Kesehatan DKI Jakarta"
    }
  ],

  "handlingTimeMinutes": 15
}
```

**What this tells the network:**
- Bu Sari is open Mon–Sat 08:00–21:00, Sunday 09:00–18:00
- She's closed on Independence Day and Christmas
- She has a valid PBF (Pedagang Besar Farmasi) pharmacy licence
- KYC approved — she can publish catalogs and accept orders
- 15 minutes from confirm to package ready

---

## Step 3 — Catalogue published

Dinda publishes 5 items via `POST /catalog/publish`. Each resource has its `resourceAttributes` filled according to its CRC.

---

### Item 1 — Panadol 500mg (OTC Medicine)

**CRC:** `TRC-health-beauty` → triggers pharmacy attribute schema  
**Variant triggered:** none (prescriptionRequired = false)

```json
{
  "id": "RES-APOTEK-SEHAT-001",
  "descriptor": {
    "name": "Panadol 500mg 4 Tablet",
    "shortDesc": "Obat pereda nyeri dan penurun demam. Mengandung paracetamol 500mg.",
    "longDesc": "Panadol 500mg meringankan sakit kepala, sakit gigi, demam, nyeri otot, dan nyeri haid.",
    "images": ["https://cdn.apotek-sehat.id/panadol-500-4tab.jpg"]
  },
  "resourceAttributes": {
    "@context": "https://schema.ion.id/trade/resource/v1/context.jsonld",
    "@type": "ion:TradeResource",

    "quantity": { "value": 1, "unit": "piece" },
    "resourceStructure": "PLAIN",
    "resourceTangibility": "PHYSICAL",
    "images": ["https://cdn.apotek-sehat.id/panadol-500-4tab.jpg"],
    "availability": { "status": "IN_STOCK", "maxOrderQty": 5 },
    "ageRestricted": false,
    "countryOfOrigin": "IDN",
    "logisticsServiceType": "HYPERLOCAL",

    "identity": {
      "brand": "Panadol",
      "sku": "PAN-500-4TAB"
    },

    "packaged": {
      "netQuantity": { "value": 4, "unit": "PIECE" },
      "manufacturerOrPacker": {
        "type": "MANUFACTURER",
        "name": "PT Haleon Indonesia",
        "address": "Jl. Raya Bogor Km 28, Ciracas, East Jakarta 13740"
      },
      "expiryDate": "2027-09-30",
      "commonOrGenericName": "Parasetamol"
    },

    "regulatory": {
      "registrations": [
        {
          "scheme": "BPOM_MD",
          "id": "NA18190100521",
          "role": "BRAND_OWNER",
          "validUntil": "2028-01-01"
        }
      ]
    },

    "pharmacy": {
      "prescriptionRequired": false,
      "dosageForm": "TABLET",
      "strength": "500 mg",
      "activeIngredients": ["Parasetamol"],
      "coldChainRequired": false
    }
  }
}
```

**CDS validation result:** ✓ ACCEPTED  
- BPOM_MD registration present ✓  
- pharmacy sub-object complete ✓  
- No prescription flag → no delivery-time-kyc required  
- Indexed under `TRC-health-beauty` in the CDS

---

### Item 2 — Amoxicillin 500mg (Prescription / Scheduled Drug)

**CRC:** `TRC-health-beauty` → triggers pharmacy schema  
**Variant triggered:** `delivery-time-kyc` — MANDATORY because `prescriptionRequired = true`

> **The key moment:** Bu Sari marks `prescriptionRequired: true`. She doesn't configure a variant. The network derives delivery-time-kyc as mandatory automatically.

```json
{
  "id": "RES-APOTEK-SEHAT-002",
  "descriptor": {
    "name": "Amoxicillin 500mg 10 Kapsul",
    "shortDesc": "Antibiotik. Hanya dengan resep dokter.",
    "images": ["https://cdn.apotek-sehat.id/amoxicillin-500-10caps.jpg"]
  },
  "resourceAttributes": {
    "@context": "https://schema.ion.id/trade/resource/v1/context.jsonld",
    "@type": "ion:TradeResource",

    "quantity": { "value": 1, "unit": "piece" },
    "resourceStructure": "PLAIN",
    "resourceTangibility": "PHYSICAL",
    "images": ["https://cdn.apotek-sehat.id/amoxicillin-500-10caps.jpg"],
    "availability": { "status": "IN_STOCK" },
    "ageRestricted": false,
    "countryOfOrigin": "IDN",
    "logisticsServiceType": "DOMESTIC_SURFACE",
    "dataResidency": "DOMESTIC_ONLY",

    "identity": {
      "brand": "Sanbe Farma",
      "sku": "AMX-500-10CAP"
    },

    "packaged": {
      "netQuantity": { "value": 10, "unit": "PIECE" },
      "manufacturerOrPacker": {
        "type": "MANUFACTURER",
        "name": "PT Sanbe Farma",
        "address": "Jl. Industri Cimareme No. 8, Padalarang, West Bandung"
      },
      "expiryDate": "2026-12-31",
      "commonOrGenericName": "Amoksisilin"
    },

    "regulatory": {
      "registrations": [
        {
          "scheme": "BPOM_MD",
          "id": "GKL0309337710A1",
          "role": "MANUFACTURER",
          "validUntil": "2027-06-01"
        }
      ]
    },

    "pharmacy": {
      "prescriptionRequired": true,
      "dosageForm": "CAPSULE",
      "strength": "500 mg",
      "activeIngredients": ["Amoksisilin trihidrat"],
      "coldChainRequired": false
    }
  }
}
```

**CDS validation result:** ✓ ACCEPTED  
- BPOM_MD present ✓  
- `prescriptionRequired: true` → CDS flags: delivery-time-kyc variant MANDATORY on all transactions for this resource  
- `dataResidency: DOMESTIC_ONLY` → BAPs with foreign processing infrastructure cannot surface this item

---

### Item 3 — Metformin 500mg (Subscription / Chronic Medication)

**CRC:** `TRC-health-beauty` → pharmacy schema  
**Pattern:** subscription  
**Variant triggered:** `delivery-time-kyc` — because `prescriptionRequired = true`

> **What's different here:** This is the subscription pattern. Pak Ahmad will confirm once. Every month, the next cycle auto-initiates. He needs a valid prescription on file. Each delivery still requires KYC at the door.

```json
{
  "id": "RES-APOTEK-SEHAT-003",
  "descriptor": {
    "name": "Metformin 500mg 30 Tablet",
    "shortDesc": "Obat diabetes tipe 2. Hanya dengan resep dokter. Tersedia dalam paket langganan bulanan.",
    // Type 2 diabetes medicine. Prescription only. Monthly subscription available.
    "images": ["https://cdn.apotek-sehat.id/metformin-500-30tab.jpg"]
  },
  "resourceAttributes": {
    "@context": "https://schema.ion.id/trade/resource/v1/context.jsonld",
    "@type": "ion:TradeResource",

    "quantity": { "value": 1, "unit": "piece" },
    "resourceStructure": "PLAIN",
    "resourceTangibility": "PHYSICAL",
    "images": ["https://cdn.apotek-sehat.id/metformin-500-30tab.jpg"],
    "availability": {
      "status": "IN_STOCK",
      "maxOrderQty": 3
    },
    "ageRestricted": false,
    "countryOfOrigin": "IDN",
    "logisticsServiceType": "HYPERLOCAL",
    "dataResidency": "DOMESTIC_ONLY",

    "identity": {
      "brand": "Kimia Farma",
      "sku": "MET-500-30TAB"
    },

    "packaged": {
      "netQuantity": { "value": 30, "unit": "PIECE" },
      "manufacturerOrPacker": {
        "type": "MANUFACTURER",
        "name": "PT Kimia Farma Tbk",
        "address": "Jl. Veterans No. 9, Bandung 40394"
      },
      "expiryDate": "2027-03-31",
      "commonOrGenericName": "Metformin Hidroklorida"
    },

    "regulatory": {
      "registrations": [
        {
          "scheme": "BPOM_MD",
          "id": "GKL9614008010A1",
          "role": "MANUFACTURER",
          "validUntil": "2028-01-01"
        }
      ]
    },

    "pharmacy": {
      "prescriptionRequired": true,
      "dosageForm": "TABLET",
      "strength": "500 mg",
      "activeIngredients": ["Metformin hidroklorida"],
      "coldChainRequired": false
    }
  }
}
```

**CDS validation result:** ✓ ACCEPTED  
**Subscription offer attached separately** — offer declares `paymentTermsPolicy: ion://policy/payment-terms.subscription.monthly_autopay`

---

### Item 4 — Aqua 600ml Mineral Water

**CRC:** `TRC-food-bev` → DIFFERENT schema — food/beverage sub-object, NOT pharmacy  
**Variant triggered:** none

> **The key shift:** This is NOT a medicine. Different CRC entirely. BPOM registration is still required — but it's BPOM_MD for food (MD = Makanan Dalam Negeri), not pharmacy. The pharmacy sub-object does NOT appear. The food sub-object appears instead.

```json
{
  "id": "RES-APOTEK-SEHAT-004",
  "descriptor": {
    "name": "AQUA Air Mineral 600ml",
    "shortDesc": "Air mineral alami dari sumber pegunungan.",
    "images": ["https://cdn.apotek-sehat.id/aqua-600ml.jpg"]
  },
  "resourceAttributes": {
    "@context": "https://schema.ion.id/trade/resource/v1/context.jsonld",
    "@type": "ion:TradeResource",

    "quantity": { "value": 1, "unit": "piece" },
    "resourceStructure": "PLAIN",
    "resourceTangibility": "PHYSICAL",
    "images": ["https://cdn.apotek-sehat.id/aqua-600ml.jpg"],
    "availability": {
      "status": "IN_STOCK",
      "maxOrderQty": 12
    },
    "ageRestricted": false,
    "countryOfOrigin": "IDN",
    "logisticsServiceType": "HYPERLOCAL",

    "identity": {
      "brand": "AQUA",
      "sku": "AQUA-600ML"
    },

    "packaged": {
      "netQuantity": { "value": 600, "unit": "MILLILITRE" },
      "manufacturerOrPacker": {
        "type": "MANUFACTURER",
        "name": "PT Tirta Investama (AQUA)",
        "address": "Jl. Raya Mekarsari, Citeureup, Bogor 16810"
      },
      "expiryDate": "2028-04-01",
      "commonOrGenericName": "Air Mineral Dalam Kemasan"
    },

    "regulatory": {
      "registrations": [
        {
          "scheme": "BPOM_MD",
          "id": "MD265510003015",
          "role": "MANUFACTURER",
          "validUntil": "2029-01-01"
        }
      ],
      "foodRegulatoryDeclaration": {
        "nutritionalInfo": "Energi 0 kkal. Tidak mengandung lemak, protein, atau karbohidrat.",
        // Energy 0 kcal. Contains no fat, protein, or carbohydrates.
        "additivesInfo": "Tidak mengandung pengawet"  // Contains no preservatives
      }
    },

    "food": {
      "classification": "HALAL",
      "allergens": []
    }
  }
}
```

**CDS validation result:** ✓ ACCEPTED  
- `TRC-food-bev` schema enforced — food sub-object required ✓  
- BPOM_MD (food) present ✓  
- No pharmacy sub-object — correct, this is not a medicine  
- No `prescriptionRequired` — no delivery-time-kyc triggered  
- BAP renders: BPOM badge, Halal badge, no prescription warning

---

### Item 5 — Optimum Nutrition Whey Protein 2.27kg

**CRC:** `TRC-food-bev` (protein powder = food supplement = BPOM food registration, NOT medicine)  
**Variant triggered:** none

> **Classification note:** ON Gold Standard Whey is BPOM-registered as Pangan Olahan (processed food) with ML (Makanan Luar Negeri — imported) registration. It is NOT a medicine or supplement under PerBPOM pharma regulations. CRC = `TRC-food-bev`, not `TRC-health-beauty`. The fact Bu Sari sells it in her apotek does not change the item's regulatory nature.

```json
{
  "id": "RES-APOTEK-SEHAT-005",
  "descriptor": {
    "name": "Optimum Nutrition Gold Standard 100% Whey 5lb (2.27kg) - Double Rich Chocolate",
    "shortDesc": "Protein whey isolat dan konsentrat premium. 24g protein per sajian.",
    "longDesc": "ON Gold Standard 100% Whey mengandung whey protein isolat, konsentrat, dan peptida. 24g protein per sajian 30g. Cocok untuk pemulihan pasca latihan.",
    "images": [
      "https://cdn.apotek-sehat.id/on-whey-5lb-chocolate.jpg",
      "https://cdn.apotek-sehat.id/on-whey-5lb-nutrition-facts.jpg"
    ]
  },
  "resourceAttributes": {
    "@context": "https://schema.ion.id/trade/resource/v1/context.jsonld",
    "@type": "ion:TradeResource",

    "quantity": { "value": 1, "unit": "piece" },
    "resourceStructure": "VARIANT",
    "resourceTangibility": "PHYSICAL",
    "images": [
      "https://cdn.apotek-sehat.id/on-whey-5lb-chocolate.jpg",
      "https://cdn.apotek-sehat.id/on-whey-5lb-nutrition-facts.jpg"
    ],
    "availability": { "status": "IN_STOCK", "maxOrderQty": 2 },
    "ageRestricted": false,
    "countryOfOrigin": "USA",
    "logisticsServiceType": "DOMESTIC_SURFACE",

    "parentResourceId": "RES-APOTEK-SEHAT-005-BASE",
    "isDefaultVariant": true,
    "variantGroup": {
      "id": "VG-ON-WHEY-FLAVOUR",
      "name": { "id": "Pilihan Rasa", "en": "Flavour" },
      "variantOn": "flavour"
    },

    "identity": {
      "brand": "Optimum Nutrition",
      "model": "Gold Standard 100% Whey",
      "sku": "ON-WHEY-5LB-CHOC"
    },

    "productCode": {
      "type": "GTIN",
      "value": "0748927028927"
    },

    "physical": {
      "weightKg": 2.27
    },

    "packaged": {
      "netQuantity": { "value": 2270, "unit": "GRAM" },
      "manufacturerOrPacker": {
        "type": "IMPORTER",
        "name": "PT Nutrisi Prima Indonesia",
        "address": "Jl. Gatot Subroto Kav. 36-38, South Jakarta 12950"
      },
      "expiryDate": "2027-06-30",
      "commonOrGenericName": "Suplemen Protein Whey",
      "hsnCode": "2106.10"
    },

    "regulatory": {
      "registrations": [
        {
          "scheme": "BPOM_ML",
          "id": "ML235233011014",
          "role": "IMPORTER",
          "validUntil": "2028-06-01"
        }
      ],
      "foodRegulatoryDeclaration": {
        "nutritionalInfo": "Per sajian 30g: Energi 120 kkal, Protein 24g, Lemak Total 1.5g, Karbohidrat 3g, Gula 1g, Natrium 130mg",
        // Per 30g serving: Energy 120 kcal, Protein 24g, Total Fat 1.5g, Carbs 3g, Sugar 1g, Sodium 130mg
        "additivesInfo": "Mengandung sucralose, soya lecithin"  // Contains sucralose, soya lecithin
      },
      "nutrition": [
        { "nutrient": "Protein", "value": { "unitCode": "G", "unitQuantity": 24 } },
        { "nutrient": "Lemak Total", "value": { "unitCode": "G", "unitQuantity": 1.5 } },
        { "nutrient": "Karbohidrat", "value": { "unitCode": "G", "unitQuantity": 3 } },
        { "nutrient": "Energi", "value": { "unitCode": "KCAL", "unitQuantity": 120 } },
        { "nutrient": "Natrium", "value": { "unitCode": "MG", "unitQuantity": 130 } }
      ]
    },

    "food": {
      "classification": "HALAL",
      "allergens": ["DAIRY", "SOY"],
      "freshProduce": null
    }
  }
}
```

**CDS validation result:** ✓ ACCEPTED  
- `TRC-food-bev` schema enforced  
- BPOM_ML (imported food) present ✓  
- Nutrition facts declared ✓  
- Allergens declared (DAIRY, SOY) ✓  
- VARIANT structure — flavour variants will be child resources  
- GTIN + hsnCode declared (imported product) ✓  
- BAP renders: BPOM badge, Halal badge, allergen warnings, nutrition panel

---

### Offer Attributes — same structure across all 5 items, tuned per item

Each resource gets an `Offer` with `offerAttributes`. Here's how they differ:

```
Panadol (OTC — Over the Counter, no prescription needed):
  cancellationPolicy: ion://policy/cancel.prepacked.free
  returnable: false
  returnPolicy: ion://policy/return.pharmacy.nonreturnable
  warrantyPolicy: ion://policy/warranty.standard.none
  availableOnCod: true
  paymentTermsPolicy: ion://policy/payment-terms.upfront.full

Amoxicillin (Prescription — requires valid prescription):
  cancellationPolicy: ion://policy/cancel.standard.none
  returnable: false
  returnPolicy: ion://policy/return.pharmacy.nonreturnable
  warrantyPolicy: ion://policy/warranty.standard.none
  availableOnCod: false        ← COD not permitted for prescription medicine
  proofOfDeliveryType: OTP     ← OTP at door required
  paymentTermsPolicy: ion://policy/payment-terms.upfront.full

Metformin (Subscription — monthly recurring delivery):
  cancellationPolicy: ion://policy/cancel.subscription.prorated-midcycle
  returnable: false
  returnPolicy: ion://policy/return.pharmacy.nonreturnable
  warrantyPolicy: ion://policy/warranty.standard.none
  availableOnCod: false
  proofOfDeliveryType: OTP
  paymentTermsPolicy: ion://policy/payment-terms.subscription.monthly_autopay

Aqua (Water):
  cancellationPolicy: ion://policy/cancel.prepacked.free
  returnable: false
  returnPolicy: ion://policy/return.grocery.sameday-defect-only
  warrantyPolicy: ion://policy/warranty.standard.none
  availableOnCod: true
  paymentTermsPolicy: ion://policy/payment-terms.upfront.full

ON Whey Protein:
  cancellationPolicy: ion://policy/cancel.prepacked.free
  returnable: true
  returnPolicy: ion://policy/return.standard.7d-sellerpays
  sellerPickupReturn: true
  warrantyPolicy: ion://policy/warranty.standard.none
  availableOnCod: true
  returnEvidenceRequirement: PHOTO_REQUIRED
  returnEvidenceMinPhotos: 2
  paymentTermsPolicy: ion://policy/payment-terms.upfront.full
```

---

# PHASE 2 — DISCOVERY
*Wizard cross-reference: Problem 4 — "My store is set up — how do customers find me?"*

*ION hierarchy layers engaged: CRC (discovery indexing function)*

## Pak Ahmad searches for his medicines

Pak Ahmad opens the buyer app. He searches: *"metformin langganan"* (metformin subscription)

**POST /discover:**
```json
{
  "context": {
    "action": "discover",
    "domain": "ion:trade",
    "transactionId": "TXN-2026-8f3a21",
    "messageId": "MSG-2026-d4b891",
    "bapId": "buyerapp.sehat-kita.ion.id",
    "bapUri": "https://buyerapp.sehat-kita.ion.id/beckn",
    "timestamp": "2026-04-19T09:15:00+07:00"
  },
  "message": {
    "intent": {
      "item": {
        "descriptor": { "name": "metformin" }
      },
      "provider": {
        "location": {
          "gps": "-6.2611,106.8154",
          "radius": { "value": "2", "unit": "km" }
        }
      }
    }
  }
}
```

**POST /on_discover** returns Bu Sari's catalog. Pak Ahmad sees:
- Metformin 500mg 30 Tablet — IDR 45,000 — Available for subscription
- "Prescription required" badge
- "Subscription available — monthly delivery" badge
- BPOM registration shown ✓

He also sees Panadol, Aqua, and ON Whey in the catalog — because all are within 800m of him and Bu Sari's store is OPEN.

---

# PHASE 3 — TRANSACTIONS
*Wizard cross-reference: Problems 5, 6, 7 — Standard order (Panadol) · Prescription order (Amoxicillin) · Subscription (Metformin)*

*ION hierarchy layers engaged: Pattern (storefront, subscription) · Variant (delivery-time-kyc)*

## Transaction A: Panadol (OTC, happy-flow)

A different buyer, Ibu Rina, wants Panadol for her fever.

### /select
```json
{
  "context": {
    "action": "select",
    "domain": "ion:trade",
    "transactionId": "TXN-2026-PAN-001",
    "messageId": "MSG-2026-sel-001",
    "bapId": "buyerapp.sehat-kita.ion.id",
    "bapUri": "https://buyerapp.sehat-kita.ion.id/beckn",
    "bppId": "sellerapp.apotek-sehat.ion.id",
    "bppUri": "https://sellerapp.apotek-sehat.ion.id/beckn",
    "timestamp": "2026-04-19T10:30:00+07:00"
  },
  "message": {
    "contract": {
      "commitments": [
        {
          "id": "CMT-001",
          "commitmentAttributes": {
            "@context": "https://schema.ion.id/trade/commitment/v1/context.jsonld",
            "@type": "ion:TradeCommitment",
            "lineId": "L01",
            "resourceId": "RES-APOTEK-SEHAT-001",
            "offerId": "OFFER-PANADOL-001",
            "quantity": { "count": 2 }
          }
        }
      ]
    }
  }
}
```

### /on_select
BPP returns quote. 2 × Panadol + delivery:
```json
{
  "context": { "action": "on_select", "...": "..." },
  "message": {
    "contract": {
      "commitments": [
        {
          "id": "CMT-001",
          "commitmentAttributes": {
            "@context": "https://schema.ion.id/trade/commitment/v1/context.jsonld",
            "@type": "ion:TradeCommitment",
            "lineId": "L01",
            "resourceId": "RES-APOTEK-SEHAT-001",
            "offerId": "OFFER-PANADOL-001",
            "quantity": { "count": 2 },
            "price": { "value": "24000", "currency": "IDR" }
          }
        }
      ],
      "considerations": [
        {
          "id": "CON-ITEM",
          "considerationAttributes": {
            "@context": "https://schema.ion.id/trade/consideration/v1/context.jsonld",
            "@type": "ion:TradeConsideration",
            "breakupLineType": "ITEM",
            "totalAmount": 24000,
            "currency": "IDR"
          }
        },
        {
          "id": "CON-DELIVERY",
          "considerationAttributes": {
            "@context": "https://schema.ion.id/trade/consideration/v1/context.jsonld",
            "@type": "ion:TradeConsideration",
            "breakupLineType": "DELIVERY",
            "totalAmount": 12000,
            "currency": "IDR"
          }
        }
      ]
    }
  }
}
```

### /init → /on_init → /confirm → /on_confirm

Ibu Rina proceeds through init (provides address) and confirms. Payment via QRIS.

**on_confirm** returns the active contract with performance attributes:

```json
{
  "contract": {
    "id": "ORD-2026-APOTEK-001",
    "status": "ACTIVE",
    "contractAttributes": {
      "@context": "https://schema.ion.id/trade/contract/v1/context.jsonld",
      "@type": "ion:TradeContract",
      "fulfillingLocationId": "LOC-KEMANG-47"
    },
    "performance": [
      {
        "id": "PERF-001",
        "status": { "code": "ACCEPTED" },
        "performanceAttributes": {
          "@context": "https://schema.ion.id/trade/performance/v1/context.jsonld",
          "@type": "ion:TradePerformance",
          "performanceMode": "DELIVERY",
          "supportedPerformanceModes": ["DELIVERY", "SELF_PICKUP"],
          "sla": {
            "min": "PT30M",
            "max": "PT90M",
            "unitBasis": "ORDER_CONFIRMATION"
          }
        }
      }
    ]
  }
}
```

### /on_status — DISPATCHED

30 minutes later, order is on the way:

```json
{
  "performance": [{
    "status": { "code": "OUT_FOR_DELIVERY" },
    "performanceAttributes": {
      "@type": "ion:TradePerformance",
      "performanceMode": "DELIVERY",
      "supportedPerformanceModes": ["DELIVERY"],
      "sla": { "min": "PT30M", "max": "PT90M", "unitBasis": "ORDER_CONFIRMATION" },
      "agentName": "Budi Santoso",
      "agentPhone": "081298765432",
      "agentId": "RIDER-BSN-0091",
      "agentVehicleType": "MOTORCYCLE",
      "agentPlateNumber": "B 4821 KLM",
      "estimatedDeliveryTime": "2026-04-19T11:15:00+07:00",
      "lspSubscriberId": "lsp.gosend.ion.id"
    }
  }]
}
```

### /on_status — DELIVERED

Delivery confirmed. No OTP required for OTC medicine — standard delivery.

```json
{
  "performance": [{
    "status": { "code": "DELIVERED" },
    "performanceAttributes": {
      "@type": "ion:TradePerformance",
      "performanceMode": "DELIVERY",
      "supportedPerformanceModes": ["DELIVERY"],
      "sla": { "min": "PT30M", "max": "PT90M", "unitBasis": "ORDER_CONFIRMATION" },
      "deliveryProofUrl": "https://sellerapp.apotek-sehat.ion.id/proofs/ORD-2026-APOTEK-001.jpg"
    }
  }]
}
```

**Happy path complete.** ✓

---

## Transaction B: Amoxicillin (Prescription — delivery-time-kyc variant)

Pak Budi needs antibiotics. He has a prescription from his doctor.

**The difference from Transaction A:** `prescriptionRequired = true` → delivery-time-kyc variant is MANDATORY.

### /on_confirm — with OTP

BPP generates a delivery OTP at confirm time:

```json
{
  "contract": {
    "id": "ORD-2026-APOTEK-002",
    "status": "ACTIVE",
    "performance": [{
      "id": "PERF-002",
      "status": { "code": "ACCEPTED" },
      "performanceAttributes": {
        "@type": "ion:TradePerformance",
        "performanceMode": "DELIVERY",
        "supportedPerformanceModes": ["DELIVERY"],
        "sla": {
          "min": "PT30M",
          "max": "PT2H",
          "unitBasis": "ORDER_CONFIRMATION"
        },
        "deliveryOtp": "482930",
        "ageVerificationRequired": false
      }
    }]
  }
}
```

Pak Budi receives OTP "482930" in his buyer app.

### /on_status — OUT_FOR_DELIVERY

Agent arrives at the door. Agent shows ID, Pak Budi shows OTP "482930" on his phone.

```json
{
  "performance": [{
    "status": { "code": "OUT_FOR_DELIVERY" },
    "performanceAttributes": {
      "@type": "ion:TradePerformance",
      "performanceMode": "DELIVERY",
      "supportedPerformanceModes": ["DELIVERY"],
      "sla": { "min": "PT30M", "max": "PT2H", "unitBasis": "ORDER_CONFIRMATION" },
      "agentName": "Dewi Rahayu",
      "agentPhone": "081234509876",
      "agentId": "RIDER-DRY-0047",
      "deliveryOtp": "482930",
      "agentVehicleType": "MOTORCYCLE"
    }
  }]
}
```

### /on_status — DELIVERED

Agent confirms OTP match. Amoxicillin handed over. Proof of delivery captured.

```json
{
  "performance": [{
    "status": { "code": "DELIVERED" },
    "performanceAttributes": {
      "@type": "ion:TradePerformance",
      "performanceMode": "DELIVERY",
      "supportedPerformanceModes": ["DELIVERY"],
      "sla": { "min": "PT30M", "max": "PT2H", "unitBasis": "ORDER_CONFIRMATION" },
      "deliveryProofUrl": "https://sellerapp.apotek-sehat.ion.id/proofs/ORD-2026-APOTEK-002.jpg"
    }
  }]
}
```

**Delivery-time-kyc variant complete.** ✓

---

## Transaction C: Metformin Subscription (Monthly auto-renewal)

Pak Ahmad sets up his monthly Metformin subscription.

### /confirm — subscription setup

```json
{
  "context": {
    "action": "confirm",
    "domain": "ion:trade",
    "transactionId": "TXN-2026-MET-SUB-001"
  },
  "message": {
    "contract": {
      "contractAttributes": {
        "@context": "https://schema.ion.id/trade/contract/v1/context.jsonld",
        "@type": "ion:TradeContract",
        "subscriptionBillingCycle": "MONTHLY",
        "subscriptionNextBillingDate": "2026-05-19T00:00:00+07:00"
      }
    }
  }
}
```

### /on_confirm — subscription active

```json
{
  "contract": {
    "id": "ORD-2026-SUB-MET-001",
    "status": "ACTIVE",
    "contractAttributes": {
      "@type": "ion:TradeContract",
      "subscriptionBillingCycle": "MONTHLY",
      "subscriptionNextBillingDate": "2026-05-19T00:00:00+07:00",
      "fulfillingLocationId": "LOC-KEMANG-47"
    },
    "performance": [{
      "id": "PERF-SUB-001",
      "status": { "code": "ACCEPTED" },
      "performanceAttributes": {
        "@type": "ion:TradePerformance",
        "performanceMode": "DELIVERY",
        "supportedPerformanceModes": ["DELIVERY"],
        "sla": {
          "min": "PT2H",
          "max": "P1D",
          "unitBasis": "ORDER_CONFIRMATION"
        },
        "deliveryOtp": "719034"
      }
    }]
  }
}
```

**What happens every month:**
1. BPP initiates the next subscription cycle via `/update`
2. BAP auto-charges Pak Ahmad's registered payment method
3. Fresh OTP generated for each delivery
4. Metformin delivered with OTP verification at door every cycle
5. If Pak Ahmad wants to pause: `subscriptionPauseType: PAUSED_UNTIL_DATE`

---

# PHASE 4 — SOMETHING GOES WRONG (IGM)
*Wizard cross-reference: Problem 8 — "A buyer says a bottle of Aqua was missing — what does Bu Sari do?"*

*ION hierarchy layers engaged: Grievance Management (IGM) · Participant Extensions (seller app internal fields)*

## The Problem

A buyer named Ibu Sinta ordered Aqua 600ml × 6 bottles. She received 5 bottles, not 6. One was missing.

She raises a complaint in her buyer app.

## /support — IGM ticket raised

```json
{
  "context": {
    "action": "support",
    "domain": "ion:trade",
    "transactionId": "TXN-2026-AQUA-003",
    "bapId": "buyerapp.sehat-kita.ion.id",
    "bppId": "sellerapp.apotek-sehat.ion.id"
  },
  "message": {
    "support": {
      "orderId": "ORD-2026-APOTEK-003",
      "channels": [
        {
          "@context": "https://schema.ion.id/core/support/v1/context.jsonld",
          "@type": "ion:SupportTicket",

          "id": "ISSUE-2026-00004821",
          "category": "ITEM",
          "subCategory": "ITM01",

          "complainantInfo": {
            "person": { "name": "Sinta Rahayu" },
            "contact": {
              "phone": "081234567890",
              "email": "sinta@email.com"
            }
          },

          "description": {
            "name": "Pesanan kurang 1 botol",  // Order missing 1 bottle
            "shortDesc": "Memesan 6 botol AQUA 600ml, hanya menerima 5 botol. 1 botol tidak ada dalam paket.",
              // Ordered 6 bottles of AQUA 600ml, received only 5. 1 bottle missing from the package.
            "images": [
              "https://buyerapp.sehat-kita.ion.id/evidence/ISSUE-2026-00004821-1.jpg"
            ]
          },

          "source": {
            "networkParticipantId": "buyerapp.sehat-kita.ion.id",
            "type": "CONSUMER"
          },

          "expectedResponseTime": "PT2H",
          "expectedResolutionTime": "P1D",
          "status": "OPEN",
          "severity": "MEDIUM",
          "escalationLevel": "SELLER",
          "createdAt": "2026-04-19T15:30:00+07:00",
          "updatedAt": "2026-04-19T15:30:00+07:00"
        }
      ]
    }
  }
}
```

## /on_support — Bu Sari responds within PT2H SLA

Bu Sari's team reviews the complaint. They confirm the missing bottle. They agree to reship.

```json
{
  "message": {
    "support": {
      "channels": [
        {
          "@type": "ion:SupportTicket",
          "id": "ISSUE-2026-00004821",
          "status": "RESOLVED",
          "escalationLevel": "SELLER",

          "issueActions": {
            "respondentActions": [
              {
                "respondentAction": "RESOLVED",
                "shortDesc": "Kami konfirmasi kekurangan 1 botol AQUA. Penggantian akan dikirim dalam 2 jam.",
                "updatedAt": "2026-04-19T16:45:00+07:00",
                "updatedBy": {
                  "org": { "name": "sellerapp.apotek-sehat.ion.id::ion:trade" },
                  "person": { "name": "Tim CS Apotek Sehat Mandiri"  // Customer Service Team, Apotek Sehat Mandiri }
                },
                "cascadedLevel": 1
              }
            ]
          },

          "resolution": {
            "shortDesc": "1 botol AQUA pengganti akan dikirim.",  // 1 replacement AQUA bottle will be sent.
            "longDesc": "Kami mohon maaf atas kekurangan dalam pesanan Anda. 1 botol AQUA 600ml pengganti akan dikirimkan dalam 2 jam tanpa biaya tambahan.",
            // We apologise for the shortage in your order. 1 replacement AQUA 600ml bottle will be delivered within 2 hours at no extra charge.
            "actionTriggered": "RESHIP"
          },

          "updatedAt": "2026-04-19T16:45:00+07:00"
        }
      ]
    }
  }
}
```

**SLA met:** Response within PT2H ✓. Resolution: RESHIP. Ibu Sinta rates the resolution: 5/5.

---

# PHASE 5 — RECONCILIATION
*Wizard cross-reference: Problem 9 — "When do I get paid? And how much?"*

*ION hierarchy layers engaged: Reconciliation · Experimental Extensions (future)*

## After Panadol delivery — BAP settles with BPP

After the return window closes (7 days, though Panadol is non-returnable anyway), the BAP initiates reconciliation.

### /reconcile

```json
{
  "context": {
    "action": "reconcile",
    "domain": "ion:trade",
    "transactionId": "TXN-2026-PAN-001"
  },
  "message": {
    "contract": {
      "id": "ORD-2026-APOTEK-001",
      "settlements": [
        {
          "id": "SETTLE-001",
          "settlementAttributes": {
            "@context": "https://ion.id/vocab/reconcile/v1/context.jsonld",
            "@type": "ion:IONReconciliation",

            "reconId": "RECON-2026-00009182",
            "contractId": "ORD-2026-APOTEK-001",

            "settlementBasis": "AFTER_DELIVERY",
            "settlementWindow": "P1D",
            "collectedBy": "BAP",

            "counterparty": {
              "collectorAppId": "buyerapp.sehat-kita.ion.id",
              "receiverAppId": "sellerapp.apotek-sehat.ion.id"
            },

            "amounts": {
              "baseContractAmount": 36000,
              "buyerFinderFeeType": "PERCENT",
              "buyerFinderFeeAmount": 3600,
              "withholdingAmount": 0,
              "netPayableToReceiver": 32400
            },

            "returnWindowExpiry": "2026-04-26T23:59:59+07:00",

            "bankDetails": {
              "settlementType": "BI_FAST",
              "accountNumber": "1234567890",
              "beneficiaryName": "Usaha Dagang Sehat Mandiri",
              "bankName": "Bank Mandiri"
            },

            "reconStatus": "PENDING"
          }
        }
      ]
    }
  }
}
```

**Breakdown:**
```
Base order:      IDR 36,000 (2 × Panadol IDR 24,000 + delivery IDR 12,000)
Finder fee:      IDR 3,600  (10% BAP commission)
Withholding:     IDR 0      (non-returnable medicine, no withholding needed)
Net to Bu Sari:  IDR 32,400
```

### /on_reconcile — AGREED

Bu Sari's seller app agrees with the amounts:

```json
{
  "message": {
    "contract": {
      "settlements": [
        {
          "settlementAttributes": {
            "@type": "ion:IONReconciliation",
            "reconId": "RECON-2026-00009182",
            "reconStatus": "AGREED",
            "recon_status": "01",
            "netSettlementAmount": 32400
          }
        }
      ]
    }
  }
}
```

**reconStatus = AGREED → Settlement transitions DRAFT → COMMITTED.**  
Bu Sari receives IDR 32,400 via BI-Fast. ✓

---

# Summary — What Each Layer Did for Bu Sari

Every layer of the ION entity hierarchy was engaged in this journey. Here is what each one did — and where in this document it appeared.

| ION Layer | What it did for Bu Sari | Where in this document |
|---|---|---|
| **Organisation** | Registered UD Sehat Mandiri with NIB and NPWP as a legal entity on ION | Phase 1 — Step 2 |
| **Role** | Bu Sari = Seller (merchant). Dinda's team = Seller App (platform operator). Two distinct roles — the Seller App connects the Seller to the network | Phase 1 — Step 1 |
| **Profile** | `seller / trade` — Bu Sari's unit of authorisation. One role + one sector = one profile. If she opens a restaurant later, that is a separate profile | Phase 1 — Step 1 |
| **Sector** | Trade — buyers end up owning her products. Determined which governance rules, patterns, and policies apply | Phase 1 — Step 1 |
| **Segment** | TRD-04 Pharmacy & Health Retail — triggered PBF licence requirement. Mapped KBLI 47730, 47740 | Phase 1 — Step 1 |
| **CRC (TRC-health-beauty)** | Enforced BPOM pharma registration, pharmacy sub-object (dosage form, strength, active ingredients, prescription flag). Triggered delivery-time-kyc variant when prescriptionRequired=true. Triggered DOMESTIC_ONLY data residency for Rx drugs | Phase 1 — Items 1, 2, 3 |
| **CRC (TRC-food-bev)** | Enforced food BPOM registration, food sub-object (halal, allergens, nutrition facts). Completely different schema from TRC-health-beauty — correctly separated because Aqua and ON Whey are food, not medicine | Phase 1 — Items 4, 5 |
| **Resource / Item** | Each of Bu Sari's 5 products became a Resource in the ION catalogue — the atomic unit of what can be transacted | Phase 1 — Items 1–5 |
| **Pattern (storefront)** | Standard browse-and-buy flow for Panadol, Amoxicillin, Aqua, ON Whey. The transaction shape: discover → select → confirm → fulfil → settle | Phase 3 — Transactions A, B |
| **Pattern (subscription)** | Recurring monthly flow for Metformin. Pak Ahmad confirms once. System auto-initiates every month. Bu Sari receives a standard order notification each cycle | Phase 3 — Transaction C |
| **Variant (delivery-time-kyc)** | Triggered automatically by prescriptionRequired=true on Amoxicillin and Metformin. OTP generated at confirm. Agent verifies at door. Bu Sari did NOT configure this — it was derived from her product attribute | Phase 3 — Transactions B, C |
| **Offer attributes** | Different cancellation policies, return policies, COD eligibility, and payment terms per item. COD blocked for prescription medicines. Non-returnable policy for pharmacy. Subscription auto-pay for Metformin | Phase 1 — Offer section |
| **Performance attributes** | Delivery agent details, AWB, live tracking, OTP at door, delivery proof — all surfaced correctly per the pattern and variant in play | Phase 3 — all transactions |
| **Grievance / IGM** | Ibu Sinta's missing Aqua complaint. Structured ticket with 2-hour SLA. Bu Sari responded in 75 minutes. RESHIP action triggered. SLA met — no escalation, no penalty | Phase 4 |
| **Participant Extensions** | The seller app (`sellerapp.apotek-sehat.ion.id`) may add its own internal fields to tickets and catalogue entries — internal case IDs, team assignments, loyalty tiers — under its own namespace. These never travel on the ION wire | Phase 4 — IGM note |
| **Reconciliation** | BAP settled net amount to BPP after delivery. 10% finder fee deducted. Zero withholding for non-returnable medicines. 10% holdback for returnable ON Whey. Bu Sari received IDR 32,400 for the Panadol order via BI-Fast | Phase 5 |
| **Experimental Extensions** | Not yet active for Bu Sari's items. Future examples: carbon footprint per shipment, loyalty point accrual — piloted by the ION Sector Working Group before becoming standard fields | Phase 5 — noted |
| **Test Set / Test Cases** | Pre-go-live — Dinda's team ran the ION conformance test suite for the `storefront`, `subscription`, and `delivery-time-kyc` combinations before Bu Sari's store went live. Bu Sari never saw this. It ensured the seller app was built correctly before any real customer was affected | Pre-Phase 1 |

---

**Bu Sari never used the words "variant," "CRC," "profile," or "reconcile."**
She answered: what business she runs, what she sells, and how each product should behave.
The network derived everything else.

---

*This journey document covers the complete ION lifecycle for a Trade/TRD-04 pharmacy seller across 5 item types, 2 patterns, 2 variants, the IGM flow, and financial reconciliation.*
*It is the technical companion to the Bu Sari Wizard document — read the Wizard first for context, then this document for the exact API calls and payloads behind each step.*
