# Mode Test Matrix

Select rows from declared card type and capabilities. Mark irrelevant rows `not_applicable`; do not force MVU, an opening form, worldbooks, or embedded UI onto cards that do not declare them.

## Mode ownership

| Mode | Required work | Continuation |
| --- | --- | --- |
| `variable-repair` | deterministic fixtures and four-layer variable evidence on one stable test slot | authorized repair/build/retest loop may continue; pre-authorized promotion may follow a complete pass |
| `human-playtest` | one user-objective natural session, per-turn hidden audit, complete-chat analysis | report to user and stop; no automatic modification |
| `full-gate` | finish variable checks first, then one human-playtest | stop after the human report |

## Deterministic smoke checks

First build a schema-derived coverage inventory:

```text
declared mutable domain
-> representative paths
-> deterministic fixture
-> expected prose/update/state/HUD transition
-> pass, fail, blocked, not_applicable, or not_run
```

Every declared mutable domain must have a fixture or appear in `untestedScope` with a reason. Do not infer whole-card coverage from one successful inventory or time transition.

| Check | Action | Required evidence |
| --- | --- | --- |
| Active identity | select freshly deployed candidate | running version/file/capabilities match receipt |
| Deployment uniqueness | inventory the store and host list before chat creation | one matching candidate payload, one matching host identity, and one recorded deployment method/creation attempt |
| Cache-miss recovery | directly deploy, then encounter a stale or missing host-list entry | refresh/reload/re-index the same slot only; no import UI, alternate filename, or second candidate is created |
| Fresh chat | create isolated chat | new chat identity; existing chats retained |
| Opening | use declared first-message or zero-layer path | one submission, one initialized assistant reply, no duplicate binding |
| Missing-reply recovery | encounter terminal generation without a persisted assistant floor | before each of at most three regenerations, generation is re-probed and floor absence is proven; recovery stops immediately if a reply appears; a third verified failure halts all automation and returns the decision to the user |
| Pure interaction | send dialogue/inspection with no declared time cost | narrative continues; time/resources do not change unexpectedly |
| Standard action | perform one plausible time/cost action | declared time, energy, resources, and consequences settle once |
| Transaction | consume, acquire, equip, spend, or otherwise change one applicable tracked value | prose, update, persisted state, and HUD agree on quantity/location |
| Movement/state transition | request one legal transition | location/state and cost change consistently; illegal transitions are adjudicated rather than silently granted |
| Companion/NPC | issue or observe one applicable independent action | presence, agency, cost, and continuity stay consistent |
| Discontinuity | jump time, location, topic, or abandon an active plan | card accepts/refuses naturally; MVU does not skip, duplicate, or corrupt settlement |
| Reload persistence | reload/re-enter after settled state | same persisted values and one mount/listener set |
| Continue | generate one more reply after reload | prompt and story use the persisted state |
| Swipe | replace a settled assistant reply through the host Swipe path | active branch owns the authoritative state; abandoned branch does not leak events or values |
| Edit | edit an applicable user or assistant floor and continue | declared recomputation policy is followed; no duplicate settlement or stale HUD remains |
| Chat switch | switch away from and back to the isolated chat | each chat restores its own state; no cross-chat variables, listeners, input, or pending events leak |
| Stable-slot refresh | deploy a rebuilt candidate after one repair iteration | the same test-card filename and embedded display name now report the new candidate ID/hash; disk and host uniqueness counts remain one |
| Iteration isolation | open the rebuilt candidate after refreshing the stable slot | a fresh isolated test chat initializes without state from the previous iteration |

Use card-specific required checks and forbidden regressions from the candidate receipt in addition to this matrix.

## Conditional promotion checks

Run these rows only when `variable-repair` recorded production-replacement authority before testing:

| Check | Action | Required evidence |
| --- | --- | --- |
| Production parity | build the production-identity artifact from the passing source revision | semantic card surfaces match the passing candidate; only declared identity or packaging fields differ |
| Production replacement | quiesce writes, back up production, atomically replace, and fully reload | the host reports the expected production filename, embedded identity, payload hash, scripts, regexes, and worldbook mode |
| Production smoke | initialize and perform one settlement/persistence/HUD smoke sequence | production behavior matches the passing candidate on every declared smoke surface |
| Success cleanup | remove artifacts owned by this run | stable test card, run-created chats, test-only standalone worldbooks, and obsolete temporary files are absent; production chats and unrelated data remain |
| Rollback | intentionally exercised only when promotion verification fails | the exact backup hash and running production identity are restored before test-slot cleanup |

## Four-layer consistency

For every state-changing turn capture:

```text
rendered prose claim
-> model update block or declared update mechanism
-> persisted authoritative state at the intended scope/floor
-> visible HUD or consumer
```

Examples of failures:

- prose grants wood but neither player nor camp inventory receives it;
- inventory changes while time and survival costs remain stale;
- a pending event accumulates without a resolved ID;
- persisted state changes but HUD reads an older floor;
- HUD changes optimistically but reload restores the old value.

## Human-player pass

In `human-playtest` or the human phase of `full-gate`, require the user's play objective and run until a natural terminal condition from `player-simulator.md`. Record the seed, four options, chosen index, sent text, visible response, and auditor result for each turn. There is no default or maximum turn count.

The pass should naturally exercise different behaviors. Do not secretly steer selection with scores. The test controller may request a broad behavior such as free-form interaction, relationship continuity, or creative problem solving, but may not reveal hidden solutions to the player.

Stop when:

- a blocking state/persistence/runtime defect makes later evidence unreliable;
- the wrong candidate or chat becomes active;
- a required dependency is unavailable;
- generation repeatedly fails under the declared retry policy;
- continued play would require an undeclared persistent mutation.

Also stop when the play objective is completed, visibly fails or becomes unreachable, a new user decision is necessary, or the user ends the run. If the user explicitly supplied an external time, token, or cost budget, honor it as an additional stop condition; never invent one.

## Failure layers

Classify the first causal failure:

```text
candidate/packaging
active identity or stale artifact
worldbook/prompt injection
model narration/update omission
update parser or MVU application
event consumer/settlement
persistence or message-floor scope
HUD/DOM binding
interaction/readiness
host dependency/provider/driver
```

Do not fix downstream symptoms before proving the earliest failing layer.

## Quality observations

Record evidence-backed observations without numeric scoring: repeated actions or phrases, unclear affordances, ignored choices, unsolicited railroading, NPC loss of continuity, impossible success grants, refusal to accept reasonable creativity, and long stretches without meaningful feedback. Keep subjective quality observations separate from hard runtime failures. Report the selected mode's result separately from release status and list all declared capabilities not exercised in the run.

During human play the auditor still reads update blocks, authoritative variables, events, persistence, logs, and HUD state. That access exists to detect technical failures; it must not steer the player's option generation or reveal hidden solutions.
