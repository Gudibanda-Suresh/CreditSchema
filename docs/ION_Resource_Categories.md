# ION Resource Categories

**Version:** 0.1-draft  
**Status:** For review  
**Scope:** Resource Category taxonomy for ION across all sectors.

---

## How to read this

| Column | Meaning |
|---|---|
| **ION Name** | Human-readable name used in seller onboarding and BAP display |
| **Short Code** | Machine-readable code used on the wire in `resourceAttributes.ionResourceCategory` |
| **GPC L0 Mapping** | GPC L0 crosswalk — for seller onboarding only, never transmitted on wire |
| **Status** | ACTIVE = schema defined, validation enforced · INACTIVE = recognised, no enforcement yet · GOVERNED = blocked pending ION Council policy decision |
| **Description** | What this category covers — the test a seller uses to self-classify |

---

## Trade Sector (`ion:trade`)

> **Principle: buyer ends up owning a thing**

| ION Name | Short Code | GPC L0 Mapping | Status | Description |
|---|---|---|---|---|
| Fashion & Accessories | `TRC-fashion` | Apparel & Accessories | ACTIVE | Clothing, footwear, bags, jewellery, watches, eyewear, and all wearable accessories. Includes menswear, womenswear, childrenswear, traditional dress (batik, kebaya), sportswear, and fashion accessories. |
| Electronics & Gadgets | `TRC-electronics` | Electronics · Cameras & Optics | ACTIVE | Consumer electronics, mobile phones, laptops, tablets, cameras, audio equipment, gaming devices, wearables, smart home devices, and electronic accessories. Cameras & Optics sit as a sub-category here — not a separate top-level — reflecting how Indonesian sellers actually classify on Tokopedia and Shopee. |
| Food, Beverages & Tobacco | `TRC-food-bev` | Food, Beverages & Tobacco | ACTIVE | Packaged food, beverages (non-alcoholic and alcoholic), snacks, condiments, coffee, tea, cooking ingredients, tobacco products. Fresh produce is included but carries additional freshness attributes. Tobacco and alcohol require specific regulatory declarations. **Does not cover restaurant meals or food delivery — those are Hospitality (`HSC-restaurant-ordering`).** |
| Health & Beauty | `TRC-health-beauty` | Health & Beauty | ACTIVE | Skincare, haircare, cosmetics, personal care, OTC vitamins and supplements, consumer-grade medical devices, wellness products, hygiene items. Prescription medicines sit under Pharmacy & Health Retail (TRD-04) and carry stricter attribute requirements. |
| Home & Garden | `TRC-home-garden` | Home & Garden | ACTIVE | Household appliances, kitchen equipment, home décor, lighting, garden tools and supplies, cleaning products, storage and organisation, bedding and bath. |
| Furniture | `TRC-furniture` | Furniture | ACTIVE | Indoor and outdoor furniture — sofas, beds, tables, chairs, wardrobes, shelving — flat-pack and assembled. Kept separate from Home & Garden because of different logistics (oversized/freight), different attributes (dimensions, material, assembly required), and different regulatory treatment (SNI for certain items). |
| Vehicles & Parts | `TRC-automotive` | Vehicles & Parts | ACTIVE | Motorcycles, cars, automotive parts and accessories, tyres, lubricants, helmets, and vehicle care products. Includes new, used, and aftermarket parts. |
| Sporting Goods | `TRC-sports` | Sporting Goods | ACTIVE | Sports equipment, fitness gear, outdoor and adventure equipment, bicycles, water sports, team sports, and sportswear accessories. |
| Toys, Games & Baby | `TRC-toys-baby` | Toys & Games · Baby & Toddler | ACTIVE | Toys, board games, puzzles, educational materials for children, baby clothing, feeding, nursery items, strollers, and baby care products. Merged because seller profiles and buyer journeys significantly overlap in the Indonesian market. Can be split by SWG if regulatory treatment diverges. |
| Books & Media | `TRC-media` | Media | ACTIVE | Physical books, magazines, music CDs, DVDs, Blu-rays, and other physical media. Digital versions are under Digital Goods. |
| Digital Goods | `TRC-digital` | Software | ACTIVE | Software licenses, mobile top-up (pulsa), data packages, PLN token, game credits, digital vouchers, gift cards, e-books, digital subscriptions — all delivered electronically with no physical logistics. |
| Business & Industrial | `TRC-b2b` | Business & Industrial | ACTIVE | Raw materials, industrial equipment, office machinery, packaging materials, safety equipment, bulk commodities, B2B wholesale goods. Primarily serves TRD-06 (B2B Trade & Wholesale) and TRD-07 (Energy & Commodity) business categories. |
| Arts, Crafts & Collectibles | `TRC-arts` | Arts & Entertainment | ACTIVE | Art supplies, craft materials, handmade products, collectibles, antiques, musical instruments, hobby supplies, kerajinan tangan (Indonesian handicrafts). |
| Religious & Ceremonial | `TRC-religious` | Religious & Ceremonial | ACTIVE | Prayer equipment (sajadah, tasbih, mukena), religious texts, ceremonial items, traditional ceremony supplies, items for Lebaran, Natal, Imlek, and other religious observances. Significant category given Indonesia's diverse religious landscape. |
| Luggage & Bags | `TRC-luggage` | Luggage & Bags | ACTIVE | Suitcases, travel bags, backpacks, laptop bags, school bags, and travel accessories. Kept separate from Fashion & Accessories because of different attributes (volume, wheel type, lock type) and different buyer intent (travel utility vs fashion). |
| Animals & Pet Supplies | `TRC-pets` | Animals & Pet Supplies | INACTIVE | Pet food, accessories, grooming, and care products. Recognised but low volume at launch — no attribute schema enforcement yet. Sellers may publish but validation is not applied. |
| Hardware & Tools | `TRC-hardware` | Hardware | INACTIVE | Hand tools, power tools, building materials, plumbing and electrical supplies. Low volume as a distinct top-level at launch — sellers may use Business & Industrial in the interim. |
| Office Supplies | `TRC-office` | Office Supplies | INACTIVE | Stationery, office equipment, paper products, writing instruments. Low volume at launch — sellers may use Business & Industrial. |
| Mature | `TRC-mature` | Mature | GOVERNED | Adult content and products. Blocked pending ION Council policy decision on whether and how adult commerce is permitted, age verification requirements, and BAP rendering obligations. |

