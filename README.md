# feedforward-reconstruction-papers

## Strictly Feed-Forward Methods


| Paper | 3D Recon | Tracking | Frame | Predicted Camera Params | Metric | Predicted Occ/Vis | Multi-view | Context | GT Calib Needed | Moving Cam | Dynamic Scene |
| ----- | -------- | -------- | ----- | ----------------------- | ------ | ----------------- | ---------- | ------- | --------------- | ---------- | ------------- |
| **DePT3R** Dec 2025 [arXiv](https://arxiv.org/abs/2512.13122) · [Project](https://github.com/StructuresComp/DePT3R) | Dense | 2D+3D | Fixed world | Extrinsics + Intrinsics | No | No | No | Global | No | Yes | Yes |
| **D4RT** Dec 2025 [arXiv](https://arxiv.org/abs/2512.08924) · [Project](https://d4rt-paper.github.io/) | Dense | 2D+3D | Selectable | Extrinsics + Intrinsics | No | Yes | No | Global | No | Yes | Yes |
| **LAPA** Dec 2025 [arXiv](https://arxiv.org/abs/2512.04213) · [Project](https://github.com/ostadabbas/Look-Around-and-Pay-Attention-LAPA-) | No; tracking only | 2D+3D | N/A | No | Yes | No | Yes | Hybrid | Extrinsics + Intrinsics | No | Yes |
| **MV-TAP** Dec 2025 [arXiv](https://arxiv.org/abs/2512.02006) · [Project](https://cvlab-kaist.github.io/MV-TAP/) | No; tracking only | 2D | N/A | No | No | Yes | Yes (arbitrary) | Global | Extrinsics + Intrinsics | Yes | Yes |
| **DepthAnything3** Nov 2025 [arXiv](https://arxiv.org/abs/2511.10647) · [Project](https://depth-anything-3.github.io/) | Dense | No | Selectable | Extrinsics + Intrinsics | Partial | No | Yes | Global | No | Yes | No |
| **MapAnything** Sep 2025 [arXiv](https://arxiv.org/abs/2509.13414) · [Project](https://map-anything.github.io/) | Dense | No | Selectable | Extrinsics + Intrinsics | Yes | No | Yes (arbitrary) | Global | No | Yes | No |
| **G-CUT3R** Aug 2025; ICLR 2026 [arXiv](https://arxiv.org/abs/2508.11379) | Dense | No | Fixed world | Extrinsics | Yes | No | No | Hybrid | No | Yes | Yes |
| **π³** Jul 2025; ICLR 2026 [arXiv](https://arxiv.org/abs/2507.13347) · [Project](https://yyfz.github.io/pi3/) | Dense | No | Egocentric | Extrinsics | No | No | Yes (arbitrary) | Global | No | Yes | Yes |
| **SpatialTrackerV2** Jul 2025 [arXiv](https://arxiv.org/abs/2507.12462) · [Project](https://spatialtracker.github.io/) | Dense | 2D+3D | Fixed world | Extrinsics + Intrinsics (focal only) | Yes | Yes | No | Global | No | Yes | Yes |
| **TAPIP3D** Apr 2025; NeurIPS 2025 [arXiv](https://arxiv.org/abs/2504.14717) · [Project](https://tapip3d.github.io/) | No; tracking only | 2D+3D | Fixed world | Pipeline | Partial | Yes | No | Fixed-window | Intrinsics | Yes | Yes |
| **St4RTrack** Apr 2025 [arXiv](https://arxiv.org/abs/2504.13152) · [Project](https://st4rtrack.github.io/) | Dense | 2D+3D | Fixed world | Extrinsics + Intrinsics (partial) | No | No | No | Pairwise | No | Yes | Yes |
| **VGGT** Mar 2025 [arXiv](https://arxiv.org/abs/2503.11651) · [Project](https://vgg-t.github.io/) | Dense | 2D+3D | Selectable | Extrinsics + Intrinsics | No | Yes | Yes (arbitrary) | Global | No | Yes | Partial |
| **Dynamic Point Maps** Mar 2025 [arXiv](https://arxiv.org/abs/2503.16318) · [Project](https://www.robots.ox.ac.uk/~vgg/research/dynamic-point-maps/) | Dense | 2D+3D | Fixed world | Extrinsics + Intrinsics | No | No | No | Pairwise | No | Yes | Yes |
| **UniDepthV2** Feb 2025 [arXiv](https://arxiv.org/abs/2502.20110) · [Project](https://lpiccinelli-eth.github.io/pub/unidepth/) | Dense | No | Egocentric | Intrinsics | Yes | No | No | N/A | No | Yes | Partial |
| **L4P** Feb 2025 [arXiv](https://arxiv.org/abs/2502.13078) | Dense | 2D+3D | Selectable | Extrinsics + Intrinsics (partial) | No | Yes | No | Hybrid | No | Yes | Yes |
| **Fast3R** Jan 2025 [arXiv](https://arxiv.org/abs/2501.13928) · [Project](https://fast3r-3d.github.io/) | Dense | No | Fixed world | Extrinsics + Intrinsics | No | No | Yes (arbitrary) | Global | No | Yes | Partial |
| **CUT3R** Jan 2025 [arXiv](https://arxiv.org/abs/2501.12387) · [Project](https://cut3r.github.io/) | Dense | No | Fixed world | Extrinsics | Yes | No | No | Global | No | Yes | Yes |
| **DELTA** Oct 2024; ICLR 2025 [arXiv](https://arxiv.org/abs/2410.24211) · [Project](https://snap-research.github.io/DELTA/) | Dense | 2D+3D | Egocentric | No | No | Yes | No | Fixed-window | No | Yes | Yes |
| **CoTracker3** Oct 2024 [arXiv](https://arxiv.org/abs/2410.11831) · [Project](https://cotracker3.github.io/) | No | 2D | N/A | No | No | Yes | No | Hybrid | No | Yes | Yes |
| **Spann3R** Aug 2024 [arXiv](https://arxiv.org/abs/2408.16061) · [Project](https://hengyiwang.github.io/projects/spanner) | Dense | No | Fixed world | No | No | No | No | Hybrid | No | Yes | No |
| **MCTR** Aug 2024 [arXiv](https://arxiv.org/abs/2408.13243) | No | No | N/A | No | No | No | Yes | Fixed-window | No | No | Yes |


## Other / Hybrid / Optimization-Based Methods


| Paper | 3D Recon | Tracking | Frame | Predicted Camera Params | Metric | Predicted Occ/Vis | Multi-view | Context | GT Calib Needed | Moving Cam | Dynamic Scene |
| ----- | -------- | -------- | ----- | ----------------------- | ------ | ----------------- | ---------- | ------- | --------------- | ---------- | ------------- |
| **AMB3R** Nov 2025 [arXiv](https://arxiv.org/abs/2511.20343) · [Project](https://hengyiwang.github.io/projects/amber) | Dense | No | Selectable | Extrinsics | Yes | No | Yes | Hybrid | No | Yes | Partial |
| **VGGT-SLAM** May 2025 [arXiv](https://arxiv.org/abs/2505.12549) · [Project](https://github.com/MIT-SPARK/VGGT-SLAM) | Dense | No | Fixed world | Extrinsics + Intrinsics | No | No | No | Hybrid | No | Yes | No |
| **FoundationStereo** Jan 2025 [arXiv](https://arxiv.org/abs/2501.09898) · [Project](https://nvlabs.github.io/FoundationStereo/) | Dense | No | Egocentric | No | Yes | No | Yes (fixed stereo pair) | Pairwise | Extrinsics + Intrinsics (rectified) | No | No |
| **MASt3R-SLAM** Dec 2024 [arXiv](https://arxiv.org/abs/2412.12392) · [Project](https://edexheim.github.io/mast3r-slam/) | Dense | 2D | Fixed world | Extrinsics | No | No | No | Hybrid | No | Yes | No |
| **SLAM3R** Dec 2024 [arXiv](https://arxiv.org/abs/2412.09401) | Dense | No | Fixed world | No | No | No | No | Hybrid | No | Yes | No |
| **MegaSAM** Dec 2024 [arXiv](https://arxiv.org/abs/2412.04463) · [Project](https://mega-sam.github.io/) | Dense | No | Fixed world | Extrinsics + Intrinsics (shared focal) | Partial | No | No | Hybrid | No | Yes | Yes |
| **MONST3R** Oct 2024; ICLR 2025 [arXiv](https://arxiv.org/abs/2410.03825) · [Project](https://monst3r-project.github.io/) | Dense | No | Selectable | Extrinsics + Intrinsics | No | No | No | Hybrid | No | Yes | Yes |
| **MASt3R-SfM** Sep 2024 [arXiv](https://arxiv.org/abs/2409.19152) · [Project](https://github.com/naver/mast3r) | Dense | 2D | Fixed world | Extrinsics + Intrinsics | Yes | No | Yes (arbitrary) | Hybrid | No | Yes | No |
| **MASt3R** Jun 2024 [arXiv](https://arxiv.org/abs/2406.09756) · [Project](https://europe.naverlabs.com/blog/mast3r-matching-and-stereo-3d-reconstruction/) | Dense | 2D | Selectable | Pipeline | Yes | No | Yes (fixed 2) | Pairwise | No | Yes | No |
| **DUSt3R** Dec 2023; CVPR 2024 [arXiv](https://arxiv.org/abs/2312.14132) · [Project](https://europe.naverlabs.com/research/publications/dust3r-geometric-3d-vision-made-easy/) | Dense | 2D | Selectable | Extrinsics + Intrinsics | No | No | Yes (arbitrary) | Hybrid | No | Yes | No |
| **COLMAP** Jun 2016; CVPR 2016 [Project](https://colmap.github.io/) | Sparse | 2D | Fixed world | Extrinsics + Intrinsics | No | No | Yes (arbitrary) | Global | No | Yes | No |

