**Paper:** CoTracker3: Simpler and Better Point Tracking by Pseudo-Labelling Real Videos, arXiv Oct 2024, CoTracker3
**Summary:** CoTracker3 is a transformer-based 2D video point tracker that takes monocular RGB video frames and query points as input, then predicts 2D pixel-space trajectories, per-point visibility, and confidence in online (sliding-window) or offline (full-video-window) variants. It is pre-trained on synthetic Kubric data and fine-tuned via a semi-supervised pseudo-labelling protocol using frozen off-the-shelf teacher trackers on unlabelled real videos; inference is a direct learned forward pass with no per-scene optimisation. It does not produce a point cloud or any 3D representation.

1. **3D reconstruction:** No. CoTracker3 is not a 3D reconstruction method. The task is explicitly defined as predicting 2D image-plane tracks P_t = (x_t, y_t) ∈ R², visibility V_t, and confidence C_t from video frames and query points. 3D reconstruction is mentioned only as a downstream use case (Section 3, p. 3; Conclusion, p. 10).

2. **Dense reconstruction:** No. The model tracks an arbitrary set of queried points; it does not output a dense per-pixel representation or point cloud. Grid visualisations (e.g. 100×100 in Figure 4) are evaluation artefacts, not a per-pixel model output (Section 3, p. 3; Figure 4, p. 10).

3. **Reference frame:** N/A. CoTracker3 is a 2D point tracker (Q1 = No); it produces no 3D geometry output.

4. **Metric scale:** No. Predictions are pixel-coordinate tracks. Evaluation uses pixel-threshold metrics (δ_avg) and resized 256×256 videos, with no metric 3D scale involved (Section 3, p. 3; Section 4, p. 7).

5. **Point tracking:** Yes. Point tracking is the core capability. Given a query point Q = (t^q, x^q, y^q) in a video, CoTracker3 predicts the corresponding point location P_t = (x_t, y_t) across all video frames t = 1, …, T (Section 3, p. 3; entire paper).

6. **3D tracks:** No. CoTracker3 explicitly predicts 2D tracks P_t = (x_t, y_t) ∈ R²; no 3D point trajectories are output anywhere in the paper (Section 3, p. 3).

7. **2D tracks:** Yes. 2D pixel-coordinate tracks are the primary model output (Section 3, p. 3).

8. **Occlusion / visibility prediction:** Yes. The model outputs per-point visibility V_t ∈ [0, 1] (visible vs. occluded) and confidence C_t ∈ [0, 1], each supervised with a Binary Cross Entropy loss at every iterative update (Section 3, pp. 3–4; Equations 3–4, p. 6; Occlusion Accuracy evaluated in Tables 1–2, pp. 7–8).

9. **Multiple views:** No. Input is a single monocular video sequence (I_t)_{t=1}^T plus query points. Both online and offline variants process temporal frames of one video, not simultaneous multi-camera views (Section 3, p. 3; Section 3.2, pp. 4–5).

10. **Global context vs. pairwise vs. fixed-window:** Hybrid. The offline variant processes the entire video as a single window (effectively global attention over all frames that fit in memory), while the online variant uses a fixed sliding window of size T' that advances T'/2 frames at a time, initialising from overlapping predictions of the previous window. Within any window, the transformer applies factorised time and cross-track attention jointly over all frames and all tracks (Section 3.2, pp. 4–5; Section 3.3, p. 6).

11. **No GT extrinsics required at inference:** Yes. Inputs are video frames and query points only; ground-truth camera extrinsics are neither required nor used (Section 3, pp. 3–4).

12. **Extrinsics as output:** No. Outputs are 2D tracks, visibility, and confidence. No camera extrinsics are estimated or output (Section 3, pp. 3–4).

13. **No GT intrinsics required at inference:** Yes. The model is formulated entirely in image coordinates and evaluation resizes videos to 256×256 without camera calibration input (Section 3, pp. 3–4; Section 4, p. 7).

14. **Intrinsics as output:** No. Camera intrinsics are not estimated or output (Section 3, pp. 3–4).

15. **Moving cameras:** Yes. The model is evaluated on TAP-Vid-Kinetics, explicitly described as featuring "complex camera motion and cluttered backgrounds," and the image-coordinate formulation makes no static-camera assumption (Section 4, p. 7).

16. **Dynamic content:** Yes. Training data consists of ~100,000 Internet videos featuring humans and animals in dynamic scenes; evaluation benchmarks include Dynamic Replica (articulated 3D models), RoboTAP (robotic manipulation with moving arms), and TAP-Vid-DAVIS (real-world videos with moving subjects) (Section 3.1, p. 4; Section 4, pp. 7–8).

17. **Feed-forward inference:** Yes. Inference consists of M fixed iterative neural updates within a single forward pass of a trained network; there is no per-scene optimisation or test-time training. Runtime is reported in microseconds per frame per tracked point (Section 3.2, pp. 5–6; Table 2, p. 8; Appendix B, pp. 14–15).

**Notes:** CoTracker3 is a 2D tracking-any-point model — not an SfM, depth estimation, camera-pose, or 3D reconstruction system. Input modality is monocular RGB video plus sparse query points. Training uses synthetic supervision (Kubric) plus pseudo-labels from multiple frozen teacher trackers (CoTracker, TAPIR, CoTracker3 online/offline) on unlabelled real videos; the confidence and visibility head is frozen during real-video fine-tuning to prevent forgetting (Section 3.3, p. 6; Table 7, p. 10). The offline variant tracks in both temporal directions and handles occlusions better; the online variant runs in real time with sliding windows. Scaling performance saturates beyond ~30k real training videos due to the limited knowledge ceiling of the teacher ensemble (Appendix D, p. 16). The paper explicitly positions CoTracker3 as a building block for downstream tasks such as 3D tracking, controlled video generation, and dynamic 3D reconstruction — none of which are performed by the model itself (Conclusion, p. 10).
