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

The web interface (`httpd`) serves files from a directory controlled by the
**`web_dir`** NVRAM variable (default: `/www`).  
Each page loads a CSS file named by the **`web_css`** NVRAM variable:

```html
<!-- in every .asp page -->
<link rel="stylesheet" href="<web_css>.css">
```

So setting `web_css=at-ctp-mocha` causes every page to request
`at-ctp-mocha.css` from whichever directory `web_dir` points to.

> [!IMPORTANT]
> `/www` is part of the **read-only SquashFS firmware image**.  
> You **cannot write files there** — `cp file /www/` will fail with
> `Read-only file system`.  
> Use one of the writable methods below instead.

---

## Installation

### Method 1 — JFFS (recommended, survives reboots)

JFFS is a writable, persistent partition stored in the router's flash.
Enable it first under **Administration → JFFS2** if you haven't already.

```bash
# 1. Create a directory to hold the custom GUI files
mkdir -p /jffs/www

# 2. Download the theme file (replace with your chosen flavor)
wget -O /jffs/www/at-ctp-mocha.css \
  https://raw.githubusercontent.com/thelok1s/catppuccin-freshtomato/main/at-ctp-mocha.css

# --- OR copy from your machine over SCP ---
# scp at-ctp-mocha.css root@192.168.1.1:/jffs/www/

# 3. Point the web server at your JFFS directory
#    CAUTION: if web_dir is wrong you'll lose web access — double-check the path
nvram set web_dir=/jffs/www

# 4. Select the theme (filename without .css extension)
nvram set web_css=at-ctp-mocha

# 5. Commit and apply
nvram commit

# 6. Restart the web server (or reboot)
service httpd restart
```

Then hard-refresh your browser (**Ctrl+Shift+R** / **Cmd+Shift+R**).

> [!NOTE]
> **How `web_dir` works:** FreshTomato's httpd normally serves files from
> `/www` (read-only firmware). When you set `web_dir=/jffs/www`, the httpd
> serves files from `/jffs/www` **instead**. Your custom CSS file must live
> there. You do **not** need to copy all the firmware `.asp` / `.js` files —
> only your CSS file needs to be in that directory; the firmware falls back to
> `/www` for everything else it can't find in `web_dir`.
>
> *(This fallback behavior is confirmed in the FreshTomato httpd source.)*

---

### Method 2 — /tmp (temporary, lost on reboot)

Useful for testing a theme without committing it permanently.

```bash
# 1. Download the file to /tmp
wget -O /tmp/at-ctp-mocha.css \
  https://raw.githubusercontent.com/thelok1s/catppuccin-freshtomato/main/at-ctp-mocha.css

# --- OR copy over SCP ---
# scp at-ctp-mocha.css root@192.168.1.1:/tmp/

# 2. Point httpd at /tmp and set the theme
nvram set web_dir=/tmp
nvram set web_css=at-ctp-mocha
nvram commit

# 3. Restart the web server
service httpd restart
```

> [!WARNING]
> `/tmp` is RAM-based and is wiped on every reboot. The theme will disappear
> after a restart. Use JFFS for a permanent install.

---

### Method 3 — Web UI (no SSH required)

If you prefer not to use SSH:

1. First download the CSS file to your PC.
2. Go to **Administration → JFFS2** — enable JFFS and wait for it to format.
3. Use a tool like [WinSCP](https://winscp.net) or `scp` to place the file in `/jffs/www/`.
4. Go to **Administration → Admin Access**.
5. Set **"Directory with GUI files"** to `/jffs/www`.
6. Set **"Theme UI"** to `at-ctp-mocha` (the filename stem, no `.css`).
7. Click **Save**.

---

## Switching Flavors

After initial setup, switching is a one-liner (no path change needed):

```bash
nvram set web_css=at-ctp-latte && nvram commit && service httpd restart
```

Download all four flavors at once:

```bash
for f in mocha macchiato frappe latte; do
  wget -O /jffs/www/at-ctp-$f.css \
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
| Dropdown arrow ▾ | `mauve` (SVG fill override) |
| Checkbox ✓ | `mauve` (SVG fill override) |
| Radio • | `mauve` (SVG fill override) |
| Error / warning ⚠ | `mauve` (SVG fill override) |
| Control borders | `surface1` |
| Scrollbar thumb | `surface2` |
| Bandwidth graphs | `invert` filter (dark flavors only) |
| Box shadows | `mauve` RGB glow (dark) / `text` RGB (Latte) |

---

## How the CSS Is Loaded (technical detail)

Every FreshTomato page contains:

```html
<link rel="stylesheet" href="tomato.css">
<link rel="stylesheet" href="<web_css>.css" id="guicss">
```

The second `<link>` is the theme slot. Setting `web_css=at-ctp-mocha` causes
the browser to request `at-ctp-mocha.css` from the router's httpd.

Our theme files start with `@import url('at.css')` to load the base AdvaFresh
stylesheet, then override only the `--tomato-*` CSS custom properties and a
few additional selectors (links, scrollbars, icon fills). No JavaScript is
needed, and no other files need to be replaced.

---

## Preview

Open `preview/index.html` in any modern browser to see all 4 flavors
interactively before installing on a router.

---

## License

MIT — matching the Catppuccin project license.

## Credits

- **[Catppuccin](https://github.com/catppuccin/catppuccin)** — palette & design system
- **[tsg2k2](https://github.com/tsg2k2)** — AdvaFresh theme for FreshTomato (base `at.css`)
- **[FreshTomato Project](https://github.com/FreshTomato-Project/freshtomato-mips)** — firmware
- **[FreshTomato Wiki — Admin Access](https://wiki.freshtomato.org/doku.php/admin_access)** — official documentation
