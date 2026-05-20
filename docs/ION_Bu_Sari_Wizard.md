# Bu Sari Joins ION
## A Step-by-Step Guide for Sellers

**What this is:** A plain-language walkthrough of joining ION as a seller — told through the story of Bu Sari, who runs a pharmacy in Kemang, South Jakarta.

**Who this is for:** Business owners, product managers, regulators, investors — anyone who wants to understand what ION does without reading protocol specifications.

**Language note:** Bu Sari's world is in Bahasa Indonesia — her store, her customers, her screens. English labels and explanations sit alongside everything so any reader can follow. Where an Indonesian term appears, its meaning is given immediately.

**The technical companion:** Every step here has a corresponding section in the **Bu Sari Complete Journey** document, which contains the full API calls and JSON payloads for developers.

---

## The Players in This Story

**Bu Sari** — owner of Apotek Sehat Mandiri (Independent Healthy Pharmacy), Jl. Kemang Raya No. 47, South Jakarta. Has run the pharmacy for 11 years. Three staff. Loyal neighbourhood customers.

**Dinda** — Bu Sari's daughter. Handles the technology. Her team builds and runs the seller app that connects Bu Sari's pharmacy to ION.

> *In ION's terms: Dinda's team is the **Seller App** — the platform operator. Bu Sari is the **Seller** — the merchant. These are two distinct roles on the network. A Seller App can connect many Sellers. A Seller is one merchant on that platform.*

**Pak Ahmad** — a regular customer. Diabetic. Buys Metformin monthly. Lives 800m away.

**Ibu Rina** — a nearby customer who orders Panadol for her fever.

**Ibu Sinta** — a customer who had a complaint.

**Pak Budi** — a customer who needed prescription antibiotics.

---

## What Bu Sari Sells

| # | Item | What it is |
|---|---|---|
| 1 | Panadol 500mg | OTC painkiller / fever reducer |
| 2 | Amoxicillin 500mg | Prescription antibiotic |
| 3 | Metformin 500mg | Chronic diabetes medicine — monthly subscription |
| 4 | Aqua 600ml | Mineral water |
| 5 | ON Gold Standard Whey 2.27kg | Protein powder supplement |

Five items. Same seller. But each behaves differently on the network — different rules, different delivery procedures, different payment terms. The story of *why* is what this document tells.

---

# PROBLEM 1
## "I want to sell online — but where do I even start?"

**Bu Sari's problem:** She has never sold online. She doesn't know which platform to join, what category she belongs to, or what documents she needs. Every platform asks different questions and wants different information.

**How ION solves it:** Asks one foundational question. Everything else follows from the answer.

> **ION hierarchy layer engaged:** This is where ION establishes Bu Sari's **Sector** (which of ION's six commerce domains she belongs to) and her **Segment** (her specific business category within that domain). The Sector determines which governance rules apply. The Segment determines which regulations, licences, and compliance checks she needs.

---

### Step 1 — "What kind of business do you run?"

The seller app shows Bu Sari a simple screen:

```
┌─────────────────────────────────────────────────────┐
│  Welcome to ION                                     │
│                                                     │
│  Bisnis Anda menjual apa?                           │
│  (What does your business sell?)                    │
│                                                     │
│  ○  Produk fisik                                    │
│     Physical goods — items that get shipped         │
│  ○  Makanan & minuman siap saji                     │
│     Prepared food & beverages                       │
│  ○  Transportasi & perjalanan                       │
│     Transport & travel                              │
│  ○  Layanan keuangan                                │
│     Financial services                              │
│  ○  Jasa profesional                                │
│     Professional services                           │
│  ○  Pengiriman & logistik                           │
│     Delivery & logistics                            │
└─────────────────────────────────────────────────────┘
```

Bu Sari selects: **Produk fisik (Physical goods)**

```
┌─────────────────────────────────────────────────────┐
│  Jenis toko Anda?                                   │
│  (What type of store?)                              │
│                                                     │
│  ○  Supermarket / minimarket                        │
│  ●  Apotek & produk kesehatan   ← Bu Sari selects  │
│     Pharmacy & health products       this           │
│  ○  Fashion & pakaian                               │
│     Fashion & clothing                              │
│  ○  Elektronik — Electronics                        │
│  ○  Otomotif — Automotive                           │
│  ○  Grosir — Wholesale                              │
│  ○  Lainnya — Other                                 │
└─────────────────────────────────────────────────────┘
```

**Two answers. ION now knows Bu Sari's complete business classification:**

```
✓  Sektor (Sector):    Perdagangan — Trade
                       Buyers end up owning her products.
                       This is the right sector — not
                       Hospitality (experiences), not
                       Finance (money), not Services
                       (expertise).

✓  Segmen (Segment):   TRD-04 — Pharmacy & Health Retail
                       Her specific business category
                       within Trade.

✓  KBLI diperlukan:    47730 — retail pharmaceutical goods
   (Required business  47740 — retail medical equipment
    classification     These should already be on her
    codes from         NIB (Nomor Induk Berusaha —
    Indonesia's OSS    Business Registration Number)
    system)
```

**What Bu Sari sees:**

