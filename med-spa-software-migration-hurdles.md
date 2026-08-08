# Major Hurdles in Merging Data From Existing Med Spa Platforms

**Prepared:** August 2026
**Context:** Technical/product hurdles in building reliable migration from incumbent platforms (Zenoti, Vagaro, Boulevard, AestheticsPro, Mangomint) into a new med spa software platform.

---

Building reliable migration into a new platform is genuinely one of the harder problems in this space — harder than most of the product itself. Below is a breakdown of the major hurdles.

---

## 1. API access is inconsistent and often adversarial

This is the first wall you hit, and it's worse than people expect:

- **Zenoti**: production API keys are customer-gated — generated from inside an existing customer's account, with no public self-serve developer signup. There are no official SDKs, so integrations are built by hand against raw REST endpoints, the API surface spans 200+ endpoints across appointments, guests, invoices, memberships, and more, and per-organization rate limits (around 60 calls/minute) force careful batching. Webhook coverage is uneven, so much of the sync work relies on polling rather than real-time events.
- **Boulevard** has a real public developer portal, which is the exception rather than the rule.
- **Vagaro and Mangomint** have much thinner or non-existent public APIs for bulk data extraction — a lot of third-party tooling for these platforms exists as scraping/automation layers (e.g., Apify actors), not official integrations, which is fragile and can break or get blocked without notice.

**Practical implication:** you can't assume a clean, symmetric API relationship with any of these vendors. Some data access may require the *customer* to request their own export (CSV dumps, manual PDF exports) rather than you pulling programmatically — and incumbents have limited incentive to make it easy for you to take their customers.

---

## 2. No shared data model — every platform structures things differently

Even where data is accessible, there's no common schema across the industry:

- **Client/patient identity**: dedup logic differs (phone-based vs. email-based vs. internal IDs), so merging two platforms' client lists reliably without creating duplicates or false matches is a real entity-resolution problem.
- **Clinical records**: AestheticsPro's SOAP notes and injectable mapping are structured very differently from Vagaro's lighter-weight notes fields — some clinical data may exist only as free text or embedded PDFs, not structured fields you can map cleanly.
- **Financial history**: packages, memberships, and outstanding balances often have platform-specific business logic (proration rules, expiration policies) that doesn't translate 1:1 — a "membership" in Zenoti's beauty-bank model isn't structurally the same object as a Boulevard membership.
- **Appointment history**: recurring appointments, waitlists, and multi-provider bookings each have edge cases that are easy to get subtly wrong (e.g., a series appointment showing up as 12 separate migrated records instead of one recurring one).

This means migration isn't a simple export/import — it requires a genuine mapping/transformation layer per source platform, maintained and re-validated every time a source platform changes its schema.

---

## 3. Data quality on the source side is often bad to begin with

You're not migrating clean data — you're migrating years of manual entry:

- Inconsistent formatting (phone numbers, dates, names with typos)
- Duplicate client records already existing in the source system
- Incomplete or missing consent forms, especially older ones stored as scanned PDFs rather than structured data
- Orphaned records (appointments referencing deleted staff or discontinued services)

Your migration tool has to include cleaning and validation logic, and — critically — a way to flag ambiguous cases for human review rather than silently guessing, especially for clinical data.

---

## 4. Compliance risk is amplified during migration, not reduced

This is the part people underweight. Every record you touch is PHI:

- You need HIPAA-compliant handling (encryption in transit and at rest, audit logging of who accessed what) for data that's temporarily sitting in your migration pipeline, not just your production database.
- Business Associate Agreements (BAAs) need to be in place — with the practice, and potentially with the source vendor if you're accessing their API on the client's behalf.
- Chain-of-custody matters for consent forms and clinical notes specifically — if a form doesn't migrate cleanly, you can create a genuine compliance gap for the practice (no valid consent on file), which is a liability issue, not just a UX bug.
- Different states have different med-spa-specific regulations (delegated authority for injectables, retention periods for records), which affects what "accurate migration" even means legally.

---

## 5. Verification is fundamentally hard — "accurate" is expensive to prove

Once data is moved, how do you know it moved correctly?

- At meaningful scale (thousands of clients, years of history), manual spot-checking doesn't scale, but automated reconciliation requires building comparison tooling against the *old* system's data — which means maintaining read access to source platforms even after migration, at least temporarily.
- Silent data loss is the dangerous failure mode — a migration that visibly errors is fixable; one that quietly drops or corrupts records (e.g., an allergy note that doesn't transfer) can cause real patient harm downstream and won't surface until someone notices something's missing.

---

## 6. Vendor incentive misalignment — they may not want you to succeed

Worth naming directly: incumbents have no reason to make switching easy, and some make it actively hard (export limitations, API access revoked once they detect a competitor's traffic pattern, customer support that's unhelpful for "I'm leaving" requests). You may end up designing around **manual export formats** (CSV/PDF dumps a practice can pull themselves) as a fallback path precisely because programmatic access can't be relied on for every source.

---

## 7. Operational/scaling hurdles once you have more than one source platform

Each additional source platform you support is roughly its own engineering project — new schema mapping, new edge cases, new compliance review. This is why "we migrate from anything" is a much bigger promise than it sounds: realistically you'd want to launch supporting migration from the 1–2 platforms your target segment most commonly uses (probably Vagaro and Mangomint, given the underserved-middle segment identified in the market opportunity analysis) and expand deliberately rather than promising universal migration on day one.

---

## Bottom line

The technical integration work is real but solvable; the harder problems are (a) inconsistent/gated API access you don't control, (b) the absence of a shared data model across the industry, and (c) the compliance stakes of getting clinical data migration wrong. Migration reliability should be treated as a core product investment with its own dedicated team and QA process, not a one-time onboarding script — that's likely the difference between this being a real differentiator versus a source of churn from botched switches.
