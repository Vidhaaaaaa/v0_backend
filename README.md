| **# Pools** | **Same-Pool Pairing**                        | **Near-pool Pairing**                       | **Half-Pool Pairing**                     | **Far-Pool Pairing**                     |
|-------------|----------------------------------------------|---------------------------------------------|-------------------------------------------|------------------------------------------|
| **2 Pools** | A1 vs A2, B1 vs B2                           | A1 vs B2, A2 vs B1                         | A1 vs A2, B1 vs B2 (same as "same-pool")  | A1 vs B2, A2 vs B1                       |
| **4 Pools** | A1 vs A2, B1 vs B2, C1 vs C2, D1 vs D2       | A1 vs B2, A2 vs B1, C1 vs D2, C2 vs D1     | A1 vs B2, A2 vs B1, C1 vs D2, C2 vs D1    | A1 vs D2, A2 vs D1, B1 vs C2, B2 vs C1   |
| **8 Pools** | A1 vs A2, B1 vs B2, C1 vs C2, D1 vs D2, E1 vs E2, F1 vs F2, G1 vs G2, H1 vs H2 | A1 vs B2, A2 vs B1, C1 vs D2, C2 vs D1, E1 vs F2, E2 vs F1, G1 vs H2, G2 vs H1 | A1 vs D2, A2 vs D1, E1 vs H2, H1 vs E2    | A1 vs H2, A2 vs H1, B1 vs G2, B2 vs G1, C1 vs F2, C2 vs F1, D1 vs E2, D2 vs E1 |

---

# Khel Club – Walkover Match Outcome Support (v0)

## Overview

This submission adds support for a new match outcome: **Walkover**, to the existing Khel Club tournament management system.

The goal was not just to add a one-off condition, but to extend the system in a way that:

* Keeps existing scoring logic intact
* Clearly separates **match outcome** from **match scoring**
* Makes it easy to add future outcomes such as *forfeit* or *bye*

The codebase is v0 and intentionally imperfect; the changes focus on clarity, correctness, and extensibility rather than refactoring everything.

---

## How the Existing Scoring Mechanism Works

In the current system:

* A match has:

  * `status` (`pending`, `on-going`, `completed`)
  * `is_final`
  * `winner_team_id`
* Scores are stored in the `score` table per team per match
* For normal matches:

  * Scores are updated via the scoring endpoint
  * When a match is finalized, the winner is derived from score comparison
  * Knockout matches propagate winners to successor matches

Scoring and match finalization were already tightly coupled to **score input**.

---

## What Was Added for Walkover Support

### 1. Match Outcome as a First-Class Concept

A new field was introduced on the `match` model in models.py:

* `match_outcome`

This allows the system to distinguish:

* *How* a match ended (walkover, forfeit, etc.)
* From *how* scores are calculated (or skipped)

This avoids hard-coding logic based on scores alone.

---

### 2. Centralized Outcome Handling

A dedicated backend endpoint in match_core.py was added to **explicitly mark match outcomes**.

When a terminal outcome (e.g. `walkover`) is set:

* `status` is set to `completed`
* `is_final` is set to `true`
* `winner_team_id` is required and validated
* No score mutation is required

This keeps outcome logic centralized and avoids duplicating logic in scoring routes.

Future outcomes (forfeit, bye) can be added by extending the allowed outcome list without refactoring scoring logic.

---

### 3. Scoring Logic Remains Unchanged for Normal Matches

* Existing score-based flow continues to work exactly as before
* Walkover matches bypass score comparison
* Normal matches are unaffected

This was an explicit design choice to avoid regressions.

---

### 4. Knockout Successor Propagation

For walkover matches:

* Once finalized, the winner is propagated to the successor match
* This reuses the existing successor update logic

This demonstrates end-to-end correctness for tournament progression.

---

## Frontend Changes

* Match data now includes `match_outcome`
* Existing UI components were reused
* Walkover matches are displayed using outcome-aware rendering

  * The result clearly indicates a walkover instead of a numeric score
* Match tables and exports include outcome information

No UI redesign was done, as per instructions.

---

## Schema / Data Changes

* Added `match_outcome` column to `match` table
* No destructive changes
* Existing data remains valid

---

## Assumptions Made

* Walkover is an **admin-controlled** action
* A walkover always has a single winner
* Walkover matches should not require score input
* Standings calculation can rely on `winner_team_id` for terminal outcomes

---

## What I Would Improve With More Time

* Refactor standings aggregation to cleanly separate:

  * Score-based wins
  * Outcome-based wins (walkover / forfeit)
* Add enums or a lookup table for match outcomes
* Improve frontend visibility in fixture cards for all terminal outcomes
* Add automated tests for outcome handling

These were intentionally left out to avoid introducing breaking changes close to the deadline.

---

## Final Notes

This solution prioritizes:

* Extensibility over hacks
* Clear separation of concerns
* Minimal impact on existing behavior

Partial improvements (especially in standings) were left documented rather than rushed, in line with the assignment guidance.

