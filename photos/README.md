# Photos

The files here are placeholders. Replace each one with a real photo at the same
filename and the README and docs will pick it up with no edits.

| File | Shot | Used in |
|---|---|---|
| `01-finished-box.jpg` | Hero image — the finished box, buttons and LED visible | `README.md` |
| `02-in-use.jpg` | Your son pressing a button or listening | `README.md` |
| `03-interior-wiring.jpg` | Interior — Pi, wiring, buttons and LED mounted | `README.md` |
| `04-components.jpg` | Components laid out before assembly — board, USB WiFi, USB mic, speaker | `docs/hardware-port.md` |
| `05-gpio-header.jpg` | Close-up of the GPIO header with jumpers, pins countable | `docs/wiring.md` |
| `06-buttons-mounted.jpg` | The two buttons and LED mounted through the enclosure face | `docs/wiring.md` |

Worth having, if you took them: a shot mid-drilling or mid-assembly, and the
microSD imaging step.

## Before committing photos

- No faces you don't want public, and no phone screens showing the group chat,
  contact names, or numbers.
- No shots of a terminal with the group identifier, the interface PIN, or WiFi
  credentials on screen.
- Photos carry EXIF metadata including GPS coordinates. Strip it:

  ```bash
  # macOS, with exiftool (brew install exiftool)
  exiftool -all= -overwrite_original photos/*.jpg
  ```

- Resize anything enormous so the repo stays light:

  ```bash
  # macOS built-in
  sips -Z 1600 photos/*.jpg
  ```
