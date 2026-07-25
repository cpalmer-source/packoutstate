# Pack Out State — NC State Homecoming 2026

Standalone landing page. No build step, no dependencies.

## Files
- index.html — the entire site (styles + content + logic)
- hero-carter-finley.jpg — hero background
- flyer-*.jpg — event flyers

## Updating content
Open index.html and edit the two labeled zones at the top of the <script>:
1. EVENT_START / EVENT_END — countdown dates
2. EVENTS array — the lineup (add/remove/reorder events)

To swap in a new flyer: add the image file to the repo and set the
event's `flyer` field to its filename.
