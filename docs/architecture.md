# :material-sitemap: Architecture

## System overview

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {
    'primaryColor': '#1a1a2e',
    'primaryTextColor': '#e0e0ff',
    'primaryBorderColor': '#9d27b0',
    'lineColor': '#00ffa2',
    'secondaryColor': '#0f0f1a',
    'tertiaryColor': '#12121f',
    'edgeLabelBackground': '#0a0a15',
    'clusterBkg': '#0f0f1a',
    'clusterBorder': '#9d27b0',
    'titleColor': '#e0e0ff',
    'nodeTextColor': '#e0e0ff'
}, 'flowchart': {'nodeSpacing': 30, 'rankSpacing': 40}}}%%
flowchart TD
    subgraph capture ["Signal Capture"]
        BR["BioRadio<br/>EMG sensors"]
        GUI["Data collection GUI<br/>hackathon_gui.py"]
    end

    subgraph bridge ["Real-time Bridge · cosmic_ritual.py"]
        CR["LSL consumer<br/>250 ms windows, 50% overlap"]
        SP["Bandpass + notch<br/>signal_processing.py"]
        PP["Feature extraction<br/>pipeline.py"]
        RF["RandomForest classifier<br/>8 gesture classes"]
    end

    subgraph audio ["Audio Output"]
        MC["MidiController<br/>midi_engine.py"]
        FS["FluidSynth + SoundFont"]
        SPK["Speakers"]
    end

    BR -->|"raw EMG"| GUI
    GUI -->|"LSL stream + status callbacks"| CR
    CR -->|"raw window"| SP
    SP -->|"filtered"| PP
    PP -->|"feature vector"| RF
    RF -->|"gesture + amplitude"| MC
    MC -->|"MIDI messages"| FS
    FS -->|"audio"| SPK

    style BR fill:#1a1a2e,stroke:#9d27b0,stroke-width:2px,color:#e0e0ff
    style GUI fill:#1a1a2e,stroke:#9d27b0,stroke-width:2px,color:#e0e0ff
    style CR fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style SP fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style PP fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style RF fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style MC fill:#1a1a2e,stroke:#d05ce3,stroke-width:2px,color:#e0e0ff
    style FS fill:#1a1a2e,stroke:#d05ce3,stroke-width:2px,color:#e0e0ff
    style SPK fill:#1a1a2e,stroke:#d05ce3,stroke-width:2px,color:#e0e0ff
```

---

## Signal flow

| Stage | File | What it does |
|---|---|---|
| **Capture** | <nobr>`hackathon_gui.py`</nobr> | PyQt6 GUI that streams raw EMG from BioRadio serial, an LSL inlet, or a mock source. Records labeled CSVs and hosts the Music Mode toggle that starts the real-time bridge. |
| **Real-time bridge** | <nobr>`cosmic_ritual.py`</nobr> | Consumes the LSL stream, windows the samples (250 ms, 50% overlap), classifies each window, and feeds the MIDI engine. Reports connection + classifier status back to the GUI via callbacks. Falls back to `SimpleFeatureClassifier` if `models/classifier.pkl` is missing. |
| **Preprocessing** | <nobr>`signal_processing.py`</nobr> | Bandpass filter (20-450 Hz) to isolate the EMG band, plus a 60 Hz notch to kill power-line interference. |
| **Feature extraction** | <nobr>`pipeline.py`</nobr> | Five time-domain features per window: RMS, MAV (mean absolute value), variance, waveform length, zero crossings. |
| **Classification** | <nobr>`pipeline.py`</nobr> | `RandomForestClassifier` trained on the team's labeled CSVs under `data/`. Five other variants (KNN, LDA, SVM, XGBoost, Ensemble) ship under `models/` for offline comparison. |
| **Music synthesis** | <nobr>`midi_engine.py`</nobr> | Gesture-to-chord and gesture-to-instrument mapping, state machine, FluidSynth driver. Auto-selects WASAPI / DirectSound / WaveOut on Windows. |

---

## Gesture mapping

### Right hand: chord selection

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {
    'primaryColor': '#1a1a2e',
    'primaryTextColor': '#e0e0ff',
    'primaryBorderColor': '#00ffa2',
    'lineColor': '#00ffa2',
    'edgeLabelBackground': '#0a0a15',
    'nodeTextColor': '#e0e0ff'
}}}%%
flowchart LR
    PUO["palm_up_out"] --> C["C major"]
    PDO["palm_down_out"] --> Am["A minor"]
    PDU["palm_down_up"] --> Em["E minor"]
    FDO["fist_down_out"] --> G["G major"]
    FDU["fist_down_up"] --> Dm["D minor"]
    PO["peace_out"] --> F["F major"]
    AU["arm_up"] --> D["D major"]
    AD["arm_down"] --> REST["Rest"]

    style PUO fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style PDO fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style PDU fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style FDO fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style FDU fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style PO fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style AU fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style AD fill:#1a1a2e,stroke:#9d27b0,stroke-width:2px,color:#b0b0d0
    style C fill:#1a1a2e,stroke:#d05ce3,stroke-width:2px,color:#e0e0ff
    style Am fill:#1a1a2e,stroke:#d05ce3,stroke-width:2px,color:#e0e0ff
    style Em fill:#1a1a2e,stroke:#d05ce3,stroke-width:2px,color:#e0e0ff
    style G fill:#1a1a2e,stroke:#d05ce3,stroke-width:2px,color:#e0e0ff
    style Dm fill:#1a1a2e,stroke:#d05ce3,stroke-width:2px,color:#e0e0ff
    style F fill:#1a1a2e,stroke:#d05ce3,stroke-width:2px,color:#e0e0ff
    style D fill:#1a1a2e,stroke:#d05ce3,stroke-width:2px,color:#e0e0ff
    style REST fill:#0a0a15,stroke:#9d27b0,stroke-width:2px,stroke-dasharray:5 5,color:#b0b0d0
```

