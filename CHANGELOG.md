# Changelog

## 1.2.0 — 2026-11-20

Fidelity audit: every claim was re-checked against the paper's full text. No numeric errors were found. These statements went beyond the evidence and were corrected:

- Rule 25 assumed Claude has *low* event-type diversity; the paper lists the feature without a direction. The rule now asks the agent to check variety.
- Rule 28 said "each person should sound different"; the paper only says Claude's narrative voice is the most uniform.
- Group E now states that its traits compare Claude with other models, with no human baseline.
- The editing-robustness result is described as one test on 278 Gemini stories, and "cannot remove the signal" became "did not remove it in that test". The pre-writing principle is labeled as the skill's inference, supported by the paper's remark that narrative features need structural rewrites (citing Namuduri et al., 2025).
- Rule 6 "largest gap" is now "largest percentage-point gap"; rule 10 refers to depth of interior access, not breadth; rule 31 no longer claims every model would write the same structure.
- Added the LLM-annotation limit to the skill; corpus length stated as a 4,753-word mean; Table 16's 33 rows vs. 30 reported features explained; reader-address units footnoted.
- Report rewritten to lead with results, report the draft's errors, and state the skill's cost and over-application risk.

## 1.1.0 — 2026-11-20

- Documented installation and use outside Claude Code: OpenAI Codex, Gemini CLI, Cursor, GitHub Copilot and OpenCode, with per-client paths and invocation.
- README: full plugin usage guide (automatic and explicit invocation, update, disable, uninstall, project scope).
- Skill: added the `license` frontmatter field from the Agent Skills specification. When the skill runs on a non-Claude model, group E is marked optional and the study's fingerprints for GPT, Gemini, DeepSeek and Kimi are given instead.
- Technical report: new §4.2 Portability.

## 1.0.0 — 2026-11-20

First public release.

- English skill `storyscope-rules`: 31 pre-writing rules, precedence of truth and user request, final audit.
- Claude Code plugin and single-plugin marketplace manifests.
- Technical report with a rule-by-rule evidence index checked against the camera-ready paper (COLM 2026).
- Correction compared with the earlier Spanish draft: the post-editing figure (93.9 macro-F1) is measured on 278 LAMP-edited Gemini stories whose unedited score was 95.5. It is not a re-test of the 93.2 headline result.
