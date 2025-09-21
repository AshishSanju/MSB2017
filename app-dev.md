# Hookah Lounge POS on ERPNext — README

Operate a hookah lounge end‑to‑end on ERPNext using a focused custom Frappe app. This README gives you:
- The **tech stack**
- The **software development approach**
- The **domain model (DocTypes)**
- **POS configuration**
- **AI features** to add
- **Security, testing, deployment** checklists
- **Starter code** and **commands**

> Target: ERPNext/Frappe v15+

---

## 1) Tech Stack

- **ERPNext** (Selling, Stock, Accounts, POS) — billing, pricing, taxes, inventory, loyalty.
- **Frappe Framework** (Python + JavaScript) — DocTypes, permissions, REST, realtime, background jobs.
- **Bench CLI** — scaffold apps/sites; manage dev/prod.
- **Optional**: *frappe_docker* for production containers; Payments app for online gateways.

**Official docs**
- Frappe: https://frappeframework.com/docs
- ERPNext: https://docs.erpnext.com/
- Bench: https://frappeframework.com/docs/user/en/bench

---

## 2) Software Development Approach (Phased)

### Phase A — Environment & Scaffolding
```bash
# Install bench (see official docs for OS prereqs)
bench init dev-bench --frappe-branch version-15
cd dev-bench

# Create app and site
bench new-app hookah_pos
bench new-site lounge.local

# Install ERPNext + your app
bench --site lounge.local install-app erpnext
bench --site lounge.local install-app hookah_pos

# Start dev
bench start
```

Enable developer mode so DocTypes and customizations are tracked in Git:
```bash
bench --site lounge.local set-config developer_mode 1
bench --site lounge.local clear-cache
```

### Phase B — Model Your Domain
Create the DocTypes below and link them to ERPNext masters (Item, POS Profile, Warehouse). Keep validations in controllers; use least‑privilege permissions.

### Phase C — POS Setup
Configure **POS Profile** (warehouse, price list, payment modes, allow rate/discount edits, default print formats). Dry‑run billing flow on the POS screen.

### Phase D — App Logic & UX
- **Hooks** to react on submit/validate events.
- **Client Scripts** to streamline cashier UX (shortcuts, scan‑to‑add, mix picker).
- **Server Scripts / Controllers** for business rules.
- **Realtime** to push live prep queue/table map updates.

### Phase E — Reporting
Start with **Report Builder**; add Script/Query Reports for KPIs (mix sales, session duration, shrinkage, staff productivity).

### Phase F — Background Jobs & Scheduler
Use `frappe.enqueue` and `scheduler_events` for nightly forecasts, auto‑reorder suggestions, hourly low‑stock alerts, and end‑of‑day roll‑ups.

### Phase G — Testing, Packaging, Deployment
- Tests under `hookah_pos/tests/`; run with `bench --site lounge.local run-tests`.
- Package with Git; deploy via Bench or *frappe_docker*.
- Use separate **staging** and **production** sites; enable HTTPS/TLS.

### Phase H — Security & Compliance (Must‑Have)
- Role‑based permissions; token auth for APIs; rate limiting.
- Do **not** store card data in ERPNext. For card‑present, use a P2PE terminal and record settlement results only (PCI DSS scope control).
- Follow OWASP ASVS for general web app controls.


---

## 3) Domain Model (DocTypes)

> Keep custom DocTypes minimal; reuse ERPNext Items/Stock wherever possible.

**Core**
- **Flavor** — SKU for tobacco flavor (brand, strength, barcode, batch/expiry if tracked).
- **Mix Recipe** (parent) + **Mix Ingredients** (child) — define blends (like a sales BOM).
- **Hookah Session** — table, server, head type, chosen mix, start/end times, linked POS/Sales Invoice.
- **Lounge Table** — table number, capacity, zone, status (Available / Occupied / Cleaning).
- **Preparation Ticket** — generated from POS invoice items; printed to prep station.
- **Age Verification Log** — minimal audit (e.g., checkbox, timestamp, shift, hashed reference); avoid storing PII.
- **Shift** — open/close times, float cash, counted totals (ties to POS Closing).
- **Membership/Loyalty** — reuse ERPNext Loyalty Program; add fields if you need extras.

**ERPNext Masters to Configure**
- **Items**: Hookah Session (Service), Refills (Service), Flavors/Coals/Drinks (Stock).
- **Item Groups**: Services, Flavors, Consumables, Drinks.
- **Item Variants / Attributes**: e.g., brand, strength.
- **UoM**: `session`, `refill`, `hour` as needed.
- **Barcodes** for quick scan.
- **Product Bundles** for offers (e.g., Hookah + Drink).

**Inventory Controls**
- **Warehouse** per location.
- **Batch** (and expiry) for tobacco where required.
- **Stock Reconciliation** for periodic counts.

