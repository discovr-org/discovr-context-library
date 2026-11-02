# Interface rules

Everything to keep in mind while designing screens for discovr. Written for a dark-only, mobile-first web app. Every rule has a reason attached, so change the reason before you change the rule.

## The five that matter most

1. **One primary action per screen.** `brand-primary` appears once. The moment two bone buttons are in view, neither is the primary.
2. **Uniform geometry beats colour.** Fixed aspect ratio, fixed radius, fixed gap. This is what makes a feed of unrelated photos read as one product.
3. **Every photo gets `photo-lift`**, and every photo has its ratio reserved before it loads. The one exception is a photo of a document, such as a menu page, which keeps a hairline border instead so its bottom lines stay readable.
4. **85% of every screen is neutral.** Bone and pink are events, not surfaces.
5. **Nothing renders light by accident.** A dark-only site still sits inside a browser full of light defaults. See the web section below; this is the failure mode unique to your setup.

## Layout

- **Gutter** is `space-4` (16px). Never less, because `body` at 16px needs the measure.
- **Vertical rhythm**: `space-2` inside a component, `space-3` between items in a grid, `space-5` between cards in a single-column feed, `space-6` between sections.
- **Two-up grid** for browsing, single column for a recommendation. Never both in one scroll.
- **Thumb zone.** The bottom third is where the hand lives. Primary actions, the booking bar and the tab bar go there. Search and filters can sit at the top because they are deliberate, occasional actions.
- **Sticky booking bar** on any detail screen once bookings launch, `surface-raised` with a `line-subtle` top edge. Price left, action right. It never scrolls away, because the decision to book happens at an unpredictable scroll position. While bookings are off it is the call-to-book form, and it shares the bottom of the screen with the TabBar.
- **Chrome on a detail screen.** Past the hero gallery, the AppBar and TabBar hide while scrolling down and come back on scroll up. They always move together.
- **Sheets over new screens** for anything under half a screen: dates, guests, filters.
- **No horizontal scroll** on the page body. Wide content scrolls inside its own container.
- **Desktop.** It is a website, so people will open it on a laptop. Cap the content column at about 480px and centre it rather than stretching a phone layout across 1440px. A three-up grid above 900px is the only layout change worth making for the MVP.

## The detail screen

The screen an ExperienceCard opens. Until bookings launch, discovr is discovery only and hands the user to the host, so the screen ends in contact, not checkout. Its order is fixed:

1. Hero gallery: every photo for the listing, full bleed, swiped one at a time, with a "1 / 3" count. No text on the photos. The only other photo is the one at the end of About.
2. AppBar over it: back, the title (shown once the gallery has scrolled away), share, save.
3. Title in `display-md`, `space-5` below the gallery. Experiences add the byline "by Surf Wala" in `body` `ink-muted`, `space-1` below. Places show the name only.
4. InfoList, `space-5` below: for experiences — price per person (+ taxes / inclusive); the area pin; "Open today" and the slot times; duration and group size. For places — no price or people row; the area pin; open/closed now with today's hours (expand for days / notes).
5. Verdict, `space-5` below: quote-only on a hairline rule, no label. One or two sentences in our voice.
6. ActionRow, `space-5` below: Call, Directions, Instagram, Share (Instagram omitted only when there is no handle).
7. A `line-subtle` rule `space-6` below, then the long-form sections `space-6` apart, each a `title` heading with `space-3` to its content. Blocks with no data are skipped.
   - **Place (bar, restaurant):** About (description in italics, then a photo); Price (the sheet's line, e.g. "₹3,000 for two (approx)"); Menu (cover + "7 pages"); Know before you go and FAQs only when the listing has them.
   - **Experience:** About (italic short pitch, description, photo); Details (Age, Fitness level, Student to instructor ratio); What's included (ticks) and What's not included (crosses) as two boxes side by side; The process (1 Find it, 2 Call and book, 3 Show up); Health advisory (the one `state-notice` box); Things to get with you (checklist); Know before you go (tips box; falls back to the booking and cancellation notes plus the final-terms line); FAQs (tap to open); "Run by / Hosted by / Owned by {host}" card with the verified line.
   - **Both, last:** the Disclaimer box ("Read the full disclaimer"), the "Report wrong information" and "Any problems with the experience? Contact us" links, then More in Goa as a 2-column grid.
8. BookingBar. Experiences: "From ₹3,000 per person", "Call to book", and "or book on {host}" when the listing has a booking link (else who picks up). Places: "Ask for the price" and "Call to book". Fixed at every scroll position.
9. TabBar, hidden while the gallery is on screen; past it, it and the AppBar hide on scroll down and come back on scroll up. When it appears, the BookingBar rides up on top of it.

**Key details before the verdict.** The ExperienceCard has already shown the verdict, so someone who taps through is deciding whether they can go: price, place and time answer that. The verdict follows immediately, still above the fold on a 390 by 844 screen, so the opinion is never more than a glance away.

No ratings and no reviews: everything listed is a pick, and the verdict carries our view. No badges while there is no inventory to be honest about. No meeting point section: the place row in the InfoList and the Directions tile already cover it. Place vs experience comes from `content.format` (`venue` / `trip`); without it, tags decide. When bookings launch, the BookingBar switches from calling to its booking states and the TabBar behaviour stays the same.

States: populated, loading (skeletons at the gallery, title, InfoList, verdict and ActionRow dimensions, with the bars shown), offline (notice Banner above the title), error (EmptyState in place of the content, "Try again" as a secondary Button).

## Web and PWA

The rules that only exist because this runs in a browser. Most dark-mode bugs live here.

- **Declare the scheme twice**: `<meta name="color-scheme" content="dark">` and `color-scheme: dark` in CSS. Without it, form controls, scrollbars, date pickers and `select` menus render in the browser's light chrome and look like somebody else's app dropped into yours.
- **Autofill will override your input styling.** Chrome forces its own background on autofilled fields. Handle `:-webkit-autofill` explicitly or your payment form turns pale yellow at the worst possible moment.
- **`theme-color` is `surface-canvas`.** It tints the browser chrome and the status bar area in standalone mode. Getting this wrong leaves a white band above your dark app.
- **Use `dvh`, not `vh`.** Mobile browser chrome grows and shrinks on scroll, and `100vh` is measured against the wrong one. A sticky booking bar positioned with `vh` will sit under the address bar.
- **`viewport-fit=cover` plus `env(safe-area-inset-*)`.** In standalone mode there is no native shell handling the notch and the home indicator for you.
- **Test in real Safari on a real iPhone.** It is not Chrome, it lags on features, and it is a large share of this audience.
- **Offline matters more than usual.** A user on a beach with no signal still needs their booking confirmation. Cache confirmed bookings in the service worker and make the tickets view work offline. This is the single highest-value PWA feature for a travel product.
- **iOS has no install prompt.** `beforeinstallprompt` does not fire in Safari, so you need a designed "Add to Home Screen" instruction. Show it after a successful booking, never on first load.
- **Push needs an installed PWA on iOS.** Do not build a notification strategy that assumes reach you will not have.
- **Performance is a design constraint.** Your users are on mobile data with patchy coverage. Budget the hero image, self-host and subset the fonts, and inline the critical CSS. A beautiful screen that takes six seconds is a worse screen.

## Components, and their states

Twenty-nine components, grouped. Full specs and rules live on each component's own page; this is the map and the choosing rules.

| Group | Components |
| --- | --- |
| **Controls** | Button, IconButton, ActionRow, Chip, SelectionControl, Stepper, SegmentedControl, Input |
| **Navigation** | AppBar, TabBar |
| **Content** | ExperienceCard, Hero, InfoList, FactList, MenuPages, Review, Badge, Avatar, Rating, Skeleton, EmptyState, Disclaimer |
| **Feedback** | Sheet, Dialog, Toast, Banner |
| **Commerce** | BookingBar, PriceBreakdown, PolicyDisclosure |

**The choices people get wrong.** Each of these pairs looks similar and means something different:

- **Chip or SegmentedControl.** Chip is many of many and additive; SegmentedControl is one of few and exclusive. If a user could want two at once, it is a Chip.
- **Checkbox or Toggle.** A toggle applies the moment it moves. If the change needs a Save button, it is a checkbox.
- **Toast, Banner or Dialog.** Toast: already happened, nothing required, it disappears. Banner: an ongoing condition, stays until it ends. Dialog: a decision is required before anything else. Never put an action a user must take into a Toast.
- **Sheet or Dialog.** A Sheet is a choice within the current task and can be dragged away. A Dialog blocks until answered, and only a real consequence earns one.
- **Skeleton or Spinner.** Skeleton for content the user is waiting to read. Spinner for an action the user took.
- **Badge or Chip.** A Badge states a fact and is not tappable. If it needs a tap, it is a Chip.
- **InfoList or FactList.** InfoList is the four at-a-glance lines under a title, on the canvas. FactList is a raised card of facts with sub-lines, further down. If it needs a sub-line, it is a FactList.
- **PolicyDisclosure or Disclaimer.** PolicyDisclosure is this host's terms, which people need before they call, so it's a raised card. Disclaimer is discovr's standard text, the same on every listing, so it's a quiet footnote. Never style one like the other.
- **FactList or body copy.** A FactList is for facts a user acts on: when, where, who, the terms. If it needs a verb and a second sentence, it is a paragraph under a `title` heading.

**What only appears once per screen:** one `brand-primary` fill, one AppBar, one Hero, one BookingBar, one TabBar, one Toast. On a detail screen the BookingBar stacks above the TabBar; nothing else shares the bottom edge.

**Every interactive component needs six states**, and most design work ships with two. Draw all six or the engineer invents them: default, hover, focus-visible, pressed, disabled, loading.

- **Focus-visible** is `focus-ring` at 2px with a 2px offset. Never remove it. On the web this is not optional politeness; keyboard users on desktop are a real share of a website's traffic.
- **Hover is a desktop state.** It must never be the only way to reveal anything, because half your traffic has no pointer.
- **Pressed is the state that matters on touch.** Give every control a visible active state or the app feels dead on a phone.
- **Disabled** is `opacity-disabled` on the whole component, not a grey fill.
- **Loading** on a button keeps its width and swaps the label for a spinner.

**Every screen needs four states** too: populated, empty, loading, error. The empty state is a design surface, not a fallback.

- **Loading** uses `surface-sunken` skeletons at the exact dimensions of the real content. No spinners for content, only for actions.
- **Add an offline state** to that list for any screen a user might open with no signal.

## Dark-only specifics

One theme is less work, but dark interfaces fail in ways light ones do not:

- **Pure black and pure white are both wrong.** `#000` makes OLED smearing visible when scrolling, and `#fff` text on a dark ground haloes, appearing to glow and blur. This is why `surface-canvas` is `#061412` and `ink-primary` is `#eaebe7`.
- **The heavier the type, the more halation shows.** Do not reach for white on a display heading.
- **Avoid large saturated fills.** A full-screen `accent-pink` panel on a dark app is physically uncomfortable. Pink is for badges and dots.
- **Shadows barely work.** Separate elements with a surface change or `line-subtle`. Reaching for a third shadow level means the thing should be a sheet.
- **Test at low brightness in the dark and at full brightness in sunlight.** Those are the actual usage conditions for a travel app, and mid-brightness indoors is the one condition you will design in.
- **Contrast is now checked once**, which is the real saving from dropping the light theme. The token usage notes carry the measured ratios.

## Accessibility, as a design constraint rather than an audit

- **4.5:1** for text, **3:1** at 24px and above, **3:1** for control borders, focus rings and meaningful icons.
- **44 by 44** minimum tap target, always. Space targets at least `space-2` apart.
- **Never colour alone.** Confirmed states get the word. Errors get text, not a red border.
- **Text scaling to 200%**, plus browser zoom to 400% on desktop. No fixed heights on anything containing text.
- **Real elements.** `<button>`, `<a href>`, `<input>` with a `<label>`. A div with a click handler is invisible to keyboards and screen readers. `aria-label` on every icon-only button.
- **Keyboard order follows visual order**, and every interactive element is reachable by tab. On a website this is table stakes.
- **Reduced motion.** Respect `prefers-reduced-motion` and drop to opacity-only transitions.
- **Dark-only is a small accessibility cost.** Some users with astigmatism read dark-on-light more comfortably. For a photo-led app with little long-form text the cost is low, but it is a reason to keep `body` generous and never go below `ink-subtle`.

## Motion

- 180ms `cubic-bezier(0.2, 0, 0, 1)` entering, 120ms linear leaving. Entering is slower because arrivals need noticing and departures need to get out of the way.
- Sheets slide from the bottom edge they will return to.
- **No motion on content.** Cards do not animate in on scroll. It looks expensive in a demo and feels slow on the hundredth scroll.
- Opacity and transform only. Animating layout properties drops frames on the mid-range Android phones a lot of your users have.

## Copy inside the interface

- Sentence case everywhere except `caption`, which is caps.
- No full stop on buttons, labels, chips or single-line headings.
- Numerals as digits. "4 spots", "3 hours", "6:30pm".
- Second person. "Your booking", never "My booking".
- Buttons name the outcome, not the mechanism. "Book for tonight", not "Continue".
- Errors say what happened, whether money moved, and what to do next, in that order.
- No emoji in the interface.

## Booking and money

This is a payments product, and the interface carries trust or destroys it.

- **Show the all-in price** from the first screen. If a fee appears later than the first price a user saw, you have lied to them.
- **State the cancellation terms next to the button**, not on a policy page. While bookings are off, the detail screen's PolicyDisclosure carries them, with the free-cancellation cut-off visible even when it is closed.
- **Never fake scarcity.** Show a spots-left count only when the number is real and under five.
- **Confirmations carry three facts**: when, where, and that it is paid. Everything else is secondary. And they must be readable offline.
- **Failures reassure about money first.** "Nothing was charged" comes before the retry instruction.
- **Never block on account creation** before someone has seen what they are buying.
- **Support UPI**, and remember payment flows on the web often leave your tab. Design the return: a user who comes back from a bank page must land on a state that tells them what happened, not a blank app shell.

## Anti-patterns

- Two bone buttons in view.
- A tint or border in a photo's colour on the card containing it.
- Full-bleed photos in a feed.
- Text sitting directly on an image with no scrim.
- Icon-only tab bars.
- Carousels as the primary browse mechanism.
- Modal onboarding before the first screen of content.
- A spinner where a skeleton belongs.
- Pure white type on `surface-canvas`.
- `100vh` anywhere.
- Any value not on the spacing scale.

## Before you ship a screen

- One primary action, and only one.
- Every photo carries `photo-lift` and a reserved aspect ratio (menu pages excepted, see MenuPages).
- All four screen states drawn, plus offline where it applies.
- All six states drawn on every interactive element.
- Contrast checked, tap targets 44 by 44, tested at 200% text.
- Keyboard reachable in visual order, focus ring visible throughout.
- Checked in real Safari on a real iPhone, and in standalone mode after installing.
- No raw hex values in the code. If a colour is not a token, it is a bug. The same now goes for `44px`, `52px`, `180ms` and `z-index: 50`: control heights live in the `size` family, timings in `motion`, layers in `zIndex`.
