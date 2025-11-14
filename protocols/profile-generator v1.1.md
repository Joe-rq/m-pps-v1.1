
---
title: "PROFILE-GENERATOR-SPEC v1.0"
author: "Entropy Control Theory"
version: "1.0"
status: "MVP"
layer: "system-execution / initialization"
based_on:
  - "Structure DNA v1.0"
  - "M-PPS v1.1"
created: "2025-11-03"
license: "MIT"
file: "/docs/profile-generator-spec-v1.0.md"
---

# PROFILE-GENERATOR-SPEC v1.0

> **Goal**: Guarantee that a Structure DNA–compliant **`/ledger/profile.json`** exists in every runtime,  
> so downstream Skills can perform **energy-aware** scheduling and feedback without touching dispatch rules.

---

## 1) Scope

This spec defines a **tiny, surgical initialization contract** used by existing Skills:

- **ledger.registry** — *must* create `profile.json` if missing.
- **goal.manager** — *may* ensure `profile.json` before writing a new goal.
- **personal.schedule.manager** — *must* read `profile.json` to choose time windows.
- **reflection.manager** — *may* read `profile.json` to compute `energy_match`.

No changes are required in `dispatch.rules.json`.

---

## 2) Responsibilities

| Component | Responsibility | MUST/SHOULD |
|-----------|----------------|-------------|
| `ensure_profile()` (utility) | Create `/ledger/profile.json` from templates if absent | **MUST** be callable by all Skills |
| `ledger.registry` | Call `ensure_profile()` before building `manifest.json`; rescan after creation | **MUST** |
| `personal.schedule.manager` | Call `ensure_profile()` at start; load preferences and map task → window | **MUST** |
| `goal.manager` | Cold-start safety; call `ensure_profile()` before first goal | **SHOULD** |
| `reflection.manager` | Read preferences to compute `energy_zone`/`energy_match` | **SHOULD** |

---

## 3) Files & Templates

### 3.1 Ledger Container Skeleton (profile.schema.json)
```json
{
  "module": "profile",
  "schema": "StructureDNA-v1.0",
  "last_updated": "1970-01-01T00:00:00Z",
  "data": [],
  "metadata": {
    "source_skill": "auto.profile.generator",
    "version": "1.0.0",
    "checksum": "sha256:auto"
  }
}
````

### 3.2 Defaults for `data[0]` (profile.defaults.json)

```json
{
  "id": "P-001",
  "title": "Personal Chrono-Cognitive Preferences",
  "profile": "personal_preferences",
  "owner": { "name": "Entropy Control Theory", "timezone": "EST (UTC-5)" },
  "work_schedule": { "daily_start": "10:00", "daily_end": "22:00", "total_hours": "11-12h" },
  "energy_curve": {
    "low_energy":       { "time": "09:00-11:00", "level": "low" },
    "moderate_energy":  { "time": "11:00-14:00", "level": "moderate" },
    "stable_energy":    { "time": "14:00-19:00", "level": "stable" },
    "peak_creativity":  { "time": "19:00-22:00", "level": "peak" }
  },
  "constraints": { "earliest_start": "09:00", "latest_end": "22:00", "must_stop_by": "22:00" },
  "break_preferences": {
    "lunch":  { "time": "12:30-13:30", "duration": "60min" },
    "dinner": { "time": "18:00-19:00", "duration": "60min" }
  },
  "work_style": { "preferred_block_length": "90-120min", "focus_method": "pomodoro" },
  "task_allocation_rules": {
    "morning_10_11":  ["warm-up", "review", "planning", "light reading"],
    "midday_11_14":   ["research", "organizing", "outlining"],
    "afternoon_14_18":["implementation", "testing"],
    "evening_19_22":  ["creative writing", "design", "conceptual thinking", "strategic planning"]
  },
  "tags": ["profile", "biological-rhythm", "energy-aware"]
}
```

---

## 4) Runtime Contract

### 4.1 Utility: `ensure_profile(ledger_dir="/ledger")`

* If `/ledger/profile.json` exists → return path.
* Else:

  1. Load `profile.schema.json`.
  2. Load `profile.defaults.json`.
  3. Set `container.last_updated = now_iso()`.
  4. Set `container.data = [defaults]`.
  5. Write `/ledger/profile.json`.
  6. Return the path.

**Reference pseudocode**

```python
def ensure_profile(ledger_dir="/ledger"):
    path = f"{ledger_dir}/profile.json"
    if os.path.exists(path):
        return path
    container = load_json("profile.schema.json")
    defaults  = load_json("profile.defaults.json")
    container["last_updated"] = now_iso()
    container["data"] = [defaults]
    save_json(path, container)
    return path
