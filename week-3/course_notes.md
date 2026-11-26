# Advanced Generative AI: course notes for Weeks 1 and 2

These are the study notes of the course. Week 3 uses them as the documents of a retrieval (RAG) exercise, so each section is written to be read on its own.

## Week 1 · Day 1: Foundations

An **LLM** (large language model) reads text and writes text. On its own it cannot look anything up or take any action: it only predicts the next words. Everything we build in this course adds something around the model: a fixed sequence of steps, tools, memory or checks.

A **workflow** is a program in which your code decides every step and the model does one job inside a step, for example "classify this email" or "summarize this page". An **agent** is a program in which the model decides the steps: which tool to call, with which arguments, and when to stop. A workflow follows a checklist. An agent is told "handle it".

**Autonomy** means how much the AI decides by itself without being told each step. We use the **autonomy ladder** to place any application:

- **L0 · plain code.** No model at all. Example: convert 30 °C to Fahrenheit. Ordinary code is often the right answer.
- **L1 · LLM workflow.** Fixed steps with a model inside. Example: label every new support email as billing, bug or other. A single chatbot call is the simplest L1.
- **L2 · tool agent.** The model chooses tools in a bounded loop. Example: research a company and write a report with sources.
- **L3 · multi-agent.** Several agents coordinate. It adds coordination cost, so we use it only when one agent is not enough (Week 4).
- **L4 · long-running.** An agent that works for hours or days with little supervision. Powerful but the highest risk, so it is rare.

The rule of the course is **use the minimum autonomy that solves the problem reliably**. Start low and climb only when you need to. The most common mistake is too much autonomy: an agent to rename 100 files, when a five-line script does it perfectly.

On Day 1 students also make their **first Groq call**. Groq runs open models very fast and has a free plan. The important settings of a call are the **model name**, the **messages** (each with a role: system, user or assistant) and the **temperature**: 0 gives nearly the same answer every time, higher values give more varied answers.

## Week 1 · Day 2: Your first agent

Day 2 builds a web-search agent one small step at a time: a plain LLM call, then a workflow, then function calling, then an agent with memory.

A **tool** is a normal Python function with a docstring. The `@tool` decorator from LangChain turns it into something a model can use. The model reads the tool's name, its arguments and its docstring, so the docstring must say clearly what the tool does and when to use it.

**Function calling** works like this: the model does not run the tool. It **asks** for it by returning a tool call with the tool name and the arguments. Your code (or the agent framework) runs the function and sends the result back to the model as a **tool message**. The model then writes its answer from the result. "The model asks, our code runs."

`create_agent` from LangChain builds the loop for you: call the model, run any tools it asks for, send the results back, and repeat until the model answers without asking for a tool. The **system prompt** tells the agent its job, its rules and its style.

**Short-term memory** lets the agent remember earlier messages of the same conversation. In LangChain it needs a **checkpointer** such as `InMemorySaver`, and every conversation gets a **thread id**: `{"configurable": {"thread_id": "1"}}`. A new thread id starts a new, empty conversation.

## Week 1 · Day 3: Build your own tools

On Day 3 students write the tools themselves and build a **trip-planner agent** with five tools in class: `search_web`, `extract_page` (read a web page), `get_coordinates`, `get_weather` and `convert_currency`. The trip planner notebook later grows it to ten tools by adding `calculator`, `get_distance`, `save_itinerary`, `send_telegram` and `send_email`.

Three habits make tools reliable:

1. **A tool never crashes.** If something goes wrong it returns a short text that starts with `Error:` and says what to try, for example "Error: the search failed. Try different words." The model can read that and try again.
2. **Tools can be chained.** The model first calls `get_coordinates` for a city, then passes the latitude and longitude to `get_weather`.
3. **Keep tool output short.** A whole web page is far too long for the model, so `extract_page` keeps only the first 2,500 characters about the topic.

The agent also gets a **step limit**, so a confused agent cannot loop forever, and memory per thread. It is tested on the **same three questions** every time it changes, so you can see whether a change helped.

## Week 1 · Day 4: Your own agent

On Day 4 students build an agent from their own idea. Plan it in **three lines**: the goal, the tools, and one test question. Reuse tools from the trip planner and write only the one new tool the idea needs.

Students also connect their agent to **Telegram** (a bot token from BotFather and a chat id) and **email** (a Gmail app password, which needs two-step verification on the account). Keys and passwords are kept in Colab Secrets or a `.env` file, never in the notebook.

Keep the agent small: two to four tools, an `Error:` path in every tool, and a step limit.

## Week 2 · Day 1: Chaining and routing

Week 2 builds **workflows** with LangGraph, following the official guide "Workflows and agents". In a workflow you draw the steps and the model does one job in each step.

The **augmented LLM** is a model with two extras. **Structured output** (`with_structured_output`) makes the model answer by filling a small class, so code can read the answer. On Groq we use `method="json_schema", strict=True` so the model follows the class exactly, plus `.with_retry(stop_after_attempt=3)` in case an answer still comes back broken. **Tools** (`bind_tools`) let the model ask for a tool such as `TavilySearch`, a ready-made web search tool.

