**Paper:** DePT3R: Joint Dense Point Tracking and 3D Reconstruction of Dynamic Scenes in a Single Forward Pass, arXiv 2026, DePT3R
**Summary:** DePT3R is a feed-forward framework that jointly performs dense 3D reconstruction and dense 3D point tracking of dynamic scenes from unposed monocular RGB video in a single forward pass. It extends the VGGT backbone with a dedicated motion head and an optional intrinsic embedding, predicting per-pixel pointmaps, depth maps, motion maps (3D tracks to a query frame), uncertainty maps, and camera extrinsics/intrinsics simultaneously. It is trained exclusively on synthetic datasets and evaluated on dynamic-scene benchmarks including PointOdyssey, DynamicReplica, Panoptic Studio, and TUM RGB-D.

1. **3D reconstruction:** Yes. The model predicts per-pixel pointmaps and depth maps for all input frames expressed in the first frame's coordinate system (Sec 3.1–3.2, Fig 2). Quantitative 3D reconstruction evaluation is reported on PointOdyssey and TUM RGB-D (Table 3, Sec 4.2).

2. **Dense reconstruction:** Yes. Sec 3.2 states the model predicts "two pixel-wise maps" for each frame — the pointmap and the motion map — covering every pixel. A dedicated depth DPT head also produces per-pixel depth (Sec 3.2, Fig 2). Sec 5 describes "dense tracking of all visible points in a single forward pass."

3. **Reference frame:** Fixed world. All motion estimates are expressed relative to the first-frame coordinate system, which is fixed as the world frame; there is no mechanism to reassign the anchor to a different frame.

4. **Metric scale:** No. Evaluation uses global median scale alignment before computing APD and EPE metrics (Eq. 8, p.6; Table 2, Table 3 captions). The alignment factor s is computed post-hoc from predictions vs. ground truth. No claim of metric-scale output is made; Sec 5 (Limitations) identifies absolute-scale behavior as an open issue.

5. **Point tracking:** Yes. The motion head predicts a motion field ^1M_q^t that "directly maps points from time of frame t to the time of query frame q," establishing correspondences without frame-to-frame chaining (Sec 3.2, p.4). Evaluated on PointOdyssey, DynamicReplica, and Panoptic Studio (Table 2).

6. **3D tracks:** Yes. The motion map ^1M_q^t = ^1X̂_q^t − ^1X̂_t^t gives 3D positions of tracked points at query time, producing 3D point trajectories in the first frame's coordinate system (Sec 3.2, p.4). 3D tracking is the primary evaluation task (Table 2, Sec 4.2).

7. **2D tracks:** Yes. Q6=Yes and the model outputs camera intrinsics and extrinsics (Q12=Yes, Q14=Yes), making projection of 3D tracks to 2D trivial. Additionally, Fig 1 and Fig 3 show projected 2D trajectory visualizations as part of the paper's qualitative results.

8. **Occlusion / visibility prediction:** No. The output heads (Fig 2) are: pointmap, depth, motion, and camera — no occlusion or visibility flag head is described or evaluated. Visibility is used only in training data generation to filter non-visible mesh vertices (Sec 4.1), not as a model output.

9. **Multiple views:** No. The method operates on unposed monocular video — N temporal frames from a single moving camera. Per the disambiguation rules, monocular video does not count as multi-view/multi-camera input. The paper consistently describes the input as "monocular image sequences" (Abstract, Sec 3, Sec 5).

10. **Global context vs. pairwise vs. fixed-window:** Global. The model uses a "globally aggregated transformer backbone" where global attention layers process all N input frames simultaneously (Sec 3.2, p.4; Introduction). The paper explicitly contrasts this with pairwise processing (DUSt3R) and fixed-window approaches, and notes the model can process large frame counts in a single forward pass without windowing (Sec 4.2, p.7).

11. **No GT extrinsics required at inference:** Yes. The abstract explicitly states "DePT3R operates without requiring camera poses," and the Introduction confirms it works "without requiring auxiliary depth or external pose inputs" (Abstract p.1, Introduction p.2).

12. **Extrinsics as output:** Yes. The camera head processes camera tokens through four self-attention layers followed by a linear projection to predict normalized camera extrinsics and intrinsics (Sec 3.2, p.4; Fig 2). Supervised by camera loss L_camera (Eq. 2).

13. **No GT intrinsics required at inference:** Yes. The intrinsic embedding (Sec 3.3) takes focal lengths and principal point y-coordinate as optional input, but the ablation in Table 4 shows a "No intrinsic embedding" variant that still runs and produces valid (if degraded) results. The model can operate at inference without GT intrinsics; providing them improves accuracy.

14. **Intrinsics as output:** Yes. The camera head predicts both normalized camera extrinsics and intrinsics (Sec 3.2, p.4; Fig 2). Supervised jointly with extrinsics via the camera loss (Eq. 2).

15. **Moving cameras:** Yes. The method targets unposed monocular sequences where the camera moves freely. Training datasets (PointOdyssey, DynamicReplica, Virtual KITTI 2, TartanAir, Kubric) all feature camera motion. TUM RGB-D evaluation involves significant camera motion (Sec 4.1–4.2).

16. **Dynamic content:** Yes. The central contribution is handling dynamic scenes with moving/non-rigid content. The motion head is specifically designed to model scene dynamics, and all primary evaluation benchmarks (PointOdyssey, DynamicReplica, Panoptic Studio) contain non-rigid moving objects (Title, Abstract, Sec 3.2, Sec 4).

17. **Feed-forward inference:** Yes. The title, abstract, and conclusion all state operation "in a single forward pass" with no per-scene optimization. Sec 4.2 (p.7) contrasts DePT3R with methods requiring windowing or global optimization for extended sequences.

**Notes:** Input modality is unposed monocular RGB video (N frames, 2–10 during training). The output reference frame is always the first input frame's camera coordinate system — this is effectively a world frame, not an egocentric per-frame output. All reported 3D metrics use global median scale alignment (not metric scale). The motion head predicts displacement to a specified query frame rather than a continuous trajectory function; quantitative tracking evaluation is restricted to trajectories originating from the first frame only. The intrinsic embedding optionally incorporates known f_x, f_y, p_y for better scale disambiguation. The model is trained exclusively on synthetic datasets but generalizes to real-world benchmarks. A key practical advantage is memory efficiency: DePT3R handles 268k dense query points at 12 GB GPU memory vs. competitors hitting OOM at 22.5k–40k points.
