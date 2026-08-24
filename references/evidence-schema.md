# Acceptance Evidence Receipt

Produce JSON plus a concise human summary. Use `pass`, `fail`, `blocked`, `not_applicable`, or `not_run` for checks. Do not report `pass` without direct evidence from the relevant layer.

## Receipt shape

```json
{
  "runId": "YYYY-MM-DDThh:mm:ssZ__candidate-id__seed",
  "mode": "full-gate",
  "status": "fail",
  "candidate": {
    "id": "example-card-test.2",
    "sourceRevision": null,
    "jsonArtifact": "absolute/path/candidate.json",
    "jsonSha256": "...",
    "pngArtifact": "absolute/path/candidate.png",
    "pngSha256": "...",
    "installedPath": "absolute/discovered/test-file.png",
    "runningVersion": "test.2",
    "identityVerified": true
  },
  "stableTestSlot": {
    "displayName": "Example Card [Test]",
    "fileName": "__cardplayer_test__example-card.png",
    "installedPath": "absolute/discovered/test-file.png",
    "iterations": [
      {
        "candidateId": "example-card-test.2",
        "pngSha256": "...",
        "chatIdentity": "isolated test chat identity"
      }
    ]
  },
  "deployment": {
    "method": "host-managed-update",
    "targetInitialState": "existing",
    "targetInitialSha256": "...",
    "targetWasLoaded": true,
    "quiescenceEvidence": ["verified host-managed update contract"],
    "creationAttempts": 1,
    "firstPostDeploySha256": "...",
    "postSaveBoundarySha256": "...",
    "decodedIdentity": {
      "firstPostDeploy": {
        "displayName": "Example Card [Test]",
        "version": "test.2",
        "worldbookMode": "linked",
        "worldbookName": "__cardplayer_test__example-card-worldbook",
        "linkedWorldbook": {
          "pointer": "worldbook:__cardplayer_test__example-card-worldbook",
          "contentIdentity": "Example Card [Test] / test.2"
        }
      },
      "postSaveBoundary": {
        "displayName": "Example Card [Test]",
        "version": "test.2",
        "worldbookMode": "linked",
        "worldbookName": "__cardplayer_test__example-card-worldbook",
        "linkedWorldbook": {
          "pointer": "worldbook:__cardplayer_test__example-card-worldbook",
          "contentIdentity": "Example Card [Test] / test.2"
        }
      }
    },
    "runningIdentity": {
      "firstPostDeploy": {
        "displayName": "Example Card [Test]",
        "version": "test.2",
        "worldbookMode": "linked",
        "worldbookName": "__cardplayer_test__example-card-worldbook",
        "linkedWorldbook": {
          "pointer": "worldbook:__cardplayer_test__example-card-worldbook",
          "contentIdentity": "Example Card [Test] / test.2"
        }
      },
      "postSaveBoundary": {
        "displayName": "Example Card [Test]",
        "version": "test.2",
        "worldbookMode": "linked",
        "worldbookName": "__cardplayer_test__example-card-worldbook",
        "linkedWorldbook": {
          "pointer": "worldbook:__cardplayer_test__example-card-worldbook",
          "contentIdentity": "Example Card [Test] / test.2"
        }
      }
    },
    "hostObjectReversion": false,
    "diskPayloadMatchCount": 1,
    "hostIdentityMatchCount": 1,
    "visibilityRecovery": ["page reload"],
    "worldbookInstall": {
      "method": "installed-version card-lore conversion",
      "sourceFormat": "v2-character-book",
      "previousStableWorldbookRemoved": true,
      "newCharacterInMemoryBeforeImport": true,
      "nativeEntryShape": "entries keyed by uid",
      "contentIdentity": "Example Card [Test] / test.2",
      "criticalEntries": {"[InitVar]": "present", "update protocol": "present"},
      "readable": true,
      "postImportPngIdentityPreserved": true
    }
  },
  "environment": {
    "sillyTavernVersion": "detected version",
    "url": "detected SillyTavern URL",
    "activeUser": "discovered identifier",
    "driver": "in-app browser",
    "viewport": "width x height",
    "dependencies": []
  },
  "authority": {
    "allowedMutations": ["isolated candidate deploy", "isolated chat", "obsolete run-owned artifact cleanup"],
    "forbiddenMutations": ["production replacement", "unrelated artifact cleanup"],
    "preExistingChatReadScope": []
  },
  "chat": {
    "identity": "test chat identity",
    "startingMessageCount": 0,
    "endingMessageCount": 6,
    "diagnosticRegenerations": [
      {
        "originalFloor": "floor identity",
        "reason": "isolated likely model-format deviation",
        "samePreTurnStateVerified": true,
        "replacementFloor": "floor identity",
        "result": "variance | repeated"
      }
    ]
  },
  "player": {
    "seed": "recorded seed",
    "userPlayObjective": "complete the user-supplied objective",
    "terminationReason": "objective completed, failed, blocked, needs user decision, or user stopped",
    "turns": [
      {
        "turn": 1,
        "options": ["...", "...", "...", "..."],
        "chosenIndex": 2,
        "sent": "...",
        "visibleOutcome": "concise summary"
      }
    ]
  },
  "checks": [
    {
      "id": "standard-action",
      "status": "fail",
      "expected": "time and declared costs settle once",
      "proseEvidence": "...",
      "updateEvidence": "...",
      "stateEvidence": "...",
      "hudEvidence": "..."
    }
  ],
  "firstFailure": {
    "layer": "model narration/update omission",
    "minimalReproduction": ["..."],
    "expected": "...",
    "actual": "...",
    "relevantLogs": ["sanitized causal lines"]
  },
  "analysis": {
    "chatScope": "complete isolated test chat identity",
    "issues": [
      {
        "issueId": "H-01",
        "symptom": "...",
        "floorEvidence": ["floor identities"],
        "causalChain": ["visible stimulus", "chosen option", "card response", "later effect"],
        "classification": "technical | systemic | variance | blocked",
        "confidenceBasis": "repetition, replay, or rule evidence",
        "occurrenceCount": 1,
        "independentChatCount": 1,
        "disposition": "observe | repair | blocked",
        "affectedSurface": "source | worldbook | prompt | UI | variable layer | provider",
        "recommendedChange": "smallest plausible correction",
        "expectedSideEffects": "...",
        "futureReplay": "...",
        "passCondition": "observable result"
      }
    ]
  },
  "mutations": [
    {"actor": "automation", "type": "candidate-file-created", "target": "...", "recoverable": true},
    {"actor": "automation", "type": "test-chat-created", "target": "...", "recoverable": true},
    {"actor": "user", "type": "manual-regeneration", "target": "floor identity", "recoverable": true}
  ],
  "promotion": {
    "authorized": false,
    "productionInitialState": "absent",
    "productionOperation": "create",
    "sourceRevisionMatched": null,
    "productionArtifactSha256": null,
    "originalProductionSha256": null,
    "backupPath": null,
    "backupSha256": null,
    "writesQuiescent": null,
    "productionSmokeStatus": "not_run",
    "rollbackStatus": "not_applicable"
  },
  "cleanup": {
    "status": "pass",
    "removed": ["older run-created test chats", "temporary worldbooks and files"],
    "retained": ["stable test card", "stable test worldbook", "latest run-created test chat", "receipt"],
    "preserved": ["production card and chats", "unrelated cards and worldbooks"]
  },
  "retainedArtifacts": ["..."],
  "acceptance": {
    "validatedScope": ["active candidate identity"],
    "untestedScope": ["Swipe", "Edit", "Chat switch", "human play feel"],
    "modeStatus": "fail",
    "releaseGates": [
      {"id": "core-mvu", "status": "fail"},
      {"id": "stateful-branch-operations", "status": "not_run"},
      {"id": "human-play-feel", "status": "not_run"}
    ],
    "releaseStatus": "not_ready"
  },
  "nextOwner": "user"
}
```