```
✓  Sektor Anda: Perdagangan (Trade)
✓  Kategori: Apotek & Produk Kesehatan (TRD-04)
✓  KBLI yang diperlukan: 47730, 47740
   Langkah berikutnya: Verifikasi NIB dan izin usaha
   (Next step: Verify your NIB and business licences)
```

> **For the developer:** `Sector: Trade`, `Segment: TRD-04`, `Profile: seller / trade`. See Journey Document Phase 1 — Step 1.

---

# PROBLEM 2
## "I have my NIB and pharmacy licence — how do I register my store?"

**Bu Sari's problem:** She has all her documents — NIB, NPWP (tax ID), PBF licence (Pedagang Besar Farmasi — pharmaceutical distribution licence) from Dinas Kesehatan DKI Jakarta (Jakarta Health Authority). Every platform asks for these differently.

**How ION solves it:** Asks for exactly what it needs. Validates in the background.

> **ION hierarchy layers engaged:**
>
> **Organisation** — ION registers Bu Sari as a legal entity (UD Sehat Mandiri) with her NIB and tax ID.
>
> **Role** — Bu Sari's role on ION is **Seller** (merchant). Dinda's team has a separate role: **Seller App** (platform operator). These are distinct — Bu Sari could switch seller apps without changing her Seller registration.
>
> **Profile** — The combination of Role + Sector becomes Bu Sari's **Profile** on ION: `seller / trade`. A Profile is the unit of authorisation. If Bu Sari later opens a restaurant, that would be a separate profile: `seller / hospitality`. Each profile is independently authorised, independently governed.

---

### Step 2 — "Tell us about your store"

```
┌─────────────────────────────────────────────────────┐
│  Data Toko (Store Details)                          │
│                                                     │
│  Nama toko (Store name):                            │
│    Apotek Sehat Mandiri                             │
│                                                     │
│  Alamat (Address):                                  │
│    Jl. Kemang Raya No. 47, Jakarta Selatan          │
│                                                     │
│  NIB (Business Registration No.):                   │
│    8120017xxxxxx   ✓ verified against OSS           │
│                    (Indonesia's business portal)    │
│                                                     │
│  NPWP (Tax ID):                                     │
│    12.345.678.9-012.000  ✓                          │
│                                                     │
│  Jam buka (Opening hours):                          │
│  Senin–Sabtu / Monday–Saturday:  08.00 – 21.00 WIB │
│  Minggu / Sunday:                09.00 – 18.00 WIB │
│                                                     │
│  Izin kategori (Category licence):                  │
│  PBF Pharmacy licence:                              │
│    PBF-DKI-2024-004892  ✓                          │
│  Berlaku hingga / Valid until: 30 June 2027         │
│                                                     │
│  KYC Status (Identity verification): ✓ APPROVED    │
│  Valid until: 1 April 2028                          │
└─────────────────────────────────────────────────────┘
```

ION validates her NIB against OSS in the background. Her PBF pharmacy licence is on record.

**What Bu Sari sees:**

```
✓  Toko terdaftar di ION (Store registered on ION)
✓  KYC: DISETUJUI (APPROVED)
✓  Izin apotek: VALID (Pharmacy licence: valid)
✓  Siap menambahkan produk (Ready to add products)
```

**What this unlocks:**
- Any buyer app on ION can now find and transact with her store
- Her operating hours are live — no orders accepted outside these windows
- Her PBF licence on record means ION knows she is authorised to sell medicines
- KYC approved — she can publish a catalogue and receive payments

> **For the developer:** `Provider.providerAttributes` with `@type: ion:TradeProvider`. See Journey Document Phase 1 — Step 2.

---

# PROBLEM 3
## "I have 5 products to list — how do I describe them?"

**Bu Sari's problem:** She has five products to list. Each is different. She's worried about making mistakes — especially with the medicine listings.

**How ION solves it:** When Bu Sari adds any product, the first question is: *"What kind of thing is this?"* Her answer determines exactly which fields appear — no more, no less.

> **ION hierarchy layers engaged:**
>
> **CRC (Catalogue Resource Category)** — ION's item classification system. When Bu Sari selects a category for her product, ION assigns a CRC code. The CRC enforces which fields are mandatory and tells buyer apps how to present the item.
>
> **Resource / Item** — each product Bu Sari lists becomes a Resource in ION's catalogue — the atomic unit of what can be transacted.
>

---

### Step 3A — Listing Panadol (OTC Medicine)

Bu Sari taps "Tambah Produk" (Add Product). First question:

```
┌─────────────────────────────────────────────────────┐
│  Produk ini termasuk kategori apa?                  │
│  (What category is this product?)                   │
│                                                     │
│  ●  Obat & suplemen kesehatan   ← Bu Sari selects  │
│     Medicines & health supplements   this           │
│  ○  Kosmetik & perawatan diri                       │
│     Cosmetics & personal care                       │
│  ○  Makanan & minuman                               │
│     Food & beverages                                │
│  ○  Peralatan medis — Medical equipment             │
└─────────────────────────────────────────────────────┘
```

ION assigns CRC: **`TRC-health-beauty`** (Health & Beauty Catalogue Resource Category)

The form that appears is specific to medicines — not a generic product form:

```
┌─────────────────────────────────────────────────────┐
│  Tambah Obat / Suplemen                             │
│  (Add Medicine / Supplement)                        │
│                                                     │
│  Nama produk (Product name):                        │
│    Panadol 500mg 4 Tablet                           │
│  Merek (Brand): Panadol                             │
│  Foto (Photo): [upload]  ← required                 │
│                                                     │
│  ── WAJIB DIISI (REQUIRED) ──────────────────────── │
│  ION enforces these because of the CRC selected     │
│                                                     │
│  Nomor registrasi BPOM (BPOM reg. no.):             │
│    NA18190100521                                    │
│  (BPOM = Badan Pengawas Obat dan Makanan,           │
│   Indonesia's food & drug authority)                │
│                                                     │
│  Bentuk sediaan (Dosage form):                      │
│  ● Tablet  ○ Kapsul  ○ Sirup  ○ Tetes  ○ ...       │
│                                                     │
│  Kekuatan (Strength): 500 mg                        │
│                                                     │
│  Zat aktif (Active ingredient): Parasetamol         │
│                                                     │
│  Perlu resep? (Prescription required?)              │
│  ● Tidak (No)   ○ Ya (Yes)                          │
│                                                     │
│  Perlu cold chain? (Cold chain required?)           │
│  ● Tidak (No)   ○ Ya (Yes)                          │
│                                                     │
│  Tanggal kedaluwarsa (Expiry): September 2027       │
│  Negara asal (Country of origin): Indonesia         │
│                                                     │
│  Harga (Price): Rp 12.000 / strip                   │
│  Stok (Availability): ● Tersedia (In stock)         │
└─────────────────────────────────────────────────────┘
```

Bu Sari marks **Perlu resep: Tidak** (Prescription required: No) — Panadol is OTC.

**What Bu Sari sees after saving:**

```
✓  Panadol 500mg — terdaftar (registered)
✓  BPOM: NA18190100521 — valid
✓  Pengiriman: Standar
   Standard delivery — no special procedure needed
```

**What happened behind the scenes Bu Sari doesn't see:**
- ION validated BPOM number NA18190100521 against the BPOM registry ✓
- CRC `TRC-health-beauty` enforced all mandatory pharmacy fields ✓
- `prescriptionRequired = false` → no special delivery variant triggered
- Panadol is now indexed and discoverable by all buyer apps on ION

---

### Step 3B — Listing Amoxicillin (Prescription Medicine)

Same form. Same CRC. Bu Sari fills in the Amoxicillin details. This time — one answer changes everything:

```
│  Perlu resep? (Prescription required?)              │
│  ○ Tidak (No)   ● Ya (Yes)   ← THIS changes        │
│                                   everything         │
```

**What Bu Sari sees after saving:**

```
✓  Amoxicillin 500mg — terdaftar (registered)
✓  BPOM: GKL0309337710A1 — valid

⚠  Produk ini memerlukan verifikasi saat pengiriman
   (This product requires verification at delivery)

   ION will automatically require the delivery agent
   to confirm the recipient's identity before handing
   over this medicine.

   The buyer will receive a one-time passcode (OTP)
   in their app. The agent must match this code before
   delivering the package.

   You don't need to set anything up — this is
   handled automatically.
```

**The key moment:** Bu Sari marked one field — Prescription required: Yes. She didn't configure an OTP system. She didn't set up a verification procedure. ION derived the entire delivery verification flow automatically from that single answer.

> **ION hierarchy layer engaged — Variant:**
>
> When Bu Sari marked `prescriptionRequired = true`, ION flagged the **`delivery-time-kyc` Variant** as mandatory for this resource. A **Variant** is a named modification of a transaction flow. The underlying **Pattern** is still `storefront` (standard browse-and-buy). The Variant adds one step: identity verification at the door. Bu Sari never configured a Variant. Her product attribute activated it.

---

### Step 3C — Listing Metformin (Subscription Medicine)

Same form. Prescription required: Yes again. But Bu Sari also enables the subscription option:

```
┌─────────────────────────────────────────────────────┐
│  Tersedia sebagai langganan?                        │
│  (Available as subscription?)                       │
│  ● Ya (Yes)                                         │
│                                                     │
│  Siklus pengiriman (Delivery cycle):                │
│  ● Bulanan (Monthly)                                │
│                                                     │
│  Harga langganan (Subscription price):              │
│  Rp 45.000 / bulan (per month)                      │
└─────────────────────────────────────────────────────┘
```

**What Bu Sari sees:**

```
✓  Metformin 500mg — terdaftar (registered)
✓  Langganan bulanan aktif
   Monthly subscription: active — Rp 45.000/month

⚠  Verifikasi pengiriman aktif (Delivery verification active)
   (Same as Amoxicillin — prescription required)

ℹ  Pelanggan konfirmasi sekali. Sistem proses otomatis
   tiap bulan.
   Customer confirms once. System processes automatically
   every month.
```

> **ION hierarchy layer engaged — Pattern:**
>
> Metformin uses a different **Pattern** from Panadol. Panadol uses `storefront` — a single purchase. Metformin uses `subscription` — a recurring transaction. Same Sector, same Segment, same CRC, same seller — but a completely different transaction flow. The Pattern defines the shape of the transaction. On top of that, the `delivery-time-kyc` Variant still applies on every subscription cycle — because `prescriptionRequired = true`.

---

### Step 3D — Listing Aqua 600ml (Mineral Water)

