# ReuseOps

**Use. Return. Ready again.**

Reusable cups, plates, containers, cutlery and shared event gear, tracked across a whole venue network — issued, returned, collected, washed, inspected and put back into service, without anyone having to guess where the stock is.

| | |
| --- | --- |
| **Live app** | <https://reuseops.lovable.app> |
| **Walkthrough video (43 s)** | <https://youtu.be/NQTnBzg5EzU> |
| **Demo workspaces** | Helsinki & Amsterdam — proposed demo settings, not confirmed partnerships |

---

## Why ReuseOps exists

A reusable cup only saves money and waste if it comes back and is used again. In practice that breaks down in four places:

1. **Nobody knows where an item is.** Is the cup on a shelf, in someone's bag, in the wash, or already retired? Spreadsheets say "30 cups", reality says "9 are usable".
2. **Returns and re-issues are not linked.** The same cup gets counted as issued twice, or a lost item quietly disappears from the stock.
3. **The wash loop is invisible.** Nobody can prove an item was collected, washed and checked before it went back out.
4. **Impact claims are invented.** "We saved 4,000 cups" with nothing behind it.

ReuseOps fixes all four by making **the return the centre of the system**: an item is only ever in exactly one recorded state, at one confirmed checkpoint, and every claim on the impact page traces back to a stored transaction.

```text
        ┌──────────────────────── ReuseOps loop ────────────────────────┐
        │                                                               │
  ready │──issue──▶ issued / borrowed ──return──▶ returned ──collect──▶ │
   ▲    │                                                               │
   │    │                            washing ──▶ awaiting_inspection    │
   │    │                                     │                         │
   └────┴────────── inspect (pass) ◀───────────┘                         │
                 │                                                      │
                 └── inspect (fail) ──▶ cleaning / maintenance ──▶ back ─┘

  Anywhere in the loop: transfer between venues, quarantine, retire.
```

---

## Try it in 60 seconds

1. Open <https://reuseops.lovable.app> and create an account (email + password, pick a role at onboarding).
2. You are automatically joined to the **Helsinki** and **Amsterdam** demo workspaces — both clearly labelled as demo settings. Prefer a clean slate? Choose **Start empty pilot**.
3. Go to **Scan / Borrow / Return**, pick a location, and type a code: `CUP-001`, `PLATE-001`, `KIT-001`, `SHARED-001`.
4. ReuseOps only offers the actions that are legal for that item's current state and your role. Anything else is refused by the database, not just hidden in the UI.
5. Switch roles under **Settings → Demo role** (Borrower, Venue staff, Operator, Organiser) to see the same item from every side.

---

## What each screen does

| Screen | Path | Purpose |
| --- | --- | --- |
| Home | `/` | Public introduction to the return-first idea. |
| Sign in / up | `/auth` | Email + password, role choice at onboarding, joins the demo workspaces. |
| Overview | `/overview` | Ready items, items out on loan, items in the wash loop, things needing attention, latest activity. |
| Scan / Borrow / Return | `/scan` | Scan a QR or type a code, choose your location, take the valid next action. |
| Inventory | `/inventory` | Every item with status, home venue and last confirmed checkpoint; bulk add for organisers. |
| Stations | `/stations` | Venues, return stations, washing hubs, maintenance points, and each category's workflow rules. |
| Operations | `/operations` | Collection, washing, inspection, cleaning and maintenance tasks, created automatically as items move. |
| Network | `/network` | Stock per venue and the transfer history; ownership always stays with the issuing venue. |
| Impact & costs | `/impact` | Counts from recorded transactions only; estimates only where a sourced assumption exists; CSV export. |
| Settings | `/settings` | Demo role switch, demo reset, printable QR labels, sign out. |
| Item passport | `/item/{code}` | One item's permanent history, step by step. |
| **Public** scan | `/qr` | Camera scan or typed code with no account; multi-pool matches disambiguated. |
| **Public** near me | `/near` | Map + list of participating shared areas, filtered by group, borrow/return, open now, compatibility. |
| **Public** item page | `/i/{id}` | Item passport reachable straight from a QR label, with borrow / return request. |
| **Public** area page | `/a/{id}` | Area details, opening hours, accepted items, and the staff queue for confirming handovers. |
| My loans | `/loans` | Signed-in view of items out to you and your pending requests, with cancel. |
| Shared areas | `/areas` | Organiser admin: create and edit areas, locate by address or map pin, set hours, accepted categories, capacity, staff. |

---

