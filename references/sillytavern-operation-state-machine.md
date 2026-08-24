# SillyTavern Operation State Machine

Use this reference before touching a live host. Discover exact behavior from the target version; labels and storage paths vary.

## Nonvisual-first adapter

Choose the first available layer that proves and performs the operation:

| Priority | Surface | Typical use |
| --- | --- | --- |
| 1 | Filesystem or verified host API | candidate deployment, hashes, persisted test-chat checks, character inventory |
| 2 | Documented runtime bridge | MVU data, message-floor state, events, extension readiness |
| 3 | Stable DOM/semantic locator | select character, create chat, fill opening, send/regenerate, interact with the real UI |
| 4 | Screenshot/visual/coordinate control | unknown dialogs, canvas-only controls, layout and responsive acceptance |

Do not take screenshots before every click. Do not use coordinates for controls with IDs, roles, labels, or text. Do not bypass the UI with an internal function when the feature under test is the UI interaction itself.

## Preflight

Record:

- target URL and SillyTavern version;
- installation and active user data root, discovered rather than assumed;
- character, worldbook, chat, and temporary directories;
- current active character, avatar filename, chat identity, and message count;
- exact chat-read scope and access level for any pre-existing chat;
- relevant extensions and capability versions;
- driver capabilities and missing operations;
- current chat input value, focus owner, and any unsent text;
- authorized writes, retry limit, cleanup policy, and generation budget.

An isolated test chat created by this run is in scope. A pre-existing chat is not: require an exact character/chat identity and one of `metadata-variable-structure`, `selected-technical-fields`, or `full-transcript` before reading it. Do not inspect unrelated chats, credentials, cookies, or private extension settings.

## Mutation visibility and actor attribution

Before each persistent or externally visible automation action, briefly state its target, expected visible effect, and what existing data will remain untouched. Do not bury several mutations behind one generic progress update.

Record every material action with `actor: automation | user | model | host`. A user click or manual regeneration may become valid evidence, but never claim it as an automation result. After external intervention, invalidate stale observations and restart from the last proven state.

## Candidate deployment

Classify the declared target before any candidate write and record one immutable `deploymentMethod` for the iteration. The operation state machine owns these mechanics; the handoff supplies only the target and authority inputs.

- If the slot is absent on disk and in host inventory, lock `direct-store-create` or `host-managed-create`.
- If the slot exists and the host is stopped, or target unload plus write quiescence are directly proven, lock `quiescent-store-replace`.
- For an existing run-owned/disposable test slot, `disposable-delete-reimport` is eligible only when target unload and write quiescence are proven, `deleteReimportAuthorized` is true, and `preserveChatLinkage` is false.
- Otherwise, if the slot exists or its identity is active, loaded, cached, or may be host-saved, lock `host-managed-update` through the installed-version verified replace/update contract.
- If no classification can be proved, stop `blocked` before mutation.

Method meanings are fixed:

- `direct-store-create` copies once to a non-card temporary extension, verifies its size and SHA-256 against the candidate receipt, then atomically renames it into a proven-absent slot.
- `disposable-delete-reimport` is a single locked delete/re-import operation, selected before any candidate write, only for an existing run-owned/disposable test slot after its target unload and write quiescence are proven, with `deleteReimportAuthorized: true` and `preserveChatLinkage: false`. It is never eligible for production promotion.
- `host-managed-create` is a version-verified host create operation for a proven-absent slot.
- `host-managed-update` preserves the exact stable-slot identity while the host synchronizes its cache, metadata, and PNG. A UI file chooser and a verified local update request are transports of this same method, not different deployment methods.
- `quiescent-store-replace` atomically replaces the recorded path only after its current hash matches the receipt and no live stale object can write it.

Do not switch methods after a candidate write. A transport failure before any write may use another verified transport for the same locked method, but may not select a new deployment method. `disposable-delete-reimport` cannot be a transport or cache recovery after another locked method. The import UI is reserved for an explicitly accepted import-workflow transport and is never a recognition or cache workaround.

