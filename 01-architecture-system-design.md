# TheBrokrs — Complete System Architecture & Product Design

> **Vision:** A unified Indian super-app that brokers trust between users and high-value services (Investment, Hybrid Farming, IT, Insurance, Hospitals), and turns every rupee spent into a points/referral flywheel that loops users back through partner brands (Bikanerwala, Cafe Coffee Day, etc.).

---

## 1. Product Pillars

| Pillar | What it does | Revenue model | Phase |
|---|---|---|---|
| **Investment** | Mutual funds, FDs, digital gold, P2P lending | Commission / AUM fees | P1 |
| **Hybrid Farming** | Invest in managed farm projects (crop + livestock + agroforestry) | % of yield + capital Mgmt fee | P1 |
| **IT Services** | SaaS subscription, custom dev, cloud mgmt | Project / retainer | P1 |
| **Insurance** | Health, life, vehicle, crop — comparison & claim assist | Brokerage from insurer | P1 |
| **Hospitals** | OPD booking, video consult, insurance cashless, second opinion | Commission per lead / booking | P1 |
| **Partners** | Brand discounts (Bikanerwala, CCD, Zomato, etc.) | Affiliate / co-marketing | P1 |
| **Referrals & Points** | Earning/spending engine spanning all pillars | — | P1 |
| **Wallet / Payments** | UPI, cards, netbanking, BNPL | Interchange + float | P2 |
| **Marketplace (pro sellers)** | List products, escrow, dispute | Commission | P2 |
| **Loans & Credit** | Personal, business, LAP | Origination fee | P3 |

---

## 2. High-Level Architecture (Cloud-native, India-first)

```
┌─────────────────────────────────────────────────────────────────────┐
│                       CLIENTS (Multi-channel)                        │
│  Android (Kotlin) · iOS (Swift) · Web (Next.js) · Partner Portal    │
│           │                                                          │
│           │   (TLS 1.3, JWT + Refresh, Cert Pinning)                │
└───────────┼──────────────────────────────────────────────────────────┘
            ▼
┌─────────────────────────────────────────────────────────────────────┐
│              EDGE  ──  CloudFront / Cloudflare                       │
│   • WAF + Bot Mgmt  • DDoS shield  • Rate-limit per IP/device      │
└────────────────────────────────────┬────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  API GATEWAY  (Kong / AWS API GW)                    │
│   • AuthN (JWT, OAuth2)  • Quota  • Routing  • Request shaping      │
│   • GraphQL Federation (Apollo)  • REST fallback (legacy partners)  │
└────────────────────────────────────┬────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  BFF  (Backend-for-Frontend)                         │
│           Node.js (NestJS) — aggregates per-client view              │
└────────────────────────────────────┬────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                MICROSERVICES  (Kubernetes / EKS Mumbai)              │
│                                                                      │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐ │
│  │ Auth & KYC   │ │ User Profile │ │ Referrals &  │ │  Partner   │ │
│  │ (Phone/Aadhaar)│ │ & Identity   │ │   Points     │ │  Catalog   │ │
│  └──────────────┘ └──────────────┘ └──────────────┘ └────────────┘ │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐ │
│  │  Investment  │ │   Hybrid     │ │  IT Services │ │ Insurance  │ │
│  │   Service    │ │   Farming    │ │  Marketplace │ │   Broker   │ │
│  └──────────────┘ └──────────────┘ └──────────────┘ └────────────┘ │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐ │
│  │   Hospital   │ │   Payment    │ │ Notification │ │  Search &  │ │
│  │   Network    │ │  & Wallet    │ │  (Push/SMS)  │ │  Discovery │ │
│  └──────────────┘ └──────────────┘ └──────────────┘ └────────────┘ │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                │
│  │   KYC &      │ │  Analytics   │ │  Admin / CMS │                │
│  │ Compliance   │ │  (ClickHouse)│ │   (Internal) │                │
│  └──────────────┘ └──────────────┘ └──────────────┘                │
└────────────────────────────────────┬────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        DATA PLANE                                    │
│                                                                       │
│  PostgreSQL  (Primary OLTP, per-service schemas)                     │
│  MongoDB     (catalogs, CMS, partner offers, CMS content)             │
│  Redis       (sessions, hot cache, leaderboards, rate-limits)        │
│  ClickHouse  (analytics, BI, fraud signals)                          │
│  Elasticsearch (search, geo, partner discovery)                     │
│  S3 / MinIO   (KYC docs, claim files, farm photos, media)             │
│  Kafka       (event bus — CDC, referrals, points ledger)             │
└─────────────────────────────────────────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                EXTERNAL  (India-compliant)                            │
│  • Setu / Digio / Signzy — Aadhaar eKYC, eSign, Video KYC           │
│  • CIBIL/Experian — Credit                                       │
│  • Karvy / CAMS / KFintech — MF order routing                      │
│  • BSE Star / NSE NMF II — MF platform                              │
│  • Razorpay / Cashfree / PhonePe — Payments                          │
│  • IRDAI-licensed Insurer APIs                                      │
│  • NABARD / APEDA — Hybrid farming compliance                        │
│  • FHIR/HL7 hospitals — Appointment & EMR (with consent)             │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.1 Why this shape?

- **Multi-service**, so each pillar evolves independently. A new insurance partner ships in days, not quarters.
- **Event-driven core (Kafka)** so points/referrals get appended to a **single immutable ledger** — no double-credit, perfect auditability, regulator-friendly.
- **BFF per client** lets mobile, web, and partner portal each optimize payload, latency, and bundling.
- **India-region** primary (Mumbai `ap-south-1`); DR in Hyderabad; CDN edge in Delhi/Bangalore/Chennai for low-p95 mobile latency.
- **Tier-2/3 users** are real. We must support **low-bandwidth mode** (compress images, lazy-load, offline-first for last-viewed screens, lite web build).

---

## 3. Authentication & Onboarding (KYC) Flow

```
Phone # entry → OTP (Firebase / MSG91)
       │
       ▼
   Email + DOB (optional) → PAN entry → Aadhaar eKYC (OTP or biometric)
       │
       ▼
   Video KYC fallback (for high-value actions)
       │
       ▼
   Profile created → Wallet created → Welcome bonus (50 pts)
