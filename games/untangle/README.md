# Untangle (21-C C7) — staged content for the scenes repo

Built on 2026-09-19 by core `scripts/author-templates.cjs` (branch `feat/29-c6-template`, `MODULE_DEFS`
includes `untangle`) against the lane at 5216 with the packed `untangle` 2.0.0 zip from modules
`feat/29-c6-dungeon`:

    APP_URL=https://theprototype.app:5216/ MODULES_REPO=<modules checkout with untangle.zip> \
      node scripts/author-templates.cjs --only untangle --out <scratch>

The def is the module's: `modules/untangle/untangle.def.json`, emitted by `npm run build:untangle`
(src/def.js + emit-def.mjs). The scene carries the room (floor, back wall, the pedestal the board
node targets, a ring frame, two lamps), 20 nodes (Untangle Board level 2 / pose, value readouts into
HUD Text, the solved event into a Counter, Start / pause / quit), the 3-screen HUD document, the
sunset env and the play block. The board itself is regenerated from the graph's node on load — the
file carries the LEVEL (seed input), never the positions.

Files: scene.tpscene 8946 B · thumb.webp 2802 B · index-row.json (bytes 8946).

## Release steps (scenes repo, the integrator)

1. `cp scene.tpscene thumb.webp <scenes>/games/untangle/`
2. Append `index-row.json` to the `games` array of `<scenes>/index.json`.
3. Commit on a branch, PR to `main`, merge; `git tag -f format-2 && git push -f origin format-2`;
   purge the jsDelivr cache for `gh/theprototype-app/scenes@format-2/index.json` (+ the two new
   files). Never `v2`: jsDelivr resolves a semver-looking ref ONCE and a retag of it is a no-op
   forever (the root README's Serving section, core #230).
4. THEN add `"template": "games/untangle"` to the `untangle` row of the modules repo `index.json`
   (deliberately NOT in the lane's modules commit, so the Browse card never points at a template
   before it is served). The `untangle` 2.0.0 zip must be on the modules CDN before the tag moves.
