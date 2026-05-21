---
title: BioRadio Music
hide:
  - navigation
  - toc
  - footer
---

<div class="hero" markdown>

# <span class="material-symbols-outlined" style="vertical-align: middle; margin-right: 8px; font-size: 1.2em;">graphic_eq</span> BioRadio Music

**Real-time biosignal-controlled musical performance**
{ .hero-subtitle }

<div class="hero-buttons" markdown>

[:octicons-rocket-24: Get Started](getting-started.md){ .md-button .md-button--primary }
[:octicons-book-24: Architecture](architecture.md){ .md-button }

</div>

<div class="hero-tagline" markdown>

<span class="material-symbols-outlined" style="vertical-align: middle; margin-right: 4px;">back_hand</span> 8 Gestures | <span class="material-symbols-outlined" style="vertical-align: middle; margin-right: 4px;">music_note</span> 7 Chords | <span class="material-symbols-outlined" style="vertical-align: middle; margin-right: 4px;">piano</span> 6 Instruments | <span class="material-symbols-outlined" style="vertical-align: middle; margin-right: 4px;">library_music</span> 6 Songs
{ .hero-modes }

</div>

</div>

---

## :material-power-plug: What it does

EMG signals from a forearm-mounted GLNeuroTech BioRadio stream over Lab Streaming Layer into a trained RandomForest classifier. The classifier picks one of eight hand gestures every 125 ms. Your right hand selects the chord; your left hand selects the instrument; EMG amplitude maps to MIDI velocity. FluidSynth renders the result through your speakers in one Python process. No DAW, no virtual MIDI cable, no offline rendering.

```
[BioRadio EMG] --LSL--> [250ms windows / 50% overlap] --> [bandpass + notch] -->
[5 features] --> [RandomForest (8 classes)] --> [state machine] --> [FluidSynth] --> [audio]
```

---

## :material-auto-fix: Capabilities

<div class="grid cards" markdown>

-   :material-back-hand:{ .lg .middle } __8 EMG-classified gestures__

    ---

    Five features per 250 ms window (RMS, MAV, variance, waveform length, zero crossings) feed a RandomForest trained on team-recorded EMG. Five other classifier variants (KNN, LDA, SVM, XGBoost, Ensemble) ship for comparison.

    [:octicons-arrow-right-24: Gesture map](architecture.md#gesture-mapping)

-   :material-piano:{ .lg .middle } __6 General MIDI instruments__

    ---

    Piano, Nylon Guitar, Steel Guitar, Electric Guitar, Strings, and a warm Pad. Swap the active voice with the left hand without interrupting the chord on the right.

    [:octicons-arrow-right-24: Instruments](midi-engine.md#instruments)

-   :material-library-music:{ .lg .middle } __6 songs as chord progressions__

    ---

    Save Your Tears, Blinding Lights, Careless Whisper, Love Story, Firework, and Secrets. Verse / chorus / bridge sections + a full song structure per JSON.

    [:octicons-arrow-right-24: Playlist](playlist.md)

-   :material-waves:{ .lg .middle } __Live velocity dynamics__

    ---

    EMG amplitude maps linearly to MIDI velocity (40-127). Squeeze harder mid-chord and the held notes re-voice at the new velocity. Drop the arm and the engine returns to idle.

    [:octicons-arrow-right-24: Engine internals](architecture.md#midi-engine-internals)

</div>

---

## :material-account-group: Built by

Built in 24 hours at the **AWARE-AI Spring Hackathon at RIT (Feb 2026)** organized around the GLNeuroTech BioRadio. Team: Victor Lockwood, Parth Kapur, Grant Bosworth, Sophia Caruana, and AJ Barea.
