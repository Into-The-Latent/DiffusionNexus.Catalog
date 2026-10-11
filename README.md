# DiffusionNexus Catalog

Workloads and ComfyUI workflows for the [DiffusionNexus Installer](https://github.com/Into-The-Latent/DiffusionNexus.Installer)
and the [DiffusionNexus](https://github.com/Little-God1983/DiffusionNexus) app.
Content is published through GitHub Releases; the apps fetch `manifest.json` and `catalog.zip` from a release.

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

- `releases/latest` = stable. Draft releases are invisible. To roll back, tag a new release that
  carries the old content (`git revert`, then `gh release create v<N+1>`). Deleting the newest
  release only makes `releases/latest` serve an older publication, and installed apps ignore a
  manifest packed before the one they applied on the same channel (SDK 2.1.0): they would keep
  the deleted content.
- Pre-release `preview` tracks `main` and is rebuilt on every push. It is numbered by content: same number as the latest stable release while nothing has changed, stable + 1 as soon as `main` carries a catalog change.

## How the catalog is distributed

One catalog serves both apps. Every release is a full snapshot: an app that applies a release
gets all of it, whichever app the workloads are for.

- **Channels.** Stable = `releases/latest`. Preview = the `preview` tag, rebuilt from `main` on every push.
- **Installed copies** (SDK 2.3.0+) live in `%LocalAppData%\ITL-Shared\format-N\catalog-stable\` and
  `...\catalog-preview\`, where `N` is `catalog.json`'s `schemaVersion`. Both apps read these folders and
  either app may pull a release into them.
- **Each app picks its own channel** (Settings → Catalog in both). Two apps on the same channel share the
  folder, so they always see the same catalog version. One app on Stable and the other on Preview keep two
  folders side by side.
- **Each app announces only changes to its own workloads**, decided by `workloadTarget` (`Installer` or
  `DiffusionNexusCore`) in the manifest. A workflow counts for the apps of the workloads it lists. So a
  release that changes only DiffusionNexus workloads is applied by the installer too, but installer users
  see no "update available" notice. Manifests packed before `dn-catalog` 2.3.0 have no target, and every
  change in them counts for both apps.
- **Applying automatically** is a per-app setting, off by default. With it off the app shows the notice
  and the list of changes and the user applies.
- **Format changes.** A release whose `schemaVersion` is `N` lands in `format-N`. An app that can't read
  format `N` never downloads it; it keeps its last readable catalog and says that a newer version of the
  app is needed. **Move Stable to a new format only once both apps read it**, or the slower app stays
  frozen on its last Stable catalog until it updates. Preview can go first.
- **Rollback** of Stable is a new tag carrying the old content; see above.

## Validate locally

```
dotnet tool install -g DiffusionNexus.Installer.SDK.Catalog.Tool --add-source https://nuget.pkg.github.com/Little-God1983/index.json
dn-catalog validate .            # structural
dn-catalog validate . --online   # also HEAD-checks every URL
```
