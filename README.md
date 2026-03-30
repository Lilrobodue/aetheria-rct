# Aetheria: Resonant Coherence Training

**A Muse 2 EEG neuro-adaptive frequency healing PWA**

Brain-driven cymatics, biofield visualization, and coherence training powered by the Aetheria 27-frequency architecture.

## What It Does

Connect a Muse 2 EEG headband via Web Bluetooth. The app reads your brainwaves in real-time and:

- **Diagnoses your biofield state** across 27 distinct neurological states
- **Prescribes the Aetheria frequency** your nervous system needs right now
- **Plays the healing tone** (432Hz Solfeggio-based) that shifts as your brain changes
- **Visualizes cymatics** — Chladni patterns on 3D sphere, torus, or flat plate
- **Shows 4-channel EEG** — live oscilloscope with coherence convergence effect
- **Captures your Aetheria frequency signature** — before/after brain fingerprint
- **Records raw EEG data** — export CSV/JSON for research analysis

## The 27-Frequency System

27 frequencies across three regimes, all based on 432Hz tuning and Solfeggio foundations. Every frequency reduces to a digital root of 3, 6, or 9.

| Regime | Frequencies | Purpose |
|--------|------------|---------|
| **GUT** (1-9) | 174Hz – 963Hz | Body, grounding, physical healing |
| **HEART** (10-18) | 1206Hz – 3150Hz | Emotion, relationship, flow |
| **HEAD** (19-27) | 3504Hz – 6336Hz | Mental clarity, mastery, transcendence |

Each frequency is mapped to:
- An I Ching hexagram
- A sacred geometry form
- A Techgnosis guided meditation script
- A diagnostic brain state

## Technical Architecture

Following [Perplexity's pipeline analysis](https://perplexity.ai):

```
Muse BLE → onMuseData() → prescribe() → audio.setFrequency() → updateUI()
```

- **AudioEngine**: Singleton, never destroyed. Smooth frequency transitions via `linearRampToValueAtTime`
- **NeuroState**: Central state object updated by Muse callback
- **Prescription Engine**: 27-state diagnostic using log-PSD normalized band power
- **Band Power**: Power Spectral Density per IFCN standards, log-normalized for display
- **UI Updates**: Targeted `getElementById` — `render()` only on tab switch

## Requirements

- A **Muse 2** EEG headband (connects via Web Bluetooth)
- A browser with **Web Bluetooth** support (Chrome, Edge, Opera — not Firefox/Safari)
- Works on desktop and Android. iOS does not support Web Bluetooth.

## Installation

This is a Progressive Web App. Visit the hosted page, then:
- **Android**: Tap "Add to Home Screen" in browser menu
- **Desktop**: Click the install icon in the address bar

## Usage

1. Open the app
2. Tap **Connect** on the Muse bar
3. Select your Muse 2 from the Bluetooth picker
4. The system auto-starts: reads brainwaves → prescribes frequency → plays tone
5. Explore tabs: Home (dashboard), Cymatics (3D visualizer), Biofield (field map), Flower (3-6-9 meditation), History (sessions + export)

## Data Export

- **Session History**: Export JSON/CSV with coherence stats, prescription paths, and before/after signatures
- **Raw EEG Recording**: Start/stop recording captures every sample (~4Hz) with all 5 bands, coherence, and prescription state

## The Science

- **EEG Band Power**: Computed via FFT on 256-sample windows, Hanning-windowed, PSD-normalized per IFCN standards
- **Coherence**: Alpha+Theta power relative to total power (all four Muse channels averaged)
- **Prescription**: Log-PSD normalized bands fed into 27-state diagnostic tree with 10-second hold timer
- **Frequency Signature**: 15-second averaged brainwave capture with continuous position scoring (not bucket-based, can't be gamed)

## Credits

Created by **Joseph (Jobo) Lewis** — Air Force EOD veteran, TBI survivor, creator of the Aetheria frequency healing system.

Built with love by the Lewis family: Jobo, Alisha, and Selah.

**Website**: [Aetheria432.com](https://aetheria432.com)

## License

This project is licensed under the **GNU General Public License v3.0** — free to use, study, modify, and share. Any derivative work must also be open source under GPL v3. See [LICENSE](LICENSE) for details.

The Aetheria frequency system, Techgnosis scripts, and associated intellectual property are © Joseph Lewis / Aetheria.

💜 Healing the world heART
