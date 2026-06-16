# How Astrolabe maps your music

*A look under the hood at how Astrolabe turns a music library into a navigable
3-D "sky." Current as of v0.1.0 — the feature set and parameters are an active
area of refinement, so specifics may shift in later versions.*

Astrolabe places every song in your library as a **star** in a three-dimensional
map. Songs that *sound* alike sit close together; songs that sound nothing alike
end up on opposite sides. There are no genre tags, no scraped metadata, no
"people who liked X also liked Y." The position of each star is derived entirely
from **the audio itself** — and all of it is computed on your device.

This page explains how that works, end to end.

---

## The big picture

```
audio file
   │  decode → mono PCM
   ▼
loudness normalization          (so recording level doesn't dominate)
   │
   ▼
feature extraction              → a 44-number "fingerprint" per song
   │
   ▼
standardization (z-score)       (put every feature on the same scale)
   │
   ▼
family weighting                (emphasize timbre vs rhythm vs harmony…)
   │
   ▼
UMAP                            → 44 dimensions squeezed down to 3
   │
   ├──► star positions (the Sky)
   ├──► k-means clustering        → the default star colors
   └──► nearest / farthest graph  → the "similar" and "different" edges
```

Each step is described below. The key idea: we summarize each song as a vector
of numbers that capture *how it sounds*, then use dimensionality reduction to
lay those vectors out in a space you can fly through.

---

## Step 1 — From audio to a 44-number fingerprint

Astrolabe decodes each track to mono PCM and computes a **44-dimensional feature
vector** (internally called the "Enriched" vector). Every number is a summary
statistic over the whole track. They fall into five **families**:

| Dims | Feature | Family | What it captures |
|---|---|---|---|
| `0–12` | MFCC mean (13) | **Timbre** | The overall *texture/color* of the sound |
| `13–25` | MFCC std-dev (13) | **Timbre** | How much that texture *varies* over the track |
| `26–37` | Chroma mean (12) | **Harmony** | Energy in each of the 12 pitch classes (key/tonality) |
| `38` | Spectral centroid | **Brightness** | The "center of mass" of the spectrum (bright vs dark) |
| `39` | Spectral bandwidth | **Brightness** | How spread-out the spectrum is (pure vs noisy/dense) |
| `40` | Onset density | **Rhythm** | How many note onsets per second (busy vs sparse) |
| `41` | Onset-rate regularity | **Rhythm** | How steady/periodic the rhythm is (metronomic vs free) |
| `42` | Onset strength mean | **Rhythm** | How percussive/punchy the attacks are |
| `43` | RMS loudness (dB) | **Dynamics** | Overall energy/loudness (relative, see below) |

