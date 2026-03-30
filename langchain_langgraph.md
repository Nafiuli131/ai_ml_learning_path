# LangChain & LangGraph — Complete Guide (100% Context)

## Part 1: LangChain

---

## What is LangChain?

LangChain is a **Python/JS framework** that makes it easy to build applications powered by LLMs.

### The Problem LangChain Solves

Without LangChain, building an LLM app requires you to manually handle:

```python
# ❌ Without LangChain — you do EVERYTHING manually

import openai

# 1. Manually format prompts
prompt = f"You are a helpful assistant.\n\nContext: {context}\n\nQuestion: {question}"

# 2. Manually call the API
response = openai.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": prompt}]
)

# 3. Manually parse the response
answer = response.choices[0].message.content

# 4. Manually handle memory (conversation history)
history.append({"role": "user", "content": question})
history.append({"role": "assistant", "content": answer})

# 5. Manually load documents
with open("file.txt") as f:
    text = f.read()

# 6. Manually chunk documents
chunks = [text[i:i+500] for i in range(0, len(text), 500)]

# 7. Manually embed and store
embeddings = [get_embedding(c) for c in chunks]

# 8. Manually search
results = search_similar(query_embedding, embeddings)

# Imagine doing this for 50 different LLM features... 😩
```

```python
# ✅ With LangChain — pre-built components you plug together

from langchain_openai import ChatOpenAI
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import Chroma
from langchain.chains import RetrievalQA

# Load → Chunk → Embed → Store → Search → Answer — in a few lines
loader = PyPDFLoader("file.pdf")
docs = loader.load()
chunks = RecursiveCharacterTextSplitter(chunk_size=500).split_documents(docs)
vectorstore = Chroma.from_documents(chunks, embeddings)
chain = RetrievalQA.from_chain_type(llm=ChatOpenAI(), retriever=vectorstore.as_retriever())
answer = chain.invoke("What is this document about?")
```

### LangChain = LEGO blocks for LLM apps

Each block does one thing. You snap them together to build complex applications.

---

## But Wait — Does LangChain Just "Remove Code"? (Common Misconception)

No! "Less code" is a side effect, not the purpose. Here's what LangChain **actually** gives you:

### 1. Portability (Swap LLM providers without rewriting your app)

```python
# Your entire app uses this ONE line to define the LLM:
llm = ChatOpenAI(model="gpt-4")

# Tomorrow your boss says "switch to Claude" — you change ONE line:
llm = ChatAnthropic(model="claude-sonnet-4-20250514")

# EVERYTHING else stays the same — prompts, chains, tools, memory, RAG — untouched.
```

Without LangChain, switching providers means rewriting every API call, response parsing, error handling, and streaming logic. With LangChain, it's one line.

### 2. Ecosystem (700+ integrations you don't have to build)

```
Without LangChain:
  Load a PDF → write PyPDF2 code
  Load a web page → write BeautifulSoup code
  Load from Notion → write Notion API code
  Load from YouTube → write youtube-transcript-api code
  Each one: different API, different format, different error handling

With LangChain:
  Load a PDF → PyPDFLoader("file.pdf").load()
  Load a web page → WebBaseLoader("url").load()
  Load from Notion → NotionDirectoryLoader("path").load()
  Load from YouTube → YoutubeLoader.from_youtube_url("url").load()
  All return the SAME Document format. Write once, works with everything.
```

### 3. Free Superpowers (Streaming, Batch, Async — you get them without writing them)

When you build a chain with LCEL (`prompt | llm | parser`), you automatically get:

| Feature | Without LangChain | With LangChain |
|---------|-------------------|----------------|
| **Streaming** | Write complex async generator code | `chain.stream()` — free |
| **Batch processing** | Write threading/multiprocessing code | `chain.batch()` — free |
| **Async** | Write asyncio code for every step | `chain.ainvoke()` — free |
| **Retries** | Write retry logic with backoff | Built into Runnables |
| **Logging/Tracing** | Write custom logging | Callbacks + LangSmith — free |

### 4. Composability (Build complex pipelines from simple blocks)

```
Without LangChain:
  You write ONE giant function that does everything — hard to test,
  hard to modify, hard to reuse parts.

With LangChain:
  prompt_chain = prompt | llm | parser
  rag_chain = retriever | format_docs | prompt | llm | parser

  Each piece is independent. You can:
  - Test each piece separately
  - Reuse pieces across different chains
  - Swap one piece without touching others
  - Read the flow at a glance
```

### 5. State Management (Memory, conversation history, sessions)

Managing conversation history across multiple requests, multiple users, with different storage backends (RAM, Redis, SQL) — this is genuinely hard to build correctly. LangChain handles it.

### Summary: LangChain is NOT just "less code"

```
┌──────────────────────────────────────────────────────────┐
│  What LangChain gives you:                                │
│                                                           │
│  1. PORTABILITY    → Switch LLM providers in one line     │
│  2. ECOSYSTEM      → 700+ integrations, same interface    │
│  3. SUPERPOWERS    → Streaming, batch, async for free     │
│  4. COMPOSABILITY  → Build complex from simple (LCEL)     │
│  5. STATE          → Memory, sessions, persistence        │
│  6. OBSERVABILITY  → Logging, tracing, debugging          │
│                                                           │
│  When you DON'T need LangChain:                           │
│  - Simple one-off API call to OpenAI                      │
│  - Tiny script with no complexity                         │
│                                                           │
│  When you DO need LangChain:                              │
│  - RAG pipelines (load + chunk + embed + search + answer) │
│  - Multi-provider support                                 │
│  - Chatbots with memory                                   │
│  - Tools + agents                                         │
│  - Complex multi-step workflows                           │
└──────────────────────────────────────────────────────────┘
```

---

## LangChain Architecture (The Big Picture)

```
┌─────────────────────────────────────────────────────────────────────┐
│                        LANGCHAIN ECOSYSTEM                         │
│                                                                     │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │  langchain-core   │  │  langchain       │  │  langchain-      │  │
│  │                   │  │                  │  │  community       │  │
│  │  - Base classes   │  │  - Chains        │  │                  │  │
│  │  - LCEL           │  │  - Agents        │  │  - 700+ integra- │  │
│  │  - Runnables      │  │  - Memory        │  │    tions         │  │
│  │  - Prompts        │  │  - Retrieval     │  │  - Vector stores │  │
│  │  - Output parsers │  │  - Callbacks     │  │  - Doc loaders   │  │
│  └──────────────────┘  └──────────────────┘  │  - LLM providers │  │
│                                               └──────────────────┘  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │  langchain-openai │  │  langchain-      │  │  langgraph       │  │
│  │                   │  │  anthropic       │  │                  │  │
│  │  OpenAI models    │  │  Claude models   │  │  Agent workflows │  │
│  │  & embeddings     │  │  & embeddings    │  │  State machines  │  │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘  │
│                                                                     │
│  ┌──────────────────┐  ┌──────────────────┐                        │
│  │  LangSmith       │  │  LangServe       │                        │
│  │                   │  │                  │                        │
│  │  Debugging &      │  │  Deploy chains   │                        │
│  │  Monitoring       │  │  as REST APIs    │                        │
│  └──────────────────┘  └──────────────────┘                        │
└─────────────────────────────────────────────────────────────────────┘
```

### Installation

```bash
# Core LangChain
pip install langchain

# LLM providers (install what you need)
pip install langchain-openai        # For OpenAI (GPT-4, embeddings)
pip install langchain-anthropic     # For Anthropic (Claude)
pip install langchain-google-genai  # For Google (Gemini)

# Community integrations
pip install langchain-community

# Vector stores
pip install chromadb faiss-cpu pinecone-client

# Document loaders
pip install pypdf unstructured

# LangGraph (for agents)
pip install langgraph
```

---

## LangChain Core Concepts

### 1. Chat Models (Talking to LLMs)

The most fundamental building block — a wrapper around LLM APIs.

```python
from langchain_openai import ChatOpenAI
from langchain_anthropic import ChatAnthropic

# OpenAI
llm = ChatOpenAI(
    model="gpt-4",
    temperature=0,         # 0 = deterministic, 1 = creative
    api_key="sk-..."
)

# Anthropic (Claude)
llm = ChatAnthropic(
    model="claude-sonnet-4-20250514",
    temperature=0,
    api_key="sk-ant-..."
)

# Use it — same interface regardless of provider!
response = llm.invoke("What is Python?")
print(response.content)
# "Python is a high-level programming language..."
```

**Why this matters:** You can swap `ChatOpenAI` with `ChatAnthropic` and your entire app still works. LangChain abstracts away the provider differences.

#### Message Types

LLMs work with messages, not plain text:

```python
from langchain_core.messages import SystemMessage, HumanMessage, AIMessage

messages = [
    SystemMessage(content="You are a helpful Python tutor."),
    HumanMessage(content="What is a list comprehension?"),
    # AIMessage(content="...") ← the LLM's response goes here
]

response = llm.invoke(messages)
print(response.content)
# "A list comprehension is a concise way to create lists in Python..."
print(type(response))
# AIMessage
```

```
Message Types:
┌─────────────────┬─────────────────────────────────────┐
│ SystemMessage   │ Instructions for the LLM's behavior │
│ HumanMessage    │ The user's input                    │
│ AIMessage       │ The LLM's response                  │
│ ToolMessage     │ Result from a tool/function call     │
└─────────────────┴─────────────────────────────────────┘
```

---

### 2. Prompt Templates

Instead of hardcoding prompts with f-strings, use templates:

```python
# ❌ Bad: Hardcoded f-string
prompt = f"Translate {text} to {language}"

# ✅ Good: LangChain template (reusable, composable)
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a translator. Translate text to {language}."),
    ("human", "{text}")
])

# Fill in the variables
formatted = prompt.invoke({"language": "French", "text": "Hello world"})
print(formatted)
# [SystemMessage(content="You are a translator. Translate text to French."),
#  HumanMessage(content="Hello world")]

# Chain it with an LLM
chain = prompt | llm
response = chain.invoke({"language": "French", "text": "Hello world"})
print(response.content)
# "Bonjour le monde"
```

#### Different Prompt Template Types

