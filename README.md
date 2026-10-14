# Prediction-Informed Landing Window Planning for UAV Landing on USV in High Sea States

[![arXiv](https://img.shields.io/badge/arXiv-2610.13190-b31b1b.svg)](https://arxiv.org/abs/2610.13190)
[![Conference](https://img.shields.io/badge/IEEE%20ICCEIC-2026-blue)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**"Prediction-Informed Landing Window Planning for UAV Landing on USV in High Sea States"** (IEEE ICCEIC 2026).

## 📄 Abstract
<img width="3769" height="1149" alt="介绍图" src="https://github.com/user-attachments/assets/75c6bc25-b0ab-4bfd-b406-e3476294cfaf" />

Autonomous landing of an unmanned aerial vehicle (UAV) on an unmanned surface vehicle (USV) is difficult in high sea states. Existing hierarchical landing methods often infer future USV motion only implicitly from short observation histories, which limits their ability to adapt to severe and rapidly changing sea conditions. This paper presents a **prediction-informed hierarchical control framework**: a pretrained model explicitly predicts the USV attitude during high-level landing decisions. The prediction helps identify suitable landing windows and choose the right touchdown time, while a conventional low-level flight controller maintains UAV stability and tracks motion commands. To evaluate the framework under realistic disturbances, a **real-sim validation setup** is constructed, in which USV-motion data collected in real sea conditions are transferred to a MuJoCo USV. Experiments demonstrate that explicit attitude prediction improves landing-window selection and supports timing-aware UAV landing planning in rapidly varying marine environments.

## 🔔 Highlights

- **Prediction-informed hierarchical landing framework** — couples explicit platform-attitude forecasting with high-level landing-window planning, while keeping a conventional low-level flight controller for stabilization and tracking.
- **Pretrained temporal predictor** — an encoder-decoder LSTM forecasts short-term USV roll/pitch (2.0 s history → 1.0 s forecast @ 0.1 s), providing future attitude information for touchdown-window assessment.
- **Real-sim validation** — USV motion data collected in real sea conditions are replayed on a MuJoCo landing platform, enabling evaluation under realistic disturbance patterns rather than idealized wave models.

## 🛠️ Method


The system adopts a two-level architecture:

| Level | Module | Role |
|---|---|---|
| High level | Pretrained LSTM + PPO policy | Predicts USV attitude, evaluates landing windows, outputs a continuous vertical-velocity command in [-0.8, 0.8] m/s |
| Low level | SE(3) flight controller | Computes desired attitude/moments and allocates motor thrusts for stabilization and tracking |

**Observation.** Relative height, relative vertical velocity, deck tilt, tilt rate, plus a 10-step (1.0 s) predicted roll sequence.

**Predictor.** Encoder-decoder LSTM with 256 hidden units; each input window is independently normalized and restored after prediction. Trained with MSE loss (MAE for monitoring), Adam + weight decay + LR scheduling + early stopping. Inference is refreshed every 0.1 s.

**Reward.** Additive multi-component design: height-dependent descent progress, touchdown safety (activated below 0.50 m / 0.25 m), forecast terms (forecast stability + peak-tilt excess computed from the predicted sequence), action-rate smoothness, and safety penalties (tilt > 15°, descent speed > 1.0 m/s). +5000 for a successful landing, −2000 otherwise.

**Training setup.** PPO with actor/critic MLPs (2 × 128); lr 3e-4, rollout length 2048, batch size 64, 10 update epochs per rollout, γ = 0.99, GAE λ = 0.95, clip range 0.2. Episodes start with the UAV at 2.0 m altitude and a random horizontal position within [-2, 2] m. Trained on a single NVIDIA RTX 5060 Ti.


**Policy behavior.** The vertical-velocity command consistently *leads* the USV attitude by several phases: when a large deck tilt is predicted, the policy reduces the descent rate, hovers, or briefly climbs *before* the unfavorable window opens — no manually designed state-switching rules are required. See the paper for full motion-segment analyses.


## 📚 Citation

If you find this work useful, please cite:

```bibtex
@inproceedings{li2026prediction,
  title     = {Prediction-Informed Landing Window Planning for UAV Landing on USV in High Sea States},
  author    = {Li, Kaizhen and Zhou, Xi and Zhang, Dan and Li, Xiaowei},
  booktitle = {2026 IEEE International Conference on Communication, Electronic Information and Computing (ICCEIC)},
  year      = {2026}
}
```

## 📬 Contact

- **Kaizhen Li** — [likaizhen@shu.edu.cn](mailto:likaizhen@shu.edu.cn)
- GitHub: [@likaizhen-code](https://github.com/likaizhen-code)
