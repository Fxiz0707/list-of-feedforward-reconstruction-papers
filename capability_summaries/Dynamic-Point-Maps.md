**Paper:** Dynamic Point Maps: A Versatile Representation for Dynamic 3D Reconstruction, arXiv March 2025, Dynamic-Point-Maps
**Summary:** DPM extends DUSt3R's point-map representation to dynamic scenes by making point maps invariant to both viewpoint and time. Given a pair of RGB images (I_1, I_2) captured at possibly different times, a single feed-forward transformer network predicts four pixel-aligned 3D point maps — two per image, one anchored to each timestamp — all expressed in the reference frame of the first camera. This representation directly encodes scene flow, 3D correspondences, and rigid object motion, enabling state-of-the-art results on video depth estimation, dynamic point cloud reconstruction, scene flow, and object pose tracking without per-scene optimization.

1. **3D reconstruction:** Yes. The network outputs four dense point maps P_i(t_j, π_1) ∈ ℝ^{3×HW}, each associating every pixel with a 3D point in a shared coordinate frame. Dynamic point cloud reconstruction is a primary evaluation task (Sec. 4.2, Table 3; Sec. 6 describes the network as densely predicting 3D point clouds from a pair of images).

2. **Dense reconstruction:** Yes. Point maps are pixel-aligned by definition: each pixel u receives a 3D point p = P(u) ∈ ℝ³ (Sec. 3.1). All four predicted point maps maintain this per-pixel density (Sec. 3.2, Eq. 1).

