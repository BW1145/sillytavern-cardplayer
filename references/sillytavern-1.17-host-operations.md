# SillyTavern 1.17.0 Host Operations

This is the Cardplayer adapter for exactly SillyTavern `1.17.0`. The pinned tag resolves to commit `e3f41666c69db032e17e079fcddcf40cf47e8593`. The immutable source links below are the authority for the 1.17.0 shapes recorded here. Other versions require fresh source or installed-runtime inspection; if a required fact cannot be matched, stop and mark it unresolved rather than guessing.

This reference says how Cardplayer can perform or observe a host operation. The shared [operation state machine](sillytavern-operation-state-machine.md) remains the only source for ordering, safety gates, retry policy, rollback, cleanup policy, and production promotion. A UI/internal flow is not presented as a public API.

## Authority and execution contexts

- **Host core:** the browser's SillyTavern page and its module context. The external bridge is `window.SillyTavern.getContext()`; it exposes the selected character/chat, chat array, event source/types, generation, chat, worldbook, and swipe helpers listed below. Exact availability must still be probed in the running page.
- **UI/internal flow:** functions in `public/script.js` and `public/scripts/world-info.js`. These are source-backed host implementation surfaces, not stable third-party APIs; use semantic UI when the UI itself is under test.
- **HTTP calls:** browser requests shown in the pinned source. They require the runtime's request headers/CSRF token and the active-user context; do not replay them from an unrelated process without a runtime probe.
- **Optional extensions:** Tavern Helper/JS-Slash-Runner and MVU are not implied by this core source. Their presence, version, iframe exposure, event timing, and settlement behavior require independent runtime probes.

## Operation inventory

Each entry records the operation ID, purpose, actual 1.17.0 surface, context, inputs and observations, side effects, preconditions, postcondition evidence, and immutable source.

### VER-001 — version and active-user data root

