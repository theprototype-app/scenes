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
```

`index.json`:

```json
{
	"version": 1,
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
	"examples": []
}
```

- `scene`/`thumb` are repo-relative; absolute `https://` URLs pass through
  untouched (use `raw.githubusercontent.com` for files over ~20 MB — the
  jsDelivr per-file cap).
- A `.tpscene` is the app's session zip (`session.json` + optional `assets/`):
  save one from the app via logo menu ▸ Save ▸ Scene.

## Serving

The app fetches `index.json` via the **tag-pinned** jsDelivr mirror
(`https://cdn.jsdelivr.net/gh/theprototype-app/scenes@v1/...` — see `SCENES_BASE`
in the core repo's `src/lib/sceneTemplates.js`). jsDelivr caches tags
aggressively, so content releases are: commit → re-point the tag → the app
picks it up without a redeploy.

```
git tag -f v1 && git push -f origin v1     # re-point the serving tag
```

## Adding a scene

1. Build the scene in the app (primitives keep it small; assets bundle into the
   zip when saved with assets enabled).
2. Save as `.tpscene`, drop it under `templates/<slug>/` or `examples/<slug>/`
   with a `thumb.webp` (480x270).
3. Add its row to `index.json` (`bytes` = the file size).
4. Commit, re-point the tag.

The core repo's `scripts/author-templates.cjs` regenerates the seed templates
programmatically and can write this repo's tree via `--out`.

## License

Content in this repo is [CC0 1.0](LICENSE) unless a scene's `license` field
says otherwise.
