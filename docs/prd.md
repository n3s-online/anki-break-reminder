# Product Requirements Document
## Anki Break Reminder Add-on

---

## 1. Overview

**Product Name:** Break Reminder for Anki  
**Version:** 1.0  
**Author:** Will  
**Target Platform:** Anki Desktop (2.1.50+)

### Problem Statement
Long Anki study sessions lead to mental fatigue and reduced retention. Users often lose track of time and skip necessary breaks, resulting in burnout and decreased learning effectiveness.

### Solution
An add-on that automatically tracks study time and prompts users to take breaks with suggested activities, improving focus and long-term study sustainability.

---

## 2. User Stories

| ID | As a... | I want to... | So that... |
|----|---------|--------------|------------|
| US1 | Student | Get automatic break reminders | I don't burn out during long study sessions |
| US2 | User | Configure how often breaks occur | I can match my personal focus capacity |
| US3 | User | See suggested break activities | I take effective, refreshing breaks |
| US4 | User | Snooze a break reminder | I can finish my current card/thought |
| US5 | User | End a break early | I can return to studying when ready |
| US6 | User | Hear audio cues | I notice break start/end even if not looking |
| US7 | User | Customize the activity pool | Breaks feel relevant to me |

---

## 3. Features

### 3.1 Core Features (MVP)

| Feature | Description | Priority |
|---------|-------------|----------|
| **Auto-start Timer** | Timer begins when user starts reviewing cards | P0 |
| **Break Popup** | Modal dialog appears when study timer completes | P0 |
| **Break Countdown** | Visual timer showing remaining break time | P0 |
| **Snooze Button** | Delay break by configurable amount (default 5 min) | P0 |
| **Start Break Button** | Begin the break countdown | P0 |
| **Dismiss Early** | Return to studying before break ends | P0 |
| **Random Activity** | Display random activity from pool | P0 |

### 3.2 Configuration Options

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `study_duration_minutes` | Integer | 25 | Time before break prompt |
| `break_duration_minutes` | Integer | 5 | Length of break |
| `snooze_duration_minutes` | Integer | 5 | Snooze delay time |
| `enable_sounds` | Boolean | true | Master sound toggle |
| `sound_on_break_start` | Boolean | true | Play sound when break begins |
| `sound_on_break_end` | Boolean | true | Play sound when break ends |
| `break_activities` | Array | [see below] | Pool of activity suggestions |
| `activities_enabled` | Object | {} | Enable/disable individual activities |

### 3.3 Default Activity Pool

```
- "Stand up and stretch for 2 minutes"
- "Get a glass of water"
- "Look at something 20 feet away for 20 seconds"
- "Do 10 deep breaths"
- "Walk around the room"
- "Do 10 jumping jacks"
- "Rest your eyes (close them for 1 minute)"
- "Step outside for fresh air"
```

---

## 4. User Interface

### 4.1 Break Reminder Popup

```
┌─────────────────────────────────────┐
│         ⏰ Time for a Break!         │
├─────────────────────────────────────┤
│                                     │
│   Suggested Activity:               │
│   "Stand up and stretch for         │
│    2 minutes"                       │
│                                     │
│         ┌─────────────┐             │
│         │    5:00     │             │
│         │  remaining  │             │
│         └─────────────┘             │
│                                     │
│  ┌─────────┐  ┌─────────────────┐   │
│  │ Snooze  │  │   Start Break   │   │
│  │ (5 min) │  │                 │   │
│  └─────────┘  └─────────────────┘   │
│                                     │
│       [ Back to Studying ]          │
│                                     │
└─────────────────────────────────────┘
```

**States:**
1. **Prompt State** - Shows snooze + start break buttons
2. **Active Break State** - Shows countdown timer + dismiss button
3. **Break Complete** - Auto-closes or shows "Resume" button

### 4.2 Settings Dialog

Accessible via: `Tools → Add-ons → Break Reminder → Config`

Sections:
- Timer Settings (study duration, break duration, snooze duration)
- Sound Settings (toggles + file selection)
- Activity Manager (list with enable/disable checkboxes, add/remove buttons)

### 4.3 Menu Integration

- `Tools → Break Reminder → Settings`
- `Tools → Break Reminder → Start Timer`
- `Tools → Break Reminder → Pause Timer`
- `Tools → Break Reminder → Reset Timer`

---

## 5. Technical Requirements

### 5.1 Compatibility
- Anki 2.1.50 or higher
- PyQt5 and PyQt6 compatible
- Windows, macOS, Linux

### 5.2 Dependencies
- None external (uses Anki's bundled PyQt)

### 5.3 File Structure
```
break_reminder/
├── __init__.py
├── config.json
├── config.md
├── timer.py
├── dialogs.py
├── activities.py
├── sounds.py
└── user_files/
    ├── break_start.wav
    └── break_end.wav
```

### 5.4 Hooks Used
| Hook | Purpose |
|------|---------|
| `profile_did_open` | Initialize timer on profile load |
| `reviewer_did_show_question` | Detect active reviewing |
| `reviewer_will_end` | Pause timer when leaving reviewer |

---

## 6. Behavior Specifications

### 6.1 Timer Logic

```
START: User opens reviewer and views first card
  → Study timer starts (configurable, default 25 min)
  
DURING REVIEW: Timer runs in background
  → Timer pauses if user leaves reviewer
  → Timer resumes when user returns to reviewer
  
TIMER COMPLETE: Study duration reached
  → Play break sound (if enabled)
  → Show break popup
  
USER CLICKS "SNOOZE":
  → Close popup
  → Restart study timer for snooze duration
  
USER CLICKS "START BREAK":
  → Begin break countdown
  → Show countdown in popup
  
BREAK COMPLETE:
  → Play resume sound (if enabled)
  → Auto-restart study timer
  → Close popup (or show "Resume" button)
  
USER CLICKS "BACK TO STUDYING":
  → Close popup
  → Restart study timer immediately
```

### 6.2 Edge Cases

| Scenario | Behavior |
|----------|----------|
| User closes Anki during break | Timer state not persisted; resets on reopen |
| User switches profiles | Timer resets for new profile |
| Popup already open, timer fires again | Ignore (shouldn't happen) |
| No activities enabled | Show "Take a break!" without activity |
| Sound file missing | Fail silently, log warning |

---

## 7. Success Metrics

| Metric | Target |
|--------|--------|
| User rating on AnkiWeb | ≥ 4.0 stars |
| Successful installs | Track via AnkiWeb |
| GitHub issues (bugs) | < 5 open bugs at any time |

---

## 8. Future Enhancements (Post-MVP)

| Feature | Description |
|---------|-------------|
| Statistics | Track breaks taken, study streaks |
| Long break | Every X breaks, prompt longer break |
| Custom sounds | Let users select their own audio files |
| System notifications | Option for OS-level notifications |
| Sync settings | Store config in Anki profile for sync |
| Keyboard shortcuts | Hotkeys for snooze/start/dismiss |

---

## 9. Timeline

| Phase | Duration | Deliverable |
|-------|----------|-------------|
| Development | 1-2 weeks | Working add-on |
| Testing | 3-5 days | Bug fixes, edge cases |
| Documentation | 1-2 days | README, config.md |
| Release | 1 day | AnkiWeb submission |

---

## 10. Open Questions

1. Should the timer persist across Anki restarts?
2. Should there be a visual indicator (status bar) showing time until next break?
3. Should the popup be modal (blocking) or non-modal?
4. Should we integrate with system-level focus/break apps?

---

*Document Version: 1.0*  
*Last Updated: December 2025*
