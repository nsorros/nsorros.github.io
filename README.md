# nsorros-site

Personal landing page for Nick Sorros. Built from the **personal brand handoff**
in `~/Downloads/design_handoff_personal_brand/` — monochrome, JetBrains-Mono-only,
terminal aesthetic with a 16×16 pixel portrait and blinking `▮` brand mark.

## Stack

Plain HTML + CSS + ~30 lines of vanilla JS. No build step. No framework.
Deploys cleanly to GitHub Pages, Vercel, Netlify, Cloudflare Pages.

## Structure

```
index.html      single page, all CSS + pixel portrait inline
assets/         favicon
```

## Local preview

```
python3 -m http.server 8765
open http://127.0.0.1:8765
```

## Sections (per the handoff)

1. Hero — 3-col (meta / title+sub+CTAs / pixel portrait) + shell code block
2. About — pull-quote + body + key/value side panel
3. Selected work — 4 entries (Stealth · Vertical AI / Mantis NLP / Wellcome / earlier)
4. Writing & Talks — 10 most recent, links to nsorros.com archive
5. Trusted by — text-logo grid
6. Contact — `> ssh` block + `$ compose` mailto button
7. Footer — `built in vim · deployed $(date)`

## Hard rules

- JetBrains Mono only (300/400/500/600/700 + 400-italic). Font features
  `ss01 ss02 calt zero` enabled.
- Monochrome. Only non-mono color is the green online dot (`#16a34a`).
- No border-radius. Sharp corners everywhere.
- Brand `▮` and CTA `_` blink on a 1.2s cycle.
- Sticky topbar with backdrop blur. Status dot has a halo.
- Section markers: `[NN] ─ ─ ─ // section_name`.
- Light is default; dark mode toggle respects `prefers-color-scheme` on first
  visit and persists to `localStorage`.

## Things that still need real input

- **Pixel portrait** is the abstract head from the handoff. Replace with a real
  16×16 likeness via Aseprite/Piskel — paste the new map into `PIXEL_MAP` in
  `index.html`. Keep it 1bpp + one highlight (no extra shades).
- **Trust By logos** — confirm the eight names are ones to publicly reference;
  swap to greyscale SVGs if real logos are available.
- **Twitter handle** — `@nsorros` is a guess; confirm or remove.
- **PGP fingerprint** — handoff suggests including one; intentionally omitted
  here. Add if you want it.