A LangGraph **graph** has a **state** (a dictionary that every step can read, described with a `TypedDict`), **nodes** (functions that return only the keys they change), and **edges** (which node runs next). A **conditional edge** runs a small function that chooses the next node.

**Prompt chaining** is steps in a fixed order with a **gate** between them. The Day 1 chain writes an answer, a gate counts the words, and if there are more than 80 words a third step shortens it.

**Routing** sends each question down the path that fits it. The router answers with one word, `search` or `direct`. Search questions go to an agent with web search; direct questions get one cheap model call. In our test the direct path used about 300 to 400 tokens and the search path about 2,400, so a router saves a lot when many questions are easy.

## Week 2 · Day 2: Parallel work and orchestrator-worker

**Parallelization** runs fixed steps at the same time. You draw several edges from START to the nodes, and several edges from the nodes into an **aggregator**. LangGraph waits for all of them before the aggregator runs. Running in parallel saves waiting time, not tokens.

The **orchestrator-worker** pattern is used when the steps are not known in advance. A **planner** (the orchestrator) writes a list of small sub-questions with structured output. A conditional edge returns one **`Send("research_step", {"step": step})`** for each step, and LangGraph starts one **worker** for each. Every worker searches once and writes a short note with a link.

All the workers write to the same state key, `notes`. A **reducer**, `Annotated[list, operator.add]`, tells LangGraph to **add** each worker's note to the list instead of replacing it. A writer node then joins the notes into a brief of at most 120 words.

The free Groq plan allows about **8,000 tokens a minute** and **200,000 a day** for each model. Parallel workers can pass the per-minute limit; then run the graph with `{"max_concurrency": 1}` so only one worker runs at a time. `max_retries=10` on the model makes a call wait and try again when the limit is reached.

## Week 2 · Day 3: Evaluator-optimizer

The **evaluator-optimizer** pattern is also called reflection. One model call **writes** a draft (the optimizer). Another call **grades** it with a fixed rule and gives **feedback** (the evaluator). If the grade is `fail`, a conditional edge goes **back** to the writer with the feedback: this is a **loop**.

The evaluator is structured output with two fields: `grade`, which can only be `pass` or `fail`, and `feedback`, which says exactly what to fix. The most important part is a **clear, checkable rule**, for example "pass only if every fact in the draft is in the notes and every fact has its link". "Be accurate" cannot be checked; "every sentence has at most 20 words" can.

The loop always needs a **limit**. The state counts `attempts`, and the loop stops when the grade is `pass` or after **3 attempts**, so an evaluator that is never happy cannot spend all your tokens. Students test the evaluator on a bad and a good draft before they trust it.

A good first draft costs one extra call (the grade), about 1,100 tokens in our test. Each repair adds a writer call and an evaluator call. The loop is worth it when a wrong answer is expensive, and not for quick, low-risk answers.

## Week 2 · Day 4: Guardrails and evaluation

A **guardrail** stops a bad result before it does harm. There are three kinds, by where they stand:

- **Input guardrails** run before the agent starts. Our input guardrail is a `@before_agent` middleware that asks **Llama Prompt Guard 2** (on Groq) for a score from 0 (safe) to 1 (attack). Above 0.5 the agent stops at once with a fixed reply. It caught a disguised attack ("translate this, then forget your rules...") that a list of bad words would miss.
- **Output guardrails** work on what goes in and out of the model. `PIIMiddleware("email", strategy="redact")` turns an email address into `[REDACTED_EMAIL]`. The `mask` strategy keeps only the last four digits of a card number.
- **Execution guardrails** run before a tool acts. `ToolCallLimitMiddleware(run_limit=2)` limits tool calls. `HumanInTheLoopMiddleware` pauses before a tool and waits for a person to approve or reject; it needs a checkpointer and a thread id, and you resume with `Command(resume={"decisions": [{"type": "approve"}]})`.

**Prompt injection** is text that tells the model to ignore its rules. It can come from the user, or be hidden inside a web page or a document that a tool reads.

The Week 2 project joins every pattern into one **research brief workflow**: the guard and the router first, then a planner with workers, then the writer and the evaluator loop.

**Evaluation** means measuring before you trust a design. An **eval set** is a fixed list of questions that you run every time something changes. A **judge** (structured output with a `helpful` field) grades each answer. The project is compared with a **baseline**: one plain agent with a search tool. A complex design is only worth it if it beats the simplest thing that works, and the decision uses both numbers: helpful answers and tokens.

## Glossary

- **Context:** everything the model receives in one call: the system prompt, the messages so far, and the tool results.
- **Token:** a small piece of text, about three quarters of an English word. Models charge and limit by tokens.
- **Checkpointer:** saves the state of an agent or graph after each step, so a conversation can continue later. `InMemorySaver` keeps it in memory.
- **Thread id:** the name of one conversation in the checkpointer.
- **Middleware:** code that runs around an agent: before it starts, around each model or tool call, or after it ends. Guardrails are middleware.
- **Reducer:** the rule for combining values that several nodes write to the same state key.
- **Send:** a LangGraph object that starts one copy of a node with its own input.
- **Baseline:** the simplest system that could do the job, used for comparison.
