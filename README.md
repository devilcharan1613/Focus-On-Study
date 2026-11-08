# Focus On Study

A mobile-first study dashboard inspired by the feature set of [RohitN22595/Start-Focus-Desk](https://github.com/RohitN22595/Start-Focus-Desk), rebuilt as a browser-based Progressive Web App for Android, tablets and desktop.

## Features
- Responsive dark-first dashboard and light theme.
- Focus timer: 25, 45, 60-minute presets, custom duration, pause, resume and reset.
- Editable daily study goal, daily totals, goal progress and streak.
- Study session history saved in browser localStorage.
- Optional camera monitoring: face visibility, eye-closure estimation and cell-phone object detection.
- Camera starts only after the user taps Enable camera; Stop camera ends the stream.
- Installable PWA with cached application shell.

## Run locally
Camera access requires HTTPS or localhost.

```bash
git clone https://github.com/devilcharan1613/Focus-On-Study.git
cd Focus-On-Study
python -m http.server 8080
```

Open http://localhost:8080. On Android, deploy to GitHub Pages or another HTTPS static host, open in Chrome and choose **Add to Home screen** / **Install app**. No backend or build step is required.

## AI monitoring and limitations
When camera monitoring is enabled, the app loads MediaPipe Tasks Vision and TensorFlow.js COCO-SSD from jsDelivr, plus the MediaPipe face-landmarker model from Google's model storage. First-time model loading needs internet. The app shell and timer work offline after caching; offline AI availability depends on browser caching of third-party libraries and model files.

- Face visibility is based on face landmark results.
- Eye closure is estimated from eye landmark geometry. It is approximate and affected by lighting, camera angle and glasses.
- Phone detection uses the COCO-SSD `cell phone` class and may miss phones or produce false positives.
- A browser app cannot reliably determine whether someone is studying, nor keep camera monitoring active after the page is closed or suspended.
- Treat these as optional focus reminders, not medical, safety, proctoring or identity-verification features.

## Privacy
Camera permission is opt-in. Video is processed in the current browser tab and is not uploaded by this app to a project backend. Sessions and preferences are stored in this browser's localStorage. Clearing site data removes them. AI libraries and model files are downloaded from their respective CDNs when required.

## Attribution
This project is a mobile-focused reimplementation of the study-monitor concept and feature set from [RohitN22595/Start-Focus-Desk](https://github.com/RohitN22595/Start-Focus-Desk), published under the MIT License. This repository does not copy its Python application or bundled model/audio binaries. See the source project's [LICENSE](https://github.com/RohitN22595/Start-Focus-Desk/blob/main/LICENSE) for original terms. Third-party libraries and models retain their own licenses.

## Manual test checklist
- [ ] Load at phone and desktop widths.
- [ ] Start, pause, resume and reset timer.
- [ ] Complete a session and verify history.
- [ ] Change goal and verify progress.
- [ ] Reload and verify local persistence.
- [ ] Grant and revoke camera permission.
- [ ] Verify face, eye and phone statuses when AI models load.
- [ ] Install over HTTPS and verify cached shell opens offline.

## License
MIT. See [LICENSE](LICENSE).
