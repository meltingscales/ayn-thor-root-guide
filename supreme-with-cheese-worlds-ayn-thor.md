# Installing the All Supreme Worlds Pack into GameNative (AYN Thor)

Installs `all_supreme_worlds.zip` (529 user-made worlds from hamumu.com) into the GameNative install of Dr. Lunatic Supreme With Cheese.

**Prereqs:** rooted Thor with Magisk (see [ayn-thor-root-guide.md](ayn-thor-root-guide.md)), game location known (see [gamenative-find-game.md](gamenative-find-game.md)).

Game dir: `/data/data/app.gamenative/Steam/steamapps/common/Dr. Lunatic Supreme With Cheese`

## Where files must go (important)

The Steam re-release is the **HamSandwich port** (`supreme.exe` + layered VFS), not the 2011 standalone build. Web instructions saying "unzip into the game folder / `Worlds` directory" are for the old build and **do not work** — the port never scans the game-root `worlds/` folder. It reads, in priority order:

1. `appdata/supreme/` — read-write layer (saves, profiles, **user-added content**)
2. `assets/supreme/` — overrides
3. `installers/supreme8_install.exe/` — the unpacked original installer (stock 85 worlds live here)

So the zip's `worlds/`, `user/`, and `music/` dirs go under **`appdata/supreme/`**.

## 1. Unzip on the PC and push to the Thor

```bash
unzip -q ~/Downloads/all_supreme_worlds.zip -d /tmp/supreme_worlds
adb push /tmp/supreme_worlds /storage/emulated/0/Download/supreme_worlds
```

(~478 MB / 2594 files; took ~30 s over USB.)

## 2. Merge into appdata as root

Two gotchas the script handles:

- Files copied by root into `/data/data/` are root-owned — `chown` back to the game's uid.
- SELinux: app data files need the app's per-uid category label (e.g. `s0:c128,c256,c512,c768`). **Do not use `restorecon`** — it strips the categories, which silently breaks the app's *writes* (saves). Use `chcon` with the label copied from an untouched sibling dir (`ls -dZ` on the game dir's parent shows it).

```bash
cat > /tmp/install_worlds.sh <<'EOF'
GAME="/data/data/app.gamenative/Steam/steamapps/common/Dr. Lunatic Supreme With Cheese"
APP="$GAME/appdata/supreme"
CTX="u:object_r:app_data_file:s0:c128,c256,c512,c768"   # from ls -dZ on a sibling dir
OWN=$(stat -c "%u:%g" "$GAME") || exit 1
for d in worlds user music; do
  mkdir -p "$APP/$d" || exit 1
  cp -a "/storage/emulated/0/Download/supreme_worlds/$d/." "$APP/$d/" || exit 1
done
chown -R "$OWN" "$APP" || exit 1
chcon -R "$CTX" "$APP" || exit 1
echo "DONE worlds=$(ls "$APP/worlds" | wc -l)"
EOF
adb push /tmp/install_worlds.sh /storage/emulated/0/Download/
adb shell "su -c 'sh /storage/emulated/0/Download/install_worlds.sh'"
```

Expected: `DONE worlds=530` (529 + a `[name here].dlw` placeholder from the zip).

Quoting note: `adb shell su -c '<multi-line script>'` gets mangled by the local shell — push the script and run it with `sh` instead.

## 3. Clean up staging

```bash
adb shell rm -r /storage/emulated/0/Download/supreme_worlds /storage/emulated/0/Download/install_worlds.sh
```

## 4. Verify in-game

Launch DLSWC in GameNative → world select should list the pack alongside the stock worlds.

## Notes

- **No Steam Workshop for this game** — the in-game world browser uses Hamumu's leaderboard service (synced to `Steam/userdata/<uid>/2547330/leaderboard/worlds/` inside the Wine prefix via Steam Cloud). The 529-pack is a superset of it.
- The Wine-prefix path to the game (`.../imagefs_shared/home/xuser-STEAM_2547330/.wine/drive_c/...`) is a symlink to the real install — one copy, don't install things twice.
