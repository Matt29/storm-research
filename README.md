# Storm Recherche

A self-contained Claude Code skill that turns one topic into a verified, multi-perspective HTML research briefing. It simulates five expert lenses (practitioner, academic, skeptic, economist, historian), maps where they contradict each other, synthesizes everything into a single HTML report, then adversarially peer-reviews its own output and checks every citation against its primary source before delivering.

It is a hand-built adaptation of Stanford's STORM method, wrapped as a drop-in skill with an honest-broker verification pass baked into the pipeline.

## What you get

- Five parallel research agents, one per expert lens, each doing real web research with cited sources.
- A contradiction map that surfaces where the lenses disagree, what they all agree on, and the blind spot none of them covered.
- One self-contained HTML report: a 60-second summary, five findings ranked by reliability, a hidden connection, the missing sixth lens, concrete actions, a claim-safety guide (assert / caveat / avoid), and a frontier question.
- A mandatory verification phase: every citation is checked against its primary source, and the report header shows a truthful count of how many claims were fabricated, corrected, or demoted.

## Requirements

- Claude Code. The skill uses only built-in tools: the `Agent` tool with the built-in `general-purpose` agent, `Write`, and web search/fetch inside those agents.
- No external scripts, APIs, or paid services.

## Install

Copy the `storm-recherche/` folder into your `~/.claude/skills/` directory:

```
~/.claude/skills/
└── storm-recherche/
    ├── SKILL.md
    └── report-template.html
```

Keep both names exactly as they are. The folder name must match the skill's `name:` field, and the skill loads `report-template.html` by that exact filename.

## Usage

In Claude Code:

```
run a storm recherche on [your topic]
```

No slash command needed, the skill triggers on its own. The output lands in `storm-reports/` as an HTML file. It also works under Codex or any other agent that reads a skills folder.

### Quick install check

Run it once on a topic you know well, then verify two things on the final report:

- The header banner shows a real count (`X fabricated, Y corrected, Z demoted`). If it is empty or generic, the verification phase was skipped.
- Each finding carries a 1 to 10 reliability score and "supported by / challenged by" chips.

If either is missing, it is not a real Storm Recherche run.

## How it works

- Phase 0, scope the topic and the reader's role.
- Phase 1, five expert lenses run in parallel, each doing real web research.
- Phase 2, map the contradictions: direct conflicts, strongest vs weakest evidence, universal agreement, and the blind spot.
- Phase 3, synthesize the HTML report from the template.
- Phase 4, adversarial self-review plus primary-source verification of every citation, then apply corrections.

A run spawns roughly 9 to 11 agents. That is expected, and it is the point.

## Honest broker by design

Three guardrails are non-negotiable, and they are stated inside every report:

- The panel is hand-built. The five lenses share one framing, so agreement across them is a strong hypothesis, not independent proof, and never a consensus of the field.
- Reliability scores evidence quality, not confidence. A high score means the data is solid, not that the strategic reading is certain.
- Verification is mandatory. Every citation is traced to a primary source. Nothing is invented. Unverifiable figures are demoted or cut, never papered over.

## Output language

The skill produces French briefings by default, since it was built for the IA IRL audience. To switch the output to English, edit the "Langue et style" section and the five lens prompts in `SKILL.md`. The pipeline itself is language-agnostic.

## Credits and prior work

STORM is a real research method from the Stanford OVAL Lab: Synthesis of Topic Outlines through Retrieval and Multi-perspective Question Asking (NAACL 2024).

- Paper: https://arxiv.org/abs/2402.14207
- Stanford code: https://github.com/stanford-oval/storm
- Live demo: https://storm.genie.stanford.edu

One honest note: the original STORM discovers its perspectives dynamically by surveying related articles. This skill instead fixes five lenses by hand. That is an effective shortcut popularized in the creator community, not the original method. The report says so explicitly, so the two are never passed off as the same thing.

## License

MIT, see [LICENSE](LICENSE).

Built by Matthieu Caillaud, IA IRL, beacons.ai/ia.irl
