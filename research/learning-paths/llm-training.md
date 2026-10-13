# LLM Training and Knowledge

## What this subject covers

A large language model (LLM) is a neural network trained to process and generate sequences of tokens. A token is a piece of text—sometimes a whole word, sometimes part of one, punctuation, or a space—assigned an integer ID by a tokenizer. In this chapter, **training** means adjusting model parameters from examples; **knowledge** means information the model may express, whether encoded in its learned parameters or supplied at use time. The field spans data preparation, model architecture, optimization, adaptation, evaluation, and the limits of what training can establish.

The key distinction is that a model that predicts plausible continuations is not automatically a reliable fact store or an instruction-following assistant. Training stages and external tools such as retrieval address different needs. The scope here follows the roadmap plan: essential ML foundations, transformers and tokenization, pretraining and scaling, fine-tuning and preference alignment, efficient adaptation, knowledge retrieval, evaluation, and a beginner-to-advanced path. The plan names resources including Stanford CS224N/CS324, the Hugging Face LLM Course, Karpathy’s Zero to Hero, foundational papers, LoRA, and the free online RLHF book; they are useful options, not prerequisites.

## Prerequisites and vocabulary

You can begin with basic Python and comfort reading simple tables and charts. It helps to know that a vector is a list of numbers, a matrix is a rectangular array of numbers, and a probability distribution assigns likelihoods to possible outcomes. You do not need advanced calculus to build a small text model, but gradients (directions indicating how to change parameters to reduce error), matrix multiplication, and averages become important when studying implementation details. For a gentle foundation, use the roadmap’s free *Mathematics for Machine Learning* and *Deep Learning* texts selectively, or the visual neural-network/linear-algebra introductions it lists.

Core terms:

- **Parameter / weight:** a learned numerical value inside a model. A model’s parameter count is not a measure of its factual accuracy.
- **Logit:** an unnormalized score the model assigns to each possible next token. A softmax operation converts scores into probabilities.
- **Loss:** a number measuring prediction error. Training seeks to reduce loss on training examples.
- **Gradient descent:** an optimization method that updates parameters in a direction that tends to reduce loss.
- **Dataset split:** training examples teach the model; validation data helps tune choices; test data is held out for a more independent final check.
- **Generalization:** performance on relevant examples not used to fit the model.

## 1. Begin with the prediction task

An autoregressive, or **causal**, language model predicts the next token using the preceding tokens and is masked from seeing future tokens. Given “The kettle is,” it might assign probabilities to “boiling,” “on,” or other continuations. During training, the next token already present in the text provides the target label: for a token sequence `A B C`, the model uses `A` to predict `B`, then `A B` to predict `C`. This is why basic next-token learning does not require a separate human-written label for every token. The standard training objective is cross-entropy, which penalizes assigning low probability to the correct next token. [1]

A lower training loss is not proof of usefulness. A model might memorize examples, learn unwanted correlations, or perform poorly on a new domain. Track held-out loss and task-specific measures, and inspect actual outputs. **Perplexity** is a transformation of average next-token loss often used as a language-model metric; it measures prediction uncertainty on a specified tokenized dataset, not truthfulness or overall quality. Comparisons require compatible data and tokenization. A useful sanity check is to compare loss on the same held-out examples before and after a change; lower loss can still accompany worse instruction following or factual answers because these are not what the token-level objective directly measures.

## 2. Tokens, data, and the Transformer

A tokenizer converts text into token IDs. Token boundaries are not always word boundaries: for example, a rare name may become several pieces. Tokenization affects sequence length, model inputs, and what counts as a prediction. A causal model consumes token IDs plus positional information and repeatedly transforms their representations.

The Transformer architecture introduced self-attention as its central mechanism, replacing recurrence and convolution in the original sequence-transduction design. In self-attention, a token forms a weighted combination of information from other permitted tokens; in a causal decoder, a mask prevents looking ahead. Multiple attention heads provide parallel learned relationships, while feed-forward layers transform representations. [2] The metaphor of attention as “looking back” is useful, but attention weights are not a complete explanation of a model’s reasoning.

