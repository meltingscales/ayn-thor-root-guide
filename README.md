# AYN Thor Root & GameNative Guides

Guides for rooting the AYN Thor handheld from Linux and doing useful things with root afterward. Written against a real device (CachyOS PC, Thor on stock ROM); every command was actually run.

## Guides

1. **[Rooting the AYN Thor](ayn-thor-root-guide.md)** — bootloader check/unlock, dumping stock `init_boot` via the Thor's built-in root script runner, patching with Magisk, flashing, and revert paths (including where to get the full stock ROM now that the Discord link is dead).
2. **[Finding a GameNative game install via adb](gamenative-find-game.md)** — locating a game's files inside GameNative's app data with a Magisk root shell, using Dr. Lunatic Supreme With Cheese as the example; explains the Wine-prefix symlink layout.
3. **[Installing the All Supreme Worlds pack](supreme-with-cheese-worlds-ayn-thor.md)** — installing hamumu.com's 529-world pack into the Steam re-release (HamSandwich port): why files go in `appdata/supreme/` and not the folder old instructions claim, plus the SELinux/ownership gotchas of writing into `/data/data/` as root.

## Layout

- `data/` — firmware images, dumps, and downloads referenced by the guides. Gitignored; nothing copyrighted or device-specific is committed.

## Key hard-won facts

- `adb root` does not work on the Thor (production build) — use **Settings → Thor settings → Run script as root**.
- The full stock ROM (`Thor_20251120_ota.zip`) is attached to [post #5 of the XDA rooting thread](https://xdaforums.com/t/ayn-thor-rooting-guide.4767974/), not the AYN Discord.
- Never `restorecon` inside `/data/data/<app>` — it strips per-uid SELinux categories and silently breaks the app's writes. Derive the label with `ls -dZ` and apply with `chcon`.
