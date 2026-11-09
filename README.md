# Estivi

**SSTV, on a schedule.**

Most SSTV software sends one picture when you tell it to. Estivi runs a channel. You lay out a week — programs that repeat, one-off blocks, the pictures and bumpers inside them — and the transmitter keys the radio and plays it, callsign and all. On the other side, a browser receiver turns the radio's audio back into pictures and a calendar, so a listener can follow the schedule as it airs.

Slow-scan television (SSTV) sends still images over a narrowband voice channel, one line at a time. Estivi treats them like programming.

<p align="center">
  <img src="https://raw.githubusercontent.com/fluoroom/estivi-tx/main/docs/screenshots/main.png" alt="Estivi transmitter: the weekly grid, the next transmission, and the selected block" width="920">
</p>

<p align="center"><em>The transmitter: the weekly grid, the next transmission, and the selected block.</em></p>

## The three repositories

| Repository | Role | Stack |
|---|---|---|
| **[estivi-tx](https://github.com/fluoroom/estivi-tx)** | The station. A desktop app that holds the channel, draws the grid, renders every SSTV mode, keys the radio, answers CW and DTMF, and logs each transmission. | Rust, Slint |
| **[estivi-rx](https://github.com/fluoroom/estivi-rx)** | The listener. One web page that demodulates radio audio in the browser, decodes the pictures and the AX.25 guide, and keeps a timeline, a calendar, and saved PNGs. | JavaScript, AudioWorklet, nginx |
| **[estivi-webpage](https://github.com/fluoroom/estivi-webpage)** | The front door. The project's public landing page, just getting started. | — |

This repository is the umbrella. It pins the three projects as submodules at known commits and holds no code of its own; each submodule keeps its own history, releases, and documentation.

## How it fits together

```
 .estivi channel ──▶ estivi-tx ──▶ radio ═══ air ═══▶ radio ──▶ estivi-rx
   your schedule      audio + PTT     SSTV + AX.25     microphone  pictures & guide
```

There is no private link between the two ends. Everything the receiver knows arrives as sound: SSTV pictures with their VIS headers, and 1200-baud AX.25 UI frames addressed to `ESTIVI` that carry the station's guide and name each piece. A listener who misses the guide still gets the pictures; the calendar fills in as the frames arrive.

## Try it

### Transmit

Download the [latest release](https://github.com/fluoroom/estivi-tx/releases/latest) — Linux x86_64, Windows x86_64/i686, or macOS universal — unpack it, and run `estivi`. Open [`ejemplo/canal-demo.estivi`](https://github.com/fluoroom/estivi-tx/blob/main/ejemplo/canal-demo.estivi) to see a full channel: two programs, stored blocks, activations, a lower third, a QR, and a hidden weather layer. Every picture it uses is inside `ejemplo/`, so the sample runs anywhere.

Set your callsign, audio output, and PTT device, then **Start grid**. No radio at hand? Export any range to FLAC instead.

### Receive

The public receiver is at **[estivi-rx.lu1aqy.radio](https://estivi-rx.lu1aqy.radio)**. Open it, allow the microphone, and point it at your receiver's speaker, or feed it the receiver's line audio. Decoding happens in the page; the server only serves static files.

To run your own copy:

```sh
git clone --recurse-submodules https://github.com/fluoroom/estivi.git
cd estivi/estivi-rx
docker compose up --build
# open http://localhost:8086
```

Browsers only expose the microphone over HTTPS or on localhost, so put a TLS terminator in front when you deploy it.

If you cloned without `--recurse-submodules`, fill the submodules in with `git submodule update --init --recursive`.

## What the transmitter does

- **A week, not a queue.** Each weekday gets its own start, end, and interval. Monday can run fifteen minutes; Saturday can run for hours. Set it once and leave it.
- **Blocks for everything else.** One date, one card: a special event, a net, a contest. Move it, skip it for a day, or delete it.
- **Pictures with a cast.** Welcome and bye go out on the first and last turn; pre- and post-bumpers travel with their picture. A picture inherits its mode or overrides it.
- **It answers the radio.** CW and DTMF triggers run a picture, a command, an API reply, or a sequence of actions.
- **Graphics without another tool.** Text, image, and QR layers per channel, program, or block, standing on the next live picture or on a checkerboard, color bars, a grid, or a gray ramp.
- **Live and next.** Watch the picture fill line by line while it goes out, and see what is coming with its clock and file name.
- **One executable.** Software renderer, seven themes, English or Spanish, and a CSV log of every transmission.

## What the receiver does

- **Decode in the page.** Radio audio goes through a gain stage into an AudioWorklet; AX.25 and SSTV run entirely in the browser, with no decoder on the server.
- **Read the guide.** The AX.25 frames name the station and the schedule, so the receiver can build a 30-day calendar and an hour row from them.
- **Hold what it hears.** The visor shows lines as they scan in; finished pictures go to a timeline, can be saved as PNG, and survive a reload in IndexedDB — up to 40 of them.
- **Keep working.** Installable as an app on Chromium browsers, add-to-home-screen on iPhone and iPad, and a service worker that serves a cached copy after the first visit.

## On the air

- **SSTV.** 22 modes — Martin M1–M4, Scottie S1/S2/DX, Robot 24/36/72, Robot 12 color, Robot B/W 8/12/24, PD 50, 90, 120, 160, 180, 240, and 290, plus YoniQ. Every piece is preceded by its VIS.
- **AX.25.** Bell 202 at 1200 baud, UI frames addressed to `ESTIVI`. The guide carries `FREQ`, `CALL`, `NAME`, `RANGE`, `DAYS`, `DAYSLEFT`, `NEXT`, and `COMMENT`; mode frames announce each piece, as in `ESTIVI MODE R36 BUMPER PRE`.
- **Identification.** Optional CW before and after the pictures, with the same callsign. PTT is serial or CAT through `rigctld`.
- **Audio.** Synthesized at 11025, 22050, 44100, or 48000 Hz, and exported as FLAC.

## Development

### The submodules

Each submodule is a normal repository. To move the pins to the current `main` of all three:

```sh
git submodule update --remote --merge
git add estivi-tx estivi-rx estivi-webpage
git commit -m "Update submodules"
```

Work happens in the submodule that owns it; a change in the umbrella is only a new pin.

### Transmitter

Stable Rust 1.85 or newer. Linux needs ALSA, fontconfig, and GTK 3; macOS needs the Xcode command line tools; Windows needs the MSVC toolchain.

```sh
cd estivi-tx
cargo build --release   # target/release/estivi
cargo test
```

The [manual](https://github.com/fluoroom/estivi-tx/blob/main/docs/MANUAL.md) covers the order of a turn, every window behavior, the file formats, import and export, and what has been verified on the air.

### Receiver

There is nothing to build: `www/` is the page, and the compose file serves it with nginx. The decoder tests run on Node's built-in test runner:

```sh
cd estivi-rx
npm test
```

## Documentation

- [Transmitter README](https://github.com/fluoroom/estivi-tx#readme) — features, screenshots, releases, and build notes.
- [Transmitter manual](https://github.com/fluoroom/estivi-tx/blob/main/docs/MANUAL.md) — the long version.
- [AX.25 interface](https://github.com/fluoroom/estivi-tx/blob/main/AX25.md) — the exact bytes and text a client reads.
- [Receiver README](https://github.com/fluoroom/estivi-rx#readme) — the page tour, the decoder, and storage.