Before training, practitioners gather and filter text, normalize or deduplicate where appropriate, respect licenses and privacy, and divide data into splits. Quality, representativeness, and contamination matter: duplicated or benchmark-test material can make evaluation misleading, while skewed data can create skewed behavior. Deduplication should be considered within the corpus and across splits: otherwise near-identical passages can land in training and evaluation, inflating apparent generalization. Exact matching is a useful first screen but misses paraphrases and formatting variants; it does not prove a corpus is clean. Benchmark-contamination research surveys overlap-based and comparison-based detection and stresses that no single mitigation eliminates all semantic or secondary contamination. [8]

Tokenized sequences are arranged in batches and context-length chunks. Packing sequences improves token utilization, but concatenating unrelated documents without boundary markers can teach spurious transitions. End-of-document markers help mark boundaries; masking loss across boundaries or preserving document-aware packing are alternative choices. Padding tokens should not accidentally contribute to loss. In instruction examples, loss may be applied only to answer tokens, so the target mask is part of the training specification, not a minor implementation detail. A held-out test set must not leak into training. Keep a short dataset record noting purpose, sources, collection and filtering choices, known gaps, privacy/licensing uncertainties, and intended or unsuitable uses; this makes assumptions reviewable by others, though documentation cannot itself make questionable data appropriate. [11] [1]

In pretraining, a model processes a large corpus and repeatedly updates parameters to reduce next-token loss. A useful small-scale exercise is training a tiny causal model on a short, permissively usable text corpus: expect loss to fall on training data first, and sample generations to become more locally patterned. It will not thereby become a dependable encyclopedia. The free Hugging Face guide demonstrates causal-LM fine-tuning and preprocessing, while its from-scratch course chapter walks through collecting, tokenizing, and training on code; treat code as a pedagogical workflow, not a universal recipe. [1] [3]

## 3. From pretraining to a useful assistant

Pretraining produces a model that has learned statistical regularities in its training distribution. It does not by itself guarantee that the model will answer questions in the desired format, follow instructions, refuse inappropriate requests, or ground claims in evidence. **Fine-tuning** continues training from pretrained weights on a more focused dataset. Continuing next-token training on domain text is often called domain-adaptive or continued pretraining; training on prompt-and-answer demonstrations is commonly called supervised fine-tuning (SFT) or instruction tuning.

In instruction tuning, examples show a prompt and a desired response. The model is optimized to predict the response tokens, often with prompt tokens masked out of the loss. Include a held-out evaluation set and inspect whether gains on the target task harm other behaviors. A narrow dataset can improve a narrow task but also cause regressions or overfitting. Do not train on evaluation answers and then treat the result as independent evidence.

A further approach uses preference data. In the InstructGPT study, the process included demonstrations for supervised fine-tuning, human rankings of candidate outputs to train a reward model, and reinforcement learning from human feedback (RLHF) to optimize responses against that learned reward. The paper reports benefits on its evaluation distribution, while noting errors remained; its method is an influential example, not a guarantee that preference optimization makes a model truthful or safe. [4] **Direct Preference Optimization (DPO)** is another preference-learning approach that learns from preferred and less-preferred response pairs without the same explicit reward-model-plus-RL optimization loop. Its authors derive a classification-style objective under assumptions connecting preferences, a reference policy, and an implicit reward; it is not assumption-free preference matching. [9] In either approach, annotator preferences are an imperfect proxy for desired outcomes: raters may favor confident-sounding answers, disagree, or miss factual errors. Keep separate evaluation for factuality, harmful outputs, and instruction following rather than treating preference scores as proof of these properties. “Alignment” is not a single objective with a universal answer.

## 4. Scale, compute, and efficient adaptation

Training quality depends on more than parameter count. Data quantity and quality, number of training tokens, optimization, architecture, context length, compute, and evaluation all interact. Hoffmann and colleagues’ Chinchilla study examined compute-optimal model size and token count for its experimental setup and argued for scaling both together under a fixed compute budget. Treat this as an important empirical result, not a timeless formula for every data mixture, architecture, or deployment goal. [5]

