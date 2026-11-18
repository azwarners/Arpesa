# Arpesa — Living Design Roadmap (WIP)

> **Status:** Working design notes, captured from exploratory discussions on November 17, 2026.
>
> **Not a finalized specification or implementation commitment.** This document preserves agreed direction, candidate approaches, and unresolved decisions. Validate assumptions with prototypes before turning them into READY implementation issues.
>
> **Pronunciation:** ar-PEH-sah. **Expansion:** AI Role Play Enhancement Software for Applications.

## 1. Vision and experience

Arpesa is middleware for persistent, dynamically generated, text-first roleplay worlds. The player participates as one character; AI narrators and characters portray the surroundings and everyone else. The player supplies actions, dialogue, and broad creative direction. The AI is encouraged to improvise, surprise, and develop the world.

The experience can be quiet slice-of-life, exploration, relationships, mystery, fantasy adventure, or conventional RPG/LitRPG gameplay. It must **not** require quests or action to remain engaging: a player should be able to sit near an airport runway, at a spring-break beach, or in a state park and receive changing ambient narration without typing.

**Guiding rule:** Generate on encounter. Persist on creation. Retrieve on relevance. Nothing off-screen needs to run continuously merely to justify later history, but once fiction becomes established canon, continuity matters.

An example: Thomas Anderson leaves Chandler for Flagstaff. A diner, waitress Elaine, and an NAU-student daughter are invented only when relevant. Much later Elaine can mention that Thomas's friend Sarah stopped in last month. That encounter is generated *at that moment*, checked against existing world facts and knowledge, and accepted into history per configured canon policy—never simulated independently while the player was absent.

Worlds are not constrained to contemporary Arizona. Other players could create unrelated fantasy, historical, or science-fiction worlds.

### Deliberate non-goals of the underlying engine
- No mandatory continuous simulation of cities, crowds, populations, or NPCs outside active sessions.
- No precreation of billions of NPC profiles or geography before exploration.
- No requirement to be a videogame, render graphics, or follow a scripted narrative.
- No requirement to use two physical models, SillyTavern, or any particular frontend forever.
- No replacement of existing Aggregare capabilities where they can be composed.

## 2. Product naming and ownership

**Arpesa** names the reusable middleware, not any one player's world. **Nickville** is the working name of the example personal universe. Alternate world names and multiple independent fictional universes should be possible; world isolation and save/branch behavior are still design questions.

Proposed component boundaries, pending integration validation:

| Component | Responsibility |
| --- | --- |
| Arpesa | World state/canon, fictional chronology, narrative orchestration, lazy world/history generation, continuity, quest state, communications rules, approvals |
| Apmatia | Narrator/inhabitant agent identities and prompt configurations; agents are dormant configuration records until invoked |
| Sidecaravan | Memory capture, persistence and contextual retrieval; planned per-agent memory separation |
| Ysparr | LLM request/response gateway |
| SillyTavern | Candidate first frontend, adapted with an Arpesa extension; **not** authoritative world storage |
| Aggregare | Catalog, deployment, configuration and integration through reusable Ansible roles/playbooks |

**Mandatory deployment principle:** Arpesa, extensions, and all accompanying component configuration must be handled repeatably by **Aggregare Ansible automation**, for other users as well as the author's VM. Do not make manually edited local installation state the product. Keep component contracts independent so other clients can be supported.

SillyTavern's built-in character cards, lorebooks, vectorization and summaries overlap with Apmatia/Sidecaravan and should not silently become competing sources of truth. It is worth evaluating as a frontend rather than excluding it without testing.

## 3. Characters and social life

