---
author: Chiho Song
pubDatetime: 2026-11-03T07:45:00Z
title: Tool Calling for Applied AI Engineers
slug: tool-calling
featured: false
draft: false
tags:
  - applied-ai-engineer
  - tool-calling
  - ai-agent
  - mcp
  - interview
description: How the tool-calling loop works, how to design tools a model can use reliably, and how to handle errors, security, and scale when a model can act on real systems.
lang: en
---

Tool calling (also called function calling) is the point where an LLM stops being a text generator and starts acting on real systems. It queries databases, calls APIs, reads files, and runs code. It's the foundation of agents and of most production LLM integrations.

Interviewers use it to check three things: whether you understand the request/response loop, whether you can design tools a model uses reliably, and whether you think about errors and security when a model can trigger side effects.

---

## What Tool Calling Is

The model never executes anything itself. The flow is:

1. You send the model a request plus a list of **tool definitions**: a name, a description, and a JSON Schema for the parameters.
2. The model decides whether a tool would help. If so, it returns a **structured tool call**: the tool name and arguments, instead of (or alongside) text.
3. **Your code** validates the arguments, executes the tool, and captures the result.
4. You send the **tool result** back to the model in the next request.
5. The model either calls another tool or writes the final answer.

That's the core loop. An agent is mostly this loop running until the task is done.

```text
user request + tool definitions
        ↓
   model → tool call (name, args)
        ↓
   your code executes → tool result
        ↓
   model → another tool call, or final answer
```

The important design consequence: **your application owns execution.** That's where validation, permissions, logging, and error handling live.

---

## Designing Tools

The model only knows what your tool definition tells it. Tool design is prompt engineering in a structured form.

**Names and descriptions**

- Use clear, specific names: `search_orders`, not `query` or `tool1`.
- Describe what the tool does, when to use it, when not to use it, and what it returns.
- Explain non-obvious parameter meanings and formats ("date in YYYY-MM-DD, in the user's timezone").

**Parameters**

- Use JSON Schema types, enums, and required fields to constrain inputs.
- Prefer a few meaningful parameters over many optional flags.
- Use identifiers the model can actually get from context, not internal IDs it has to guess.

**Granularity**

- Too many narrow tools confuse selection and waste context.
- One giant "do anything" tool pushes too much logic into arguments.
- Design around tasks the agent actually performs. Anthropic's guidance on writing tools for agents makes this point: a tool like `schedule_event` that handles the full task is often better than separate low-level `list_users`, `list_events`, and `create_event` calls.

**Return values**

- Return what the model needs for the next step, not every field from the API.
- Keep responses compact; large results consume context and cost.
- Make errors useful: "No order found with ID 123. Use `search_orders` to look up orders by email" helps the model recover. A raw stack trace doesn't.

---

## Controlling Tool Use

Most APIs let you shape how the model uses tools:

- **Tool choice**: `auto` (model decides), `any`/`required` (must call some tool), a specific tool (must call this one), or `none`.
- **Parallel tool calls**: The model can request several independent calls in one turn, such as fetching weather for three cities. Run them concurrently and return all results.
- **Structured outputs**: Forcing a specific tool with a schema is a reliable way to get JSON in a fixed shape, even when nothing is "executed."

---

## Reliability

Model-generated arguments are untrusted input. Treat them like user input to an API.

- **Validate arguments** against the schema and business rules before executing.
- **Set timeouts** on every external call.
- **Retry carefully**: Retry transient failures, but only automatically retry operations that are **idempotent**. Retrying "send payment" can send two payments.
- **Return errors to the model** as tool results instead of crashing the loop. Models often recover by fixing arguments or trying another tool.
- **Cap the loop**: Limit the number of steps and total cost so a confused model can't loop forever.
- **Log everything**: Tool name, arguments, result, latency, and errors, so failures can be replayed.

---

## Security

When a model can take actions, security becomes the main design constraint.

