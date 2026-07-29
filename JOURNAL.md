## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/68

**Issue title:** Add a safety event count to the health check endpoint

**Tier:** [ ] Tier 1  [x] Tier 2  [ ] Tier 3

**Problem summary:**
The `/health` endpoint reports service status but not safety system activity, so operators have to check the monitoring dashboard separately to see how many safety events have occurred. Two bugs stand between here and a working `safety_events_last_hour` field: (1) `api/routes/health.py` hardcodes the field to `0` instead of calling a real counting function, and (2) the existing helper, `SafetyMonitor.get_event_count` in `safety/monitoring.py`, doesn't actually scope to the last hour — it reads a flat 24-hour Redis counter, so even wiring it up as-is would report the wrong window under the right name. The fix needs to add hour-scoped counting to `SafetyMonitor` and call it from the health endpoint. Scope is limited to `api/routes/health.py` and `safety/monitoring.py`; no database changes required.

**Branch name:** `feat/68-safety-event-health-count`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/Alexandramejia/pathreview/commit/4c1da22373d50da6d1f3a5a2b5760d23487108e1

**Reproduction summary:**
I started the backend and Redis/Postgres, then loaded `/health` in the browser. No matter what, it always showed `"safety_events_last_hour": 0`. That confirms the bug: the number is just hardcoded to 0 instead of actually being counted.

**PLAN.md link:** https://github.com/Alexandramejia/pathreview/blob/feat/68-safety-event-health-count/PLAN.md

**Walkthrough video (recommended):** [link to your Loom video, ≤2 min — recommended, not graded]

**Blockers or open questions:**
While testing, `/health` also showed as "unhealthy" for two other reasons that aren't part of this bug: the Postgres check and the Redis check are both broken in small unrelated ways. Not something I need to fix for this issue, but flagging it in case it matters later.

