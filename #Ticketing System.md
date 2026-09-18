# Ticketing System

When community theatres look for "open source" systems, they usually need to solve two distinct needs without heavy SaaS fees: **box office/event ticketing** (seating maps, patron payment, e-tickets) and **show/adjudication evaluation** (adjudicator scoring, play reading feedback, or production reviews).

True open-source options specifically designed for theatre are rare due to compliance overhead like PCI-DSS payment security. However, several dedicated self-hosted solutions, highly customizable open-source frameworks, and budget-friendly community alternatives exist.

---

### Open Source & Self-Hosted Ticketing Systems

**1. osConcert**

* **Type:** Dedicated Open-Source / Self-Hosted Box Office System
* **Best For:** Community venues needing interactive seat maps without per-ticket fees.
* **Key Features:** Reserved seating maps, general admission, e-ticketing, box office POS integration, and payment processing via Stripe or PayPal.
* **Why it fits:** It was built specifically for performing arts venues and open-air/community theatres wanting full control over patron data and zero ticket-commission overhead.

**2. Odoo (Community Edition) + Event / POS Modules**

* **Type:** Open-Source ERP Suite (LGPLv3)
* **Best For:** Theatres that want an all-in-one system for ticketing, volunteer tracking, and donor management.
* **Key Features:** Event registration, portal for patrons, point-of-sale for front-of-house, built-in CRM for donor tracking.
* **Why it fits:** Completely free to self-host; highly modular if you have a volunteer developer to configure seating or ticket printing integrations.

**3. WordPress + Open-Source Ticketing Plugins**

* **Type:** Open-Source CMS Integration
* **Best For:** Small venues wanting a seamless experience inside their existing website.
* **Key Plugins:**
* **FooEvents** or **Event Espresso:** Offers general admission, seating charts, and QR code scanning apps.
* **WooCommerce:** Handles payments, discount codes for subscribers, and patron accounts.



---

### Open Source Evaluation & Feedback Systems

Community theatres generally use evaluation systems for two purposes: **script reading/selection committees** and **adjudication/award scoring**.

**1. LimeSurvey**

* **Type:** Open-Source Survey & Evaluation Platform (GPL)
* **Best For:** Play-reading committees, judge/adjudicator evaluation forms, and post-show audience feedback.
* **Key Features:** Complex matrix scoring (e.g., grading directing, acting, sound, lighting on customized rubrics), anonymous submission, detailed reporting, and role-based permissions.
* **Why it fits:** Extremely flexible for creating structured rubrics for play selections or adjudications.

**2. Formbricks**

* **Type:** Open-Source Experience & Feedback Platform (AGPLv3)
* **Best For:** Audience reviews, show ratings, and modern patron feedback collection.
* **Key Features:** In-app or link-based surveys, segmentation, clean dashboard for evaluating viewer responses after a performance run.

---

### Low-Cost / Free Alternatives (Non-Open Source)

If self-hosting or managing code updates is too resource-intensive for your volunteer staff, these popular non-open-source platforms cater directly to non-profit community theatres:

| Platform | Best Used For | Pricing Model | Key Features |
| --- | --- | --- | --- |
| **Ludus** | All-in-one Community Theatre | Free for venue (passes low fee to buyer) | Seating charts, ticketing, concessions, class registrations, volunteer management. |
| **TicketSource** | General Box Office | Free for free events / Low fee for paid | Interactive seat designer, front-of-house scanner apps, donor add-ons. |
| **Castter / Submittable** | Script / Audition Evaluation | Free tier or low monthly fee | Managing play-reading submissions and judge rubrics. |

---

# Recommender

> **See also:** [`RecommendationPath.md`](RecommendationPath.md) — evaluation of connecting the **current** Arts People system to an external recommender via a periodic extract. That document covers the data-access constraints (no public API, unverified scheduled export) which this section, written from a greenfield/self-hosted angle, does not address.

Off-the-shelf, open-source ticketing platforms rarely ship with built-in machine learning recommender engines. Because live events suffer from a "cold start" problem (shows have short runs and go dark quickly), traditional recommendation algorithms often struggle out of the box.

However, you can easily pair an open-source ticketing system with an open-source recommendation microservice or plug-in module.

---

### Plug-and-Play Recommender Engines (To Pair with Ticketing)

Instead of building ML models from scratch, these standalone open-source engines connect via REST APIs to your ticketing database to suggest shows, cross-sell seating, or personalize newsletter recommendations.

**1. Gorse**

