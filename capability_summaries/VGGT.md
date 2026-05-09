**Paper:** VGGT: Visual Geometry Grounded Transformer, arXiv:2503.11651, Mar 2025, VGGT
**Summary:** VGGT is a large feed-forward transformer (~1.2B parameters) that takes an unordered set of N RGB images (from 1 to hundreds) as input and directly predicts all key 3D scene attributes — camera intrinsics and extrinsics, per-pixel depth maps, dense 3D point maps, and 2D/3D point tracks — in a single forward pass without per-scene optimization. The model uses an Alternating-Attention architecture (alternating frame-wise and global self-attention) trained end-to-end on a large heterogeneous collection of 3D-annotated datasets, achieving state-of-the-art results on camera pose estimation, dense reconstruction, and point tracking while running in under one second.

1. **3D reconstruction:** Yes. VGGT predicts dense point maps P_i ∈ ℝ^{3×H×W} for every input image, associating each pixel with a 3D scene point in a shared world coordinate frame. Together these constitute a dense point cloud of the scene. (Sec. 3.1, p. 3; Table 3 on ETH3D point map estimation, p. 7; Figs. 3–4.)

2. **Dense reconstruction:** Yes. Point maps and depth maps are per-pixel (full H×W resolution) outputs produced by a DPT head applied to the transformer's output tokens. The paper explicitly labels these "Dense Predictions." (Sec. 3.3 "Dense Predictions," p. 5; Sec. 4.2 multi-view depth on DTU, p. 7.)

3. **Reference frame:** Selectable. The reference frame is the first input image; because any image can be placed first, the user can freely designate any frame as the anchor, making both world-space and per-frame egocentric output available depending on input ordering.

4. **Metric scale:** No. VGGT outputs are up-to-scale. During training, ground truth is normalized by the average Euclidean distance of all 3D points to the origin, and the model learns this normalized variant. No absolute metric scale is recovered at inference. (Sec. 3.4 "Ground Truth Coordinate Normalization," p. 6; Sec. 5 "Normalizing Prediction," p. 10.)

5. **Point tracking:** Yes. VGGT includes a tracking module T (CoTracker2-based architecture), jointly trained end-to-end with the main transformer, that takes a query point in any input image and outputs 2D correspondences in all other frames, establishing cross-frame point correspondences. (Sec. 3.1, p. 3; Sec. 3.3 "Tracking," p. 5; Sec. 3.4 tracking loss, p. 6; Sec. 4.4 image matching, p. 8.)

6. **3D tracks:** Yes. The paper cites "3D point tracks" as a core primary output (abstract, p. 1; introduction, p. 2; conclusion, p. 11). 3D tracks are obtained by composing the 2D tracking correspondences from T with the dense 3D point maps from f — both are integrated, jointly trained components of VGGT. This composition is native to the model. (Note: the tracking loss itself is on 2D predictions; 3D track extraction via point-map lookup is inherent to the integrated system.)

7. **2D tracks:** Yes. The tracking module T directly outputs 2D point positions ỹ_{j,i} ∈ ℝ² in each image I_i for a given query point. The tracking loss L_track is defined on these 2D predictions. (Sec. 3.3 "Tracking," p. 5: "the tracking head T predicts the set of 2D points T((y_j)^M, (T_i)^N) = ((ỹ_{j,i})^N)^M_{j=1}"; Sec. 3.4, p. 6.)

8. **Occlusion / visibility prediction:** Yes. VGGT explicitly applies a visibility loss (binary cross-entropy) to predict whether a tracked point is visible in each frame. Occlusion Accuracy (OA) is reported as an evaluation metric in Table 8 on TAP-Vid benchmarks. (Sec. 3.4, p. 6: "we apply a visibility loss (binary cross-entropy) to estimate whether a point is visible in a given frame"; Table 8, p. 10.)

9. **Multiple views:** Yes — arbitrary number of views. VGGT simultaneously processes N input images where N is flexible from 1 to hundreds. The abstract states it accepts "from one, a few, or hundreds of its views." Table 9 shows measured scaling from 1 to 200 input frames. There is no fixed N constraint. (Abstract, p. 1; Fig. 1 caption; Table 9, p. 10.)

10. **Global context vs. pairwise vs. fixed-window:** Global. VGGT's Alternating-Attention (AA) architecture alternates between frame-wise self-attention (within each frame) and global self-attention (across all N frames jointly at once). The ablation study (Table 5) confirms global attention is essential to performance versus fixed-window or cross-attention alternatives. (Sec. 3.2 "Alternating-Attention," p. 4; Table 5, p. 8.)

11. **No GT extrinsics required at inference:** Yes. VGGT predicts camera extrinsics as output without requiring them as input. This is the key departure from traditional MVS methods. Table 2 explicitly places VGGT in the lower section of methods that "do not know the ground-truth camera" at test time. (Abstract; Sec. 3.1, p. 3; Table 2, p. 7.)

