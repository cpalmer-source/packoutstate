# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page marketing site for Pack Out State — NC State Homecoming, Oct 29–31 2026,
at packoutstate.com. Client work for DCC Social Inc.

## Build / run / test

There is none. No `package.json`, no bundler, no dependency install, no test suite, no
linter, no CI. `index.html` is the entire site — inline `<style>`, static markup, inline
`<script>`.

To preview, open the file directly or serve the folder:

```
python3 -m http.server 8000     # then http://localhost:8000/index.html
```

A local server (not `file://`) is preferable: the calendar feature builds `.ics`
downloads with `URL.createObjectURL`, and Google Fonts / the DSEG CDN need network.

"Verify" here means opening it in a browser and checking the manual list in
[Quality gates](#quality-gates) below. There is nothing to run that will tell you the
page is correct.

## Files

| File | Role |
|---|---|
| `index.html` | The live site. Everything. ~900 lines. |
| `ui-preview.html` | Internal component showcase, `noindex`. Not linked from the site. |
| `UI-SYSTEM.md` | The build reference — spacing, grid, BEM rules, anti-patterns. Authoritative. |
| `DESIGN.md` | A visual direction that was built and then **reverted**. Not applied. See below. |
| `hero-carter-finley.jpg`, `flyer-*.jpg` | Hero background and the four event flyers. |

## DESIGN.md is a proposal, not a record

`DESIGN.md` describes a rebuild onto the flyers' warm-bone ground with different
typefaces (Ultra / Anybody / Figtree), a chenille badge wall, and the ticker and
animated sky deleted. **The client reverted all of it.** The live `index.html` is the
original design: Anton headings, dark ground, LED scoreboard countdown, red marquee
ticker, animated dusk sky.

Do not treat anything in `DESIGN.md` as describing current code, and do not "restore
consistency" with it. **The heading typeface is Anton and does not change without
explicit agreement.** The parked `HOMECOMING` vertical spine lives in commit `257f610`
if it is ever asked for.

The one part of `DESIGN.md` that *is* live is its status block at the top, which lists
the few post-revert changes to the real site (Eastern-time date formatting, the
fade-to-black hero, the `--red-bright` hero eyebrow).

## Architecture

### Content is data-driven — edit the arrays, not the markup

Two labeled edit zones at the top of `index.html`'s `<script>` are the single source of
truth for all schedule content:

- `EVENT_START` / `EVENT_END` — drive the countdown, the hero date line, and the
  live / wrapped end-states.
- `DAYS` and `EVENTS` — drive three separate surfaces: the sticky day rail
  (`#dayRail`), the weekend-at-a-glance grid (`#glance`), and the event cards
  (`#events`). Order in the array is order on the page.

Adding a fifth event means editing one object. If a change makes you edit rendered
markup in more than one place, the rendering is wrong, not the data.

Optional per-event fields degrade gracefully — `flyer: null` renders a 3:4 placeholder
well so the layout does not shift; `address`, `mapUrl`, `ticketsUrl`, `detailsUrl`,
`ages`, `onSaleNote`, and `extra` each render their row/button only when present.
`desc` accepts a string or an array of paragraphs.

Everything below the `RENDER — no edits needed below this line` marker is template
logic. All interpolated content goes through `esc()`.

### Times and timezones

Every date is written with an explicit Eastern offset (`-04:00` EDT / `-05:00` EST) and
the hero date line formats with `timeZone: 'America/New_York'`. This is deliberate and
was a bug fix: the event is in North Carolina, so a 9 PM Thursday kickoff must read
"October 29" whether the page is opened in Raleigh or in London. Do not remove the
timezone pinning.

Each event carries machine-readable `start` / `end` alongside the human `time` string.
The display string cannot be parsed — most of these events cross midnight.

### Add to calendar

Built in-browser with no third-party service. `.ics` bodies are assembled as strings and
handed to `URL.createObjectURL`; Google Calendar gets a `render?action=TEMPLATE` URL.
Blob hrefs are wired *after* render, in the IIFE that queries `[data-ics]`.

Two RFC 5545 details are load-bearing and easy to break: `fold()` wraps at **75 octets,
not characters** (a naive character count silently overruns on multi-byte text and
strict clients reject the file), and `ice()` escapes backslash, semicolon, comma and
newline. The `#calAll` button removes itself if no event has times.

### CSS conventions

Vanilla CSS with custom properties and BEM — `.block`, `.block__element`,
`.block--modifier`, plus `is-` / `has-` state classes. There is no Tailwind and no
utility layer; §6 of `UI-SYSTEM.md` has a mapping if migration is ever wanted, but
nothing depends on it.

- Colors and type come from `:root` tokens only — `--red`, `--red-deep`,
  `--red-bright`, `--ink`, `--carbon`, `--bone`, `--white`, `--ash`, `--display`
  (Anton), `--sans` (Archivo). No hardcoded hex outside `:root`, no new
  `font-family` declarations. `DSEG7-Classic` is countdown digits only.
- `--red-bright` (`#FF3B30`) exists because `--red` measures 3.36:1 on the now-solid
  black hero top — under AA. Use it for red text on `--ink`; use `--red-deep` for red
  text on bone/white grounds.
- Never nest element names (`.event__cta`, never `.event__body__cta`). Never style a
  bare element outside the reset. No IDs in selectors, no `!important` outside the
  reduced-motion block.
- JS toggles state classes; it should not write inline styles for anything CSS can
  express. (The hero load-in sequence is the deliberate exception.)

**Spacing scale caveat:** `UI-SYSTEM.md` §1 specifies `--sp-xs` … `--sp-5xl` and a
"never write a raw px value for margin/padding/gap" rule. Those tokens are currently
defined in `ui-preview.html` only — `index.html` still uses raw px throughout and has
not been migrated. When touching spacing in `index.html`, either follow the surrounding
raw-px style or migrate the whole block deliberately; do not leave a component
half-converted.

### ui-preview.html duplicates the tokens

The showcase copies the `:root` token block verbatim from `index.html` and is not
generated from it. **Any token change must be made in both files.** The preview is
`noindex, nofollow` and is not linked from the site — it is a reference for the client,
not a page.

### Progressive enhancement and motion

`IntersectionObserver` handles scroll reveal (`.reveal` → `.is-in`) and the day-rail
scrollspy. Every animation path checks
`matchMedia('(prefers-reduced-motion: reduce)')` — the hero load-in sets elements to
full opacity immediately, and smooth scrolling falls back to `auto`. Any new animation
goes inside that guard.

## Quality gates

Check before committing markup or CSS changes — nothing automates these:

- No horizontal scroll at 360px wide
- `prefers-reduced-motion: reduce` leaves everything at full opacity with no animation
- Visible focus on every interactive element (`:focus-visible` is a 3px `--red` outline)
- Touch targets ≥44px
- Red text meets 4.5:1 against its actual ground — sample rendered pixels rather than
  assuming the backdrop
- No hardcoded hex outside `:root`, no new `font-family`

## Copy is the client's

Event descriptions, taglines, FAQ answers, section headings, leads, and button labels
are the client's words, carried verbatim. Do not rewrite, tighten, or fix them
unprompted — including the two known redundancies ("NC State Homecoming" appearing
twice in the hero, and the ticker restating the date line). Flag, don't edit.
