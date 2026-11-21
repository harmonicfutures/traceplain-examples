# Reviewing nested GenAI token usage without double-counting

When an agent delegates work, token-usage numbers on a trace can describe different scopes. A parent span may report only the parent's model calls, or an accumulated total that includes descendants. Adding both kinds across a tree can count the same model call more than once.

This guide is a review method for inspecting that ambiguity. It is not an OpenTelemetry semantic convention, and the example values below are synthetic.

## The review question

For each reported number, ask: does this value mean **direct usage** (model calls made by this invocation only) or **inclusive usage** (this invocation plus nested agents)? If the producer's meaning is unknown, keep it unknown; do not relabel it as per-agent cost.

The open [GenAI semantic-conventions discussion on attributing token usage to agents](https://github.com/open-telemetry/semantic-conventions-genai/issues/695) describes the difficulty: inference-usage counters may not carry an agent identity, while an `invoke_agent` span's usage may include descendants. Producer behavior also differs. The discussion is unresolved; no proposed attribute or histogram should be treated here as settled guidance.

## Synthetic delegation tree

Assume a coordinator delegates to a researcher, which calls a web-search agent; the coordinator also delegates to a writer.

| Invocation | Direct input / output tokens | Inclusive input / output tokens |
| --- | ---: | ---: |
| coordinator | 100 / 20 | 1,000 / 210 |
| researcher | 300 / 50 | 700 / 110 |
| web_search | 400 / 60 | 400 / 60 |
| writer | 200 / 80 | 200 / 80 |

These values are illustrative. The model calls total 1,000 input and 210 output tokens. Summing the *inclusive* values across every row instead produces 2,300 / 460 because descendants are repeated in their ancestors. Conversely, the researcher’s direct 300 / 50 is not its full delegated subtree total of 700 / 110. Both numbers can be useful, but they answer different questions.

## A bounded inspection procedure

1. Start with model-inference spans and record which spans actually contain usage values. Keep each span's model, invocation identity, parent relationship, and any explicit agent identity alongside the value.
2. Inspect one parent/child example. Compare the parent value with its own inference spans and the descendants. Do not assume that a field named `gen_ai.usage.*` is either direct or inclusive without evidence for that instrumentation.
3. Choose one aggregation boundary for the question being answered. For request-level totals, count only a verified top-level inclusive value, or sum verified direct inference usage once. For per-agent own usage, use direct values with verified agent attribution. Do not sum both parent aggregates and child values.
4. If sampling, missing spans, or absent agent identity prevents reconciliation, mark the allocation **unknown**. A partial trace is not a complete cost ledger.
5. Preserve the source evidence and the reviewer’s next decision separately: what the telemetry recorded, what the producer claims the aggregate represents, what the reviewer could verify, and what remains unknown.

## What a reviewer can safely conclude

- A usage value is present on a particular recorded span: **evidence**.
- A parent span's value equals the sum of visible descendants: evidence for this sample only, not proof of a universal producer rule.
- The value is called “total” or “agent usage” in a dashboard: a label or claim, not proof of attribution semantics.
- A missing agent name or sampled-out child makes per-agent allocation uncertain; do not invent a split.

Do not use an unresolved convention proposal as a billing, reimbursement, or quota rule. Verify the instrumentation version and aggregation semantics first. If they cannot be verified, retain the uncertainty for the human who owns that decision.

For a browser-local, evidence-labelled handback workflow, see [Traceplain](https://traceplain.zakgov.com/?utm_source=github&utm_medium=examples&utm_campaign=otel-nested-token-attribution-v1). Do not paste private traces into the guide or send them to a hosted service.
