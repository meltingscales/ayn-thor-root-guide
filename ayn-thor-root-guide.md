# AYN Thor Rooting Guide (CachyOS / Arch Linux)

**Status:** Bootloader already shows `DEVICE STATE - unlocked` → no data wipe needed.
All installed games, GameNative containers, and saves survive this process.

**Reference:** XDA thread "Ayn thor rooting guide" — https://xdaforums.com/t/ayn-thor-rooting-guide.4767974/
(Contains the init_boot dump script, a pre-dumped stock init_boot, and a pre-patched Magisk image.)

---

## 0. Insurance first

> **Update 2026-07-19:** The full stock ROM is **no longer downloadable from the AYN Discord**. Emailed AYN support asking for it — awaiting reply.

Without a full ROM, the insurance plan is:

- [ ] **Dump your own stock `init_boot.img` (step 2) and back it up in ≥2 places** (PC + cloud). This is now the primary revert path — and it's genuinely sufficient for the root process itself, since `init_boot` is the *only* partition this guide flashes. Flashing the stock dump back fully unroots.
- [ ] Optional extra insurance: while dumping, also grab `boot.img` and `vendor_boot.img` (same dump method, different partition names). Not needed for this guide, but cheap to do now and impossible after a bad flash.
- [ ] Secondary source: the pre-dumped stock `init_boot.img` attached to the XDA thread (verify firmware version matches yours).
- [ ] If AYN support sends the full ROM, save it to `data/` — it's the only fix for a catastrophic brick (wrong partition flashed, corrupted flash, failed OTA). Fine to proceed without it given the small blast radius of an `init_boot`-only flash, but don't flash anything beyond `init_boot` until you have it.

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

Preferred: dump your own (guarantees firmware match). The Thor's stock ROM permits `adb root`, so no exploit or existing root needed — the XDA thread's "dump script" is just `dd` over rooted adb:

```bash
adb root                          # restarts adbd with root
adb shell getprop ro.boot.slot_suffix   # note active slot (_a or _b)
adb exec-out "dd if=/dev/block/bootdevice/by-name/init_boot_a 2>/dev/null" > data/init_boot_a.img
adb exec-out "dd if=/dev/block/bootdevice/by-name/init_boot_b 2>/dev/null" > data/init_boot_b.img
adb unroot
```

- If `adb root` returns `adbd cannot run as root in production builds`, look for a root/ADB toggle in the Thor's settings (AYN ships a permissive build; the XDA thread runs this same script via adb root).
- `exec-out` streams the partition straight to the PC — nothing written on the Thor. Sanity-check the result: each file should be several MiB (typically 8 MiB), not 0 bytes.
- Users in the thread found both slots identical — verify yours: `sha256sum data/init_boot_a.img data/init_boot_b.img`. Use the active slot's image for patching.
- Same method dumps the optional extras from step 0: swap in `boot_a`/`vendor_boot_a` etc. as the partition name.

Shortcut: use the pre-dumped `init_boot.img` attached to the thread (only if your firmware version matches).

Keep a backup copy: `cp data/init_boot_a.img data/init_boot.stock.img` (assuming `_a` is active)

All images live in `data/` (gitignored — never committed).

## 3. Patch with Magisk

1. Install the latest **Magisk APK** on the Thor (from the official GitHub: https://github.com/topjohnwu/Magisk/releases).
2. Copy the active slot's image to the Thor:
   ```bash
   adb push data/init_boot_a.img /sdcard/Download/
   ```
3. In Magisk: **Install → Select and Patch a File** → choose the image.
4. Pull the result back:
   ```bash
   adb pull /sdcard/Download/magisk_patched-XXXXX.img data/
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

If things go badly wrong *beyond* init_boot (shouldn't happen if you only run the commands above): full stock ROM required. Not currently downloadable from Discord — waiting on AYN support (emailed 2026-07-19). Other options: ask in the XDA thread or AYN Discord for a re-upload.
