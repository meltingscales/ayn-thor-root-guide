# AYN Thor Rooting Guide (CachyOS / Arch Linux)

**Status:** Bootloader already shows `DEVICE STATE - unlocked` → no data wipe needed.
All installed games, GameNative containers, and saves survive this process.

**Reference:** XDA thread "Ayn thor rooting guide" — https://xdaforums.com/t/ayn-thor-rooting-guide.4767974/
(Contains the init_boot dump script, a pre-dumped stock init_boot, and a pre-patched Magisk image.)

---

## 0. Insurance first

- [ ] Join the AYN Discord and download the **full stock ROM** someone posted there — do this *before* flashing anything, so you have a revert path.
- [ ] Keep an untouched copy of your stock `init_boot.img` (step 2) somewhere safe.

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

Preferred: dump your own (guarantees firmware match) using the script from the XDA thread.
Shortcut: use the pre-dumped `init_boot.img` attached to the thread (only if your firmware version matches).

Keep a backup copy: `cp init_boot.img init_boot.stock.img`

## 3. Patch with Magisk

1. Install the latest **Magisk APK** on the Thor (from the official GitHub: https://github.com/topjohnwu/Magisk/releases).
2. Copy `init_boot.img` to the Thor:
   ```bash
   adb push init_boot.img /sdcard/Download/
   ```
3. In Magisk: **Install → Select and Patch a File** → choose the image.
4. Pull the result back:
   ```bash
   adb pull /sdcard/Download/magisk_patched-XXXXX.img .
   ```

(The XDA thread includes a pre-patched image, but patching yourself gets the current Magisk version.)

## 4. Flash

```bash
adb reboot bootloader
fastboot devices                 # must list the Thor
fastboot flash init_boot magisk_patched-XXXXX.img
fastboot reboot
```

Optional (done by a user in the thread after confirming both slots were identical):
```bash
fastboot flash init_boot_a magisk_patched-XXXXX.img
fastboot flash init_boot_b magisk_patched-XXXXX.img
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
fastboot flash init_boot init_boot.stock.img
```
If things go badly wrong, restore the full stock ROM from the AYN Discord.