12. **Extrinsics as output:** Yes. The camera head predicts per-frame rotation quaternion q ∈ ℝ^4 and translation vector t ∈ ℝ^3 for each frame i, derived from the camera output tokens via four additional self-attention layers followed by a linear layer. (Sec. 3.1, p. 3; Sec. 3.3 "Camera Predictions," p. 5.)

13. **No GT intrinsics required at inference:** Yes. VGGT does not require ground-truth intrinsics as input. Intrinsics are predicted by the same camera head alongside extrinsics. (Sec. 3.1, p. 3; no intrinsics input described in the inference pipeline anywhere in the paper.)

14. **Intrinsics as output:** Yes. The camera head predicts per-frame field of view f ∈ ℝ^2 (two focal-length parameters) as part of the 9-dimensional camera parameter vector g_i = [q, t, f]. The principal point is assumed at the image center (common SfM framework assumption, not predicted). (Sec. 3.1, p. 3: "the field of view f ∈ ℝ^2. We assume that the camera's principal point is at the image center, which is common in SfM frameworks.")

15. **Moving cameras:** Yes. VGGT estimates per-frame camera extrinsics (rotation and translation), explicitly designed for scenes captured from multiple viewpoints with moving cameras. Evaluated on RealEstate10K (moving camera video sequences), CO3Dv2, ETH3D, and the IMC phototourism benchmark. (Sec. 4.1, Table 1, p. 7; Appendix C, Table 10, p. 12.)

16. **Dynamic content:** Partial. The core VGGT model targets static or near-static scenes. The paper explicitly states as a limitation: "although our model handles scenes with minor non-rigid motions, it fails in scenarios involving substantial non-rigid deformation." (Sec. 5 "Limitations," p. 10.) A separately fine-tuned downstream model (CoTracker2 with VGGT features as backbone) is evaluated on TAP-Vid dynamic benchmarks (Table 8), but this is a fine-tuned variant, not the core VGGT.

17. **Feed-forward inference:** Yes. VGGT performs a single forward pass with no per-scene optimization or test-time training. The paper repeatedly emphasizes this property. Runtime is approximately 0.2s for typical 10-frame scenes on an H100 GPU; Table 9 reports 0.04s to 8.75s for 1 to 200 input frames. Optional bundle adjustment post-processing (VGGT + BA) is available but not part of the model itself. (Abstract, p. 1; Sec. 4.1, p. 7: "VGGT achieves superior performance while only operating in a feed-forward manner, requiring just 0.2 seconds"; Table 9, p. 10.)

---

**Notes:**
- **Input modality:** Unordered set of N RGB images (N = 1 to ~200+ tested) of the same scene. No temporal ordering required; the model is permutation-equivariant for all frames except the first (which anchors the reference frame).
- **Scale:** Output is up-to-scale. The model normalizes 3D predictions by the mean 3D point distance to origin. Downstream applications requiring metric scale need an external scale-recovery step.
- **Intrinsics assumption:** Principal point is assumed at image center; fisheye and panoramic lenses are not supported (explicit limitation, Sec. 5, p. 10).
- **Better point clouds from depth:** The paper reports that unprojecting predicted depth maps using predicted camera parameters ("Depth + Cam") yields better point clouds than using the point map head directly (Table 3, Sec. 4.3, p. 8). Both routes are integrated VGGT outputs.
- **Tracking head details:** The tracker T uses the CoTracker2 architecture initialized from dense tracking features T_i (from the DPT head). It does not assume temporal ordering — it can track across any set of input images. The tracking loss and visibility loss are jointly trained with the full model.
- **Supervision:** End-to-end multi-task supervision: camera loss L_camera (Huber on quaternion/translation/FoV), depth loss L_depth (uncertainty-weighted gradient + scale-invariant), point map loss L_pmap, and tracking loss L_track (2D position L2 + visibility binary cross-entropy). All losses trained simultaneously.
- **Optional BA post-processing:** VGGT + Bundle Adjustment (external, optional) substantially improves camera pose accuracy (AUC@10 from 71.26 to 84.91 on IMC, Table 10). BA is not part of the model; VGGT's accurate point/depth predictions serve as a high-quality initialization that eliminates the need for triangulation, making BA much faster than in traditional pipelines (~1.8s total vs. >10s for VGGSfM).
- **Downstream fine-tuning:** VGGT's pretrained backbone transfers well to novel view synthesis and dynamic point tracking with minimal architectural changes, demonstrating strong generalizable 3D features.
- **Limitations:** Fails on fisheye/panoramic images, degrades under extreme input rotations, and fails under substantial non-rigid deformation. Single-view reconstruction works surprisingly well despite not being explicitly trained for it (Sec. 5, p. 10; Appendix D, Fig. 7).
