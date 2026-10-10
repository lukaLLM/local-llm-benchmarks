# RadixArk/Qwen3.8-Flash-Next-NVFP4

▶ **Video: [Which Qwen 3.8 Should You Run? 27B vs Flash Next vs Uncensored in 10 Real Tests!](https://youtu.be/VOtfja_Toj4)**  

RadixArk/Qwen3.8-Flash-Next-NVFP4 on the official SGLang image, built-in MTP (NEXTN) drafter.

## Scorecard: Which Qwen 3.8 Should You Run? 27B vs Flash Next vs Uncensored in 10 Real Tests!

In this video it won 5 of 10 tests (1 tie) against RadixArk/Qwen3.8-27B-NVFP4 and orcarouter/Qwen3.8-27B-Uncensored-NVFP4.

![Scorecard](../../scorecards/qwen38-27b-vs-flash-next-vs-uncensored.png)

| Test | RadixArk/Qwen3.8-27B-NVFP4 | orcarouter/Qwen3.8-27B-Uncensored-NVFP4 | RadixArk/Qwen3.8-Flash-Next-NVFP4 |
|---|---:|---:|---:|
| Speed <sub>Prefill, full window (lower is better)</sub> | 97 s | 99 s | **22.4 s** |
| Tool use <sub>BFCL core</sub> | **73.3 %** | 70.8 % | 64.5 % |
| Long context <sub>Recall over the grid</sub> | 27 / 27 100% | 27 / 27 100% | 81 / 81 100% |
| Battle arena <sub>Open arena score (1 blind army)</sub> | **700 / 1000** | 603 / 1000 | 535 / 1000 |
| Drawing <sub>Facts correct</sub> | **5 / 5** | 3 / 5 | 4 / 5 |
| Video editing <sub>both cases, out of 20</sub> | 8 + 8 = 16 | 5 + 8 = 13 | **9 + 10 = 19** |
| Voxel <sub>Rank by eye</sub> | 2nd of 3 | 3rd of 3 | **1st of 3** |
| Design <sub>Rank by eye</sub> | 2nd of 3 (shared) | 2nd of 3 (shared) | **1st of 3** |
| Rube Goldberg <sub>Ball in the cup</sub> | no | no | **yes** |
| CAPTCHA <sub>Solved</sub> | **24 / 40** | 19 / 40 | 21 / 40 |

- One run per model per test.
- Speed: Flash-Next's speed test ran at a 600 W GPU power cap, the 27B models' at 400 W. Prefill slows as the cap drops, so part of the gap is the cap.
- On the 27B, the DFlash2 drafter took Spec-Bench from 75 to 210 tok/s for one user (2.8x).

How each test works, and why it won or lost: [in the video](https://youtu.be/VOtfja_Toj4).

## Run it

```bash
docker compose up -d        # the first start downloads the weights
docker compose logs -f      # wait for: The server is fired up
curl -s http://127.0.0.1:8000/v1/models
```

It downloads [RadixArk/Qwen3.8-Flash-Next-NVFP4](https://huggingface.co/RadixArk/Qwen3.8-Flash-Next-NVFP4) at `7b719225242a` into `$HF_HOME` (default `~/.cache/huggingface`), the revisions that were measured. A gated model needs `HF_TOKEN` set and its terms accepted on Hugging Face. It serves on `127.0.0.1:8000` at the full window, so run one model at a time. The header of `compose.yaml` explains every value; change what you need for your card.

## Engine image

Measured with `lmsysorg/sglang:dev-qwen38-next-local` (`sha256:9d2a843c706c74bc259c0d9abf360551eb2734e1e7d255ab012a6965f10480b6`).

Engines move fast. If you read this months later, try the newest image first (edit the `image:` line, e.g. `lmsysorg/sglang:latest`). The pinned image is the one the numbers come from.

## Things that bite

- It needs host RAM: it pins a 47.68 GiB embedding table in system memory, and the container peaked at 86 GiB. Run nothing else large next to it.
- With images (CAPTCHA, Rube Goldberg) it ran with `SGLANG_MEM_FRACTION_STATIC=0.93 docker compose up -d`. At 0.96 and 0.95, image prefill hit CUDA OOM.
- One slot on purpose: the cookbook's 16 slots leave too little KV cache for a 128K or full-window prompt.
- Tool use (BFCL) ran on the released `lmsysorg/sglang:v0.5.20` image: `compose.v0520.yaml`.

## Other launches

- [compose.v0520.yaml](compose.v0520.yaml): see its header for when it was used.
