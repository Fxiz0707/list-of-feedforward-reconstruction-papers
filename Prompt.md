# Paper Capability Analysis

You will analyze a folder of computer-vision research papers and produce
a capability summary for each one. The output is per-paper prose — a
downstream model will assemble tables later, so do not produce a table
yourself.

Produce **one Markdown file per paper**. Do not concatenate papers into
a single document. Each file's filename should be based on the paper's
short identifier.

## Scope

This analysis is primarily about methods that output 3D point clouds
(or equivalent dense 3D representations) for a scene. Q1 is the gating
question. If a paper does not produce a point cloud output (e.g. a pure
2D-tracking, pose-only, or image-synthesis method), say so prominently
in the summary so downstream filtering is easy; the remaining questions
can still be answered where applicable.

## Orchestration

For each paper in the folder, launch a chain of three agents. Each agent
operates on a single paper; do not batch papers together.

1. **Analyzer agent.** Reads the full paper and produces the initial
   capability summary using the reading approach, questions, and
   disambiguation rules below.

2. **Evaluator agent.** Receives the analyzer's output along with the
   same paper. Independently re-reads the paper and audits the analyzer's
   summary: checks each verdict against the evidence, flags weak or wrong
   answers, and notes missing evidence.

3. **Reviewer agent.** Receives the original paper, analyzer summary,
   and evaluator critique. Produces the final summary by incorporating
   valid evaluator findings, defending original answers where appropriate,
   and tightening justifications.

Run the three agents strictly in sequence per paper:
analyzer → evaluator → reviewer.

Different papers can be processed in parallel, but within a single paper
the chain is sequential.

## Reading approach

Each agent must read the **full paper** before producing its output,
including appendix / supplementary material if present.

A reasonable order is:

1. Skim the abstract and teaser figure.
2. Read the method section carefully to determine what the model itself
   outputs versus what is post-processed by external tools.
3. Read experiments / evaluation to see what inputs and modalities are
   actually demonstrated.
4. Read implementation details, training setup, and limitations.
5. Read appendix / supplementary if present.
6. Re-skim anything uncertain before finalizing answers.

## Questions to answer

Every question must include a justification with a section or page
reference.

For most questions, the verdict is one of:
**Yes / No / Partial / Unclear / Pipeline**.
Q3 and Q10 have additional vocabulary noted inline.

1. **3D reconstruction** — Does the method produce a point cloud or
   equivalent dense 3D representation per scene?

2. **Dense reconstruction** — Does the model produce dense
   reconstruction, as opposed to sparse keypoint-style 3D only?

3. **Reference frame** — What coordinate frame do outputs live in, and
   how flexible is the choice? Vocabulary:
   - **Selectable** — the user can designate any frame as the world /
     anchor; the method natively supports both world-space and
     egocentric output depending on which frame is chosen.
   - **Fixed world** — outputs are expressed in a single consistent
     global frame, but the anchor is hard-coded (e.g. always the first
     frame) and cannot be freely reassigned.
   - **Egocentric** — outputs are in each frame's own camera coordinate
     system with no globally consistent frame across frames/views.
   - **N/A** — the method produces no 3D geometry (Q1 = No), or is a
     single-image method with no temporal/multi-view component.

4. **Metric scale** — Are outputs at true metric scale, as opposed to
   only up-to-scale / arbitrary units?

5. **Point tracking** — Does it establish correspondences between the
   points / point clouds it predicts?

6. **3D tracks** — Does it output 3D point trajectories?

7. **2D tracks** — Does it output 2D point tracks? If Q6 = Yes and Q12 =
   Yes, answer Yes because 2D tracks are recoverable by projection.

8. **Occlusion / visibility prediction** — Does the model output
   per-point or per-track occlusion or visibility flags?

9. **Multiple views** — Does it handle simultaneous multi-view /
   multi-camera input? Multiple views means multiple simultaneous
   views/cameras of the scene, not sequential frames in a monocular
   video. Do not count a monocular video, moving-camera video,
   unordered image collection, or temporal sequence as multi-view unless
   the method accepts multiple simultaneous camera views/images as input
   in the relevant sense. If Yes, state whether the method accepts an
   arbitrary number of simultaneous views or is restricted to a
   fixed-size input.

10. **Global context vs. pairwise vs. fixed-window** — How does the
    model aggregate information across frames or views at inference
    time? Vocabulary: Global / Pairwise / Fixed-window / Hybrid / N/A.

11. **No GT extrinsics required at inference** — Can the model run at
    inference time without ground-truth camera extrinsics being supplied
    as input?

12. **Extrinsics as output** — Does the model output per-frame camera
    extrinsics as part of its own predictions?

13. **No GT intrinsics required at inference** — Can the model run at
    inference time without ground-truth camera intrinsics being supplied
    as input?

14. **Intrinsics as output** — Does the model output camera intrinsics?

15. **Moving cameras** — Does it handle non-static / moving cameras?

16. **Dynamic content** — Does it handle moving / non-rigid scene
    content?

17. **Feed-forward inference** — Is inference a single forward pass of
    the trained network, with no per-scene optimization, test-time
    training, or iterative refinement loop?

## Disambiguation rules

Judge the method itself, not external systems built around it.

Use **Pipeline** only if the paper itself describes a pipeline combining
its model with other components and evaluates that pipeline on the task.

Distinguish moving cameras from dynamic content.

Keep “GT extrinsics required” separate from “extrinsics as output”.
Keep “GT intrinsics required” separate from “intrinsics as output”.

If a capability is shown only in supplementary or as a small side
experiment, answer Partial.

If genuinely ambiguous, answer Unclear.

## Output format

Each paper's output is a standalone Markdown file:

**Paper:** <title>, <venue year>, <short identifier>  
**Summary:** 2–3 sentences describing what the model does, its input
modality, and its training/inference regime. If Q1 = No, state this
prominently.

Then answer each of the 17 questions in order as a numbered list, with
the question label, verdict word, and justification.

Example:

1. **3D reconstruction:** Yes. <justification with section/page ref>
2. **Dense reconstruction:** Partial. <justification with section/page ref>

End with:

**Notes:** caveats that do not fit the questions — input modality,
supervision regime, limitations, or other downstream-relevant details.