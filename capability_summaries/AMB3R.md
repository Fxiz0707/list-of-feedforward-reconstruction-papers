**Paper:** AMB3R: Accurate Feed-forward Metric-scale 3D Reconstruction with Backend, arXiv Nov 2025, AMB3R
**Summary:** AMB3R is a feed-forward multi-view model for dense, metric-scale 3D reconstruction that extends a frozen VGGT front-end with a newly trained sparse-voxel backend transformer and a lightweight metric-scale head. It takes an arbitrary number of RGB images as input and produces per-pixel pointmaps, metric depth, and camera poses in a single forward pass without any per-scene optimization or test-time fine-tuning. The system extends seamlessly to uncalibrated visual odometry (online) and structure-from-motion (large-scale, unordered) via keyframe memory and image-clustering strategies, again without task-specific optimization or bundle adjustment.

1. **3D reconstruction:** Yes. AMB3R is described as "a multi-view feed-forward model for dense 3D reconstruction on a metric-scale" (Abstract, p. 1). The front-end predicts per-pixel pointmaps fused into a sparse voxel grid by the backend, which is unserialized back to dense 3D point clouds. 3D reconstruction is benchmarked on ETH3D, DTU, and 7-Scenes (Sec. 5.5, Table 5) and illustrated in Figs. 3, 9–13.

2. **Dense reconstruction:** Yes. The model inherits the pointmap paradigm from VGGT: every pixel in each input image receives a predicted 3D coordinate (per-pixel 2D-to-3D regression). Sec. 3 (Preliminaries) defines per-image dense pointmaps; Sec. 4.2 fuses per-pixel features into voxels; Sec. 4.3 trains with both depth and pointmap losses.

3. **Reference frame:** Selectable. The world frame is set by the first-designated image; by choosing which image is placed first, the user can designate any frame as the reference, making both world-space and egocentric output available.

4. **Metric scale:** Yes. Metric-scale recovery is a primary contribution (Sec. 4.1, "Metric-scale Reconstruction"). A scale head maps VGGT encoder features and DPT depth features via a 3-layer MLP and 2-layer convolution to per-frame metric log-depth, supervised with an L1 loss. At inference, per-frame scales are estimated and their median aligns the full reconstruction to metric space. Evaluated on NYUv2, KITTI, ETH3D, ScanNet, DIODE (Table 1) and metric-scale multi-view depth (Table 4).

5. **Point tracking:** No. AMB3R does not establish or output explicit point correspondences or tracks across frames. Sec. 4.3 explicitly states the training loss excludes the tracking loss used in VGGT: the loss is "depth, pointmap, and camera pose (except tracking loss)" (p. 4). Confidence-weighted voxel fusion aggregates geometry but does not define or output a track graph.

6. **3D tracks:** No. 3D point trajectories are not defined, described, or evaluated anywhere in the paper.

7. **2D tracks:** No. 2D point tracks are not defined, described, or evaluated. (Q6=No, so the Auto-Yes rule does not apply.)

8. **Occlusion / visibility prediction:** No. The model outputs per-pixel confidence scores C_t (Eq. 5), trained with a confidence-aware loss. These reflect reconstruction reliability/validity rather than occlusion or cross-view visibility status. No per-point occlusion or visibility flag is defined or evaluated in any section.

9. **Multiple views:** Yes, arbitrary number of views. The front-end formulates input as {I_t}_{t=1}^T and uses global attention across all T frames simultaneously (type-(c) encoding, Eq. 4, Sec. 3). Training uses 5–16 frames per sample (Sec. 4.3). In VO mode, up to 8 new frames plus up to 10 active keyframes are processed jointly per window. In SfM mode, image clusters of adaptive size are processed together. All modes take images from a single moving camera, not a synchronized multi-camera rig.

10. **Global context vs. pairwise vs. fixed-window:** Hybrid. The front-end uses Global attention across all frames in the current input set (type-(c) paradigm, explicitly distinguished from pairwise and fixed-window in Sec. 3). The VO extension adds a sliding Fixed-window with active keyframe memory (Sec. 4.4), and the SfM extension uses image clustering with hierarchical global mapping (Sec. 4.5). The base model alone is Global; the full extended system (which the paper presents as an integrated contribution) is Hybrid.

11. **No GT extrinsics required at inference:** Yes. A central claim throughout: "uncalibrated visual odometry without the need for fine-tuning or an optimization-based backend" (Abstract, p. 1). Table 16 marks AMB3R with "X" (not used) for GT poses. Camera poses are predicted outputs, not inputs.

