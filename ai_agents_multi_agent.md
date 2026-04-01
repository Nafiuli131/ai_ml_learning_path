# AI Agents & Multi-Agent Systems — Complete Guide

## What is an AI Agent?

An AI Agent is an **LLM that can take actions** — not just generate text, but actually DO things in the real world (search the web, read files, call APIs, write code, send emails).

### Regular LLM vs Agent

```
REGULAR LLM (ChatGPT without tools):
  User: "What's the weather in Tokyo?"
  LLM:  "I don't have access to real-time weather data."
        (can only use training knowledge — STUCK)

AI AGENT (LLM + Tools + Reasoning Loop):
  User: "What's the weather in Tokyo?"
  Agent: *thinks* "I need to check the weather. I have a weather tool."
         *acts*   calls weather_api("Tokyo")
         *observes* result: "25°C, Sunny"
         *responds* "The weather in Tokyo is 25°C and Sunny!"
        (actually DID something — USEFUL)
```

### The Key Difference

```
LLM      = A brain in a jar. Can think, but can't do anything.
AI Agent = A brain with hands, eyes, and legs. Can think AND act.

┌─────────────┐     ┌─────────────────────────────────────┐
│  Regular LLM│     │          AI AGENT                    │
│             │     │                                      │
│  Input →    │     │  Input →  Brain (LLM)                │
│  Brain →    │     │           ↓                          │
│  Output     │     │           Decides what to do         │
│             │     │           ↓                          │
│  (text only)│     │           Uses Tools (search, code,  │
│             │     │           APIs, databases)            │
│             │     │           ↓                          │
│             │     │           Observes results            │
│             │     │           ↓                          │
│             │     │           Thinks again... (loop)      │
│             │     │           ↓                          │
│             │     │           Final Output                │
└─────────────┘     └─────────────────────────────────────┘
```

---

## How AI Agents Work — The Agent Loop

Every AI agent follows the same fundamental loop: **Think → Act → Observe → Repeat**

```
┌──────────────────────────────────────────────────┐
│                  THE AGENT LOOP                   │
│                                                   │
│         ┌──────────────┐                          │
│         │   PERCEIVE   │ ← receive input/feedback │
│         └──────┬───────┘                          │
│                │                                  │
│                ▼                                  │
│         ┌──────────────┐                          │
│         │    THINK      │ ← LLM reasons about     │
│         │   (Reason)    │   what to do next        │
│         └──────┬───────┘                          │
│                │                                  │
│                ▼                                  │
│         ┌──────────────┐                          │
│         │     ACT       │ ← execute a tool/action  │
│         │   (Tool Use)  │   (search, code, API)    │
│         └──────┬───────┘                          │
│                │                                  │
│                ▼                                  │
│         ┌──────────────┐                          │
│         │   OBSERVE     │ ← see the result         │
│         └──────┬───────┘                          │
│                │                                  │
│           Done? ──── NO ───→ loop back to THINK   │
│                │                                  │
│               YES                                 │
│                │                                  │
│                ▼                                  │
│         ┌──────────────┐                          │
│         │   RESPOND     │ ← give final answer      │
│         └──────────────┘                          │
└──────────────────────────────────────────────────┘
```

### Real Example: "Book me a flight to Tokyo next Friday"

```
LOOP 1:
  THINK:   "I need to find flights to Tokyo. Let me search."
  ACT:     search_flights(destination="Tokyo", date="next Friday")
  OBSERVE: Found 5 flights: JAL $800, ANA $750, Delta $900...

LOOP 2:
  THINK:   "I found flights. The cheapest is ANA $750. But I should check
            the user's preferences. Do they have a preferred airline?"
  ACT:     check_user_preferences(user_id="123")
  OBSERVE: User prefers JAL, has JAL miles.

LOOP 3:
  THINK:   "User prefers JAL. JAL flight is $800. Let me book it."
  ACT:     book_flight(flight="JAL-123", date="next Friday", class="economy")
  OBSERVE: Booking confirmed! Confirmation #JAL-789456

LOOP 4:
  THINK:   "Flight is booked. I should send the confirmation to the user."
  RESPOND: "I've booked your JAL flight to Tokyo for next Friday!
            Confirmation: JAL-789456. Cost: $800. Departure: 10:30 AM."
```

The agent made **3 tool calls** across **4 thinking loops** — no human intervention needed after the initial request.

---

## Agent Components (The 5 Parts of Every Agent)

```
┌─────────────────────────────────────────────────────────────┐
│                    ANATOMY OF AN AI AGENT                     │
│                                                              │
│  ┌────────────────┐                                          │
│  │  1. BRAIN       │  The LLM (GPT-4, Claude, etc.)          │
│  │     (LLM)       │  Makes decisions, reasons, plans         │
│  └────────────────┘                                          │
│                                                              │
│  ┌────────────────┐                                          │
│  │  2. TOOLS       │  Functions the agent can call            │
│  │     (Actions)   │  search_web(), send_email(),             │
│  │                 │  query_database(), run_code()            │
│  └────────────────┘                                          │
│                                                              │
│  ┌────────────────┐                                          │
│  │  3. MEMORY      │  What the agent remembers                │
│  │     (State)     │  Short-term: current conversation        │
│  │                 │  Long-term: past interactions, facts     │
│  └────────────────┘                                          │
│                                                              │
│  ┌────────────────┐                                          │
│  │  4. PLANNING    │  How the agent breaks down tasks         │
│  │     (Strategy)  │  "First do X, then Y, then Z"           │
│  │                 │  Can re-plan if something fails          │
│  └────────────────┘                                          │
│                                                              │
│  ┌────────────────┐                                          │
│  │  5. PERSONA     │  System prompt / instructions            │
│  │     (Role)      │  "You are a customer support agent..."   │
│  │                 │  Defines behavior, boundaries, tone      │
│  └────────────────┘                                          │
└─────────────────────────────────────────────────────────────┘
```

