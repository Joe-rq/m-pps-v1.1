
---
name: goal-manager
description: >
  Parses natural-language intentions into Structure DNA goal entries (G-xxx).
  **Version:** 1.1  **Author:** Entropy Control Theory  **License:** MIT  
  **Based on:** Structure DNA v1.0, M-PPS v1.0, LLC v1.0, PROFILE-GENERATOR-SPEC v1.0
---

# goal.manager — v1.1

## Goal
Turn a user's natural-language intention into a schedulable **G-** entry following the Structure DNA Field Genome and Unified State Machine.  
**v1.1 Extension:** Ensure that `/ledger/profile.json` exists before creating any goals, providing energy-context initialization for downstream scheduling.

---

## Inputs
| Field | Type | Required | Description |
|--------|------|-----------|-------------|
| `intent` | string | ✅ | Natural-language goal statement |
| `constraints` | object | ⛔ | Optional metadata such as timebox, tags, or related IDs |

---

## Outputs
Writes a new entry to `/ledger/goal.json`:

**Required**
- `id (G-xxx)`
- `title`
- `status = "open"`
- `created_at`, `updated_at`

**Optional**
- `goal_id` (linking)
- `tags`, `notes`, `related_entries`
- `dispatch_to = "personal.schedule.manager"` (suggested downstream Skill)

---

## Mechanism
1. **(NEW)** Ensure `/ledger/profile.json` exists  
   - Call `ensure_profile(ledger_dir="/ledger")` before parsing intent.  
   - This guarantees the personal energy preferences ledger is available for all subsequent scheduling.
2. Parse `intent` → extract Primitive IR → compile into Structure DNA fields.
3. Initialize unified state = `open` (do **not** schedule here).
4. If any `constraints.start|due|duration` are included, store them as metadata only.
5. Optionally attach `dispatch_to = "personal.schedule.manager"` for automated scheduling.

---

## Pseudocode

```python
def create_goal(intent, constraints=None, ledger_dir="/ledger"):
    # 1. Ensure personal profile ledger exists (NEW)
    ensure_profile(ledger_dir)

    # 2. Parse and compile to Structure DNA format
    goal_entry = {
        "id": new_id("G-"),
        "title": normalize_intent(intent),
        "status": "open",
        "tags": extract_tags(intent),
        "notes": constraints.get("notes") if constraints else None,
        "created_at": now_iso(),
        "updated_at": now_iso(),
        "dispatch_to": "personal.schedule.manager"
    }

    # Merge additional metadata
    if constraints:
        goal_entry.update(constraints)

    append_to_ledger(f"{ledger_dir}/goal.json", goal_entry)
    return goal_entry
````

---

## Example

**Input**

```json
{
  "intent": "Finish and publish Structure DNA whitepaper by November",
  "constraints": {
    "tags": ["writing", "StructureDNA"],
    "due": "2025-11-30T23:59:00-05:00"
  }
}
```

**Output**

```json
{
  "id": "G-001",
  "title": "Finish and publish Structure DNA whitepaper by November",
  "status": "open",
  "tags": ["writing","StructureDNA"],
  "created_at": "2025-11-03T12:00:00-05:00",
  "updated_at": "2025-11-03T12:00:00-05:00",
  "dispatch_to": "personal.schedule.manager",
  "related_entries": []
}
```

---

## Notes

* Ensures cold-start stability: even if the user has never configured preferences, the system auto-creates `/ledger/profile.json` using `profile.schema.json` and `profile.defaults.json`.
* Does **not** alter scheduling logic; it simply guarantees the environment is ready for downstream energy-aware execution.
* Fully compatible with:

  * `ledger.registry v1.1` (auto-checksum and manifest sync)
  * `personal.schedule.manager v1.1` (energy-aware scheduling)
  * `reflection.manager v1.1` (energy feedback logging)
* This change completes the minimal “Language → Structure → Scheduler” initialization chain.

> “Every goal now begins with awareness of time, energy, and rhythm.”
> — *Entropy Control Theory, 2025*

```

---