Bu Sari adds Aqua. This time she selects a **completely different category:**

```
┌─────────────────────────────────────────────────────┐
│  Produk ini termasuk kategori apa?                  │
│  (What category is this product?)                   │
│                                                     │
│  ○  Obat & suplemen kesehatan (Medicines)           │
│  ●  Makanan & minuman          ← DIFFERENT          │
│     Food & beverages              category          │
└─────────────────────────────────────────────────────┘
```

ION assigns CRC: **`TRC-food-bev`** (Food, Beverages & Tobacco Catalogue Resource Category)

The form that appears is completely different from the medicine form:

```
┌─────────────────────────────────────────────────────┐
│  Tambah Makanan / Minuman                           │
│  (Add Food / Beverage)                              │
│                                                     │
│  Nama produk: AQUA Air Mineral 600ml                │
│  (Product name: AQUA Mineral Water 600ml)           │
│                                                     │
│  ── WAJIB DIISI (REQUIRED) ──────────────────────── │
│                                                     │
│  Nomor BPOM (BPOM reg. no.): MD265510003015         │
│  (MD = Makanan Dalam Negeri — domestic food.        │
│   Different from medicine BPOM numbers)             │
│                                                     │
│  Berat bersih (Net weight): 600 ml                  │
│  Tanggal kedaluwarsa (Expiry): April 2028           │
│  Nama & alamat produsen                             │
│  (Manufacturer name & address):                     │
│    PT Tirta Investama (AQUA), Bogor                 │
│  Negara asal (Country of origin): Indonesia         │
│                                                     │
│  Sertifikasi halal? (Halal certified?)              │
│  ● Ya (Yes)  ○ Tidak (No)                           │
│                                                     │
│  Kandungan alergen (Allergens): None                │
│                                                     │
│  Kandungan gizi (Nutrition facts): [fill or skip]   │
└─────────────────────────────────────────────────────┘
```

**Notice what is absent from this form:**
- No "Perlu resep?" (Prescription required?) field
- No dosage form, strength, or active ingredients
- Instead: halal certification, allergens, nutrition facts — food-specific fields

**What Bu Sari sees:**

```
✓  AQUA 600ml — terdaftar (registered)
✓  BPOM: MD265510003015 — valid (food registration)
✓  Halal: terverifikasi (verified)
✓  Pengiriman: Standar (Standard delivery)
   No special procedure — this is water, not medicine
```

> **ION hierarchy layer — CRC change:**
>
> Aqua has a different **CRC** from Panadol — `TRC-food-bev` vs `TRC-health-beauty`. Same seller, same store, completely different item classification. The CRC determines which fields are mandatory. BPOM registration appears in both CRCs — but for Aqua it is a food BPOM (MD prefix), not a pharmaceutical BPOM. Two different regulatory regimes, derived from one category selection.

---

### Step 3E — Listing ON Whey Protein (Protein Powder)

Bu Sari adds the protein powder. She selects **Makanan & minuman** (Food & beverages) — not medicines.

> **Why food and not medicine?** BPOM classifies protein powder as *Pangan Olahan* (processed food) with ML registration (Makanan Luar Negeri — imported food product). It is not classified as a pharmaceutical. The fact that Bu Sari sells it in her pharmacy does not change what the product legally is. The item's regulatory classification — not where it is sold — determines its CRC.

```
┌─────────────────────────────────────────────────────┐
│  Tambah Makanan / Minuman                           │
│  (Add Food / Beverage)                              │
│                                                     │
│  Nama produk:                                       │
│    ON Gold Standard Whey 2.27kg                     │
│    Double Rich Chocolate                            │
│                                                     │
│  Nomor BPOM: ML235233011014                         │
│  (ML = Makanan Luar Negeri — imported food product) │
│                                                     │
│  Berat bersih (Net weight): 2.270 gram              │
│  Negara asal (Country of origin): United States     │
│  Importir (Importer): PT Nutrisi Prima Indonesia    │
│  Tanggal kedaluwarsa (Expiry): June 2027            │
│                                                     │
│  Halal? ● Ya (Yes)                                  │
│                                                     │
│  Alergen (Allergens):                               │
│  ☑ Susu / Dairy    ☑ Kedelai / Soy                 │
│                                                     │
│  Informasi gizi per sajian 30g                      │
│  (Nutrition facts per 30g serving):                 │
│  Protein: 24g                                       │
│  Lemak / Fat: 1.5g                                  │
│  Karbohidrat / Carbs: 3g                            │
│  Kalori / Calories: 120 kcal                        │
│                                                     │
│  Variasi rasa? (Flavour variants?)                  │
│  ● Ya (Yes)                                         │
│  Double Rich Chocolate ← default                    │
│  Vanilla Ice Cream                                  │
│  Mocha Cappuccino                                   │
└─────────────────────────────────────────────────────┘
```

**What Bu Sari sees:**

```
✓  ON Whey Protein — 3 varian rasa (3 flavour variants)
✓  BPOM ML: ML235233011014 — valid (imported food)
✓  Alergen ditampilkan (Allergens shown to buyers):
   Dairy, Soy — buyer apps will display allergen warnings
✓  Dapat dikembalikan 7 hari (Returnable: 7 days)
   Unlike medicines — this product CAN be returned
```

