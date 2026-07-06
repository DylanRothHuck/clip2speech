# clip2speech

Read your clipboard aloud with **Piper TTS** — fast, local, offline. Press a key to
speak whatever you copied. Press it again to stop.

Designed for **Arch Linux + Hyprland (Wayland)**, but works on any Linux with
`wl-clipboard`, `piper-tts`, and `aplay`.

## Demo

```
Copy text  →  press SUPER+U  →  hear it spoken
             press again     →  stops immediately
```

## Dependencies

| Package | Purpose |
|---------|---------|
| [piper-tts](https://github.com/rhasspy/piper) | Neural TTS engine |
| `alsa-utils` | Provides `aplay` for audio playback |
| `wl-clipboard` | Provides `wl-paste` for Wayland clipboard |

## Installation

### 1. Install dependencies

```bash
# Arch — from AUR (recommended)
yay -S piper-tts alsa-utils wl-clipboard

# Arch — manual binary
# Download piper from https://github.com/rhasspy/piper/releases
# and place it somewhere in your PATH, e.g. ~/.local/bin/
```

### 2. Download voice models

Piper needs an `.onnx` model file plus its `.json` config. Place them in:

```
~/.local/share/piper-voices/
```

Download from the [Piper voice repository](https://huggingface.co/rhasspy/piper-voices/tree/main):

```bash
mkdir -p ~/.local/share/piper-voices

# English — high quality
wget -P ~/.local/share/piper-voices \
  https://huggingface.co/rhasspy/piper-voices/resolve/main/en/en_US/lessac/high/en_US-lessac-high.onnx \
  https://huggingface.co/rhasspy/piper-voices/resolve/main/en/en_US/lessac/high/en_US-lessac-high.onnx.json

# Spanish — high quality (default voice)
wget -P ~/.local/share/piper-voices \
  https://huggingface.co/rhasspy/piper-voices/resolve/main/es/es_AR/daniela/high/es_AR-daniela-high.onnx \
  https://huggingface.co/rhasspy/piper-voices/resolve/main/es/es_AR/daniela/high/es_AR-daniela-high.onnx.json
```

### 3. Install the scripts

```bash
cp clip2speech clip2speech-en ~/.local/bin/
chmod +x ~/.local/bin/clip2speech ~/.local/bin/clip2speech-en
```

Make sure `~/.local/bin` is in your `PATH`.

## Usage

```bash
clip2speech           # reads clipboard (regulr copy)
clip2speech selection # reads primary selection (highlighted text)
clip2speech text "Hello world"  # speaks the given text directly
clip2speech-en        # same but with English voice
```

Run again while speaking to **stop** the current utterance (toggle behavior).

## Configuration

Optional config file at `~/.config/clip2speech/config`:

```bash
# Voice model directory
PIPER_VOICE_DIR="$HOME/.local/share/piper-voices"

# Default voice filename
DEFAULT_VOICE="es_AR-daniela-high.onnx"

# Speed (lower = faster, default 0.6)
LENGTH_SCALE=0.6

# Silence between sentences in seconds (default 0.2)
SENTENCE_SILENCE=0.2

# Piper binary path (auto-detected from PATH if not set)
PIPER_BIN="/usr/bin/piper"
```

You can also set these as environment variables.

## Hyprland keybindings

Add to your `~/.config/hypr/hyprland.conf`:

```conf
# Read clipboard aloud
bindd = SUPER, U, Speak clipboard, exec, ~/.local/bin/clip2speech
bindd = SUPER ALT, U, Speak clipboard (EN), exec, ~/.local/bin/clip2speech-en
# Highlight text and speak
bindd = SUPER SHIFT, U, Speak selection, exec, ~/.local/bin/clip2speech selection
```

With `bindd` the keybinding label shows up in your keybind layer overlay.

## Mako notification

Create `~/.config/hypr/scripts/clip2speech-notify`:

```bash
#!/usr/bin/env bash
~/.local/bin/clip2speech "$@"
if [[ $? -eq 0 ]]; then
    notify-send -t 1500 "clip2speech" "Speaking clipboard"
fi
```

And bind to that script instead.

For a stop notification, wrap the toggle:

```bash
#!/usr/bin/env bash
if pidof piper aplay &>/dev/null; then
    pkill piper aplay 2>/dev/null
    notify-send -t 1500 "clip2speech" "Stopped"
else
    ~/.local/bin/clip2speech "$@"
    notify-send -t 1500 "clip2speech" "Speaking clipboard"
fi
```

## Waybar module

Add a custom module to `~/.config/waybar/config.jsonc`:

```jsonc
"custom/clip2speech": {
    "format": "🔊 {}",
    "interval": "once",
    "exec": "if pidof piper >/dev/null; then echo 'SPK'; else echo ''; fi",
    "exec-if": "pidof piper || pidof aplay",
    "on-click": "~/.local/bin/clip2speech",
    "tooltip": "Click to speak/stop clipboard"
}
```

And style in `~/.config/waybar/style.css`:

```css
#custom-clip2speech {
    color: #a6e3a1;
    min-width: 30px;
}
```

## Toggle behavior

- Run `clip2speech` once → starts speaking
- Run it again (even with different text) → kills the current utterance
- This makes it safe to keybind — no accidental overlapping speech

## Voices

Piper offers [dozens of voices](https://huggingface.co/rhasspy/piper-voices/tree/main) in
multiple languages and qualities (low, medium, high). The filename format is:

```
<lang>_<COUNTRY>-<voice>-<quality>.onnx
```

Set `DEFAULT_VOICE` in your config to switch.

## License

MIT
