---
name: sillytavern-cardplayer
description: Use when a SillyTavern rolecard candidate needs real-host automatic playtesting, human-like multi-turn interaction, MVU/state/persistence verification, or a modification-test acceptance loop after packaging; also use when browser-driven card tests are slow, flaky, load the wrong artifact, misdiagnose readiness, or contaminate existing cards and chats.
---

# SillyTavern Cardplayer

Close rolecard acceptance in a real SillyTavern host. Keep deterministic variable repair separate from open-ended human play, and keep the simulated player's visible knowledge separate from the technical auditor's evidence.

## Select one mode

- `variable-repair`: reproduce MVU, state, settlement, persistence, or HUD faults with deterministic actions. Reuse one stable test slot with a filename and embedded display name distinct from production; every rebuilt candidate gets a fresh isolated test chat. An authorized repair/build/retest loop may continue, and pre-authorized production promotion may follow a complete pass.
- `human-playtest`: simulate one continuous player pursuing a user-supplied play objective. Do one natural play session, analyze the complete isolated chat, report targeted proposals, then stop for the user's decision. Do not edit, rebuild, or retest automatically.
- `full-gate`: complete `variable-repair` first, then run one `human-playtest`. After the human-playtest report, stop; do not turn its proposals into automatic modifications.

Use the mode named by the user. When both technical correctness and human experience are requested, use `full-gate`. If a human-playtest objective is not already explicit, ask what the simulated player should try to accomplish before starting that phase. Never substitute a fixed turn count for that objective.

## Required routing

- Before a host write, use `consult-tavernweave-library` with the runtime-debug route and read A0.
- For exact version-sensitive paths, endpoints, events, scopes, or host APIs, use `sillytavern-api-reference` or inspect the named installation. Do not rely on remembered SillyTavern internals.
- If the handoff lacks an install-ready PNG, source parity, or a new fixed candidate, return it to `sillytavern-card-pipeline`; do not patch packed output.
- When a failure needs causal runtime diagnosis, use `sillytavern-runtime-debug`, then resume this same acceptance receipt after a rebuilt candidate arrives.

## Load only what the run needs

- Always read [modification-test-handoff.md](references/modification-test-handoff.md) and [evidence-schema.md](references/evidence-schema.md).
- Read [sillytavern-operation-state-machine.md](references/sillytavern-operation-state-machine.md) before touching the host.
- Read [test-matrix.md](references/test-matrix.md) before selecting mode-specific checks.
- For `variable-repair`, read [mvu-variable-repair.md](references/mvu-variable-repair.md).
- For `human-playtest`, read [player-simulator.md](references/player-simulator.md) and [human-playtest-analysis.md](references/human-playtest-analysis.md).
- For `full-gate`, read both mode references, but do not leak the technical phase's hidden evidence into the simulated player's choices.

## Control ladder

Prefer filesystem or verified host API, then documented runtime bridge, then semantic DOM, and only then visual or coordinate control. Ordinary candidates go directly to the verified character store; use the import UI only when import behavior is under test. Human simulation concerns decisions, not slow mouse imitation. Read [sillytavern-operation-state-machine.md](references/sillytavern-operation-state-machine.md) for the operational contract.

Classify the target and lock the deployment method before the first write using [sillytavern-operation-state-machine.md](references/sillytavern-operation-state-machine.md). Creating an absent slot, updating a host-owned slot, and replacing a proven-quiescent file are distinct methods. A transport failure does not authorize a method switch. Do not create a chat until disk payload, host identity, uniqueness, and the post-save reverse-write guard all pass.

## Shared host loop

1. Record authority, host, active character/chat, preservation boundary, stop conditions, and allowed cleanup.
2. Validate the immutable candidate receipt, JSON/PNG parity, hashes, identity, and worldbook mode.
3. Deploy atomically, refresh the host, and prove the running identity; a disk write alone is `deployed-unverified`. Apply the state machine's linked-worldbook deployment and readiness policy before generation.
4. Create the isolated chat, prove runtime readiness, and run only the selected mode's checks. Compare prose, update mechanism, persisted state, and HUD for every state-changing turn.
5. Diagnose the first failing layer and apply the state machine's diagnostic-regeneration and observation-tolerance policy. Human play still ends with complete-chat analysis and returns to the user.
6. Promote production only through the pre-authorized `variable-repair` path, then apply the state machine's rollback and cleanup policy.

## Hard gates

- Perform one persistent mutation at a time and prove its postcondition. A timeout is not a verdict; external user action invalidates stale observations.
- Common host, model, provider, network, extension, persistence, and rendering anomalies can resemble card failures. Consult the common-anomaly table in `test-matrix.md` and do not attribute those symptoms to the card without causal evidence. If an anomaly makes reliable testing impossible, stop, preserve the observed and environmental evidence, list the untested scope, and return to the user.
- Apply the missing-reply recovery policy in the host state machine.
- Do not treat character replacement as worldbook replacement. An unreadable, stale, or wrongly converted worldbook is a deployment failure and is never eligible for diagnostic regeneration.
- Never patch a deployed card, packed output, live worldbook, or live message variables to make acceptance pass.
- Preserve production chats and unrelated cards, worldbooks, settings, input text, credentials, and private extension data.
- A run-created isolated chat is readable. Any pre-existing chat requires exact identity and authorized access level before reading.
- Announce each automation mutation's target, visible effect, and preservation boundary; attribute material actions to automation, user, model, or host.
- Never claim initialization, settlement, persistence, or a fix from one visible layer. Pass only the declared scope and list untested capabilities.
- During human play, hidden auditor evidence must not influence player choices. After the session, read the complete isolated chat, report, and stop.

Return the structured receipt plus a concise report. Include mode, objective when applicable, termination reason, mutations, validated and untested scope, mode and release status, first failure, reproduction seed, promotion/rollback/cleanup result when applicable, retained artifacts, and next owner.
