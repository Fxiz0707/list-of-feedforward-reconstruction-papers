**Paper:** UniDepthV2: Universal Monocular Metric Depth Estimation Made Simpler, arXiv:2502.20110v2 [cs.CV] / December 2025 (journal extension of CVPR 2024 UniDepth), identifier: UniDepthV2

**Summary:** UniDepthV2 is a universal monocular metric depth estimation (MMDE) model that takes a single RGB image as input and directly predicts metric 3D points without requiring any external camera information (neither intrinsics nor extrinsics) at inference time. It uses a self-prompting Camera Module that predicts a dense pinhole-based camera ray representation, which conditions a Depth Module to produce a pseudo-spherical per-pixel 3D output (θ, φ, z_log) equivalent to a metric point cloud. Training is fully supervised on 23 diverse public datasets (16M images); inference is a single feed-forward pass with no per-scene optimization.

---

1. **3D reconstruction:** Yes. UniDepthV2 directly outputs a dense 3D point cloud O ∈ R^{H×W×3} — the concatenation of predicted camera rays C and log-depth Z — for every pixel of the input image. The paper states "UniDepthV2 directly predicts metric 3D points from a single image" (Sec. III, p. 3), and Fig. 3 (p. 6) shows predicted point clouds as qualitative outputs across all evaluated domains.

2. **Dense reconstruction:** Yes. The output is per-pixel (H×W), providing a 3D point for every pixel in the image. The depth output Z_log ∈ R^{H×W×1} and the camera ray map C ∈ R^{H×W×2} together constitute a fully dense reconstruction; the final 3D output O ∈ R^{H×W×3} = C || Z (Sec. III-A p. 4; Sec. III-E p. 5).

3. **World-space output:** No. UniDepthV2 processes a single image at a time and produces outputs in that image's own camera coordinate frame. There is no multi-frame fusion, no pose estimation, and no mechanism to place outputs in a shared world coordinate frame across images or frames.

4. **Camera-space output (egocentric):** Yes. All 3D predictions are expressed relative to the camera coordinate frame of the input image. The pseudo-spherical representation (θ, φ, z_log) and its Cartesian equivalent are camera-centric by construction (Sec. III-A, p. 3–4).

5. **Flexible camera frame:** N/A. UniDepthV2 is strictly monocular single-image; the output is always in the one input image's camera frame. The concept of choosing a reference camera does not apply (Q4 = Yes, but only one camera frame ever exists at inference).

6. **Metric scale:** Yes. UniDepthV2 is explicitly designed for Monocular Metric Depth Estimation (MMDE), predicting true metric 3D coordinates without scale ambiguity. The abstract states it is "capable of reconstructing metric 3D scenes from solely single images across domains," and the training loss L_AMSE directly supervises metric 3D output in the pseudo-spherical space (Sec. III-E, Eq. 4, p. 5).

7. **Point tracking:** No. UniDepthV2 is a single-image model with no temporal or cross-frame processing. It establishes no correspondences between predicted points across frames or views.

8. **3D tracks:** No. The model has no tracking capability; it processes each image independently with no notion of point identity across time.

9. **2D tracks:** No. No tracking of any kind is present in the architecture or experiments.

10. **Occlusion / visibility prediction:** No. The model outputs a per-pixel prediction uncertainty Σ (Sec. III-E p. 5; Table VII p. 9), which quantifies depth confidence (supervised as L1 against log-space depth error), not occlusion or visibility state. There are no tracks, so per-track occlusion flags are not applicable.

11. **Multiple views:** No. UniDepthV2 takes a single RGB image as input at inference time ("with only one image as input," Abstract p. 1; Fig. 1 caption p. 1). The two images (I_1, I_2) appearing in Fig. 2 are geometric augmentations of the same training image used to compute the geometric invariance loss L_con; at inference, only one image is processed (Sec. III-C p. 4–5).

12. **Global context vs. pairwise vs. fixed-window:** N/A. UniDepthV2 is a single-image model with no cross-frame or cross-view aggregation at inference time. The ViT encoder applies global self-attention within the single image, but this is intra-image context, not cross-temporal or cross-view aggregation (Sec. III-E p. 5).