---

## 4) Pricing, Taxes, Loyalty, Memberships

- **Price Lists & Item Prices**: Standard, VIP, Happy Hour.
- **Pricing Rules**: time‑bound discounts, quantity breaks, combos.
- **Taxes**: Sales Taxes and Charges Template; enable “tax inclusive” if your menu is tax‑inclusive.
- **Modes of Payment**: Cash, Card (via external terminal), Wallet.
- **Loyalty Program**: earn/redeem points on POS.
- **Memberships**: use **Subscriptions** for monthly memberships; auto‑invoice.

---

## 5) POS Configuration & Daily Ops

- **POS Profile**: default Warehouse, Price List, allowed edits, payment modes, receipt format.
- **POS Screen**: add items, apply discounts (if allowed), collect payment.
- **Prep Tickets**: auto‑created on submit; routed to prep station printer.
- **POS Closing Voucher**: end‑of‑day cashup & reconciliation; use **POS Invoice Consolidation** to post to accounts.

---

## 6) Customizations (No New App Code Required)

- **Custom Fields** on Sales/POS Invoice: `Table #`, `Hookah #`, `ID Verified`, `Prep Notes`.
- **Client Scripts**: UI logic (auto‑set prep notes, warn on multiple hookahs per table).
- **Server Scripts**: enforce rules (e.g., block submit if `ID Verified` is false; auto‑print prep ticket).
- **Print Format** (Builder/Jinja): branded receipts and internal prep tickets with QR, table, server, mix.

---

## 7) AI Features (Build Faster + Add Smart Value)

**For Developers**
- Use an LLM to scaffold DocTypes, hooks, tests, and reports; iterate quickly.

**In‑App (Business Value)**
- **Demand Forecasting & Auto‑Reorder** — train nightly on invoice history; write model to private files.
- **Mix Recommendations** — “customers who liked X also liked Y.”
- **Anomaly Detection** — spot unusual voids/discounts/refunds.
- **Staff Co‑Pilot** — an in‑Desk helper for SOPs (“how to close shift”, “how to add mix”).

**Implementation Notes**
- Run training in background jobs (`frappe.enqueue`), schedule via `scheduler_events`.
- Store models under **private** files; log prompts/outputs; redact PII.
- Consider OWASP **AISVS** for AI feature hardening.

---

## 8) Security & Compliance Checklist

- **AuthN/Z**: Role‑based permissions; **User Permissions** (restrict by Company/Warehouse).
- **API**: Token‑based auth; add **rate limiting** on any custom endpoints.
- **Transport**: HTTPS/TLS in production.
- **Payments**: Keep cardholder data out of ERPNext; use terminals/gateways; store tokens/refs only.
- **Audit**: Use POS Closing, shift logs, and immutable logs for financial events.

References:
- OWASP ASVS: https://owasp.org/ASVS/
- PCI DSS (general info): https://www.pcisecuritystandards.org/

---

## 9) Testing & CI

- Place tests under `hookah_pos/tests/` named `test_*.py`.
- Unit tests for controllers and scripts; integration tests for event flows.
- Run: `bench --site lounge.local run-tests`
- Seed fixtures for stable test data.

Docs:
- Frappe Testing: https://frappeframework.com/docs/user/en/testing

---

## 10) Starter Code (Copy‑Paste)

**hooks.py**
```python
# hookah_pos/hooks.py
doc_events = {
"Sales Invoice": {
"on_submit": "hookah_pos.api.on_pos_invoice_submit"
}
}

scheduler_events = {
"cron": {
"0 3 * * *": ["hookah_pos.ai_jobs.retrain_demand_model"], # nightly
"0 * * * *": ["hookah_pos.ops.low_stock_check"] # hourly
}
}
```

**DocType Controller Example**
```python
# hookah_pos/hookah_pos/doctype/hookah_session/hookah_session.py
import frappe
from frappe.model.document import Document

class HookahSession(Document):
def validate(self):
if not self.age_verification_log:
frappe.throw("Age verification is required for a session.")
```

**POS Submit Hook**
```python
# hookah_pos/api.py
import frappe

@frappe.whitelist()
def on_pos_invoice_submit(doc, method=None):
inv = doc if isinstance(doc, dict) else doc.as_dict()
for item in inv.get("items", []):
# mark items needing a prep ticket, e.g., via a custom field
if item.get("is_hookah_mix"):
_create_prep_ticket(inv, item)
frappe.publish_realtime("prep_queue_update", {"invoice": inv.get("name")})
```

**Background Jobs**
```python
# hookah_pos/ai_jobs.py
def retrain_demand_model():
# pull recent invoices, update model in private files
pass

# hookah_pos/ops.py
def low_stock_check():
# compute par levels per flavor; notify manager if below threshold
pass
```

