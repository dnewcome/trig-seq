# Trig Seq — Design Plan

## Core Concept

A MIDI sequencer modeled after a **pair of DJ turntables and a mixer**, but for MIDI clips instead of audio. The DJ metaphor is the organizing principle for every design decision.

In a traditional DAW, clips sit in a grid and you navigate to them. Here, you are the DJ — you stand at the mixer, clips come to you, and you manipulate them in real time.

---

## The Turntable Metaphor

### Fixed Playhead / Scrolling Clip
A turntable needle doesn't move — the record moves under it. This sequencer works the same way: the **playhead is fixed** on screen (left of center), and the **clip scrolls left** under it. When the clip ends, it repositions (loops) seamlessly — like a record on a platter that never stops spinning.

This is a deliberate inversion of the standard DAW timeline. It keeps your attention anchored at the playhead rather than chasing it across the screen.

### Two Decks (A / B)
Two independent playback channels, each capable of holding **multiple layered clips** simultaneously. This maps to the two turntables in a DJ setup. You can:
- Play one deck while preparing the next
- Blend both with the crossfader
- Layer multiple clips on one deck (kick + snare + melody on deck A)

### Clip Layering
Each deck accepts multiple clips at once, all sharing the same phase reference. Their notes are rendered on a **shared piano roll grid** so you can see exactly how they interact — kick on row 36, snare on row 38, melody on rows 60-72, all visible simultaneously within one deck lane.

This is different from scenes: there is no grid of rows and columns to navigate. You throw clips onto a deck live.

### Retrigger / Loop Reset
When you launch a clip onto a running deck, all clips on that deck restart from beat 0 — the deck offset resets. This is analogous to **dropping the needle at the start of a record** or hitting a loop restart button. It's how you keep things in sync when switching patterns mid-performance.

---

## The Mixer Metaphor

The center section between the two decks is a **mixer channel strip**, currently including:

### Crossfader (implemented)
Equal-power crossfade between Deck A and Deck B. Snap keys (`A`, `B`) cut hard to one side; nudge keys (`-` / `=`) allow smooth blending. Velocity-scales all MIDI notes from each deck accordingly.

### Planned Mixer Features

#### Headphone Cue (Pre-listen)
In DJ setups, the headphone CUE button routes a channel to the monitor output (headphones) independently of the master mix. For MIDI, this could mean:
- A separate MIDI output port (or channel) designated as the "cue bus"
- Pressing a cue key on either deck sends its MIDI to the cue port without affecting the master output
- Lets you prepare and test a clip in your headphones before bringing it into the mix
- Could also work as a MIDI monitor: cue output goes to a software synth on your headphones while the master goes to hardware

#### 3-Band EQ Per Channel
DJ mixers have High / Mid / Low knobs per channel. For MIDI this translates naturally to **pitch range filtering**:
- **High band** — notes above a threshold (e.g., pitch > 60): melodies, leads
- **Mid band** — notes in the middle range (e.g., pitch 36–60): basslines, pads
- **Low band** — notes below a threshold (e.g., pitch < 36): kick, sub bass

Cutting a band silences notes in that pitch range for that deck. This lets you do EQ-style mixing: cut the bass from deck B while bringing in deck A's bass, just like a DJ EQ-blends two tracks.

Velocity scaling per band is another option (attenuate rather than hard mute), which would feel smoother.

#### Filter Per Channel
A single sweepable filter (cutoff pitch, resonance) that silences notes above or below a threshold. The classic DJ build-up move: sweep the filter up before the drop.

#### Channel Faders
Independent volume control per deck (separate from the crossfader). Maps to channel volume on a mixer. Would affect MIDI velocity scaling per deck.

---

## The Piano Roll Display

Each deck shows a **shared chromatic piano roll** for all its layered clips:
- Notes are positioned by absolute MIDI pitch — the same pitch always occupies the same row
- The Y axis range auto-fits to the combined pitch content of all active clips on the deck
- Black key rows are shaded darker (standard piano roll convention)
- C notes get brighter divider lines and octave labels
- A narrow piano key indicator column sits on the left edge

This lets you read a drum loop (kick=36, snare=38, hh=42) exactly like a melody — both are just notes on a shared grid. When you layer them, you instantly see how they relate.

---

## VIM-Style Keyboard Control

The interface is intentionally **mouseless**. Everything is keyboard-driven, with modal operation inspired by Vim:

- **PERFORM mode** — live launching, crossfader, transport. Default mode.
- **EDIT mode** — step-entering notes into a clip using hjkl navigation.
- **RECORD mode** — (planned) real-time MIDI input captured to the active clip.
- **COMMAND mode** — colon-prefixed commands for clip management (`:bpm 140`, `:new kick 32`).

The reasoning: during live performance, a clip is always scrolling. You can't reliably click a moving target. Keyboard shortcuts with muscle memory are faster and more reliable.

---

## Keyboard Map (Current)

| Key | Action |
|-----|--------|
| `Space` | Play / Stop |
| `R` | Toggle record |
| `Tab` | Switch active deck |
| `1`–`8` | Toggle clip on active deck (add/remove) |
| `Shift+1`–`8` | Toggle clip on Deck B |
| `Q W E R T Y U I` | Retrigger clip 1–8 → active deck |
| `A` | Crossfader snap → Deck A |
| `B` | Crossfader snap → Deck B |
| `F` | Crossfader center |
| `-` / `=` | Nudge crossfader |
| `E` | Enter Edit mode |
| `Esc` | Back to Perform mode |
| `N` | New empty clip |
| `,` / `.` | BPM −5 / +5 |
| `?` | Help overlay |
| `:` | Command mode |

**Edit mode:**

| Key | Action |
|-----|--------|
| `H` / `L` | Cursor left / right |
| `J` / `K` | Cursor pitch down / up |
| `Shift+J/K` | Cursor pitch by octave |
| `A` | Add note at cursor |
| `X` | Delete note at cursor |
| `[` / `]` | Halve / double quantize |

---

## Open Questions / Future Directions

- **Sync between decks** — should deck B auto-quantize its retrigger to the nearest beat of deck A? (like beatmatching)
- **Slip mode** — underlying beat position keeps advancing even while you hold a loop, so releasing it snaps back into the global timeline
- **Cue points** — mark specific beat positions within a clip to jump to (like CDJ cue points)
- **Hot cues** — instant-retrigger to a cue point via a single key
- **Loop in/out** — define a sub-loop within a clip and spin it down to a smaller window
- **FX sends** — route deck output to a send channel (reverb, delay) before it hits the MIDI output
- **Multiple MIDI outputs** — deck A → port 1, deck B → port 2, cue → port 3
- **MIDI clock output** — send clock to sync external gear
- **Visual feedback for cue** — second deck preview pane (small, dimmed) showing what's cued up

---

## Implementation Notes

- **Single HTML file** — no build step, no dependencies. Open in browser and play.
- **Web MIDI API** — native MIDI output, falls back to Web Audio oscillators for demo use.
- **Web Audio** — fallback synth only; not the focus. A real performance uses hardware MIDI instruments.
- **Canvas 2D rendering** — 60fps requestAnimationFrame loop drives both audio scheduling and visuals.
- **Lookahead scheduling** — notes are scheduled 0.25 beats ahead of the playback head to avoid timing jitter from frame rate variation.