## Item states and workflow rules

An item is always in exactly one of these states:

`ready` · `issued` · `borrowed` · `returned` · `collected` · `washing` · `awaiting_inspection` · `cleaning` · `maintenance` · `quarantined` · `retired`

Supported actions: `issue`, `return`, `collect`, `start_wash`, `finish_wash`, `inspect` (pass / fail), `finish_cleaning`, `finish_maintenance`, `release_quarantine`, `transfer`.

Rules come from the item's **category**, so different gear can follow different loops:

| Category flag | Effect |
| --- | --- |
| `washing_required` | Food-contact items must pass through washing before they can be ready again. |
| `cleaning_allowed` / `maintenance_allowed` | Shared gear can be cleaned or repaired instead of washed. |
| `inspection_required` | An inspection record is needed before the item returns to service. |
| `return_deadline_hours` | Drives the due date and the overdue list. |
| `compatible_location_ids` | Where the item may physically be handed over. |
| `deposit_amount` | Recorded as a deposit when the item is issued. |
| `max_uses` | Use count is tracked per item for retirement decisions. |

Demo categories: Reusable cup (48 h, deposit 1), Lunch plate (24 h), Lunch box (48 h), Cutlery set (48 h), Shared charger (8 h), Shared tool (72 h).

Two safety nets matter most:

- **One open loan per item.** A second issue while an item is on loan is rejected by the database.
- **Idempotent writes.** Every action carries a request id; scanning the same code twice cannot double-count anything.

---

## Roles

| Role | Can do |
| --- | --- |
| **Borrower** | Request borrow and return, see their own loans and requests. |
| **Venue staff** | Confirm handovers, handle collection and return tasks at their assigned areas. |
| **Operator** | Washing, cleaning, inspection and maintenance work, plus the operational queues. |
| **Organiser** | Everything above, plus locations, categories, items, areas, staff assignment, impact assumptions, demo reset. |

Roles live in a protected `user_roles` table and are checked inside the database on every operation — never taken from user-editable profile data.

---

## Shared public areas, QR codes and staff-confirmed handovers

Anyone — even without an account — can open a QR label and see the item, or browse participating areas near them.

- **QR labels encode the public app address plus a stable public id** (`https://reuseops.lovable.app/i/{public_id}` and `/a/{public_id}`). No personal data, no session tokens, no database identifiers.
- **Public pages read only through security-definer functions** (`public_areas`, `public_area`, `public_item`, `resolve_code`). Anonymous visitors never receive direct table access.
- **A scan never moves an item.** Borrow and return requests are stored in `custody_requests` and only change item state when staff physically confirm the handover (`resolve_custody_request`: confirm issued, confirm received, received damaged, decline). A QR scan proves intent, not possession.
- **Requests are idempotent** — the same request id returns the original result instead of creating a duplicate queue entry.
- **Location privacy:** "Find near me" uses your device location only at the moment you press the button and never stores it; you can search by address or tap the map instead.
- **Map and geocoding:** OpenStreetMap tiles with attribution, plus address lookup, both loaded only in the browser.

---

## Impact & costs — what is counted and what is not

- Every **count** (items issued, returned, reused, in the wash, on loan) is derived from stored `transactions` rows. Nothing is added by the interface.
- Every **estimate** (cost per drink, CO₂ avoided, waste avoided) appears only when an organiser has saved a value **with a named source** in `impact_assumptions`. Unset assumptions are shown as "not provided", never filled in with a plausible-looking number.
- **Demo workspaces are excluded** from any real-world claim and are labelled as such on the page itself.
- **CSV export** gives the underlying transaction rows so the numbers can be checked elsewhere.
- ReuseOps **records checks, it does not certify hygiene or safety compliance.** That disclaimer is shown wherever inspection results appear.

---

## Verified journeys

These were driven end to end through the running app on a phone-sized screen:

1. **Cup across venues** — `CUP-001` issued at Venue A → returned at Venue B → collected at Station 2 → washed → inspected pass → `ready` again, with a collection task created automatically and every step written to the permanent history.
2. **Shared item damaged** — `SHARED-001` lent out → returned damaged → maintenance point → maintenance finished → inspected pass → available.
3. **Kit with a missing piece** — `KIT-001` returned with one component missing raises the missing-item alert instead of silently closing the loan.
4. **Public borrow request** — signed-out visitor opens `/near` and `/qr`, types `CUP-002`, chooses the Helsinki or Amsterdam copy; a signed-in visitor requests to borrow, staff confirm on the area page, and it appears under **My loans** with a due date.
5. **Guardrails** — invalid actions are blocked server-side, duplicate scans cannot inflate counts, and external URLs in QR codes are refused.

