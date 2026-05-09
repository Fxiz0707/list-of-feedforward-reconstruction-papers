**Paper:** MegaSaM: Accurate, Fast, and Robust Structure and Motion from Casual Dynamic Videos, arXiv Dec 2024, MegaSAM
**Summary:** MegaSaM estimates per-frame camera poses (SE(3)), a shared focal length, and dense per-pixel depth maps from casual monocular videos of dynamic scenes. It is a learned SLAM/SfM-style pipeline built on a differentiable bundle adjustment framework (extended DROID-SLAM), augmented with motion probability maps and mono-depth priors to handle dynamic content. Inference involves iterative BA and an optional per-scene consistent video depth optimization stage — it is not a single feed-forward pass.

1. **3D reconstruction:** Yes. The system outputs per-frame dense depth maps and camera poses; the paper explicitly shows "3D point clouds unprojected by predicted video depths" as the reconstruction output (Abstract, p. 1; Fig. 1, p. 1; Sec. 3, p. 2).

2. **Dense reconstruction:** Yes. Per-pixel depth maps D̂_i are estimated for every frame; a dedicated consistent video depth (CVD) stage produces higher-resolution video depth maps at full frame resolution (Sec. 3.1, p. 2; Sec. 3.3, p. 5; Table 4, p. 6).

3. **World-space output:** Yes. Camera poses G_i ∈ SE(3) are estimated in a single consistent world reference frame through global BA over all video frames; 3D point clouds shown in Fig. 1 and Fig. 7 are in this unified world frame (Sec. 3.2.2, pp. 4–5; Sec. 4.1, p. 7).

4. **Camera-space output (egocentric):** Yes. Per-frame disparity/depth maps D̂_i are defined in each frame's own camera coordinate frame and are available directly from the system before any transformation by the estimated poses (Sec. 3, p. 2; Appendix A.1, p. 12).

5. **Flexible camera frame:** No. The system fixes the gauge by aligning the first camera poses during initialization and provides no mechanism to freely choose a reference camera at inference; camera-space depths per frame are available (Q4), but this is not a selectable flexible reference (Sec. 3.2.2, pp. 4–5; Appendix A.4, p. 14).

6. **Metric scale:** Partial. MegaSaM initializes disparity with metric estimates from UniDepth [43] (providing absolute scale) and DepthAnything [71] (providing relative depth quality), so metric scale is actively pursued; however, camera evaluation uses Sim(3) alignment and depth evaluation uses global scale-and-shift re-alignment, so the paper does not demonstrate or validate true absolute metric scale in its benchmarks (Sec. 3.2.2, p. 4; Sec. 4.1, p. 7).

7. **Point tracking:** No. Pairwise 2D flow fields û_ij are predicted internally and used within differentiable BA, but they are not output as point correspondences or tracks; the system's declared outputs are camera poses, focal length, and depth maps (Sec. 3.1, pp. 2–3; Sec. 3.2.1, p. 3; Sec. 3, p. 2).

8. **3D tracks:** No. MegaSaM does not output 3D point trajectories; dynamic content is handled by motion probability weighting and uncertainty masking within BA, not by reconstructing explicit 3D tracks for moving objects (Sec. 3.2.1, pp. 3–4; Sec. 3.3, p. 5).

9. **2D tracks:** No. The system does not output 2D point tracks; internal pairwise flow fields are used only within the BA framework and are not surfaced as user-facing tracks (Sec. 3.1, pp. 2–3; Sec. 3.3, p. 5).

10. **Occlusion / visibility prediction:** No. The system predicts per-pixel object movement probability maps m_i (to downweight dynamic regions in BA) and per-pixel flow/depth uncertainty maps, but these are motion and uncertainty masks, not per-point or per-track occlusion/visibility flags in the standard sense (Sec. 3.2.1, pp. 3–4; Sec. A.3, p. 12).