1. Resolve the character directory to an absolute path and prove it is inside the active user's data root.
2. Verify the target store's accepted format from the installed version. For the common PNG store, require a decoded and parity-checked character PNG; never copy the source JSON as a character file.
3. Classify the target using disk and refreshed host inventory. On first `variable-repair` deployment, the stable test-slot path must be absent. On later iterations, replace only the recorded path after its current hash matches the previous deployment receipt. Other modes fail rather than overwrite an existing character. For an existing slot, first record its exact path, current hash, decoded identity/version/worldbook metadata, and active character/chat.
4. Before an existing-slot replacement, move away from the target and prove it inactive. Prove generation, variable settlement, helper writes, character saves, and relevant persistence are quiescent using the strongest version-supported signal; elapsed time alone is insufficient. If unload or quiescence cannot be proven, only the already locked `host-managed-update` with its verified update contract may proceed; otherwise stop `blocked`. These gates never authorize hot replacement.
5. Before updating a candidate with a linked test worldbook, remove the previous same-name worldbook only when its exact stable-test identity and deletion authority are proven; otherwise stop before mutation. Then perform the one locked character method. `direct-store-create` and `host-managed-create` may only create a proven-absent slot; `quiescent-store-replace` may only replace the recorded hash-matching path; `disposable-delete-reimport` may only act under its proven disposal contract; `host-managed-update` may only use the verified installed-version update contract. Never patch a deployed PNG.
6. Treat character and linked-worldbook updates as separate ordered host operations. Immediately after the character update, fully refresh/reload, reselect the exact test identity, and prove the in-memory character and decoded PNG match the new candidate before touching worldbook controls. Then reuse the one stable test-worldbook name. When its source is V2 `character_book.entries[]`, use the installed version's built-in **Import Card Lore** conversion path, or a verified equivalent that invokes the same converter, before saving native world info. Never submit the V2 array directly as native worldbook data. A worldbook operation that runs while an old character object remains loaded can save that stale object back into the PNG; therefore re-read the PNG hash and decoded identity after the import.
7. Fully refresh or reload the host, then prove exactly one stored payload and exactly one host identity. At the first post-deploy observation, verify all declared candidate surfaces, including the hash, decoded payload, running identity/version, worldbook and its linked pointer when applicable, scripts, regexes, and capabilities. For a linked worldbook, read the saved native object, require the installed version's native entry shape, match its declared content/version identity, and find every receipt-declared critical entry such as `[InitVar]` and the update protocol. A matching list name alone is insufficient. Resolve the candidate by the declared test filename plus embedded/display identity, activate it with a version-verified host operation, and re-read all declared candidate surfaces. If direct activation is unavailable, use exact search and require one semantic result before activation. Never use list position, recency sorting, avatar artwork, visual card-wall scanning, or coordinates to discover the candidate.
8. Observe the strongest available host-save or quiescence boundary, then at the post-save-boundary observation re-read all declared candidate surfaces, including the disk hash, decoded metadata, and linked-worldbook pointer, native structure, content identity, readability, and critical entries when applicable. Only a matching second read, together with the decoded and running identity evidence, becomes `active-verified` and may create a chat. A host-object reversion or unreadable/stale worldbook is a deployment failure: preserve evidence and stop without switching methods or spending a model regeneration.

If the refreshed host does not reveal the candidate:

1. Re-read the declared slot path, SHA-256, and decoded embedded identity. If any differs, fail at deployment integrity.
2. Use only the installed version's safe list refresh, full page reload, or documented re-index operation against the same locked method and target.
3. Re-inventory the host list. Do not copy again, change filenames, or use a different transport or deployment method as a visibility retry.
4. If the same target still cannot be recognized, stop `blocked` with the file and cache evidence. Do not create a chat.

Before step 8 can become `active-verified`, inventory both layers: exactly one stored payload may match the candidate hash/slot identity, and exactly one host entry may expose the declared test display name. If either count exceeds one, stop before testing. Clean up only duplicates created by this run and only within recorded cleanup authority; otherwise report the collision to the user.

State after step 5 is `deployed-unverified`; a visibility miss stays in that state; only step 8 plus the two-layer uniqueness check is `active-verified`.

Every rebuilt candidate gets a new isolated chat unless migration or old-chat compatibility is the acceptance target. Reusing the stable test slot must not reuse earlier MVU state.

## UI and runtime lifecycle

| State | Allowed action | Required postcondition |
| --- | --- | --- |
| `active-verified` | create a new isolated chat | chat identity changes; old chat still exists |
| `chat-ready` | wait for scripts/opening/runtime | declared readiness markers and opening DOM exist |
| `runtime-ready` | configure and submit opening once | exactly one intended user message is persisted |
| `opening-submitted` | wait for generation | generation starts or a visible error is recorded |
| `generating` | cheap polling only | assistant floor persists or a terminal error appears |
| `reply-persisted` | wait for variable lifecycle | MVU/update processing reaches terminal event or declared timeout |
| `state-settled` | audit prose/update/state/HUD | all applicable assertions recorded |

Never advance two states from one click. Re-read the postcondition.

## Condition-based waits

Prefer events or cheap markers over fixed sleeps:

- active character ID/name/version;
- chat identity and message count;
- opening region/button enabled state;
- script registration or capability marker;
- generation-started/ended state;
- new assistant floor persisted;
- MVU update-ended/settlement marker;
- target `stat_data` floor changed;
- HUD revision or rendered value changed.

Split a long operation into initiation and observation. A driver timeout may occur after success; a resolved driver call may precede saving.

## Opening and message retry policy

