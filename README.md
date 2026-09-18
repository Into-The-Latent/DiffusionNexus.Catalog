# DiffusionNexus Catalog

Workloads and ComfyUI workflows for the [DiffusionNexus Installer](https://github.com/Into-The-Latent/DiffusionNexus.Installer).
Content is published through GitHub Releases; the installer fetches `manifest.json` and `catalog.zip` from the latest release.

All rights reserved. No licence is granted for the content of this repository.

## Layout

| Path | What |
|---|---|
| `catalog.json` | `{ "schemaVersion": 1 }` — bump only when the file format changes |
| `repositories.json` | custom-node git repositories shared by workloads |
| `wheels.json` | llama.cpp wheels |
| `workloads/<slug>/workload.json` (+ `thumbnail.png`) | one workload |
| `workflows/<slug>/workflow.json` + `meta.json` | one ComfyUI workflow; `workflow.json` is the untouched ComfyUI export |
| `schema/` | JSON Schemas for editor autocomplete |

## Adding a workflow

1. Export the workflow from ComfyUI (Workflow → Export) and save it as `workflows/<slug>/workflow.json`. Slug = lowercase, digits and dashes.
2. Create `workflows/<slug>/meta.json`:
   ```json
   { "id": "<new GUID>", "name": "My workflow", "version": 1, "subVersion": 0, "workloads": ["<workload GUID>"] }
   ```
3. Commit and push. CI validates and updates the `preview` release.
4. When ready for everyone: `gh release create v<N>` (N = previous + 1). CI attaches the assets.

## Updating content

Bump `version`/`subVersion` of the item you changed — CI warns when content changed but the version did not.

## Versions and channels

- `releases/latest` = stable. Draft releases are invisible; deleting a release rolls back.
- Pre-release `preview` tracks `main` and is rebuilt on every push. It is numbered by content: same number as the latest stable release while nothing has changed, stable + 1 as soon as `main` carries a catalog change.

## Validate locally

```
dotnet tool install -g DiffusionNexus.Installer.SDK.Catalog.Tool --add-source https://nuget.pkg.github.com/Little-God1983/index.json
dn-catalog validate .            # structural
dn-catalog validate . --online   # also HEAD-checks every URL
```
