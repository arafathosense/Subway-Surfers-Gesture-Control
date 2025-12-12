# Subway Surfers Gesture Control


## Overview

This Python project reads your webcam, detects a single hand with MediaPipe, recognizes simple gestures (left/right by horizontal hand position, open palm for jump, hang-loose for duck), and simulates arrow-key presses using pynput. Use it to control games that accept keyboard arrow keys. This script captures video frames from your webcam, processes them with MediaPipe Hands to get 21 hand landmarks, determines which fingers are extended and the horizontal position of the hand, then maps gestures to keyboard arrow key presses. The program displays FPS and the currently detected action in a window.



## Output

<img width="1365" height="729" alt="Screenshot_1" src="https://github.com/user-attachments/assets/f76e5775-d50c-437e-86a3-95c98687f956" />
<img width="1365" height="730" alt="Screenshot_2" src="https://github.com/user-attachments/assets/f7096c88-d5ea-4e6d-85e7-817ec5c927a3" />
<img width="1365" height="726" alt="Screenshot_3" src="https://github.com/user-attachments/assets/7853efb3-c41a-413c-93f9-e72a42d2d0dc" />
<img width="1363" height="724" alt="Screenshot_4" src="https://github.com/user-attachments/assets/7039a4da-5f70-41db-b10e-7da7ccc2b35a" />


## Table of Contents

* Overview
* Features
* Dependencies
* Installation
* Run (step-by-step)
* How it works — step-by-step (detailed)
* Gesture ↔ Key mapping
* Important functions (what each part does)
* Tuning & parameters you can change
* Troubleshooting
* Safety notes
* Future improvements
* License





---

## Features

* Real-time hand detection and landmark drawing.
* Finger extension detection for each finger (thumb, index, middle, ring, pinky).
* Lane-based left/right detection using the horizontal center between thumb and index finger.
* Gesture-based Jump (open palm) and Duck (thumb + pinky only).
* Debounce state to avoid repeated key taps.
* On-screen FPS and action text.

---

## Dependencies

* Python 3.8+
* OpenCV (`opencv-python`)
* MediaPipe (`mediapipe`)
* pynput (`pynput`)

Install with pip:

```bash
pip install opencv-python mediapipe pynput
```

---

## Run (step-by-step)

1. Make sure your webcam is connected and free.
2. Save the provided script to a file, e.g. `gesture_control.py`.
3. Open a terminal in the file directory.
4. Install dependencies if you haven't already.
5. Run:

```bash
gesture_subway.py
```

6. A window titled **Subway Surfers Gesture Control** will open showing the webcam feed, hand landmarks, FPS, and current action.
7. Press `q` in the window (or focus the terminal and press Ctrl+C) to quit.

---

## How it works — step-by-step (detailed)

1. **Import libraries** — `cv2`, `mediapipe`, `pynput`, `time`, `math`.
2. **Initialize keyboard controller** — `Controller()` from `pynput` to simulate key presses.
3. **Initialize MediaPipe hands** (`mp_hands.Hands`) in video mode with `max_num_hands=1`.
4. **Start webcam** with `cv2.VideoCapture(0)` and set frame size to 640×480.
5. **Main loop** (runs while webcam is open):

   * Read a frame; if none, continue.
   * Flip the frame horizontally (mirror) for natural control.
   * Convert BGR → RGB and call MediaPipe `hands.process()` to get landmarks.
   * If a hand is found:

     * Extract thumb-tip and index-tip coordinates.
     * Compute `center_x = (thumb_tip.x + index_tip.x) / 2` to represent the horizontal position.
     * Call `count_extended_fingers()` to get a list of booleans for [thumb, index, middle, ring, pinky].
     * Draw hand landmarks on the frame.
     * Set `current_action` booleans according to the detected pose and position.
   * Compare `current_action` with `gesture_state` (previous frame state) to decide whether to `press_and_release()` a key (prevents rapid repeats).
   * Update `active_action` string for UI.
   * Compute and display FPS and `active_action` on the frame.
   * Show the frame with `cv2.imshow()`.
6. When `q` is pressed, release the webcam and close windows.

---

## Gesture ↔ Key mapping

* **Move Left** — hand center_x < `LEFT_LANE_BOUND` (0.33) → tap **Left Arrow** (`Key.left`)
* **Move Right** — hand center_x > `RIGHT_LANE_BOUND` (0.63) → tap **Right Arrow** (`Key.right`)
* **Jump** — all five fingers extended (`all(extended) == True`) → tap **Up Arrow** (`Key.up`)
* **Duck** — Hang-loose (thumb + pinky extended, others folded) → tap **Down Arrow** (`Key.down`)

---

## Important functions & variables (what each part does)

* `keyboard = Controller()` — object to send key events.
* `hands = mp_hands.Hands(...)` — MediaPipe hand detector and tracker.
* `cap = cv2.VideoCapture(0)` — webcam capture.
* `press_and_release(key_code)` — uses `keyboard.press()` and `keyboard.release()` to perform a short key tap.
* `count_extended_fingers(landmarks)` — returns `[Thumb, Index, Middle, Ring, Pinky]` booleans by comparing tip vs joint y-coordinates.
* `gesture_state` — memory dict (`move_left`, `move_right`, `jump`, `duck`) to avoid repeated key taps every frame.
* `LEFT_LANE_BOUND`, `RIGHT_LANE_BOUND` — lane thresholds (fractions of frame width in normalized landmark coordinates).
* `FINGER_TIPS` — landmark ids for finger tips.
* `mp_drawing.draw_landmarks()` — draws the skeleton on display for debugging/visual feedback.

---

## Tuning & parameters you can change

* `LEFT_LANE_BOUND`, `RIGHT_LANE_BOUND` — move these to change sensitivity for lane detection.
* `min_detection_confidence`, `min_tracking_confidence` — raise to reduce false positives (but initial detection slower).
* Add debounce timing: right now a gesture is considered active as a boolean; you can track timestamps and add a minimum interval between repeated key taps (e.g., 0.25s).
* Switch to continuous hold behavior: instead of tapping keys, call `keyboard.press()` when gesture begins and `keyboard.release()` when gesture ends.

---

## Troubleshooting

* **Camera not opening** — try different index: `cv2.VideoCapture(1)`. Check that no other application uses camera.
* **Landmarks not detected** — improve lighting, remove busy backgrounds. Make sure hand is roughly facing camera.
* **Keys not registering in game** — some games block simulated inputs or need focus; run the game in windowed mode and focus it, or run with administrative privileges. Some games require direct input APIs and won't accept simulated `pynput` events.
* **FPS very low** — reduce frame size or skip frames; increase `min_tracking_confidence` may reduce CPU usage in some cases.

---


## Future improvements (ideas)

* Add a small GUI to tune bounds, thresholds, and debounce timing.
* Support multi-hand gestures and more complex gestures.
* Add logging and a calibration mode to auto-adjust lane bounds per user.
* Replace simple heuristics with a small gesture classifier (ML) for more robust recognition.
* Add key-hold behavior for continuous lateral movement when hand stays in left/right lanes.

---


