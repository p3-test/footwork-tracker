# ⚽ ApexTouch — camera-based soccer footwork trainer

A free, browser-only take on the DribbleUp idea for kids: prop a phone on the floor, use **any ball**, and the app
tracks the ball and your feet to count reps, run timed challenges, and check kicking form. No subscription, no
special ball, no app store, no server. Everything runs in the browser and nothing leaves the device.

Live: https://p3-test.github.io/footwork-tracker/

## How it works

| Piece | What it actually does |
| :--- | :--- |
| **Ball tracking** | You tap the ball once on the setup screen to lock its color. Each frame is downscaled to 160 px wide and scanned for connected blobs of that color; the roundest, best-placed blob near the last known position is the ball. Runs in well under a millisecond per frame. |
| **Foot tracking** | MediaPipe Pose (loaded from CDN) gives knee, ankle, heel and toe landmarks. Only the lower body is used. If the full model is too slow for the device the app drops to the lite model automatically. |
| **Drill logic** | All distances are measured in *ball radii*, so the same thresholds work whether the phone is 4 feet or 10 feet away. |
| **Replays** | A rolling 3-second frame buffer of the mirrored camera view plus overlay. Strikes and challenge wins are saved as slow-motion clips you can scrub and export. |
| **Audio** | Web Audio metronome and rep tones, pitch rising with your streak. A voice coach via the browser's speech synthesis (see below). |
| **Storage** | Session history and settings live in `localStorage`. Replay clips are kept in memory for the current visit only. |

## Drills