---

### How the 5 products compare

After listing all five, this is what Bu Sari's catalogue means in ION terms — though she never sees these labels:

| Product | Item Category (CRC) | Pattern | Variant |
|---|---|---|---|
| Panadol | Health & Beauty (`TRC-health-beauty`) | storefront | none |
| Amoxicillin | Health & Beauty (`TRC-health-beauty`) | storefront | delivery-time-kyc |
| Metformin | Health & Beauty (`TRC-health-beauty`) | subscription | delivery-time-kyc |
| Aqua | Food & Beverages (`TRC-food-bev`) | storefront | none |
| ON Whey | Food & Beverages (`TRC-food-bev`) | storefront | none |

Same seller. Same Segment. Two different CRCs, two different Patterns, one Variant — all derived from how Bu Sari honestly described her products.

---

# PROBLEM 4
## "My store is set up — how do customers find me?"

**Bu Sari's problem:** She worries nobody will find her. Big platforms have complicated SEO and listing requirements.

**How ION solves it:** Nothing extra needed. Once her catalogue is published, she is automatically discoverable by every buyer app on ION — health apps, general marketplaces, hyperlocal delivery apps, grocery apps — all at once.

> **ION hierarchy layer engaged — CRC as discovery index:**
>
> The ION Catalogue Discovery Service (CDS) has indexed all five products by CRC. Buyer apps query the CDS with search terms and GPS coordinates. Bu Sari's products appear for anyone within range. The CRC is what organises the index — a health app searching for "antibiotics" finds Amoxicillin under `TRC-health-beauty`. A grocery app searching for "mineral water" finds Aqua under `TRC-food-bev`. Without the CRC, Bu Sari would be a single undifferentiated catalogue entry that no app could intelligently filter.

---

### Step 4 — "Your store is live"

```
┌─────────────────────────────────────────────────────┐
│  Apotek Sehat Mandiri sudah live di ION! 🎉         │
│  (Your store is live on ION!)                       │
│                                                     │
│  Produk aktif (Active products):                    │
│  ✓ Panadol 500mg                                    │
│  ✓ Amoxicillin 500mg — perlu resep (Rx required)   │
│  ✓ Metformin 500mg — langganan tersedia             │
│                        (subscription available)     │
│  ✓ AQUA 600ml                                       │
│  ✓ ON Whey Protein — 3 rasa (3 flavours)           │
│                                                     │
│  Jangkauan pengiriman (Delivery radius): 2km        │
│  Status toko (Store status): BUKA (OPEN)            │
│                                                     │
│  Your products are now discoverable by all buyer    │
│  apps connected to ION within your delivery area.   │
└─────────────────────────────────────────────────────┘
```

---

# PROBLEM 5
## "Someone ordered Panadol — what happens now?"

**Bu Sari's problem:** First order arrives. Standard OTC medicine. What does she actually do?

> **ION hierarchy layer engaged — Pattern (storefront):**
>
> This is the `storefront` **Pattern** in action — ION's blueprint for standard catalogue browse-and-buy transactions for physical goods. No Variant active. This is the baseline happy-flow. The Pattern defines the sequence: discover → select → confirm → fulfil → settle.

---

### Step 5 — A Standard Order (Panadol)

Ibu Rina, 400m away, orders 2 strips of Panadol via a health app on ION.

**Bu Sari's seller app:**

```
┌─────────────────────────────────────────────────────┐
│  📦 Pesanan Baru! (New Order!)                      │
│                                                     │
│  Order #ORD-2026-001                                │
│  Ibu Rina — Jl. Kemang Dalam No. 12                │
│                                                     │
│  2× Panadol 500mg 4 Tab       Rp 24.000            │
│  Ongkos kirim (Delivery fee)  Rp 12.000            │
│  Total                        Rp 36.000            │
│                                                     │
│  Pembayaran (Payment): QRIS ✓ paid                  │
│                                                     │
│  [TERIMA (ACCEPT)]   [TOLAK (DECLINE)]              │
└─────────────────────────────────────────────────────┘
```

Bu Sari accepts. Packs the order. 15 minutes later the GoSend rider arrives. Bu Sari taps:

```
  [PESANAN SIAP DIAMBIL KURIR]
  (ORDER READY FOR RIDER PICKUP)
```

**Ibu Rina's buyer app shows:**

```
  Rider: Budi Santoso, Honda Beat, B 4821 KLM
  Estimasi tiba (ETA): 25 minutes
  Live tracking available
```

Delivered. ✓

**Bu Sari's app:**

```
✓  Order #ORD-2026-001 — TERKIRIM (DELIVERED)
   Pembayaran akan diproses dalam 1 hari
   (Payment will be processed within 1 day)
```

---

# PROBLEM 6
## "Someone ordered Amoxicillin — why is this different?"

**Bu Sari's problem:** Pak Budi ordered Amoxicillin. Her app mentions "verifikasi pengiriman" (delivery verification). She's unsure what she needs to do differently.

**How ION solves it:** Bu Sari does nothing differently. She just packs the order. The buyer app and delivery app handle verification automatically.

