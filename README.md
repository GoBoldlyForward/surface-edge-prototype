# Surface Edge Order Desk

Clickable prototype of an order-tracking back office and customer portal for Surface Edge, the large-project division of Wanke Cascade. Purchase orders arrive by email, get broken into shipments, move through factory, ocean, port and delivery milestones, and close when every shipment is delivered and invoiced.

**Live:** https://goboldlyforward.github.io/surface-edge-prototype/

Sample data only. "Today" is pinned to Oct 26, 2026 so the milestone math stays stable.

## Working on it

- **One page per section, one script.** Every sidebar section is a folder with its own `index.html`: the dashboard is the root `index.html`, then `todo/`, `feed/`, `orders/`, `chats/`, `vendors/`, `customers/`, `contacts/`, `changes/`, `constraints/` and `portal/`. The pages are thin shells that load `app.css`, `app.js` and the partials. All data, views and behavior live in `app.js`. Bootstrap 5.3.3 and Bootstrap Icons 1.11.3 are stored in the repo, so there is no build step and no CDN dependency for styles.
- **Routes.** The section is the path and the view inside it is the hash: `/orders/#po-4510`, `/customers/#customer-ilg-chats`. Pinpoint files pins by pathname, which is why each section needs its own path. A link written as `href="#route"` works from anywhere: when the route belongs to another section, the click (or an old `/#po-4510` link) is sent to that section's page. Links in shared chrome use `href(route)` so the real URL shows on hover.
- **Partials.** `partials/sidebar.html` is fetched into the `<div data-partial="sidebar">` placeholder on each page. To add a partial, drop a file in `partials/` and a placeholder in the pages that need it. The top bar and breadcrumbs are built by `topbar()` in `app.js`. To add a section, copy a folder, set its `data-section` and `<title>`, and add it to `SECTIONS` and `sectionOf()` in `app.js`.
- **State between pages.** Each page is a full load, so edits to the sample data (approvals, replies, lead times, who you are viewing as) are kept in `sessionStorage` and merged back by id. "Reset sample data" in the user menu clears them; so does closing the tab.
- **Run it locally:** `python3 -m http.server 8765`, then open http://127.0.0.1:8765/. The partials are fetched, so open it over http, not as a file.
- **GitHub `main` is the source of truth.** Pages deploys from `main` at the root (`.nojekyll` keeps Jekyll out of the way).
- **Pull before editing.** Changes also arrive as pull requests from contributors, so pull `main` before editing and never overwrite it from an older local copy.
- **Reviewing PRs.** Most PRs touch `app.js`, so review them by merging them locally in order and clicking through every section before merging on GitHub.
- **claude.ai preview.** A preview copy of the old single page was published as a claude.ai artifact. It predates the split into sections and Pinpoint does not run there, so feedback belongs on the live site.

## Product decisions

### What the system tracks

- **Purchase order → shipments → milestones.** The shipment is the unit of tracking. "Container 1" or "Pallet 1" is just a name the team chooses. The kind is either an ocean container or an LTL pallet.
- **Milestones come from editable lead times** (Constraints page). Change a lead time and every planned date recalculates. Ocean transit by port uses medians from the order history. The other numbers are placeholders until purchasing confirms them.
- **Ship-by dates.** Ship-by is the date the container departs the origin port to make the need-by (Ryan, Oct 28), and the mill finishes production ahead of it. Purchasing sets it to the need-by minus 28 days for West Coast ports and 35 for Gulf and East Coast. A confirmed booking replaces the estimated departure and arrival. A milestone that passes with nothing recorded is flagged, and everything after it moves.
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
- **Chats replace the inbox.** Emails and portal messages are merged into threads. Open means they spoke last and we owe a reply. Awaiting reply means we spoke last. My inbox is everything assigned to me that isn't closed. A reply can be drafted from the order, the change request and the thread with one click. It is never pre-filled: the Chats page counts how often Draft a reply is clicked and whether the draft is sent as written, edited or discarded, so the team can decide later whether drafting earns its place.
- **Open order status email.** Annalisa's report, kept as the format everyone already reads: one row per open shipment with job, PO and order numbers, product and spec, quantity and cartons, destination, placement date, the PO's need-by next to the current need-by, ETD, ETA to the US port and to the destination, and a notes log the system writes from the order record. It goes to the customer's primary contact on the cadence in Constraints (weekly) and asks them to confirm the dates; a reply that moves a date becomes a change request. The team copy adds internal references. Preview, download and send it from the customer page.
- **Delivery appointments.** The site contact's notice rule lives on the shipment (24 hours and an appointment for TCC sites, 48 hours for ILG). When the carrier cannot reach the site contact, the system proposes a notice to the customer flagged "Action required" and dispatch holds until a window is confirmed.
- **Recorded, not typed.** Milestones and steps that came from vendor emails and portals carry a robot icon naming the source. A click is only for a decision or for something that happens outside the system.

### Customer portal

Second priority. Alan (Oct 29) wants the open order status email before customer system access, so the portal stays in the prototype but the demo leads with the report.

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
- The production deadline purchasing uses ahead of the ship-by (departure) date.
- Who owns the delivery appointment, the carrier or Surface Edge, and how long a no-answer waits before it escalates.
- The ERP replacing the current system in February 2027. Until then the bridge is a CSV export, not an integration.
