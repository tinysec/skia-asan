# skia-asan

ASAN-instrumented build of **Skia** — Chrome's 2D graphics and image-decoding library. Rebuilt automatically whenever the revision Chrome uses changes.

![build](https://github.com/tinysec/skia-asan/actions/workflows/build.yml/badge.svg)
![track](https://github.com/tinysec/skia-asan/actions/workflows/track.yml/badge.svg)
![release](https://img.shields.io/github/v/release/tinysec/skia-asan?label=release)

## Staying current

`chrome.lock` pins the exact Skia revision a specific Chrome stable version
ships (Skia is pinned directly in Chrome DEPS). The
[`track-chrome`](.github/workflows/track.yml) workflow runs every 6 hours: it
resolves the latest Chrome stable, reads the Skia SHA from Chrome DEPS, and
when it has changed it bumps `chrome.lock`, tags `chrome-<version>`, and
triggers [`build`](.github/workflows/build.yml). Each `chrome-<version>` tag
becomes a GitHub release.

## Release artifacts

Each release is published at its `chrome-<version>` tag as **one zip per
platform** (so the full `include/` header tree is preserved without name
collisions in GitHub's flat asset namespace). Each zip contains:

- `lib/libskia.a` (Linux) / `lib/skia.lib` (Windows): the ASAN **static**
  Skia library — link it into your own harness. This is the primary artifact
  for fuzzing. (Skia is shipped static only; no dynamic ASAN lib.)
- `lib/include/`: the full Skia public header tree.
- `fuzz/fuzz` (Linux only): the Skia multiplexed fuzz driver (own `main()`,
  replays seed files). Windows omits it: Skia's GN leaves `is_clang=false` for
  the MSVC-ABI toolchain, so the `assert(is_clang)` fuzz-driver targets are
  unavailable there. SanitizerCoverage stays on via `fuzzer-no-link` so a
  downstream libFuzzer / WinAFL harness still gets coverage feedback.

## ASAN runtime model

Both platforms link the **static** ASAN runtime (depot_tools' bundled clang
ships the full static sanitizer runtime). The libraries and the `fuzz` driver
are self-contained — no `clang_rt.asan_dynamic` DLL is needed at runtime.

## Fuzzing

- **Linux:** `fuzz/fuzz` is a ready-to-run ASAN fuzz executable with its own
  `main()` (replays seeds, not a libFuzzer engine).
- **Windows (AFL++ / WinAFL):** link `lib/skia.lib` into a harness exposing a
  target function, then point WinAFL at that function. The static ASAN lib is
  self-contained.

> These builds provide the ASAN-instrumented library to target. They do not
> bundle the AFL++ / WinAFL runner itself (a separate DynamoRIO-based
> toolchain on Windows).