13. **No GT extrinsics required at inference:** Yes. The model requires no external camera information whatsoever. The abstract explicitly states inference "without any reliance on additional external information, such as camera parameters." No extrinsics (pose) are consumed at inference; Tables I–III confirm UniDepthV2 carries no "†" marker denoting GT camera requirement.

14. **Extrinsics as output:** No. The Camera Module outputs camera intrinsic parameters — focal length residuals (Δf_x, Δf_y) and principal point residuals (Δc_x, Δc_y) — not extrinsic pose (rotation/translation) (Sec. III-B p. 4). The model has no mechanism to predict camera pose relative to any reference frame.

15. **No GT intrinsics required at inference:** Yes. This is a central contribution of the paper. The Camera Module self-predicts intrinsic parameters from image content alone, requiring no externally supplied focal length or principal point (Abstract p. 1; Introduction p. 1–2; Sec. III-B p. 4).

16. **Intrinsics as output:** Yes. The Camera Module predicts four scalar intrinsic residuals (Δf_x, Δf_y, Δc_x, Δc_y) relative to a canonical pinhole initialization, then converts them into a dense per-pixel camera ray map C ∈ R^{H×W×2} encoding azimuth and elevation angles (Sec. III-B p. 4). Camera prediction quality is measured by ρ_A (average angular error of camera rays up to 15°; Sec. IV-A p. 7; Tables I–III).

17. **Moving cameras:** Yes. UniDepthV2 makes no assumption of a static camera and is designed for arbitrary images from any camera setup. Training data includes multiple moving-camera datasets (NuScenes, DDAD, DrivingStereo, Waymo, TartanAir, etc.), and evaluation covers moving-camera sequences in all three domain groups (indoor, outdoor, challenging; Sec. IV-A p. 6–7; Tables I–III).

18. **Dynamic content:** Partial. UniDepthV2 produces per-pixel metric depth for whatever content is in the input frame, including moving objects, and is trained on datasets containing dynamic content (DynamicReplica, BEDLAM, HOI4D, PointOdyssey — Sec. IV-A p. 6). However, the model has no explicit mechanism to detect, flag, or specially handle dynamic/non-rigid content; it treats each scene as a static snapshot with no temporal reasoning, and produces no motion or dynamic-object outputs.

19. **Feed-forward inference:** Yes. UniDepthV2 performs a single forward pass per image with no per-scene optimization, test-time training, or iterative refinement. The paper explicitly states "we do not exploit any test-time augmentations, and we utilize the same weights for all zero-shot evaluations" (Sec. IV-A p. 6–7). Inference latencies are 23.0 ms (Small), 35.1 ms (Base), and 65.4 ms (Large) at 0.5 MP on an A6000 GPU (Table VIII p. 10).

---

**Notes:**
- **Input modality:** Single RGB image only; no depth, stereo, IMU, or camera parameter input is required or used at inference.
- **Pinhole assumption and non-pinhole limitation:** The Camera Module models a pinhole camera internally (Sec. III-B p. 4). Non-pinhole cameras (fisheye, panoramic, orthographic) are a stated limitation; the model "struggles to represent non-pinhole projection" (Sec. IV-D p. 11; Fig. 7).
- **Optional camera injection:** Although no intrinsics are required, the architecture is modular — known camera rays from an external source can optionally be injected at inference instead of using the Camera Module output: "the ray bundle C need not come from UniDepthV2's Camera Module: at inference, it can be injected from any camera model with known parameters and a specified unprojection operator" (Sec. III-E p. 5).
- **Uncertainty output:** A per-pixel depth confidence map Σ is produced as an optional additional output, supervised by L1 loss against log-space depth error with no extra annotation (Sec. III-E Eq. 5; Table VII). This enables downstream confidence-aware masking or control applications.
- **Model variants:** Three ViT backbone sizes — Small, Base, Large — allowing a speed/accuracy trade-off.
- **Training supervision:** Fully supervised with metric ground-truth depth from 23 public datasets totaling 16M images. No self-supervised, generative, or unsupervised components.
- **No temporal modeling:** Despite evaluation on video-sourced datasets, the model processes each frame independently with no recurrence, cross-frame attention, optical flow, or pose graph.
- **3D estimation metric:** The F_A metric (AUC of F1 score for 3D point cloud accuracy) is used specifically to measure 3D reconstruction quality, distinct from standard per-pixel depth metrics (δ_1, AbsRel, SI_log) (Sec. IV-A p. 7).