```python
# Simple string template
from langchain_core.prompts import PromptTemplate

template = PromptTemplate.from_template(
    "Tell me a {adjective} joke about {topic}"
)
result = template.invoke({"adjective": "funny", "topic": "cats"})

# Chat template with multiple messages
from langchain_core.prompts import ChatPromptTemplate

chat_template = ChatPromptTemplate.from_messages([
    ("system", "You are a {role}."),
    ("human", "{question}"),
])

# Few-shot template (give examples)
from langchain_core.prompts import FewShotChatMessagePromptTemplate

examples = [
    {"input": "happy", "output": "sad"},
    {"input": "hot", "output": "cold"},
]

example_prompt = ChatPromptTemplate.from_messages([
    ("human", "{input}"),
    ("ai", "{output}")
])

few_shot = FewShotChatMessagePromptTemplate(
    example_prompt=example_prompt,
    examples=examples,
)

final_prompt = ChatPromptTemplate.from_messages([
    ("system", "Give the opposite of each word."),
    few_shot,
    ("human", "{input}"),
])

chain = final_prompt | llm
response = chain.invoke({"input": "big"})
print(response.content)  # "small"
```

---

### 3. Output Parsers (Structured Output)

LLMs return raw text. Output parsers convert that into structured data.

```python
# ❌ Without parser: raw text
response = llm.invoke("List 3 Python frameworks")
print(response.content)
# "Here are 3 Python frameworks:\n1. Django\n2. Flask\n3. FastAPI"
# This is a string — hard to use in code!

# ✅ With parser: structured data
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.pydantic_v1 import BaseModel, Field

# Define the structure you want
class Frameworks(BaseModel):
    frameworks: list[str] = Field(description="List of framework names")

parser = JsonOutputParser(pydantic_object=Frameworks)

prompt = ChatPromptTemplate.from_messages([
    ("system", "List programming frameworks. {format_instructions}"),
    ("human", "List {count} {language} frameworks")
])

chain = prompt | llm | parser

result = chain.invoke({
    "count": 3,
    "language": "Python",
    "format_instructions": parser.get_format_instructions()
})

print(result)
# {"frameworks": ["Django", "Flask", "FastAPI"]}
print(type(result))
# dict ← now it's structured data you can use in code!
```

#### Common Output Parsers

```python
# String parser (just get the text)
from langchain_core.output_parsers import StrOutputParser
chain = prompt | llm | StrOutputParser()
result = chain.invoke({...})  # returns: "plain string"

# JSON parser
from langchain_core.output_parsers import JsonOutputParser
chain = prompt | llm | JsonOutputParser()
result = chain.invoke({...})  # returns: {"key": "value"}

# List parser
from langchain_core.output_parsers import CommaSeparatedListOutputParser
parser = CommaSeparatedListOutputParser()
chain = prompt | llm | parser
result = chain.invoke({...})  # returns: ["item1", "item2", "item3"]

# Pydantic parser (type-safe structured output)
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel

class MovieReview(BaseModel):
    title: str
    rating: float
    summary: str

parser = PydanticOutputParser(pydantic_object=MovieReview)
# Returns a MovieReview object with .title, .rating, .summary attributes
```

---

### 4. LCEL — LangChain Expression Language (The Pipe `|` Syntax)

#### What is LCEL? (Theory)

LCEL stands for **LangChain Expression Language**. It is a **declarative** way to compose LangChain components into chains (pipelines). Instead of writing imperative step-by-step code ("do this, then do that, then do that"), you **declare** what the pipeline looks like using the `|` (pipe) operator, and LangChain figures out how to execute it.

#### Why Was LCEL Created?

Before LCEL, LangChain had a class-based approach to building chains:

```python
# ❌ OLD way (before LCEL) — verbose, rigid, lots of boilerplate
from langchain.chains import LLMChain, SequentialChain

chain1 = LLMChain(llm=llm, prompt=prompt1, output_key="summary")
chain2 = LLMChain(llm=llm, prompt=prompt2, output_key="translation")

overall_chain = SequentialChain(
    chains=[chain1, chain2],
    input_variables=["text"],
    output_variables=["translation"]
)
result = overall_chain.run("Hello world")
# Problems:
# - Lots of classes to remember (LLMChain, SequentialChain, TransformChain...)
# - No streaming support
# - No async support
# - Hard to customize
# - Hard to read
```

The LangChain team realized that most LLM workflows follow a simple pattern: **data flows from one component to the next**, just like water through pipes. So they created LCEL — a simpler, more powerful way to express this.

```python
# ✅ NEW way (LCEL) — clean, readable, powerful
chain = prompt1 | llm | StrOutputParser() | prompt2 | llm | StrOutputParser()
result = chain.invoke({"text": "Hello world"})
# Same result, but:
# - One line instead of ten
# - Streaming works automatically
# - Async works automatically
# - Batch works automatically
# - Easy to read: "prompt → llm → parse → prompt → llm → parse"
```

#### The Core Idea: Runnables

Everything in LCEL is a **Runnable**. A Runnable is any object that has these methods:

```
┌─────────────────────────────────────────────────────┐
│                 RUNNABLE INTERFACE                    │
│                                                       │
│  Every component in LCEL implements these methods:    │
│                                                       │
│  .invoke(input)        → Single input, single output  │
│  .stream(input)        → Single input, streaming out  │
│  .batch([inputs])      → Multiple inputs, parallel    │
│                                                       │
│  .ainvoke(input)       → Async invoke                 │
│  .astream(input)       → Async stream                 │
│  .abatch([inputs])     → Async batch                  │
│                                                       │
│  Runnables include:                                   │
│  - ChatModels (ChatOpenAI, ChatAnthropic)            │
│  - Prompts (ChatPromptTemplate)                      │
│  - Output Parsers (StrOutputParser, JsonOutputParser) │
│  - Retrievers (vectorstore.as_retriever())           │
│  - Custom functions (via RunnableLambda)             │
│  - Other chains (chains are Runnables too!)          │
└─────────────────────────────────────────────────────┘
```

Because every component follows the **same interface**, they can all connect to each other. This is the power of LCEL — **uniformity**. A prompt, an LLM, a parser, a retriever, a custom function — they're all Runnables, they all have `.invoke()`, `.stream()`, `.batch()`, and they can all be piped together.

#### The Pipe `|` Operator — What It Really Means

The `|` operator is **not** some LangChain magic. It's Python's `__or__` dunder method. When you write:

```python
chain = prompt | llm | parser
```

Python translates this to:

```python
chain = prompt.__or__(llm).__or__(parser)
```

Which creates a **RunnableSequence** — a new Runnable that, when invoked, runs each component in order, passing the output of one as the input to the next.

```
prompt | llm | parser

Is equivalent to:

RunnableSequence(steps=[prompt, llm, parser])

When you call chain.invoke(input):
  1. result_1 = prompt.invoke(input)      → PromptValue
  2. result_2 = llm.invoke(result_1)      → AIMessage
  3. result_3 = parser.invoke(result_2)   → string/dict/object
  4. return result_3
```

#### Analogy: LCEL is Like a Factory Assembly Line

```
RAW MATERIALS (input)
    │
    ▼
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ Station 1│──→│ Station 2│──→│ Station 3│──→│ Station 4│
│ (Prompt) │   │ (LLM)   │   │ (Parser) │   │ (Custom) │
│          │   │          │   │          │   │          │
│ Takes    │   │ Takes    │   │ Takes    │   │ Takes    │
│ raw vars │   │ formatted│   │ AI       │   │ parsed   │
│ formats  │   │ prompt   │   │ message  │   │ data     │
│ them     │   │ generates│   │ extracts │   │ does     │
│          │   │ response │   │ text/json│   │ something│
└──────────┘   └──────────┘   └──────────┘   └──────────┘
    │
    ▼
FINISHED PRODUCT (output)

Each station:
- Receives the output of the previous station
- Does its ONE job
- Passes the result to the next station
- Doesn't need to know what happens before or after it
```

This is why LCEL is powerful:
- **Each component is independent** — it does one thing
- **Components are reusable** — use the same prompt in different chains
- **Components are swappable** — replace `ChatOpenAI` with `ChatAnthropic`, nothing else changes
- **The chain is readable** — you can see the flow at a glance

#### LCEL vs Java Streams/Lambdas — Same Concept!

If you know Java Streams, you already understand LCEL. The core idea is identical: **chain operations together, each transforms the data, pass it to the next.**

```java
// ☕ JAVA STREAMS — chain operations with .method()
List<String> result = names.stream()
    .filter(name -> name.length() > 3)      // step 1: filter
    .map(String::toUpperCase)                // step 2: transform
    .sorted()                                // step 3: sort
    .collect(Collectors.toList());           // step 4: collect
```

```python
# 🐍 LCEL — chain operations with | (pipe)
result = (
    prompt                    # step 1: format the input
    | llm                     # step 2: generate response
    | StrOutputParser()       # step 3: extract text
)
```

**Side-by-side comparison:**

```
Java Stream:   data.stream()  .filter()     .map()       .sorted()     .collect()
                    │             │             │             │              │
LCEL:          input           prompt          llm          parser         output
                    │             │             │             │              │
                    └──── each step transforms data and passes to the next ─┘
```

**Detailed mapping:**

| Concept | Java Streams | LCEL (LangChain) |
|---------|-------------|------------------|
| **Core idea** | Chain data transformations | Chain LLM operations |
| **Syntax** | `.method().method()` (dot chaining) | `component \| component` (pipe) |
| **Each step** | Takes input, returns output | Takes input, returns output |
| **Lazy/Eager** | Lazy (runs when `.collect()` called) | Lazy (runs when `.invoke()` called) |
| **Parallel** | `.parallelStream()` | `.batch()` / `RunnableParallel` |
| **Async** | `CompletableFuture` | `.ainvoke()` / `.astream()` |
| **Reusable** | Store stream pipeline in variable | Store chain in variable |
| **Composable** | Chain any number of operations | Chain any number of components |

**The deeper similarity — Functional Programming:**

Both LCEL and Java Streams borrow from **functional programming** concepts:

```
Functional Programming Principle:
  f(g(h(x)))  =  x → h → g → f → result

Java:    stream → filter → map → sort → collect
LCEL:    input  → prompt → llm → parser → result

Both are:
1. A PIPELINE of transformations
2. Each step is a PURE FUNCTION (input → output)
3. Steps are COMPOSABLE (snap together in any order)
4. The pipeline is REUSABLE (define once, run many times)
5. The pipeline is LAZY (nothing runs until you trigger it)
```

**Key difference:** Java Streams transform regular data (strings, numbers, objects). LCEL transforms **AI-specific data** (prompts, LLM calls, embeddings, parsed outputs). But the pattern is exactly the same.

