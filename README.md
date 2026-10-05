# CRT-Mame-Advance-Four (RetroArch Slang)

A feature-rich CRT simulation shader for RetroArch, ported from a [ReShade shader](https://github.com/JBW-byte/CRT-Mame-Advance-Four)
 originally written for the MAME emulator. It is a 3-pass Slang preset with a shared parameter include, and every effect is toggleable from the RetroArch shader parameter menu.

> **Version:** v5.3 Production Edition

---

## Features

| Area | What it does |
|---|---|
| **Quality tiers** | Fast 2-line, Balanced 3-line or Ultra 5-line scanline reconstruction |
| **Beam model** | Brightness-dependent beam width, adjustable Gaussian sharpness, edge defocus, scanline gain and density |
| **Phosphor masks** | Aperture grille, slot mask, dot-triad shadow mask, with RGB / BGR / WOLED / QD-OLED host subpixel layouts, bloom fade, scaling and auto brightness compensation |
| **Analog signal** | Horizontal Gaussian blur or asymmetric RC bleed, plus optional NTSC composite demodulation (chroma bleed, dot crawl, crosstalk) |
| **DPX filmic tone** | Filmic tone curve with adjustable gain, strength, contrast, colourfulness and saturation |
| **Glass and tube** | Curvature, anti-aliased rounded corners, halation, wide diffuse glow |
| **Convergence** | Static X/Y misconvergence, luma-dependent bleed, radial yoke corner fringing |
| **Colour** | Input/output gamma, master saturation, black level, optional P22 arcade phosphor gamut |
| **Timing effects** | Interlacing (480i fields) or 240p line jitter, refresh sweep, AC hum bar |
| **Tube aging** | Phosphor aging and burn-in tint |
| **HDR output** | SDR, HDR10 (PQ / Rec.2020) and scRGB, with adjustable paper white and peak nits |

Effects that are switched off are skipped at runtime where possible, so unused features cost very little.

---

## How it works

The shader is split into three passes (the same pattern used by `crt-geom-deluxe`) so that the expensive colour maths runs once per source pixel instead of once per output pixel per tap.

| Pass | File | Resolution | Job |
|---|---|---|---|
| 0 | `CRT-Mame-Advance-Four-linear.slang` | Source | DPX tone, gamma linearisation, P22 gamut, saturation |
| 1 | `CRT-Mame-Advance-Four-signal.slang` | Source | Horizontal blur / RC bleed / NTSC composite |
| 2 | `CRT-Mame-Advance-Four.slang` | Viewport | Curvature, scanlines, convergence, mask, halation / glow, HDR output |

All passes include `CRT-Mame-Advance-Four-params.inc`, which holds the shared parameter list and uniform block.

---

## Requirements

- RetroArch with a video driver that supports Slang shaders:
  - **Vulkan**, **OpenGL (glcore)**, **Direct3D 11 / 12** or **Metal**
  - The legacy OpenGL driver (`gl`) does not support Slang
- A GPU that supports half-float render targets (any recent GPU)

---

## Installation

### 1. Copy the files

Put **all five files together in one folder**. The preset loads the passes by relative path.

```
CRT-Mame-Advance-Four.slangp
CRT-Mame-Advance-Four.slang
CRT-Mame-Advance-Four-linear.slang
CRT-Mame-Advance-Four-signal.slang
CRT-Mame-Advance-Four-params.inc
```

Place that folder inside your RetroArch shaders directory, for example:

| Platform | Typical location |
|---|---|
| Windows | `RetroArch\shaders\shaders_slang\crt\` |
| Linux | `~/.config/retroarch/shaders/shaders_slang/crt/` |
| macOS | `~/Library/Application Support/RetroArch/shaders/shaders_slang/crt/` |

If you are unsure where your shader folder is, open RetroArch and check **Settings → Directory → Video Shaders**.

### 2. Load the preset

1. Start a game or content.
2. Open the Quick Menu.
3. Go to **Shaders → Load → Load Preset** (the option may read *Load Shader Preset*).
4. Browse to `CRT-Mame-Advance-Four.slangp` and select it.
5. Make sure **Shaders → Video Shaders** is set to **On**.

### 3. Save it (optional)

Under **Shaders → Save**, you can save the settings as:

- **Core preset**: applies to every game on this core
- **Content directory preset**: applies to games in the same folder
- **Game preset**: applies to this game only

---

## Tuning

Open **Quick Menu → Shaders → Shader Parameters** to adjust settings live.

Suggested starting points:

- **Performance:** set *Quality Tier* to `0` (Fast) or `1` (Balanced). Tier `2` is the most expensive.
- **Brighter image:** raise *Scanline Brightness Gain* or *Manual Mask Brightness Boost*, or increase *Auto Mask Compensation*.
- **Sharper picture:** lower *Beam Bandwidth / Blur Width* or turn *Analog Signal Filtering* off.
- **Flat screen:** turn off *Enable Tube Glass Curvature* and set *Corner Rounding Radius* to `0`.
- **Heavy glow:** if halation or wide glow looks too strong, reduce *Halation Strength* or *Wide Glow Strength*. Both are off by default.
- **HDR:** set *Display HDR Profile* to match your display and enable HDR in RetroArch first.

Convergence (static shift or radial yoke) and tier 2 are the most costly options. Leave them off if you need to save GPU time.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Shader fails to load or shows black | Check that all five files are in the same folder and that your video driver supports Slang (see Requirements). |
| Compile error in the log mentioning `GL_` | Use the files from this repository. GLSL reserves the `GL_` prefix, so the wide glow parameters are named `WG_*`. |
| Halation or wide glow does nothing | They require mipmaps on the final pass. Use the included `.slangp`, which sets `mipmap_input2 = "true"`. |
| Old saved settings missing | Wide glow parameters were renamed from `GL_*` to `WG_*`. Re-save your preset. |
| Banding in dark areas | The first two passes use float framebuffers. Do not remove `float_framebuffer0` / `float_framebuffer1` from the preset. |

---

## Credits and licence

Written by the L.E.D. owner. MIT Licence.
