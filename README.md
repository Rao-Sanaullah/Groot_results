# GR00T N1.6 & N1.7 — FFW-SG2 AI Worker Results

> Closed-loop VLA evaluation of NVIDIA GR00T N1.6 and N1.7 on the ROBOTIS FFW-SG2 AI Worker — simulation and real-robot deployment for bimanual pegboard manipulation.

**[📊 Full Results Page](https://rao-sanaullah.github.io/groot-sg2-results/)** · **[🔧 GR00TN1.6 & N1.7 Setup Guide](https://rao-sanaullah.github.io/GR00TN16_and_N17_Installation_Guide/)** · **[⭐ StarVLA Results](https://rao-sanaullah.github.io/starVLA_results/)** · **[📄 Paper](#)**

---

## Key Results

| Model | Sim success | Real-robot | Trials | Semantic failures |
|---|---|---|---|---|
| **GR00T N1.6** | **100%** | **88%** | 50 sim · 100 real | 0 |
| **GR00T N1.7** | **100%** | Sim only (EA) | 50 sim | 0 |
| StarVLA (no-state) | 52% | — | 50 sim | 0 |
| StarVLA (state) | 28% | — | 50 sim | Wrong arm 24% · wrong object 14% |
| OpenVLA | — | — | Excluded | Action dimension collapse |

> GR00T N1.7 is evaluated in simulation only due to its [Early Access release status](https://github.com/NVIDIA/Isaac-GR00T). Real-robot deployment is reserved for the GA release.

---

## Real-Robot Failure Analysis — GR00T N1.6 (100 trials)

```
✅  Successful placements    88 / 100  (88%)
⚠️  Grasp drops               10 / 100  (10%)   during pegboard extraction
⚠️  Missed placements          2 / 100   (2%)   released near crate, not inside
❌  Approach failures           0 / 100   (0%)
❌  Wrong arm / semantic        0 / 100   (0%)
```

All 12 failures are physical contact failures — not semantic. The 12pp sim-to-real gap is entirely attributable to contact dynamics that IsaacLab rigid-body simulation does not model. Grasp drops are distributed randomly across all 100 trials with no concentration in early or late trials.

**Evaluation protocol:** Single continuous run. Human assistant repositions brush after each trial — no robot reset between trials. Policy runs uninterrupted throughout.

---

## Training Summary

| Run | Steps | Initial loss | Final loss | Reduction | Hardware |
|---|---|---|---|---|---|
| N1.6 simulation | 40,000 | 1.3309 | 0.0053 | 99.6% | RTX 5090 |
| N1.7 simulation | 20,000 | 1.3884 | 0.0415 | 97.0% | RTX 5090 |
| N1.6 real-world | 100,000 | 1.2901 | 0.0019 | 99.9% | DGX Spark |

**Deployed checkpoints:** N1.6 sim → step 20k · N1.7 sim → step 20k · N1.6 real → step 100k

**Key finding:** GR00T N1.7 shows 1.86× higher offline NormMAE than N1.6 (1.45% vs 0.94%) yet achieves identical 100% simulation success — confirming that offline MAE is not a reliable predictor of closed-loop task performance for diffusion-based VLA policies.

---

## Datasets

### Simulation dataset
```
HuggingFace : Dongkkka/sim2real_1224_brush_pick
Episodes    : 2,000
Frames      : 306,766
FPS         : 10 Hz
Camera      : cam_head · 376×672 · AV1/H.264
Format      : LeRobot v2.1
```

### Real-world dataset
```
Identifier  : ffw_sg2_rev1_pick_place_500ep
Episodes    : 499
Frames      : 172,715
FPS         : 30 Hz
Cameras     : cam_head 376×672 · cam_wrist_left 240×424 · cam_wrist_right 240×424
Codec       : H.264/libx264
Format      : LeRobot v2.1
Teleop      : FFW-LG2 Leader (DYNAMIXEL-X · 22 DoF · ~3kg)
Duration    : ~96 min demonstration data
```

> **Video preprocessing note:** Head camera videos are re-encoded to 256×256 before GR00T N1.6 fine-tuning to remove rotation metadata that causes orientation errors in the model's image decoder:
> ```bash
> find . -name "*.mp4" | while read mp4; do
>   ffmpeg -y -i "$mp4" \
>     -vf "scale=256:256:force_original_aspect_ratio=disable,setsar=1" \
>     -metadata:s:v rotate=0 -c:v libx264 -pix_fmt yuv420p -crf 18 -preset fast \
>     "${mp4%.mp4}_fixed.mp4" && mv "${mp4%.mp4}_fixed.mp4" "$mp4"
> done
> ```

---


---

## Inference Configuration

| Parameter | GR00T N1.6 | GR00T N1.7 |
|---|---|---|
| Protocol | ZMQ (req/rep) | ZMQ (req/rep) |
| Port | 5555 | 5556 |
| Chunk size T | 16 steps | 16 steps |
| EMA α (sim) | 0.3 | 0.3 |
| EMA α (real) | 0.8 | — |
| Control Hz (sim) | 60 Hz | 60 Hz |
| Control Hz (real) | 100 Hz | — |
| State input | ✓ named groups | ✓ named groups |
| Image resolution | 256×256 | 256×256 |
| Denoising steps | 4 | 4 |

---

## Repository Structure

```
groot-sg2-results/
├── index.html                          # Results webpage
├── README.md                           # This file
│
├── plots/
│   ├── groot_n16_training_analysis.png
│   ├── groot_n16_mae_analysis.png
│   ├── groot_n17_training_analysis.png
│   ├── groot_n17_mae_analysis.png
│   ├── plot7_training_analysis_paper.png
│   ├── plot4_training_comparison.png
│   ├── plot1_episode_statistics.png
│   ├── plot3_joint_trajectories.png
│   ├── plot5_action_std_comparison.png
│   └── plot6_frame_montage.png
│
└── videos/
    ├── groot_n16_real_success.mp4
    ├── groot_n16_real_failure.mp4
    ├── groot_n16_sim.mp4
    └── groot_n17_sim.mp4
```

---

## Related Links

- 🔧 [GR00T N1.6 & N1.7 Setup Guide](https://rao-sanaullah.github.io/GR00TN16_and_N17_Installation_Guide/) — full installation and fine-tuning walkthrough
- ⭐ [StarVLA on FFW-SG2 Results](https://rao-sanaullah.github.io/starVLA_results/) — state-free vs state-conditioned comparison
- 🤖 [Isaac-GR00T](https://github.com/NVIDIA/Isaac-GR00T) — official NVIDIA repository
- 📦 [ROBOTIS FFW-SG2](https://ai.robotis.com/) — robot platform

---

## Citation

If you use these results or configurations in your work, please cite:

```bibtex
@article{sanaullah2025groot_ffwsg2,
  title   = {#},
  author  = {Sanaullah and others},
  journal = {#},
  year    = {2026}
}
```

---

<p align="center">
  GR00T N1.6 &amp; N1.7 · ROBOTIS FFW-SG2 AI Worker · NVIDIA IsaacLab · DGX Spark<br>
  Designed by <strong>Sanaullah</strong>
</p>