Omit mode-inapplicable or unavailable optional detail rather than inventing it. `player` and `analysis` are required for `human-playtest`; deterministic fixture evidence is required for `variable-repair`. `stableTestSlot` and `deployment` are required for a repair loop. `deployment` must record the single locked method, target initial state and hash, target-loaded state, quiescence evidence, creation-attempt count, both post-deploy hashes, and stage-keyed `decodedIdentity` and `runningIdentity` evidence for the `firstPostDeploy` and `postSaveBoundary` reads (each with display name, version, worldbook mode, and worldbook name), plus host-object reversion and disk/host uniqueness counts before chat creation. When worldbook mode is `linked`, also record the stable pointer plus `worldbookInstall`: prior stable-book removal, refreshed in-memory character identity before import, verified import/conversion method, source format, native entry shape, content identity, readability, declared critical-entry results, and preserved post-import PNG identity. Character replacement alone is never evidence of a worldbook update. For an existing slot, both post-deploy hashes, both decoded identity reads, and both running identity reads are required before `active-verified`. Record an optional one-shot diagnostic regeneration only after readiness passes, and include issue recurrence counts/disposition so isolated noncritical observations do not become automatic repairs. `promotion` and `cleanup` are required when production replacement was authorized, including failed attempts, rollback evidence, deletion of older run chats, and retention of only the latest run-created test chat. Keep exact message text only when needed to reproduce; otherwise summarize. Never include secrets, cookies, API keys, authentication headers, private unrelated settings, or unrelated chat content.

