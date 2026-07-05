# SPECTRUM v0.2: The Fieldworthy Release

*Field Instrument 064 grows real filters, hears real shortwave, and tunes like it means it.*

Version 0.1 of SPECTRUM was a foundation: a browser-native SDR console that could see the spectrum and make sounds. Version 0.2 is the release where it becomes a receiver you would actually sit with.

The headline is filtering. The Foundation release used the simplest possible channel filters, and simple filters leak. A strong SSB station five kilohertz up the band would bleed straight into your passband. The whole chain is now FIR: a windowed-sinc anti-alias decimator brings the sample stream down from megasamples to audio rate, and 63-tap channel filters shape the passband for each mode. The difference is not subtle. On the bench, a signal 5 kHz away from the 2.4 kHz USB filter is now 55 dB down. That is the difference between a crowded band being a wall of noise and being a row of separate conversations.

Second: direct sampling. The RTL-SDR's tuner does not reach below 24 MHz, which is inconvenient when everything interesting to me lives below 24 MHz. Dongles with the direct sampling modification route the antenna straight into the ADC, and SPECTRUM now speaks that mode: set it to Auto on the Radio page and the driver switches itself over whenever you tune below 24 MHz, retuning with the demodulator's digital downconverter instead of the tuner PLL. Forty meters without an upconverter.

Third: your hands. Tuning gestures landed on both scopes. Click to jump, drag to slew across the band like turning a weighted knob, scroll to step, and Ctrl+scroll to zoom the display from the full span down to a sixteenth of it, where a single CW signal becomes a bright ribbon you can center by eye.

The release gate grew with the release. Three new self-tests verify the FIR designer's stopband, the decimator's alias rejection, and the zoom geometry, eleven lamps total, all green. The adjacent-channel measurement above came from the same harness that ships in the System page.

The hardware driver, direct sampling included, remains experimental until it logs real air time. The simulator remains the proving ground. But the gap between the two keeps shrinking.

*Make. Hack. Learn. Share. Repeat.*