- Wait for declared readiness before clicking an opening button.
- Before an opening click or normal send, read the current input value and focus owner. If unexpected text exists, preserve it and stop for a fresh baseline; do not clear or overwrite it.
- After clicking, verify that the expected user floor was added. If not, re-check readiness and active identity before diagnosing the button.
- Reload/rebind only when the target lifecycle declares it safe, then retry the opening once.
- For a normal message, fill the real input through DOM semantics and submit through the user-facing path unless message-send API behavior itself is the intended setup path.
- After a successful automated send, release focus to a verified neutral element. Before the next input, re-read the field so keystrokes from another page or user action cannot be mistaken for card behavior.
- Missing-reply recovery allows at most three regenerations after the initial generation; the initial request is not counted as a regeneration.
- Before each regeneration, re-read active identity, chat identity, generation state, terminal error state, and persisted message count. If generation is still active, keep observing and do not spend an attempt. If an assistant floor has appeared, stop recovery and continue from that persisted floor. Only when generation is terminal, no assistant floor exists, and the message count still matches the expected baseline may automation record logs, regenerate that same assistant reply, and increment the counter.
- After each regeneration, wait for a terminal generation state and repeat the same persisted-floor check. A driver timeout, resolved request, or stopped spinner alone does not prove that no reply was received.
- If the third regeneration also reaches a verified terminal state without a persisted assistant floor, stop every pending action. Do not regenerate a fourth time, send or resend a user message, edit/delete the chat, continue testing, repair the card, promote production, or clean up the failure scene. Report the three attempts, identities, message counts, terminal states, relevant sanitized logs, and retained artifacts to the user, then wait for their decision.
- Never send a second user action while generation or variable settlement is still active.

## Diagnostic regeneration and observation tolerance

This policy applies only after candidate, pointer, native worldbook structure, content identity, critical entries, and runtime readiness have passed.

- For one persisted reply with an isolated likely model-format or settlement deviation, record the original floor and four-layer evidence. If the host can replace the branch from the identical pre-turn state without leaking abandoned state, perform at most one diagnostic regeneration/Swipe of that reply.
- If the diagnostic reply passes, classify the first result as `variance`, keep both observations in the receipt, and continue without a card change by default. If it repeats, classify the earliest proven causal layer; do not add aliases or prompt rules merely to accommodate every one-off output.
- A noncritical prose/update/state/HUD mismatch is an observation, not an automatic repair trigger. Record it and continue on a trustworthy state when safe. Escalate to a fix candidate when the same defect reaches a third occurrence, appears in two independent chats, blocks a required core path, or corrupts later evidence.
- If the mismatch makes authoritative state ambiguous or unsafe to continue, create a fresh isolated chat for further coverage. This preserves evidence but does not by itself prove the card needs modification.

## External interaction and stale state

The user may click, regenerate, switch chats, or edit input while automation is running. On any unexpected message count, dialog, active character, chat, or input change:

1. stop the pending action sequence;
2. discard stale selectors and expected floor numbers;
3. re-read active identity, chat, messages, dialogs, and generation state;
4. preserve unexpected input, record the responsible actor, and incorporate the user's action as observed evidence;
5. resume only from a valid state.

## Visual fallback

Use visual inspection only when semantic access is insufficient or the acceptance row is visual. Capture the relevant region, not the entire page by default. After a visual click, verify the same semantic postcondition as any other operation. A screenshot never proves persistence, console health, MVU state, or active artifact identity.

## Authorized production promotion

Enter this sequence only after all declared `variable-repair` release gates pass and the run already records production-promotion authority and whether the exact production target initially is `existing` or `absent`:

```text
test-passed
-> production-identity artifact parity verified
-> test card no longer active
-> host and helper writes quiescent
-> existing: exact production PNG backed up outside host data stores, then atomically replaced
-> absent: zero path and host-identity collisions proven, then production PNG atomically created
-> host fully reloaded
-> production identity and complete payload verified
-> initialization, settlement, persistence, and HUD smoke passed
-> obsolete run-owned artifacts removed; stable test pair and latest test chat retained
```

Prove quiescence using the strongest available host or extension signal; elapsed time alone is insufficient. The production artifact must come from the same source revision as the passing candidate and may differ only in declared identity fields and required packaging metadata.

If any post-promotion check fails, roll back according to the recorded initial state before cleanup. For `existing`, restore the exact backup atomically and verify its SHA-256 and running identity after reload. For `absent`, remove only the run-created production target and reload until both its exact path and host identity are absent. Record the failed promotion and rollback evidence; a filesystem mutation alone is not a verified rollback.

## End and cleanup

Mark every mutation: deployed file, standalone worldbook, created test chat, generated floor, setting change, reload, retry, production replacement, rollback, and deletion. Restore temporary nonpersistent UI state when safe.

Unless a stop policy explicitly freezes the failure scene for user review, after writing the receipt delete only run-owned obsolete artifacts: older test chats from this run, duplicate/temporary worldbooks, and temporary deployment files. Keep one stable test card, its one stable test worldbook when declared, and only the latest run-created test chat: the latest accepted chat on success or the latest diagnostic chat on failure/blockage. Preserve production chats, unrelated cards and worldbooks, extension settings, the receipt, and one rollback backup outside SillyTavern data stores only when production previously existed. On failure or blockage, capture evidence before cleanup and do not mutate production beyond the verified rollback path.