```

| Step | What we collect | KYC level unlocked | Limits |
|---|---|---|---|
| Phone OTP | phone, device | L0 | Browse only |
| + Email | email, basic profile | L1 | Points earn, browse |
| + PAN | PAN, name, DOB | L2 | Invest up ₹50k/mo, partner coupons |
| + Aadhaar (OTP) | address | L3 | Full investment, hospital cashless, insurance buy |
| + Video KYC | face match | L4 | No limits, claim disbursement, partner payouts |

Aadhaar/PAN data **never** touches our DB — we get a tokenized reference from Setu/Signzy and store only the masked ID. Compliant with DPDP Act 2023.

---

## 4. The Heart of the App — Referrals & Points System

### 4.1 Earning Engine

| Source | Points rule | Cap / Cooldown |
|---|---|---|
| First signup (referrer) | 200 pts | one-time per referee |
| First signup (referee)  | 100 pts | one-time |
| First investment ≥ ₹1,000 | 500 pts | one-time |
| Every ₹100 spent on any service | 5 pts | 1,000 pts/day |
| Daily check-in (streak) | 5–50 pts (rising) | once/day, breaks at 7 |
| Hospital booking via app | 300 pts | 3/month |
| Insurance policy bought | 1,000 pts | one per policy/yr |
| Farm investment (P1) | 2 pts per ₹100 (×2 multiplier = 4) | 5,000 pts/yr |
| Partner spend (Bikanerwala etc.) | 1 pt per ₹20 | 500 pts/brand/day |
| Review left | 20 pts | 1/day |
| Profile 100% complete | 250 pts | one-time |
| Milestone (1, 5, 10, 50, 100 referrals) | 1k / 5k / 25k / 1L / 5L pts + badge | one-time |

### 4.2 Tier System (Status)

| Tier | Lifetime points | Perks |
|---|---|---|
| Bronze | 0 | Base earn rate |
| Silver | 2,500 | +5% earn boost, free delivery on partner coupons |
| Gold | 10,000 | +10% boost, premium partner access, priority support |
| Platinum | 50,000 | +20% boost, concierge, exclusive farm/insurer deals |
| Diamond | 2,00,000 | +35% boost, invite to private deals, brand ambassador path |

Tiers also map to **partner-brand multiplier days** — e.g. Gold users get 2× points at Bikanerwala every Wednesday.

### 4.3 Spending Engine

| Redemption | Cost (approx) | Notes |
|---|---|---|
| ₹50 partner coupon (Bikanerwala, CCD, etc.) | 500 pts | capped 2/month brand |
| Free video consult (hospital) | 800 pts | 1/month |
| ₹200 insurance premium waiver | 2,000 pts | tier-restricted |
| Farm investment top-up (₹500) | 5,000 pts | capped |
| Skip-the-queue hospital token | 200 pts |  |
| Donate to charity partner (GiveIndia) | any |  |
| Convert to wallet cash | 100 pts = ₹1 | min 1,000 pts, 5x/yr |

### 4.4 Referral Mechanics (3 Loops)

```
┌────────────┐  share link  ┌────────────┐
│  Referrer  │─────────────▶│  Referee   │
│  (User A)  │              │  (User B)  │
└─────┬──────┘              └─────┬──────┘
      │                           │
      │                           ▼
      │              B signs up + completes KYC
      │                           │
      │                           ▼
      │              B makes first qualifying action
      │              (investment, insurance, partner spend)
      │                           │
      │                           ▼
      └────────  A gets reward, B gets welcome bonus
                  + unlock "Power Referrer" badge