- AI invents characters naturally as the player encounters them, and may introduce friends, relatives, colleagues and other connections of established inhabitants.
- **An encountered inhabitant can be an Apmatia agent.** Agents are small stored prompt/config records, not running services or processes. No special argument against many dormant agents based solely on their count.
- Partial identities are fine: Elaine can mention a daughter at NAU before the story has invented the daughter's name, appearance, and personality. Complete her profile when encountered.
- Persist stable IDs, alias/name changes, relationships, relevant history, and last established state. Never confuse identities because of matching names.
- People have separate social circles; the world does not imply that everyone knows everyone. Characters need not know the player.
- Per-agent memory should limit a character's personal recollection to conversations and information they participated in or plausibly learned, while Arpesa retains shared objective canon. Knowledge, belief, rumor, lies, and objective truth must not be conflated.
- If a recurring character returns years later, retrieve relevant memories and world updates so they can recognize the player consistently.
- **Open design question:** when several NPCs are present, perform separate Apmatia agent inferences or compose their agent prompts/memory into one dialogue-model call? Cost, latency and character separation must be tested.

## 4. Memory, canon and retrieval

**The world needs an overview and a way to drill down.** A large vector index alone is not sufficient: semantic similarity can miss exact identities, relationships, chronology and contradictions.

Proposed searchable knowledge layers:
- Global world rules, setting, tone, era, major established history and current story state.
- Hierarchical geography and locations (world/region/city/venue).
- Apmatia identities and structured relationships among characters.
- Events, chronology and dependencies, including off-screen events invented retroactively.
- Sidecaravan recollections and player/agent-specific knowledge.
- A scene-specific context package assembled from relevant portions of these layers.

Design goal: retrieve sufficient context to avoid forgetting important facts without loading the whole world into every LLM context. Structured indexes, persistent identity links, event provenance, and targeted vector search can complement one another. Exact storage technologies, schemas and retrieval strategy remain undecided.

**Canon is not equivalent to all generated prose.** Distinguish proposed facts, committed world facts, individual character reports, and discarded branches/rerolls. Record what was established, when it happened in world time, who knows it, and relevant source/dependency information. Proposed/rejected alternatives must not leak into authoritative canon or Sidecaravan personal memory.

Before inventing an event involving established NPCs, retrieve pertinent character, place and time records to avoid contradictions (e.g. Sarah cannot visit Flagstaff during a documented hospitalization elsewhere). A narrator can convey unreliable testimony without making it objective truth.

## 5. Lazy generation of geography and history

Use a **Minecraft-chunks analogy for text**, not an actual continuously running world simulator. Invent people, venues, and regional details on approach, constrained by already-established neighboring context, real geography when appropriate, world rules and chronology. Once discovered, retain consistent identities, locations and relationships.

Off-screen history is likewise created **on demand**. If Elaine says Sarah visited last month, that is a proposed retrospective event generated when it becomes relevant; no off-screen NPC activity was required. The system checks its fit with established history and applies the configured approval process.

The AI should preserve creative freedom and narrative surprise. A player may also change direction, revise, or reroll unwanted fiction. We have not decided how much causal planning is needed before an event is narrated.

## 6. Clock, active world and scene pacing

**Agreed desired experience:** active-world narration (option A). During an active, unpaused session, fictional time progresses and the narrator can emit events **without waiting for user input**. A player may simply sit and observe aircraft, people, weather or wildlife.

- Begin with **1× real-world clock speed** to allow believable conversations and daily activities; avoid very short Minecraft-style days.
- Include a prominent **Pause / Resume** control (including bathroom breaks). Pausing freezes fictional time and unsolicited narration. Resume must not fast-forward the paused interval.
- Consider configurable game-clock offset to shift time of day. A fixed offset alone does **not** solve repeated same-time daily sessions.
- Support **narrative time jumps** (travel, sleep, skipping uneventful hours) without forcing literal waiting or continuously simulating the intervening world.
- Distinguish wall time, fictional time, and time consumed by narrated actions. Preserve continuity across jumps, pauses and speed/offset changes.
- Prefer variable, context-sensitive **ambient narration pacing** over a fixed flood of LLM requests. Differentiate ordinary ambience, social opportunities, and rare significant incidents; don't force danger into every peaceful scene.
- No autonomous background population simulation when the player is offline. Whether fictional time advances or pauses **between sessions** remains unresolved. Decide world/session policies explicitly, including disconnect behavior.

An active-session scheduler/director can be mostly ordinary application logic; it need not require a third LLM. Stop timers and inference when paused/closed. Treat overlapping narration/player interaction, cancellations and scene changes deterministically.

