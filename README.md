# IconWorkbench — Rust CLI proof of concept

A locally executed image-to-vector and icon export experiment. No API keys,
network service, macOS frameworks, or Python are required at runtime.

## Build and run

Install Rust, then:

```sh
cargo build --release --locked
./target/release/iconworkbench trace samples/flat.png my-icon.svg poster
./target/release/iconworkbench trace samples/gradient.png my-photo.svg photo
./target/release/iconworkbench icon samples/flat.png my-icons
```

On a Mac these commands build a native Mac executable. This bundle was tested
on Linux only; it does not contain a tested Mac binary.

- `trace INPUT OUTPUT.svg [poster|photo|bw]`: VTracer 0.6.5, explicitly pinned
  for reproducibility, not claimed to be the latest or highest quality engine.
- `icon INPUT NEW_DIRECTORY`: preserve aspect ratio, add 12.5% margin on each
  side, generate 16/32/64/128/256/512/1024 PNGs and a PNG-backed ICNS container.
- Existing output files/directories are refused.
- SVG is not accepted as input by the icon exporter in this POC.

## Reproduce verification

```sh
python3 -m pip install Pillow CairoSVG
python3 tests/verify.py
```

CairoSVG may require system Cairo on a Mac. Python is used only to synthesize
fixtures and independently decode/check outputs; the CLI itself is Rust.
The verification script regenerates the bundled `results/icons` directory.

Three 512x512 synthetic fixtures exercise flat transparent colors, JPEG
compression, and smooth gradients. Six SVG conversions are rasterized using
CairoSVG; tests check real paths (no embedded bitmap), transparent corners,
PNG dimensions, ICNS chunk lengths and independent Pillow ICNS decoding,
and failure behavior for missing files/existing outputs.

## Scope and limitations

This validates the plumbing, not professional artwork quality. No real AI
image or Vectorizer.AI reference output was supplied, so no quality parity
claim is made. VTracer may flatten fine details, posterize gradients, and
produce many paths. Partial-alpha behavior, arbitrary malformed/large inputs,
color profiles, premultiplied-alpha resize quality, and production resource
limits need further work. Inputs should be trusted and modest in size.

ICNS was decoded on Linux, not tested in Finder or an installed Mac app.
This is not a complete Xcode asset catalog, Retina slot mapping, signed app,
or Icon Composer layered document. UI, background removal, AI generation,
SVG editing, automatic visual optimization, and distribution are out of scope.

Inspect results/comparison.png, results/metrics.json and REPORT.md.

## Automated CLI regression tests

```sh
cargo test --release --locked
```

14 black-box integration tests cover success/error paths, non-destructive output,
PNG/JPEG tracing presets, aspect ratio, padding, alpha, Unicode paths, and ICNS
payloads. See TEST_CASES.md for the full matrix and explicit quality gaps.
