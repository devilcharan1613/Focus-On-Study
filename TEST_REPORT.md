# Focus On Study - Developer Test Report

**Review date:** 8 November 2026  
**Repository:** https://github.com/devilcharan1613/Focus-On-Study  
**Reviewed branch:** `main`

## Result

The latest review found and fixed a CI test-harness defect: the mocked audio elements did not implement the browser `pause()` method used by the new sound-test handler. The application code itself passed JavaScript syntax checks, but the smoke test failed until this mock was corrected. Re-run CI on the new commit before treating validation as green. Real-device checks remain outstanding.

## Fixes applied

1. **Local date consistency:** Replaced locale-dependent date serialization with an explicit local `YYYY-MM-DD` format. This keeps daily totals and streak grouping consistent across browser locale settings.
2. **Stored-data validation:** Added defensive normalization for localStorage data. Invalid goals, malformed session collections, invalid durations, and unsupported theme values no longer flow directly into dashboard calculations.
3. **Offline navigation fallback:** The service worker now serves the cached app shell only for failed page navigations. It no longer returns `index.html` for failed CSS, JavaScript, or other asset requests, which could cause confusing MIME/type errors.
4. **Cache refresh:** Bumped the service-worker cache key so updated shell assets can replace the older cached version.
5. **Automated smoke tests:** Added a Node.js built-in test harness covering app boot, required DOM references, manifest paths, service-worker fallback rules, timer start/pause/resume/reset, and daily-goal editing.
6. **CI improvements:** GitHub Actions now checks both JavaScript files, runs the smoke test, validates required files, and parses the web manifest.

## Automated test results

| Check | Result |
|---|---|
| `node --check app.js` | PASS |
| `node --check sw.js` | PASS |
| `node --check tests/smoke.cjs` | PASS |
| App initialization in mocked browser runtime | PASS |
| Timer start, pause, resume and reset handlers | PASS |
| Custom-session dialog opens | PASS |
| Daily goal edit/save | PASS |
| HTML element references | PASS |
| Web app manifest JSON and relative paths | PASS |
| Service-worker navigation fallback checks | PASS |

The previous validation run on commit `9103239d1288391c4fa8f0761e4462681c6418b0` failed at the audio smoke-test step with `TypeError: a.pause is not a function`. The mock `El` class now implements `pause()`, `play()`, and the basic media state used by the handler. The next Actions run is the source of truth for whether this correction passes.

## Remaining manual/device tests

These checks require a real browser, camera, and device and are not represented as passed by the automated smoke test:

- Verify camera permission grant, denial, and revocation on Android Chrome.
- Verify MediaPipe face/eye estimation and COCO-SSD phone detection on the target device.
- Verify model download and camera performance on slower networks and lower-memory phones.
- Verify PWA installation and offline launch after the first successful online visit.
- Verify timer behavior after Android suspends the browser tab or locks the screen.
- Verify the live GitHub Pages URL after the Pages deployment workflow completes.

## Known limitations

- Camera checks are approximate attention prompts, not reliable proctoring or proof of studying.
- AI model libraries and model weights are fetched from external CDNs on first use; camera AI is not guaranteed to work offline until those resources are cached.
- Browsers can suspend timers and camera processing when a tab is backgrounded, the device is locked, or the browser is closed.
- Study history and settings are stored in browser localStorage on that device. Clearing browser site data removes them.
- No real-device camera, cross-browser, accessibility audit, or penetration test was performed in this automated pass.

## Files changed

- `app.js`
- `sw.js`
- `tests/smoke.cjs` (new)
- `.github/workflows/validate.yml`



## Original Focus Desk alert audio integration

The three MP3 files from the reference repository were added under `assets/` and connected to the matching alerts:
- `alarm.mp3`: eyes-closed alert and focus timer completion.
- `faudio.mp3`: face out of view.
- `paudio.mp3`: phone detected.

The audio is loaded on demand by the browser (`preload="none"`) and is included in the service-worker app-shell cache. Browser audio autoplay restrictions still apply: alerts can play after the user has interacted with the page. The audio files retain any third-party rights and attribution requirements that apply to them; the source repository's README distinguishes the MIT-licensed code from audio/model assets.


## MP3 playback reliability fix

The prior playback handler silently switched to a short oscillator beep when the MP3 play promise rejected. That fallback made audio-load, autoplay, or path errors sound like a successful but incorrect beep. It has been removed. Alert events now request their corresponding source MP3 at full HTML media volume (1.0), and failed playback is logged and reported in the on-page status text instead of synthesizing a beep.

The monitor attempts alert audio after the user has interacted with the page. The three individual sound-test buttons provide direct user-gesture playback for troubleshooting. Audio elements are preloaded, use explicit relative paths, and are stopped when the camera monitor is stopped. Timer completion uses the eye/alarm MP3 as a completion cue; the eye, face, and phone monitor alerts retain their respective mapping.


## Responsive camera, full-length audio, and phone detection update

