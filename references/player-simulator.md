# Single Human Player Simulator

Use this reference for autonomous multi-turn play. The player and auditor are logical roles with separate information boundaries; they may run in one agent as long as the boundary is explicit and maintained.

## Inputs

The player may see only:

- public card name, tags, description, and user-facing setup;
- rendered chat history, visible HUD, available buttons, and interaction feedback;
- its own player memory and the selected message it previously sent;
- the play objective supplied by the user, such as “reach a declared story milestone,” plus any public behavioral constraint the user explicitly gave;

The player may not see source files, worldbook secrets, future scenes, hidden variables, update blocks, console logs, expected loot, or auditor conclusions. A behavioral test intention must not disclose the expected answer.

## Establish the player

Before the human-playtest begins, require a user-supplied play objective. If it is already explicit in the request, record it without asking again; otherwise ask the user. Do not replace this gate with an inferred genre goal, a default turn count, or “play generally.”

At the start, infer and record only the remaining player context:

- the role and voice implied for `{{user}}`;
- current needs, curiosities, commitments, and uncertainty;
- the card's publicly stated rules and the player's apparent capabilities.

Treat the objective as a long-term motivation, not a command to choose the shortest path. A real player may prepare, explore, socialize, investigate, retreat, or pursue a side interest when the visible situation makes that natural. Do not invent a comprehensive persona unless the opening asks for one. Keep one continuous player rather than switching archetypes mid-run.

## Memory

Retain a compact visible-memory ledger:

```yaml
recent_actions: []
recent_phrasing: []
active_plans: []
promises: []
known_facts: []
unknown_or_doubted_facts: []
failed_or_refused_attempts: []
subjective_relationships: []
current_emotion: ""
```

Update it only from visible outcomes. Use it to avoid repeatedly observing the room, checking the same inventory, asking the same question, or recycling the same sentence.

## Generate four eligible options

Each turn create exactly four internal options. They must:

- connect naturally to the visible context while allowing a real change of mind;
- be concise, specific, and meaningfully different;
- use an emoji prefix expressing `{{user}}`'s intention or emotion;
- optionally include `{{user}}`'s speech, action, or brief internal reaction;
- describe an attempt, method, request, or decision, never an unearned result;
- use only visible knowledge and plausible abilities;
- give the character or system something concrete to answer;
- avoid recent actions, language, and events unless repetition is itself natural.

At most one option may be negative, coercive, high-risk, rule-challenging, or abruptly change time, location, or topic. Such an option is valid when a real player might enter it. It may attempt an impossible result, but must phrase the attempt rather than grant success so the card can refuse or adjudicate it.

Do not force four fixed categories. A relationship scene may yield four social choices; a construction scene may yield four methods. Diversity is contextual, not a slot template.

## Choose without scoring

Do not rank, optimize, or score the options. Reject only options that violate the eligibility rules, regenerate replacements, then choose uniformly among indices `0..3` using the run's recorded random seed. Send only the selected option to SillyTavern. Save all four options, the chosen index, and the seed state in audit evidence; do not expose them to the card.

## Continue like a player

After each response:

1. read only rendered, player-visible consequences;
2. update the visible-memory ledger;
3. notice urgency, invitations, confusion, and unresolved consequences;
4. generate four fresh options and randomly choose again.

The player may use short, vague, playful, emotional, cautious, impulsive, or creative language. It may abandon a plan, wait, jump time, move elsewhere, or change topics. The auditor, not the player, checks whether the card handled those discontinuities correctly.

If the response is absent, duplicated, visibly truncated, or the UI says generation failed, the operator follows the retry policy in the operation state machine. The player does not hallucinate a reply.

## Continue until a natural terminal condition

Do not predeclare a turn count or stop because a round budget was reached. Continue until one of these is observable:

- the user-supplied objective is completed;
- the objective has visibly failed or become unreachable in the current run;
- a blocking card, host, model, provider, or driver fault makes further play invalid;
- progress requires a new user decision, new authority, or a materially different objective;
- the user stops the run.

Nonblocking technical findings do not automatically end play. The auditor records them while the player continues from visible information.
