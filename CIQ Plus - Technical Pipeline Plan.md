# CIQ Plus — Technical Pipeline Plan

**How to turn the Canberra IQ back catalogue and daily feed into paid topic deep dives, using Airtable + n8n + Claude.**

Prepared June 2026. Based on two decisions you've locked in:

- **You review every report before it publishes** (Claude drafts, you approve).
- **Reports live as standalone web pages, sold one at a time** (pay-per-report), independent of the main Canberra IQ site.

---

## The big idea in one picture

The pipeline is just an automated version of what was done by hand to build the Aged Care sample:

1. **Get the news items into a tidy table** (one row per item: date, topic tags, summary, source link).
2. **Pick a topic** and pull every item about it.
3. **Let Claude write the story** — the narrative, the key milestones, the headline facts.
4. **You read and tweak it**, then approve.
5. **A template turns it into the web page** you saw, and publishes it.
6. **A payment button sells it**, and a teaser goes out to Canberra IQ readers.

The important design choice: **Claude only does the writing and judgement.** All the dates, links and the 600-item library come straight from your real data. Claude never invents a source. That's what keeps a paid product trustworthy.

---

## Part 1 — The data foundation (Airtable)

Everything starts with clean data. You need two tables.

### Table A — "Bulletin Items" (your raw material)

One row per news item. This is the structured version of the bulletins:

| Field | Example | Notes |
|---|---|---|
| Date | 2025-11-02 | Real date, sortable |
| Edition | Fourth Edition | Optional |
| Topics | AGED CARE, REGULATION | The CAPS tags from the bulletin |
| Summary | "Minister Rae marked the start of the Aged Care Act 2024." | Canberra IQ's one-liner |
| Source URL | https://… | The primary-source link |
| Themes | Rules & Legislation | Added automatically (see Part 3) |

Two ways items get here:

- **Back catalogue (one-time):** your `all-bulletins 2025.txt` (and prior years) gets parsed into rows and imported. I've already written and tested the parser that does this — it turned your 2025 file into **30,751 clean rows**, **600** of them on aged care. This is a one-off job I can run for you and hand you a ready-to-import file.
- **Daily feed (ongoing):** your live updates already land in Airtable. We just make sure each new item carries the same fields (date, topics, summary, link). If the live feed already has these, you're done; if not, a small n8n step tidies them on the way in.

> **One thing to watch:** Airtable caps records per base (50,000 on the Team plan). You're adding roughly 120 items/day ≈ 44,000/year, so a single base holds about a year. The fix is simple — archive older years into a second base, or move the full history into a cheaper store and keep only the recent rolling window in Airtable. Not urgent, but worth knowing before you import five years of catalogue.

### Table B — "Reports" (one row per deep dive)

This is your control panel. One row = one topic report.

| Field | Purpose |
|---|---|
| Topic name | "Aged Care Reform 2025" |
| Filter rule | Which items belong (e.g. tag = AGED CARE, year = 2025) |
| Status | Draft → In review → Published |
| Draft content | Claude's narrative, milestones, facts (filled by n8n) |
| Approved? | Checkbox you tick |
| Published URL | The live page link |
| Price / payment link | For selling it |

You'll mostly live in this table: create a row, click "generate", review, approve.

---

## Part 2 — Picking a topic and gathering the items

To start, keep topic selection **manual** — you know which topics your readers care about. You create a Reports row and set the filter (e.g. "all items tagged AGED CARE in 2025").

n8n then pulls every matching item from Bulletin Items. That might be 50 items or 600 — the pipeline handles either.

(Later, the system can *suggest* hot topics by spotting tags with rising volume — the activity chart in the sample is exactly that signal. Not needed for v1.)

---

## Part 3 — Generation (n8n + Claude), done the cost-smart way

This is the engine. It runs in n8n in a few steps:

**Step 1 — Tag the themes (free, no AI).** A small code step sorts each item into buckets (Support at Home, Workforce & Pay, Rules & Legislation, etc.) using keyword matching. This is instant and costs nothing — it's how the sample's filterable library was built. No need to spend AI on 600 items.

**Step 2 — Build a compact digest.** n8n makes a date-ordered list of just the summaries and dates. A whole year of one topic is only ~15,000 words — small enough to hand to Claude in one go.

**Step 3 — One Claude call does the writing.** n8n sends that digest to the Claude API with an instruction like: *"Here is every 2025 item on this topic, in date order. Write a four-paragraph plain-language story of how it unfolded; pick the 10–15 turning-point milestones with their exact dates from this list; and give five headline facts. Return it as structured data."* Claude returns the narrative, milestones and facts.

