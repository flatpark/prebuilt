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

Every stack is built with `no-debuginfo: true` **and** `strip: true`. The first
alone only suppresses the `.Debug` extension — it leaves the DWARF inside each
library, which then travels in the archive and, for a stack consumed as an
archive module, into the consuming app's OSTree commit. Symbols are recovered by
rebuilding from the manifest at the tag, not by shipping them to every user.

## Stacks

| Stack | Contents | Built against | Used by |
|---|---|---|---|
| `ayatana-stack` | libdbusmenu 16.04.0, ayatana-ido 0.10.4, libayatana-indicator 0.9.4, libayatana-appindicator 0.5.94 | `org.gnome.Sdk//50` | Tauri apps needing a tray icon |
| `mpv-stack` | libass 0.17.3, libplacebo v7.360.1, mpv v0.40.0 (`libmpv.so.2` **and** the `mpv` command-line player) | `org.gnome.Sdk//50` | Apps that play video with mpv — `dk.nikse.subtitleedit` (dlopens libmpv for the video preview), `site.harbor.Harbor.Beta` (embeds libmpv, and spawns `mpv` for thumbnails, clip encoding and multiview) |
| `libxdo` | libxdo from xdotool 3.20211022.1, shared library only (soname `libxdo.so.3`) — no headers, no `xdotool` CLI | `org.gnome.Sdk//50` | Apps that link libxdo for X11 input automation (the enigo crate), e.g. `io.github.thewh1teagle.vibe` |
| `opencv-imgproc` | OpenCV 4.13.0 (core + imgproc), dev-complete (libs + headers + CMake/pkg-config); no `share/opencv4` cascade data | `org.freedesktop.Sdk//25.08` | `wemeet-screenshare-hook` builds against it; `com.tencent.wemeet` ships it as **extra-data** because the hook `dlopen`s OpenCV by unversioned soname |
| `openssl-1.1-compat` | OpenSSL 1.1.1w shared libraries only (`libssl.so.1.1`, `libcrypto.so.1.1`) — no headers, runtime shim | `org.freedesktop.Sdk//25.08` | Legacy payloads whose bundled runtime predates OpenSSL 3 support (e.g. self-contained .NET 5) — **1.1.1 is EOL, see the manifest header** |
| `wemeet-screenshare-hook` | libportal 0.9.1 + xuwd1/wemeet-wayland-screenshare `libhook.so` (built against `opencv-imgproc`; OpenCV not shipped but **dlopen'd at runtime**, so the app must also ship `opencv-imgproc`) | `org.freedesktop.Sdk//25.08` | `com.tencent.wemeet` (XWayland screen-share hook) |
| `krb5-gss` | MIT krb5 1.22.1, the load-time closure of `libgssapi_krb5.so.2` and nothing else (`libkrb5`, `libk5crypto`, `libcom_err`, `libkrb5support`) — no KDC/kadmin libraries, no plugin tree, no headers | `org.freedesktop.Sdk//25.08` | Payloads bundling a Qt built with the GSSAPI feature, whose `libQt6Network` then hard-links `libgssapi_krb5.so.2` — `com.interactivebrokers.ibkrdesktop`. **Consumed as extra-data**, see below |
| `x264` | x264 (commit `0480cb05`), `libx264.so.165`, dev-complete; no CLI | `org.gnome.Sdk//50` | Apps encoding H.264; `ffmpeg-full` builds against it |
| `x265` | x265 4.2, `libx265.so.216`, 8-bit only, dev-complete; no CLI | `org.gnome.Sdk//50` | Apps encoding H.265; `ffmpeg-full` builds against it |
| `lame` | LAME 3.100, `libmp3lame.so.0`, dev-complete; no front ends | `org.gnome.Sdk//50` | Apps encoding MP3; `ffmpeg-full` builds against it |
| `rubberband` | Rubber Band v4.0.0, `librubberband.so.3` + its CLIs; built-in FFT and resampler pinned, so it links nothing beyond libstdc++/libm/libgcc/libc | `org.gnome.Sdk//50` | Time-stretch / pitch-shift; `ffmpeg-full` builds against it for the `rubberband` filter |
| `libass` | libass 0.17.4, `libass.so.9`, dev-complete, fontconfig enabled | `org.gnome.Sdk//50` | ASS/SSA rendering — `ffmpeg-full` (the `ass` and `subtitles` filters) and, in future, mpv. Released separately so an app shipping both does not end up with two `libass.so.9` deciding by module order |
| `ffmpeg-full` | FFmpeg n9.0, `ffmpeg` + `ffprobe` + `libav*`; libx264, libx265, libvpx-vp9, prores_ks, VAAPI and NVENC, aac/ac3/libmp3lame/libopus/libvorbis, `ass`/`subtitles`/`drawtext`. **No QSV or AMF** — see the manifest header | `org.gnome.Sdk//50` | Apps that drive ffmpeg as a tool because the runtime's own has no libass — `dk.nikse.subtitleedit`. Ships none of the codec libraries it links; the app pins those releases too |
| `leptonica` | Leptonica 1.85.0, `libleptonica.so.6`, dev-complete; no demo programs | `org.gnome.Sdk//50` | Image processing; `tesseract` builds against it and the consuming app ships both |
| `tesseract` | Tesseract 5.5.1, `libtesseract.so.5` + the `tesseract` CLI + `share/tessdata` presets. No leptonica, no language data, no training tools | `org.gnome.Sdk//50` | OCR — `dk.nikse.subtitleedit`. Stage `.traineddata` into its `share/tessdata` and point `TESSDATA_PREFIX` there |
| `uchardet` | uchardet 0.0.8, `libuchardet.so.0`, dev-complete; no CLI | `org.gnome.Sdk//50` | Guessing the encoding of text files an app did not write, e.g. subtitle files |
| `sevenzip` | 7-Zip 25.01 `7zr`, one dependency-free binary; .7z only | `org.gnome.Sdk//50` | Apps unpacking .7z downloads at run time |
| `sql-clients` | MariaDB Connector/C 3.4.9 (`libmariadb.so.3` + its authentication plugins), PostgreSQL 18.6 `libpq.so.5`, FreeTDS 1.5.9 `libsybdb.so.5`, sshpass 1.10; runtime-only — no headers, no CLIs beyond sshpass | `org.freedesktop.Sdk//25.08` | Database clients that reach their driver at run time rather than linking it — `com.heidisql.HeidiSQL`, which enumerates them by parsing `ldconfig -p`. The runtimes ship `libsqlite3` and nothing else. **Consumed as extra-data**, see below |

## Archive module or extra-data

A stack can be consumed either way, and the choice decides where the bytes live:

- **`type: archive` build module** (`ayatana-stack`, `mpv-stack`, `libxdo`,
  `wemeet-screenshare-hook`) — the tree is copied into `/app` at build time, so
  it becomes part of the app's OSTree commit and is stored in FlatPark's own
  repository. Content-addressed storage means a stack shared by many apps is
  held once; `ayatana-stack` is one object set for thirteen apps.
- **`type: extra-data`** (`krb5-gss`, `openssl-1.1-compat`, `opencv-imgproc`, and the
  ffmpeg and OCR stacks: `x264`, `x265`, `lame`, `rubberband`, `libass`,
  `ffmpeg-full`, `leptonica`, `tesseract`, `uchardet`, `sevenzip`, `sql-clients`) —
  the archive is downloaded from this repository's release at install time and
  unpacked by the app's `apply_extra`
  into `/app/extra/<stack>/`. FlatPark's repository holds nothing, and the
  bandwidth is GitHub's. The consuming wrapper must put `/app/extra/<stack>/lib`
  on `LD_LIBRARY_PATH`, since that path is not on the loader's default search
  path — which also means the stack shadows nothing else in the sandbox.

Prefer extra-data. FlatPark's repository is meant to hold app metadata, not
built bytes, and a stack with one or two consumers gives content-addressed
storage nothing to deduplicate anyway. The archive module remains the right
answer where many apps share one stack (`ayatana-stack`, thirteen consumers):
there the repository holds one object set, while extra-data would put a private
copy on every user's disk and re-download it per app.

## Cutting a release

Run the `release` workflow (workflow_dispatch) with a tag (e.g. `ayatana-v1`,
`opencv-imgproc-v1`) and the manifest to build. Rebuild whenever the target
runtime major bumps or a stack component is updated; consuming manifests pin the
archive by sha256 and migrate explicitly.

Some stacks build against another stack's release (e.g. `wemeet-screenshare-hook`
consumes `opencv-imgproc` for OpenCV headers): cut the dependency's release
first, pin its archive URL + sha256 in the dependent manifest, then cut that one.
