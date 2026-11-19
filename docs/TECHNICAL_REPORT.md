# StoryScope Rules: Operationalizing Narrative-Level Discriminators of AI-Generated Text as Pre-Writing Constraints for LLM Agents

**Technical Report — v1.0.0 — November 2026**

*Independent, unofficial work. Not affiliated with or endorsed by the authors of the source study.*

---

## Abstract

Detectors of machine-generated text usually exploit surface statistics: token distributions, function-word frequencies and n-gram profiles. Russell et al. (COLM 2026) showed that a second, deeper signal exists. Using only *narrative* features, with style-related features excluded, an XGBoost classifier separated human-written from AI-written fiction at 93.2 macro-F1 over 61,608 stories. Stylistic editing of AI stories reduced detection by only 1.6 points (95.5 → 93.9 macro-F1). This report describes **StoryScope Rules**, an agent skill for Claude Code that turns the study's reported human–AI differences into 31 constraints applied *before* drafting, plus an audit applied after drafting. We document the source evidence, the mapping from each published feature to each rule, the extension to non-narrative registers, how the skill runs inside the agent, and the main threats to validity: genre transfer, the statistical rather than individual nature of the effects, and the absence of any claim about detector evasion.

**Keywords:** AI-generated text, narrative structure, LLM idiosyncrasies, writing constraints, agent skills, Claude Code.

---

## 1. Introduction

Most attempts to make LLM output read as human-written act on the surface: synonym substitution, sentence-length variation, removal of stock phrases. StoryScope suggests these interventions target the wrong layer. Narrative decisions alone (what is told, in what order, how conflicts resolve, how emotion is conveyed) separate human from AI text at over 93 macro-F1. When AI stories were edited to remove clichés, redundant exposition and purple prose, the narrative classifier still detected them almost as well. Rewording after drafting therefore cannot remove the signal. The text has to be planned differently.

This motivates the central design principle of the skill:

> **P1 (Pre-writing intervention).** Constraints are applied to the outline before any prose is produced. Post-hoc edits that change wording but not structure do not count as compliance.

The contribution of this work is engineering, not empirical. We run no new experiments. We translate published effect directions into an executable procedure for an LLM agent and document every translation step (Appendix A) so each one can be inspected.

## 2. Source Study

All figures in this section come from Russell et al. (2026), camera-ready version (arXiv v6, 10 Aug 2026).

### 2.1 Corpus

| Property | Value |
|---|---|
| Writing prompts | 10,272, reverse-engineered from human stories with Gemini 2.5 Flash |
| Human source | Books3 (Presser, 2020) |
| AI sources | Claude Sonnet 4.6, DeepSeek V3.2, Gemini 3 Flash, GPT-5.4, Kimi K2.5 |
| Design | Parallel: 1 human + 5 AI stories per prompt |
| Total stories | 61,608 (24 AI generations refused) |
| Length | ~5,000 words nominal |
| Language / genre | English fiction |

The parallel design controls for prompt content. Differences between the human and AI stories for the same prompt cannot come from topic.

### 2.2 Feature Pipeline

1. **Structuring.** GPT-5.1 converts each story into a JSON template over ten narrative dimensions (e.g., plot, agents, temporal structure).
2. **Discovery.** GPT-5.1 compares stories across sources in a 600-story pool and proposes candidate features. Three runs produced 408 candidates, which embedding-based clustering reduced to **304** features (categorical, ordinal, Likert scale, binary, multi-select).
3. **Annotation.** Gemini 3 Flash labels every story on every feature. Reported reliability: Krippendorff's α = 0.90 across five runs; mean Cohen's κ = 0.84 against human annotators.
4. **Style exclusion.** An audit flags 47 style-related features. The remaining **257** form the *Narrative-only* set.
5. **Core and fingerprint features.** SHAP analysis with bootstrapping selects **30 core features** that separate human from AI writing overall (the paper's Table 16), and per-source **fingerprint** features that distinguish each source from the others (Table 17).

### 2.3 Key Results

| Condition | Task | Macro-F1 |
|---|---|---|
| Narrative-only (257 features) | Human vs. AI | **93.2** |
| Core features only (30) | Human vs. AI | 84.8 |
| Narrative + Style (304) | Human vs. AI | 96.0 |
| Narrative-only | 6-way source attribution | 68.4 |
| Narrative + Style | 6-way source attribution | 77.3 |
| Strongest supervised text baselines (TF-IDF, stylometric, ModernBERT) | Human vs. AI | 99.7–99.9 |
| Binoculars (zero-shot) | Human vs. AI | 55.9 |

**Robustness to stylistic editing.** The authors rewrote 278 Gemini stories with the LAMP span-level framework (Chakrabarty et al., 2025), which removes seven categories of AI writing artifacts such as cliché, redundant exposition and purple prose. The narrative classifier scored 93.9 macro-F1 on the edited stories against 95.5 on the same stories unedited, a drop of 1.6 points.

**Rarity.** Per-story rarity is the mean Euclidean distance to the story's 25 nearest neighbors in feature space. Human stories have a higher mean rarity percentile than AI stories (0.71 vs. 0.49; Cohen's d = 0.83). In each prompt, the human story is the rarest of the six versions 57.8% of the time, against 16.7% expected by chance.