---

## Agent Reasoning Patterns

How does the agent decide what to do? There are several well-known patterns:

### 1. ReAct (Reasoning + Acting) — Most Popular

The agent alternates between **Reasoning** (thinking out loud) and **Acting** (using tools).

```
User: "What's the population of the capital of France?"

Agent:
  Thought: I need to find the capital of France first.
  Action:  search("capital of France")
  Observation: Paris is the capital of France.

  Thought: Now I need the population of Paris.
  Action:  search("population of Paris")
  Observation: Paris has a population of about 2.1 million (city), 12 million (metro).

  Thought: I now have enough information to answer.
  Answer:  The capital of France is Paris, with a population of about 2.1 million
           in the city proper and 12 million in the metropolitan area.
```

```
ReAct Pattern:

  Thought → Action → Observation → Thought → Action → Observation → ... → Answer

  The LLM explicitly WRITES its reasoning at each step.
  This makes the agent more reliable because it "thinks before acting."
```

```python
# ReAct in LangGraph (simplified)
from langgraph.prebuilt import create_react_agent

agent = create_react_agent(
    model=ChatOpenAI(model="gpt-4"),
    tools=[search_tool, calculator_tool]
)

result = agent.invoke({
    "messages": [HumanMessage("What's the population of France's capital?")]
})
```

### 2. Plan-and-Execute

The agent first creates a **full plan**, then executes each step.

```
User: "Create a blog post about AI, with research and images."

PLANNING PHASE:
  Agent: Here's my plan:
    Step 1: Research current AI trends
    Step 2: Find statistics and data points
    Step 3: Write the blog post outline
    Step 4: Write each section
    Step 5: Generate/find relevant images
    Step 6: Format and finalize

EXECUTION PHASE:
  Executing Step 1: search("AI trends 2025")... done ✅
  Executing Step 2: search("AI statistics 2025")... done ✅
  Executing Step 3: writing outline... done ✅
  Executing Step 4: writing sections... done ✅
  Executing Step 5: generate_image("AI concept art")... done ✅
  Executing Step 6: formatting... done ✅

  RE-PLANNING (if needed):
  "Step 5 failed — image generation API is down.
   New plan: Use stock photos instead."
  Executing Step 5b: search_stock_photos("AI")... done ✅
```

```
Plan-and-Execute Pattern:

  Plan entire task → Execute step 1 → Execute step 2 → ...
                         ↓ (if something fails)
                    Re-plan remaining steps → Continue

  Better for complex, multi-step tasks.
  The upfront planning prevents the agent from getting lost.
```

### 3. Reflexion (Self-Reflection)

The agent evaluates its own work and improves it.

```
User: "Write a Python function to sort a list."

ATTEMPT 1:
  Agent writes: def sort_list(lst): return sorted(lst)

  REFLECTION: "This works but it's too simple. The user might want
   to see an actual sorting algorithm, not just a built-in function."

ATTEMPT 2:
  Agent writes: def sort_list(lst):  # bubble sort implementation...

  REFLECTION: "Bubble sort works but is O(n²). Let me also add
   quicksort for a better solution."

ATTEMPT 3:
  Agent writes: both implementations with docstrings and examples.

  REFLECTION: "This looks good. Both algorithms, documented, tested."
  FINAL OUTPUT: returns the refined code.
```

```
Reflexion Pattern:

  Generate → Evaluate → Critique → Improve → Evaluate → ... → Final

  The agent is its own reviewer/critic.
  Produces higher quality output through self-improvement.
```

### 4. Tool-Use (Function Calling)

The simplest pattern — the LLM decides which function to call based on the user's question.

```
User: "What's 25 * 48 + 17?"

LLM internally decides:
  → tool_call: calculator(expression="25 * 48 + 17")
  → result: 1217

LLM: "25 * 48 + 17 = 1,217"
```

This is what OpenAI calls "function calling" and Anthropic calls "tool use." It's the foundation all other patterns build on.

### Pattern Comparison

| Pattern | Complexity | Best For | Weakness |
|---------|-----------|----------|----------|
| **Tool-Use** | Low | Simple tasks (weather, calc) | No multi-step reasoning |
| **ReAct** | Medium | Research, Q&A, general agents | Can get stuck in loops |
| **Plan-and-Execute** | High | Complex multi-step tasks | Upfront planning can be wrong |
| **Reflexion** | High | Code generation, writing | Slower (multiple attempts) |

---

## Agent Memory Systems

