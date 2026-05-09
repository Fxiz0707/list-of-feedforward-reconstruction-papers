**Paper:** Fast3R: Towards 3D Reconstruction of 1000+ Images in One Forward Pass, arXiv Jan 2025, Fast3R
**Summary:** Fast3R is a transformer-based multi-view reconstruction model that takes N unordered, unposed RGB images and predicts dense per-pixel local and global 3D pointmaps plus confidence maps in a single forward pass. It uses a shared ViT encoder per image followed by a global all-to-all fusion transformer, trained end-to-end with normalized pointmap regression losses on up to 20-view samples, and is designed to scale to 1000+ views at inference without pairwise matching or iterative global alignment.

1. **3D reconstruction:** Yes. Fast3R maps N RGB images to N local and global pointmaps X_L, X_G ∈ ℝ^(N×H×W×3), where each pointmap is a set of 3D locations indexed by pixels, constituting a dense point cloud of the scene (Section 3.1; Figure 3 shows RGB point cloud output; Section 6 summarizes the method as predicting 3D locations for all pixels in a common frame).

2. **Dense reconstruction:** Yes. A pointmap is explicitly defined as "a set of 3D locations indexed by pixels in an image I" with shape H×W×3, i.e., one 3D point per pixel (Section 3.1). DPT decoding heads produce dense local and global pointmaps and confidence maps (Section 3.3).

3. **Reference frame:** Fixed world. All pointmaps are expressed in the coordinate frame of the first input image I_1; the world frame is fixed by architecture; the first image can be changed by reordering inputs.

4. **Metric scale:** No. Training uses a normalized 3D pointwise regression loss (Eq. 2) that independently normalizes predictions and targets by their mean Euclidean distance to the origin: z = (1/|X|) Σ ||x||₂. The paper does not claim metric scale output and provides no metric-scale supervision or evaluation.

5. **Point tracking:** No. Fast3R does not define, train, or evaluate any point tracking capability. Its outputs are pointmaps and confidence maps, with no correspondence indices, match identities, or track outputs. The 4D reconstruction experiment in Appendix F.1 visualizes tracks using ground-truth TAP-Vid-DAVIS annotations (Figure 16 caption: "tracks are visualized using ground-truth track annotations from TAP-Vid-DAVIS"), confirming tracking is not a model output.

6. **3D tracks:** No. Fast3R outputs pointmaps, not 3D point trajectories across time. The 4D reconstruction experiment (Appendix F.1) is purely qualitative and uses ground-truth track annotations for visualization; the model does not predict 3D tracks.

7. **2D tracks:** No. Fast3R produces no 2D point tracks or optical flow. Supplementary Section F.1 explicitly contrasts Fast3R with MonST3R, which uses a separate model for optical flow, while Fast3R retains the pointmap regression architecture.

8. **Occlusion / visibility prediction:** No. Fast3R outputs per-pixel confidence maps (Σ_L, Σ_G), which are used as regression loss weights and for filtering low-confidence frames (Section 5.4). These are confidence scores for pointmap quality, not binary per-point occlusion or visibility flags.

9. **Multiple views:** Yes, arbitrary number. Fast3R takes N unordered, unposed RGB images simultaneously with no assumption on image ordering (Section 3.1: "a set of (N) unordered and unposed RGB images"). It generalizes to N values larger than seen during training (trained on N=20; Section 4.1 reports processing up to 1500 views in a single pass on a single A100).

10. **Global context vs. pairwise vs. fixed-window:** Global. The fusion transformer performs all-to-all self-attention over concatenated encoded patch features from all N input views: "performs all-to-all self-attention. This operation provides Fast3R with full context from all views, beyond the information provided in pairs alone" (Section 3.3). This is explicitly contrasted with DUSt3R's pairwise approach and Spann3R's sliding window.

11. **No GT extrinsics required at inference:** Yes. The model input is defined as "unordered and unposed RGB images" with no camera pose information (Section 3.1; Abstract). Camera extrinsics are not an input to the model.

12. **Extrinsics as output:** Yes. Camera rotation and translation are estimated from the predicted global pointmaps using RANSAC-PnP (Section 4.2: "we estimate the focal length, camera rotation, and camera translation from the predicted global pointmaps"). This procedure uses only Fast3R's own outputs and is evaluated on camera pose benchmarks (Table 1). RANSAC-PnP is a classical deterministic algorithm integrated into the method, not an off-the-shelf learned model.

13. **No GT intrinsics required at inference:** Yes. Fast3R does not require ground-truth intrinsics. Table 1 explicitly notes "Fast3R does not assume known camera intrinsics." The pose estimation pipeline initializes focal length guesses from image resolution (Section 4.2).

14. **Intrinsics as output:** Yes. Focal length (and thus camera intrinsics) are estimated as part of the method's RANSAC-PnP pose estimation procedure, which selects the best focal length guess and computes intrinsic and extrinsic camera matrices (Section 4.2). This is described and evaluated within the paper using only the model's own pointmap outputs.

15. **Moving cameras:** Yes. Fast3R is designed for multi-view reconstruction from moving cameras with arbitrary, unknown camera trajectories. It is evaluated on moving-camera datasets including CO3D (41 object categories, 10 random-sampled views; Table 1), 7-Scenes and Neural RGB-D video trajectories of 500–1500 frames (Table 3), and DTU (Table 4).

16. **Dynamic content:** Partial. The main paper and trained model target static scenes. Appendix F.1 demonstrates that Fast3R can be fine-tuned for 4D (dynamic) reconstruction by swapping training data to PointOdyssey and TartanAir datasets with the same architecture, and shows qualitative results on unseen DAVIS dynamic scenes (Figure 16). This capability is supplementary and qualitative only; the standard released model is for static scenes.

17. **Feed-forward inference:** Yes. The defining contribution of Fast3R is reconstructing N images in a single forward pass without iterative global alignment or per-scene optimization (Abstract; Section 3.3; title). All N pointmaps and confidence maps are predicted simultaneously. Optional bundle adjustment via Gaussian Splatting (GS-BA using InstantSplat) is described in Appendix D and Table 6 as a post-processing step that can further improve poses but is explicitly "not necessary."

**Notes:** Input modality is RGB images only (no depth input). The model is initialized from DUSt3R pretrained weights for the ViT encoder and global DPT head; the local DPT head is trained from scratch. Training data is a mix of real-world object-centric and scene scan datasets: CO3D, ScanNet++, ARKitScenes, Habitat, BlendedMVS, MegaDepth (6 of DUSt3R's 9 datasets). The scale-normalized loss means output is up-to-scale; evaluation uses normalized metrics. Camera extrinsics and intrinsics are recovered via lightweight RANSAC-PnP post-processing on the predicted pointmaps, not via a dedicated neural head. For 3D reconstruction evaluation, local pointmaps are aligned to the global coordinate frame using ICP. At extreme view counts (>300), the paper notes drifting behavior for low-confidence frames. Bundle adjustment (GS-BA via InstantSplat) is an optional post-processing step. Dynamic scene support exists only as a fine-tuned qualitative experiment in the supplementary.
