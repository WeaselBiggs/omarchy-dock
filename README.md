# My Dock

A clone of Omarchy's stock dock (`omarchy.dock`) with one addition: a short
accent-coloured line under every icon whose app has a window open, so a glance
at the dock says what is already running and what is still just pinned. The
stock dock only fades the icons of pinned apps that are not running; the line
is the same signal in a form that reads at any distance and on any icon.

Everything else is the stock dock: it shows on floating workspaces only,
pinned apps first, then anything else with a window, then the app grid.
`Dock.qml` is the service (window list, pins, launching), `DockWindow.qml`
draws one screen's dock, `AppGrid.qml` is the grid of every app,
`DockModel.js` matches windows to desktop entries.

## Install

```bash
omarchy plugin add https://github.com/WeaselBiggs/omarchy-dock.git --enable
```

Or drop the folder into `~/.config/omarchy/plugins/dan.dock/` and run
`omarchy plugin enable dan.dock`. The manifest marks the plugin as a clone of
`omarchy.dock`, so enabling it takes the stock dock out of service and
`omarchy plugin disable dan.dock` puts the stock dock back. Nothing beyond
Omarchy 4.5 is needed.

The clone started life as `omarchy plugin clone omarchy.dock`; the stock code
is Omarchy's, under the MIT licence, and `LICENSE` says so.

## Running mark

`runningMark` in `DockWindow.qml` is a `Rectangle` in each app slot:

- **Where** — between the icon and the screen edge, centred on the icon. It
  follows the dock position: under the icon for a bottom dock, above it for a
  top dock, beside it on the edge side for a left or right dock.
- **Size** — 2 px thick (scaled with the font scale), 40 % of the icon width
  long, rounded ends.
- **Colour** — the theme accent (`Commons.Color.accent`), so it follows a
  theme change like the rest of the shell.
- **When** — shown while the app has at least one window the dock counts.
  Hidden while the app is in its launch bounce, so the line appears the moment
  the window lands rather than flickering under the bouncing icon. Fades in
  and out over 180 ms; with animations off it just switches.
- **Drag** — it carries the same shift as the icon while another icon is being
  dragged past, so it stays under its own app.

What counts as a window is unchanged from the stock dock: windows on special
workspaces (the quake console, for instance) are left out, and a window is
matched to a pinned app by its desktop entry, so an app whose window class
differs from its desktop id shows as a separate unpinned entry instead.

## What the clone had to fix

A cloned plugin gets a scoped shell handle rather than the first-party one,
and that handle carries no `shellConfig`, `defaultsConfig` or app library. Out
of the box the clone therefore showed no pins and an empty app grid. Two
changes put that back:

- `Dock.qml` reads the `dock` section of `~/.config/omarchy/shell.json` and of
  `$OMARCHY_PATH/config/omarchy/shell.json` itself, through watched
  `FileView`s, so `omarchy dock ...` still takes effect live.
- `manifest.json` declares the `menu` kind with an inert `Menu.qml`, which is
  what makes the shell hand a plugin an app library. The dock has no menu of
  its own; the file is an empty `Item`.

## Using it

Left-click an icon to bring the app forward (or launch it), middle-click to
launch another instance, right-click for pin/unpin, close and the like. Drag
icons to reorder pins. The grid button opens every app; type to narrow, Enter
launches.

```bash
omarchy dock position [top|bottom|left|right]
omarchy dock autohide [on|off|toggle]
omarchy dock pinned <desktop-id>...      # set the whole pin list
omarchy dock pin <desktop-id>            # or unpin
omarchy-shell omarchy.dock apps          # open the app grid
```

The IPC name stays `omarchy.dock`, so existing keybindings keep working.

## Files

```
manifest.json    plugin manifest (service + menu kinds)
Dock.qml         service: windows, pins, config files, launching
DockWindow.qml   one screen's dock, app slots, the running mark
AppGrid.qml      the all-apps grid
DockModel.js     window-to-entry matching, ordering
Menu.qml         inert; only there to unlock the app library
```

QML edits hot-reload ("Local plugin changed, reloading" in the shell log),
but a manifest change needs `omarchy restart shell`. When the stock dock
changes in an Omarchy update, `diff -ru /usr/share/omarchy/shell/plugins/dock
~/.config/omarchy/plugins/dan.dock` shows exactly what the clone carries on
top of it.

## Related plugins

More of the same for the Omarchy shell, each installed with `omarchy plugin add <url> --enable`:

- [omarchy-vapour](https://github.com/WeaselBiggs/omarchy-vapour) — playtime for Steam, Battle.net and anything else that opens a window, in the bar.
- [omarchy-thunderclock](https://github.com/WeaselBiggs/omarchy-thunderclock) — the stock clock with Thunderbird calendar events marked on the calendar.
- [omarchy-thermals](https://github.com/WeaselBiggs/omarchy-thermals) — system temperatures and fans from hwmon, with a panel listing every probe.
