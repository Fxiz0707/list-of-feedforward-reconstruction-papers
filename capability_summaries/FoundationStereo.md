**Paper:** FoundationStereo: Zero-Shot Stereo Matching, arXiv Jan 2025 (NVIDIA), FoundationStereo
**Summary:** FoundationStereo is a zero-shot stereo depth estimation model that takes a rectified left/right stereo image pair and predicts a dense per-pixel disparity map, from which metric depth and a metric 3D point cloud follow directly given the known stereo baseline and focal length (as demonstrated in Fig. 1). The network integrates a frozen DepthAnythingV2 backbone adapted via a Side-Tuning Adapter, an Attentive Hybrid Cost Filtering module (3D Axial-Planar Convolution + Disparity Transformer), and iterative GRU-based disparity refinement; it is trained on a large-scale 1M-pair synthetic dataset plus public stereo data with fixed weights and no per-scene optimization at inference.

1. **3D reconstruction:** Yes. Figure 1 (p. 1) explicitly shows "Metric Point Cloud" as a model output row alongside Left RGB and Disparity across diverse real-world scenes. Stereo disparity with a known calibrated baseline and focal length yields dense metric 3D directly; the paper presents this as a primary output rather than an external post-process.

2. **Dense reconstruction:** Yes. The model predicts a disparity value at every pixel via soft-argmin initialization (Eq. 2, Sec. 3.2, p. 5) and iterative convex upsampling through GRU refinement (Sec. 3.3, p. 5). Training loss (Eq. 11) supervises all pixels; benchmark evaluation uses average end-point error (EPE) per pixel (Sec. 4.2, p. 6).

3. **World-space output:** No. The method processes a single rectified stereo pair and outputs left-camera-frame disparity. There is no multi-frame or multi-view fusion into a global world coordinate frame (Fig. 2, p. 4; Sec. 3.1–3.4).

4. **Camera-space output (egocentric):** Yes. The disparity map is defined in the left camera's image plane, from which 3D points in the left camera coordinate frame are obtained via standard backprojection. The output is inherently in the current (left) camera's egocentric frame (Fig. 2, p. 4; Sec. 3.2).

5. **Flexible camera frame:** No. The architecture is fixed around the stereo convention: the left image provides context features and the right image provides the matching reference; the left camera is always the reference frame with no mechanism to choose an arbitrary reference (Sec. 3.1–3.3, pp. 3–5).

6. **Metric scale:** Yes. Figure 1 (p. 1) labels the output "Metric Point Cloud." Stereo geometry (known baseline and focal length) provides absolute metric scale as a geometric property of the stereo setup; the paper trains with ground-truth metric disparity and evaluates with metric EPE. The synthetic training dataset randomizes baseline and focal length to cover diverse metric configurations (Sec. 3.5, p. 5; Supplement Sec. 11, p. 14).

7. **Point tracking:** No. FoundationStereo establishes horizontal stereo correspondences between a left and right image at a single time instant; it does not assign persistent identities to points or track correspondences across temporal frames (Sec. 3.2–3.4, pp. 4–5).

8. **3D tracks:** No. There is no temporal axis in the model; only a single stereo pair is processed at once (Sec. 3.1–3.4).

9. **2D tracks:** No. A stereo disparity map is not a 2D point track; the model has no temporal tracking component (Sec. 3.2–3.4).

10. **Occlusion / visibility prediction:** No. The model outputs only disparity. Standard benchmarks are evaluated on non-occluded regions (Sec. 4.2, p. 6: "evaluations are performed on half resolution and non-occluded regions"), and no per-pixel or per-track occlusion/visibility flag is predicted.

11. **Multiple views:** Yes — fixed 2 views (rectified stereo pair). The model simultaneously takes a left and right rectified image as a fixed stereo pair, constituting exactly two simultaneous camera views. This is not a monocular temporal sequence (Fig. 2, p. 4; Sec. 3.1, p. 3).

12. **Global context vs. pairwise vs. fixed-window:** Pairwise. FoundationStereo processes a single left/right stereo pair per inference call. Within that pair, the Disparity Transformer performs full self-attention over the entire disparity dimension of the 4D cost volume ("providing long range context for global reasoning," Sec. 3.2, p. 5), and the APC hourglass aggregates spatially — but this is all within one fixed stereo pair, not across a sequence or arbitrary views. The inference regime is therefore pairwise (one stereo pair at a time) with global attention within the pair.

13. **No GT extrinsics required at inference:** Yes. The network inputs are only rectified left and right RGB images; no extrinsic parameters (rotation, translation) are passed to the model (Sec. 3.1, p. 3; Sec. 4.1, p. 6). Note: rectification of raw stereo cameras requires external calibration, but all evaluation benchmarks supply pre-rectified pairs.

14. **Extrinsics as output:** No. The model outputs a disparity map only; no camera pose or extrinsic parameters are predicted (Fig. 2, p. 4; Sec. 3.2–3.4).

15. **No GT intrinsics required at inference:** Yes. The model takes RGB image pairs and outputs disparity in pixels with no intrinsic parameters (focal length, principal point) as network inputs (Sec. 3.1, p. 3; Sec. 4.1, p. 6). Metric depth conversion requires focal length and baseline externally, but this is outside the model.

16. **Intrinsics as output:** No. The model predicts disparity only; no focal length, principal point, baseline, or other intrinsic parameters are output (Fig. 2, p. 4; Sec. 3.4, p. 5).

17. **Moving cameras:** No. FoundationStereo processes a single static stereo pair with no temporal component; it does not model camera ego-motion, predict poses, or maintain consistency across a moving-camera sequence. It can be applied frame-by-frame to stereo video from a moving platform, but this is no different from applying any single-frame method.

18. **Dynamic content:** No. The model processes a single instantaneous stereo pair and predicts static per-pixel disparity. It has no mechanism for handling scene dynamics, scene flow, or non-rigid motion. The presence of "dynamically animated digital humans" in the synthetic FSD training data (Supplement Sec. 11, p. 14) refers to appearance diversity in static renders, not temporal dynamic modeling.

19. **Feed-forward inference:** Yes. Inference runs with fixed, pre-trained weights: 32 GRU refinement iterations at inference (Sec. 4.1, p. 6) are a fixed recurrent forward pass through trained parameters — no per-scene optimization or test-time training occurs. Iterative GRU refinement (as in RAFT, RAFT-Stereo) is the established feedforward paradigm in this field.

**Notes:** FoundationStereo is a stereo disparity estimator, not a general multi-view or monocular 3D reconstructor. It requires rectified stereo image pairs as input (fixed 2-camera setup). Metric 3D output requires known baseline and focal length (supplied externally or from the stereo rig calibration). The model achieves zero-shot generalization across domains without fine-tuning, demonstrated on Middlebury, ETH3D, KITTI-12, KITTI-15, Scene Flow, Booster (transparent/specular objects), and in-the-wild images (Sec. 4.3–4.4 and Supplement Secs. 6–10). Fine-tuning further boosts performance to state-of-the-art on ETH3D and Middlebury leaderboards (Supplement Secs. 6–7). Limitations: not optimized for efficiency (0.7 s per image at 375×1242 on A100); limited transparent-object coverage in training data (Sec. 5, p. 8).