```

- **Direct refer**: 1:1, both get bonus on first qualifying action.
- **Tiered (2-level)**: A → B → C. A gets 10% of B's first-action reward, B gets 100%.
- **Team / MLM-lite** (opt-in, capped): Up to 3 levels, no monetary payout, only points + badge. **Strictly non-Ponzi**: payouts come from real service margin, not new-user money. Documented in T&C, reviewed by legal before launch.
- **Partner-referral**: Hospitals/insurers can offer "refer a friend, both get ₹X off" co-funded campaigns.

### 4.5 Anti-Fraud Guardrails

- One device + one phone = one account (with manual override for legitimate multi-user).
- Referral payouts **vest** for 14 days (refundable if referee chargebacks).
- Partner spend reward requires **verified bill** (geo-fence at outlet + OCR of bill amount).
- Velocity rules: max 10 referrals/day, max 5,000 pts earning/day for first 90 days.
- Graph analysis: ring detection (A→B→A), device-fingerprint clustering.

### 4.6 Points Ledger (immutable, append-only)

Every credit/debit is a Kafka event → ClickHouse + Postgres projection. Schema (Postgres):

```sql
CREATE TABLE points_ledger (
  id            BIGSERIAL PRIMARY KEY,
  user_id       UUID NOT NULL,
  delta         INT NOT NULL,        -- +/-
  balance_after INT NOT NULL,
  reason        TEXT NOT NULL,       -- 'REFERRAL_SIGNUP','PARTNER_SPEND',...
  ref_id        UUID,                -- FK to source txn/referral/offer
  expires_at    TIMESTAMPTZ,         -- default +12 months
  created_at    TIMESTAMPTZ DEFAULT now()
);
CREATE INDEX ON points_ledger (user_id, created_at DESC);
```

- Idempotent by `(user_id, reason, ref_id)` — duplicate events cannot double-credit.
- Expiry sweep nightly; user gets push 7 days before points lapse.

---

## 5. Per-Service Flow (Phase 1)

### 5.1 Investment

```
Browse → Risk profile quiz → Recommendations
       → MF/FD/Gold/Stock basket
       → KYC check (auto if L2+)
       → One-time mandate (eNACH / eMandate)
       → Order → RTA confirmation → portfolio
       → SIP / Lumpsum / Switch / Redeem
       → Tax harvesting nudge (March)
       → Statements (CAS-ready)
```

- **Aggregator** approach — BSE Star, NSE NMF II, Kuvera, or our own MFU tie-up.
- **AI suggestion**: "Based on your farm investment, consider balanced-advantage fund" — only on opt-in, never crosses silos without consent.
- **Tax P/L** view; **capital gains** statement export (PDF + Excel).

### 5.2 Hybrid Farming

```
Browse Projects (crop/livestock/agroforestry)
  → Project detail (location, risk grade, expected IRR, photos, SPV name)
  → Choose plot/units (min ₹5,000)
  → KYC + eSign agreement
  → Pay (UPI/Card/Netbanking)
  → Monthly farm updates (photos, weather, agronomist notes)
  → Harvest → yield credited to wallet
  → Re-invest or withdraw
