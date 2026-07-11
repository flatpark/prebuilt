# flatpark/prebuilt

Prebuilt supporting libraries for [FlatPark](https://flatpark.org) app
manifests — the pieces the shared runtimes lack and several apps would
otherwise compile from source on every build.

Only redistributable open-source libraries are prebuilt here. App payloads are
never mirrored: FlatPark apps are extra-data packages that download the
vendor's own official release at install time.

## Provenance

Each release is built by the `release` workflow in this repository from the
manifest at the release tag. The manifest pins every source (URL + sha256, or
git tag + commit) and carries the patches verbatim, so a release can be
reproduced and audited from the tag alone. Release notes name the workflow run
and commit that produced the artifact.

## Stacks

| Stack | Contents | Built against | Used by |
|---|---|---|---|
| `ayatana-stack` | libdbusmenu 16.04.0, ayatana-ido 0.10.4, libayatana-indicator 0.9.4, libayatana-appindicator 0.5.94 | `org.gnome.Sdk//50` | Tauri apps needing a tray icon |

## Cutting a release

Run the `release` workflow (workflow_dispatch) with a tag like `ayatana-v1`.
Rebuild whenever the target runtime major bumps or a stack component is
updated; consuming manifests pin the archive by sha256 and migrate explicitly.
