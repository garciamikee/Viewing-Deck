# Logistics &amp; Inventory Viewing Deck

A separate companion site to the
[Inventory-Logistics System](../inventory-logistics-system) — same Firebase
project, same live data, but with **no edit, delete, or check-in/out
screens**. Meant to be handed out broadly inside the company for visibility
(status of client deliveries, current stock). The one thing it can do is file
a **Request for Testing**.

It shows:
- **Logistics dashboard** — (with a search box across customer, PO, item, serial, tracking) For Delivery / In Transit / Delivered counts for
  client deliveries, click a card to see the entries (compact table, scrolls when long).
- **Request for testing** — a clickable option that opens a short form:
  request date, customer name, a required **Reference** file (named after the PO/RFQ no.), a required **assigned technical** (dropdown), and one or more items (a
  **single item / multiple items** toggle, like the check-out form), each
  with an item from the inventory list, quantity and serial number(s).
  "Requested by" is filled from the signed-in email and can't be changed.
  The customer is a dropdown of the Delivery Tracker customers with a **+ Add new customer…** option. A serial number that was already used (on another request, checked out in the Delivery Tracker, or not in stock) is refused.
  Filing a request takes the units out of stock straight away (stock is
  re-read from the database at that moment, so two people can't both take the
  last units) and creates a record in the main system, where its status is
  managed (On going → Passed / Incomplete / Defective / Returned).
- **Testing requests** — a read-only list of every request (customer, reference,
  items with serials, who filed it, status) with a search box. It's there so
  people can check before filing. The form also refuses a serial number that's
  already on another request. Compact table that scrolls when long.
- **Inventory list** — every SKU and its current stock, with search. Compact rows in a fixed-height box with a scroll bar on the side.

## Setup

This site shares the **same Firebase project** as the main system, so there's
nothing new to create in Firebase. Just:

1. `firebase-config.js` here is already a copy of the main system's — if you
   ever rotate Firebase projects, copy the updated file here too.
2. The same `firestore.rules` (in the main system's repo) already governs
   this site too, since it's the same database. No separate rules needed.

## Deploy

Same pattern as the main system — this is a second, independent GitHub repo
and Vercel project (so it gets its own URL, separate from the editable
system):

```bash
git init
git add .
git commit -m "Initial commit: Viewing Deck"
git remote add origin https://github.com/<you>/<repo-name>.git
git branch -M main
git push -u origin main
```

Then import that repo at https://vercel.com/new (framework preset: **Other**,
no build step). Sign-in here uses the same email/password accounts as the
main system — anyone who already created an account there can sign in here
too, nothing extra to configure.

## Why sign-in is still required

Firestore's rules (shared with the main system) require a signed-in
`@1wan.ph` account for every read, not just writes — so this page can't be
handed to literally anyone on the internet, only to people with a company
account. That was a deliberate trade-off: the alternative (open read access
to anyone with the link) would also expose customer names, PO numbers and
delivery routes to the open internet.

## Technical dashboard

Shows each technical's units out on test and every assignment (technical,
customer, item, quantity, serial no., status, PDI). The technical a request
is assigned to — matched on their sign-in email — can **Mark Passed /
Returned** and **Upload PDI** for it; everyone else sees it read-only. The
main system downloads the PDI once, which removes it. Technicals and emails
are listed in `TECHNICALS` in `index.html` (keep in step with the main
system). See the main system's README for details and the limits of this
protection.

## Testing status

The assigned technical (and only them) can set a request to **Passed**,
**Incomplete**, **Defective** or **Returned** from the Technical dashboard:

| From | Can go to |
| --- | --- |
| On going | Passed, Incomplete, Defective, Returned |
| Incomplete | Passed, Defective, Returned |
| Passed | Returned |

- **Incomplete** — the test isn't finished. The units stay out of stock and
  still count as units out for testing.
- **Defective** — the units failed. They stay out of stock (they are **not**
  added back to the inventory list) and the status is final.
- **Returned** — puts the units back in stock, as before.

All tables on the deck are compact and show a scroll bar when they overflow.
