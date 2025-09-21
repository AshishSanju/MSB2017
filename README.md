Hookah Lounge POS on ERPNext

Operate a hookah lounge end-to-end on ERPNext: fast POS, inventory for flavors/consumables, pricing/promos, taxes (incl. tax-inclusive), loyalty, memberships, roles/permissions, custom tickets, and reporting.

This guide maps lounge workflows to native ERPNext features and links to the official docs for each feature.

⸻

Core Data Model
	•	Items
	•	Model Hookah Session as a Service item (no stock movement).
	•	Model Flavors, Coals, Drinks/Snacks as stock items.
	•	Organize with Item Group; add Item Attributes (e.g., strength/brand) and generate Item Variants where needed.  ￼
	•	UoM
	•	Add units like session, refill, or hour as needed. Enforce whole numbers where applicable.  ￼
	•	Barcodes
	•	Maintain per-item (even UoM-specific) barcodes for quick scan at POS.  ￼
	•	Combos
	•	Build “Hookah + Flavor + Coal” or “Hookah + Drink” offers as Product Bundles (sales BOM).  ￼

⸻

Pricing & Promotions
	•	Price Lists & Item Prices
	•	Maintain lounge vs. VIP vs. happy-hour price lists, and item rates via Item Price.  ￼
	•	Pricing Rules
	•	Create discount/markup rules by item, item group, customer group, quantity, etc. (e.g., “2 refills = 10% off”).  ￼
	•	Tax-Inclusive Pricing
	•	If you price “tax included,” mark the tax row as Is this tax included in Basic Rate? in your Sales Taxes and Charges Template.  ￼

⸻

POS Configuration & Daily Operations
	•	POS Profile
	•	Configure default Warehouse, Price List, payment modes, and permissions like Allow user to edit Rate/Discount, Print before Pay, and Display Items in Stock.  ￼
	•	Billing Flow
	•	From Point of Sale, pick a customer (optional), add items, adjust qty/discount (if allowed by profile), then collect payment. Warehouse preference comes from POS Profile.  ￼
	•	End-of-Day
	•	Use POS Closing Voucher to close shifts and reconcile counted cash/cards. POS bills are consolidated to accounting via POS Invoice Consolidation.  ￼
	•	Modes of Payment
	•	Define Cash, Card, Mobile Wallet, etc., and map default accounts.  ￼

⸻

Loyalty & Memberships
	•	Loyalty Program
	•	Award/redeem points on POS sales for frequent guests (tiers optional).  ￼
	•	Memberships / Passes
	•	Use Subscriptions for monthly lounge memberships (auto-create Sales Invoices). Configure Subscription Plans (period, pricing).  ￼

⸻

Taxes & Compliance
	•	Configure Sales Taxes and Charges Template per jurisdiction; combine with tax-inclusive setting if pricing includes tax.  ￼

⸻

Inventory Controls
	•	Batch Tracking
	•	Track flavor/tobacco lots with Batch (and expiry dates if relevant). Negative stock is disallowed for serial/batch items in v15+.  ￼
	•	Stock Reconciliation
	•	Run regular counts for flavors, coals, and packaged goods and reconcile to system.  ￼

⸻

Roles, Users & Permissions
	•	Roles & Access
	•	Lock down POS to cashiers, managers, etc., via Role-based Permissions and Users & Permissions.  ￼
	•	Granular User Permissions
	•	Restrict to a specific Company, Warehouse, or Territory for multi-branch setups.  ￼

⸻

Printing & Station Tickets
	•	Receipts & Prep Tickets
	•	Customize receipts with lounge branding using Print Format (Builder/Jinja). Create an internal Prep Ticket format (e.g., prints flavor, bowl type, table #) to route to the prep station. Select default print formats in the POS Profile.  ￼

⸻

Customizations (No App Code Required)
	•	Custom Fields
	•	Add fields like Table #, Hookah #, Server, ID Verified, Prep Notes on POS Invoice or Sales Invoice (Customize Form).  ￼
	•	Client Scripts
	•	UI logic (e.g., auto-set Prep Notes from chosen flavor; warn if multiple hookahs per table).  ￼
	•	Server Scripts
	•	Back-end rules (e.g., block posting if ID Verified is unchecked; auto-print prep ticket on submit).  ￼

If you outgrow configuration-only changes, you can still create app-level DocTypes and hooks, but most lounge needs fit within the above toolkit.  ￼

⸻

Reporting
	•	Built-in Sales Analytics
	•	Use Selling Analytics and report builder for top flavors, session counts, avg. ticket, and repeat customers.  ￼

⸻

Step-by-Step Setup Checklist
	1.	Master Data
	•	Create Item Groups (Hookah Services, Flavors, Consumables, Drinks). Create items; enable Variants where needed (e.g., flavor brand/strength). Add Barcodes.  ￼
	2.	Pricing & Taxes
	•	Create Price Lists and Item Prices. Add Sales Taxes and Charges Template; set tax-inclusive if required. Add Pricing Rules (combos/discounts).  ￼
	3.	Inventory
	•	Enable Batch for tobacco where you need lot/expiry tracking. Add opening stock; set Warehouses.  ￼
	4.	Payments
	•	Add Modes of Payment (Cash, Card, Wallet). Map default accounts.  ￼
	5.	POS
	•	Configure POS Profile (warehouse, price list, allowed edits, default print format). Test Point of Sale screen.  ￼
	6.	Loyalty & Membership
	•	Define Loyalty Program and Subscription Plans (if you offer memberships).  ￼
	7.	Permissions
	•	Assign roles; restrict cashiers to POS and specific company/warehouse via User Permissions.  ￼
	8.	Prints & Scripts
	•	Build receipt & prep ticket Print Formats; add Custom Fields; wire simple Client/Server Scripts for lounge rules.  ￼
	9.	Go-Live Ops
	•	Train staff on POS flow. Use POS Closing Voucher nightly to reconcile and post consolidated invoices.  ￼

⸻

Operational Tips (Mapped to Docs)
	•	Prefer Service items for sessions/refills (clean accounting; no stock). Use Batch for tobacco (expiry).  ￼
	•	Keep time-bound promos with Pricing Rules; keep separate Price Lists for VIP vs. standard.  ￼
	•	Use Print Format to add QR codes, membership info, and table number on receipts/prep tickets.  ￼
	•	Lock down the POS with Role-based Permissions + User Permissions; audit with the nightly POS Closing Voucher.  ￼

⸻

Appendix: Doc Links (Official)
	•	POS: Point of Sale (usage), POS Profile, POS Closing, POS Invoice Consolidation.  ￼
	•	Items & Inventory: Item, Item Variants, UoM, Barcode, Batch, Stock Reconciliation.  ￼
	•	Pricing & Tax: Price Lists, Item Price, Pricing Rule, Sales Taxes & Charges, Tax Inclusive Accounting.  ￼
	•	Payments & Roles: Mode of Payment, Role-based Permissions, Users & Permissions, User Permissions.  ￼
	•	Loyalty & Membership: Loyalty Program, Subscription, Subscription Plan.  ￼
	•	Customization & Printing: Custom Field, Client Script, Server Script, Print Format, Report Builder.  ￼

⸻

What’s not in core (plan customizations)
	•	Table layout management with live table occupancy, kitchen routing/UIs, or multi-screen KOT: do-able via Custom Fields/Scripts/Print Formats and (optionally) app-level customization, but not a turnkey core feature in official docs. Use the Customizations section above to implement a light version.  ￼