3. **Reference frame:** Fixed world. All four pointmaps are expressed in the fixed reference frame π_1 (the first frame's camera coordinate system); the anchor is hard-coded and not user-selectable.

4. **Metric scale:** No. Sec. 3.3 (Training loss) explicitly states: "the scale of a 3D scene cannot be determined uniquely from any number of views; therefore, we relax the predictions to be determined up to a scaling factor." The regression loss normalizes by the median norm of the point cloud. Output is up-to-scale.

5. **Point tracking:** Yes. Establishing cross-image correspondences is a core downstream capability. Given pixel u_1 in I_1, the matching pixel in I_2 is found by minimizing ||P_1(t_1,π_1)(u_1) − P_2(t_2,π_1)(u_2)||, because both maps share the same spatio-temporal reference frame (Sec. 5, point correspondence; Appendix A.4, correspondence matrix formulation; Fig. 6). This works for dynamic objects as well as under camera motion.

6. **3D tracks:** Yes. The two point maps per image directly encode 3D point trajectories: P_1(t_1,π_1) gives 3D locations of I_1 pixels at time t_1, and P_1(t_2,π_1) gives where those same scene points are at time t_2. Scene flow is P_1(t_2,π_1) − P_1(t_1,π_1); object flow is P_2(t_1,π_1) − P_1(t_1,π_1) (Sec. 3.2; Sec. 4.2, Table 4; Appendix B.1).

7. **2D tracks:** Yes. With 3D tracks (Q6 = Yes) and recoverable camera intrinsics (Q14 = Yes), 2D projection is trivial. Additionally, the pixel-level correspondence mechanism in Sec. 5 directly yields dense 2D correspondences between the two input images (correspondence matrix C_12, Appendix A.4).

8. **Occlusion / visibility prediction:** No. The model predicts a per-pixel confidence map C_i(t_j, π_1) ∈ [0,1]^HW alongside each point map (Sec. 3.3), but this reflects reconstruction confidence, not occlusion or visibility status. Motion segmentation masks (Fig. 5) are derived from point-map differences, not occlusion reasoning. No explicit occlusion or visibility flags are produced.

9. **Multiple views:** No. The learned network takes exactly two images (I_1, I_2) as input. Extension to longer video sequences is handled by applying pairwise predictions sequentially with optional bundle adjustment post-processing (Sec. 4.1), not by simultaneous multi-view processing within a single model call.

10. **Global context vs. pairwise vs. fixed-window:** Pairwise. The core model processes image pairs via cross-attention between the two images (Fig. 2, Sec. 3.3), aggregating information within a fixed two-frame window. For video depth evaluation, pairwise outputs are fused via bundle adjustment (Sec. 4.1), but this is an optional external post-processing step, not part of the model's inference.

11. **No GT extrinsics required at inference:** Yes. The model takes raw image pairs and predicts point maps directly with no ground-truth camera poses or extrinsics as input. This is the core design principle, inherited from DUSt3R and extended throughout (Sec. 3.1; Appendix A.2).

12. **Extrinsics as output:** Yes. Relative camera extrinsics are recoverable from the predicted point maps. Appendix A.4 shows that the relative rotation R* and translation t* between cameras can be extracted via Procrustes alignment of point clouds at the same timestamp. Camera tracking (recovering camera poses while ignoring dynamic distractors) is demonstrated as a downstream application (Sec. 5; Fig. 7).

13. **No GT intrinsics required at inference:** Yes. The model ingests raw images without requiring ground-truth camera intrinsics as input. Randomized intrinsics are used in training (Sec. 3.3, MOVi-G pipeline) to improve generalization.

14. **Intrinsics as output:** Yes. Camera intrinsics K_1 are recoverable from the predicted point map via U_1Λ_1 = K_1 P_1(t_1,π_1) (Appendix A.1, A.4). The intrinsics of both cameras can be obtained (K_2 by swapping inputs), following the same analytical recovery mechanism as DUSt3R.

15. **Moving cameras:** Yes. The DPM formulation is explicitly designed to be viewpoint-invariant, handling non-static cameras. Camera motion is disentangled from scene motion by the representation. The method is evaluated on datasets with complex camera trajectories (MOVi-G, Waymo, KITTI), and camera tracking is a demonstrated downstream application (Sec. 4.2; Sec. 4.3; Sec. 5; Fig. 7).

16. **Dynamic content:** Yes. This is the paper's primary contribution. DPM extends DUSt3R specifically to handle moving, non-rigid scene content that standard DUSt3R/MonST3R cannot represent with invariant point maps. Scene flow, object flow, motion segmentation, and 3D object tracking are all evaluated on sequences with dynamic objects (Sec. 1; Sec. 3.2; Sec. 4.2; Sec. 5; Tables 3–4).

17. **Feed-forward inference:** Yes. The core model Φ(I_1, I_2) → {P_i(t_j, π_1)} is a single forward pass of a transformer encoder-decoder network (Fig. 2; Sec. 3.3; stated explicitly in abstract and Fig. 1 caption: "predicting DPMs from a pair of images in a feed-forward manner"). No per-scene optimization or test-time training is required. The optional bundle adjustment used in the video depth pipeline (Sec. 4.1) is a post-processing step external to the model.

---

**Notes:**
- **Input modality:** Monocular RGB image pairs. No depth, LiDAR, stereo rig, or multi-camera setup required at inference.
- **Scale ambiguity:** Output is up-to-scale (Sec. 3.3). Metric reconstruction would require an external scale reference.
- **Two-frame core model:** The learned network processes only pairs of frames. Video-length reconstruction requires iterative pairwise application plus optional bundle adjustment (Sec. 4.1), analogous to DUSt3R's multi-view extension.
- **Supervision:** Trained on a mixture of synthetic (MOVi-G, PointOdyssey, Spring, ScanNet++, BlendedMVS, MegaDepth) and real (Waymo with LiDAR-derived point maps) datasets. Supervision varies: full four-map supervision for dynamic datasets (Kubric/Waymo), same-time-only supervision for semi-dynamic datasets (PointOdyssey/Spring), and static reconstruction supervision for static datasets (Table 5).
- **Architecture:** Fine-tuned from DUSt3R with two additional regression heads per image (φ_{i1} and φ_{i2}) to predict cross-time point maps, trained at (512, 288) and (512, 336) resolutions (Sec. 3.3).
- **Confidence maps:** Per-pixel confidence scores C_i ∈ [0, 1] are predicted alongside each point map; used in the loss weighting (Eq. 2) but are not occlusion/visibility flags.
- **Downstream tasks derived analytically:** Scene flow, object flow, motion segmentation, point correspondence, camera tracking, and object tracking are all derived analytically from the four predicted point maps without additional learned modules (Sec. 5; Appendix A.4).
- **Baseline comparisons:** Main RGB-only results are compared to MonST3R; RAFT-3D (an RGBD method using ground-truth depth) is also compared in scene flow evaluations, where DPM achieves comparable or better results using only RGB input (Table 4).
