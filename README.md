# Discord-Orbs-Quest-Completer
A powerful, standalone Python automation tool designed to scanning, accepting, and completing Discord Quests. 

**<h1 align="center">R Ξ Λ P Σ R</h1>**
<br>

<div align="center">
  <picture>
  <!-- TODO I couldn't figure out how to properly add local links in place of these images. These should be fixed later. - @PrasyIkuzo !-->
    <source media="(prefers-color-scheme: light)" srcset="https://user-images.githubusercontent.com/65187002/172940015-d9d072e7-c47d-4ddd-83f6-8e7717a721b8.png">
    <img src="https://user-images.githubusercontent.com/65187002/172940773-7ef23b63-3356-4634-9e52-34f2676e2854.png">
  </picture><br>
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="https://user-images.githubusercontent.com/65187002/172941127-4061fac1-736b-4c24-b7ea-c210b3578cc5.png">
    <img width="50%" src="https://user-images.githubusercontent.com/65187002/172941149-31258408-bfc3-496a-8a58-e34794b21813.png">
  </picture>
</div>

---

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [How to Use](#how-to-use)
- [Configuration](#configuration)
- [Code Architecture](#code-architecture)
  - [Constants & Configuration](#constants--configuration)
  - [Logging](#logging)
  - [Build Number](#build-number)
  - [DiscordAPI](#discordapi)
  - [Quest Helpers](#quest-helpers)
  - [QuestAutocompleter](#questautocompleter)
- [Workflow](#workflow)
- [Supported Task Types](#supported-task-types)
  
---

## Features

| Feature | Description |
|---|---|
| Auto Scan | Automatically scans for new quests periodically (`POLL_INTERVAL`). |
| Auto Accept | Automatically enrolls in quests that haven't been accepted yet. |
| Auto Complete | Automatically completes quests by sending progress or heartbeat signals. |
| Rate Limit Handling | Automatically waits and retries when rate-limited (HTTP 429). |
| Build Number Fetch | Automatically fetches the latest `client_build_number` from the Discord web app. |

---

## Requirements

- Python 3.7+
- `requests`

```bash
pip install requests
```

---

## Installation

    git clone https://github.com/r3aper007/Discord-Orbs-Quest-Completer.git
    cd Discord-Orbs-Quest-Completer

---

## How to Use

### 1. Pass token via argument

    python3 main.py <DISCORD_TOKEN>

### 2. Save token to a `.token` file

Create a file named `.token` in the same directory, paste your token inside, and then run:

    python3 main.py

### 3. Manual token input

Run the script without arguments, and the program will prompt you to enter your token:

    python3 main.py

> Note: Press `Ctrl+C` at any time to stop the program.

---

## Configuration

Configuration constants are located at the top of `main.py`:

| Variable | Default | Description |
|---|---|---|
| `API_BASE` | `https://discord.com/api/v9` | Discord API base URL |
| `POLL_INTERVAL` | `60` | Seconds between each quest scan |
| `HEARTBEAT_INTERVAL` | `20` | Seconds between sending heartbeat signals |
| `AUTO_ACCEPT` | `True` | Automatically accept all available quests |
| `LOG_PROGRESS` | `True` | Show detailed progress logs |
| `DEBUG` | `True` | Enable detailed debug logs |

---

## Code Architecture

### Constants & Configuration

- `SUPPORTED_TASKS` — List of task types that the tool supports for automatic completion.

### Logging

- `Colors` — ANSI color codes for terminal output.
- `log(msg, level)` — Logging function with levels:
  - `info` — General information
  - `ok` — Success
  - `warn` — Warning
  - `error` — Error
  - `progress` — Progress status
  - `debug` — Debug info

### Build Number

- `fetch_latest_build_number()` — Scrapes the Discord web app to fetch the latest `client_build_number`. Falls back to a default value if it fails.
- `make_super_properties(build_number)` — Generates the `X-Super-Properties` header in base64 format, simulating the Discord Desktop Client.

### DiscordAPI

A class that manages HTTP sessions with the Discord API.

| Method | Description |
|---|---|
| `__init__(token, build_number)` | Initializes the session with headers mimicking the Discord Client |
| `get(path)` | Sends a GET request |
| `post(path, payload)` | Sends a POST request |
| `validate_token()` | Checks if the token is valid and prints user info |

### Quest Helpers

Utility functions to extract data from the quest object, supporting both camelCase and snake_case:

| Function | Description |
|---|---|
| `_get(d, *keys)` | Gets a value from a dictionary using multiple fallback keys |
| `get_task_config(quest)` | Gets the task configuration from the quest |
| `get_quest_name(quest)` | Gets the quest name |
| `get_expires_at(quest)` | Gets the expiration time |
| `get_user_status(quest)` | Gets the user's status for the quest |
| `is_completable(quest)` | Checks if the quest can be completed |
| `is_enrolled(quest)` | Checks if the user is enrolled in the quest |
| `is_completed(quest)` | Checks if the quest is completed |
| `get_task_type(quest)` | Gets the quest's task type |
| `get_seconds_needed(quest)` | Total seconds required to complete the quest |
| `get_seconds_done(quest)` | Total seconds already completed |
| `get_enrolled_at(quest)` | The time when the quest was enrolled |

### QuestAutocompleter

The main class controlling the entire auto-complete logic.

| Method | Description |
|---|---|
| `fetch_quests()` | Calls the API to get the quest list and handles rate limits |
| `enroll_quest(quest)` | Enrolls in a quest (retries up to 3 times) |
| `auto_accept(quests)` | Automatically enrolls in all unaccepted quests |
| `complete_video(quest)` | Completes video quests (sends `video-progress`) |
| `complete_heartbeat(quest)` | Completes game playing/streaming quests (sends `heartbeat`) |
| `complete_activity(quest)` | Completes activity quests (sends `heartbeat`) |
| `process_quest(quest)` | Processes a single quest by selecting the appropriate method |
| `run()` | Main loop: scan → accept → complete → wait → repeat |

---

## Workflow

    Startup
      │
      ├─ Read token (argument / .token / manual input)
      ├─ Fetch build number from Discord
      ├─ Validate token
      │
      └─ Main loop (every POLL_INTERVAL seconds):
           │
           ├─ Fetch quest list
           ├─ Display overview (total / enrolled / completed)
           ├─ Auto-accept unaccepted quests
           ├─ Filter quests that need completion
           │
           └─ For each quest needing completion:
                │
                ├─ WATCH_VIDEO / WATCH_VIDEO_ON_MOBILE
                │     └─ Sends POST `/quests/{id}/video-progress` continuously
                │
                ├─ PLAY_ON_DESKTOP / STREAM_ON_DESKTOP
                │     └─ Sends POST `/quests/{id}/heartbeat` every 20s
                │
                └─ PLAY_ACTIVITY
                      └─ Sends POST `/quests/{id}/heartbeat` every 20s

---

## Supported Task Types

| Task | API Endpoint | Mechanism |
|---|---|---|
| `WATCH_VIDEO` | `/quests/{id}/video-progress` | Sends increasing timestamps roughly every 7s |
| `WATCH_VIDEO_ON_MOBILE` | `/quests/{id}/video-progress` | Same as `WATCH_VIDEO` |
| `PLAY_ON_DESKTOP` | `/quests/{id}/heartbeat` | Heartbeat every 20s with a random `stream_key` |
| `STREAM_ON_DESKTOP` | `/quests/{id}/heartbeat` | Same as `PLAY_ON_DESKTOP` |
| `PLAY_ACTIVITY` | `/quests/{id}/heartbeat` | Heartbeat every 20s with a fixed `stream_key` |

---

## ⚠️ Disclaimer

This project is for educational purposes only. Using automation tools may result in account restrictions.
