# Recommender Inputs — Item Metadata and the Audience Rating Signal

**Status:** Feasibility / input assessment
**Date:** 2026-09-25
**Scope:** The two *inputs* a recommender needs — item metadata (tags) and explicit audience
rating — plus a reframing of the highest-value use case at this venue's scale.
**Companion to:** [`RecommendationPath.md`](RecommendationPath.md) (data extraction and engine
selection). This document assumes that document's conclusion: the engine is a commodity and the
extraction path is the risk.

---

## 1. Why inputs are the binding constraint

The engine decision is reversible and cheap to defer. The *input* decision is neither. A
recommender fed with purchases only and no item tags produces generic popularity rankings; the
same engine fed with a good tag vocabulary and an explicit rating stream produces something a
theatre can actually act on and explain to a patron.

There are exactly two inputs to secure:

| Input | Source | Status |
|---|---|---|
| **Item metadata** — genre, form, era, tone | Wikidata (partial), local curation | Free but only partly usable — §3 |
| **Explicit rating** — did they like it | Post-show email, attendees only | Mechanism exists; details unverified — §4 |

Neither comes from a purchased database. That is the finding of this document.

---

## 2. Venue scale and the revised density argument

The venue runs **~300 performances a year across 2 venues and 4 stages**.

This corrects an earlier, weaker assumption in `RecommendationPath.md` §6 that the programme was
small enough for sparsity to be a non-issue. It is not. But the performance count does not settle
the question either, because the recommender's item catalogue is **productions, not performances**.
300 performances is roughly 15 productions at 20 performances each, or 40 productions at 7–8 each
— and those are materially different problems.

**Productions per year is the single most important unconfirmed number in this project.**

Modelled at ~120 seats and a ~70% house:

| Productions/year | Performances each | Tickets/year | Unique patrons (~3 tickets each) | Purchase-matrix density |
|---|---|---|---|---|
| 15 | 20 | ~25,000 | ~8,000 | ~13% |
| 20 | 15 | ~25,000 | ~8,000 | ~10% |
| 40 | 7–8 | ~25,000 | ~8,000 | ~5% |

**Consequences:**

- At 10–13% density over 15–20 items, collaborative filtering is thin but **no longer hopeless**.
  It becomes a defensible majority-or-minority signal from **year 2–3**.
- At 40 items and ~5% density, item-to-item co-occurrence still works within a season, but the
  catalogue churns fast enough that content tags must carry the early results.
- Either way the per-patron signal accumulates at only ~3–5 events/year, so **content/tag affinity
  and item quality carry years 1–2**; personalisation compounds behind them.

This strengthens the case for Gorse relative to `RecommendationPath.md` §8, which was written
under the smaller assumption. The step-1 co-occurrence proof still stands — it is cheap and it
tests the extract — but a real engine is more clearly justified by year 2 at this volume.

---

## 3. Item metadata — is there a free "movie database" for plays and musicals?

The question splits in two, and the halves have different answers.