```
So if you understand this Java code:

  List<String> pipeline = items.stream()
      .filter(x -> x.isActive())
      .map(x -> x.getName())
      .collect(toList());
  // Nothing runs until .collect() is called

Then you already understand this LCEL code:

  chain = prompt | llm | parser
  // Nothing runs until .invoke() is called
  result = chain.invoke({"topic": "cats"})
```

#### Declarative vs Imperative — Why It Matters

```python
# IMPERATIVE (without LCEL): You tell the computer EACH STEP
prompt_result = prompt.invoke({"topic": "cats"})
llm_result = llm.invoke(prompt_result)
parsed_result = parser.invoke(llm_result)
return parsed_result
# You manage: error handling, streaming, async, batching — ALL manually

# DECLARATIVE (LCEL): You describe WHAT the pipeline looks like
chain = prompt | llm | parser
result = chain.invoke({"topic": "cats"})
# LangChain manages: error handling, streaming, async, batching — ALL automatically
```

The declarative approach means LangChain can **optimize** things behind the scenes:
- **Streaming**: LangChain knows the full pipeline, so it can stream tokens through every step
- **Batching**: It can parallelize multiple inputs automatically
- **Async**: It can run steps asynchronously without you writing async code
- **Tracing**: It can log every step for debugging (via LangSmith)

You describe the "what", LangChain handles the "how."

#### LCEL is Like Unix Pipes

If you've used Linux/Mac terminal commands, LCEL will feel natural:

```
Unix:    cat file.txt | grep "error" | sort | head -5
LCEL:    prompt | llm | parser
```

Both follow the same philosophy:
- Each command/component does ONE thing well
- Output of one becomes input of the next
- You compose complex behavior from simple building blocks
- Easy to read left-to-right

---

#### LCEL in Action — Code Examples

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Build a chain using | (pipe)
chain = (
    ChatPromptTemplate.from_template("Tell me a joke about {topic}")
    | ChatOpenAI(model="gpt-4")
    | StrOutputParser()
)

# Use it
result = chain.invoke({"topic": "programming"})
print(result)
# "Why do programmers prefer dark mode? Because light attracts bugs!"
```

#### How LCEL Works Under the Hood

```
chain = prompt | llm | parser

When you call chain.invoke({"topic": "cats"}):

Step 1: prompt.invoke({"topic": "cats"})
        → ChatPromptValue([HumanMessage("Tell me a joke about cats")])

Step 2: llm.invoke(ChatPromptValue)
        → AIMessage("Why don't cats play poker? Too many cheetahs!")

Step 3: parser.invoke(AIMessage)
        → "Why don't cats play poker? Too many cheetahs!"

Each component's OUTPUT becomes the next component's INPUT.
```

#### LCEL Features: Streaming, Batch, Async

Every LCEL chain automatically gets these methods for free:

```python
chain = prompt | llm | StrOutputParser()

# 1. invoke — single input, single output
result = chain.invoke({"topic": "cats"})

# 2. stream — get output token by token (for real-time UIs)
for chunk in chain.stream({"topic": "cats"}):
    print(chunk, end="", flush=True)
# "Why" "don't" "cats" "play" "poker?" ...

# 3. batch — process multiple inputs at once (parallel)
results = chain.batch([
    {"topic": "cats"},
    {"topic": "dogs"},
    {"topic": "fish"},
])
# Returns list of 3 jokes

# 4. async versions
result = await chain.ainvoke({"topic": "cats"})
async for chunk in chain.astream({"topic": "cats"}):
    print(chunk, end="")
results = await chain.abatch([{"topic": "cats"}, {"topic": "dogs"}])
```

#### Complex LCEL Chains

```python
from langchain_core.runnables import RunnableParallel, RunnablePassthrough, RunnableLambda

# Parallel execution — run multiple chains simultaneously
chain = RunnableParallel(
    joke=ChatPromptTemplate.from_template("Tell a joke about {topic}") | llm | StrOutputParser(),
    poem=ChatPromptTemplate.from_template("Write a haiku about {topic}") | llm | StrOutputParser(),
    fact=ChatPromptTemplate.from_template("Tell a fun fact about {topic}") | llm | StrOutputParser(),
)

result = chain.invoke({"topic": "cats"})
print(result["joke"])  # A cat joke
print(result["poem"])  # A cat haiku
print(result["fact"])  # A cat fact
# All 3 ran in PARALLEL — faster than sequential!

# Passthrough — forward input unchanged
chain = RunnableParallel(
    context=retriever,                    # fetches relevant docs
    question=RunnablePassthrough()        # passes the question through unchanged
) | prompt | llm | StrOutputParser()

# Lambda — custom function in the chain
def word_count(text: str) -> str:
    return f"{text}\n\n(Word count: {len(text.split())})"

chain = prompt | llm | StrOutputParser() | RunnableLambda(word_count)
```

#### LCEL Chain Composition Visual

```
Simple Chain:
  prompt ──→ llm ──→ parser ──→ result

Parallel Chain:
            ┌── joke_prompt → llm → parser ──┐
  input ──→ ├── poem_prompt → llm → parser ──├──→ {"joke": ..., "poem": ..., "fact": ...}
            └── fact_prompt → llm → parser ──┘

Branching Chain:
                    ┌── retriever ──→ context ──┐
  input ──→ split ──┤                           ├──→ prompt → llm → answer
                    └── passthrough → question ──┘

Sequential Chain:
  input → chain_1 → chain_2 → chain_3 → final_output
          (summarize)  (translate)  (format)
```

---

### 5. Document Loaders

Load data from any source into LangChain's `Document` format.

```python
from langchain_core.documents import Document

# A Document is just: page_content (text) + metadata (dict)
doc = Document(
    page_content="Python is a programming language.",
    metadata={"source": "wiki.txt", "page": 1}
)
```

#### Common Document Loaders

```python
# PDF files
from langchain_community.document_loaders import PyPDFLoader
docs = PyPDFLoader("report.pdf").load()

# Text files
from langchain_community.document_loaders import TextLoader
docs = TextLoader("notes.txt").load()

# Web pages
from langchain_community.document_loaders import WebBaseLoader
docs = WebBaseLoader("https://example.com/article").load()

# CSV files
from langchain_community.document_loaders import CSVLoader
docs = CSVLoader("data.csv").load()

# JSON files
from langchain_community.document_loaders import JSONLoader
docs = JSONLoader("data.json", jq_schema=".[]").load()

# Word documents
from langchain_community.document_loaders import Docx2txtLoader
docs = Docx2txtLoader("document.docx").load()

# YouTube transcripts
from langchain_community.document_loaders import YoutubeLoader
docs = YoutubeLoader.from_youtube_url("https://youtube.com/watch?v=...").load()

# GitHub repos
from langchain_community.document_loaders import GitLoader
docs = GitLoader(repo_path="./my-repo", branch="main").load()

# Notion
from langchain_community.document_loaders import NotionDirectoryLoader
docs = NotionDirectoryLoader("notion_export/").load()

# SQL Database
from langchain_community.document_loaders import SQLDatabaseLoader
# ... and 700+ more loaders
```

#### Load a Directory of Files

```python
from langchain_community.document_loaders import DirectoryLoader

# Load all PDFs from a folder
loader = DirectoryLoader(
    "my_documents/",
    glob="**/*.pdf",          # pattern to match
    loader_cls=PyPDFLoader    # which loader to use
)
docs = loader.load()
print(f"Loaded {len(docs)} pages from all PDFs")
```

---

### 6. Text Splitters (Chunking)

Split documents into smaller chunks for embedding and retrieval.

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

# Most common splitter (recommended default)
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,          # max characters per chunk
    chunk_overlap=50,        # overlap between chunks
    separators=["\n\n", "\n", ". ", " ", ""]
)

# Split loaded documents
chunks = splitter.split_documents(docs)
print(f"Split {len(docs)} documents into {len(chunks)} chunks")

# Or split raw text
texts = splitter.split_text("Your long text here...")
```

```python
# Other splitters available:

# For Markdown files
from langchain.text_splitter import MarkdownHeaderTextSplitter

# For Python code
from langchain.text_splitter import PythonCodeTextSplitter
splitter = PythonCodeTextSplitter(chunk_size=1000)

# For HTML
from langchain.text_splitter import HTMLHeaderTextSplitter

# Token-based (more precise for LLM context limits)
from langchain.text_splitter import TokenTextSplitter
splitter = TokenTextSplitter(chunk_size=200, chunk_overlap=20)

# Semantic (split by meaning change)
from langchain_experimental.text_splitter import SemanticChunker
```

---

### 7. Embeddings

Convert text to vectors using embedding models.

```python
# OpenAI Embeddings
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# Embed a single text
vector = embeddings.embed_query("Hello world")
print(f"Dimensions: {len(vector)}")  # 1536

# Embed multiple texts (for documents)
vectors = embeddings.embed_documents(["Hello", "World", "Python"])
print(f"Embedded {len(vectors)} documents")

# Free alternative: HuggingFace
from langchain_community.embeddings import HuggingFaceEmbeddings
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
```

---

### 8. Vector Stores

Store embeddings and search them.

```python
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma, FAISS

embeddings = OpenAIEmbeddings()

# ── ChromaDB ──
vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# ── FAISS ──
vectorstore = FAISS.from_documents(chunks, embeddings)
vectorstore.save_local("./faiss_index")  # save to disk
vectorstore = FAISS.load_local("./faiss_index", embeddings)  # load later

# ── Pinecone ──
from langchain_pinecone import PineconeVectorStore
vectorstore = PineconeVectorStore.from_documents(
    chunks, embeddings, index_name="my-index"
)

# ── pgvector ──
from langchain_community.vectorstores import PGVector
vectorstore = PGVector.from_documents(
    chunks, embeddings,
    connection_string="postgresql://user:pass@localhost/db",
    collection_name="my_docs"
)

# ── Search (same for ALL vector stores) ──
results = vectorstore.similarity_search("What is Python?", k=3)
for doc in results:
    print(doc.page_content)

# Search with scores
results = vectorstore.similarity_search_with_score("What is Python?", k=3)
for doc, score in results:
    print(f"[{score:.4f}] {doc.page_content}")

# As a retriever (for use in chains)
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})
docs = retriever.invoke("What is Python?")
```

---

### 9. Memory (Conversation History)

Make your LLM remember previous messages in a conversation.

```python
# ── Simple: Message history in a list ──
from langchain_core.chat_history import InMemoryChatMessageHistory

history = InMemoryChatMessageHistory()
history.add_user_message("My name is Rahim")
history.add_ai_message("Hello Rahim! How can I help you?")
history.add_user_message("What is my name?")

print(history.messages)
# [HumanMessage("My name is Rahim"),
#  AIMessage("Hello Rahim!..."),
#  HumanMessage("What is my name?")]

# ── With a chain: auto-manages history ──
from langchain_core.runnables.history import RunnableWithMessageHistory

chain = prompt | llm | StrOutputParser()

# Store for managing multiple conversations
store = {}

def get_session_history(session_id: str):
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]

chain_with_memory = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history",
)

# Each session remembers its own conversation
response1 = chain_with_memory.invoke(
    {"input": "My name is Rahim"},
    config={"configurable": {"session_id": "user_123"}}
)

response2 = chain_with_memory.invoke(
    {"input": "What is my name?"},
    config={"configurable": {"session_id": "user_123"}}
)
# "Your name is Rahim!"  ← It remembered!

response3 = chain_with_memory.invoke(
    {"input": "What is my name?"},
    config={"configurable": {"session_id": "user_456"}}  # different session
)
# "I don't know your name yet."  ← Different session, no memory
```

#### Memory Types

```
┌────────────────────────┬──────────────────────────────────────────┐
│ Type                   │ What it does                             │
├────────────────────────┼──────────────────────────────────────────┤
│ InMemoryChatHistory    │ Stores all messages in RAM               │
│                        │ Simple, lost when app restarts           │
│                        │                                          │
│ RedisChatHistory       │ Stores messages in Redis                 │
│                        │ Persistent, fast, good for production    │
│                        │                                          │
│ SQLChatHistory         │ Stores messages in SQL database          │
│                        │ Persistent, queryable                    │
│                        │                                          │
│ Summary Memory         │ LLM summarizes old messages              │
│                        │ Saves tokens for long conversations      │
│                        │                                          │
│ Window Memory          │ Only keeps last N messages               │
│                        │ Prevents context overflow                │
└────────────────────────┴──────────────────────────────────────────┘
```

---

### 10. Tools (Giving LLMs Abilities)

Tools let the LLM take ACTIONS — search the web, run code, query databases, call APIs.

```python
from langchain_core.tools import tool

# Define a custom tool using the @tool decorator
@tool
def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    # In real app, call a weather API
    weather_data = {"London": "15°C, Rainy", "Tokyo": "25°C, Sunny"}
    return weather_data.get(city, "Weather data not available")

@tool
def calculate(expression: str) -> str:
    """Calculate a math expression. Example: '2 + 3 * 4'"""
    try:
        return str(eval(expression))
    except Exception as e:
        return f"Error: {e}"

@tool
def search_database(query: str) -> str:
    """Search the company database for information."""
    # Simulated database search
    return f"Results for '{query}': Product A ($29.99), Product B ($49.99)"

# Give tools to the LLM
tools = [get_weather, calculate, search_database]
llm_with_tools = llm.bind_tools(tools)

# The LLM now decides WHICH tool to call based on the question
response = llm_with_tools.invoke("What's the weather in Tokyo?")
print(response.tool_calls)
# [{'name': 'get_weather', 'args': {'city': 'Tokyo'}}]
```

#### Built-in Tools

```python
# Web search (using Tavily)
from langchain_community.tools.tavily_search import TavilySearchResults
search = TavilySearchResults(max_results=3)
results = search.invoke("Latest Python news")

# Wikipedia
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
wiki = WikipediaQueryRun(api_wrapper=WikipediaAPIWrapper())

# Python REPL (run code)
from langchain_experimental.tools import PythonREPLTool
python_tool = PythonREPLTool()

# Shell commands
from langchain_community.tools import ShellTool
shell = ShellTool()

# Requests (HTTP calls)
from langchain_community.tools import RequestsGetTool
```

---

### 11. Chains (Combining Everything)

A chain connects multiple components into a pipeline.

#### Simple Chain (LCEL)

```python
# Translation chain
chain = (
    ChatPromptTemplate.from_messages([
        ("system", "Translate the following to {language}"),
        ("human", "{text}")
    ])
    | ChatOpenAI(model="gpt-4")
    | StrOutputParser()
)

result = chain.invoke({"language": "Japanese", "text": "Hello, how are you?"})
```

#### Sequential Chain (Chain of Chains)

```python
# Chain 1: Write a story
story_chain = (
    ChatPromptTemplate.from_template("Write a short story about {topic}")
    | llm
    | StrOutputParser()
)

# Chain 2: Summarize the story
summary_chain = (
    ChatPromptTemplate.from_template("Summarize this story in one sentence:\n{story}")
    | llm
    | StrOutputParser()
)

# Chain 3: Translate the summary
translate_chain = (
    ChatPromptTemplate.from_template("Translate to French:\n{summary}")
    | llm
    | StrOutputParser()
)

# Connect them: story → summary → translation
full_chain = (
    story_chain
    | (lambda story: {"story": story})
    | summary_chain
    | (lambda summary: {"summary": summary})
    | translate_chain
)

result = full_chain.invoke({"topic": "a robot learning to cook"})
# Returns a French translation of a one-sentence summary of a story about a cooking robot
```

#### RAG Chain

```python
from langchain_core.runnables import RunnablePassthrough

# Setup
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

rag_prompt = ChatPromptTemplate.from_messages([
    ("system", "Answer using only the context below. If unsure, say 'I don't know.'\n\nContext:\n{context}"),
    ("human", "{question}")
])

# RAG chain
rag_chain = (
    {
        "context": retriever | format_docs,     # search → format
        "question": RunnablePassthrough()        # pass question through
    }
    | rag_prompt
    | llm
    | StrOutputParser()
)

answer = rag_chain.invoke("What is our refund policy?")
```

---

### 12. Callbacks (Logging, Debugging, Monitoring)

Track what's happening inside your chains.

```python
from langchain_core.callbacks import BaseCallbackHandler

class MyCallback(BaseCallbackHandler):
    def on_llm_start(self, serialized, prompts, **kwargs):
        print(f"🚀 LLM started with prompt: {prompts[0][:50]}...")

    def on_llm_end(self, response, **kwargs):
        print(f"✅ LLM finished. Tokens used: {response.llm_output}")

    def on_chain_start(self, serialized, inputs, **kwargs):
        print(f"⛓️ Chain started with inputs: {inputs}")

    def on_tool_start(self, serialized, input_str, **kwargs):
        print(f"🔧 Tool called: {serialized['name']} with input: {input_str}")

# Use it
chain.invoke({"topic": "cats"}, config={"callbacks": [MyCallback()]})
```

#### LangSmith (Production Monitoring)

```python
# Set environment variables
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "your-langsmith-key"
os.environ["LANGCHAIN_PROJECT"] = "my-project"

# Now ALL chains are automatically traced in the LangSmith dashboard!
# You can see: every step, latency, tokens, cost, errors, etc.
```

---

## Complete LangChain Application Example

Here's a full chatbot with RAG, memory, and tools:

```python
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_core.tools import tool

# ============================================
# 1. Setup LLM and Embeddings
# ============================================
llm = ChatOpenAI(model="gpt-4", temperature=0)
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# ============================================
# 2. Load and index documents
# ============================================
loader = PyPDFLoader("company_handbook.pdf")
docs = loader.load()
chunks = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50).split_documents(docs)
vectorstore = Chroma.from_documents(chunks, embeddings, persist_directory="./db")
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

# ============================================
# 3. Define tools
# ============================================
@tool
def search_documents(query: str) -> str:
    """Search company documents for relevant information."""
    docs = retriever.invoke(query)
    return "\n".join([doc.page_content for doc in docs])

# ============================================
# 4. Create the chatbot chain
# ============================================
prompt = ChatPromptTemplate.from_messages([
    ("system", """You are a helpful company assistant. Use the provided context
to answer questions. If you don't know, say so. Be concise."""),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}"),
])

chain = prompt | llm | StrOutputParser()

# ============================================
# 5. Add memory
# ============================================
store = {}

def get_history(session_id: str):
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]

chatbot = RunnableWithMessageHistory(
    chain,
    get_history,
    input_messages_key="input",
    history_messages_key="history",
)

# ============================================
# 6. Use the chatbot
# ============================================
config = {"configurable": {"session_id": "user_001"}}

print(chatbot.invoke({"input": "What is the vacation policy?"}, config=config))
print(chatbot.invoke({"input": "How many days specifically?"}, config=config))
print(chatbot.invoke({"input": "What did I just ask about?"}, config=config))
# "You asked about the vacation policy and the specific number of days."
```

---

---

## LangChain vs LangGraph — Do I Use One or Both?

This is a very common confusion. The answer: **You use BOTH together. LangGraph is built ON TOP of LangChain.**

```
Think of it like this:

  LangChain = The BRICKS (individual components)
  LangGraph = The ARCHITECTURE (how you arrange those bricks)

  You can't build a house with just bricks (no plan).
  You can't build a house with just a blueprint (no material).
  You need BOTH.
```

### The Relationship

```
┌─────────────────────────────────────────────────────────────┐
│                     YOUR APPLICATION                         │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   LANGGRAPH                          │    │
│  │           (orchestration / workflow)                  │    │
│  │                                                      │    │
│  │   Node A ──→ Node B ──→ Node C                       │    │
│  │     │           │          │                          │    │
│  │     ▼           ▼          ▼                          │    │
│  │  ┌──────┐  ┌──────────┐ ┌───────┐                    │    │
│  │  │LangCh│  │ LangChain│ │LangCh │    LANGCHAIN       │    │
│  │  │prompt │  │ llm +    │ │parser │    (components)    │    │
│  │  │+ llm  │  │ retriever│ │+ tool │                    │    │
│  │  └──────┘  └──────────┘ └───────┘                    │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

LangGraph USES LangChain components INSIDE its nodes.
```

### Real Example: They Work Together

```python
from langchain_openai import ChatOpenAI                    # ← LangChain
from langchain_core.prompts import ChatPromptTemplate      # ← LangChain
from langchain_community.vectorstores import Chroma        # ← LangChain
from langchain_core.tools import tool                      # ← LangChain

from langgraph.graph import StateGraph, START, END         # ← LangGraph

# LangChain provides the COMPONENTS
llm = ChatOpenAI(model="gpt-4")                           # ← LangChain component
retriever = Chroma(...).as_retriever()                     # ← LangChain component
prompt = ChatPromptTemplate.from_template("...")           # ← LangChain component

# LangGraph provides the WORKFLOW
def search_node(state):
    docs = retriever.invoke(state["question"])             # ← uses LangChain inside
    return {"context": docs}

def answer_node(state):
    chain = prompt | llm                                   # ← uses LangChain LCEL inside
    response = chain.invoke({"context": state["context"]})
    return {"answer": response.content}

# LangGraph orchestrates the flow
graph = StateGraph(State)
graph.add_node("search", search_node)                     # node uses LangChain
graph.add_node("answer", answer_node)                     # node uses LangChain
graph.add_edge(START, "search")
graph.add_edge("search", "answer")
graph.add_edge("answer", END)
```

### So When Do You Use What?

```
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│  "I just need to call an LLM with a prompt"                 │
│   → LangChain ONLY (no need for LangGraph)                  │
│     chain = prompt | llm | parser                            │
│                                                              │
│  "I need a RAG pipeline (search docs + answer)"              │
│   → LangChain ONLY (it's a linear pipeline)                  │
│     chain = retriever | format | prompt | llm | parser       │
│                                                              │
│  "I need an agent that decides which tools to use"           │
│   → LangGraph + LangChain                                   │
│     LangGraph = controls the decide→act→observe loop         │
│     LangChain = provides LLM, tools, prompts                │
│                                                              │
│  "I need a multi-step workflow with loops and branches"       │
│   → LangGraph + LangChain                                   │
│     LangGraph = controls the routing, loops, state           │
│     LangChain = provides components inside each node         │
│                                                              │
│  "I need a simple chatbot with memory"                       │
│   → LangChain ONLY (RunnableWithMessageHistory)              │
│                                                              │
│  "I need a complex chatbot that can search, calculate,       │
│   ask clarifying questions, and escalate to humans"          │
│   → LangGraph + LangChain                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Analogy: Restaurant Kitchen

```
LangChain = The INGREDIENTS and KITCHEN TOOLS
  - Knives (LLMs)
  - Pans (Prompts)
  - Spices (Parsers)
  - Fridge (Vector Store)
  - Oven (Embeddings)

LangGraph = The HEAD CHEF's RECIPE / WORKFLOW
  - "First prep the vegetables" (Node 1)
  - "While that's cooking, make the sauce" (Parallel nodes)
  - "Taste it — if too salty, add water and taste again" (Loop)
  - "If customer is VIP, add extra garnish" (Conditional edge)
  - "Plate and serve" (End)

You ALWAYS need ingredients (LangChain).
For a simple dish (boiled egg), you don't need a recipe (no LangGraph).
For a complex dish (5-course meal), you need both ingredients AND a recipe.
```

### Summary

| Question | Answer |
|----------|--------|
| Can I use LangChain without LangGraph? | **Yes** — for simple chains, RAG, chatbots |
| Can I use LangGraph without LangChain? | **Technically yes** but you'd lose all the pre-built components (LLMs, prompts, tools, loaders, vector stores) — not practical |
| Should I use both? | **Yes, when your app needs loops, decisions, or complex multi-step logic** |
| Which do I learn first? | **LangChain first** — it's the foundation. LangGraph builds on top |
| Are they alternatives/competitors? | **No** — LangGraph is part of the LangChain ecosystem. They're designed to work together |

### Installing LangChain gives you LangGraph? NO!

They are **separate packages**. Neither automatically includes the other.

```bash
pip install langchain       # ❌ Does NOT include LangGraph
pip install langgraph       # ❌ Does NOT include full LangChain
pip install langchain langgraph   # ✅ Now you have BOTH
```

Here's how the packages are structured:

```
┌──────────────────────────────────────────────────────────┐
│                  THE LANGCHAIN ECOSYSTEM                   │
│                  (all separate pip packages)               │
│                                                           │
│  ┌─────────────────┐                                      │
│  │ langchain-core   │ ← The BASE. Defines Runnable,       │
│  │                  │   prompts, messages, parsers.        │
│  │  pip install     │   BOTH LangChain and LangGraph      │
│  │  langchain-core  │   depend on this.                    │
│  └────────┬─────────┘                                      │
│           │                                                │
│     ┌─────┴──────────────────┐                             │
│     │                        │                             │
│  ┌──▼──────────────┐   ┌────▼─────────────┐               │
│  │ langchain        │   │ langgraph         │              │
│  │                  │   │                   │              │
│  │ Chains, Memory,  │   │ StateGraph, Nodes │              │
│  │ Agents, LCEL     │   │ Edges, Loops,     │              │
│  │                  │   │ Checkpointing     │              │
│  │ pip install      │   │                   │              │
│  │ langchain        │   │ pip install        │              │
│  └──┬───────────────┘   │ langgraph          │              │
│     │                   └────────────────────┘              │
│     │                                                      │
│  ┌──▼──────────────────────────────────────────┐           │
│  │ langchain-community                          │          │
│  │                                              │          │
│  │ 700+ integrations (document loaders,         │          │
│  │ vector stores, tools, etc.)                  │          │
│  │                                              │          │
│  │ pip install langchain-community              │          │
│  └──────────────────────────────────────────────┘          │
│                                                            │
│  ┌──────────────┐  ┌──────────────────┐                    │
│  │langchain-    │  │langchain-        │                    │
│  │openai        │  │anthropic         │  ... more          │
│  │              │  │                  │  provider           │
│  │pip install   │  │pip install       │  packages           │
│  │langchain-    │  │langchain-        │                    │
│  │openai        │  │anthropic         │                    │
│  └──────────────┘  └──────────────────┘                    │
└────────────────────────────────────────────────────────────┘
```

**What happens when you install each:**

```bash
pip install langchain
# You get: langchain + langchain-core
# You DON'T get: langgraph, langchain-openai, langchain-community

pip install langgraph
# You get: langgraph + langchain-core (as a dependency)
# You DON'T get: langchain, langchain-openai, langchain-community

pip install langchain-openai
# You get: langchain-openai + langchain-core + openai
# You DON'T get: langchain, langgraph
```

**So for a typical project you install:**

```bash
# For a simple LangChain app (no LangGraph):
pip install langchain langchain-openai langchain-community

# For a LangGraph agent app:
pip install langchain langgraph langchain-openai langchain-community

# Minimum for LangGraph (bare bones):
pip install langgraph langchain-openai
# This works because langgraph installs langchain-core automatically,
# and langchain-openai gives you ChatOpenAI.
# But you won't have document loaders, vector stores, etc.
```

**The key insight:** `langchain-core` is the shared foundation. Both LangChain and LangGraph depend on it. But LangChain and LangGraph are separate packages that you install independently based on what you need.

### Can LangGraph Work Without LangChain?

**Short answer:** LangGraph can work without the `langchain` package, but it NEEDS `langchain-core`.

Let's understand the difference between these three things:

```
┌────────────────────────────────────────────────────────────────┐
│                                                                 │
│  langchain-core  =  The FOUNDATION (base classes only)          │
│                     - Runnable interface                        │
│                     - Message types (HumanMessage, AIMessage)   │
│                     - PromptTemplate                            │
│                     - OutputParser                              │
│                     - Tool interface                            │
│                     Size: Small, lightweight                    │
│                                                                 │
│  langchain       =  The FRAMEWORK (built on top of core)        │
│                     - Everything in langchain-core PLUS:        │
│                     - Chains (RetrievalQA, etc.)                │
│                     - Memory (ConversationBufferMemory, etc.)   │
│                     - Agents (legacy agent types)               │
│                     - Text splitters                            │
│                     Size: Medium                                │
│                                                                 │
│  langchain-community = The ECOSYSTEM (700+ integrations)        │
│                     - Document loaders (PDF, web, etc.)         │
│                     - Vector stores (Chroma, FAISS, etc.)       │
│                     - Tools (Wikipedia, search, etc.)           │
│                     Size: Large                                 │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

**LangGraph only requires `langchain-core`.** So technically:

```python
# ✅ This WORKS — LangGraph with ONLY langchain-core (no langchain package)
# pip install langgraph langchain-openai

from langgraph.graph import StateGraph, START, END
from langchain_openai import ChatOpenAI               # from langchain-openai (not langchain)
from langchain_core.messages import HumanMessage       # from langchain-core (not langchain)

class State(TypedDict):
    messages: Annotated[list, add_messages]

llm = ChatOpenAI(model="gpt-4")

def agent(state):
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

graph = StateGraph(State)
graph.add_node("agent", agent)
graph.add_edge(START, "agent")
graph.add_edge("agent", END)
app = graph.compile()

# This works perfectly WITHOUT pip install langchain!
result = app.invoke({"messages": [HumanMessage("Hello!")]})
```

```python
# ❌ This WON'T work without langchain or langchain-community
# These features need the full langchain/langchain-community package:

from langchain_community.vectorstores import Chroma        # needs langchain-community
from langchain_community.document_loaders import PyPDFLoader # needs langchain-community
from langchain.text_splitter import RecursiveCharacterTextSplitter  # needs langchain
from langchain.chains import RetrievalQA                    # needs langchain
```

**But you can also use LangGraph with NO LangChain at all — even without langchain-core:**

```python
# ✅ LangGraph with ZERO LangChain — using plain Python + raw OpenAI
# pip install langgraph openai

from langgraph.graph import StateGraph, START, END
from typing import TypedDict
import openai  # raw OpenAI, not langchain-openai

client = openai.OpenAI()

class State(TypedDict):
    question: str
    answer: str

def think(state: State) -> dict:
    # Using raw OpenAI API — no LangChain at all!
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": state["question"]}]
    )
    return {"answer": response.choices[0].message.content}

graph = StateGraph(State)
graph.add_node("think", think)
graph.add_edge(START, "think")
graph.add_edge("think", END)
app = graph.compile()

# Works! No LangChain anywhere!
result = app.invoke({"question": "What is Python?"})
print(result["answer"])
```

### So What EXACTLY Does Each Package Need?

```
LangGraph nodes are just Python functions.
Inside those functions, you can use ANYTHING:
  - LangChain components (most common, most convenient)
  - Raw OpenAI SDK
  - Raw HTTP requests
  - Any Python library
  - Your own custom code

LangGraph doesn't care what's inside the nodes.
It only cares about: State in → State out.
```

```
                    ┌──────────────────────────────┐
                    │     WHAT LANGGRAPH NEEDS      │
                    │                               │
                    │  ✅ langgraph (the package)    │
                    │  ✅ Python functions as nodes   │
                    │  ✅ A State definition          │
                    │                               │
                    │  That's it. Everything else    │
                    │  is optional.                  │
                    └──────────────────────────────┘

But in PRACTICE, most people use LangChain components inside
LangGraph nodes because:
  - ChatOpenAI is easier than raw openai SDK
  - Prompts, parsers, tools are pre-built
  - Switching providers is one line change
  - You get streaming/batch for free
```

### Final Answer

| Statement | True or False? |
|-----------|---------------|
| "LangGraph REQUIRES the `langchain` package" | **False** — it only needs `langchain-core` (auto-installed) |
| "LangGraph REQUIRES `langchain-core`" | **True** — it's a dependency, installed automatically |
| "LangGraph can work with raw Python + raw OpenAI" | **True** — nodes are just functions, use anything inside |
| "In practice, everyone uses LangChain with LangGraph" | **Mostly true** — because LangChain components are convenient |
| "You MUST install them separately" | **True** — `pip install langchain` does NOT give you `langgraph` |

---

# Part 2: LangGraph

---

## What is LangGraph?

LangGraph is a framework for building **stateful, multi-step AI agent workflows** using a **graph** (nodes + edges) structure.

### Why Do We Need LangGraph? What's Wrong with LangChain Chains?

LangChain chains are **linear** — they go from A to B to C in a straight line:

```
LangChain Chain (Linear):
  Input → Step A → Step B → Step C → Output

  - Always follows the same path
  - No decisions, no branching
  - No loops (can't retry or go back)
  - Simple but limited
```

Real-world AI agents need to:
- **Make decisions** (if X, do this; else do that)
- **Loop** (try again if the answer isn't good enough)
- **Use tools** (search, calculate, call APIs) and process results
- **Maintain state** across multiple steps
- **Handle errors** and recover

```
LangGraph Workflow (Graph):

                    ┌──────────┐
                    │  START   │
                    └────┬─────┘
                         │
                    ┌────▼─────┐
              ┌─────│ Classify │─────┐
              │     │ Question │     │
              │     └──────────┘     │
         "simple"              "complex"
              │                      │
         ┌────▼─────┐         ┌─────▼────┐
         │ Direct   │         │ Search   │
         │ Answer   │         │ Tools    │
         └────┬─────┘         └─────┬────┘
              │                     │
              │               ┌─────▼────┐     "not good"
              │               │ Evaluate │──────────┐
              │               │ Answer   │          │
              │               └─────┬────┘          │
              │                "good"│         ┌────▼─────┐
              │                     │         │ Retry    │
              │                     │         │ Search   │──── loops back
              │                     │         └──────────┘
              │                     │
              └────────┬────────────┘
                  ┌────▼─────┐
                  │   END    │
                  └──────────┘
```

### LangChain vs LangGraph — When to Use Which

| Feature | LangChain (Chains) | LangGraph |
|---------|-------------------|-----------|
| **Flow** | Linear (A → B → C) | Any direction (loops, branches) |
| **Decisions** | No (always same path) | Yes (conditional routing) |
| **Loops** | No | Yes (retry, iterate) |
| **State** | Passed through pipe | Explicit state object |
| **Tools** | Basic tool calling | Full tool loop with feedback |
| **Use case** | Simple pipelines | Complex agents, multi-step reasoning |
| **Complexity** | Low | Medium-High |

**Rule of thumb:**
- Use **LangChain** for: simple chains, RAG pipelines, basic chatbots
- Use **LangGraph** for: agents that think/act/observe, multi-step workflows, anything with loops or decisions

---

## LangGraph Core Concepts

### The 3 Building Blocks

```
┌─────────────────────────────────────────┐
│              LANGGRAPH                   │
│                                          │
│  1. STATE  = The shared data/memory      │
│              (a TypedDict or Pydantic)   │
│                                          │
│  2. NODES  = The workers/functions       │
│              (each does one task)        │
│                                          │
│  3. EDGES  = The connections/routes      │
│              (how nodes connect)         │
│                                          │
│  Together they form a GRAPH              │
└─────────────────────────────────────────┘
```

Let me explain each one:

---

### 1. State

State is a **shared data container** that flows through the graph. Every node can read and write to it.

```python
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

# Define what data your graph tracks
class State(TypedDict):
    messages: Annotated[list, add_messages]   # conversation history
    question: str                              # the user's question
    context: str                               # retrieved documents
    answer: str                                # the final answer
    attempt_count: int                         # how many retries
```

```
Think of State like a FORM that gets filled out as it moves through the graph:

Start:   { messages: [],  question: "What is RAG?", context: "", answer: "", attempt_count: 0 }
           │
           ▼ (retrieval node fills in context)
Step 1:  { messages: [],  question: "What is RAG?", context: "RAG is...", answer: "", attempt_count: 0 }
           │
           ▼ (generation node fills in answer)
Step 2:  { messages: [],  question: "What is RAG?", context: "RAG is...", answer: "RAG stands for...", attempt_count: 1 }
           │
           ▼ (done!)
```

**`Annotated[list, add_messages]`** is special — it tells LangGraph to **append** new messages instead of replacing the list. This is how conversation history accumulates.

---

### 2. Nodes

Nodes are **Python functions** that take the current state, do something, and return updated state.

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

# Each node is just a function: State in → partial State out
def retrieve(state: State) -> dict:
    """Search for relevant documents."""
    question = state["question"]
    docs = retriever.invoke(question)
    context = "\n".join([doc.page_content for doc in docs])
    return {"context": context}    # only return what changed

def generate(state: State) -> dict:
    """Generate an answer using the context."""
    prompt = f"Context: {state['context']}\n\nQuestion: {state['question']}\nAnswer:"
    response = llm.invoke(prompt)
    return {
        "answer": response.content,
        "attempt_count": state["attempt_count"] + 1
    }

def evaluate(state: State) -> dict:
    """Check if the answer is good enough."""
    prompt = f"Is this a good answer? Answer YES or NO.\nQ: {state['question']}\nA: {state['answer']}"
    response = llm.invoke(prompt)
    return {"evaluation": response.content}
```

```
Node Rules:
- Input:  receives the FULL state
- Output: returns a PARTIAL dict (only the fields that changed)
- LangGraph MERGES the output into the existing state
```

---

### 3. Edges

Edges define **how nodes connect** — the flow of your graph.

```python
from langgraph.graph import StateGraph, START, END

# Create the graph
graph = StateGraph(State)

# Add nodes
graph.add_node("retrieve", retrieve)
graph.add_node("generate", generate)
graph.add_node("evaluate", evaluate)

# Add edges (connections between nodes)

# a) Normal edge — always goes from A to B
graph.add_edge(START, "retrieve")        # Start → retrieve
graph.add_edge("retrieve", "generate")   # retrieve → generate
graph.add_edge("generate", "evaluate")   # generate → evaluate

# b) Conditional edge — decides where to go based on state
def should_retry(state: State) -> str:
    """Decide: retry or finish?"""
    if "NO" in state.get("evaluation", "") and state["attempt_count"] < 3:
        return "retrieve"      # try again
    else:
        return END             # we're done

graph.add_conditional_edges(
    "evaluate",                 # from this node
    should_retry,               # use this function to decide
    {
        "retrieve": "retrieve", # if function returns "retrieve" → go to retrieve node
        END: END                # if function returns END → finish
    }
)

# Compile the graph
app = graph.compile()

# Run it!
result = app.invoke({
    "question": "What is RAG?",
    "messages": [],
    "context": "",
    "answer": "",
    "attempt_count": 0
})

print(result["answer"])
```

#### Edge Types Visual

```
1. Normal Edge (always follows this path):
   A ──────────────→ B

2. Conditional Edge (decides based on state):
                    ┌──→ B  (if condition_1)
   A ── function ──┤
                    └──→ C  (if condition_2)

3. Entry Edge (where the graph starts):
   START ──→ A

4. End Edge (where the graph finishes):
   A ──→ END
```

---

## Building Your First LangGraph Agent (Step by Step)

Let's build an agent that can search the web and answer questions:

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_core.tools import tool

# ============================================
# STEP 1: Define the State
# ============================================
class AgentState(TypedDict):
    messages: Annotated[list, add_messages]

# ============================================
# STEP 2: Define Tools
# ============================================
@tool
def search_web(query: str) -> str:
    """Search the web for information."""
    # Simulated web search
    results = {
        "weather tokyo": "Tokyo: 25°C, Sunny",
        "python creator": "Python was created by Guido van Rossum",
        "langchain": "LangChain is a framework for building LLM applications",
    }
    for key, value in results.items():
        if key in query.lower():
            return value
    return f"No results found for: {query}"

@tool
def calculator(expression: str) -> str:
    """Calculate a math expression."""
    return str(eval(expression))

tools = [search_web, calculator]

# ============================================
# STEP 3: Create LLM with Tools
# ============================================
llm = ChatOpenAI(model="gpt-4", temperature=0)
llm_with_tools = llm.bind_tools(tools)

# ============================================
# STEP 4: Define Nodes
# ============================================
def agent(state: AgentState) -> dict:
    """The 'brain' — decides what to do next."""
    messages = state["messages"]
    response = llm_with_tools.invoke(messages)
    return {"messages": [response]}

def tool_executor(state: AgentState) -> dict:
    """Execute the tool the agent chose."""
    last_message = state["messages"][-1]

    results = []
    for tool_call in last_message.tool_calls:
        # Find and execute the right tool
        tool_map = {t.name: t for t in tools}
        tool_to_call = tool_map[tool_call["name"]]
        result = tool_to_call.invoke(tool_call["args"])

        # Return result as a ToolMessage
        from langchain_core.messages import ToolMessage
        results.append(
            ToolMessage(content=str(result), tool_call_id=tool_call["id"])
        )

    return {"messages": results}

# ============================================
# STEP 5: Define Routing Logic
# ============================================
def should_use_tools(state: AgentState) -> str:
    """Check if the agent wants to use a tool or is done."""
    last_message = state["messages"][-1]

    if last_message.tool_calls:
        return "tools"     # agent wants to use a tool
    else:
        return END         # agent is done, has a final answer

# ============================================
# STEP 6: Build the Graph
# ============================================
graph = StateGraph(AgentState)

# Add nodes
graph.add_node("agent", agent)
graph.add_node("tools", tool_executor)

# Add edges
graph.add_edge(START, "agent")                  # start with the agent
graph.add_conditional_edges(
    "agent",
    should_use_tools,
    {"tools": "tools", END: END}
)
graph.add_edge("tools", "agent")                # after tool → back to agent

# Compile
app = graph.compile()

# ============================================
# STEP 7: Run the Agent!
# ============================================
result = app.invoke({
    "messages": [HumanMessage(content="What's the weather in Tokyo?")]
})

# Print the conversation
for msg in result["messages"]:
    role = type(msg).__name__
    print(f"{role}: {msg.content}")
```

**Output:**
```
HumanMessage: What's the weather in Tokyo?
AIMessage:                                     ← (empty, has tool_calls)
ToolMessage: Tokyo: 25°C, Sunny
AIMessage: The weather in Tokyo is 25°C and Sunny!
```

### The Agent Loop Visualized

```
┌─────────────────────────────────────────────────┐
│                  AGENT LOOP                      │
│                                                   │
│   ┌─────────┐    has tool calls?   ┌──────────┐  │
│   │         │─── YES ─────────────→│          │  │
│   │  Agent  │                      │  Tools   │  │
│   │ (LLM)  │←────────────────────── │ (execute)│  │
│   │         │                      │          │  │
│   └────┬────┘                      └──────────┘  │
│        │                                          │
│   NO tool calls                                   │
│   (has final answer)                              │
│        │                                          │
│   ┌────▼────┐                                     │
│   │   END   │                                     │
│   └─────────┘                                     │
│                                                   │
│   This loop continues until the LLM decides       │
│   it has enough info to answer without tools.     │
└─────────────────────────────────────────────────┘
```

---

## LangGraph Pre-built Agent (Shortcut)

For the common "LLM + tools" pattern, LangGraph has a shortcut:

```python
from langgraph.prebuilt import create_react_agent
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool

llm = ChatOpenAI(model="gpt-4")

@tool
def search(query: str) -> str:
    """Search for information."""
    return f"Results for {query}: ..."

@tool
def calculate(expr: str) -> str:
    """Calculate math."""
    return str(eval(expr))

# One line — creates the entire agent graph!
agent = create_react_agent(
    model=llm,
    tools=[search, calculate],
)

# Use it
result = agent.invoke({
    "messages": [("user", "What is 25 * 4 + 10?")]
})
print(result["messages"][-1].content)
# "25 * 4 + 10 = 110"
```

This creates the exact same graph we built manually, but in one line.

---

## Advanced LangGraph Patterns

### 1. Human-in-the-Loop (Ask User for Approval)

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    messages: Annotated[list, add_messages]
    approved: bool

def plan_action(state: State) -> dict:
    """Agent plans what to do."""
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

def execute_action(state: State) -> dict:
    """Execute the approved action."""
    return {"messages": [AIMessage(content="Action executed successfully!")]}

def check_approval(state: State) -> str:
    if state.get("approved", False):
        return "execute"
    return END  # wait for human approval

graph = StateGraph(State)
graph.add_node("plan", plan_action)
graph.add_node("execute", execute_action)
graph.add_edge(START, "plan")
graph.add_conditional_edges("plan", check_approval, {"execute": "execute", END: END})
graph.add_edge("execute", END)

# Add checkpointing (saves state so we can resume later)
memory = MemorySaver()
app = graph.compile(checkpointer=memory, interrupt_before=["execute"])

# Run — it will STOP before "execute" and wait for approval
config = {"configurable": {"thread_id": "1"}}
result = app.invoke({"messages": [HumanMessage("Delete all old files")], "approved": False}, config)

# User reviews... then approves:
app.update_state(config, {"approved": True})
result = app.invoke(None, config)  # resume from where it stopped
```

### 2. Parallel Execution (Fan-out / Fan-in)

Run multiple nodes at the same time:

```python
class ResearchState(TypedDict):
    topic: str
    web_results: str
    wiki_results: str
    paper_results: str
    final_summary: str

def search_web(state: ResearchState) -> dict:
    return {"web_results": f"Web results for {state['topic']}..."}

def search_wiki(state: ResearchState) -> dict:
    return {"wiki_results": f"Wikipedia results for {state['topic']}..."}

def search_papers(state: ResearchState) -> dict:
    return {"paper_results": f"Paper results for {state['topic']}..."}

def summarize(state: ResearchState) -> dict:
    """Combine all results into a summary."""
    combined = f"""
    Web: {state['web_results']}
    Wiki: {state['wiki_results']}
    Papers: {state['paper_results']}
    """
    response = llm.invoke(f"Summarize these research results:\n{combined}")
    return {"final_summary": response.content}

graph = StateGraph(ResearchState)
graph.add_node("search_web", search_web)
graph.add_node("search_wiki", search_wiki)
graph.add_node("search_papers", search_papers)
graph.add_node("summarize", summarize)

# Fan-out: START → all 3 searches in PARALLEL
graph.add_edge(START, "search_web")
graph.add_edge(START, "search_wiki")
graph.add_edge(START, "search_papers")

# Fan-in: all 3 searches → summarize
graph.add_edge("search_web", "summarize")
graph.add_edge("search_wiki", "summarize")
graph.add_edge("search_papers", "summarize")
graph.add_edge("summarize", END)

app = graph.compile()
```

```
         ┌── search_web ──┐
         │                 │
START ───┼── search_wiki ──┼──→ summarize → END
         │                 │
         └── search_papers─┘

All 3 searches run in PARALLEL, then results merge into summarize.
```

### 3. Subgraphs (Graph Inside a Graph)

Break complex workflows into smaller, reusable graphs:

```python
# ── Inner graph: handles research ──
class ResearchState(TypedDict):
    query: str
    results: str

def search(state: ResearchState) -> dict:
    return {"results": f"Found info about {state['query']}"}

research_graph = StateGraph(ResearchState)
research_graph.add_node("search", search)
research_graph.add_edge(START, "search")
research_graph.add_edge("search", END)
research_subgraph = research_graph.compile()

# ── Outer graph: uses the research subgraph ──
class MainState(TypedDict):
    question: str
    research_results: str
    answer: str

def do_research(state: MainState) -> dict:
    result = research_subgraph.invoke({"query": state["question"]})
    return {"research_results": result["results"]}

def answer(state: MainState) -> dict:
    response = llm.invoke(
        f"Based on: {state['research_results']}\nAnswer: {state['question']}"
    )
    return {"answer": response.content}

main_graph = StateGraph(MainState)
main_graph.add_node("research", do_research)
main_graph.add_node("answer", answer)
main_graph.add_edge(START, "research")
main_graph.add_edge("research", "answer")
main_graph.add_edge("answer", END)
app = main_graph.compile()
```

### 4. Persistence & Checkpointing (Resume from Where You Left Off)

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.checkpoint.sqlite import SqliteSaver

# In-memory checkpointing (for development)
memory = MemorySaver()

# SQLite checkpointing (for production — persists to disk)
db_saver = SqliteSaver.from_conn_string("checkpoints.db")

# Compile with checkpointer
app = graph.compile(checkpointer=memory)

# Every invocation with same thread_id continues the conversation
config = {"configurable": {"thread_id": "user_123"}}

result1 = app.invoke({"messages": [HumanMessage("My name is Rahim")]}, config)
result2 = app.invoke({"messages": [HumanMessage("What is my name?")]}, config)
# "Your name is Rahim!" ← remembers across invocations!

# Different thread = different conversation
config2 = {"configurable": {"thread_id": "user_456"}}
result3 = app.invoke({"messages": [HumanMessage("What is my name?")]}, config2)
# "I don't know your name" ← separate conversation
```

### 5. Streaming (Real-Time Output)

```python
# Stream node-by-node updates
for event in app.stream({"messages": [HumanMessage("Search for Python info")]}):
    for node_name, output in event.items():
        print(f"--- Node: {node_name} ---")
        print(output)

# Stream token-by-token from LLM
async for event in app.astream_events(
    {"messages": [HumanMessage("Tell me about Python")]},
    version="v2"
):
    if event["event"] == "on_chat_model_stream":
        print(event["data"]["chunk"].content, end="")
```

---

## Real-World LangGraph Example: Customer Support Agent

A multi-step agent that handles customer support:

```python
from typing import TypedDict, Annotated, Literal
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage

# ============================================
# State
# ============================================
class SupportState(TypedDict):
    messages: Annotated[list, add_messages]
    category: str          # billing, technical, general
    sentiment: str         # positive, negative, neutral
    escalated: bool        # needs human agent?
    resolved: bool

# ============================================
# Nodes
# ============================================
llm = ChatOpenAI(model="gpt-4", temperature=0)

def classify_intent(state: SupportState) -> dict:
    """Classify the customer's intent and sentiment."""
    last_msg = state["messages"][-1].content

    response = llm.invoke([
        SystemMessage(content="""Classify this customer message.
        Return JSON: {"category": "billing|technical|general", "sentiment": "positive|negative|neutral"}"""),
        HumanMessage(content=last_msg)
    ])

    import json
    classification = json.loads(response.content)
    return {
        "category": classification["category"],
        "sentiment": classification["sentiment"]
    }

def handle_billing(state: SupportState) -> dict:
    """Handle billing-related questions."""
    response = llm.invoke([
        SystemMessage(content="""You are a billing support agent.
        Available actions: check balance, process refund, update payment method.
        Be helpful and professional."""),
        *state["messages"]
    ])
    return {"messages": [response], "resolved": True}

def handle_technical(state: SupportState) -> dict:
    """Handle technical support questions."""
    response = llm.invoke([
        SystemMessage(content="""You are a technical support agent.
        Help troubleshoot issues step by step.
        If the issue is complex, recommend escalation."""),
        *state["messages"]
    ])

    needs_escalation = "escalat" in response.content.lower()
    return {
        "messages": [response],
        "escalated": needs_escalation,
        "resolved": not needs_escalation
    }

def handle_general(state: SupportState) -> dict:
    """Handle general questions."""
    response = llm.invoke([
        SystemMessage(content="You are a friendly customer support agent. Answer general questions."),
        *state["messages"]
    ])
    return {"messages": [response], "resolved": True}

def escalate_to_human(state: SupportState) -> dict:
    """Escalate to a human agent."""
    return {
        "messages": [AIMessage(content="I'm connecting you with a human specialist who can help further. Please hold on.")],
        "escalated": True
    }

def check_satisfaction(state: SupportState) -> dict:
    """Ask if the customer is satisfied."""
    return {
        "messages": [AIMessage(content="Is there anything else I can help you with?")]
    }

# ============================================
# Routing Functions
# ============================================
def route_by_category(state: SupportState) -> str:
    if state["sentiment"] == "negative" and state["category"] == "billing":
        return "escalate"  # angry billing → go straight to human
    return state["category"]

def check_if_escalated(state: SupportState) -> str:
    if state.get("escalated", False):
        return "escalate"
    return "satisfaction"

# ============================================
# Build the Graph
# ============================================
graph = StateGraph(SupportState)

# Add nodes
graph.add_node("classify", classify_intent)
graph.add_node("billing", handle_billing)
graph.add_node("technical", handle_technical)
graph.add_node("general", handle_general)
graph.add_node("escalate", escalate_to_human)
graph.add_node("satisfaction", check_satisfaction)

# Add edges
graph.add_edge(START, "classify")

graph.add_conditional_edges(
    "classify",
    route_by_category,
    {
        "billing": "billing",
        "technical": "technical",
        "general": "general",
        "escalate": "escalate",
    }
)

graph.add_conditional_edges(
    "technical",
    check_if_escalated,
    {"escalate": "escalate", "satisfaction": "satisfaction"}
)

graph.add_edge("billing", "satisfaction")
graph.add_edge("general", "satisfaction")
graph.add_edge("escalate", END)
graph.add_edge("satisfaction", END)

# Compile
support_agent = graph.compile()

# ============================================
# Test it!
# ============================================
result = support_agent.invoke({
    "messages": [HumanMessage(content="I can't log into my account, I keep getting error 503")],
    "category": "",
    "sentiment": "",
    "escalated": False,
    "resolved": False,
})

for msg in result["messages"]:
    role = "Customer" if isinstance(msg, HumanMessage) else "Agent"
    print(f"{role}: {msg.content}\n")
```

```
The support agent graph:

                    ┌──────────┐
                    │ Classify  │
                    │ Intent    │
                    └─────┬────┘
                          │
            ┌─────────────┼──────────────┐
            │             │              │
       ┌────▼────┐  ┌────▼─────┐  ┌────▼─────┐
       │ Billing │  │Technical │  │ General  │
       └────┬────┘  └────┬─────┘  └────┬─────┘
            │             │              │
            │        escalated?          │
            │        ┌────┴────┐         │
            │       YES       NO         │
            │        │         │         │
            │   ┌────▼────┐    │         │
            │   │Escalate │    │         │
            │   │to Human │    │         │
            │   └────┬────┘    │         │
            │        │    ┌────▼─────┐   │
            │        │    │Satisfac- │◄──┘
            │        │    │tion Check│◄──────────┘
            │        │    └────┬─────┘
            │        │         │
            ▼        ▼         ▼
                    END
```

---

## LangGraph RAG Agent (RAG + Tools + Reasoning)

Combining RAG with agent capabilities:

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage
from langchain_core.tools import tool

# ============================================
# Setup
# ============================================
llm = ChatOpenAI(model="gpt-4", temperature=0)
embeddings = OpenAIEmbeddings()
vectorstore = Chroma(persist_directory="./db", embedding_function=embeddings)
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

# ============================================
# State
# ============================================
class RAGState(TypedDict):
    messages: Annotated[list, add_messages]
    question: str
    context: list[str]
    answer: str
    needs_more_info: bool
    search_queries: list[str]

# ============================================
# Nodes
# ============================================
def analyze_question(state: RAGState) -> dict:
    """Analyze the question and generate search queries."""
    question = state["messages"][-1].content

    response = llm.invoke([
        SystemMessage(content="""Given a question, generate 1-3 search queries to find relevant information.
        Return as JSON: {"queries": ["query1", "query2"]}"""),
        HumanMessage(content=question)
    ])

    import json
    queries = json.loads(response.content)["queries"]
    return {"question": question, "search_queries": queries}

def retrieve_documents(state: RAGState) -> dict:
    """Search the vector store with multiple queries."""
    all_contexts = []
    for query in state["search_queries"]:
        docs = retriever.invoke(query)
        for doc in docs:
            if doc.page_content not in all_contexts:
                all_contexts.append(doc.page_content)
    return {"context": all_contexts}

def generate_answer(state: RAGState) -> dict:
    """Generate an answer from the retrieved context."""
    context_str = "\n\n".join(state["context"])

    response = llm.invoke([
        SystemMessage(content="""Answer the question using ONLY the provided context.
        If the context doesn't contain enough info, set needs_more_info to true.
        Return JSON: {"answer": "your answer", "needs_more_info": false}"""),
        HumanMessage(content=f"Context:\n{context_str}\n\nQuestion: {state['question']}")
    ])

    import json
    result = json.loads(response.content)
    return {
        "answer": result["answer"],
        "needs_more_info": result["needs_more_info"],
        "messages": [AIMessage(content=result["answer"])]
    }

def refine_search(state: RAGState) -> dict:
    """Generate better search queries based on what was missing."""
    response = llm.invoke([
        SystemMessage(content="The previous search didn't find enough info. Generate 2 NEW, different search queries."),
        HumanMessage(content=f"Original question: {state['question']}\nPrevious queries: {state['search_queries']}")
    ])

    import json
    new_queries = json.loads(response.content)["queries"]
    return {"search_queries": new_queries}

# ============================================
# Routing
# ============================================
def check_answer_quality(state: RAGState) -> str:
    if state.get("needs_more_info", False):
        return "refine"
    return "done"

# ============================================
# Build Graph
# ============================================
graph = StateGraph(RAGState)

graph.add_node("analyze", analyze_question)
graph.add_node("retrieve", retrieve_documents)
graph.add_node("generate", generate_answer)
graph.add_node("refine", refine_search)

graph.add_edge(START, "analyze")
graph.add_edge("analyze", "retrieve")
graph.add_edge("retrieve", "generate")
graph.add_conditional_edges(
    "generate",
    check_answer_quality,
    {"refine": "refine", "done": END}
)
graph.add_edge("refine", "retrieve")   # loop back!

rag_agent = graph.compile()
```

```
RAG Agent Flow:

START → analyze → retrieve → generate ──→ END
            ▲                    │       (answer is good)
            │                    │
            │                    ▼ (needs more info)
            │               refine
            │                    │
            └────────────────────┘  (loop: try different search queries)
```

---

## Visualizing Your Graph

```python
# Generate a visual diagram of your graph
from IPython.display import Image, display

# As PNG
display(Image(app.get_graph().draw_mermaid_png()))

# As Mermaid text (paste into mermaid.live)
print(app.get_graph().draw_mermaid())
```

---

## LangGraph vs Other Agent Frameworks

| Feature | LangGraph | AutoGen | CrewAI | Raw LangChain Agents |
|---------|-----------|---------|--------|---------------------|
| **Graph-based** | Yes | No | No | No |
| **Loops** | Yes | Yes | Limited | Limited |
| **State management** | Explicit | Implicit | Implicit | Implicit |
| **Checkpointing** | Built-in | No | No | No |
| **Human-in-loop** | Built-in | Basic | No | No |
| **Streaming** | Built-in | Limited | Limited | Basic |
| **Multi-agent** | Yes (subgraphs) | Yes (conversations) | Yes (crews) | No |
| **Debugging** | LangSmith | Limited | Limited | LangSmith |
| **Learning curve** | Medium | Medium | Low | Low |

---

## Quick Reference Cheat Sheet

### LangChain

```python
# LLM
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-4")
response = llm.invoke("Hello")

# Prompt Template
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate.from_template("Tell me about {topic}")

# LCEL Chain
chain = prompt | llm | StrOutputParser()
result = chain.invoke({"topic": "Python"})

# Streaming
for chunk in chain.stream({"topic": "Python"}):
    print(chunk, end="")

# Batch
results = chain.batch([{"topic": "Python"}, {"topic": "Java"}])

# RAG
from langchain_community.vectorstores import Chroma
vectorstore = Chroma.from_documents(chunks, embeddings)
retriever = vectorstore.as_retriever()
rag_chain = {"context": retriever, "question": RunnablePassthrough()} | prompt | llm

# Tools
from langchain_core.tools import tool
@tool
def my_tool(x: str) -> str:
    """Description."""
    return result
```

### LangGraph

```python
# State
class MyState(TypedDict):
    messages: Annotated[list, add_messages]
    data: str

# Node (function)
def my_node(state: MyState) -> dict:
    return {"data": "updated"}

# Graph
from langgraph.graph import StateGraph, START, END
graph = StateGraph(MyState)
graph.add_node("node_name", my_node)
graph.add_edge(START, "node_name")
graph.add_edge("node_name", END)

# Conditional edge
def router(state) -> str:
    return "path_a" if condition else "path_b"
graph.add_conditional_edges("node", router, {"path_a": "a", "path_b": "b"})

# Compile & run
app = graph.compile()
result = app.invoke({"messages": [...], "data": ""})

# With memory/checkpointing
from langgraph.checkpoint.memory import MemorySaver
app = graph.compile(checkpointer=MemorySaver())
result = app.invoke(input, config={"configurable": {"thread_id": "123"}})

# Pre-built agent
from langgraph.prebuilt import create_react_agent
agent = create_react_agent(model=llm, tools=[tool1, tool2])

# Streaming
for event in app.stream(input):
    print(event)
```

---

## Summary

```
LANGCHAIN = Building blocks for LLM apps
├── Chat Models      → Talk to any LLM (OpenAI, Claude, Gemini)
├── Prompts          → Reusable prompt templates
├── Output Parsers   → Convert LLM text → structured data (JSON, lists, objects)
├── LCEL (pipes)     → Chain components: prompt | llm | parser
├── Document Loaders → Load PDFs, web pages, CSVs, etc.
├── Text Splitters   → Chunk documents for RAG
├── Embeddings       → Convert text to vectors
├── Vector Stores    → Store & search vectors (Chroma, FAISS, pgvector)
├── Memory           → Conversation history across messages
├── Tools            → Give LLMs abilities (search, calculate, API calls)
├── Chains           → Connect everything into pipelines
└── Callbacks        → Logging, monitoring, debugging

LANGGRAPH = Stateful agent workflows
├── State            → Shared data container (TypedDict)
├── Nodes            → Worker functions (each does one task)
├── Edges            → Connections (normal, conditional)
├── Loops            → Retry, iterate, refine
├── Checkpointing    → Save/resume state
├── Human-in-Loop    → Pause for human approval
├── Streaming        → Real-time output
├── Subgraphs        → Nested graphs for complex workflows
└── Prebuilt Agents  → create_react_agent() for quick setup

WHEN TO USE WHAT:
├── Simple Q&A            → LangChain (prompt | llm | parser)
├── RAG pipeline          → LangChain (loader → splitter → vectorstore → retriever → chain)
├── Chatbot with memory   → LangChain (RunnableWithMessageHistory)
├── Tool-using agent      → LangGraph (agent + tool loop)
├── Multi-step workflow   → LangGraph (nodes + conditional edges)
├── Agent with retries    → LangGraph (loops)
└── Complex orchestration → LangGraph (subgraphs, parallel, human-in-loop)
```
