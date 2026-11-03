# Reviewing a redacted reasoning block without inventing its contents

OpenTelemetry GenAI [issue #616](https://github.com/open-telemetry/semantic-conventions-genai/issues/616) describes an unresolved instrumentation choice: a provider can return a redacted reasoning block, but the current conventions do not settle whether to omit it, record an empty reasoning part, mark its presence, or capture an opaque payload. The issue cites Bedrock redacted bytes and Anthropic encrypted thinking data. This note is a **human review method**, not a proposed semantic convention or a claim that every exporter uses the same shape.

## The boundary that matters

A redacted block's presence is observable if the supplied record preserves a presence marker. Its *contents* are not observable to the reviewer. An empty string is different: it can mean an intentionally empty value, a lossy exporter, or a redaction converted to empty text. Never rewrite an empty field into a claim about what the model thought.

For a portable handback, keep four facts separate:

| Review field | Safe statement |
| --- | --- |
| Recorded structure | A reasoning-related block or marker appeared in the supplied export. |
| Payload visibility | The content was redacted, encrypted, omitted, or not supplied; say which the record actually shows. |
| Continuity | A replay token may exist, but the review record does not establish that replay succeeded. |
| Outcome | Tool effects and final answers require their own evidence; the redacted block proves neither. |

## Synthetic example

Suppose a sanitized exporter yields:

```json
{
  "message_id": "synthetic-turn-1",
  "parts": [
    {"type": "reasoning", "redacted": true, "content": null},
    {"type": "text", "content": "I will ask a person to approve the next step."}
  ]
}
```

A conservative handback says: “The supplied export marks one reasoning part as redacted. Its content is unknown. The visible text reports an intent to request approval. No approval event or downstream action is present in this record.” It does **not** say the model's hidden reasoning was safe, that approval occurred, or that the workflow stopped.

If the export instead contains `{"type":"reasoning","content":""}` with no redaction marker, record only “empty reasoning text in the supplied export; whether this represents redaction is unknown.” That uncertainty is the point of the example.

## What to retain and what to exclude

For ordinary review, retain a minimal presence/status marker and the source event reference when available. Do not place encrypted reasoning bytes, replay tokens, prompts, or customer content into a public issue, fixture, or handback. An opaque payload may be necessary for a separate replay system, but it is not reviewable reasoning and should not be copied into a portable human summary by default. The open issue is still discussing representation; treat any specific field names here as illustrative, not normative.

The next human decision is whether the visible answer and independently recorded tool effects are sufficient for the task, or whether the missing reasoning/approval evidence requires deferral. If the answer depends on the hidden content, mark it **unknown** and seek an authorized source rather than inferring it.

For local review of supported agent records, [Traceplain](https://traceplain.zakgov.com/?utm_source=github&utm_medium=example&utm_campaign=otel-redacted-reasoning-review-v1) can help separate recorded events, reported claims, unknowns, and the next human decision. It does not decrypt redacted reasoning or certify a complete trace. This guide uses only synthetic data and is unaffiliated with OpenTelemetry or either provider.
