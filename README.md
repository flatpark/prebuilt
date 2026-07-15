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
| `opencv-imgproc` | OpenCV 4.13.0 (core + imgproc), dev-complete (libs + headers + CMake/pkg-config) | `org.freedesktop.Sdk//25.08` | Apps that build against OpenCV |
| `wemeet-screenshare-hook` | libportal 0.9.1 + xuwd1/wemeet-wayland-screenshare `libhook.so` (built against `opencv-imgproc`; OpenCV is build-time only, not shipped) | `org.freedesktop.Sdk//25.08` | `com.tencent.wemeet` (XWayland screen-share hook) |

## Cutting a release

Run the `release` workflow (workflow_dispatch) with a tag (e.g. `ayatana-v1`,
`opencv-imgproc-v1`) and the manifest to build. Rebuild whenever the target
runtime major bumps or a stack component is updated; consuming manifests pin the
archive by sha256 and migrate explicitly.

Some stacks build against another stack's release (e.g. `wemeet-screenshare-hook`
consumes `opencv-imgproc` for OpenCV headers): cut the dependency's release
first, pin its archive URL + sha256 in the dependent manifest, then cut that one.