Agents need memory to be useful across time. There are 3 types:

```
┌──────────────────────────────────────────────────────────────┐
│                    AGENT MEMORY TYPES                          │
│                                                               │
│  ┌──────────────────────────────────────┐                     │
│  │  1. SHORT-TERM MEMORY                │                     │
│  │     (Working Memory / Context Window) │                     │
│  │                                       │                     │
│  │  = The current conversation           │                     │
│  │  = What the agent is thinking NOW     │                     │
│  │  = Disappears when conversation ends  │                     │
│  │                                       │                     │
│  │  Like: your RAM                       │                     │
│  │  Human analogy: what you're thinking  │                     │
│  │  about right now                      │                     │
│  └──────────────────────────────────────┘                     │
│                                                               │
│  ┌──────────────────────────────────────┐                     │
│  │  2. LONG-TERM MEMORY                 │                     │
│  │     (Persistent Memory)              │                     │
│  │                                       │                     │
│  │  = Facts stored in a database         │                     │
│  │  = User preferences, past decisions   │                     │
│  │  = Survives across conversations      │                     │
│  │                                       │                     │
│  │  Like: your hard drive                │                     │
│  │  Human analogy: things you remember   │                     │
│  │  from weeks ago                       │                     │
│  └──────────────────────────────────────┘                     │
│                                                               │
│  ┌──────────────────────────────────────┐                     │
│  │  3. EPISODIC MEMORY                  │                     │
│  │     (Experience-based)               │                     │
│  │                                       │                     │
│  │  = Past successes and failures        │                     │
│  │  = "Last time I tried X, it failed"   │                     │
│  │  = Helps avoid repeating mistakes     │                     │
│  │                                       │                     │
│  │  Like: experience/wisdom              │                     │
│  │  Human analogy: "I tried that before  │                     │
│  │  and it didn't work"                  │                     │
│  └──────────────────────────────────────┘                     │
└──────────────────────────────────────────────────────────────┘
```

### How Memory Works in Practice

```python
# Short-term: just the message list (built into every agent)
messages = [
    HumanMessage("My name is Rahim"),
    AIMessage("Hello Rahim!"),
    HumanMessage("What is my name?"),   # agent sees all previous messages
]

# Long-term: stored in a database, retrieved when relevant
# Using a vector store for semantic memory
def remember(agent_state):
    # Save important facts
    memory_store.add("User prefers Python over JavaScript")
    memory_store.add("User works at Acme Corp")

def recall(agent_state):
    # Retrieve relevant memories
    query = agent_state["current_question"]
    relevant_memories = memory_store.similarity_search(query, k=3)
    return relevant_memories

# Episodic: stored as past interaction summaries
episodes = [
    "2024-01-15: User asked to deploy to prod. I forgot to run tests first. Deployment failed.",
    "2024-01-20: User asked to deploy again. This time I ran tests first. Success.",
    # Agent learns: ALWAYS run tests before deploying
]
```

---

## Types of AI Agents

### 1. Simple Reflex Agent

Follows pre-defined rules. No reasoning, no memory.

```
IF user says "hello" → respond "Hi! How can I help?"
IF user says "refund" → call refund_tool()
IF user says "bye"   → respond "Goodbye!"

No thinking. Just pattern matching.
Like a vending machine — press button, get result.
```

### 2. Model-Based Agent

Has an internal model of the world. Updates its understanding.

```
Agent knows:
  - User is a premium customer (from database)
  - It's 3 AM in user's timezone
  - Last order had a problem

Agent decides:
  "User is premium + had a recent issue + it's late → be extra apologetic,
   offer a discount, don't suggest calling back during business hours."
```

### 3. Goal-Based Agent

Works toward a specific goal. Plans steps to achieve it.

```
Goal: "Deploy the new feature to production"

Agent plans:
  1. Run unit tests → ✅ passed
  2. Run integration tests → ❌ failed!
  3. Fix the failing test → ✅ fixed
  4. Re-run integration tests → ✅ passed
  5. Create PR → ✅ created
  6. Wait for review → ✅ approved
  7. Merge and deploy → ✅ done

Goal achieved! ✅
```

### 4. Utility-Based Agent

Chooses actions based on a **utility function** — what gives the best outcome.

```
User: "Find me a restaurant"

Option A: Expensive, 5-star, 30 min away → utility score: 7
Option B: Moderate, 4-star, 5 min away  → utility score: 9  ← PICK THIS
Option C: Cheap, 3-star, 10 min away    → utility score: 6

Utility function considers: price, rating, distance, user preferences
Picks the option with highest utility score.
```

### 5. Learning Agent

Improves over time based on feedback.

```
Day 1: Agent recommends restaurants. User rejects expensive ones.
Day 2: Agent learns: "This user is price-sensitive."
        Now prioritizes affordable options.
Day 3: Agent recommends affordable restaurant. User loves it!
        Agent reinforces: "Price-sensitive preference confirmed."
```

### Agent Type Comparison

| Type | Reasoning | Memory | Learning | Example |
|------|-----------|--------|----------|---------|
| Simple Reflex | None | None | No | Chatbot with fixed responses |
| Model-Based | Basic | World model | No | Context-aware assistant |
| Goal-Based | Planning | Goals + State | No | Task automation agent |
| Utility-Based | Optimization | Preferences | No | Recommendation agent |
| Learning | Full | All types | Yes | Adaptive personal assistant |

