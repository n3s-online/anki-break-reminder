# Anki Add-on Development Research Report
## Break Reminder Plugin

---

## Overview

Anki add-ons are Python modules that extend Anki's functionality. The desktop application is built with Python and PyQt (Qt GUI framework), making it straightforward to create plugins that integrate with the UI, respond to events, and add custom dialogs.

---

## Technical Foundation

### Language & Framework
- **Python 3** - All add-ons are Python modules
- **PyQt5/PyQt6** - GUI toolkit (Anki 2.1.50+ supports both)
- **Import from `aqt.qt`** instead of directly from PyQt for cross-version compatibility

### Key Imports
```python
from aqt import mw           # Main window object
from aqt import gui_hooks    # Event hooks system
from aqt.qt import *         # Qt widgets (QTimer, QDialog, etc.)
from aqt.utils import showInfo, showWarning  # Utility dialogs
```

---

## Add-on Structure

### Minimal Structure
```
your_addon/
├── __init__.py      # Entry point (required)
├── config.json      # Default configuration (optional)
├── config.md        # Config documentation (optional)
├── manifest.json    # For distribution outside AnkiWeb (optional)
└── user_files/      # Preserved on upgrade (optional)
```

### Installation Location
- Add-ons are installed in: `~/.local/share/Anki2/addons21/` (Linux)
- Or: `%APPDATA%\Anki2\addons21\` (Windows)
- Or: `~/Library/Application Support/Anki2/addons21/` (macOS)

---

## Hooks System (Critical for Your Plugin)

Anki uses a hooks system to let add-ons respond to application events. Key hooks for a break reminder:

| Hook | Purpose |
|------|---------|
| `gui_hooks.profile_did_open` | Fires when user profile loads - good for initialization |
| `gui_hooks.reviewer_did_show_question` | Fires when a card question is shown |
| `gui_hooks.reviewer_did_answer_card` | Fires after answering a card |
| `gui_hooks.main_window_did_init` | Fires when main window initializes |

### Hook Usage Example
```python
from aqt import gui_hooks

def on_question_shown(card):
    # Your timer logic here
    pass

gui_hooks.reviewer_did_show_question.append(on_question_shown)
```

---

## Configuration System

### config.json (Default Values)
```json
{
    "study_duration_minutes": 25,
    "break_duration_minutes": 5,
    "break_activities": [
        "Stretch for 2 minutes",
        "Get a glass of water",
        "Look out the window",
        "Do 10 jumping jacks"
    ],
    "play_sound_on_break": true,
    "play_sound_on_resume": true,
    "sound_file_break": "break.mp3",
    "sound_file_resume": "resume.mp3"
}
```

### Accessing Config
```python
from aqt import mw

config = mw.addonManager.getConfig(__name__)
study_time = config['study_duration_minutes']

# Save modified config
config['study_duration_minutes'] = 30
mw.addonManager.writeConfig(__name__, config)
```

### Custom Config Dialog
```python
def show_options():
    # Your custom dialog code
    pass

mw.addonManager.setConfigAction(__name__, show_options)
```

---

## Key Components for Break Reminder

### 1. Timer Implementation (QTimer)
```python
from aqt.qt import QTimer
from aqt import mw

class BreakReminder:
    def __init__(self):
        self.timer = QTimer(mw)
        self.timer.timeout.connect(self.on_timer_finished)
        
    def start_study_timer(self, minutes):
        self.timer.start(minutes * 60 * 1000)  # milliseconds
        
    def on_timer_finished(self):
        self.show_break_popup()
```

### 2. Popup Dialog (QDialog)
```python
from aqt.qt import QDialog, QVBoxLayout, QLabel, QPushButton, QTimer

class BreakPopup(QDialog):
    def __init__(self, parent, activity, break_duration):
        super().__init__(parent)
        self.setWindowTitle("Break Time!")
        self.break_duration = break_duration
        self.remaining = break_duration * 60
        
        layout = QVBoxLayout()
        self.activity_label = QLabel(f"Activity: {activity}")
        self.timer_label = QLabel(self.format_time(self.remaining))
        
        self.snooze_btn = QPushButton("Snooze (5 min)")
        self.start_btn = QPushButton("Start Break")
        self.dismiss_btn = QPushButton("Back to Studying")
        
        layout.addWidget(self.activity_label)
        layout.addWidget(self.timer_label)
        layout.addWidget(self.snooze_btn)
        layout.addWidget(self.start_btn)
        layout.addWidget(self.dismiss_btn)
        
        self.setLayout(layout)
        
        # Break countdown timer
        self.countdown = QTimer(self)
        self.countdown.timeout.connect(self.update_countdown)
