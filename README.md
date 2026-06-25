# 🐱 Catppuccin for FreshTomato

> Soothing pastel theme for the [FreshTomato](https://github.com/FreshTomato-Project/freshtomato-mips) router firmware web UI.

[![Catppuccin](https://img.shields.io/badge/Catppuccin-pastel_theme-CBA6F7?style=flat-square&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0naHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmcnIHZpZXdCb3g9JzAgMCAyNCAyNCc+PHBhdGggZmlsbD0nI2NiYTZmNycgZD0nTTEyIDJDNi40NzcgMiAyIDYuNDc3IDIgMTJzNC40NzcgMTAgMTAgMTAgMTAtNC40NzcgMTAtMTBTMTcuNTIzIDIgMTIgMnonLz48L3N2Zz4=)](https://catppuccin.com)

## Flavors

| File | Flavor | Background | Feel |
|------|--------|------------|------|
| `at-ctp-mocha.css` | 🌙 Mocha | `#1e1e2e` | Deepest dark — ideal for night use |
| `at-ctp-macchiato.css` | 🌺 Macchiato | `#24273a` | Rich violet-dark — evening |
| `at-ctp-frappe.css` | 🪴 Frappé | `#303446` | Cool medium-dark — relaxed |
| `at-ctp-latte.css` | 🌻 Latte | `#eff1f5` | Warm cream — daytime / light |

## Installation

### Via SSH (recommended)

```bash
# 1. Copy the desired theme file to the router's web root
scp at-ctp-mocha.css root@192.168.1.1:/www/

# 2. Activate the theme (no reboot required)
nvram set web_css=at-ctp-mocha.css
nvram commit

# 3. Hard-refresh your browser (Ctrl+Shift+R / Cmd+Shift+R)
```

Replace `at-ctp-mocha.css` with any other flavor filename.

### Via the Web UI

1. **Administration → Admin Access**
2. Scroll to the **"Custom CSS"** (or "Web CSS") field
3. Enter the filename you uploaded (e.g. `at-ctp-mocha.css`)
4. Click **Save**

> [!NOTE]
> The CSS files use `@import url('at.css')` — they must sit alongside the
> existing `at.css` file in `/www/` on the router. The base `at.css` is
> already present in every FreshTomato build; you only need to upload the
> Catppuccin theme file.

## What's Themed

| Element | Catppuccin Role |
|---------|----------------|
| Page background | `base` |
| Sidebar / nav panel | `mantle` |
| Card / input backgrounds | `crust` |
| Primary text | `text` |
| Links | `blue` |
| Link hover | `mauve` |
| Buttons, active states | `mauve` (accent) |
| Nav icon masks | `mauve` (via `--tomato-accent-color`) |
| Dropdown arrow ▾ | `mauve` (SVG fill override) |
| Checkbox ✓ | `mauve` (SVG fill override) |
| Radio • | `mauve` (SVG fill override) |
| Error / warning ⚠ | `mauve` (SVG fill override) |
| Control borders | `surface1` |
| Scrollbar thumb | `surface2` |
| Bandwidth graphs | `invert` filter (dark flavors) |
| Box shadows | `mauve` RGB glow (dark) / `text` RGB (Latte) |

## How It Works

FreshTomato's modern web UI (`at.css`) exposes its entire color system as CSS
custom properties (`--tomato-*`). Each theme file simply:

1. `@import url('at.css')` — loads the full base stylesheet
2. Overrides the 7 core color variables in `:root { }`
3. Redefines the 4 SVG form-control icons with Catppuccin mauve fill
4. Adds link colors, scrollbar styles, and graph image filters

No modifications to `at.css` are needed.

## Preview

Open `preview/index.html` in any modern browser to see all 4 flavors
side-by-side before installing on a router.

## License

MIT — matching the Catppuccin project license.

## Credits

- **[Catppuccin](https://github.com/catppuccin/catppuccin)** — palette & design system
- **[tsg2k2](https://github.com/tsg2k2)** — AdvaFresh theme for FreshTomato (base `at.css`)
- **[FreshTomato Project](https://github.com/FreshTomato-Project/freshtomato-mips)** — firmware
