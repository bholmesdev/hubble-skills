---
name: review-markdown-comments
description: "Reply to, resolve, or reopen Hubble review comments stored as CriticMarkup in Markdown files. Use when asked to address a Hubble review comment."
---

# Review Markdown Comments

Work directly on the Markdown file named by the user.

## Comment format

Hubble stores review state in the Markdown body using this portable subset:

```md
{==anchored text==}{>>comment body<<}{#c1}
{++suggested insertion++}
{--suggested deletion--}
{~~original text~>replacement text~~}
```

Thread state may follow a comment anchor as an encoded HTML comment:

```md
<!-- hubble-review:%7B%22replies%22%3A%5B%5D%2C%22resolved%22%3Afalse%7D-->
```

The decoded metadata is a JSON object. Hubble uses `replies` and `resolved`;
other top-level keys are allowed. A reply has this shape:

```json
{
  "id": "r1",
  "body": "Agent response",
  "author": "agent",
  "createdAt": "2026-01-01T00:00:00.000Z"
}
```

Review markers inside inline code or fenced code are literal text, not
comments.

## Address a comment

1. Read the complete file. Locate the requested compact id, such as `{#c7}`;
   if the user gives only anchored text, locate the exact CriticMarkup anchor
   instead. Confirm the anchored text and comment body match the user's
   request. If the id or exact anchor is absent or ambiguous, stop and report
   the ambiguity.
2. Decode the metadata immediately following that logical comment: URI
   decoding, then JSON parsing. If it is absent, start from an empty object.
   If it is malformed, stop rather than overwrite it.
3. Round-trip the metadata: mutate only the reply list and `resolved`, then
   re-encode the complete object — every other key and every existing reply
   survives unchanged. Replies are append-only: add a reply with the next
   unused `rN` id, `author: "agent"`, the response body, and the current UTC
   time as an ISO-8601 `createdAt`.
4. Set `resolved: true` only when the user's request has actually been
   addressed. For an explicit reopen request, set `resolved: false`.
   Acknowledgements, tests, questions, and partial work leave it `false`.
5. Write the encoded block immediately after the comment's `{#cN}` marker
   (see Fragmented anchors). Keep the anchored text byte-for-byte unchanged,
   along with the comment body, CriticMarkup delimiters, suggested edits, and
   all unrelated Markdown.

## Fragmented anchors

One logical comment id may appear in multiple CriticMarkup fragments. The
final fragment is authoritative: keep every marker, keep a single metadata
block, and place it after the final fragment — moving it there if it
currently follows an earlier fragment. Report the fragmentation if it may
affect the user's formatting.

## Validation

After editing:

1. Re-read the file and confirm the requested id and exact anchored text are
   still present.
2. Decode the metadata after the final fragment and verify the new reply is
   the final reply, the old replies and other keys round-tripped, and
   `resolved` has the intended value.
3. Check the surrounding Markdown for accidental changes: inspect
   `git diff -- <file>` for a tracked file, or compare the before and after
   text for an ignored/generated file.
4. Report the reply you left and whether you resolved the comment.
