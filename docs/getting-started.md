# :material-rocket-launch: Getting Started

## Prerequisites

| Requirement | Notes |
|---|---|
| **Python 3.10+** | Required for `pylsl`, `pyqt6`, `neurokit2` |
| **Conda** | Recommended (pulls FluidSynth as a system dep cleanly) |
| **GLNeuroTech BioRadio** | Optional for development (mock mode works without hardware) |
| **Speakers / headphones** | FluidSynth renders directly via WASAPI / DirectSound / WaveOut on Windows; equivalent OS drivers on macOS / Linux |

---

## 1. Install dependencies

```bash
make setup
conda activate hackathon
```

`make setup` resolves the Conda env from `environment.yml`, installing Python, the scientific stack (NumPy, SciPy, NeuroKit2, scikit-learn), PyQt6 + pyqtgraph for the GUI, `pylsl`, FluidSynth, and `pyfluidsynth` bindings.

If Conda is not an option:

```bash
pip install -r requirements.txt
# plus your platform's FluidSynth: apt install fluidsynth / brew install fluid-synth / etc.
```

---

## 2. Smoke check audio

```bash
make music
```

This runs `src/midi_demo.py`, which cycles through every instrument and chord through FluidSynth + the bundled `GeneralUser_GS.sf2` SoundFont. If you hear sound, your audio chain is correctly configured. If you hear nothing on Windows, confirm WASAPI is available in **Sound Settings -> Advanced -> Audio devices** before falling back to DirectSound or WaveOut.

---

## 3. Launch the GUI

```bash
make gui          # mock data, no hardware required
make gui-live     # connects to the BioRadio via Bluetooth serial
```

Inside the GUI:

1. Click **Scan Ports** to detect the BioRadio (Bluetooth pairing must already be set up at the OS level).
2. Click **Connect**.
3. Set each channel's signal type (EMG / EOG / EEG / GSR) in the per-channel dropdown.
4. Click **Apply Config**, then **START** to begin acquisition.
5. Check **Stream to LSL** to expose the data over Lab Streaming Layer.

---

## 4. Train the classifier

The team's labeled recordings live under `data/<recorder>/<gesture>_<index>_<timestamp>.csv`. To rebuild the classifier from scratch:

```bash
make train
```

This calls `src/pipeline.py` directly, which:

1. Loads every CSV under `data/`
2. Bandpasses + notches the EMG (20-450 Hz, 60 Hz)
3. Extracts five features per 250 ms window (RMS, MAV, Variance, Waveform Length, Zero Crossings)
4. Fits a `RandomForestClassifier` (defaults from `sklearn`)
5. Writes `models/classifier.pkl`

Five additional model variants ship pre-trained under `models/` for comparison: `classifier_knn.pkl`, `classifier_lda.pkl`, `classifier_svm.pkl`, `classifier_xgb.pkl`, and `classifier_ensemble.pkl`.

---

## 5. Start the real-time bridge

```bash
make ritual
```

`src/cosmic_ritual.py` is the bridge between the LSL stream and the MIDI engine. On startup it:

1. Initializes the MIDI engine (FluidSynth + SoundFont).
2. Loads `models/classifier.pkl` (falls back to `SimpleFeatureClassifier` if missing).
3. Discovers the `BioRadio` LSL stream (retries 8 times, 1 second between attempts).
4. Pulls 250 ms windows with 50% overlap from the stream.
5. Calls the classifier, then forwards the predicted gesture + amplitude to the MIDI engine.

You can also start the bridge from inside the GUI by ticking **Music Mode** during acquisition. The GUI surfaces the bridge's status (LSL connect / classifier load / stream pulled) inline so you can debug without a second terminal.

---

## 6. Drive MIDI from your own script

If you are building a custom classifier or want to control the engine directly:

```python
from midi_engine import MidiController

controller = MidiController()
controller.start()

# Called from your classifier loop, as fast as you like:
controller.on_classification(
    right_hand="palm_up_out",    # chord (8 gestures)
    left_hand="fist_down_out",   # instrument (same 8-gesture vocabulary)
    amplitude=0.73,              # 0.0-1.0 -> MIDI velocity 40-127
)

controller.stop()
```

!!! tip "Thread safety"
    `on_classification()` is thread-safe and never blocks. The MIDI engine debounces (3 consecutive frames by default) and renders audio on its own background thread, so the classifier loop can run at any rate without back-pressure.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `make setup` fails on FluidSynth | Confirm Conda is on the latest version (`conda update conda`); on Apple Silicon, prefer Conda-Forge channel for `pyfluidsynth` |
| No BioRadio ports detected | Power-cycle the device and re-pair via Bluetooth at the OS level; `make gui-live` reads serial directly |
| GUI connects but plots are flat | Check that **START** was clicked after **Connect**, and that the per-channel signal type matches the physical electrode setup |
| `make ritual` times out finding LSL | Make sure the GUI is running with **Stream to LSL** checked; LSL discovery is local-network broadcast |
| Silent at `make music` | WASAPI not available -> falls back automatically; if still silent, confirm OS-level volume + correct output device |
| Noisy classification | Apply the bandpass + notch (already in `signal_processing.py`); check electrode contact and skin prep; record more training data for the noisy gesture |
| macOS Bluetooth serial issues | macOS Sonoma 14+ has known BT-serial regressions; run the GUI on a Windows host with **Stream to LSL** and consume the LSL stream from macOS |
