# LLM Fundamentals

## 1. How LLMs Work

### What is an LLM?

A **Large Language Model (LLM)** is a neural network trained on massive amounts of text data to predict the next token in a sequence. At its core, an LLM is a very sophisticated autocomplete system — but one powerful enough to reason, summarize, translate, write code, and more.

### The Architecture: Transformers

Modern LLMs are built on the **Transformer** architecture (introduced in the 2017 paper *"Attention Is All You Need"*).

Key components:

- **Embedding Layer** — Converts each input token into a high-dimensional vector (a list of numbers) that captures its meaning.
- **Self-Attention Mechanism** — Allows the model to look at *all* other tokens in the input when processing any single token. This is how the model understands context (e.g., knowing that "bank" means a river bank vs. a financial bank based on surrounding words).
- **Feed-Forward Layers** — After attention, each token's representation passes through dense neural network layers that transform and refine it.
- **Multiple Layers (Depth)** — These attention + feed-forward blocks are stacked dozens or hundreds of times. Early layers capture syntax and simple patterns; deeper layers capture abstract reasoning and world knowledge.
- **Output Layer** — Produces a probability distribution over the entire vocabulary for the next token.

### The Training Process

LLMs are trained in stages:

#### Stage 1: Pre-training

- The model reads billions of documents (books, websites, code, etc.).
- Objective: **next-token prediction** — given all previous tokens, predict the next one.
- The model adjusts its billions of internal parameters to minimize prediction error.
- This stage requires thousands of GPUs running for weeks/months and costs millions of dollars.
- After pre-training, the model has broad knowledge but is not yet useful as an assistant.

#### Stage 2: Fine-tuning / Instruction Tuning

- The model is trained on curated datasets of (instruction, response) pairs.
- Teaches the model to follow instructions, answer questions, and behave helpfully.

#### Stage 3: RLHF (Reinforcement Learning from Human Feedback)

- Human raters rank different model outputs for the same prompt.
- A reward model is trained on these rankings.
- The LLM is then optimized to produce outputs the reward model scores highly.
- This aligns the model to be helpful, harmless, and honest.

### How Generation Works (Inference)

When you send a prompt to an LLM:

```
Input: "The capital of France is"
```

1. The input is **tokenized** (split into tokens).
2. Tokens pass through all transformer layers.
3. The model outputs a probability distribution for the next token:
   - "Paris" → 92%, "Lyon" → 2%, "the" → 1%, ...
4. A token is **sampled** from this distribution (influenced by temperature — see Section 4).
5. The selected token is appended to the input, and the process **repeats** until a stop condition is met.

This is called **autoregressive generation** — the model generates one token at a time, feeding its own output back as input.

---

## 2. Tokens

### What is a Token?

A **token** is the smallest unit of text that an LLM processes. Tokens are NOT the same as words.

A token can be:
- A whole word: `"hello"` → 1 token
- A part of a word: `"understanding"` → `"under"` + `"standing"` → 2 tokens
- A single character: `"a"` → 1 token
- Punctuation: `"."` → 1 token
- A space + word: `" the"` → 1 token (leading space is part of the token)

### Tokenization

LLMs use algorithms like **Byte-Pair Encoding (BPE)** to build their vocabulary:

1. Start with individual characters as tokens.
2. Find the most frequently occurring pair of adjacent tokens in the training data.
3. Merge that pair into a single new token.
4. Repeat thousands of times until the vocabulary reaches a target size (e.g., 50,000–100,000 tokens).

**Example** (BPE in action):

```
Original text:  "lower lowest"
Character level: l o w e r   l o w e s t
After merges:    lo w er   lo w est
After more:      low er   low est
Final tokens:    "low" "er" " " "low" "est"
```

### Why Tokens Matter

| Aspect | Why It Matters |
|---|---|
| **Cost** | API pricing is per-token (both input and output). More tokens = higher cost. |
| **Speed** | Each token requires a forward pass through the model. More tokens = slower generation. |
| **Context limit** | Every model has a max token limit (see Section 3). Your prompt + response must fit within it. |
| **Behavior** | The model "thinks" in tokens, not words. Rare words get split into more tokens, which can affect how well the model handles them. |

### Token Counts — Rules of Thumb (English)

- 1 token ≈ 4 characters ≈ 0.75 words
- 1 word ≈ 1.3 tokens
- 100 tokens ≈ 75 words
- 1 page of text ≈ 300–400 tokens
- A typical novel (80,000 words) ≈ 100,000 tokens

### Special Tokens

Models also use invisible **special tokens** to structure input:

| Token | Purpose |
|---|---|
| `<BOS>` / `<s>` | Beginning of sequence |
| `<EOS>` / `</s>` | End of sequence (tells model to stop) |
| `<PAD>` | Padding for batch processing |
| `<|system|>` | Marks the system prompt |
| `<|user|>` | Marks user messages |
| `<|assistant|>` | Marks model responses |

