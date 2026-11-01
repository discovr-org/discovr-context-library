# Logo and app icon

**The logo is the full name, always.** There is no monogram, no standalone letter, no abbreviated mark. Wherever discovr appears, the whole word appears: the app icon, the favicon, the splash, social posts, email, the corner of a video. A single letter is never an acceptable substitute at any size.

## The wordmark

The wordmark is the only logo asset. It is **outlined Bricolage Grotesque, weight 700, all lowercase, tracked at -0.03em**, drawn at the 96pt optical size and shaped with the typeface's own kerning.

It ships as outlines rather than live text so it renders identically everywhere, including places that cannot load a font: SVGs inside `<img>`, email clients, PDFs, and other people's platforms.

- Aspect ratio **4.238 to 1**. Fixed. Never stretched or condensed.
- `discovr`, never `Discovr` or `DISCOVR`, including at the start of a sentence.
- Two colourways only: `discovr-wordmark-bone.svg` for dark grounds, `discovr-wordmark-teal.svg` for light ones.

## Clear space and minimum size

- **Clear space** on all four sides is half the wordmark's height. Nothing enters it.
- **Minimum width 80px.** Below that the counters in `s`, `c` and `o` close up and it turns into a grey bar. If you cannot give it 80px, use nothing.

## The app icon

The icon is the wordmark, bone on a full-bleed `surface-canvas` square. It uses a different cut of the same typeface: **weight 800 at the 12pt optical size, tracked at -0.02em.** That is the optical size axis doing exactly what it exists for, opening the spacing and thickening the thin joins so the word survives being small. The aspect ratio at this cut is 4.596 to 1.

**Two files, because Android crops.**

| File | Wordmark width | For |
| --- | --- | --- |
| `discovr-app-icon.svg` | 88% of the square | Manifest `purpose: "any"`, `apple-touch-icon`, favicon, splash |
| `discovr-app-icon-maskable.svg` | 76% of the square | Manifest `purpose: "maskable"` |

A maskable icon is cropped to the inner 80% circle. At 88% width the ends of the word fall outside that circle and Android would clip the `d` and the `r`. The maskable file holds the wordmark to 76%, which puts its corners at a radius of 200 against a safe radius of 205. Do not use one file for both purposes.

## The honest limitation

At a home-screen size of roughly 60px, the wordmark gives you about 10px of ascender height and 7px of x-height. It is legible, but it reads as a shape a user recognises rather than a word they read. That is the accepted cost of the all-name rule, and it is worth knowing so nobody is surprised by it on a home screen.

Two consequences to design around:

- **Favicons at 16px are below the floor.** Ship the SVG favicon so modern browsers scale it properly, and accept that the 16px PNG fallback is a texture rather than a word.
- **The icon needs its dark ground to work.** The contrast between bone and `#061412` is what holds the letterforms apart at small sizes. Never put the wordmark on a mid-tone.

## In the product

- The wordmark appears once per screen at most, and usually not at all. Inside the app the user already knows where they are; the top of a screen is for the screen's own title in `display-md`.
- Use it on: the splash, onboarding, the empty first-run state, the account screen footer, and email.
- On a photo, place it on a `surface-canvas` chip with `radius-pill` rather than knocking it out.

## Do not

- Use a single letter, a monogram, or any abbreviation of the name, anywhere, for any reason.
- Rotate, skew, outline, or add a shadow to the wordmark.
- Recolour it outside the two colourways.
- Set it below 80px wide.
- Re-typeset it in live text when an outlined file will do. Live text will fall back to a different face somewhere and you will not find out until a customer sees it.
- Add a tagline to it. Taglines live in layout, not in the logo.
- Stretch it to fill a space. The ratio is fixed; change the container.

## Before you commit to this name

Two checks worth doing this week, because both can force a rebrand later:

1. A trademark search in your class, travel and booking services, in India. "discovr" and "discover" are likely to be treated as confusingly similar to existing marks, and the dropped vowel does not protect you. This matters more now that the whole word is the logo.
2. Domain and Instagram handle availability for `discovr`.

Neither is legal advice, and a trademark attorney is worth an hour before you print anything.
