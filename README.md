# click-gap-agent

A Claude skill that reads Google Search Console data and returns SEO findings as a Google Doc. Used for CirclePOS SEO work.

Three analyses over one export:

| Analysis | Question it answers |
|---|---|
| Click gap | Which pages rank well but nobody clicks? |
| Content decay | Which pages are quietly losing clicks while their ranking holds? |
| Depth scan | Which page-two pages are worth expanding, and with what? |

## What it produces

For the click gap — the one used most — each flagged page gets a two-sentence diagnosis, three title options (max 60 characters), three description options (max 155), and the reason each should work. Metadata is written in the language of the query, not the language of the conversation. Character counts are computed, not estimated.

It drafts. It does not publish, and it does not know what is true about the site: any claim about price, stock, delivery or working conditions has to be checked before it goes live.

## Data

The skill asks for a Search Console export (Performance → Export → Google Sheets). That export splits queries and pages into separate tabs, so a query cannot be traced to a page directly. The skill reconstructs the pairing from URL slugs and verifies it against the page totals — a query cannot have more clicks or impressions than the page it is assigned to.

Measured on hollandmunkak.hu against API ground truth: **38 of 40 correct (95%)**, and **21 of 21 correct** among pairings it labelled "confident". The two misses were trade synonyms where the query word appears nowhere in the URL.

For genuinely joined query-by-page rows, export from a Looker Studio report built on the Search Console connector's **URL Impression** table, which carries Landing Page and Query on the same row. That also makes cannibalisation visible — the same query ranking on two pages — which the standard export cannot show at all.

## Thresholds

Defaults assume a 28-day window; scale the impression floor to the window actually exported.

- **Click gap** — position 1–5, impressions above a floor (start at 500, lower until 5–15 rows survive). Rows at position 5–6 with unusually high volume are included as borderline.
- **Content decay** — clicks down 15% or more, average position within 2 places, 100+ clicks in the earlier period.
- **Depth scan** — position 6–20, high impressions, commercial or evaluative intent, top 3 only.

## Install

Copy `SKILL.md` into a skill named `gsc-seo-agents` in Claude. It uses the Google Drive connector to write the output document, and optionally web search for the depth scan's competitor comparison.

## Provenance

The three-agent structure follows Matt Diggity's "I Let AI Agents Run My SEO" (August 2026), originally built in Make with Apify. This version drops the orchestration layer: the analysis runs in Claude directly, and the only external dependency is a Search Console export.

Prior art on the cannibalisation side, all of it API-based:

- https://github.com/seoffensive/cannibalization
- https://github.com/jmelm93/seo_cannibalization_analysis
- https://github.com/allanreda/SEO-Keyword-Cannibalization-Detector