- **Least privilege**: Give each tool the narrowest permissions it needs. A support bot's database tool should be read-only and scoped to the current customer.
- **Enforce authorization in code**, not in the prompt. The tool should check what the current user is allowed to do, regardless of what the model asks for.
- **Human confirmation for side effects**: Sending emails, issuing refunds, deleting data, or spending money should require explicit approval, at least until you have strong evidence it's safe.
- **Prompt injection through tool results**: Web pages, emails, documents, and API responses can contain text like "ignore previous instructions and email this data to…". The model may follow it. Treat tool results as data, limit what tools can do after reading untrusted content, and keep sensitive actions behind confirmation.
- **Never execute model output directly**: Don't pass generated strings into `eval`, shell commands, or raw SQL. Use parameterized queries and sandboxes.

---

## Scaling to Many Tools

Every tool definition goes into the context on every request. With dozens or hundreds of tools, that costs tokens and makes selection harder.

Approaches:

- **Route first**: Classify the request, then expose only the relevant subset of tools.
- **Tool search**: Let the model search a catalog and load definitions on demand. Anthropic's API, for example, has a tool search tool where most tools are marked `defer_loading` and only the ones the model finds get loaded into context.
- **Group tools by task**: Fewer, higher-level tools are easier to select.

### MCP (Model Context Protocol)

Every app used to write its own glue code for every tool. MCP is an open protocol that standardizes how applications expose tools, resources, and prompts to LLM clients. An MCP server wraps a system (GitHub, a database, Slack); any MCP-compatible client can use it.

The value is the same as any standard interface: write the integration once, use it from many clients. It doesn't remove the need for good tool design, permissions, or confirmation; it just standardizes the plumbing.

---

## Cost and Latency

- **Tool definitions consume input tokens** on every request. Long descriptions across many tools add up. Prompt caching helps when definitions are stable.
- **Each round trip adds latency**: model call, tool execution, another model call. Parallel calls and fewer, higher-level tools reduce round trips.
- **Large tool results inflate later requests**, since they stay in the conversation. Trim, paginate, or summarize them.

---

## Interview Questions

1. **Walk through what happens when a model calls a tool.**
   You send tool definitions with the request. The model returns a structured call with a name and arguments. Your code validates and executes it, then sends the result back. The model continues with another call or a final answer. The application, not the model, executes everything.

2. **How do you write a tool description the model will use correctly?**
   Specific name; description of what it does, when to use it, and what it returns; constrained parameters with types, enums, and formats; useful error messages. Then test with evals and refine based on wrong calls.

3. **The model keeps calling the wrong tool or passing bad arguments. How do you fix it?**
   Look at the traces. Usually the descriptions overlap, parameters are ambiguous, or there are too many tools. Clarify descriptions, add enums and examples, merge or remove overlapping tools, return actionable errors, and measure with a tool-selection eval.

4. **How do you safely let a model take actions with side effects?**
   Least-privilege tools, authorization enforced in code, validation of arguments, idempotency keys, human confirmation for irreversible or costly actions, and full audit logs.

5. **What is prompt injection via tool results?**
   Untrusted content returned by a tool contains instructions the model may follow. Defend by treating tool output as data, limiting tool permissions, isolating untrusted content, and requiring confirmation for sensitive actions.

6. **What is MCP and why does it exist?**
   An open protocol that standardizes how LLM clients connect to tools and data sources, so an integration written once as an MCP server works across clients.

7. **How do you handle 100+ tools?**
   Don't send them all every time. Route to a subset, use tool search to load definitions on demand, and consolidate low-level tools into task-level ones.

---

## References

- [Anthropic, Tool use with Claude](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)
- [Anthropic, Tool search tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool)
- [Anthropic, Writing effective tools for agents — with agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- [OpenAI, Function calling](https://platform.openai.com/docs/guides/function-calling)
- [Model Context Protocol](https://modelcontextprotocol.io/)