| Drill | Rep counts when… | Feedback |
| :--- | :--- | :--- |
| **Foundations (Inside-Outs)** | Start with the ball at one foot. That spot is the home ring; the far ring (3.6 ball radii across, toward the other foot) lights first. Direction comes from the pose (ball right of the feet midpoint means go left). Rings slide with the ball's depth. A touch counts when a foot arrives at the ball and the ball moves 0.6 radii toward the lit side, or when the ball centre reaches the lit ring. Tap the video to reset. | Big LEFT/RIGHT arrow, lower ding for right, higher for left |
| **Toe Taps** | A toe lands on top of the ball (within 1.3 radii and above the ball's center) after being away from it. | "Alternate feet" if the same foot taps twice |
| **L-Drags** | The ball rolls back (away from the camera) at least one radius, then across at least two radii within 1.5 s. | "Faster" if the push doesn't follow the drag |
| **Wall Pass & Control** | The ball is struck (fast, more than 1.2 radii in 150 ms), travels at least 3 radii away or leaves the frame, then comes back and stops within 2.5 radii of a foot. | "Clean control" inside 1.6 radii, otherwise "Keep it closer" |
| **Kicking Form** | A strike is detected while the ball was sitting still. The frame just before impact is scored. | Plant-foot distance and ankle lock, 0–100 score, slow-motion replay |

### Kicking form scoring

* **Plant foot**: lateral distance from the plant ankle to the ball center must be 1.2–3.2 ball radii.
* **Ankle lock**: knee–ankle–toe angle of the striking leg at impact. Inside-foot pass expects 75–120°, laces drive expects 120–175°, and the angle must not change by more than 18° through contact.
* Score = plant (50) + ankle (50). ≥ 90 is "Perfect strike", ≥ 70 "Good strike", otherwise "Fix your form".

These thresholds are heuristics tuned on a low, front-facing camera. They are in `DRILLS.kicking_form.K` in `index.html` if you want to adjust them.

## Voice coach

Uses the phone's built-in speech voices, picked with the **Coach voice** dropdown on the Train screen (English voices listed; iPhone and Android ship several). The **voice** checkbox turns it off.

What it says: a spoken 3-2-1 and a go cue per drill; streak shout-outs at 5, 10, 20, 30 and 50; the count every 10 reps; a technique cue every ~14 seconds per drill ("knees bent, stay on your toes"); a nudge if nothing happens for 8 seconds; "Halfway", "Ten seconds" and a five-count in timed modes; minute marks in free play; "Faster!" when cadence drops; strike verdicts in Kicking Form; and a summary at the end with reps, time, best streak and whether it's a personal best.

Lines have priorities so the coach doesn't talk over itself: the countdown and results interrupt, counts and streaks queue, technique cues only fill silence. The last six seconds of a timed drill are kept clear for the count.

## Controls

* **📷 button (header)** turns the camera off and on from any screen. Turning it off during a drill pauses the drill.
* **Pause / Resume** on the Train screen pauses the drill, stops the clock and turns the camera off. Reps are kept. Resume turns the camera back on and continues.
* **Re-pick ball** on the Setup screen clears the locked color so you can tap the ball again. You can also just tap the ball again at any time while on Setup.
* **End & Save** stores the session in the Vault.

## Field-test aids

* **Giant counter** on the video itself, with a green flash on every rep, readable from across the room.
* **Wake lock** keeps the screen on while you're on the Setup or Train screens (Chrome Android, Safari 16.4+).
* **Buzz**: a short vibration per rep, a double buzz on a form warning, a long one on a challenge win (Android; iOS Safari has no vibration API).
* **Mask** toggle (Setup and Train): paints every pixel that matches the ball color pink and shows fps, pose latency, ball radius, blob area and the current color. If the ring jumps to a sock, turn this on and you'll see why. Screenshot it when reporting a tracking problem.
* **Color adaptation**: once locked, the target color drifts slowly toward what the ball actually looks like, so moving into shade doesn't drop the lock. Hue can't drift more than 25° and saturation can't fall below 70% of the tapped color, so it won't creep onto skin or floor.

## Guided workouts

Pick one from the **Guided workout** dropdown on the Train screen and press Start Workout. The coach introduces it, counts you into each block, calls time, runs the rest periods (with the next move animated on screen and a Skip button), and finishes with a spoken summary. Each block has a rep goal. The Vault shows the last workout block by block, and the history logs the workout as one row.

| Workout | Length | Blocks |
| :--- | :--- | :--- |
| **Quick Touch** | 5 min | Foundations, Toe Taps, L-Drags, Foundations, Toe Taps (40–45 s each, 15 s rests) |
| **Footwork Builder** | 9 min | Seven 60 s blocks cycling the three ground moves, 20 s rests |
| **First Touch** | 7 min | Wall-pass blocks with footwork between. Needs a wall. |
| **Striker** | 6 min | Footwork warm-up, two 90 s scored strike blocks, toe taps to finish |

Pause works during blocks and rests. End & Save mid-workout saves what was completed as a partial. Workouts are defined in `WORKOUTS` in `index.html`; adding one is a few lines.

## Challenges

* **Free Play** — open-ended, timer counts up.
* **30-Second Sprint** — as many reps as possible in 30 s.
* **40 Reps in 30s** — hit 40 before the clock runs out for confetti and a saved replay.

Timed modes nag with "Speed up!" if cadence drops below 1.33 reps per second.

## Setup that works

1. Front camera facing the player, portrait. A low stand (6–12 inches) or a tripod tilted down both work. Stand back until the ball sits around the **middle of the picture**, not the bottom edge. For Foundations, start with the ball at one foot: that becomes the home ring and the far ring lights first. Rings follow the ball's depth and reset when you tap the video (or if the ball sits far away for 4 seconds); a big LEFT/RIGHT arrow at the top of the video shows the next target.
2. Ball on the floor in view. **Tap it on screen** to lock its color. Re-tap any time the lighting changes.
3. Use a **bright ball** (orange, yellow, pink) that doesn't match socks, shoes or the floor. White balls on light floors and dark balls in dim rooms track poorly.
4. Feet and ball should both be in frame. Avoid direct sun glare on the floor.

## Running it

It is one file, `index.html`, with no build step. Cameras need HTTPS or localhost.

* **GitHub Pages**: push to `main`, enable Pages from the root, open the URL on the phone.
* **Local**: `python -m http.server 8787` in the repo folder, then open `http://localhost:8787` (desktop) or expose the port on your Wi-Fi for a phone.

Testing without a camera: `?demo=1` swaps in a synthetic feed with a rolling orange ball; `?pose=0` disables pose tracking so the ball tracker can be tested on its own.

## Browser notes

| Issue | Fix |
| :--- | :--- |
| "Camera permission denied" | Serve over https or localhost; allow camera for the site in browser settings and tap Retry. |
| Ball ring flickers or jumps to a sock | Tap the ball again to re-sample its color. Change socks or ball if they match. |
| Rings land off the edge | The video box follows the camera's real aspect, so nothing is cropped. If the ball rests near an edge you'll get "Move to the middle". |
| Rings are far from the ball | Tap the video and the rings re-centre on the ball. |
| No sound | Tap anywhere once; browsers block audio until a touch. Voice cues need a browser with speech synthesis (Safari and Chrome have it). |
| Replay export does nothing | Export needs `MediaRecorder`; Safari on iOS supports mp4 export in recent versions, older ones don't. Screen-record the replay instead. |
| Slow on an old phone | The app switches to the lite pose model automatically. Close other apps and turn off battery saver. |

## Known limits

* Color tracking only, so one ball at a time and nothing else of the same color in the frame.
* Kicking form uses the front camera, so it sees plant-foot width and ankle angle but not backswing or follow-through.
* Pose landmarks are 2D. Depth is inferred from screen position, which is why the camera needs to be low and the player facing it.

## License

MIT. Use it for your kids, your club, or your own fork.