> **ION hierarchy layer engaged — Variant (delivery-time-kyc):**
>
> The `delivery-time-kyc` **Variant** is now in action. When Bu Sari marked `prescriptionRequired = true` during listing, ION flagged this Variant as mandatory for every transaction involving this resource. The Pattern (`storefront`) handles the overall flow. The Variant adds one modification: identity confirmation at the door via OTP (one-time passcode). This happens automatically on every Amoxicillin transaction — Bu Sari never configures it each time.

---

### Step 6 — A Prescription Order (Amoxicillin)

```
┌─────────────────────────────────────────────────────┐
│  📦 Pesanan Baru — Obat Resep                       │
│     (New Order — Prescription Medicine)             │
│                                                     │
│  Order #ORD-2026-002                                │
│  Pak Budi — Jl. Kemang Timur No. 8                 │
│                                                     │
│  1× Amoxicillin 500mg 10 Kap   Rp 35.000           │
│  Ongkos kirim (Delivery fee)   Rp 12.000           │
│  Total                         Rp 47.000           │
│                                                     │
│  Payment: QRIS ✓ paid                               │
│                                                     │
│  ⚠  Prescription medicine — the delivery agent     │
│     will verify the recipient's identity before     │
│     handing over this order.                        │
│     A one-time passcode (OTP) has been sent         │
│     to the buyer.                                   │
│                                                     │
│  [TERIMA (ACCEPT)]   [TOLAK (DECLINE)]              │
└─────────────────────────────────────────────────────┘
```

Bu Sari accepts. Packs. Rider picks up.

**What happens at Pak Budi's door — without Bu Sari doing anything:**

```
  Pak Budi's app:
  "Tunjukkan kode ini ke kurir:"
  (Show this code to your rider:)
  482930

  Rider's app:
  "Masukkan kode dari penerima"
  (Enter code from recipient)
  → Pak Budi shows: 482930
  → Rider enters:   482930
  ✓ Kode cocok — serahkan paket
    (Code matches — hand over package)
```

**Bu Sari's app:**

```
✓  Order #ORD-2026-002 — TERKIRIM (DELIVERED)
   Verifikasi identitas: BERHASIL
   (Identity verified: SUCCESS)
```

**The principle:** Bu Sari described her product honestly — "prescription required = yes." The network derived the correct delivery procedure automatically. She did not build an OTP system. She did not instruct the rider. The Variant handled it.

---

# PROBLEM 7
## "Pak Ahmad wants his Metformin every month — without reordering each time"

**Bu Sari's problem:** Pak Ahmad has been a regular for 4 years. He asks if there's a way to get his Metformin delivered automatically. Bu Sari doesn't know how to set up recurring deliveries.

**How ION solves it:** Bu Sari already enabled the subscription option when she listed Metformin. Pak Ahmad just sets it up from his end.

> **ION hierarchy layers engaged — Pattern (subscription) + Variant (delivery-time-kyc):**
>
> The `subscription` **Pattern** handles the recurring billing and auto-initiated deliveries. It is a different Pattern from `storefront` — the transaction shape is fundamentally different: instead of single-purchase, it auto-renews each cycle. The `delivery-time-kyc` **Variant** still applies on every delivery cycle — because `prescriptionRequired = true` on the Metformin resource. The two work together: subscription handles *when* it delivers; the variant handles *how* it verifies at the door.

---

### Step 7 — Setting Up a Subscription

**In Pak Ahmad's buyer app:**

```
┌─────────────────────────────────────────────────────┐
│  Metformin 500mg 30 Tablet — Apotek Sehat Mandiri  │
│                                                     │
│  Rp 45.000 / bulan (per month)                      │
│                                                     │
│  ● Langganan bulanan (Monthly subscription)         │
│  ○ Beli sekali (Single purchase)                    │
│                                                     │
│  With subscription:                                 │
│  ✓ Auto-delivered every month                       │
│  ✓ No need to reorder                              │
│  ✓ Price locked at Rp 45.000/month                  │
│  ✓ Pause or cancel anytime                         │
│  ✓ Prorated refund if cancelled mid-month           │
│                                                     │
│  [MULAI LANGGANAN (START SUBSCRIPTION)]             │
└─────────────────────────────────────────────────────┘
```

Pak Ahmad confirms. First month charged.

**Bu Sari's seller app:**

```
✓  Langganan baru (New subscription): Pak Ahmad
   Metformin 500mg 30 Tab — every month
   Pengiriman pertama (First delivery): today
   Pengiriman berikutnya (Next delivery): 19 May 2026
```

**Every month — automatically:**

```
  1. System charges Pak Ahmad's registered payment
  2. Bu Sari receives an order notification — same as
     any regular order
  3. She packs the Metformin
  4. Rider picks up and delivers
  5. New OTP sent to Pak Ahmad each cycle
     (prescription medicine — always verified at door)
  6. Rider verifies OTP
  7. Metformin delivered ✓
```

**If Pak Ahmad wants to pause:**

```
  [JEDA HINGGA TANGGAL]   [BATALKAN]
  (PAUSE UNTIL DATE)       (CANCEL)

  Cancelled mid-month → prorated refund
  for unused days, automatically.
```

---

# PROBLEM 8
## "A buyer says a bottle of Aqua was missing — what does Bu Sari do?"

**Bu Sari's problem:** Ibu Sinta ordered 6 bottles of Aqua and received only 5. She's upset. Bu Sari wants to fix it quickly and avoid the complaint getting bigger.

