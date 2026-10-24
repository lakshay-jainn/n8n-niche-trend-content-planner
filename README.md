# Niche trend content planner for n8n

This n8n workflow finds the Google searches that are **rising in your niche**, throws out the junk, and turns the rest into content ideas. Each idea comes with the likely reason it's rising, an angle, a hook, a format and **the facts to check before you publish**.

Most "Google Trends to content" workflows use **Trending Now**: whatever is trending nationally today, which is usually sports, celebrities and news. That's useless if you write about skincare, SaaS or home espresso. This one starts from **your niche** and uses Google Trends' **rising related queries and related topics** around it.

## Who it's for, and what they type in

Two people touch this workflow, and the inputs are shaped around each.

- **The owner** imports it into n8n, adds the Apify and AI credentials, and picks the settings once in **Defaults**: AI on or off, ideas per niche, default country. Often that's a creator doing it for themselves; often it's a freelancer setting it up for a client.
- **The user** fills in the form or the sheet: a creator, marketer or small business owner who wants to know what to post this week. They know their niche and where they publish. They usually don't know Google Trends, and they shouldn't have to.

So the form starts from what the person wants, in plain words, asks only what they can answer straight away, and requires only the first question. The button says **Get my content ideas**.

| Question | Why it's asked |
|---|---|
| **What do you want to make content about?** (required) | The niche, in their own words: "skincare for beginners", "home espresso", "money tips for students" |
| **Any specific topics you want to cover?** | Optional: "retinol, acne, morning routine". Leave it empty and AI finds topics (see below) |
| **Who do you want to reach?** | Optional. Makes the ideas specific instead of generic |
| **Where do you want to post?** | Blog, YouTube, Shorts / Reels / TikTok, Instagram carousel, Newsletter, LinkedIn, Podcast, X. The ideas use these formats |
| **Where do you want to reach people?** | A country, or Worldwide. Empty uses the owner's default |
| **How recent should the trends be?** | Last 30 days (recommended), last 7 days for fast-moving topics, last 3 months for slower niches, last 12 months |
| **Want ideas for several niches at once? Upload a CSV** | For agencies and bulk: columns `niche`, `keyword`, `audience`, `country` |

Whether AI writes the ideas is the owner's choice (`useAI` in **Defaults**), not a question on the form. The person filling it in shouldn't have to know whether there's an AI key behind it.

## How keywords work (you don't have to guess what's booming)

A keyword here is a **seed**: a core topic of your niche that people search all the time, like "retinol" or "espresso machine". It doesn't need to be rising itself. For every seed, Google Trends reports the searches that **are** rising around it, measured from real search data: "can you use niacinamide with tretinoin" +300%, "halloween latte art" Breakout. Those are the booming terms, and they come from Google's data, not from anyone's guess.

So the only choice is the seeds, and good seeds are the obvious, concrete things a niche is made of: products, brands, ingredients, tools, common problems. Descriptions of the niche don't work, because nobody types them into Google. In a test with real data:

| Seeds for "AI for product management" | Rising searches found |
|---|---|
| Descriptions: "ai prd", "ai product roadmap", "generative ai product"… | 4 |
| Five hand-picked: "ai agents", "llm evals", "prompt engineering"… | 53 |
| Eight concrete ones: "ai agents", "cursor", "vibe coding", "claude"… | 108 |
| The niche name alone | 0 |

That's why:
- **With AI**, a niche with fewer than 3 keywords (`suggestKeywordsBelow`) gets 8 suggestions (`suggestedKeywords`), and the AI is told to follow those rules. Keywords Google has no data for cost nothing.
- **Without AI**, a niche with no keywords is searched as written, with a note asking for 3–5 keywords.
- **Every result shows the topics it looked at** and how many rising searches each one found, for example `coffee grinder (50), espresso machine (27)`. Suggested topics are marked so people can keep the ones they like, topics with no data are crossed out, and the weekly summary names the ones that found little. After a week or two you know which to keep.

## Three ways to use it

Each one is labelled on the canvas (**Way 1 · Web form**, **Way 2 · Google Sheet**, **Way 3 · Test run**).

| | Best for | What you give it | Where results go |
|---|---|---|---|
| **1. Web form** | One-off checks, or a link for a client who doesn't use n8n | Your niche; keywords optional; a CSV for several niches | A results page, right after you submit |
| **2. Google Sheet, every Monday** | A standing weekly plan, any number of niches | One row per keyword (or just a niche) in a `Keywords` tab | A row per idea in an `Ideas` tab, plus a summary in Slack or Telegram |
| **3. Run it now** | Trying it out in the editor | Edit the **Test input** node | Same as the sheet |

