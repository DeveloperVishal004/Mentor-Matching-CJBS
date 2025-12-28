# Multi-Label Intent Classification for Mentor–Mentee Conversations

This project tackles a tricky problem: understanding what mentors and mentees actually mean when they say what they need before a mentoring session starts. The catch? Most texts express multiple intents at once, and the same words mean different things depending on whether a mentor or mentee wrote them.

Instead of forcing everything into single categories (which doesn't work well here), I built an entailment-based multi-label classifier that accepts that real communication is messy and multi-intentional.

---

## What We're Trying to Solve

When mentors and mentees describe what they need before starting, their messages are rarely just about one thing. Take "Prepare questions about your business goals" — that's preparation, goal-setting, AND business understanding all wrapped together. Traditional single-label classification forces you to pick just one, losing valuable information.

**What we did differently:** Instead of one label, we predict a `primary_label` (the main intent) plus an `all_labels` list (other valid intents in the text). A prediction is correct if the primary label shows up in the all_labels list — simple, but it reflects how people actually communicate.

---

## The Dataset

The dataset we're working with is available [here](https://drive.google.com/file/d/11OIC3hBRzPGXzG9oIo797_p7CcDTnEQ-/view?usp=drive_link). It contains short free-text responses in multiple languages (English, Spanish, Indonesian, Arabic, Russian, etc.) collected from mentor-mentee matching platforms. These are all from the "before we start" stage — basically, people explaining what information or preparation they think is needed before a mentoring session begins.

### Why This Is Hard

A few things make this problem particularly tricky:

1. **Everything overlaps:** Most texts aren't about just one thing. "Prepare questions about your business goals" mixes preparation, goal-setting, and business understanding. Real communication is messy like that.

2. **Same words, different meanings:** The same sentence means different things depending on who wrote it. When a mentor says "Bring your business plan," they mean practical preparation. When a mentee says the same thing, they might be thinking about getting feedback (Understanding Mentor Background).

3. **It's multilingual:** We're dealing with English, Spanish, Indonesian, Arabic, Russian, and more. Keyword matching won't cut it — you need actual semantic understanding.

4. **Sometimes it's just ambiguous:** Some responses are vague or meta-level ("Let's talk!" or "I'm not sure yet"). Forcing these into categories feels wrong.

5. **No roadmap to start:** Unlike typical classification tasks where the labels are obvious, we had no predefined taxonomy. I started with SMAT (Stanford Mentor Alignment Tool) categories, but they were too abstract and didn't match real responses. We had to discover what the intents actually were through clustering.

---

## How We Got Here

This project went through several iterations. Here's the actual journey:

### Phase 0: Starting with SMAT (Didn't Work)

I first looked at the [Stanford Mentor Alignment Tool (SMAT)](https://docs.google.com/document/d/1XTQfkDcxw6us7KKD7Ktgwr1qWFyNALkqIgSA50cA1rs/edit), which uses these categories:
- Purpose / Motivation
- Expectations
- Communication Preferences
- Boundaries / Limits
- Goals
- Feedback / Evaluation

I tried classifying the first 100 rows using these categories. They didn't capture the variety in real mentor-mentee responses. Too abstract, didn't match what people actually write.

**Lesson learned:** Theoretical frameworks don't always map to real-world data. Time to let the data speak.

### Phase 1: Let the Data Show Us What's There (Clustering)

After SMAT failed, I tried a data-driven approach: clustering to discover what intents actually exist.

**What I did:**
1. Grabbed 6,000 unique non-NaN texts from the dataset
2. Created multilingual sentence embeddings using `paraphrase-multilingual-MiniLM-L12-v2`
3. Used UMAP to reduce dimensions to 5D (way faster for clustering)
4. Ran HDBSCAN clustering to find natural groups
5. Merged the 96 initial clusters down to ~12 categories that actually made sense

**What I found:**
- 96 initial clusters — way more diversity than expected
- The biggest merged cluster had ~92% of texts. That's a red flag for single-label classification right there.
- Boundaries were super fuzzy. Lots of texts could belong to multiple categories.
- Examples from the clustering showed clear patterns (see the clusters with top terms like 'business', 'internet', 'data', 'waktu/schedule', etc.)

Check out `Clustering_(1).ipynb` if you want to see the full exploration process.

**The key insight:** Clustering revealed that forcing single labels would fail. Too much overlap, too much ambiguity. Also, the discovered categories were more aligned with real-world responses than the theoretical SMAT framework.

### Phase 2: Building the Taxonomy from Clusters

After analyzing the clusters and their top terms/examples, I developed 13 intent categories that actually match what people are saying. Here's what each one means:

#### 1. Mindset and Motivation

Messages about the mentee's mindset, confidence, motivation, willingness to learn, or readiness to grow.

**Examples:**
- "It takes belief to get started; a growth mindset turns dreams into reality."
- "Show commitment and a strong desire to learn."
- "Be positive and stay motivated throughout the journey."

#### 2. Clarity of Business and Self-Understanding

Messages where the mentee talks about understanding their business, market knowledge, product, customers, or competitors.

**Examples:**
- "Know the real market before you begin."
- "Understand your business, your customers, and your competitors."
- "Be clear about what your product is and who it is for."

#### 3. Practical Preparation and Readiness

Messages asking the mentee to prepare specific items before the session, such as questions, challenges, topics, or documents.

**Examples:**
- "Come prepared with focused questions about your business and challenges."
- "Make a list of what you want to achieve so we can discuss it."
- "Prepare the topics you want to explore in our meeting."

#### 4. Technology and Platform Requirements

Messages about technical needs such as internet, devices, software, Zoom/Teams, or other platforms.

**Examples:**
- "Please ensure you have a laptop and a stable internet connection."
- "Zoom must be installed before the session."
- "Have your smartphone and data ready."

#### 5. Business Information and Supporting Documents

Requests for business documents, financial details, reports, business plans, legal information, or other paperwork.

**Examples:**
- "Have your business plan and company profile ready."
- "Please share any financial figures you have."
- "Submit your market research or survey documents if available."

#### 6. No Preparation Needed

Messages clearly stating that no preparation or materials are required.

**Examples:**
- "Nothing specific required."
- "No preparation is needed — just show up."
- "Nothing to do beforehand."

#### 7. Time Availability and Commitment

Messages about scheduling, available time, session timing, or consistency of attendance.

**Examples:**
- "Please be available at the scheduled time."
- "Set aside some free time for our sessions."
- "Commit to being consistent during the mentoring process."

#### 8. Product Presentation and Portfolio Materials

Requests to bring product photos, presentation decks, demos, or portfolio materials.

**Examples:**
- "Bring photos of your product."
- "Prepare a simple pitch deck or project presentation."
- "Have your sample designs or prototypes ready."

#### 9. Contact Details and Communication Channels

Messages explaining how to contact the mentor or preferred communication methods such as WhatsApp, email, or messaging.

**Examples:**
- "Send me a message anytime, and I will reply as soon as possible."
- "Share your email or WhatsApp number so I can contact you."
- "Feel free to reach out anytime you need help."

#### 10. Active Listening and Openness to Feedback

Messages about being open to feedback, listening carefully, or accepting guidance.

**Examples:**
- "Be prepared to listen actively and take feedback seriously."
- "Come with an open mind and be ready to receive constructive criticism."
- "Listening carefully will help you get the most out of our sessions."

#### 11. Goal Clarity and Direction Setting

Messages asking the mentee to define goals, expectations, or what they want to achieve.

**Examples:**
- "What is your goal for this mentorship?"
- "Think about what you want to achieve and share it with me."
- "What outcome are you hoping for?"

#### 12. Understanding Mentor Background

Messages telling the mentee to learn about the mentor's experience, profile, or professional background.

**Examples:**
- "Research who I am and what I do."
- "Please check my website or LinkedIn to know my background."
- "Get familiar with my experience so we can save time."

#### 13. Social Media and Digital Presence

Messages related to social media marketing, digital platforms, online presence, or discussing digital content.

**Examples:**
- "Be ready to talk about social media for your business."
- "Prepare your questions about digital marketing."
- "Think about your brand's presence on Instagram and other platforms."

#### 14. UNKNOWN

**About UNKNOWN:** I added this because some texts are just... unclear. "Let's talk!" or "..." or "Aún no lo sé" (I don't know yet) don't fit anywhere. Some are meta-level ("I'm not a mentor") or too short/incomplete. Better to admit we don't know than force a wrong label.

### Phase 3: Actually Building the Classifier

I tried the standard softmax single-label approach first. It didn't work. Too much overlap. So I switched to zero-shot entailment classification instead.

**The setup:**
- **Model:** `joeddav/xlm-roberta-large-xnli` (XLM-RoBERTa Large XNLI) — multilingual, handles all the languages in the dataset
- **How it works:** Zero-shot classification using hypothesis templates for each intent (in English, Spanish, and Arabic)
- **Hypothesis templates:** For each category, I created templates that ask "Does this text support this hypothesis?" The model returns probabilities, and high probability = the text supports that intent.

**Multi-label logic:**
- If a label scores ≥ 0.75, it's a strong label
- If it's between 0.60-0.75, it's secondary
- Max 3 labels per text (otherwise it gets messy)
- If nothing scores high enough → UNKNOWN

**Why this approach:**
- Entailment understands meaning, not just keywords. Critical when you have 5+ languages.
- NLI models naturally give you probabilities per label — perfect for multi-label
- Zero-shot means I can tweak labels without retraining (useful when categories evolved)
- Works across languages without translation

**Initial testing:** Classified the first 100 unique non-NaN rows. Distribution showed Understanding Mentor Background (26), Clarity of Business (20), Goal Clarity (19), Mindset (17), and others. This validated the approach.

**What we output:**
- `primary_label`: The main intent
- `all_labels`: Up to 3 semantically valid intents
- Raw scores for everything (transparency is good)

All the code is in `CJBSTaskCorrected.ipynb` if you want the details.

### Phase 4: Handling Mentor vs Mentee Separately

Even though mentors and mentees use the same labels, I process them separately. Why? Because the same words mean different things depending on who's saying them.

**The files we generate:**
- `person1_before_we_start__mentordf.csv` — mentors from person1
- `person1_before_we_start__menteedf.csv` — mentees from person1  
- `person2_before_we_start__mentordf.csv` — mentors from person2
- `person2_before_we_start__menteedf.csv` — mentees from person2

Right now they use the same model and templates, but keeping them separate makes it easier to add role-specific logic later if needed.

---

## How We Evaluate

### The Multi-Label Approach

For each text, we predict:
1. **Primary Label:** The main intent (highest probability)
2. **All Labels:** Other valid intents that also make sense (up to 3)

### How We Judge Success

A prediction is **correct** if the `primary_label` shows up in the `all_labels` list.

**Why this matters:** Real texts have multiple valid intents. Under strict single-label accuracy, you'd mark a model wrong even if it correctly identified one valid intent but missed another. That's not fair.

### Example

**Text:** "Prepare questions about your business goals"

**What we predict:**
- Primary: Practical Preparation and Readiness  
- All Labels: [Practical Preparation and Readiness, Goal Clarity and Direction Setting, Clarity of Business and Self-Understanding]

**Is it correct?** ✓ Yes, because the primary is in the all_labels list.

If we used strict single-label accuracy, this would be wrong because the text clearly has three intents. But our approach says: "You got the main one right, and you recognized the others exist — that's good enough."

---

## Results

On the test set, we get **100% correctness** under our entailment-based evaluation. That means the primary label always appears in the all_labels list — the model consistently identifies at least one valid intent correctly.

**Important caveat:** This doesn't mean we catch *every* valid intent. Just that the primary label is always semantically valid. For multi-intentional texts, that's actually pretty good.

**What this tells us:**
1. The model understands meaning, not just keywords
2. Multi-label classification is working as intended
3. It handles multiple languages well

Could it be better? Sure. But it's honest about uncertainty (via UNKNOWN) and doesn't force labels where they don't fit.

---

## The Messy Reality: Ambiguous Examples

Check out `ambiguous_examples.md` for 10 real examples that show why this is hard.

**Common patterns I noticed:**
1. Preparation + Goal Clarity show up together a lot
2. Mindset and active learning often go hand in hand
3. Business understanding usually needs information sharing
4. Technology requirements come with preparation tasks

Bottom line: overlap is the norm. Single-label would lose information. Multi-label captures reality.

---

## What's Not Perfect (Yet)

Honest limitations:

1. **No conversation context:** We're looking at isolated texts. Adding conversation history would probably help.

2. **Mentor/mentee templates are identical:** They're processed separately, but use the same hypothesis templates. Role-specific templates might work better.

3. **Our evaluation is lenient:** We only check if primary is in all_labels. A stricter eval would require identifying *all* valid intents.

4. **Thresholds are guesswork:** 0.75 and 0.60 were picked based on what looked good. Could probably optimize these.

5. **Max 3 labels might miss some:** Some texts legitimately have 4+ valid intents, but we cap at 3 to avoid chaos.

6. **Zero-shot only:** No fine-tuning yet. Could probably squeeze out more performance with task-specific training.

---

## What's Next

Things I'd like to try:

1. **Add more context:** Use conversation history, explicitly model role, maybe role-specific templates
2. **Fine-tune:** Could probably get better with task-specific training
3. **Better evaluation:** Build ground-truth multi-label annotations, do proper precision/recall per label
4. **More languages:** Test on more languages, maybe language-specific templates for low-resource ones
5. **Hierarchical structure:** Some intents are clearly related (e.g., different types of preparation). Could model that explicitly.
6. **Explainability:** Would be nice to explain *why* we assigned multiple labels, maybe highlight supporting text segments

---

## Repository Contents

### Notebooks

- **`Clustering_(1).ipynb`:** Exploratory intent discovery through unsupervised clustering
  - Documents the journey from raw text to discovered intent clusters
  - Includes cluster merging and interpretation

- **`CJBSTaskCorrected.ipynb`:** Entailment-based multi-label classification implementation
  - Zero-shot classification using multilingual NLI model
  - Multi-label decision logic and threshold parameters
  - Output generation for mentor and mentee datasets

### Output Files

- `person1_before_we_start__mentordf.csv`: Classified mentor texts (person1)
- `person1_before_we_start__menteedf.csv`: Classified mentee texts (person1)
- `person2_before_we_start__mentordf.csv`: Classified mentor texts (person2)
- `person2_before_we_start__menteedf.csv`: Classified mentee texts (person2)

Each CSV contains:
- `text`: Original text
- `primary_label`: Most representative intent
- `all_labels`: List of semantically valid intents
- `num_labels`: Count of labels assigned
- `all_raw_scores`: Raw probability scores for all candidate labels

### Documentation

- **`decision_log.md`:** Detailed reasoning behind all methodological decisions
  - Starting with SMAT categories (and why they failed)
  - Why clustering was performed to discover categories
  - Why single-label classification was insufficient
  - Why entailment-based multi-label approach was chosen
  - Why UNKNOWN class was introduced
  - Why mentor/mentee data were handled separately

- **`ambiguous_examples.md`:** 10 representative examples demonstrating semantic overlap
  - Shows why multi-label classification is necessary
  - Illustrates cases that would fail under strict single-label accuracy

**Note:** Original internship notes documenting the SMAT → clustering evolution and initial classification attempts are available [here](https://docs.google.com/document/d/1XTQfkDcxw6us7KKD7Ktgwr1qWFyNALkqIgSA50cA1rs/edit).

---

## Main Lessons

1. **Start with exploration:** Clustering showed me the overlap problem before I built anything. Data-first approach paid off.

2. **Multi-label is necessary:** Real communication is multi-intentional. Single-label throws away information.

3. **Semantics > Keywords:** Especially with multiple languages, you need actual understanding.

4. **Be honest about uncertainty:** UNKNOWN is better than forcing wrong labels.

5. **Role changes everything:** Same words, different meanings. Process separately.

6. **Evaluation should match reality:** Our criterion (primary in all_labels) reflects how people actually communicate, not some idealized single-intent world.

---

## References

- [XLM-RoBERTa: Unsupervised Cross-lingual Representation Learning at Scale](https://arxiv.org/abs/1911.02116)
- [Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks](https://arxiv.org/abs/1908.10084)
- [UMAP: Uniform Manifold Approximation and Projection for Dimension Reduction](https://arxiv.org/abs/1802.03426)
- [HDBSCAN: Hierarchical density based clustering](https://hdbscan.readthedocs.io/)

---

## Citation

If you use this work, please cite:

```bibtex
@misc{mentor_intent_classification,
  title={Multi-Label Intent Classification for Mentor-Mentee Conversations},
  author={[Your Name]},
  year={2024},
  howpublished={\url{https://github.com/[your-repo]/MentorMatching}}
}
```

---

*This project started as exploration and evolved into something that actually works for real-world mentor-mentee communication. Check out the decision log and notebooks if you want to see how we got here.*

