**Paper:** MV-TAP: Tracking Any Point in Multi-View Videos, arXiv Dec 2025, MV-TAP
**Summary:** MV-TAP is a feed-forward multi-view point tracker that takes synchronized multi-view RGB videos, a set of N query points specified per view, and known camera intrinsics and extrinsics as input, and predicts 2D pixel-space trajectories and per-point occlusion states across all views and frames. The model uses a CNN correlation encoder, Plücker-coordinate camera embeddings, and a Multi-View Spatio-Temporal Transformer with interleaved temporal, spatial, and view attention, trained on a new large-scale synthetic multi-view Kubric dataset. This paper does not produce any 3D reconstruction; it operates entirely in 2D camera pixel space.

1. **3D reconstruction:** No. MV-TAP outputs 2D pixel-space trajectories T ∈ R^{V×T×N×2} and occlusion states O ∈ R^{V×T×N×1}. No point cloud, depth map, mesh, or any 3D representation is produced. Table 1 (p. 5) explicitly labels MV-TAP as "2D" target, "Camera" space, with no depth input or output, contrasting it with MVTracker [31] which operates in 3D world space.

2. **Dense reconstruction:** No. The method tracks a finite set of N user-specified query points, not all pixels. Experiments evaluate up to 500 query points (Table 5, p. 7), but this is sparse/semi-dense point correspondence, not per-pixel dense scene reconstruction. (Sec. 3.2, p. 3; Sec. 4.3, p. 7)

3. **Reference frame:** N/A. MV-TAP is a 2D multi-view point tracker (Q1 = No); it produces no 3D geometry output.

4. **Metric scale:** No. All outputs are 2D pixel coordinates. No metric 3D information is produced or implied. Plücker embeddings are explicitly normalized to unit length for scale invariance. (Sec. 3.4, p. 4: "The direction d is normalized to unit length to ensure a scale-invariant representation.")

5. **Point tracking:** Yes. Core capability. Given N query points each specified by (view, frame, pixel location), MV-TAP establishes their correspondences across all T frames and V views through iterative refinement of track hypotheses. (Abstract, p. 1; Sec. 3.2, p. 3; Sec. 3.3, 3.6)

6. **3D tracks:** No. Output trajectories are 2D pixel coordinates. MV-TAP deliberately avoids 3D lifting to sidestep reprojection errors, positioning itself in contrast to MVTracker [31] which requires depth input for 3D world-space tracking. (Abstract, p. 1; Sec. 2, p. 2; Sec. 3.2, p. 3; Table 1, p. 5)

7. **2D tracks:** Yes. Primary output: 2D pixel-space trajectories T ∈ R^{V×T×N×2} for N query points across T frames and V views. Evaluated on standard TAP-Vid metrics (AJ, < δ_avg, OA). (Sec. 3.2, p. 3; Table 1, p. 5)

8. **Occlusion / visibility prediction:** Yes. The model jointly predicts per-point occlusion states O ∈ R^{V×T×N×1} indicating visibility/occlusion for every point at every frame in every view, refined iteratively alongside trajectories, trained with Binary Cross-Entropy loss. Evaluated via Occlusion Accuracy (OA) metric. (Sec. 3.2, p. 3; Sec. 3.6, p. 5; Sec. 3.7, Eq. 10, p. 5; Table 1, p. 5)

9. **Multiple views:** Yes — arbitrary number of views. Input is V synchronized multi-view RGB videos. MV-TAP is trained with 1 to 4 randomly selected views but the attention-based architecture generalizes at inference to an arbitrary number of views; evaluated with 2, 4, 6, and 8 views in Table 2 (p. 6). Sec. 4.3 (p. 7) states: "MV-TAP can handle an arbitrary number of views even larger than 4." View count is not fixed.

10. **Global context vs. pairwise vs. fixed-window:** Global. The Multi-View Spatio-Temporal Transformer applies full-sequence temporal attention over all T frames (Eq. 4), full-set spatial attention over all N query points (Eq. 5), and full-set view attention over all V views (Eq. 6) — all without windowing or pairwise restriction. The triangulation "Window" variants in Appendix C are auxiliary post-processing experiments, not part of the core model. (Sec. 3.5, pp. 4–5)

11. **No GT extrinsics required at inference:** No. Ground-truth camera extrinsics (R_{v,t} ∈ SO(3) and t_{v,t} ∈ R^3, per view and per frame) are required inputs to compute the Plücker-coordinate camera embeddings used throughout the model. (Sec. 3.2, p. 3: G = {G_{v,t} = K[R_{v,t}|t_{v,t}]}; Sec. 3.4, p. 4, Eqs. 2–3)

