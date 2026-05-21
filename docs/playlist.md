# :material-playlist-music: Playlist

Six songs are ready to play. Each lives as a JSON file in `playlist/` with named sections and a full song structure that the MIDI engine steps through one chord per triggered gesture.

---

## Available songs

| Song | Artist | BPM | Chords used |
|---|---|---|---|
| **Save Your Tears** | The Weeknd & Ariana Grande | 118 | C, Am, Em, G, Dm, F |
| **Blinding Lights** | The Weeknd | 171 | Dm, Am, C, G |
| **Careless Whisper** | George Michael | 76 | Am, D, F, Em |
| **Love Story** | Taylor Swift | 119 | G, D, Em, C |
| **Firework** | Katy Perry | 124 | G, Am, Em, C, D |
| **Secrets** | The Weeknd | 92 | F, G, D |

---

## Song details

### Save Your Tears

| Section | Progression |
|---|---|
| Verse | C - Am - Em - G |
| Chorus | Dm - Am - F - G |
| Refrain | C - Am - Em - G |

```python
controller.set_progression("save_your_tears")
controller.set_progression("save_your_tears", section="verse")
controller.set_progression("save_your_tears", section="chorus")
```

### Blinding Lights

| Section | Progression |
|---|---|
| Verse | Dm - Am - C - G |
| Chorus | Dm - Am - C - G |

!!! note
    Simplified from the original key of F minor (Fm - Cm - Eb - Bb) so every chord exists in the engine's seven-voicing palette.

```python
controller.set_progression("blinding_lights")
```

### Careless Whisper

| Section | Progression |
|---|---|
| Verse | Am - D - F - Em |
| Chorus | Am - D - F - Em |

```python
controller.set_progression("careless_whisper")
```

### Love Story

| Section | Progression |
|---|---|
| Verse | G - D - Em - C |
| Chorus | G - D - Em - C |

```python
controller.set_progression("love_story")
```

### Firework

| Section | Progression |
|---|---|
| Verse | G - Am - Em - C |
| Chorus | G - Am - Em - C |
| Bridge | Em - C - G - D |

```python
controller.set_progression("firework")
controller.set_progression("firework", section="bridge")
```

### Secrets

| Section | Progression |
|---|---|
| Verse | F - G - F - G |
| Chorus | F - G - D - G |

```python
controller.set_progression("secrets")
```

---

## Adding a new song

Create a JSON file under `playlist/`:

```json title="playlist/my_song.json"
{
    "title": "My Song",
    "artist": "Artist Name",
    "bpm": 120,
    "sections": {
        "verse":  ["Am", "F", "C", "G"],
        "chorus": ["F", "G", "Am", "Am"]
    },
    "full_structure": [
        "verse", "verse",
        "chorus",
        "verse",
        "chorus", "chorus"
    ]
}
```

Available chords are limited to the seven voicings the engine ships: `C`, `Am`, `Em`, `G`, `Dm`, `F`, `D`. If a song uses a chord outside this set, simplify or transpose into the palette (see Blinding Lights for an example).

Load the new song with:

```python
controller.set_progression("my_song")
controller.set_progression("my_song", section="verse")
```

---

## How the progression advances

`set_progression()` loads a section (or the song's `full_structure`) into the engine's chord queue. From then on, every classifier-triggered chord transition advances the queue by one step instead of mapping the gesture directly to a chord. This lets a performer "play along" with a known song while still controlling timing and velocity via EMG.

Calling `clear_progression()` returns the engine to free-play, where the right-hand gesture maps directly to a chord per the [gesture map](architecture.md#right-hand-chord-selection).