- **Item metadata: partly yes, and it is free.** [Wikidata](https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service)
  is CC0-licensed, exposes a [SPARQL endpoint](https://query.wikidata.org/sparql) returning JSON,
  and covers theatrical works.
- **Ratings: no.** There is no open corpus of audience ratings for stage productions. MovieLens and
  IMDb have no theatre counterpart. (Goodreads carries ratings for published plays, but its public
  API was retired.) The rating half of a MovieLens-style exercise has **no external source** — it
  can only come from the theatre's own audience, which is what §4 is about.

### 3.1 What Wikidata is good for, and what it is not

**Good for:** canonical title, author/composer, premiere year, and play-vs-musical *form*. As a
title normaliser across seasons and spellings, it is genuinely useful automation.

**Bad for:** the taste tags that actually drive recommendations. Wikidata's `P136` (genre) property
is polluted by **adaptation genres** — a film or cast-recording adaptation inherits its genre onto
the underlying work.

### 3.2 Probe results (2026-09-25)

Fourteen titles typical of a community-theatre programme were matched against Wikidata by exact
label, in two passes. This was a quick probe, not an audit; a production system would use the
search API with review and would do better on canonical titles.

**Pass 1 — naive exact-label match.** 14/14 returned *something*, but the matches were
disambiguation pages, albums, films, and in one case a Python package. Genres returned were film
and music genres — *Sweeney Todd* resolved to "glam rock".

**Pass 2 — filtered to theatrical `instance of` types.**

| Measure | Result |
|---|---|
| Titles matched to a theatrical work | 12/14 |
| Titles carrying any genre tag (`P136`) | **4/14** |
| Of those, genres actually usable for recommendation | a minority |

Examples of the failure mode: *Into the Woods* → "folk rock, hard rock"; *Mamma Mia!* → "K-pop,
Korean ballad, electropop"; *A Christmas Carol* → "Christmas film". Two bread-and-butter
community-theatre titles — *The Play That Goes Wrong* and *Silent Sky* — had **no theatrical item
at all** under those labels. *The Crucible* resolved to an entity typed as a "dramatico-musical
work", evidently an adaptation, illustrating the merge/disambiguation risk.

### 3.3 Other sources assessed

| Source | Verdict |
|---|---|
| **Wikidata** | ✅ Free (CC0), usable for identity and form. Unreliable for genre. |
| **IMDb metadata** | ❌ No longer a free dataset — now licensed commercially via [AWS Data Exchange](https://data.imdb.com/). Film/TV only in any case. |
| **TMDB** | ❌ Film/TV; commercial use requires a paid agreement. |
| **Rights-holder catalogues** (Concord Theatricals, MTI, Dramatists Play Service) | ⚠️ The industry's *real* taxonomy — genre, cast size, tone, suitability — but no open API, and ToS restricts scraping. **Manual reference only, not an import.** |
| **Open Library / Library of Congress** | ⚠️ Useful for published play scripts (editions, authors). Metadata only; no genre/tone. |
| **MusicBrainz** | ⚠️ Relevant to musicals' cast recordings and composers, not to the stage works themselves. Licence is mixed — verify before commercial use. |

### 3.4 Recommendation

Build a **local curated vocabulary of roughly 20–40 tags** — form (musical/play), genre
(comedy/drama/thriller), era, family-suitability, local-vs-touring, studio-vs-mainstage —
**seeded from Wikidata and completed by hand**. Use Wikidata to normalise titles and confirm form;
do not expect it to supply taste tags.

The manual tagging requirement flagged in `RecommendationPath.md` §6 therefore stands, now
confirmed empirically rather than assumed. Note the parallel: title disambiguation in Wikidata is
the same class of problem as patron identity resolution — both are entity resolution, and both are
the real work.

---

## 4. The audience rating signal

The theatre already sends an email to ticket holders after the performance, apparently only to
those who attended. **This is the most valuable element of the whole plan** — more valuable than
the metadata question.

### 4.1 What it solves

- **It converts implicit to explicit.** Purchase data cannot distinguish "bought it because my
  spouse wanted to go" from "loved it." A 1–5 rating can.
- **It is attendance-conditioned.** Because it only reaches people who came, it doubles as the
  attendance signal `RecommendationPath.md` §6 identifies as the strongest one — and it answers the
  "do we have scan data?" question without needing scan data.
- **It supplies item quality**, which no purchased or public dataset provides.

### 4.2 What remains unverified — and decides everything

1. **Does the survey link carry the patron or ticket ID?** An anonymous response is usable only as
   aggregate item quality and is useless for personalisation.
2. **Where do responses land** — inside Arts People, or in an external survey tool?
3. **Are responses exportable with the patron ID attached?** If ratings cannot be joined to the
   purchase record, the input is half-lost.
4. **Do historical responses exist?** If these emails have been running for years, there may already
   be a rating corpus in hand — potentially years of the scarcest data in the project. **Mine this
   before building anything.**

### 4.3 Bias and interpretation

**Self-selection is severe.** Respondents skew toward enthusiasts, regulars, and the digitally
engaged, so averages run high and are not a random sample.

- Use ratings for **relative ranking, spread, and segment discovery**.
- Do **not** present them as absolute satisfaction figures — particularly not to funders.
- A 4.6 average is nearly meaningless; "this production split the audience" is a finding.
- Ratings arrive post-performance, so a single weak night contaminates a production's average.
  Acceptable noise at production level; a feature if performance-level analysis is ever wanted.

### 4.4 Survey design

Keep it short — every additional question costs response rate:

1. one 1–5 rating,
2. one "would you recommend" (NPS-style),
3. optional free text,
4. optionally one question about **genre appetite**, which is what builds a durable taste profile
   across seasons, unlike the item-level rating.

Send within 24 hours; response decays quickly.

### 4.5 Accumulation horizon — calibrate expectations

| Signal | Reaches usefulness | Notes |
|---|---|---|
| Item quality (per production) | **1–2 seasons** | ~125–190 ratings per production at this volume |
| Genre affinity (per patron) | **1–2 seasons** | The durable, compounding asset |
| Per-user collaborative filtering | **4–7 years** | Patrons generate only ~3–5 events/year |

A MovieLens-style per-user model is therefore a multi-year horizon, not a first-season deliverable.
The compounding asset is a **tagged patron taste profile**, not a trained model.

### 4.6 Privacy

Ratings tied to a patron are personal data. Fine for the organisation's own use, but they must be
covered by the privacy notice, and the position changes if a third party processes the data or the
client resells the result (see `RecommendationPath.md` §7).

---

## 5. The demand-smoothing reframe

At 300 performances and ~6 performances a week, the highest-value application is **not** "what
should I see next season." It is **in-run demand smoothing** — deciding which under-selling
performance to put in front of which patron this week. That is where the money is, and it is a
stronger business case than a newsletter recommendation block.

**Architectural consequence:** recommendations must be **filtered against live seat inventory at
send time**. Items remain productions (per `RecommendationPath.md` §6), but the performance
calendar and remaining capacity become *item attributes*.

| | Season-announcement use | In-run demand smoothing |
|---|---|---|
| Trigger | Season launch / renewal | Weekly, continuous |
| Data | Prior-season affinity | Affinity + live availability |
| Value | Subscription upsell | Filling soft performances |
| Cadence | Seasonal batch | Weekly batch, joined to live inventory at send |

Also relevant at this scale: with 2 venues and 4 stages, **venue and stage affinity** is a real
signal (mainstage vs studio), and should be part of the tag vocabulary in §3.4.

---

## 6. What this changes in `RecommendationPath.md`

| Item | Change |
|---|---|
| §6 density framing | Corrected — sparsity is real at ~300 performances/year; CF defensible from year 2–3, not year 1. Conditional on productions/year. |
| §8 co-occurrence-first | **Unchanged and still correct** — cheap, and it tests the extract. |
| §5 Gorse recommendation | Strengthened — a real engine is more clearly justified by year 2 at this volume. |
| §6 manual tagging | Confirmed empirically (§3), not merely assumed. |
| New requirement | Availability-aware filtering at send time (§5). |

---

## 7. Questions to take to the theatre manager

**Scale — determines whether collaborative filtering can ever work**

1. How many **productions** per year, and how many performances does each typically get?
2. Is programming **rolling** through the year, or a fixed season with a dark period?
3. Roughly how many **unique patrons** per year, and what share buy more than one production?
4. Do we hold **attendance/scan** data, or sales only?

**The rating signal — the decisive operational questions**

5. Does the post-show email link carry the **patron or ticket ID**?
6. Where do responses land — Arts People, or a separate survey tool?
7. Can responses be **exported joined to the patron record**?
8. **How far back do historical responses go?**
9. What exactly does the email currently ask, and what is the response rate?

**Data access and commercials**

10. Is there a **scheduled report or feed** out of Arts People, or only manual downloads?
11. Is the ~$99/month figure an actual quote, and does the theatre already license **Neon CRM**?

**Delivery**

12. Who owns the patron **email list** and the send? Is there a named person who would act on
    recommendations?
13. What is the primary goal — cross-sell existing patrons, win back lapsed ones, or fill soft
    performances mid-run?

---

## 8. References

- Wikidata Query Service (SPARQL endpoint): <https://query.wikidata.org/sparql>
- Wikidata SPARQL service documentation: <https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service>
- Wikidata licensing (CC0 for structured data): <https://www.wikidata.org/wiki/Wikidata:Data_access>
- IMDb metadata licensing (commercial, AWS Data Exchange): <https://data.imdb.com/>
- Neon CRM developer documentation: <https://developer.neoncrm.com/getting-started/>
- Neon CRM authentication (API key + Org ID): <https://developer.neoncrm.com/authentication/>
- Gorse recommender engine: <https://github.com/gorse-io/gorse>
