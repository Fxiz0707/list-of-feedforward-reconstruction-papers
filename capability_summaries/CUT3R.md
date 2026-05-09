**Paper:** Continuous 3D Perception Model with Persistent State, arXiv Jan 2025, CUT3R
**Summary:** CUT3R (Continuous Updating Transformer for 3D Reconstruction) is a feed-forward recurrent model for online dense 3D perception from a stream of RGB images (videos or unordered photo collections). It maintains a persistent latent state that is updated with each incoming image and read out to produce metric-scale per-pixel pointmaps in both the current camera frame and a shared world frame, together with camera extrinsics, without requiring any camera calibration at inference. The model handles both static and dynamic scenes and achieves competitive or state-of-the-art performance on monocular depth estimation, camera pose estimation, and 3D reconstruction benchmarks while operating fully online.

1. **3D reconstruction:** Yes. Per-pixel pointmaps predicted in a common world coordinate frame accumulate into a coherent, dense scene reconstruction as new images arrive. The 7-Scenes and NRGBD 3D reconstruction benchmarks are evaluated in Sec. 4.3 (Tables 4–5, pp. 6–7). The abstract explicitly describes the output as "dense 3D geometry with each incoming frame" (Abstract, p. 1).

2. **Dense reconstruction:** Yes. The core output is dense per-pixel pointmaps (one 3D point per pixel) with associated confidence maps, not sparse keypoints. Both depth evaluation (Sec. 4.1) and 3D reconstruction evaluation (Sec. 4.3) operate on this per-pixel representation (Eqs. 3–4, p. 3; Tables 1–4, pp. 5–7).

3. **World-space output:** Yes. The model explicitly predicts world-frame pointmaps X̂_t^world via Head_world, where the world frame is fixed to the coordinate frame of the initial image. Accumulated world-frame pointmaps form the global reconstruction (Sec. 3.1, p. 3; Eq. 4, p. 3; Fig. 3, p. 4).

4. **Camera-space output (egocentric):** Yes. The model simultaneously predicts self-frame pointmaps X̂_t^self via Head_self in each input image's own camera coordinate frame (Sec. 3.1, p. 3; Eq. 3, p. 3). Both self and world pointmaps are standard per-frame outputs.

5. **Flexible camera frame:** No. The camera-space output (Q4) is always in the current input image's own frame — it is not freely selectable. The world frame is fixed to the initial image's coordinate frame. The virtual-view raymap query (Sec. 3.2) can read the state for an arbitrary virtual camera, but that is a separate "state readout" mechanism, not the per-frame standard output pipeline (Sec. 3.1–3.2, pp. 3–4).

6. **Metric scale:** Yes. The paper states explicitly: "All pointmaps and poses are in metric scale (i.e., meters)" (p. 4). The model is trained with metric-scale 3D annotations and evaluated under metric-scale conditions in Tables 2 and 7 (Sec. 3.3, p. 4; Sec. 4.1, pp. 5–6). Training on non-metric datasets uses scale-normalized supervision.

7. **Point tracking:** No. The paper does not describe or evaluate point tracking. World-frame pointmaps implicitly place different frames in the same coordinate system, but no correspondence, match field, or tracking output is produced or evaluated. PointOdyssey (a tracking dataset) appears in training data (Table 6, p. 16), but no tracking metrics are reported in any experiment.

8. **3D tracks:** No. No 3D point trajectory outputs are described, produced, or evaluated anywhere in the paper or appendix (Sec. 3.1–4.4, pp. 3–8).

9. **2D tracks:** No. Q8 = No and no 2D tracking output or evaluation is present in the paper. The model's outputs are self/world pointmaps, confidence maps, camera pose, and optional raymap-query readouts (Sec. 3.1–3.2, pp. 3–4).

10. **Occlusion / visibility prediction:** No. The model produces confidence maps C_t^self and C_t^world alongside pointmaps, but these are regression confidence weights (used in the training loss, Eq. 7, p. 5) and are not described as occlusion or visibility flags. No per-point occlusion prediction is evaluated (Sec. 3.1, p. 3; Sec. 3.3, p. 4).

