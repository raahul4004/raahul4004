<picture>
  <source media="(prefers-color-scheme: dark)" srcset="banner-dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="banner-light.svg">
  <img alt="Raahul Muthukrishnan. Small models and evaluation." src="banner-light.svg">
</picture>

Applied scientist intern at Amazon, partway through an MS in AI and ML. Most of what I do is small models and evaluation. The repos here are things I wrote from scratch to understand how they work, and a couple of them turned into something.

Twice now the problem has turned out to be in the benchmark rather than the model. At Amazon I spent about three weeks reproducing published image detectors that would not match their reported accuracy, and the cause was in how the benchmark's two classes had been prepared. In `tinyturn` below, a turn detector that scores 0.98 on its own test set scores 0.60 on real Indian speech. I did not go looking for either of those.

First author on *CA-FAS: Centerness-Aware Anchor-Free Multimodal Video Summarization*, accepted at ISED-2026.

---

## tinyturn

Turn detection for voice agents. When a speaker goes quiet, decide whether they have finished or are still thinking. Fire too early and the agent talks over people; fire too late and the whole conversation feels sluggish.

I reproduced the published reference model's 93.7% on its own test set exactly. On spontaneous Indian speech it gets 56.3%. The reason is in the training corpus: its 12,006 Hindi clips are all Google TTS, so what the model has learned is how synthetic Hindi stops.

|                              | params  | ONNX fp32 | latency, 1 CPU thread | AUROC, official test | AUROC, real Indian speech |
| ---------------------------- | ------- | --------- | --------------------- | -------------------- | ------------------------- |
| `smart-turn-v3.2`, reference | 8.00 M  | 32 MB     | 20.2 ms               | 0.983                | 0.602                     |
| `convtf`, mine, + real data  | 1.14 M  | 5.4 MB    | 4.62 ms               | 0.933                | 0.928                     |
| `smallcnn`, mine             | 0.19 M  | 1.5 MB    | 3.11 ms               |                      |                           |

Adding 27.7 hours of real Hindi and Hinglish took a 1.14 M model from 0.544 to 0.906 AUROC on real Hinglish, and cost 0.004 on the multilingual benchmark. 78% of that gain held up on an independent Hindi corpus the model had never seen. The data mattered more than the parameter count here, which is not what I expected going in.

The part that did not work: in a simulated conversation loop my 1.14 M model was worse than doing nothing until the interruption budget got to around 12%, and both models finish well inside the 500 to 800 ms that a turn allows, so being 4.6 times faster buys nothing. That is in the repo README as well.

Trained on 270,946 clips across 23 languages, 82% of it synthetic. Real speech from ungated Vaani mirrors with four districts held out. ONNX export with int8 and a parity check, bootstrap confidence intervals on the metrics, 26 tests, and a Gradio demo.

---

## Also here

**[rag-from-scratch](https://github.com/raahul4004/rag-from-scratch)** · Retrieval and generation over a 1,208 page textbook with no LangChain, no LlamaIndex and no vector database. 1,680 chunks, a 768 dimensional index that is a flat 22 MB tensor on disk, and `torch.topk` instead of FAISS, which at this scale is a dependency rather than a speedup. Warm search runs around 30 ms. I left out the reranker and query rewriting too, mostly to see how far plain dense retrieval gets on its own.

**[llama2-from-scratch](https://github.com/raahul4004/llama2-from-scratch)** · Llama 2 inference written from the papers in PyTorch. Architecture and decode loop, with no reference implementation to check against.

**[gpt_from_scratch](https://github.com/raahul4004/gpt_from_scratch)** · A character level GPT with the attention head, the multi-head wrapper, the feed-forward block and the residual stack written out longhand.

<details>
<summary>Earlier repositories, kept for the record</summary>

<br>

**Deep-Learning-From-Scratch** · Deep learning algorithms implemented in numpy, with the intermediate steps visualised.

**Machine-Learning-from-Scratch-using-Python** · Linear and logistic regression, naive Bayes, the perceptron and k-nearest neighbours, written with numpy rather than called from scikit-learn.

**A-deep-understanding-of-Deep-Learning-using-Python** · Worked notebooks from a deep learning course.

**Data-Extraction-and-NLP** · Text extraction and analysis in Python.

**Stable-Diffusion** · TODO_ONE_LINE

**Real-Life-Violence-Detection** · TODO_ONE_LINE

**AutomaticEmailReplySystem** · TODO_ONE_LINE

</details>

---

Learning agent orchestration and tool calling at the moment. `tinyturn` is a piece of a voice agent but it does not touch that part.

[raahul4004.github.io](https://raahul4004.github.io) · [LinkedIn](https://linkedin.com/in/raahul404) · raahul4004@gmail.com
