# Trig Seq

A MIDI clip sequencer modeled after a pair of DJ turntables and a mixer. No DAW, no scene grid, no mouse required — just clips, two decks, and a crossfader.

Open `index.html` in a browser. No build step, no dependencies.

---

## What is this?

Most MIDI sequencers ask you to arrange clips on a timeline and press play. Trig Seq works the other way around: **you stand at the mixer and the music comes to you.**

The playhead is fixed — a vertical line left of center, like a turntable needle. Clips scroll left under it and loop continuously, like records spinning on a platter. You launch clips onto one of two decks, layer them, blend between the decks with a crossfader, and switch patterns on the fly. There is no arrangement view, no scene grid, no timeline to scroll.

The inspiration is a classic two-turntable DJ setup, but for MIDI. Instead of mixing audio from vinyl, you're mixing MIDI note sequences and sending them to your synthesizers, drum machines, or DAW.

---

## Inspiration

### The Turntable as Sequencer

A turntable has no timeline. The record loops. The needle doesn't chase the groove — the groove comes to the needle. Trig Seq borrows this directly: the scrolling piano roll is the record, the playhead is the needle, and looping means resetting the record's position rather than chasing a moving cursor across a screen.

This makes the display feel alive during performance. Something is always moving. You read the upcoming notes the same way a DJ reads a waveform — you can see what's coming before it hits.

### The DJ Mixer

In a two-turntable setup, the mixer sits between the decks and gives you:
- A crossfader to blend between sources
- EQ per channel to shape each source before it hits the mix
- A cue/headphone bus to pre-listen before the audience hears it

Trig Seq is building this structure for MIDI. The crossfader is already there. EQ as pitch-range filtering (cut the "low" band to mute kick and bass notes from one deck) and headphone cue routing are on the roadmap.

### Why No Scene Grid?

