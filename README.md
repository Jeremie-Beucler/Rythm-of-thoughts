# Rythms of Thought: Exploring the Temporal Trajectories of Deliberation in Reasoning

This document describes the analysis of deliberation function trajectories extracted from verbal reasoning data. Using LLM-based scoring of reasoning segments (chunks), we model how different cognitive functions unfold over time within transcriptions. Here, we use the term trajectory to refer to the evolving activation levels of distinct cognitive functions as reasoning unfolds over time.

---

## Introduction and Motivation

Understanding how people deliberate when solving reasoning problems is a key question in cognitive psychology. Recent theoretical work by [De Neys (2025)](https://files.osf.io/v1/resources/f7436_v1/providers/osfstorage/67a6196095008bb4100c5869?action=download&direct&version=1) has highlighted critical gaps in our knowledge of the dynamics of deliberation.

Most traditional models treat deliberation as a monolithic process, typically focusing only on response control — the inhibition of an intuitive but incorrect answer. However, De Neys proposes that deliberation involves multiple distinct cognitive functions:

- *Response Control*: Inhibiting or resisting an intuitive or prepotent response.
- *Response Generation*: Actively searching for new answers, alternatives, or hypotheses.
- *Response Justification*: Providing reasons, arguments, or explanations for a response.
- *Response Regulation*: Monitoring, evaluating, or allocating effort in the reasoning process.

We use this broad theoretical taxonomy as the basis for our analyses. Crucially, De Neys raises an open question: when people engage in deliberation, do these functions follow a specific activation order or sequence? Or is their engagement driven by situational factors, such as the level of uncertainty experienced by the reasoner?

Currently, there is no direct empirical evidence to address this question. Exploring the temporal dynamics of deliberation functions could provide valuable insights into how people engage in reflective thinking. Our work aims to contribute to this effort by using language models to automatically segment and score verbal reasoning data. This approach allows us to track, at a fine-grained level, how different deliberation functions unfold over time within participants' verbalizations.

---

## Data and Audio Transcription

The data come from Study 2 of [Byrd et al. (2023)](https://www.mdpi.com/2079-3200/11/4/76), conducted online with 102 adult participants from Prolific. Participants completed a verbal version of the Cognitive Reflection Test (vCRT) by thinking aloud as they solved each problem.

Participants' verbalizations were recorded as audio files. We subsequently transcribed using a state-of-the-art automatic speech recognition model: `Whisper Large v3 Turbo` (because the automatic transcription already present in the dataset was of poor quality).

---

## Chunking and Scoring of Transcriptions

We used the `meta-llama/Llama-3.3-70B-Instruct` model served via the HuggingFace Inference API.

The generation parameters were set to:

- `temperature = 1`
- `top_p = 1`

These settings encouraged the model to generate coherent but flexible segmentations of participants' transcriptions into distinct ideas or reasoning steps.

### Chunking Procedure

The model was prompted to segment each transcription into coherent units corresponding to distinct ideas, reasoning steps, or meta-comments. Each chunk represents a semantically and functionally coherent unit of verbalization, corresponding to a single cognitive step or thought.

The chunking prompt was the following:

```text
You are an expert in cognitive psychology and verbal protocol analysis.

You are given a transcription of a participant thinking aloud while solving a problem.

Your task is to segment this transcription into meaningful chunks.

A chunk should correspond to a coherent idea, thought, or step in the participant's reasoning or verbal expression — including hesitations, repetitions, or meta-comments. The goal is not only to segment explicit reasoning steps but to preserve the full structure of the verbalization.

Guidelines:
- Do not remove or suppress any part of the original text.
- Do not segment based on arbitrary word count or length.
- Split only when the participant clearly moves to another distinct thought, idea, or reasoning step (e.g., shifting from generating an answer to justifying it, or reflecting on their uncertainty).
- Be conservative in splitting: avoid unnecessary fragmentation.
- Preserve the original wording exactly in each chunk.

Provide your output strictly in the following structure:

Chunk 1:
[exact text of chunk 1]

Chunk 2:
[exact text of chunk 2]

Chunk 3:
[exact text of chunk 3]

Be exhaustive.
```

### Scoring Procedure

Each chunk was then automatically scored by the LLM for the intensity of four deliberation functions:

- Response Control
- Response Generation
- Response Justification
- Response Regulation

Scores range from 0 (not at all present) to 100 (very strongly present). The scoring prompt was the following:

```text
You are an expert in cognitive psychology.

You are given a short chunk of a participant's think-aloud transcription during a reasoning task.

Your task is to rate how strongly this chunk expresses each of the following deliberation functions.

Definitions of the deliberation functions:

- Response Control: Inhibiting, rejecting, or resisting an obvious or intuitive response that first comes to mind. Typical signs include expressions of doubt, suppression of initial answers, hesitation, or stopping oneself from blurting out an impulsive response.

- Response Generation: Actively searching for new possible answers, alternatives, or hypotheses. This includes exploring options, mentally simulating scenarios, considering possibilities, or applying step-by-step logical reasoning.

- Response Justification: Providing explicit reasons, arguments, or explanations to support a response that is currently being considered (whether intuitive or not). This includes defending a choice, explaining why an answer makes sense, or making an argument.

- Response Regulation: Reflecting on one's own reasoning process, monitoring one's performance, allocating effort, expressing uncertainty, or deciding whether to continue thinking or stop. This includes metacognitive monitoring or strategic regulation of effort.

Important Instructions:

- Each score should reflect the extent to which the chunk expresses the function (even partially), using a continuous scale from 0 (not at all present) to 100 (very strongly present).
- These functions are not mutually exclusive — a chunk may score highly on multiple functions if they co-occur.
- If the chunk contains no trace of any of these 4 functions, assign 0 to all functions. This is perfectly acceptable.
- If the chunk expresses a completely different kind of function (not captured by the 4 above), mention it below using a very broad and generic label (e.g., "Reading Aloud", "Task Repetition", "Social Comment", etc.). This should only happen rarely and only if clearly justified by the content of the chunk.
- Be conservative: If you are unsure whether a function is expressed, prefer giving a low score (0-10).
- Do not explain or justify the scores unless the chunk clearly expresses a different kind of function.

Output strictly in this structure (and nothing else):

Response Control: [score between 0 and 100]
Response Generation: [score between 0 and 100]
Response Justification: [score between 0 and 100]
Response Regulation: [score between 0 and 100]

[Optional broad label for a different function — only if clearly needed]
```

## LLM Function Scores Correlation and Validation

We examined the relationships between the four deliberation functions scored by the LLM: Control, Generation, Justification, and Regulation. This analysis allows us to check whether the functions capture distinct aspects of the reasoning process, as theoretically expected.

We computed a single correlation matrix including both the LLM function scores and hand-coded indicators of reflective thinking. These hand-coded variables correspond to two key components of reflection:

- Deliberateness: Whether the participant verbally reconsidered their initial response (named *reconsidered _initial_resp* in the Figure); should correlate with Control and Regulation.
- Consciousness: Whether the participant verbalized reasons for or against a response (named *verbalized_reasons* in the Figure); should correlate with Justification the most.

These ratings were assigned based on the procedure described by Byrd et al., with moderate inter-rater agreement.

![Correlation heatmap between LLM function scores and hand-coded variables.](./Output/deliberation_function_handscored_variables_correlation_heatmap_handscored.png)

*Figure 1. Correlation heatmap between LLM function scores and hand-coded variables from Byrd et al. (2023). This joint analysis allows us to verify whether LLM-derived functions capture distinct constructs while aligning with human-rated indicators. Only significant correlations (p < .05) are shown.*

---

## Distribution of Number of Chunks

We can have a look at the distribution of the number of reasoning segments (chunks) per question.

### Overall Distribution

![Histogram of the number of chunks per question.](./Output/histogram_n_chunks_overall.png)

*Figure 2. Histogram showing the distribution of the number of reasoning chunks per question.*

### By Accuracy

![Histogram of the number of chunks per question by accuracy.](./Output/histogram_n_chunks_by_response.png)

*Figure 3. Histogram of the number of chunks per question by response type.*

### By Lure Consideration

Lure consideration corresponds to whether participants reported having thought of the intuitive but incorrect lure response during their reasoning process. Following each question, participants were explicitly asked whether the lure answer occurred to them (e.g., "Have you thought at any point that '1st place' could be the answer?").

![Histogram of the number of chunks per question by lure consideration.](./Output/histogram_n_chunks_by_lure_consideration.png)

*Figure 4. Histogram of the number of chunks per question by lure consideration.*

### By Accuracy and Lure Consideration

![Histogram of the number of chunks per question by response type and lure consideration.](./Output/histogram_n_chunks_by_response_lure_consideration.png)

*Figure 5. Histogram of the number of chunks per question by accuracy and lure consideration.*

---

## Trajectories of Deliberation Functions Over Time

We model the dynamics of each deliberation function over normalized time within transcriptions.

Normalized time corresponds to the relative position of each reasoning chunk within a given transcription. This measure captures the sequential unfolding of thoughts, independent of the actual duration or word length of each chunk.

Technically, the normalized position of a chunk is computed as:

```normalized_position = (chunk_id - 1) / (n_chunks - 1)```

where chunk_id is the sequential number of the chunk within the transcription, and n_chunks is the total number of chunks for that participant and question.

This transformation ensures that the first chunk always has a normalized position of 0, the last chunk a position of 1, and intermediate chunks are evenly distributed between them. Importantly, this is not a measure of real time or duration — it reflects the ordinal progression of reasoning steps.

Thus, normalized time provides a positional representation of thought dynamics, allowing us to compare the trajectory of deliberation functions across responses of varying length.

Since we are interested in trajectory, we exclude the responses with only one chunk identified by the LLM (which were quite rare).

## Chunk Length by Dominant Function (Control Analysis)

Reasoning chunks correspond to meaningful units of thought within participants' verbalizations. However, the length of these chunks may vary depending on their dominant deliberation function.

To explore this, we identified the dominant function of each chunk based on the highest LLM-assigned score (Control, Generation, Justification, or Regulation). In case of ties, the chunk was excluded from this analysis.

We then examined whether chunk length (measured by word count) differed across functions. Intuitively, chunks associated with Response Generation might be longer, as participants explore and verbalize possible answers or reasoning steps.

![Boxplot of word count per chunk by dominant function.](./Output/boxplot_word_count.png)

*Figure 7. Distribution of word count per chunk by dominant function. Points represent individual chunks; yellow markers indicate the mean word count per function.*

### Overall Trajectory

First, let's have a first look at the trajectories using simple LOESS smoothing over the mean function scores. Note that the GAMM results below are more robust, this is just a quick first look.

![Mean trajectory of deliberation functions over normalized time.](./Output/overall_trajectory_loess_preliminary.png)

*Figure 8. Mean trajectory of each deliberation function across all responses (smoothed using a local regression smoother, LOESS).*

### Trajectory by Accuracy

![Mean trajectory of deliberation functions by response type (biased vs unbiased).](./Output/trajectory_by_response_loess_preliminary.png)

*Figure 9. Mean trajectory of deliberation functions by accuracy (smoothed using a local regression smoother, LOESS).*

---

## Difference in Trajectories Between Response Types

We computed the difference in function trajectories between correct and incorrect responses. Here a positive difference indicates that the function is more activated in correct responses than in incorrect ones.

### Overall Difference

![Difference in trajectories between unbiased and biased responses for each deliberation function.](./Output/difference_trajectory_unbiased_biased_faceted_loess_preliminary.png)

*Figure 10. Difference in trajectories (correct minus incorrect) across deliberation functions (smoothed using a local regression smoother, LOESS).*

---

## GAMM-Predicted Trajectories

We fitted generalized additive mixed models (GAMMs) to predict function trajectories.

GAMMs offer a flexible statistical framework that allows us to model the potentially non-linear dynamics of deliberation functions over normalized time. They are particularly well-suited for this analysis because they can capture complex trajectory shapes without making strong assumptions about their form.

In addition, GAMMs include random intercepts for both participants and questions. This accounts for individual variability and question-specific effects, ensuring that the estimated trajectories reflect generalizable patterns rather than idiosyncrasies of the data.

Given the noisy nature of our LLM-annotated data, GAMMs provide an important advantage: they smooth the trajectories while preserving meaningful signal. This smoothing facilitates interpretation, allowing us to assess whether the function trajectories are theoretically plausible and robust across participants.

**Warning: I am not very used to GAM(M); so the results may not be perfect yet.**

Below is the overall trajectory of each deliberation function (across all responses), as predicted by the GAMM.

![GAM-predicted trajectories for each deliberation function. Confidence intervals represent 95% CI.](./Output/gam_trajectory_functions_overall_trajectory.png)

*Figure 11. GAMM-predicted trajectories for each deliberation function. Shaded areas represent 95% confidence intervals.*

---

## Pairwise Differences Between Functions (GAMM)

We computed pairwise differences between function trajectories based on GAM predictions (across all responses).

![Pairwise differences between function trajectories. Grey areas indicate significant differences after FDR correction.](./Output/gam_pairwise_functions_difference_trajectory.png)

*Figure 12. Pairwise differences between deliberation function trajectories. Shaded grey areas indicate significant differences (p < .05, FDR corrected).*

---

## Trajectories by Accuracy and Function (GAMM)

Next we can look at the GAMM-predicted trajectories by accuracy, to compare correct and incorrect responses.

![GAM-predicted trajectories by response type (biased vs unbiased) and deliberation function.](./Output/gam_trajectory_per_response_and_function.png)

*Figure 13. GAMM-predicted trajectories by accuracy and deliberation function. Shaded grey areas indicate significant differences (p < .05, FDR corrected).*

---

## Conclusions

**Overall Trajectory**

- Generation happens early
- Control happens later
- Justification happens later too
- Regulation stays low overall — increasing slightly at the end (likely to check or monitor the final answer?)

**Correct vs. Incorrect Responses Comparison**

- Generation happens even earlier for incorrect responses — participants jump quickly to an answer without extensive search. Correct responders take more time to generate options.
- Control shows the clearest difference — correct responders show much more Control throughout their reasoning, while incorrect responders stay low and quite flat.
- Justification is higher (and later) for correct responders — they verbalize more reasons for their choices.
- Regulation doesn’t differ much — it stays low for both groups, increasing slightly at the end.

---

**Byrd et al. Results (2023)**

Note that these findings — although more precise in their depiction of temporal dynamics and in their exploration of deliberation subfunctions — also align with the results of Byrd et al. (2023): correct responses typically involve more deliberate and reflective processing, while incorrect responses often rely on fast, intuitive answers without reconsideration.

**Methodological Implications**

Beyond theoretical insights, our approach offers methodological gains. The use of LLM-based chunking and scoring allows for a fine-grained, automated, and scalable analysis of deliberation dynamics in verbal (audio or text) reasoning data. This method opens the door to modeling individual trajectories of reasoning functions in larger and more naturalistic datasets.

---

## Next Steps

Building on this first, there are several next possible steps:

- **Looking at the slopes instead of the differences**: Beyond examining absolute differences between functions, we could compare their slopes to assess how quickly each function increases or decreases over the course of reasoning. This would provide additional insight into the dynamics and timing of function activation.

- **Validation with human ratings**: Systematically compare the LLM-derived function scores with human-coded ratings on a new set of transcriptions to further validate the approach.

- **Bottom-up taxonomy of deliberation functions**: Use the LLM not only for scoring but also to generate a more data-driven and fine-grained taxonomy of reasoning functions based on participants' transcriptions.

- **Trajectories through question order**: Examine the evolution of deliberation function trajectories across the sequence of questions within the task (but then, issues if order not randomized because question will be a confounder).

- **Collecting new data or identifying similar datasets**: Apply the method to other think-aloud datasets or collect new data specifically designed to probe deliberation dynamics.

- **Participant profiling**: Investigate individual profiles or clusters of participants based on their trajectory patterns, identifying distinct reasoning styles or strategies.

- **Analysis of specific subcases**:
  - Correct "intuitive" responses: Cases where the correct answer is generated quickly without much evidence of control or justification.
  - Incorrect "deliberate" responses: Cases where participants engage in effortful reasoning but still arrive at an incorrect answer.

- **Use better LLM models such as GPTs, and improve prompt (e.g., give examples)**

- **Adopt a more bottom-up, unsupervised approach to classify the functions**: Instead of using a pre-defined taxonomy, we could use clustering or topic modeling techniques to identify distinct reasoning functions based on the LLM-generated chunks. This would allow for a more data-driven exploration of the functions involved in deliberation (complementary).

---

## Complementary Analyses

### Trajectories by Lure Consideration (GAMM)

![GAM-predicted trajectories by lure consideration (lure considered vs lure non-considered; self-reported) and response type.](./Output/gam_trajectory_lure_vs_no_lure_per_response_and_function.png)

*Figure 14. GAMM-predicted trajectories by lure consideration and accuracy. Shaded grey areas indicate significant differences (p < .05, FDR corrected).*

---

### Trajectories by Familiarity (GAMM)

The participants also indicated whether they were familiar with the vCRT material. We can look at the trajectories by familiarity (familiar vs unfamiliar).

![GAM-predicted trajectories by participant familiarity with the test.](./Output/gam_trajectory_familiar_vs_unfamiliar_per_function.png)

*Figure 15. GAMM-predicted trajectories by participant familiarity with the vCRT material. Shaded grey areas indicate significant differences (p < .05, FDR corrected).*