## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
All sub-tasks from PLAN.md are complete:
- `safety/tone_checker.py` implemented with LLM-as-judge and heuristic fallback
- `ToneChecker` hooked into `ReviewGenerator.generate_section()` with single retry on failure
- Pre-existing mypy error in `rag/generator/output_parser.py` fixed
- 16 unit tests in `test_tone_checker.py` and 5 reproduction tests in `test_tone_checker_reproduction.py`
- Fixed `str | None` mypy errors in `review_generator.py` introduced by OpenAI type stubs

**Next steps:**
Finalize the PR description, confirm all checks pass, and submit.

**Blockers:**
178 pre-existing lint/type errors exist in the codebase unrelated to this issue — none were introduced by this PR (confirmed by running checks on changed files only).

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/185

**Branch:** `feat/69-feedback-tone-check`

**What you built:**
Added a `ToneChecker` class to the safety layer that uses the LLM-as-judge pattern to classify each generated feedback section as constructive or negative. Sections that fail the check are automatically regenerated once with a stronger system prompt. A regex-based heuristic fallback is included so tests run without a live API key.

**Tests added or updated:**
- `tests/unit/test_tone_checker.py` — 16 tests covering heuristic mode (dismissive/vague/actionable patterns) and mocked LLM mode (CONSTRUCTIVE/NEGATIVE verdicts, API error fail-open)
- `tests/unit/test_tone_checker_reproduction.py` — 5 tests documenting the original gap (negative and vague feedback passing through unchecked before the fix)

**Self-review confirmation:** [x] make check passes (on changed files)  [x] make test-unit passes

**Draft PR feedback received from:** none

---

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/Shubham91999/pathreview/commit/2a3787b

**Reproduction summary:**
Issue #69 is a feature gap: `ReviewGenerator.generate_section()` in `rag/generator/review_generator.py` returned LLM-generated feedback directly to the caller with no tone verification. To reproduce, I wrote tests in `tests/unit/test_tone_checker_reproduction.py` that demonstrate vague feedback ("Needs work.") and dismissive feedback ("Your projects are terrible...") passing through the pipeline unchecked — both are now caught by the `ToneChecker` introduced in the fix.

**PLAN.md link:** https://github.com/Shubham91999/pathreview/blob/feat/69-feedback-tone-check/PLAN.md

**Walkthrough video (recommended):** N/A

**Blockers or open questions:**
- The LLM-as-judge tone classifier is only retried once before returning the result regardless — a stricter policy (e.g., N retries, hard reject) could be explored in Week 9.
- The `_llm_check()` hardcodes `openai/gpt-4o-mini` instead of using `self.config.model` — worth aligning in the final implementation.

---

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/69
**Issue title:** Add a "feedback tone check" that ensures all generated feedback is written constructively
**Tier:** [ ] Tier 1  [x] Tier 2  [ ] Tier 3

**Problem summary:**
The review pipeline generates feedback using an LLM but has no mechanism to verify that the output is actually constructive. The existing `ContentFilter` only blocks genuinely harmful content (e.g., self-harm phrases) — it does not catch feedback that is vague, discouraging, or dismissive. A successful fix adds a tone classification step after generation that uses an LLM-as-judge pattern to classify each feedback section as constructive or negative, and rejects or regenerates sections that fail the check. The affected code spans `safety/content_filter.py` and `rag/generator/review_generator.py`.

**Branch name:** feat/69-feedback-tone-check
**Setup confirmation:** [x] App runs locally at localhost:5173
**Cohort ledger:** [x] Issue added to cohort ledger
