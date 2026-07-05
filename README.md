# SPECTRUM

**Field Instrument 064. A WebSDR Studio.**

SPECTRUM turns a Chrome or Edge browser into a standalone software-defined radio console for the RTL-SDR. No server, no install, no build step. The browser drives the dongle directly over WebUSB, demodulates in JavaScript, and paints the spectrum and waterfall on canvas. A built-in simulator provides a believable stretch of HF so every page can be exercised with no hardware attached.

Aesthetic: U.S. Army Signal Corps field equipment. Olive drab panels, rivets, stencil plates, phosphor readouts.

## Quick start

1. Open `index.html` in Chrome or Edge (any modern browser works for the simulator).
2. Click **SIMULATOR**. The spectrum and waterfall come alive with synthetic stations.
3. Click the headphone button in the Audio panel and raise **VOLUME**. Browsers require a click before audio can start.
4. Tune to **7.050.000** in **CW** mode. You will hear a keyed beacon sending `VVV DE N1HNP`. Open the **Decode** page and watch it copy.
5. Tune with gestures: click the spectrum or waterfall to jump, drag to slew, scroll to step, and Ctrl+scroll (or the Zoom buttons) to zoom from 1x to 16x.
6. With an RTL-SDR plugged in, click **CONNECT USB**. WebUSB requires HTTPS or localhost, and no other SDR application may hold the dongle.

## Pages

| Page | Purpose |
|---|---|
| Dashboard | Realtime spectrum, waterfall, frequency keypad, band buttons, memory channels |
| Radio | Demodulator, RF gain, sample rate, PPM correction |
| Signals | Strongest carriers in the span (click to tune), live measurements at the tuned frequency |
| Decode | Morse (CW) decoder with adjustable threshold |
| RF Lab | Raw IQ constellation, FFT settings, IQ and audio recording to WAV, replay of IQ recordings |
| Journal | Local logbook with automatic frequency and mode capture, CSV export |
| Geospatial | Station position, Maidenhead grid, bearing and distance to a target grid |
| System | Self-test harness, five-step hardware commissioning checklist, JSON export and import, environment report, debug flag |

## Simulated stations

| Frequency | Content |
|---|---|
| 7.050.000 | CW beacon, `VVV DE N1HNP`, ~15 WPM |
| 14.100.000 | Bare carrier |
| 14.230.000 | AM broadcast with two-tone modulation |
| 14.265.000 | USB two-tone (700 Hz and 1900 Hz) |
| 21.300.000 | CW beacon |
| 28.400.000 | USB two-tone |

## Architecture

- **Single file, no build step.** All HTML, CSS, and JavaScript live in `index.html`. Fonts are the only external fetch.
- **Local-first.** Settings, memories, and journal entries live in `localStorage` under the key `spectrum-fi064`. System page exports and imports everything as JSON. Journal entries are archived (soft delete), never destroyed until an explicit export.
- **Sources are interchangeable.** `SimSource` and `RtlSource` share one interface: `rate`, `setCenter()`, `start()`, `stop()`, and an `onBlock(iq)` callback delivering interleaved Float32 IQ. Adding an `rtl_tcp` WebSocket source later means one new class.
- **DSP.** Iterative radix-2 FFT with Hann window and power averaging. The channel chain is FIR throughout: a windowed-sinc anti-alias decimator from the source rate to ~48 kHz, then 63-tap channel filters at half the mode bandwidth. Demodulators: AM (envelope with DC block), NFM (quadrature discriminator), WFM (two-stage FIR decimation with deemphasis), USB and LSB (Weaver method with FIR arms), CW (600 Hz BFO, narrow Weaver). Measured adjacent-signal rejection at +5 kHz on the 2.4 kHz USB filter: 55 dB. Audio flows through a ring buffer to a ScriptProcessorNode with linear resampling.
- **Recording and replay.** IQ capture writes standard 2-channel 16-bit WAV (I left, Q right) with an SDR-style frequency tag in the filename, interchangeable with SDR# and similar tools. Audio capture records the demodulated output at line level. Replay feeds any IQ WAV back through the full pipeline, so signals become shareable and bug reports reproducible. Bench-verified round trip: a recorded beacon replays within 0.3 dB of live SNR.
- **Commissioning.** The System page carries a five-step checklist in the SHAKEDOWN spirit: verify device, measure frequency error against a known reference, walk the gain range, calibrate the S-meter against a known level, and mark commissioned. The calibration record (serial, date, PPM, dBm offset, gain curve) persists in the store and travels with the JSON export; a stored offset replaces the default S-meter constant everywhere.
- **RTL driver.** Implements the librtlsdr control protocol over WebUSB for the RTL2832U with an R820T or R820T2 tuner: baseband init, demod register writes, I2C tuner access, PLL programming, coarse gain mapping.

