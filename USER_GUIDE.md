# MapStack — User Guide

MapStack is a free, collaborative **sitemap builder**. You build a visual page
tree, drag to nest and re-order pages, attach metadata and links, recommend
**resources and CTAs** per page, map out **user journeys** and the **off-site
channels** that funnel people into your site, leave feedback, and collaborate
live with your team.

This guide walks through everything the app does. If you just want to hand an
LLM the rules for generating import-ready files, jump to
[Building files with an LLM](LLM-IMPORT-GUIDE.md).

---

## Contents

1. [Signing in](#signing-in)
2. [The dashboard](#the-dashboard)
3. [The editor at a glance](#the-editor-at-a-glance)
4. [Building your sitemap](#building-your-sitemap)
5. [Page details, links & status](#page-details-links--status)
6. [Page Collections](#page-collections)
7. [Off-site entry points (channels)](#off-site-entry-points-channels)
8. [User journeys](#user-journeys)
9. [Comments & @-mentions](#comments--mentions)
10. [Live collaboration](#live-collaboration)
11. [Sharing & teams](#sharing--teams)
12. [Versions](#versions)
13. [Import & export](#import--export)
14. [AI recommendations](#ai-recommendations)
15. [Keyboard shortcuts](#keyboard-shortcuts)
16. [Notes on availability](#notes-on-availability)

---

## Signing in

- **Sign up / sign in** from the landing card. New accounts currently need an
  approved email domain; sign-in is unaffected.
- **Forgot your password?** Use the link on the sign-in card. You'll get a reset
  email; following the link brings you back to a *set a new password* screen.

Your **display name** (set under the account menu → *My Profile*) is how
teammates see you on cursors, presence avatars, and comments.

---

## The dashboard

The dashboard lists your sitemaps and lets you create new ones.

- **New sitemap** — creates an empty project and drops you into the editor.
- **Open** — click any project to open it. The open sitemap lives in the URL
  (`#/p/<id>`), so refresh, bookmark, and the browser back button all work.
- **Import** (upload icon) — build a brand-new sitemap from a CSV file. See
  [Import & export](#import--export).

---

## The editor at a glance

The editor is a canvas with your page tree laid out as an org chart: the root
and its top-level pages run **horizontally**, and deeper levels stack
**vertically** beneath their parent.

**Topbar (left → right):** brand · project name ▾ · version chip · live page
count.

**Topbar (right cluster):** journeys · AI · comments · import · download ·
share. (Journeys and AI appear only if enabled for your account.)

**Canvas controls (bottom-left):** zoom in/out/fit, then undo/redo below them.

**Panning & zoom:** two-finger scroll pans, pinch zooms. The **Fit** control
frames the whole map.

---

## Building your sitemap

### Add pages

Hover any node to reveal its **+** buttons:

- **+ before / + after** (left and right edges) — add a sibling on either side.
- **+ child** (bottom) — add a page nested underneath.

Every new node starts as an editable *Untitled* page — type its name and press
Enter.

### Node types

A node is one of:

- **Page** — a real page in your site (has a path/URL, status, links).
- **Hierarchy label** — a grouping/section label, not a page. Use these to
  organize the tree without implying a URL.
- **Submap** or **Catalog** — a **Page Collection**: a single node standing
  in for a large group of real pages, kept collapsed on the canvas. See
  [Page Collections](#page-collections).

Set the type in the node's **details panel** (Info tab → *Node type*).

### Re-arrange the tree

- **Drag to nest** — drop a node *onto* another; a green ring shows it will
  become a child.
- **Drag to re-order** — drop a node *beside* a sibling; a blue insertion bar
  shows exactly where it will land.
- **Keyboard re-order** — select a node and press **Shift + Arrow** to move it
  among its siblings (←/→ across horizontal tiers, ↑/↓ within vertical stacks).
  Plain arrow keys are intentionally disabled.

### Collapse, rename, delete

- **Collapse/expand** a branch with the chevron on a parent node (view-only —
  not saved).
- **Rename / Delete** from the pill on the node's top border, or the **⋮** kebab
  (top-right) which opens the details panel.
- Deletes always use an in-app confirm dialog.

### Undo / redo

Every tree edit is undoable. Use the **undo/redo buttons** (bottom-left, below
the zoom controls) or **⌘Z / ⌘⇧Z** (Ctrl on Windows; Ctrl+Y also redoes).
Deleting a whole subtree and undoing restores the entire subtree. History is
in-memory only — reloading or switching projects clears it.

---

## Page details, links & status

Open a node's **details panel** (kebab **⋮**, or double-click behavior via the
Rename pill). It has two tabs, plus two more (feature-flagged — see
[Notes on availability](#notes-on-availability)) for pages:

### Info tab

- **Type** — Page or Hierarchy label.
- **Status** — see below.
- **Label** — the node's title.
- **Extension path** — the URL slug for this page (e.g. `/pricing`). Pages only.
- **Notes** — free-form notes.
- **Links** (pages only) — a **live page URL**, a **Google Doc**, and a
  **Figma** file.
  - The Google Doc and Figma links each have an **⤢ expand** button that opens
    an in-panel **preview**: the panel widens and embeds a read-only view. Edit
    opens the source in a new tab. Both require link-sharing to be on for the
    source file to render.

### Status

Every page carries a status, shown as a colored dot on the node:

| Status | Meaning |
| --- | --- |
| **Complete** | Green — done. |
| **Draft** | Amber — in progress. |
| **Unassigned** | No dot — not yet triaged. |

### Feedback tab

Per-page comments — see [Comments & @-mentions](#comments--mentions).

### Resources tab

Recommended content links for the page — video, white paper, ebook, solution
brief, reference architecture, blog, report, or a custom kind. Pick a **kind**,
add a **title** and **url**; **drag** (or select and press **Shift + ↑/↓**) to
re-order. At least 3 blank rows are always available to fill in.

### CTAs tab

The page's calls-to-action — a **label** and a **url** each. Same
add/reorder interaction as Resources. The **first** CTA is the page's
**primary** one; the rest are secondary — drag a CTA into the first slot to
make it primary.

Both tabs' lists travel with the sitemap CSV (see
[Import & export](#import--export)), and — one layer up — a journey can
re-rank a SUBSET of a page's resources/CTAs just for its own narrative; see
[User journeys](#user-journeys).

---

## Page Collections

For content-heavy sites — a docs library, a blog archive, hundreds of
location pages — showing every page as its own canvas node makes the map
unusable. A **Page Collection** groups them behind one node that stays
collapsed on the canvas, while every page inside it remains a full,
first-class page: its own status, comments, metadata, and journey
participation all keep working exactly as they do for any other page.

A collection comes in two flavors, set from its **Node type**:

### Submap

The default. Opening a Submap (click its chevron) enters a focused canvas
view of just its members — drag, rename, add children, and re-order exactly
as you would on the main canvas. A **breadcrumb** at the top of the editor
always shows your way back (← All sitemaps → project name → each collection
you've entered); click any segment to jump there directly.

### Catalog

For a large collection — hundreds of pages — panning a submap stops being
practical. A **Catalog** skips the canvas view entirely: its chevron opens
straight to a flat, searchable **Pages** tab listing every member (title,
status, journey participation), with **Open** and **Remove** actions per row.

> **Catalog is permanent.** Converting a collection to a Catalog can't be
> undone — it can't convert back to a Page, a Hierarchy label, or a Submap.
> MapStack asks you to confirm before making the switch.

Because a Catalog has no canvas to drag a new page onto, its Pages tab adds
two extra ways to grow it, alongside the usual **+ Add pages** (pick from
pages that already exist elsewhere in the sitemap):

- **+ New page** — adds a blank page as a direct member.
- **Import CSV** — drop a MapStack-format CSV to add many pages at once (see
  [Import & export](#import--export)); the file's own hierarchy is ignored —
  every row becomes a flat, direct member of this one collection.

A page inside a Catalog can't change its own node type (it has to stay an
ordinary page) — but every other field (status, notes, links, resources,
CTAs, comments) works normally. Open any member's own panel from the Pages
tab, and use the **← [Collection name]** link at the top of its panel to get
back.

### Converting a page

Open a page's details panel → **Info** tab → **Node type**, and choose
**Submap** or **Catalog**. Converting a page with existing children turns
them into the collection's members automatically — nothing moves. Converting
a Submap back to a Page or a Hierarchy label asks you to confirm if it still
has members, since they'll become ordinary siblings of the collection on the
canvas (a Catalog can't convert back at all — see above).

### Landing page

A collection can optionally point at one specific member as its **Landing
page** — the one page that best represents the collection as a whole (e.g.
a section's own index page). Set it from the Info tab; the picker only
offers the collection's current members.

---

## Off-site entry points (channels)

Entry points model the **off-site channels** that funnel visitors into your
site — organic search, an LLM citation, paid media, a social post, an in-person
event, email, a referral, or a custom channel. They render in a **lane to the
left** of the sitemap.

- **Add one** — use the entry-points affordance on the canvas; hovering an
  existing entry point reveals a **+** to add another (same interaction as
  sitemap nodes). The canvas auto-fits so the new lane is in view.
- **Edit** — click the **⋮** on an entry point to open its popover: set the
  **label**, pick the **channel** (which sets its color and glyph), and tick one
  or **more landing pages** — the pages this channel's traffic arrives on. A
  dashed connector links the channel to each landing page (the card shows
  “→ First page +N”).
- **Re-order** — drag an entry point within the lane (a drop bar shows the
  position), or select it and press **Shift + ↑/↓**.
- **In a journey** — when a journey starts at a channel, the canvas hides that
  channel's usual landing lines and draws a single line (in the journey's color)
  from the channel to the journey's **next step**, so the path reads
  continuously even if that page isn't one of the channel's landing pages.
- **Close the popover** — click anywhere outside it, or press Esc.
- **Import / export** — entry points travel **with the sitemap CSV**: a sitemap
  export includes them, and importing that CSV recreates them (each channel's
  landing pages re-attach by path, then title). See
  [Import & export](#import--export).

---

## User journeys

A **journey** is an ordered path through your pages — the route you want a
visitor to take. Open the **journeys bar** from the topbar (it drops down above
the canvas).

- **Create** — *+ New journey*. Give it a name (double-click a journey pill to
  rename) and a **color** (swatch → palette).
- **Add steps** — turn on **+ Add steps**, then click nodes on the canvas to
  append them as steps. The active journey highlights its member nodes (accent
  border + a numbered step badge) and dims the rest.
- **Re-order steps** — in the journey flow strip, **drag** a step chip (a drop
  bar shows where it lands) or focus a chip and press **Shift + ←/→**. Click a
  chip to center that page on the canvas; **✕** removes it.
- **Start at a channel** — a journey may *optionally* begin at an off-site
  [entry point](#off-site-entry-points-channels) instead of a page. When it
  does, the channel shows as a dashed step 1.
- **Prioritize resources/CTAs for a step** — click a step chip's **⋮** to open
  a small popover over that page's [Resources/CTAs](#page-details-links--status)
  (feature-flagged). Check the ones that matter for THIS journey and drag
  (or **Shift + ↑/↓**) to rank them — this doesn't touch the page's own
  default order, just this journey's narrative. A dot on the **⋮** shows a
  step already has some prioritized.
- **Import / export** — journeys round-trip as CSV. See
  [Import & export](#import--export).

Collapsing the journeys bar clears the canvas highlights; re-opening restores
the active journey.

---

## Comments & @-mentions

MapStack has two kinds of comments, both sharing the same composer and thread
UI:

- **Page feedback** — the **Feedback tab** in a node's details panel. Each page
  node shows a badge with its count of **unresolved** comments.
- **Sitemap comments** — general comments on the whole map, via the **comment
  icon** in the topbar's right cluster.

In either composer, type **@** to mention a teammate. Mentioned members get an
**email** with a deep link straight to the comment (the node's Feedback tab, or
the sitemap comments panel). You can **edit**, mark **complete/resolved**, or
**delete** your comments from the **⋮** on each one.

### Notification bell

The **bell** in the topbar (left of your name) badges the count of **unread**
@-mentions of you across *all* your sitemaps. Open it to see each mention
(author, snippet, project, time); clicking one marks it read and deep-links you
there. **Mark all read** clears the badge.

---

## Live collaboration

Everything syncs live:

- **Presence** — avatars of everyone currently viewing the map.
- **Cursors** — teammates' cursors move in real time, and show an activity label
  (e.g. while they drag a node).
- **Edits** — page changes, comments, journeys, and entry points all propagate
  instantly to everyone in the project.

---

## Sharing & teams

Open **Share** (topbar right cluster) to control access.

- **Invite people** — type-ahead search by name or email; invited members get
  editor access.
- **Share into a team** — the same search also matches your **teams** (tagged
  *Team*). Sharing a map into a team gives every team member access.
- **Make owner** — the owner can hand ownership to another member (confirm step;
  the previous owner stays on as an editor).

**Teams** (account menu → *Teams*) are named workspaces. Create a team, invite
or remove members, transfer team ownership, or leave. Any map you own can be
shared into a team you belong to.

### Public preview links

Share a **read-only, no-account** view of your sitemap — handy for
stakeholders who don't (yet) have a MapStack account.

- Open **Share** → **Public preview link** → **Create link**, then copy the
  URL. Anyone with the link can view the page tree, node details, and
  existing feedback — no sign-in required.
- Previewers can't edit anything; every editing control is hidden.
- To leave feedback, a previewer is prompted to **create an account** first —
  existing comments are visible, but posting a new one needs sign-in.
- **Disable** pauses the link without losing it; **regenerate** issues a new
  URL and permanently retires the old one.
- Available to any project member, not just the owner.

---

## Versions

A sitemap keeps up to **3 saved versions** (rolling — the oldest non-current one
is pruned when you add a fourth). The **version chip** in the topbar shows the
active one; the project menu (name ▾) lets you:

- **+ New version** — snapshots the current tree into a new version.
- **Switch** — click a version to make it active. Switching is **global**:
  everyone follows the active version live.
- **Delete** — remove a non-current version.
- **Save as** — duplicate the whole sitemap into a **new project** you own.

> Journeys and entry points are tied to a specific version (page ids differ per
> version). Creating a new version does **not** copy journeys.

---

## Import & export

### Export (download menu, topbar right cluster)

- **CSV** — the full page tree, one row per node, **including each page's
  resources/CTAs** (feature-flagged), each **Page Collection** as a
  `Submap`/`Catalog`-typed row (its members collapse to nothing extra — they
  just round-trip as ordinary rows underneath it), and **any off-site entry
  points** (channels) as extra rows (see the
  [format spec](LLM-IMPORT-GUIDE.md#sitemap-csv)).
- **SVG** — a crisp, standalone vector render of the sitemap.
- **Journeys CSV** — one row per journey step (shown when journeys exist).

### Import

- **New sitemap** — the dashboard's upload icon: a sitemap CSV becomes a
  brand-new project (its page resources/CTAs and entry-point rows are
  recreated too).
- **New version of the open map** — the editor's import icon: a sitemap CSV is
  added as a **new version** and made active for everyone (again including any
  page resources/CTAs and entry points in the file).
- **Journeys** — the journeys bar's **Import** pill: a journeys CSV re-attaches
  its steps to the **current version's** pages (matched by path, then title).

The exact CSV shapes — and copy-paste prompts for generating them with an LLM —
are in the **[LLM Import Guide](LLM-IMPORT-GUIDE.md)**.

---

## AI recommendations

When enabled, a **sparkle icon** in the topbar opens the AI panel. It uses
**your own OpenRouter key** (added under *My Profile*), so nothing is billed to
a shared account. Two modes:

- **Optimize** — reviews your sitemap and returns severity-tagged
  recommendations; click one to center the page it refers to.
- **Suggest a journey** — from an optional goal, proposes an ordered journey you
  can preview and create with one click.

---

## Keyboard shortcuts

| Action | Shortcut |
| --- | --- |
| Re-order selected node among siblings | **Shift + ← / → / ↑ / ↓** |
| Re-order selected entry point | **Shift + ↑ / ↓** |
| Re-order focused journey step | **Shift + ← / →** |
| Undo | **⌘Z** (Ctrl+Z on Windows) |
| Redo | **⌘⇧Z** (Ctrl+Shift+Z or Ctrl+Y) |
| Close a panel / popover | **Esc** |

---

## Notes on availability

Some features are gated per-account and only appear if enabled for you:

- **Journeys**, **entry points**, **AI recommendations**, **public preview
  links**, **page Resources/CTAs** (each of the two tabs its own flag), and
  **Page Collections** are feature-flagged.
- **Import/export** is available to everyone.

If you expect a feature and don't see it, it may not be enabled for your
account yet.