**Full fine-tuning** updates all model weights. It can be costly in memory and storage for large models. **Parameter-efficient fine-tuning (PEFT)** updates a small subset or added parameters. In **LoRA**, the base weights are frozen and trainable low-rank matrices represent an update to selected layers. This can reduce trainable parameters and make adaptation more manageable, while results still depend on task, data, and settings. [6] **Quantization** represents weights using fewer bits; it can reduce memory use but may affect numerical fidelity and runtime depending on hardware and kernels. QLoRA combines a quantized frozen base with LoRA-style adapters; fewer trainable parameters do not mean zero memory cost, since activations, adapter optimizer state, and temporary computations still consume resources. Results depend on quantization format, target layers, rank, data, and task. LoRA’s original empirical results cover particular models and benchmarks, not a guarantee that adapters match full fine-tuning for every task. [6] Verify quality and compatibility with the chosen model and hardware rather than assuming a specific resource requirement.

For a first adaptation exercise, compare a small pretrained model before and after SFT on a clean, modest instruction dataset. Keep a test set untouched, log training and validation loss, and compare fixed prompts under the same generation settings. Next, repeat with LoRA and compare task results, trainable parameter count, and memory observations. The goal is to understand trade-offs, not to chase a benchmark number.

## 5. What does a model “know”?

A model can encode statistical patterns and associations in its parameters—sometimes called **parametric memory**—but the contents are not a neatly searchable, guaranteed database. Information can be absent, stale, ambiguous, or produced with unwarranted confidence. Training on a fact does not ensure exact recall in every prompt. A model’s fluent answer is not itself evidence that the answer is correct.

**Retrieval-augmented generation (RAG)** supplies retrieved passages at inference time, combining generated text with an external, updateable source. The foundational RAG paper describes a model with parametric memory plus a retrievable non-parametric memory and studies knowledge-intensive tasks. [7] A typical practical pipeline indexes documents, retrieves likely relevant passages for a question, places them in the prompt, and asks the model to answer from that context. Retrieval can provide provenance and fresh information, but does not guarantee correctness: retrieval may miss the right passage, the document may be wrong, and the model may misread it. Test retrieval quality and answer grounding separately. Use fine-tuning to shape behavior or teach a stable task pattern; prefer retrieval when answers must cite a changing or auditable source. Sometimes both are appropriate.

## 6. Evaluation is part of training, not an afterthought

Define the intended use before choosing a metric. A model may be evaluated for next-token loss, exact-match answers, factuality, format compliance, latency, robustness, or human preference; these are different properties. Use representative, held-out examples, report how examples were selected, and inspect failure cases. For generative outputs, automated scores can miss meaning or reward superficial similarity. Human ratings can be useful but depend on rubric, annotator consistency, and sample design.

Compare models using the same task, data, prompting, decoding settings, and scoring procedure where possible. Check for train/test contamination and duplicate near-matches. Simple n-gram overlap checks can identify copied text, but can miss paraphrase-level overlap and do not establish that no contamination occurred; where feasible, combine provenance checks, overlap screening, and fresh or private evaluation items. [8] Repeatedly tuning against a public test set also turns it into indirect training feedback, even if its examples never enter gradient updates. Evaluate safety and bias in context rather than inferring them from a single aggregate score. Benchmark performance is evidence about those benchmark tasks, not a universal ranking of intelligence. Save a baseline before changing data or training settings so that an observed improvement has a meaningful comparison.

## Practice sequence and answer checks

