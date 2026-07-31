# Building import-ready files with an LLM

MapStack imports **CSV**. There are two files:

- a **sitemap CSV** — the page tree, each page's optional **resources/CTAs**,
  and (optionally) the off-site **entry points** (channels), all in the *same
  file*; and
- a **journeys CSV** — ordered paths through pages that already exist.

This document is the precise spec for both, plus copy-paste prompts you can hand
to an LLM (ChatGPT, Claude, etc.) so it produces files MapStack accepts on the
first try. Everything here matches MapStack's actual parser.

> **Give an LLM this whole file.** The prompt blocks below are self-contained;
> paste one into your LLM along with your content, and it will emit a valid file.

---

## Contents

- [How import works](#how-import-works)
- [Sitemap CSV](#sitemap-csv)
  - [Pages](#pages)
  - [Page Collection rows](#page-collection-rows)
  - [Page resources & CTAs](#page-resources--ctas)
  - [Entry-point rows (channels)](#entry-point-rows-channels)
- [Journeys CSV](#journeys-csv)
- [Prompt: generate a sitemap CSV](#prompt-generate-a-sitemap-csv)
- [Prompt: generate a journeys CSV](#prompt-generate-a-journeys-csv)
- [Validation checklist](#validation-checklist)

---

## How import works

- **Sitemap CSV → pages (+ resources/CTAs + entry points).** Importing from the
  **dashboard** creates a *new project*; importing from inside an **open
  sitemap** (the editor's import icon) adds everything as a *new version* and
  makes it active. The same file carries the page tree, each page's own
  `Resources`/`CTAs`, **and** the off-site entry points: any row whose `Type`
  is **"Entry point"** becomes a channel, and its optional landing page
  re-attaches to a page by **Path first, then Title**.
- **Journeys CSV → journeys.** Importing (the journeys bar's **Import** pill)
  re-attaches each step to the **current version's** pages, matched by **Path
  first, then Title**. The pages must **already exist** — so import the sitemap
  first, then the journeys that reference it.
- Import never sends ids. Hierarchy, ordering, and links are rebuilt from the
  columns described below.

---

## Sitemap CSV

### Header row

```
Level,Title,Type,Path,Status,Notes,Page URL,Google Doc,Figma,Channel,Landing,Resources,CTAs
```

- Header matching is **case-insensitive and space-insensitive**.
- Only **`Level`** and **`Title`** are required. Every other column is optional
  and may be omitted entirely (drop the column) or left blank per row.
- **`Channel`** is only used by entry-point rows. **`Landing`** is used by
  entry-point rows (a channel's landing pages) AND by `Submap`/`Catalog` rows
  (a collection's optional single landing page) — see both sections below.
  **`Resources`**/**`CTAs`** are only used by ordinary page rows — omit any
  of these columns if you don't need them.
- Accepted **aliases**: `Path` ↔ `Slug`, `Page URL` ↔ `URL`, `Google Doc` ↔
  `Doc`.

### Pages

| Column | Required | Values | Notes |
| --- | --- | --- | --- |
| **Level** | ✅ | integer `0, 1, 2, …` | Tree depth. Top-level pages are `0`. A row's **parent is the nearest row above it with `Level` one less**. Increase by exactly **1** per level of nesting. |
| **Title** | ✅ | text | The page name. Blank becomes `Untitled`. |
| **Type** | — | `Page`, `Hierarchy label`, `Submap`, or `Catalog` | Anything that isn't `Hierarchy label`, `Submap`, `Catalog`, or `Entry point` is a **Page**. Use `Hierarchy label` for grouping/section rows that aren't real pages. `Submap`/`Catalog` mark a **Page Collection** — see [below](#page-collection-rows). Defaults to `Page`. |
| **Path** | — | text, e.g. `/pricing` | The URL slug / extension path. Used to match journey steps and channel landings, so keep it stable and unique. |
| **Status** | — | `complete`, `draft`, `unassigned` | Case-insensitive. Anything unrecognized becomes `unassigned`. |
| **Notes** | — | text | Free-form. May contain commas/newlines if quoted. |
| **Page URL** | — | URL | The live page URL. |
| **Google Doc** | — | URL | A linked Google Doc. |
| **Figma** | — | URL | A linked Figma file. |
| **Landing** | — | text (page Path or Title) | **`Submap`/`Catalog` rows only** — see [below](#page-collection-rows). |
| **Resources** | — | see [below](#page-resources--ctas) | Recommended content links for this page (video, white paper, ebook, …). |
| **CTAs** | — | see [below](#page-resources--ctas) | Calls-to-action for this page; the first is primary. |

### Page Collection rows

A **Page Collection** groups a large set of pages behind one collapsed
canvas node. It's an ORDINARY page row in the tree (same `Level`/nesting
rules as any other row) — its children in the file (the following rows at
`Level + 1`) become its members automatically, nothing else marks them.

| Column | Value |
| --- | --- |
| **Type** | `Submap` — a normal collection, opened as a focused canvas view. Or `Catalog` — for a large collection (hundreds of pages), opened as a flat, searchable list instead. **Catalog is permanent**: once imported/converted, it can't become a Page, Hierarchy label, or Submap. |
| **Landing** | *optional* — one specific member that best represents the collection (its Path, falling back to Title). At most one value — not pipe-separated like an entry point's `Landing`. Resolved against the collection's own members first, so it can't accidentally match a same-titled page in a different collection elsewhere in the file. |

A `Submap`/`Catalog` row's own `Resources`/`CTAs`/links behave exactly like
any other page row's.

**Encoding the hierarchy (important).** There are **no id or parent columns**.
The tree is rebuilt purely from the **`Level` integer + row order**: rows are
read top to bottom, and a row's parent is the closest preceding row whose
`Level` is exactly one smaller. So a child must appear **after** its parent and
use `parent Level + 1`.

```
Home                (Level 0)
  Products          (Level 1)
    Widgets         (Level 2)
    Gadgets         (Level 2)
  Pricing           (Level 1)
About               (Level 0)
```

### Page resources & CTAs

A page row may recommend content assets (**`Resources`**) and calls-to-action
(**`CTAs`**) it should drive. Both columns pack **one or more entries**,
separated by **a real line break inside the quoted cell** (not a pipe, comma,
or any other character). Each entry's own sub-fields are separated by
**`;;`** (two semicolons, not one — a real asset title routinely contains a
literal colon or semicolon, e.g. "State of AI: 2026 Edition", and a single
`;` would silently split it in the wrong place).

A line break inside a double-quoted CSV field is ordinary, valid CSV — a
quoted field is explicitly allowed to span multiple physical lines, and
Excel, Google Sheets, and MapStack's own parser all round-trip it correctly.
(Entries used to be `' | '`-joined instead; that was changed because a real
title can just as easily contain a literal `" | "` itself — an SEO-style
"Keyword Phrase | Brand Name" title is common in the wild — which silently
fractured one entry into two garbage ones on import. A file exported before
this change still imports correctly: MapStack's parser only switches to the
newline-based split when a cell actually contains one, and falls back to the
old pipe-split otherwise.)

| Column | Entry format | Notes |
| --- | --- | --- |
| **Resources** | `Kind;; Title;; URL` | `Kind` is one of the names below (unrecognized/blank → **Other**). |
| **CTAs** | `Label;; URL` | The **first** CTA in the list is the page's primary CTA; the rest are secondary. |

**Resource kinds**

| Name |
| --- |
| Video |
| White paper |
| Ebook |
| Solution brief |
| Reference architecture |
| Blog |
| Report |
| Other |

Example cell values (each block below is ONE quoted CSV cell — the line
break is real, inside the quotes, not a new row):

```
Resources: "White paper;; State of AI: 2026 Edition;; https://example.com/wp
Video;; Overview video;; https://example.com/video"

CTAs:      "Contact us;; https://example.com/contact
Learn more;; https://example.com/learn"
```

> A journey may *optionally* re-rank a SUBSET of a page's resources/CTAs just
> for its own narrative — that per-journey-step override is set in the app
> (from a journey chip's kebab), not in any CSV; only each page's own default
> list round-trips through import/export.

### Entry-point rows (channels)

Off-site **entry points** are the channels that funnel visitors into your site
(organic search, paid media, social, email, an in-person event, an LLM
citation, a referral, or a custom channel). Add them as extra rows **after the
pages**, using these columns:

| Column | Value for an entry-point row |
| --- | --- |
| **Type** | `Entry point` (this is what marks the row as a channel) |
| **Title** | the channel's **label** (e.g. `Google – brand terms`) |
| **Channel** | the channel kind — a name like `Organic search` or the key `organic_search`. Unknown/blank → **Other channel**. Sets the color automatically. |
| **Landing** | *optional* — one or more landing pages this channel drives traffic to, **pipe-separated** (e.g. `/pricing \| /home`). Each is given as its **Path** or **Title** and matched Path-first, then Title. |
| *(all other columns)* | leave blank |

- The channel's color comes from its kind — there's no color column.
- A channel can land on **several pages** — separate them with `|`. Any
  reference that doesn't match a page is simply skipped; the channel still
  imports (with its other/zero landings). Import the sitemap's pages in the same
  file, so landings resolve.

**Channel names**

| Name | Key |
| --- | --- |
| Organic search | `organic_search` |
| LLM citation | `llm_citation` |
| Paid media | `paid_media` |
| Social post | `social` |
| In-person event | `event` |
| Email | `email` |
| Referral | `referral` |
| Other channel | `custom` |

> A journey may *optionally* start at one of these channels, but that link is
> set in the app (in journey build mode), not in any CSV.

### Quoting rules (CSV correctness)

MapStack's parser is RFC-4180-style. To stay safe:

- **Wrap every field in double quotes** (this is what MapStack's own export does).
- Escape a literal double-quote inside a field by **doubling it**: `"` → `""`.
- Use `,` as the delimiter. `CRLF` or `LF` line endings are both fine; a UTF-8
  BOM is tolerated.

### Complete example

Note: the `Home` row below spans two physical lines because its `CTAs` cell
contains a real embedded line break between its two entries — it's still one
row (its `Level`/`Title`/etc. aren't repeated on the second line).

```csv
"Level","Title","Type","Path","Status","Notes","Page URL","Google Doc","Figma","Channel","Landing","Resources","CTAs"
"0","Home","Page","/","complete","Primary landing page","https://example.com/","","","","","White paper;; State of AI: 2026 Edition;; https://example.com/wp","Contact us;; https://example.com/contact
Learn more;; https://example.com/learn"
"1","Products","Hierarchy label","","unassigned","Section grouping","","","","","","",""
"2","Widgets","Page","/products/widgets","draft","","","","","","","",""
"2","Gadgets","Page","/products/gadgets","draft","Launch in Q3","","","","","","",""
"1","Pricing","Page","/pricing","complete","","https://example.com/pricing","","","","","",""
"0","About","Page","/about","complete","","","","","","","",""
"0","White Papers","Catalog","","unassigned","","","","","","overview","",""
"1","State of AI 2026","Page","/resources/state-of-ai","complete","","","","","","","",""
"1","overview","Page","/resources/overview","complete","","","","","","","",""
"","Google – brand terms","Entry point","","","","","","","Organic search","/","",""
"","Q3 launch campaign","Entry point","","","","","","","Paid media","/ | /pricing","",""
"","Newsletter","Entry point","","","","","","","Email","","",""
```

This produces a two-root sitemap (`Home`, `About`) with `Products` grouping
`Widgets`/`Gadgets`, plus a `White Papers` **Catalog** whose two members are
its following `Level 1` rows (`Landing` points at `overview`, one of its own
members — not some other page named "overview" elsewhere), plus three
off-site channels — Google on `Home`, the Q3 campaign on **both** `Home` and
`Pricing`, and the newsletter with no landing. `Home` also carries one
resource and two CTAs (`Contact us` is primary, `Learn more` secondary);
every other page has none.

---

## Journeys CSV

A journeys CSV describes ordered paths **through pages that already exist** in
the sitemap you're importing into.

### Header row

```
Journey,Color,Step,Title,Path
```

- Only **`Journey`** and **`Title`** are required.

### Columns

| Column | Required | Values | Notes |
| --- | --- | --- | --- |
| **Journey** | ✅ | text | The journey name. **All rows with the same name form one journey.** |
| **Color** | — | hex, e.g. `#38bdf8` | 3–8 hex digits after `#`. Only the **first** row of each journey is read for color; invalid/blank falls back to a default. |
| **Step** | — | integer `1, 2, 3, …` | Order within the journey. If omitted, rows keep their file order. |
| **Title** | ✅ | text | Must match a page's title in the target version (fallback match). |
| **Path** | — | text, e.g. `/pricing` | The page's path. **Matched first**, before Title — the reliable key. |

### How steps are matched to pages

Each step is resolved against the **current version's** pages by **Path**
(exact, trimmed), then **Title** (exact, trimmed). A step that matches **no**
page is **skipped**; a journey where **every** step fails to match is **not
created**. So prefer `Path`, and make `Path`/`Title` exactly match the sitemap.

### Complete example

```csv
"Journey","Color","Step","Title","Path"
"Buyer path","#22a06b","1","Home","/"
"Buyer path","#22a06b","2","Pricing","/pricing"
"Buyer path","#22a06b","3","Widgets","/products/widgets"
"Research path","#7c5cff","1","Home","/"
"Research path","#7c5cff","2","About","/about"
```

---

## Prompt: generate a sitemap CSV

Paste this into your LLM, then describe the site you want (or paste existing
content, a URL list, a spec, etc.):

```text
You are generating a CSV file that will be imported into MapStack, a sitemap
builder. Output ONLY the CSV — no prose, no code fences.

Rules:
- First line is exactly this header:
  Level,Title,Type,Path,Status,Notes,Page URL,Google Doc,Figma,Channel,Landing,Resources,CTAs
- Wrap EVERY field in double quotes. Escape any double quote inside a field by
  doubling it ("").
- PAGE rows: Level is the tree depth as an integer (top-level = 0; a child uses
  parent Level + 1; a row's parent is the nearest row above with Level exactly
  one less, so list parents before children). Type is "Page" for real pages or
  "Hierarchy label" for section/grouping rows. Path is the URL slug (e.g.
  /pricing), unique. Status is one of: complete, draft, unassigned. Notes,
  Page URL, Google Doc, Figma are optional. Leave Channel blank on page rows.
- OPTIONAL page collections: for a large group of similar pages (e.g. dozens of
  location pages, a docs library), use Type "Submap" (opens as a focused canvas
  view) or "Catalog" (opens as a flat searchable list — use this for a LARGE
  group, hundreds of pages) instead of "Page". Its children in the file (the
  following rows at Level + 1) automatically become its members — nothing else
  marks them. You MAY put one member's Path (or Title) in Landing to mark it as
  the collection's representative page. Leave Landing blank otherwise.
- OPTIONAL page resources/CTAs: a page row may carry recommended-content links
  in Resources and calls-to-action in CTAs. Resources entries look like
  "Kind;; Title;; URL" (Kind is one of: Video, White paper, Ebook, Solution
  brief, Reference architecture, Blog, Report, Other); CTAs entries look like
  "Label;; URL". Separate multiple entries with an ACTUAL LINE BREAK inside
  the quoted cell (press Enter within the quotes) — not a pipe or any other
  character; a properly double-quoted CSV field is allowed to span multiple
  physical lines, and this is still one cell in one row, not a new CSV row.
  The FIRST CTA is primary, the rest secondary. Leave both blank if a page has
  none.
- OPTIONAL off-site channels: after the page rows, you MAY add entry-point rows.
  For each, set Type to "Entry point", put the label in Title, the channel in
  Channel (one of: Organic search, LLM citation, Paid media, Social post,
  In-person event, Email, Referral, Other channel), and OPTIONALLY the landing
  page's path (or title) in Landing — for several landing pages, separate them
  with a pipe (e.g. "/ | /pricing"). Leave Level and the page columns blank.
- Do not invent id or parent columns. Hierarchy comes only from Level + order.

Now build the sitemap for: <DESCRIBE YOUR SITE HERE>
```

---

## Prompt: generate a journeys CSV

Do this **after** the sitemap exists. The easiest source of exact titles/paths is
the sitemap itself: in MapStack, open **Download → CSV** and paste that file in.

```text
You are generating a CSV file of user journeys to import into MapStack. Output
ONLY the CSV — no prose, no code fences.

Rules:
- First line is exactly this header: Journey,Color,Step,Title,Path
- Wrap EVERY field in double quotes; escape inner quotes by doubling them ("").
- Each journey is a set of rows sharing the same "Journey" name.
- Step is an integer starting at 1 giving the order of pages in that journey.
- Color is a hex value like #38bdf8. Use the SAME color on every row of a given
  journey; the first row's color wins.
- Title and Path must EXACTLY match pages that already exist in the sitemap.
  Prefer Path as the match key (matched before Title). A step that matches no
  page is dropped, and a journey with zero matching steps is not created.

Here is the current sitemap CSV, exported from MapStack (use the Title and Path
of its PAGE rows; ignore any row whose Type is "Entry point"):
<PASTE THE DOWNLOADED SITEMAP CSV HERE>

Now build these journeys: <DESCRIBE THE JOURNEYS YOU WANT>
```

---

## Validation checklist

Before importing, confirm the file:

**Sitemap CSV**
- [ ] First line is the header (at least `Level,Title`; add `Channel,Landing` if
      you include entry-point rows, `Resources,CTAs` if you include page
      resources/CTAs).
- [ ] Page rows: `Level` is an integer, children come after parents, nesting
      increases by 1; `Type` is `Page`/`Hierarchy label`/`Submap`/`Catalog`;
      `Status` is `complete`/`draft`/`unassigned` (or blank).
- [ ] Page Collection rows (`Type` = `Submap`/`Catalog`): the rows immediately
      following at `Level + 1` become its members; an optional `Landing`
      value (Path or Title, not pipe-separated) names one of those members.
- [ ] Page resources/CTAs: each `Resources`/`CTAs` entry is `;;`-separated
      (`Kind;; Title;; URL` / `Label;; URL`), multiple entries separated by a
      real line break inside the quoted cell (not `|` — that's the legacy
      format, still accepted on import but no longer what MapStack itself
      exports); an unrecognized `Kind` becomes `Other`.
- [ ] Entry-point rows: `Type` is `Entry point`, `Channel` is a known name/key
      (or left blank → `custom`), and each `Landing` (pipe-separated for several)
      matches a page (unmatched ones are skipped).
- [ ] Fields are double-quoted; inner quotes doubled.

**Journeys CSV**
- [ ] Header is `Journey,Color,Step,Title,Path` (at least `Journey,Title`).
- [ ] Every `Path`/`Title` matches an existing page in the target version.
- [ ] `Step` numbers order each journey; `Color` is a valid `#hex`, consistent
      within each journey.
- [ ] The sitemap was imported **first**.
```
