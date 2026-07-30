# Pack Out State — NC State Homecoming 2026

Standalone landing page. No build step, no dependencies, no third-party CDN.

## Files
- `index.html` — the entire site (styles + content + logic)
- `fonts/` — self-hosted webfonts (Ultra, Anybody, Figtree; latin subset, ~92 KB)
- `hero-carter-finley.jpg` — background for the "Every class" band
- `flyer-*.jpg` — event flyers
- `DESIGN.md` — the design direction: palette, type, layout, signature element

## Updating content
Open `index.html` and edit the two labeled zones at the top of the `<script>`:

1. **EVENT_START / EVENT_END** — countdown dates. Times are Eastern.
2. **EVENTS** — the lineup. Order in this array is the order on the page and
   the order of the badges. Keep events in the order they actually happen.

### Per-event fields
| Field | What it does |
|---|---|
| `short` | The name on the badge. Keep it to about two short words. |
| `note` | The line under the badge. Falls back to `ticketsLabel`, then `onSaleNote`. |
| `tone` | `'dark'` gives the event a full-bleed dark chapter. Use it once — it only works because it's rare. Currently on The Talley Tapes. |
| `flyer` | Image filename. `null` shows a 3:4 placeholder so the layout doesn't shift. |
| `extra` | Optional expandable block: `schedule`, `highlights`, `why`, `faq`. |

To swap in a new flyer, add the image to the repo and set the event's `flyer`
field to its filename.

## Fonts
Self-hosted on purpose — no render-blocking request to Google Fonts, and the
page works offline. To update a face, drop the new `.woff2` into `fonts/` and
adjust the matching `@font-face` block at the top of the `<style>`.

Ultra, Anybody, and Figtree are all licensed under the SIL Open Font License.

## Checks worth re-running after edits
- Dates render in Eastern time regardless of the visitor's timezone
- No horizontal scroll at 360px wide
- Badges are reachable by keyboard and visibly focused
- `prefers-reduced-motion` leaves everything visible with no animation
