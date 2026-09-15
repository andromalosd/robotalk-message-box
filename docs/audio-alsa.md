# ALSA configuration for a split playback/capture setup

The single most valuable file in this build. Playback runs on the Pi's onboard
3.5 mm jack (ALSA card `Headphones`) and capture runs on a USB microphone (ALSA
card `CMTECK`, capture-only). Two different cards means the default PCM has to
be asymmetric.

## The config

Lives at **`/etc/asound.conf`** — system-wide, *not* `~/.asoundrc`. See
"Why not `~/.asoundrc`" below.

```
pcm.!default {
  type asym
  playback.pcm "plughw:Headphones,0"
  capture.pcm "plughw:CMTECK,0"
}
ctl.!default {
  type hw
  card Headphones
}
```

To create or restore it:

```bash
sudo tee /etc/asound.conf > /dev/null << 'EOF'
pcm.!default {
  type asym
  playback.pcm "plughw:Headphones,0"
  capture.pcm "plughw:CMTECK,0"
}
ctl.!default {
  type hw
  card Headphones
}
EOF
sync
sudo systemctl restart messagebox
```

Substitute your own card names — `aplay -l` and `arecord -l` will list them.

## Names, not numbers

`plughw:Headphones,0` rather than `plughw:0,0`, deliberately.

Card numbers are assigned in enumeration order at boot, so they are not stable
identifiers. After one reboot on this box, HDMI came up as card 0 and playback
started failing with:

```
audio open error: Unknown error 524
```

Card 0 was now an HDMI output with no monitor attached, which cannot be opened.
Nothing had changed in software; the number simply pointed somewhere else.

Card *names* are stable across reboots and across replugging USB devices. If a
device path in any config is really an enumeration index, treat it as a reboot
bug waiting to happen.

## Why not `~/.asoundrc`

The config originally lived in `~/.asoundrc` and disappeared twice from a
working box. Both times followed the box losing power without a clean shutdown —
recent writes had not been flushed to the microSD.

`/etc/asound.conf` is written deliberately and rarely, is followed by an
explicit `sync`, and is not touched again at runtime. Between that and the
shutdown discipline below, it has stayed put.

**Always check this file first when audio stops working:**

```bash
cat /etc/asound.conf
```

Empty or missing means recreate it with the command above.

## Volume

The Pi's onboard 3.5 mm jack is quiet. Raise it and persist the setting:

```bash
amixer -c Headphones sset PCM 100%
sudo alsactl store      # without this, the level is lost on reboot
```

The web UI's volume slider does **not** work on this hardware — the application
issues its mixer command against a hardcoded card name that only exists on the
kit's supported sound card. Harmless, but it means volume is set from the shell
here, with `alsamixer` (F6 selects the card, F3 the playback view) or the command
above.

## Verifying both directions

```bash
sudo systemctl stop messagebox      # release the devices first

speaker-test -t wav -c 2 -l 1       # playback
arecord -d 5 /tmp/test.wav          # capture 5 seconds
aplay /tmp/test.wav                 # play it back

sudo systemctl start messagebox
```
