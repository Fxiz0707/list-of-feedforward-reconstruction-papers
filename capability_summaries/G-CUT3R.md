**Paper:** G-CUT3R: Guided 3D Reconstruction with Camera and Depth Prior Integration, ICLR 2026, G-CUT3R
**Summary:** G-CUT3R is a feed-forward, recurrent 3D scene reconstruction method that extends CUT3R by integrating optional geometric priors — camera intrinsics, extrinsic poses, and/or depth maps — via lightweight modality-specific encoders fused into the decoder through zero-initialized convolutional layers. It takes sequential monocular RGB images (with optional auxiliary inputs) and outputs dense per-pixel 3D pointmaps in a shared world coordinate frame alongside camera poses, all in a single forward pass without per-scene optimization. The model is trained on a diverse mixture of indoor/outdoor static and dynamic datasets and achieves real-time inference speeds.

1. **3D reconstruction:** Yes. The model predicts dense 3D pointmaps X̂ ∈ ℝ^{3×H×W} per frame, evaluated for scene-level 3D reconstruction quality (Accuracy, Completeness, Normal Consistency) on 7-Scenes and NRGBD benchmarks (Sec. 3.3, p. 5; Table 1, p. 7).

2. **Dense reconstruction:** Yes. Outputs are per-pixel (H×W) pointmaps, not sparse keypoints. Every pixel in each input frame receives a predicted 3D point, following CUT3R's dense pointmap regression paradigm (Sec. 3.1–3.3, pp. 3–5).

3. **World-space output:** Yes. All predicted pointmaps are accumulated in a single shared world coordinate frame maintained through the recurrent state token S across frames. Global 3D reconstruction metrics in Table 1 confirm consistency across views (Sec. 3.1, p. 3; Fig. 2, p. 4).

4. **Camera-space output (egocentric):** No. The model outputs pointmaps in a global shared world frame maintained via the recurrent state. No mechanism for outputting per-frame egocentric (camera-local) pointmaps is described or evaluated in the paper.

5. **Flexible camera frame:** N/A (Q4 = No).

6. **Metric scale:** Yes. The paper explicitly states: "For CUT3R and G-CUT3R, metrics are computed without scale alignment, because they estimate metric depth" (Sec. 4.4, p. 8). Video depth evaluation on Bonn and ScanNet uses absolute metrics (Abs. Rel) without scale alignment, confirming true metric-scale output.

7. **Point tracking:** No. The method does not establish or output correspondences or tracks between predicted points across frames. No tracking capability is described, trained for, or evaluated.

8. **3D tracks:** No. No 3D point trajectory output is described or evaluated in the paper.

9. **2D tracks:** No. No 2D point track output is described or evaluated. (Q8 = No, so the auto-Yes rule does not apply.)

10. **Occlusion / visibility prediction:** No. The model predicts per-pixel confidence maps Ĉ ∈ ℝ^{H×W} used in the training loss (Eq. 7, Sec. 3.3, p. 5), but these reflect reconstruction confidence, not per-point occlusion or visibility flags in a tracking sense.

11. **Multiple views:** No. G-CUT3R processes frames sequentially in a recurrent manner — "The views are passed sequentially to the network G" (Sec. 3, p. 3). Training uses short sequences of 4 images (Sec. 3.4, p. 6). This is monocular sequential video processing, not simultaneous multi-camera/multi-view input.

12. **Global context vs. pairwise vs. fixed-window:** Hybrid. The recurrent state token S carries accumulated global scene context across the entire sequence (inherited from CUT3R), while local decoder processing operates on short fixed windows of 4 images during training. At inference, the state propagates globally over arbitrarily long sequences (Sec. 3.1, 3.4, pp. 3, 6).

13. **No GT extrinsics required at inference:** Yes. Camera poses P are one optional prior input within Φ ⊆ {K, P, D}; the model runs without any guidance at all, as shown by the no-guidance baseline rows (G-CUT3R without K, R|t, D) in Tables 1–2 (Sec. 3, p. 3; Sec. 3.4, p. 6).

14. **Extrinsics as output:** Yes. Camera poses P̂ are explicitly predicted outputs; the training loss includes L_pose (Sec. 3.3, p. 5). Pose estimation is evaluated with ATE, RPE trans, and RPE rot on Sintel, TUM dynamics, and ScanNet (Appendix D, Table 5, p. 13).

15. **No GT intrinsics required at inference:** Yes. Camera intrinsics K are an optional prior input; the model operates without them, as shown by the no-K baseline rows in Tables 1–2 (Sec. 3, p. 3; Sec. 3.4, p. 6).

16. **Intrinsics as output:** No. Camera intrinsics are an optional input modality encoded as ray images (Eq. 1–2, Sec. 3.2, pp. 4–5) but are not predicted as outputs. Output heads predict pointmaps, confidences, and camera poses only (Sec. 3.3, p. 5).

17. **Moving cameras:** Yes. The method is designed for video sequences with moving cameras and evaluates pose estimation (ATE, RPE) on Sintel, TUM dynamics, ScanNet, and Waymo — all involving moving cameras (Sec. 4.1, p. 6; Sec. 4.5, p. 8; Table 5, p. 13).

18. **Dynamic content:** Yes. Explicitly evaluated on Bonn (indoor scenes with "highly dynamic objects involving human activities") and Waymo (outdoor scenarios with "moving agents and LiDAR depth data") (Sec. 4.1, p. 6; Tables 2–3, pp. 8–9). Training data includes dynamic datasets (ARKitScenes, TartanAir — dynamic, Table 4, p. 13).

19. **Feed-forward inference:** Yes. The paper describes G-CUT3R as "a novel real-time feed-forward method" (Sec. 1 contributions, p. 2). The recurrently updated state eliminates the need for "computationally expensive global optimization" (Abstract, p. 1; Conclusion, p. 9). Inference is sequential recurrent network execution with no per-scene optimization or test-time training.

**Notes:**
- Input modality: Sequential monocular RGB images, with optionally any combination of camera intrinsics K, extrinsic poses R|t, and/or sparse/dense depth maps D. The model is modality-agnostic and handles missing priors via a unified training strategy that randomly drops modality subsets (Sec. 3.4, p. 6).
- Depth inputs D may be sparse (e.g., from LiDAR) and are paired with binary validity masks M; the model handles incomplete depth natively (Eq. 3, Sec. 3.2, p. 5).
- G-CUT3R is fine-tuned from pre-trained CUT3R weights using only a subset of CUT3R's training data. Only the decoder is modified; the ViT-Large image encoder is preserved, and modality encoders are ViT-Base (Sec. 3.4, p. 6).
- Real-time capable: reported FPS of 13–24 at 224 resolution and 13–18 at 512 resolution on NVIDIA A40 (Table 1, p. 7), substantially faster than Pow3R (0.3 FPS).
- Pose evaluation uses 7-DoF similarity alignment, whereas depth/pointmap evaluation is metric-scale without scale alignment (Sec. 4.4–4.5, p. 8).
- The confidence map Ĉ is used for training regularization only, not as a visibility or occlusion predictor.
- Point tracking, 2D/3D tracks, and intrinsics prediction are not capabilities of this model.
- Evaluated benchmarks: 7-Scenes, NRGBD (3D reconstruction); Bonn, ScanNet (video depth); Sintel, TUM dynamics, ScanNet (pose estimation); Waymo, ScanNet++ (ablation).