## Test harness

The System page carries the release gate. **Run Self Test** exercises the FFT (bin placement and Parseval energy), Maidenhead round trips, great-circle bearing, the full Morse table, store serialization, the waterfall colormap, and the frequency formatter. All lamps must be green before a release ships.

## Known limitations

- **The RTL-SDR hardware driver is experimental.** It was validated against the simulator pipeline and the librtlsdr protocol documentation, not against physical hardware. First hardware session should be treated as commissioning: expect to touch tuner gain mapping and PLL edge cases. Only the R820T and R820T2 tuners are recognized.
- WFM is mono, with approximate deemphasis. No stereo pilot decoding.
- The S-meter uses a rough default offset (-37) until the commissioning checklist stores a per-dongle calibration. PPM sign convention on step 2 is unverified against hardware; the checklist says to re-measure after applying and flip the sign if the error grows.
- ANF (automatic notch filter) is a placeholder button. NB and NR are live but simple.
- Decode covers CW only. RTTY and FT8 are roadmap items.
- IQ recordings live in browser memory until downloaded; the limit control caps capture at 60 seconds (roughly 96 MB at 2.4 MS/s per 10 seconds).
- ScriptProcessorNode is deprecated in favor of AudioWorklet. It still works everywhere; migration is planned.
- Direct sampling is experimental like the rest of the hardware driver. Q branch matches the common dongle modification; stock dongles without the mod hear HF only weakly. The tuner is not powered down in direct mode, so a faint birdie from its LO is possible. Meaningful coverage is 0 to 14.4 MHz; above that the ADC serves aliases.

## License

GPL-3.0

## Changelog

- **v0.3.0 Shakedown.** IQ recording to standard 2-channel WAV with SDR-style frequency-tagged filenames, audio recording at line level, and replay of any IQ WAV through the complete receiver (spectrum, demodulators, decoder) with the dial following the filename tag. Five-step commissioning checklist on the System page: device verification, frequency error measurement against a reference with PPM apply, automated gain walk, S-meter calibration against a known level, and a persistent commissioning record that overrides the default calibration everywhere. Three new self-tests (WAV round trip, FileSource replay, calibration override), 14 total.

- **v0.2.0 Fieldworthy.** The receiver grew real filters: the boxcar decimator and one-pole channel filters were replaced with a windowed-sinc FIR chain (anti-alias decimation plus 63-tap channel selectivity), measured at 55 dB adjacent-signal rejection where the old chain leaked freely. Direct sampling support for HF below 24 MHz (Auto, Off, I, or Q branch on the Radio page) with DDC tuning, so modified dongles hear shortwave without an upconverter. Tuning gestures: drag to slew, scroll to step, Ctrl+scroll or Zoom buttons for 1x to 16x span zoom on both scopes. Three new self-tests cover FIR design, decimator alias rejection, and zoom geometry (11 total).

- **v0.1.2** Connect USB and Simulator are now start/stop toggles. While a source runs, its button turns red and reads Disconnect or Stop Sim. Disconnect no longer reloads the page, and an intentional stop no longer reports a lost stream.

- **v0.1.1** Footer now sits at the true bottom of the page instead of overlapping content (the layout was pinned to viewport height). Left menu is collapsible to an icon rail via the Collapse Menu control, and the state persists across sessions.
- **v0.1.0** Foundation release.