## Status rules

- `pass`: direct evidence proves the expected result and forbidden regression did not occur.
- `fail`: the candidate or declared integration violated an acceptance requirement.
- `blocked`: environment, authority, dependency, provider, budget, or driver capability prevented a valid test.
- `not_applicable`: the capability is not declared for this card.
- `not_run`: the capability is declared but outside this run's selected scope or explicitly deferred. It must appear in `untestedScope` and cannot support release readiness.

`modeStatus` is `fail` when any required check in the selected acceptance scope fails, `blocked` when no scoped check fails but at least one cannot run, and `pass` only when all checks required by that mode pass. `releaseStatus` is `ready` only when every declared release gate passes with direct evidence; otherwise use `not_ready` or `unassessed`.

## Next-owner rules

- `variable-repair`: `nextOwner` may be the source/runtime/pipeline skill while an authorized repair loop remains active; otherwise it is `user`.
- `human-playtest`: `nextOwner` is always `user`. The report cannot authorize its own recommendations.
- `full-gate`: technical failures may route through the repair loop, but after the human phase `nextOwner` is `user`.

`human-playtest` never authorizes or populates a production promotion. In `full-gate`, leave promotion pending after the human report until the user makes a new decision. In `variable-repair`, promotion may proceed only when the authority, exact production target, and its `existing` or `absent` initial state were recorded before testing. Use `replace` plus backup restoration for `existing`; use `create` plus removal-and-absence verification for `absent`.

No mode result may silently fill an untested release gate. `validatedScope`, `untestedScope`, and actor-attributed mutations are required even when the run stops early.

Allowed mutation actors are `automation`, `user`, `model`, and `host`. When an external user action contributes evidence, record it explicitly rather than attributing the result to automation.

## Modification-ready failure

A failure handoff must answer:

- Which immutable candidate was running?
- Which exact action and seed reproduced it?
- What was visible to the player?
- Which of prose, update, persisted state, and HUD first diverged?
- Which earlier layers were proven healthy?
- What acceptance check should pass after the next source fix?
- What candidate files, worldbooks, chats, or settings remain?

Do not prescribe a source implementation unless diagnosis proves it. The modifying skill owns the fix; this receipt owns reproduction and acceptance.
