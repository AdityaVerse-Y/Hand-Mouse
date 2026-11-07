# Hand Mouse: Touchless Mouse Control

Control your computer's mouse with hand gestures using just a webcam. Hand Mouse tracks your hand in real time, classifies the gesture you are making, and turns it into cursor movement, clicks, drag and drop, scrolling and browser navigation.

A state machine with multi-frame confirmation, cooldowns and action locks keeps control smooth and prevents accidental clicks when your hand moves between gestures.

> **Platform:** Windows only (uses the DirectShow camera backend and Windows APIs).

<!-- Add a demo GIF or screenshot here:
![Hand Mouse demo](docs/demo.gif)
-->

## Features

- **Real-time hand tracking** with MediaPipe Hands (21 landmarks per frame)
- **Smooth cursor movement** from the index fingertip, with adjustable smoothing
- **Left click, right click, double click** through distinct finger gestures
- **Drag and drop** by pinching, then closing into a fist
- **Touchless scrolling** with deadzone and reverse-stroke protection
- **Browser back** with a four-finger gesture (sends `Alt + Left`)
- **Pause and idle gestures** so you can rest your hand without moving the cursor
- **Gesture debouncing:** majority-vote buffer, confirmation frames, cooldowns and action locks
- **Window safety:** clicks and scrolls are blocked when the cursor is over the Hand Mouse camera window
- **On-screen HUD** showing tracking status, current gesture, state, last action and a gesture legend

## Gesture Guide

| Gesture | Hand shape | Action |
|---|---|---|
| **POINT** | Index finger extended, others curled | Move cursor; move finger up/down to scroll |
| **PINCH** | Thumb and index tips together | Left click (exactly one per pinch) |
| **TWO FINGERS** | Index + middle extended (peace sign) | Right click |
| **THREE FINGERS** | Index + middle + ring extended | Double click |
| **FOUR FINGERS** | Four fingers extended, thumb tucked | Browser back (`Alt + Left`) |
| **PINCH, then FIST** | Pinch, then close the hand within 2 s | Drag: hold left button; open hand to drop |
| **OPEN PALM** | All fingers and thumb extended | Pause control |
| **FIST** | All fingers curled | Idle (no action) |

## How It Works

```text
Webcam
  -> OpenCV (frame capture, mirror flip, BGR to RGB)
  -> MediaPipe Hands (21 landmarks)
  -> Landmark analysis (finger extension, normalized pinch distance)
  -> Gesture classification
  -> Debouncing and state machine (rolling vote, confirmation frames, cooldowns, locks)
  -> PyAutoGUI (cursor move, click, drag, scroll, hotkey)
```

## Gesture Reliability

Hands pass through intermediate shapes while changing gesture (for example, raising three fingers briefly looks like two). Hand Mouse handles this with:

| Safeguard | Setting |
|---|---|
| Rolling majority vote | Last 7 frames |
| Confirmation frames | Pinch: 2, Right click: 4, Double click: 4, Four fingers: 3 |
| Cooldowns | Pinch 0.5 s, Right/Double click 0.6 s, Browser back 0.8 s |
| Action locks | After an action fires, conflicting gestures are suppressed until the hand returns to a neutral pose (POINT, OPEN PALM, FIST) for 3 frames |
| Transition protection | Three-to-four finger transitions cannot trigger a double click |
| Scroll deadzone | +/- 20 px absorbs hand tremor |
| Reversal delay | 0.65 s delay absorbs the hand's return stroke so it does not scroll backwards |
| Drag arming | FIST must follow PINCH within 2 s, confirmed over 2 frames; drop needs 4 non-fist frames |

All of these values are constants at the top of `hand_mouse.py` and can be tuned.

## Requirements

- Windows 10 or 11
- Python 3.9 or newer
- A webcam

Dependencies (`requirements.txt`): `opencv-python`, `mediapipe`, `numpy`, `pyautogui`

## Installation

```powershell
git clone https://github.com/<your-username>/hand-mouse.git
cd hand-mouse
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

## Usage

```powershell
python hand_mouse.py
```

- Keep your hand inside the grey guide box shown in the camera window. Cursor movement is mapped from this box to your full screen.
- Press **`q`** with the camera window focused to quit. This also releases any active drag and frees the camera.

## Configuration

Edit the constants at the top of `hand_mouse.py`:

| Constant | Purpose |
|---|---|
| `SMOOTH_FACTOR` | Higher = smoother cursor, more lag |
| `MARGIN_X`, `MARGIN_Y` | Size of the interaction box in the camera view |
| `PINCH_DISTANCE_THRESHOLD` | How close thumb and index must be to count as a pinch |
| `SCROLL_CLICKS` | Scroll amount per pulse |
| `SCROLL_TRIGGER_DISTANCE` | Finger travel (px) needed to trigger a scroll |
| `*_COOLDOWN`, `MIN_*_CONFIRM` | Per-gesture cooldowns and confirmation frames |

## Testing

```powershell
# Syntax check
python -m py_compile hand_mouse.py

# Gesture classification and state-machine tests (15 tests)
python test_gestures.py
```

The tests cover:

1. Gesture classification and mutual exclusivity
2. Three fingers produce a double click only (no right clicks while raising or lowering)
3. Three fingers produce no left click during pinch-like transition frames
4. Three fingers then fist does not start a drag
5. Two fingers produce a right click only
6. Pinch produces exactly one left click, even when held
7. Pinch then fist starts and ends a drag cleanly
8. Holding double click or right click fires only once
9. Four fingers fire exactly one browser back
10. Holding four fingers does not repeat
11. Releasing and repeating four fingers fires twice
12. Three-to-four transition fires browser back, not double click
13. Four-to-three transition does not fire a double click
14. A deliberate three-finger hold still fires one double click
15. Four fingers fire no left or right clicks

## Troubleshooting

- **Scrolling does nothing:** run `python scroll_diagnostic.py`, focus a browser window, and press `u` / `d` to check that mouse wheel events reach your system.
- **Camera will not open:** close other apps using the webcam, or change the camera index in `cv2.VideoCapture(0, cv2.CAP_DSHOW)`.
- **Clicks are blocked:** the cursor is over the Hand Mouse window. Move it to another window.
- **Gestures misfire:** use even lighting, keep your hand fully in frame, and raise `PINCH_DISTANCE_THRESHOLD` or the confirmation frame counts.

## Project Structure

```text
hand-mouse/
├── hand_mouse.py          # Application: gesture detection, state machine, HUD, main loop
├── test_gestures.py       # Gesture classification and state-machine tests
├── scroll_diagnostic.py   # Standalone Windows mouse wheel test utility
├── requirements.txt       # Python dependencies
├── .gitignore
├── LICENSE
└── README.md
```

## Safety Notes

Hand Mouse takes control of your real mouse. If something goes wrong, press `q` in the camera window or close the terminal. `pyautogui.FAILSAFE` is disabled, so do not rely on moving the mouse to a screen corner to stop it.

## Future Scope

- Configurable gesture-to-action mapping and keyboard shortcuts
- More gestures (browser forward, volume control)
- Sensitivity sliders in the UI
- Two-hand support
- Per-application gesture profiles
- Cross-platform support (macOS, Linux)

## Author

**Aditya Yadav**
