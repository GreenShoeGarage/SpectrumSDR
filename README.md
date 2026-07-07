# SPECTRUM

**Field Instrument 064. A WebSDR Studio.**

SPECTRUM turns a Chrome or Edge browser into a standalone software-defined radio console for the RTL-SDR. No server, no install, no build step. The browser drives the dongle directly over WebUSB, demodulates in JavaScript, and paints the spectrum and waterfall on canvas. A built-in simulator provides a believable stretch of HF so every page can be exercised with no hardware attached.

Aesthetic: U.S. Army Signal Corps field equipment. Olive drab panels, rivets, stencil plates, phosphor readouts. Three themes cycle from the header: Night (default), Day (field-manual paper), and High Contrast. Every text pair in every theme is held to WCAG AA by the self-test harness; a theme change that breaks contrast fails the release gate. Scopes and meters keep dark instrument faces in all themes.

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
- **DSP.** Iterative radix-2 FFT with Hann window and power averaging. The channel chain is FIR throughout: a windowed-sinc anti-alias decimator from the source rate to ~48 kHz, then 63-tap channel filters at half the mode bandwidth. Demodulators: AM (envelope with DC block), NFM (quadrature discriminator), WFM (two-stage FIR decimation with deemphasis), USB and LSB (Weaver method with FIR arms), CW (600 Hz BFO, narrow Weaver). Measured adjacent-signal rejection at +5 kHz on the 2.4 kHz USB filter: 55 dB. Audio flows to an AudioWorklet sink (loaded from a Blob URL, no build step) with linear resampling off the main thread; a repaired ScriptProcessor fallback covers browsers without worklet support. The Audio panel shows a live chain status line: sink type, context state, sample rate, volume, and buffer fill.
- **Recording and replay.** IQ capture writes standard 2-channel 16-bit WAV (I left, Q right) with an SDR-style frequency tag in the filename, interchangeable with SDR# and similar tools. Audio capture records the demodulated output at line level. Replay feeds any IQ WAV back through the full pipeline, so signals become shareable and bug reports reproducible. Bench-verified round trip: a recorded beacon replays within 0.3 dB of live SNR.
- **Commissioning.** The System page carries a five-step checklist in the SHAKEDOWN spirit: verify device, measure frequency error against a known reference, walk the gain range, calibrate the S-meter against a known level, and mark commissioned. The calibration record (serial, date, PPM, dBm offset, gain curve) persists in the store and travels with the JSON export; a stored offset replaces the default S-meter constant everywhere.
- **RTL driver.** Implements the librtlsdr control protocol over WebUSB for the RTL2832U with an R820T or R820T2 tuner: baseband init, demod register writes, I2C tuner access, PLL programming, coarse gain mapping.

## Test harness

The System page carries the release gate. **Run Self Test** exercises the FFT (bin placement and Parseval energy), Maidenhead round trips, great-circle bearing, the full Morse table, store serialization, the waterfall colormap, and the frequency formatter. All lamps must be green before a release ships.

## Known limitations

- **The RTL-SDR hardware driver is experimental but has had first hardware contact.** v0.4.1 corrected the register map against librtlsdr after a real dongle connected but streamed nothing (wrong USB register addresses, zeroed FIR coefficients, missing I2C repeater, shifted tuner init table, PLL packing). The connection panel now shows live MB/s so streaming health is visible at a glance. Only the R820T and R820T2 tuners are recognized.
- WFM is mono, with approximate deemphasis. No stereo pilot decoding.
- The S-meter uses a rough default offset (-37) until the commissioning checklist stores a per-dongle calibration. PPM sign convention on step 2 is unverified against hardware; the checklist says to re-measure after applying and flip the sign if the error grows.
- ANF (automatic notch filter) is a placeholder button. NB and NR are live but simple.
- The band plan strip covers simplified US full-license extents plus broadcast and NOAA segments; it is a reading aid, not an operating authority.
- Decode covers CW only. RTTY and FT8 are roadmap items.
- IQ recordings live in browser memory until downloaded; the limit control caps capture at 60 seconds (roughly 96 MB at 2.4 MS/s per 10 seconds).
- Direct sampling is experimental like the rest of the hardware driver. Q branch matches the common dongle modification; stock dongles without the mod hear HF only weakly. The tuner is not powered down in direct mode, so a faint birdie from its LO is possible. Meaningful coverage is 0 to 14.4 MHz; above that the ADC serves aliases.

## License

GPL-3.0

## Changelog

- **v0.4.3** Audio chain made observable and bisectable while chasing RF-works-but-silent on hardware. The sink now primes 100 ms of depth before speaking and re-primes on underrun, so buffer fill reads a meaningful 7-12% when healthy instead of hovering at 0% by design. The status line adds a demodulator output-rate counter (should read ~48.0k sps) and an underrun counter. New TONE button injects a 600 Hz test tone inside the audio sink itself, bypassing the entire receiver: tone audible means the sink, browser, and OS path are good and the problem is upstream; tone silent means the problem is the browser or OS output, not SPECTRUM's receiver. Bench note: the demod chain measures 12-14% CPU at 2.4 MS/s producing exactly 48k samples per second; an earlier "too slow for realtime" bench verdict was the test's own signal generator being timed, not the demodulator.