11. **Multiple views:** No. The input is a continuous monocular video sequence V = {I_i}; the frame graph and BA operate over temporal video frames from a single camera, not simultaneous multi-camera or multi-view input (Abstract, p. 1; Sec. 3, p. 2).

12. **Global context vs. pairwise vs. fixed-window:** Hybrid. Inference uses a sliding-window frontend BA for initial keyframe registration, followed by a global BA backend over all keyframes, then a pose-graph optimization for non-keyframes, and a final all-frame global BA refinement; the optional CVD stage adds fixed-interval pairwise optical-flow losses (Sec. 3.2.2, pp. 4–5; Appendix A.3, pp. 12–13).

13. **No GT extrinsics required at inference:** Yes. Estimating camera poses is the primary goal; no ground-truth extrinsics are provided or required at inference, as confirmed by evaluation on both calibrated and uncalibrated settings (Abstract, p. 1; Sec. 3, p. 2; Tables 1–3, pp. 6–7).

14. **Extrinsics as output:** Yes. Per-frame camera poses G_i ∈ SE(3) are an explicit primary output, evaluated with ATE, RTE, and RRE metrics (Sec. 3, p. 2; Sec. 4.1, p. 7; Tables 1–3, pp. 6–7).

15. **No GT intrinsics required at inference:** Yes. In the uncalibrated setting, focal length is initialized from UniDepth predictions and jointly optimized during BA; GT intrinsics are not required, and focal optimization can be disabled automatically when unobservable (Sec. 3.1, p. 3; Sec. 3.2.2, pp. 4–5; Tables 1–3, pp. 6–7).

16. **Intrinsics as output:** Yes. A shared focal length f̂ per video is estimated alongside camera poses and depths; note that only a single shared focal length is estimated (not per-frame or full K with distortion), and the system cannot handle varying focal lengths or strong radial distortion within a video (Sec. 3, p. 2; Sec. 3.2.2, pp. 4–5; Sec. 5, p. 8).

17. **Moving cameras:** Yes. The method is specifically designed for unconstrained camera paths, including handheld capture with limited parallax, near-rotational motion, and varying focal lengths; both the training modifications and the uncertainty-aware BA are motivated by these challenging moving-camera conditions (Abstract, p. 1; Sec. 1, pp. 1–2; Fig. 2, p. 3; Fig. 7, p. 8).

18. **Dynamic content:** Yes. Handling dynamic scene content is a core contribution: MegaSaM learns a motion probability module F_m trained on dynamic video data, uses it to downweight dynamic elements during BA, and is evaluated on dynamic benchmarks (Sintel, DyCheck, in-the-wild, DAVIS); failure cases include scenes where moving objects dominate the entire frame (Sec. 3.2.1, pp. 3–4; Sec. 4, pp. 6–8; Sec. 5, p. 8; Appendix B, p. 14).

19. **Feed-forward inference:** No. Inference is not a single forward pass; it involves iterative differentiable BA with multiple optimization steps for camera tracking, followed by an optional per-scene consistent video depth optimization (100 warm-up + 400 joint steps). No per-video network fine-tuning is performed, but per-scene iterative optimization is integral to the method (Sec. 3.2.2, pp. 4–5; Sec. 3.3, p. 5; Appendix A.3, p. 13).

**Notes:** Input modality is monocular video only (single camera, temporally ordered). The system depends on off-the-shelf models at inference: UniDepth [43] for metric scale and focal length initialization, DepthAnything [71] for relative depth priors, and an off-the-shelf optical flow model [58] (RAFT) for the optional CVD stage. Training uses synthetic data (TartanAir + Kubric static, Kubric dynamic) with no real-data supervision; strong generalization to real-world dynamic video is demonstrated. Only a single shared focal length per video is estimated; varying focal lengths and strong radial distortion are stated limitations. The system does not produce 4D dynamic scene representations, persistent object tracks, or explicit segmentation of moving vs. static regions as outputs — the motion probability is an internal weighting mechanism. Average running time for the full pipeline (with CVD) is approximately 1.3 FPS on a single A100 GPU.
