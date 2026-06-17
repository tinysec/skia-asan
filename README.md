# skia-asan

ASAN-instrumented fuzzer build of **Skia** — Chrome's 2D graphics and image-decoding library. Rebuilt automatically when the revision Chrome uses changes.

![build](https://github.com/tinysec/skia-asan/actions/workflows/build.yml/badge.svg)
![track](https://github.com/tinysec/skia-asan/actions/workflows/track.yml/badge.svg)
![release](https://img.shields.io/github/v/release/tinysec/skia-asan?label=release)

**Engine:** libFuzzer (Linux + Windows) · **Sanitizer:** ASAN · **Symbols:** Windows `.pdb` included

`track.yml` polls Chrome stable **daily** and resolves the Skia SHA Chrome actually ships (Chrome DEPS → `skia_revision`). Unchanged → nothing happens. Changed → bump + tag `chrome-<version>` → `build.yml` runs → new Release.

> No PAT or secret needed — the tracker runs on the default `GITHUB_TOKEN` and dispatches the build via `workflow_dispatch`.

```bash
git tag v1 && git push origin v1   # manual trigger
```

No harness is committed here — Skia's built-in `fuzz/` targets (image codecs, SKP, text blobs, SkSL) are built from the source tree.
