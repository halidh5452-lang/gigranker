# GigForge AI - Fiverr SEO Gig Studio

Enter any service you can sell (logo design, data entry, video editing, Excel work, SEO audit...) and the app
researches the marketplace and builds a publish-ready, page-1-optimised gig:

- Market research: search intent, buyer persona, what top-ranked gigs do, and the gap you can own
- 5 SEO titles (max 80 chars, primary keyword front-loaded) with a recommended pick
- Full gig description: hook, body, why-me, deliverables, process, 5 FAQs, buyer requirements, CTA
- Exactly 5 tags (max 20 chars each, head + longtail mix)
- Keyword map: 15-25 terms with volume / competition / relevance scores and where to place each one
- Basic / Standard / Premium pricing packages matched to your tier
- 3 gig image concepts rendered at Fiverr's 1280x769, exportable as PNG or SVG
- 48-hour first-order plan, profile checklist and honest cautions
- A deterministic Fiverr SEO score (0-100) with per-factor tips

## Quick start

```bash
npm install
npm run dev
```

Open http://localhost:3000, type a service and press **Generate my page-1 gig**.

## Configuration

All settings live in `.env.local` (gitignored, never sent to the browser):

| Variable | Purpose |
| --- | --- |
| `OPENROUTER_API_KEY` | Your OpenRouter key. Server-side only. |
| `OPENROUTER_MODEL` | Primary model, default `deepseek/deepseek-v4-flash-0731:free`. |
| `OPENROUTER_FALLBACK_MODELS` | Comma-separated models tried when the primary is rate limited. |
| `OPENROUTER_REASONING_EFFORT` | `low` (default), `medium`, `high` or `off`. |
| `OPENROUTER_SITE_URL` | Optional OpenRouter attribution header. |

## How it works

1. `POST /api/generate` splits the job into **two focused prompts that run in parallel** (`lib/prompt.ts`):
   one for research + gig copy, one for the keyword map, gig images and launch plan. Running them
   concurrently roughly halves the wall-clock time of a free model.
2. `lib/openrouter.ts` handles retries, backoff, model fallbacks and a JSON repair round-trip when a
   model answers with prose instead of JSON.
3. `lib/normalize.ts` coerces whatever the model returned into a strict, constraint-respecting shape -
   it re-trims titles to 80 chars, dedupes and clamps tags to 20 chars, and fills gaps rather than
   throwing away a whole generation.
4. `lib/seo-score.ts` scores the finished gig (title placement, tag limits, keyword density, package
   ladder, image copy, trust blocks).
5. `lib/gig-image.ts` renders the gig images as SVG in three layouts (`split`, `centered`, `badge`) across
   six colour palettes. `components/GigImageStudio.tsx` previews them at thumbnail sizes and rasterises
   to a 1280x769 PNG in the browser - no image API needed.

### Performance note

`deepseek-v4-flash` is a reasoning model. Without tuning it spent ~4,500 reasoning tokens per answer,
which pushed a single generation to ~280s and could empty the completion entirely. Setting
`reasoning.effort: "low"` cuts that to ~400 tokens: the same answer in about half the time. A full pack
currently takes roughly 90-180 seconds, most of it waiting on the free model. Add a fast paid model to
`OPENROUTER_FALLBACK_MODELS` if you want it quicker.

## Scripts

```bash
npm run dev        # development server
npm run build      # production build
npm run start      # run the production build
npm run typecheck  # tsc --noEmit
```

## Deploying

Works on any Node host (Vercel, Render, Fly). Set `OPENROUTER_API_KEY` and `OPENROUTER_MODEL` as
environment variables in the host dashboard - do not commit `.env.local`. Requests can run for two to
three minutes on the free tier, so pick a plan whose function timeout allows it (`maxDuration` is set to
300 in the API route).

## Honest limits

- **No scraping.** Fiverr blocks automated collection and it violates their terms. The research step uses
  the model's knowledge of how gigs rank, plus the marketplace rules encoded in the prompt. Keyword volume
  and competition values are informed estimates, not live Fiverr data.
- **No guarantee of page 1 or a first order in 48 hours.** Gig copy and tags control the relevance half of
  ranking. The other half is response time, samples, pricing, category competition and conversion - which
  is what the 48-hour plan targets. Any tool promising a guaranteed rank is lying.
- **No text in generated images from an AI image model.** The DeepSeek endpoint is text-only, so the gig
  images are rendered locally as vector art. Each concept also ships a `photoPrompt` you can paste into an
  image generator if you want a photographic background instead.
- **Prices are suggestions.** Check current Fiverr pricing for your category before publishing.
