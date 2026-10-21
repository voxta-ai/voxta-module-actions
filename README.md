# Voxta module workflows

Reusable GitHub Actions workflows that build, release and publish [Voxta](https://voxta.ai) modules. The
[module template](https://github.com/voxta-ai/voxta-module-template) calls them; any module repository can.

| Workflow | Does |
|---|---|
| `build.yml` | Builds and packs every module (locked restore, 32 MB zip limit). |
| `release.yml` | On a tag: builds the module it names, attests it (public repositories), creates the GitHub release, and with `publish: true` lists it in Voxta's module source. |
| `edit-release.yml` | Caps a release's Voxta range or yanks it without rebuilding; with `publish: true` Voxta's source follows. |
| `bump-voxta.yml` | Raises `Voxta.Sdk.Modules`, the `module.json` range, the lock files and the module version, builds, opens a pull request. |

```yaml
# .github/workflows/release.yml
on:
  push:
    tags: ['v*', '*/v*']
jobs:
  release:
    uses: voxta-ai/voxta-module-actions/.github/workflows/release.yml@v1
    permissions:
      contents: write
      id-token: write
      attestations: write
    with:
      publish: ${{ vars.VOXTA_PUBLISH == 'true' }}
```

Reference them by a `v*` tag of this repository: Voxta Cloud accepts publishes only from those (or `main`).

`release.yml` and `edit-release.yml` read the module with the `Voxta.ModulePublish` version that matches the
`Voxta.Sdk.Modules` version in its `packages.lock.json`. A module therefore keeps releasing against its own Voxta
version until `bump-voxta.yml` moves it to a newer one. Pass `tool-version` to override this.

## Publishing to Voxta's module source

Voxta lists a module in its own source when Voxta trusts the repository it comes from (portal: **Admin → Module
repos**). The publishing job proves which repository, commit, tag and workflow it is with its GitHub OIDC token; no
key to store. Jobs that build or load the module hold no write or publish token: the job that publishes runs none of
the module's code.

- Public repositories only, except Voxta's own.
- Tags protected by a ruleset (Settings → Rules → Rulesets, target tags `v*`): Voxta refuses new files from an
  unprotected tag.
- Zips up to 32 MB. Download CUDA, models and other large dependencies at runtime.
