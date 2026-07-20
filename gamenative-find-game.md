# Finding a GameNative Game Install via adb (Dr. Lunatic Supreme With Cheese)

**Prereq:** rooted Thor (see [ayn-thor-root-guide.md](ayn-thor-root-guide.md)), USB debugging on.

## 1. Get a root shell

```bash
adb shell
su          # approve the Magisk prompt on the Thor's screen the first time
```

Prompt should change to `#`. All commands below run in this shell.

## 2. Set the package name

GameNative's package is `app.gamenative`:

```bash
PKG=app.gamenative
```

## 3. Locate the game

GameNative keeps data in two places — private app data and app-external storage. Search both:

```bash
find /data/data/$PKG /storage/emulated/0/Android/data/$PKG -iname '*lunatic*' 2>/dev/null
```

If that finds nothing (name stored differently), look for the Steam library layout instead:

```bash
find /data/data/$PKG /storage/emulated/0/Android/data/$PKG -ipath '*steamapps/common*' -maxdepth 8 2>/dev/null
```

The install dir is the `steamapps/common/<Game Name>/` folder. Orient yourself:

```bash
ls -la "/path/found/above"
```

## 4. Pull files to the PC (optional)

From the PC (not the adb shell) — `adb pull` can't read root-only paths directly, so stage through internal storage:

```bash
adb shell su -c 'cp -a "/data/data/app.gamenative/.../Dr. Lunatic Supreme With Cheese" /storage/emulated/0/Download/drl-backup'
adb pull /storage/emulated/0/Download/drl-backup data/
adb shell rm -r /storage/emulated/0/Download/drl-backup
```

If the game turned out to live under `/storage/emulated/0/Android/data/...` instead, try `adb pull` on that path directly first — fall back to the root-staging method above only if it's permission-denied.

## Result on my Thor (2026-07-19)

Install found at:

```
/data/data/app.gamenative/Steam/steamapps/common/Dr. Lunatic Supreme With Cheese
```

The same folder also appears inside the Wine prefix (container for Steam appid 2547330) — confirmed a symlink to the path above (via `/data/user/0/`, which is the same location as `/data/data/`), not a second copy:

```
/data/data/app.gamenative/files/imagefs_shared/home/xuser-STEAM_2547330/.wine/drive_c/Program Files (x86)/Steam/steamapps/common/Dr. Lunatic Supreme With Cheese
```

Back up the first path. Ignore `.DepotDownloader/staging/` — download-time chunks, not game data.

## Notes

- Dr. Lunatic is an old-school game: profiles/saves live **inside the install folder** (`profiles/`, `*.sav`), not in a separate save dir — back up the whole game folder and you have everything.
- The Wine prefix (`drive_c`, per-container settings) lives under `/data/data/$PKG` too; the searches above will surface it if you need files the game wrote to `C:\`.
