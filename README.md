# EyeTalk

A gaze-tracking AAC (Augmentative and Alternative Communication) system that lets users who cannot speak or use their hands communicate through eye movement alone. Built as a final-year software engineering capstone project.

## What it does

EyeTalk tracks a user's gaze in real time using an ordinary webcam, translates where they're looking into on-screen selections, and lets them build and send messages through a dwell-based ("look and wait") interface - no touch, mouse, or dedicated eye-tracking hardware required.

## Architecture

The system is made up of three cooperating components:

- **JavaFX client** - the on-screen keyboard/interface the user interacts with, including calibration screens and a dwell-based clicking mechanism (fixed dwell time and target radius).
- **Python / MediaPipe gaze server** - captures webcam frames, extracts facial landmarks, and estimates gaze direction. Communicates with the Java client over a TCP socket (binary, length-prefixed frames for images; newline-delimited CSV for gaze coordinates).
- **Flask / GPT-2 chat server** - a fine-tuned GPT-2 language model (fine-tuned on the DailyDialog dataset) that suggests replies and completions, with added safety layers appropriate for an AAC context (action-commitment filtering, polarity checks, fixed templates for yes/no questions) and Hebrew↔English translation.

### Gaze pipeline

Head pose is estimated with `solvePnP` from iris and facial landmark geometry. Raw gaze predictions are standardized (z-score) and mapped to screen coordinates using ridge regression, with leave-one-out cross-validation selecting between a linear or quadratic model per user. Predictions are smoothed with a One-Euro filter (implemented on both the Python and Java sides) to reduce jitter without adding perceptible lag.

### Design decisions worth noting

- Gaze prediction runs decoupled from the camera preview thread, which fixed frame-rate drops in the UI.
- A dual-ESC safety mechanism (via `jnativehook`) lets a caregiver interrupt the system at the OS level, independent of the app's own UI state.
- A dedicated `MessageOverlay` component handles confirmation dialogs so irreversible actions (like sending a message) require a deliberate second look.

## Tech stack

| Layer | Technology |

| Desktop UI | Java, JavaFX |
| Gaze estimation | Python, MediaPipe, OpenCV |
| Language model | Python, Flask, GPT-2 (Hugging Face), PyTorch |
| Communication | TCP sockets (custom binary + CSV protocols) |

## Project history

The gaze pipeline evolved from an earlier 2D polynomial-regression approach to a full 3D, head-pose-invariant model, which also cut the calibration procedure from 16 points down to 9. Before adopting MediaPipe, eye and head detection were originally implemented from scratch in Java using Local Binary Pattern (LBP) features and Haar-like contrast features, without relying on OpenCV's built-in detectors.

## Status

Completed capstone project (2025-2026), including full project documentation.

## Model weights

The fine-tuned GPT-2 weights are large and are stored externally (Google Drive) rather than in this repository - reach out if you'd like access for evaluation purposes.