## 7. Multiple output panes and model roles

**Desired UI behavior:** an **output-only Narrator pane** is separate from the interactive player/character conversation pane. The narrator describes environment, actions, sounds, incoming communications, reactions and surprises (phone rings, lightning, someone enters the diner), and can speak unsolicited during active play. The player does not need to prompt the narrator directly.

**Encourage, but do not require, two LLMs:**
1. Narration-focused model/request role.
2. Character dialogue/conversation-focused model/request role.

One physical/model endpoint may serve both roles via different prompts/calls. Arpesa's Director coordinates time, context, scene state, narration and characters. The scene's events must be shared consistently across panes so narration and dialogue do not contradict one another.

A separate pane is not just a cosmetic change: the client needs a way to receive narration initiated without a preceding user chat request. Do not assume standard OpenAI-compatible chat UI transport already supports spontaneous push messages.

## 8. Creative control and canon approval

**Configurable by decision category**, not just one global yes/no. Candidate categories:
- Ambient details/narration
- Random encounters
- Quest trigger events
- New characters
- Retroactively generated history
- Major life/world changes (e.g. deaths, relocation, lasting consequences)
- Additional categories as the engine evolves

Candidate policies: **Automatic**, **Review before commit**, or **Off** where meaningful, plus optional **reroll** (and potentially edit) for generated proposals. Support global defaults, per-world overrides and temporary session overrides.

Important subtlety: allowing an unexpected event to be narrated before approval preserves surprise, but any such event and dependent details must remain **provisional** until committed. Rejected/rerolled events must not leave side effects in NPC memory, quest state or history. How to stage/retract narration already shown and handle dependent events needs design work.

For reports by NPCs, approving that a statement was made is distinct from verifying that its contents are objectively true.

## 9. Gamification and quests

**Agreed direction: configurable gamification, not mandatory quests.** Arpesa should support quiet slice-of-life observation and also traditional RPG/LitRPG experiences with explicit quest journals, quest trees, prerequisites, objectives, branching outcomes and rewards.

Quest presentation modes to explore:
- Explicitly offered and visible quests.
- Discovered quests appearing in the journal when the player commits to a situation.
- Hidden/secret quests and achievements when appropriate.
- Per-world selection of formal RPG UI versus immersive narrative presentation.

**Test-driven quest principle:** define desired outcomes / acceptance conditions, **not one scripted implementation**. Players can invent solutions; the engine judges results against committed world facts. Quest trees may still have dependencies, branches, progress indicators, multiple valid outcomes, partial completion and failure. Narrative and quest progress must stay coherent.

Rewards can be mechanical (XP, items, levels, currency, skills), social (friendship, trust, introductions), world-related (access, places, opportunities), or purely narrative (discovery, resolution). No requirement that passive observation earns the same progression as active challenges. Each world may choose its progression model or disable it.

For mysteries, consider optional **private/hidden quest canon** to establish underlying facts before the solution is discovered; open-ended social and political conflicts may instead evolve dynamically. Whether/when hidden truths are precommitted remains undecided.

## 10. Fictional distance communication

**Do not hardcode a smartphone into the core engine.** The requirement is for world-aware communication among characters who are not physically co-present.

Examples: cell texting, voice calls, telegrams, letters, Pony Express, magical sending stones, telepathy, holographic calls. The same underlying concepts—participants, message history, medium, range, delivery delay, privacy, visibility to nearby characters, reliability, device/access constraints and knowledge transfer—can be presented with different skins.

A positive social interaction may, by configurable world convention, lead characters to offer a contact method freely. The player might text Elaine from Chandler; a friend present might notice and ask whom they are contacting. A later in-person meeting should incorporate established communications.

Messages are **world events**, not isolated UI transcripts. Fictional delivery/reply timing must use the game clock. Replies need not require an NPC running in the background; they may be generated at narratively appropriate delivery time.

