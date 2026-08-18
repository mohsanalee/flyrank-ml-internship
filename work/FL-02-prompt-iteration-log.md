# FL-02 — Prompt Iteration Log

## Task

Use prompting techniques to improve an ML/content-prioritization task from my FL-01 audit.

### FL-01 task

> Use content and performance signals to rank pages by priority so an editor can decide which pages to review first.

The task comes from my content-refresh/editorial-prioritization lane. The starter dataset contains 30,000 pseudonymized content items and 44 columns. The data includes trailing-90-day performance signals and a `trend_direction` field. In the initial data check, 16,262 items had a recorded `"down"` trend.

The decision is not to automatically declare that a page needs a refresh. The decision is to help an editor prioritize limited review time.

---

# Prompt Ladder

## Run 0 — Naive baseline

### Technique
Naive one-line prompt.

### Prompt
> Help me rank content that needs improvement.

### Representative output
The model misunderstood the task and responded about my diabetic retinopathy project. It ranked evaluation/documentation, frontend/inference workflow, Grad-CAM, model improvement, and deployment polish.

### What changed in the prompt?
Nothing. This was the deliberately weak baseline.

### What improved in the output?
Nothing useful for the actual task. The output was polished but addressed the wrong project.

### What still failed?
The model had no role, context, dataset information, or decision definition.

### What would I try next?
Add a clear role so the model knows who it is helping.

---

## Run 1 — Role assignment

### Technique
**Role assignment**

### Prompt
> You are an ML/data analyst helping a content editor prioritize pages for review.
>
> Help me rank content that needs improvement.

### Representative output
The model moved to the correct editorial problem and asked for page-level information. It proposed considering traffic, business value, engagement, search metrics, and content quality, and suggested a ranked list with a priority score, reason, and recommended action.

### What changed in the prompt?
I added only a role: an ML/data analyst helping a content editor prioritize pages.

### What improved in the output?
The model stopped discussing diabetic retinopathy and correctly focused on content-page prioritization.

### What still failed?
It assumed fields such as conversion rate, bounce rate, and last-updated date that were not confirmed in the starter dataset.

### What would I try next?
Add the real task context and motivation.

---

## Run 2 — Context and motivation

### Technique
**Context and motivation**

### Prompt
> You are an ML/data analyst helping a content editor prioritize pages for review.
>
> Help me rank content that needs improvement.
>
> The task is part of a content-refresh analysis using a starter dataset of 30,000 pseudonymized content items with 44 columns and trailing-90-day content and performance signals. The editor has limited time, so the goal is to prioritize which pages should be reviewed first rather than treating every page equally. The dataset shows meaningful variation in performance and trend direction, including 16,262 items with a recorded "down" trend.
>
> The ranking should support editorial review and should not be treated as proof that a page definitely needs a refresh.

### Representative output
The model proposed a review-priority score combining performance impact, decline signals, and data confidence. It also proposed review tiers and emphasized that the ranking should identify pages worth investigating rather than automatically label pages as needing a refresh.

### What changed in the prompt?
I added the real dataset/task context and the motivation: editors have limited time and need to prioritize pages for review.

### What improved in the output?
The response became much more aligned with the actual editorial decision.

### What still failed?
It introduced assumptions such as conversions, content age, and engagement deterioration, and suggested arbitrary example weights without evidence from the dataset.

### What would I try next?
Add few-shot examples showing how a good prioritization should behave.

---

## Run 3 — Few-shot examples

### Technique
**Few-shot examples**

### Prompt
The Run 2 prompt was kept and these examples were added:

> Example 1: A page with strong recent performance but a clear negative trend should receive higher priority because a decline on a high-impact page may represent a larger editorial opportunity.
>
> Example 2: A page with very low traffic and a negative trend should not automatically receive the highest priority because the potential impact may be small.
>
> Example 3: A page with missing or uncertain signals should be flagged for caution rather than given a confident high-priority recommendation.
>
> Use these examples as guidance for the reasoning, not as fixed rules.

### Representative output
The model became more explicit about impact, decline, and confidence. It consistently distinguished high-impact declines from low-impact declines.

### What changed in the prompt?
I added three examples showing the desired prioritization behavior.

### What improved in the output?
The examples made the expected reasoning more concrete and the explanations clearer.

### What still failed?
The model continued to introduce unconfirmed signals such as conversions, content age, and engagement.

### What would I try next?
Add a strict output structure and explicitly tell the model not to treat unconfirmed fields as available data.

---

## Run 4 — Output structure

### Technique
**Output structure**

### Prompt
The Run 3 prompt was kept and these instructions were added:

> Return your answer in exactly these four sections:
>
> 1. Recommended ranking approach — maximum 4 bullets.
> 2. Priority logic — a small table with Signal, Higher priority when, and Caution.
> 3. Example ranked output — exactly 5 example rows with Rank, Content ID, Priority, Reason, and Confidence.
> 4. Limitations — maximum 3 bullets.
>
> Do not invent dataset columns. If a signal is not confirmed to exist in the dataset, label it as a possible future feature rather than using it as if it were available.

### Representative output
The model produced four clearly separated sections, a priority-logic table, five example rows, and limitations. It treated content freshness as a possible future feature instead of assuming it existed.

### What changed in the prompt?
I added only the required output structure and a constraint against inventing dataset columns.

### What improved in the output?
The answer became much easier to read and use.

### What still failed?
The example Content IDs were illustrative rather than calculated from the real 30,000-row dataset.

### What would I try next?
Add step decomposition.

---

## Run 5 — Step decomposition

### Technique
**Step decomposition**

### Prompt
The Run 4 prompt was kept and this instruction was added:

> Before giving the final recommendation, work through these steps in order:
> 1. Identify which available signals are relevant to editorial prioritization.
> 2. Separate impact signals from decline/risk signals.
> 3. Decide how uncertainty or missing data should affect the ranking.
> 4. Check that the proposed approach supports the editor's decision rather than claiming causality.
> 5. Then produce the final four-section answer.

### Representative output
The model explicitly separated impact signals from decline/risk signals, treated missing information as a confidence issue, and maintained the decision-support framing.

### What changed in the prompt?
I added only the five-step reasoning/decomposition sequence.

### What improved in the output?
The response became more disciplined about impact, decline/risk, uncertainty, and non-causal decision support.

### What still failed?
The example Content IDs remained illustrative rather than being calculated from the actual dataset. Some possible signals were still discussed as future features.

### What would I try next?
Run the same final prompt on another model and compare the results.

---

# Cross-model comparison

The final Run 5 prompt was tested on both ChatGPT and Claude.

## ChatGPT

**Tone:** Practical, structured, and easy to follow.

**Accuracy:** Generally good, but it was more willing to introduce assumptions about available performance or future signals.

**Structure:** Followed the requested four-section format clearly.

**Failure points:** The example Content IDs were illustrative rather than actual results from the dataset. Some possible signals were discussed even though they were not confirmed as available columns.

## Claude

**Tone:** More cautious and analytical.

**Accuracy:** Stronger dataset discipline. Claude explicitly stated that only a performance/impact measure and the `trend_direction` field were confirmed from the prompt and avoided assuming fields such as bounce rate, backlinks, CTR, or content age were present.

**Structure:** Followed the required structure and explicitly worked through the five decomposition steps before the final sections.

**Failure points:** Claude also used illustrative Content IDs rather than calculating a real ranking. Its performance signal was described from the prompt rather than tied to a specific confirmed column name.

## Overall comparison

Claude was stronger on **accuracy, uncertainty handling, and avoiding unsupported assumptions**.

ChatGPT was stronger on **concise practical presentation and editorial framing**.

Both models shared the same major limitation: the example ranked Content IDs were illustrative, not an actual ranking computed from the 30,000-row dataset.

---

# Final reusable prompt template

> You are an ML/data analyst helping a **[role/person]** make **[decision]**.
>
> The task is **[describe the real ML/analysis task]** using **[dataset description]**. The person making the decision has **[constraint/motivation]**, so the output should help them **[specific action]**.
>
> The goal is decision-support, not causal proof. Do not claim that the output proves **[what cannot be claimed]**.
>
> Use these examples as guidance:
>
> - Example 1: **[good behavior]**
> - Example 2: **[important edge case]**
> - Example 3: **[uncertain/missing-data case]**
>
> Return the answer in this structure:
>
> 1. **Recommended approach** — maximum [N] bullets.
> 2. **Decision logic** — table with [required columns].
> 3. **Example output** — exactly [N] rows with [required fields].
> 4. **Limitations** — maximum [N] bullets.
>
> Do not invent dataset columns or unavailable information. If a useful signal is not confirmed to exist, label it as a possible future feature.
>
> Before the final answer:
> 1. Identify the relevant available signals.
> 2. Separate **[signal category 1]** from **[signal category 2]**.
> 3. Explain how missing or uncertain data should affect the output.
> 4. Check that the recommendation supports the stated decision without making causal claims.
> 5. Produce the final structured answer.

---

# Self-check

- [x] Six runs completed: baseline + five techniques
- [x] Each iteration uses one named technique
- [x] Actual output differences are documented
- [x] Honest failures are documented
- [x] Final prompt tested on ChatGPT and Claude
- [x] Cross-model comparison is specific
- [x] Final prompt is reusable by a stranger
- [x] Task comes from the real FL-01 content-prioritization work

**Note:** Example ranked Content IDs in the model outputs are illustrative and were not calculated from the actual starter CSV. They are included as representative output examples, as allowed by the assignment.