---

## Hospitality Sector (`ion:hospitality`)

> **Principle: time-bounded reservation of capacity**

GPC has no meaningful L0 for Hospitality. ION defines independently.

| ION Name | Short Code | GPC L0 Mapping | Status | Description |
|---|---|---|---|---|
| Accommodation | `HSC-accommodation` | — | ACTIVE | Hotel rooms, villas, homestays, guesthouses, serviced apartments, glamping. Resource is a bookable unit with check-in/check-out datetime. |
| Restaurant & Dining | `HSC-restaurant` | — | ACTIVE | Table reservations at restaurants and dining venues. Resource is a time-slot at a specific table or area. |
| Restaurant & Food Ordering | `HSC-restaurant-ordering` | — | ACTIVE | Restaurant meals and beverages ordered for delivery or self-pickup — GoFood, GrabFood, Tokopedia Food, and any ION-connected food delivery platform. Resource is a prepared food item made after order placement. Covers online food ordering end-to-end. **Not for packaged food products — those are Trade (`TRC-food-bev`).** |
| Events & Experiences | `HSC-events` | — | ACTIVE | Tours, activities, attractions, entertainment, wellness experiences, MICE bookings. Resource is a participation slot with defined start time and participant count. |
| Recreation & Wellness | `HSC-wellness` | — | ACTIVE | Gym, spa, salon, fitness classes, pool access, wellness services. Resource is a time-bounded slot or session. |

---

## Logistics Sector (`ion:logistics`)

> **Principle: movement and storage of goods**

GPC has no meaningful L0 for Logistics services. ION defines independently.

| ION Name | Short Code | GPC L0 Mapping | Status | Description |
|---|---|---|---|---|
| Last-Mile Delivery | `LGC-lastmile` | — | ACTIVE | Parcel and hyperlocal delivery from sender to recipient. Resource is a shipment slot covering courier, same-day, next-day, and on-demand delivery. |
| Freight Transport | `LGC-freight` | — | ACTIVE | Trucking, rail, and air freight for larger cargo. Resource is a freight capacity booking. |
| Sea & Air Freight | `LGC-international` | — | ACTIVE | International and domestic sea/air freight including Ro-Ro vessel movement. Resource is a cargo space booking with route and schedule. |
| Warehousing | `LGC-warehouse` | — | ACTIVE | Storage and fulfilment centre services. Resource is a capacity commitment covering inbound, storage, and outbound operations. |
| Freight Forwarding | `LGC-forwarding` | — | ACTIVE | Customs clearance, freight forwarding, and logistics support. Resource is a forwarding service engagement. |

---

## Mobility Sector (`ion:mobility`)

> **Principle: movement of people**