**Step 4 — Re-attach the real links.** n8n matches each milestone Claude chose back to its original row in Bulletin Items, pulling the **real** source URL. This guarantees every link is genuine — Claude picks *which* moments matter; your data supplies the *facts and links*.

**Step 5 — Save the draft** back into the Reports row for you to review.

> **Cost:** one Claude call per report, on a modest amount of text — a few dollars at most per report, versus whatever you charge for it. (Worth confirming current Anthropic API pricing when you set this up, as rates change.)

---

## Part 4 — Your review

You open the Reports row (or a tidy Airtable "Interface" screen) and read Claude's draft: the four-paragraph story, the milestone list, the facts. You edit any wording, drop or add a milestone, then tick **Approved**.

This is the quality gate that makes the product safe to charge policy professionals for. It takes minutes, not hours, because the heavy lifting is done.

---

## Part 5 — Publish the web page

When you approve, n8n:

1. **Merges your approved content + the full item library into the HTML template** (the exact one behind the Aged Care sample — it's reusable, you just swap in the data).
2. **Uploads the finished page to a web host.** Strong recommendation: **use the AWS S3 setup Canberra IQ already has** (your assets already live on `assets.canberraiq.com.au`, which is S3). n8n can drop the HTML file straight there and you get a public link instantly — no new tools, no servers. Cloudflare Pages or Netlify are equally fine free alternatives if you'd rather keep it separate.
3. **Optionally makes a PDF** of the same page, as a second format buyers can download.

Each report gets its own permanent URL.

---

## Part 6 — Sell it (pay-per-report)

The cleanest no-code setup for selling standalone pages:

- **A free public teaser page** — the "story in brief" plus a few milestones. This is your marketing and SEO asset; anyone can read it.
- **An "Unlock the full deep dive — $X" button** that goes to a checkout.
- **On payment, the buyer is sent the full report** (the complete page's private link + the PDF).

For the checkout, I'd suggest a **merchant-of-record** tool like **Lemon Squeezy** or **Paddle** over raw Stripe, for one practical reason: they handle sales tax/GST and send the delivery email automatically. As an Australian selling digital products, that saves you real admin. (Plain Stripe Payment Links also work and are slightly cheaper, but you handle tax and fulfilment yourself. Worth a quick chat with your accountant on GST — not my area.)

Gating a static page isn't Fort Knox — the full URL is unguessable but not login-protected. For a niche B2B audience paying for current research, that's a normal and acceptable trade-off for v1. A proper login can come later if piracy ever becomes a real problem (it won't early on).

**Driving sales:** when a report publishes, an n8n or Zapier step posts a one-line teaser into the daily Canberra IQ bulletin and email — *"New deep dive: Aged Care Reform 2025 →"*. Your existing readership is the launch audience.

---

## n8n vs Zapier — which does what

- **n8n** is the engine: the multi-step generate → review → publish flow, the Claude call, the S3 upload. It's built for exactly this.
- **Zapier** is optional glue for simple one-step jobs (e.g. "payment received → send Slack ping", or "report published → add to newsletter"). You can do these in n8n too; use whichever you find easier.

---

## Suggested build order

Don't build it all at once. Prove demand first, automate second.

**Phase 0 — Data ready (days).** Confirm the Bulletin Items fields; I run the catalogue parser and hand you an import file. Set up the Reports table.

**Phase 1 — Manual MVP (this week).** Generate *one* report semi-by-hand (I can do the generation for you), publish it to S3, put one payment link on it, and post a teaser to your list. Goal: **see if people actually pay.** No automation needed yet.

**Phase 2 — The n8n generator.** Build the generate → save-draft flow so you can create a report draft with one click and review it.

**Phase 3 — Auto-publish + selling.** Wire approval → publish to S3 → create the payment product → push the teaser. Now it's a repeatable machine.

**Phase 4 — Scale & polish.** Topic suggestions from trending tags, "this report was updated" refreshes when big news lands, a small library/index page of all your deep dives.

---

## What I can do next

- **Run the back-catalogue parser** and give you a clean Airtable-ready import file (I've already built and tested it).
- **Build the first n8n workflow** for you (I have direct access to your n8n) — the generate-and-draft step is the natural starting point.
- **Set up the Airtable tables** with the right fields.
- **Make the HTML template reusable** so any topic's data drops straight in.

Tell me which to start with.
