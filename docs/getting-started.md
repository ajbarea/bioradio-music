# :material-rocket-launch: Getting Started

## Prerequisites of the Ritual

| Vessel | What you need |
|---|---|
| **Python 3.10+** | Required for `pylsl`, `pyqt6`, `neurokit2` |
| **Conda** | Recommended (pulls FluidSynth cleanly as a system dep) |
| **GLNeuroTech BioRadio** | Optional for development; mock mode summons the same data shape without hardware |
| **Speakers / headphones** | FluidSynth renders directly via WASAPI / DirectSound / WaveOut on Windows; equivalent OS drivers on macOS / Linux |

---

## 1. Set up the environment

```bash
make setup
conda activate hackathon
```

This installs Python, all scientific deps, FluidSynth (the C library), and pyfluidsynth (Python bindings) in one shot.

## 2. Verify audio works

```bash
make music
```

This runs the `src/midi_demo.py` script, which cycles through instruments and chords to ensure FluidSynth is correctly configured and communicating with your speakers.

If you hear nothing, check that your system volume is on and the correct audio driver (WASAPI is preferred) is available in Windows Settings.

## 3. Launch the GUI

```bash
# Without hardware (mock data for development):
make gui

# With BioRadio connected:
make gui-live
```

## 4. Train the classifier

The ML pipeline lives in `src/pipeline.py`. Victor's `main()` trains on the team's 8 gesture classes from recorded EMG data:

```bash
make train
```

This produces a saved model in `models/classifier.pkl`.

## 5. Perform the Ritual

Once your classifier is trained and the GUI is streaming to LSL, start the real-time bridge:

```bash
make ritual
```

This will:

1. Initialize the MIDI engine (FluidSynth + SoundFont).
2. Load `models/classifier.pkl` (falls back to a mock classifier if not found).
3. Search for the `BioRadio` LSL stream (retries up to 8 times, 1 second each).
4. Process EMG signals in 250ms windows with 50% overlap.
5. Classify gestures and trigger the MIDI engine.

You can also start the ritual from the GUI by enabling the **Music Mode** checkbox during acquisition. The GUI shows real-time status updates as the ritual connects.

## 6. Manual Integration (Alternative)

If you're building a custom script, connect MIDI to your own classifier loop:

```python
from midi_engine import MidiController

controller = MidiController()
controller.start()

# Called every classification frame:
controller.on_classification(
    right_hand="palm_up_out",   # chord selection
    left_hand="fist_down_out",  # instrument selection
    amplitude=0.73              # EMG amplitude -> velocity
)

controller.stop()
```

!!! tip "Thread safety"

    `on_classification()` is thread-safe and never blocks. Call it from the
    classifier loop as fast as you want — the MIDI engine handles debouncing
    and audio rendering on its own background thread.

---

## When the Ritual Falters

| Symptom | The remedy |
|---|---|
| `make setup` fails on FluidSynth | Update Conda first (`conda update conda`); on Apple Silicon, prefer Conda-Forge channel for `pyfluidsynth` |
| No BioRadio ports detected | Power-cycle the device and re-pair via Bluetooth at the OS level; `make gui-live` reads serial directly |
| GUI connects but plots are flat | Confirm **START** was clicked after **Connect**, and that each channel's signal type matches the physical electrode placement |
| `make ritual` times out finding LSL | Make sure the GUI is running with **Stream to LSL** checked; LSL discovery is local-network broadcast |
| Silent at `make music` | WASAPI not available -> falls back automatically; if still silent, confirm OS-level volume + correct output device |
| Classifier output is chaotic | Apply the bandpass + notch (already in `signal_processing.py`); check electrode contact and skin prep; record more training data for the noisy gesture |
| macOS Bluetooth serial issues | macOS Sonoma 14+ has known BT-serial regressions; run the GUI on a Windows host with **Stream to LSL** and consume the LSL stream from macOS |
| Pages-deploy build is silent | Confirm `docs.yml` was triggered by the push and that Pages is set to **Build and deployment / Source: GitHub Actions** in repo settings |
