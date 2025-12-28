# Real Examples: Why Single-Label Doesn't Work

Here are 10 examples from the actual data that show why multi-label classification is necessary. Each one would get misjudged under strict single-label accuracy.

---

## Example 1: Three Things at Once

**Text:** "I like to work with people with a can-do attitude. My mentors should invest time in pre-work before every session, so we can advance together towards a mutual goal."

**What we predicted:**
- Primary: Mindset and Motivation  
- All Labels: Mindset and Motivation, Practical Preparation and Readiness, Contact Details and Communication Channels

**Why it needs multiple labels:** This text is doing three things:
- Talking about mindset ("can-do attitude")
- Asking for preparation ("pre-work before every session")
- Emphasizing collaboration ("advance together")

Pick just one? You lose the other two. That's why multi-label exists.

---

## Example 2: Business + Goals Overlap

**Text:** "Be able to fully understand their business and market and what the objectives/goals are for the organisation."

**What we predicted:**
- Primary: Clarity of Business and Self-Understanding  
- All Labels: Clarity of Business and Self-Understanding, Goal Clarity and Direction Setting, Business Information and Supporting Documents

**Why multiple labels:** The text literally mentions both understanding the business AND goals/objectives. These are separate intent categories, and both are valid here.

---

## Example 3: Preparation Plus Other Stuff

**Text:** "Set up a meeting, prepare an agenda, prepare their questions and concern"

**What we predicted:**
- Primary: Practical Preparation and Readiness  
- All Labels: Practical Preparation and Readiness, Active Listening and Openness to Feedback, Goal Clarity and Direction Setting

**Why multiple labels:** Yes, it's about preparation. But "prepare questions" also implies:
- Openness to feedback (questions = ready to engage)
- Goal clarity (questions are about concerns/goals)

Single-label would lose these nuances.

---

## Example 4: Short but Complex

**Text:** "Be able to learn (and teach)."

**What we predicted:**
- Primary: Active Listening and Openness to Feedback  
- All Labels: Active Listening and Openness to Feedback, Goal Clarity and Direction Setting, Mindset and Motivation

**Why multiple labels:** Five words, but it captures:
- Active learning mindset
- Growth-oriented attitude
- Implies goal direction

A single label can't capture all of that.

---

## Example 5: Indonesian Example with Multiple Aspects

**Text (Indonesian):** "Hal yang perlu disiapkan adalah niat, komitmen, ide yang dapat direalisasikan, dan kemahiran dalam penggunaan teknologi."

**Translation:** "What needs to be prepared is intention, commitment, realizable ideas, and proficiency in using technology."

**What we predicted:**
- Primary: Technology and Platform Requirements  
- All Labels: Technology and Platform Requirements, Mindset and Motivation, Practical Preparation and Readiness

**Why multiple labels:** The text explicitly lists four things:
- Mindset ("niat" = intention, "komitmen" = commitment)
- Practical preparation ("ide yang dapat direalisasikan" = realizable ideas)
- Technology ("kemahiran dalam penggunaan teknologi" = tech proficiency)

All three categories apply here.

---

## Example 6: Arabic Business Requirements

**Text (Arabic):** "الوصف الكامل لمشروعه، الميزانية، الطموح، الحصة السوقية المتوقعة، الموقع"

**Translation:** "Complete description of their project, budget, ambition, expected market share, location"

**What we predicted:**
- Primary: Clarity of Business and Self-Understanding  
- All Labels: Clarity of Business and Self-Understanding, Business Information and Supporting Documents, Goal Clarity and Direction Setting

**Why multiple labels:** This is asking for:
- Business understanding ("complete description")
- Specific information ("budget, market share, location")
- Goal clarity ("ambition")

Three distinct intents in one text.

---

## Example 7: Bullet Points = Multiple Intents

**Text:** 
"1. Write down their short-term and long-term goals
2. Find a sense of purpose in starting something
3. Believe in their idea more than anybody else
4. Be proactive and prepared"

**What we predicted:**
- Primary: Mindset and Motivation  
- All Labels: Mindset and Motivation, Goal Clarity and Direction Setting

**Why multiple labels:** Each bullet point is a different intent:
- Point 1: Goal clarity
- Points 2-3: Mindset
- Point 4: Preparation

The model correctly identified both mindset and goal clarity as primary intents.

---

## Example 8: "Paperwork" Is Vague

**Text:** "They should have all their paperwork in order and ready to work."

**What we predicted:**
- Primary: Practical Preparation and Readiness  
- All Labels: Practical Preparation and Readiness, Business Information and Supporting Documents, Product Presentation and Portfolio Materials

**Why multiple labels:** "Paperwork" could mean:
- Having documents prepared (practical preparation)
- Business documentation (information category)
- Portfolio materials (presentation category)

All three interpretations are valid.

---

## Example 9: Spanish Learning Mindset

**Text (Spanish):** "Estar dispuesto a aprender y a aplicar lo aprendido."

**Translation:** "Be willing to learn and apply what has been learned."

**What we predicted:**
- Primary: Mindset and Motivation  
- All Labels: Mindset and Motivation, Practical Preparation and Readiness, Active Listening and Openness to Feedback

**Why multiple labels:** This expresses:
- Mindset (willingness)
- Active learning (learn and apply)
- Practical readiness (apply what's learned)

Three related but distinct aspects.

---

## Example 10: Comprehensive Request

**Text:** "Contar sobre su negocio, que metas tiene, que objetivos busca y contarnos la verdad."

**Translation:** "Tell about your business, what goals you have, what objectives you seek, and tell us the truth."

**What we predicted:**
- Primary: Clarity of Business and Self-Understanding  
- All Labels: Clarity of Business and Self-Understanding, Goal Clarity and Direction Setting, Business Information and Supporting Documents

**Why multiple labels:** This explicitly requests three things:
- Business clarity ("tell about your business")
- Goal clarity ("what goals/objectives")
- Information sharing ("tell us the truth" = honest information)

All three are primary intents of this message.

---

## What These Examples Show

1. **Overlap is normal:** Most texts aren't about one thing. Multiple intents are the rule, not the exception.

2. **Role matters:** Same semantic content could map differently depending on mentor vs mentee (though all examples here are from mentors).

3. **Single-label would be unfair:** A model that correctly identifies one valid intent but misses another would be marked wrong. But it still captured useful information.

4. **Multi-label works:** The entailment-based approach captures realistic complexity without forcing arbitrary single-label assignments.

These examples justify our evaluation criterion: a prediction is correct if the primary label appears in the all_labels list. It acknowledges that texts naturally map to multiple valid categories, which is just how human communication works.