**Client Script (example)**
```javascript
// Client Script for Sales Invoice
frappe.ui.form.on('Sales Invoice', {
refresh(frm) {
// add quick buttons or barcode handler
}
});
```

---

## 11) Suggested Repo Structure

```
hookah_pos/
├─ hookah_pos/
│ ├─ hooks.py
│ ├─ api.py
│ ├─ ai_jobs.py
│ ├─ ops.py
│ └─ doctype/
│ ├─ hookah_session/
│ │ ├─ hookah_session.json
│ │ └─ hookah_session.py
│ ├─ mix_recipe/
│ └─ mix_ingredient/
├─ public/ # assets for desk/pos (icons, js)
├─ fixtures/ # roles, custom fields, print formats (optional)
├─ tests/
│ └─ test_flow.py
└─ README.md
```

---

## 12) ERPNext Feature References (Official Docs)

- **POS User Guide**: https://docs.erpnext.com/docs/v15/user/manual/en/retail/point-of-sale
- **POS Profile**: https://docs.erpnext.com/docs/v15/user/manual/en/retail/pos-profile
- **POS Closing Voucher**: https://docs.erpnext.com/docs/v15/user/manual/en/retail/pos-closing-voucher
- **POS Invoice Consolidation**: https://docs.erpnext.com/docs/v15/user/manual/en/retail/pos-invoice-consolidation
- **Item / Variants / UoM / Barcode**:
- Item: https://docs.erpnext.com/docs/v15/user/manual/en/stock/item
- Item Variants: https://docs.erpnext.com/docs/v15/user/manual/en/stock/item-variant
- UoM: https://docs.erpnext.com/docs/v15/user/manual/en/stock/unit-of-measure
- Barcode: https://docs.erpnext.com/docs/v15/user/manual/en/stock/barcode
- **Pricing**:
- Price List: https://docs.erpnext.com/docs/v15/user/manual/en/selling/price-list
- Item Price: https://docs.erpnext.com/docs/v15/user/manual/en/selling/item-price
- Pricing Rule: https://docs.erpnext.com/docs/v15/user/manual/en/selling/pricing-rule
- **Taxes**:
- Sales Taxes & Charges: https://docs.erpnext.com/docs/v15/user/manual/en/accounts/sales-taxes-and-charges-template
- Tax Inclusive: https://docs.erpnext.com/docs/v15/user/manual/en/accounts/taxes-and-charges#is-this-tax-included-in-basic-rate
- **Inventory**:
- Batch: https://docs.erpnext.com/docs/v15/user/manual/en/stock/batch
- Stock Reconciliation: https://docs.erpnext.com/docs/v15/user/manual/en/stock/stock-reconciliation
- **Payments & Roles**:
- Modes of Payment: https://docs.erpnext.com/docs/v15/user/manual/en/accounts/modes-of-payment
- Role Permissions: https://frappeframework.com/docs/user/en/guides/app-development/roles-and-permissions
- User Permissions: https://frappeframework.com/docs/user/en/guides/desk/user-permissions
- **Customization & Printing**:
- Custom Field: https://frappeframework.com/docs/user/en/customize-erpnext/custom-field
- Client Script: https://frappeframework.com/docs/user/en/guides/app-development/client_script
- Server Script: https://frappeframework.com/docs/user/en/guides/app-development/server_script
- Print Format: https://frappeframework.com/docs/user/en/guides/print/print-format
- Report Builder: https://frappeframework.com/docs/user/en/reporting/report-builder
- **Realtime / Background Jobs**:
- Realtime Events: https://frappeframework.com/docs/user/en/guides/app-development/realtime
- Background Jobs & Scheduler: https://frappeframework.com/docs/user/en/guides/background_jobs
- **Bench**:
- Bench Commands: https://frappeframework.com/docs/user/en/bench
- Developer Mode: https://frappeframework.com/docs/user/en/guides/app-development/developer_mode
- **Payments App (optional)**:
- https://frappeframework.com/docs/user/en/guides/integration/payments

---

## 13) Roadmap (Recommended Next Steps)

1. Scaffold app & site; enable developer mode.
2. Create DocTypes (`Hookah Session`, `Mix Recipe`, etc.).
3. Configure POS Profile, Items, Price Lists, Taxes.
4. Build hooks for POS submit → Prep Tickets + Realtime queue.
5. Add Client/Server scripts for cashier UX and rule enforcement.
6. Implement nightly forecast + low‑stock jobs.
7. Create reports for sales/mixes/shift KPIs.
8. Lock down roles, API auth, and rate limits; add tests.
9. Package for staging → production; enable HTTPS; train staff.

---

**Author**: Ashish / Mist Hookah Lounge
**License**: MIT (suggested)
**Contact**: ops@yourdomain.example
