# Changelog

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