```

- Each project is a **SPV** (Special Purpose Vehicle) — we broker, not own. Investors get co-ownership certificates, eSigned.
- Insurance bundled at project level (crop/weather). Insurer partner (Bajaj Allianz / ICICI Lombard) shown on detail page.
- **Trust signals**: NABARD empanelment, photo logs, satellite imagery quarterly.
- **Geo page**: "Farms near you" → drives tier-2/3 trust.

### 5.3 IT Services

```
Catalog (Website pkg ₹9,999 → App pkg ₹49,999 → Custom quote)
  → Requirement form (5-step wizard)
  → AI pre-scope (estimate) → human sales call
  → Milestone-based escrow (we hold money, release on acceptance)
  → Project dashboard (chat, files, milestones, reviews)
  → Post-launch AMC plans
```

- Service providers (vendors) are **verified** with GST, prior work, references.
- Two-sided ratings. Disputes go to mediation team (24h SLA).

### 5.4 Insurance

```
Compare plans (5 insurers at once)
  → Filter by sum insured, premium, claim ratio
  → Buy in 3 taps (KYC pre-filled)
  → Policy PDF + claim procedure in-app
  → Renew in 1 tap
  → Claim: photo + FIR (motor) / hospital pre-auth (health)
  → Status tracker → settlement in wallet
```

- **IRDAI Corporate Agent** license (or Composite Broker) — must be in place before launch. Compliance:
  - All recommendations come with **disclosure** of commission.
  - No "free" promises. Clear T&C.
  - Grievance officer named on screen.

### 5.5 Hospitals

```
Find by city / specialty / insurance network
  → OPD booking (3 taps)
  → Video consult (in-app, ₹X or free with points)
  → Cashless pre-auth (we push to insurer TPA)
  → Health locker: prescriptions, reports, vitals (FHIR)
  → Family profiles (4 members)
  → Emergency: nearest hospital, one-tap call, share live location
```

- We start as **info + booking** (no PHI stored beyond user's consent). Phase 2 we add **ABDM** (Ayushman Bharat Digital Mission) integration for ABHA-linked records.

### 5.6 Partners (the "earn-while-you-spend" loop)

```
Partners tab → Categories (Food, Travel, Health, Recharge, Fashion)
  → Brand list (Bikanerwala, CCD, Dominos, MakeMyTrip, Tata 1mg...)
  → Offer details (terms, expiry, geo)
  → "Activate offer" → QR / coupon code shown
  → Pay at outlet → bill OCR or geo + auto-verify within 24h
  → Points credited to wallet
```

- Partner revenue share: 3–8% on every coupon redeemed. We share 1–2% with user as points.
- **Geo-aware** home screen: "Bikanerwala 0.4 km — flat 20% off, earn 80 pts on ₹200+".
- Brand pages get a **mini-storefront** (catalog, ratings, ongoing offers).

---

## 6. Database Schema (Core Tables, abridged)

```sql
-- Users
users (id uuid pk, phone unique, email, pan_masked, aadhaar_ref, kyc_level,
       tier, lifetime_points, ref_code unique, referred_by uuid, created_at)

-- Wallets
wallets (user_id pk, balance_paise bigint, locked_paise bigint, updated_at)

-- Referrals
referrals (id, referrer_id, referee_id, status, level int, reward_paise, vested_at)

-- Points ledger (above)

-- Partners
partners (id, name, slug, logo_url, category, status)
partner_offers (id, partner_id, title, terms, points_reward, valid_till, geo)
partner_redemptions (id, user_id, offer_id, bill_amount, verified, credited_pts)

-- Services
investments (id, user_id, scheme_code, folio, units, avg_nav, type)   -- 'SIP'/'LUMP'
farm_projects (id, name, spv, location, irr_min, irr_max, tenure, risk_grade, media[])
farm_investments (id, user_id, project_id, units, amount, status)
insurance_policies (id, user_id, insurer, plan, premium, sum_insured, doc_url, valid_till)
hospital_appointments (id, user_id, hospital_id, doctor_id, slot, mode, status)
it_projects (id, user_id, vendor_id, scope, escrow_amount, status, milestones jsonb)

-- Transactions (single source of truth for money)
transactions (id, user_id, type, amount_paise, status, gateway, gateway_ref,
              service_ref, created_at)
```

---

## 7. API & Event Contracts (sample)

```
REST (per service):
  GET  /v1/services/investments/portfolio
  POST /v1/services/farms/projects/{id}/invest
  GET  /v1/partners?lat=28.7&lng=77.1&cat=food
  POST /v1/referrals/code  → returns ref_code + share assets

GraphQL (BFF, mobile/web):
  query Me { user, wallet, points, tier, refStats, topPartners }