```

### 4.2 `ledger.registry` Integration (MUST)

* After scanning `/ledger`:

  * If **no** `module == "profile"` in parsed ledgers → `ensure_profile()` → **rescan**.
* Build/refresh `manifest.json` (include profile), recompute checksums.

```python
ledgers = scan_ledgers("/ledger")
if not any(L["module"] == "profile" for L in ledgers):
    ensure_profile("/ledger")
    ledgers = scan_ledgers("/ledger")
manifest = build_manifest(ledgers)
write_manifest_with_checksums(manifest)
```

### 4.3 `personal.schedule.manager` Integration (MUST)

* Call `ensure_profile()` at entry.
* Load `profile.data[0]` and map task *kind* → window:

  * `creative` → `19:00–22:00` (`peak`)
  * `implementation` → `14:00–19:00` (`stable`)
  * `warmup` → `10:00–11:00` (`low`)
* Respect `must_stop_by` and meal breaks; split blocks by `preferred_block_length` (90–120 min).
* Annotate `energy_zone` (field or `[energy_zone:...]` in notes).

### 4.4 `goal.manager` (SHOULD)

* Call `ensure_profile()` before emitting a new goal to guarantee downstream readiness.

### 4.5 `reflection.manager` (SHOULD)

* Extract `energy_zone` from the source schedule (or infer from time).
* Compute `energy_match` against `profile` windows; store both in R-entry.

---

## 5) Backward Compatibility

* Dispatch rules remain unchanged.
* All new behavior is **opt-in at runtime** via the presence of `/ledger/profile.json`.
* If templates are missing, Skills should proceed without energy features (fail-soft).

---

## 6) Integrity & Manifest

* `ledger.registry` owns manifest consistency:

  * Auto-include `profile.json` when present.
  * Recompute `metadata.checksum` for each ledger and the manifest.
* `profile.json` follows the **Ledger Container Schema**; *do not* store code, only preferences.

---

## 7) Versioning & Evolution

* `v1.0` — Single profile, static defaults, energy-aware windows.
* `v1.1+` — Add cognitive/motivational fields, learning metrics (`energy_match_rate`, `entropy_delta_avg`).
* `v2.0` — Multiple profiles (work/creative/travel), agent-specific profiles, merging strategies.

---

## 8) Acceptance Criteria

1. Running **any** of: `ledger.registry`, `goal.manager`, `personal.schedule.manager`

   * **creates** `/ledger/profile.json` if missing.
2. `manifest.json` lists an entry with `"module": "profile"`.
3. Newly scheduled entries include an energy annotation (`energy_zone`).
4. Reflections optionally include `energy_zone` and `energy_match`.

---

## 9) Test Script (natural language)

1. “Run the registry and ensure my preferences ledger exists.”
2. “Create a *creative* goal and schedule it for this week.”
3. “Show the scheduled window and confirm it matches *peak* hours.”
4. “Mark it done and write a reflection including `energy_zone` and `energy_match`.”

---

## 10) Security & Privacy

* Keep `profile.json` local by default; avoid embedding secrets.
* If anchoring on-chain, **hash only** (checksum) or publish a redacted subset.
* Prefer manifest-level discovery; never hardcode absolute paths in Skills.

---

## 11) Changelog

* **1.0 (2025-11-03)** — Initial MVP spec: utility function, registry integration, scheduler mapping, optional reflection feedback.

---

```
```

