# Lecture 02: N-gram Language Models

September 16, 2026 · CS40008.01 · Three 45-minute teaching periods

## Central question and learning objectives

How can a model assign a probability to a sentence, and how do we know whether
one model is better than another?

Students should be able to write the chain-rule factorization of a sentence
probability, state the Markov assumption behind an $N$-gram model, estimate
bigram parameters by maximum likelihood, explain perplexity as a normalized
inverse probability and as a branching factor, apply additive smoothing and
interpolation to unseen $n$-grams, explain why pretraining pipelines still score
data with n-gram models, describe why a neural probabilistic LM replaces count
tables with embeddings, and identify size, truncation, and sampling confounds
in an experiment that replaces its training data with model samples.

The deck is a port of the instructor's
[Spring 2026 Lecture 02](https://baojian.github.io/llm-26/slides/lecture-02-slides/)
into the shared Reveal.js template. The content is kept; dense Spring slides
are split so that each slide holds one idea at the template's font sizes.
The deck has **39 slides** in three sections (N-gram LMs and smoothing;
evaluation and perplexity; neural probabilistic LMs), with three repeated
outlines and a final reading page. Smoothing is one page at the end of the
first section, right after the count table and OOV where the zeros are on
screen; notebook practices P02 and P03 retain worked examples of additive
smoothing and held-out interpolation. Two slides are new in the Fall version and tie the
lecture to the course pipeline: the n-gram perplexity filter (closing the
evaluation section) and the self-training loop (before the readings). The numbers on the perplexity, filter, and loop slides come from
`scripts/lecture02_experiments.py` on held-out shards with the Qwen3
tokenizer; `assets/lecture02-results.json` holds the full output.

The companion note [How good is a language model? The metrics in use](../../docs/lecture-02-lm-metrics.md)
shows that loss in nats, bits per token, perplexity, and bits per byte are
one quantity in four units and cites where GPT-3, Kaplan, Chinchilla, Llama 3,
DeepSeek, OLMo 2, DCLM, and Qwen3 report each one; the PDFs are in `papers/`.

## Teaching sequence

| Period | Minutes | Slides | Content and activity |
| --- | --- | --- | --- |
| 1 | 0–10 | 1–4 | Title, outline, why sentences need probabilities: speech recognition, MT, spell correction |
| 1 | 10–25 | 5–9 | Unknown data distribution, KL objective, empirical log-likelihood, factorization, training samples |
| 1 | 25–45 | 10–15 | Chain rule, next-token prediction clip (play 2 min), unigram/bigram/trigram, Markov assumption, build steps, parameter matrix and MLE |
| 2 | 0–20 | 16–21 | E01 (5 min): toy bigram MLE; restaurant-review counts; sentence boundaries; OOV and UNK; smoothing in one page (zeros in the table, add-δ, interpolation, Kneser–Ney in two sentences) |
| 2 | 20–38 | 22–27 | Outline; data split and extrinsic evaluation; E02 (3 min): propose a metric, and one zero makes the test set score −∞; perplexity; E03 (3 min): digits example; BPB definition; WSJ and demonstration rows |
| 2 | 38–45 | 28–30 | Unigram sampling, interval figure, WSJ samples; bigram sampling question |
| 3 | 0–10 | 31–32 | The perplexity filter as stage 2b of the pipeline; N-gram summary |
| 3 | 10–36 | 33–37 | Outline; four NPLM pages: task and embeddings, forward inference, training, improvements over N-grams (with the Week 9 preview) |
| 3 | 36–42 | 38 | The loop in miniature: a bigram retrained on its own samples; identify changing token budgets and propose an equal-budget control |
| 3 | 42–45 | 39 | Toolkits and readings; preview of Week 3 (embeddings) |

Breaks fall between periods and are outside the 135 teaching minutes. The three
E exercises total 11 minutes; the notebook's P01–P04 are for after class. These are
**ungraded practices**. Assignment A1 (Tokenization and language models) is released this
week and defined by the course website and the instructors' repository.

## Notebook correspondence

The classroom notebook is [lecture-02-exercise.ipynb](lecture-02-exercise.ipynb).
It runs offline with the standard library only.

| Slide | Notebook | Evidence to discuss |
| --- | --- | --- |
| Toy example of training a bigram LM | E01 | Six MLE fractions; the column for history `I` sums to one |
| Intrinsic evaluation | E02 | One unseen bigram makes the test log-likelihood $-\infty$ |
| Perplexity: interpretation | E03 | Uniform digits give perplexity 10 for every length |
| Sentence sampling | P01 | Bigram sampling from BOS until EOS |
| Smoothing N-gram LMs in one page | P02 (optional) | The 8×8 Laplace probability and reconstituted-count tables recomputed from the counts |
| Smoothing N-gram LMs in one page | P03 (optional) | Interpolation weight tuned on a held-out sentence (best $\lambda=0.6$); test perplexity 5.61 and 0.829 bits per byte, the course's shared unit |
| The loop, in miniature | P04 (optional) | A bigram retrained on its own samples: held-out loss 1.587 → 3.066 in five rounds (seed 7); exact mixture sampling on a fixed vocabulary, forty capped samples per round |

## Differences from the Spring deck

- Dense Spring slides were split (objective, chain rule, $N$-gram definitions,
  practical issues, evaluation, perplexity, sampling) so each slide fits the
  shared template without inline styles.
- The smoothing section (Spring slides 20–27: intuition, additive smoothing
  with the 8×8 tables, interpolation, Katz backoff, Good–Turing, Kneser–Ney)
  is one page, per the instructor's decision on September 11, 2026. The
  worked examples live in notebook practices P02 and P03.
- The bigram MLE on the parameter-matrix slide is written with the history in
  the denominator, $C(v_j v_i)/C(v_j)$; the Spring slide had $C(v_i v_j)/C(v_i)$.
- The closing slide lists Chapter 3 as this lecture's reading; the Spring slide
  pointed to Chapters 4–5 for its next lecture.
- Whiteboard prompts became timed exercises E01–E03 with revealed answers.
- Slide *Comparing language models* keeps the WSJ word perplexities and
  defines BPB and adds three demonstration rows on held-out text
  (TinyStories, OpenWebText, Fineweb-Edu-Chinese; a 24 MiB training cap
  per source, Qwen3 tokenizer, interpolation weights tuned on a dev split):
  trigram 1.12 / 2.03 / 2.04.
  These are separate from A1's 10 MB training samples and student-trained BPE.
  The current evaluator supports n-grams; later neural adapters should share
  its scored-token, text-byte, and document-boundary conventions.
- Slide *Smoothing in one page* gained two sentences on Kneser–Ney, because
  the NPLM results table names back-off KN as the baseline.
- The Spring outline had four sections; smoothing is no longer its own
  section but the last page of the first one (instructor's decision,
  September 15, 2026), and the deck has three outlines instead of four.
- New slide *N-grams in a 2026 pipeline*: a WikiText-103 reference trigram
  scores 1,200 web documents and 300 TinyStories documents. Both histograms
  use shared bins and percentages. The chart is a CCNet-inspired teaching
  analogue with a course-selected cutoff, not the original score or a quality
  guarantee. The JSON records the recomputed medians and cutoff. Dolma is
  discussed as a counterexample that chose heuristic quality filters.
- New slide *The loop, in miniature*: a bigram on the original TinyStories
  corpus is retrained on 2,000 capped samples per round; BPB rises from 1.30
  to 1.94 in this run. The training budget shrinks from 6,076,505 tokens to
  at most 256,000. The slide states the confounds and asks for controls; it
  does not establish the benefit of a filter or judge. Hover shows each
  round's actual token budget. P04 is a separate twelve-sentence example.
- The summary slide now says "the first neural LM (NPLM)" for today and defers
  RNNs and Transformers to Week 4; the Spring text said "RNN/Transformer".
- The readings slide adds Bengio et al. (2003), Jurafsky and Martin Ch. 6 (current draft),
  CCNet (Wenzek et al., 2020), and Brants et al. (2007), and drops SRILM.
- Notebook P03 is interpolation with a held-out $\lambda$ instead of Good–Turing;
  P04 (self-training loop) is new. Tests in `tests/test_lecture_02.py` check
  both, plus `tests/test_pipeline_ngram.py` for the shared estimator.
- The experiments use the original local September 15 Hugging Face slices,
  rerun after the September 16 review. Input hashes, split fingerprints,
  normalized overlap counts, and actual selected reference-article counts
  are recorded in the JSON. The upstream commit revisions were not recorded
  during the original download; see [experiment provenance](assets/README.md).
- Review fixes cover default vocabulary inference, matching sampling and
  scoring distributions, fixed notebook vocabulary, scored EOS counts,
  UTF-8 byte limits, and article-level reference splits. Regression tests
  check these behaviors rather than requiring one seed to demonstrate collapse.
