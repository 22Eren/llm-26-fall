# Course pretraining plan: a model between GPT-2 and Marin

Draft, September 11, 2026. A plan for the model the class trains during the
semester: what size, how many tokens, which data, on what schedule, and what
must be verified first. Numbers marked *measured* come from tests run on the
course cluster on September 11; numbers marked *assumed* are stated
assumptions to be replaced by measurements.

## 1. Target in one sentence

Train a **1B-parameter decoder** on **about 35B tokens** of English, Chinese,
code, and math with the Qwen3 tokenizer, plus a **ladder of 30M / 125M /
350M proxies** on the same data, so that every lecture from Week 7 on has a
real run to look at and every student ablation is cheap.

Why this point: GPT-2 XL is 1.5B parameters on about 10B tokens (2019);
Marin 8B is 8B parameters on about 12T tokens on TPU pods; Qwen3-0.6B/1.7B
are trained on 36T tokens. A 1B model on 35B tokens is a few times past the
compute-optimal 20 tokens/parameter, is affordable on the course cluster in
hours, and is directly comparable to public baselines of the same size
(Qwen3-0.6B-Base, SmolLM2-360M/1.7B, GPT-2 XL). Anything Qwen3-scale in data
(trillions of tokens) is out of reach by two orders of magnitude and would
not teach more.

## 2. Compute budget

Course cluster (shared, K8s "distributed training" jobs have no per-job cap):

| Pool | Cards | Memory per card | Status |
| --- | ---: | ---: | --- |
| Alibaba PPU (ZW810 class) | 56 (7 nodes × 8) | 96 GB | one card *measured*: 397 TFLOP/s bf16, 107 TFLOP/s fp32 on 8192² matmul; torch 2.6 with a CUDA-12.6-compatible SDK; SDPA and bf16 work |
| MetaX MXC550 | 120 (15 nodes × 8) | 64 GB | not yet tested; separate vendor toolchain |

Training cost with the 6·N·D rule (weights only; attention adds about 10% at
sequence length 2048):

| Run | Params N | Tokens D | FLOPs | Hours on 8 PPUs | Hours on 32 PPUs |
| --- | ---: | ---: | ---: | ---: | ---: |
| proxy | 30M | 0.6B | 1.1e17 | 0.1 | — |
| proxy | 125M | 2.5B | 1.9e18 | 0.5 | — |
| proxy | 350M | 7B | 1.5e19 | 3.5 | 0.9 |
| **flagship** | **1B** | **35B** | **2.1e20** | **49** | **12** |
| stretch | 1.5B | 50B | 4.5e20 | 104 | 26 |

*Assumed*: 35% model-FLOP utilization end to end, i.e. about 140 TFLOP/s per
PPU, and linear scaling to 8–32 cards. Both assumptions are to be replaced by
the two measurements in Section 6. Even at 20% utilization the flagship is a
one-day job on 32 cards, so the plan has slack for one full restart.

## 3. Data: about 35B tokens, five sources

All sources are public, downloadable through the Hugging Face mirror that the
cluster can reach, and already filtered by their publishers; we add our own
light pass (Section 4) so students see the pipeline, not to improve on it.

| Domain | Source | Tokens used | Why this source |
| --- | --- | ---: | --- |
| English web | FineWeb-Edu (`HuggingFaceFW/fineweb-edu`, `sample-100BT`) | 14B | classifier-filtered educational web text; the current default for small models (SmolLM) |
| Chinese web | CCI3-HQ (`BAAI/CCI3-HQ`) and FineWeb-2 `zho_Hans` | 10B (7B + 3B) | the two largest openly licensed high-quality Chinese web corpora; two sources so students can compare them |
| Code | The Stack v2 smol (`bigcode/the-stack-v2-train-smol-ids`) or StarCoderData, Python-heavy | 4B | small, deduplicated, permissively licensed |
| Math | FineMath 4+ (`HuggingFaceTB/finemath`) and OpenWebMath | 4B (2B + 2B) | the two math corpora whose extraction we study in the regex tasks |
| Reference | Wikipedia EN and ZH (`wikimedia/wikipedia`), Cosmopedia v2 (synthetic textbooks) | 3B (2B + 1B) | clean, encyclopedic; Cosmopedia is the only synthetic source and is labelled as such |

Mixture: English 40%, Chinese 29%, code 11%, math 11%, reference 9%. The
Chinese share is deliberately high for a Fudan class and is the main lever
students can vary in the Week 8 data-policy ablation.

