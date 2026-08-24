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

Use direct deployment to the verified character store for ordinary candidate tests. The SillyTavern import UI is reserved for an explicit import-workflow acceptance row; it is not the fallback for missing upload automation and must not be used merely to simulate a human.

Record one immutable `deploymentMethod` before the first write:

- `direct-store` is the default. Its only candidate-creation mutation is the temporary copy plus atomic rename into the declared stable slot.
- `import-ui` is allowed only when the receipt explicitly includes import-workflow acceptance. In that case the direct-store path must remain absent and the UI submission may occur once.
- Do not switch methods within an iteration. A timeout, stale list, or missing visible card after either method is a recognition failure, not authorization for a second creation attempt.

1. Resolve the character directory to an absolute path and prove it is inside the active user's data root.
2. Verify the target store's accepted format from the installed version. For the common PNG store, require a decoded and parity-checked character PNG; never copy the source JSON as a character file.
3. Resolve the declared target. On first `variable-repair` deployment, the stable test-slot path must be absent. On later iterations, replace only that recorded path after its current hash matches the previous deployment receipt. Other modes fail rather than overwrite an existing character.
4. Copy to a non-card temporary extension in the same target directory, verify size and SHA-256, then atomically rename or replace the final extension. Never patch a deployed PNG.
5. Deploy declared standalone/linked test worldbooks under run-scoped names only when required. Embedded `character_book` content needs no separate copy.
6. Trigger the target version's safe list refresh or page reload. Do not infer live recognition from the file alone.
7. Resolve exactly one candidate in the refreshed runtime inventory by the declared test filename plus embedded/display identity, activate it with a version-verified host operation, and re-read its running display name, version, filename, worldbook mode, scripts, regexes, and capability markers. If direct activation is unavailable, use exact search and require one semantic result before activation. Never use list position, recency sorting, avatar artwork, visual card-wall scanning, or coordinates to discover the candidate.

If step 6 does not reveal a directly deployed candidate:

1. Re-read the declared slot path, SHA-256, and decoded embedded identity. If any differs, fail at deployment integrity.
2. Use only the installed version's safe list refresh, full page reload, or documented re-index operation against the same slot.
3. Re-inventory the host list. Do not copy again, change filenames, or open the import UI.
4. If the same file still cannot be recognized, stop `blocked` with the file and cache evidence. Do not create a chat.

Before step 7 can become `active-verified`, inventory both layers: exactly one stored payload may match the candidate hash/slot identity, and exactly one host entry may expose the declared test display name. If either count exceeds one, stop before testing. Clean up only duplicates created by this run and only within recorded cleanup authority; otherwise report the collision to the user.

State after step 4 is `deployed-unverified`; a visibility miss stays in that state; state after step 7 plus the two-layer uniqueness check is `active-verified`.

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
-> run-owned test artifacts removed
```

Prove quiescence using the strongest available host or extension signal; elapsed time alone is insufficient. The production artifact must come from the same source revision as the passing candidate and may differ only in declared identity fields and required packaging metadata.

If any post-promotion check fails, roll back according to the recorded initial state before deleting the test slot. For `existing`, restore the exact backup atomically and verify its SHA-256 and running identity after reload. For `absent`, remove only the run-created production target and reload until both its exact path and host identity are absent. Record the failed promotion and rollback evidence; a filesystem mutation alone is not a verified rollback.

## End and cleanup

Mark every mutation: deployed file, standalone worldbook, created test chat, generated floor, setting change, reload, retry, production replacement, rollback, and deletion. Restore temporary nonpersistent UI state when safe.

After successful authorized promotion, delete only artifacts recorded as owned by this run: the stable test card, its run-created chats, run-owned standalone test worldbooks, and obsolete temporary files. Preserve production chats, unrelated cards and worldbooks, extension settings, the structured receipt, and one rollback backup outside SillyTavern data stores only when production previously existed. On failure or blockage, capture evidence first and do not mutate production beyond the verified rollback path.