---

## Building an AI Agent — Code Examples

### Simple Agent with Tools (LangGraph)

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage

# 1. Define tools
@tool
def search_web(query: str) -> str:
    """Search the internet for information."""
    return f"Search results for '{query}': Python is a popular programming language..."

@tool
def get_weather(city: str) -> str:
    """Get current weather for a city."""
    return f"Weather in {city}: 22°C, Partly cloudy"

@tool
def calculate(expression: str) -> str:
    """Calculate a mathematical expression."""
    return str(eval(expression))

@tool
def send_email(to: str, subject: str, body: str) -> str:
    """Send an email to someone."""
    return f"Email sent to {to} with subject '{subject}'"

tools = [search_web, get_weather, calculate, send_email]

# 2. Create LLM with tools
llm = ChatOpenAI(model="gpt-4", temperature=0).bind_tools(tools)

# 3. Define state
class AgentState(TypedDict):
    messages: Annotated[list, add_messages]

# 4. Define nodes
def agent_node(state: AgentState) -> dict:
    """The brain — decides what to do."""
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

def should_continue(state: AgentState) -> str:
    """Check if agent wants to use a tool or is done."""
    last_message = state["messages"][-1]
    if last_message.tool_calls:
        return "tools"
    return END

# 5. Build graph
graph = StateGraph(AgentState)
graph.add_node("agent", agent_node)
graph.add_node("tools", ToolNode(tools))
graph.add_edge(START, "agent")
graph.add_conditional_edges("agent", should_continue, {"tools": "tools", END: END})
graph.add_edge("tools", "agent")

agent = graph.compile()

# 6. Run
result = agent.invoke({
    "messages": [HumanMessage("What's the weather in Tokyo and what's 42 * 58?")]
})

for msg in result["messages"]:
    print(f"{type(msg).__name__}: {msg.content}")
```

### Agent with Memory and Planning

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.checkpoint.memory import MemorySaver
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, SystemMessage

class AgentState(TypedDict):
    messages: Annotated[list, add_messages]
    plan: list[str]
    current_step: int
    results: dict

llm = ChatOpenAI(model="gpt-4", temperature=0)

def create_plan(state: AgentState) -> dict:
    """Break the task into steps."""
    response = llm.invoke([
        SystemMessage(content="Break this task into 3-5 clear steps. Return as a numbered list."),
        state["messages"][-1]
    ])

    steps = [line.strip() for line in response.content.split("\n") if line.strip()]
    return {"plan": steps, "current_step": 0}

def execute_step(state: AgentState) -> dict:
    """Execute the current step of the plan."""
    step = state["plan"][state["current_step"]]

    response = llm.invoke([
        SystemMessage(content=f"Execute this step and return the result: {step}"),
        *state["messages"]
    ])

    results = state.get("results", {})
    results[f"step_{state['current_step']}"] = response.content

    return {
        "current_step": state["current_step"] + 1,
        "results": results,
        "messages": [response]
    }

def check_progress(state: AgentState) -> str:
    """Check if all steps are done."""
    if state["current_step"] < len(state["plan"]):
        return "execute"   # more steps to do
    return "summarize"     # all done

def summarize(state: AgentState) -> dict:
    """Summarize all results into a final answer."""
    all_results = "\n".join([f"{k}: {v}" for k, v in state["results"].items()])

    response = llm.invoke([
        SystemMessage(content="Summarize these results into a clear final answer:"),
        HumanMessage(content=all_results)
    ])

    return {"messages": [response]}

# Build graph
graph = StateGraph(AgentState)
graph.add_node("plan", create_plan)
graph.add_node("execute", execute_step)
graph.add_node("summarize", summarize)

graph.add_edge(START, "plan")
graph.add_edge("plan", "execute")
graph.add_conditional_edges("execute", check_progress, {"execute": "execute", "summarize": "summarize"})
graph.add_edge("summarize", END)

# Add memory so agent remembers across conversations
memory = MemorySaver()
agent = graph.compile(checkpointer=memory)

# Run with persistent memory
config = {"configurable": {"thread_id": "user_001"}}
result = agent.invoke(
    {"messages": [HumanMessage("Research and summarize the top 3 Python web frameworks")]},
    config
)
```

```
Flow:
START → Plan → Execute Step 1 → Execute Step 2 → Execute Step 3 → Summarize → END
                  ↑                                    │
                  └──────── loop until all done ────────┘
```

---

---

# Multi-Agent Systems

---

## What is a Multi-Agent System?

A Multi-Agent System (MAS) is when **multiple AI agents work together** to accomplish a task, each with their own specialty.

### Single Agent vs Multi-Agent

```
SINGLE AGENT:
  One agent does EVERYTHING — research, write, review, code, test.
  Like one person doing all the work alone.

  Problem: Jack of all trades, master of none.
  The prompt becomes massive, the agent gets confused.

MULTI-AGENT:
  Multiple specialized agents collaborate.
  Like a TEAM where each person has a role.

  Researcher Agent → Writer Agent → Reviewer Agent → Publisher Agent
  Each agent is focused, expert in their area.
```

