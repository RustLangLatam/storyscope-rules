---
name: storyscope-rules
description: "Mandatory writing rules derived from the StoryScope study (Russell et al., 2026; 61,608 stories) on the content and structure decisions that give away AI-written text. Use it BEFORE starting any task that will produce text for people, even when not asked: proposals, reports, emails, documentation, scripts, pitches, fiction, interface copy, PR descriptions and long messages. Also use it when asked to make something not sound AI-written, sound human, or have less filler. In code-only tasks it applies to comments, commits and the final report."
license: MIT
---

# StoryScope Rules

The study compared the same writing prompt answered by a person and by five models (Claude, GPT, Gemini, DeepSeek and Kimi). Ignoring style entirely, narrative structure alone separated human from AI text at 93.2 macro-F1. In the study's one editing test (278 Gemini stories rewritten with the LAMP framework to remove clichés and purple prose), detection barely moved: 95.5 → 93.9 macro-F1.

**That is why these rules apply before writing.** What gives the text away is what gets told and in what order. The paper notes that changing narrative features requires "significant structural rewrites rather than simple post-hoc edits". Applying the rules before drafting is this skill's inference from that; the paper did not test it.

## How to use it

1. **Before starting**, classify the task:
   - **Fiction or narrative:** all rules apply as written.
   - **Non-narrative text:** rules apply through the *Non-narrative* translation given for each one.
   - **Code only:** rules apply to comments, commits, PR descriptions and the final report. The code itself does not change because of these rules.
2. **Before drafting**, write the outline in your reasoning: what is told, in what order, what stays unresolved and which concrete references it carries. Check the outline against the rules.
3. **When done**, run the final audit below and restructure whatever fails. Swapping words does not count as a fix.

## What overrides these rules

The rules hold except in two cases, because a false text is worse than one that sounds like AI:

- **Truth.** Never invent a reference, figure, anecdote, doubt or complication to satisfy a rule. If there is no real reference to name, there is none.
- **The user's explicit request and mandatory formats** (a contract, a specification, a form, data that must be in order). If they clash with a rule, the request wins.

If you break a rule for one of these reasons, say so in one line. If a rule makes no sense for the task, such as the one about smells in a report, do not force it: leaving smells out already satisfies it.

## Rules

Figures: **human vs. AI** in the study; scales run from 1 to 5.

### A. Stop explaining the meaning

1. **Do not spell out the moral or the theme.** The narrator explains it in 77% of AI texts and 52% of human ones. Delete the final paragraph that sums up what it all means and let the reader infer it.
   *Non-narrative:* end on the data point, the decision or the next step, with no lesson sentence.
2. **Lower the moralizing, philosophical tone.** Explicitness 3.28 vs. 3.94; moral weight 3.26 vs. 3.68. No grand reflections on what is at stake. *Non-narrative:* no "the future of…" and no closing general principles.
3. **Dialogue is not a debate of ideas.** 34% vs. 59%. Guidance (not from the paper): people talk to get something, avoid something or get out of a spot. *Non-narrative:* do not raise straw objections just to answer them.
4. **Not everything must serve a single idea.** Thematic unity 4.41 vs. 4.74. Keep what matters even if it does not fit the thesis, including data that complicates it.
5. **Name references instead of vague allusions.** Vague allusions: 50% vs. 72%. Explicit named reference: 47% vs. 24%. Both combined: 37% vs. 16%. Say which work, author, company, report, place or version. *If no real one exists, see "What overrides these rules".*

### B. Stop acting it out through body and atmosphere

6. **Name the emotion when that is natural.** Somatic: 38% vs. 81%. Direct label ("she was afraid"): 29% vs. 8%. This is the largest percentage-point gap among the study's core features. No tightening chests, cold sweat or closing throats by default.
7. **The setting does not mirror the mood.** 3.58 vs. 4.07. No rain in the sad scene, no dimming lights. *Non-narrative:* no metaphors of storms, horizons or landscapes.
8. **Fewer smells.** 57% vs. 82%. Do not add smells by reflex.
9. **Less sensory density and less nature.** Density 3.66 vs. 3.93; nature 2.83 vs. 3.21. Use one chosen detail, not a list of senses.
10. **Do not render inner life in depth all the time.** Depth of interior access 3.67 vs. 3.93. Let some of what a person thinks be inferred from what they do.

### C. Break the tidy pipeline

