# Traceplain agent-evidence examples

Synthetic activity records for testing how AI-agent work is reviewed by a human.

The examples cover three common export shapes:

- Codex `exec --json` events;
- Claude Code stream JSON messages;
- OpenTelemetry OTLP JSON with GenAI semantic attributes.

Open any sample in the free browser-local reviewer at [traceplain.zakgov.com](https://traceplain.zakgov.com/). The file is parsed on your device; Traceplain does not upload imported activity.

## Why these fixtures exist

An agent's final message is not proof that its work happened. A useful review keeps separate:

- **observed** — a concrete event or result appears in the supplied record;
- **reported** — the agent or runtime states something the record does not independently establish;
- **prevented** — the record says an action was blocked;
- **unknown** — the supplied evidence is incomplete or ambiguous;
- **needs human** — an outcome still needs an independent check.

Each fixture deliberately includes a gap between recorded activity and the broader conclusion a person might be tempted to draw. See [`expected/`](expected/) for a conservative reading.

## Try a sample

See the result immediately, without downloading anything:

- [Open the safe Codex example as a Traceplain handback](https://traceplain.zakgov.com/?demo=codex#review)
- [Open the safe Claude Code example as a Traceplain handback](https://traceplain.zakgov.com/?demo=claude#review)
- [Open the safe OpenTelemetry example as a Traceplain handback](https://traceplain.zakgov.com/?demo=otel#review)

To inspect or modify the underlying fixtures yourself:

1. Download or open a file in [`samples/`](samples/).
2. Open [Traceplain](https://traceplain.zakgov.com/#review).
3. Choose the file or paste its contents.
4. Compare the generated handback with the matching note in [`expected/`](expected/).

For capture instructions, use the [Codex JSONL guide](https://traceplain.zakgov.com/codex-exec-jsonl-viewer), [Claude Code JSONL guide](https://traceplain.zakgov.com/claude-code-jsonl-viewer), or [OpenTelemetry GenAI trace guide](https://traceplain.zakgov.com/opentelemetry-genai-trace-viewer).

## Generate a CI handback

The free [Traceplain Agent Review Action](https://github.com/harmonicfutures/traceplain-review-action) can turn a Codex JSONL or OTLP/JSON file into bounded Markdown inside a GitHub Actions runner:

```yaml
- name: Create agent handback
  id: traceplain
  uses: harmonicfutures/traceplain-review-action@v1
  with:
    path: artifacts/codex-run.jsonl
```

Safe mode is the default. It suppresses imported command text, paths, messages, model names, service names, and tool names, and it makes no outbound request to Traceplain. The resulting file stays on the runner unless the workflow explicitly uploads or publishes it.

## Safety boundary

These files are synthetic. Do not commit real agent transcripts, prompts, secrets, customer information, private paths, or proprietary tool results to this repository. Before sharing any activity record, review and redact it under your own authority and data-handling rules.

Traceplain can interpret only the events supplied. It does not establish that a log is complete, verify an external system of record, or turn ordinary telemetry into governed evidence.

## Contributing an example

Open an issue describing the public format and the review problem it exposes. Do not paste a real log. A contribution should be synthetic, minimal, and explicitly state what the record proves and what remains unknown.

No affiliation with OpenAI, Anthropic, or OpenTelemetry is implied. Product and project names belong to their respective owners.
