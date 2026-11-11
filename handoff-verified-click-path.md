# Handoff: Wire up the Everpure Artifact API's verified click-path signal

> Handoff spec for a Claude Code session in this repo. Written from the Everpure
> Artifact API side (verified against its live code) — not a prescription for how
> journey-generate itself should change. See "Open questions" below for what this
> session needs to decide.

## Context

The Everpure Artifact API now returns a **verified** (never inferred) click-through
signal between consecutive journey steps — whether the page you just showed the
prospect actually has a real `<a href>` link to the next page you're about to
recommend. This closes a real bug: `journey-generate`'s current `do_action` text
("Click X link") is synthesized locally from a template/LLM-phrasing pass with no
relationship to whether that link actually exists on the page. This spec is about
wiring the real signal in so that text (or a new field alongside it) reflects
ground truth.

**Two separate integration points exist in `supabase/functions/journey-generate/index.ts`,
and both currently discard the data.** This spec covers both. Which one ends up
driving the rendered `do_action` text is a design decision for whoever picks this
up — see "Open questions" below.

---

## What changed on the Artifact API side (verified facts)

**`POST /v1/journeys/build`** — each item in the response's `steps[]` array now has
an optional `navigation` field:

```ts
interface VerifiedNavigation {
  linkVerified: boolean          // true only if a real <a href> resolves preceding-step-page -> this page
  verifiedTargetUrl: string | null  // set only when linkVerified
  suggestedCta: string           // ALWAYS present, honest either way — e.g. "This page links directly to
                                  // '{title}' — safe to render as a click-through." or "Share this resource
                                  // directly — no confirmed on-page link from the previous step."
}
```
`navigation` is absent on the first step (nothing precedes it) and present from the
second step onward. **No request change needed for this endpoint** — the API
computes it server-side automatically by comparing consecutive steps' real crawled
outbound links.

**`POST /v1/artifacts/recommend`** — each item in `recommendations[]` gets the same
`navigation` field, but **only when the request includes** a field that already
existed in the public contract and was simply never sent:
```ts
{
  prospectProfile: { ... },
  journeyContext: { precedingArtifactId: "<uuid of the artifact you just showed this prospect>" },
  options: { limit: 12 }
}
```
Without `journeyContext.precedingArtifactId`, `navigation` is omitted entirely (not
`null` — the key doesn't appear).

Both endpoints' existing fields are unchanged — this is purely additive.

---

## Where this needs to plug in (two integration points, both confirmed by reading the current code)

### 1. `callJourneyBuildApi` (line ~878) → `JourneyBuildStep` (line ~856)

Already calls `/v1/journeys/build` once per generation. Its response parser (line
~942-950) currently keeps only `stage`, `artifact.url`, `artifactType`, `title`
from each step — **`rec.navigation` is sitting right there in `data.steps[]` and is
silently dropped.**

To use it: add `navigation?: VerifiedNavigation` to the `JourneyBuildStep`
interface, capture `rec.navigation as VerifiedNavigation | undefined` in the parse
loop, and thread it wherever `journeyBuildSteps` gets consumed downstream (currently
only used for the `tier0Matches` destination-page bias around line 2283 — that
logic would need to decide whether/how to surface this per-stage, since today it
discards everything from a step except the URL match).

### 2. `callArtifactIntelligenceApi` (line ~776) → `normalizeArtifactMatches` (line ~681) → `ArtifactMatch` (line ~628)

Two gaps here, not one:
- **The request never sends `journeyContext`** (line ~812-827, `body` object) — so
  the API never has anything to compute. Fixing this requires the **preceding
  step's Everpure artifact ID**, which MapStack doesn't currently track —
  `ArtifactMatch.resourceRef` only carries `{title, url, kind}`, no `id`. The real
  API's response (`recommendations[].artifact.id` / `journeys/build`'s
  `steps[].artifact.id`) does include it — it's just not being captured today.
  This needs threading through: capture `id` in `normalizeArtifactMatches`, store
  it as "last used artifact ID" across the per-stage loop, pass it as
  `journeyContext.precedingArtifactId` on the next stage's call.
- **The response parser never reads `rec.navigation`** (line ~703-721) even once
  it starts arriving — add it to `ArtifactMatch` and capture it in the parse loop,
  same shape as above.

---

## Open questions for this session to resolve

I traced the code far enough to find both integration points precisely, but
stopped short of prescribing the fix itself — the `do_action`/narrative pipeline
is genuinely complex (tier0/tier1/tier2 destination-page candidate pools, campaign
branch logic, an LLM phrasing pass with a deterministic fallback via
`base.do_action`) and deserves someone working in this codebase with full context,
not a guess from outside. Concretely, unresolved:

1. **Which of the two integration points should drive the rendered text?**
   `journeys/build`'s navigation is "free" (no request change) but only fires for
   the tier0 destination-page bias path, not the per-stage supporting-content
   picks. `artifacts/recommend`'s navigation is richer (per-recommendation) but
   needs the artifact-ID-threading work above.
2. **Should `navigation.suggestedCta` replace `do_action` outright, or just gate
   it** (e.g., only let the LLM phrasing pass write "click through" language when
   `linkVerified: true`, otherwise force a generic "explore" framing)? The
   honest-fallback string the API returns is meant to be render-ready, but it's
   written in the API's voice, not necessarily JRNY's.
3. **Does the LLM phrasing pass (`phrased?.do` merge, ~line 3358) need a rule
   change** so it never invents click-through language when `linkVerified: false`,
   or is that already implicit if `do_action`'s base value is fixed first?

---

## Verification

- Both endpoints are live now at `EVERPURE_ARTIFACT_API_URL` (same secret already
  in use).
- Quick manual check: call `/v1/journeys/build` with any real
  `prospectProfile`/`targetStages` and confirm `steps[1].navigation` is present
  (steps[0] should have no `navigation` key).
- For `/v1/artifacts/recommend`: call it twice — once with
  `journeyContext.precedingArtifactId` set to a real artifact ID known to link to
  one of the returned candidates, once without — and confirm `navigation` appears
  only in the first case, and `linkVerified` is `true` for the linked candidate,
  `false` for the rest.
- End state: regenerate the "Technical leader / Virtualization" reference journey
  from the bug report and confirm the rendered click-through copy for each step
  now reflects a real verified link (or the honest fallback) rather than an
  invented "Click X link."
