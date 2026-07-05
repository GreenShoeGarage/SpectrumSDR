# SPECTRUM: A Radio Console That Lives in a Browser Tab

*Field Instrument 064, v0.1.0 Foundation Release*

There is a particular pleasure in plugging a thirty-dollar dongle into a laptop and hearing the whole HF spectrum pour out of it. What has always bothered me is the software side of that ritual. Most SDR applications are installs, drivers, dependency chains, and someone else's idea of a user interface. I wanted the opposite: one HTML file that turns the browser itself into the receiver.

SPECTRUM is that file. Open it in Chrome, click Connect, pick the RTL-SDR from the browser's own device chooser, and the dongle starts streaming IQ samples straight into JavaScript over WebUSB. No server. No install. The FFT, the waterfall, the demodulators, the S-meter needle, all of it runs in the tab. Close the tab and nothing is left behind except your settings and logbook, which stay in localStorage where they belong: on your machine.

The instrument wears Signal Corps olive drab, because a field radio should look like a field radio. Riveted panels, stencil plates, a phosphor-green frequency flag, a proper needle meter with the red region starting at S9.

Eight pages cover the workflow: a Dashboard for tuning and memories, a Signals page that lists the strongest carriers in the span and tunes when you click one, a Decode page with a working Morse copier, an RF Lab with a raw IQ constellation, a Journal that captures frequency and mode with every entry, and a Geospatial page that speaks Maidenhead.

The part I am proudest of is the simulator. Click Simulator instead of Connect and SPECTRUM generates a believable stretch of HF: an AM broadcaster, two-tone SSB, bare carriers, static crashes, and a CW beacon on 7.050 endlessly keying VVV DE N1HNP at fifteen words per minute. Tune it in CW mode and the Decode page copies it letter by letter. Every part of the instrument can be learned, tested, and demonstrated with no antenna and no hardware. That is also how the release gate works: the System page carries a self-test harness, and all lamps must be green before a version ships.

The hardware driver speaks the librtlsdr control protocol directly: baseband init, tuner PLL programming over I2C, the whole handshake. I am calling it experimental until it has logged real air time, and the README says so plainly. The filters are soft, WFM is mono, and RTTY is still a promise. That is what a Foundation release is for.

Owning your data and understanding your tools is worth the learning curve. This one runs anywhere a browser does.

*Make. Hack. Learn. Share. Repeat.*