Feature extraction uses [Essentia](https://essentia.upf.org/) with
librosa-compatible parameters (a 2048-sample FFT window, 512-sample hop) so the
numbers match Astrolabe's research bench.

### Timbre — MFCCs (dims 0–25)

**Mel-Frequency Cepstral Coefficients** are the workhorse of audio similarity.
Intuitively, they describe the *shape* of the spectrum — the qualities that let
you tell a flute from a distorted guitar from a breathy vocal, independent of
the actual notes being played. We keep 13 coefficients and store two statistics
of each across the track:

- the **mean** (dims 0–12) → the song's characteristic timbre, and
- the **standard deviation** (dims 13–25) → how much the timbre *changes* (a
  track that stays in one texture vs one that lurches between sections).

This is the largest family (26 of 44 numbers) because timbre carries most of
what people perceive as "sounds like."

### Harmony — Chroma (dims 26–37)

A **chroma** vector folds the whole spectrum down onto the 12 pitch classes
(C, C♯, D, … B), ignoring octave. The 12 numbers are the average energy in each
pitch class over the track, which captures key and broad harmonic character —
e.g. a song centered on a minor key vs a bright major one, or something
atonal/noisy with energy smeared across all twelve. Astrolabe computes it with a
constant-Q transform so each pitch class is resolved cleanly.

### Brightness — Spectral centroid & bandwidth (dims 38–39)

Two single numbers about *where* the energy sits in the spectrum:

- **Spectral centroid** — the frequency "center of mass." High = bright, airy,
  hissy; low = dark, warm, bassy.
- **Spectral bandwidth** — how widely energy is spread around that centroid.
  Low = pure/tonal; high = noisy, dense, or distorted.

### Rhythm — Onset features (dims 40–42)

Derived from an **onset envelope** (a half-wave-rectified flux of a log-power mel
spectrogram — essentially "how much new energy is appearing moment to moment"):

- **Onset density** — onsets per second. Busy/percussive vs sparse/ambient.
- **Onset-rate regularity** — the periodicity of those onsets (via
  autocorrelation). A steady four-on-the-floor scores high; rubato or free-time
  music scores low.
- **Onset strength mean** — the average punch of the attacks.

### Dynamics — Loudness (dim 43)

A single **RMS loudness** value in decibels. Critically, every track is
**loudness-normalized before extraction** (see next section), so this dimension
reflects *relative* dynamics, not how hot a particular file was mastered.

---

## Step 2 — Loudness normalization

Raw recording level is mostly an artifact of mastering, not musical content — a
quietly-mastered jazz record and a brick-walled pop single could sit far apart
for no meaningful reason. So before extracting features, Astrolabe RMS-normalizes
each track to a common target with a gentle soft-knee limiter. This keeps the
map organized by *how music sounds*, not by how loud the file happens to be.

---

## Step 3 — Standardization (z-scoring)

The 44 features live on wildly different scales (a chroma value and a loudness in
dB aren't comparable). If we fed them to UMAP as-is, the large-magnitude features
would dominate the geometry. So each dimension is **standardized** across your
library — recentered to mean 0 and rescaled to unit variance — so every feature
gets an equal vote.

The scaling statistics are **frozen at your first brew**. New songs added later
are transformed with those same frozen statistics, so the map stays stable as
your library grows (a new import doesn't reshuffle everything).

---

## Step 4 — Family weighting

After standardizing, each family's block of columns is multiplied by a **family
weight** (Timbre, Harmony, Brightness, Rhythm, Dynamics). By default all weights
are `1.0` — every family contributes equally. Raising a weight stretches that
family's axes, pulling the map's organization toward it (e.g. weight rhythm up
and the sky reorganizes more by groove than by timbre). These are exposed as
sliders in Settings → Sky for experimentation.

---

## Step 5 — UMAP: 44 dimensions → 3

You can't look at a 44-dimensional space, so Astrolabe projects it down to 3
dimensions with **[UMAP](https://umap-learn.readthedocs.io/)** (Uniform Manifold
Approximation and Projection). UMAP builds a graph of each song's nearest
neighbors in the full 44-D space, then finds a 3-D layout that preserves that
neighborhood structure as faithfully as possible — keeping local groups of
similar songs together while still laying out the global shape.

Default parameters (tunable in Settings → Sky, applied on a full rebuild):

| Parameter | Default | Effect |
|---|---|---|
| Components | 3 | Output dimensions (the Sky is 3-D) |
| `n_neighbors` | 15 | Higher = more global structure; lower = tighter local clumps |
| `min_dist` | 0.1 | How tightly points may pack together |
| `spread` | 1.0 | Overall scale of the embedding |
| seed | 42 | Fixed, so the same library lays out the same way |

The output 3-D coordinates *are* the positions of the stars in the Sky.

> **Worth knowing:** UMAP distances are about *neighborhoods*, not absolute
> meaning. "These two stars are close" is trustworthy; "this cluster is exactly
> twice as far from that one" is not. Treat the Sky as a relational map, not a
> ruler.

---

## Step 6 — Clusters: the default colors

To color the stars, Astrolabe runs **k-means clustering** (default **K = 6**) on
the same standardized, family-weighted 44-D vectors (not the 3-D positions, so
the grouping reflects the full fingerprint). Each song is assigned to the nearest
of 6 cluster centers, and each cluster gets one of six jewel-toned colors. These
are the "timbral families" you see in the default **Color → Cluster** mode —
loose neighborhoods of songs that share an overall sonic character.

You can also color by **Collection**, by **Artist**, or by several **graph
measures** (below).

---

## Step 7 — Edges: the "similar" and "different" webs

The lines between stars are a second view of the same distances:

- **Similar edges (kNN):** each star links to its nearest neighbors in the
  44-D space. "Connections per star" is a *maximum* — a star always keeps its
  single nearest link (so nothing is isolated), and additional links are only
  drawn if they're within a length limit (a multiple of the typical
  nearest-neighbor distance). Dense regions fill in; far-flung outliers stay
  sparse. Both knobs are live in the Sky's **Edges** menu.
- **Different edges (k-farthest):** each star links to the songs *least* like it
  — a way to deliberately leap across the constellation.
- **Per-collection constellations:** when you filter the Sky to one collection,
  it shows that collection's own connected graph (a minimum spanning tree plus
  nearest-neighbor links), guaranteed to span the whole collection.

**Walks** (the auto-playlists) are computed directly in the 44-D space, *not*
along these drawn edges — so changing the edge settings reshapes the visual web
without changing how a walk steps.

---

## Bonus — Graph-structure colorings

Beyond cluster/album/artist, the **Color** menu can paint stars by where they sit
in the *similarity graph* — useful for finding structurally interesting songs.
These are computed on-device over the similarity edges:

- **Weighted degree** — local hubs (songs many others are near).
- **Betweenness** — bridges that connect otherwise-separate regions.
- **Closeness** — how central a song is to the whole graph.
- **Clustering coefficient** — how tight-knit a song's immediate neighborhood is.
- **Influence (PageRank)** — songs linked to other well-connected songs.

---

## What this is — and isn't

- It's a map of **acoustic similarity**, derived from signal features. Two songs
  near each other share measurable sonic qualities (timbre, harmony, rhythm,
  brightness, dynamics).
- It is **not** a genre classifier or a taste model. It doesn't know what a song
  is "about," who made it, or whether you'll like it — only how it sounds.
- The current 44-feature set is a deliberate, compact starting point. Expanding
  and refining it (and making the analysis faster) is ongoing work.

## Privacy

All of the above — decoding, feature extraction, standardization, UMAP, k-means,
graph metrics — runs **entirely on your device**. No audio, no features, and no
listening data are ever uploaded. See the
[privacy policy](privacy-policy.md) for details.
