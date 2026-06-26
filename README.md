# 🐱 Catppuccin for FreshTomato

> Soothing pastel theme for the [FreshTomato](https://github.com/FreshTomato-Project/freshtomato-mips) router firmware web UI.

[![Catppuccin](https://img.shields.io/badge/Catppuccin-pastel_theme-CBA6F7?style=flat-square)](https://catppuccin.com)

## Flavors

| File | Flavor | Background | Feel |
|------|--------|------------|------|
| `at-ctp-mocha.css` | 🌙 Mocha | `#1e1e2e` | Deepest dark — ideal for night use |
| `at-ctp-macchiato.css` | 🌺 Macchiato | `#24273a` | Rich violet-dark — evening |
| `at-ctp-frappe.css` | 🪴 Frappé | `#303446` | Cool medium-dark — relaxed |
| `at-ctp-latte.css` | 🌻 Latte | `#eff1f5` | Warm cream — daytime / light |

---

## How FreshTomato Theming Works

Every page loads two stylesheets:

```html
<link rel="stylesheet" href="tomato.css">
<link rel="stylesheet" href="<web_css>.css" id="guicss">
```

The second `<link>` is the theme slot — its filename is controlled by the **`web_css`** NVRAM variable.

The httpd serves files from the directory set in **`web_dir`** NVRAM (default: `/www`).

> [!IMPORTANT]
> `/www` is part of the **read-only SquashFS firmware image** — you cannot write
> files there. `cp file /www/` will fail with `Read-only file system`.
>
> `web_dir` **replaces** `/www` entirely — it is not an overlay. If you set
> `web_dir` to a directory that doesn't contain all the firmware UI files
> (`.asp`, `.js`, `.css`…), the web interface will return **500 Unknown Read error**
> on every page. You cannot point `web_dir` at just a single CSS file.

---

## Restoring Access After a 500 Error

If you set `web_dir` incorrectly and lost web access, fix it over SSH:

```bash
nvram unset web_dir
nvram commit
service httpd restart
```

---

## Installation

The supported approach is to mirror all of `/www` into a writable location,
drop the theme CSS there, and point `web_dir` at that location.

### Method A — Init Script into /tmp (recommended, no extra flash needed)

This copies `/www` into a tmpfs ramdisk on every boot, then adds the theme CSS.
`/tmp` is reset on each reboot — the copy takes ~1–2 seconds and uses ~3–5 MB of RAM.

**Step 1 — Store the CSS in JFFS (persistent)**

```bash
# Enable JFFS first: Administration → JFFS2 → Enable

mkdir -p /jffs

# Download your chosen flavor:
wget -O /jffs/at-ctp-macchiato.css \
  https://raw.githubusercontent.com/thelok1s/catppuccin-freshtomato/main/at-ctp-macchiato.css

# Or copy from your PC:
# scp at-ctp-macchiato.css root@192.168.1.1:/jffs/
```

**Step 2 — Create the init script**

Go to **Administration → Scripts → Init** and paste:

```sh
#!/bin/sh
# Catppuccin FreshTomato theme — boot setup
mkdir -p /tmp/www
cp -a /www/. /tmp/www/
cp /jffs/at-ctp-macchiato.css /tmp/www/
```

**Step 3 — Set NVRAM and apply**

```bash
nvram set web_dir=/tmp/www
nvram set web_css=at-ctp-macchiato
nvram commit
```

Run the init script once manually (so you don't have to reboot yet), then restart httpd:

```bash
mkdir -p /tmp/www
cp -a /www/. /tmp/www/
cp /jffs/at-ctp-macchiato.css /tmp/www/
service httpd restart
```

Hard-refresh your browser (**Ctrl+Shift+R**). On every subsequent reboot the init script will repopulate `/tmp/www` automatically before httpd starts.

---

### Method B — Permanent copy in JFFS

Copies all of `/www` into JFFS flash — survives reboots without a script,
but uses ~3–5 MB of JFFS space permanently.

**Check available space first:**

```bash
df -h /jffs
du -sh /www
```

If you have enough free space:

```bash
mkdir -p /jffs/www
cp -a /www/. /jffs/www/

# Download the theme CSS:
wget -O /jffs/www/at-ctp-macchiato.css \
  https://raw.githubusercontent.com/thelok1s/catppuccin-freshtomato/main/at-ctp-macchiato.css

# Or copy from PC:
# scp at-ctp-macchiato.css root@192.168.1.1:/jffs/www/

nvram set web_dir=/jffs/www
nvram set web_css=at-ctp-macchiato
nvram commit
service httpd restart
```

> [!WARNING]
> If JFFS space runs low in the future, the router may misbehave.
> Method A is safer for flash longevity.

---

## Switching Flavors

After initial setup, switching is instant — just update the NVRAM value and restart httpd:

```bash
# Copy the new CSS first (adjust path for Method A or B):
wget -O /jffs/at-ctp-latte.css \
  https://raw.githubusercontent.com/thelok1s/catppuccin-freshtomato/main/at-ctp-latte.css
cp /jffs/at-ctp-latte.css /tmp/www/   # Method A only

# Switch:
nvram set web_css=at-ctp-latte
nvram commit
service httpd restart
```

**Download all four flavors at once:**

```bash
for f in mocha macchiato frappe latte; do
  wget -O /jffs/at-ctp-$f.css \
    https://raw.githubusercontent.com/thelok1s/catppuccin-freshtomato/main/at-ctp-$f.css
done
```

---

## Reverting to Default

```bash
nvram unset web_dir
nvram set web_css=at
nvram commit
service httpd restart
```

---

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
| Dropdown arrow ▾ | `mauve` (SVG fill) |
| Checkbox ✓ / Radio • | `mauve` (SVG fill) |
| Control borders | `surface1` |
| Scrollbar thumb | `surface2` |
| Bandwidth graph images | `grayscale + invert` filter (dark flavors) |
| Box shadows | `mauve` RGB glow |

---

## How the Theme CSS Works

Our theme files do not replace the entire stylesheet — they load the
base AdvaFresh stylesheet first, then override only the color variables:

```css
@import url('at.css');   /* loads the full base UI stylesheet */

:root {
  --tomato-bg:        #1e1e2e;  /* base */
  --tomato-nav-bg:    #181825;  /* mantle */
  --tomato-accent-color: #cba6f7; /* mauve */
  /* … */
}
```

This means `at.css` must be reachable from the same httpd directory as the
theme CSS — which is why `/tmp/www` (or `/jffs/www`) must contain both.

---

## Preview

Open [`preview/index.html`](preview/index.html) in any modern browser to see all 4 flavors
interactively before installing on a router.

---

## License

MIT — matching the Catppuccin project license.

## Credits

- **[Catppuccin](https://github.com/catppuccin/catppuccin)** — palette & design system
- **[tsg2k2](https://github.com/tsg2k2)** — AdvaFresh theme for FreshTomato (base `at.css`)
- **[FreshTomato Project](https://github.com/FreshTomato-Project/freshtomato-mips)** — firmware
- **[FreshTomato Wiki](https://wiki.freshtomato.org/doku.php/admin_access)** — official documentation