**Frontend research candidates:**
- **EchoText**: promising lightweight character DM interface, potentially skin/theme it; investigate interface assumptions and extension hooks.
- **SillyPhone**: richer simulated-phone UI, possibly overkill for a medium-agnostic core.
- **NPC State**: inspect as a possible NPC dossier/presence frontend or reference, **not** an automatic second source of truth. Review data ownership, archived NPC retention/deletion and Apmatia compatibility before adopting.

No decision has been made to fork, install, or depend on any of these.

## 11. SillyTavern decision and first proof of concept

**Decision:** Do not rule out SillyTavern without a practical experiment. Its mature roleplay UI, extension ecosystem and OpenAI-compatible connectivity may justify using it as the **first frontend**, despite overlap with Apmatia and Sidecaravan. Arpesa core must remain client-agnostic.

**First bounded issue:** [Arpesa #1 — SillyTavern narrator pane, game clock and unsolicited ambient narration](https://github.com/azwarners/Arpesa/issues/1)

**BLOCKED, NOT READY**, dependent on [Aggregare #52 — standalone SillyTavern Ansible installation and smoke test](https://github.com/azwarners/Aggregare/issues/52). Aggregare #52 deliberately does **not** integrate Ysparr, Apmatia, or Sidecaravan; don't silently expand that prerequisite.

POC:
1. Install/verify SillyTavern through Aggregare #52.
2. Add a separate read-only narrator pane through a minimal extension.
3. Provide a 1× clock with Pause / Resume.
4. Generate at least two ambient outputs while the player is idle, through a configurable OpenAI-compatible endpoint (intended Ysparr).
5. Keep normal chat usable, handle in-flight requests, pause/disconnect/scene changes, and test mobile-sized UI.
6. Document reproducible configuration inputs for **Aggregare-owned Ansible roles** and evaluate go/no-go on SillyTavern as the first frontend.

**Explicitly out of scope for the first POC:** full world canon, generated NPC persistence, Apmatia/Sidecaravan orchestration, quests, canon approval and fictional communication skins. Do not multiply READY issues or treat a one-off local proof as a portable installation.

## 12. Open decisions / validation risks

These have been discussed but **not settled**:
- Exactly how the game clock behaves while no player is connected; offset, timezone, time-jump and disconnect semantics.
- The transport mechanism for unsolicited narrator streaming and how to isolate narration from standard chat history.
- One shared dialogue inference versus multiple character-agent calls in busy scenes.
- The reliable canon extraction/commit mechanism; avoiding hallucinated, duplicated and discarded facts.
- Persistence technologies, indexing strategy, scalability and retrieval accuracy; how Arpesa/Sidecaravan/Apmatia communicate and who commits which records.
- Whether approval occurs before narration or after provisional display, and how dependent story revisions roll back.
- Quest mystery generation, hidden truths, evaluation against committed canon and optional RPG mechanics.
- Multiple-world lifecycle, savepoints, branches, import/export and privacy/ownership boundaries.
- Evaluation of EchoText versus SillyPhone and NPC State, including licensing, update compatibility and whether adapting existing extensions is cheaper than building thin UI components.
- Responsiveness and LLM cost of ambient narration plus dialogue on locally served models.
- How additional frontends can participate in the same ongoing world.

## 13. Roadmap direction (not a schedule)

1. **Prerequisite:** Aggregare #52 installs and smoke-tests standalone SillyTavern with repeatable Ansible.
2. **POC:** Arpesa #1 tests narrator pane, active clock/pause and unsolicited ambient narration, and captures frontend limitations.
3. **Architecture validation:** design the request/event path and authoritative state/memory boundaries among Arpesa, Apmatia, Sidecaravan and Ysparr.
4. **Vertical slice:** meet Elaine, create her persistent identity, exchange a communication method, leave and return, generate a plausible retroactive event involving Sarah, review/commit it, and confirm continuity.
5. **Later optional capabilities:** richer communications skins, dynamic quests and quest trees, gamification profiles, canon controls and world/save management.

Keep each future issue small, independently testable, explicit about dependencies and readiness, and built for repeatable Aggregare deployment.

---

*This roadmap records an evolving direction, not an agreement that all proposed features must ship. Change it as experiments reveal better choices.*
