---
name: test-first-metric
description: The test-first workflow for every metric and detector in this project. Use when CESC starts a metric, detector or simulator rule, or a plan step says "test first".
user-invocable: false
---

# Test first, one metric at a time

1. **Hand-made events.** In `tests/test_<metric>.py`, build 5–10 `Event` objects by hand where the right answer is easy to work out on paper. Say the expected value out loud in a comment.
2. **Failing test.** Write one test asserting that value. Run `pytest tests/test_<metric>.py` and show that it fails (the function doesn't exist yet). Seeing red first proves the test can catch a mistake.
3. **Function.** Write the smallest pure function in `src/interpret/` that makes it pass. No AI, no I/O.
4. **Green.** Run `pytest` again. All green.
5. **Edge case.** Add one more test for an edge case (no events, one team only, the zone flip between teams for PPDA).
6. Stop. One metric per step. Commit with `/done`.

In coach mode, CESC writes the test and the function; you review and point at problems without rewriting them. In pair mode you write them and explain the assertion in one line.

IT comparison: a test is like validating a config against a known-good baseline before pushing it to production.