11. **Multiple views:** No. The model ingests images sequentially one at a time as a stream; each incoming image interacts with the persistent state via separate state-update and state-readout operations (Sec. 3.1, p. 3; Fig. 3, p. 4). There is no simultaneous multi-camera input mode. Support for photo collections is achieved by sequential online processing of unordered images, not parallel multi-view ingestion.

12. **Global context vs. pairwise vs. fixed-window:** Global. The persistent recurrent state aggregates compressed information from all previously seen frames, and each new image reads from this accumulated global context. There is no fixed window cutoff — all past observations are encoded in the state. The paper notes potential drift on very long sequences due to the absence of explicit global alignment (Sec. 3.1, p. 3; Limitations, p. 9).

13. **No GT extrinsics required at inference:** Yes. The paper explicitly states the method eliminates "the need for known camera extrinsics or intrinsics" and takes raw image streams without any camera information (Sec. 1, p. 2; Sec. 3, p. 3). Camera pose estimation experiments confirm no GT extrinsics are provided (Sec. 4.2, p. 6; Table 3, p. 6).

14. **Extrinsics as output:** Yes. The model predicts the 6-DoF ego-motion P̂_t (rigid transformation from current frame to world frame) via Head_pose applied to the pose token z'_t (Eq. 5, p. 3–4). Camera pose estimation on Sintel, TUM-dynamics, and ScanNet is a full evaluation task (Sec. 4.2, Tables 3 and 8, pp. 6, 17).

15. **No GT intrinsics required at inference:** Yes. Same as Q13 — no camera calibration of any kind is required at inference (Sec. 1, p. 2; Sec. 3, p. 3; Sec. 4.2, p. 6).

16. **Intrinsics as output:** No. The method section specifies three prediction heads: Head_self (self pointmap), Head_world (world pointmap), and Head_pose (6-DoF extrinsic pose). No intrinsics output head is defined in the architecture, and intrinsics prediction is never evaluated. The informal mention of "camera parameters" in the abstract and introduction (pp. 1–2) refers to extrinsics and implicitly encoded geometry in pointmaps, not an explicit intrinsics output (Sec. 3.1, Eqs. 3–5, pp. 3–4).

17. **Moving cameras:** Yes. CUT3R is designed for moving camera video streams and predicts per-frame ego-motion. It is evaluated on camera pose estimation benchmarks (Sintel, TUM-dynamics, ScanNet) with moving cameras and video depth benchmarks with non-static viewpoints (Sec. 4.1–4.2, pp. 5–6; Tables 2–3).

18. **Dynamic content:** Yes. Dynamic scenes are an explicit target. The abstract notes support for "both static and dynamic content" (Abstract, p. 1). Training stage 2 adds dynamic scene datasets (Sec. 3.4, p. 5). Evaluations on TUM-dynamics (moving objects) and Sintel (dynamic content) are included, and Fig. 1 shows "Dynamic Scene" as a primary use case.

19. **Feed-forward inference:** Yes. CUT3R processes each incoming frame in a single forward pass (encoder + two decoders + heads) with no per-scene optimization or test-time training. The recurrent state update is part of the forward pass. Online FPS is reported (16.58 FPS, Tables 2 and 4). The "Ours Revisit" variant in Sec. 4.4 runs a second sequential pass over the same images with a frozen state and improves accuracy, but this is an analysis variant, not the primary inference mode (Sec. 4.4, p. 7–8; Tables 4–5).

**Notes:** Input modality is RGB images only (no depth, no IMU, no calibration). The model is trained on 32 diverse datasets with mixed static/dynamic, indoor/outdoor, metric/non-metric, and partial-annotation regimes; dense depth annotations for some datasets are generated via MVS reconstruction. The virtual-view state readout (Sec. 3.2) accepts a raymap encoding arbitrary camera intrinsics and extrinsics to query the persistent state for unseen regions — this enables novel-view structure inference but requires the user to supply the query camera geometry as a raymap. Raymap training is disabled for non-metric datasets to avoid scale inconsistency. The model may drift on very long sequences due to the absence of explicit loop closure or global alignment. Training uses a three-stage curriculum: static scenes first, then dynamic, then long-context fine-tuning on multi-view data at 512px resolution.
