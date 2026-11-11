# AI answer visibility

Goal: discovr named and linked when people ask ChatGPT, Gemini, Perplexity about Goa.

- Hub copy: `discovr-frontend/src/features/hubs/utils/guides.ts` (answer, table, FAQs per hub; `AS_OF` date). Add an entry for every new hub. Keep `public/llms.txt` in sync.
- `robots.ts` allows the named AI crawlers.
- Questions and how to run a round: `questions.md`. Results: `tracking.csv`.
- Re-ask at 2 and 4 weeks after the guides go live.
