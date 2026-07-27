# 🏋️‍♂️ AI Gym Trainer — Forge AI Coach

A **real-time AI fitness coach** built with Streamlit that watches you exercise through your webcam, evaluates your form using computer-vision pose estimation, and gives you **live, spoken feedback** like a real personal trainer — powered by an LLM voice-coaching pipeline.

---

## ✨ Features

- 🎥 **Real-time webcam analysis** — live video streamed and processed frame-by-frame in the browser (no video upload needed).
- 🤖 **Pose-based form detection** — tracks joint angles (knee, elbow, back, hip, torso) to judge exercise form for each rep.
- 🔢 **Automatic rep & set counting** — counts reps per set and tracks progress against a workout plan (sets × reps) you define upfront.
- 🗣️ **AI voice coaching** — an LLM (via Groq) generates contextual coaching feedback (e.g. on workout start, bad form, set/workout completion), which is converted to speech and auto-played in-browser.
- 🏋️ **Multi-exercise support** — Squats, Push-ups, Biceps Curls (Dumbbell), Shoulder Press, and Lunges, each with its own set of tracked form metrics.
- 📊 **Live metrics dashboard** — sidebar shows per-exercise metrics in real time (e.g. Knee Angle, Depth Status, Body Alignment, Swing Detection, Back Arch, Balance Status).
- 🔐 **Login wall** — simple authentication gate before accessing the trainer.
- 🗄️ **Workout history & persistence** — every session is saved to a local database and displayed as an aggregated history table (reps, sets, and time per exercise per day).
- 🎨 **Custom UI theming** — custom CSS and a bundled font are injected into the Streamlit app for a more polished look.

---

## 🧠 How It Works

1. The user logs in and sets up a **workout plan** (exercise, target sets, target reps) from the sidebar.
2. On clicking **Start Workout**, the app activates the webcam via `streamlit-webrtc` and starts streaming video to a custom `VideoProcessor`.
3. Each frame is analyzed by a pose-estimation pipeline that extracts body landmarks and derives joint angles specific to the selected exercise.
4. Exercise-specific detector logic evaluates form quality (e.g. is the squat deep enough, is the back straight, is there swing on a curl) and increments the rep/set counters.
5. Key events (workout started, a set completed, poor form detected, workout completed) are sent through a **voice coaching pipeline**:
   - An **LLM (Groq)** generates a short, natural-language coaching message for the event.
   - The message is converted to speech (**gTTS**) and auto-played in the browser.
6. Live metrics and progress are rendered in the sidebar in real time via Streamlit's session state.
7. When the workout ends, the session's reps/sets/time are persisted to a local database, and the **Workout History** table (aggregated by exercise and date) is updated using pandas.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **App framework / UI** | [Streamlit](https://streamlit.io/) |
| **Real-time video** | [streamlit-webrtc](https://github.com/whitphx/streamlit-webrtc) (WebRTC + STUN server for browser-based live video) |
| **Pose estimation / Computer vision** | [MediaPipe](https://developers.google.com/mediapipe), [OpenCV](https://opencv.org/) (`opencv-python-headless`) |
| **Numerical / data processing** | NumPy, pandas |
| **AI coaching (LLM)** | [Groq API](https://groq.com/) (`groq` Python SDK) for real-time LLM-generated coaching feedback |
| **Text-to-Speech** | [gTTS](https://pypi.org/project/gTTS/) (Google Text-to-Speech) |
| **Persistence** | Local database layer for users' exercise history (custom repository module) |
| **Config / secrets management** | `python-dotenv`, Streamlit secrets |
| **Styling** | Custom CSS + custom font injection (`static/style.css`, `static/AdobeClean.otf`) |

**Core dependencies (`requirements.txt`):**
```
streamlit==1.54.0
streamlit-webrtc==0.64.5
mediapipe==0.10.21
opencv-python-headless==4.10.0.84
numpy==1.26.4
pandas==2.2.3
groq>=0.12.0
gtts==2.5.3
python-dotenv==1.2.2
```

> System-level packages required for video/audio processing are declared in `packages.txt` (used for platforms like Streamlit Community Cloud that need OS-level libraries alongside Python packages).

---

## 📁 Project Structure

```
AI_Gym_Trainer/
├── main.py                  # Streamlit entry point — UI, session/state wiring, workout flow
├── core/                    # Core application logic / shared utilities
├── detectors/                # Per-exercise pose/form detection logic (angle & rep-counting rules)
├── ml_models/                # Pose-estimation / ML model assets used by the detectors
├── services/
│   ├── auth/                # Login wall / authentication (login_wall.py)
│   ├── state/                # Streamlit session-state defaults (session_defaults.py)
│   ├── config/                # Exercise options & workout configuration (workout_config.py)
│   ├── ui/                     # CSS/font loading & WebRTC style injection (style_loader.py)
│   ├── persistence/         # Exercise history DB layer (exercise_repository.py)
│   ├── vision/                 # Real-time video processor for pose analysis (exercise_video_processor.py)
│   ├── tracking/              # Live metrics syncing from the video processor (metrics.py)
│   └── coaching/              # AI voice coach: LLM + TTS + orchestration
│       ├── llm.py             # LLMCoach — generates feedback via Groq
│       ├── tts.py             # TextToSpeech — converts feedback to audio
│       └── voice_pipeline.py  # VoicePipeline — orchestrates LLM → TTS → autoplay
├── static/                  # CSS and font assets for custom UI styling
├── requirements.txt          # Python dependencies
├── packages.txt              # OS-level dependencies (for cloud deployment)
└── .gitignore
```

---

## 🏋️ Supported Exercises & Tracked Metrics

| Exercise | Metrics Tracked |
|---|---|
| **Squats** | Knee Angle, Back Angle, Depth Status |
| **Push-ups** | Elbow Angle, Body Alignment, Hip Position |
| **Biceps Curls (Dumbbell)** | Elbow Angle, Shoulder Stability, Swing Detection |
| **Shoulder Press** | Elbow Angle, Arm Extension, Back Arch |
| **Lunges** | Front Knee Angle, Torso Angle, Balance Status |

---

## 🚀 Getting Started

### Prerequisites
- Python 3.9+
- A webcam
- A [Groq API key](https://console.groq.com/) (for the AI voice coach)

### Installation

```bash
# Clone the repository
git clone https://github.com/Rachgit28/AI_Gym_Trainer.git
cd AI_Gym_Trainer

# (Recommended) create a virtual environment
python -m venv venv
source venv/bin/activate    # on Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Configuration

Create a `.env` file in the project root with your Groq API key:

```
GROQ_API_KEY=your_groq_api_key_here
```

Alternatively, if deploying on Streamlit Community Cloud, add `GROQ_API_KEY` to your app's **Secrets**.

### Run the app

```bash
streamlit run main.py
```

Then open the local URL Streamlit prints (usually `http://localhost:8501`), log in, set your workout plan (exercise + sets + reps), and click **Start Workout** to activate your camera and begin your session.

---

## 🗺️ Roadmap Ideas

- Additional exercises and form-check rules
- Personalized workout plans based on history
- Multi-user analytics dashboard
- Mobile-friendly camera support

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome. Feel free to open an issue or submit a pull request.

## 📄 License

No license has been specified for this repository yet. Consider adding one (e.g. MIT) if you intend for others to reuse this code.