12. **Extrinsics as output:** No. Camera extrinsics are consumed as inputs only. The model outputs 2D pixel trajectories and occlusion states exclusively; no camera pose estimation is performed. (Sec. 3.2, p. 3; Sec. 3.6, p. 5)

13. **No GT intrinsics required at inference:** No. The shared camera intrinsic matrix K ∈ R^{3×3} is a required input used in the Plücker ray construction (K^{-1} applied to pixel coordinates). (Sec. 3.2, p. 3; Sec. 3.4, p. 4, Eq. 3)

14. **Intrinsics as output:** No. Intrinsics are input only. The model predicts trajectory and occlusion updates, not camera parameters. (Sec. 3.2, p. 3; Sec. 3.6, p. 5)

15. **Moving cameras:** Yes. Camera extrinsics are parameterized per view and per frame (R_{v,t}, t_{v,t}), so the architecture natively supports moving cameras. Evaluation datasets include Harmony4D and Panoptic Studio, which involve camera rigs that are not static, and the training dataset samples camera positions on hemispheres with varying positions (Appendix D.1, p. 12). (Sec. 3.2, p. 3; Sec. 3.4, p. 4; Sec. 4.1, pp. 5–6; Appendix D.1, p. 12)

16. **Dynamic content:** Yes. Explicitly designed for dynamic, non-rigid scenes. The abstract states the core motivation is "understanding dynamic objects in multi-view systems." Benchmarks include DexYCB (hand-object manipulation with fast, erratic motion), Panoptic Studio (human body motion), and Harmony4D (dynamic human performance capture). (Abstract, p. 1; Sec. 1, p. 1; Sec. 4.1, pp. 5–6; Figure 4, p. 4)

17. **Feed-forward inference:** Yes. MV-TAP performs a fixed M=4 iterative refinement steps in a single forward pass — no per-scene optimization or test-time training. Trajectory and occlusion updates are computed by the Transformer in each refinement step (Eqs. 7–8, p. 5). Optional triangulation-based post-processing is explored in Appendix C as an external pipeline add-on applied to the model's 2D output; it is not part of the trained model and does not affect the feed-forward nature of the core method. (Sec. 3.6, p. 5; Fig. 3, p. 3; Appendix C, p. 11)

**Notes:**
- **Input modality:** Synchronized multi-view RGB videos. Camera intrinsics K (shared across views) and per-frame-per-view extrinsics (R_{v,t}, t_{v,t}) are required at both training and inference. No depth input required or used.
- **Query format limitation:** Each query point must be specified in all V input views (view index, frame index, pixel coordinates). A key limitation acknowledged by the authors is that this assumption is often impractical; robust cross-view query initialization from a single-view query remains unreliable (Appendix A, pp. 9–10; Appendix F, p. 13).
- **Training data:** New large-scale synthetic multi-view dataset of 5,000 dynamic scenes with 4 viewpoints generated via Kubric [13]. MV-TAP is initialized from CoTracker3 [20] pretrained weights; the feature extraction network is frozen during training, while all other parameters (camera encoding MLP, transformer layers) are updated. Trained for 50K steps on 4 NVIDIA A6000 GPUs.
- **Training view count vs. inference:** Trained with 1–4 views (randomly sampled per scene) but generalizes to 8+ views at inference due to the attention-based design.
- **No 3D output:** Unlike MVTracker [31] (which lifts 2D tracks to 3D world space using depth), MV-TAP avoids 3D lifting entirely and remains in 2D pixel space to prevent reprojection error accumulation.
- **Optional pipeline extension:** Triangulation-based refinement (Appendix C / Table 11) can be applied as a post-processing step using the model's 2D outputs and known camera parameters to estimate 3D points and reproject them for refined 2D coordinates. This is a Pipeline capability for any 3D use, not an intrinsic model capability.
- **Benchmarks:** DexYCB, Panoptic Studio, Kubric, Harmony4D. Standard TAP-Vid metrics: Average Jaccard (AJ), position accuracy < δ_avg, Occlusion Accuracy (OA).
- **Performance highlights:** MV-TAP achieves consistent top performance across all four benchmarks over both single-view and multi-view baselines in 2D camera-space tracking (Table 1, p. 5). Performance improves monotonically as more input views are added (Table 2, p. 6).