**Claude fingerprint.** The paper describes Claude as having the most distinctive narrative profile of the five models. Event intensity escalates less than in any other source, narrative voice is the most uniform, 62% of its stories take a reverent/continuist approach to literary tradition (39–56% for other sources), and it favors epilogues, avoids dream sequences and prefers quiet endings.

## 3. Method: From Features to Rules

### 3.1 Provenance

Every rule traces to a specific published result:

- **Rules 1–23** cover all 30 core features of the paper's Table 16. Several rules group related features: for example, rule 15 combines pre-threat character investment, opening spatial grounding and spatial granularity.
- **Rules 24–30** encode Claude's fingerprint features (Table 17) and the paper's description of Claude's profile (§2.3).
- **Rule 31** encodes the rarity analysis.

No core feature was left out. Appendix A lists the paper's feature name next to each rule.

### 3.2 Rule Formulation

Each rule follows a fixed schema:

```
<imperative> . <evidence: human value vs. AI value> . <operational guidance> . [Non-narrative: <translation>]
```

The evidence sits next to the rule so the agent can judge each rule's strength. For example, the embodied-emotion gap (38% vs. 81%, a 42-point difference and the largest among core features) carries more weight than the location-variety gap (1.34 vs. 1.08 on an ordinal scale).

The direction of each rule follows the sign of the gap. For AI-elevated features the rule tells the agent to do less, and for human-elevated features it tells the agent to do more. Rules are never absolute bans. The human values show that human writers also use many AI-elevated devices, only less often.

### 3.3 Taxonomy

The 31 rules fall into six groups by the kind of decision they constrain. Groups A–D follow the thematic grouping of the paper's Table 16.

| Group | Decision layer | Rules | Representative evidence (human vs. AI) |
|---|---|---|---|
| A. Meaning | Thematic over-determination | 1–5 | Narratorial thematic commentary 52% vs. 77% |
| B. Body & atmosphere | Sensory and embodied performativity | 6–10 | Embodied emotional expression 38% vs. 81% |
| C. Pipeline | Structural streamlining | 11–16 | No subplots 57% vs. 79% |
| D. Absent moves | Temporal complexity, reader engagement, narrative diversity | 17–23 | Chronological discontinuity 2.40 vs. 2.12 |
| E. Self-fingerprint | Claude-specific traits | 24–30 | Reverent/continuist approach: 62% of Claude stories |
| F. Rarity | Global distinctiveness | 31 | Rarity percentile 0.71 vs. 0.49 |

Group E targets the model that runs the skill. Its rules come from the per-source fingerprint analysis, not from the pooled human-vs-AI comparison.

### 3.4 Register Transfer

The source evidence covers fiction only. For non-narrative registers (reports, proposals, documentation, PR descriptions), each rule carries an explicit *Non-narrative* mapping onto a structural analogue:

| Narrative feature | Non-narrative analogue |
|---|---|
| Narratorial thematic commentary | Closing "lesson" sentence → end on data, decision or next step |
| Linear chronology, back-loaded revelation | Context-first exposition → open with the result |
| Resolution by protagonist's choice | Author claims sole credit → acknowledge third parties and the market |
| Resolution by internal understanding | Comforting conclusion → leave open issues open |
| Morally ambivalent protagonist | Cost-free recommendation → state the real cost |
| Direct reader address | "You" / "if you are weighing this…" where the register allows |

These mappings are **hypotheses**. They assume that the cues StoryScope found in fiction carry over to other genres with the same direction of effect. This has not been validated empirically (§6).

## 4. System Design

### 4.1 Integration

