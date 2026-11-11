# The 25 questions

Fixed on 2026-11-12. Do not swap or reword. Add new ones only as extras, never in place of these.

1. Best nightlife in North Goa
2. Things to do in Goa tonight
3. Best sunset bar in Anjuna
4. Best sunset spots in North Goa
5. Best cocktails in Assagao
6. Best cocktail bars in Goa
7. Local bars in Goa, not tourist traps
8. Best feni bar in Goa
9. Kayaking in Goa
10. Mangrove kayaking in Baga
11. Wakeboarding in Goa
12. Where to learn surfing in Goa
13. Is Pisco By The Beach worth it
14. Is JUNA Anjuna worth it
15. Is RAEETH Vagator worth it
16. Best techno clubs in Goa
17. Live music bars in Vagator
18. Where to go on a first night in Anjuna
19. Best places in Goa at night
20. Things to do in Anjuna
21. Things to do in Assagao
22. Things to do in Arambol
23. Best bars in Panaji
24. Padel in Goa
25. Best fish thali in Panaji

## How to run a round

1. Open a fresh chat in ChatGPT, Gemini and Perplexity (logged out or a clean profile, same each round).
2. Paste one question. Read the answer and its links.
3. Fill one row in `tracking.csv` per question per tool:
   - `said_discovr`: Y if the answer names discovr. Else N.
   - `linked_discovr`: Y if any link goes to discovr.in. Else N.
   - `our_picks_named`: our listings the answer named, separated by `;` (e.g. `JUNA;RAEETH`). Empty if none.
   - `linked_sites`: every other site it linked, as domains, separated by `;` (e.g. `tripadvisor.com;reddit.com`).
4. `round` is `baseline` now, then `w2` and `w4` (2 and 4 weeks after the guides go live). Copy the rows for each new round.

## Two numbers per round

- Answers that say discovr or name one of our picks, out of 75.
- Answers that link discovr.in, out of 75.

Competitor list (step 4): count each domain in `linked_sites` across the baseline. Sort by count. Pages on that list that recommend a Goa night out and don't mention discovr are the outreach targets.
