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
| **Foundations (Inside-Outs)** | Ball in the guide circle between the feet. Rings sit 2.2 ball radii either side and slide with the ball's depth. A touch counts when a foot arrives at the ball and the ball moves 0.6 radii toward the lit side, or when the ball centre reaches the lit ring. Tap the video to re-centre. | Big LEFT/RIGHT arrow, lower ding for right, higher for left |
| **Toe Taps** | A toe lands inside the ball's disc (within 1.15 radii, not below its centre) after being away from it. Works from a high or low camera. | "Alternate feet" if the same foot taps twice |
| **L-Drags** | The ball rolls back toward the player at least 0.8 radii, then across at least 1.6 radii within 2.5 s. | "Push across" if the push doesn't follow the drag |
| **V-Pulls** | The ball rolls back at least 0.8 radii, then out on a diagonal from the deepest point: at least 1.4 radii sideways and 0.5 forward within 2.5 s. A sideways push with no forward travel is an L, not a V, and doesn't count. | "Same foot for the pull and the push" when the pose is sure two different feet did it |
| **Scissors (beta)** | The ball stays still while a toe crosses from one side of it to the other in front of it: lower in the picture than the ball's centre, or briefly covering the ball. One count per foot per 0.7 s, so the foot coming back isn't a second rep. Pose smoothing is off for this drill. | "Switch feet" when the same foot goes twice and the pose is sure |
| **Wall Pass & Control** | The ball is struck (more than 1 radius in 150 ms), travels at least 3 radii away or leaves the frame, then comes back and stops within 2.5 radii of a foot. | "Clean control" inside 1.6 radii, otherwise "Keep it closer" |
| **Kicking Form** | Camera **side-on**: the player kicks across the picture. A strike is detected while the ball was sitting still; the frame just before impact, the impact frame and a frame 350 ms later are scored. | Plant foot beside the ball, ankle lock, knee over the ball, follow-through; 0–100 score; slow-motion replay |

### Kicking form scoring (side-on camera)

Just before impact the plant foot is beside the ball and the kicking foot is back in its swing, which is how the two feet are told apart. Distances are in ball radii along the kick direction (ahead of the ball = positive).

* **Plant foot** (30): plant ankle between 2.0 radii behind and 1.0 ahead of the ball.
* **Ankle lock** (30): knee–ankle–toe angle of the kicking leg. Inside-foot pass expects 75–120°, laces drive 120–175°, and it must not change by more than 18° through contact.
* **Knee over the ball** (20): kicking knee no more than 1.8 radii behind the ball at impact.
* **Follow-through** (20): kicking foot at least 1.5 radii past the ball 350 ms after impact.
* ≥ 90 "Perfect strike", ≥ 70 "Good strike", otherwise "Fix your form", with the top two fixes spoken and shown.

Thresholds are in `DRILLS.kicking_form.K` in `index.html`.

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
| **Footwork Builder** | 9 min | Seven 60 s blocks: Foundations, Toe Taps, L-Drags, V-Pulls, 20 s rests |
| **First Touch** | 8 min | Foundations and Toe Taps, then the camera moves side-on for three wall-pass blocks. Needs a wall. |
| **Striker** | 7 min | Foundations and Toe Taps, then the camera moves side-on for two 90 s scored strike blocks |

Workouts are ordered so the camera moves at most once. Where the setup changes from front to side, a 45 s "Move the camera" step replaces the rest: the coach reads out where the phone goes, and the next block waits for the framing check. Pause works during blocks and rests. End & Save mid-workout saves what was completed as a partial. Workouts are defined in `WORKOUTS` in `index.html`; adding one is a few lines.

## Challenges

* **Free Play** — open-ended, timer counts up.
* **30-Second Sprint** — as many reps as possible in 30 s.
* **40 Reps in 30s** — hit 40 before the clock runs out for confetti and a saved replay.

Timed modes nag with "Speed up!" if cadence drops below 1.33 reps per second.

## Camera setups

Each drill names one of three setups, shown on the Train screen under the drill. The pose model finds a person by the face first, so for the front setups the **whole body, head included**, needs to be in the picture.

| Setup | Where the phone goes | Drills |
| :--- | :--- | :--- |
| **Front, low** | Low, facing the player, 8 to 10 ft back | Foundations, Toe Taps |
| **Front, raised** | Tripod about chest height, tilted down, facing the player | L-Drags, V-Pulls, Scissors |
| **Side-on, low** | Off to the side at shin to knee height, 8 to 10 ft away, so the player moves across the picture | Wall Pass, Kicking Form |

Front low and front raised count as the same group, so a workout doesn't stop to move the phone between them.

**Framing check.** Press Start and walk back. Until the drill begins, the video shows a yellow guide (ball circle and foot outlines) and a checklist: ball seen, ball size (camera too close or too far), ball still, and with pose on, head, hips and feet in view (one foot is enough side-on). The countdown starts once everything passes for a moment. After 20 seconds it starts anyway and says what was missing. The coach reads out the first missing item every few seconds.

## Setup that works

1. Put the phone where the drill's camera setup says. Stand back until the ball sits around the **middle of the picture**, not the bottom edge. For Foundations the rings are placed either side of the ball when the countdown ends, follow the ball's depth, and re-centre when you tap the video; a big LEFT/RIGHT arrow at the top of the video shows the next target.
2. Ball on the floor in view. **Tap it on screen** to lock its color. Re-tap any time the lighting changes.
3. Use a **bright ball** (orange, yellow, pink) that doesn't match socks, shoes or the floor. White balls on light floors and dark balls in dim rooms track poorly.
4. Avoid direct sun glare on the floor.

## Test recordings (measuring the counting)

Tick **Record this drill for testing** under the Train buttons (single drills only, not workouts) and do the drill. The app saves the camera video plus, for every frame, what the ball tracker and the pose model saw and when the app counted. Recordings stay on the phone (the newest six are kept, up to two minutes each).

In the Vault, under **Test Recordings**:

* **Mark touches** plays the video with a big TOUCH button. Tap it at every real touch; half speed or slower is most accurate, and pausing on a frame places a mark exactly. Undo and Clear are there. On a computer, the space bar marks too.
* **Score** compares your marks with three ways of counting: what the app counted as it ran, a replay of the recorded frames through the drill's logic with the feet, and the same replay with the ball only. Each shows counted, caught (within a third of a second of a mark), missed and phantom counts.
* **Download** saves the data as JSON and the video file, for tuning the thresholds on a computer.

This is how to tell, per drill and per setup, whether the feet actually reduce misses or the ball alone is enough.

## Feedback the app only gives when it's sure

Every foot point carries the pose model's visibility score. Anything the app says about the feet, such as "alternate feet", "same foot for the pull and the push" or a kicking correction, needs a score of 0.75 (0.7 for kicking) or the check is skipped. A wrong correction costs more trust than a missing one.

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
* Pose landmarks are 2D. Front and back are inferred from screen position and from the foot covering the ball, which is why each drill names its camera setup.
* Pose tracking uses MediaPipe's older Pose solution, which Google no longer updates. Its newer Pose Landmarker hasn't been tried yet; the test recordings are meant to compare the two before switching.
* Scissors counting is a beta. It hasn't been tuned on real footage yet.
* Juggling isn't attempted: the feet leave the frame and the ball is in the air.

## License

MIT. Use it for your kids, your club, or your own fork.
