# Runbook

Operating procedures for a box that lives in a child's room and has to recover
on its own.

## Everyday commands

```bash
ssh <user>@messagebox.local            # connect to the box
sudo systemctl status messagebox       # is it running?
sudo systemctl restart messagebox      # restart the application
sudo journalctl -u messagebox -f       # live logs (Ctrl+C to exit)
sudo systemctl stop messagebox         # stop before any manual testing
```

Always stop the service before manual audio or GPIO tests, or you will get
`EADDRINUSE: address already in use :::8080` and `lgpio.error: 'GPIO busy'`.

## Shutdown discipline

**Never cut power to the box directly.**

```bash
sudo poweroff
```

Wait for the green activity LED to stop blinking before unplugging. Pulling
power while writes are in flight has already cost this box its audio config
twice and can corrupt the microSD outright.

## Failure: the box beeps continuously and does nothing

**Symptom** — a beep every few seconds, indefinitely; the box neither sends nor
plays messages.

**What it actually is** — the beep is the application's *startup* tone. WhatsApp
has invalidated the linked-device session, the app fails on launch, systemd
restarts it five seconds later, and every restart replays the startup tone. The
"beeping" is an audible crash loop.

**Diagnose:**

```bash
sudo journalctl -u messagebox -n 40 --no-pager
```

Confirmed by a logged-out warning from the app together with repeated
`Scheduled restart job, restart counter is at N` lines.

**Recover** (procedure verified 2026-09-01):

```bash
sudo systemctl stop messagebox                       # 1. break the restart loop
cd ~/messagebox
cp -r auth ~/auth-backup-$(date +%Y%m%d)             # 2. back up the session — do not skip
rm -rf auth                                          # 3. drop the dead session
node listar-chats.js                                 # 4. scan the QR with the dedicated line
                                                     #    confirm the group is unchanged, Ctrl+C
sudo systemctl reset-failed messagebox               # 5. clear the failure counter
sudo systemctl start messagebox
sudo journalctl -u messagebox -n 15 --no-pager       # wait for the connected confirmation
```

Before re-linking, check the phone: WhatsApp → Settings → Linked devices. If the
box is simply gone from the list, re-linking is enough. If the **account** is
blocked, re-linking will not help — that is the known Baileys risk, and it means
evaluating a different line.

## Failure: no audio

Check in this order.

1. **The config file** — `cat /etc/asound.conf`. Empty or missing is the most
   common cause; recreate it (see [`audio-alsa.md`](audio-alsa.md)).
2. **Card names** — `aplay -l` and `arecord -l`. If a card's name has changed,
   the config needs updating. Never fall back to card numbers.
3. **Volume** — `amixer -c Headphones sset PCM 100%` then `sudo alsactl store`.
4. **Ignore the mixer warning in the logs.** The recurring ALSA volume warning
   is a hardcoded card name in the application and is never the cause.

## Failure: box hangs or reboots on its own

This board has 1 GB of RAM against the kit's specified 2 GB, so memory is the
first suspect.

```bash
free -h
sudo journalctl -u messagebox -n 100 --no-pager
vcgencmd get_throttled     # 0x0 = no undervoltage events
```

## Before any update

Back up privately — never into a repository:

- `auth/` — WhatsApp session
- `config.json` — configuration and secrets
- `mensajes/` — mailbox contents
- `sonidos/anuncios/` — custom announcement audio

## Never publish

Session data (`auth/`, `auth-*`), `config.json`, mailbox audio, family
recordings, the interface PIN, the authorized group identifier, and WiFi
credentials. `.gitignore` catches the known cases; a human check before pushing
is still required.