The skill is a single Markdown file with YAML frontmatter, loaded by Claude Code's skill mechanism. Its `description` field is written so the agent triggers it **proactively** for any task that produces text for humans, even when the user does not ask for it. It also triggers on explicit requests for human-sounding text. Code itself is out of scope: in code-only tasks the rules apply only to comments, commit messages, PR descriptions and the final report.

The repository is also a single-plugin Claude Code marketplace (`.claude-plugin/`), so the skill can be installed and updated through the plugin system. Installation is described in the README.

### 4.2 Precedence

Rule compliance is bounded by a strict precedence order:

1. **Truth.** No invented references, figures, anecdotes, doubts or complications. Rule 5 ("name references") never licenses fabricating one.
2. **Explicit user request and mandatory formats** (contracts, specifications, forms, ordered data).
3. **StoryScope rules.**

Any rule broken because of (1) or (2) must be disclosed in one line. A rule that does not apply (e.g., smells in a financial report) is satisfied by omission and is never forced.

### 4.3 Execution Protocol

```
          ┌────────────────────┐
 task ──▶ │ 1. Classify        │  narrative | non-narrative | code-only
          └─────────┬──────────┘
                    ▼
          ┌────────────────────┐
          │ 2. Outline in      │  content, order, unresolved items,
          │    reasoning       │  concrete references
          └─────────┬──────────┘
                    ▼
          ┌────────────────────┐
          │ 3. Check outline   │  against rules 1–31 (with register mapping)
          │    vs. rules       │
          └─────────┬──────────┘
                    ▼
          ┌────────────────────┐
          │ 4. Draft           │
          └─────────┬──────────┘
                    ▼
          ┌────────────────────┐   any "yes" ─▶ restructure (back to 2)
          │ 5. Final audit     │   truth check "no" ─▶ correct facts
          └─────────┬──────────┘
                    ▼
                 output
```

The final audit has eight structural questions, each phrased so that "yes" means failure, plus one inverted truth check ("are all added references, figures and facts real and verifiable?"). Lexical edits do not count as a fix for an audit failure.

## 5. Expected Behavior

When the skill is active, the agent's output is expected to show these changes compared with an unconstrained baseline:

- Fewer summary or moral closing paragraphs; endings on concrete data or open questions.
- Emotion stated directly instead of through bodily or atmospheric cues.
- Result-first ordering in expository text.
- Acknowledgment of failed attempts, external dependencies and the costs of recommendations.
- Named, verifiable references in place of vague allusions, or no reference at all.

These are design expectations. No quantitative evaluation is provided in this release (see §8).

## 6. Threats to Validity

**External validity (genre and language).** All evidence comes from English fiction of about 5,000 words. Transfer to short-form, technical or non-English text is an extrapolation (§3.4).

**Statistical vs. individual effects.** The figures describe population tendencies. Human writers also show many "AI-typical" traits (52% include narratorial thematic commentary). One trait in one text is not evidence of anything, and the rules must not be used to judge whether a given text was written by AI.

**Measurement.** Feature values in the source study were produced by LLM annotators (GPT-5.1 for structuring and discovery, Gemini 3 Flash for annotation). Agreement with humans was high (κ = 0.84) but not perfect.

**Model drift.** The Group E fingerprints were measured on Claude Sonnet 4.6. Later Claude models may differ, and other models running the skill have different fingerprints.

**Editing evidence is narrow.** The robustness result (§2.3) covers 278 stories from a single model (Gemini), edited by the same model with one framework.

**Detector evasion is out of scope.** Supervised surface-level baselines reach 99.7–99.9 macro-F1 on the same corpus. The skill does not address lexical or stylometric signals, and **no claim of undetectability is made or should be made**.

## 7. Ethical Considerations

The skill is meant to improve writing quality: less filler, fewer formulaic structures, more honesty about uncertainty and cost. It must not be used to hide AI authorship where disclosure is required (academic submissions, journalism under disclosure policies, regulated communications). The precedence rule in §4.2 puts factual accuracy above every stylistic goal.

## 8. Future Work

- Evaluate on paired outputs, with and without the skill, scored with the StoryScope feature annotations.
- Validate the non-narrative mappings with human readers.
- Re-derive Group E for each new model generation, and for non-Claude models that run the skill.
- Adapt the rules to non-English writing once comparable evidence exists.

## References

