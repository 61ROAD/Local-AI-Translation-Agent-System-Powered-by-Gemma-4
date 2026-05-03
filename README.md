# **⚡ Local AI Translation Agent System Powered by Gemma 4**

## **📌 Project Overview**

In daily development and reading, finding a translation tool that fully protects privacy while delivering an ultra-fast response is a core need for many users. To solve this challenge, I developed this **fully local, offline, zero-background-service** standalone smart translation desktop application.

Designed specifically for AI code generation agents and full-stack desktop developers who demand extreme performance, this project is driven by the latest **Gemma 4** large language model. The underlying system strictly follows a decoupled architecture separating the UI thread from the worker thread, complemented by native screen OCR and TTS technologies. It provides a minimalist UI/UX experience while ensuring millisecond-level response times and absolute system fluidity.

## **📁 Project Structure**

```
.
├── main.py                # Single entry point of the application
├── config.json            # Global config file (auto-generated or read at runtime)
├── dictionary.txt         # Local English dictionary for input box autocomplete
├── src/                   # Source code directory
│   ├── __init__.py  
│   ├── ui/                # UI components layer
│   │   ├── main_window.py # Main UI class (Input box, Markdown output area)
│   │   ├── settings_ui.py # Settings panel UI class
│   │   └── styles.qss     # Global stylesheet (minimalist design)
│   ├── core/              # Core business logic layer
│   │   ├── llm_engine.py  # llama.cpp wrapper & QThread Worker
│   │   ├── db_manager.py  # SQLite operations & local cache logic
│   │   ├── autocomplete.py# Trie/Binary search autocomplete algorithm
│   │   └── ocr_engine.py  # Windows.Media.Ocr native text extraction logic
│   └── utils/             # Utility classes
│       ├── hotkey.py      # Global hotkey registration & listener
│       └── tts_player.py  # pyttsx3 audio player independent thread
├── models/                # Directory for local models
│   └── gemma-4-e4b-q4_k_m.gguf # User-downloaded GGUF model file goes here
└── data/                  # Local user data directory
    └── app_data.db        # SQLite search history DB (auto-generated at runtime)
```

## **✨ Key Features**

- **Ultra-Smooth Immersive UI:** Features a frameless design and system-level auto-hide on blur (`Qt.FramelessWindowHint` | `Qt.WindowStaysOnTopHint`), with the output area supporting advanced Markdown rendering.
- **Global Hotkeys & Native OCR:** Supports hotkeys (e.g., `Alt+Q`) to instantly wake up a transparent overlay for screen region selection. Uses pure local Windows Native OCR API to extract text, auto-fill, and trigger translation with **0 network requests**.
- **Smart Clipboard Monitoring:** Real-time monitoring of pure text in the clipboard, with a built-in long-text filter (character limit) to prevent system lag when copying large files or massive code blocks.
- **Ultra-Fast Caching & Autocomplete:** Integrates an SQLite local cache and Trie tree algorithm. Prioritizes local history queries on user input; cache hits yield instantaneous responses (0 VRAM consumption).
- **Independent Native TTS Engine:** A multi-threaded voice reading feature based on `pyttsx3`, perfectly solving the pain point of voice playback blocking the main UI thread.

## **🧠 Technical Architecture**

To guarantee that the desktop application remains absolutely lag-free while running large language models, this system implements a strict multi-threaded concurrency model and hybrid scheduling architecture:

### **1. Software Lifecycle & Concurrency Model**

- **Approach:** Strictly adhere to the physical isolation of the **UI Thread and Worker Thread**.
- **Tech:** The Main Thread handles only PySide6 UI rendering and user interaction; LLM inference, database queries, and OCR tasks are dispatched to independent worker threads inheriting from `QThread`.
- **Optimization:** Utilizes Qt's Signal and Slot mechanism for character-by-character streaming (`stream=True`) output, ensuring stable UI frame rates.

### **2. Core AI Engine Layer (LLM Engine)**

- **Approach:** Encapsulates the LLM into an independent inference module, dynamically adjusting Prompt strategies based on input length and format (word parsing/abbreviation expansion/long sentence translation).
- **Tech:** Uses the CUDA-compiled version of `llama-cpp-python` as the inference backend.
- **Optimization:** Instantiation forces `n_gpu_layers=-1` (offloading the model 100% to VRAM, e.g., RTX 5060) and caps CPU threads, completely preventing system lag caused by CPU bottlenecks.

### **3. Ultra-Fast Cache & Data Layer**

- **Approach:** Trades space for time, avoiding redundant inference for frequently translated words.
- **Tech:** Built-in lightweight `sqlite3` database, executing a `SELECT` priority hit logic. Only dispatches tasks to the LLM Worker Thread upon a cache miss.

### **4. Autocomplete Algorithm**

- **Approach:** Listens to the input box's `textChanged` signal with a 150ms debounce delay to achieve predictive text at minimal overhead.
- **Tech:** Builds a Trie tree in memory or utilizes the `bisect` module to perform binary searches on a pre-sorted local dictionary.

### **🚀 The Translation Engine**

The final translation generation is coordinated by the system's "state machine engine." The engine automatically routes tasks based on the user's input source (clipboard, OCR, manual input) and dynamically determines whether to trigger the cache:

## **🛠️ Tech Stack**

- **Core Language:** Python 3.10+
- **UI Framework:** PySide6 (Strictly follows the signal-slot mechanism; PyQt5/Tkinter are prohibited)
- **LLM Backend:** llama-cpp-python (Hardware acceleration via CUDA required)
- **OS API:** winsdk (`winsdk.windows.media.ocr` for native optical character recognition)
- **System Interaction:** pynput / keyboard (Global Hotkeys), pyperclip (Clipboard)
- **Local Services:** sqlite3 (Cache & Data Persistence), pyttsx3 (Native TTS)
- **Packaging:** PyInstaller

## **⚙️ Installation & Usage**

1. **Clone the repository:**

   ```
   git clone [https://github.com/yourusername/Local-AI-Translation-Agent-System-Powered-by-Gemma-4.git](https://github.com/yourusername/Local-AI-Translation-Agent-System-Powered-by-Gemma-4.git)
   cd Local-AI-Translation-Agent-System-Powered-by-Gemma-4
   ```
2. **Configure Python virtual environment and install dependencies:**

   ```
   python -m venv venv
   .\venv\Scripts\activate
   pip install -r requirements.txt
   ```
   *(Note: Ensure you install the CUDA-accelerated version of `llama-cpp-python` for optimal performance)*
3. **Download model weights:** Please download the `gemma-4-e4b-q4_k_m.gguf` model yourself and place it in the `models/` folder located in the project's root directory.
4. **Run the application:**

   ```
   python main.py
   ```
