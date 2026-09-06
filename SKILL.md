---
name: gsc-seo-agents
description: "Run click gap, content decay, or striking-distance analysis on a site's Google Search Console data and deliver the findings as a Google Doc. Use when asked for a click gap report, CTR audit, decay check, or content expansion brief."
---

# Search Console analysis agents

Three analyses over one data source. Ask which one is wanted if the request is ambiguous; "check the SEO" usually means the click gap report.

| Analysis | Question it answers | Data needed |
|---|---|---|
| Click gap | Which pages rank well but nobody clicks? | One period, queries + pages |
| Content decay | Which pages are quietly losing clicks while still ranking? | Two consecutive periods of equal length, pages |
| Depth scan | Which page 2 pages are worth expanding, and with what? | One period, queries + pages |

## Step 1 — Get the data

Ask the user to export: Search Console → Performance → set the date range → Export → Google Sheets, then paste the link. It takes them about thirty seconds and always works. If a CSV or Sheet was already attached or named, use that instead.

Never read numbers off a screenshot of the Search Console UI. Export or ask.

Confirm which property was used. `sc-domain:` and URL-prefix properties can hold different data for the same site.

**Read the real date window from the export.** The UI defaults to 3 months, not 28 days; the Dates or Filter tab states the actual range. Read it, do not assume, and quote it in the output. The thresholds below assume a 28-day window — on a 3-month export multiply impression floors by about three, on a 7-day one divide.

**Read the other tabs too.** Devices and Countries take one line each and often explain the finding: 5.9% mobile CTR against 1.7% on desktop is a snippet problem that shows up mostly on phones, and that belongs in the report.

## Step 1b — Pair queries to pages

The UI export puts queries and pages in separate tabs, so there are no query-by-page pairs. Reconstruct them for the rows you are about to act on — not for the whole tab, which is wasted effort and where most of the error comes from.

Propose pairings by meaning, then verify them by arithmetic. Do this in one pass; do not fan out subagents to cross-check each other, because two guesses about the same row cannot adjudicate each other. The page totals can.

1. **Normalise both sides.** Lowercase, strip accents, split the URL slug on hyphens and slashes into tokens. Do the same to the query.
2. **Score by token overlap**, allowing for morphology — "gipszkartonos" and "gipszkarton-szerelo" share a stem even though no token matches exactly. This is the part that needs language judgement rather than string equality.
3. **Verify against the Pages tab, in code.** A query cannot have more clicks than the page it is assigned to, or more impressions. The queries assigned to one page cannot sum past that page's totals. Reject any pairing that breaks this and re-examine it.
4. **Label every pairing** confident, probable, or unattributed. Report the label. Never present a guess as a fact, and never silently drop a row that could not be paired — list it as unattributed with its numbers.
5. **Generic and brand queries go to the home page** unless a deeper page matches better.
6. **When two pages both plausibly serve one query, that is a finding, not a failure.** It is the cannibalisation signal. Report both candidates and say that a page-filtered export would confirm it: in Search Console, filter by that page, then export its queries.

If the site is large enough that most rows come back merely probable, say so and ask for page-filtered exports of the few pages that matter rather than reporting weak attributions.

## Step 2 — Filter before analysing

Filter in code, not by eye, and state the thresholds used in the output. Character counts and percentages must be computed, never estimated.

**Click gap:** position 1–5, impressions above a floor. Start at 500 for a 28-day window on a site with real traffic, scaled to the window as above; if that returns nothing, lower it until five to fifteen rows survive, and say which floor was used. Never report "no findings" without having tried a lower floor.

Rows at position 5–6 with unusually high volume are worth including as borderline, labelled as such. The largest impression source on a site often sits just outside the band, and dropping it silently is the wrong call.

**Content decay:** compare the two periods per URL. Flag a page only if clicks fell 15% or more, average position held (moved less than 2 places), and the earlier period had at least 100 clicks. A page that dropped 5+ positions is a ranking problem, not decay — exclude it and say so.

**Depth scan:** position 6–20, high impressions, commercial or evaluative intent. Take the top 3. Skip branded, login, contact, PDF and tag pages.

## Step 3 — Analyse

### Click gap

For each page whose CTR is too low for its position — roughly 10–25% expected at positions 1–3, 4–8% at 4–5 — give:

- a two-sentence diagnosis of why the snippet is not earning the click
- 3 title options, max 60 characters
- 3 description options, max 155 characters
- the reason each option should work

Write the metadata in the language of the query, not the language of the conversation.

Do not flag: queries Google answers on the results page (definitions, conversions, simple facts), or searches for a different company by name. A 0% CTR on a competitor's brand name is correct behaviour, not a problem.

List what was considered and not flagged, with the reason, in a short table. It is how the reader comes to trust the ones that were.

### Content decay

For each flagged page, name the most likely cause and give 2–4 concrete actions. The four causes:

1. **Freshness** — dates, statistics, examples or year-based intent gone stale
2. **Competitor leapfrog** — someone published something better, or the SERP intent moved
3. **Cannibalisation** — another page on the same site now competes for the query
4. **Page experience** — layout, ads, load time, thin content above the fold

Order pages by severity.

### Depth scan

For each of the three pages: fetch the target page and its top 3 competitors for the target query (WebSearch then WebFetch, or Claude in Chrome). Compare on topical completeness, format match, and questions competitors answer that the page does not. Then produce a content expansion brief:

- exact H2/H3 headings to insert
- what each section must cover — concepts, entities, related terms
- where on the page each edit goes
- 1–2 high-intent FAQs and the angle to answer them from
- one visual or interactive asset worth adding

Do not write the page. The output is a brief for a human writer.

## Step 4 — Deliver

Write the findings to a Google Doc through the Drive connector, titled `<Analysis> – <domain> – <date>`, and give the user the link. Use tables for the metadata options with the reasoning as its own column — it is the part people actually read.

Open the document with the window analysed, the thresholds used, and how page attribution was established. A reader who cannot see those cannot judge the findings.

If the user asks for a rolling log instead, append to their existing doc rather than creating a new one.

## Guardrails

- Suggestions are drafts. Metadata that promises a price, stock, delivery time or working condition the page cannot back up costs more in bounces than it gains in clicks. Say this once in the output, not in every row.
- Report what the data supports. If only three rows clear the threshold, report three; do not pad the list to look thorough.
- Log the date changes were made, so the next run has something to compare against.
- Character counts on titles and descriptions must be real. Count them.