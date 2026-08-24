# Human Playtest Analysis

Use this reference for one user-defined human-play task. The session tests whether the card remains believable, responsive, and technically coherent during natural play; it is not an automatic card-optimization loop.

## Start gate

Record `userPlayObjective`. If the user has not already supplied it, ask what the simulated player should try to accomplish. Do not impose a turn count. Create one isolated chat and use one continuous simulated player.

Keep two logical compartments:

- **Player:** sees only rendered chat, public setup, visible HUD and controls, its own prior actions, and the supplied play objective.
- **Auditor:** reads update blocks, authoritative variables, events, logs, persistence, and HUD each turn to catch technical faults.

Auditor evidence must never enter the four player options. A blocking technical fault may stop the run; a nonblocking fault is recorded while natural play continues.

## End and read the whole run

End only on a natural terminal condition from `player-simulator.md`. Then read the complete isolated test chat and all per-turn audit records. Do not sample only the last messages and do not read unrelated chats.

For every material issue, reconstruct:

```text
visible stimulus
-> four generated options
-> random selection
-> card response
-> immediate effect
-> later effect
-> issue classification
```

Classify the evidence as one of:

- technical defect;
- systemic card-design problem;
- isolated model/provider variance;
- insufficient evidence or environmental blocker.

One awkward sentence is usually a variance candidate. Repetition, deterministic replay, or a direct contradiction of a card rule supports a systemic classification. Keep technical faults separate from experience improvements.

## Targeted proposal

Each proposed change must contain:

```yaml
issue_id: stable run-local id
symptom: what the player experienced
floor_evidence: relevant message or floor identities
causal_chain: stimulus through downstream effect
classification: technical | systemic | variance | blocked
confidence_basis: repetition, replay, or rule evidence
affected_surface: source, worldbook, prompt, UI, variable layer, or provider
recommended_change: smallest plausible correction
expected_side_effects: likely tradeoffs or none observed
future_replay: scenario that would test the proposal
pass_condition: observable acceptance result
```

Finish with the run result, termination reason, technical findings, experience findings, and proposals. Set `nextOwner` to `user` and stop. Do not edit source, rebuild a candidate, or start another playtest until the user decides what to change.
