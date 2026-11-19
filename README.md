# storyscope-rules

An [Agent Skill](https://agentskills.io/specification) that changes **what an agent decides to write and in what order**, before it writes a word. It works in Claude Code, OpenAI Codex, Gemini CLI, Cursor, GitHub Copilot, OpenCode and any other client that reads `SKILL.md` folders.

The rules come from the StoryScope study (Russell et al., COLM 2026; [paper](https://jenna-russell.github.io/assets/pdf/storyscope.pdf), [arXiv:2604.03136](https://arxiv.org/abs/2604.03136)). Its main finding: narrative structure alone separates human from AI fiction at 93.2 macro-F1. Rewriting AI stories to strip clichés and purple prose lowered detection by only 1.6 points (95.5 → 93.9).

So rewording a draft does not help. The skill works on the outline.

> This is an independent, unofficial adaptation. It is not affiliated with or endorsed by the StoryScope authors.

## Contents

- [Install in Claude Code (plugin)](#install-in-claude-code-plugin)
- [Using the plugin](#using-the-plugin)
- [Install in other clients](#install-in-other-clients)
- [What it does](#what-it-does)
- [What it does not do](#what-it-does-not-do)
- [Documentation](#documentation)

## Install in Claude Code (plugin)

Inside a Claude Code session:

```
/plugin marketplace add RustLangLatam/storyscope-rules
/plugin install storyscope-rules@storyscope-rules
```

Or from a terminal:

```bash
claude plugin marketplace add RustLangLatam/storyscope-rules
claude plugin install storyscope-rules@storyscope-rules
```

Start a new session afterwards so the skill is loaded. Pass `--scope project` to `install` to enable it only for the current repository (the setting is written to `.claude/settings.json`, so collaborators get it too).

**Without the plugin system**, copy the folder instead:

```bash
git clone https://github.com/RustLangLatam/storyscope-rules.git
cp -r storyscope-rules/skills/storyscope-rules ~/.claude/skills/
```

## Using the plugin

### Automatic use (the default)

You do not need to call it. The skill's description tells the agent to load it **before any task that produces text for people**: reports, proposals, emails, documentation, PR descriptions, interface copy, fiction, long messages. It also loads when you ask for text that "doesn't sound like AI" or has "less filler". In coding tasks it only affects comments, commit messages, PR text and the final summary. The code itself is left alone.

### Explicit use

Call it by name when you want to be sure it runs:

| Installed as | Command |
|---|---|
| Plugin | `/storyscope-rules:storyscope-rules` |
| Plain skill (`~/.claude/skills/`) | `/storyscope-rules` |

Examples:

```
/storyscope-rules:storyscope-rules write the release announcement for v2.3 from CHANGELOG.md
/storyscope-rules:storyscope-rules review this proposal draft and restructure what fails the audit
```

### What you will see

Before drafting, the agent outlines in its reasoning what will be told, in what order and what stays open. After drafting it runs the final audit. If it breaks a rule because you asked for something specific or because the truth required it, it says so in one line. Expect result-first openings, no closing "lesson" paragraph, named sources instead of vague allusions, and explicit costs and open issues.

### Managing it

| Task | Command |
|---|---|
| Check that it is installed | `claude plugin list` |
| Update to the latest version | `claude plugin marketplace update storyscope-rules` then `claude plugin update storyscope-rules@storyscope-rules` |
| Turn off temporarily | `claude plugin disable storyscope-rules@storyscope-rules` |
| Turn back on | `claude plugin enable storyscope-rules@storyscope-rules` |
| Uninstall | `claude plugin uninstall storyscope-rules@storyscope-rules` |

Inside a session, `/plugin` opens the same options interactively.

## Install in other clients

The skill is one standard folder, `skills/storyscope-rules/`, with only `name`, `description` and `license` in its frontmatter. It uses no Claude-specific variables or tools, so the same file works everywhere. The plugin manifests in `.claude-plugin/` are only read by Claude Code; other clients ignore them.

**One copy covers most clients.** Codex, Gemini CLI, Cursor and OpenCode all read `~/.agents/skills/`:

```bash
git clone https://github.com/RustLangLatam/storyscope-rules.git
mkdir -p ~/.agents/skills
cp -r storyscope-rules/skills/storyscope-rules ~/.agents/skills/
```

To install for a single repository instead, copy it to `<repo>/.agents/skills/`. To receive updates with `git pull`, use `ln -s "$(pwd)/storyscope-rules/skills/storyscope-rules" ~/.agents/skills/` instead of `cp`.

| Client | User directory | Project directory | How it is invoked |
|---|---|---|---|
| **Claude Code** | `~/.claude/skills/` (or the plugin) | `.claude/skills/` | Automatic, or `/storyscope-rules` |
| **OpenAI Codex** | `~/.agents/skills/` | `.agents/skills/` | Automatic, or `$storyscope-rules`; `/skills` lists them |
| **Gemini CLI** | `~/.gemini/skills/` or `~/.agents/skills/` | `.gemini/skills/` or `.agents/skills/` | Automatic (asks to activate); `/skills list` to check |
| **Cursor** | `~/.cursor/skills/` or `~/.agents/skills/` (also reads `~/.claude/skills/`) | `.cursor/skills/` or `.agents/skills/` | Automatic, or `/storyscope-rules` in Agent chat |
| **GitHub Copilot** (VS Code) | `~/.copilot/skills/` | `.github/skills/` | Automatic, or `/storyscope-rules` |
| **OpenCode** | `~/.config/opencode/skills/` or `~/.agents/skills/` (also reads `~/.claude/skills/`) | `.opencode/skills/` or `.agents/skills/` | Automatic, or `@storyscope-rules` |

Client-specific notes:

- **Gemini CLI** can install straight from GitHub:
  ```bash
  gemini skills install https://github.com/RustLangLatam/storyscope-rules.git --path skills/storyscope-rules
  ```
  Google's documentation states that Gemini CLI was succeeded by Antigravity CLI in June 2026; check that client's docs for its current path.
- **GitHub Copilot** does not read `~/.agents/skills/` in its own harness. Copy the folder to `~/.copilot/skills/` or `.github/skills/`.
- **Cursor and OpenCode** already read `~/.claude/skills/`. If you installed the plain skill for Claude Code there, they pick it up with no extra step. The Claude Code *plugin* install lives elsewhere, so those clients do not see it.
- **Several clients on one machine**: install once in `~/.agents/skills/`, and add a copy in `~/.claude/skills/` (or the plugin) for Claude Code and in `~/.copilot/skills/` for Copilot. Avoid having two copies visible to the same client, or it may list the skill twice.

**On non-Claude models**, group E of the rules (Claude's own writing habits) is marked optional, and the skill gives the habits the study measured for GPT, Gemini, DeepSeek and Kimi instead. All other rules apply to every model.

These paths are taken from each client's documentation as of November 2026 ([Agent Skills spec](https://agentskills.io/specification), [Codex](https://developers.openai.com/codex/skills), [Gemini CLI](https://geminicli.com/docs/cli/skills/), [Cursor](https://cursor.com/docs/context/skills), [VS Code Copilot](https://code.visualstudio.com/docs/copilot/customization/agent-skills), [OpenCode](https://opencode.ai/docs/skills)). Clients change quickly. If a path stops working, check the linked page and open an issue.

## What it does

1. **Classifies** the task as narrative, non-narrative or code-only.
2. **Outlines in reasoning** what will be told, in what order, what stays unresolved and which real references it carries.
3. **Checks the outline** against 31 rules in six groups:

   | Group | Targets | Example |
   |---|---|---|
   | A. Meaning | Explaining the moral | End on the decision, not the lesson |
   | B. Body & atmosphere | Embodied emotion, mood-weather | Say "she was afraid" (29% human vs. 8% AI) |
   | C. Pipeline | Single tidy causal line | Include what was tried and failed |
   | D. Absent moves | Techniques AI avoids | Open with the result, then go back |
   | E. Model habits | Claude's fingerprints (others listed) | Do not keep intensity flat |
   | F. Rarity | The first idea every model has | Discard it if another serves equally |

4. **Drafts**, then runs an 8-question **audit**. Any "yes" sends it back to restructure. Word swaps don't count.

**Truth comes first.** The skill never invents a reference, figure or anecdote to satisfy a rule. An explicit user request or a mandatory format (contracts, specs, forms) overrides it, and the agent says so in one line when that happens.

## What it does not do

- It does **not** make text undetectable. Surface-level detectors score 99.7–99.9 on the same corpus, and this skill doesn't touch that layer.
- The evidence is **English fiction**. Transfer to reports, emails or other languages is a documented hypothesis, not a result.
- It is **not** for hiding AI authorship where disclosure is required.

## Documentation

- [`docs/TECHNICAL_REPORT.md`](docs/TECHNICAL_REPORT.md): source study, how features became rules, register transfer, execution protocol, portability, threats to validity, and a rule–evidence index.
- [`skills/storyscope-rules/SKILL.md`](skills/storyscope-rules/SKILL.md): the skill itself.
- [`CHANGELOG.md`](CHANGELOG.md): release history and corrections.

## Repository layout

```
.claude-plugin/
  plugin.json          Claude Code plugin manifest
  marketplace.json     single-plugin marketplace (enables /plugin marketplace add)
skills/storyscope-rules/
  SKILL.md             the skill (Agent Skills standard, client-neutral)
docs/
  TECHNICAL_REPORT.md
CITATION.cff           citation metadata (GitHub "Cite this repository")
CHANGELOG.md
LICENSE
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
