# scenes

Curated starting scenes for [theprototype.app](https://theprototype.app) — the
content behind the **Templates** modal (logo menu ▸ Templates). Maintainer-curated;
community submissions go to
[community-gallery](https://github.com/theprototype-app/community-gallery) instead.

## Layout

```
index.json                     the manifest the app fetches
templates/<slug>/scene.tpscene starting-point scenes (General tab)
templates/<slug>/thumb.webp    480x270 card thumbnail
examples/<slug>/...            curated showcase scenes (Examples tab)
games/<slug>/...               playable games (Games tab)
contests/<slug>/scene.tpscene  contest STARTER scenes ("Start from the starter" on a contest page)
contests/<slug>/thumb.webp     480x270 card thumbnail
contests/<slug>/contest.json   the brief, rules, window and credits the contest page reads
```

`index.json`:

```json
{
	"version": 2,
	"templates": [
		{
			"slug": "level-blockout",
			"title": "Level blockout",
			"description": "…",
			"author": "theprototype",
			"license": "CC0-1.0",
			"bytes": 3279,
			"scene": "templates/level-blockout/scene.tpscene",
			"thumb": "templates/level-blockout/thumb.webp"
		}
	],
	"examples": [],
	"games": [
		{
			"slug": "towers",
			"title": "Towers",
			"description": "…",
			"author": "theprototype",
			"license": "CC0-1.0",
			"tags": ["physics", "stacking", "co-op", "vr"],
			"modules": [{ "id": "collectible", "version": "1.0.0" }],
			"bytes": 13166,
			"scene": "games/towers/scene.tpscene",
			"thumb": "games/towers/thumb.webp"
		}
	]
}
```

**Format 2** adds the `games` array. A game row is a template row plus `tags` and
`modules` — the modules the app installs before it loads the scene. The section is
optional: the app treats an absent `games` key as an empty tab, so a v1 index still
loads in a current build.

- `scene`/`thumb` are repo-relative; absolute `https://` URLs pass through
  untouched (use `raw.githubusercontent.com` for files over ~20 MB — the
  jsDelivr per-file cap).
- A `.tpscene` is the app's session zip (`session.json` + optional `assets/`):
  save one from the app via logo menu ▸ Save ▸ Scene.

## Contests

`contests/` holds the STARTER scene of each contest plus the text its contest page
shows. The index lists them in a `contests` array — optional like `games`, and **not
read by the app**: the cloud repo's `scripts/seed-contests.mjs` reads it from a
checkout, creates the `contests` records and uploads each starter under the org
account, so the serving tag is not moved for a contest.

```json
	"contests": [
		{
			"slug": "make-a-mirror",
			"title": "Make a mirror",
			"description": "…",
			"author": "theprototype",
			"license": "CC0-1.0",
			"tags": ["contest", "primitives", "co-op"],
			"bytes": 12345,
			"scene": "contests/make-a-mirror/scene.tpscene",
			"thumb": "contests/make-a-mirror/thumb.webp",
			"contest": "contests/make-a-mirror/contest.json"
		}
	]
```

`contest.json`:

| key | meaning |
| --- | --- |
| `slug`, `title` | the contest's id and name (the same as the index row) |
| `brief` | markdown, under 1500 chars — what to build and how it is judged |
| `rules` | markdown, under 1500 chars — what an entry must and must not do |
| `starter` | repo-relative path of the starter scene (always `contests/<slug>/scene.tpscene`) |
| `durationDays` | how long the contest stays open |
| `opensAfterDays` | offset from the FIRST contest's opening day; contests overlap by a week (0, 7, 14, …) |
| `judging` | one line: how winners are picked |
| `credits` | `[{ what, title, author, license, source }]` — every third-party asset the starter ships (a CC0 track, say), with its source URL |

A starter that ships a track carries the bytes inside the `.tpscene` (`assets/`), so
every entry plays the same file; the credit line above is where its license lives.
Both starters here are authored by the core repo's `scripts/author-templates.cjs`
(`--out <this checkout> --only make-a-mirror,follow-the-beat`) — the `kind: 'contest'`
defs are the source of truth for the scenes, the briefs and the rules.

## Serving

The app fetches `index.json` via the **ref-pinned** jsDelivr mirror
(`https://cdn.jsdelivr.net/gh/theprototype-app/scenes@format-2/...` — see `SCENES_BASE`
in the core repo's `src/lib/sceneTemplates.js`). jsDelivr caches a ref for up to
12 hours, so content releases are: commit → re-point the ref → purge → the app
picks it up without a redeploy.

```
git tag -f format-2 && git push -f origin format-2     # re-point the serving ref
curl https://purge.jsdelivr.net/gh/theprototype-app/scenes@format-2/index.json
# ...and each NEW file path under the ref (games/<slug>/scene.tpscene, thumb.webp)
```

**THE REF MUST NOT LOOK LIKE A VERSION.** jsDelivr parses a ref such as `v2` as a
SEMVER VERSION (its data API lists this repo as `versions: ["1","2"], tags: {}`), and a
version's files are cached PERMANENTLY (`cache-control: immutable`, one year) — a retag
of `v2` is a NO-OP forever, and `purge.jsdelivr.net` reports `finished` without
re-resolving it. Measured 2026-09-20 (core issue #230): 16 hours and four purges after
the Jam Room retag, `scenes@v2/index.json` still listed three games while `@main` listed
six. A ref that cannot be parsed as a version — tag or branch — is reported as
`x-jsd-version-type: branch` with a 12-hour `s-maxage`, which is what makes the
retag-and-purge ritual work. That is why the serving refs are `format-N`, never `vN`.
`v1` and `v2` are DEAD: they stay exactly where jsDelivr first resolved them, for the
builds that shipped against them.

The ref name tracks the INDEX FORMAT, and a format bump takes a NEW ref — never
reuse an old one, because builds already in the wild keep reading the ref they were
built against. `SCENES_BASE` in core and the serving ref here must move together:
core pointing at a ref that does not exist yet is a 404, and the app falls back to
the small bundled seed with an empty Games tab.

## Adding a scene

1. Build the scene in the app (primitives keep it small; assets bundle into the
   zip when saved with assets enabled).
2. Save as `.tpscene`, drop it under `templates/<slug>/`, `examples/<slug>/` or
   `games/<slug>/` with a `thumb.webp` (480x270).
3. Add its row to `index.json` (`bytes` = the file size; a game also lists the
   `modules` it needs).
4. Commit, re-point the tag.

The core repo's `scripts/author-templates.cjs` regenerates the seed templates
programmatically and can write this repo's tree via `--out`.

## License

Content in this repo is [CC0 1.0](LICENSE) unless a scene's `license` field
says otherwise.