12. **Extrinsics as output:** Yes. Camera pose estimation is a primary evaluated task (Sec. 5.3, Table 2: AUC@30 on RealEstate10K). VO and SLAM evaluations (Tables 7–11) report ATE RMSE on predicted camera trajectories. Sec. 4.3 includes a camera-pose loss in training, and the VO system explicitly maintains per-frame rotation R and normalized translation τ (Eq. 11).

13. **No GT intrinsics required at inference:** Yes. The paper explicitly frames AMB3R's VO/SLAM mode as "uncalibrated" with "unknown camera intrinsics" (Sec. 5.7, p. 7). Table 16 marks AMB3R with "X" for GT intrinsics (not used). AMB3R is contrasted with MVS baselines that require known calibration, poses, and intrinsics.

14. **Intrinsics as output:** No. Although the frozen VGGT front-end encodes camera geometry internally, AMB3R's method description and training loss (Sec. 4.3) mention only depth, pointmap, and camera-pose (extrinsic) outputs. No intrinsics output or calibration evaluation is presented for AMB3R. The paper's "uncalibrated" framing confirms intrinsics are not required as input, but this does not imply they are explicitly output.

15. **Moving cameras:** Yes. All VO, SLAM, SfM, and multi-view depth experiments involve freely moving cameras. Moving-camera scenarios are the primary intended use case (Secs. 5.3–5.8, evaluated on RealEstate10K, TUM, ETH SLAM, 7-Scenes, ETH3D, Tanks&Temples, IMC Phototourism).

16. **Dynamic content:** Partial. AMB3R was not trained on dynamic scenes and the paper explicitly states: "Since our model has not been trained for dynamic scenarios, it heavily relies on statics cues for dynamic environment. Thus, it might fail when the target scene is dominated by dynamic objects" (Sec. 7.1, p. 9). Despite this, the model is evaluated on TUM Dynamic (Table 11) where it outperforms MUSt3R, and on Sintel/Bonn/KITTI video depth (Table 6). Dynamic handling is an emergent generalization result, not a design target.

17. **Feed-forward inference:** Yes. The central architectural commitment: "no task-specific fine-tuning or test-time optimization" (Abstract, Fig. 1 caption, Conclusion, p. 8). The front-end (frozen VGGT) and the backend transformer are both applied feed-forward. The VO and SfM extensions use keyframe selection, coordinate alignment, and fusion heuristics, but none of these involve gradient updates, bundle adjustment, or per-scene optimization — they are all deterministic feed-forward operations. This is confirmed by the comparison to optimization-based methods (COLMAP, MASt3R-SfM, ACE-Zero) which AMB3R surpasses without any optimization.

---

**Notes:**
- **Input modality:** Monocular RGB images or video from a single moving camera. The "multi-view" capability is multiple frames from one camera over time, not a synchronized multi-camera rig.
- **Front-end freezing:** Only the backend transformer and metric-scale head are trained (~80 H100 GPU hours total). VGGT weights are frozen, dramatically reducing training cost relative to training from scratch. Zero-convolution layers (ControlNet-style) inject backend features into frozen front-end decoders.
- **Backend architecture:** Sparse voxel grid (voxel size 0.01 in normalized space) serialized via Hilbert space-filling curves, processed by Point Transformer v3 (U-Net-like). KNN interpolation recovers per-pixel features from voxels.
- **Scale head:** Uses DPT depth branch features from VGGT; ROE solver used as fallback when GT depth values are missing. Scale is estimated per-frame and aligned via median at inference.
- **VO runtime:** ~4.2 FPS average on NVIDIA RTX 4090 at (392, 518) resolution. Computational complexity scales with 3D content (amount of voxels), not number of frames, unlike type-(b) memory-based methods.
- **SfM scale:** Demonstrated on large unordered collections including IMC Phototourism and Cambridge Landmarks (Fig. 13) without optimization-based BA. Divide-and-conquer image clustering handles arbitrarily large image sets.
- **Limitations (Sec. 7):** Backend is a local scene representation and cannot perform joint reasoning across chunks; no explicit loop closure or relocalization in VO mode; performance may degrade on dynamic-object-dominated scenes; mapping window selection by pose distance may miss wide-baseline overlapping views; final camera poses are weighted-average feed-forward predictions and may not satisfy strict pose coherence constraints (unlike BA-optimized poses).
- **Training data (Sec. 8.2):** ScanNet, ScanNetPP, WildRGBD, Mapfree, Aria, Waymo, Virtual Kitti2, GTASfM, MVS-Synth, OminiObject3d, Hypersim. Co3D excluded due to VGGT overfitting concerns.