- **Surface/context:** page initialization calls `GET /version`; browser page context only.
- **Input/observation:** no body is constructed by the source. Read JSON fields `pkgVersion`, `agent`, and optional `gitRevision`/`gitBranch`; the UI displays them.
- **Side effects:** updates only the version display and in-memory version fields.
- **Precondition/postcondition:** page and CSRF initialization must be available; record the exact returned version before using this adapter.
- **Important boundary:** the inspected 1.17.0 public client source does not expose a stable active-user directory API. The server internally maps the session/default user to `request.user.profile.handle` and `request.user.directories`, but an absolute browser-side data root is not a public client contract. Discover it from the installed authenticated runtime or an installation-specific filesystem contract; if it cannot be proven to belong to the active user, stop before file operations.
- **Source:** [`getClientVersion`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/src/server-main.js#L262-L265), [`user directory mapping`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/src/users.js#L637-L650), [`user directories`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/src/users.js#L850-L892), [`client version display`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L501-L518).

### CTX-001 — current character, chat, and generation state

- **Surface/context:** `window.SillyTavern.getContext()` in the top-level host page; the source returns `characterId`, `chatId`, `characters`, `chat`, `streamingProcessor`, `eventSource`, and `eventTypes`. Core exports also expose `isGenerating()` and `getCurrentChatId()` inside the module context.
- **Input/observation:** call the installed bridge only after probing `typeof window.SillyTavern?.getContext === 'function'`; read the returned identity and message count. Observe generation through the context's event source/types and streaming processor, not a guessed DOM spinner.
- **Side effects:** read-only, except extension getters may expose live mutable objects; snapshot observations before a later mutation.
- **Precondition/postcondition:** active page and character/chat selection are required; postcondition is a re-read with matching character ID/avatar, chat ID, count, and generation state.
- **Source:** [`getContext` return shape](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/scripts/st-context.js#L111-L151), [`isGenerating` and chat identity](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L540-L569), [`globalThis.SillyTavern`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L279-L283).

### CHAR-001 — list and read the candidate characters

- **Surface/context:** core `getCharacters()` / `getOneCharacter(avatarUrl)` or the equivalent character-list UI.
- **Input/observation:** `POST /api/characters/all` with `{}` returns the list; `POST /api/characters/get` with `{ avatar_url }` refreshes one entry. Observe the unique avatar filename, display name, data/configuration, and current chat name.
- **Side effects:** list refresh repopulates the in-memory array and may reselect the previous avatar; single read replaces that array element.
- **Precondition/postcondition:** use the active user session and a unique declared test identity; after refresh, prove exactly one matching stored/host entry by filename plus embedded/display identity.
- **Source:** [`getCharacters` and `getOneCharacter`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L1219-L1324).

### CHAR-002 — create a test character

- **Surface/context:** `createOrEditCharacter(e)` internal form flow or the character-create UI; no stable public Cardplayer create API is declared.
- **Input/observation:** the source builds `FormData` from `#form_create`, then `POST /api/characters/create` with the avatar and character fields. A successful response returns an avatar ID as text, followed by `getCharacters()`.
- **Side effects:** creates a host-owned character file, refreshes the list, and may update auxiliary-book links from the form state.
- **Precondition/postcondition:** settings ready, target absent, valid card transport and active CSRF token; verify response success, returned avatar identity, one stored payload, one host entry, and a fresh list read.
- **Source:** [`createOrEditCharacter` create branch](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L9639-L9751), [`create endpoint`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/src/endpoints/characters.js#L1014-L1040).

### CHAR-003 — edit current character fields

- **Surface/context:** the same internal `createOrEditCharacter` form flow or semantic edit UI; not a public endpoint contract for arbitrary callers and not the full-card replacement path.
- **Input/observation:** `POST /api/characters/edit` with the host-generated `FormData`; successful update is followed by `getOneCharacter(formData.get('avatar_url'))` and a `CHARACTER_EDITED` event.
- **Side effects:** host may save the PNG and update its in-memory object; the post-edit read is required to observe the result.
- **Precondition/postcondition:** the exact run-owned target and host edit authority are proven; re-read disk/decoded identity and active host identity after the edit. Do not use this field-edit flow to claim that an immutable candidate card file was imported or replaced.
- **Source:** [`createOrEditCharacter` edit branch](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L9758-L9793), [`edit endpoint`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/src/endpoints/characters.js#L1089-L1127).

### CHAR-004 — replace/update a test character from a card file

- **Surface/context:** the character-management `replace_update` flow calls `processDroppedFiles([file], map)` with the current avatar filename as the preserved name; that reaches internal `importCharacter(file, { preserveFileName })` and `POST /api/characters/import`. This is the 1.17.0 host-owned full-card replacement path, not the character field-edit endpoint.
- **Input/observation:** multipart `FormData` contains `avatar`, `file_type`, `user_name`, and `preserved_name`. The server parses the preserved filename, imports the supplied JSON/PNG/YAML/CHARX/BYAF card, and returns `file_name`; the client reloads the existing thumbnail and then reopens the previously selected chat.
- **Side effects:** replaces the card payload under the stable avatar identity and refreshes host presentation, while preserving chats/assets/group memberships. It does not run `Import Card Lore` and therefore does not update a same-named linked worldbook.
- **Precondition/postcondition:** no generation is active; the exact run-owned stable slot, expected current hash, candidate file, and replacement authority are proven. Follow the state machine's refresh/reselect boundary, then verify decoded PNG identity, running character identity, preserved chat linkage, and unchanged worldbook until the separate lore import runs.
- **Source:** [`replace_update` UI flow](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L12266-L12327), [`processDroppedFiles` and `importCharacter`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L10352-L10485), [`import endpoint and preserved name`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/src/endpoints/characters.js#L1410-L1458).

### CHAR-005 — select the test character

- **Surface/context:** core `selectCharacterById(id, { switchMenu })`, or the semantic character-selection UI.
- **Input/observation:** numeric in-memory character index, not filename; switching clears the current in-memory chat, sets the character ID, resets chat metadata, and calls `getChat()`.
- **Side effects:** selection can clear the current chat view and load the selected character's current chat; it refuses while chat save/generation conditions block switching.
- **Precondition/postcondition:** resolve the unique character by declared filename plus embedded/display identity before converting to its current index; re-read active character and chat identity after selection.
- **Source:** [`selectCharacterById`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L861-L905).

### CHAR-006 — refresh and reselect after host changes

- **Surface/context:** `getCharacters()` plus `printCharacters(true)` and the selection flow; a full page reload is the host-level fallback, not a guessed cache API.
- **Input/observation:** call the source-backed refresh, then resolve the same unique identity and select it again. The list endpoint and rendered list are separate observations.
- **Side effects:** repopulates the list and may reselect the previous avatar; a full page reload aborts streaming on `beforeunload`.
- **Precondition/postcondition:** only after the preceding operation is quiescent; prove the refreshed in-memory character and decoded payload match before a worldbook operation.
- **Source:** [`getCharacters` refresh](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L1290-L1324). A full browser reload is a host lifecycle action; its before-unload streaming abort is visible in [`beforeunload`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L12475-L12484).

### WI-001 — delete the run-owned test worldbook

- **Surface/context:** core `deleteWorldInfo(worldInfoName)` or the semantic worldbook UI.
- **Input/observation:** exact worldbook name; the source sends `POST /api/worldinfo/delete` with `{ name }` and returns `false` when the name is not in the loaded world list or the response is not successful.
- **Side effects:** deletes the stored worldbook, clears its cache entry/selection, refreshes the worldbook list, and updates character-world UI when applicable.
- **Precondition/postcondition:** exact run-owned identity and deletion authority must already be proven; re-read the world list and confirm the name is absent. Do not use deletion as an unverified cache workaround.
- **Source:** [`deleteWorldInfo`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/scripts/world-info.js#L4210-L4245), [`worldbook delete endpoint`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/src/endpoints/worldinfo.js#L81-L96).

### WI-002 — convert and bind Import Card Lore

- **Surface/context:** `importEmbeddedWorldInfo(skipPopup)` and its internal `convertCharacterBook(characterBook)` in the host page.
- **Input/observation:** the current character's V2 `data.character_book`; the converter assigns missing IDs by index and returns `{ entries: { [uid]: nativeEntry }, originalData }`. Import saves under the book name, refreshes the list, and sets `#character_world` to that name.
- **Side effects:** creates or overwrites the native worldbook, refreshes the editor, and binds the character's primary worldbook pointer through the UI field. The converter maps keys, content, enabled/disabled state, insertion order, position, and extension fields into native entries.
- **Precondition/postcondition:** the refreshed in-memory character must be the new candidate before import; after import, read the native worldbook, confirm entries are keyed by native UID, verify critical entries/content identity, confirm the pointer, then re-read the card identity. A V2 `character_book.entries[]` array is not a native `entries.{uid}` object and must not be submitted as-is.
- **Source:** [`convertCharacterBook`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/scripts/world-info.js#L5480-L5541), [`importEmbeddedWorldInfo`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/scripts/world-info.js#L5594-L5631).

### WI-003 — read the native worldbook and critical entries

- **Surface/context:** core `loadWorldInfo(name)` and `updateWorldInfoList()`; bridge exposure is `getContext().loadWorldInfo`/`updateWorldInfoList` in this tag.
- **Input/observation:** `loadWorldInfo(name)` reads cache or sends `POST /api/worldinfo/get` with `{ name }`, returning the parsed native object or `null`; `updateWorldInfoList()` refreshes `world_names` from `/api/settings/get`.
- **Side effects:** populates the in-memory worldbook cache and refreshes editor/list controls.
- **Precondition/postcondition:** exact worldbook name is known; require a readable object, `entries` object, stable UIDs, expected content identity, and every declared critical entry before generation.
- **Source:** [`loadWorldInfo` and list refresh](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/scripts/world-info.js#L2036-L2077), [`native entry template`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/scripts/world-info.js#L3977-L4050), [`worldbook get/edit endpoints`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/src/endpoints/worldinfo.js#L39-L79), [`worldbook edit endpoint`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/src/endpoints/worldinfo.js#L134-L156).

### CHAT-001 — list and choose/create a test chat

- **Surface/context:** character chat list UI and `POST /api/characters/chats`; no separate stable “create empty chat” endpoint is exposed by the inspected client flow.
- **Input/observation:** send `{ avatar_url }`; the response is an object of chat summaries with `file_name` and timestamps. To create a fresh chat, choose a unique file name through the host `doNewChat()` flow; the empty chat becomes persisted when `getChatResult()` calls `saveChatConditional()`.
- **Side effects:** selecting a chat updates the character's current chat pointer and may save the character metadata.
- **Precondition/postcondition:** active character is verified and target chat name is run-owned; re-read the chat list and current `chatId`/message count.
- **Source:** [`character chat listing`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L8395-L8418), [`doNewChat`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L10509-L10537), [`empty-chat persistence path`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L7585-L7603).

### CHAT-002 — switch/load a test chat

- **Surface/context:** bridge `openCharacterChat(file_name)` or the semantic chat selector.
- **Input/observation:** file name without an invented endpoint; the host waits for chat saving, clears the in-memory chat, sets the character's chat pointer, calls `getChat()`, updates the selector, and emits the host's new-chat event when applicable.
- **Side effects:** clears/replaces the visible chat and may save character metadata via `createOrEditCharacter(new CustomEvent('newChat'))`.
- **Precondition/postcondition:** no generation or pending save; re-read active character, chat identity, header, and complete message count after load.
- **Source:** [`openCharacterChat`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L7645-L7653), [`getChat`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L7535-L7583).

### CHAT-003 — read, send/save, and observe persistence

- **Surface/context:** `getChat()` / `saveChat()` / `saveChatConditional()` through the bridge or the user-facing input path.
- **Input/observation:** `getChat()` posts `{ ch_name, file_name, avatar_url }` to `/api/chats/get`; `saveChat()` posts `{ ch_name, file_name, chat, avatar_url, force }` to `/api/chats/save`, where `chat` contains the header and message array. Observe the persisted response, in-memory `chat`, and rendered floors separately.
- **Side effects:** chat metadata and message arrays are saved; integrity errors can trigger a host confirmation flow, so do not force overwrite without the declared authority.
- **Precondition/postcondition:** exact active character/chat identity and expected input text are known; after a send/generation, re-read the persisted chat and require the intended new floor, not just a resolved browser call.
- **Source:** [`getChat`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L7535-L7569), [`saveChat`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L7296-L7351), [`chat get/save endpoints`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/src/endpoints/chats.js#L470-L543), [`context save/generate bridge`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/scripts/st-context.js#L135-L153).

### MSG-001 — ordinary user message

- **Surface/context:** user-facing `#send_textarea`/send control is the preferred ordinary path; the host context exposes `generate`, `addOneMessage`, and save helpers, but those are host implementation surfaces.
- **Input/observation:** enter one intended user message through semantic DOM interaction, submit once, and observe the new user floor before generation. Do not use `generateRaw` for a normal chat turn; it supplies a custom prompt and is not equivalent to normal chat persistence.
- **Side effects:** inserts/renders a user floor and triggers message events; it does not itself generate the assistant reply. The normal user-facing send flow may then call `Generate`.
- **Precondition/postcondition:** input and focus are read immediately before submission; postcondition is exactly one intended persisted user floor with the correct active chat identity.
- **Source:** [`sendMessageAsUser`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L5785-L5833), [`getContext` message/generation surfaces](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/scripts/st-context.js#L135-L153), [`host generation entry`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L4207-L4238).

### GEN-001 — generation start/end and reply persistence

- **Surface/context:** `getContext().eventSource` with `eventTypes.GENERATION_STARTED`, `GENERATION_STOPPED`, `GENERATION_ENDED`, and `MESSAGE_RECEIVED`; the host's `streamingProcessor` gives live state, while `saveReply()`/`saveChatConditional()` establish persistence.
- **Input/observation:** normal generation is started by `Generate(type, options, dryRun)`; record the event payloads and then verify an assistant floor in both `chat` and `/api/chats/get` output. A hidden stop button or completed request is not sufficient evidence.
- **Side effects:** streaming can create/update a message before the final chat save; generation end events can fire on stop-button cleanup as well as successful completion.
- **Precondition/postcondition:** active runtime-ready chat and no concurrent generation; postcondition requires terminal generation state plus a persisted assistant floor or a visible terminal error.
- **Source:** [`event constants`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/scripts/events.js#L3-L25), [`Generate start`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L4207-L4217), [`saveReply`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L6543-L6603).

### SWIPE-001 — swipe or regenerate one message

- **Surface/context:** bridge `swipe.to`, `swipe.left`, `swipe.right`, and `swipe.refresh`, or the semantic message swipe/regenerate controls.
- **Input/observation:** `swipe(event, direction, { source, repeated, message, forceMesId, forceSwipeId, forceDuration })`; the host updates `swipe_id`/`swipes`, may generate an alternate, waits for that generation, and saves/reloads as needed. Observe the selected floor and persisted swipe data.
- **Side effects:** changes the selected alternate or creates one; it refuses while generation is active or when the message is not swipeable, and overswipe behavior is host-configured.
- **Precondition/postcondition:** target message exists, current generation is terminal, and the chosen swipe is authorized; postcondition is the expected message/swipe identity in memory, rendered UI, and saved chat.
- **Source:** [`swipe bridge`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/scripts/st-context.js#L242-L250), [`swipe implementation`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L9845-L9981), [`overswipe behavior`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L10286-L10315).

### EXT-001 — MVU/UI settlement boundary

- **Surface/context:** optional Tavern Helper/JS-Slash-Runner and MVU globals, not SillyTavern core. The core context only exposes generic event/generation/message surfaces; it does not prove an MVU settlement API.
- **Input/observation:** at runtime probe `getTavernVersion`, `getTavernHelperVersion`, `window.TavernHelper`, and `window.Mvu` in the exact execution context. If present, use the installed declarations/events and observe the extension's update-ended or UI-ready marker separately from core generation and chat save.
- **Side effects:** extension scripts may update message-floor variables, inject/render UI, or persist on their own schedule.
- **Precondition/postcondition:** the exact extension versions and capability markers are known; no core-only result may be reported as MVU/UI settlement. If the marker or version contract is unresolved, stop that capability as unverified.
- **Source:** no core 1.17.0 source proves these optional capabilities. Use the installed extension source/declarations and a runtime probe; do not substitute a moving branch or guessed symbol.

### CLEAN-001 — delete only this run's artifacts

- **Surface/context:** use the same verified host operation for each recorded run-owned character, worldbook, chat, or temporary file; chat deletion is `deleteCharacterChatByName(characterId, fileName)` or the semantic chat UI, and worldbook deletion is `deleteWorldInfo(name)`.
- **Input/observation:** exact recorded identities and cleanup authority only. Re-read each target after deletion and record absence; do not enumerate or delete unrelated data.
- **Side effects:** deleting the active chat can cause the host to select/create another chat; deleting a worldbook refreshes selection and links. These are observations to re-check, not a new policy.
- **Precondition/postcondition:** the state machine has authorized the cleanup target; postcondition is exact target absence and preserved unrelated data. The state machine remains the source for what is retained or when cleanup is allowed.
- **Source:** [`chat deletion`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/script.js#L1354-L1398), [`chat delete endpoint`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/src/endpoints/chats.js#L579-L601), [`worldbook deletion`](https://github.com/SillyTavern/SillyTavern/blob/e3f41666c69db032e17e079fcddcf40cf47e8593/public/scripts/world-info.js#L4210-L4245).

## Required host hazards

- **Character replacement does not update a same-named worldbook automatically.** The replacement flow reloads the current chat after importing the new card; the separate `Import Card Lore` operation is what converts and saves the embedded book. Treat character replacement and worldbook update as separate observations.
- **V2 and native worldbook shapes differ.** `character_book.entries[]` is converted into native `entries` keyed by UID. Never pass the V2 array directly as native `entries.{uid}`.
- **Stale character objects can reverse-save old PNG data.** The source's character/worldbook flows retain a live character object and can save character metadata; therefore the state machine's refresh, reselect, and post-import PNG/identity re-reads are mandatory. This adapter records the hazard only; it does not duplicate the state-machine sequence.

## Version boundary

The facts above are verified against the 1.17.0 tag and pinned commit only. For any other SillyTavern, Tavern Helper, or MVU version, inspect the installed source/declarations and run the required capability probes. If an exact operation, signature, event payload, active-user path, or settlement marker cannot be confirmed, stop rather than infer it from this document.
