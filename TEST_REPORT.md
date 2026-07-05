# v0.3.0 Test Report

Browser-runnable self-test harness (System page), 14 tests:

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
- Frequency formatter: PASS

Bench tests (Node harness with DOM stubs):

- Pipeline smoke: sim beacon 57.8 dB SNR, all demodulators finite: PASS
- CW decoder: copies "CQ DE N1HNP" from synthetic keying: PASS
- Selectivity: 55.3 dB adjacent rejection at +5 kHz, USB 2.4 kHz filter: PASS
- CW sidetone: 599 Hz measured against 600 Hz target: PASS
- Connection toggle state machine: PASS
- Record/replay round trip: 1 s sim capture to WAV, replayed beacon within
  0.3 dB of live SNR (57.5 vs 57.8 dB): PASS

Hardware driver (WebUSB RTL2832U/R820T, including direct sampling) remains
experimental: validated against protocol documentation and the simulator
pipeline, not yet against physical hardware.