All three feed the same pipeline, so they behave the same way.

### 1. Web form

Activate the workflow and open the **Form** node's production URL (it ends in `/form/rising-in-my-niche`). Fill in the niche and submit; everything else is optional.

For several niches at once, upload a CSV. A row with a niche and no keyword means "suggest keywords for this niche":

```csv
niche,keyword
skincare for beginners,retinol
skincare for beginners,niacinamide
home espresso,espresso machine
home espresso,coffee grinder
sun care,
```

Expect about 1.5–2 minutes per niche with AI, under a minute without. The results page shows each niche with its keywords, then its idea cards (or its report table).

### 2. Google Sheet, every Monday

Make a sheet with a tab named `Keywords`:

| niche | keyword | audience | country | formats | active |
|---|---|---|---|---|---|
| skincare for beginners | retinol | people new to skincare | US | | |
| skincare for beginners | niacinamide | | | | |
| home espresso | espresso machine | people making coffee at home | | youtube, blog | |
| home espresso | moka pot | | | | no |
| sun care | | | | | |

- `niche` is required. Rows without one are skipped and listed in the summary.
- `keyword` is optional: a niche with fewer than 3 keywords gets AI suggestions (when AI is on).
- `audience`, `country` (two-letter code), `formats` (where you publish) and `active` are optional. `active` set to `no` pauses a row.
- Column order and capitalisation don't matter.

Each niche is planned on its own, with its own ideas and its own memory. Point **Read keyword sheet** at the sheet and **Add to Google Sheet** at the same file (tab `Ideas`), then activate the workflow.

### 3. Run it now

Edit **Test input** (niche, keywords, audience, country) and click **Execute workflow**.

## With AI, or without

The AI step changes what you get, so this workflow treats both modes as first-class instead of making AI mandatory.

| | **With AI** (recommended) | **Without AI: rising searches report** |
|---|---|---|
| You get | Ready ideas: title, angle, hook, format, likely reason it's rising, fact-check list. Keywords suggested when a niche has few | The cleaned list of rising searches: growth, type, variants, which of your keywords it came from, a Google Trends link |
| Relevance | The model scores every search 0–10 for your niche and audience. It catches risers that share no word with your keywords, like a new ingredient or brand name, and drops homonyms ("German Unity Day" under *unity*) | A search has to share a real word with your niche name or keywords. Simple and predictable, but it misses those and lets homonyms through |
| Needs | An API key (Gemini has a free tier) | Nothing beyond Apify |
| Risk | The model can get facts wrong, so the prompt keeps them out of titles and hooks unless they're on the fact-check list, bans made-up personal experience, and words every "why" as a likely reason. Nothing is published automatically | None from AI. The data is exactly what Google Trends returned |
| Time | About 1.5–2 minutes per niche | Under a minute per niche |

Why not AI only? Three reasons:

