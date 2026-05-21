# :material-piano: MIDI Engine

`src/midi_engine.py` is the entry point for everything between the classifier and the speakers. It owns the gesture-to-chord map, the gesture-to-instrument map, the state machine, and the FluidSynth instance.

```
classifier output --> MidiController --> FluidSynth --> speakers
                      (thread-safe)     (multi-driver)
```

`pyfluidsynth` renders audio directly. No DAW, no virtual MIDI cable, no offline MIDI files. One process, one SoundFont (`soundfonts/GeneralUser_GS.sf2`), one audio device.

---

## Quick start

```python
from midi_engine import MidiController

controller = MidiController()
controller.start()

# Called from your classifier loop, as fast as you like:
controller.on_classification(
    right_hand="palm_up_out",    # chord (8 gestures, see below)
    left_hand="fist_down_out",   # instrument (same 8-gesture vocabulary)
    amplitude=0.73,              # 0.0-1.0 -> MIDI velocity 40-127
)

controller.stop()
```

`on_classification()` is thread-safe and non-blocking. Debouncing, state transitions, note-on / note-off ordering, and audio rendering happen on the engine's own background thread.

---

## Chords

All seven voicings span five to six notes. These are five-finger guitar-style voicings rather than block triads, which is what gives the engine its richer, more sustained sound:

| Chord | MIDI notes | Pitches |
|---|---|---|
| **C** | 48, 52, 55, 60, 64, 67 | C3 E3 G3 C4 E4 G4 |
| **Am** | 45, 52, 57, 60, 64 | A2 E3 A3 C4 E4 |
| **Em** | 40, 47, 52, 55, 59, 64 | E2 B2 E3 G3 B3 E4 |
| **G** | 43, 47, 50, 55, 59, 67 | G2 B2 D3 G3 B3 G4 |
| **Dm** | 38, 50, 53, 57, 62 | D2 D3 F3 A3 D4 |
| **F** | 41, 48, 53, 57, 60, 65 | F2 C3 F3 A3 C4 F4 |
| **D** | 38, 45, 50, 54, 57, 62 | D2 A2 D3 F#3 A3 D4 |

---

## Instruments

Six General MIDI programs are selectable via the left hand:

| Name | GM program | Gesture |
|---|---|---|
| Piano | 0 | `fist_down_out` |
| Nylon Guitar | 24 | `palm_up_out` |
| Steel Guitar | 25 | `palm_down_out` |
| Electric Guitar | 27 | `palm_down_up` |
| Strings | 48 | `fist_down_up` |
| Pad (Warm) | 88 | `peace_out` |

`arm_up` and `arm_down` fall back to Nylon Guitar. Those gestures are not meant for instrument switching.

---

## Chord progression mode

When the controller is locked to a song, gestures advance through the song's chord sequence instead of free-mapping to chords. Useful for the playable-along demo:

```python
controller.set_progression("save_your_tears")               # full song
controller.set_progression("careless_whisper", section="verse")  # just verse
controller.clear_progression()                              # back to free-play
```

Free-play maps gesture-to-chord directly per the [right-hand map](architecture.md#right-hand-chord-selection). Progression mode advances the song's stored chord list one step per triggered gesture. See the [Playlist](playlist.md) page for all available songs and their sections.

---

## Tuning parameters

| Parameter | Default | Effect |
|---|---|---|
| `strum_delay_ms` | 15 | Per-note delay inside a chord (guitar strum). Set to `0` for block chords. |
| `debounce_frames` | 3 | Consecutive same-gesture classifier frames required before triggering. Increase if the classifier is noisy. |
| `gain` | 0.8 | Master output gain (0.0 - 1.0+, clipping above 1.0). |

```python
controller = MidiController(strum_delay_ms=25, debounce_frames=5, gain=1.0)
```

!!! note "Velocity mapping"
    EMG amplitude (0.0 to 1.0) maps linearly to MIDI velocity in the range **40 to 127**. The floor of 40 keeps soft gestures audible; the ceiling of 127 is MIDI's hard limit.

    During the `Sustain` state, velocity updates re-voice the held notes in real time. Squeeze harder mid-chord and the chord swells without re-triggering note-on.

---

## Other methods

| Method | Effect |
|---|---|
| `set_instrument("piano")` | Switch instrument directly, bypassing the left-hand gesture |
| `play_chord("Am", velocity=90, duration=1.0)` | Play a chord; **blocks** the calling thread for `duration` seconds |
| `play_chord("Am", blocking=False)` | Play a chord without blocking; notes auto-stop after `duration` |
| `play_note(60, velocity=100, duration=0.5)` | Play a single MIDI note |
| `play_note(60, blocking=False)` | Same, non-blocking |
| `panic()` | All-notes-off emergency stop. Equivalent to MIDI CC 123. |
| `get_state()` | Returns a dict: current chord, current instrument, progression position, velocity |

---

## Programmatic playback

If you want to script a song without a classifier in the loop, drive the controller directly:

```python
from midi_engine import MidiController

controller = MidiController()
controller.start()

# Load a song's verse and step through it manually:
controller.set_progression("blinding_lights", section="verse")
for _ in range(8):
    controller.advance_progression(velocity=100, duration=1.0)
controller.clear_progression()
controller.stop()
```

This is the path `src/midi_demo.py` uses to exercise FluidSynth without any biosignal hardware. Run `make music` to hear it.
