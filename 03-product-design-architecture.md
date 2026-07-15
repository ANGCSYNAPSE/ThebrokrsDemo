# TheBrokrs — Product Design Architecture & Operating Flow

> **Purpose:** the single product reference for what users, partners, support agents, finance operators, compliance reviewers, catalog managers, and super-admins can do—and how services, credits, rewards, referrals, payments, and exceptions connect.

## 1. Product definition

TheBrokrs is a **trusted-services marketplace and lifecycle manager**, not a single checkout super-app. It helps a user discover, evaluate, buy or request, track, and get support for:

1. Insurance
2. Financial investments
3. Land investment
4. Property purchase, rental, and related services
5. Hybrid and specialty farming projects
6. ClickCard membership / benefit card
7. IT solutions
8. Healthcare and future approved service categories
9. Partner offers and benefit redemption

The common promise is: **discover with clarity → verify only when required → transact safely → track the outcome → earn useful value → return for management and support.**

## 2. What was wrong in the previous flow

| Problem | Why it breaks | Correction |
|---|---|---|
| Every service used one `Detail → eSign → Pay` path | Insurance, land, property, IT, healthcare, and offers do not complete in the same way | Introduce a reusable lifecycle with category-specific fulfillment branches |
| “Points,” “credits,” wallet money, discounts, and investment value were visually mixed | Users cannot know what is cash, what expires, and what can be withdrawn | Separate Money, Service Credit, Reward Points, Discount, and Asset Value |
| Rewards appeared immediately after payment | Payments can be late-authorized, refunded, cancelled, disputed, or reversed | Use `pending → vested → redeemed/expired/reversed` reward states |
| Referral ended at “friend joins” | It did not define attribution, qualification, vesting, fraud hold, cancellation, or reversal | Add a referral state machine tied to a qualifying service event |
| KYC was treated as one universal level | Each regulated service has different identity, suitability, document, and consent requirements | Use a capability-based verification matrix per action |
| Land, property, and ClickCard were absent | Core business scope was not represented | Add explicit domain flows and admin catalog ownership |
| No admin flow | Catalogs, leads, approvals, reconciliation, disputes, rewards, and risk had no operator | Add role-based admin control plane with maker-checker approval |
| No partner lifecycle | Offers and service providers appeared magically active | Add partner onboarding, verification, contract, catalog, fulfillment, settlement, and suspension |
| Success was a single final state | Real transactions are async and often pending | Use an order timeline with submitted, processing, completed, failed, cancelled, refunded, disputed |
| Home was rewards-first for every user | It can hide the main service intent | Home is intent-first; rewards are a persistent supporting value layer |

## 3. Roles and surfaces

### Customer app

- Guest: browse public categories and educational content.
- Registered user: save, compare, enquire, refer, and manage profile.
- Verified customer: perform actions unlocked by required checks.
- Family member: access explicitly shared health, insurance, or documents.

### Partner portal

- Service partner: manage leads, quotes, documents, fulfillment, and SLAs.
- Merchant partner: manage outlets, offers, QR/code validation, and claims.
- Project operator: publish verified farm/land/project milestones and evidence.
- Partner finance: view commissions, invoices, deductions, and settlements.

### Admin console

- Catalog manager: categories, products, content, offers, and merchandising.
- Operations agent: orders, leads, bookings, escalations, and manual tasks.
- KYC/compliance reviewer: verification queues, suitability, consent, and holds.
- Rewards operator: earning rules, campaigns, liability, expiry, and reversals.
- Finance operator: payments, refunds, commissions, settlement, reconciliation.
- Support/grievance agent: conversations, tickets, SLA, escalation, resolution.
- Risk analyst: referral abuse, account/device/payment/merchant anomalies.
- Super admin: roles, policy, integrations, configuration; no routine case handling.
- Auditor: read-only access to immutable history and exported evidence.

## 4. Customer information architecture

The five stable bottom destinations are:

1. **Home** — resume tasks, active services, next actions, personalized discovery.
2. **Explore** — categories, search, compare, saved items, recently viewed.
3. **Activity** — enquiries, applications, orders, bookings, projects, claims, timelines.
4. **Rewards** — points, credits, referrals, vouchers, tiers, expiry.
5. **Profile** — KYC, family, documents, payment methods, security, consent, support.

Offers are part of **Rewards** and contextually appear in Home/Explore. This is more scalable than spending one permanent navigation slot on offers while hiding all active service activity.

### Global objects

- Search
- Notification inbox
- Help
- Saved/compare
- Global status banner for unresolved payment/KYC/support actions

