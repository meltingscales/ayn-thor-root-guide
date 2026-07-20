# AYN Thor Rooting Guide (CachyOS / Arch Linux)

**Prerequisite: unlocked bootloader.** Check yours before starting — reboot into the bootloader (`adb reboot bootloader`) and read the screen:

- `DEVICE STATE - unlocked` → good, skip ahead. No data wipe needed; installed games, GameNative containers, and saves all survive the rooting process itself.
- `DEVICE STATE - locked` → you must unlock first. **WARNING: unlocking the bootloader performs a full factory reset — it wipes ALL data on the device** (apps, games, saves, accounts). Back up everything you care about first. Then: **Settings → Developer Options → OEM unlocking** (enable), reboot to bootloader, and run:
  ```bash
  fastboot flashing unlock
  ```
  Confirm on the device screen (volume keys + power). An unlocked bootloader also shows a warning screen on every boot and may trip strict app integrity checks — that's the trade for root.

**Reference:** XDA thread "Ayn thor rooting guide" — https://xdaforums.com/t/ayn-thor-rooting-guide.4767974/
(Contains the init_boot dump script, a pre-dumped stock init_boot, and a pre-patched Magisk image.)

---

## 0. Insurance first

> **Update 2026-07-19:** The full stock ROM is **no longer downloadable from the AYN Discord** (emailed AYN support, but moot): it's attached to **post #5 of the XDA thread** as `Thor_20251120_ota.zip` (1.8 GB, build 2025-11-20 — old but can OTA-update afterward).

- [ ] **Download `Thor_20251120_ota.zip` from XDA post #5 into `data/`** — the brick-recovery path. Note (post #9): flashing it may require extracting `payload.bin` and fastboot-flashing each image manually (`payload-dumper-go` handles extraction).

Remaining insurance:

- [ ] **Dump your own stock `init_boot.img` (step 2) and back it up in ≥2 places** (PC + cloud). Primary revert path for the root process itself — `init_boot` is the *only* partition this guide flashes, and flashing the stock dump back fully unroots.
- [ ] Optional extra insurance: while dumping, also grab `boot.img` and `vendor_boot.img` (same dump method, different partition names). Not needed for this guide, but cheap to do now and impossible after a bad flash.
- [ ] Secondary source: the pre-dumped stock `init_boot_a.img` attached to the XDA thread (verify firmware version matches yours).

## 1. PC setup (CachyOS)

```bash
sudo pacman -S android-tools android-udev
```

- `android-tools` provides `adb` and `fastboot`.
- `android-udev` installs udev rules so fastboot works without root on the PC. Reboot or run `sudo udevadm control --reload && sudo udevadm trigger` after installing.
- No Windows driver hassle on Linux (the driver issue in the XDA thread is Windows-only). If `fastboot devices` shows nothing, try a different USB-C cable/port and check `lsusb`.

On the Thor: **Settings → About → tap Build Number 7×** to enable Developer Options, then enable **USB debugging**.

Verify:
```bash
adb devices        # authorize the prompt on the Thor
```

## 2. Get your init_boot image

Preferred: dump your own (guarantees firmware match). `adb root` does **not** work (`adbd cannot run as root in production builds`) — instead the Thor's stock ROM has a built-in root script runner: **Settings → Thor settings → Run script as root**.

1. On the PC, create the dump script and push it over:
   ```bash
   printf '%s\n' \
     'dd if=/dev/block/bootdevice/by-name/init_boot_a of=/storage/emulated/0/init_boot_a.img' \
     'dd if=/dev/block/bootdevice/by-name/init_boot_b of=/storage/emulated/0/init_boot_b.img' \
     > data/dump_init_boot.sh
   adb push data/dump_init_boot.sh /storage/emulated/0/
   ```
2. On the Thor: **Settings → Thor settings → Run script as root** → select `dump_init_boot.sh`.
3. Pull the dumps and clean up:
   ```bash
   adb pull /storage/emulated/0/init_boot_a.img data/
   adb pull /storage/emulated/0/init_boot_b.img data/
   adb shell rm '/storage/emulated/0/init_boot_*.img' /storage/emulated/0/dump_init_boot.sh
   ```

- Paths use `/storage/emulated/0` (internal storage). `/sdcard` is a symlink to the same place — no physical SD card is involved at any point.
- Sanity-check: each file should be ~8 MiB, not 0 bytes.
- Both slots are normally identical (differ only right after an OTA: `_a` newer, `_b` older). Verify: `sha256sum data/init_boot_a.img data/init_boot_b.img`. Active slot: `adb shell getprop ro.boot.slot_suffix`. Use the active slot's image for patching.
- Same method dumps the optional extras from step 0: add `boot_a`/`vendor_boot_a` etc. lines to the script.

Shortcut: use the pre-dumped `init_boot.img` attached to the thread (only if your firmware version matches).

Keep a backup copy: `cp data/init_boot_a.img data/init_boot.stock.img` (assuming `_a` is active)

All images live in `data/` (gitignored — never committed).

## 3. Patch with Magisk

1. Install the latest **Magisk APK** on the Thor (from the official GitHub: https://github.com/topjohnwu/Magisk/releases).
2. Copy the active slot's image to the Thor:
   ```bash
   adb push data/init_boot_a.img /storage/emulated/0/Download/
   ```
3. In Magisk: **Install → Select and Patch a File** → choose the image.
4. Pull the result back:
   ```bash
   adb pull /storage/emulated/0/Download/magisk_patched-XXXXX.img data/
   ```

(The XDA thread includes a pre-patched image, but patching yourself gets the current Magisk version.)

## 4. Flash

```bash
adb reboot bootloader
fastboot devices                 # must list the Thor
fastboot flash init_boot data/magisk_patched-XXXXX.img
fastboot reboot
```

Optional (done by a user in the thread after confirming both slots were identical):
```bash
fastboot flash init_boot_a data/magisk_patched-XXXXX.img
fastboot flash init_boot_b data/magisk_patched-XXXXX.img
```

## 5. Verify

Open the Magisk app → should show **Installed: <version>**. Done — you're rooted.

## 6. Post-root

- **GameNative files:** use a root-capable file manager (MiXplorer, Material Files + root, FX) to browse `/data/data/` → GameNative's package folder. Containers, Wine prefixes, `drive_c`, and saves are all there with full read/write. Back up saves for games without Steam Cloud.
- **Root detection:** some apps may complain; the **Play Integrity Fix** Magisk module reportedly resolves it on the Thor. Netflix untested per the thread.
- **OTA updates:** system updates will likely replace init_boot and remove root — re-patch and re-flash after updating.

## Revert path

Flash the stock `init_boot.stock.img` back:
```bash
fastboot flash init_boot data/init_boot.stock.img
```
This undoes everything this guide changes — no full ROM needed for that.

If things go badly wrong *beyond* init_boot (shouldn't happen if you only run the commands above): flash the full stock ROM — `data/Thor_20251120_ota.zip` from XDA post #5. May require extracting `payload.bin` and fastboot-flashing each image manually (`payload-dumper-go`).
