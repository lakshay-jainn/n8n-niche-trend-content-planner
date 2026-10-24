# Niche trend content planner for n8n

Every week, this n8n workflow finds the Google searches that are **rising in your niche**, throws out the junk, and turns the rest into content ideas in a review queue. Each idea comes with why it's probably rising, an angle, a hook and **the facts to check before you publish**.

Most "Google Trends to content" workflows use **Trending Now**: whatever is trending nationally today, which is usually sports, celebrities and news. That's useless if you write about skincare, SaaS or home espresso. This one starts from **your own keywords** and uses Google Trends' **rising related queries and related topics** for each of them.

## What a run looks like

Real output (US, past 30 days, Gemini 3.8 Flash) for six skincare keywords: `retinol, sunscreen, niacinamide, hyaluronic acid, vitamin c serum, skin cycling`. The niche is "skincare for beginners".

| Rising search | Growth | Idea |
|---|---|---|
| is niacinamide good for rosacea | Breakout | Can You Use Niacinamide If You Have Rosacea? |
| new sunscreen ingredient fda bemotrizinol | +700% | Why Everyone Is Talking About Bemotrizinol (BEMT), and What It Means for Your Sunscreen |
| Flushing (topic, under niacinamide) | +180% | Did Niacinamide Turn Your Face Bright Red? Here's What Just Happened |
| when should i use vitamin c serum | +190% | Morning or Night? When You Should Actually Apply Vitamin C |
| medix 5.5 retinol cream reviews | Breakout | Medix 5.5 Retinol Cream: Can You Really Use an $18 Body Cream on Your Face? |

Every row in your sheet looks like this:

| Column | Example |
|---|---|
| idea | Does Niacinamide Cause Purging? The Myth That's Ruining Your Skin Barrier |
| whyRising | Confusion is high because niacinamide is in almost every product, and people mistake reactions for a purge. |
| angle | Explain that niacinamide doesn't speed up cell turnover, so breakouts are reactions, not a purge. |
| hook | If someone told you your skin is just "purging" from niacinamide, they gave you bad advice. |
| format | short video |
| checkBeforePublishing | Confirm that faster cell turnover is specific to retinoids and exfoliating acids, not niacinamide |
| risingSearch, growth, type | does niacinamide cause purging · +200% · search |
| alsoSearchedAs, newThisWeek, yourKeywords, relevance, googleTrends | variants · yes/no · niacinamide · 10 · link |

## How it picks what's worth writing about

**1. Junk out.** Google Trends data in 2026 is full of what looks like bot and AI-agent traffic. With the six skincare keywords above, these came back as **Breakout** searches:
- generic ones under unrelated keywords: `how to make sushi`, `what is a 401k`, `best time to visit maldives`, `bread`
- prompt-like ones: `explain what niacinamide is and its main benefits for skin health`

Searches that look like prompts, and generic searches repeated under several of your keywords, are removed.

**2. Relevance.**
- **With AI**, the model scores every search 0–10 for your niche and audience, and you keep those at or above `minRelevance`. It catches what a word match can't, like "Bemotrizinol" (a new sunscreen ingredient) or "Flushing" linked to niacinamide.
- **Without AI**, a search has to share a real word with its keyword or your niche description.

**3. Variants merged.** "trader joe's retinol" and "trader joes retinol" become one idea. A topic and a search naming the same product merge too.

**4. Ranked by growth**, boosted when something rises under several of your keywords or as both a search and a topic.

**5. Memory.** It never repeats an idea within 8 weeks, and marks what's **new since the last run**.

**6. Balance.** Ideas are spread across your keywords, so one keyword can't take every slot.

**7. Quiet weeks** send a short "nothing new this week" note instead of nothing, including which keywords Google has no data for.

All tuning values are commented at the top of the **Clean and rank** node.

## How it works

```
Every Monday / Run it now
  → Your settings → Build Actor input → Google Trends (Apify node)
  → Collect rising searches → Clean and rank → Anything new? ──no──→ Nothing new this week ─┐
                                                   │ yes                                     │
                                                Use AI? ──no──→ Basic ideas ──┐              │
                                                   │ yes                      │              │
       Prepare AI request → Write content ideas (LLM Chain + Gemini + Structured Output Parser)
                          → Match ideas to searches ──────────────────────────┤              │
                                                                Review queue rows             │
                                                         ├→ Add to Google Sheet               │
                                                         └→ Weekly summary ──→ Slack / Telegram ←┘
```

