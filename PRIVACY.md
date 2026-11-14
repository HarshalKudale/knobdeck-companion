# Privacy

The KnobDeck companions (the VS Code extension and the JetBrains IDE plugin):

- **collect nothing and send nothing.** They make no network requests, have no analytics, no
  telemetry and no crash reporting.
- **listen only on your own computer** (`127.0.0.1`), so other devices on your network can't reach
  them, and they refuse requests from web pages.
- **answer only KnobDeck**, which asks them to run editor commands and to list your tasks, launch or
  run configurations, terminals and open files so you can pick one on a button. That information
  goes no further than KnobDeck on the same computer.
- **store only their own settings** (port, optional password, on/off) in the editor's settings.

The password, if you set one, is stored in plain text in the editor's settings and in KnobDeck's
plugin settings on the same computer. It stops other programs on your computer from using the
companion; it isn't meant to protect anything beyond that.

For KnobDeck itself see the [KnobDeck privacy policy](https://knobdeck.harshalkudale.com/legal/privacy).
Questions: support@harshalkudale.com.
