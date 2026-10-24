# Niche trend content planner for n8n

Every week, this n8n workflow finds the Google searches that are **rising in your niche**, throws out the junk, and turns the rest into content ideas in a review queue.

Most "Google Trends to content" workflows use **Trending Now**: whatever is trending nationally today, which is usually sports, celebrities and news. That's useless if you write about skincare, SaaS or home espresso. This one starts from **your own keywords** and uses Google Trends' **rising related queries and related topics** for them.

## What a run looks like

Real output from a test run (US, past 30 days) with six skincare keywords: `retinol, sunscreen, niacinamide, hyaluronic acid, vitamin c serum, skin cycling`.

| Rising search | Growth | Idea (Gemini 3.8 Flash) |
|---|---|---|
| is niacinamide good for rosacea | +3,600% | Can You Use Niacinamide If You Have Rosacea? |
| medix 5.5 retinol cream reviews | Breakout | Medix 5.5 Retinol Cream: Can You Actually Use It on Your Face? |
| Beauty Of Joseon Daily Tinted Fluid Sunscreen (topic) | Breakout | Beauty of Joseon Tinted Sunscreen: The Ultimate 2-in-1 for Lazy Mornings |
| timeless skin care 20% vitamin c e ferulic acid serum | +450% | Why Beginners Should Be Careful with Timeless 20% Vitamin C |
| good molecules sakura hyaluronic acid boosting essence | Breakout | How to Layer the Good Molecules Sakura Hyaluronic Acid Essence |

Each idea comes with an angle, a hook and a format (blog post, short video, carousel or newsletter), plus a link to the search on Google Trends.

The run found 236 rising searches and dropped 152 of them as off-topic or noise; more on that below. It took 58 seconds.

## The noise problem (and how it's handled)

Google Trends data in 2026 is full of junk that looks like bot and AI-agent traffic. In the test above, these came back as **Breakout** searches under skincare keywords:

- `how to make sushi`, `what is a 401k`, `best time to visit maldives`, `bread`, `schools`, `jobs`
- `explain what niacinamide is and its main benefits for skin health`, which reads like a prompt, not a person

Keep them and your content plan fills with nonsense. The workflow drops:

1. **Off-topic searches:** ones that share no real word with your keyword or your niche description (`onlyRelated`)
2. **Prompt-like searches:** ones that start like an instruction to an AI ("explain", "list", "summarize"...) or run to 9+ words
3. **Repeated junk:** the same search turning up under several of your keywords while sharing no word with them (`noiseMinKeywords`)

It also spreads the ideas across your keywords and skips anything it already sent you in the last 8 weeks.

## How it works

```
Schedule (Mondays 8:00) ─► Your settings ─► Google Trends (Apify) ─► Pick the real risers ─► AI ideas (optional) ─► Review queue
                                                                                                              ├─► Google Sheets
                                                                                                              └─► Slack / Telegram summary
```

- **Google Trends data:** the [Google Trends Actor on Apify](https://apify.com/clean-fetch/google-trends-scraper), called over Apify's API with only the parts this workflow needs (rising related queries, optionally related topics).
- **AI step:** any OpenAI-compatible API: OpenAI, Gemini, OpenRouter, Groq or a local Ollama. It's optional. Without it, you still get a simple idea per rising search.
- **Nothing is published automatically.** Ideas land in a sheet marked "to review".

## Setup (about 5 minutes)

1. **Import** `workflow.json` into n8n (Workflows → Import from file).
2. **Apify token:** create a free account at [apify.com](https://apify.com), copy your API token (Settings → API & Integrations). In n8n, create a **Header Auth** credential with name `Authorization` and value `Bearer YOUR_TOKEN`, and select it in the **Run Google Trends** node.
3. **Edit "Your settings":**

   | Setting | What it does | Default |
   |---|---|---|
   | `keywords` | Your niche keywords, comma-separated (up to 50) | six skincare keywords |
   | `niche` | One line about what you make content about. Used for relevance and by the AI | skincare for beginners |
   | `country` | Two-letter country code, or empty for worldwide | US |
   | `timeRange` | `now 7-d`, `today 1-m` or `today 3-m` | today 1-m |
   | `includeTopics` | Also use rising related topics | true |
   | `ideasPerRun` | How many ideas per run | 10 |
   | `onlyRelated` | Drop searches that share no word with your keyword or niche | true |
   | `noiseMinKeywords` | The same unrelated search under this many keywords counts as junk | 3 |
   | `aiEnabled` | Use an AI model for the ideas | false |
   | `aiBaseUrl` / `aiModel` | Any OpenAI-compatible endpoint and model | OpenAI, gpt-4o-mini |

4. **AI (optional):** set `aiEnabled` to true and create a second Header Auth credential (`Authorization` / `Bearer YOUR_KEY`) for the **Ask AI** node.
   - **Gemini:** `aiBaseUrl` = `https://generativelanguage.googleapis.com/v1beta/openai`, `aiModel` = `gemini-3.8-flash` (or another Gemini model)
   - **Local Ollama:** `aiBaseUrl` = `http://localhost:11434/v1`
5. **Outputs:** enable **Add to Google Sheet** (pick your sheet; a tab named `Ideas`) and **Send to Slack** or **Send to Telegram** if you want the weekly summary.
6. **Activate** the workflow. Click **Run it now** first to see a result straight away.

## Cost

The Google Trends Actor charges per keyword, only for data it actually delivers: rising related queries are $0.50 per 1,000 keywords and related topics $1.50 per 1,000. **20 keywords every week is about $0.16 a month**, well inside Apify's free monthly credit. Keywords Google has no data for are free. The AI step costs whatever your model charges for one short request per run.

## How it was tested

On n8n 2.44.6 (self-hosted), against the live Actor:

- **without AI:** every node ran. 245 rising searches, 10 ideas, the noise filter dropped 149, and the second run skipped 1 idea it had already sent
- **with a mock OpenAI-compatible server:** checked the request (model, JSON mode, auth header), the parsing, and that an idea the model marks `skip` is dropped
- **with Gemini 3.8 Flash:** 8 ideas in 58 seconds; 2 off-niche risers skipped by the model

Google Sheets, Slack and Telegram are standard n8n nodes. They weren't connected to real accounts in these tests.

## Honest limits

- Google Trends numbers are **relative**: "+3,600%" means fast growth, not big volume. Some keywords have no rising data at all, and they're skipped for free.
- The noise filter is a **heuristic**. It can drop a real riser that shares no word with your keywords (for example a new brand name). Set `onlyRelated` to false and let the AI judge relevance if that matters to you.
- **AI ideas can include made-up details**, like prices or launch claims. Check facts before you publish; that's why there's a review queue.
- "Already sent" memory only persists for active (scheduled) runs, not for manual test runs in the editor.
- The Actor call waits up to 5 minutes, so keep it to 50 keywords per run.

## License

MIT. Disclosure: I built the Google Trends Actor this workflow uses.
