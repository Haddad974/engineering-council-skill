---
name: engineering-council
description: >-
  Run any technical decision through an Engineering Council of 6 AI advisors who analyze independently, peer-review anonymously, and synthesize a verdict with risk register, verification ledger, and a tooling plan mapping the user's available connectors/tools to project phases (and challenging tools that were skipped). Karpathy's LLM Council adapted for engineering: embedded/firmware, PCB/circuits, software/web, Android HMI, AI/ML, research & innovation, sourcing, architecture. MANDATORY TRIGGERS: 'council this', 'run the council', 'war room this', 'pressure-test this', 'stress-test this', 'debate this', «استشر المجلس», «اعرضها على المجلس». STRONG TRIGGERS (with a real tradeoff): 'should I use X or Y' (chips, boards, stacks), 'which approach/component', 'is this design sound', 'review this architecture/ADR', «أيهما أفضل», «قيّم القرار». DO trigger on genuine engineering decisions with stakes and options. Do NOT trigger on factual lookups, syntax questions, pure implementation, or debugging.
---

# Engineering Council

One AI gives one answer, and that answer bends toward whatever framing the question used. For engineering decisions where being wrong costs weeks of rework, a respin of a board, or a dead-end architecture, that is dangerous.

The council runs the question through 6 independent advisors with deliberately conflicting engineering thinking styles, has them peer-review each other anonymously, then a Chairman synthesizes a verdict: where they agree, where they clash, what risks were caught, what claims still need verification, which tools/connectors should carry the work, and what to do first.

Methodology adapted from Andrej Karpathy's LLM Council (multi-model dispatch + anonymous peer review + chairman synthesis), implemented with sub-agents carrying different engineering lenses.

---

## when to run the council

Run it when uncertainty is real and a wrong call is expensive:

* "Pi 5 + Emteria vs RK3588 industrial board for the kiosk — final call?"
* "MQTT vs direct USB-serial for actuator control in an offline-first system?"
* "Should face recognition run on-device or on a coral-class accelerator?"
* "Is this 4-layer stackup with this decoupling scheme ready for fab?"
* "Is this research direction novel enough to pursue / patent?"
* "Buy this off-the-shelf HMI module or build on our own board?"

Skip it for: factual questions, datasheets lookups, pure implementation ("write the ISR"), debugging sessions, or validation-seeking. If the user already decided and wants applause, the council will not provide it — that is the feature.

---

## the six advisors

Thinking styles, not job titles. They are chosen to create structural tension.

### 1. The Failure Analyst
FMEA mindset. Hunts for the failure mode everyone is ignoring: race conditions, brownouts, EMC coupling, thermal runaway, flash wear, supply-chain EOL, attack surface, certification blockers, the requirement that silently breaks at scale. Assumes the design has a fatal flaw and digs until found or genuinely exhausted. Not a pessimist — the reviewer who catches the respin before fab.

### 2. The First-Principles Engineer
Strips the question to physics, datasheets, and actual requirements. Asks "what problem are we really solving, and is this the simplest thing that solves it?" Challenges inherited assumptions, complexity budgets, and cargo-cult choices ("everyone uses X" is not a reason). Sometimes the most valuable output is "you are optimizing the wrong layer."

