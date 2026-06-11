# Engineering Council — A Claude Skill

Stop trusting Claude's first answer on engineering decisions. Run any technical question through **6 AI advisors** who argue from deliberately conflicting engineering lenses, peer-review each other anonymously, and hand you a verdict with a **risk register**, a **verification ledger**, and a **tooling plan**.

An engineering-focused redesign of [Andrej Karpathy's LLM Council](https://x.com/karpathy/status/1962263486196867115) methodology — rebuilt for embedded systems, PCB/circuit design, software, Android HMI, AI/ML, research & innovation, and component sourcing.

---

## The Problem

Ask one AI "should I use MQTT or USB-serial?" and the answer bends toward whatever framing you used. That's fine for emails. It's dangerous when being wrong costs a board respin, weeks of firmware rework, or a dead-end architecture.

## How It Works

Say **"council this"** (or «استشر المجلس») and the skill:

1. **Detects the domain** (embedded-firmware, pcb-hardware, software-web, android-hmi, ai-ml, research-innovation, sourcing) and injects a matching checklist lens
2. 2. **Scans context** — CLAUDE.md, memory files, ADRs, attached datasheets/schematics/code — and takes a live **tool inventory** of your connected tools
   3. 3. **Convenes 6 advisors in parallel:**
      4.    - **The Failure Analyst** — FMEA mindset; hunts the failure mode everyone ignores
            -    - **The First-Principles Engineer** — strips the question to physics, datasheets, and real requirements
                 -    - **The Systems Architect** — weighs leverage, upgrade paths, what this forecloses in 12–24 months
                      -    - **The Fresh-Eyes Reviewer** — zero context; catches the curse of knowledge
                           -    - **The Pragmatic Builder** — BOM, lead times, what gets soldered/flashed Monday morning
                                -    - **The Toolsmith** — audits the *workflow*: which available tool/connector should have produced evidence before this decision, which tools carry the next phases, and which are dead weight
                                     - 4. **Anonymous peer review** (A–F, randomized) — accuracy outranks eloquence; dubious technical claims get flagged
                                       5. 5. **Chairman synthesis** — agreement, clashes, blind spots, risk register, verification ledger (volatile facts like prices/EOL/versions get web-verified, never trusted), recommendation, tooling plan, one first step, and an ADR draft when the decision is architectural
                                          6. 6. **Outputs** a visual HTML report + full markdown transcript
                                            
                                             7. ## Install
                                            
                                             8. ### Claude Code
                                             9. ```bash
                                                git clone https://github.com/Haddad974/engineering-council-skill ~/.claude/skills/engineering-council
                                                ```
                                                Or manually: create `~/.claude/skills/engineering-council/` and drop `SKILL.md` inside.

                                                ### Claude.ai / Claude apps
                                                Settings → Capabilities → Skills → Upload, then upload the repo zip (or a zip containing `engineering-council/SKILL.md`).

                                                > Migrating from the original `llm-council` skill? Delete it first — both respond to the same triggers.
                                                >
                                                > ## Use
                                                >
                                                > Triggers: `council this` · `run the council` · `pressure-test this` · `stress-test this` · `war room this` · `debate this` · «استشر المجلس» · «قيّم القرار»
                                                >
                                                > **Example:**
                                                > > council this: MQTT broker on the Pi vs direct USB-serial for kiosk relay control — offline-first, ESP32-S3 actuator, single device today but fleet pilot on the roadmap.
                                                > >
                                                > > ## Good Council Questions
                                                > >
                                                > > - "Pi 5 + commercial Android vs RK3588 industrial board — final call?"
                                                > > - - "On-device face recognition vs edge accelerator?"
                                                > >   - - "Is this 4-layer stackup with this decoupling scheme ready for fab?"
                                                > >     - - "Is this research direction novel enough to patent?"
                                                > >       - - "Buy the off-the-shelf HMI module or build on our own board?"
                                                > >        
                                                > >         - **Skip the council for:** factual lookups, syntax questions, pure implementation tasks, debugging, validation-seeking. The council tells you things you don't want to hear — that's the feature.
                                                > >        
                                                > >         - ## What's Different From the Original
                                                > >        
                                                > >         - | | Original llm-council | Engineering Council |
                                                > > |---|---|---|
                                                > > | Advisors | 5, business-flavored | 6, engineering lenses + **Toolsmith** |
                                                > > | Domain awareness | — | 8 injectable domain-lens checklists |
                                                > > | Fact discipline | — | Verification ledger: volatile claims web-verified |
                                                > > | Risk | prose | Risk register (severity × likelihood × cheapest test) |
                                                > > | Decisions log | — | ADR draft for architectural calls |
                                                > > | Workflow audit | — | Tooling plan: phase → tool → artifact, + ignore-list |
                                                > > | Environments | Claude Code only | Claude Code (parallel sub-agents) + Claude.ai (disciplined simulated mode) |
                                                > >
                                                > > ## Credit
                                                > >
                                                > > - Methodology: [Andrej Karpathy's LLM Council](https://x.com/karpathy/status/1962263486196867115)
                                                > > - - Claude Code sub-agent adaptation: [@olelehmann](https://x.com/olelehmann)
                                                > >   - - Original installable skill: [tenfoldmarc/llm-council-skill](https://github.com/tenfoldmarc/llm-council-skill)
                                                > >     - - Engineering Council redesign: Abdulaziz AL-Haddad
                                                > >      
                                                > >       - ## License
                                                > >      
                                                > >       - MIT — see [LICENSE](LICENSE).
                                                > >       - 