Async events (Kafka topic: brokrs.events.v1):
  user.signed_up         { user_id, ref_code, channel }
  investment.created     { user_id, amount, scheme }
  farm.invested          { user_id, project_id, units }
  partner.offer_redeemed { user_id, partner_id, bill, points }
  points.credited        { user_id, delta, reason, ref }
  points.debited         { user_id, delta, reason, ref }
  insurance.policy_bound { user_id, policy_id }
  hospital.appointment   { user_id, doctor_id, slot }
```

Schema registry (Protobuf/Avro) for all events. Consumers (points-calculator, fraud, analytics, push) all subscribe off this bus.

---

## 8. Observability, Reliability, Security

| Concern | Tool / approach |
|---|---|
| Tracing | OpenTelemetry → Jaeger / Tempo |
| Metrics | Prometheus + Grafana (GOLDEN signals per service) |
| Logs | Loki / ELK, sampled 100% errors, 10% info |
| SLOs | 99.9% availability per service; p95 mobile API < 400 ms |
| Secrets | HashiCorp Vault; short-lived service tokens |
| PII | Tokenized at edge; KMS-encrypted at rest; Aadhaar never persisted |
| Backups | Postgres WAL shipped to S3; PITR 35 days |
| DR | Multi-AZ, cross-region async replica; RPO 5 min, RTO 30 min |
| Pentest | Annual + per major release; bug-bounty from day 1 |
| DPDP / RBI / IRDAI | DPO named in-app; consent management; data export + delete |

---

## 9. Scalability for the Indian Market

- **Cost-aware** infra: spot for analytics, reserved for OLTP; aggressive caching of partner catalog (Redis + 5-min CDN).
- **Multi-lingual** from day one: Hindi, Tamil, Telugu, Marathi, Bengali, Gujarati. i18n keys for every string, no hard-coded English.
- **Voice-first onboarding** for low-literacy users — record voice note, we transcribe & create ticket.
- **Lite mode**: app < 25 MB; web < 200 KB first paint; works on 2G.
- **Offline**: last-seen offers, points balance, saved prescriptions cached.
- **UPI deep-link** first, card fallback (cards fail ~12% of the time in India).
- **T+0 settlement** illusion: wallet is funded instantly; partner settlement T+1.
- **Network engineering**: WebSocket for live farm updates (vs polling), image lazy-load with WebP/AVIF, aggressive prefetch of bottom-tab screens.
- **Regional data residency**: primary in `ap-south-1`; sensitive workloads in Mumbai-only subnet.

---

## 10. Growth & Go-To-Market (Phase 1 → 100k users)

1. **Cohort 1 — own farming circle** (operators + their families, ~5k).
2. **Cohort 2 — Hyderabad/Bangalore tier-1 SaaS crowd** (IT services lead magnet).
3. **Cohort 3 — partner-led acquisition** (Bikanerwala table-top QR, "scan to get 100 pts on first bill ≥ ₹300").
4. **Referral flywheel** becomes primary CAC channel by month 6.

CAC target: < ₹120 blended. LTV: > ₹900 within 12 months. Points cost as % of GMV: < 6%.

---

## 11. Phased Roadmap

| Phase | Services | Geography | Headline metric |
|---|---|---|---|
| **P1 (0–9 mo)** | Investment, Hybrid Farming, IT, Insurance, Hospitals, Partners, Referrals | Hyderabad + Bangalore | 50k MAU, ₹25 Cr GMV |
| **P2 (9–18 mo)** | Wallet, UPI, P2P transfers, Marketplace, ABHA health locker | Top 6 metros | 5L MAU |
| **P3 (18–36 mo)** | Loans, Credit, Mutual-fund advisor, Wealth Pro, Bharat-1 lite (feature phone) | Pan-India tier-2/3 | 50L MAU |

---

## 12. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Regulator scrutiny (referrals could look like MLM) | Legal sign-off; non-monetary rewards; cap team-lvl at 3; T&C clear |
| Aadhaar misuse | Tokenize at vendor; never store; consent receipt every time |
| Insurance commission conflict | IRDAI disclosure on every recommendation; no "advisory" claim |
| Partner fraud (fake bills) | Geo-fence + bill OCR + random merchant audit; claw-back policy |
| KYC fatigue | One KYC, reuse across all services in-app |
| Tier-3 infra limits | Lite mode + voice onboarding + offline cache |
| Points inflation | Annual 10–15% decay; tiered expiry; basket-of-redemptions policy |