`arm_down` is the explicit silence gesture. It stops any held chord and returns the engine to idle.

### Left hand: instrument selection

| Gesture | Instrument |
|---|---|
| `fist_down_out` | Piano |
| `palm_up_out` | Nylon Guitar |
| `palm_down_out` | Steel Guitar |
| `palm_down_up` | Electric Guitar |
| `fist_down_up` | Strings |
| `peace_out` | Pad (Warm) |
| `arm_up` | Nylon Guitar |
| `arm_down` | Nylon Guitar |

Left-hand gestures share the same eight-gesture vocabulary as the right hand but map onto six General MIDI programs. The classifier is one model trained on muscle activity at the forearm; which hand is "left" or "right" is a convention enforced at the engine layer.

---

## MIDI engine internals

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {
    'primaryColor': '#1a1a2e',
    'primaryTextColor': '#e0e0ff',
    'primaryBorderColor': '#00ffa2',
    'lineColor': '#00ffa2',
    'secondaryColor': '#0f0f1a',
    'tertiaryColor': '#12121f',
    'nodeTextColor': '#e0e0ff'
}}}%%
flowchart TD
    START(( )) --> IDLE

    IDLE["Idle"]
    PLAYING["Playing chord"]
    SUSTAIN["Sustain"]

    IDLE -->|"new gesture"| PLAYING
    PLAYING -->|"hold gesture"| SUSTAIN
    SUSTAIN -->|"change gesture"| PLAYING
    PLAYING -->|"arm_down"| IDLE
    SUSTAIN -->|"arm_down"| IDLE

    style START fill:#00ffa2,stroke:#00ffa2,color:#00ffa2
    style IDLE fill:#1a1a2e,stroke:#9d27b0,stroke-width:2px,color:#e0e0ff
    style PLAYING fill:#1a1a2e,stroke:#00ffa2,stroke-width:2px,color:#e0e0ff
    style SUSTAIN fill:#1a1a2e,stroke:#d05ce3,stroke-width:2px,color:#e0e0ff
```

The state machine **debounces** noisy classifier output (default: 3 consecutive same-gesture frames before triggering a transition) and **handles chord transitions cleanly** by firing note-off on the outgoing chord before note-on on the incoming one.

**Velocity mapping.** EMG amplitude (0.0-1.0) maps linearly to MIDI velocity in the range **40-127**. The floor of 40 ensures soft gestures stay audible while preserving dynamic range for tense ones.

**Dynamic re-voicing.** During the `Sustain` state, amplitude updates re-voice the held notes at the new velocity. Squeeze harder mid-chord and the held notes swell in real time without re-triggering note-on.

**Audio driver fallback.** On Windows, the engine attempts **WASAPI** first (lowest latency), falling back to **DirectSound** then **WaveOut** if WASAPI is unavailable. Linux + macOS use the platform default through `pyfluidsynth`.

---

## Key files

| File | Purpose |
|---|---|
| <nobr>`src/midi_engine.py`</nobr> | MIDI engine: state machine, `MidiController`, FluidSynth driver, playlist loader |
| <nobr>`src/cosmic_ritual.py`</nobr> | Real-time bridge: LSL consumer, windowing, classifier call, GUI status callbacks |
| <nobr>`src/midi_demo.py`</nobr> | Standalone audio smoke test: cycles instruments and chords through FluidSynth |
| <nobr>`src/pipeline.py`</nobr> | ML pipeline: feature extraction + classifier training entry point |
| <nobr>`src/pipeline_v2.py`</nobr> | Iteration of the pipeline with a richer feature set + classifier comparison |
| <nobr>`src/realtime_process.py`</nobr> | Realtime processing helpers (windowing, debounce, amplitude estimation) |
| <nobr>`src/hackathon_gui.py`</nobr> | PyQt6 data-collection GUI + LSL streamer + Music Mode toggle |
| <nobr>`src/signal_processing.py`</nobr> | Filters, EMG features, EOG / GSR / IMU utilities (kept from the hackathon starter) |
| <nobr>`models/classifier.pkl`</nobr> | Active classifier loaded by the real-time bridge |
| <nobr>`models/classifier_*.pkl`</nobr> | KNN / LDA / RF / SVM / XGB / Ensemble variants for offline comparison |
| <nobr>`playlist/*.json`</nobr> | Per-song chord progressions (6 songs) |
| <nobr>`soundfonts/GeneralUser_GS.sf2`</nobr> | FluidSynth SoundFont (~30 MB, bundled) |

---

## Latency budget

End-to-end latency from a muscle activation to audible MIDI output:

| Stage | Approx. time |
|---|---|
| EMG sample to BioRadio buffer | 4-10 ms (sensor + Bluetooth serial) |
| BioRadio to LSL stream | < 5 ms |
| 250 ms window collection | 125 ms (with 50% overlap, classifier fires every 125 ms) |
| Feature extraction + classification | < 5 ms (RandomForest, ~5 features) |
| Debounce (3 frames default) | up to 375 ms additional |
| `MidiController.on_classification()` -> note-on | < 1 ms |
| FluidSynth render -> WASAPI | typically < 10 ms |

The dominant term is the window + debounce path. Reducing window size or debounce count trades responsiveness for classifier stability; the defaults are the team's tuned compromise.
