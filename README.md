# megaasr-exp

Experimentation workspace for evaluating **[Mega-ASR](https://github.com/xzf-thu/Mega-ASR)** — a noise-robust ASR foundation model (Qwen3-ASR-1.7B base + LoRA + router) from Tsinghua, released May 2026.

This repo is **not a fork**. Upstream Mega-ASR is cloned into `Mega-ASR/` (gitignored) and we keep only our scripts, eval data, and results under version control.

## Why evaluate

The authors claim **up to ~30% WER gains over Qwen3-ASR, Gemini-3-Pro, Seed-ASR, and Whisper** on in-the-wild audio (7 atomic acoustic conditions × 54 compound scenarios). Worth verifying on:

1. Their published examples (sanity check the demo).
2. **Our own** real-world recordings (IPEVO meeting / conference-room audio is the relevant target — far-field, reverb, fan noise).
3. Clean academic sets (LibriSpeech `test-clean`) — to check the authors' caveat that *"Mega-ASR is trained on high-WER data, which leads to slight degradation in basic recognition"* (mitigated by their router; we should measure router on vs. off).

## Upstream resources

| Resource | Link |
|---|---|
| Code | https://github.com/xzf-thu/Mega-ASR |
| Weights | https://huggingface.co/zhifeixie/Mega-ASR |
| Training dataset | https://huggingface.co/datasets/zhifeixie/Voices-in-the-Wild-2M |
| Robustness benchmark | https://github.com/xzf-thu/Voices-in-the-Wild-Bench |
| Technical report | https://arxiv.org/abs/2605.19833 |
| Project page | https://xzf-thu.github.io/Mega-ASR/ |
| License | Apache-2.0 |

## Plan

### Phase 1 — Reproduce the demo
- [ ] Clone upstream into `Mega-ASR/`
- [ ] `conda create -n mega-asr python=3.10` + `pip install -r requirements.txt`
- [ ] `python scripts/download.py` to fetch weights (base + LoRA + router)
- [ ] `bash scripts/inference.sh` with default audio → confirm it runs

### Phase 2 — Build our eval set
- [ ] Collect ~30–50 audio clips covering the scenarios we care about (meeting room, lecture hall, phone-call, far-field). Place in `data/audio/`.
- [ ] Hand-transcribe ground truth → `data/eval.jsonl` in upstream's format: `{"audio": "...", "answer": "..."}`

### Phase 3 — Run evaluation
- [ ] Mega-ASR with router (default): `python src/MegaASR/eval/evaluate_wer.py --ckpt_dir ckpt/Mega-ASR --input_jsonl data/eval.jsonl --output_jsonl results/mega_router.jsonl`
- [ ] Mega-ASR without router (force LoRA): add `--no-routing` → `results/mega_no_router.jsonl`
- [ ] Baseline: same clips through Whisper-large-v3 and/or Qwen3-ASR-1.7B for an apples-to-apples comparison
- [ ] Aggregate WER/CER → `results/summary.md`

### Phase 4 — Decision
- [ ] Worth integrating? Inference latency / VRAM / accuracy delta on **our** audio, not theirs.

## Hardware notes

- Qwen3-ASR-1.7B base ≈ 3–4 GB FP16; LoRA + router add a few hundred MB.
- Should fit on a single 12 GB+ GPU for inference. Training (A2S-SFT) is out of scope for this eval.

## Repo layout (planned)

```
megaasr-exp/
├── README.md           # this file
├── .gitignore
├── Mega-ASR/           # upstream clone — gitignored
├── ckpt/               # downloaded weights — gitignored
├── data/
│   ├── audio/          # eval clips (gitignored if large)
│   └── eval.jsonl      # ground-truth references
├── results/            # prediction JSONLs + summary
└── scripts/            # any helpers we add (download, batch eval, baseline runners)
```