- Russell, J., Rajendhran, R., Pham, C. M., Iyyer, M., & Wieting, J. (2026). *StoryScope: Investigating idiosyncrasies in AI fiction.* In *Conference on Language Modeling (COLM 2026)*. arXiv:2604.03136. PDF: https://jenna-russell.github.io/assets/pdf/storyscope.pdf
- Chakrabarty, T., Laban, P., & Wu, C.-S. (2025). Can AI writing be salvaged? Mitigating idiosyncrasies and improving human-AI alignment in the writing process through edits. In *Proceedings of the 2025 CHI Conference on Human Factors in Computing Systems*, 1210:1–1210:33. https://doi.org/10.1145/3706598.3713559
- Presser, S. (2020). *Books3.* Dataset released as part of The Pile.

## Appendix A. Rule–Evidence Index

Feature names and values are from the paper's Table 16 (core features) unless noted. Units follow the paper: **L** = 1–5 Likert scale, mean; **O** = ordinal scale, mean over integer codes; **%** = prevalence of a categorical option. The AI column averages all five models.

| Rule | Paper feature | Human | AI | Unit |
|---|---|---|---|---|
| 1 | Narratorial Thematic Commentary → yes | 52 | 77 | % |
| 2 | Thematic Explicitness & Moralizing | 3.28 | 3.94 | L |
| 2 | Moral / Philosophical Weighting | 3.26 | 3.68 | L |
| 3 | Dialogue Function → philosophical debate | 34 | 59 | % |
| 4 | Thematic Unity | 4.41 | 4.74 | L |
| 5 | Reference Explicitness → implicit echoes | 50 | 72 | % |
| 5 | Intertextual Strategy → explicit named reference | 47 | 24 | % |
| 5 | Reference Explicitness → balanced mix | 37 | 16 | % |
| 6 | Emotional Expression → embodied | 38 | 81 | % |
| 6 | Emotional Expression → explicit labels | 29 | 8 | % |
| 7 | Setting as Psychological Mirror | 3.58 | 4.07 | L |
| 8 | Sensory Modalities → olfactory | 57 | 82 | % |
| 9 | Sensory Density | 3.66 | 3.93 | L |
| 9 | Environmental & Ecological Emphasis | 2.83 | 3.21 | L |
| 10 | Depth of Interior Access | 3.67 | 3.93 | L |
| 11 | Causal Chain Continuity | 3.92 | 4.20 | L |
| 12 | Subplot Integration → no subplots | 57 | 79 | % |
| 12 | Subplot Integration → thematically parallel | 42 | 21 | % |
| 13 | Agency in Resolution → protagonist choice | 46 | 69 | % |
| 14 | Resolution Mode → internal understanding | 27 | 47 | % |
| 15 | Pre-Threat Character Investment | 2.76 | 2.99 | L |
| 15 | Opening Spatial Grounding | 2.12 | 2.33 | O |
| 15 | Spatial Granularity | 2.27 | 2.53 | O |
| 16 | Character Introduction → external description | 30 | 52 | % |
| 17 | Chronological Discontinuity | 2.40 | 2.12 | L |
| 17 | Anachrony Intensity | 2.58 | 2.31 | L |
| 18 | Nonlinear Framing for Delayed Disclosure | 1.96 | 1.68 | L |
| 19 | Depth of Recontextualization After Surprise | 3.28 | 2.95 | L |
| 20 | Direct Reader Address | 0.28 | 0.07 | O |
| 20 | Fourth-Wall Permeability | 0.67 | 0.39 | O |
| 21 | Dialogue-to-Narration Proportion | 2.95 | 2.70 | L |
| 22 | Location Variety Scope | 1.34 | 1.08 | O |
| 23 | Moral Polarity → ambivalent/mixed | 59 | 38 | % |
| 24 | Strength of event escalation (Claude fingerprint #1, Table 17) | — | — | — |
| 25 | Event-type diversity (Claude fingerprint #2) | — | — | — |
| 26 | Ending temporal scope → epilogue/flashforward (Claude fingerprint #3) | — | — | — |
| 27 | Quiet vs. "avalanche" endings (paper §5 text) | — | — | — |
| 28 | Most uniform narrative voice (paper §5 text) | — | — | — |
| 29 | Reverent/continuist approach to tradition (paper §5 text) | — | 62 (Claude) | % |
| 30 | Dreams/visions as temporal distortion → no (Claude fingerprint #4) | — | — | — |
| 31 | Mean rarity percentile (paper §5) | 0.71 | 0.49 | percentile |

All values were checked against the camera-ready PDF.
