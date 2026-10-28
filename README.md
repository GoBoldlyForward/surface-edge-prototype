# Surface Edge Order Desk

Clickable prototype of an order-tracking back office and customer portal for Surface Edge, the large-project division of Wanke Cascade. Purchase orders arrive by email, get broken into shipments, move through factory, ocean, port and delivery milestones, and close when every shipment is delivered and invoiced.

**Live:** https://goboldlyforward.github.io/surface-edge-prototype/

Sample data only. "Today" is pinned to Oct 26, 2026 so the milestone math stays stable.

## Working on it

- **One static page.** Everything is in `index.html`. Bootstrap 5.3.3 and Bootstrap Icons 1.11.3 are stored in the repo, so there is no build step and no CDN dependency for styles.
- **Run it locally:** `python3 -m http.server 8765`, then open http://127.0.0.1:8765/.
- **GitHub `main` is the source of truth.** Pages deploys from `main` at the root (`.nojekyll` keeps Jekyll out of the way).
- **Pull before editing.** Changes also arrive as pull requests from contributors, so pull `main` before editing and never overwrite it from an older local copy.
- **Reviewing PRs.** PRs branch from the same file, so review them by merging them locally in order and clicking through every route before merging on GitHub.
- **claude.ai preview.** A preview copy of the page is also published as a claude.ai artifact. It is `index.html` without the `<!doctype>`/`<head>`/`<body>` wrapper. Regenerate it from `index.html` after each change so the two never drift.

## Product decisions

### What the system tracks

- **Purchase order → shipments → milestones.** The shipment is the unit of tracking. "Container 1" or "Pallet 1" is just a name the team chooses. The kind is either an ocean container or an LTL pallet.
- **Milestones come from editable lead times** (Constraints page). Change a lead time and every planned date recalculates. Ocean transit by port uses medians from the order history. The other numbers are placeholders until purchasing confirms them.
- **Ship-by dates.** The mill produces to a ship-by date, not to the day it's ordered. A confirmed booking replaces the estimated departure and arrival. A milestone that passes with nothing recorded is flagged, and everything after it moves.
- **Need-by dates can be a date, ASAP, or relative to another shipment** ("six weeks after Container 1"). The customer's requested date is kept next to the date we committed to.
- **Companies hold roles.** Customers and vendors are both companies. Vendor roles are factory, freight forwarder, customs broker, drayage carrier, LTL carrier, and warehouse, and each role owns a lead time. Contacts are people at a company. The primary contact gets standing emails and reminders.

### Order lanes

| Lane | Means |
| --- | --- |
| Intake | Arrived by email or portal; the system has parsed it, matched the customer, and replied |
| Queue | Captured but not actionable: a missing need-by, open paperwork, held, or too early to place |
| Production | At least one shipment is placed with a factory or warehouse |
| Complete | Every shipment is delivered and invoiced |

The lane is worked out from the shipments, not stored. Individual shipments can be held in the queue.

### How work gets done

- **The system proposes, a person approves.** Every next step (place an order, send a notice, chase a document, reply to a thread) is drafted and waits for a click. Nothing goes out on its own. Actions approved almost every time can later be switched to automatic.
- **A checklist runs before the factory order:** account package, credit, deposit, preliminary notice, tax status, signed acknowledgement, and delivery contact. Any item can be overridden, and the override is logged on the order.
- **Change requests live in one place.** Each one has a context (shipment, order, customer, vendor, or contact). The statuses mirror the action names: Approve → Approved, Modify → Modified, Reject → Rejected.
- **Chats replace the inbox.** Emails and portal messages are merged into threads. Open means they spoke last and we owe a reply. Awaiting reply means we spoke last. My inbox is everything assigned to me that isn't closed. Replies are drafted from the order, the change request, and the thread for a person to edit or send.

### Customer portal

- **Sign-in is a link from email.** The portal shows orders in production, in queue, and completed, plus a "What we need from you" list.
- **Customers change dates in place.** Setting or changing a need-by creates a change request for the team, not an email.
- **Lead times can be hidden per customer.**
- **A chat bubble on every page** starts a message or continues an open conversation. It uses the same threads the team works in Chats, and internal system notes are never shown to the customer.

### Dashboard

The dashboard is split by job instead of one long page:

- **Overview** is the glance: summary cards, what needs attention in production, the to-do mix, and a performance table (lifetime, yearly, quarterly, monthly, and weekly columns).
- **To do** is the work queue, most urgent first, typed (Decisions, Place orders, Paperwork, Replies, Exceptions) and owned.
- **Feed** is everything that happened, newest first, filtered by source (system, customers, vendors, milestones, team).

A single feed was considered and rejected. A time-ordered list buries urgent approvals under newer events, so actions, status, and history each get their own view.

## Interface conventions

- **Layout** follows the GoBoldlyForward admin templates: a fixed dark sidebar, a breadcrumb top bar, a page header, and a data table.
- **Sidebar groups** open and close by clicking the row. The section icon becomes a down chevron while open, and the sub-item rail lines up under it. Opening a group also loads its first sub-item, unless you're already on a page inside that group. Sub-items match the page's tabs.
- **Sidebar counts appear only when something needs action.** Red means it blocks production. Yellow means work in progress. Neutral totals are not shown.
- **Index pages:**
  - The header stays the same across tabs, with a one-sentence subtitle.
  - Tabs sit on the left, and search, group by, filter, and sort sit on the right, in one row. More than four tabs roll into a More dropdown.
  - The note under the tabs is one line with an ellipsis and expands on click.
  - Group by, Filter, and Sort are icon buttons that open a menu with a header. A label appears next to the icon only when a non-default option is chosen.
  - Tables fill the viewport so pagination sits near the bottom.
- **Record pages** (order, customer, vendor, contact) share one layout. The header has a back link and actions. A status banner says what matters right now. A tab row has the content tabs on the left and that tab's tools on the right. The active tab fills the main column, and reference details stay in a persistent side column. The first tab lives at the bare record link, and each other tab gets its own link.

  | Record | Tabs | Side column |
  | --- | --- | --- |
  | Order | Shipments, Change requests, Chats, Activity | Checklist, items, source, acknowledgement, attachments |
  | Customer | Orders, Contacts, Change requests, Chats, Activity | Account, notes |
  | Vendor | Shipments, Forecast (factories), Contacts, Change requests, Chats, Activity | Details, notes |
  | Contact | Orders (customer contacts), Change requests, Chats, Activity | Details, others at the company |

  Activity on customer, vendor and contact pages is the Feed filtered to that record, so history reads the same everywhere. Shipments on an order collapse and have their own edit page.
- **Theme** has a top-bar toggle for Auto, Light, and Dark.

## Waiting on

- Confirmed lead times from purchasing. Two things are still unknown: whether the order matrix's production date means finished or scheduled, and how much of the gap before sailing is waiting for a booking.
- Sample PO emails (good, bad, typical, and non-PDF) and a customer list export.
- The ERP replacing the current system in February 2027. Until then the bridge is a CSV export, not an integration.
