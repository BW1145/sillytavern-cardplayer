# Modification–Test Handoff

Use an immutable candidate receipt for every iteration. A filename or “latest build” claim is not a handoff.

## Required candidate receipt

```json
{
  "candidateId": "example-card-test.3",
  "sourceRevision": "project-defined revision or null",
  "jsonArtifact": "absolute/path/candidate.json",
  "jsonSha256": "...",
  "pngArtifact": "absolute/path/candidate.png",
  "pngSha256": "...",
  "decodedPayloadSha256": "...",
  "testSlot": {
    "displayName": "Example Card [Test]",
    "fileName": "__cardplayer_test__example-card.png",
    "installedPath": "absolute/discovered/test-file.png"
  },
  "deployment": {
    "method": "host-managed-update",
    "targetInitialState": "existing",
    "expectedInstalledPath": "absolute/discovered/test-file.png",
    "expectedCurrentSha256": "...",
    "targetWasLoaded": true,
    "quiescenceRequirement": "verified host-managed update contract",
    "deleteReimportAuthorized": false,
    "preserveChatLinkage": true,
    "creationAttemptsAllowed": 1
  },
  "iteration": 3,
  "newChatPerIteration": true,
  "worldbookMode": "linked",
  "standaloneWorldbooks": [
    {
      "name": "__cardplayer_test__example-card-worldbook",
      "pointer": "worldbook:__cardplayer_test__example-card-worldbook",
      "contentIdentity": "Example Card [Test] / test.3",
      "sourceFormat": "v2-character_book",
      "importMethod": "installed-version card-lore conversion",
      "replacePolicy": "canonical linked-worldbook replacement",
      "criticalEntries": ["[InitVar]", "update protocol"]
    }
  ],
  "changedSurfaces": ["MVU settlement", "worldbook grouping"],
  "acceptanceScope": ["core MVU settlement", "reload persistence"],
  "requiredChecks": ["time advances", "pending events settle"],
  "releaseGates": ["core MVU", "stateful branch operations", "human play feel"],
  "forbiddenRegressions": ["opening submit breaks", "existing chat mutates"],
  "promotion": {
    "authorized": true,
    "productionInitialState": "absent",
    "productionDisplayName": "Example Card",
    "productionFileName": "example-card.png",
    "cleanupOnSuccess": true
  }
}
```

Accept project-specific field names when their meanings are unambiguous. Reject or return the handoff when:

- the install-ready PNG is missing for a PNG character store;
- hashes do not match the named files;
- the PNG payload cannot be decoded or is not semantically equal to the JSON artifact;
- the test-slot filename or embedded display name collides with production;
- the deployment method is absent, changes within an iteration, or permits a second creation attempt after the candidate reaches the store;
- an existing deployment target does not declare its expected installed path and current hash;
- required checks or forbidden regressions are absent;
- linked-worldbook source format, stable name, replacement order, import/conversion method, content identity, or critical entries are absent;
- the acceptance scope or complete release gate set is missing, making a scoped verdict impossible;
- promotion is authorized but the production target's initial state is not recorded as `existing` or `absent`;
- the candidate was modified after its receipt was created.

Do not make a convenient PNG inside the test skill. Ask `sillytavern-card-pipeline` to produce and verify it so the test exercises the real packaging chain.

## Mode-specific iteration protocol

### `variable-repair`

When the user authorized repair, return each first-failure receipt to the owning runtime/source/pipeline skill, receive a newly built immutable candidate, and replay the same deterministic fixture. The loop may continue until the named checks pass or a declared blocker or authority boundary stops it.

### `human-playtest`

Deploy one immutable candidate, run one user-objective natural session, analyze the complete isolated chat, and return proposals to the user. Do not convert those proposals into source edits, a new build, or a second playtest automatically.

### `full-gate`

Complete the authorized `variable-repair` loop first. After technical checks pass, run one `human-playtest`. The human report ends the run and returns control to the user even when it contains plausible improvements.

Never patch the deployed PNG, packed JSON, live worldbook, message variables, or HUD to make acceptance pass. Diagnostic prototypes belong to `sillytavern-runtime-debug` and cannot become the delivered fix.

## Identity isolation

- In `variable-repair`, one run reuses one stable test slot. Its filename and embedded display name must both differ from production; changing only the filename is invalid.
- Record the existing target's current installed path and hash in the handoff and require them to match the previous deployment receipt before refresh.
- Keep candidate IDs, source revisions, and hashes immutable per iteration.
- Declare a fresh isolated test chat after every refresh unless the acceptance target is migration or old-chat compatibility.
- Declare whether worldbooks are embedded and the one stable test name of any standalone or linked worldbook. Linked mode must also declare its pointer, content identity, source format, installed-version import/conversion method, replacement policy, and critical entries. The state machine owns the operation order and readiness gates.
- Before creating the iteration chat, expect exactly one stored payload and one host identity for the declared test display name.

Deployment, recovery, activation, and reverse-write execution follow [sillytavern-operation-state-machine.md](sillytavern-operation-state-machine.md).

## Failure return

Return the first causal layer, minimal reproduction, player seed, selected option, expected observable behavior, actual prose/update/state/HUD evidence, relevant logs, recurrence count across chats, any one-shot diagnostic regeneration result, and retained candidate/chat identity. Separate a card defect from a host, model provider, driver, unavailable dependency, or permission blocker. Do not route a noncritical first or second occurrence into repair unless it blocks a core path or corrupts later evidence.

For human-playtest experience issues, return the full-chat causal analysis and targeted proposals described in `human-playtest-analysis.md`; set the user as the next owner. Only deterministic technical failures may enter an already authorized repair/build/retest loop.

## Promotion and cleanup

Passing means the selected acceptance scope passed; it does not imply that untested release gates passed. Mark whole-card release ready only when the receipt declares the complete release gate set and every gate has direct evidence.

Production promotion is allowed only for `variable-repair` when its authority, production target, initial target state, and cleanup policy were recorded before testing. Require a separately built production-identity artifact from the same passing source revision. It may differ from the test artifact only in declared identity and required packaging metadata; verify both artifacts before host mutation. The exact promotion, rollback, and cleanup sequence is defined only in [sillytavern-operation-state-machine.md](sillytavern-operation-state-machine.md); this handoff supplies its authority and target inputs. `human-playtest` never promotes; `full-gate` stops after the human report and waits for the user.