- some people can't or don't want to send data to an AI provider;
- the report is still useful on its own (it's a cleaner Trends than Trends);
- AI calls fail sometimes: a wrong key, rate limits, a provider outage.

So each AI call (keyword suggestions and ideas) is **retried up to 3 times**, and **if it still fails, the run carries on**: keyword suggestions fall back to the keywords you gave (or the niche name), and ideas fall back to the report. Each comes with a note saying why ("check the AI credential and model name", "the AI account is out of credit or quota", "the AI provider is rate limiting"). A Monday run never comes back empty because of the AI.

AI is on or off for the whole workflow: `useAI` in **Defaults**. It's the owner's setting, so it isn't asked on the form.

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

Removed: searches that read like prompts (they start with "explain", "describe", "list"…, or contain phrases like "and its main benefits"), "searches" longer than 15 words, and generic searches repeated under several unrelated keywords. Real long questions stay, like "what is a high yield savings account and how does it work", and so do paper and product titles.

**2. Relevance.**
- **With AI**, the model scores each search 0–10 with a fixed scale: a specific question, product, rule or story for your audience scores high; a broad name on its own ("Claude", "Steam", "Serum") scores 5 or less, even if the model can imagine a good angle for it. A search that's just one of your keywords again is capped at 5 in code, whatever the model says.
- **Without AI**, a search has to share a real word with your niche name or one of its keywords.

**3. Variants merged.** "trader joe's retinol" and "trader joes retinol" become one. A topic and a search naming the same product merge too. With AI, several searches about the same story or product type (four walking-pad brands, three headlines about one lawsuit) become one idea that covers them all.

**4. Ranked by growth**, boosted when something rises under several of your keywords or as both a search and a topic. A search that just repeats your keyword ranks lower.

**5. Spread.** Ideas are spread across your keywords, so one keyword can't take every slot.

**6. Memory (weekly sheet runs).** It never repeats an idea within 8 weeks, and marks what's **new since the last run**. Form and test runs don't touch the memory.

**7. Quiet weeks** get a short "nothing new is rising" line in the summary (and on the results page) instead of silence, naming the keywords Google has no data for. No empty rows are added to your sheet.

All tuning values are commented at the top of the **Clean and rank** node.

## How it works

```
Form ──────────→ From form ───────┐
Every Monday → Read keyword sheet → From sheet ──┼→ Defaults → Group by niche
Run it now ────→ Test input → From test input ┘                       │
                    Suggest keywords? ── yes (AI on, < 3 keywords) → Ask for keyword ideas → Suggest keywords
                            │ no                                        ├ ok    → Add suggested keywords ─┐
                            │                                           └ error → Keep given keywords ────┤
                            └─────────────────────────────────────────────────────────────→ All niches → Build jobs
                                                                      │ (one per niche, 40 keywords max)
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

Each input has a small adapter that turns it into the same rows: `keyword, niche, audience, country, timeRange, formats, useAI, source`. An empty keyword means "suggest keywords for this niche"; other empty fields are filled from **Defaults**. One **Gemini model** node serves both AI steps, so there is one AI credential to set up. To add another source (Airtable, Notion, a webhook), add one adapter that outputs those rows and connect it to **Defaults**. Nothing else changes.

All native nodes: the **Apify** node for Google Trends, a **Basic LLM Chain** with a **Google Gemini Chat Model** and a **Structured Output Parser** (so the AI always returns valid fields), **n8n Form**, **Google Sheets**, **Slack** and **Telegram**.

## Setup (about 5 minutes)

1. **Import** `workflow.json` (Workflows → Import from file, or Import from URL with the raw link to `workflow.json`).
2. **Apify:** if n8n asks, install the Apify node (a verified community node; search "Apify"). On n8n Cloud, the instance owner may need to allow verified community nodes in the Cloud Admin Panel first. Then open the **Google Trends** node and create a credential:
   - **n8n Cloud:** click the **sign-in button** and approve in Apify. No token to copy. The workflow ships with this option selected.
   - **Self-hosted n8n:** Apify's sign-in isn't available there, so set *Authentication* to **API Key** and paste your token from [apify.com](https://apify.com) (Settings → API & Integrations).
3. **AI (recommended):** add a **Google Gemini (PaLM) API** credential in the **Gemini model** node. A free key is available at [aistudio.google.com](https://aistudio.google.com). To use OpenAI, Anthropic, Groq or a local Ollama model, replace the **Gemini model** node with that chat model node and connect it to both **Suggest keywords** and **Write content ideas**. No AI? Set `useAI` to false in **Defaults**.
4. **Pick your inputs:** the form works as soon as the workflow is active. For the weekly plan, set up the sheet (above) and pick it in **Read keyword sheet** and **Add to Google Sheet**. Enable **Send to Slack** or **Send to Telegram** if you want the summary.
5. Optional: adjust **Defaults**.

   | Setting | What it does | Default |
   |---|---|---|
   | `useAI` | AI keyword suggestions and ideas, for every input. False gives the rising searches report | true |
   | `ideasPerNiche` | Ideas (or report rows) per niche | 10 |
   | `minRelevance` | Keep AI ideas scored at least this (0–10) | 6 |
   | `suggestKeywordsBelow` | Suggest keywords when a niche has fewer than this. 0 turns suggestions off | 3 |
   | `suggestedKeywords` | How many keywords the AI suggests | 8 |
   | `includeTopics` | Also use rising related topics | true |
   | `defaultCountry` | Used when a row has no country. Two-letter code, or empty for worldwide | US |
   | `defaultTimeRange` | `now 7-d`, `today 1-m`, `today 3-m` or `today 12-m` | today 1-m |
   | `defaultFormats` | What you publish, when the input doesn't say | blog post, short video, carousel, newsletter |
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

**20 keywords every week comes to about $0.16 a month**, well inside Apify's free monthly credit. AI suggestions add 8 keywords to a niche that has few, about $0.016 a run. With AI, it's one model request per niche for ideas, plus one for keyword suggestions when they're needed.

## How it was tested

On n8n 2.44.6 (self-hosted) with the Apify node 0.9.0 and Gemini 3.8 Flash, against live data. Forms were submitted over HTTP the way the form page does it.

| Test | Result |
|---|---|
| **Weekly sheet**, two niches, plus a paused row and a row without a niche | One Google Trends run per niche, 10 ideas each, 20 sheet rows. The paused row was ignored and the row without a niche was listed as skipped. Later runs skipped the ideas already sent and flagged what was new |
| **AI fails** (invalid key, and an AI account out of credit) | After the retries, every niche fell back to the report with the matching note, on the form, the sheet and test runs |
| **Quiet week** (keywords Google has no data for) | A "nothing new is rising" summary naming those keywords, and no sheet rows |
| **Without AI**, Run it now | 10 report rows with the same 19 columns |
| **Niche only, AI suggests the keywords** ("home espresso", nothing else) | 8 suggested keywords, every one with data: 200 rising searches, against 44 for the niche name alone. Ideas like "Bottomless Portafilter vs Regular" and "How to Make a DIY WDT Tool" |
| **Niche only, the hard case** ("AI for product management") | 8 suggested keywords (prompt engineering, ai agents, rag, langchain…): 288 rising searches, against 0 for the niche name and 4 for descriptive phrases. Top ideas: the JPMorgan block on external AI agents, indirect prompt injection |
| **Sheet: two niches with 2 keywords each, plus a niche-only row** ("sun care") | All three got 8 suggestions; 30 sheet rows. "Sun care" went from 3 weak rising searches (niche name) to 249 |
| **Web form with AI**: niche only, then 3 typed keywords + a CSV for a second niche | 10 idea cards in 85 seconds; then 20 cards for both niches in about 3.5 minutes, each keyword shown with its count. The niche with 3 keywords got no suggestions, as set |
| **Web form, AI off**, niche only | Results in about 50 seconds, with a note asking for a few keywords |

Across these live runs the AI suggested 34 keywords and none came back without Google data; most found 20–48 rising searches each, and the weakest ("dialing in espresso", 5) is flagged by its count.

Google Sheets, Slack and Telegram are standard n8n nodes. In these tests the sheet was read from a stand-in node with the same rows, and the outputs weren't connected to real accounts.

### Idea quality, across niches

The AI instructions were tuned on **8 different niches**, with **3 runs each** on the same live Google Trends data: AI for product management, skincare, home espresso, personal finance (US and India), indie game dev, home workouts and small business marketing. The workflow's own Code nodes ran with real Gemini calls, and every idea was read, not just counted.

| 24 runs each | First prompt (240 ideas) | Final prompt (229 ideas) |
|---|---|---|
| Unchecked numbers or amounts in titles and hooks ("$500 vs $2,500") | 88 | 6 |
| Made-up personal experience ("I gave it access to…") | 2 | 0 |
| A "why" stated as fact instead of a likely reason | 111 | 0 |
| Broad one-word searches turned into ideas ("Claude", "Serum") | 12 | 3 |
| The same picks across the 3 runs of a niche | about 74% | about 77% |

What a good week looks like, from these runs: "jpmorgan external ai agents blocked" (Breakout) for product managers, "income tax audit due date 2026" in India in October, "halloween latte art" for home espresso, "does creatine cause hair loss" for home workouts. In thin niches (indie game dev, where "unity" also brings "German Unity Day") you get fewer ideas instead of filler: 6–8 instead of 10.

## Honest limits

- Google Trends numbers are **relative**: "+3,600%" means fast growth, not big volume.
- Without AI, the word match can drop a real riser that shares no word with your keywords, such as a new brand name, and lets homonyms in ("Weight cycling" under *skin cycling*).
- **AI ideas can still contain details that aren't true.** The prompt keeps facts out of titles and hooks unless they're on the fact-check list, but checking that list is your job; nothing is published automatically.
- Results are only as good as your keywords (see [How keywords work](#how-keywords-work-you-dont-have-to-guess-whats-booming)). The counts next to each keyword show which ones are pulling their weight.
- The "already sent" memory belongs to weekly sheet runs on an active workflow; runs from the editor start fresh.
- The form waits for the whole run, about 1.5–2 minutes per niche with AI. For dozens of niches, use the sheet.

## License

MIT. Disclosure: I built the Google Trends Actor this workflow uses.
