# v0.4.4 Test Report

Browser-runnable self-test harness (System page), 19 tests:

- FFT: 1 kHz tone lands in the right bin: PASS
- FFT: Parseval energy within 1%: PASS
- Maidenhead: home grid round trip: PASS
- Bearing: Cumberland to London: PASS
- Morse: encode/decode agree for full alphabet: PASS
- Store: JSON round trip preserves memories: PASS
- Colormap: 256 entries, warm end: PASS
- FIR design: unity DC gain, 40 dB stopband: PASS
- FIR decimator: passband survives, alias down 40 dB: PASS
- Zoom: visible range centers and scales: PASS
- WAV: header and parser round trip: PASS
- Replay: FileSource reads samples and filename frequency tag: PASS
- Commissioning: calibration offset overrides default: PASS
- Squelch: opens on SNR, not absolute level: PASS
- Themes: every text pair meets WCAG AA in all three themes: PASS
- CW confidence: 80 ms unit reads 15 WPM: PASS
- Band plan: segments ordered, typed, and non-degenerate: PASS
- RTL driver: FIR table repacks to librtlsdr defaults: PASS
- Frequency formatter: PASS

Bench tests (Node harness with DOM stubs):

- Pipeline smoke: sim beacon 57.8 dB SNR, all demodulators finite: PASS
- CW decoder: copies "CQ DE N1HNP" from synthetic keying: PASS
- Selectivity: 55.3 dB adjacent rejection at +5 kHz, USB 2.4 kHz filter: PASS
- CW sidetone: 599 Hz measured against 600 Hz target: PASS
- Connection toggle state machine: PASS
- Record/replay round trip: 1 s sim capture to WAV, replayed beacon within
  0.3 dB of live SNR (57.5 vs 57.8 dB): PASS
- Gain-invariant squelch: 20 dB gain change moves absolute level 20.0 dB,
  SNR decision under 2 dB; old dBFS scheme flips at the same threshold: PASS
- Sim broadcast: AM 1010 kHz, WFM 98.1 MHz, NFM 162.550 MHz all produce
  finite audio with real SNR through their matching demodulators: PASS
- Demod at hardware rate: WFM/AM/USB at 2.4 MS/s each produce exactly
  the expected 48k samples per second of RF at 12-14% CPU: PASS
- Full onIQ path with 64 KB transfer blocks: 1 s of 2.4 MS/s FM through
  ring, capture hook, and demod in 174 ms: PASS

Hardware telemetry trail (S/N oUKs9mTRIMlx3j47): v0.4.1 restored
streaming; v0.4.3 instrumentation showed buf 2% with drops climbing,
convicting the single-transfer USB pump of continuous FIFO overflow;
v0.4.4 streams with 8 concurrent 64 KB transfers. Awaiting retest.

Hardware status: S/N oUKs9mTRIMlx3j47. v0.4.1 fixed streaming (waterfall
and S-meter confirmed live on air). v0.4.2 rebuilds the audio sink on
AudioWorklet after RF-works-but-silent was reported; the old zero-input
ScriptProcessorNode is a known silent-failure configuration in Chrome.
Awaiting audio retest.