```

### 3. Sound Playback
```python
from aqt.sound import av_player
import os

def play_sound(filename):
    addon_dir = os.path.dirname(__file__)
    sound_path = os.path.join(addon_dir, "user_files", filename)
    if os.path.exists(sound_path):
        av_player.play_file(sound_path)
```

### 4. Menu Integration
```python
from aqt import mw
from aqt.qt import QAction

def setup_menu():
    action = QAction("Break Reminder Settings", mw)
    action.triggered.connect(show_settings)
    mw.form.menuTools.addAction(action)
```

---

## Existing Similar Add-ons (Reference)

| Add-on | AnkiWeb Code | Notes |
|--------|--------------|-------|
| Pomodore/Tomato Clock | 811976365 | Basic pomodoro timer |
| Pomodium | 678094152 | Feature-rich pomodoro |
| Just Do It Timer | Various | Includes animations |

These can serve as code references for implementation patterns.

---

## Implementation Approach for Your Plugin

### Core Architecture
```
break_reminder/
├── __init__.py          # Entry point, hook registration
├── config.json          # Default settings
├── config.md            # User documentation
├── timer.py             # Timer logic class
├── dialogs.py           # Break popup, settings dialog
├── activities.py        # Activity pool management
├── sounds.py            # Sound playback utilities
└── user_files/
    ├── break.mp3        # Default break sound
    └── resume.mp3       # Default resume sound
```

### Startup Flow
1. `__init__.py` loads on Anki startup
2. Register `profile_did_open` hook to initialize timer
3. Register `reviewer_did_show_question` hook to start/track timer
4. Timer counts down during review sessions
5. On timeout → show popup with snooze/start options
6. During break → countdown display with early dismiss option
7. On break end → play sound, auto-close or require acknowledgment

---

## Development Tips

1. **Testing**: Hold Shift while starting Anki to disable all add-ons (useful for debugging)

2. **Debugging**: Use `print()` statements - output appears in terminal if Anki is run from command line

3. **IDE Setup**: Use VS Code or PyCharm with `aqt` stubs for code completion

4. **Qt Compatibility**: Always import from `aqt.qt` rather than PyQt directly

5. **Distribution**: 
   - Create `.ankiaddon` zip file (don't include parent folder)
   - Upload to ankiweb.net/shared/addons
   - Delete `__pycache__` folders before zipping

---

## Resources

- **Official Add-on Docs**: https://addon-docs.ankiweb.net/
- **Hooks Reference**: `pylib/tools/genhooks.py` and `qt/tools/genhooks_gui.py` in Anki source
- **Qt Documentation**: https://doc.qt.io/
- **PyQt Documentation**: https://www.riverbankcomputing.com/static/Docs/PyQt6/
- **Anki Source Code**: https://github.com/ankitects/anki
- **Forums**: https://forums.ankiweb.net/ (Development category)

---

## Quick Start Template

```python
# __init__.py
from aqt import mw, gui_hooks
from aqt.qt import QTimer, QDialog, QVBoxLayout, QLabel, QPushButton
import random

class BreakReminder:
    def __init__(self):
        self.config = mw.addonManager.getConfig(__name__)
        self.study_timer = QTimer(mw)
        self.study_timer.timeout.connect(self.show_break_popup)
        self.is_reviewing = False
        
    def on_profile_open(self):
        # Initialize when profile loads
        pass
        
    def on_review_start(self, card):
        if not self.study_timer.isActive():
            duration = self.config['study_duration_minutes'] * 60 * 1000
            self.study_timer.start(duration)
            
    def show_break_popup(self):
        self.study_timer.stop()
        activity = random.choice(self.config['break_activities'])
        # Show popup dialog...

# Initialize
reminder = BreakReminder()
gui_hooks.profile_did_open.append(reminder.on_profile_open)
gui_hooks.reviewer_did_show_question.append(reminder.on_review_start)
```

---

*Report generated: December 2025*
