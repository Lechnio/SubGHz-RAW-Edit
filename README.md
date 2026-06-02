# SubGHz RAW Trim

A tiny on-device waveform editor for **Flipper Zero** that trims RAW `.sub`
captures down to just the part you care about. Open a recording, let the app
find the actual signal hiding inside a long stretch of silence, slide two
markers to the start and end of one clean frame, and save it as a new `.sub`.

Built for cleaning up `Read RAW` captures before decoding or replaying them —
a long recording full of repeats and noise becomes a single tidy frame.

> **Receive/analysis only.** This app never transmits. It only reads, displays
> and rewrites files you already have on the SD card.

---

## Why

`Read RAW` on the Flipper records a continuous stream of pulse/gap durations. A
single button press on a remote often lands in the middle of seconds of
silence, repeated frames, and background noise. Most decoders work best on one
clean copy of the frame. Trimming that by hand means editing the `.sub` text
file on a computer and guessing where the signal starts. This app does it
visually, on the device.

## Features

- **Auto-locates the signal.** On open it scans the whole capture, finds the
  longest clean burst (the most complete frame copy) and jumps the view there
  with the A/B markers already placed around it — no scrolling through empty
  space.
- **Two view modes, switched automatically by zoom:**
  - *Envelope* when zoomed out — bar height shows signal activity, so you can
    see at a glance where the bursts are.
  - *Real square wave* when zoomed in past ~30 ms — inspect individual pulses
    to place cuts precisely.
- **Overview strip** along the top shows the whole recording with a bracket
  marking where your current zoomed view sits.
- **Zoom out past the edges** of the data, so a short burst is shown with empty
  margins on both sides (a clean "silence -> frame -> silence" picture).
- **Memory-safe.** Long silence gaps are clamped to fit a compact `int16`
  buffer, the buffer grows in small chunks and shrinks to the file's real size
  after loading. If there genuinely isn't enough free RAM, you get a clear
  "reboot and try again" message instead of a crash.
- **Clean output.** The saved frame is aligned to start on a pulse and end on a
  gap, with the original `Frequency` and `Preset` preserved.

## Controls

| Button       | Action                                          |
|--------------|-------------------------------------------------|
| Left / Right | Move the active cut marker (hold = move faster) |
| Up / Down    | Zoom in / out (centered on the active marker)   |
| OK (short)   | Switch the active marker between **A** and **B**|
| OK (hold)    | Save the trimmed file                           |
| Back         | Exit                                            |

The active marker is the one drawn as a solid line with a small box on top, and
marked with `>` in the bottom bar. The other marker is dotted.

## Build

Works with the official firmware, Momentum, Unleashed and other forks.

### With your firmware tree (fbt)

```bash
# copy the folder into the firmware sources
cp -r subghz_trim <firmware>/applications_user/subghz_trim

cd <firmware>
./fbt fap_subghz_trim
# result: build/latest/.extapps/subghz_trim.fap
```

Build it from the **same firmware tree** that's flashed on your Flipper,
otherwise you'll get an "Outdated app" / API mismatch error.

Build and run on a connected device in one step:

```bash
./fbt launch APPSRC=applications_user/subghz_trim
```

### With ufbt (standalone, no firmware tree)

```bash
pip install --upgrade ufbt
ufbt update --channel=release        # or point at your fork's SDK
# from the subghz_trim/ folder:
ufbt                                 # builds dist/subghz_trim.fap
ufbt launch                          # builds + uploads + runs
```

### Manual install

Copy the built `subghz_trim.fap` onto the SD card at `/ext/apps/Sub-GHz/` and
launch it from **Apps -> Sub-GHz** on the Flipper.

## Usage

1. Launch **SubGHz RAW Trim** from *Apps -> Sub-GHz*.
2. Pick a RAW `.sub` file from `/ext/subghz`.
3. The view opens centered on the detected frame. Zoom out (Down) to see the
   whole signal, zoom in (Up) to see individual pulses.
4. Place **A** at the start of one clean frame, press OK, place **B** at the
   end.
5. Hold **OK** to save. The trimmed copy is written to
   `/ext/subghz/<name>_trim.sub`.

## File format

Reads and writes the standard Flipper RAW format:

```
Filetype: Flipper SubGhz RAW File
Version: 1
Frequency: 433920000
Preset: FuriHalSubGhzPresetOok270Async
Protocol: RAW
RAW_Data: 257 -926 637 -526 ...
```

`Frequency` and `Preset` are carried over from the source file. Only the
`RAW_Data` durations between the A and B markers are written.

## Notes & limitations

- **`Read RAW` does not record silence.** It starts capturing at the first
  detected pulse and stops shortly after the last one, so the empty seconds
  before/after a button press never reach the file. The empty margins this app
  shows when fully zoomed out are drawn for context — they are not stored data.
- Durations are clamped to +/-32 ms so the capture fits a compact buffer. This
  only ever touches inter-frame silence; the signal pulses themselves
  (hundreds of microseconds) are stored exactly.
- Up to ~24000 samples (about 47 KB) per file; larger captures are truncated
  rather than failing. A real frame is only a few hundred samples, so this is
  plenty in practice.
- Times are 32-bit microseconds, so captures up to ~35 minutes are handled.