---

## 3. Context Window

### What is a Context Window?

The **context window** (or context length) is the maximum number of tokens the model can process in a single request — including both the input (prompt) and the output (response).

Think of it as the model's **working memory**. Everything the model "knows" about your current conversation must fit inside this window.

### Context Window Sizes

| Model | Context Window |
|---|---|
| GPT-2 (2019) | 1,024 tokens |
| GPT-3 (2020) | 2,048 tokens |
| GPT-3.5 Turbo | 4,096 / 16,384 tokens |
| GPT-4 (2023) | 8,192 / 128,000 tokens |
| Claude 3.5 / 4 | 200,000 tokens |
| Gemini 1.5 Pro | 1,000,000+ tokens |

### How the Context Window is Used

```
|<-------------- Context Window (e.g., 200K tokens) ------------->|
| System Prompt | Conversation History | Latest User Msg | Response |
|   ~500 tokens |    ~2,000 tokens     |   ~200 tokens    | ~1,000   |
```

Everything inside the window is visible to the model. **Nothing outside exists** — the model has zero memory of anything beyond the current window.

### What Happens When You Exceed It?

- The request fails with an error, OR
- Older messages are **truncated** (dropped from the beginning), OR
- A summarization strategy compresses older context.

### Why Context Window Size Matters

- **Short context** → the model "forgets" earlier parts of long conversations or documents.
- **Long context** → the model can process entire codebases, books, or lengthy conversation histories in a single call.
- **Cost** → larger context = more computation = higher latency and cost.

### The "Lost in the Middle" Problem

Research shows that LLMs pay the most attention to information at the **beginning** and **end** of the context window. Information in the **middle** is more likely to be overlooked. This means:

- Put important instructions at the start (system prompt).
- Put the most relevant context near the end (close to the question).
- Don't rely on the model perfectly recalling details buried deep in a long document.

---

## 4. Temperature

### What is Temperature?

**Temperature** is a parameter that controls the **randomness** of the model's output. It adjusts the probability distribution over tokens before sampling.

### How It Works (Mathematically)

After the model computes raw scores (logits) for each possible next token, temperature is applied:

```
adjusted_probability(token) = softmax(logit / temperature)
```

| Temperature | Effect on Distribution | Behavior |
|---|---|---|
| **0** | All probability mass on the top token | Deterministic — always picks the most likely token. Greedy decoding. |
| **0.1 – 0.3** | Very peaked distribution | Highly focused and predictable. Minor variation. |
| **0.7 – 1.0** | Moderate spread | Balanced — coherent but with some creativity. |
| **1.5 – 2.0** | Very flat distribution | Highly random — creative but often incoherent or nonsensical. |

### Visual Intuition

Imagine the model choosing the next word after "The cat sat on the":

```
Temperature = 0 (Deterministic)
  "mat"   ████████████████████  95%
  "floor" █                      3%
  "bed"   ▏                      1%
  "roof"  ▏                      1%

Temperature = 1.0 (Balanced)
  "mat"   █████████████          50%
  "floor" ██████                 25%
  "bed"   ████                   15%
  "roof"  ██                     10%

Temperature = 2.0 (Creative/Chaotic)
  "mat"   █████                  28%
  "floor" ████                   24%
  "bed"   ████                   23%
  "roof"  ████                   22%
```

### When to Use Each Temperature

| Use Case | Recommended Temperature |
|---|---|
| Code generation | 0 – 0.2 |
| Factual Q&A | 0 – 0.3 |
| Data extraction / structured output | 0 |
| General conversation | 0.5 – 0.7 |
| Creative writing / brainstorming | 0.7 – 1.0 |
| Poetry / experimental text | 1.0 – 1.5 |

### Related Sampling Parameters

Temperature often works alongside these:

- **Top-k sampling** — Only consider the top `k` most likely tokens (e.g., top 40). Cuts off the long tail of unlikely tokens.
- **Top-p (nucleus) sampling** — Only consider tokens whose cumulative probability reaches `p` (e.g., 0.9 = top 90% of probability mass). Dynamically adjusts how many tokens are considered.
- **Frequency / Presence penalty** — Reduces the probability of tokens that have already appeared, reducing repetition.

**Example combining parameters:**

```
temperature: 0.7    → moderate randomness
top_p: 0.9          → consider tokens covering 90% of probability
top_k: 50           → but never more than 50 options
presence_penalty: 0.6 → discourage repeating topics
```

---

## Quick Reference Summary

| Concept | One-Liner |
|---|---|
| **LLM** | A neural network that predicts the next token using the Transformer architecture |
| **Token** | The smallest text unit the model reads/writes (~4 chars in English) |
| **Context Window** | The model's total working memory for input + output (measured in tokens) |
| **Temperature** | Controls randomness: 0 = deterministic, 1+ = creative/chaotic |