Scene grids in live performance tools (Ableton Live's session view, etc.) are powerful but they impose a structure: rows of clips that must fire together, columns that define "what plays at the same time." This encourages preparation over improvisation.

Trig Seq has no rows, no columns, no scenes. Any clip can go on any deck at any time. You layer clips freely within a deck and remove them just as easily. The structure emerges from what you play, not from a grid you designed in advance.

### Why Keyboard Only?

During a live performance, clips are always moving. There is no stable position to click on. Keyboard shortcuts with muscle memory are faster, more reliable, and let you keep your eyes on the output rather than the screen.

The keyboard layout is loosely inspired by Vim: modal operation with a default performance mode, an edit mode for entering notes, and a command mode for configuration. The home row keys (`Q W E R T Y U I`) are clip launchers — you can trigger any of the first eight clips with one hand without looking.

---

## Concepts

### Decks

There are two decks, A and B. Each deck is an independent playback channel. You control which deck is "active" with `Tab` — the active deck receives clip launches from the number keys.

Both decks run on the same clock at the same BPM. When you retrigger a deck, all its clips reset to beat 0 together — they stay in phase with each other.

### Clips

A clip is a looping sequence of MIDI notes with a defined length in beats. Clips live in the clip library (the strip at the bottom of the screen, slots 1–8). They don't belong to a deck — they're available to either deck at any time.

### Layering

Each deck can hold multiple clips simultaneously. Press `1`–`8` to toggle a clip on or off the active deck. All clips on a deck share the same loop phase, so they play in sync. The piano roll shows all of them on the same pitch grid so you can see how they interact.

A typical use: slot 1 is kick, slot 2 is snare, slot 3 is hi-hat, slot 4 is bass. Layer 1+2+3 on deck A for the full drum kit. Layer just 1+3 on deck B for a sparse version. Crossfade between them.

### Retrigger

The `Q W E R T Y U I` keys are retrigger keys for clips 1–8. Pressing one adds that clip to the active deck and resets the deck's loop position to beat 0. This is the equivalent of dropping the needle back at the start of a record. Use it to punch in a new pattern in sync, or to restart a loop for emphasis.

### Crossfader

The crossfader blends the MIDI output between Deck A and Deck B using an equal-power curve. At the center position both decks play at ~70% velocity. Snap hard to one side with `A` or `B`. Nudge with `-` / `=`.

The crossfader works by scaling MIDI note velocity — it doesn't affect timing. A note from a fully-faded-out deck is simply not sent.

### Piano Roll Display

Each deck displays its clips as a **shared chromatic piano roll**. Every MIDI pitch maps to the same fixed row regardless of which clip it comes from. This means:

- Kick (MIDI 36) always appears on the same row as other clips' note 36
- Snare (MIDI 38) is always two rows above kick
- A melody spanning C4–G5 occupies the same rows whether it's on deck A or deck B

The Y axis auto-fits to the combined pitch range of all active clips on the deck. Black key rows are shaded darker. C notes get brighter divider lines and octave labels. A piano key indicator runs along the left edge.

---

## Getting Started

1. Open `index.html` in Chrome or Firefox (Web MIDI API required for MIDI output; Chrome is recommended)
2. If you have a MIDI interface connected, the browser will ask for permission — allow it
3. Press `Space` to start playback
4. The demo loads five clips: kick, snare, hi-hat, bass, and melody
5. Press `1` to toggle the kick onto Deck A, `2` for snare, `3` for hi-hat
6. Press `Tab` to switch to Deck B, then press `4` and `5` to load bass and melody
7. Use `A` and `B` to snap the crossfader, or `-` / `=` to blend

To edit a clip: press `E` to enter edit mode, use `H`/`L` to move the beat cursor and `J`/`K` to change pitch, then `A` to place a note. Press `Esc` to return to perform mode.

---

## Key Reference

### Perform Mode (default)

| Key | Action |
|-----|--------|
| `Space` | Play / Stop |
| `Tab` | Switch active deck (A ↔ B) |
| `R` | Toggle record mode |
| `1`–`8` | Toggle clip on active deck |
| `Shift+1`–`8` | Toggle clip on Deck B |
| `Q W E R T Y U I` | Retrigger clip 1–8 on active deck |
| `A` | Crossfader → full Deck A |
| `B` | Crossfader → full Deck B |
| `F` | Crossfader center |
| `-` / `=` | Nudge crossfader left / right |
| `,` / `.` | BPM −5 / +5 |
| `E` | Enter Edit mode |
| `N` | New empty clip |
| `?` | Key binding help |
| `:` | Enter Command mode |

### Edit Mode (`E` to enter, `Esc` to exit)

| Key | Action |
|-----|--------|
| `H` / `L` | Move cursor left / right by quantize step |
| `J` / `K` | Move cursor pitch down / up |
| `Shift+J/K` | Move cursor pitch by octave |
| `A` | Add note at cursor (cursor advances) |
| `X` | Delete note at cursor |
| `[` / `]` | Halve / double quantize step |

### Command Mode (`:` to enter, `Enter` to execute, `Esc` to cancel)

| Command | Action |
|---------|--------|
| `:bpm 140` | Set BPM |
| `:new kick 32` | New clip named "kick", 32 beats long |
| `:len 2 32` | Set clip 2 length to 32 beats |
| `:clear 3` | Delete all notes from clip 3 |
| `:rename 2 bass` | Rename clip 2 |

---

## MIDI Setup

Trig Seq uses the **Web MIDI API**. On first load, your browser will prompt for MIDI device access — allow it. The first available MIDI output is used automatically.

If no MIDI output is found, the app falls back to Web Audio oscillators for demo purposes. These are basic waveforms, not a performance instrument.

**Recommended setup:**
- Chrome (best Web MIDI support)
- A USB MIDI interface or USB-enabled synthesizer/drum machine
- Route Trig Seq's MIDI output to your instrument(s) in your OS MIDI routing or DAW

---

## Roadmap

- **Headphone cue** — route a deck to a separate MIDI port for pre-listen before bringing it into the mix
- **3-band EQ per deck** — filter note output by pitch range (low/mid/high bands), enabling EQ-style mixing of MIDI note content
- **Channel faders** — independent per-deck volume separate from the crossfader
- **Sweep filter** — single-knob pitch cutoff filter per deck for build-up/drop gestures
- **Quantized retrigger** — option to snap retrigger to the next beat boundary rather than immediately
- **MIDI clock output** — sync external gear to Trig Seq's BPM
- **Multiple MIDI outputs** — assign deck A and deck B to different ports or channels
- **Real-time record** — play notes into a clip from an external MIDI keyboard while the deck runs

---

## Technical Notes

Trig Seq is a single HTML file with no external dependencies. All rendering is done on an HTML5 Canvas at 60fps. MIDI note scheduling uses a 0.25-beat lookahead buffer to smooth over frame rate jitter.

The project is currently in demo/prototype stage. See `PLAN.md` for full design rationale and future direction.
