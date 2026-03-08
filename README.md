# ZeroPlay
A lightweight H.264/HEVC video player for the Raspberry Pi, built as a modern replacement for the discontinued __omxplayer__. Uses the V4L2 M2M hardware decoder, DRM/KMS display, and ALSA audio — zero CPU video decode, zero X11 dependency.

```
have a nice day ;)
```

## Supported Hardware
| Device | OS |
|---|---|
| Pi Zero 2 W | Raspberry Pi OS Lite 64-bit (Trixie) |
| Pi 4 | Raspberry Pi OS Lite 64-bit (Trixie) |

## Supported Formats
| Codec | Container |
|---|---|
| H.264 (up to High@L4.1) | MP4, MKV, MOV |
| VP8 | MKV, WebM |
| VP9 | MKV, WebM |

H.264 is hardware decoded via the bcm2835 VPU. VP8 and VP9 hardware support depends on firmware version — if unsupported, ZeroPlay will report a clear error.

## Installation

```
curl -fsSL https://raw.githubusercontent.com/HorseyofCoursey/zeroplay/main/install.sh | sudo bash
```

This will install dependencies, build from source, and place the binary at `/usr/local/bin/zeroplay`.

### Manual build

If you'd prefer to build yourself:

```
sudo apt install git gcc make \
  libavformat-dev libavcodec-dev libavutil-dev libswresample-dev \
  libdrm-dev libasound2-dev

git clone https://github.com/HorseyofCoursey/zeroplay.git
cd zeroplay
make
sudo make install
```

## Usage

```
zeroplay [options] <file>
```

### Options
| Flag | Description |
|---|---|
| `--loop` | Loop playback indefinitely |
| `--no-audio` | Disable audio |
| `--vol n` | Initial volume, 0–200 (default: 100) |
| `--pos n` | Start position in seconds |
| `--audio-device dev` | ALSA device override |
| `--verbose` | Print decoder and driver info on startup |
| `--help` | Show usage |

### Examples

```
# Play a file
zeroplay movie.mp4

# Loop
zeroplay --loop movie.mp4

# Start at 1h 30min
zeroplay --pos 5400 movie.mp4

# Start at 80% volume
zeroplay --vol 80 movie.mp4

# No audio
zeroplay --no-audio movie.mp4

# Override ALSA output device
zeroplay --audio-device plughw:CARD=Headphones,DEV=0 movie.mp4

# Show decoder and driver details on startup
zeroplay --verbose movie.mp4
```

### Controls
| Key | Action |
|---|---|
| `p` / `Space` | Pause / resume |
| `←` / `→` | Seek −/+ 1 minute |
| `↑` / `↓` | Seek −/+ 5 minutes |
| `+` / `=` | Volume up 10% |
| `-` | Volume down 10% |
| `m` | Mute / unmute |
| `i` | Previous chapter |
| `o` | Next chapter |
| `q` / `Esc` | Quit |

## Audio Device
ZeroPlay defaults to HDMI audio (`plughw:CARD=vc4hdmi,DEV=0`). To list available devices:

```
aplay -L
```

Then pass the device name with `--audio-device`.

## How It Works
* **Demux** — libavformat reads the container and routes packets
* **Video decode** — V4L2 M2M hardware decoder (`/dev/video10`) via bcm2835-codec
* **Display** — DRM/KMS atomic modesetting with DMABUF zero-copy from decoder to scanout
* **Audio** — libavcodec software decode → libswresample → ALSA
* **Sync** — Wall-clock pacing against video PTS, audio runs independently

No X11, no Wayland, no GPU compositing. Runs directly on the framebuffer from a TTY or SSH session.

## Differences from omxplayer
| Feature | omxplayer | ZeroPlay |
|---|---|---|
| Hardware decode | OpenMAX (deprecated) | V4L2 M2M |
| Display | dispmanx (deprecated) | DRM/KMS |
| Subtitles | Yes | No |
| Chapter skip | No | Yes |
| Seeking | Yes | Yes |
| Volume control | Yes | Yes |
| Loop | Yes | Yes |
| Runs on modern OS | No | Yes |
