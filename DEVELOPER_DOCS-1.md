# AEGIS Developer Documentation
**Version:** 4.0 | **Platform:** Android 9+ / DroidScript | **Author:** Omkareshwar Sinha

---

## Table of Contents
1. [Project Overview](#overview)
2. [Architecture Deep Dive](#architecture)
3. [File Structure](#files)
4. [Command System Reference](#commands)
5. [AI Provider Integration](#ai)
6. [Intent System](#intents)
7. [TTS Engine](#tts)
8. [Custom App Creator](#custom-apps)
9. [Plugin System](#plugins)
10. [DroidScript Bridge](#bridge)
11. [NLP Command Map](#nlp)
12. [Troubleshooting](#troubleshoot)
13. [Adding New Commands](#new-commands)

---

## 1. Project Overview {#overview}

AEGIS is a single-page AI assistant running inside DroidScript's AppType HTML (Simple template) on Android 9+. It uses a layered pipeline:

```
User Input → NLP Check → AI Provider → Command Parser → Intent Executor → TTS
```

**Key Design Decisions:**
- **No external JS files in HTML** — everything embedded in `Aegis.html`
- **`Aegis.js`** is the DroidScript companion that sets up the WebView and bridges console.log messages
- **`app.*` calls** happen directly from the HTML (via the WebView→Native bridge)
- **XMLHttpRequest only** — no fetch() (broken in Chrome 67 WebView on Android 9)

---

## 2. Architecture Deep Dive {#architecture}

### 2.1 The Pipeline

```javascript
// ENTRY POINT
function dashProcess(input) {
  // STEP 1: Lock — prevent double response
  if (_dashBusy) return;
  _dashBusy = true;
  // Disable send button for 6 seconds
  
  // STEP 2: Show user message
  _showMsg('user', input);
  
  // STEP 3: NLP local check
  var localCmd = _nlpCheck(input);
  if (localCmd) {
    _runCmd(localCmd);        // execute directly
    _tts('Done!');            // TTS feedback
    _dashBusy = false;
    return;                   // STOP — don't call AI
  }
  
  // STEP 4: AI call (single path via _jarvisAI.ask)
  _callAI(input, function(reply, source) {
    _dashBusy = false;
    
    // STEP 5: Parse AI response for commands
    var parsed = _parseAIResponse(reply);
    
    // STEP 6: Execute commands
    if (parsed.multiCmds.length) {
      parsed.multiCmds.forEach((cmd, i) => 
        setTimeout(() => _runCmd(cmd), i * 500)
      );
    } else if (parsed.cmd) {
      _runCmd(parsed.cmd);
    }
    
    // STEP 7: Show text + TTS
    _showMsg('bot', parsed.text);
    _aegisTTS(parsed.text);
  });
}
```

### 2.2 Dual Response Prevention

The most critical bug was dual responses. Fixed by:

```javascript
// _callAI routes ONLY through _jarvisAI.ask — never falls through
function _callAI(q, cb) {
  if (window._jarvisAI && typeof _jarvisAI.ask === 'function') {
    _jarvisAI.ask(q, cb);
    return;  // HARD RETURN — nothing else fires
  }
  // Only reaches here if _jarvisAI unavailable
}

// _dashBusy global lock
var _dashBusy = false;
// Set true on input, reset after AI replies or 6s timeout
```

### 2.3 Intent Execution

All commands go through `window._aegisRunSingleCmd` (alias `_runCmd`):

```javascript
// From Pipeline v5 — injected at page end
function _runCmd(cmd) {
  var parts = cmd.split(':');
  var action = parts[0].toLowerCase();
  
  switch(action) {
    case 'open':    launchApp(parts[1]); break;
    case 'call':    makeCall(parts.slice(1).join(':')); break;
    case 'whatsapp': handleWhatsApp(parts); break;
    // ... 30+ cases
  }
}
```

---

## 3. File Structure {#files}

```
/sdcard/DroidScript/Aegis/
├── Aegis.html              (Main file — ~2MB, 37000+ lines)
│   ├── <head> CSS          Lines 1-1200
│   ├── HTML structure      Lines 1200-2000
│   ├── Core JS:
│   │   ├── _showMsg()      Line ~2308  — render chat messages
│   │   ├── _tts()          Line ~2637  — TTS function
│   │   ├── _launchPkg()    Line ~2675  — app launch helper
│   │   ├── _execJarvisCmd() Line ~2685 — command router
│   │   ├── dashProcess()   Line ~3340  — main pipeline
│   │   ├── _dashCfg{}      Line ~3420  — dashboard config
│   ├── Sub-app sections    Lines 4000-30000
│   ├── _jarvisAI object    Line ~27428
│   ├── _forceInjectSubApps Line ~36582
│   └── Pipeline v5 script  Last ~200 lines
│
└── Aegis.js                (DroidScript bridge — 681 lines)
    ├── OnStart()           WebView setup + permissions
    ├── _onConsole()        Bridge dispatcher
    ├── _routeJarvisCmd()   CMD router
    ├── _launchApp()        4-method launch
    ├── _setFlashlight()    Torch control
    └── _hindiTTSFix()      Hindi word replacement
```

---

## 4. Command System Reference {#commands}

### Format
All commands use colon-separated format: `action:param1:param2:...`

### Complete Command Table

| Command | Format | DroidScript API | Notes |
|---------|--------|-----------------|-------|
| Open App | `open:pkg` | `app.LaunchApp(pkg)` | 5-method fallback |
| Call | `call:number` | `app.SendIntent(CALL, tel:num)` | Needs CALL_PHONE permission |
| WhatsApp msg | `whatsapp:number:NUM:MSG` | `app.SendIntent(whatsapp://)` | Direct scheme |
| WhatsApp contact | `whatsapp:contact:NAME:MSG` | `app.SendIntent(whatsapp://)` | Looks up localStorage |
| YouTube search | `youtube:search:QUERY` | `app.SendIntent(SEARCH, query[])` | Fallback to URL |
| Spotify search | `spotify:search:QUERY` | `app.SendIntent(spotify:search:)` | Deep link |
| Chrome search | `chrome:search:QUERY` | `app.SendIntent(VIEW, google.com)` | |
| Chrome URL | `chrome:url:URL` | `app.SendIntent(VIEW, url)` | |
| Maps navigate | `maps:navigate:PLACE` | `app.SendIntent(VIEW, geo:0,0?q=)` | |
| Instagram profile | `instagram:profile:USER` | `app.SendIntent(VIEW, ig.com/USER)` | |
| Telegram msg | `telegram:msg:USER:MSG` | `app.SendIntent(VIEW, tg://resolve)` | |
| Torch on | `torch:on` | `app.SetFlashlight(true)` | Fallback: system intent |
| Torch off | `torch:off` | `app.SetFlashlight(false)` | |
| WiFi on | `wifi:on` | `app.SetWifiEnabled(true)` | Deprecated Android 10+ |
| WiFi off | `wifi:off` | `app.SetWifiEnabled(false)` | |
| Bluetooth on | `bluetooth:on` | `app.SetBluetoothEnabled(true)` | |
| Volume set | `volume:5` | `app.SetVolume('Music', 5)` | Range: 0-15 |
| Volume up | `volume:up` | `app.SetVolume('Music', cur+2)` | |
| Volume mute | `volume:mute` | `app.SetVolume('Music', 0)` | |
| Screenshot | `screenshot` | `app.TakeScreenshot(path)` | Fallback: system intent |
| Silent | `silent` | `app.SetRingerMode('silent')` | |
| Vibrate | `vibrate` | `app.SetRingerMode('vibrate')` | |
| Brightness | `brightness:50` | `app.SetScreenBrightness(0.5)` | Range: 0-100 |
| Alarm | `alarm:set:07:30` | `app.SendIntent(SET_ALARM, extras)` | |
| Timer | `timer:5min` | `app.SendIntent(SET_TIMER, extras)` | |
| Camera | `camera` | `app.SendIntent(IMAGE_CAPTURE)` | |
| Video | `camera:video` | `app.SendIntent(VIDEO_CAPTURE)` | |
| SMS | `sms:NUM:MSG` | `app.SendIntent(SENDTO, smsto:)` | |
| Copy | `copy:TEXT` | `app.SetClipboard(text)` | |
| Share | `share:TEXT` | `app.SendIntent(SEND, text/plain)` | |
| Lock | `lock` | `app.LockScreen()` | |
| Play Store | `playstore:PKG` | `app.SendIntent(VIEW, market://)` | |
| Settings | `settings:wifi` | `app.SendIntent(WIFI_SETTINGS)` | |
| Multi cmd | `multi:cmd1\|cmd2\|cmd3` | Sequential, 500ms gap | |
| Parallel | `parallel:cmd1\|cmd2` | All fire at once | |
| Split screen | `splitscreen:pkg1\|pkg2` | LAUNCH_ADJACENT flag | |
| Recent apps | `recent` | TOGGLE_RECENTS intent | |
| Home | `home` | HOME category intent | |
| Raw DS | `ds:pkg:action:cat:uri` | `app.SendIntent(...)` | Advanced |

---

## 5. AI Provider Integration {#ai}

### Supported Providers

| Provider | Endpoint | Auth | Notes |
|----------|----------|------|-------|
| Ollama Local | `http://localhost:11434/api/chat` | None | Fully offline |
| Ollama Cloud | Custom URL | None/Bearer | Remote Ollama |
| Pollinations | `https://text.pollinations.ai/` | None | Free, no key |
| Gemini | `https://generativelanguage.googleapis.com/...` | API Key | Fast |
| Groq | `https://api.groq.com/openai/v1/chat/completions` | Bearer | Ultra-fast |
| OpenAI | `https://api.openai.com/v1/chat/completions` | Bearer | |
| OpenRouter | `https://openrouter.ai/api/v1/chat/completions` | Bearer | Multi-model |
| Grok | `https://api.x.ai/v1/chat/completions` | Bearer | |
| Cerebras | `https://api.cerebras.ai/v1/chat/completions` | Bearer | Fast |

### System Prompt Structure

```
You are AEGIS, Android AI assistant.
Language: Hinglish.

When device action needed:
Single: {"jarvis_cmd":"command"}
Multi:  {"jarvis_cmd_multi":["cmd1","cmd2"]}

Available commands: [full list...]

TTS Rules:
- Replace robotic English with natural Hindi
- No markdown, URLs, JSON in spoken text
```

### Request Format (XMLHttpRequest only)

```javascript
// Ollama example
var xhr = new XMLHttpRequest();
xhr.open('POST', 'http://localhost:11434/api/chat', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.timeout = 6000; // 6 second timeout
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    if (xhr.status === 200) {
      var data = JSON.parse(xhr.responseText);
      var reply = data.message.content;
      callback(reply, 'ollama');
    } else {
      callback(null, 'error');
    }
  }
};
xhr.send(JSON.stringify({
  model: selectedModel,
  messages: conversationHistory,
  stream: false
}));
```

---

## 6. Intent System {#intents}

### App Launch — 5 Method Chain

```javascript
function launchApp(pkg) {
  var ok = false;
  // Method 1: DroidScript native (fastest)
  if(!ok) try { app.LaunchApp(pkg); ok=true; } catch(e) {}
  
  // Method 2: MAIN + LAUNCHER + NEW_TASK (most compatible)
  if(!ok) try {
    app.SendIntent(pkg, null, 'android.intent.action.MAIN',
      'android.intent.category.LAUNCHER', null, null, 'flag:0x10000000');
    ok = true;
  } catch(e) {}
  
  // Method 3: MAIN without category
  if(!ok) try {
    app.SendIntent(pkg, null, 'android.intent.action.MAIN',
      null, null, null, 'flag:0x10000000');
    ok = true;
  } catch(e) {}
  
  // Method 4: intent:// URI scheme (Chrome 67 Android 9)
  if(!ok) try {
    app.SendIntent(null, null, 'android.intent.action.VIEW', null,
      'intent://' + pkg + '#Intent;scheme=android-app;end', null, null);
    ok = true;
  } catch(e) {}
  
  // Method 5: Play Store (app not installed)
  if(!ok) app.SendIntent(null, null, 'android.intent.action.VIEW', null,
    'market://details?id=' + pkg, null, null);
}
```

### WhatsApp — Best Practices for Android 9

```javascript
// ALWAYS use whatsapp:// scheme first, wa.me as fallback
var uri = 'whatsapp://send?phone=' + phone + '&text=' + encodeURIComponent(msg);

// Try 1: Direct to WhatsApp package
app.SendIntent('com.whatsapp', null, 'android.intent.action.VIEW',
  null, uri, null, null);

// Try 2: wa.me (opens in browser if WhatsApp not installed)
app.SendIntent(null, null, 'android.intent.action.VIEW', null,
  'https://wa.me/' + phone + '?text=' + encodeURIComponent(msg), null, null);
```

### Android Settings Intents

```javascript
var SETTINGS_INTENTS = {
  'wifi':         'android.settings.WIFI_SETTINGS',
  'bluetooth':    'android.settings.BLUETOOTH_SETTINGS',
  'location':     'android.settings.LOCATION_SOURCE_SETTINGS',
  'sound':        'android.settings.SOUND_SETTINGS',
  'display':      'android.settings.DISPLAY_SETTINGS',
  'security':     'android.settings.SECURITY_SETTINGS',
  'apps':         'android.settings.MANAGE_APPLICATIONS_SETTINGS',
  'battery':      'android.settings.BATTERY_SAVER_SETTINGS',
  'storage':      'android.settings.INTERNAL_STORAGE_SETTINGS',
  'network':      'android.settings.WIRELESS_SETTINGS',
  'airplane':     'android.settings.AIRPLANE_MODE_SETTINGS',
  'language':     'android.settings.LOCALE_SETTINGS',
  'about':        'android.settings.DEVICE_INFO_SETTINGS',
  'developer':    'android.settings.APPLICATION_DEVELOPMENT_SETTINGS',
  'data':         'android.settings.DATA_USAGE_SETTINGS',
  'nfc':          'android.settings.NFC_SETTINGS',
  'notification': 'android.settings.NOTIFICATION_POLICY_ACCESS_SETTINGS'
};
```

### Multitasking Flags

```javascript
// Important Android flags for multitasking (7th param in SendIntent)
'flag:0x10000000'  // FLAG_ACTIVITY_NEW_TASK — required for new task
'flag:0x04000000'  // FLAG_ACTIVITY_CLEAR_TOP — bring existing to front
'flag:0x00002000'  // FLAG_ACTIVITY_LAUNCH_ADJACENT — split screen
'flag:0x10002000'  // NEW_TASK + LAUNCH_ADJACENT (split screen combo)
'flag:0x08000000'  // FLAG_ACTIVITY_NO_HISTORY
```

---

## 7. TTS Engine {#tts}

### TTS Function

```javascript
window._aegisTTS = function(text) {
  if (!text) return;
  
  // SKIP: code blocks — never read code aloud
  if ((text.match(/```/g) || []).length >= 2) {
    text = text.replace(/```[\s\S]*?```/g, 'code block.');
  }
  
  // CLEAN: remove JSON, markdown, URLs, emojis
  var clean = text
    .replace(/\{[\s\S]*?"jarvis_/g, '')
    .replace(/[*_#>`|]/g, '')
    .replace(/https?:\/\/\S+/g, '')
    .replace(/\s+/g, ' ').trim()
    .substring(0, 280);
  
  if (!clean || clean.length < 2) return;
  
  // DroidScript TTS
  try {
    app.TextToSpeech(clean, 1.05, 0.85, null, 'music', 'hi_IN');
  } catch(e) {
    try { app.TextToSpeech(clean, 1, 1); } catch(e2) {}
  }
};
```

### Hindi Word Replacements

```javascript
var HINDI_FIXES = {
  'opening':    'khol raha hoon',
  'calling':    'call kar raha hoon',
  'playing':    'chala raha hoon',
  'searching':  'dhundh raha hoon',
  'loading':    'load ho raha hai',
  'sending':    'bhej raha hoon',
  'processing': 'kar raha hoon',
  'screenshot': 'screenshot le raha hoon',
  'AEGIS':      'Ayjis',
  'WiFi':       'vaayefaaye',
  'OK':         'theek hai',
  'AI':         'aye aaye'
};
```

### TTS Parameters

```javascript
app.TextToSpeech(
  text,          // String — the text to speak
  1.05,          // pitch — 1.0 is normal, higher = more feminine
  0.85,          // rate — slower for clearer Hindi
  null,          // callback — null is fine
  'music',       // stream — 'music' or 'ring'
  'hi_IN'        // locale — underscore NOT hyphen for DroidScript!
);
```

---

## 8. Custom App Creator {#custom-apps}

### How It Works

1. User clicks **+** in sidebar
2. `openCustomAppCreator()` builds a modal (not `window.open` — that's blocked in WebView)
3. User writes HTML/JS code
4. `_saveCustomApp()` stores in `localStorage['aegis_custom_apps_v1']`
5. `_showAppInOverlay()` renders app in `<iframe srcdoc="...">` (sandboxed)

### Storage Format

```javascript
// localStorage key: 'aegis_custom_apps_v1'
[
  {
    id: 'capp_1715123456789',
    name: 'My Calculator',
    icon: '🔢',
    code: '<!DOCTYPE html>...',
    created: 1715123456789,
    updated: 1715123456789
  }
]
```

### Iframe Security

```html
<!-- Sandbox attributes — safe for user content -->
<iframe 
  sandbox="allow-scripts allow-forms allow-same-origin allow-popups"
  srcdoc="USER_HTML_CODE"
></iframe>
```

**Why `srcdoc` not `src`?** — `window.open()` and file:// URLs are blocked in DroidScript WebView. `srcdoc` directly embeds HTML into the iframe, no external URL needed.

---

## 9. Plugin System {#plugins}

### Plugin Format (AI-generated)

When AI returns a plugin, it includes:
```json
{
  "jarvis_plugin": {
    "name": "Battery Check",
    "trigger": "battery dekho",
    "code": "try { var b = app.GetBatteryLevel(); J.addMsg('bot', 'Battery: ' + b + '%'); } catch(e) {}"
  }
}
```

### Plugin Storage

```javascript
// localStorage key: 'aegis_plugins_v2'
[
  {
    name: 'Battery Check',
    trigger: 'battery dekho',
    code: 'try { var b = app.GetBatteryLevel(); ... } catch(e) {}'
  }
]
```

### Plugin Execution

```javascript
// Plugin code has access to:
// - app.* (DroidScript APIs)
// - window.J (AEGIS object)
// - J.addMsg(who, text) — add chat message
// - J.toast(text, type) — show toast
// - window._runCmd(cmd) — execute command
```

---

## 10. DroidScript Bridge {#bridge}

### WebView Setup (Aegis.js)

```javascript
function OnStart() {
  // Critical settings for DroidScript AppType HTML
  web = app.CreateWebView(1, 1, 
    "AllowJavascript,HardwareAccelerated,IgnoreErrors,EnableZoom");
  
  // AdjustResize = keyboard pushes content up (not overlay)
  web.AdjustResize(true);
  
  // Enable localStorage
  web.SetDomStorageEnabled(true);
  
  // Console bridge (HTML → DroidScript)
  web.SetOnConsole(_onConsole);
  
  web.LoadUrl("file:///sdcard/DroidScript/Aegis/Aegis.html");
  app.AddLayout(web);
}
```

### Console Message Formats

```javascript
// TTS
console.log("SPEAK:text to speak");

// Intent (JSON format)
console.log('AEGIS_INTENT:' + JSON.stringify({
  cmd: 'launch',
  pkg: 'com.whatsapp'
}));

// Intent (raw code format)
console.log('AEGIS_INTENT:app.SendIntent("com.whatsapp", null, ...)');

// Legacy CMD
console.log('CMD:torch:on');
```

### Permissions Requested at Startup

```javascript
var PERMISSIONS = [
  'RECORD_AUDIO',           // Voice input
  'CAMERA',                 // Camera + flashlight
  'CALL_PHONE',             // Direct calls
  'READ_CONTACTS',          // Contact lookup
  'WRITE_EXTERNAL_STORAGE', // Screenshots
  'ACCESS_FINE_LOCATION'    // Location features
];
perms.forEach(p => app.RequestPermission(p, null));
```

---

## 11. NLP Command Map {#nlp}

### Pattern Format

```javascript
var NLP_MAP = [
  // [regex_pattern, command_string]
  [/whatsapp\s*(kholo|open|launch)/i, 'open:com.whatsapp'],
  [/torch\s*(on|jala|켜|start)/i,     'torch:on'],
  [/torch\s*(off|band|bujha)/i,       'torch:off'],
  [/screenshot/i,                      'screenshot'],
  [/volume\s*(up|badha)/i,            'volume:up'],
  [/volume\s*(down|kam|ghata)/i,      'volume:down'],
  [/wifi\s*(on|chalu)/i,              'wifi:on'],
  [/wifi\s*(off|band)/i,              'wifi:off'],
  // ... 150+ patterns
];
```

### Multitask Splitting

```javascript
// Split on these connectors
var SPLIT_WORDS = [
  'aur', 'and', 'then', 'phir', 'fir', 'bhi',
  'uske baad', 'iske baad', 'ke saath', 'also', 'also do'
];

function _splitMultitask(input) {
  var regex = new RegExp('\\b(' + SPLIT_WORDS.join('|') + ')\\b', 'gi');
  return input.split(regex).filter(s => s.trim() && !SPLIT_WORDS.includes(s.toLowerCase().trim()));
}
```

---

## 12. Troubleshooting {#troubleshoot}

### App says "done" but nothing happens

**Root cause:** The `_si()` helper had wrong parameter order. Fixed in v4.

```javascript
// WRONG (old buggy code)
function _si(action, data, pkg) {
  app.SendIntent(action, null, data, null, null, pkg||null, null);
  //             ^ this was sending action as pkg!
}

// CORRECT (fixed)
function _si(action, uri, pkg) {
  app.SendIntent(pkg||null, null, action, null, uri||null, null, null);
}
```

### Double AI response

**Cause:** `_dashProcessV5Hooked` was wrapping `dashProcess` with another `_callAI` call.
**Fix:** Removed the patcher. `_callAI` now returns immediately after `_jarvisAI.ask()`.

### Keyboard hides input box

**Fix:** `AdjustResize(true)` in Aegis.js + `visualViewport` resize listener in HTML.

```javascript
if (window.visualViewport) {
  window.visualViewport.addEventListener('resize', function() {
    var dash = document.getElementById('dashboard');
    dash.style.height = window.visualViewport.height + 'px';
  });
}
```

### Vivo-specific issues

- **iManager killing app:** Settings → Power → Manage → DroidScript → No restrictions
- **Flashlight not working:** `app.SetFlashlight()` not documented; use system intent fallback
- **localStorage not persisting:** Ensure `web.SetDomStorageEnabled(true)` in Aegis.js

### TTS not speaking

```javascript
// Common fixes:
// 1. Locale uses underscore, not hyphen
app.TextToSpeech(text, 1.05, 0.85, null, 'music', 'hi_IN');  // ✅
app.TextToSpeech(text, 1.05, 0.85, null, 'music', 'hi-IN');  // ❌

// 2. Wrap in try/catch
try { app.TextToSpeech(clean, 1.05, 0.85, null, 'music', 'hi_IN'); }
catch(e) { try { app.TextToSpeech(clean, 1, 1); } catch(e2) {} }

// 3. Check Google TTS is installed on device
```

---

## 13. Adding New Commands {#new-commands}

### Step 1: Add to `_runCmd` in Pipeline v5

```javascript
} else if (action === 'mynewcmd') {
  // p1 = first param, p2 = second, rest = everything after action
  app.SendIntent(null, null, 'android.intent.action.YOUR_ACTION',
    null, 'your://uri', null, null);
```

### Step 2: Add NLP patterns

```javascript
[/my new command pattern/i, 'mynewcmd:param'],
```

### Step 3: Add to Modelfile system prompt

```
{"jarvis_cmd":"mynewcmd:param"}  — description of when to use
```

### Step 4: Add to Aegis.js `_routeJarvisCmd` (backup bridge)

```javascript
} else if (act === 'mynewcmd') {
  // DroidScript code
}
```

---

## API Quick Reference

```javascript
// Most used DroidScript APIs in AEGIS:

app.LaunchApp(packageName)                          // Launch app
app.SendIntent(pkg, cat, action, type, uri, extras, flags)  // Send intent
app.MakeCall(phoneNumber)                           // Phone call
app.SetFlashlight(boolean)                          // Torch
app.SetWifiEnabled(boolean)                         // WiFi
app.SetBluetoothEnabled(boolean)                    // Bluetooth
app.SetVolume(stream, value)                        // Volume (stream: "Music","Ring")
app.GetVolume(stream)                               // Get volume
app.SetScreenBrightness(0.0-1.0)                    // Brightness
app.SetRingerMode("silent"|"vibrate"|"normal")      // Ringer
app.TakeScreenshot(filePath)                        // Screenshot
app.TextToSpeech(text, pitch, rate, cb, stream, locale) // TTS
app.SetClipboard(text)                              // Copy to clipboard
app.MakeFolder(path)                                // Create directory
app.RequestPermission(name, callback)               // Runtime permission
app.Debug(message)                                  // Log to DroidScript console
```

---

*AEGIS Developer Docs v4.0 — Omkareshwar Sinha*
