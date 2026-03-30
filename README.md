# Discord-Orbs-Quest-Completer
A powerful, standalone Python automation tool designed to scanning, accepting, and completing Discord Quests. 

> 
Table of Contents
 * Features
 * Requirements
 * How to Use
 * Configuration
 * Source Code Architecture
   * Constants & Configuration
   * Logging
   * Build Number
   * DiscordAPI
   * Quest Helpers
   * QuestAutocompleter
 * Workflow
 * Supported Task Types
Features
| Feature | Description |
|---|---|
| Auto Scan | Automatically scans for new quests based on a cycle (POLL_INTERVAL) |
| Auto Accept | Automatically enrolls in quests that haven't been accepted yet |
| Auto Complete | Automatically completes quests by sending progress/heartbeat signals |
| Rate Limit Handling | Automatically waits and retries when hitting rate limits (429) |
| Build Number Fetch | Automatically retrieves the latest client_build_number from the Discord web app |
Requirements
 * Python 3.7+
 * Library: requests
<!-- end list -->
pip install requests

How to Use
1. Pass token via argument
python3 main.py <DISCORD_TOKEN>

2. Save token to a .token file
Create a .token file in the same directory, paste your token, then run:
python3 main.py

3. Manual token entry
Run without arguments, and the program will prompt you to enter the token:
python3 main.py

> Note: Use Ctrl+C to stop the program.
> 
Configuration
Configuration constants are located at the top of the main.py file:
| Variable | Default | Description |
|---|---|---|
| API_BASE | https://discord.com/api/v9 | Discord API Base URL |
| POLL_INTERVAL | 60 | Seconds between quest scans |
| HEARTBEAT_INTERVAL | 20 | Seconds between heartbeat signals |
| AUTO_ACCEPT | True | Automatically accept all available quests |
| LOG_PROGRESS | True | Display progress logs |
| DEBUG | True | Enable detailed debug logging |
Source Code Architecture
Constants & Configuration
 * SUPPORTED_TASKS — List of task types supported for automated completion.
Logging
Colors — Class containing ANSI color codes for terminal output.
log(msg, level) — Logging function with different levels:
| Level | Meaning |
|---|---|
| info | General information |
| ok | Success |
| warn | Warning |
| error | Error |
| progress | Progress (can be toggled via LOG_PROGRESS) |
| debug | Debug (can be toggled via DEBUG) |
Build Number
 * fetch_latest_build_number() — Scrapes the Discord web app to fetch the latest client_build_number. Falls back to a default value if it fails.
 * make_super_properties(build_number) — Creates a base64-encoded X-Super-Properties header to spoof the Discord Desktop Client.
DiscordAPI
A class to manage the HTTP session with the Discord API.
| Method | Description |
|---|---|
| __init__(token, build_number) | Initializes the session with headers spoofing the Discord Client |
| get(path) | Sends a GET request |
| post(path, payload) | Sends a POST request |
| validate_token() | Validates the token and prints user information |
Quest Helpers
Utility functions to extract data from quest objects (supports both camelCase and snake_case):
| Function | Description |
|---|---|
| _get(d, *keys) | Retrieves a value from a dict with multiple fallback keys |
| get_task_config(quest) | Retrieves the task configuration from a quest |
| get_quest_name(quest) | Retrieves the quest name (with multiple field fallbacks) |
| get_expires_at(quest) | Retrieves the expiration time |
| get_user_status(quest) | Retrieves the user's status for the quest |
| is_completable(quest) | Checks if the quest can be completed |
| is_enrolled(quest) | Checks if the quest is already enrolled |
| is_completed(quest) | Checks if the quest is already finished |
| get_task_type(quest) | Retrieves the specific task type of the quest |
| get_seconds_needed(quest) | Total seconds required for completion |
| get_seconds_done(quest) | Seconds already completed |
| get_enrolled_at(quest) | The timestamp when the quest was enrolled |
QuestAutocompleter
The main class controlling the entire auto-complete logic.
| Method | Description |
|---|---|
| fetch_quests() | Calls the API to get the quest list; handles rate limits |
| enroll_quest(quest) | Enrolls in a quest (retries up to 3 times) |
| auto_accept(quests) | Automatically enrolls in all unaccepted quests |
| complete_video(quest) | Completes video quests (sends video-progress) |
| complete_heartbeat(quest) | Completes play/stream quests (sends heartbeat) |
| complete_activity(quest) | Completes activity quests (sends heartbeat) |
| process_quest(quest) | Processes a single quest by choosing the appropriate method |
| run() | Main loop: scan → accept → complete → wait → repeat |
Workflow
Startup
  │
  ├─ Read token (argument / .token / manual entry)
  ├─ Fetch build number from Discord
  ├─ Validate token
  │
  └─ Main Loop (every POLL_INTERVAL seconds):
       │
       ├─ Fetch quest list
       ├─ Display overview (Total / Enrolled / Completed)
       ├─ Auto-accept unaccepted quests
       ├─ Filter quests needing completion
       │
       └─ For each quest needing completion:
            │
            ├─ WATCH_VIDEO / WATCH_VIDEO_ON_MOBILE
            │     └─ Send POST /quests/{id}/video-progress continuously
            │
            ├─ PLAY_ON_DESKTOP / STREAM_ON_DESKTOP
            │     └─ Send POST /quests/{id}/heartbeat every 20s
            │
            └─ PLAY_ACTIVITY
                  └─ Send POST /quests/{id}/heartbeat every 20s

Supported Task Types
| Task | API Endpoint | Mechanism |
|---|---|---|
| WATCH_VIDEO | /quests/{id}/video-progress | Sends increasing timestamps, speed ~7s/request |
| WATCH_VIDEO_ON_MOBILE | /quests/{id}/video-progress | Same as WATCH_VIDEO |
| PLAY_ON_DESKTOP | /quests/{id}/heartbeat | Heartbeat every 20s with a random stream_key |
| STREAM_ON_DESKTOP | /quests/{id}/heartbeat | Same as PLAY_ON_DESKTOP |
| PLAY_ACTIVITY | /quests/{id}/heartbeat | Heartbeat every 20s with a fixed stream_key |
Would you like me to create a License file (like MIT or GPL) for your repository as well?
