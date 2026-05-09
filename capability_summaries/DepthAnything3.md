**Paper:** Depth Anything 3: Recovering the Visual Space from Any Views, arXiv Nov 2025, DepthAnything3
**Summary:** Depth Anything 3 (DA3) is a single feed-forward transformer model that takes an arbitrary number of images (one, multiple unordered views, or video frames), optionally with known camera poses, and predicts dense pixel-aligned depth maps plus per-pixel world-frame ray maps. These outputs can be combined into consistent dense point clouds, and the model simultaneously predicts camera intrinsics and extrinsics; the paper also trains separate monocular relative-depth and metric-depth student variants, and demonstrates a feed-forward 3D Gaussian splatting application on top of the geometry backbone.

1. **3D reconstruction:** Yes. DA3 reconstructs dense 3D geometry by combining predicted depth and ray maps via `P = t + D(u,v) * d` to place per-pixel points into world coordinates. The benchmark directly evaluates reconstructed point clouds using Chamfer Distance and F1-score across five datasets (Sec. 3.1, p. 6; Sec. 6.1, p. 12; Fig. 1, p. 3; Fig. 6, p. 16).

2. **Dense reconstruction:** Yes. The model outputs a full per-pixel depth map D ∈ R^{H×W} and a full per-pixel ray map M ∈ R^{H×W×6} for each input view, producing a reconstruction at every image pixel rather than at sparse keypoints (Sec. 3.1, p. 6; Sec. 3.2, p. 7).

3. **Reference frame:** Selectable. Cross-view attention establishes a consistent world frame relative to the first input view; by placing any view first, the user designates the reference frame, providing both world-space and per-frame egocentric output depending on input ordering.

4. **Metric scale:** Partial. The main any-view DA3 model is scale-consistent across all input views (normalization is by a common scene scale factor during training), but it is not inherently metric in absolute units; reconstruction evaluation aligns predicted poses to ground truth before scoring. A separately trained monocular DA3-Metric student achieves true metric scale by applying a canonical camera space transformation following Metric3Dv2 (Sec. 3.3, p. 7; Sec. 4.4, p. 10; Sec. 6.1, p. 12; Sec. 7.4, p. 19).

5. **Point tracking:** No. DA3 predicts dense depth, ray maps, and camera parameters but produces no explicit point correspondences, persistent point identities, or tracking head across views or frames (Sec. 3.1, p. 6).

6. **3D tracks:** No. The paper provides no 3D point trajectory outputs; temporal or cross-view correspondence is not established by the model (Secs. 3–5).

7. **2D tracks:** No. Although DA3 supports video and multi-view inputs, its outputs are depth maps, ray maps, and camera parameters — not 2D point tracks or trajectories (Secs. 3–5).

8. **Occlusion / visibility prediction:** No. DA3 does not output per-point or per-track visibility or occlusion labels. The 3DGS application predicts a per-Gaussian opacity σ_i for rendering, which is not an occlusion or visibility flag for reconstruction or tracking purposes (Sec. 5.1, p. 11).

9. **Multiple views:** Yes. The input is defined as I = {I_i}_{i=1}^{N_v} for arbitrary N_v ≥ 1; for N_v > 1 it is a multi-view or video set. Cross-view self-attention jointly processes all view tokens. Training samples 2–18 views uniformly; at inference the model handles hundreds to thousands of images (limited by GPU memory), as reported in Table 8 (Sec. 3.1, p. 6; Sec. 3.2, p. 7; Sec. 3.4, p. 8; Table 8, p. 18).

10. **Global context vs. pairwise vs. fixed-window:** Global. The input-adaptive cross-view self-attention in the final L_g transformer layers operates jointly over tokens from all N_v input views, enabling full global information exchange in a single forward pass. This is not pairwise or fixed-window processing (Sec. 3.2, p. 7; Sec. 7.2.2, p. 17).

11. **No GT extrinsics required at inference:** Yes. The model explicitly supports "with or without known camera poses." When extrinsics are unavailable, a shared learned camera token c_l is used in place of the encoded camera. Pose-free reconstruction settings are fully evaluated in Table 3 (Abstract, p. 1; Sec. 3.2, p. 7; Table 3, p. 15).

12. **Extrinsics as output:** Yes. Camera extrinsics are derived from the output ray map via DLT and RQ decomposition, and the lightweight camera head D_C additionally predicts rotation quaternion q ∈ R^4 and translation t ∈ R^3 per view. Pose accuracy is a primary evaluation metric across five benchmark datasets (Sec. 3.1, pp. 6–7; Table 2, p. 14).

13. **No GT intrinsics required at inference:** Yes. Intrinsics are not required at inference; in the pose-free setting the model uses the learned camera token and derives intrinsics from the ray map. All benchmark pose-free settings operate without supplied camera parameters (Sec. 3.1, pp. 6–7; Sec. 3.2, p. 7; Table 3, p. 15).

14. **Intrinsics as output:** Yes. The camera head D_C predicts field-of-view parameters f ∈ R^2 per view. Additionally, Sec. 3.1 describes recovering the intrinsic matrix K from the ray-map homography via RQ decomposition of the optimal H* found by DLT (Sec. 3.1, pp. 6–7).

15. **Moving cameras:** Yes. DA3 explicitly targets video streams and multi-view collections captured from moving cameras as primary input modalities. Camera trajectory estimation is evaluated over video datasets including 7Scenes, and Fig. 5 shows camera trajectories recovered from moving-camera video sequences (Abstract, p. 1; Sec. 1, p. 3; Fig. 5, p. 14; Table 2, p. 14).

16. **Dynamic content:** No. The paper does not model or claim support for moving or non-rigid scene content. Dynamic-scene reasoning is explicitly listed as future work (Sec. 8, p. 22). Notably, the ground-truth camera trajectories used in pose evaluation are derived with dynamic objects masked out (Fig. 5 caption, p. 14), further indicating the model is designed for static scenes.

17. **Feed-forward inference:** Yes. DA3 is a single transformer that performs one forward pass over all input views to produce depth maps, ray maps, and camera parameters, with no per-scene optimization or test-time training. The 3DGS application is equally feed-forward (Abstract, p. 1; Sec. 2, p. 5; Sec. 5, p. 11; Table 8, p. 18).

**Notes:** The core representation is dense depth plus dense world-frame ray maps; camera pose and intrinsics are derived or predicted as a lightweight by-product. The main any-view model is scale-consistent across its input set but not metric; true metric depth is only available through the separately trained monocular DA3-Metric variant. The model handles static scenes with moving cameras well but does not address dynamic scene content (stated future work). The feed-forward 3DGS application (Sec. 5) is produced by fine-tuning DA3 with an additional DPT head for Gaussian parameters and should be treated as a downstream demonstration, not a capability of the base model. Training uses a teacher-student paradigm: a monocular teacher (trained on synthetic data) generates pseudo-labels that are RANSAC-aligned to sparse real-world depth for the any-view student. All models are trained exclusively on public academic datasets.