All native nodes: the **Apify** node for Google Trends, a **Basic LLM Chain** with a **Google Gemini Chat Model** and a **Structured Output Parser** (so the AI always returns valid fields), then **Google Sheets**, **Slack** and **Telegram**.

## Setup (about 5 minutes)

1. **Import** `workflow.json` (Workflows → Import from file, or Import from URL with the raw link to `workflow.json`).
2. **Apify node:** if n8n asks, install it. It's a verified community node; search "Apify" in the nodes panel. On n8n Cloud, the instance owner may need to allow verified community nodes in the Cloud Admin Panel first. Then create a free account at [apify.com](https://apify.com), copy your API token (Settings → API & Integrations), and add an **Apify API** credential in the **Google Trends** node.
3. **Edit "Your settings":**

   | Setting | What it does | Default |
   |---|---|---|
   | `keywords` | Your niche keywords, comma-separated (up to 50) | six skincare keywords |
   | `niche` | What you make content about | skincare for beginners |
   | `audience` | Who it's for. The AI writes for them | people new to skincare… |
   | `contentFormats` | What you publish | blog post, short video, carousel, newsletter |
   | `country` | Two-letter code, or empty for worldwide | US |
   | `timeRange` | `now 7-d`, `today 1-m`, `today 3-m` or `today 12-m` | today 1-m |
   | `includeTopics` | Also use rising related topics | true |
   | `ideasPerRun` | Ideas per run | 10 |
   | `useAI` | Write ideas with an AI model | false |
   | `minRelevance` | Keep AI ideas scored at least this (0–10) | 6 |

4. **AI (recommended):** set `useAI` to true, then add a **Google Gemini (PaLM) API** credential in the **Gemini model** node. A free key is available at [aistudio.google.com](https://aistudio.google.com). To use OpenAI, Anthropic, Groq or a local Ollama model, replace the **Gemini model** node with that chat model node. Nothing else changes.
5. **Outputs:** enable **Add to Google Sheet** (pick your sheet; a tab named `Ideas`) and **Send to Slack** or **Send to Telegram**.
6. Click **Run it now** to see a result, then **activate** the workflow for the weekly schedule.

## Make it yours

- **Another niche:** change `keywords`, `niche` and `audience`. That's all.
- **Faster or slower niches:** `timeRange` `now 7-d` for fast-moving topics, `today 3-m` for slow ones.
- **Different output:** every idea is one flat row with fixed column names, so Notion, Airtable, email or a webhook can replace Google Sheets in one step.
- **Different idea style:** edit the instructions in **Prepare AI request**, for example video scripts, newsletter sections or product ideas.
- **Stricter or looser filtering:** raise `minRelevance`, or change the commented values at the top of **Clean and rank**.

## Cost

The [Google Trends Actor](https://apify.com/clean-fetch/google-trends-scraper) charges per keyword, only for data it actually delivers:
- rising related queries cost **$0.50** per 1,000 keywords;
- related topics cost **$1.50** per 1,000 keywords;
- keywords Google has no data for are free.

**20 keywords every week comes to about $0.16 a month**, well inside Apify's free monthly credit. The AI step is one request per run.

## How it was tested

On n8n 2.44.6 (self-hosted) with the Apify node 0.9.0, against live data:

| Test | Result |
|---|---|
| **Without AI** | Every node ran (241 rising searches, 10 ideas). A second run skipped the 12 ideas already sent and served new ones. |
| **With Gemini 3.8 Flash** | 10 ideas in about 1.5 minutes. The structured parser returned valid fields, and 10 off-topic searches were dropped by relevance. |
| **Quiet week** (keywords Google has no data for) | The "nothing new" note was sent and named those keywords. |

Google Sheets, Slack and Telegram are standard n8n nodes. They weren't connected to real accounts in these tests.

## Honest limits

- Google Trends numbers are **relative**: "+3,600%" means fast growth, not big volume.
- Without AI, the word match can drop a real riser that shares no word with your keywords, such as a new brand name. With AI, those get a chance.
- **AI ideas can contain details that aren't true**, like prices or launch claims. That's why each idea lists what to check, and nothing is published automatically.
- The "already sent" memory is kept for active (scheduled) runs; manual test runs in the editor start fresh.
- The Actor call waits up to 5 minutes, so keep it to 50 keywords per run.

## License

MIT. Disclosure: I built the Google Trends Actor this workflow uses.
