# Local LLM benchmarks

Every model I benchmark for [my YouTube channel](https://www.youtube.com/@LukaszGawendaAI), on one GPU at home: the exact Docker launch, so you can run it with one command, and its scorecard. How each test works, and why a model won or lost, is in the video.

| Model | Video | Result | Config |
|---|---|---|---|
| ukisai/Swift-1.5-Qwen3.8-27b-NVFP4 | [58% Less Thinking? Swift 1.5 vs Qwen 3.8 27B in 10 Real Tests!](https://www.youtube.com/watch?v=9WOEjWC9zYs) | won 6 of 10 tests (3 ties) against RadixArk/Qwen3.8-27B-NVFP4, 24% fewer thinking tokens | [models/swift15-27b](models/swift15-27b) |
| RadixArk/Qwen3.8-27B-NVFP4 | [58% Less Thinking? Swift 1.5 vs Qwen 3.8 27B in 10 Real Tests!](https://www.youtube.com/watch?v=9WOEjWC9zYs) | won 1 of 10 tests (3 ties) against ukisai/Swift-1.5-Qwen3.8-27b-NVFP4 | [models/qwen38-27b](models/qwen38-27b) |
| RadixArk/Qwen3.8-27B-NVFP4 | [Which Qwen 3.8 Should You Run? 27B vs Flash Next vs Uncensored in 10 Real Tests!](https://youtu.be/VOtfja_Toj4) | won 4 of 10 tests (1 tie) against orcarouter/Qwen3.8-27B-Uncensored-NVFP4 and RadixArk/Qwen3.8-Flash-Next-NVFP4 | [models/qwen38-27b](models/qwen38-27b) |
| orcarouter/Qwen3.8-27B-Uncensored-NVFP4 | [Which Qwen 3.8 Should You Run? 27B vs Flash Next vs Uncensored in 10 Real Tests!](https://youtu.be/VOtfja_Toj4) | won 0 of 10 tests (1 tie) against RadixArk/Qwen3.8-27B-NVFP4 and RadixArk/Qwen3.8-Flash-Next-NVFP4 | [models/qwen38-27b-uncensored](models/qwen38-27b-uncensored) |
| RadixArk/Qwen3.8-Flash-Next-NVFP4 | [Which Qwen 3.8 Should You Run? 27B vs Flash Next vs Uncensored in 10 Real Tests!](https://youtu.be/VOtfja_Toj4) | won 5 of 10 tests (1 tie) against RadixArk/Qwen3.8-27B-NVFP4 and orcarouter/Qwen3.8-27B-Uncensored-NVFP4 | [models/qwen38-flash-next](models/qwen38-flash-next) |

## Hardware

- GPU: NVIDIA RTX PRO 6000 Blackwell Workstation Edition, 95 GB
- CPU: AMD Ryzen 9 9950X 16-Core Processor
- RAM: 96 GB DDR5

## Using a config

Each `models/<id>/` folder has a `compose.yaml` and a README with the download commands. Download the checkpoint at the listed revision, then `docker compose up -d` in that folder. The engine image is pinned to the one that was measured; engines move fast, so if it is months old, try the newest image first (each README says how).

## More

- Earlier: Qwen3.8 Flash-Next engine comparison (llama.cpp vs SGLang vs FreeToken): https://github.com/lukaLLM/Qwen3.8-Flash-Next-VRAM-Benchmark
