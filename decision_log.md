# Decision Log: Why We Made These Choices

This is basically a journal of the decisions I made during this project and why. Each one came from running into a problem and figuring out how to solve it.

---

## 1. Starting with SMAT, Then Moving to Clustering

**What I tried first:** I looked at the Stanford Mentor Alignment Tool (SMAT) categories:
- Purpose / Motivation
- Expectations
- Communication Preferences
- Boundaries / Limits
- Goals
- Feedback / Evaluation

I tried classifying the first 100 rows using these. They didn't work — too abstract, didn't match what people actually write in real responses.

**What I did next:** Since SMAT failed, I ran unsupervised clustering on 6,000 unique non-NaN texts using sentence transformers, UMAP, and HDBSCAN.

**Why clustering:** I had no idea what intents actually existed in the real data. Unlike standard classification tasks where labels are obvious (sentiment, topic, etc.), mentor-mentee "before we start" texts didn't have an established taxonomy. I needed to discover what was actually there.

**What I learned:** The clustering found 96 initial groups that I merged down to ~12 categories. More importantly, it showed me that a huge cluster (918 out of 1000 texts) had massive overlap. The discovered categories were much more aligned with real-world responses than SMAT's theoretical framework. That was my first clue that single-label classification wouldn't work.

---

## 2. What the Clustering Actually Revealed

**The ugly truth:** When I merged 96 clusters down to 10 categories, the biggest one had ~92% of all texts. That's not normal. It means:
- Boundaries between categories are fuzzy
- Many texts legitimately belong to multiple categories
- Single-label classification would force artificial choices

**Specific example:** Categories like "Practical Preparation," "Goal Clarity," and "Business Understanding" kept overlapping. The same text could reasonably belong to all three.

Also noticed: mentor and mentee texts clustered differently. Same words, different meanings depending on who's saying them.

**The conclusion:** Clustering gave me hard evidence that multi-label was the only realistic path forward.

---

## 3. Why Single-Label Didn't Work

**What I tried first:** Standard softmax-based single-label classification.

**Why it failed:**

1. **Texts are naturally multi-intentional.** Take "Prepare questions about your business goals" — that's clearly preparation AND goal-setting AND business understanding. Forcing it into one category loses information.

2. **Role matters.** "Bring your business plan" means different things:
   - From a mentor: practical preparation
   - From a mentee: understanding mentor background (expecting feedback)

3. **Information loss.** Single-label forces you to pick one, throwing away valid secondary intents.

4. **Evaluation is unfair.** If a model correctly identifies one valid intent but misses another, you'd mark it wrong. But it still captured useful information.

**Real example:** "I want to clarify my goals and prepare documents" would be forced into either "Goal Clarity" OR "Preparation" — but both are clearly present.

---

## 4. Why Entailment-Based Multi-Label

**What I chose:** Zero-shot classification using `joeddav/xlm-roberta-large-xnli` (XLM-RoBERTa Large XNLI) that predicts both a primary label and an all_labels list.

**The reasoning:**

1. **It understands meaning, not just keywords.** Entailment models evaluate semantic compatibility between text and hypotheses. With 5+ languages in the dataset, keyword matching would fail completely.

2. **Zero-shot is flexible.** I can tweak labels or add new ones without retraining. When clustering revealed new patterns (or when SMAT categories failed), I could adapt quickly.

3. **Multi-label is natural.** NLI models give independent probability scores per label. Perfect for saying "this text supports both intent A and intent B."

4. **Works across languages.** XLM-RoBERTa was trained on multilingual data, so it handles Spanish, Indonesian, Arabic, etc. without translation.

**How it works:**
- **Hypothesis templates:** For each intent category, I created templates in multiple languages (English, Spanish, Arabic). The model checks "Does this text support this hypothesis?" and returns probabilities.
- Raw probability thresholds: ≥0.75 for strong labels, ≥0.60 for secondary
- Cap at 3 labels max (otherwise it gets chaotic)
- UNKNOWN when nothing scores high enough

**Initial validation:** Tested on 100 unique rows. Results showed Understanding Mentor Background (26), Clarity of Business (20), Goal Clarity (19), Mindset (17) — distribution looked reasonable, so I proceeded.

---

## 5. Why I Added an UNKNOWN Class

**The problem:** Some texts are just... unclear. "Let's talk!" or "Aún no lo sé" (I don't know yet) don't fit any category well. Others are meta-level ("I'm not a mentor") or too short ("c", ".....").

**My solution:** Explicitly support UNKNOWN rather than forcing a label.

**Why this matters:**
- Avoids label noise from forced misclassification
- Preserves trust in the system (better to say "I don't know" than guess wrong)
- Handles edge cases gracefully

**When it triggers:** When all label probabilities fall below 0.30 threshold, or when the text is genuinely ambiguous/meta-level.

**The philosophy:** It's better to admit uncertainty than pretend we know.

---

## 6. Why Mentor and Mentee Data Are Separate

**What I do:** Process mentor and mentee texts separately, even though they use the same model and labels.

**The reason:** Same words, different meanings. "Prepare questions" from a mentor means "you should prepare questions." From a mentee, it means "I want to ask questions" (Understanding Mentor Background or Goal Clarity).

**Other considerations:**
- Different intent distributions: mentors emphasize preparation more, mentees focus on understanding the mentor
- Keeping them separate preserves role context for future improvements
- Easier to evaluate role-specific performance

**Current state:** They're processed separately but use identical hypothesis templates. Could add role-specific templates later, but separate processing was the minimum necessary change.

---

## Summary

Every decision came from hitting a problem:
- SMAT categories → too abstract, didn't match real data
- Clustering → discovered overlap and real-world categories
- Multi-label → overlap made single-label impossible
- Entailment → needed semantic understanding across languages
- UNKNOWN → needed to handle ambiguity honestly
- Separate processing → role changes meaning

The methodology emerged from the data, not from theory. Started with a framework (SMAT), found it didn't work, then let the data guide us through clustering. That's why it works.
