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

## Serving

The app fetches `index.json` via the **tag-pinned** jsDelivr mirror
(`https://cdn.jsdelivr.net/gh/theprototype-app/scenes@v2/...` — see `SCENES_BASE`
in the core repo's `src/lib/sceneTemplates.js`). jsDelivr caches tags
aggressively, so content releases are: commit → re-point the tag → the app
picks it up without a redeploy.

```
git tag -f v2 && git push -f origin v2     # re-point the serving tag
```

The tag name tracks the INDEX FORMAT, and a format bump takes a NEW tag — never
reuse an old one, because builds already in the wild keep reading the tag they were
built against. `SCENES_BASE` in core and the serving tag here must move together:
core pointing at a tag that does not exist yet is a 404, and the app falls back to
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
