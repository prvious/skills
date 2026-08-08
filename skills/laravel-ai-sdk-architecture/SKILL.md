---
name: laravel-ai-sdk-architecture
description: Guide architecture decisions in applications using the first-party Laravel AI SDK by choosing between normal Laravel code, tools, agents, sub-agents, and automations. Use when deciding where an AI responsibility should live or reviewing whether a new agent, tool, sub-agent, or automation boundary is necessary.
metadata:
    short-description: "Apply when designing or reviewing Laravel AI SDK responsibilities across code, tools, agents, sub-agents, and automations."
---

# Laravel AI SDK Architecture

Apply this skill only to applications using the first-party `laravel/ai` package. Assume sub-agent guidance requires SDK support for `CanActAsTool`.

Choose the simplest architecture that gives the model only the decisions and permissions it needs. Reuse an existing agent unless a separate agent boundary provides a clear benefit.

Classify individual responsibilities, not entire features. One feature may combine deterministic Laravel code, model-selected tools, an agent, and an automation.

## Decide in This Order

1. If the step must always happen, use normal Laravel code.
2. If the model may choose one bounded action or lookup, create a tool.
3. If the task needs different instructions, permissions, model, context, or output format, create an agent.
4. If a parent agent must delegate a complete specialist task, create a sub-agent.
5. If an event or schedule starts the work, use an automation.
6. Otherwise, reuse the existing agent.

## Use Normal Laravel Code

Use normal Laravel code for deterministic application behavior and orchestration.

Keep validation, authorization, persistence, event handling, and required side effects outside the model. Do not ask the model to decide work the application already knows must happen.

## Create a Tool

Create a tool when the model needs the option to perform one concrete action or data lookup.

Give each tool one responsibility, explicit inputs, and a predictable result. Enforce validation and permissions in Laravel. Expose only the control the model needs.

Do not create an agent merely to wrap a single capability.

## Create an Agent

Create an agent only when the task requires a meaningfully different:

- Set of instructions.
- Set of tools or permissions.
- Model or provider.
- Conversation context.
- Response format.

Do not create separate agents solely to group related tools.

## Create a Sub-Agent

Create a sub-agent only when a parent agent must delegate a complete specialist task.

Expose the sub-agent from the parent agent's `tools()` method. Implement `CanActAsTool` with a clear name and description. Pass a complete task because the sub-agent does not inherit the parent conversation.

## Use an Automation

An automation decides when work starts. It is not an agent.

A schedule, webhook, inbox event, upload, or queued job should invoke normal Laravel code or an existing agent with only the tools required for that workflow.

Do not create a new agent merely because work starts from a different event.
