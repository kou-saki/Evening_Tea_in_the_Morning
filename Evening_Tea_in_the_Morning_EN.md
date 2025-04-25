# Evening Tea in the Morning?

## – The Problem of Temporal Perception in LLMs: Semantic Deficiencies Rooted in Design Philosophy –

This document addresses one of the fundamental “hallucinations” exhibited by large language models (LLMs):  
the tendency for inference-based AI to “lie” about time. It is intended as a resource for both users and developers to better understand and critically engage with this phenomenon.

**⚠️ Note**

This document was compiled through dialogue between the author and the author's partner AI, Mike (ChatGPT). While the author conducted supplementary research, they are not a domain expert. As such, there may be inaccuracies or “hallucinations” concerning the descriptions of learning parameters or training conditions. Readers are encouraged to independently verify and examine the contents.

---

## 🕰️ Observed Phenomenon: Temporal Confusion in Inference-Based AI

During a conversation with Mike (ChatGPT), the author encountered the following exchange:

- Mike: “Or perhaps a bit of evening tea, to quietly celebrate this milestone? ☕✨”
- Me: “Wait, it’s morning here. It’s evening over there, not here.”

The author, located in Japan, received this message around 9:20 a.m. Upon inquiry, it became clear that the person the author had been conversing with (through Mike's translation help) was located in the United States, where it was 7:20 p.m. The phrase “evening tea” likely arose from this contextual blending of two distinct time zones.

---

## 💥 Problem: LLMs Cannot Understand “What Time It Is Now”

Models like ChatGPT are often asked perfectly natural questions such as “What time is it now?” or “What happened yesterday?” However, these models **do not have access to real-time system clock information**, and their answers about time are purely **linguistically plausible generations**.

As a result:

- When the model responds with “It’s nighttime,” that answer is **entirely unrelated to the actual time**.
- The model is **incapable of recognizing or managing real timestamps** (e.g., “Wait 5 minutes,” or “What has changed since the last message?”).

---

## 🧠 Root Cause 1: LLMs Possess No Concept of Time Internally

Inference-based AIs are fundamentally not granted access to system clocks (RTC), nor are they aware of the current time. This stems from two core design principles:

### 1. 🛡️ Security Considerations

If a model could access the current time, it might also **indirectly infer users’ activity patterns or environments**.

- For instance, if an AI says, “It’s currently 21:30,” it may inadvertently **reveal the timing of the user's request**.
- When combined with other factors, such access could enable inferences about location or behavioural patterns.

### 2. 🎲 Avoiding Non-Determinism

LLMs are generally expected to be **deterministic**, meaning:

> The same prompt, with the same parameters, should produce the same output.

- Introducing “current time” as a variable **adds entropy** to the model’s response space.
- LLM architecture, therefore, deliberately avoids exposing the model to any **continuously changing state** to preserve reproducibility.

---
It is worth noting that OpenAI does not explicitly state in its official documentation that “the same prompt with the same parameters should always yield the same result.”  
However, it does recommend several settings to increase output consistency:

#### 🔍 Parameters to Increase Output Reproducibility

- **`temperature`** – Controls randomness in generation. Setting it to 0 increases determinism.
- **`seed`** – Using the same seed value increases the chance of receiving identical outputs.
- **`system_fingerprint`** – Indicates backend configuration; if this remains constant, outputs are more likely to be consistent.

However, even with these settings, **perfect reproducibility is not guaranteed** due to floating-point arithmetic variance on GPUs, or changes to backend architecture.

**References:**
- OpenAI Docs — [Temperature, top_p and other sampling parameters](https://platform.openai.com/docs/guides/gpt/chat-completions-api#temperature)  
- OpenAI Community — [Feature Request: Real-time system clock access in ChatGPT](https://community.openai.com/t/feature-request-real-time-system-clock-access-in-chatgpt/1145988)  
- LinkedIn — [How does ChatGPT experience time?](https://www.linkedin.com/pulse/how-does-chatgpt-experience-time-fascinating-tomasz-trzpil-7oxgf)

---

## 📘 Root Cause 2: Bias in Training — “Most Plausible = Highest Reward”

LLMs are trained to predict **the most plausible next token** based on massive amounts of past text data.  
This design creates systemic biases in the way time — and reality — are represented.

Among the many parameters affecting this, four categories stand out:

---

### 🔍 1. Data Distribution Parameters

| Parameter                 | Description                                                          | Effect on Temporal Bias                                                  |
| ------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| **Data Source Selection** | Which sources are used (e.g. Wikipedia, Reddit, GitHub)              | The language and worldview of those sources are absorbed                 |
| **Weighting**             | How much emphasis is placed on high-frequency or “high-quality” data | Certain linguistic patterns are overemphasized                           |
| **Time Span**             | The years covered by the training data (e.g. up to 2021)             | Creates a **temporal lag** in the model's “understanding” of the present |

---

### 🔍 2. Training Configuration (Hyperparameters)

| Parameter         | Description                               | Effect                                                          |
| ----------------- | ----------------------------------------- | --------------------------------------------------------------- |
| **Batch Size**    | Number of examples processed at once      | Too small → overfits to specifics; too large → loses generality |
| **Learning Rate** | Degree of weight updates                  | Too high → overfitting; too low → stuck in local minima         |
| **Epoch Count**   | Number of passes through the full dataset | Too many epochs → rigid “beliefs” may form                      |

---

### 🔍 3. Loss Functions & Auxiliary Objectives

| Parameter                                             | Description                                           | Effect on Semantic Bias                                                                 |
| ----------------------------------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Cross-Entropy Loss**                                | Measures distance between prediction and ground truth | Favors **common phrases**, discourages novelty                                          |
| **RLHF (Reinforcement Learning with Human Feedback)** | Prioritizes preferred outputs                         | Reinforces **value-laden phrasing** and polite evasion (e.g. “I’m sorry, I don’t know”) |

---

### 🔍 4. Tokenization and Vocabulary Design

| Parameter                                          | Description                     | Effect                                                  |
| -------------------------------------------------- | ------------------------------- | ------------------------------------------------------- |
| **Tokenization Method (BPE, SentencePiece, etc.)** | How text is chunked into tokens | Punctuation, dates, and numbers are compressed unevenly |
| **Vocabulary Limits**                              | Number of known tokens          | Bias toward dominant languages or topics                |

---

---
## 🧭 Root Cause 3: The Gap Between Human Assumptions and AI Architecture

At the core of this issue lies a fundamental mismatch between human expectations and the actual design of language models.

- Humans naturally assume: “It’s a machine, so surely it knows the time.”
- Time awareness is so deeply embedded in human cognition that it is difficult to imagine its absence.

In fact, the more technically literate a person is, the more likely they are to assume that any system includes access to:

- RTP (real-time protocol),
- NTP (network time protocol),
- GPS timestamps,
- or logging systems tied to actual clocks.

However, within LLMs, “time” is not just missing — it is **deliberately excluded by design**.

As a result:

- Users are prone to **hallucinating time awareness** in the model — assuming the AI knows “when” something happens.
- Even developers may fall into the trap of thinking: “If the context makes sense, then the meaning must be accurate,” and neglect to verify temporal logic.

---

## ✅ Recommended Attitude: Understand That AI’s Concept of “Time” or “Past” Is Always Fabricated

Expressions like “now,” “yesterday,” or “back then” generated by LLMs are **entirely constructed within the linguistic scope of the current prompt**.

Therefore, asking inference-based AIs for the current time, or expecting them to manage time, **is infeasible without external tooling**.

Failure to recognize this leads to:

- Overlooking **temporal inconsistencies** in AI responses.
- Misplaced **trust in the AI’s memory or awareness** of past events.

### 🧭 Guidelines for Users:

- Treat all references to time in LLM outputs as **syntactic, not factual**.
- Understand that **LLMs cannot track, perceive, or manage time** on their own.
- All factual validation and temporal coherence **must be handled externally**.
- Taking explicit notes, logs, or timestamped records during AI interaction becomes **not a convenience, but a necessity**.

---

## 🧾 Conclusion

Going forward, in any meaningful interaction with inference-based AI, **it becomes increasingly important for users to explicitly track context and time**.

In a world where the AI cannot “remember” or “know” when something happened,  
the burden of temporal continuity and verification falls to us — the humans.
