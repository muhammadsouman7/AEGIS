# AEGIS — Autonomous Environment & Guidance Intelligence System

AEGIS is an AI-powered assistive navigation system designed to help blind and visually impaired users move safely and independently, using only a smartphone camera and a pair of earphones — no specialized hardware required.

Inspired by the "EDITH" glasses concept, AEGIS acts as a voice-first companion that sees the world on the user's behalf and communicates what matters through speech: obstacles ahead, text on signs and labels, and the location of specific objects — all triggered hands-free with a spoken wake word.

## Features

- **Obstacle Alert** — Real-time detection of people, furniture, vehicles, and everyday objects using YOLOv8, with distance estimated in footsteps (not meters) and direction (left / center / right) for intuitive, non-visual guidance. Objects that are very close trigger urgent turn guidance ("stop, turn left").
- **Text Reader** — Reads signboards, labels, menus, or any printed text aloud using OCR.
- **Find My Object** — Say what you're looking for ("where is the microwave") and AEGIS tells you its direction and distance.
- **Hands-Free Voice Control** — Always-listening wake word ("Hey Guide") lets the user issue commands naturally, without touching the screen — critical for an accessibility-first tool.
- **Live Bounding Box Overlay** — For sighted assistance or debugging, detected objects are drawn live on the camera feed.

## Architecture

```
Phone browser (camera + mic via Web APIs)
        │
        ├── Live video frames ──► Flask backend (Google Colab, free GPU)
        │                              │
        │                         YOLOv8 (object detection)
        │                         EasyOCR (text reading)
        │                              │
        ├── Spoken commands ──► LLM intent parser (OpenRouter, free tier)
        │
        └── Spoken alerts ◄── Web Speech API (on-device text-to-speech)
```

- **Frontend**: Static HTML/JS, hosted on GitHub Pages. Uses the phone's rear camera, the Web Speech API for both speech recognition (wake word + commands) and text-to-speech (spoken alerts), and a canvas overlay for bounding boxes.
- **Backend**: Flask server running on Google Colab (free GPU), exposed publicly via ngrok so the frontend can reach it from any network.
- **Detection**: YOLOv8n with pretrained COCO weights.
- **OCR**: EasyOCR.
- **Intent parsing**: A free LLM via OpenRouter converts natural spoken phrases ("what does this say", "find my bottle") into structured commands.

This split keeps latency-critical inference on a GPU-backed server while keeping the client lightweight enough to run in any mobile browser with no installation.

## Repository Contents

| File | Purpose |
|---|---|
| `AEGIS-ModelSetup.ipynb` | Sets up and prepares the YOLO object detection model (pretrained COCO weights, with an optional fine-tuning path). Run this once to produce/verify the model used by the backend. |
| `AEGIS-Backend.ipynb` | Runs the Flask server (obstacle detection, OCR, find-object, and voice intent parsing routes) and exposes it publicly via ngrok. Run this every time you want to start a session. |
| `yolov8n.pt` | Pretrained YOLOv8-nano weights (COCO) used for object/obstacle detection. Loaded directly by both notebooks. |
| `index.html` | The frontend — open this on a phone browser (via GitHub Pages) to use AEGIS. |

## Setup Instructions

### 1. Model Setup (one-time)
1. Upload `yolov8n.pt` (included in this repo) to your Google Drive, in the same AEGIS project folder you'll use for the notebooks.
2. Open `AEGIS-ModelSetup.ipynb` in Google Colab.
3. Run all cells in order. This loads the included `yolov8n.pt` weights (pretrained on COCO) and verifies the model is ready for use — no training required to get started.
4. Mount your Google Drive when prompted so the model persists across sessions.

### 2. Backend Setup (every session)
1. Open `AEGIS-Backend.ipynb` in Google Colab.
2. Set **Runtime → Change runtime type → GPU** for faster inference.
3. Make sure `yolov8n.pt` is uploaded to the same Drive folder referenced in the notebook (or update the file path in the model-loading cell if you place it elsewhere).
4. You will need two free API credentials:
   - **ngrok auth token** — sign up free at [ngrok.com](https://ngrok.com), copy your authtoken from the dashboard, and paste it into the designated cell.
   - **OpenRouter API key** — sign up free at [openrouter.ai](https://openrouter.ai), generate a key, and paste it into the designated cell. This powers the voice command intent parser (using a free-tier LLM).
4. Run all cells in order. The final cell will print a public backend URL that looks like:
   ```
   https://xxxx-xx-xx-xx-xx.ngrok-free.app
   ```
5. **Keep this Colab notebook running** for the duration of your session — closing it or letting the runtime disconnect will take the backend offline. Note that the ngrok URL changes each time you restart the tunnel.

### 3. Frontend Setup
1. Deploy `index.html` via GitHub Pages (Settings → Pages → deploy from branch), or open it locally.
2. Open the deployed page on your phone (Android Chrome recommended for full voice support).
3. In the page, update the `BACKEND_URL` constant with the ngrok URL printed by your backend notebook.
4. Grant camera and microphone permissions when prompted.
5. Say **"Hey Guide"** followed by a command, for example:
   - *"Hey Guide, what's around me"* → obstacle alert mode
   - *"Hey Guide, read this"* → text reading mode
   - *"Hey Guide, find my bottle"* → object search mode

## Usage Notes

- Distances are communicated in footsteps rather than meters, since footstep-based framing is more intuitive for navigation without vision.
- Alerts use a smart cooldown system: the same object won't be repeatedly announced unless it moves significantly closer, a different object appears, or the situation becomes urgent (very close obstacles trigger an immediate turn instruction).
- Voice recognition and text-to-speech run via each browser's native Web Speech API — Android Chrome currently offers the most reliable support for always-on wake-word listening.

## Roadmap

This prototype implements Module 1 (Obstacle Detection) and Module 2 (Voice Interaction) of a larger six-module vision, which also includes:
- Navigation & turn-by-turn routing
- Face recognition & memory
- Local currency recognition (Pakistani banknotes)
- Ride-booking integration (Careem / Uber)

Future work includes fine-tuning detection on region-specific obstacles (stairs, curbs, local street furniture) using self-collected and labeled data, moving inference on-device for lower latency, and expanding language support for local accessibility.

## Acknowledgments

Built using [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics), [EasyOCR](https://github.com/JaidedAI/EasyOCR), [OpenRouter](https://openrouter.ai), and [ngrok](https://ngrok.com).
