# SPECTRUM v0.3: The Shakedown Release

*Field Instrument 064 learns to remember what it heard, and to prove what it measures.*

Two things arrived in v0.3, and they share a philosophy: a measurement you cannot reproduce is an anecdote.

First, recording. SPECTRUM now captures raw IQ to a standard 2-channel WAV, I in the left channel and Q in the right, the same convention SDR# and half the SDR world already speak. The filename carries the frequency the way those tools expect, so a capture from SPECTRUM opens anywhere, and a capture from anywhere opens in SPECTRUM. Because the replay path feeds the file back through the entire receiver, spectrum, waterfall, demodulators, Morse decoder and all, a strange signal stops being a story you tell and becomes a file you hand someone. On the bench, a recorded beacon replays within a third of a decibel of its live signal-to-noise ratio. The demodulated audio records too, at line level, untouched by the volume knob.

Second, commissioning. The hardware driver has been honest about being experimental since v0.1, and v0.3 finally gives it the instrument it needs to graduate: a five-step checklist on the System page, run in order with a dongle attached and something trustworthy on the air. Verify the device. Tune a known reference like WWV and measure the frequency error down to the hertz, then apply the PPM correction. Walk the gain range and record the noise floor at each step. Feed it a signal of known level and calibrate the S-meter, at which point the dBm readout stops being a polite fiction and starts being a measurement. Mark it commissioned, and the record, serial number, date, correction, calibration, gain curve, lives in the store and travels with your data export.

Anyone who has commissioned a building will recognize the shape of this. You do not trust the mechanical schedule; you trust the functional test with your initials on it. A thirty-dollar dongle deserves the same courtesy.

Fourteen self-test lamps now, all green, including a WAV round trip and a replay test that runs a synthetic recording end to end. The simulator remains the dry-run path for every step, marked as such, so the whole checklist can be rehearsed before the dongle ever leaves the drawer.

*Make. Hack. Learn. Share. Repeat.*
