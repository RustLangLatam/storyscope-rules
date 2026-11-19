# StoryScope Rules: Operationalizing Narrative-Level Discriminators of AI-Generated Text as Pre-Writing Constraints for LLM Agents

**Technical Report — v1.0.0 — November 2026**

---

## Abstract

Detectors of machine-generated text usually exploit surface statistics: token distributions, function-word frequencies and n-gram profiles. Russell et al. (2026) showed that a second, deeper signal exists. Using only *narrative* features, with style features excluded, an XGBoost classifier separated human-written from AI-written fiction at 93.2 macro-F1 over 61,608 stories, and the signal survived stylistic editing (93.9 macro-F1). This report describes **StoryScope Rules**, an agent skill for Claude Code that turns the study's reported human–AI feature differences into 31 constraints applied *before* drafting, plus an 8-item audit applied after drafting. We describe the source evidence, how features became rules, the extension to non-narrative registers, how the skill runs inside the agent, and the main threats to validity: genre transfer, the statistical rather than individual nature of the effects, and the absence of any claim about detector evasion.

**Keywords:** AI-generated text, narrative structure, LLM idiosyncrasies, writing constraints, agent skills, Claude Code.

---

## 1. Introduction

Most attempts to make LLM output read as human-written act on the surface: synonym substitution, sentence-length variation, removal of stock phrases. StoryScope suggests these interventions target the wrong layer. If narrative decisions alone (what is told, in what order, how conflicts resolve, how emotion is conveyed) separate human from AI text at over 93 macro-F1, and editing style leaves that separation intact, then rewording after drafting cannot remove it. The text has to be planned differently.

This motivates the design principle of the skill:

> **P1 (Pre-writing intervention).** Constraints are applied to the outline before any prose is produced. Post-hoc edits that change wording but not structure do not count as compliance.

The contribution of this work is engineering, not empirical. We do not run new experiments. We translate published effect directions into an executable procedure for an LLM agent, and we document each translation step so it can be inspected.

## 2. Source Study

### 2.1 Corpus

| Property | Value |
|---|---|
| Writing prompts | 10,272, reverse-engineered from human stories with Gemini 2.5 Flash |
| Human source | Books3 (Presser, 2020) |
| AI sources | Claude Sonnet 4.6, DeepSeek V3.2, Gemini 3 Flash, GPT-5.4, Kimi K2.5 |
| Design | Paired: 1 human + 5 AI stories per prompt |
| Total stories | 61,608 (51,336 AI after 24 refusals) |
| Length | ~5,000 words nominal; corpus mean 4,753 |
| Language / genre | English fiction |

The paired design controls for prompt content. Differences between the human and AI story for the same prompt cannot come from topic.

### 2.2 Feature Pipeline

1. **Structuring.** GPT-5.1 converts each story into a JSON template over ten NarraBench dimensions.
2. **Discovery.** GPT-5.1 compares stories across sources in a 600-story pool and proposes candidate features. Three runs produced 408 candidates, which embedding-based clustering reduced to **304** features (categorical, ordinal, scale, binary, multi-select).
3. **Annotation.** Gemini 3 Flash labels every story on every feature. Reported reliability: Krippendorff's α = 0.90 across five runs; mean Cohen's κ = 0.84 against human annotators.
4. **Style exclusion.** An audit flags 47 style-related features. The remaining **257** form the *Narrative-only* set.

### 2.3 Key Results

| Feature set | Task | Macro-F1 |
|---|---|---|
| Narrative-only (257) | Human vs. AI | **93.2** |
| Narrative-only, after stylistic editing (LAMP) | Human vs. AI | **93.9** |
| Core features only (30) | Human vs. AI | 84.8 |
| Narrative + Style (304) | Human vs. AI | 96.0 |
| Narrative-only | 6-way source attribution | 68.4 |
| TF-IDF + XGBoost | Human vs. AI | 99.7 |
| Stylometric + XGBoost | Human vs. AI | 99.8 |
| Binoculars (zero-shot) | Human vs. AI | 55.9 |

