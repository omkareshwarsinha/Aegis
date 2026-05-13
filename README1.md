<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Orbitron&weight=900&size=40&duration=3000&pause=1000&color=9B6DFF&center=true&vCenter=true&width=700&lines=AEGIS+JARVIS;Advanced+Electronic+Guardian;Intelligence+System+v15" alt="AEGIS JARVIS" />

<br/>

![Version](https://img.shields.io/badge/Version-15.0_FINAL-9b6dff?style=for-the-badge&logo=android&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Android_9-3ddc84?style=for-the-badge&logo=android&logoColor=white)
![Engine](https://img.shields.io/badge/Engine-DroidScript-ff6b35?style=for-the-badge)
![AI](https://img.shields.io/badge/AI-Multi--Provider-ff4757?style=for-the-badge&logo=openai&logoColor=white)
![Language](https://img.shields.io/badge/Language-Hinglish_AI-ff9500?style=for-the-badge)
![SubApps](https://img.shields.io/badge/Sub--Apps-40%2B-00d2ff?style=for-the-badge)
![License](https://img.shields.io/badge/License-Personal-34d399?style=for-the-badge)

<br/>

```
╔═══════════════════════════════════════════════════════════════╗
║  A · E · G · I · S  —  Your Android AI Companion             ║
║  Built by Om (Omkareshwar Sinha) · DroidScript AppType HTML  ║
║  Chrome 67 WebView · Android 9 · ES5 · Hinglish First        ║
╚═══════════════════════════════════════════════════════════════╝
```

</div>

---

## ⚡ What is AEGIS?

**AEGIS** (Advanced Electronic Guardian and Intelligence System) is a **fully offline-capable Android AI assistant** built as a single HTML file running inside DroidScript's AppType HTML — giving it direct access to native Android APIs while maintaining a rich web UI.

> Talk to it in **Hinglish**. It understands. It acts. For real.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                        USER                             │
│              (Voice / Text / Touch)                     │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│              COMMAND LAYER (dashProcess)                          |
│  NLP → Intent Detection → Command Router                         │
└──────────────────────┬──────────────────────────────────┘
                       │
           ┌───────────┴───────────┐
           ▼                       ▼
┌──────────────────┐    ┌──────────────────────────────┐
│  Native Bridge   │    │     AI ENGINE (Multi-LLM)    │
│  console.log     │    │  Ollama/Gemini/GPT/Groq/Grok │
│  CMD:action:val  │    │  Pollinations/OpenRouter/HF  │
└────────┬─────────┘    └──────────────┬───────────────┘
         │                             │
         ▼                             ▼
┌──────────────────────────────────────────────────────┐
│                   Aegis.js (DS Bridge)               │
│  app.TextToSpeech · app.SendIntent · app.LaunchApp   │
│  TTS hi_IN · Torch · Call · SMS · WhatsApp · Alarm   │
└──────────────────────────────────────────────────────┘
```

---

## 🎯 Features

### 🤖 AI Providers (Switchable Live)
| Provider | Type | Speed |
|----------|------|-------|
| 🧠 **Ollama** | Local LLM | Ultra Fast |
| 🌐 **Pollinations** | Free GPT-4o | Fast |
| ✨ **Gemini** | Google AI | Fast |
| 🤖 **OpenAI** | GPT-4o/mini | Medium |
| ⚡ **Groq** | Free LLaMA | Ultra Fast |
| 𝕏 **Grok** | xAI | Fast |
| 🔀 **OpenRouter** | Multi-model | Variable |
| 🧬 **Cerebras** | Fast Inference | Ultra Fast |
| 🤗 **HuggingFace** | Open models | Medium |

### 📱 Native Android Actions (via DS Bridge)
```
📞 call Mom              → Makes real phone call
💬 WhatsApp Raj ko message → Sends WhatsApp with pre-filled text  
🔦 torch on              → Toggles flashlight
📸 screenshot            → Takes screenshot
🎵 play Arijit Singh     → Opens Spotify/YouTube
⏰ alarm 7 bajao         → Sets system alarm
📝 note: kal milk lena   → Creates note
🌐 YouTube kholo         → Launches YouTube app
📍 maps: Connaught Place → Opens Google Maps
```

### 🧩 40+ Sub-Apps
<details>
<summary>Click to expand full list</summary>

**🤖 AI & Coding**
- Aurora AI (AI vs AI conversation)
- Ollama Local Chat
- AEGIS IDE (9 AI providers)
- AI Flashcards
- AI Excel / Spreadsheet

**📱 Personal**
- Todo List · Finance Tracker
- Habit Tracker · Reminders · Alarms
- Secure Notes (XOR encrypted)
- Calendar · Clipboard Manager
- JARVIS Memory Store

**🌐 Information**
- Weather · News Feed (RSS)
- Dictionary · World Clock
- Sunrise/Sunset · Pincode Lookup

**🔧 Tools**
- Scientific Calculator
- Unit Converter · BMI · EMI · Tip Calc
- QR Generator · Pomodoro Timer
- Text Tools · Color Picker · Morse Code

**🔒 Security**
- Password Generator & Manager (XOR encrypted vault)
- AES-XOR Encrypt/Decrypt
- Hash Calculator (SHA-256/512)
- RSA Key Generator
- Steganography · JWT Decoder

**🔍 OSINT**
- OSINT Spydox (username/email)
- Vehicle OSINT (RC number plate)
- Instagram/Social OSINT
- Password Breach Checker

**🎮 Games (15+)**
- Chess · Snake · Sudoku · Connect Four
- Flappy Bird · Simon Says · Maze Game
- Typing Race · Word Chain · Whack-a-Mole
- Ping Pong · Game of Life · and more!

</details>

---

## 🚀 Installation

### Method 1: DroidScript (Recommended)

```
1. Install DroidScript from Play Store
2. Copy files to /sdcard/DroidScript/Aegis/
   ├── Aegis.html   (main app — 36,000+ lines)
   └── Aegis.js     (DS bridge — native Android)
3. Open DroidScript → Run Aegis.js
```

### Method 2: Browser Preview
```
Open Aegis.html directly in any browser
(Native Android features won't work, AI chat will)
```

### Method 3: Ollama Setup (Local AI)
```bash
# On your PC (same WiFi)
ollama pull qwen2.5:7b-instruct
ollama create aegis-jarvis -f AEGIS_Modelfile
ollama serve --host 0.0.0.0

# In AEGIS: Settings → Ollama URL → http://<your-pc-ip>:11434
```

---

## 📁 File Structure

```
DroidScript/Aegis/
├── Aegis.html          ← Main app (36,000+ lines, all-in-one)
├── Aegis.js            ← DroidScript companion (native bridge)
└── AEGIS_Modelfile     ← Ollama model definition
```

---

## 🗣️ Command Examples (Hinglish)

```
"Kal 7 baje alarm lagao"     → Sets 7AM alarm
"Raj ko call karo"           → Calls Raj from contacts
"Aaj ka mausam batao"        → Weather info
"Password generate karo"     → Opens password generator  
"Chess khelna hai"           → Opens chess game
"Meri habits dikhao"         → Opens habit tracker
"Screenshot lo"              → Takes screenshot
"Torch on karo"              → Flashlight ON
"WhatsApp Priya: party hai"  → WhatsApp pre-filled
"Kitna battery hai"          → Battery status
```

---

## 🔧 Tech Stack

```
Frontend   : HTML5 + CSS3 (glassmorphism) + Vanilla JS (ES5)
AI Bridge  : XMLHttpRequest (no fetch — Chrome 67 file:// compat)
Storage    : localStorage (encrypted vault, memories, contacts)
TTS        : DroidScript TextToSpeech (hi_IN locale, music stream)
Native     : console.log bridge → DS intercepts → Android APIs
Crypto     : XOR cipher + djb2 key stretching (no crypto.subtle)
WebView    : Chrome 67 (DroidScript AppType HTML, Android 9)
```

---

## ⚙️ Modelfile Usage

```bash
ollama create aegis-jarvis -f AEGIS_Modelfile
ollama run aegis-jarvis

# In AEGIS app:
# Settings → AI Provider → Ollama
# Model: aegis-jarvis
# URL: http://localhost:11434 (or ngrok URL)
```

---

## 🛠️ Developer Notes

### ES5 Strict Compliance (Chrome 67 WebView Rules)
```javascript
// ❌ NEVER use these in Aegis.html
const x = ...          // → var x = ...
let y = ...            // → var y = ...
x?.method()            // → x && x.method()
x ?? y                 // → x !== null && x !== undefined ? x : y
async/await            // → XHR callbacks
fetch()                // → XMLHttpRequest
crypto.subtle          // → XOR cipher fallback
...new Uint8Array()    // → for loop

// ✅ DroidScript Bridge Pattern
console.log('SPEAK:Namaste');           // TTS → hindi
console.log('CMD:launchapp:com.pkg');   // Open Android app
console.log('CMD:call:9876543210');     // Phone call
console.log('CMD:torch:on');            // Flashlight
```

---

## 📊 Stats

```
Lines of Code    : 36,521
Sub-applications : 40+
AI Providers     : 9
Games            : 15+
OSINT Tools      : 3
Security Tools   : 6
Node.js Syntax   : ✅ ZERO ERRORS
```

---

## 👨‍💻 Built By

<div align="center">

**Om (Omkareshwar Sinha)**  
Student Developer · Android · AI · DroidScript  
India 🇮🇳

*"Ek banda, ek phone, ek AI jo sach mein kaam kare."*

</div>

---

<div align="center">

![AEGIS](https://img.shields.io/badge/AEGIS-v15_FINAL-9b6dff?style=for-the-badge)
![Made with](https://img.shields.io/badge/Made_with-💜_by_Om-ff4757?style=for-the-badge)

**⭐ Agar helpful lage toh star karo bhai!**

</div>
