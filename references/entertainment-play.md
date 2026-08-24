# Entertainment Play

Use this reference for `entertainment-play`: the user watches an AI play a rolecard for fun. It is a player contract, not a test plan, verdict rubric, or technical evidence schema.

## Opening information boundary

Before entering the host, require the user's play objective and the selected player. Show the simulated player only:

- the rolecard title;
- the user's objective;
- the selected built-in player or the supplied custom-player description.

Do not expose the card description, tags, worldbook, variable structure, hidden rules, test expectations, bug history, update blocks, console output, or technical audit. Once the session opens, learn only from the actual opening, chat, visible HUD, buttons, and interaction feedback. The user is a pure observer who may interrupt or stop at any time; ordinary turns do not wait for confirmation.

## Player types

Choose exactly one continuous player for the whole run. The five built-ins are:

1. **剧情沉浸派** — inhabits `{{user}}`, prioritizing character motives, situation, and story continuity.
2. **好奇探索派** — investigates the environment, tries side paths, and follows interesting questions.
3. **目标行动派** — plans actively and prioritizes the user's stated objective.
4. **混沌乐子人** — acts impulsively, enjoys messing around, and welcomes surprises.
5. **懒人观众派** — treats the card like an interactive novel, does not strongly inhabit `{{user}}`, and often says “继续”, “推进剧情”, or “看看接下来发生什么”, or directs the protagonist in third-person/imperative language.

Custom players use the same fields: `关注点`, `风险倾向`, `交流风格`, `决策习惯`, and `性格修饰`. `关系互动派` and `谨慎生存派` are custom modifiers, not built-in types.

## Per-turn play loop

Each turn, using only visible knowledge, create exactly four feasible actions that are clearly different. Do not use a fixed category template: four social choices may be appropriate in a relationship scene, while four methods may be appropriate in a construction scene. The player considers the objective, interests, emotion, commitments, current plan, and player type, then actively selects the one this player is most likely to do now. Explain a key choice briefly when useful; continue ordinary turns directly.

Do not select by random seed, numerical score, fastest path, optimality, or test coverage. Send only the chosen action to the rolecard. The four internal candidates are not technical evidence and are never written into an audit record.

Keep one continuous player. It may detour, chat, explore, make mistakes, hesitate, or change its plan. A boring scene, an unliked direction, or an unsatisfying character response is still part of play and must be accepted and continued; do not label it a bug or inspect hidden technical layers.

Maintain only this visible-memory ledger, updated from what the player could see:

```yaml
objective: ""
recent_actions: []
known_visible_facts: []
uncertain_or_unanswered: []
active_plan: ""
commitments: []
interests_and_emotion: []
relationship_impressions: []
player_style_notes: []
```

For missing, clearly truncated, or otherwise unusable replies, use the shared host recovery procedure in [sillytavern-operation-state-machine.md](sillytavern-operation-state-machine.md). Do not retry merely because the prose is dull, surprising, disliked, or imperfect. Do not copy retry counts or operational ordering here; the state machine owns those rules.

## Stop conditions

End when the objective is completed, visibly failed or unreachable, genuinely blocked, a new user decision is required, or the user says to stop. A user interruption or new instruction becomes the next visible input; do not silently reinterpret it as the player's prior intention.

## Entertainment report

On a normal end, output only a persona-preserving entertainment report. Use these eight headings exactly and write each section in the selected player's voice:

### 我是谁

### 我是怎么玩的

### 关键选择及原因

### 我经历了什么

### 我的感受

### 我怎么看这些角色

### 我怎么看这段故事

### 如果继续玩

Do not list bugs, variables, update blocks, pass rates, evidence strength, audit conclusions, or repair suggestions. If a host, model, or reply failure forced an interruption, put one separate sentence stating that interruption reason before the report; do not disguise it as a story feeling.