---

## Architecture

**Front end:** TanStack Start v1 (React 19, file-based routing, SSR), TypeScript, Tailwind CSS v4 (theme tokens in `src/styles.css`, Fraunces + Source Sans 3, dark mode, reduced-motion support), TanStack Query for data.
**Maps / camera:** Leaflet + OpenStreetMap (loaded only in the browser), jsQR for camera and photo-code reading, `qrcode` for printable labels.
**Back end:** Lovable Cloud — PostgreSQL with row-level security, email/password auth, and SQL functions as the transactional core.
**App server logic:** `createServerFn` from `@tanstack/react-start`; public read paths are security-definer RPCs.

### Data model

| Table | Holds |
| --- | --- |
| `workspaces` | A venue network; `is_demo` marks Helsinki and Amsterdam. |
| `user_roles` | Workspace + user + role (borrower / staff / operator / organiser). |
| `locations` | Venues, return stations, washing hubs, maintenance points; coordinates, timezone, opening hours, public flags, borrow/return permissions, accepted categories, capacity, accessibility notes. |
| `item_categories` | Workflow rules per kind of gear (see the flag table above). |
| `items` | One row per physical object: code, public id, status, home venue, current location, last checkpoint, responsible person, use count. |
| `kits` / kit members | A kit and its components, so a partial return is visible. |
| `loans` | Open and closed loans, with due date and borrower label. |
| `transactions` | **Append-only** ledger of every state change — the single source for all counting. |
| `tasks` | Collection, washing, inspection, cleaning and maintenance work, auto-created and auto-closed. |
| `inspections` | Pass / fail results with notes. |
| `transfers` | Movement of an item between venues; ownership unchanged. |
| `deposit_records` | Deposits taken, refunded or forfeited. |
| `impact_assumptions` | Organiser-entered values with sources; null means "unknown". |
| `area_staff` | Which staff may confirm handovers at which area. |
| `custody_requests` | Borrow / return requests and the staff decision on each. |

### Server functions

Transaction core: `perform_action` (atomic, idempotent, validates the transition against category rules and your role, writes the ledger row and any tasks), `add_items`, `create_pilot`.
Demo helpers: `seed_demo`, `reset_demo` (demo workspaces only), `join_demos`, `set_demo_role` (demo only).
Public access: `public_areas`, `public_area`, `public_item`, `resolve_code`, `request_custody`, `resolve_custody_request`, `area_pending_requests`, `my_loans`, `workspace_members`.

### Security invariants

- Public pages read **only** through security-definer functions; anonymous users never get table grants.
- Borrow and return **only change item state when staff confirm** the physical handover.
- QR labels encode the public app address plus a stable public id — never personal data.
- Application tables and functions stay in `public`; roles are read from `user_roles`, never from profile metadata.
- Demo workspaces are always presented as demo; impact estimates require a sourced assumption.

---

## Demo data

Seeded into Helsinki and Amsterdam, and re-seeded by **Settings → Reset demo**:

- **Venues:** Venue A and Venue B cafés (public, borrow + return).
- **Return stations:** three stations with public instructions.
- **Washing hub** and **maintenance point**.
- **Items:** `CUP-001`–`CUP-008`, `PLATE-001`–`PLATE-006`, `KIT-001` (box, cutlery, plate), `SHARED-001`, `SHARED-002`.
- **Impact assumptions:** intentionally empty, so nothing is estimated until you supply a value and a source.

Both workspaces are labelled in the interface as **proposed demo settings, not confirmed partnerships**.

---

## Running it locally

```sh
git clone <repository-url>
cd reuseops
bun install
bun run dev
```

Environment values for the backend are written to `.env` by Lovable Cloud (`VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY`). Database changes live in `drizzle/migrations/` — `0000_loopops_core.sql` (core schema, RLS, transaction functions, demo seeds) and `0001_shared_areas_qr_custody.sql` (public area fields, area staff, custody requests, public RPCs).

---

## Notes and limits

- Email confirmation is currently instant so the demo can be tried immediately; say the word and confirmation emails will be required again.
- Until real areas with an address or map pin are added under **Shared areas**, nearby searches correctly report that no participating areas were found.
- ReuseOps records that checks happened. It does not certify hygiene, food-safety or regulatory compliance.
