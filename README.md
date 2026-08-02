# gl-crt-pixel-shaders

Single-pass GLSL CRT shaders for pixel-art programs that don't support multipass shader chains —
built for **ScummVM** and **DOSBox** (Staging).

These are modified variants of [Hyllian's CRT shader](https://github.com/libretro/glsl-shaders/tree/master/crt),
tuned by hand to sit between two extremes: a hard nearest-neighbor upscale (too sharp, not how the
pixel art was intended to be seen on period CRT hardware) and a fully authentic CRT simulation (heavy
curvature, bloom, and a strongly visible phosphor mask). The goal is a plausible, general-purpose CRT
"softening" — not a faithful emulation of any one specific monitor.

## Table of contents

- [Included shaders](#included-shaders)
- [Requirements](#requirements)
- [Usage](#usage)
  - [ScummVM](#scummvm)
  - [DOSBox Staging](#dosbox-staging)
- [Tuning](#tuning)
- [Repository structure](#repository-structure)
- [Credits](#credits)
- [License](#license)

## Included shaders

Every shader in this repository is single-pass and has a matching `.glslp` preset at the repository
root, ready to use directly:

| Preset | Shader source | Notes |
| --- | --- | --- |
| `crt-hyllian.glslp` | `crt-hyllian.glsl` | **Default.** Reads the source resolution dynamically, so it works correctly at essentially any resolution. Start here. |
| `crt-hyllian_hires.glslp` | `crt-hyllian_hires.glsl` | Same dynamic-resolution behavior, but masks the CRT effect differently by default — no phosphor/subpixel mask, scanline darkening only — so it reads as scanlines rather than an incorrect-looking raster pattern at higher resolutions. |
| `crt-hyllian_180p.glslp` | `crt-hyllian_180p.glsl` | Fixed-resolution variant (320×180). |
| `crt-hyllian_200p.glslp` | `crt-hyllian_200p.glsl` | Fixed-resolution variant (320×200). |
| `crt-hyllian_240p.glslp` | `crt-hyllian_240p.glsl` | Fixed-resolution variant (320×240). |
| `crt-hyllian_400p.glslp` | `crt-hyllian_400p.glsl` | Fixed-resolution variant (320×400), for DOS titles at that resolution. |

### Which preset should I use?

Start with `crt-hyllian.glslp` — it adapts to the source resolution automatically. Reach for a
fixed-resolution variant only when the default shader can't find the correct pixel size on its own,
which shows up as scanlines/mask landing in the wrong place. Two situations where that happens:

- **DOSBox Staging (and other DOSBox variants) with doublescan enabled.** Doublescan re-creates how
  real VGA hardware actually rendered — each line genuinely gets doubled — which is accurate to the
  original hardware, but it means the shader is fed twice as many lines as the game's real pixel
  grid, so the scanline pattern ends up misaligned. Pick the fixed-resolution preset matching the
  game's actual resolution (e.g. `crt-hyllian_400p.glslp` for 320×400 DOS titles).
- **ScummVM games that render internally at a higher resolution than their intended low-res look** —
  notably AGS (Adventure Game Studio) titles, which are often built to look like an early-90s
  low-resolution game but render at a higher internal resolution. Use the fixed-resolution preset
  matching the intended low-res look, not the actual render resolution.

## Requirements

- [ScummVM](https://www.scummvm.org/) with OpenGL rendering, **or**
- [DOSBox Staging](https://dosbox-staging.org/) with the OpenGL output enabled.

## Usage

### ScummVM

1. In ScummVM, go to **Options → Graphics** and set the renderer to OpenGL.
2. Next to the **Shader** option, choose **Pick file...** and select one of this repository's root
   `.glslp` files (e.g. `crt-hyllian.glslp`).
3. Confirm the preview and keep the shader when prompted.

The `.glslp` file references its shader by a relative path (e.g. `shaders/hyllian/crt-hyllian.glsl`),
so keep the folder structure intact — if you move the `.glslp` file without its `shaders/` folder,
update the path inside it or the shader will fail to load.

### DOSBox Staging

DOSBox Staging loads a single `.glsl` file by name (no `.glslp` support), via the `glshader` setting
in the `[render]` section of `dosbox-staging.conf`:

```ini
[render]
glshader = crt-hyllian
```

To make a shader available under that name:

1. Copy the desired file from `shaders/hyllian/` (e.g. `crt-hyllian.glsl`) into DOSBox Staging's
   `glshaders` subdirectory inside its configuration directory, or into the DOSBox root directory.
2. Set `glshader` to the filename **without** the `.glsl` extension, as shown above.

If the shader fails to compile, DOSBox Staging logs an error/warning to its console output.

## Tuning

Each shader exposes its adjustable values as `#pragma parameter` entries near the top of the file —
these surface as sliders in ScummVM's shader preview and in RetroArch-style frontends. Notable ones
across the `crt-hyllian*` variants include `PHOSPHOR_LAYOUT` (mask pattern), `MASK_INTENSITY`,
`SCANLINES_STRENGTH`, `HFILTER_SHARPNESS`, and `InputGamma`/`OutputGamma`. Open a shader file and look
for the `#pragma parameter` block to see its full set and value ranges.

## Repository structure

```
crt-hyllian*.glslp          Root-level presets (single-pass, ready to use)
shaders/hyllian/
  crt-hyllian*.glsl          Shader variants (see table above)
```

## Credits

Based on [Hyllian's CRT shader](https://github.com/libretro/glsl-shaders/tree/master/crt)
(Copyright © 2011–2020 Hyllian). The phosphor mask function (`mask_weights`) used in these shaders is
adapted from cgwg's `crt-geom-deluxe` LUTs via hunterk's public-domain mask collection. See the
license header at the top of each `.glsl` file for the exact attribution and terms that apply to it.

## License

MIT — see [`LICENSE`](LICENSE). Each shader file also carries the original author's MIT-style
permission notice in its header comment; keep that notice intact if you redistribute a modified copy.
