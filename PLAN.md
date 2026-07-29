## Solution plan

**Issue:** Add a safety event count to the health check endpoint — https://github.com/ascherj/pathreview/issues/68

### Understand
The `/health` page is supposed to show how many "safety events" happened in the last hour (a safety event is something like blocking bad content or catching someone's private info). That way, whoever's watching the app doesn't have to open a separate dashboard just to check.

Right now it doesn't work, for two reasons:
1. The code just always shows `0`. It's not actually counting anything — the number is stuck.
2. Even if we made it count, the counting tool it would use doesn't know how to count "just the last hour." It only knows how to count "the last 24 hours." So the number would still be wrong, just wrong in a sneakier way.

What it should do: show the real number of safety events from the past hour.
What it actually does: always shows `0`.

### Map
- `api/routes/health.py` — where the hardcoded `0` needs to be replaced with a real function call
- `safety/monitoring.py` — where `SafetyMonitor` lives; needs a new way to count events that's actually scoped to the last hour (not the flat 24-hour counter)

### Plan
1. In `safety/monitoring.py`, add a way to log events with a timestamp (or per-hour bucket) instead of one flat counter, so we can tell which events happened in the last hour.
2. Add a new method (or fix `get_event_count`) so it actually filters to the last hour instead of returning the full 24-hour count.
3. In `api/routes/health.py`, replace the hardcoded `0` with a real call to that method, summed across all event types.
4. Test it by triggering a few safety events and checking that the count goes up, then checking that it drops off after an hour (or fake/shorten the window while testing).
5. Update/add unit tests for both files.

### Inputs & outputs
Input: safety events already being logged by `SafetyMonitor.log_event()` (things like `pii_detected`, `injection_attempt`, etc).
Output: `/health` returns an accurate `safety_events_last_hour` number instead of always `0`.

### Risks & unknowns
- Need to pick a way to track "last hour" in Redis (e.g., a sorted set with timestamps, or per-minute/per-hour keys) without breaking the existing 24-hour counter other code might rely on.
- Not sure yet if anything else in the codebase depends on `get_event_count`'s current behavior — need to check before changing its signature/meaning.
- No database changes needed, so this should stay contained to Redis + these two files.

### Edge cases
- No safety events at all in the last hour → should return `0`, not error.
- Redis unreachable → should fail gracefully (log an error) rather than crash the whole `/health` endpoint, same as the other health checks already do.
- Events that happened more than an hour ago shouldn't count anymore.