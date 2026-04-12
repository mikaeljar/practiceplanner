# Practiceplanner

A single-file web app for planning ice hockey practices — managing rosters, attendance, and group assignments. Built for HHC (Harrydahl HC) but easily adaptable for any team.

> **Note:** The user interface is entirely in Swedish.

## Features

### Roster management
- Add players with name, jersey number, level (A/B/C), and position (Forward, Center, Defence, Goalie)
- Add coaches with role (On-ice, Off-ice, Goalie coach)
- Import rosters from JSON or Excel-exported data
- Save and load rosters locally or via Google Drive

### Attendance
- Mark each player and coach as attending, absent, or unknown
- Bulk actions: mark all in / all out / reset
- Attendance summary by player type and coach role

### Group generator
- Automatically split attending players into groups
- Methods: mixed levels, by level, or random
- Optional position-aware grouping (ideal slot distribution)
- Drag-and-drop players between groups on desktop
- Long-press to move players on mobile
- Separate goalie zone

### Practice layout (Upplägg)
- Define up to 8 stations with drill descriptions
- Assign coaches to stations
- Upload a practice image (e.g. rink diagram)
- Drill history per station — quickly reuse previous drills
- Save and load layouts separately from full practices

### Saving and sharing
- Save full practices locally (browser storage) or to Google Drive
- Save rosters and layouts independently
- Download any saved item as a JSON file for backup or transfer
- Import JSON files from device

### Google Drive integration
- Optional — the app works fully without it
- Share a single Drive folder with all coaches for collaborative access
- Folder ID stored locally per device — no automatic folder discovery

### Practice card
- Generate a visual training card with groups, stations, and image
- Copy as text or save as image

## Technology

- **Single HTML file** — no build step, no dependencies to install
- **Vanilla JavaScript** — no framework
- **localStorage** for persistent state between sessions
- **Google Drive API v3** with OAuth 2.0 for optional cloud sync
- **PWA** — installable on mobile with offline support via service worker