Sizes: 35B tokens is about 140 GB of text and 70 GB tokenized (uint16 is
not enough for the 151,669-entry Qwen vocabulary; use uint32 or a packed
format), well inside the 3 TB project store. Throughput on the preprocessing
host was *measured* at 27 MB/s for tiktoken encoding across 32 processes, so
tokenizing the whole corpus is under two hours; regex filters run at similar
speed; MinHash deduplication is the slow step and should run once on the
CPU pool with datatrove.

Tokenizer: reuse the **Qwen3 tokenizer** for the main runs, so perplexities
and token counts are comparable with Qwen3-0.6B-Base and the vocabulary
covers Chinese well. Students still train their own BPE in Lecture 01–02
tasks; a 32K in-house BPE is an optional ablation, not the main line.

## 4. Pipeline (what students build, in task form)

1. **Download and inspect** each source through the mirror; record license,
   size, and a 20-document sample (Week 3–4 tasks).
2. **Filter**: the C4/Gopher line and document rules, the language ID check,
   and the PII regexes (Weeks 2–4 regex tasks are exactly these functions).
3. **Deduplicate**: exact hashes per document, then MinHash near-duplicates
   within each source (Week 8, data lecture).
4. **Decontaminate**: 10-gram overlap against every benchmark we will report
   (DeepSeekMath's rule), before tokenization.
5. **Tokenize and pack** into fixed 2048-token sequences with document
   boundaries marked; write shards with a manifest of source, tokens, and
   the filter versions used.
6. **Mix** by sampling shards according to the table above; one config file
   per policy so ablations differ only in that file.

Every stage writes a small report (documents in, documents out, top removal
reasons, three examples of each) that goes on the course site, the same way
the survey chart does.

## 5. Schedule aligned with the lectures

| Week | Lecture | Cluster work | Students |
| --- | --- | --- | --- |
| 2–4 | N-grams, embeddings, attention | verify the two measurements in Section 6; download sources; tokenizer counts | regex and tokenizer tasks |
| 5–6 | make-up class, Transformer | dedup, decontaminate, tokenize; train the 30M and 125M proxies on the full mixture | shape checks on the decoder |
| 7 | Pretraining and decoding | **launch the 1B flagship** (about one day on 32 PPUs); publish the training curve live | resumed-run and failure-diagnosis exercises on the 125M |
| 8 | Data preparation | 125M ablations: two data policies (Chinese share, with/without Cosmopedia) at equal token budget | compare the two policies |
| 9 | Compute and scaling | fit the 30M–350M ladder; predict the 1B loss, then compare with the real run | resource estimate versus measured value |
| 10 | Evaluation | evaluate the 1B against Qwen3-0.6B-Base, SmolLM2, GPT-2 XL; contamination audit | audit one benchmark item |
| 11–12 | SFT, preferences | SFT and a DPO pass on the 1B with open instruction data | before/after on one prompt set |
| 13–16 | RAG, inference, diffusion, agents | serve the model (vllm is installed); latency tables | projects use the checkpoints |

## 6. Verify before committing (two measurements, one week)

1. **End-to-end throughput on one PPU**: a training step of the 125M, 350M,
   and 1B configs (bf16, SDPA, AdamW, 2048 tokens) to get real tokens/s and
   utilization. A ready script exists; it needs the GPU session running.
2. **Multi-card scaling**: the same 350M config on 8 cards (one node) and on
   16 cards (two nodes) with PyTorch DDP/FSDP through the vendor's
   collective library. If cross-node scaling is poor, the flagship shrinks to
   what one node can do in a day (about 1B × 15B tokens at 35% utilization),
   which is still past GPT-2 XL.

Also to settle: whether the MXC550 pool runs the same PyTorch code (if yes,
proxies and student jobs move there and the PPU pool is reserved for the
flagship); and a checkpoint policy (every 1B tokens, kept on the project
store, with the exact data manifest and commit hash).

## 7. Risks and the fallback

- **Vendor accelerators, not NVIDIA.** Fused kernels, `torch.compile`, and
  flash attention may be missing or slow; the plan assumes plain SDPA. Marin's
  Levanter stack is JAX on TPU and is not an option here; the code base is
  PyTorch (nanoGPT/CS336 style).
- **No GitHub from the cluster.** Code arrives by copy over SSH; datasets via
  the Hugging Face mirror; packages via the Aliyun PyPI mirror.
- **Session storage is ephemeral.** Everything lives under the project store;
  jobs are written to resume from the last checkpoint.
- **Licensing.** Every source above is redistributable for research; Chinese
  web corpora should be checked once more against their cards before use.
- **Fallback.** If scaling or stability fails, the 350M proxy on 7B tokens
  becomes the class model. It still supports every lecture from Week 7 on.
