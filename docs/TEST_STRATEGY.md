# Kinara AI — Test Strategy

Version: 1.1
Status: MVP test baseline with automated CI.

## Continuous integration

GitHub Actions runs the automated test suite on pull requests targeting `main`
and on pushes to `main`. The workflow installs dependencies from
`requirements.txt` and runs:

```bash
python -m pytest -q
```

The CI job does not receive production credentials. Tests must use fakes,
mocks, or skip live connectivity checks when the required credentials are
not available. Never add production secrets to the repository or workflow
source.

## Categories

### 1. Unit — pure logic

Everything in `src/services/gamification.py`, `src/services/adaptive_engine.py`,
`src/utils/concepts.py`. No I/O, no mocks needed — these are plain
functions in, values out. This is the highest-value test category: it's
where score, XP, mastery, streak, and adaptive-threshold bugs live.

**Minimum expectation:** every deterministic rule in FSD.md (scoring,
XP amounts, streak trigger, mastery bands, difficulty thresholds, Grade
bands, Level thresholds once implemented) has at least one test proving
the boundary behavior (just-below / at / just-above each threshold).

### 2. Integration — service + fake Firestore

`src/services/firestore_service.py`, `src/services/session_service.py`
tested against `tests/fakes/fake_firestore.py` (a minimal in-memory
stand-in — no real network, no real project). Covers the full
generate→submit→persist wiring without needing live credentials.

**Minimum expectation:** the full submit path (score → memory update → persistence → next recommendation)
is covered end-to-end at least once against the fake, per major behavior change.

### 3. Authorization / security

`tests/test_firestore_authorization.py` proves that a FirestoreService
instance is scoped to its owning UID. Every new FirestoreService method
must ship with a scoping test.

### 4. AI contract

Contract tests mock AI clients and verify response validation and retry
behavior. Live provider connectivity tests are separate manual/release
checks and must never be required for ordinary CI.

### 5. Regression

Any fixed bug must have a regression test that reproduces the original
failure. Name the test after the behavior or defect it protects.

### 6. Manual acceptance

UI flows, live provider connectivity, Firestore deployment configuration,
and Cloud Run deployment remain manual acceptance/release checks. Track
them in `docs/ACCEPTANCE_TESTS.md`; a green unit-test workflow does not
prove production readiness.

## Known gaps

- Automated authentication service coverage remains a required follow-up
  (see AUTH-001 in the acceptance criteria).
- Continue Learning navigation and any other known no-op UI behavior must
  be resolved and manually verified before describing the MVP as complete.
- CI currently provides test execution only; dependency vulnerability
  scanning, coverage thresholds, and deployment gates are future hardening
  items.
