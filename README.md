# FrameForge Library

[![CI](https://github.com/cosmowind/frameforge-library/actions/workflows/ci.yml/badge.svg)](https://github.com/cosmowind/frameforge-library/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**A local-first library for camera footage: watch the card, tag the footage,
keep the catalogue in a plain CSV.**

No cloud account, no database server, no upload required. Point it at a folder
of clips, or let it watch for a memory card, and it maintains a spreadsheet of
everything you have shot -- with thumbnails, durations, tags and export.

```
  card / folder        FrameForge Library              anything you like
 ┌──────────────┐      ┌────────────────────────┐      ┌──────────────────┐
 │ DCIM/ hmd/   │ ───▶ │ watch → copy → tag     │ ───▶ │ CSV ⇄ TXT        │
 │ *.mp4 *.hmd  │      │ preview, rotate, split │      │ thumbnails       │
 └──────────────┘      │ catalogue in a CSV     │      │ an upload target │
                       └────────────────────────┘      └──────────────────┘
```

---

## Why a CSV

The catalogue is a single CSV file. That is a deliberate choice: it is
greppable, diffable, editable in a spreadsheet, and it survives the app
disappearing. Postgres is supported as an *additive* layer -- more query
power -- but it is never a prerequisite.

## What it does

- **Watches for removable media.** A background thread polls for card drives
  (Windows), recognises the camera families it knows, copies sessions into your
  library folder, and can free the card afterwards.
- **A spreadsheet-style editor in the browser.** Sort by any column with a
  multi-column sort stack, filter, resize, reorder columns, and tag rows
  inline. `Ctrl+S` saves everything.
- **Previews in place.** Thumbnails, in-page playback, and a gallery.
- **Rotate and split.** ffmpeg-backed tools for the fixes that always come up.
- **Exports.** TXT or CSV, for whatever wants to consume it next.
- **Optional upload.** A generic delivery client signs and uploads a session
  and reports progress back; the parameters are a schema you can swap.

## Quick start

```bash
pip install -r requirements.txt

cp config.example.json runtime/data/config.json
cp samples/*.csv runtime/data/          # optional: synthetic sample rows

python src/run.py                        # -> http://localhost:5000
```

Windows users can double-click `启动视频管理.bat` instead.

### About the example config

`config.example.json` ships with the device monitor switched **off** and
`deleteAfterCopy` set to **false**. That is intentional: the monitor deletes
the source session from the card after copying it, which is what you want once
the workflow is set up and not what you want on a first run.

Read [`README.zh-CN.md`](README.zh-CN.md) before turning it on -- it documents
every field and endpoint in Chinese. The keys the app actually reads are:

| key | meaning |
|---|---|
| `scanDirectory` | the folder to scan for clips |
| `deviceMonitor.enabled` | master switch for card watching (default `true`) |
| `deviceMonitor.destinationBase` | where copies land (defaults to `scanDirectory`) |
| `deviceMonitor.deleteAfterCopy` | delete the source after copying (default `true`) |
| `deviceMonitor.excludeDrives` | drives to skip |
| `deviceMonitor.scanInterval` | seconds between scans |
| `deviceMonitor.hmdPatterns` / `dataPatterns` | folder names for head-mounted / data devices |

## Repository layout

```
.
├── src/
│   ├── app.py               Flask application
│   ├── config.py            paths, column definitions
│   ├── run.py               entry point (web + device monitor)
│   ├── models/              CSV and Postgres stores
│   ├── routes/              HTTP endpoints
│   ├── services/            business logic
│   ├── scripts/             one-off maintenance tools
│   ├── static/              browser assets
│   ├── templates/           the single-page editor
│   └── vendor-upload-pkg/   the generic delivery client
├── scripts/                 operational scripts (backup, migration, diagnostics)
├── samples/                 synthetic sample data
├── docs/                    design notes and reference
└── config.example.json      a known-good starting configuration
```

Runtime state lives under `runtime/` and is git-ignored: the CSV catalogue,
config, thumbnails, caches and logs.

## The catalogue

One row per clip, 14 columns, defined in `src/config.py` as
`DEFAULT_FIELDNAMES`: `path`, `location_status`, `scan_date`,
`original_history_date`, `uploaded`, `extra_description`, `score`,
`duration_sec`, `file_type`, `device_id`, `recorder_id`, `task_id`, `env_id`,
`source_device`. Dimension tables (`recorders.csv`, `devices.csv`,
`tasks.csv`) supply the labels.

`samples/video_manager.csv` uses exactly that schema -- CI checks it.

## Documentation

| document | what it covers |
|---|---|
| [`README.zh-CN.md`](README.zh-CN.md) | the full Chinese project README: every feature, field and endpoint |
| [`docs/EXPORT.md`](docs/EXPORT.md) | the export format |
| [`docs/SCAN_FEATURE_UPDATE.md`](docs/SCAN_FEATURE_UPDATE.md) | how the media scan works |
| [`docs/CODING_STANDARDS.md`](docs/CODING_STANDARDS.md) | conventions to follow when changing this |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | how the pieces fit together |

## Status

A published snapshot of an internal tool. The CSV path is the well-trodden
one; the Postgres path is newer. Expect rough edges in the operational
scripts under `scripts/`.

Everything shipped here is synthetic. No real footage, device serial numbers,
operator rosters or credentials are included.

## License

MIT -- see [LICENSE](LICENSE).