**How ION solves it:** The complaint arrives directly in Bu Sari's seller app with a clear timer. She has 2 hours to respond. If she does — resolved. If she doesn't — ION escalates automatically.

> **ION hierarchy layer engaged — Grievance Management (IGM):**
>
> This is ION's consumer complaint channel — built into Beckn's `/support` endpoint using ION's extension. The complaint flows from Ibu Sinta's buyer app through the ION network to Bu Sari's seller app as a structured ticket. The grievance SLA that Bu Sari's offer declared at catalogue time — "2 hours to respond, 1 day to resolve" — is now being enforced automatically. Miss the SLA and the complaint escalates through five levels: Seller → Marketplace Mediation → BPSK (Badan Penyelesaian Sengketa Konsumen — Indonesia's government consumer dispute body) → Kominfo (national consumer portal) → ION Network formal raise ticket. Bu Sari doesn't need to know any of this. She just needs to respond within 2 hours.
>
> This also illustrates the **Participant Extensions** zone — the seller app may add its own internal fields to the complaint ticket (internal case ID, team assignment, escalation notes) under its own namespace. These never travel on the ION network wire — they are the seller app's private operational data.

---

### Step 8 — Handling a Complaint

**Ibu Sinta raises a complaint in her buyer app:**

```
┌─────────────────────────────────────────────────────┐
│  Laporkan masalah (Report a problem)                │
│  Order #ORD-2026-003                                │
│                                                     │
│  Masalahnya apa? (What is the problem?)             │
│  ●  Barang kurang / tidak lengkap                   │
│     Missing or incomplete order                     │
│  ○  Barang salah (Wrong item)                       │
│  ○  Barang rusak (Damaged item)                     │
│  ○  Pesanan tidak datang (Order didn't arrive)      │
│                                                     │
│  Deskripsi singkat:                                 │
│  "Pesan 6 botol AQUA, terima 5 botol saja."        │
│  "Ordered 6 AQUA bottles, only received 5."        │
│                                                     │
│  [Lampirkan foto (Attach photo)]                    │
│  [KIRIM LAPORAN (SUBMIT REPORT)]                    │
└─────────────────────────────────────────────────────┘
```

**Bu Sari's seller app immediately shows:**

```
┌─────────────────────────────────────────────────────┐
│  🔔 Komplain masuk — perlu direspons                │
│     (Complaint received — response required)        │
│                                                     │
│  From: Ibu Sinta                                    │
│  Order: #ORD-2026-003 — AQUA 600ml × 6             │
│  Issue: Missing item — 1 bottle not received       │
│                                                     │
│  ⏱  Batas respons: 2 jam                            │
│     Response deadline: 2 hours                      │
│     (before 17:45 today)                            │
│                                                     │
│  Pilih tindakan (Choose action):                    │
│  ● Kirim ulang barang yang kurang                   │
│    Reship the missing item                          │
│  ○ Refund sebagian (Partial refund)                 │
│  ○ Minta info lebih lanjut (Request more info)      │
│                                                     │
│  Tambahkan pesan (Add a message):                   │
│  "Maaf Bu Sinta. 1 botol AQUA pengganti akan        │
│   kami kirim dalam 2 jam."                          │
│  "Apologies. We will reship 1 AQUA bottle           │
│   within 2 hours."                                  │
│                                                     │
│  [KIRIM RESPONS (SEND RESPONSE)]                    │
└─────────────────────────────────────────────────────┘
```

Bu Sari selects Reship. Sends the response.

**Ibu Sinta's app:**

```
✓  Penjual sudah merespons (Seller has responded)
   Tindakan / Action: Pengiriman ulang (Reship)
   "1 botol AQUA pengganti dalam 2 jam"
```

**Bu Sari responded in 75 minutes.** Within her 2-hour SLA. No escalation. No penalty. Ibu Sinta rates the resolution 5 out of 5.

**What would have happened without a response:**

```
  2 hours:  → Escalated to marketplace mediation
  5 days:   → Escalated to BPSK
               (government consumer dispute body)
  14 days:  → Kominfo consumer protection portal
  Further:  → ION Network formal raise ticket
```

None of this happened — Bu Sari responded in time.

---

# PROBLEM 9
## "When do I get paid? And how much?"

**Bu Sari's problem:** She wants to understand when the money arrives, how much she actually receives, and whether there are deductions she should know about.

**How ION solves it:** Shows her a transparent settlement summary. No surprises.

> **ION hierarchy layer engaged — Reconciliation:**
>
> This is the ION-specific financial true-up between the buyer app (which collected payment from customers) and Bu Sari's seller app (which fulfilled the orders). After delivery, the two systems automatically agree on the net amounts — base order value minus the platform's commission, minus any holdback for returnable items — and the money moves to Bu Sari via BI-Fast (Bank Indonesia's instant payment rail). Bu Sari just sees the clean result.
>
> This is also where the **Experimental extension zone** could be relevant in future — for example, carbon footprint offsets per order, or loyalty point accrual calculations, piloted by the ION Sector Working Group before becoming standard fields.

---

### Step 9 — Getting Paid

**Bu Sari's seller app — settlement view:**

```
┌─────────────────────────────────────────────────────┐
│  Pembayaran akan masuk (Incoming payment)           │
│                                                     │
│  Order #ORD-2026-001 — Panadol                      │
│  Status: Siap dibayar (Ready to settle)             │
│                                                     │
│  Rincian (Breakdown):                               │
│  Total pesanan (Order total):     Rp  36.000        │
│  Biaya platform 10%              -Rp   3.600        │
│  (Platform fee 10%)                                 │
│  Penahanan jaminan (Holdback):    Rp       0        │
│  (No holdback — medicines are non-returnable)       │
│  ────────────────────────────────────────           │
│  Yang Anda terima (You receive):  Rp  32.400        │
│                                                     │
│  Via: BI-Fast → Bank Mandiri                        │
│  ETA: today before 17.00 WIB                        │
└─────────────────────────────────────────────────────┘
```

**For the ON Whey Protein order — slightly different:**

```
┌─────────────────────────────────────────────────────┐
│  Order — ON Whey Protein                            │
│                                                     │
│  Total pesanan:              Rp 450.000             │
│  Biaya platform 10%:        -Rp  45.000             │
│  Penahanan jaminan 10%:     -Rp  45.000             │
│  (7-day return holdback —                           │
│   protein powder can be returned)                   │
│  ────────────────────────────────────────           │
│  Anda terima sekarang:       Rp 360.000             │
│  Sisa setelah 7 hari:        Rp  45.000             │
│  (Remainder released after 7 days)                  │
│                                                     │
│  Total yang akan Anda terima: Rp 405.000            │
└─────────────────────────────────────────────────────┘
```

**Why the difference?**
- Panadol: non-returnable medicine → zero holdback → full net payment same day
- ON Whey: 7-day return window → 10% held temporarily → released automatically if no return

Bu Sari understands exactly what is held and why. No surprises.

---

# THE COMPLETE PICTURE

Every hierarchy layer Bu Sari encountered — and when:

```
ION LAYER             When Bu Sari encountered it
──────────────────────────────────────────────────────────

Organisation          Step 2 — registered as UD Sehat Mandiri
                      with NIB and NPWP

Role                  Implicit throughout — Bu Sari is a Seller.
                      Dinda's team is the Seller App.
                      Different roles, different responsibilities.

Profile               Step 2 — seller / trade
                      Her unit of authorisation on ION.
                      One profile = one role + one sector.

Sector                Step 1 — Trade
                      (buyers end up owning her products)

Segment               Step 1 — TRD-04 Pharmacy & Health Retail
                      (her specific business category)

CRC                   Step 3 — item classification
                      TRC-health-beauty: Panadol, Amoxicillin,
                                         Metformin
                      TRC-food-bev:      Aqua, ON Whey

Resource / Item       Step 3 — each product listed becomes
                      a Resource in the ION catalogue

Pattern               Steps 5–7 — the transaction flow shape
                      storefront:   Panadol, Amoxicillin,
                                    Aqua, ON Whey
                      subscription: Metformin

Variant               Steps 6–7 — modification of the pattern
                      delivery-time-kyc: Amoxicillin,
                                         Metformin
                      Triggered by prescriptionRequired=true.
                      Bu Sari never configured this.

Grievance / IGM       Step 8 — complaint from Ibu Sinta
                      2-hour SLA enforced automatically
                      5-level escalation path ready
                      (never triggered — Bu Sari responded
                       in time)

Participant           Seller app's internal fields on the
Extensions            complaint ticket — case ID, team
                      assignment. Private to the seller app,
                      never travels on ION wire.

Reconciliation        Step 9 — financial true-up
                      Platform fee deducted
                      Holdback for returnable items
                      Net settled via BI-Fast

Experimental          Not yet active for Bu Sari's items.
Extensions            Future: carbon offset per shipment,
                      loyalty point accrual — piloted by
                      SWG before becoming standard fields.

Test Set /            Pre-go-live — Dinda's team ran the
Test Cases            ION conformance test suite before
                      Bu Sari's store went live. Bu Sari
                      never saw this. It ensured the
                      seller app was built correctly.
```

---

## What Bu Sari Never Had To Do

```
✗  Configure a delivery OTP system
✗  Build a subscription billing engine
✗  Set up a grievance escalation procedure
✗  Calculate platform fees or holdback amounts
✗  Know what a "Variant" or "Pattern" or "CRC" is
✗  Know what "delivery-time-kyc" means
✗  Integrate separately with each buyer app
✗  Register on Tokopedia, Shopee, GoApotik separately
```

## What Bu Sari Did

```
✓  Answered 2 classification questions
✓  Filled in her store details once
✓  Listed each product in the right category
✓  Marked "prescription required = yes" for medicines
✓  Responded to one complaint within 2 hours
✓  That's it.
```

---

*This document is the non-technical companion to the **Bu Sari Complete Journey** document, which contains the full API calls, JSON payloads, and field-level technical detail for every step described here.*

*Every layer of the ION entity hierarchy model is engaged in this journey — Organisation, Role, Profile, Sector, Segment, CRC, Resource, Pattern, Variant, Grievance/IGM, Participant Extensions, Reconciliation, Experimental Extensions, Test Set, and Test Cases. Bu Sari encountered all of them. She just never needed to know their names.*
