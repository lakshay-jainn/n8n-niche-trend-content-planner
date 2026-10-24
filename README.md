# Niche trend content planner for n8n

This n8n workflow finds the Google searches that are **rising in your niche**, throws out the junk, and turns the rest into content ideas. Each idea comes with the likely reason it's rising, an angle, a hook, a format and **the facts to check before you publish**.

Most "Google Trends to content" workflows use **Trending Now**: whatever is trending nationally today, which is usually sports, celebrities and news. That's useless if you write about skincare, SaaS or home espresso. This one starts from **your own keywords** and uses Google Trends' **rising related queries and related topics** for each of them.

## Three ways to use it

| | Best for | How you give keywords | Where results go |
|---|---|---|---|
| **1. Web form** | One-off checks, sharing with a client or teammate who doesn't use n8n | Type them, or upload a CSV | A results page, right after you submit |
| **2. Google Sheet, every Monday** | A standing weekly plan, any number of niches | One keyword per row in a `Keywords` tab | A row per idea in an `Ideas` tab, plus a summary in Slack or Telegram |
| **3. Run it now** | Trying it out in the editor | Edit the **Test input** node | Same as the sheet |

All three feed the same pipeline, so they behave the same way.

### 1. Web form

Activate the workflow and open the **Form** node's production URL (it ends in `/form/rising-in-my-niche`). The form asks for:

- **Keywords**: one per line or comma-separated
- **Niche** and **Audience**: what you make content about and who it's for
- **Country** and **Time range**
- **Content formats** (blog post, short video, carousel, newsletter, podcast, email)
- **Write ideas with AI**: yes, or just the rising searches report
- **Or upload keywords (CSV)**: for bulk. Columns: `keyword`, and optionally `niche`, `audience`, `country`. Comma or semicolon separated.

A CSV row's `niche` overrides the form's niche, so **one upload can plan several niches at once**. For example:

```csv
keyword,niche
retinol,skincare for beginners
niacinamide,skincare for beginners
espresso machine,home espresso
coffee grinder,home espresso
```

Expect about 1–2 minutes per niche. The results page then shows each niche with its idea cards (or its report table).

### 2. Google Sheet, every Monday

Make a sheet with a tab named `Keywords`:

| keyword | niche | audience | country | active |
|---|---|---|---|---|
| retinol | skincare for beginners | people new to skincare | US | |
| niacinamide | skincare for beginners | | | |
| espresso machine | home espresso | people making coffee at home | | |
| moka pot | home espresso | | | no |

- `keyword` and `niche` are required. Rows without a niche are skipped and listed in the summary.
- `audience`, `country` (two-letter code) and `active` are optional. `active` set to `no` pauses a row.
- Column order and capitalisation don't matter.

Each niche is planned on its own, with its own ideas and its own memory. Point **Read keyword sheet** at the sheet and **Add to Google Sheet** at the same file (tab `Ideas`), then activate the workflow.

### 3. Run it now

Edit **Test input** (keywords, niche, audience, country, AI on or off) and click **Execute workflow**.

## With AI, or without

The AI step changes what you get, so this workflow treats both modes as first-class instead of making AI mandatory.

| | **With AI** (recommended) | **Without AI: rising searches report** |
|---|---|---|
| You get | Ready ideas: title, angle, hook, format, likely reason it's rising, fact-check list | The cleaned list of rising searches: growth, type, variants, which of your keywords it came from, a Google Trends link |
| Relevance | The model scores every search 0–10 for your niche and audience. It catches risers that share no word with your keywords, like a new ingredient or brand name | A search has to share a real word with its keyword or your niche. Simple and predictable, but it can miss those |
| Needs | An API key (Gemini has a free tier) | Nothing beyond Apify |
| Risk | The model can add details that aren't true (prices, specs). Every idea lists what to check, and nothing is published automatically | None from AI. The data is exactly what Google Trends returned |
| Time | About 1–2 minutes per niche | Under a minute per niche |

Why not AI only? Three reasons:

- some people can't or don't want to send data to an AI provider;
- the report is still useful on its own (it's a cleaner Trends than Trends);
- AI calls fail sometimes: a wrong key, rate limits, a provider outage.

So **if the AI call fails, the niche falls back to the report automatically**, with a note saying why ("check the AI credential and model name", "the AI provider is rate limiting"). A Monday run never comes back empty because of the AI.

Turn AI on or off per run: the **Write ideas with AI** question on the form, `useAI` in **Test input**, or `useAIOnSchedule` in **Defaults** for the weekly sheet.

## What a run looks like

Real output from the test runs below (US, past 30 days, Gemini 3.8 Flash), for two niches: "skincare for beginners" and "home espresso".

| Niche | Rising search | Growth | Idea |
|---|---|---|---|
| skincare | is niacinamide good for rosacea | +4,950% | Is Niacinamide Good for Rosacea, or Will It Cause a Flare-Up? |
| skincare | can you use niacinamide with tretinoin | +300% | Can You Use Niacinamide with Tretinoin? How to Layer Them |
| skincare | is adapalene a retinol | +40% | Is Adapalene a Retinol? The Direct Beginner Comparison |
| espresso | breville smart grinder pro | +250% | How to Calibrate the Breville Smart Grinder Pro for Real Espresso |
| espresso | breville espresso machine | +140% | How to Use a Breville Espresso Machine: The 5-Step First Shot Checklist |

A full idea, as it lands in your sheet:

| Column | Example |
|---|---|
| idea | Is Niacinamide Good for Rosacea? What Beginners Need to Know About Percentages |
| angle | Clarify that while niacinamide supports the skin barrier and reduces redness, high concentrations (10%+) frequently trigger flare-ups in rosacea-prone skin. |
| hook | If your face turns beet red every time you use niacinamide, you are probably using the wrong percentage. |
| format | blog post |
| whyRising | Likely driven by people with sensitive, reactive skin seeking soothing ingredients to calm facial flushing and barrier irritation. |
| checkBeforePublishing | Clinical consensus on effective, well-tolerated niacinamide concentrations (typically 2% to 5%) for sensitive or rosacea-prone skin |
| risingSearch · growth · type | is niacinamide good for rosacea · Breakout · search |

Without AI, a niche gets the report instead (home espresso, from the form test):

| Rising search | Growth | Type | Your keyword |
|---|---|---|---|
| Burr Coffee Grinder | +4,450% | search + topic | coffee grinder |
| Varia VS3 Grinder | Breakout | topic (Coffee grinder) | coffee grinder |
| JURA E8 Fully Automatic Espresso Machine | +2,650% | topic | espresso machine |
| Semi-Automatic Espresso Machine | +2,050% | topic | espresso machine |

### The sheet columns

Ideas and report rows use the **same 19 columns**, so one `Ideas` tab holds both:

`date, niche, mode, status, idea, angle, hook, format, whyRising, checkBeforePublishing, risingSearch, growth, type, alsoSearchedAs, newThisWeek, yourKeywords, relevance, googleTrends, note`

- `mode` is `ideas` or `report`.
- `status` starts as `to review`.
- `note` explains a fallback, for example when the AI was unavailable.

## How it picks what's worth writing about

Per niche:

**1. Junk out.** Google Trends data in 2026 is full of what looks like bot and AI-agent traffic. With six skincare keywords, these came back as **Breakout** searches:
- generic ones under unrelated keywords: `how to make sushi`, `what is a 401k`, `best time to visit maldives`, `bread`
- prompt-like ones: `explain what niacinamide is and its main benefits for skin health`

Searches that read like prompts, overly long ones, and generic searches repeated under several unrelated keywords are removed.

**2. Relevance.** The AI score, or the word match without AI (see above). With AI, unrelated candidates are capped at a third of what the model sees, so it spends its attention on likely winners.

**3. Variants merged.** "trader joe's retinol" and "trader joes retinol" become one. A topic and a search naming the same product merge too.

**4. Ranked by growth**, boosted when something rises under several of your keywords or as both a search and a topic.

**5. Spread.** Ideas are spread across your keywords, so one keyword can't take every slot.

**6. Memory (weekly sheet runs).** It never repeats an idea within 8 weeks, and marks what's **new since the last run**. Form and test runs don't touch the memory.

**7. Quiet weeks** get a short "nothing new is rising" line in the summary (and on the results page) instead of silence, naming the keywords Google has no data for. No empty rows are added to your sheet.

All tuning values are commented at the top of the **Clean and rank** node.

## How it works

```
Form ──────────→ From form ───────┐
Every Monday → Read keyword sheet → From sheet ──┼→ Defaults → Build jobs (one per niche, 40 keywords max)
Run it now ────→ Test input → From test input ┘                       │
                                        Loop over jobs: Build Actor input → Google Trends (Apify) → Collect
                                                                      │
                                                               Clean and rank
                                                                      │
                                      Write ideas with AI? ── yes → Prepare AI request → Write content ideas
                                              │ no                    (LLM Chain + Gemini + Structured Output Parser)
                                              │                         ├ ok    → Add AI ideas ─────┐
                                              │                         └ error → AI unavailable: report ┤
                                              └──────────────────────────────────────────────→ Combine niches
                                                                                                    │
                                                         Results → From the form? ── yes → Results page → Show results
                                                                         │ no
                                                                         ├→ One row per idea → Add to Google Sheet
                                                                         └→ Weekly summary → Slack / Telegram
```

Each input has a small adapter that turns it into the same rows: `keyword, niche, audience, country, timeRange, formats, useAI, source`. Empty fields are filled from **Defaults**. To add another source (Airtable, Notion, a webhook), add one adapter that outputs those rows and connect it to **Defaults**. Nothing else changes.

All native nodes: the **Apify** node for Google Trends, a **Basic LLM Chain** with a **Google Gemini Chat Model** and a **Structured Output Parser** (so the AI always returns valid fields), **n8n Form**, **Google Sheets**, **Slack** and **Telegram**.

## Setup (about 5 minutes)

1. **Import** `workflow.json` (Workflows → Import from file, or Import from URL with the raw link to `workflow.json`).
2. **Apify node:** if n8n asks, install it. It's a verified community node; search "Apify" in the nodes panel. On n8n Cloud, the instance owner may need to allow verified community nodes in the Cloud Admin Panel first. Then create a free account at [apify.com](https://apify.com), copy your API token (Settings → API & Integrations), and add an **Apify API** credential in the **Google Trends** node.
3. **AI (recommended):** add a **Google Gemini (PaLM) API** credential in the **Gemini model** node. A free key is available at [aistudio.google.com](https://aistudio.google.com). To use OpenAI, Anthropic, Groq or a local Ollama model, replace the **Gemini model** node with that chat model node.
4. **Pick your inputs:** the form works as soon as the workflow is active. For the weekly plan, set up the sheet (above) and pick it in **Read keyword sheet** and **Add to Google Sheet**. Enable **Send to Slack** or **Send to Telegram** if you want the summary.
5. Optional: adjust **Defaults**.

   | Setting | What it does | Default |
   |---|---|---|
   | `ideasPerNiche` | Ideas (or report rows) per niche | 10 |
   | `minRelevance` | Keep AI ideas scored at least this (0–10) | 6 |
   | `includeTopics` | Also use rising related topics | true |
   | `defaultCountry` | Used when a row has no country. Two-letter code, or empty for worldwide | US |
   | `defaultTimeRange` | `now 7-d`, `today 1-m`, `today 3-m` or `today 12-m` | today 1-m |
   | `defaultFormats` | What you publish, when the input doesn't say | blog post, short video, carousel, newsletter |
   | `useAIOnSchedule` | Write ideas with AI on the weekly sheet run | true |
   | `maxKeywordsPerRun` | A safety cap on keywords per run | 300 |

6. Click **Run it now** to see a result, then **activate** the workflow for the form and the weekly schedule.

## Make it yours

- **Another niche:** add rows to the sheet, or type it in the form. Nothing to edit.
- **Faster or slower niches:** `defaultTimeRange` `now 7-d` for fast-moving topics, `today 3-m` for slow ones (the form has its own picker).
- **Different output:** every idea is one flat row with fixed column names, so Notion, Airtable, email or a webhook can replace Google Sheets in one step.
- **Different idea style:** edit the instructions in **Prepare AI request**, for example video scripts, newsletter sections or product ideas.
- **Stricter or looser filtering:** raise `minRelevance`, or change the commented values at the top of **Clean and rank**.

## Cost

The [Google Trends Actor](https://apify.com/clean-fetch/google-trends-scraper) charges per keyword, only for data it actually delivers:
- rising related queries cost **$0.50** per 1,000 keywords;
- related topics cost **$1.50** per 1,000 keywords;
- keywords Google has no data for are free.

**20 keywords every week comes to about $0.16 a month**, well inside Apify's free monthly credit. With AI, it's one model request per niche per run.

## How it was tested

On n8n 2.44.6 (self-hosted) with the Apify node 0.9.0 and Gemini 3.8 Flash, against live data:

| Test | Result |
|---|---|
| **Web form, AI on**: 2 keywords typed + a CSV with 3 more, two niches | Submitted over HTTP the way the form page does it. Results page after about 2 minutes: both niches, 20 idea cards |
| **Web form, AI off**, same input | Results page in about 50 seconds: both niches, 20 report rows |
| **Weekly sheet**, two niches, plus a paused row and a row without a niche | One Google Trends run per niche, 10 ideas each, 20 sheet rows. The paused row was ignored and the row without a niche was listed as skipped. Later runs skipped the ideas already sent and flagged what was new |
| **AI fails** (invalid key) | The niche fell back to the report, with a note saying to check the AI credential |
| **Quiet week** (keywords Google has no data for) | A "nothing new is rising" summary naming those keywords, and no sheet rows |
| **Without AI**, Run it now | 10 report rows with the same 19 columns |

Google Sheets, Slack and Telegram are standard n8n nodes. In these tests the sheet was read from a stand-in node with the same rows, and the outputs weren't connected to real accounts.

## Honest limits

- Google Trends numbers are **relative**: "+3,600%" means fast growth, not big volume.
- Without AI, the word match can drop a real riser that shares no word with your keywords, such as a new brand name.
- **AI ideas can contain details that aren't true**, like prices or specs. That's why each idea lists what to check, and nothing is published automatically.
- The "already sent" memory belongs to weekly sheet runs on an active workflow; runs from the editor start fresh.
- The form waits for the whole run, about 1–2 minutes per niche with AI. For dozens of niches, use the sheet.

## License

MIT. Disclosure: I built the Google Trends Actor this workflow uses.