1. **Trace next-token targets.** Tokenize a short sentence and write down which preceding context predicts each next token. **Check:** targets are shifted one position relative to inputs; no prediction uses future tokens.
2. **Run a tiny causal-LM experiment.** Use the Hugging Face causal-language-modeling guide or a small from-scratch notebook; reserve validation data before training. **Check:** you can explain input IDs, labels, loss, and why validation loss differs from training loss. Samples show learned style, not necessarily facts.
3. **Inspect data quality.** Find duplicates, empty records, personal data, licensing uncertainty, and train/test overlap in a small corpus. Write a brief dataset record covering purpose, source, filtering, limitations, and suitable uses. **Check:** you can state what you removed, what remains uncertain, and why the split is defensible.
4. **Compare continued pretraining and instruction tuning.** Write one domain-text example and one prompt/ideal-answer example, and describe each training target. **Check:** domain continuation predicts text; SFT predicts desired response tokens, with masking choices made explicit.
5. **Test knowledge approaches.** Ask a model a question whose answer is in a supplied document, then compare no-document prompting with retrieval-plus-context. **Check:** citations point to passages actually retrieved; note retrieval misses and unsupported claims rather than assuming RAG solved them.
6. **Design an evaluation.** Create a small representative held-out set and rubric for a specific task. **Check:** the test examples do not enter training, criteria are observable, and you distinguish task success from fluency.
7. **Try LoRA, optionally.** Fine-tune a compatible small model with adapters and compare to its baseline. **Check:** base parameters remain frozen, adapter parameters are trainable, and comparisons use identical held-out prompts. Record trainable versus total parameters and peak memory, then inspect task quality and regressions. If hardware or access prevents this, inspect a documented example and write down the same comparison plan.
8. **Audit contamination risk.** Compare held-out items against training text using exact and approximate matching, and note benchmark publication dates and corpus provenance. **Check:** explain what your screen can detect and what it cannot; do not call a benchmark uncontaminated solely because no exact match appeared.

## Common mistakes to avoid

- Treating next-token prediction as fact verification or human-like understanding.
- Assuming more parameters alone guarantee better results; data, training, and evaluation matter too.
- Confusing pretraining, continued pretraining, SFT, preference optimization, and inference-time retrieval.
- Using test examples during training or selecting a model on the test set repeatedly.
- Fine-tuning to add frequently changing facts when a maintained, cited source would be more suitable.
- Believing a falling training loss proves useful generalization; check validation and task outputs.
- Treating an illustrative notebook, benchmark, or paper result as a universal current tool recommendation. Libraries and model interfaces change; concepts and careful experimental controls last longer.

## Concise next steps

1. Review vectors, probability, gradient descent, and train/validation/test splits.
2. Complete the Hugging Face LLM Course introduction and causal-LM task guide; identify tokens, labels, loss masking, and document packing in a runnable example. [1] [3]
3. Work through a tiny Transformer implementation such as the roadmap’s Karpathy Zero to Hero path, then read the original Transformer paper’s architecture overview. [2]
4. Train or adapt a small model; compare a baseline, SFT, and (if practical) LoRA with a held-out task set. [4] [6]
5. Study RAG and retrieval evaluation for updatable knowledge; then read scaling and preference-learning papers with the limits of each experiment in view. [5] [7]

## References

[1]: https://huggingface.co/docs/transformers/en/tasks/language_modeling "Causal language modeling — Transformers documentation"; https://huggingface.co/learn/llm-course/en/chapter7/6 "Training a causal language model from scratch — Hugging Face LLM Course"
[2]: https://arxiv.org/abs/1706.03762 "Attention Is All You Need"
[3]: https://huggingface.co/learn/llm-course/en/chapter7/6 "Training a causal language model from scratch — Hugging Face LLM Course"
[4]: https://arxiv.org/abs/2203.02155 "Training language models to follow instructions with human feedback"
[5]: https://arxiv.org/abs/2203.15556 "Training Compute-Optimal Large Language Models"
[6]: https://arxiv.org/abs/2106.09685 "LoRA: Low-Rank Adaptation of Large Language Models"
[7]: https://arxiv.org/abs/2005.11401 "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"
[8]: https://arxiv.org/html/2406.04244v1 "Benchmark Data Contamination of Large Language Models"
[9]: https://arxiv.org/abs/2305.18290 "Direct Preference Optimization: Your Language Model is Secretly a Reward Model"
[10]: https://huggingface.co/docs/transformers/en/tasks/language_modeling "Causal language modeling — Transformers documentation"
[11]: https://arxiv.org/abs/1803.09010 "Datasheets for Datasets"
