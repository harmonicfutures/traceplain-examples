# Reviewing externalized GenAI content without overstating evidence

**Campaign:** `otel-externalized-content-review-v1`  
**Source need:** [OpenTelemetry GenAI issue #610](https://github.com/open-telemetry/semantic-conventions-genai/issues/610), opened 20 October 2026. The discussion considers applying an upload hook to individual message parts (for example, large image, audio, or video content) and representing the part with a URI. This is an open discussion, not an accepted convention or settled exporter behavior.

## The review question

When a trace contains a reference to separately stored message content, what does the trace actually establish?

A recorded URI can establish that the supplied record contains a locator. It does not, by itself, establish that the object was fetched, that the reviewer can access it, that its bytes match the object used by a model, or that any downstream action was based on it. Keep those claims separate.

## An illustrative handback

The following is pseudocode for the evidence distinction, **not an OpenTelemetry schema**:

```text
recorded part: kind=image; representation=external-reference
reference: present in supplied record
retrieval event: not present in supplied record
content observed by reviewer: no
downstream effect: not established
```

A conservative summary is:

> The supplied trace contains an external reference for an image part. This record does not show a retrieval event or the referenced content. Whether the content was available to the model or reviewer, and whether it affected a later action, is unknown.

Do not turn the URI into “the image was reviewed” or “the model saw this image.” Those conclusions need separate, trustworthy evidence.

## Preserve five boundaries

| Question | What the supplied record may show | What remains unknown without more evidence |
| --- | --- | --- |
| Was a part recorded? | A content-part entry or reference appears in the trace. | Whether the exporter omitted other parts. |
| Was it inline or external? | The recorded representation is inline content, a locator, redacted, or absent—if that distinction is present. | Whether a locator still resolves or is authorized. |
| Was retrieval attempted? | A separate retrieval event and its recorded result, if included. | Successful retrieval when no result is recorded. |
| Is it the same content? | A trusted integrity reference may support a comparison. | Byte identity from a URI alone; a digest is only as trustworthy as its source and handling. |
| Did it affect an outcome? | A later event may record an action or result. | Causation or correctness merely because the events are adjacent. |

Missing retrieval status is **unknown**, not success or failure. A URI that was not dereferenced during review must stay a locator, not be presented as visible content.

## A practical review sequence

1. Record the event and part type exactly as supplied; distinguish inline, external-reference, redacted, and absent only when the input supports that distinction.
2. Keep retrieval separate from reference creation. Look for an independently recorded retrieval result before saying that a payload was available.
3. Treat provenance and integrity as separate questions. If an authorized system provides an integrity check, state what it compares and what it does not prove.
4. Do not automatically fetch arbitrary imported URIs as part of transcript parsing. If content needs to be inspected, use a separately authorized retrieval path and apply the relevant access and retention controls.
5. Review tool calls, decisions, and downstream effects as separate evidence. An external media reference does not prove what the model interpreted or what a later system did.

Avoid copying actual prompts, media, signed URLs, credentials, customer records, or private object paths into examples or handbacks. This guide uses no imported content and makes no claim that externalized content is a settled OpenTelemetry feature.

For a local human review of supported agent records, [Traceplain's OpenTelemetry reviewer](https://traceplain.zakgov.com/opentelemetry-genai-trace-viewer?utm_source=github&utm_medium=example&utm_campaign=otel-externalized-content-review-v1) can help separate recorded structure, reported claims, and unknowns. It does not dereference external content, verify trace completeness, or certify downstream effects.
