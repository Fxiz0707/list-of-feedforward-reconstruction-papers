**Paper:** MASt3R-SfM: a Fully-Integrated Solution for Unconstrained Structure-from-Motion, arXiv October 2024, MASt3R-SfM
**Summary:** MASt3R-SfM is a fully-integrated SfM pipeline that takes an unconstrained, unordered collection of N images of a static scene and jointly recovers per-image camera intrinsics, extrinsics, and dense 3D scene geometry (pointmaps) in a single shared world coordinate system. It builds on a frozen MASt3R model for pairwise local reconstruction and matching, constructs a sparse scene graph via image retrieval, and then aligns all local reconstructions through two successive gradient-descent optimization stages. The full pipeline is not feed-forward: it performs per-scene optimization at inference time.

1. **3D reconstruction:** Yes. Sec. 4 defines per-image pointmaps X^n ∈ R^(H×W×3) that map every pixel to a 3D scene point in world coordinates (p. 3). Appendix A confirms dense outputs of approximately 200K 3D points per image (p. 10), and Fig. 5 shows qualitative dense point clouds on ETH-3D and Tanks&Temples.

2. **Dense reconstruction:** Yes. Although correspondences are sparse, the constrained pointmap χ^n is defined for every pixel via the inverse reprojection function π^(-1)(·) applied to the depth map Z^n. Appendix A (p. 10) explicitly states: "the inverse reprojection function π^(-1)(·) can be used to infer a 3D point for every pixel, i.e. not just those belonging to sparse matches."

3. **Reference frame:** Fixed world. The global optimisation stage aligns all pairwise predictions into a single world-frame point cloud; while the pairwise network always uses the first image as the local reference, the system-level world frame is determined by bundle adjustment initialisation and is not user-reassignable.

4. **Metric scale:** Yes. Sec. 3 describes MASt3R as recovering "(metric) depthmaps" from which "camera intrinsics and (metric) depthmaps can straightforwardly be recovered" (p. 3). MASt3R-SfM inherits this: intrinsics K_n and scale σ_n are jointly optimized, and the per-scene scale normalization (min_n σ_n = 1) is a gauge fix, not a loss of metric accuracy. Evaluations on ETH-3D and Tanks&Temples use metric ATE (Sec. 5.1, p. 6).

5. **Point tracking:** Yes. Pixel-wise correspondences M^(n,m) are established between all graph-edge image pairs via MASt3R's FastNN algorithm (Sec. 3, p. 3; Sec. 4.2, p. 4). Sec. 4.4 further forms pseudo-tracks by tying pixels to anchor points so that correspondences that do not overlap exactly are still coupled across multiple image pairs, enabling multi-image correspondence chains.

6. **3D tracks:** No. The pseudo-tracks (Sec. 4.4, p. 6) are 3D anchor-point constructs used internally in the coarse 3D matching loss (Eq. 3, Sec. 4.3) and 2D reprojection loss (Eq. 4, Sec. 4.4). The system output is a static SfM reconstruction — pointmaps and camera poses — not explicit 3D point trajectories over time.

7. **2D tracks:** No. Pairwise 2D pixel correspondences M^(n,m) and pseudo-tracks are internal optimization constructs; they are not delivered as a primary output product (multi-frame 2D tracks for downstream use). No 2D tracking output is described or evaluated.

8. **Occlusion / visibility prediction:** No. The described outputs are pointmaps, depths, camera intrinsics and extrinsics, and sparse correspondences with confidence scores (Secs. 3–4, pp. 3–5). MASt3R's confidence maps reflect match quality, not per-point occlusion or visibility flags. No occlusion/visibility prediction is described anywhere in the paper.

9. **Multiple views:** Yes — arbitrary number. Sec. 4 frames the input as "an unordered collection of N images" with N ranging from 1 to more than 1000 (Abstract, p. 1; Sec. 4, p. 3). MASt3R-SfM simultaneously processes the full image collection through the scene graph and global optimization, and is evaluated across a wide range of N. The number of views is arbitrary and not fixed.

10. **Global context vs. pairwise vs. fixed-window:** Hybrid. MASt3R performs pairwise local 3D reconstruction for each graph edge (Sec. 4.2, p. 4). MASt3R-SfM then performs two stages of global gradient-descent optimization (coarse alignment Sec. 4.3 and 2D refinement Sec. 4.4) that jointly optimize all cameras and depths across the entire image collection.

11. **No GT extrinsics required at inference:** Yes. The fundamental goal of the SfM pipeline is to recover camera poses P_n from images alone without any pose priors (Sec. 4, p. 3; Abstract, p. 1). No ground-truth extrinsics are taken as input at any stage.

12. **Extrinsics as output:** Yes. Camera poses P_n ∈ R^(4×4) (rotation R_n and translation t_n) are explicit optimization variables (Fig. 3, p. 5) and primary outputs. All evaluations measure pose accuracy — RTA, RRA, ATE, registration rate — against ground-truth extrinsics (Sec. 5, pp. 6–8).

13. **No GT intrinsics required at inference:** Yes. Canonical intrinsics K_n are initialized from MASt3R canonical pointmaps via the Weiszfeld algorithm (Sec. 4.2, p. 4) and further refined during optimization (Eq. 4, Sec. 4.4). The paper notes that typically a shared per-scene focal length is optimized without providing any GT intrinsics (Sec. 5.1, p. 6).

14. **Intrinsics as output:** Yes. K_n ∈ R^(3×3) (focal length and principal point under a pinhole model) are explicit optimization variables (Fig. 3, p. 5) and outputs. Shared vs. per-camera intrinsics are ablated in Sec. 5.3 (p. 8).

15. **Moving cameras:** Yes. The method is designed for images captured by freely moving cameras with unknown poses (Sec. 4, p. 3). It is evaluated on video-derived datasets, unordered photograph collections, sparse/random frame samplings, and even purely rotational camera configurations with zero translation (Fig. 1 bottom, p. 2; Appendix C, p. 10).

16. **Dynamic content:** No. Sec. 4 explicitly assumes "an unordered collection of N images of a static 3D scene" (p. 3). The method models no moving or non-rigid scene content and is not evaluated on dynamic scenes.

17. **Feed-forward inference:** No. The underlying MASt3R model is run in feed-forward mode per image pair, but MASt3R-SfM is not feed-forward overall: it performs per-scene gradient-descent optimization in two stages — coarse 3D alignment (ν₁ = 300 Adam iterations) and 2D reprojection refinement (ν₂ = 300 Adam iterations) — whose runtime ranges from ~8 to ~29 GB GPU memory and 8–22 minutes depending on configuration (Sec. 4.3–4.4, p. 5; Sec. 5.1, p. 6; Table 4).

**Notes:** MASt3R-SfM is training-free given an off-the-shelf MASt3R checkpoint; no fine-tuning is performed (Sec. 5.1, p. 6). Input is RGB images only — no depth sensors, no IMU, no GPS. The pinhole camera model is assumed with centered principal point and square pixels for canonical intrinsic recovery (Sec. 4.2, p. 4); the approach could be extended to other camera models in principle. The method completely eliminates RANSAC, relying instead on MASt3R's robustness to outlier matches (Abstract; Sec. 4.3, p. 5). It handles purely rotational scenes (zero translation) where traditional triangulation-based SfM fails (Fig. 1 bottom; Appendix C). Failure cases are caused by outlier matches between similar-looking parts of the same scene (Appendix C failure cases, p. 10). Complexity is quasi-linear in N (from O(N) scene graph edges) rather than the quadratic complexity of naive all-pairs matching.
