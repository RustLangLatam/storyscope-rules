# storyscope-rules

A Claude Code skill that changes **what an agent decides to write and in what order**, before it writes a word. The rules come from the StoryScope study (Russell et al., COLM 2026; [paper](https://jenna-russell.github.io/assets/pdf/storyscope.pdf), [arXiv:2604.03136](https://arxiv.org/abs/2604.03136)). Its main finding: narrative structure alone separates human from AI fiction at 93.2 macro-F1. Rewriting AI stories to strip clichés and purple prose lowered detection by only 1.6 points (95.5 → 93.9).

So rewording a draft does not help. The skill works on the outline.

> This is an independent, unofficial adaptation. It is not affiliated with or endorsed by the StoryScope authors.

## Install

### As a plugin (recommended: one command, gets updates)

Inside Claude Code:

```
/plugin marketplace add RustLangLatam/storyscope-rules
/plugin install storyscope-rules@storyscope-rules
```

The skill is then available as `/storyscope-rules:storyscope-rules`, and the agent loads it on its own when a task produces text for people.

### As a plain skill (no plugin system)

```bash
git clone https://github.com/RustLangLatam/storyscope-rules.git
mkdir -p ~/.claude/skills
cp -r storyscope-rules/skills/storyscope-rules ~/.claude/skills/
```

It is then available as `/storyscope-rules`. For a single project, copy it into `<project>/.claude/skills/` instead.

### Uninstall

`/plugin uninstall storyscope-rules`, or `rm -r ~/.claude/skills/storyscope-rules`.

## What it does

1. **Classifies** the task as narrative, non-narrative or code-only (code-only: rules apply only to comments, commits, PR text and the final report).
2. **Outlines in reasoning** what will be told, in what order, what stays unresolved and which real references it carries.
3. **Checks the outline** against 31 rules in six groups:

   | Group | Targets | Example |
   |---|---|---|
   | A. Meaning | Explaining the moral | End on the decision, not the lesson |
   | B. Body & atmosphere | Somatic emotion, mood-weather | Say "she was afraid" (29% human vs. 8% AI) |
   | C. Pipeline | Single tidy causal line | Include what was tried and failed |
   | D. Absent moves | Techniques AI avoids | Open with the result, then go back |
   | E. Claude's own habits | Model fingerprints | Do not keep intensity flat |
   | F. Rarity | The first idea every model has | Discard it if another serves equally |

4. **Drafts**, then runs an 8-question **audit**. Any "yes" sends it back to restructure. Word swaps don't count.

**Truth comes first.** The skill never invents a reference, figure or anecdote to satisfy a rule. An explicit user request or a mandatory format (contracts, specs, forms) overrides it, and the agent says so in one line when that happens.

## What it does not do

- It does **not** make text undetectable. Surface-level detectors score 99.7–99.9 on the same corpus, and this skill doesn't touch that layer.
- The evidence is **English fiction**. Transfer to reports, emails or other languages is a documented hypothesis, not a result.
- It is **not** for hiding AI authorship where disclosure is required.

## Documentation

- [`docs/TECHNICAL_REPORT.md`](docs/TECHNICAL_REPORT.md): source study, how features became rules, register transfer, execution protocol, threats to validity, and a rule–evidence index.
- [`skills/storyscope-rules/SKILL.md`](skills/storyscope-rules/SKILL.md): the skill itself.
- [`CHANGELOG.md`](CHANGELOG.md): release history and corrections.

## Repository layout

```
.claude-plugin/
  plugin.json          plugin manifest
  marketplace.json     single-plugin marketplace (enables /plugin marketplace add)
skills/storyscope-rules/
  SKILL.md             the skill
docs/
  TECHNICAL_REPORT.md
```

## Citation

If you use this in research, cite the source study:

```bibtex
@inproceedings{russell2026storyscope,
  title         = {StoryScope: Investigating idiosyncrasies in AI fiction},
  author        = {Russell, Jenna and Rajendhran, Rishanth and Pham, Chau Minh and Iyyer, Mohit and Wieting, John},
  booktitle     = {Conference on Language Modeling (COLM)},
  year          = {2026},
  eprint        = {2604.03136},
  archivePrefix = {arXiv}
}
```

## License

MIT for this skill and its documentation. The StoryScope paper and data belong to their authors.
