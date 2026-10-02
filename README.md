# Qwen3.8 on one RTX PRO 6000: the configs

These are the Docker Compose files behind my video comparing three local Qwen3.8 checkpoints in 10 tests on one NVIDIA RTX PRO 6000 Blackwell (96 GB). Every file here is the exact launch the tests ran on.

- Video: **https://www.youtube.com/watch?v=VOtfja_Toj4**
- Channel: [Lukasz Gawenda AI](https://www.youtube.com/@LukaszGawendaAI)

## The configs

| File | Model | Engine | Drafter | Slots | Used for |
| --- | --- | --- | --- | ---: | --- |
| [qwen38-27b-nvfp4.sglang.yaml](docker/qwen38-27b-nvfp4.sglang.yaml) | [RadixArk/Qwen3.8-27B-NVFP4](https://huggingface.co/RadixArk/Qwen3.8-27B-NVFP4) | `lmsysorg/sglang:v0.5.20` | [DFlash2](https://huggingface.co/incoai/Qwen3.8-27B-DFlash2) | 4 | every test |
| [qwen38-27b-uncensored-nvfp4.sglang.yaml](docker/qwen38-27b-uncensored-nvfp4.sglang.yaml) | [orcarouter/Qwen3.8-27B-Uncensored-NVFP4](https://huggingface.co/orcarouter/Qwen3.8-27B-Uncensored-NVFP4) | `lmsysorg/sglang:v0.5.20` | DFlash2 | 4 | every test |
| [qwen38-flash-next-nvfp4.sglang.yaml](docker/qwen38-flash-next-nvfp4.sglang.yaml) | [RadixArk/Qwen3.8-Flash-Next-NVFP4](https://huggingface.co/RadixArk/Qwen3.8-Flash-Next-NVFP4) | `lmsysorg/sglang:dev-qwen38-next-local` | built-in MTP | 1 | every test except BFCL |
| [qwen38-flash-next-nvfp4.sglang-v0520.yaml](docker/qwen38-flash-next-nvfp4.sglang-v0520.yaml) | RadixArk/Qwen3.8-Flash-Next-NVFP4 | `lmsysorg/sglang:v0.5.20` | built-in MTP | 1 | BFCL |

All of them use the full 262,144-token window and serve on `127.0.0.1:8000`, so run one at a time. Each file's header explains its values.

## Quick start

```bash
# 1. Download the checkpoint (and, for the 27B models, the drafter) at the revision that was measured
hf download RadixArk/Qwen3.8-27B-NVFP4 --revision 319f741cce68d7914884900c138a1fbb70a42f30
hf download incoai/Qwen3.8-27B-DFlash2 --revision 015e795645c74b1a0eeef3b570031fb62e769bc5

# 2. Start the server
docker compose -f docker/qwen38-27b-nvfp4.sglang.yaml up -d

# 3. Check it
curl -s http://127.0.0.1:8000/v1/models
```

Other revisions:
- Uncensored model (gated; accept its terms on Hugging Face first): `orcarouter/Qwen3.8-27B-Uncensored-NVFP4 --revision 96d4d0b66d94314909d1af73f955e94bf68335ac`
- Flash-Next: `RadixArk/Qwen3.8-Flash-Next-NVFP4 --revision 7b719225242aacd3dbd3f9407468c2ee9a9d2594`

The composes mount `$HF_HOME` (default `~/.cache/huggingface`) read-only and point at the snapshot folder by its hash. If you downloaded a different revision, edit `--model-path`.

## Things that bite

- **Flash-Next needs host RAM.** It pins a 47.68 GiB embedding table in system memory, and the container peaked at 86 GiB. Run nothing else large next to it.
- **Flash-Next with images:** for CAPTCHA and the Rube Goldberg test I used `--mem-fraction-static 0.93`. At 0.96 and 0.95, image prefill hit CUDA OOM.
- **Flash-Next is one slot on purpose.** The cookbook's 16 slots leave too little KV cache for a 128K or full-window prompt.
- **The DFlash2 drafter is BF16.** `--speculative-draft-model-quantization unquant` stops SGLang from applying the target's FP4 quantization to it.

## Results

One run per model per test.

| Test | RadixArk/Qwen3.8-27B-NVFP4 | orcarouter/Qwen3.8-27B-Uncensored-NVFP4 | RadixArk/Qwen3.8-Flash-Next-NVFP4 |
| --- | ---: | ---: | ---: |
| Prefill, full window (s, lower is better) | 97 | 99 | **22.4** |
| Tool use, BFCL subset (thinking off) | **73.3%** | 70.8% | 64.5% |
| Needle in a haystack | 27/27 | 27/27 | 81/81 |
| Battle arena, open (out of 1000) | **700** | 603 | 535 |
| SVG, facts correct | **5/5** | 3/5 | 4/5 |
| Video editing (two cases, out of 20) | 16 | 13 | **19** |
| Voxel, ranked by eye | 2nd | 3rd | **1st** |
| Design, ranked by eye | 2nd (shared) | 2nd (shared) | **1st** |
| Rube Goldberg, ball in the cup | no | no | **yes** |
| CAPTCHA, solved in 7 min | **24/40** | 19/40 | 21/40 |

Speed comes with a caveat: Flash-Next's speed test ran at a 600 W GPU power cap, the 27B models' at 400 W. Prefill slows as the cap drops, so part of the gap is the cap.

On the 27B, the DFlash2 drafter took Spec-Bench from 75 to 210 tok/s for one user (2.8×).

## Hardware

- GPU: NVIDIA RTX PRO 6000 Blackwell, 96 GB (driver 595.84)
- CPU: AMD Ryzen 9 9950X
- RAM: 96 GB DDR5

## Links

- SGLang cookbook, Qwen3.8-27B on RTX PRO 6000 (RadixArk NVFP4, DFlash): https://docs.sglang.io/cookbook/autoregressive/Qwen/Qwen3.8-27B#hw=rtx6000&variant=default&quant=nvfp4-fp4-head&nodes=single&spec=dflash&tier=low-latency&ssmDtype=float32
- SGLang cookbook, Qwen3.8-Flash-Next on RTX PRO 6000: https://docs.sglang.io/cookbook/autoregressive/Qwen/Qwen3.8-Flash-Next#hw=rtx6000&variant=default&quant=nvfp4&strategy=low-latency&nodes=single&pleOffload=on
- vLLM recipe, Qwen3.8-27B on RTX PRO 6000: https://recipes.vllm.ai/Qwen/Qwen3.8-27B?hardware=rtx_pro_6000
- Inferact/Qwen3.8-27B-NVFP4 (the vLLM-side checkpoint in the speed comparison): https://huggingface.co/Inferact/Qwen3.8-27B-NVFP4
- Spec-Bench: https://github.com/hemingkx/Spec-Bench
- BFCL: https://github.com/ShishirPatil/gorilla/tree/main/berkeley-function-call-leaderboard
- Open CaptchaWorld: https://github.com/MetaAgentX/OpenCaptchaWorld
- Earlier Flash-Next engine comparison (llama.cpp vs SGLang vs FreeToken): https://github.com/lukaLLM/Qwen3.8-Flash-Next-VRAM-Benchmark

The test harness itself is not public yet.
