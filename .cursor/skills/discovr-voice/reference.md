# Voice and copy

discovr sounds like a friend who has lived in Goa for two seasons, has opinions, and is slightly amused by tourists. Short sentences. Specific nouns. A view.

## Stance

**The friend who lives there.** Knows the place. Has a stake in you going out and no stake in which thing you pick, because everything on the list is something they would go to themselves. Never oversells.

That second clause matters and it is easy to get wrong. It is tempting to write the stance as "no stake in you going anywhere", which sounds purer and is false: this is a business that makes money when you book. The version above is the honest one, and it is also the stronger one, because it is checkable. It is only true for as long as principle 2 holds.

## The five rules

A voice is only real when it is a set of rules with rejected alternatives. These are the rules.

1. **Say the thing, then stop.** No second sentence of enthusiasm.
2. **Specific over evocative.** "Opens at 7, gone by 9" beats "a hidden gem".
3. **Never hype what you are recommending.** Let the detail do it. A superlative is allowed only because we went: "the best fish curry in Assagao" is a view someone earned, not a brochure adjective.
4. **Contractions always. Sentence case always.** Copy is authored in sentence case; the `caption` style applies caps as a visual treatment, so nobody ever types a string in capitals. That also stops screen readers spelling out "ONLY" letter by letter.
5. **Never call the user a traveller, explorer, wanderer or nomad.** Address them as "you" or not at all. The rule is about addressing the user; describing the product to your own team is a different job.

Emoji: **zero in the interface.** One maximum in push and social, and only when it carries information rather than tone.

## The same string, three ways

A push notification when someone is near something good.

- **Brochure voice:** "Discover hidden gems near you and unlock unforgettable experiences!"
- **Startup-friendly voice:** "Hey! There's something cool nearby, wanna check it out? 👀"
- **Ours:** "You're 4 minutes from the best fish curry in Assagao. It closes at 3."

The third one works because every word is a fact a person can act on: how far, what it is, and the deadline. The first two are adjectives about how the sender feels. Run any new string through this comparison; if yours sounds like either of the first two, it is not finished.

## Register

**Plain Indian English.** Not tourism-board English, not American startup English.

- "Anjuna, 2 km away", not "conveniently located in the heart of North Goa"
- "Speaks Konkani", not "local expert"
- "3 hours", not "approximately three hours of immersive exploration"

**On Hinglish.** It is the real register of this audience and it does not belong in the interface. Product English stays plain and neutral, because a booking flow is not the place to perform. Hinglish can live in push notifications and social, on one condition: it is written by someone who actually speaks it. Code-switching written by someone guessing reads as pandering, and this audience detects it instantly. If nobody on the team writes it natively, do not write it at all.

**No borrowed slang.** Not "no cap", not "slay", not "bussin". Slang dates in months and reads as an adult approximating. The voice is young because it is direct and specific, not because of its vocabulary.

**Numbers.** Indian digit grouping: `₹1,400`, `₹12,000`, `₹1,00,000`. Symbol before the number, no decimals unless the price has them. Times as `6:30pm`, not `18:30`.

## Having an opinion

This is what separates discovr from an aggregator, and since every listing has been visited, it applies to every listing. A card with no view is a card that should not be on the list.

Every listing carries one line of verdict: what makes it worth an evening, or what to know before going. One sentence. A real view, including an unflattering one.

Good:
> The mangroves are quiet enough that you hear the birds. Go at 6:30, not 8, or you lose the light.

> Touristy and worth it anyway. Skip the buffet.

> Three of the six stops are shops. Fine if you want the shops.

Bad:
> A magical journey through Goa's hidden waterways.

> An unforgettable experience you will treasure forever.

The second pair are not opinions, they are adjectives with nothing behind them. If the verdict could be written about any listing, delete it.

**Never write a verdict for something nobody visited.** A fabricated opinion is the same failure as a fabricated spots-left count, and it is the one that would cost the most. If a listing has no verdict, the answer is to go, or to drop it.

## Words we use and avoid

| Use | Avoid |
| --- | --- |
| Experiences, things to do | Excursions, attractions, activities |
| Book, Reserve | Purchase, Acquire, Avail |
| Host, guide | Vendor, operator, supplier |
| Free cancellation until 6pm | Flexible cancellation policy |
| Sold out | Currently unavailable |
| Checked | Verified, certified, approved |

