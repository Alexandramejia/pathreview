## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/68

**Issue title:** Add a safety event count to the health check endpoint

**Tier:** [ ] Tier 1  [x] Tier 2  [ ] Tier 3

**Problem summary:**
The `/health` endpoint reports service status but not safety system activity, so operators have to check the monitoring dashboard separately to see how many safety events have occurred. Two bugs stand between here and a working `safety_events_last_hour` field: (1) `api/routes/health.py` hardcodes the field to `0` instead of calling a real counting function, and (2) the existing helper, `SafetyMonitor.get_event_count` in `safety/monitoring.py`, doesn't actually scope to the last hour — it reads a flat 24-hour Redis counter, so even wiring it up as-is would report the wrong window under the right name. The fix needs to add hour-scoped counting to `SafetyMonitor` and call it from the health endpoint. Scope is limited to `api/routes/health.py` and `safety/monitoring.py`; no database changes required.

**Branch name:** `feat/68-safety-event-health-count`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger