# LLM Fundamentals

## 1. How LLMs Work

### What is an LLM?

A **Large Language Model (LLM)** is a neural network trained on massive amounts of text data to predict the next token in a sequence. At its core, an LLM is a very sophisticated autocomplete system — but one powerful enough to reason, summarize, translate, write code, and more.

### The Architecture: Transformers

Modern LLMs are built on the **Transformer** architecture (introduced in the 2017 paper *"Attention Is All You Need"*).

### Transformer Architecture — Visual Overview

```
 INPUT: "The cat sat on"
   │
   ▼
┌──────────────────────────────────────────────────────────┐
│                   TOKEN EMBEDDINGS                       │
│                                                          │
│   "The"    "cat"    "sat"    "on"                        │
│    │        │        │        │                           │
│    ▼        ▼        ▼        ▼                           │
│  [0.2,..] [0.8,..] [0.1,..] [0.5,..]   ← each token    │
│                                           becomes a      │
│                                           vector         │
└──────────────┬───────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────┐
│               POSITIONAL ENCODING                        │
│                                                          │
│   Adds position information so the model knows           │
│   token order (Transformers have no built-in             │
│   sense of sequence unlike RNNs)                         │
│                                                          │
│   Token Vector + Position Vector = Input Embedding       │
└──────────────┬───────────────────────────────────────────┘
               │
               ▼
┌══════════════════════════════════════════════════════════╗
║            TRANSFORMER BLOCK  (× N layers)              ║
║     (GPT-3: 96 layers, Claude/GPT-4: ~100+ layers)     ║
║                                                          ║
║  ┌────────────────────────────────────────────────────┐  ║
║  │         MULTI-HEAD SELF-ATTENTION                  │  ║
║  │                                                    │  ║
║  │  Each token asks: "Which other tokens should I     │  ║
║  │  pay attention to?"                                │  ║
║  │                                                    │  ║
║  │  ┌─────────┐  ┌─────────┐  ┌─────────┐            │  ║
║  │  │ Head 1  │  │ Head 2  │  │ Head N  │            │  ║
║  │  │         │  │         │  │         │   Multiple  │  ║
║  │  │ Q  K  V │  │ Q  K  V │  │ Q  K  V │   heads    │  ║
║  │  │  ╲ │ ╱  │  │  ╲ │ ╱  │  │  ╲ │ ╱  │   attend   │  ║
║  │  │  Attn   │  │  Attn   │  │  Attn   │   to diff  │  ║
║  │  │ Scores  │  │ Scores  │  │ Scores  │   patterns  │  ║
║  │  └────┬────┘  └────┬────┘  └────┬────┘            │  ║
║  │       └────────────┼────────────┘                  │  ║
║  │                    ▼                               │  ║
║  │              [ Concatenate ]                       │  ║
║  │                    │                               │  ║
║  │              [ Linear Layer ]                      │  ║
║  └────────────────────┬───────────────────────────────┘  ║
║                       │                                  ║
║                 ┌─────▼─────┐                            ║
║                 │ Add & Norm│  ← Residual connection     ║
║                 └─────┬─────┘    + Layer normalization   ║
║                       │                                  ║
║  ┌────────────────────▼───────────────────────────────┐  ║
║  │          FEED-FORWARD NETWORK (FFN)                │  ║
║  │                                                    │  ║
║  │   Input ──→ [ Linear ] ──→ [ ReLU/GELU ] ──→      │  ║
║  │             (expand 4×)     (activation)           │  ║
║  │                                                    │  ║
║  │         ──→ [ Linear ] ──→ Output                  │  ║
║  │             (compress)                             │  ║
║  │                                                    │  ║
║  │   This is where "knowledge" is stored.             │  ║
║  │   Acts as a key-value memory bank.                 │  ║
║  └────────────────────┬───────────────────────────────┘  ║
║                       │                                  ║
║                 ┌─────▼─────┐                            ║
║                 │ Add & Norm│  ← Another residual        ║
║                 └─────┬─────┘                            ║
║                       │                                  ║
╚═══════════════════════╪══════════════════════════════════╝
         ▲              │              Repeat
         └──────────────┘              N times
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│                    OUTPUT HEAD                            │
│                                                          │
│   Final hidden state ──→ Linear Layer ──→ Softmax        │
│                                                          │
│   Produces probability over entire vocabulary:           │
│                                                          │
│     "the"   → 0.52      "mat"    → 0.03                 │
│     "a"     → 0.11      "table"  → 0.02                 │
│     "his"   → 0.07      "floor"  → 0.01                 │
│     "her"   → 0.05      ...      → ...                  │
│                                                          │
│   Selected token (via temperature/sampling): "the"       │
└──────────────────────────────────────────────────────────┘
               │
               ▼
 OUTPUT: "the"  →  append to input  →  re-run for next token
```

### Self-Attention — How It Actually Works

