# agent-monitors

Example Monte Carlo agent monitors for two reference agent implementations: an **account health agent** and a **loan underwriting agent**. These are intended as starting points — adapt the span filters, thresholds, and eval prompts to match your agent's workflow and span naming conventions.

## What's in this repo

| File | Monitor Type | Agent | Description |
|------|-------------|-------|-------------|
| `account-health-agent-recommendation-rarionalization.yaml` | Agent Evaluation (LLM-as-judge) | `account-health-agent` | Scores whether the agent's intervention recommendation is proportionate to the full account context, not just a single dominant metric |
| `account-health-agent-recommendation-rationalization` | Agent Evaluation (LLM-as-judge) | `account-health-agent` | Monte Carlo monitor export for the recommendation rationalization monitor above |
| `Token_consumption_anomaly_for_agent__account_health_agent` | Agent Metric | `account-health-agent` | Detects anomalous token consumption patterns across account health agent runs |
| `Underwriting_Agent_-_Context_Gathering_and_Tool_Call_Validation` | Agent Evaluation | `loan-underwriting-agent` | Validates that the underwriting agent gathered the expected context and executed the required tool calls before producing a decision |
| `Loan_Underwriting_Agent___LLM_inference_call_missing` | Agent Trajectory | `loan-underwriting-agent` | Alerts when a trace completes all tool checks but produces no LLM output — a silent failure indicating an incomplete underwriting decision |

## Monitor types

These examples cover three of Monte Carlo's agent monitor types:

**Agent Evaluation** — Uses an LLM-as-judge to score the quality of agent outputs against a rubric. The `intervention_proportionality_score` in the account health example scores 1–5 and alerts when the mean drops below 3.5, catching cases where the agent over-rotates on a single signal and ignores contradicting context.

**Agent Trajectory** — Validates the sequence of spans in a trace. The underwriting example alerts when `ChatBedrockConverse.chat` is never called in a `loan_underwriting` trace, meaning the agent completed its tool checks but never produced a decision.

**Agent Metric** — Monitors statistical properties of agent behavior over time. The token consumption monitor flags anomalous usage patterns that can indicate prompt injection, runaway loops, or unexpected changes in agent behavior.

## Prerequisites

- A Monte Carlo account with Agent Observability enabled
- Agents instrumented with the `montecarlo-opentelemetry` SDK, emitting OTEL traces to your configured data store
- Span names, workflow names, and task names in your traces must match the `agent_span_filters` in each monitor — update these to reflect your agent's actual span naming

## Usage

The `.yaml` files can be applied directly via the Monte Carlo CLI:

```bash
montecarlo apply --config account-health-agent-recommendation-rarionalization.yaml
```

The files without extensions are raw monitor exports from the Monte Carlo API. They document the full monitor configuration as it exists in a live environment and can be used as a reference when building equivalent monitors through the UI or CLI.

## Adapting these monitors

The two things most likely to need changes when using these in your own environment:

1. **`agent_span_filters`** — `agent`, `workflow`, `task`, and `span_name` must match the span attributes your instrumentation emits. If your spans are named differently, the monitor will never match any data.

2. **Eval prompts** — The LLM-as-judge prompts in the evaluation monitors are tightly coupled to the account health agent's domain (churn scores, NPS, seat utilization). Replace the evaluation criteria and scoring rubric with criteria appropriate to your agent's outputs.

## Resources

- [Monte Carlo Agent Observability docs](https://docs.getmontecarlo.com)
- [montecarlo-opentelemetry SDK](https://pypi.org/project/montecarlo-opentelemetry/)
