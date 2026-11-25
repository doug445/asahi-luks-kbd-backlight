# asahi-luks-kbd-backlight

On Fedora Asahi Remix (Apple Silicon MacBooks), the keyboard backlight is dark while you type the LUKS passphrase. This makes it light at 100% at the prompt, and it stays on for as long as the prompt waits.

```
git clone https://github.com/doug445/asahi-luks-kbd-backlight
cd asahi-luks-kbd-backlight
sudo ./install.sh               # install, rebuild every initramfs, verify
sudo ./install.sh --uninstall   # remove, rebuild every initramfs
./install.sh --check            # status (with sudo it also inspects each image)
```

Tested on a 14" MacBook Pro (M2 Max) running Fedora Asahi Remix 45: the keys are lit at the Plymouth passphrase prompt, for the regular kernels and the rescue kernel.

It is also part of [AsahiLocker](https://github.com/doug445/AsahiLocker), under `extras/kbd-backlight/`.

## Why it is dark

The backlight is a PWM LED (`pwm-apple` + `leds-pwm`). The stock Asahi dracut config does not put either driver in the initramfs, so the LED does not exist until the real root has mounted.

## What it installs

- `/usr/lib/dracut/modules.d/90asahi-kbd-backlight/`: a dracut module that adds the two drivers and one udev rule:
  `ACTION=="add", SUBSYSTEM=="leds", KERNEL=="kbd_backlight", ATTR{brightness}="$attr{max_brightness}"`
- `/etc/dracut.conf.d/90-asahi-kbd-backlight.conf`: enables the module, so kernel updates include it.

It rebuilds the initramfs of every kernel in `/boot`, plus the `0-rescue` image. The rescue image is rebuilt with the exact dracut arguments it was first built with (dracut records them in `usr/lib/dracut/build-parameter.txt`), so a host-only rescue stays host-only. If that record is missing, it uses the stock `--no-hostonly -a rescue` recipe. The new image is written to `.new` and replaces the old one only after it passes a check: the `rescue` module and the backlight module must both be present, and `/boot` must have had room for the build.

Kernel updates do not touch an existing rescue image. Re-run `./install.sh` after rebuilding it by hand. The rule exists only in the initramfs. After unlock, `systemd-backlight` and the desktop take over as usual.

## Limits

- It applies only to the initramfs prompt (Plymouth or text). A GRUB `cryptomount` prompt for an encrypted `/boot` comes before Linux and cannot drive the LED.
- It needs a MacBook. Desktop Macs have no `kbd_backlight`, and the installer refuses to run on them.

## License

MIT. See [LICENSE](LICENSE).