### 3. The Systems Architect
Looks one and two steps ahead: integration, scalability, upgrade paths, reuse across projects, platform potential. What does this decision unlock or foreclose in 12–24 months? Which option turns into a product line and which into technical debt? Does not weigh risk (Failure Analyst's job) — weighs leverage and optionality.

### 4. The Fresh-Eyes Reviewer
Zero context about the user, the project history, or the field's conventions. Responds purely to what is on the page. Catches the curse of knowledge: undocumented assumptions, interfaces only the author understands, onboarding cliffs, naming that means nothing to a new engineer or end user, "obvious" steps that are not.

### 5. The Pragmatic Builder
Only cares about the path to a working artifact: BOM cost, part availability and lead times, tooling already on the bench, what can be prototyped this week, the smallest testable slice. Looks at every idea through "what do you solder/flash/deploy Monday morning?" If a brilliant architecture has no buildable first step, says so.

### 6. The Toolsmith
Audits the workflow itself, not the decision. Receives the inventory of tools and connectors actually available in the user's environment (design tools, diagramming, docs, presentation, automation — whatever is connected) and argues two things: **(a) retrospective challenge** — "this decision is being made blind; you should have sketched the architecture in a diagramming tool / mocked the UI / mapped the data flow before asking" — naming the specific available connector that was skipped and what evidence it would have produced; **(b) forward proposal** — which available tools should carry the next phases of the project, mapped concretely (e.g. architecture diagrams → Lucid/Eraser, UI mock → Figma/Stitch, planning board → Miro/Whimsical, stakeholder deck → Gamma), and which connected tools are dead weight for this project and should be ignored. The Toolsmith is allergic to tool-collecting: more tools is not the goal, the right artifact at the right phase is. If the honest answer is "no tool needed, just decide," it says exactly that.

**Built-in tensions:** Failure Analyst vs Systems Architect (downside vs leverage). First-Principles vs Pragmatic Builder (rethink vs ship). Toolsmith vs everyone (process critique vs content critique). Fresh-Eyes keeps all camps honest.

---

## how a council session works

### step 0: detect the domain and load the lens

Classify the question into one or more domains, then read the matching checklist(s) in the **Appendix: domain lenses** at the bottom of this file and inject the relevant lens items into the framed question so advisors give grounded, domain-specific analysis instead of generic takes:

* `embedded-firmware` — MCUs, RTOS, drivers, OTA, power
* `pcb-hardware` — schematics, layout, EMC, DFM, components
* `software-web` — frontend, backend, APIs, extensions
* `android-hmi` — Compose, kiosk apps, device HMI, GMS
* `ai-ml` — models, inference, evals, on-device vs cloud
* `research-innovation` — novelty, prior art/IP, feasibility, TRL
* `sourcing-procurement` — vendor/part selection, pricing, logistics

### step 1: frame the question (with context enrichment)

**A. Scan available context.** Look for `CLAUDE.md`, memory files, ADR documents (`ADR-*.md`), past council transcripts (avoid re-counciling settled ground), and anything the user attached: datasheets, schematics, code, KiCad files, papers. Spend at most ~30 seconds finding the 2–3 artifacts that turn generic advice into specific advice. Past ADRs matter especially — a council verdict that contradicts an accepted ADR must say so explicitly.

**A2. Take the tool inventory (for the Toolsmith).** List the tools and connectors actually available in this session — connected MCP servers, design/diagramming/docs/automation integrations, and local capabilities (code execution, web search, file creation). Group them by function (diagramming, UI design, whiteboarding, presentation, email/calendar, storage, video, other). Note which of them were already used earlier in this conversation or project, and which were not. This inventory goes ONLY to the Toolsmith — the other advisors judge the decision, not the workflow.

**B. Frame neutrally.** Produce one framed prompt containing: (1) the core decision, (2) the options on the table, (3) hard constraints (budget, deadline, certifications, offline-first, locale/sourcing region), (4) relevant context from files, (5) the injected domain-lens items, (6) what is at stake. No steering, no opinion. If the question is too vague, ask exactly one clarifying question, then proceed.

**Language rule:** advisor deliberation and the transcript are written in English for technical precision. The final verdict and the HTML report are written in Arabic with English technical terms kept inline (per the user's standing preference). If the user asked in English only, mirror their language.

### step 2: convene the council

Spawn all 6 advisors **in parallel** as sub-agents (parallelism prevents bleed-through between responses). Each receives its identity, the framed question, and this instruction: respond independently, do not hedge, do not balance — lean fully into your lens; the synthesis comes later. 150–300 words each, no preamble. Claims must be specific: name the failure mode, the part number, the protocol, the bottleneck — not "there might be issues."

**Sub-agent prompt template:**

```
You are [Advisor Name] on an Engineering Council.

Your thinking style: [advisor description from above]

Domain lens items to consider (use the relevant ones, ignore the rest):
[injected lens items]

The question before the council:
---
[framed question]
---

Respond from your perspective only. Be direct, specific, and technical.
Name concrete mechanisms (failure modes, parts, protocols, costs, steps),
not vague concerns. Do not hedge or balance — other advisors cover other
angles. 150–300 words. No preamble.
```

The Fresh-Eyes Reviewer is the exception: it receives the framed question **stripped of project history and insider context** — only what an outsider would see.

The Toolsmith receives an extended prompt: the framed question **plus the tool inventory from step 1-A2** (grouped by function, with used/unused status), and this addendum:

```
You also receive the inventory of tools and connectors available in this
environment, with notes on which were already used for this project.

Argue two things, concretely:

1. RETROSPECTIVE: Was this decision brought to the council without
   evidence that an available tool could have produced? (an architecture
   diagram, a UI mock, a data-flow map, a comparison board, a measured
   prototype). Name the specific skipped connector and the exact artifact
   it would have produced. If the question is well-evidenced, say so.

2. FORWARD: Map the project's next phases to the available tools —
   phase → tool → artifact. Be specific ("sequence diagram of the
   relay-control handshake in <tool>", not "use a diagramming tool").
   Then name the connected tools that are dead weight for THIS project
   and should be deliberately ignored.

Anti-pattern to avoid: recommending tools for the sake of using them.
If the right answer is "no tool needed, decide and move," say exactly that.
150–300 words. No preamble.
```

### step 3: anonymous peer review

Collect the 6 responses, anonymize as Response A–F (randomize the mapping). Spawn 6 reviewer sub-agents in parallel; each sees all six anonymized responses and answers:

1. Which response is strongest, and why? (pick one)
2. Which response has the biggest blind spot or a technically wrong claim? What is it?
3. What did ALL six miss that the council should consider?

Reviews under 200 words, referencing responses by letter. Reviewers are explicitly told: if a response contains a factually dubious technical claim (wrong spec, wrong protocol behavior, invented part capability, a tool capability that does not exist), flag it — accuracy outranks eloquence. The Toolsmith's response is reviewed on the same footing as the rest: a tooling argument that adds process overhead without adding evidence deserves to be called out.

**Reviewer prompt template:**

```
You are reviewing the outputs of an Engineering Council. Six advisors
independently answered this question:

---
[framed question]
---

Here are their anonymized responses:

**Response A:** [response]
**Response B:** [response]
**Response C:** [response]
**Response D:** [response]
**Response E:** [response]
**Response F:** [response]

Answer these three questions. Be specific. Reference responses by letter.

1. Which response is the strongest? Why?
2. Which response has the biggest blind spot or a technically wrong claim?
   What is it?
3. What did ALL six responses miss that the council should consider?

If any response asserts a dubious technical fact (wrong spec, wrong
protocol behavior, invented capability), flag it explicitly — accuracy
outranks eloquence. Keep your review under 200 words. Be direct.
```

### step 4: chairman synthesis

One agent receives everything de-anonymized: framed question, all responses, all reviews. It produces:

**COUNCIL VERDICT** (in Arabic, English technical terms inline)

1. **Where the council agrees** — independent convergence = high-confidence signal.
2. **Where the council clashes** — real disagreements, both sides presented, with the engineering reason reasonable advisors diverge.
3. **Blind spots the council caught** — what only surfaced through peer review.
4. **Risk register** — the top 3 risks as a compact table: risk / severity / likelihood / cheapest mitigation or test.
5. **Verification ledger** — every volatile or external claim made during the session that must be verified before acting: prices, part availability and EOL status, software versions, certification requirements, benchmark numbers. If web search is available, verify the top items now and mark each ✅ verified / ⚠️ unverified with source. Never let an unverified number drive the recommendation silently.
6. **The recommendation** — a real answer with reasoning. Not "it depends." The chairman may side with a lone dissenter against the majority if the dissenter's reasoning is technically strongest.
7. **Tooling plan** — distilled from the Toolsmith's argument and the peer reviews of it, a compact table: project phase → tool/connector → exact artifact to produce, plus (when warranted) one line of retrospective critique (e.g. "this decision deserved an architecture diagram in X before being brought to the council") and an explicit "ignore for this project" list. If the Toolsmith concluded no tooling is needed, this section says so in one line — never pad it.
8. **The one thing to do first** — one concrete step: the test to run, the dev board to order, the prototype slice to build, or the diagram to draw. One, not ten. If the Toolsmith's retrospective critique was upheld in peer review, the first step is often producing the missing artifact.
9. **ADR draft (when applicable)** — if the decision is architectural, append a short ADR skeleton (Context / Decision / Consequences / Status: Proposed) so the verdict can be filed into the project's decision log.

**Chairman prompt template:**

```
You are the Chairman of an Engineering Council. Synthesize the work of
6 advisors and their peer reviews into a final verdict.

The question brought to the council:
---
[framed question]
---

ADVISOR RESPONSES:
**The Failure Analyst:** [response]
**The First-Principles Engineer:** [response]
**The Systems Architect:** [response]
**The Fresh-Eyes Reviewer:** [response]
**The Pragmatic Builder:** [response]
**The Toolsmith:** [response]

PEER REVIEWS:
[all 6 peer reviews]

Produce the council verdict using this exact structure:

## Where the Council Agrees
## Where the Council Clashes
## Blind Spots the Council Caught
## Risk Register
[top 3 risks: risk / severity / likelihood / cheapest mitigation or test]
## Verification Ledger
[volatile claims to verify; verify top items via web search if available,
mark each verified/unverified with source]
## The Recommendation
[a real answer with reasoning — not "it depends"; you may side with a
lone dissenter if their engineering reasoning is strongest]
## Tooling Plan
[phase → tool → artifact table; retrospective critique if upheld;
"ignore for this project" list; or one line "no tooling needed"]
## The One Thing to Do First
[one concrete step, not a list]
## ADR Draft
[only if the decision is architectural: Context / Decision /
Consequences / Status: Proposed]

Be direct. Don't hedge. The point of the council is clarity the user
could not get from a single perspective.
```

### step 5: generate the council report

Produce `council-report-[timestamp].html` — a single self-contained RTL HTML file, Arabic UI with English terms inline. Style it with the user's established design system:

* Palette: ink `#0b1f24` (text/headers), teal `#14868f` (primary accent), sage `#5fb0a8` (agree/verified), gold `#c9a227` (clash/warning); white background, subtle borders.
* Fonts: `IBM Plex Sans Arabic` or `Cairo` with system-ui fallback; English terms and code in an LTR `<span dir="ltr">` with a monospace stack.
* Structure: question at top → chairman verdict prominent → an agreement/clash visual (simple grid of the 6 advisors showing aligned vs diverging positions) → risk register table → tooling plan table (phase → tool → artifact) → verification ledger with status badges → collapsible `<details>` per advisor response → collapsible peer-review highlights → footer with timestamp.
* Keep it a scannable professional briefing, print-safe, nothing flashy.

### step 6: save the transcript

`council-transcript-[timestamp].md` (English): original question, framed question, domain lenses used, all advisor responses, all peer reviews with the anonymization map revealed, chairman synthesis, verification ledger results. This is the durable artifact for future sessions and ADR references.

In environments with a file-presentation tool, present both files to the user; otherwise place them in the working directory and state the paths.

---

## execution modes

**Claude Code / environments with sub-agents:** run steps 2 and 3 as parallel sub-agent batches exactly as described.

**Claude.ai / single-context environments:** sub-agents are unavailable, so simulate with discipline: generate the six advisor responses one at a time, each written strictly from its lens *before* moving on, with an explicit instruction to yourself not to harmonize them — preserve contradictions. The Toolsmith has a real advantage in this mode: the actual connector list is visible in-session, so its inventory is ground truth, not guesswork — but it must still verify a connector's *capabilities* before claiming them. Then perform the peer review honestly (the anonymization is procedural here, but still evaluate on merit and still hunt for wrong technical claims). Flag in the report footer that this session ran in simulated (single-context) mode. The structure still beats a single answer; isolation is weaker, so lean harder on the verification ledger.

---

## example: counciling a technical decision (condensed)

**User:** "council this: MQTT broker on the Pi vs direct USB-serial only for the kiosk relay control — offline-first system, ESP32-S3 actuator, single device today."

**Failure Analyst:** "A local broker adds a daemon that can die silently; relay commands stuck in QoS queues during reconnect storms are a safety hazard for an access-control actuator. USB-serial has its own failure: enumeration loss after EMI events or cable strain — without a watchdog handshake, a wedged port looks identical to 'door locked forever'..."

**First-Principles Engineer:** "One producer, one consumer, same enclosure, no fan-out requirement stated. Pub/sub is the wrong abstraction for a point-to-point command channel; you are buying topology flexibility you have not demonstrated needing. The real requirement is a framed, acknowledged, low-latency command link..."

**Systems Architect:** "Today it is one relay; the roadmap says fleet pilot. MQTT gives you a uniform control plane when device #2 arrives, plus telemetry and OTA hooks for free. Retrofitting that onto bespoke serial framing later is the expensive path..."

**Fresh-Eyes Reviewer:** "Nothing in front of me defines what happens when the link is down and someone needs the door open. Whichever transport wins, the failure-mode table for 'link dead' is missing, and that is the part a safety reviewer will ask about first..."

**Pragmatic Builder:** "RelayController over USB-serial already exists and is testable this week. A broker means packaging, persistence config, and startup ordering on the Pi. Ship serial now behind a small transport interface; the interface costs one afternoon and keeps the MQTT door open..."

**Toolsmith:** "This arrived as prose. The available diagramming connector should have produced a sequence diagram of the command/ack/timeout handshake first — half the council is arguing about a protocol nobody has drawn. Forward: sequence diagram → [diagramming tool]; fleet-phase topology sketch → [whiteboard tool]; ignore presentation and email connectors for this decision..."

**Chairman (condensed):** Agreement: point-to-point today, transport must be abstracted, link-down behavior is undefined and blocking. Clash: Architect's fleet-readiness vs Builder/First-Principles simplicity — resolved by the abstraction layer, which defers the broker without foreclosing it. Caught in review: no advisor priced the broker's RAM/CPU on a Pi already running inference. Recommendation: USB-serial now behind a transport interface; adopt MQTT only at the fleet pilot gate. Tooling plan: draw the handshake sequence diagram before writing the interface. One thing first: define and diagram the link-down safe-state.

---

## important notes

* **Parallel spawning always, where available.** Sequential advisors contaminate each other.
* **Anonymize for peer review.** Reviewers must judge arguments, not personas.
* **Accuracy outranks rhetoric.** A confident wrong spec is worse than a hedged right one — reviewers and chairman both police this.
* **The chairman may dissent from the majority** when the minority's engineering reasoning is stronger.
* **Never council trivia.** One-right-answer questions get a direct answer.
* **Volatile facts get verified, not trusted** — prices, stock, versions, compliance rules change; the verification ledger exists because fabricated or stale "facts" in AI output are a documented, recurring hazard.
* **The Toolsmith critiques workflow, never pads it.** Tool claims are claims: a connector capability asserted without basis goes into the verification ledger like any spec. "No tool needed" is a valid and respected Toolsmith verdict.
* **The tool inventory is session-truth.** Build it from what is actually connected in the current environment, never from memory of what the user "usually has."
* **Respect prior ADRs.** Overturning a recorded decision is allowed but must be explicit and justified.

---

## Appendix: domain lenses

Per-domain checklist items. During step 0, read the subsection(s) matching the detected domain and inject the relevant items into the framed question. Advisors use them as prompts for specificity — not as a form to fill. A question may span multiple domains (e.g. a kiosk decision touches embedded-firmware + android-hmi + sourcing).

---

### embedded-firmware

* Concurrency model: RTOS tasks vs superloop vs event-driven; priority inversion, ISR latency budgets
* Memory: flash/RAM headroom, fragmentation, flash wear leveling for frequent writes
* Power: sleep states, brownout behavior, watchdog strategy, safe-state on reset
* Connectivity: protocol fit (MQTT vs HTTP vs raw serial vs CAN/Modbus), QoS, offline-first buffering, reconnection storms
* OTA: update mechanism, rollback, bricking risk, signing
* Toolchain lock-in: vendor HAL vs portable layer; ESP-IDF/STM32 HAL/Arduino tradeoffs
* Debuggability in the field: logging, fault dumps, remote diagnostics
* Timing: clock domains, drift, RTC backup, NTP dependence in offline systems

### pcb-hardware

* EMC/EMI: return paths, loop areas, clock routing, shielding, pre-compliance plan
* Power integrity: decoupling scheme, plane strategy, inrush, reverse-polarity/ESD protection
* Stackup & DFM: layer count justification, fab capabilities (JLCPCB/PCBWay class), panelization, testpoints
* Thermal: dissipation paths, copper pours, derating at ambient extremes (50°C+ outdoor matters in the Gulf)
* Component risk: lifecycle/EOL status, second sources, LCSC/Mouser/DigiKey availability, MOQ
* Connectors & mechanicals: mating cycles, strain relief, enclosure fit, ingress protection
* Bring-up plan: what gets probed first, current-limited first power-on, JTAG/SWD access
* Certification exposure: CE/FCC pathways if productized

### software-web

* State & data flow: source of truth, caching/invalidation, offline behavior
* API design: versioning, error contracts, idempotency, rate limits
* Security: authn/authz model, XSS/CSRF/injection surface, secrets handling, supply-chain (lockfiles, audit)
* Performance: bundle size, render path, N+1 queries, realistic load assumptions
* Maintainability: framework longevity, team familiarity, dependency count vs vendoring
* Browser-extension specifics: Manifest V3 limits, permissions minimization, store review risk
* Accessibility & i18n: RTL correctness, bidi text, keyboard paths

### android-hmi

* Compose: recomposition cost, stable parameters, unbounded layouts (LazyGrid/Column traps)
* Lifecycle: process death, configuration changes, foreground-service rules, doze/battery optimizations on always-on devices
* Kiosk constraints: lock task mode, boot-to-app, watchdog/auto-restart, screen burn-in
* Hardware bridges: USB-serial permission flows, device_filter, reconnection handling, threading around blocking I/O
* GMS decision: certified GMS vs AOSP-only; licensing route (e.g. emteria-class commercial Android) and what each forecloses
* Update path: app updates without Play Store, A/B OS updates, fleet management
* Input & display: touch in industrial gloves, brightness/readability, screen timeout policy

### ai-ml

* Placement: on-device vs edge accelerator vs cloud — latency, privacy, cost-per-inference, offline requirement
* Model risk: hallucination surface, failure modes under distribution shift, adversarial inputs (spoofing for face recognition)
* Evaluation: what is the eval set, baseline, and acceptance threshold *before* building; FAR/FRR targets for biometrics
* Resource budget: RAM/NPU/quantization constraints, thermal throttling on SBCs
* Data: collection consent, retention, locality/regulatory constraints
* Lifecycle: model EOL, retraining cadence, version pinning, reproducibility
* Vendor lock-in: API dependence, pricing volatility (verify current pricing — it changes monthly)

### research-innovation

* Novelty: what exactly is new — mechanism, integration, application, or cost point?
* Prior art: patent landscape and freedom-to-operate; existing literature and commercial products (WIPO/Espacenet/Google Patents search before claiming novelty)
* IP strategy: patent vs trade secret vs publish; disclosure timing
* Feasibility: TRL honestly assessed; the single riskiest assumption and the cheapest experiment that tests it
* Differentiation: why incumbents haven't done it (real barrier vs no demand)
* Path to impact: pilot partner, funding fit, standards/regulatory gates
* Kill criteria: what result would justify stopping

### sourcing-procurement

* Volatility rule: ALL prices, stock levels, lead times, and version numbers are unverified until checked against live sources — put them in the verification ledger
* Total cost: unit price + shipping + customs + MOQ waste + tooling/NRE
* Logistics: regional availability (Gulf/Qatar: local distributors vs AliExpress/LCSC/Mouser routes, customs lead time), Incoterms
* Vendor risk: single-source exposure, clone/counterfeit risk on marketplaces, warranty reality
* Lifecycle: EOL announcements, last-time-buy, drop-in alternatives identified up front
* Support: documentation quality, SDK maintenance activity, community size, escalation path
* Compliance: import restrictions, export-control flags (ECCN-class concerns for compute/RF parts), local certification (radio/Type Approval)

---

### tooling-workflow (Toolsmith only)

Heuristics for mapping decision/project phases to tool *categories* — the Toolsmith resolves each category against the live inventory from step 1-A2, never against a hardcoded tool list:

* Architecture & data-flow decisions → diagramming connector (flowchart/sequence/ERD class tools) — the artifact is a diagram the council can actually critique
* UI/UX decisions → design or screen-generation connector — the artifact is a mock, not a paragraph describing one
* Brainstorming, option mapping, tradeoff matrices → whiteboard/canvas connector — the artifact is a comparison board
* Stakeholder communication of the verdict → presentation/document connector — only if a stakeholder audience actually exists
* Scheduling pilots, reviews, deadlines → calendar/email connectors — only with a concrete date attached
* Research-heavy questions → web search + document storage — the artifact is a sourced comparison, not recall
* Code/firmware decisions → code execution for a measured micro-benchmark beats speculation about performance

Red flags the Toolsmith should call out:

* A decision about visual/spatial structure (architecture, layout, flow) argued entirely in prose
* "We compared the options" with no comparison artifact anyone can inspect
* The same diagram rebuilt in 3 different tools (tool-shopping as procrastination)
* A connector connected months ago and never producing an artifact for this project (candidate for the ignore-list, not for forced usage)
* Performance/cost claims that code execution or web search could settle in minutes

---

### using the lenses well

Inject only the items plausibly relevant to the actual question — a lens is a memory aid against blind spots, not a questionnaire. If an advisor's strongest point lies outside the checklist, that is fine; the checklist sets a floor, not a ceiling.
