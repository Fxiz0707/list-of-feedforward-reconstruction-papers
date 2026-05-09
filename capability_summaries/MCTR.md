**Paper:** MCTR: Multi Camera Tracking Transformer, arXiv 2024, MCTR
**Summary:** MCTR does not produce a point cloud, dense 3D scene reconstruction, or other dense 3D representation. It is an end-to-end multi-camera multi-object tracking model that takes RGB images from multiple overlapping camera views, trains from labeled 2D boxes and track identities, and performs online frame-by-frame inference with learned detections, global track embeddings, and Hungarian assignment.

1. **3D reconstruction:** No. MCTR is not a 3D reconstruction method and does not output point clouds, meshes, depth maps, or dense 3D scene geometry. The paper frames the contribution as multi-camera multi-object tracking with DETR-style detections, track embeddings, and detection-to-track association (Abstract, p. 1; Sec. 3, pp. 3–4).

2. **Dense reconstruction:** No. MCTR does not perform dense reconstruction; its outputs are object-level detections, bounding boxes, and identity associations. The method predicts object classes and boxes from detection embeddings and associates them to tracks rather than reconstructing dense scene structure (Sec. 3.1–3.4, pp. 3–5).

3. **World-space output:** No. MCTR maintains globally consistent object identities across views and time, but it does not output world-space coordinates, world-space trajectories, or a shared metric 3D representation. Its "global" track embeddings encode identity and tracking state, not explicit world geometry (Sec. 3, pp. 3–4; Sec. 3.6, p. 6).

4. **Camera-space output (egocentric):** No. The method outputs 2D per-view bounding boxes and track assignments in image space, not 3D geometry in a camera coordinate frame. The DETR detection module predicts object class and 2D bounding box per view, and inference matches kept detections to tracks (Sec. 3.1, pp. 3–4; Sec. 3.6, p. 6).

5. **Flexible camera frame:** N/A. Since MCTR has no 3D camera-space output (Q4 = No), there is no reference camera frame to choose or vary. The camera views are handled by view-specific modules, but those modules do not define a selectable 3D output frame (Sec. 3.2–3.3, p. 4).

6. **Metric scale:** No. MCTR has no metric 3D output; predictions are 2D detections and tracks. The paper explicitly notes that MCTR does not use camera parameters (Table 2, p. 8; Sec. 3.1 and Sec. 3.6, pp. 3–6).

7. **Point tracking:** No. MCTR predicts associations between object detections and global object tracks, not correspondences between arbitrary image points. The association matrix gives the probability that a detection belongs to a track embedding; it is object-level, not point-level (Sec. 3.3, p. 4; Sec. 3.4, pp. 4–5).

8. **3D tracks:** No. The method does not output 3D point trajectories. Its temporal output is multi-object identity tracks over 2D bounding boxes across camera views and frames (Sec. 3.6, p. 6; Sec. 5, p. 8).

9. **2D tracks:** No. MCTR outputs object-level 2D bounding box tracks rather than 2D point tracks (trajectories of specific image points or pixel correspondences). The Q9 auto-Yes rule does not apply since Q8 = No and Q14 = No (Sec. 3.1, pp. 3–4; Sec. 3.6, p. 6).

10. **Occlusion / visibility prediction:** No. MCTR is designed to remain robust through occlusions, and the MCTR-TB variant can predict bounding boxes for people occluded in a particular view using global track embeddings, but the paper does not describe explicit per-track occlusion or visibility flags as model outputs (Sec. 3.4, pp. 4–5; Sec. 4.1, pp. 7–8).

11. **Multiple views:** Yes. MCTR processes multiple simultaneous camera views with overlapping fields of view: detectors run independently on each camera view, track embeddings are updated from detection embeddings from all camera views, and associations are produced for every camera view. This is the core and evaluated setting. However, the cross-attention modules are view-specific with no parameter sharing, so the model is tied to the fixed camera configuration used during training and is not plug-and-play for arbitrary new cameras (Abstract, p. 1; Sec. 3.2–3.3, pp. 4–5; Sec. 4.2 and Sec. 5, p. 8).

12. **Global context vs. pairwise vs. fixed-window:** Fixed-window. Inference is online and frame-by-frame (Sec. 3.6, p. 6). The conclusion explicitly describes "MCTR's Markovian approach, where track embeddings are updated only with current and previous frame information" — the model carries a fixed-size track-embedding state that is updated each frame rather than attending to all past frames directly. Across the camera set the association is global at each time step, but temporal aggregation is Markovian (fixed-window of one frame with state carried forward) (Sec. 3.5–3.6, pp. 5–6; Sec. 5, p. 8).

13. **No GT extrinsics required at inference:** Yes. Ground-truth extrinsics are not required at inference. MCTR does not use camera parameters; the paper contrasts this with leaderboard systems and ReST-style approaches that require calibration parameters (Table 2, p. 8; Sec. 4.2 footnote, p. 8).

14. **Extrinsics as output:** No. MCTR does not estimate or output camera extrinsics. Camera geometry is implicitly absorbed into the learned view-specific cross-attention modules rather than predicted explicitly (Sec. 3.2, p. 4; Sec. 5, p. 8).

15. **No GT intrinsics required at inference:** Yes. Ground-truth intrinsics are not required at inference. MCTR does not use camera parameters (Table 2, p. 8; Sec. 5, p. 8).

16. **Intrinsics as output:** No. MCTR does not estimate or output camera intrinsics. Camera handling is implicit through learned view-specific cross-attention modules, not explicit calibration prediction (Sec. 3.2, p. 4; Sec. 5, p. 8).

17. **Moving cameras:** No. MCTR targets fixed overlapping multi-camera surveillance systems and is closely tied to the camera setup used during training. Moving cameras are not supported or evaluated (Introduction, pp. 1–2; Sec. 4.2 and Sec. 5, p. 8).

18. **Dynamic content:** Yes. MCTR is designed for dynamic content: tracking multiple moving people over time across multiple camera feeds. It is evaluated on MMPTrack and AI City Challenge multi-camera tracking datasets with dynamic people and long-term identity tracking (Abstract, p. 1; Sec. 4.1–4.2, pp. 6–8).

19. **Feed-forward inference:** Yes. MCTR performs feed-forward online inference frame by frame using neural detection and embedding modules plus Hungarian matching for assignment, with no test-time optimization or bundle adjustment. The paper reports real-time-oriented scaling: approximately 21 FPS for one camera on a GeForce RTX 2080 Ti, with linear cost per additional camera (Sec. 3.6, p. 6).

**Notes:** MCTR takes RGB images from a fixed set of overlapping cameras and is supervised with 2D bounding boxes and track identities. It does not use calibration parameters at inference, but the learned view-specific cross-attention modules make it setup-specific and not naturally transferable to arbitrary or moving camera rigs (the paper acknowledges this limitation and suggests future work on camera-setup-independent models). The auxiliary track-box prediction (MCTR-TB) can extrapolate occluded-view 2D boxes and suggests learned approximate scene-camera relations, but it remains an object-tracking method rather than a 3D reconstruction or point-tracking method. MCTR is an online system evaluated on MMPTrack (5 environments, RGBD-annotated) and the 2023 AI City Challenge Track 1 dataset (Sec. 3.4, pp. 4–5; Sec. 4.1, pp. 7–8; Sec. 5, p. 8).
