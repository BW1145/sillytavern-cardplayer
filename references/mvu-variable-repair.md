# MVU Variable Repair

Use this mode for state initialization, update, settlement, persistence, event, or HUD faults. The objective is a reproducible technical pass, not believable role-play.

## Establish the fixture

Record the exact variable paths and expected transition for each check. Prefer the smallest deterministic actions that isolate one behavior:

- pure dialogue that should not advance time or consume resources;
- one standard time-consuming action;
- one acquisition or consumption transaction;
- one legal movement or state transition;
- one companion action when applicable;
- one reload and one continued reply.

Before choosing fixtures, derive a coverage inventory from the candidate's declared schema, update rules, HUD bindings, and required checks. Group paths by mutable domain rather than testing every leaf. For each domain record one fixture and expected paths, or mark it `not_run` with a reason. Typical domains include time/vitals, carried and camp inventory, location, companions/NPCs, wounds/clothing, facilities, and pending/resolved events.

The auditor may inspect the full isolated chat, update block, message-floor `stat_data`, pending and resolved events, runtime logs, persistence storage, and HUD. Use that access to locate the earliest divergence; it is not player knowledge because this mode has no simulated player.

## Verify each transition

For every fixture, capture before and after values and wait for the declared settlement terminal condition. Compare:

```text
rendered prose
-> update mechanism
-> authoritative persisted state
-> visible HUD or consumer
```

Do not send the next action while generation or settlement is still active. Do not repair a HUD symptom before proving whether persisted state is already correct.

## Model-output variance

When an otherwise valid fixture produces a missing or visibly truncated update block, first prove the active candidate and required prompt/worldbook injection. Record the sample, then perform at most one controlled replay of the same fixture when regeneration is authorized. A single sample is not evidence for a source fix. Repeated omission after proven injection may be classified as model/provider compliance or a prompt-contract defect; keep it separate from parser, settlement, and persistence failures.

## Authorized repair loop

When modification is authorized:

```text
first-failure receipt
-> runtime diagnosis when needed
-> maintained-source fix by the owning card skill
-> new immutable candidate and receipt
-> atomically refresh the recorded stable test slot
-> create a fresh isolated test chat
-> prove the active candidate ID and hash
-> replay the same fixture
```

This playtest skill never patches a deployed PNG, live worldbook, message variables, or packed output. It coordinates the handoff to the owning source/pipeline skill and retests the new candidate.

Continue only while the named failure has a bounded next experiment and the requested writes remain authorized. Stop with a receipt when the checks pass, a dependency or provider blocks valid evidence, the same blocker exhausts its retry policy, or further work needs new authority.

## Promotion handoff

When every declared check passes, promote only if the run recorded production-replacement authority before testing. Request a production-identity artifact from the same passing source revision, then use the host state machine's quiescence, backup, atomic replacement, smoke, rollback, and run-owned cleanup sequence. A mode pass without that prior authority returns a receipt and leaves production unchanged.

`human-playtest` findings cannot enter this promotion path. In `full-gate`, the human report returns control to the user before promotion even when the technical phase passed.
