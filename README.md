# RTX 3060 Unlocked vs RTX 5050: ComfyUI Benchmark

## 👉 [View the interactive results page](https://shisa84.github.io/rtx3060-unlocked-vs-rtx5050-comfyui/)

https://shisa84.github.io/rtx3060-unlocked-vs-rtx5050-comfyui/

An RTX 3060 in four setups against an RTX 5050, on three ComfyUI workflows. Power, temperature, clocks and fan speed were logged with `nvidia-smi` every 500 ms.

## Setups

|  | Stock profile (170 W) | Optim profile (102 W, +250 MHz core offset) |
|---|---|---|
| **Locked** | 3060 locked | 3060 locked optim |
| **Unlocked** | 3060 unlocked | 3060 unlocked optim |

The RTX 5050 runs at 130 W and is the baseline.

**How to unlock the 3060:** see [how-to-unlock.md](how-to-unlock.md).

## Workflows

- **Z-Image Turbo**: 1024×1024, 8 steps. Average of warm runs (model already loaded).
- **MiniMax H3 T2V**: fl2va pruned int8, CLIP qwen3vl 32b nvfp4, turbo 8-step LoRA, 0.4 MP 16:9, 24 fps.
- **LookSheets 2-stage**: two MiniMax H3 sampling stages, upscale, RelayVSR.

## Gap with the RTX 5050

Time / energy, negative is better than the 5050.

| 3060 setup | Z-Image Turbo | MiniMax H3 T2V | LookSheets 2-stage |
|---|---|---|---|
| Locked | +27.6% / +32.5% | +0.7% / +29.4% | +33.8% / +54.2% |
| Locked optim | +19.8% / −4.8% | +2.7% / −17.1% | +29.0% / +8.0% |
| Unlocked | **−20.5%** / −0.7% | **−18.8%** / +8.8% | +7.4% / +38.4% |
| Unlocked optim | −10.8% / **−30.0%** | +1.3% / **−16.9%** | +18.2% / −2.0% |

## Takeaways

- **Fastest: 3060 unlocked (170 W).** Ahead of the 5050 on Z-Image and MiniMax, but at 80–83 °C with fans at 91–99%.
- **Most efficient: 3060 unlocked optim (102 W).** Uses less energy than the 5050 on all three workflows and stays around 70 °C.
- **Avoid: 3060 locked on the stock profile.** Slower than the 5050 everywhere and the most power-hungry.
- **The 5050 still wins LookSheets 2-stage**, against every 3060 setup.

## Notes

- LookSheets and MiniMax: one run per setup, so gaps of a few percent are within run-to-run noise.
- Energy is for the graphics card only, as reported by `nvidia-smi`.
- ComfyUI launched with `--lowvram` on both cards.
