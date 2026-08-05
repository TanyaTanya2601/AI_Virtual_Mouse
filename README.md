#  AI Virtual Mouse

A gesture-based virtual mouse control system using hand detection and computer vision. Control your computer mouse, keyboard shortcuts, and system functions entirely through hand gestures captured via your webcam.

---

## Table of Contents

- [Project Overview](#project-overview)
- [System Requirements](#system-requirements)
- [Prerequisites & Dependencies](#prerequisites--dependencies)
- [Libraries](#Libraries)
- [Algorithm](#Algorithm)
- [Code Explanation](#code-explanation)
- [Gesture Controls](#gesture-controls)

---

## Project Overview

This project implements a **gesture-based mouse control system** using MediaPipe's hand detection and OpenCV for real-time video processing. Instead of using a traditional mouse, you can control your computer by making hand gestures in front of your webcam.

The system detects hand landmarks (27 points on each hand) and maps finger positions to screen coordinates, allowing intuitive gesture-based control of:
- Mouse movement
- Left/Right clicks
- Copy/Paste operations
- Volume control
- Screenshots
- Custom keyboard shortcuts

---


## System Requirements

| Requirement | Specification |
|---|---|
| **Python Version** | 3.8, 3.9, 3.10, 3.11  (3.14+ not compatible due to PyAudio issues) |
| **OS** | Windows, macOS, Linux |
| **Webcam** | USB or built-in camera required |
| **RAM** | Minimum 4GB |
| **Processor** | Intel i5/AMD Ryzen 5 or better (recommended) |

---

##  Prerequisites & Dependencies

### Python Libraries & Compatible Versions

| Library | Version | Purpose | Python Compatibility |
|---|---|---|---|
| **opencv-python** | 4.8.0 - 4.10.0 | Video capture & image processing | 3.8+ |
| **mediapipe** | 0.10.0 - 0.14.0 | Hand detection & landmark tracking | 3.8 - 3.11 |
| **numpy** | 1.24.0 - 1.26.0 | Numerical operations & interpolation | 3.8 - 3.11 |
| **autopy** | 4.0.0+ | Mouse movement & clicking (cross-platform) | 3.8 - 3.11 |
| **pyautogui** | 0.9.53+ | Keyboard automation & GUI control | 3.8 - 3.11 |

## Libraries

```bash
Import Libraries
import autopy
import cv2
import mediapipe
import numpy
import pyautogui
## Purpose
OpenCV captures webcam frames.
MediaPipe detects hands and returns 21 hand landmarks.
NumPy maps webcam coordinates to screen coordinates.
AutoPy controls the mouse cursor.
PyAutoGUI performs keyboard shortcuts and screenshots.
```
## Algorithm

Algorithm
Start


↓

Capture webcam frame

↓

Convert BGR → RGB

↓

Detect hand landmarks

↓

Identify finger positions

↓

Recognize gesture

↓

Perform corresponding mouse or keyboard action

↓

Repeat until Q is pressed

↓

Exit



##  Code Explanation

### 1. **Initialization**
```python
cap = cv2.VideoCapture(0)  # Access webcam (0 = default camera)
initHand = mediapipe.solutions.hands  # Load hand detection model
mainHand = initHand.Hands(min_detection_confidence=0.8, min_tracking_confidence=0.8)
```
- `cv2.VideoCapture(0)`: Opens the default webcam
- `min_detection_confidence=0.8`: Only detect hands with 80% confidence
- `min_tracking_confidence=0.8`: Smooth tracking between frames

### 2. **Hand Landmarks Detection** (`handLandmarks()`)
```python
def handLandmarks(colorImg):
    landmarkList = []
    landmarkPositions = mainHand.process(colorImg)
    landmarkCheck = landmarkPositions.multi_hand_landmarks
    if landmarkCheck:
        for hand in landmarkCheck:
            for index, landmark in enumerate(hand.landmark):
                # Extract 21 hand landmarks per hand
                h, w, c = img.shape
                centerX, centerY = int(landmark.x * w), int(landmark.y * h)
                landmarkList.append([index, centerX, centerY])
    return landmarkList
```

**What it does:**
- Processes RGB image through MediaPipe hand detector
- Returns list of 21 landmarks per hand (knuckles, joints, fingertips)
- Each landmark contains: `[landmark_index, pixel_X, pixel_Y]`

**Hand Landmark Indices (important):**
| Finger | Tip ID | PIP ID |
|---|---|---|
| Thumb | 4 | 2 |
| Index | 8 | 6 |
| Middle | 12 | 10 |
| Ring | 16 | 14 |
| Pinky | 20 | 18 |

### 3. **Finger Detection** (`fingers()`)
```python
def fingers(landmarks):
    fingerTips = []
    tipIds = [4, 8, 12, 16, 20]  # IDs of finger tips
    
    # Thumb: check if tip is to the right of previous joint
    if landmarks[tipIds[0]][1] > lmList[tipIds[0] - 1][1]:
        fingerTips.append(1)  # Thumb is up
    else:
        fingerTips.append(0)  # Thumb is down
    
    # Other fingers: check if tip is ABOVE previous joint (lower Y = higher on screen)
    for id in range(1, 5):
        if landmarks[tipIds[id]][2] < landmarks[tipIds[id] - 3][2]:
            fingerTips.append(1)  # Finger is up
        else:
            fingerTips.append(0)  # Finger is down
    
    return fingerTips  # [thumb, index, middle, ring, pinky]
```

**Logic:**
- Returns array of 5 values: `[thumb_status, index_status, middle_status, ring_status, pinky_status]`
- 1 = finger extended, 0 = finger folded
- Thumb: horizontal comparison (left/right)
- Other fingers: vertical comparison (up/down)

### 4. **Mouse Movement & Smoothing**
```python
if finger[1] == 1 and finger[2] == 0:  # Index up, Middle down = Move mode
    x3 = numpy.interp(x1, (75, 640 - 75), (0, wScr))
    y3 = numpy.interp(y1, (75, 480 - 75), (0, hScr))
    cX = pX + (x3 - cX) / 7  # Smoothing factor = 7
    cY = pY + (y3 - cY) / 7
    autopy.mouse.move(wScr - cX, cY)
    pX, pY = cX, cY
```

**Explanation:**
- `numpy.interp()`: Maps webcam coordinates (75-565px) to screen coordinates
- Smoothing: `(x3 - pX) / 7` prevents jittery movement
- `wScr - cX`: Mirrors X-axis for natural hand-to-screen mapping

### 5. **Click & Interaction Detection**
```python
if finger[1] == 0 and finger[0] == 1:  # Index down, Thumb up = Click
    autopy.mouse.click()

if finger[2] == 1 and finger[1] == 1:  # Index & Middle up = Right Click
    pyautogui.click(button="right")
```

---

## 🖐️ Gesture Controls

### Gesture Map & Functions

| Gesture | Fingers | Action | Function |
|---|---|---|---|
| **Mouse Move** | Index ✋, Middle ✋ | Move cursor | Maps hand position to screen |
| **Left Click** | Index 👎, Thumb 👍 | Single click | `autopy.mouse.click()` |
| **Right Click** | Index ✋, Middle ✋ | Context menu | `pyautogui.click(button="right")` |
| **Copy** | Pinky ✋, Thumb 👎 | Copy selected | `Ctrl + C` |
| **Paste** | Pinky ✋, Thumb ✋ | Paste clipboard | `Ctrl + V` |
| **Volume Up** | Middle ✋ (alone) | Increase volume | `Volume Up hotkey` |
| **Volume Down** | Ring ✋ (alone) | Decrease volume | `Volume Down hotkey` |
| **Screenshot** | Index ✋, Middle ✋, Ring ✋ | Take screenshot | Saves as `my_screenshot.png` |

### Visual Hand Gestures

```
Index Finger Up (✋)   = Detect as 1
Index Finger Down (👎) = Detect as 0

For Mouse Move:     👆 👆  (both index & middle up)
For Left Click:     👎 👍  (index down, thumb up)
For Right Click:    👆 👆  (index & middle both up)
```

```



**Happy gesture controlling!** 

