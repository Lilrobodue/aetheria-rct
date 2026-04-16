# Aetheria RCT — Muse S Athena Integration

## For Claude Code — Drop-In Library Integration

**Goal:** Add Muse S Athena support to Aetheria RCT (aetheriarct.com) using the completed `athena-core.js` library, while keeping the existing Muse 2 support intact. Add a new fNIRS visualization panel. Keep the 27-state prescription engine unchanged (EEG-only for now).

**Reference files:**
- `athena-core.js` — the completed Athena library (copy this into the RCT repo)
- `athena-test.html` — the working test harness, useful for reference when wiring callbacks

---

## Critical Process Rules

**REPORT PROGRESS AT EVERY CHECKPOINT.** Stop and wait for user verification between checkpoints. The user (Dad) has the Athena and Muse 2 both available for testing.

**DO NOT MODIFY THE 27-STATE PRESCRIPTION ENGINE.** It's validated and working. The Athena feeds it the same log-PSD normalized band powers the Muse 2 feeds it. That code stays exactly as is.

**DO NOT BREAK EXISTING MUSE 2 SUPPORT.** Existing RCT users with a Muse 2 must still be able to connect and use the app. The Athena is ADDED, not a replacement.

**KEEP FNIRS DATA PARALLEL, NOT INTEGRATED.** Display fNIRS in a new panel. Export it in session data. But do NOT fold it into the prescription engine yet. That's future research.

---

## What to Build

### Checkpoint 1: Library Drop-In & Device Detection

**Implement:**
1. Copy `athena-core.js` into the RCT repo at the same directory level as the existing HTML file
2. Add `<script src="athena-core.js"></script>` in the HTML head, before the existing app code
3. Find the existing "Connect Muse" button handler in the RCT HTML (search for `requestDevice`, `MuseClient`, or `navigator.bluetooth`)
4. Add a device type selector UI — a small dropdown or two buttons near the Connect button:
   - "Muse 2 (EEG)"
   - "Muse S Athena (EEG + fNIRS)"
5. Route the Connect button to either the existing Muse 2 code path or the new `AthenaDevice` code path based on the selector
6. Do NOT write the Athena code path yet — just the routing. When Athena is selected, show a "Not yet implemented" placeholder.

**Test:**
User confirms:
- The RCT app still loads without errors
- The existing Muse 2 connection still works when Muse 2 is selected
- The new device selector appears in the UI
- Selecting Athena shows the placeholder message

**Report and STOP.**

---

### Checkpoint 2: Athena Connection & EEG Pipeline

**Implement:**
1. Replace the Athena placeholder with a real `AthenaDevice` instance
2. Wire these callbacks to the existing RCT data handlers:
   - `onBandPowers` → feed into the existing `getAetheriaPosition()` function (the 27-state prescription engine). The band powers come already log-PSD normalized in the format `{delta, theta, alpha, beta, gamma}` as relative percentages. This is the same format the existing engine expects.
   - `onEEG` → feed into the existing EEG waveform display (if present) or ignore for now
   - `onStatus` → update the existing connection status UI ('disconnected' / 'connecting' / 'connected' / 'streaming')
3. Call `muse.connect()` when the user clicks Connect with Athena selected
4. Call `muse.disconnect()` when the user clicks Disconnect

**Critical:** The band powers coming from `athena-core.js` are already log-PSD normalized. Do NOT apply any additional normalization or noise injection. Feed them directly into the prescription engine.

**Test:**
User connects the Muse S Athena, puts it on, and confirms:
- Connection status updates correctly
- The existing RCT coherence gauge updates in real time
- The existing Aetheria position display shows positions 1-27 based on brain state
- Eye blinks and jaw clenches cause visible EEG response (if EEG waveforms are displayed)
- Switching back to Muse 2 mode still works after using Athena

**Report and STOP.**

---

### Checkpoint 3: fNIRS Visualization Panel

**Implement:**
1. Add a new collapsible panel below the existing coherence display, titled "Hemodynamic Response (fNIRS)"
2. This panel is only visible when connected to an Athena (hide it for Muse 2 connections)
3. The panel displays 4 optode positions in a 2x2 grid matching head anatomy:
   ```
   [ Left Inner  ] [ Right Inner  ]
   [ Left Outer  ] [ Right Outer  ]
   ```
4. Each optode cell shows:
   - Optode label (LI, RI, LO, RO)
   - Current HbO value (oxygenated hemoglobin, in µM)
   - Current HbR value (deoxygenated hemoglobin, in µM)
   - SQI (signal quality indicator, 0-1) — display as a colored dot: red < 0.3, yellow 0.3-0.7, green > 0.7
   - A small sparkline showing the last 30 seconds of HbDiff (HbO - HbR), which is the standard neural activity proxy
5. Wire the `onFNIRS` callback to update this panel
6. Match the existing RCT aesthetic (dark background, warm gold accents, Cormorant Garamond font for headings, monospace for values)

**Test:**
User wears the Athena, confirms:
- The fNIRS panel appears only when connected to Athena
- The 4 optode positions update with real values
- SQI colors reflect actual signal quality (LI/RI usually green, LO/RO often yellow)
- Sparklines update smoothly
- The panel does not appear when connected to Muse 2

**Report and STOP.**

---

### Checkpoint 4: Session Export Update

**Implement:**
1. Find the existing RCT session export code (CSV or JSON export, look for `Blob`, `download`, or `export` keywords)
2. When exporting from an Athena session, include the new fNIRS data alongside the existing EEG/coherence data:
   - HbO, HbR, HbDiff for each of the 4 optodes
   - Heart rate from PPG
   - Timestamps matched to existing EEG timestamps
3. When exporting from a Muse 2 session, the export format stays unchanged (backward compatibility)
4. Add a field to the export metadata identifying the device type ("muse-2" or "muse-s-athena")

**Test:**
User runs a 2-minute Athena session, exports it, confirms:
- The export file contains fNIRS data alongside EEG data
- Timestamps align properly
- Muse 2 exports still work in the original format

**Report and STOP.**

---

### Checkpoint 5: Final Polish & Documentation

**Implement:**
1. Add a brief help tooltip or info icon near the device selector explaining the difference between Muse 2 and Athena
2. Add a small "fNIRS data is experimental and for research purposes only" disclaimer above the fNIRS panel
3. Update the RCT app version number if one exists
4. Verify all 6 tabs of the RCT app still function correctly with both device types

**Test:**
User does a full end-to-end test:
- Open RCT
- Select Muse 2, connect, run a session, export, disconnect
- Select Athena, connect, run a session, export, disconnect
- Verify no console errors throughout

**Report complete.**

---

## What NOT to Do

- Do NOT modify the 27-state prescription engine or log-PSD normalization code
- Do NOT integrate fNIRS into the Aetheria position calculation (future research)
- Do NOT remove or break Muse 2 support
- Do NOT add new tabs or major UI sections beyond the fNIRS panel
- Do NOT modify `athena-core.js` (it's the canonical library — if bugs appear, report them and we fix in the library, not in the integration)
- Do NOT add signal processing code to the RCT file — the library handles all DSP

---

## Success Criteria

Integration is complete when:
1. ✅ Both Muse 2 and Athena devices can connect and stream to RCT
2. ✅ The 27-state prescription engine works identically with both devices
3. ✅ fNIRS data is displayed in a dedicated panel when Athena is connected
4. ✅ Session exports include fNIRS data for Athena sessions
5. ✅ No regressions in existing Muse 2 functionality
6. ✅ The existing RCT aesthetic is maintained throughout