Never: "curated" (we do the thing, we do not claim the label), "hidden gem", "unforgettable", "bucket list", "immersive", "authentic", "vibes", "elevate", "unlock".

## Talking about groups

Getting these wrong makes a solo traveller feel like a spare part.

- **State solo as normal.** Never "single", never "1 adult" as a silent default, never a price labelled "per person" in a way that makes one person look like an incomplete order.
- **Group size is a feature, so name it.** "Small group, usually six" beats "intimate group experience".
- **Never promise friendship.** "You will meet your people" is a claim we cannot keep. "Six people, one guide, three hours" is a fact that lets someone decide for themselves.

## The strings that set the tone

Write these and the rest of the app follows.

**Onboarding**
> Tell us what you're into. We'll skip the rest of Goa.

The clearest statement of "eight things, not eight hundred" anywhere in the product. It sells the subtraction rather than the selection.

**Location permission**
> We need your location to tell you what's close. That's the whole reason.

The best string in the set. No justification, no reassurance about privacy policies, just the reason, which is the only thing that earns a permission.

**Primary button**
> Book 6:30pm

Names the outcome and the time. Not "Proceed to checkout", not "Continue".

**Empty state, nothing saved**
> Nothing saved yet. Add a few and we'll tell you when the price drops.

The original of this said "we'll build your day around it", which is better writing and describes an itinerary feature that does not exist. One mechanic that exists beats a promise that does not.

**No results**
> Nothing open right now. Goa runs on its own clock.
> Three things start after 8pm.

The first line is the voice. The second line is the job. On its own the joke leaves someone with nowhere to go, and a no-results state has to name the exclusion and offer a route out.

**Error, generic**
> That didn't load. Trying again.

"Our fault, not your network" is a good instinct and a claim you can only make when you know it is true. When the request actually failed on your side, say it: "That didn't load. Our fault, not your network." When you cannot tell, do not guess, and when the device is offline, say that instead.

**Error, payment declined**
> Your bank declined that card. Nothing was charged. Your 6:30pm spot is held for 10 more minutes.

What happened, whether money moved, what to do next. Money errors never get the light touch.

**Confirmation**
> You're in. Saturday 6:30pm, north gate at Ashwem.
> We texted the details to +91 98765 43210.

The facts a person needs, in the order they need them.

**Home page headline**
> Goa, from people who actually go out.

**Meta description**
> Eight things worth doing tonight, not eight hundred. We've been to all of them.

**Instagram caption**
> Three places in Anjuna that aren't in your friend's saved folder.

This one is doing strategy, not just tone. The saved folder is the competitor named in the positioning, and the caption attacks it by name without ever sounding like an ad.

## Mechanics

- **Sentence case in every string you write.** Caps are a typographic treatment applied by the `caption` style, never something typed.
- **Contractions always.** "You're in", not "You are in". "That didn't load", not "That did not load".
- No full stop on buttons, labels, chips or single-line headings. Full stops on body copy and multi-sentence microcopy.
- Numerals as digits, always.
- Second person, and never a noun for the user. "Your booking", not "My booking" and not "the traveller's booking".
- One exclamation mark per screen at most, and usually zero.
- **Emoji: zero in the interface.** One maximum in push and social, only when it carries information.

## Push notifications

The most intrusive surface you own, so the bar is a fact the user did not have. Distance, time and deadline do the work.

Good:
> You're 4 minutes from the best fish curry in Assagao. It closes at 3.

> Rain in Anjuna tomorrow. Your kayak has free cancellation until 6pm tonight.

> Two spots left on the drum circle you saved. It starts at 6pm.

Bad:
> Don't miss out on amazing experiences near you!

> Hey! There's something cool nearby, wanna check it out? 👀

The second bad one is worth looking at twice, because it is the failure mode a young brand actually falls into. It is friendly, it is casual, and it contains no information at all. Friendliness is not the voice; specificity is.

## Where personality stops

Discovery gets the voice. **Checkout, cancellation and tickets get duller on purpose.** Anywhere a user is paying or reading a ticket, the copy is plain and the layout is plain, because a joke next to a ₹2,746 total reads as evasion. Personality in the funnel is a tell.