## 5. Universal service lifecycle

All categories use the same top-level lifecycle, but not the same screens:

```text
Discover
  → Understand
  → Save / Compare / Enquire
  → Eligibility & verification
  → Quote / Proposal / Cart
  → Consent & confirmation
  → Payment or booking (when applicable)
  → Partner fulfillment
  → Completion / ownership
  → Tracking / renewal / support
```

### Category fulfillment branches

| Service | Conversion | Verification | Fulfillment | Long-term object |
|---|---|---|---|---|
| Insurance | Quote → proposal → premium | Identity, insurer KYC, disclosures | Insurer underwriting / policy issue | Policy, renewal, claim |
| Financial investment | Suitability → order → payment | PAN/KYC, bank, risk profile, consent | Exchange/AMC/platform allocation | Portfolio, SIP, statements, redemption |
| Land investment | Enquiry → due diligence → reservation → agreement | Identity, funding, legal documents | Site visit, title/legal review, registration | Holding, documents, updates, exit request |
| Property | Search → visit → negotiate → reserve | Identity; owner/broker verification | Visit, legal check, booking/lease/sale | Property case, payment schedule, documents |
| Hybrid farming | Project → risk review → units → agreement | Identity, suitability where relevant | Operator acceptance, project activation | Project holding, milestones, payout |
| ClickCard | Plan → membership payment | Basic identity / eligibility | Digital/physical card activation | Benefits, usage, renewal |
| IT solutions | Package/RFP → estimate → contract → milestone payment | Business/contact verification | Escrow milestones, delivery acceptance | Project workspace, files, AMC/support |
| Healthcare | Provider → slot → booking/payment | Patient identity; consent for records | Appointment/consultation/test | Reports, prescriptions, follow-up |
| Partner offer | Offer → activate → redeem | Account and eligibility | Code/QR + bill/merchant verification | Voucher/redemption record |

## 6. Order, lead, and activity model

Do not force every service into an `order`. Use three aggregate types:

- **Lead:** enquiry, quote request, site visit, custom IT requirement.
- **Order:** a priced, accepted transaction such as membership, premium, investment order, booking, or voucher.
- **Case:** a long-running workflow such as property purchase, claim, IT project, farm project, dispute, or grievance.

Every object appears in **Activity** and has:

- Human-readable status
- Current owner (user, TheBrokrs, partner, third party)
- Next action
- Expected timeline
- Documents and messages
- Money and reward consequences
- Help/escalation action

### Canonical transaction states

```text
DRAFT → SUBMITTED → ACTION_REQUIRED → PROCESSING → COMPLETED
                  ↘ FAILED → RETRY
                  ↘ CANCELLED
COMPLETED → PARTIALLY_REFUNDED / REFUNDED / DISPUTED
```

