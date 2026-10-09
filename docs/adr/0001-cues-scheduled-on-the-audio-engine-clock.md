---
status: accepted
---

# Cues are scheduled up front on the audio engine's clock

The app targets Firefox on Android, and the runner listens to music in another app with the screen off or the app in the background. When a Session starts, every Cue for the whole Session is scheduled at once on the Web Audio clock (`AudioBufferSourceNode.start(when)`). Cues are never triggered by JavaScript timers. Firefox delays timers on a hidden page, but the audio engine plays scheduled sounds on its own clock. Pause and Resume cancel the remaining Cues and reschedule them.

## Considered Options

Tested on a Galaxy S24 Ultra, Firefox 157 on Android 16, with Spotify playing, using `prototypes/cue-audio.html` on the `prototype/cue-audio` branch:

- **Separate `<audio>` clips started by timers**: rejected. While the page was hidden, Cues came 6–11 s late and in a burst when the page became visible again.
- **Web Audio started by timers**: passed, but only because Firefox apparently doesn't slow timers while the audio engine is running. We didn't want to rely on that undocumented behaviour.
- **Web Audio, all Cues scheduled up front**: passed, and doesn't depend on timers. Chosen.

## Consequences

- Firefox played Cues over the music at full volume and didn't lower or pause it, which is the behaviour we want. The earlier plan to keep every clip under about 5 seconds came from how Chrome handles audio. It doesn't constrain this choice and isn't a requirement.
- **The screen stays on in v1 (Wake Lock).** Short screen-off and app-switch tests (about 1 minute) passed. In a 30-minute pocket test with the screen locked, the page stopped after about 10 minutes, after at least 2 Change Beeps. When unlocked, it had reloaded to an empty state. So the tab was unloaded, not just slowed. Our unverified explanation: Firefox keeps a protective foreground service running only for media elements, not for Web Audio, so nothing stopped Android or Samsung's battery management from closing the tab. Scheduling the Cues up front is still correct with the screen on, and it is the basis for any later screen-off attempt.