The study also reports that human stories occupy rarer regions of feature space (mean rarity percentile 0.71 vs. 0.49 for AI), and identifies model-specific "fingerprints". For Claude these include notably flat event escalation.

## 3. Method: From Features to Rules

### 3.1 Selection

Features were kept when they met three criteria:

- **C1 Directional.** The study reports a clear human–AI difference in proportion or mean scale value.
- **C2 Controllable.** A writer can change the feature by a planning decision. Features that depend on length or corpus artifacts were excluded.
- **C3 Truth-preserving.** Complying never requires fabricating facts (see §4.2).

### 3.2 Rule Formulation

Each rule follows a fixed schema:

```
<imperative> . <evidence: human value vs. AI value> . <operational guidance> . [Non-narrative: <translation>]
```

The evidence is stated next to the rule so the agent can judge each rule's strength. For example, the somatic-emotion gap (38% vs. 81%) carries more weight than the location count (1.34 vs. 1.08).

### 3.3 Taxonomy

The 31 rules fall into six groups by the kind of decision they constrain:

| Group | Decision layer | Rules | Representative evidence (human vs. AI) |
|---|---|---|---|
| A. Meaning | Thematic explicitness | 1–5 | Theme explained by narrator 52% vs. 77% |
| B. Body & atmosphere | Emotional and sensory encoding | 6–10 | Somatic emotion 38% vs. 81% (largest gap) |
| C. Pipeline | Causal and plot topology | 11–16 | No subplots 57% vs. 79% |
| D. Absent moves | Techniques AI under-uses | 17–23 | Time discontinuity 2.40 vs. 2.12 |
| E. Self-fingerprint | Claude-specific traits | 24–30 | Reverent convention 62% of Claude stories |
| F. Rarity | Global distinctiveness | 31 | Rarity percentile 0.71 vs. 0.49 |

Group E targets the model running the skill. Its rules come from the study's per-source fingerprints, not from the pooled human-vs-AI comparison.

### 3.4 Register Transfer

The source evidence covers fiction only. For non-narrative registers (reports, proposals, documentation, PR descriptions) each rule carries an explicit *Non-narrative* mapping onto a structural analogue:

| Narrative feature | Non-narrative analogue |
|---|---|
| Closing moral / theme explanation | Closing "lesson" sentence → end on data, decision or next step |
| Linear chronology from first clue to reveal | Context-first exposition → open with the result |
| Protagonist resolves by own decision | Author claims sole credit → acknowledge third parties and market |
| Inner-realization ending | Comforting conclusion → leave open issues open |
| Morally ambiguous protagonist | Cost-free recommendation → state the real cost |
| Direct reader address | "You" / "if you are weighing this…" where the register allows |

These mappings are **hypotheses**. They assume that the reader-side cues StoryScope found in fiction carry over to other genres with the same direction of effect. This has not been validated empirically (§6).

## 4. System Design

### 4.1 Integration

The skill is a single Markdown file with YAML frontmatter, loaded by Claude Code's skill mechanism. Its `description` field is written so the agent triggers it **proactively** for any task that produces text for humans, even when the user does not ask for it. It also covers explicit requests for human-sounding text. Code itself is out of scope: in code-only tasks the rules apply only to comments, commit messages, PR descriptions and the final report.

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
- Emotion stated directly instead of through somatic or atmospheric cues.
- Result-first ordering in expository text.
- Acknowledgment of failed attempts, external dependencies and the costs of recommendations.
- Named, verifiable references in place of vague allusions, or no reference at all.

No quantitative evaluation of these shifts is provided in this release (§7).

## 6. Threats to Validity

**External validity (genre).** All evidence comes from English fiction of ~5,000 words. Transfer to short-form, technical or non-English text is an extrapolation (§3.4).