- **v0.4.2** Audio path rebuilt after hardware testing reached RF-works-but-silent. The old sink was a ScriptProcessorNode created with zero input channels, a configuration Chrome is known to silently never fire; the deprecated API is now replaced by an AudioWorklet sink loaded from a Blob URL (resampling off the main thread, buffer fill reported back), with a repaired one-input ScriptProcessor fallback. The Audio panel gains a live status line (sink, context state, sample rate, volume, buffer fill) so the audio chain is observable instead of a black box. Demod rate changes now propagate to the sink.

- **v0.4.1** Driver hotfix from first hardware contact (dongle connected, streamed nothing). Corrected the USB/SYS register map (SYSCTL 0x2000, EPA_CTL 0x2148, EPA_MAXPKT 0x2158, DEMOD_CTL 0x3000/0x300b), wrote the real librtlsdr default FIR table where the old init had zeroed the coefficients and silenced the ADC path, bracketed all tuner access with the I2C repeater, fixed a one-byte shift in the R820T init array that misaligned every register from 0x15 up, implemented proper PLL packing (ni/si into reg 0x14, SDM into 0x15/0x16) with a lock check and VCO-current retry, and added band-dependent tracking-filter mux. The connection panel now reports live stream MB/s, showing NO DATA in amber if the pipe goes quiet. New FIR-packing self-test, 19 total.

- **v0.4.0 Field Manual.** Standards compliance release. Three themes (Night, Day, High Contrast) driven from a single JS table so the self-test harness enforces WCAG AA on every text pair in every theme; scopes and meters keep dark instrument faces throughout. In-app feedback: a System page form that composes a field report (version, source, frequency, mode, theme, browser) and opens a prefilled GitHub issue or copies it. Band plan strip along the top of the spectrum: blue CW/data, green phone, amber broadcast, teal NOAA weather, drawn zoom-aware from simplified US extents. Simulator gains broadcast content: an AM music-box broadcaster on 1010 kHz, a wideband FM station on 98.1, and a synthetic NOAA voice cadence on 162.550, all bench-verified through their matching demodulators. CW decoder confidence line shows estimated WPM, unit length, and mark counts. Waterfall history now survives canvas resizes. New 25 kHz step for walking NOAA channels. Four new self-tests, 18 total.

- **v0.3.2** Band module gains US broadcast buttons: AM (530-1700 kHz, tunes to 1000 kHz), FM (87.9-107.9 MHz, tunes to 98.1), and WX (NOAA weather, tunes to 162.550). These three also switch the demodulator (AM, WFM, NFM respectively); the amateur band buttons remain tune-only so they never override a chosen mode. Tooltips carry the band ranges.

- **v0.3.1** Squelch now measures SNR above the noise floor instead of absolute dBFS, so the setting survives gain changes; the knob reads 0 (open) to +60 dB SNR and pre-0.3.1 stored values migrate to open. The in-filter SNR readout on the Signals page is now bin-count normalized (in-filter power compared against the noise the same number of bins would hold), so an empty channel reads near 0 dB instead of inheriting a bin-count offset; the CW decoder threshold rides the same normalized figure. Bench-verified: a 20 dB gain change moves the absolute level 20.0 dB but the SNR squelch decision under 2 dB, where the old scheme flips outright.

- **v0.3.0 Shakedown.** IQ recording to standard 2-channel WAV with SDR-style frequency-tagged filenames, audio recording at line level, and replay of any IQ WAV through the complete receiver (spectrum, demodulators, decoder) with the dial following the filename tag. Five-step commissioning checklist on the System page: device verification, frequency error measurement against a reference with PPM apply, automated gain walk, S-meter calibration against a known level, and a persistent commissioning record that overrides the default calibration everywhere. Three new self-tests (WAV round trip, FileSource replay, calibration override), 14 total.

- **v0.2.0 Fieldworthy.** The receiver grew real filters: the boxcar decimator and one-pole channel filters were replaced with a windowed-sinc FIR chain (anti-alias decimation plus 63-tap channel selectivity), measured at 55 dB adjacent-signal rejection where the old chain leaked freely. Direct sampling support for HF below 24 MHz (Auto, Off, I, or Q branch on the Radio page) with DDC tuning, so modified dongles hear shortwave without an upconverter. Tuning gestures: drag to slew, scroll to step, Ctrl+scroll or Zoom buttons for 1x to 16x span zoom on both scopes. Three new self-tests cover FIR design, decimator alias rejection, and zoom geometry (11 total).

- **v0.1.2** Connect USB and Simulator are now start/stop toggles. While a source runs, its button turns red and reads Disconnect or Stop Sim. Disconnect no longer reloads the page, and an intentional stop no longer reports a lost stream.

- **v0.1.1** Footer now sits at the true bottom of the page instead of overlapping content (the layout was pinned to viewport height). Left menu is collapsible to an icon rail via the Collapse Menu control, and the state persists across sessions.
- **v0.1.0** Foundation release.
