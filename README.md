# Niche trend content planner for n8n

This n8n workflow finds the Google searches that are **rising in your niche**, throws out the junk, and turns the rest into content ideas. Each idea comes with the likely reason it's rising, an angle, a hook, a format and **the facts to check before you publish**.

Most "Google Trends to content" workflows use **Trending Now**: whatever is trending nationally today, which is usually sports, celebrities and news. That's useless if you write about skincare, SaaS or home espresso. This one starts from **what you want to make content about**, in your own words, and uses Google Trends' **rising related queries and related topics** around it.

## Who it's for, and what they type in

Two people touch this workflow, and the inputs are shaped around each.

- **The owner** imports it into n8n, connects Apify and an AI model, and picks a few settings once in **Defaults**. Often that's a creator doing it for themselves; often it's a freelancer setting it up for a client.
- **The user** fills in the form or the sheet: a creator, marketer or small business owner who wants to know what to post this week. They know their niche. They usually don't know Google Trends or "keywords", and they shouldn't have to.

So the form asks one thing in plain words and lets people answer however they like:

| Question | |
|---|---|
| **What do you want to make content about?** | Free-form. "home espresso" works; so does "skincare for beginners, mostly retinol and acne scars", or "I make YouTube videos about AI for product managers. Want to cover evals, agents, and how PMs should prototype with AI" |
| **Where do you want to post?** | Blog, YouTube, Shorts / Reels / TikTok, Instagram carousel, Newsletter, LinkedIn, Podcast, X. The ideas use these formats |
| **Where do you want to reach people?** | A country, or Worldwide. Empty uses the owner's default |
| **How recent should the trends be?** | Last 30 days (recommended), last 7 days for fast-moving topics, last 3 months for slower niches, last 12 months |
| **Several niches at once? Upload a CSV** | For agencies and batches: one row per niche (see [bulk](#several-niches-at-once-bulk)). The form explains the format right above the upload and links a template; the first box can then stay empty |

The button says **Get my content ideas**.

## The AI reads what they wrote and plans what to watch

Google Trends works from **search terms**. A term is a seed: Google reports the searches that are rising around it, measured from real search data ("can you use niacinamide with tretinoin" +300%, "halloween latte art" Breakout). So nobody has to guess what's booming; the only question is which seeds to watch, and people describe a niche differently from how they search it. Nobody types "how PMs should prototype with AI" into Google.

So the first AI step, **Plan topics**, reads the answer and returns the niche, the audience (if they mentioned one) and about 8 search terms. Their own topics are judged, not thrown away:

| What they wrote | What the planner did (live runs) |
|---|---|
| "skincare for beginners, mostly retinol for teens and acne scars" | kept **acne scars**; turned "retinol for teens" into **retinol** and **teen acne**; added salicylic acid, benzoyl peroxide, niacinamide, cerave… |
| "I make YouTube videos about AI for product managers. Want to cover evals, agents, and how PMs should prototype with AI" | turned them into **llm evals**, **ai agents**, **ai prototyping**; added prompt engineering, rag, cursor ai, v0… |
| "personal finance for college students" with the topics roth ira, "how to save money as a college student on a budget", credit card, crypto memes | kept **roth ira** and **credit card**; turned the long phrase into **budgeting**; **left out "crypto memes"** and said why; added student loans, fafsa, credit score… |
| "I post short videos about budget travel in Japan for families with kids" | niche "budget Japan family travel"; jr pass, suica card, tokyo disneyland, universal studios japan… |

Does this beat just searching what people typed? Eight free-form requests, the same day, counting rising searches only:

| | Rising searches found |
|---|---|
| What they typed, split into short terms | 112 |
| The AI's planned terms (two rounds) | 669 and 630 |

Every one of the eight did better, and two found nothing at all as typed ("budget travel in Japan", "indie game dev, I use godot"). The planner isn't perfect: now and then it picks a precise term few people search ("tokyo with toddlers", "teen acne"). Those cost nothing and show up with a count of 0.

The results show every term watched, where it came from, and how many rising searches it found, for example `retinol (19), acne scars (18, from "how to deal with acne scars"), cerave* (30)`. Terms the AI added are marked with *, terms with no data are crossed out, and the weekly summary names the ones that found little and suggests adding the good ones to your sheet.

## Why the AI is built in, and what happens when it fails

The AI does two jobs here: it reads the request and plans the terms, then it scores every rising search and writes the ideas. Making it optional would mean asking people for keywords in Google's language, which is exactly what they can't do. So it's built in. The tradeoffs, honestly:

- **You need an AI key.** Gemini has a free tier; any n8n chat model works.
- **What people write goes to that AI provider**, along with the rising searches.
- **AI calls fail sometimes**: a wrong key, no credit, rate limits. Each AI step retries 3 times. If planning still fails, what they wrote is split into short terms and searched as it is ("…mostly retinol, sunscreen and niacinamide, and how to deal with acne scars" → retinol, sunscreen, niacinamide, acne scars). If the ideas step fails, the niche gets the **list of rising searches** instead of ideas. Both come with a note saying why ("the AI key or model name needs checking", "the AI account is out of credit", "the AI service is busy, try again soon"). A run never comes back empty because of the AI.

## Three ways to use it

Each one is labelled on the canvas (**Way 1 · Web form**, **Way 2 · Google Sheet**, **Way 3 · Test run**).

| | Best for | What you give it | Where results go |
|---|---|---|---|
| **1. Web form** | One-off checks, a link for a client who doesn't use n8n, or a one-off batch | One free-form answer, and/or a CSV with one row per niche | A results page, right after you submit |
| **2. Google Sheet, every Monday** | A standing weekly plan, any number of niches | One row per niche in a `Keywords` tab | A row per idea in an `Ideas` tab, plus a summary in Slack or Telegram |
| **3. Test run** | Trying it out in the editor | Edit the **Test input** node | Same as the sheet |

All three feed the same pipeline, so they behave the same way.

### 1. Web form

Activate the workflow and open the **Form** node's production URL (it ends in `/form/rising-in-my-niche`). Write what you want to make content about and submit.

### Several niches at once (bulk)

Both bulk options use **the same file shape: one row per niche**. Only `niche` is required, written in any words. The form shows this right above the upload, with an example and a link to [bulk_template.csv](bulk_template.csv).

| Column | Required | What goes in it |
|---|---|---|
| `niche` | yes | What you want to make content about, in any words: "home espresso", or "I post short videos about budget travel in Japan for families with kids" |
| `topics` | no | Topics you want covered, separated by commas: `retinol, sunscreen, how to deal with acne scars`. Each is judged like everything else: kept, turned into a search term, or left out (and named) if it's off-niche |
| `audience` | no | Who it's for |
| `country` | no | Two-letter code (`US`, `IN`); empty uses the default |
| `formats` | no | Where you post (sheet only; the form uses its checkboxes) |
| `active` | no | `no` pauses the row (sheet only) |

```csv
niche,topics,audience,country
skincare for beginners,"retinol, sunscreen, how to deal with acne scars",people new to skincare,US
home espresso,,people making coffee at home,US
I post short videos about budget travel in Japan for families with kids,,,US
```

- **Form upload:** for a one-off batch. Leave the first box empty, or use it for one more niche. If the typed answer and a row turn out to be the same niche, they're merged.
- **Google Sheet:** for a standing weekly plan. Same columns, in a tab named `Keywords`.
- **Speed:** niches with the same country and time range share Google Trends calls (up to 40 terms per call), so four niches take one call. A live 4-niche upload took about 2.5 minutes in all. The AI still plans and writes per niche, so very large batches are better on the sheet than on a form someone waits on.
- **Old files still work:** a `keyword` column with one topic per row (the niche repeated on each row) is read too, and rows of the same niche are merged.
- **Limit:** `maxKeywordsPerRun` (300) caps a run, which is about 25 niches.

### 2. Google Sheet, every Monday

Make a sheet with a tab named `Keywords`, one row per niche (the columns above):

| niche | topics | audience | country | formats | active |
|---|---|---|---|---|---|
| skincare for beginners | retinol, niacinamide | people new to skincare | US | | |
| home espresso | espresso machine, latte art | people making coffee at home | | youtube, blog | |
| moka pot recipes | | | | | no |
| I post short videos about budget travel in Japan for families with kids | | | | | |

Rows without a niche are skipped and listed in the summary. Each niche is planned on its own, with its own ideas and its own memory. Point **Read keyword sheet** at the sheet and **Add to Google Sheet** at the same file (tab `Ideas`), then activate the workflow.

### 3. Test run

Write in **Test input** what you'd type in the form, then click **Execute workflow**.

## What a run looks like

Real output from the live tests below (US, past 30 days, Gemini 3.8 Flash):

| Asked for | Rising search | Growth | Idea |
|---|---|---|---|
| skincare, "…and how to deal with acne scars" | what helps acne scars go away | Breakout | What Actually Helps Acne Scars Go Away? |
| skincare | can you use niacinamide while pregnant | Breakout | Can You Use Niacinamide While Pregnant? |
| AI for product managers | fine tuning vs prompt engineering | Breakout | Fine-Tuning vs Prompt Engineering: The AI PM Decision Tree |
| AI for product managers | what is cursor ai used for | +350% | What Is Cursor AI Used For? A PM Guide to Rapid Prototyping |
| Japan travel with kids | suica card vs jr pass | Breakout | Suica Card vs JR Pass: Which Saves Families More Money? |
| home espresso | bottomless portafilter vs regular | Breakout | Bottomless vs Regular Portafilter: What Actually Changes? |

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

### The sheet columns

Ideas and the fallback list use the **same 19 columns**, so one `Ideas` tab holds both:

`date, niche, mode, status, idea, angle, hook, format, whyRising, checkBeforePublishing, risingSearch, growth, type, alsoSearchedAs, newThisWeek, yourKeywords, relevance, googleTrends, note`

- `mode` is `ideas`, or `report` when the AI couldn't write ideas.
- `status` starts as `to review`.
- `note` explains a fallback, for example when the AI was unavailable.

## How it picks what's worth writing about

Per niche:

**1. Junk out.** Google Trends data in 2026 is full of what looks like bot and AI-agent traffic. With six skincare keywords, these came back as **Breakout** searches:
- generic ones under unrelated keywords: `how to make sushi`, `what is a 401k`, `best time to visit maldives`, `bread`
- prompt-like ones: `explain what niacinamide is and its main benefits for skin health`

Removed, before anything reaches the AI: searches that read like prompts (they start with "explain", "describe", "write", "list", "give me", "tell me", "summarize", "compare", "create", "generate", "provide" or "act as", or contain phrases like "and its main benefits", "why do you think", "in simple terms"), "searches" longer than 15 words, and generic searches repeated under 3 or more unrelated terms. Real long questions stay, like "what is a high yield savings account and how does it work", and so do paper and product titles.

**2. Variants merged.** "trader joe's retinol" and "trader joes retinol" become one. A topic and a search naming the same product merge too.

**3. Ranked by growth**, boosted when something rises under several of your terms or as both a search and a topic. A search that just repeats a term you watch ranks lower. Picks are spread across terms, so one term can't take every slot.

**4. The AI scores each one 0–10** with a fixed scale: a specific question, product, rule or story for your audience scores high; a broad name on its own ("Claude", "Steam", "Serum") scores 5 or less, even if the model can imagine a good angle. A search that's just one of your terms again is capped at 5 in code. Several searches about the same story or product type (four walking-pad brands) become one idea that covers them all. The ideas lean towards what the person said they want to cover.

**5. Memory (weekly sheet runs).** It never repeats an idea within 8 weeks, and marks what's **new since the last run**. Form and test runs don't touch the memory.

**6. Quiet weeks** get a short "nothing new is rising" line in the summary (and on the results page) instead of silence, naming the terms Google has no data for. No empty rows are added to your sheet.

All tuning values are commented at the top of the **Clean and rank** node.

## How it works

```
Form ──────────→ From form ───────┐
Every Monday → Read keyword sheet → From sheet ──┼→ Defaults → Group by niche
Run it now ────→ Test input → From test input ┘                       │
                         Ask AI to plan topics → Plan topics (LLM Chain + Gemini + Structured Output Parser)
                                                   ├ ok    → Use planned topics ──┐
                                                   └ error → Plan without AI ─────┴→ All niches → Build jobs
                                                                      │ (same niches merged, 40 terms per call)
                                        Loop over jobs: Build Actor input → Google Trends (Apify) → Collect
                                                                      │
                                                               Clean and rank
                                                                      │
                               Anything to write about? ── yes → Prepare AI request → Write content ideas
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

Each input has a small adapter that turns it into the same rows (bulk files share one reader, so the form's CSV and the sheet behave the same): `niche, keyword, audience, country, timeRange, formats, source`, where `niche` is what the person wrote and `keyword` an optional topic they listed. Empty fields are filled from **Defaults**. One **Gemini model** node serves both AI steps, so there is one AI credential to set up. To add another source (Airtable, Notion, a webhook), add one adapter that outputs those rows and connect it to **Defaults**. Nothing else changes.

All native nodes: the **Apify** node for Google Trends, two **Basic LLM Chains** with a **Google Gemini Chat Model** and **Structured Output Parsers** (so the AI always returns valid fields), **n8n Form**, **Google Sheets**, **Slack** and **Telegram**.

## Setup (about 5 minutes)

1. **Import** `workflow.json` (Workflows → Import from file, or Import from URL with the raw link to `workflow.json`).
2. **Connect Apify** (it fetches the Google Trends data). If n8n offers to install the Apify node, click **Install** (it's a verified community node; on n8n Cloud the instance owner may need to allow verified community nodes in the Cloud Admin Panel first). Then:
   - double-click the **Google Trends** node;
   - open **Credential to connect with** and choose **Create new credential**;
   - in the window that opens, click the **sign-in button** and approve in Apify. Nothing to copy.

   *Self-hosted n8n has no Apify sign-in.* In the node, set *Authentication* to **API Key** first, then paste your token from [apify.com](https://apify.com) (Settings → API & Integrations) in that window.
3. **Connect the AI** (it reads the request, plans the search terms and writes the ideas, so it's required). Double-click **Gemini model** → **Credential to connect with** → **Create new credential**, and paste a free key from [aistudio.google.com](https://aistudio.google.com). To use OpenAI, Anthropic, Groq or a local Ollama model, replace the **Gemini model** node with that chat model node and connect it to both **Plan topics** and **Write content ideas**.
4. **Pick your ways in.** Way 1 (the form) works as soon as the workflow is active. For Way 2, set up the sheet (above) and pick it in **Read keyword sheet** and **Add to Google Sheet**. For a weekly message, turn on **Send to Slack** or **Send to Telegram** and connect it the same way.
5. Optional: adjust **Defaults**.

   | Setting | What it does | Default |
   |---|---|---|
   | `ideasPerNiche` | Ideas per niche | 10 |
   | `minRelevance` | Keep AI ideas scored at least this (0–10) | 6 |
   | `topicsPerNiche` | How many search terms the AI aims for per niche | 8 |
   | `maxTopicsPerNiche` | The most it may use, including your own | 12 |
   | `includeTopics` | Also use rising related topics, not just searches | true |
   | `defaultCountry` | Used when a row has no country. Two-letter code, or empty for worldwide | US |
   | `defaultTimeRange` | `now 7-d`, `today 1-m`, `today 3-m` or `today 12-m` | today 1-m |
   | `defaultFormats` | What you publish, when the input doesn't say | blog post, short video, carousel, newsletter |
   | `maxKeywordsPerRun` | A safety cap on search terms per run | 300 |

6. Click **Execute workflow** on **Way 3 · Test run** to see a result, then **activate** the workflow for the form and the weekly schedule.

## Make it yours

- **Another niche:** add rows to the sheet, or write it in the form. Nothing to edit.
- **Faster or slower niches:** `defaultTimeRange` `now 7-d` for fast-moving topics, `today 3-m` for slow ones (the form has its own picker).
- **Different output:** every idea is one flat row with fixed column names, so Notion, Airtable, email or a webhook can replace Google Sheets in one step.
- **Different idea style:** edit the instructions in **Prepare AI request**, for example video scripts, newsletter sections or product ideas.
- **Stricter or looser filtering:** raise `minRelevance`, or change the commented values at the top of **Clean and rank**.

## How it was tested

On n8n 2.44.6 (self-hosted) with the Apify node 0.9.0 and Gemini 3.8 Flash, against live data. Forms were submitted over HTTP the way the form page does it.

| Test (current version) | Result |
|---|---|
| **Web form, one free-form answer** ("I make YouTube videos about AI for product managers…") | Read as "AI product management"; watched llm evals, ai agents, ai prototyping (each labelled with the words it came from) plus five added terms; 10 idea cards in about 1.5 minutes |
| **Web form, bulk only**: first box empty, a 4-niche CSV in the one-row-per-niche shape | 4 niches in about 2.5 minutes, 40 idea cards, sharing one Google Trends call. The page opened with "Ideas for 4 niches: …"; "how to deal with acne scars" became **acne scars**; roth ira and credit card were kept |
| **Web form, short answer + an older keyword-per-row CSV** for two niches | "how to get rid of acne scars fast" became **acne scars**; the typed answer and the CSV's rows for the same niche were merged; 20 idea cards |
| **Test run, free-form** ("…Mostly retinol, sunscreen and niacinamide, and how to deal with acne scars") | Kept retinol, sunscreen, niacinamide; turned the last part into **acne scars**; added four terms. 4 of the 10 ideas were about acne scars, as asked |
| **Weekly sheet**: niches with topics (one of them "crypto memes", one in a `topics` column), a paused row, a row without a niche, and a free-text niche | Three niches in one Google Trends call, 29–30 sheet rows. "crypto memes" was left out and named; retinol and niacinamide kept; the paused row ignored; the row without a niche listed as skipped; the Japan row planned from its sentence |
| **AI fails** (invalid key) | Planning fell back to the typed words split into short terms (retinol, sunscreen, niacinamide, acne scars); the ideas step fell back to the rising searches list; both with the matching note |
| **Offline**: the workflow's own code, 6 inputs (form, form + CSV, older CSV, bulk-only CSV, sheet, test run) × 4 AI cases (both working, planning fails, ideas fail, both fail) | 24 of 24 pass |

From earlier rounds, on code that hasn't changed: later sheet runs skip ideas already sent and flag what's new; a quiet week gives a "nothing new is rising" summary and no sheet rows.

Google Sheets, Slack and Telegram are standard n8n nodes. In these tests the sheet was read from a stand-in node with the same rows, and the outputs weren't connected to real accounts.

### Idea quality, across niches

The ideas prompt was tuned on **8 different niches**, with **3 runs each** on the same live Google Trends data: AI for product management, skincare, home espresso, personal finance (US and India), indie game dev, home workouts and small business marketing. The workflow's own Code nodes ran with real Gemini calls, and every idea was read, not just counted.

| 24 runs each | First prompt (240 ideas) | Final prompt (229 ideas) |
|---|---|---|
| Unchecked numbers or amounts in titles and hooks ("$500 vs $2,500") | 88 | 6 |
| Made-up personal experience ("I gave it access to…") | 2 | 0 |
| A "why" stated as fact instead of a likely reason | 111 | 0 |
| Broad one-word searches turned into ideas ("Claude", "Serum") | 12 | 3 |
| The same picks across the 3 runs of a niche | about 74% | about 77% |

In thin niches (indie game dev, where "unity" also brings "German Unity Day") you get fewer ideas instead of filler.

## Honest limits

- Google Trends numbers are **relative**: "+3,600%" means fast growth, not big volume.
- **AI ideas can still contain details that aren't true.** The prompt keeps facts out of titles and hooks unless they're on the fact-check list, and it still slips now and then; checking that list is your job. Nothing is published automatically.
- The planner sometimes picks a term few people search. It shows up with a low count; replace it in your sheet if it keeps finding little.
- The "already sent" memory belongs to weekly sheet runs on an active workflow; runs from the editor start fresh.
- The form waits for the whole run, about 1.5–2 minutes per niche. For dozens of niches, use the sheet.

## License

MIT. Disclosure: I built the Google Trends Actor this workflow uses.