**Statistical vs. individual effects.** The figures describe population tendencies. Human writers also show many "AI-typical" traits (52% explain the theme). One trait in one text is not evidence of anything, and the rules should not be used to judge whether a given text was written by AI.

**Measurement.** Feature values in the source study were produced by LLM annotators (GPT-5.1, Gemini 3 Flash). Agreement with humans was high (κ = 0.84), but it was not perfect.

**Model drift.** The fingerprints in Group E were measured on Claude Sonnet 4.6. Later models may differ.

**Detector evasion is out of scope.** Surface-level detectors reach 99.7–99.9 macro-F1 on the same corpus. The skill does not address lexical or stylometric signals, and **no claim of undetectability is made or should be made**.

## 7. Ethical Considerations

The skill is meant to improve writing quality: less filler, fewer formulaic structures, more honesty about uncertainty and cost. It must not be used to hide AI authorship where disclosure is required (academic submissions, journalism under disclosure policies, regulated communications). The precedence rule in §4.2 puts factual accuracy above every stylistic goal.

## 8. Future Work

- Evaluate on paired outputs, with and without the skill, scored by the StoryScope feature annotator.
- Validate the non-narrative mappings with human readers.
- Re-derive Group E for each new model generation.
- Localize the rules for non-English writing once comparable evidence exists.

## References

- Russell, J., Rajendhran, R., Pham, C. M., Iyyer, M., & Wieting, J. (2026). *StoryScope: Investigating idiosyncrasies in AI fiction.* arXiv:2604.03136. https://arxiv.org/abs/2604.03136
- Presser, S. (2020). *Books3.* Dataset released as part of The Pile.

## Appendix A. Rule–Evidence Index

| # | Rule (short) | Human | AI | Unit |
|---|---|---|---|---|
| 1 | Do not explain theme | 52% | 77% | proportion |
| 2 | Less explicit / moralizing | 3.28 / 3.26 | 3.94 / 3.68 | scale 1–5 |
| 3 | Dialogue not a debate | 34% | 59% | proportion |
| 4 | Lower thematic unity | 4.41 | 4.74 | scale 1–5 |
| 5 | Named references | 47% | 24% | proportion |
| 6 | Somatic emotion / direct label | 38% / 29% | 81% / 8% | proportion |
| 7 | Setting mirrors mood | 3.58 | 4.07 | scale 1–5 |
| 8 | Smells | 57% | 82% | proportion |
| 9 | Sensory density / nature | 3.66 / 2.83 | 3.93 / 3.21 | scale 1–5 |
| 10 | Interiority | 3.67 | 3.93 | scale 1–5 |
| 11 | Single causal line | 3.92 | 4.20 | scale 1–5 |
| 12 | No subplots | 57% | 79% | proportion |
| 13 | Self-driven resolution | 46% | 69% | proportion |
| 14 | Realization ending | 27% | 47% | proportion |
| 15 | Prior setup | 2.76 | 2.99 | scale 1–5 |
| 16 | External introduction | 30% | 52% | proportion |
| 17 | Temporal discontinuity | 2.40 | 2.12 | scale 1–5 |
| 18 | Delaying time jumps | 1.96 | 1.68 | scale |
| 19 | Re-reading revelation | 3.28 | 2.95 | scale 1–5 |
| 20 | Reader address / fourth wall | 0.28 / 0.67 | 0.07 / 0.39 | mean |
| 21 | Dialogue share | 2.95 | 2.70 | scale 1–5 |
| 22 | Number of locations | 1.34 | 1.08 | mean |
| 23 | Ambivalent protagonist | 59% | 38% | proportion |
| 24–30 | Claude fingerprints | — | — | qualitative |
| 31 | Rarity percentile | 0.71 | 0.49 | percentile |

Values are those encoded in the skill. §2 figures were checked against the arXiv abstract and HTML (v6). Per-feature values in this table were not independently re-checked against the paper's appendices.
