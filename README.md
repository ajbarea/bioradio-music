<div align="center">

<img src="https://res.cloudinary.com/dumwa1w5x/image/upload/v1779376928/bioradio_raazmn.png" alt="BioRadio Music" width="720">

# 🕯️ BioRadio Music

### *Eldritch Biosignals · BioRadio Hackathon 2026*

*Turn biological electrical pulses into haunting melodies. EMG from a GLNeuroTech BioRadio streams over Lab Streaming Layer, gets classified into eight physical incantations, and renders to chords + instruments through FluidSynth in real time. No DAW, no virtual MIDI cable, no offline rendering. One Python process, one SoundFont, sound out.*

[![Docs](https://img.shields.io/badge/Docs-Zensical-blue?style=flat-square)](https://ajbarea.github.io/bioradio-music/)
[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-RandomForest-F7931E?style=flat-square&logo=scikit-learn&logoColor=white)](https://scikit-learn.org)
[![FluidSynth](https://img.shields.io/badge/Audio-FluidSynth-A82828?style=flat-square)](https://www.fluidsynth.org/)
[![Hackathon](https://img.shields.io/badge/AWARE--AI-Spring%202026-9d27b0?style=flat-square)](https://www.rit.edu/events/aware-ai-spring-hackathon-1)

</div>

---

## What is this?

A two-handed musical instrument controlled by forearm muscle signals. A trained classifier reads EMG windows from the BioRadio, picks a chord from your right hand and an instrument from your left, and renders the result directly to your speakers through FluidSynth. Hold a gesture and the chord sustains. Tense your arm and the velocity rises in real time. Drop your arm and the ritual ends.

```
Biological Static ──► The Ritual ──► The Oracle (ML) ──► Sonic Engine ──► Audio
   (BioRadio EMG)   (cosmic_ritual)   (RandomForest)    (midi_engine)    (Void)
```

Summoned in 24 hours of frantic ritual at the AWARE-AI Spring Hackathon at RIT (Feb 2026) by Victor Lockwood, Parth Kapur, Grant Bosworth, Sophia Caruana, and AJ Barea.

---

## Quick start

```bash
git clone https://github.com/ajbarea/bioradio-music
cd bioradio-music

make setup           # conda env + scientific deps + FluidSynth bindings
conda activate hackathon

make music           # smoke check FluidSynth + SoundFont (the void hums)
make gui             # launch the data collection GUI in mock mode
make train           # train the gesture classifier from recorded EMG
make ritual          # start the real-time bridge: BioRadio --> classifier --> audio
```

`make help` lists every target with one-line descriptions. The full walkthrough lives at [Wake the Ancient](https://ajbarea.github.io/bioradio-music/getting-started/).

---

## What's summoned

| | |
|---|---|
| **8 Physical Incantations** | `palm_up_out`, `palm_down_out`, `palm_down_up`, `fist_down_out`, `fist_down_up`, `peace_out`, `arm_up`, `arm_down` |
| **7 Chord Voicings** | C, Am, Em, G, Dm, F, D (five to six notes per voicing, guitar-style) |
| **6 Spectral Voices** | Piano, Nylon Guitar, Steel Guitar, Electric Guitar, Strings, Pad (Warm) |
| **6 Cursed Melodies** | Save Your Tears, Blinding Lights, Careless Whisper, Love Story, Firework, Secrets |
| **5 Classifier Vessels** | RandomForest (default), KNN, LDA, SVM, XGBoost, plus an Ensemble |

Full reference: [Forbidden Rituals (Architecture)](https://ajbarea.github.io/bioradio-music/architecture/) · [The Sonic Engine](https://ajbarea.github.io/bioradio-music/midi-engine/) · [The Playlist Grimoire](https://ajbarea.github.io/bioradio-music/playlist/)

---

## Signal pipeline

| Stage | File | What it does |
|---|---|---|
| Capture | `src/hackathon_gui.py` | BioRadio serial, LSL inlet, or mock stream + CSV recording + Music Mode toggle |
| The Ritual (real-time bridge) | `src/cosmic_ritual.py` | LSL consumer with 250 ms windows, 50% overlap, GUI status callbacks |
| Preprocessing | `src/signal_processing.py` | Bandpass 20-450 Hz + 60 Hz notch (kills power-line static) |
| Feature extraction | `src/pipeline.py` | RMS, MAV, Variance, Waveform Length, Zero Crossings per window |
| The Oracle (classifier) | `src/pipeline.py` | RandomForest over 8 gesture classes (other variants under `models/`) |
| Sonic Engine | `src/midi_engine.py` | Gesture-to-chord-to-MIDI state machine + FluidSynth (WASAPI / DirectSound / WaveOut) |

---

## Gesture map

**Right hand selects the chord:**

| Gesture | Chord | Mood |
|---|---|---|
| `palm_up_out` | C major | calling the light |
| `palm_down_out` | A minor | grief |
| `palm_down_up` | E minor | unease |
| `fist_down_out` | G major | resolve |
| `fist_down_up` | D minor | sorrow |
| `peace_out` | F major | repose |
| `arm_up` | D major | summons |
| `arm_down` | Rest | the void answers |

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
| Docs | Zensical (eldritch theme inside) |

---

## Repository layout

```
bioradio-music/
├── src/
│   ├── hackathon_gui.py       # PyQt6 data collection GUI + Music Mode toggle
│   ├── bioradio.py            # BioRadio serial driver
│   ├── signal_processing.py   # Filters, features, EMG/EOG/GSR/IMU utilities
│   ├── pipeline.py            # ML pipeline: preprocess, features, classifier
│   ├── midi_engine.py         # State machine + FluidSynth controller
│   ├── cosmic_ritual.py       # The Ritual: LSL --> classifier --> MIDI
│   ├── realtime_process.py    # Realtime processing helpers
│   └── midi_demo.py           # FluidSynth + SoundFont smoke demo
├── examples/                  # Minimal connect-and-stream snippets
├── data/                      # Labeled CSVs per gesture, per recorder
├── models/                    # Trained classifier vessels (.pkl)
├── playlist/                  # Song chord progressions (.json)
├── soundfonts/                # GeneralUser_GS.sf2 (~30 MB)
├── docs/                      # Zensical-built documentation site
└── presentations/             # Hackathon kickoff + signal-type slides
```

---

## Cult of Cosmic Horror

Summoned for the **AWARE-AI Spring Hackathon at RIT (Feb 2026)** organized around the GLNeuroTech BioRadio. Forked from the team's submission repo so the URL stays under our control; the docs site here is the BioRadio Music project, not the generic hackathon starter.

| | |
|---|---|
| Team | Victor Lockwood, Parth Kapur, Grant Bosworth, Sophia Caruana, AJ Barea |
| Event | [AWARE-AI Spring Hackathon (RIT)](https://www.rit.edu/events/aware-ai-spring-hackathon-1) |
| LinkedIn | [post · #hackathon · #appliedai · #hci](https://www.linkedin.com/posts/aj-barea_hackathon-appliedai-hci-share-7433372743804375040-QAOy) |

---

*Source for [ajbarea.github.io](https://ajbarea.github.io/) project entry [`bioradio-music`](https://ajbarea.github.io/projects). The void echoes.*