### Why Multi-Agent?

```
Imagine building a software project:

SINGLE AGENT approach:
  "You are a developer. Write code, test it, review it, deploy it,
   write docs, handle security, optimize performance..."
  → Agent is overwhelmed. Bad at everything.

MULTI-AGENT approach:
  Agent 1 (Architect):  "Design the system"
  Agent 2 (Developer):  "Write the code"
  Agent 3 (Tester):     "Test the code"
  Agent 4 (Reviewer):   "Review for quality"
  Agent 5 (DevOps):     "Deploy to production"
  → Each agent is focused. Good at their job.
```

### Real-World Analogy

```
Single Agent = Solo freelancer doing everything
Multi-Agent  = A company with departments

┌─────────────────────────────────────────────┐
│              COMPANY (Multi-Agent)            │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Research  │  │Marketing │  │Engineering│  │
│  │ Dept     │  │ Dept     │  │ Dept      │  │
│  │          │  │          │  │           │  │
│  │ Finds    │──│ Writes   │──│ Builds    │  │
│  │ data     │→ │ content  │→ │ product   │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│       ↑              ↑             ↑         │
│       └──────── Manager Agent ─────┘         │
│                (coordinates)                 │
└─────────────────────────────────────────────┘
```

---

## Multi-Agent Communication Patterns

How do agents talk to each other? There are 4 main patterns:

### 1. Sequential (Pipeline)

Agents work one after another. Output of Agent A becomes input of Agent B.

```
Agent A → Agent B → Agent C → Agent D → Final Output
(research)  (write)   (review)  (publish)

Like an assembly line in a factory.
```

```python
# Sequential multi-agent
class MultiAgentState(TypedDict):
    messages: Annotated[list, add_messages]
    research: str
    draft: str
    review: str
    final: str

def researcher(state):
    response = research_llm.invoke("Research this topic: " + state["messages"][-1].content)
    return {"research": response.content}

def writer(state):
    response = writer_llm.invoke("Write an article using this research:\n" + state["research"])
    return {"draft": response.content}

def reviewer(state):
    response = reviewer_llm.invoke("Review this article:\n" + state["draft"])
    return {"review": response.content}

graph = StateGraph(MultiAgentState)
graph.add_node("researcher", researcher)
graph.add_node("writer", writer)
graph.add_node("reviewer", reviewer)

graph.add_edge(START, "researcher")
graph.add_edge("researcher", "writer")
graph.add_edge("writer", "reviewer")
graph.add_edge("reviewer", END)
```

**Best for:** Content pipelines, data processing, step-by-step workflows

### 2. Hierarchical (Manager + Workers)

One "boss" agent delegates tasks to "worker" agents.

```
                ┌───────────────┐
                │  MANAGER      │
                │  (Supervisor) │
                │               │
                │  Decides who  │
                │  does what    │
                └───────┬───────┘
                        │
           ┌────────────┼────────────┐
           │            │            │
     ┌─────▼─────┐ ┌───▼─────┐ ┌───▼──────┐
     │ Worker A  │ │Worker B │ │ Worker C │
     │ (Research)│ │(Code)   │ │(Test)    │
     └───────────┘ └─────────┘ └──────────┘

Manager: "Worker A, go research. Worker B, wait.
          Ok Worker A is done. Worker B, write code using A's research.
          Worker C, test B's code."
```

```python
from langgraph.graph import StateGraph, START, END
from langchain_openai import ChatOpenAI

class TeamState(TypedDict):
    messages: Annotated[list, add_messages]
    task: str
    next_agent: str
    research_output: str
    code_output: str
    test_output: str

manager_llm = ChatOpenAI(model="gpt-4", temperature=0)

def manager(state: TeamState) -> dict:
    """The boss — decides which worker to call next."""
    response = manager_llm.invoke([
        SystemMessage(content="""You are a project manager.
        Available workers: researcher, coder, tester.
        Based on current progress, decide who should work next.
        If all work is done, say 'DONE'.
        Return ONLY the worker name or 'DONE'."""),
        HumanMessage(content=f"""
        Task: {state['task']}
        Research: {state.get('research_output', 'not done')}
        Code: {state.get('code_output', 'not done')}
        Tests: {state.get('test_output', 'not done')}
        """)
    ])

    next_agent = response.content.strip().lower()
    return {"next_agent": next_agent}

def researcher(state: TeamState) -> dict:
    response = ChatOpenAI(model="gpt-4").invoke([
        SystemMessage(content="You are a researcher. Research the given topic thoroughly."),
        HumanMessage(content=state["task"])
    ])
    return {"research_output": response.content}

def coder(state: TeamState) -> dict:
    response = ChatOpenAI(model="gpt-4").invoke([
        SystemMessage(content="You are a developer. Write code based on the research."),
        HumanMessage(content=f"Research:\n{state['research_output']}\n\nWrite code for: {state['task']}")
    ])
    return {"code_output": response.content}

def tester(state: TeamState) -> dict:
    response = ChatOpenAI(model="gpt-4").invoke([
        SystemMessage(content="You are a QA tester. Test the code and report issues."),
        HumanMessage(content=f"Code:\n{state['code_output']}\n\nTest this code thoroughly.")
    ])
    return {"test_output": response.content}

def route_to_worker(state: TeamState) -> str:
    next_agent = state.get("next_agent", "")
    if "researcher" in next_agent:
        return "researcher"
    elif "coder" in next_agent:
        return "coder"
    elif "tester" in next_agent:
        return "tester"
    else:
        return END

# Build graph
graph = StateGraph(TeamState)
graph.add_node("manager", manager)
graph.add_node("researcher", researcher)
graph.add_node("coder", coder)
graph.add_node("tester", tester)

graph.add_edge(START, "manager")
graph.add_conditional_edges("manager", route_to_worker, {
    "researcher": "researcher",
    "coder": "coder",
    "tester": "tester",
    END: END
})
# All workers report back to manager
graph.add_edge("researcher", "manager")
graph.add_edge("coder", "manager")
graph.add_edge("tester", "manager")

team = graph.compile()

result = team.invoke({
    "messages": [HumanMessage("Build a calculator API")],
    "task": "Build a calculator API",
    "next_agent": "",
})
```

