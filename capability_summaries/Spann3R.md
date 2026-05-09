**Paper:** 3D Reconstruction with Spatial Memory, arXiv August 2024, Spann3R
**Summary:** Spann3R is a DUSt3R-based transformer model for dense 3D reconstruction from ordered or unordered RGB image collections without known camera parameters. It processes images sequentially and predicts per-image dense pointmaps expressed in a single global coordinate system (the initial frame's coordinate system) by querying an external spatial memory that accumulates all previous predictions. Inference is purely feed-forward with no test-time optimization, enabling real-time online reconstruction; outputs are scale-ambiguous and the method targets static scenes captured by moving cameras.

1. **3D reconstruction:** Yes. The paper describes Spann3R as a method for "dense 3D reconstruction from ordered or unordered image collections" that directly regresses per-image pointmaps (dense per-pixel 3D point fields) expressed in a common global coordinate system (Abstract, p. 1; Sec. 3, p. 3). Experiments evaluate predicted dense pointmaps against back-projected per-point depth on 7Scenes, NRGBD, and DTU (Sec. 4.1, pp. 5–6).

2. **Dense reconstruction:** Yes. The network output head generates a per-pixel pointmap and confidence map — dense 3D point fields rather than sparse keypoints (Sec. 3.1, p. 4, Eq. 5). The evaluation directly compares predicted dense pointmaps with back-projected per-point depth, excluding invalid and background points (Sec. 4.1, p. 6).

3. **Reference frame:** Fixed world. All pointmap outputs are accumulated in a single world frame established by the first processed frame via the spatial memory; the world frame is hard-coded and not user-selectable.

4. **Metric scale:** No. The evaluation states explicitly: "the reconstruction is up to an unknown scale, we align the reconstruction following DUSt3R" before comparison (Sec. 4.1, p. 6). The confidence loss normalizes predicted and ground-truth pointmaps by their average distances, and the supplementary scale loss (Eq. 13, Supp. Sec. 6, p. 9) only prevents trivially large scale rather than enforcing absolute metric scale.

5. **Point tracking:** No. Spann3R predicts dense pointmaps and confidence maps. The spatial-memory attention links query tokens to memory tokens internally, and the paper visualizes attention weights (Fig. 8, p. 7), but explicit point correspondences, matches, or persistent point identities across frames are never defined or output (Sec. 3.1–3.2, pp. 3–4; Sec. 4.3, p. 7).

6. **3D tracks:** No. The method reconstructs each image as a dense pointmap in a common coordinate frame but does not output persistent 3D point trajectories over time. Experiments evaluate reconstruction quality, runtime, and memory behavior, not trajectory outputs (Sec. 3, pp. 3–4; Sec. 4, pp. 5–8).

7. **2D tracks:** No. Spann3R's stated outputs are pointmaps and confidence maps only (Sec. 3.1, p. 4). Since Q6 = No, the projection shortcut for trivially deriving 2D tracks does not apply.

8. **Occlusion / visibility prediction:** No. The per-pixel confidence map is defined through DUSt3R-style confidence-aware regression (Eq. 11–12, Supp. Sec. 6, p. 9) and is used for loss weighting and view selection, not as an occlusion or visibility flag. The paper does not output or evaluate occlusion states or visibility labels.

9. **Multiple views:** No. Spann3R processes images sequentially one at a time, with each forward pass taking a single current frame It and a previous query feature as input (Sec. 3.1, p. 3, Eq. 1). For unordered collections, a DUSt3R-style pairwise confidence graph is used only to determine initialization and processing order before sequential feed-forward inference — this is not simultaneous multi-camera input (Sec. 3.3, p. 5).

10. **Global context vs. pairwise vs. fixed-window:** Hybrid. At inference, the spatial memory comprises two components: a dense working memory of the most recent 5 frames (fixed-window) and a sparse long-term memory that accumulates all previous frames (global). The memory readout attends jointly over tokens from both components (Sec. 3.2, pp. 4–5; Fig. 4, p. 4). This is neither purely fixed-window nor purely global.

11. **No GT extrinsics required at inference:** Yes. Spann3R reconstructs "without any prior knowledge of the scene or camera parameters" — no ground-truth extrinsics are needed as input (Abstract, p. 1; Fig. 1 caption, p. 1; Sec. 5, p. 8).

12. **Extrinsics as output:** No. The model output head produces pointmaps and confidence maps, not per-frame camera extrinsic matrices (Sec. 3.1, p. 4, Eq. 5). The experiments evaluate dense reconstruction quality and do not include camera-pose prediction or extrinsic-estimation outputs (Sec. 4.1–4.2, pp. 5–6).

13. **No GT intrinsics required at inference:** Yes. The paper states that Spann3R operates "without any prior knowledge of the scene or camera parameters," which includes intrinsics (Abstract, p. 1; Sec. 3.1, pp. 3–4). Intrinsic matrices are not listed among inference inputs.

14. **Intrinsics as output:** No. The output head generates a pointmap and confidence only (Sec. 3.1, p. 4, Eq. 5). The paper includes no focal-length or intrinsic-parameter prediction objective or evaluation (Sec. 4, pp. 5–8).

15. **Moving cameras:** Yes. Spann3R is designed for video sequences and image collections with changing viewpoints and is evaluated on 7Scenes, NRGBD, DTU, Map-free Reloc, ETH3D, MipNeRF-360, NeRF, and TUM-RGBD (Sec. 4.1, p. 5; Fig. 9, p. 8). Limitations explicitly discuss failure cases as the camera continuously moves forward or traverses multi-room scenes, confirming moving-camera operation is the core use case (Sec. 4.4, p. 8).

16. **Dynamic content:** No. Spann3R accumulates a single common-coordinate reconstruction from previous frames and does not model independently moving or non-rigid scene content. Experiments, training data, and limitations address static indoor/outdoor and object-level scenes only (Sec. 3, pp. 3–4; Sec. 4.2–4.4, pp. 6–8).

17. **Feed-forward inference:** Yes. The paper states that "the 3D reconstruction of each image can be solved by a simple forward pass with a transformer-based architecture" and explicitly eliminates "optimization-based global alignment" and "test-time optimization" (Fig. 1 caption, p. 1; Abstract, p. 1; Sec. 5, p. 8). Default online reconstruction runs at approximately 65 fps with 11 GB GPU memory (Sec. 4.2, p. 6). For unordered collections, optional confidence-based order selection uses pairwise confidence but this is a preprocessing heuristic, not per-scene optimization.

**Notes:** Spann3R takes monocular RGB images as input (video or unordered collections). It inherits architecture and pretrained weights from DUSt3R (ViT-large encoder, ViT-base decoders) and is fine-tuned at 224x224 resolution on subsets of Habitat, ScanNet++, ARKitScenes, BlendedMVS, and Co3D-v2 (Sec. 4.1, p. 5; Sec. 6 Supp., p. 9). Training uses 5-frame sequences with curriculum sampling window size; at inference, long-term memory accumulates over all past frames with XMem-style sparsification. Key limitations: scale ambiguity, drift without bundle adjustment, loop-closure failures, limited multi-room generalization, and reliance on posed RGB-D training data (Sec. 4.4, p. 8). Although it handles large image collections, processing is always sequential/incremental, not simultaneous multi-view. The model outputs confidence maps (used for view ordering and loss weighting) that are not occlusion flags.