- Enlarged the camera preview responsively using a viewport-based height with a 320 px minimum on desktop and 340 px minimum on mobile; video and detection overlay share a contained viewport.
- Replaced the chained audio test with three separate user-initiated controls. Each button starts its complete corresponding MP3 at media volume 100%, and reports load/play errors on screen. Active alert playback is no longer restarted every six seconds, preventing long alert clips from being cut off.
- Timer completion now uses the eye/alarm sound. The face-out-of-view and phone-detected alerts keep their respective MP3s.
- Camera requests a higher ideal capture resolution and waits for video metadata before starting inference. Phone detection threshold is lowered from 0.45 to 0.30 to improve recall; this may increase false positives. Phone detection remains limited by COCO-SSD accuracy, camera angle, lighting, and device performance.
- Updated service-worker cache to v5 and expanded smoke tests for the new audio controls and camera settings.

Automated checks do not prove actual speaker output or real-world phone recognition. Those require testing on the target Android device with site sound permission, media volume enabled, camera permission granted, and adequate lighting.


## Latest CI defect correction (8 November 2026)

The smoke test invoked the eye-alert test button but its fake DOM element lacked the `pause()` method that real HTML audio elements provide. This caused the automated test to fail before subsequent validation steps could run. The test double now models `pause()`, `play()`, `currentTime`, `volume`, `muted`, `paused`, and `ended`. This is a test-harness correction, not a change to production audio playback. Confirm the new GitHub Actions run before marking the fix as verified.


## MP3 alert playback fix (8 November 2026)

### Root cause

The automatic camera-monitor alert path only called `playAlert()` after `audioUnlocked` had already become true. That flag was set by manually tapping one of the sound-test buttons, so automatic alerts could remain silent even though the MP3 files were present and cached. The previous implementation also depended on browser autoplay policy, which blocks audio started outside a user gesture on some browsers.

### Fix

- Added an audio-unlock attempt during the user's **Start focus** and **Enable camera** button interactions.
- The unlock routine primes the three bundled MP3 elements while muted, then pauses and rewinds them.
- Camera detection now attempts the correct MP3 alert directly instead of silently skipping playback while waiting for the test buttons to be used.
- No generated oscillator/beep fallback is used. If the browser blocks playback, the matching **Play** test button remains available and the browser console reports the failure.

### Validation

Smoke tests now assert the unlock function, its user-interaction entry points, and the automatic MP3 alert call. Browser autoplay restrictions vary by device, so final audible playback still needs a real-device check after deployment. The app must be opened, and the user should tap **Start focus** or **Enable camera** once before monitoring alerts are expected to play.


## Larger camera preview (8 November 2026)

- Increased the attention-monitor camera preview to a 460–680px responsive height on desktop and a 480–680px responsive height on mobile.
- Kept the video and detection overlay responsive within the enlarged preview; the preview continues to use `object-fit: contain` so the camera image is not cropped.
- Added smoke-test assertions for both responsive camera sizing rules.
- Automated source-level checks can verify the CSS rules, but actual camera rendering and permission behavior still require testing on a physical phone/browser.


## Timer and study-statistics correction (8 November 2026)

### Bug fixed
- Changing a timer preset or custom duration after pausing previously changed the countdown duration but retained the old elapsed-time accumulator. This could make the next session finish early.
- Selecting a new duration while paused could also leave the controls showing **Resume** and **Paused**, even though a fresh duration had been loaded.

### Fix
- Selecting a new preset/custom duration now clears the previous elapsed-time accumulator and returns the timer controls and status to **Start focus / Ready**.
- The timer remains protected from changing presets while actively running.
- Added a regression check that pauses a session, selects 45 minutes, and verifies the full 45:00 duration and Ready state.

### Statistics review
- Today focused, sessions done, daily-goal progress, streak, and recent history are calculated from completed sessions saved in localStorage.
- Sessions shorter than 60 seconds are intentionally not added to study history. Timer completion and statistics persistence still need a real browser test; the automated test suite does not simulate a full 5–180 minute session or Android background suspension.


## Phone detection and live study statistics update (8 November 2026)

### Phone check improvements
- Lowered the COCO-SSD phone confidence threshold to 0.22 and requests up to 20 candidate boxes, then selects the strongest `cell phone` detection.
- Draws a confidence percentage beside a detected phone so users can tell what the model recognized.
- Shows **Good** when the model completes a frame check and finds no phone, instead of leaving the indicator stuck on **Checking** before the first phone has ever been seen.
- Reports **Unavailable** if phone inference throws an error. Detection remains approximate and can miss phones or produce false positives depending on camera angle, lighting, distance, and device speed.

### Study statistics improvements
- Today focused, daily-goal completion, progress bar, and remaining time now update during active focus sessions.
- Paused time remains included in the current session's accumulated focus time; the display stops advancing while paused.
- Sessions done and Recent Activity continue to represent completed sessions saved locally. A session is added to history when the timer completes and only if it lasted at least 60 seconds.
- Bumped the service-worker cache to v6 so deployed browsers request the updated app shell.

### Validation scope
- Smoke tests check the live-statistics update path and phone-detection threshold/status behavior.
- Automated tests cannot prove real phone recognition. Verify on the actual phone with camera permission, good lighting, a phone visible in-frame, and internet access for first-time model loading.
- Statistics are stored in this browser's localStorage and are not synchronized across devices. Existing sessions are not erased by this update.

- Follow-up correctness guard: clear the active-session accumulator before refreshing completed-session totals, preventing the newly saved session from being counted twice in Today focused and goal progress. Reset and preset changes also immediately refresh the live dashboard values.