```
Flow:
                    ┌──────────┐
            ┌──────→│ Manager  │──────┐
            │       └──────────┘      │
            │            │            │
            │     "who's next?"       │
            │            │            │
       ┌────┴──┐   ┌────▼───┐   ┌───▼────┐
       │Tester │   │Research│   │ Coder  │
       │       │   │        │   │        │
       └───┬───┘   └───┬────┘   └───┬────┘
           │           │            │
           └───────────┴────────────┘
                 (back to manager)
```

**Best for:** Complex projects, when you need dynamic task assignment

### 3. Peer-to-Peer (Debate / Discussion)

Agents talk directly to each other, debating or collaborating.

```
┌──────────┐     ┌──────────┐
│ Agent A   │←──→│ Agent B   │
│ (Pro side)│     │(Con side) │
└──────────┘     └──────────┘

Round 1:
  A: "AI will create more jobs than it destroys because..."
  B: "I disagree. Automation historically replaces low-skill jobs..."

Round 2:
  A: "But new industries emerge. The internet created millions of jobs..."
  B: "Fair point, but the pace of AI replacement is unprecedented..."

Round 3:
  A: "Let's find middle ground. AI will displace some jobs but..."
  B: "Agreed. The key is retraining and education..."

Final: Balanced, well-argued analysis from both perspectives.
```

```python
def debate_agent_a(state):
    """Argues FOR the topic."""
    history = state.get("debate_history", "")
    response = llm.invoke([
        SystemMessage(content="You argue IN FAVOR of the topic. Be persuasive but fair."),
        HumanMessage(content=f"Topic: {state['topic']}\nDebate so far:\n{history}\n\nYour turn:")
    ])
    return {"debate_history": history + f"\nPRO: {response.content}"}

def debate_agent_b(state):
    """Argues AGAINST the topic."""
    history = state.get("debate_history", "")
    response = llm.invoke([
        SystemMessage(content="You argue AGAINST the topic. Be critical but fair."),
        HumanMessage(content=f"Topic: {state['topic']}\nDebate so far:\n{history}\n\nYour turn:")
    ])
    return {"debate_history": history + f"\nCON: {response.content}"}

def check_rounds(state):
    rounds = state.get("debate_history", "").count("PRO:")
    return "continue" if rounds < 3 else "summarize"

# A → B → A → B → A → B → Summarize
```

**Best for:** Analysis, decision-making, exploring multiple perspectives

### 4. Broadcast (One-to-Many Parallel)

One agent sends a task to many agents simultaneously, then collects results.

```
                   ┌──────────┐
                   │ Broadcast│
                   │ Agent    │
                   └─────┬────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
              ▼          ▼          ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ Search   │ │ Search   │ │ Search   │
        │ Google   │ │ Wikipedia│ │ Database │
        └────┬─────┘ └────┬─────┘ └────┬─────┘
             │            │            │
             └──────────┬─┴────────────┘
                        │
                  ┌─────▼─────┐
                  │ Collector  │
                  │ Agent      │
                  │ (merge)    │
                  └────────────┘

All searches run in PARALLEL, then results are merged.
```

**Best for:** Research, data gathering, when speed matters

### Communication Pattern Comparison

| Pattern | Speed | Complexity | Control | Best For |
|---------|-------|-----------|---------|----------|
| **Sequential** | Slow (one at a time) | Low | High | Step-by-step workflows |
| **Hierarchical** | Medium | Medium | High (manager) | Complex projects |
| **Peer-to-Peer** | Slow (back-and-forth) | Medium | Low | Debate, analysis |
| **Broadcast** | Fast (parallel) | Low | Medium | Research, data gathering |

---

## Multi-Agent Frameworks

### 1. LangGraph (Recommended — Most Flexible)

Build any agent architecture with graphs.

```python
from langgraph.prebuilt import create_react_agent

# Create specialized agents
research_agent = create_react_agent(model=llm, tools=[search_tool, wiki_tool])
code_agent = create_react_agent(model=llm, tools=[python_repl, file_tool])
review_agent = create_react_agent(model=llm, tools=[lint_tool, test_tool])

# Orchestrate them in a graph
graph = StateGraph(State)
graph.add_node("research", research_agent)
graph.add_node("code", code_agent)
graph.add_node("review", review_agent)
# ... add edges, conditions, loops
```

### 2. CrewAI (Easiest for Multi-Agent)

Designed specifically for multi-agent collaboration.

```python
from crewai import Agent, Task, Crew

# Define agents with roles
researcher = Agent(
    role="Senior Researcher",
    goal="Find the most relevant information",
    backstory="You are an expert researcher with 20 years of experience.",
    tools=[search_tool],
    llm=ChatOpenAI(model="gpt-4")
)

writer = Agent(
    role="Content Writer",
    goal="Write engaging, accurate content",
    backstory="You are a skilled writer for tech blogs.",
    llm=ChatOpenAI(model="gpt-4")
)

editor = Agent(
    role="Editor",
    goal="Ensure quality, accuracy, and readability",
    backstory="You are a strict editor who ensures perfection.",
    llm=ChatOpenAI(model="gpt-4")
)

# Define tasks
research_task = Task(
    description="Research the latest trends in AI agents",
    agent=researcher,
    expected_output="A detailed research report"
)

writing_task = Task(
    description="Write a blog post based on the research",
    agent=writer,
    expected_output="A 1000-word blog post",
    context=[research_task]   # depends on research
)

editing_task = Task(
    description="Review and edit the blog post",
    agent=editor,
    expected_output="Final polished blog post",
    context=[writing_task]    # depends on writing
)

# Create crew and run
crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, writing_task, editing_task],
    verbose=True
)

result = crew.kickoff()
print(result)
```

### 3. AutoGen (Microsoft — Conversation-Based)

Agents collaborate through conversations.

```python
from autogen import AssistantAgent, UserProxyAgent

# Create agents
assistant = AssistantAgent(
    name="assistant",
    llm_config={"model": "gpt-4"}
)

user_proxy = UserProxyAgent(
    name="user_proxy",
    human_input_mode="NEVER",           # no human in loop
    code_execution_config={"work_dir": "coding"}
)

# Agents chat with each other to solve the task
user_proxy.initiate_chat(
    assistant,
    message="Write a Python function to find prime numbers and test it."
)
# assistant writes code → user_proxy runs it → assistant fixes errors → repeat
```

### 4. OpenAI Swarm (Lightweight, Experimental)

Simple agent handoff framework by OpenAI.

```python
from swarm import Swarm, Agent

client = Swarm()

# Define agents
triage_agent = Agent(
    name="Triage Agent",
    instructions="Determine which department can help the user.",
)

sales_agent = Agent(
    name="Sales Agent",
    instructions="Help users with purchasing and pricing questions.",
)

support_agent = Agent(
    name="Support Agent",
    instructions="Help users with technical issues.",
)

# Triage agent hands off to the right specialist
def transfer_to_sales():
    return sales_agent

def transfer_to_support():
    return support_agent

triage_agent.functions = [transfer_to_sales, transfer_to_support]

# Run
response = client.run(
    agent=triage_agent,
    messages=[{"role": "user", "content": "I want to buy your product"}]
)
# Triage → detects sales intent → hands off to sales_agent
```

### Framework Comparison

| Feature | LangGraph | CrewAI | AutoGen | Swarm |
|---------|-----------|--------|---------|-------|
| **Architecture** | Graph (any structure) | Role-based teams | Conversation-based | Handoff-based |
| **Flexibility** | Highest | Medium | Medium | Low |
| **Ease of use** | Medium | Easiest | Medium | Easy |
| **State management** | Built-in | Basic | Basic | Minimal |
| **Human-in-loop** | Built-in | Basic | Built-in | No |
| **Loops/Retries** | Built-in | Basic | Via conversation | No |
| **Streaming** | Built-in | No | No | No |
| **Checkpointing** | Built-in | No | No | No |
| **Production-ready** | Yes | Growing | Growing | Experimental |
| **Best for** | Complex workflows | Team-based tasks | Code generation | Simple handoffs |

---

## Multi-Agent Design Patterns (Advanced)

### 1. Supervisor Pattern

One agent supervises multiple workers, decides who works when.

```
              ┌────────────────┐
              │   SUPERVISOR    │
              │                 │
              │  "Who should    │
              │   work next?"   │
              └───────┬─────────┘
                      │
         ┌────────────┼────────────┐
         │            │            │
    ┌────▼────┐  ┌───▼─────┐  ┌──▼──────┐
    │ Agent 1 │  │ Agent 2 │  │ Agent 3 │
    │ Search  │  │ Analyze │  │ Write   │
    └────┬────┘  └───┬─────┘  └──┬──────┘
         │           │           │
         └───────────┴───────────┘
              (report back)
```

### 2. Chain-of-Thought Multi-Agent

Each agent builds on the previous agent's thinking.

```
Agent 1 (Analyzer): "The problem is about sorting efficiency..."
         │
         ▼
Agent 2 (Planner):  "Based on the analysis, we should use quicksort because..."
         │
         ▼
Agent 3 (Coder):    "Here's the quicksort implementation based on the plan..."
         │
         ▼
Agent 4 (Reviewer): "The code looks good but has a bug on line 15..."
         │
         ▼
Agent 3 (Coder):    "Fixed the bug. Here's the updated code..."
```

### 3. Critic-Generator Pattern

One agent generates, another critiques, they iterate.

```
┌────────────┐     generate     ┌────────────┐
│            │ ───────────────→ │            │
│ Generator  │                  │   Critic   │
│ Agent      │ ←─────────────── │   Agent    │
│            │     critique     │            │
└────────────┘                  └────────────┘

Round 1: Generator writes code → Critic finds 3 issues
Round 2: Generator fixes code → Critic finds 1 issue
Round 3: Generator fixes code → Critic approves ✅
```

### 4. Specialist Routing

Route tasks to the right specialist agent.

```
User: "My order hasn't arrived"
         │
    ┌────▼──────┐
    │   Router   │  "This is a shipping issue"
    │   Agent    │
    └────┬──────┘
         │
    Routes to:
         │
    ┌────▼──────────┐
    │ Shipping Agent │  (specialist in delivery tracking)
    │                │
    │ "Let me check  │
    │  tracking #..."│
    └────────────────┘
```

---

## Common Pitfalls & Best Practices

### Pitfalls to Avoid

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Too many agents** | Overhead, confusion, slow | Start with 2-3 agents, add more only if needed |
| **Vague roles** | Agents overlap, duplicate work | Give each agent a CLEAR, specific role |
| **No error handling** | One agent fails, everything crashes | Add retry logic, fallback agents |
| **Infinite loops** | Agents keep calling each other forever | Set max iterations/rounds |
| **No shared context** | Agents lose track of the big picture | Use shared state, pass summaries |
| **Over-engineering** | Multi-agent when single agent would work | Use multi-agent only when single agent fails |

### Best Practices

```
1. START SIMPLE
   Begin with a single agent.
   Only add more agents when one agent can't handle the complexity.

2. CLEAR ROLES
   Each agent should have a SINGLE, well-defined responsibility.
   Bad:  "You are a general assistant"
   Good: "You are a code reviewer. ONLY review code for bugs and style."

3. LIMIT COMMUNICATION
   Agents should share summaries, not entire conversations.
   Bad:  Pass 100 messages between agents
   Good: Pass a structured summary: {"findings": "...", "recommendation": "..."}

4. SET BOUNDARIES
   Max iterations: prevent infinite loops
   Max tokens: prevent runaway costs
   Timeouts: prevent hanging agents

5. OBSERVE AND DEBUG
   Log every agent's decision and action.
   Use LangSmith or similar tools to trace multi-agent flows.
   If something goes wrong, you need to know WHICH agent messed up.

6. HUMAN-IN-THE-LOOP
   For critical decisions, pause and ask a human.
   "I'm about to delete the database. Should I proceed?"
   Don't let agents make irreversible decisions autonomously.
```

---

## When to Use Single Agent vs Multi-Agent

```
USE SINGLE AGENT when:
├── Task is straightforward (Q&A, search, calculate)
├── One set of tools is enough
├── No need for multiple perspectives
├── Speed matters more than depth
└── You're starting out (keep it simple!)

USE MULTI-AGENT when:
├── Task requires multiple DISTINCT skills (research + code + test)
├── Different sub-tasks need different tools/prompts
├── You need checks and balances (writer + reviewer)
├── Task is too complex for one prompt to handle
├── You need parallel processing (search 5 sources at once)
└── Single agent keeps failing or producing poor results
```

---

## Summary

```
AI AGENT = LLM + Tools + Memory + Reasoning Loop
├── Think:   LLM decides what to do
├── Act:     Execute a tool (search, code, API)
├── Observe: See the result
└── Repeat:  Until task is complete

REASONING PATTERNS:
├── ReAct:          Think → Act → Observe → Repeat (most common)
├── Plan-Execute:   Plan all steps → Execute one by one
├── Reflexion:      Generate → Self-critique → Improve → Repeat
└── Tool-Use:       LLM picks the right function to call

AGENT COMPONENTS:
├── Brain:    The LLM (GPT-4, Claude)
├── Tools:    Functions it can call
├── Memory:   Short-term (conversation) + Long-term (database)
├── Planning: How it breaks down tasks
└── Persona:  System prompt defining behavior

MULTI-AGENT SYSTEMS = Multiple specialized agents working together
├── Sequential:    A → B → C (pipeline)
├── Hierarchical:  Manager assigns tasks to workers
├── Peer-to-Peer:  Agents debate/discuss directly
└── Broadcast:     One sends to many in parallel

MULTI-AGENT FRAMEWORKS:
├── LangGraph:  Most flexible (graph-based, production-ready)
├── CrewAI:     Easiest for teams (role-based, beginner-friendly)
├── AutoGen:    Conversation-based (good for code generation)
└── Swarm:      Simple handoffs (experimental, lightweight)

GOLDEN RULES:
├── Start with a single agent — add more only when needed
├── Each agent = one clear role
├── Always set max iterations to prevent infinite loops
├── Log everything for debugging
└── Human-in-the-loop for critical decisions
```
