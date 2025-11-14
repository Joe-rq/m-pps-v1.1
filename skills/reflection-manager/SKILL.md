
---
name: reflection-manager
description: >
  Generates reflection (R-xxx) upon completion of a Schedule/Task and updates cognitive links.
  **Version:** 1.1  **Author:** Entropy Control Theory  **License:** MIT  
  **Based on:** Structure DNA v1.0, M-PPS v1.0, LLC v1.0, PROFILE-GENERATOR-SPEC v1.0
---

# reflection.manager — v1.1

## Goal
When a **Schedule (S-)** entry reaches `done`, generate a **Reflection (R-)** entry to capture outcomes, deltas, and next steps.  
**v1.1 Extension:** Add `energy_zone` and `energy_match` fields for energy-aware learning and meta-feedback in the Language Logic Core (LLC).

---

## Inputs
| Field | Type | Required | Description |
|--------|------|-----------|-------------|
| `source_id` | string | ✅ | The completed S-xxx or T-xxx entry |
| `result` | string/object | ⛔ | Summary of outcome, metrics, blockers |
| `next` | string/object | ⛔ | Optional next-step intention |

---

## Outputs
Writes to `/ledger/reflection.json` (Structure DNA compliant):

**Required**
- `id (R-xxx)`
- `title/action`
- `status = "done"`
- `created_at`, `updated_at`

**Optional**
- `related_entries` (links to S-xxx / G-xxx)
- `notes`, `tags`
- **New (v1.1)**:  
  - `energy_zone`: string → `"peak" | "stable" | "low"`  
  - `energy_match`: boolean → whether the execution matched the preferred energy window

---

## Mechanism
1. Load the source entry (`S-xxx` / `T-xxx`) from `/ledger/schedule.json`.  
2. Extract contextual info:
   - `goal_id`  
   - `related_entries`  
   - `notes` or prior metadata (for continuity)
3. **(NEW)** Detect `energy_zone`:  
   - Parse from `source_entry.notes` (e.g., `[energy_zone:peak]`),  
     or infer by comparing `start/due` times to profile windows.
4. **(NEW)** Compute `energy_match`:  
   - Load `/ledger/profile.json` if available.  
   - Compare actual start/due window to preferred time range for the detected task type.  
   - Mark as `true` or `false`.
5. Create a new R-entry, link it to the source and parent goal, write to `/ledger/reflection.json`.
6. Return reflection summary and optional “re-goal” suggestion for `goal.manager`.

---

## Pseudocode

```python
def create_reflection(source_id, result=None, next_intent=None, ledger_dir="/ledger"):
    source = find_entry(source_id, ledger_dir)
    profile = try_load_json(f"{ledger_dir}/profile.json")

    # Extract or infer energy zone
    zone = extract_energy_zone(source)
    match = compute_energy_match(source, zone, profile)

    reflection = {
        "id": new_id("R-"),
        "title": f"Reflection for {source_id}",
        "status": "done",
        "goal_id": source.get("goal_id"),
        "related_entries": [source_id, source.get("goal_id")],
        "notes": (result or "") + f" | energy_zone:{zone} | energy_match:{match}",
        "energy_zone": zone,
        "energy_match": match,
        "created_at": now_iso(),
        "updated_at": now_iso(),
        "dispatch_to": "goal.manager"
    }

    append_to_ledger(f"{ledger_dir}/reflection.json", reflection)
    return reflection


def extract_energy_zone(entry):
    # parse [energy_zone:peak] from notes, fallback to "unknown"
    import re
    note = entry.get("notes", "")
    m = re.search(r"\[energy_zone:(.*?)\]", note)
    return m.group(1) if m else "unknown"


def compute_energy_match(entry, zone, profile):
    if not profile or zone == "unknown":
        return None
    # compare start time vs profile window
    return time_within_profile(entry["start"], zone, profile)
````

---

## Example

**Input**

```json
{
  "source_id": "S-210",
  "result": "completed article draft; felt productive"
}
```

**Output**

```json
{
  "id": "R-310",
  "title": "Reflection for S-210",
  "status": "done",
  "goal_id": "G-101",
  "related_entries": ["S-210","G-101"],
  "energy_zone": "peak",
  "energy_match": true,
  "notes": "completed article draft; felt productive | energy_zone:peak | energy_match:true",
  "created_at": "2025-11-03T21:55:00-05:00",
  "updated_at": "2025-11-03T21:55:00-05:00",
  "dispatch_to": "goal.manager"
}
```

---

## Notes

* `energy_zone` and `energy_match` are **optional** and auto-filled if `profile.json` exists.
* No change to dispatch rules — triggers remain `done → reflection.manager`.
* These new fields can be used by LLC for meta-learning or entropy-delta tracking.
* Works seamlessly with `personal.schedule.manager v1.1`.

> “Reflection links time, energy, and intention — transforming action into learning.”
> — *Entropy Control Theory, 2025*

```

---


