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
    "installedPath": null
  },
  "deployment": {
    "method": "direct-store",
    "creationAttemptsAllowed": 1
  },
  "iteration": 3,
  "newChatPerIteration": true,
  "worldbookMode": "embedded",
  "standaloneWorldbooks": [],
  "changedSurfaces": ["MVU settlement", "worldbook grouping"],
  "acceptanceScope": ["core MVU settlement", "reload persistence"],
  "requiredChecks": ["time advances", "pending events settle"],
  "releaseGates": ["core MVU", "stateful branch operations", "human play feel"],
  "forbiddenRegressions": ["opening submit breaks", "existing chat mutates"],
  "promotion": {
    "authorized": true,
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
- required checks or forbidden regressions are absent;
- the acceptance scope or complete release gate set is missing, making a scoped verdict impossible;
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
- Keep candidate IDs, source revisions, and hashes immutable per iteration. Before refreshing the slot, require its installed path and current hash to match the previous deployment receipt.
- Create a fresh isolated test chat after every refresh unless the declared acceptance target is migration or old-chat compatibility.
- Other modes may deploy one isolated candidate without adopting the repair slot, but they must still avoid production identity collisions.
- Keep embedded character books inside the candidate payload.
- Use stable run-scoped names for standalone/linked test worldbooks and refresh only those declared in the receipt. Do not activate or overwrite global worldbooks unless the test contract explicitly requires it.
- Treat a copied file as `deployed-unverified` until SillyTavern reports the same candidate ID/version and required embedded surfaces.
- A list/cache miss does not return the slot to `absent`. Re-probe the installed path and hash, then refresh or reload against the same file; never import or copy the candidate under another filename as a visibility retry.
- Before creating the iteration chat, require exactly one matching payload in the character store and one host-visible identity for the declared test display name. A duplicate is a deployment failure, not a second candidate.
- Never patch the deployed slot. The pipeline builds and verifies each replacement PNG before atomic deployment.

## Failure return

Return the first causal layer, minimal reproduction, player seed, selected option, expected observable behavior, actual prose/update/state/HUD evidence, relevant logs, and retained candidate/chat identity. Separate a card defect from a host, model provider, driver, unavailable dependency, or permission blocker.

For human-playtest experience issues, return the full-chat causal analysis and targeted proposals described in `human-playtest-analysis.md`; set the user as the next owner. Only deterministic technical failures may enter an already authorized repair/build/retest loop.

## Promotion and cleanup

Passing means the selected acceptance scope passed; it does not imply that untested release gates passed. Mark whole-card release ready only when the receipt declares the complete release gate set and every gate has direct evidence.

Production promotion is allowed only for `variable-repair` when its authority, production target, and cleanup policy were recorded before testing. Require a separately built production-identity artifact from the same passing source revision. It may differ from the test artifact only in declared identity and required packaging metadata; verify both artifacts before host mutation.

After successful production smoke checks, remove only run-owned test-slot, test-chat, standalone test-worldbook, and temporary deployment artifacts. Keep the receipt and one rollback backup outside SillyTavern data directories. If promotion fails, verify rollback before cleanup. `human-playtest` never promotes; `full-gate` stops after the human report and waits for the user.
