# SPECTRUM v0.4: The Field Manual Release

*Field Instrument 064 answers to its own standards.*

I keep a written set of development standards for the Field Instrument series, and SPECTRUM had been quietly violating two of them since the day it was born. The standard says night-default theming with a day mode, contrast verified in every theme. The standard says an in-app feedback mechanism. SPECTRUM had one theme and no feedback path. Version 0.4 is the release where the instrument comes into compliance, which is why it is named for the field manual.

The theming work went further than adding a light stylesheet. All three themes, Night, Day in field-manual paper, and High Contrast, live in a single JavaScript table, and that table is read by the same self-test harness that gates every release. There is now a test that computes the WCAG contrast ratio of every text pair in every theme and fails the build if any pair drops below AA. A color choice that looks nice but reads poorly is no longer an opinion; it is a red lamp. The scopes and meters keep their dark instrument faces in all three themes, because a spectrum analyzer with a white screen is a whiteboard.

Feedback is a form on the System page that composes a proper field report, version, source, tuned frequency, mode, theme, browser, alongside whatever you observed, and either opens a prefilled GitHub issue or copies the report to the clipboard. A bug report that arrives with its context attached is half diagnosed.

The learning feature of the release is the band plan strip: a thin ribbon along the top of the spectrum, blue where CW and data live, green for phone, amber for broadcast, teal for NOAA weather. Zoom in and the ribbon zooms with you. After a while you stop reading it and start knowing it, which was the point.

The simulator grew a broadcast band to match last week's buttons: an AM music-box station on 1010, a wideband FM broadcaster on 98.1, and a synthetic voice cadence on the weather frequency, all verified through their matching demodulators on the bench. The Morse decoder now reports its confidence, estimated words per minute, timing unit, and element counts, so failed copy is a diagnosis instead of a mystery. Eighteen lamps on the release gate now. All green.

*Make. Hack. Learn. Share. Repeat.*