| ION Name | Short Code | GPC L0 Mapping | Status | Description |
|---|---|---|---|---|
| Ride-Hailing | `MOC-ride` | — | ACTIVE | On-demand passenger transport — motorcycles, cars, and other vehicles. Resource is a journey booking. |
| Scheduled Transport | `MOC-scheduled` | — | ACTIVE | Fixed-route scheduled services — buses, trains, ferries, flights. Resource is a seat on a specific departure. |
| Vehicle Rental | `MOC-rental` | — | ACTIVE | Short or long-term vehicle rental. Resource is a vehicle for a defined period. |
| Charter | `MOC-charter` | — | INACTIVE | Exclusive vehicle booking for a defined route or duration. Reserved for future activation. |

---

## Finance Sector (`ion:finance`)

> **Principle: money and financial instruments**

| ION Name | Short Code | GPC L0 Mapping | Status | Description |
|---|---|---|---|---|
| Lending & Credit | `FNC-lending` | — | ACTIVE | Personal loans, business loans, BNPL, P2P lending, credit facilities. Resource is a credit product with defined tenure and rate. |
| Insurance | `FNC-insurance` | — | ACTIVE | Life, health, motor, property, and travel insurance. Resource is a policy with defined coverage and premium. |
| Payments & E-money | `FNC-payments` | — | ACTIVE | Wallet top-up, bill payment, fund transfer, payment gateway services. Resource is a payment transaction. |
| Investment | `FNC-investment` | — | ACTIVE | Mutual funds, stocks, bonds, digital assets. Resource is an investable instrument. |
| Banking Services | `FNC-banking` | — | INACTIVE | Account opening and banking product activation. Reserved for future activation. |

---

## Services Sector (`ion:services`)

> **Principle: human labour and expertise**

| ION Name | Short Code | GPC L0 Mapping | Status | Description |
|---|---|---|---|---|
| Technology & Digital Services | `SVC-tech` | — | ACTIVE | Software development, IT services, SaaS, cloud, digital marketing, data services. |
| Professional & Advisory | `SVC-professional` | — | ACTIVE | Legal, accounting, consulting, financial advisory, HR services. Resource is a professional engagement. |
| Healthcare & Wellness Services | `SVC-healthcare` | — | ACTIVE | Doctor consultations, physiotherapy, diagnostics, telemedicine, mental health. Resource is a professional service slot. Healthcare products (medicines, devices) are under Trade not Services. |
| Telecom & Connectivity | `SVC-telecom` | — | ACTIVE | Internet plans, mobile plans, enterprise connectivity. Resource is a service subscription. |
| Creative & Marketing | `SVC-creative` | — | ACTIVE | Design, content production, advertising, media buying, events production. |
| Home & Facility Services | `SVC-home` | — | ACTIVE | Cleaning, repair, plumbing, electrical, pest control, home installation. Resource is a service appointment. |
| Education & Training | `SVC-education` | — | ACTIVE | Tutoring, professional courses, corporate training, e-learning. Resource is a course enrolment or session booking. |
| Automotive Services | `SVC-auto` | — | INACTIVE | Vehicle servicing, repair, and inspection. Reserved for future activation. |

---

## Design Notes

**GPC mapping applies to Trade only.**
GPC L0 is an onboarding crosswalk — never transmitted on the wire. A seller with a GPC code maps to the ION short code at onboarding. Crosswalk table maintained at `registry.ion.id/categories/crosswalk/trade`. Non-Trade sectors have no GPC equivalent — ION defines those independently.

**Why some GPC L0 categories are merged.**
Electronics + Cameras & Optics are merged because Indonesian sellers on Tokopedia and Shopee do not separate them. Toys & Games + Baby & Toddler are merged because seller profiles overlap significantly. Where merging would lose attribute enforcement (Furniture kept separate from Home & Garden) — categories are kept distinct.

**INACTIVE does not mean blocked.**
Sellers can publish under INACTIVE categories. The code is recognised and transmitted. No attribute schema validation is applied at `catalog/publish`. Enforcement will be activated when the SWG defines the schema.

**GOVERNED means blocked.**
Resources cannot be published under GOVERNED categories. Attempts return ION-2005. Unblocked only by ION Council policy decision.

**Cross-sector classification rule.**
When an item could belong to multiple sectors, apply: *what does the buyer end up with?*
- Physical medicine → Trade (`TRC-health-beauty`)
- Doctor consultation → Services (`SVC-healthcare`)
- Hotel room → Hospitality (`HSC-accommodation`)
- Ride → Mobility (`MOC-ride`)

**Short codes are ION-owned.**
GPC vocabulary inspired the naming where applicable. ION governance controls the codes — not Google. If GPC updates, the crosswalk table updates. The ION short codes do not change.

---

*Working draft — for ION Council and SWG review.*
*Total: 19 Trade + 5 Hospitality + 5 Logistics + 4 Mobility + 5 Finance + 8 Services = 46 Resource Categories across 6 sectors.*