State changes are driven by idempotent events, not page navigation. Payment providers can deliver duplicates or out-of-order events, so webhook consumers must deduplicate by event ID and tolerate late authorization. Razorpay documents both behaviors: [webhook validation and idempotency](https://razorpay.com/docs/webhooks/validate-test/) and [payment event sequencing](https://razorpay.com/docs/webhooks/payments/).

## 7. Money, credits, rewards, and value

These must never share one balance:

| Value type | Meaning | Withdrawable? | Expiry | Reversal |
|---|---|---:|---|---|
| Money | Payment/refund routed through payment provider | Only through regulated route | No | Refund/chargeback |
| Service Credit | Promotional rupee-denominated benefit usable on eligible services | No | Campaign-defined | On refund/abuse |
| Reward Point | Loyalty unit with published conversion/use rules | No | Published policy | On source reversal |
| Voucher | Single-use entitlement with terms | No | Explicit date | Void/reissue |
| Discount | Price adjustment before payment | No | Transaction-only | Recalculated |
| Asset Value | Informational holding/project/portfolio value | Not a wallet balance | N/A | Market/project update |

### Reward state machine

```text
ELIGIBLE EVENT
  → CALCULATED
  → PENDING (refund/fraud/partner verification window)
  → VESTED
  → REDEEMED or EXPIRED

Any source cancellation/refund/chargeback:
  PENDING → CANCELLED
  VESTED → REVERSED (or negative adjustment with audit link)
```

Each ledger entry stores source event, rule version, campaign, expiry lot, debit/credit, and reversal reference. The displayed balance is a projection; the ledger is the authority.

### Earning rules

- Publish “what action,” “how many points,” “when they become available,” “expiry,” and caps before action.
- Regulated/high-value services should reward completion or engagement, not encourage unsuitable transaction size.
- Use fixed milestone rewards for KYC, referral qualification, renewal, verified partner spend, and service completion.
- Never present points as investment return or guaranteed cash.

### Redemption flow

```text
Rewards catalog
  → eligibility + inventory + expiry check
  → user confirms points and terms
  → atomic points reservation
  → provider/partner fulfillment
  → voucher/service credit issued
  → used / expired / failed

Fulfillment failure → release reserved points or reissue entitlement.
```

## 8. Referral architecture

Use a simple, transparent **single-level referral** at launch. Remove MLM-lite and multi-level earning; it creates trust, compliance, and fraud risk without improving the core service experience.

### Referral state machine

```text
LINK_CREATED
  → CLICK_ATTRIBUTED
  → SIGNUP_MATCHED
  → QUALIFICATION_PENDING
  → QUALIFIED
  → FRAUD_HOLD (only if triggered)
  → REWARD_PENDING
  → REWARD_VESTED

Terminal alternatives: EXPIRED / DUPLICATE / INELIGIBLE / CANCELLED / REVERSED
```

Rules:

- Attribution window and last/first-touch rule are published.
- Existing account, same identity/device/payment method, self-referral, and duplicate household rules are explicit.
- Qualification is a real eligible service event, not only KYC or deposit.
- The UI shows inviter and invitee progress separately.
- Refund/cancellation inside the vesting period reverses the pending reward.
- Admin manual adjustments require reason, evidence, maker-checker approval, and audit history.

## 9. KYC and trust architecture

Replace universal numbered KYC levels with **capabilities**:

- `PHONE_VERIFIED`
- `IDENTITY_VERIFIED`
- `PAN_KYC_VALID`
- `BANK_VERIFIED`
- `ADDRESS_VERIFIED`
- `VIDEO_KYC_VALID`
- `RISK_PROFILE_VALID`
- `BUSINESS_VERIFIED`
- `PATIENT_CONSENT_VALID`

Each action declares required capabilities. The user completes only missing items and returns to the saved intent. A capability can expire, be rejected, or require refresh without invalidating unrelated services.

For investment journeys, show risks, fees, documents, order status, settlement timeline, and grievance escalation. Groww’s official help makes order state visible and documents KYC-related pending/failure reasons; that pattern supports a persistent Activity timeline rather than a one-time success screen: [order status](https://groww.in/help/mutual-funds/mf-sip/what-is-my-order-status-70), [failed purchase reasons](https://groww.in/help/mutual-funds/order/why-did-my-order-fail--83).

## 10. Partner lifecycle

```text
APPLICATION
  → BUSINESS VERIFICATION
  → CONTRACT & COMMERCIALS
  → BANK / TAX VERIFICATION
  → CATALOG OR OFFER DRAFT
  → ADMIN REVIEW
  → PUBLISHED
  → LEADS / ORDERS / REDEMPTIONS
  → FULFILLMENT EVIDENCE
  → COMMISSION & SETTLEMENT
  → PERFORMANCE / RENEWAL
```

Risk paths: `CHANGES_REQUIRED`, `PAUSED`, `SUSPENDED`, `TERMINATED`. Pausing a partner must preserve customer access to existing orders, policies, documents, and support.

## 11. Admin product architecture

### Navigation

1. Overview
2. Work queues
3. Customers
4. Partners
5. Catalog & content
6. Leads, orders & cases
7. Payments & settlements
8. Rewards & referrals
9. Risk & compliance
10. Support & grievances
11. Analytics
12. Configuration, roles & audit

### Admin operating flow

```text
Event / request arrives
  → routed to role-based queue
  → operator claims case
  → context panel shows customer + service + payment + reward history
  → operator takes permitted action
  → sensitive change enters maker-checker approval
  → event emitted + customer/partner notified
  → SLA and audit record updated
  → case resolved or escalated
```

### Maker-checker actions

Require two-person approval for:

- Publishing or materially changing financial/service offers
- Manual points/credits above threshold
- Refund outside policy
- Partner bank or commission change
- KYC override or account unfreeze
- Role/permission escalation
- Bulk notification or campaign launch
- Data export or deletion approval

No admin edits ledger history. Corrections use compensating entries.

## 12. Event and domain architecture for scale

### Start as a modular monolith, extract by pressure

The previous design planned 14 microservices too early. Begin with independently owned modules in one deployable application plus async workers:

1. Identity & profile
2. Catalog & discovery
3. CRM/leads/cases
4. Orders & fulfillment
5. Payments & reconciliation
6. Rewards & referrals
7. Partners & settlements
8. Content/notifications/support
9. Admin/RBAC/audit

Extract services only when scale, compliance isolation, release ownership, or reliability boundaries justify it. Payments, ledger, notifications, and search are the most likely early extractions.

### Shared platform rules

- UUID/ULID identifiers and tenant/partner ownership on every aggregate.
- Versioned APIs and schemas.
- Outbox pattern for reliable event publication.
- Idempotency key on every money, reward, redemption, referral, and admin mutation.
- At-least-once consumers with deduplication.
- Saga/process manager for long-running cross-domain workflows.
- Object storage for evidence; metadata and access policy in database.
- Search index is a projection, never the system of record.
- Redis is cache/rate limit only, never ledger authority.
- Append-only audit log for user, partner, and admin actions.
- Feature flags and category configuration for incremental launches.

### Essential events

```text
customer.registered
verification.capability_granted | rejected | expired
lead.created | assigned | qualified | closed
order.submitted | action_required | completed | failed | cancelled
payment.authorized | captured | failed | refunded | disputed
fulfillment.started | milestone_completed | accepted
reward.calculated | pending | vested | reversed | expired | redeemed
referral.attributed | qualified | held | vested | reversed
partner.approved | published | paused | suspended
support.case_opened | escalated | resolved
admin.approval_requested | approved | rejected
```

## 13. Reliability and performance model

- Separate SLOs by criticality: browse, authentication, order submission, payment confirmation, ledger, admin.
- Payment/order/reward truth is server-side; never trust client success callbacks alone.
- Provide `PROCESSING` when third parties are slow and resolve via webhook/reconciliation.
- Dead-letter queues and replay tools require audit and access control.
- Read models power Home and Activity without cross-querying every domain on each request.
- Paginate all histories and admin queues; cursor pagination for ledgers/events.
- Partition high-volume ledgers by time and/or account hash; archive without losing audit access.
- Cache public catalogs/CDN media; do not cache private KYC responses broadly.
- Bulk admin jobs are asynchronous, previewable, cancellable, and report partial failures.

## 14. Screen architecture changes required

### Add customer screens

- Activity dashboard and universal timeline
- Land investment list/detail/due-diligence/site-visit/agreement/holding
- Property search/detail/visit/offer/legal-check/case tracker
- ClickCard plan/activation/card/benefits/usage/renewal
- Generic lead/enquiry confirmation and partner response
- Quote/proposal comparison
- Payment processing/failure/refund/dispute
- Reward pending/reversed/expiry detail
- Referral status detail and ineligibility explanation
- Redemption confirmation/processing/failure
- Claims and grievances timeline

### Add partner screens

- Onboarding and verification
- Catalog/offer editor and approval state
- Lead/order queue
- Fulfillment milestones and evidence upload
- Redemption validation
- Settlement and invoice detail
- Performance and SLA dashboard

### Add admin screens

- Role-based overview and queue
- Customer 360
- Partner 360
- Lead/order/case detail
- KYC review
- Catalog approval
- Reward rule simulator and campaign approval
- Referral/fraud investigation
- Payment reconciliation/refund
- Settlement batch
- Support/grievance case
- Role management and audit explorer

## 15. Delivery order

### Foundation

Identity, catalog, Home/Explore/Activity/Profile, lead/case model, admin RBAC/audit, notifications.

### First revenue slice

Choose **one** service with full lifecycle, not five partial services. Recommended: insurance enquiry/quote or ClickCard membership because fulfillment is easier to validate before regulated investment expansion.

### Loyalty slice

Reward ledger, fixed milestone rules, catalog redemption, single-level referral, fraud review, admin controls.

### Marketplace expansion

Property/land/farming/IT category modules using the shared lead/order/case primitives.

### Regulated transaction expansion

Investment ordering, suitability, payments, allocations, statements, redemption, grievances, reconciliation.

## 16. Product success measures

- Discover → qualified lead/order conversion by service
- Time to first useful outcome, not only registration
- Verification completion and resume success
- Order/case completion and failure recovery
- Support contact rate per completed transaction
- Reward cost per incremental qualified action
- Referral qualified/vested ratio and fraud rate
- Redemption success and unfulfilled liability
- Partner acceptance, fulfillment SLA, dispute, and settlement accuracy
- Repeat use driven by active service management, not notification volume

---

**Canonical flow:** `flow.html` is the visual companion to this document. `W_Flow.html` is the low-fidelity screen inventory. `02-wireframes.html` is the current UI concept pack and should be updated against the missing-screen list before implementation.