11. **Do not chain everything into one perfect causal line.** 3.92 vs. 4.20. Some things just happen and others lead nowhere. *Non-narrative:* also say what was tried and did not work, if it really happened.
12. **Include secondary threads.** No subplots: 57% vs. 79%. Subplots that resonate with the theme: 42% vs. 21%. *Non-narrative:* include the parallel issue that affects the main one.
13. **The protagonist does not solve everything alone.** Resolution by own decision: 46% vs. 69%. Let others, chance or circumstance step in. *Non-narrative:* acknowledge what depended on third parties or the market.
14. **Do not close on an inner realization.** Acceptance or understanding: 27% vs. 47%. An ambiguous or unresolved ending is valid. *Non-narrative:* leave open what is open, with no comforting conclusion.
15. **Give less setup before the problem.** Prior investment 2.76 vs. 2.99; establishing the entry space 2.12 vs. 2.33; spatial detail 2.27 vs. 2.53. Get into it. *Non-narrative:* no opening context; start with what matters.
16. **Do not introduce anyone by their looks** (in non-narrative text, by their résumé). External description: 30% vs. 52%. Introduce them talking or doing something; introduction in dialogue is the top human fingerprint by uniqueness ratio.

### D. Do what AI avoids

17. **Break chronological order when it helps.** Discontinuity 2.40 vs. 2.12; jumps back or forward 2.58 vs. 2.31. Do not tell it from first clue to big reveal: a story can open at the funeral and go back. *Non-narrative:* open with the result, then explain how you got there.
18. **Use time jumps to delay what matters.** 1.96 vs. 1.68.
19. **Let a revelation force a re-reading.** 3.28 vs. 2.95. A late fact should change what seemed true.
20. **Address the reader.** Direct address: 0.28 vs. 0.07; breaking the fourth wall: 0.67 vs. 0.39 (ordinal means in the paper's Table 16; its prose reports them as 28% vs. 7% and 67% vs. 39%). AI writes as if no one were watching. *Non-narrative:* "you", "if you are weighing this…", when the register allows.
21. **More dialogue, less narration.** 2.95 vs. 2.70.
22. **More places.** 1.34 vs. 1.08. Do not keep everything in one setting.
23. **Ambiguous morality.** Ambivalent protagonist: 59% vs. 38%. Their choices can carry a cost and be debatable. *Non-narrative:* state the real cost of what you recommend.

### E. Avoid your own habits (Claude)

The study measured these traits on Claude Sonnet 4.6. **If you are not a Claude model**, treat rules 24–30 as optional and check your own family's habits instead: GPT leans on gossip and rumor as a plot device and on distant retrospective framing; Gemini defaults to external character description, extended denouements, tidy endings and bleak settings; DeepSeek front-loads crucial context; Kimi sits at the generic center of AI writing. Rules 1–23 and 31 apply to every model.

The study identifies these Claude traits. They compare Claude with the other models; the paper gives no human baseline for them.

24. **Do not keep intensity flat.** Claude is the model that escalates least. Let something truly get worse.
25. **Check the variety of events.** Event-type diversity is Claude's second strongest fingerprint. The paper does not state the direction, so check that the kinds of events actually vary.
26. **Do not add an epilogue or a final jump into the future.**
27. **Prefer rupture to a final calm.** Do not always end on a quiet scene.
28. **Vary the voice.** Claude's narrative voice is the most uniform of all sources. Let register and rhythm change where the content changes.
29. **Do not always honor convention.** 62% of its stories follow tradition reverently. Question it when it makes sense.
30. **Allow dreams or visions if they add something,** because Claude avoids them. Do not force them.

### F. Be an unusual choice

31. **Avoid the first idea.** Human texts are rarer in this space (rarity percentile 0.71 vs. 0.49). AI stories cluster together in narrative space, so the first structure that comes to mind is likely a common one. Discard it and pick another if it serves equally well.

## Final audit

Answer with the text in front of you. A "yes" requires restructuring:

- Does it end by explaining what it all means, or on a lesson or a calm?
- Is everything in order and on a single line, with every element serving the thesis?
- Are there vague allusions where a real name would fit?
- Do the emotions live in the body, the weather or the light?
- Is everything resolved by a decision or realization of the main figure?
- Is there a long setup before anything happens?
- Is the intensity flat, and does the voice never change?
- Is it how any model would write it for this prompt?

And the other way round, a "no" requires correction: are all added references, figures and facts real and verifiable?

## Limits worth stating

- **It is a study of fiction** in English, with stories averaging about 4,750 words (length varies widely by source). Applying it to other genres is an extrapolation.
- **These are statistical tendencies.** The average human also does many of these things (52% explain the theme), and an isolated trait proves nothing.
- **It does not guarantee passing a detector.** Trained text-based detectors (TF-IDF, stylometric, fine-tuned ModernBERT) reach 99.7–99.9 macro-F1 and this skill does not address them. Do not promise the text is undetectable.
- **The features were labeled by LLMs** (GPT-5.1 and Gemini 3 Flash; κ = 0.84 against humans), not by human readers.
- **It is not for hiding authorship where it must be disclosed** (academic work, for example). It is for writing better.

Source: Russell, Rajendhran, Pham, Iyyer and Wieting. *StoryScope: Investigating idiosyncrasies in AI fiction.* COLM 2026. arXiv:2604.03136.
