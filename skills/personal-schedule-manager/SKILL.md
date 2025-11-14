
---
name: personal-schedule-manager
description: >
  Converts a Goal (G-xxx) into executable Schedule entries (S-xxx) with temporal semantics.
  **Version:** 1.1  **Author:** Entropy Control Theory  **License:** MIT  
  **Based on:** Structure DNA v1.0, M-PPS v1.0, PROFILE-GENERATOR-SPEC v1.0
---

# personal.schedule.manager — v1.1

## Goal
Translate goal intentions into one or more **S-** entries with `start/due/duration`, moving state to `scheduled`.  
This version adds **energy-aware scheduling**: before planning, it ensures and reads `/ledger/profile.json`
to align execution time with the user’s chrono-cognitive energy curve.

---

## Inputs
| Field | Type | Required | Description |
|-------|------|-----------|-------------|
| `goal_id` | string | ✅ | Reference to the parent goal (`G-xxx`) |
| `plan` | string \| object | ⛔ | Natural-language or explicit ISO-8601 time semantics |
| `notes` | string | ⛔ | Optional free text |
| `kind` | enum | ⛔ | `creative` \| `implementation` \| `warmup` — used for energy-window mapping |

---

## Outputs
Writes entries to `/ledger/schedule.json` following the **Structure DNA Field Genome**:

Required:
- `id (S-xxx)`
- `title`
- `status = "scheduled"`
- `start | due | duration` (at least one)
- `created_at`, `updated_at`

Optional:
- `goal_id`, `tags`, `notes`, `related_entries`, `dispatch_to`

New (v1.1):
- Adds `energy_zone` (or annotation `[energy_zone:peak|stable|low]` in `notes`)

---

## Mechanism
1. **Ensure and load `profile.json`**  
   - If missing, create `/ledger/profile.json` from `profile.schema.json` + `profile.defaults.json`.
   - Load `profile.data[0]` for energy curves, constraints, and break preferences.
2. **Energy-aware window selection**  
   - `creative` → `19:00–22:00` (`peak`)  
   - `implementation` → `14:00–19:00` (`stable`)  
   - `warmup` → `10:00–11:00` (`low`)  
   - If the user supplied explicit times, they take precedence; energy windows fill gaps or conflicts.
3. **Constraint handling and block slicing**  
   - Obey `must_stop_by = 22:00`
   - Skip `lunch 12:30–13:30`, `dinner 18:00–19:00`
   - Split long sessions by `preferred_block_length = 90–120 min`
4. **Resolve temporal semantics → ISO timestamps**
5. **State transition:** set `status = "scheduled"`, preserve `goal_id`, and optionally  
   `dispatch_to = "reflection.manager"` for automatic feedback.

---

## Pseudocode

```python
def schedule_entry(entry, ledger_dir="/ledger"):
    # 1. Ensure and load profile
    ensure_profile(ledger_dir)
    profile = load_json(f"{ledger_dir}/profile.json")["data"][0]

    # 2. Determine energy window
    kind = entry.get("kind") or classify(entry)
    window = choose_window(kind, profile)

    # 3. Apply constraints and compute start/due
    slots = place_with_constraints(
        plan=entry.get("plan"),
        window=window,
        breaks=profile.get("break_preferences"),
        must_stop_by=profile["constraints"]["must_stop_by"],
        block_len=profile["work_style"]["preferred_block_length"]
    )

    # 4. Finalize schedule entry
    entry["start"], entry["due"] = slots[0].start_iso, slots[-1].end_iso
    entry["status"] = "scheduled"
    entry["updated_at"] = now_iso()
    entry["notes"] = (entry.get("notes") or "") + f" [energy_zone:{window.label}]"
    return entry

def choose_window(kind, profile):
    if kind == "creative":       return Window("19:00","22:00","peak")
    if kind == "implementation": return Window("14:00","19:00","stable")
    return Window("10:00","11:00","low")
````

---

## Example

**Input**

```json
{
  "goal_id": "G-001",
  "plan": "2 × 2h this week",
  "kind": "creative",
  "notes": "Write Substack demo"
}
```

**Output**

```json
{
  "id": "S-101",
  "title": "Write Substack demo",
  "goal_id": "G-001",
  "status": "scheduled",
  "start": "2025-11-03T19:00:00-05:00",
  "due": "2025-11-03T21:00:00-05:00",
  "duration": "2h",
  "notes": "user plan:2x2h; [energy_zone:peak]",
  "created_at": "2025-11-03T13:05:00-05:00",
  "updated_at": "2025-11-03T13:05:00-05:00",
  "dispatch_to": "reflection.manager",
  "related_entries": ["G-001"]
}
```

---

## Notes

* v1.1 keeps the dispatch rules unchanged — logic is handled **inside** the Skill runtime.
* If `/ledger/profile.json` is missing, `ensure_profile()` bootstraps it automatically.
* Compatible with all other MVP Skills (`ledger.registry`, `reflection.manager`, `goal.manager`).
* Designed for immediate use within the Structure DNA → M-PPS → LLC loop.

> “Energy-aware scheduling turns language intention into a rhythm of execution.”
> — *Entropy Control Theory, 2025*

```

---

```

