<div align="center">

<img src="https://res.cloudinary.com/dumwa1w5x/image/upload/v1779376928/bioradio_raazmn.png" alt="BioRadio Music" width="720">

# BioRadio Music

### Real-time biosignal-controlled musical performance

*EMG from a GLNeuroTech BioRadio streams over Lab Streaming Layer, gets classified into eight hand gestures, and renders to chords + instruments through FluidSynth in real time. No DAW, no virtual MIDI routing, one Python process.*

[![Docs](https://img.shields.io/badge/Docs-Zensical-blue?style=flat-square)](https://ajbarea.github.io/bioradio-music/)
[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-RandomForest-F7931E?style=flat-square&logo=scikit-learn&logoColor=white)](https://scikit-learn.org)
[![FluidSynth](https://img.shields.io/badge/Audio-FluidSynth-A82828?style=flat-square)](https://www.fluidsynth.org/)
[![Hackathon](https://img.shields.io/badge/AWARE--AI-Spring%202026-9d27b0?style=flat-square)](https://www.rit.edu/events/aware-ai-spring-hackathon-1)

</div>

---

## What is this?

BioRadio Music is a two-handed musical instrument controlled by forearm muscle signals. A trained classifier reads EMG windows from the BioRadio, picks a chord from your right hand and an instrument from your left, and renders the result directly to your speakers through FluidSynth. Hold a gesture and the chord sustains; tense your arm and the velocity rises in real time; drop your arm and the music stops.

```
[BioRadio EMG] --LSL--> [250ms windows] --> [bandpass + notch] --> [features] -->
[RandomForest (8 classes)] --> [chord + instrument + velocity] --> [FluidSynth] --> [audio]
```

Built in 24 hours at the AWARE-AI Spring Hackathon at RIT (Feb 2026) by Victor Lockwood, Parth Kapur, Grant Bosworth, Sophia Caruana, and AJ Barea.

---

## Quick start

```bash
git clone https://github.com/ajbarea/bioradio-music
cd bioradio-music

make setup           # conda env + scientific deps + FluidSynth bindings
conda activate hackathon

make music           # smoke check FluidSynth + SoundFont
make gui             # launch the data collection GUI in mock mode
make train           # train the gesture classifier from recorded CSVs
make ritual          # start the real-time bridge: BioRadio --> classifier --> audio
```

`make help` lists every target with one-line descriptions. The full setup walkthrough lives in [Getting Started](https://ajbarea.github.io/bioradio-music/getting-started/).

---

## What's included

| | |
|---|---|
| **8 gesture classes** | `palm_up_out`, `palm_down_out`, `palm_down_up`, `fist_down_out`, `fist_down_up`, `peace_out`, `arm_up`, `arm_down` |
| **7 chord voicings** | C, Am, Em, G, Dm, F, D (5-6 note voicings, not triads) |
| **6 GM instruments** | Piano, Nylon Guitar, Steel Guitar, Electric Guitar, Strings, Pad |
| **6 songs** | Save Your Tears, Blinding Lights, Careless Whisper, Love Story, Firework, Secrets |
| **5 classifier variants** | RandomForest (default), KNN, LDA, SVM, XGBoost, plus an Ensemble |

Full reference: [Architecture](https://ajbarea.github.io/bioradio-music/architecture/) · [MIDI Engine](https://ajbarea.github.io/bioradio-music/midi-engine/) · [Playlist](https://ajbarea.github.io/bioradio-music/playlist/)

---

## Signal pipeline

| Stage | File | What it does |
|---|---|---|
| Capture | `src/hackathon_gui.py` | BioRadio serial, LSL, or mock stream + CSV recording + Music Mode toggle |
| Real-time bridge | `src/cosmic_ritual.py` | LSL consumer with 250ms windows, 50% overlap, GUI status callbacks |
| Preprocessing | `src/signal_processing.py` | Bandpass 20-450 Hz + 60 Hz notch |
| Features | `src/pipeline.py` | RMS, MAV, Variance, Waveform Length, Zero Crossings |
| Classifier | `src/pipeline.py` | RandomForest over 8 gesture classes (other variants under `models/`) |
| Synthesis | `src/midi_engine.py` | Gesture-to-chord-to-MIDI state machine + FluidSynth (WASAPI / DirectSound / WaveOut) |

---

## Gesture map

**Right hand selects the chord:**

| Gesture | Chord |
|---|---|
| `palm_up_out` | C major |
| `palm_down_out` | A minor |
| `palm_down_up` | E minor |
| `fist_down_out` | G major |
| `fist_down_up` | D minor |
| `peace_out` | F major |
| `arm_up` | D major |
| `arm_down` | Rest (silence) |

**Left hand selects the instrument** (Piano / Nylon Guitar / Steel Guitar / Electric Guitar / Strings / Pad) using the same eight-gesture vocabulary.

---

## Tech stack

| | |
|---|---|
| Hardware | GLNeuroTech BioRadio (EMG via Bluetooth serial) |
| Transport | Lab Streaming Layer (`pylsl`) |
| Signal processing | NumPy + SciPy + NeuroKit2 |
| Classification | scikit-learn RandomForest (KNN / LDA / SVM / XGB variants benchmarked) |
| Audio | FluidSynth via `pyfluidsynth`, GeneralUser GS SoundFont |
| GUI | PyQt6 + pyqtgraph |
| Packaging | Conda (`environment.yml`) + pip (`requirements.txt`) |
| Docs | Zensical |

---

## Repository layout

```
bioradio-music/
├── src/
│   ├── hackathon_gui.py       # PyQt6 data collection GUI
│   ├── bioradio.py            # BioRadio serial driver
│   ├── signal_processing.py   # Filters, features, EMG/EOG/GSR/IMU utilities
│   ├── pipeline.py            # ML pipeline: preprocess, features, classifier
│   ├── midi_engine.py         # State machine + FluidSynth controller
│   ├── cosmic_ritual.py       # Real-time bridge: LSL --> classifier --> MIDI
│   ├── realtime_process.py    # Realtime processing helpers
│   └── midi_demo.py           # FluidSynth + SoundFont smoke demo
├── examples/                  # Minimal connect-and-stream snippets
├── data/                      # Labeled CSVs per gesture, per recorder
├── models/                    # Trained classifier variants (.pkl)
├── playlist/                  # Song chord progressions (.json)
├── soundfonts/                # GeneralUser_GS.sf2 (~30 MB)
├── docs/                      # Zensical-built documentation site
└── presentations/             # Hackathon kickoff + signal-type slides
```

---

## Acknowledgments

Built for the **AWARE-AI Spring Hackathon at RIT (Feb 2026)** organized around the GLNeuroTech BioRadio. Forked from the team's submission repo so the URL stays under our control; the docs site here is the BioRadio Music project, not the generic hackathon starter.

| | |
|---|---|
| Team | Victor Lockwood, Parth Kapur, Grant Bosworth, Sophia Caruana, AJ Barea |
| Event | [AWARE-AI Spring Hackathon (RIT)](https://www.rit.edu/events/aware-ai-spring-hackathon-1) |
| LinkedIn | [post · #hackathon · #appliedai · #hci](https://www.linkedin.com/posts/aj-barea_hackathon-appliedai-hci-share-7433372743804375040-QAOy) |

---

*Source for [ajbarea.github.io](https://ajbarea.github.io/) project entry [`bioradio-music`](https://ajbarea.github.io/projects).*
