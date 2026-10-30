---
author: Chiho Song
pubDatetime: 2026-06-24T00:00:00Z
title: Ghostty Korean Font Setup and Shortcuts
slug: ghostty-korean-setup
featured: false
draft: false
tags:
  - ghostty
  - terminal
  - dotfiles
  - setup
  - korean
description: How to fix broken Korean text in Ghostty with fallback fonts, plus Vim-style split shortcuts.
lang: en
---

After installing Ghostty for the first time, English text may look clean while Korean comes out broken or as tofu boxes (□). The cause is simple: the font you specified has no Hangul glyphs. Ghostty solves this by letting you declare `font-family` multiple times.

---

## Font Fallback Setup

In Ghostty, `font-family` isn't something you write only once. If you declare it multiple times, it **falls back from top to bottom in order.** If a font earlier in the list doesn't have a glyph, Ghostty looks for it in the next font.

```
font-family = "JetBrainsMonoNL Nerd Font Mono"
font-family = "MesloLGS Nerd Font"
font-family = "NanumGothic"
```

- `JetBrainsMonoNL Nerd Font Mono`: The main English coding font. Most characters are handled here.
- `MesloLGS Nerd Font`: Fills in Nerd Font icons.
- `NanumGothic`: Korean fallback. Korean glyphs missing from the first two fonts come from here.

Adding just the third line fixes broken Korean text.

---

## Full Config

```
theme = Monokai Pro Octagon

font-family = "JetBrainsMonoNL Nerd Font Mono"
font-family = "MesloLGS Nerd Font"
font-family = "NanumGothic"

font-size = 14

keybind = ctrl+h=goto_split:left
keybind = ctrl+l=goto_split:right
keybind = ctrl+k=goto_split:up
keybind = ctrl+j=goto_split:down
```

The config file lives at `~/.config/ghostty/config`.

---

## Shortcuts

### Vim-Style Split Navigation

Ghostty supports terminal splits out of the box. The default shortcuts use arrow keys, but if you use Vim, switching to `hjkl` feels natural.

```
keybind = ctrl+h=goto_split:left
keybind = ctrl+l=goto_split:right
keybind = ctrl+k=goto_split:up
keybind = ctrl+j=goto_split:down
```

### Default Shortcuts

Creating and managing splits:

| Shortcut          | Action                              |
| ----------------- | ----------------------------------- |
| `cmd+d`           | Split vertically (new pane right)   |
| `cmd+shift+d`     | Split horizontally (new pane below) |
| `cmd+w`           | Close current split                 |
| `cmd+[` / `cmd+]` | Go to previous / next split         |

Tabs:

| Shortcut                      | Action              |
| ----------------------------- | ------------------- |
| `cmd+t`                       | New tab             |
| `cmd+shift+[` / `cmd+shift+]` | Previous / next tab |
| `cmd+1` ~ `cmd+9`             | Go to tab by number |

Other:

| Shortcut      | Action             |
| ------------- | ------------------ |
| `cmd+,`       | Open config file   |
| `cmd+shift+,` | Reload config      |
| `cmd+enter`   | Toggle full screen |
| `cmd+k`       | Clear terminal     |

---

## Check Font Installation

If NanumGothic isn't installed, the fallback won't work. On macOS:

```sh
brew install --cask font-nanum-gothic
```

After installing, restart Ghostty and Korean text will render correctly.