```
 Example: "The cat sat on the mat because it was tired"
                                          ▲
                                          │
                            What does "it" refer to?

 Self-Attention computes three vectors per token:

 ┌─────────────────────────────────────────────────────────┐
 │  Q (Query)  = "What am I looking for?"                  │
 │  K (Key)    = "What do I contain?"                      │
 │  V (Value)  = "What information do I provide?"          │
 └─────────────────────────────────────────────────────────┘

 For the token "it":
   Q("it") compares against every K:

     Q("it") · K("The")     = 0.1   (low relevance)
     Q("it") · K("cat")     = 0.8   ← HIGH (it = cat!)
     Q("it") · K("sat")     = 0.2
     Q("it") · K("on")      = 0.05
     Q("it") · K("the")     = 0.1
     Q("it") · K("mat")     = 0.3
     Q("it") · K("because") = 0.1
     Q("it") · K("it")      = 0.4
     Q("it") · K("was")     = 0.15

 These scores (after softmax) become attention weights:

   "The" "cat" "sat" "on" "the" "mat" "because" "it" "was"
    2%    45%   5%   1%    2%    15%     3%      20%   7%
          ████                   ██              ███

 The output for "it" is a weighted sum of all V vectors,
 heavily influenced by "cat" → so the model understands
 "it" refers to "cat".
```

### Causal (Autoregressive) Masking

```
 LLMs use CAUSAL masking — each token can only attend
 to itself and tokens BEFORE it (not future tokens):

              The   cat   sat   on
   The     [  ✓     ✗     ✗     ✗  ]
   cat     [  ✓     ✓     ✗     ✗  ]
   sat     [  ✓     ✓     ✓     ✗  ]
   on      [  ✓     ✓     ✓     ✓  ]

   ✓ = can attend    ✗ = masked (cannot see future)

 This is why LLMs generate left-to-right, one token
 at a time — they are never "cheating" by looking ahead.
```

### Multi-Head Attention — Why Multiple Heads?

```
 Different attention heads learn to focus on different
 linguistic relationships:

 ┌──────────────────────────────────────────────────┐
 │  Head 1: Syntactic (subject-verb agreement)      │
 │  "The dogs  ───────────→  are running"           │
 │              (plural)       (plural verb)         │
 ├──────────────────────────────────────────────────┤
 │  Head 2: Coreference (pronoun resolution)        │
 │  "Mary said she ──→ would leave"                 │
 │              (she = Mary)                         │
 ├──────────────────────────────────────────────────┤
 │  Head 3: Semantic (meaning relationships)        │
 │  "The doctor treated the patient"                │
 │       ▲                    ▲                      │
 │       └── medical ─────── ┘                      │
 ├──────────────────────────────────────────────────┤
 │  Head 4: Positional (nearby context)             │
 │  "New ←→ York"  "ice ←→ cream"                  │
 │   (adjacent word compounds)                      │
 └──────────────────────────────────────────────────┘

 GPT-3 uses 96 attention heads per layer × 96 layers
 = 9,216 different attention patterns!
```

### Full Data Flow — End to End

```
 ┌─────────────┐
 │ Raw Text    │  "The cat sat on"
 └──────┬──────┘
        ▼
 ┌─────────────┐
 │ Tokenizer   │  ["The", " cat", " sat", " on"]  → [464, 3857, 3332, 319]
 └──────┬──────┘
        ▼
 ┌─────────────┐
 │ Embedding   │  Token IDs → Dense vectors (dim: 4096–12288)
 │ + Position  │  + Positional info added
 └──────┬──────┘
        ▼
 ┌─────────────┐
 │ Layer 1     │  Self-Attention → FFN → surface-level patterns
 └──────┬──────┘
        ▼
 ┌─────────────┐
 │ Layer 2     │  Self-Attention → FFN → basic syntax
 └──────┬──────┘
        ▼
      . . .
        ▼
 ┌─────────────┐
 │ Layer N/2   │  Self-Attention → FFN → semantic understanding
 └──────┬──────┘
        ▼
      . . .
        ▼
 ┌─────────────┐
 │ Layer N     │  Self-Attention → FFN → abstract reasoning
 └──────┬──────┘
        ▼
 ┌─────────────┐
 │ LM Head     │  Project to vocabulary → softmax → probabilities
 └──────┬──────┘
        ▼
 ┌─────────────┐
 │ Sampling    │  temperature / top-p / top-k → select token
 └──────┬──────┘
        ▼
 ┌─────────────┐
 │ Output      │  "the" → append & repeat
 └─────────────┘
```

### Scale of Modern LLMs

```
 ┌──────────────┬──────────────┬──────────┬─────────────────┐
 │    Model     │  Parameters  │  Layers  │  Hidden Size    │
 ├──────────────┼──────────────┼──────────┼─────────────────┤
 │ GPT-2       │    1.5B      │    48    │     1,600       │
 │ GPT-3       │    175B      │    96    │    12,288       │
 │ LLaMA 2 70B │    70B       │    80    │     8,192       │
 │ GPT-4       │   ~1.8T*     │   ~120*  │   ~12,288*      │
 │ Claude 3+   │   undisclosed│          │                 │
 └──────────────┴──────────────┴──────────┴─────────────────┘
  * estimated / rumored
```

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
