# Dungeon Realms (21-C C6-b) — staged content for the scenes repo

Built on 2026-09-19 by core `scripts/author-templates.cjs` (branch `feat/29-c6-template`, the
`MODULE_DEFS` loader region + `thumb.sceneGroups`) against the lane at 5216 with the packed
`dungeon` 2.0.0 and `dungeon-realms` 2.0.0 zips from modules `feat/29-c6-dungeon` (@9d40c2d+):

    cd <core checkout>   # feat/29-c6-template merged
    APP_URL=https://theprototype.app:5216/ MODULES_REPO=<modules checkout with dungeon.zip + dungeon-realms.zip> \
      node scripts/author-templates.cjs --only dungeon-realms --out <scratch>

The def is the module's: `modules/dungeon-realms/dungeon-realms.def.json`, emitted by
`npm run build:dungeon-realms` in the modules repo (src/def.js + emit-def.mjs — the entrance
arch is placed at the entrance room seed 1337 produces). The scene carries 6 objects (the arch),
34 nodes (the Dungeon Kit recipe wired to the plinth, rules, menu, HUD readouts, game-shell
events, pause), the 4-screen HUD document, the night env, the play block and both modules in
its requirement list; the world itself is regenerated from the graph's Dungeon node on load.

Files: scene.tpscene 9169 B · thumb.webp 3848 B · index-row.json (bytes 9169).

## Release steps (scenes repo, the integrator)

1. `cp scene.tpscene thumb.webp <scenes>/games/dungeon-realms/`
2. Append `index-row.json` to the `games` array of `<scenes>/index.json` (after `jam-room`).
3. Commit on a branch, PR to `main`, merge; then the serving ritual:
   `git tag -f v2 && git push -f origin v2` and purge the jsDelivr cache for
   `gh/theprototype-app/scenes@v2/index.json` (+ the two new files).
4. The modules `index.json` row for `dungeon-realms` already says `"template": "games/dungeon-realms"`
   — it becomes live the moment the tag moves. Both module zips (dungeon 2.0.0, dungeon-realms 2.0.0)
   must be on the modules CDN (`main`) BEFORE the tag moves, or the Games card installs 1.x.