* **Type:** Open-Source AI Recommendation Engine (Go/Apache 2.0)
* **How it works:** You send user interactions (e.g., ticket purchases, show page views) to Gorse via REST API. It automatically trains models—combining collaborative filtering with item category matching—and feeds real-time "Recommended Shows for You" back to your front-end.
* **Best for:** Self-hosters using systems like WooCommerce or custom Node/Python ticketing front-ends.

**~~2. PredictionIO (Apache)~~ — RETIRED, do not use**

* **Status:** Became an Apache Top Level Project in October 2017, was **retired in September 2020**, and the move to the Apache Attic completed in **April 2021**.
* **Why it matters:** The project is read-only archive material. It receives no security fixes, no dependency updates, and no maintenance. Deploying it against patron data would be irresponsible.
* **Historical note:** It was a machine learning server built on Spark and HBase with pre-built templates for item recommendation and similar-event discovery — but that capability is now unmaintained.
* **Rule for future candidates:** Before adopting any Apache-branded engine, check <https://attic.apache.org/projects.html>. Several ML projects have been retired since PredictionIO.

> ⚠️ Do not treat this as a live option. Any recommendation of PredictionIO for ticketing is out of date.

---

### Open-Source CMS & E-Commerce Integration Options

If you prefer an all-in-one web framework rather than running a separate microservice:

**1. WooCommerce + Merchandising Options**

* **Framework:** WordPress (GPL)
* **How to use it:** Pair a ticketing plugin (like *Event Tickets Plus* or *FooEvents*) with WooCommerce's merchandising features. Note that these are **two distinct mechanisms**, covered separately below.

**1a. Rule-based cross-sells and upsells (available today, no ML)**

* **How it works:** WooCommerce's built-in cross-sells and upsells are configured **manually** — you explicitly attach related products to a show, and they appear on the product page, in the cart, or at checkout.
* **Recommendation logic:** Deterministic. Equivalent to "pair tickets for *The Crucible* with our talkback evening."
* **Good for:** The predictable, high-value pairings — concession vouchers, opening night receptions, merchandise, season flex passes, donations at checkout.
* **No training data required.** This is configuration, not recommendation.

**1b. Algorithmic behavioural recommendations (requires additional tooling)**

* **What it needs:** Genuine behavioural recommendation in WordPress means either a plugin that computes co-occurrence over order history, or an external engine (such as Gorse) fed from the WooCommerce database.
* **Recommendation logic:** "Patrons who bought tickets to *The Crucible* also bought tickets to..." — derived from actual purchase data rather than author-configured rules.

> ⚠️ **Do not confuse view counters with recommenders.** *Wp-PostViews* counts page views; it does not generate recommendations. Likewise, "related posts" plugins such as YARPP operate on **content taxonomy** (tags/categories), not purchase behaviour — which is genuinely useful, but is not behavioural personalisation.

> 💡 **Reality check:** Because live events suffer from the cold-start problem described above, taxonomy-based similarity (genre, era, family-friendly tags) is often *more* reliable for community theatre than collaborative filtering over sparse purchase data. Manual cross-sells plus good tagging will frequently outperform a model that has little to learn from.

**2. Drupal + Recommender API Module**

* **Framework:** Drupal CMS (GPL)
* **How to use it:** Drupal has long-standing modules in this space (such as *Recommender API* and *Computing Framework*) that provide hooks for recommendation algorithms over activity data stored within the site.
* **Best for:** Performing arts centers with complex patron profiles, subscription packages, and news/blog content alongside ticket sales.
* **⚠️ Verify before relying on this:** These modules date from the Drupal 7 era, and *Recommender API* is an abstraction layer rather than a working engine — it exposes hooks for algorithms, it does not ship a trained model. Check module currency against your target Drupal release; Drupal 10/11 support is the deciding factor. This is the weakest option in this list.

---

### How Recommendation Logic Applies to Theatre

When configuring an engine for community theatre, standard "frequent buyer" algorithms usually fail because show inventories rotate quickly. The effective approaches focus on three specific strategies:

| Recommendation Strategy | Data Point Used | Example Use Case |
| --- | --- | --- |
| **Content / Genre Matching** | Tagging (e.g., *Musical, Comedy, Shakespeare*) | *"If you liked our spring musical, you'll love this summer cabaret."* |
| **Subscription Upselling** | Patron Tier & Frequency | Inviting 2x/year single-ticket buyers to buy a 4-show flex pass. |
| **Cross-Selling Add-Ons** | Cart Contents | Recommending concession vouchers, opening night reception passes, or merchandise at checkout. |

---
