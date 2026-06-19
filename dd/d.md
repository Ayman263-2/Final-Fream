# Project: Accident Detection System Technical Improvements

## Architecture
- `config.py`: Configuration variables, default thresholds, toggles (like `USE_ONNX`).
- `helpers.py`: Helper classes, including `Config` and `ConfigMeta` for live parameter updating.
- `server.py`: Flask-based backend server. Added `/update_config`, `/config`, and `/history` endpoints.
- `trackers.py`: Tracking algorithms. Added GPU-based GMC (`cv2.cuda`) and Kalman tracker matrix updates (PyTorch tensors on CUDA/CPU devices).
- `pipeline.py`: Main processing loop. Handles frame processing, YOLO inference (ONNX or PyTorch), and calling database logging.
- `db.py` (New): SQLite database interface (`archive.db` with `runs` and `events` tables).
- `static/js/app.js` and `index.html`: Dashboard settings panel rendering and updating.

## Milestones
| # | Name | Scope | Dependencies | Status |
|---|------|-------|-------------|--------|
| 1 | Setup E2E Test Suite | Build E2E test infra, design Tier 1-4 test cases | None | DONE |
| 2 | R1: ONNX Model Acceleration | config.py, helpers.py, pipeline.py - ONNX export, load, fallback | M1 | DONE |
| 3 | R2: Live Config Update API | server.py, helpers.py, index.html, static/js/app.js - dynamic update endpoint, Whitelist, Dashboard | M1 | DONE |
| 4 | R3: SQLite Archive Database | db.py, server.py, pipeline.py - SQLite storage, history API, events.json compatibility | M1 | DONE |
| 5 | R4: GPU Tracking Acceleration | trackers.py - GMC cuda flow, Kalman PyTorch tensors, fallback | M1 | DONE |
| 6 | Integration and Final Verification | Pass all E2E test suite tiers, verify no regression, perform adversarial testing (Tier 5) | M2, M3, M4, M5 | DONE |

## Interface Contracts
### Live Config Update
- Endpoint: `POST /update_config`
- Payload: JSON dictionary with keys to update (e.g. `{"CONF": 0.55}`)
- Whitelisted keys: `CONF`, `IOU_NMS`, `MATCH_THRESH`, `BAYES_THRESH`, `SUDDEN_DEC`, `FIRE_ENABLED`, `USE_GMC`, `USE_ONNX`
- Response: `{"status": "success", "config": <updated_config_dict>}`

- Endpoint: `GET /config`
- Response: `{"status": "success", "config": <current_config_dict>}`

### SQLite Database Schema
- File: `archive.db` in project root.
- Table `runs`:
  - `run_id` (TEXT PRIMARY KEY)
  - `video_name` (TEXT)
  - `timestamp` (TEXT)
  - `duration` (REAL)
  - `total_frames` (INTEGER)
  - `total_events` (INTEGER)
  - `confirmed_collisions` (INTEGER)
  - `visual_accidents` (INTEGER)
  - `actual_fps` (REAL)
  - `status` (TEXT)
  - `video_url` (TEXT)
  - `report_url` (TEXT)
  - `csv_url` (TEXT)
  - `summary_json` (TEXT)
- Table `events`:
  - `id` (INTEGER PRIMARY KEY AUTOINCREMENT)
  - `run_id` (TEXT, FK to runs)
  - `frame` (INTEGER)
  - `ts` (REAL)
  - `level` (TEXT)
  - `type` (TEXT)
  - `ids` (TEXT)
  - `score` (REAL)
  - `message` (TEXT)
  - `bbox` (TEXT)

- Endpoint: `GET /history`
- Response: `{"status": "success", "history": [<list of last 10 runs>]}`

## Code Layout
- `config.py` — Global configuration parameters.
- `helpers.py` — Config wrapper and other utilities.
- `pipeline.py` — Main processing script.
- `trackers.py` — BoT-SORT / ByteTrack trackers, Kalman filters, GMC.
- `server.py` — Flask API backend.
- `db.py` — SQLite database storage layer.
- `static/` — Frontend assets (CSS, JS, images).
- `index.html` — Frontend dashboard interface.
