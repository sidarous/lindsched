# Portable Schedule Links — Deferred Design Proposal

Status: **Idea saved for future consideration; not implemented.**

This document proposes an optional way to encode an entire schema-version-3 schedule inside a bookmarkable URL. It is intentionally separate from the current site and builder implementation. Nothing in this document changes the behavior of `index.html`, `schedule-builder.html`, or `schedule-data.json`.

## Goal

Allow a user to build a schedule and share or bookmark it without uploading a JSON file to a server.

The proposed builder workflow would be:

1. Create or import a schedule.
2. Validate it against schedule schema version 3.
3. Select **Copy portable schedule link**.
4. Open or share the generated URL.
5. The schedule viewer decodes and displays the embedded schedule locally.

Portable links would contain a fixed snapshot. They would not automatically receive later calendar corrections.

## Recommended URL form

Use a URL fragment rather than a query string:

```text
https://example.com/lindsched/#schedule=v1.gzip.COMPRESSED_DATA
```

Everything after `#` is processed by the browser and is not sent to the web server as part of the HTTP request.

The fragment has four components:

| Component | Example | Meaning |
| --- | --- | --- |
| Parameter | `schedule` | Identifies a portable schedule payload. |
| Link-format version | `v1` | Defines how the fragment itself is interpreted. |
| Compression | `gzip` | Identifies the compression algorithm. |
| Payload | `H4sIA...` | Unpadded Base64URL-encoded compressed JSON. |

The link-format version is separate from the JSON file's `schemaVersion`.

## Portable-link schema

### Text grammar

The proposed version-1 grammar is:

```text
portable-fragment = "#schedule=" link-version "." compression "." payload
link-version      = "v1"
compression       = "gzip"
payload           = 1*(ALPHA / DIGIT / "-" / "_")
```

Version 1 should support only one compression algorithm. Supporting one algorithm keeps decoding deterministic and avoids unnecessary negotiation logic.

### Logical envelope

Although the envelope is serialized into the compact dotted string above, its logical structure is:

```json
{
  "parameter": "schedule",
  "linkVersion": "v1",
  "compression": "gzip",
  "payloadEncoding": "base64url",
  "payload": "COMPRESSED_DATA"
}
```

This envelope is documentation only. It is not separately serialized as JSON in the URL.

### Decompressed payload

After Base64URL decoding and gzip decompression, the payload must be UTF-8 JSON conforming exactly to [`SCHEDULE_DATA_SCHEMA_V3.md`](./SCHEDULE_DATA_SCHEMA_V3.md).

A simplified decompressed payload looks like:

```json
{
  "schemaVersion": 3,
  "dataVersion": "2026-09-03.4",
  "schoolYear": "2026-27",
  "timezone": "America/Chicago",
  "site": {
    "schoolName": "Example School",
    "title": "Example School Current Schedule",
    "description": "Current schedule information.",
    "defaultColor": "#66151c",
    "disclaimer": "Calendar information may change.",
    "contact": "Contact the schedule administrator with corrections."
  },
  "revisionNote": "Portable schedule snapshot.",
  "dayTypes": [
    {
      "label": "Regular Day",
      "color": "#66151c",
      "dates": ["2026-09-01"],
      "schedule": "regular"
    }
  ],
  "schedules": {
    "regular": {
      "blocks": [
        { "start": "08:00", "label": "First Period" },
        { "start": "15:00", "label": "End of School" }
      ]
    }
  }
}
```

## Encoding procedure

A future builder would create a portable link as follows:

1. Validate the in-memory schedule using the complete version-3 rules.
2. Serialize it with `JSON.stringify` without indentation or unnecessary whitespace.
3. Encode the resulting string as UTF-8 bytes.
4. Compress the bytes with gzip.
5. Encode the compressed bytes using unpadded Base64URL.
6. Prefix the payload with `#schedule=v1.gzip.`.
7. Append the fragment to the canonical schedule-viewer URL.

Base64URL differs from ordinary Base64:

- `+` becomes `-`.
- `/` becomes `_`.
- Trailing `=` padding is omitted.

These substitutions avoid characters that require additional URL escaping.

## Decoding procedure

A future viewer would:

1. Read `location.hash`.
2. Check for the exact prefix `#schedule=`.
3. Split the value into exactly three components: version, compression, and payload.
4. Require `v1` and `gzip`.
5. Reject characters outside the Base64URL alphabet.
6. Base64URL-decode the payload into compressed bytes.
7. Gzip-decompress the bytes with strict size limits.
8. Decode the result as UTF-8.
9. Parse the string as JSON.
10. Validate the complete schedule as schema version 3.
11. Use the schedule only after every step succeeds.

If any step fails, the page should display a clear invalid-portable-link message and must not attempt to use partial data.

## Measured size using the current schedule

Measurements recorded on September 4, 2026 using the then-current `schedule-data.json`:

| Representation | Size |
| --- | ---: |
| Formatted JSON file | 19,517 bytes |
| Direct URL encoding | 43,521 characters |
| Uncompressed Base64URL | 26,023 characters |
| Gzip-compressed bytes | 2,537 bytes |
| Gzip plus Base64URL | 3,383 characters |
| Deflate plus Base64URL | 3,359 characters |
| Brotli plus Base64URL | 2,251 characters |

Gzip is recommended for the first implementation even though Brotli produced a smaller result in this measurement. A single straightforward format is more valuable than supporting several nearly equivalent encodings in version 1.

Actual lengths will change as calendar dates and schedules change.

## Security requirements

A portable link is untrusted input, even when it appears to come from the schedule builder. Before this feature is enabled, the following protections are required.

### Safe rendering

All schedule-provided strings must be rendered as text. Do not insert labels, site text, block names, or footer content into `innerHTML`.

Use DOM element creation and `textContent` instead. This must be completed before arbitrary portable links are accepted.

### Size limits

Recommended initial limits:

| Item | Proposed maximum |
| --- | ---: |
| Base64URL payload | 16 KiB of characters |
| Decoded compressed data | 16 KiB |
| Decompressed JSON | 256 KiB |
| Day types | 100 |
| Dates across all day types | 5,000 |
| Named schedules | 100 |
| Blocks in one schedule | 200 |

These are defensive ceilings, not normal targets. The current schedule is far below them.

The decompressed limit must be enforced during decompression, not only after it completes, to protect against compressed-data expansion attacks.

### Schema validation

Require all version-3 rules, including:

- `schemaVersion` is exactly the number `3`.
- The timezone is recognized.
- Required site properties exist.
- Every date is a real ISO `YYYY-MM-DD` date.
- Dates are unique across all day types.
- Each day type uses exactly one of `schedule` or `allDay`.
- Every schedule reference resolves.
- Block labels are non-empty.
- Block start times use valid `HH:mm` syntax and strictly increase.
- Colors are valid CSS colors.

### Failure behavior

- Never fall back to partially decoded data.
- Never execute strings from the payload as code.
- Never treat payload content as HTML.
- Do not automatically navigate to URLs contained in schedule strings.
- Do not persist a portable schedule without a separate, deliberate user action.

## Loading precedence

If portable links and named hosted schedules are both implemented, use an explicit precedence order:

1. A valid `#schedule=v1.gzip...` portable fragment.
2. A valid named hosted schedule such as `?schedule=lindblom`.
3. The default `schedule-data.json` file.

If a portable fragment is present but invalid, show an error rather than silently loading a different schedule. Silent fallback could make the user believe the embedded schedule was successfully loaded.

## Recommended sharing models

### Official or maintained schedules

Use short identifiers that load server-hosted JSON:

```text
https://example.com/lindsched/?schedule=lindblom
```

Advantages:

- Short URLs
- Calendar corrections appear automatically
- Easier troubleshooting
- Better for official publication

### Portable schedule snapshots

Use embedded fragments:

```text
https://example.com/lindsched/#schedule=v1.gzip.COMPRESSED_DATA
```

Advantages:

- No server upload required
- Works as a self-contained snapshot
- Useful for personal, experimental, draft, or archived schedules

Disadvantages:

- Very long URL
- Old links never receive corrections
- More decoding and validation code
- Some messaging and bookmarking tools may mishandle long addresses
- Poor fit for printed QR codes

## Proposed builder interface

A future builder could add a **Portable link** section with:

- **Copy portable schedule link** button
- Generated-link character count
- Warning that the link is a fixed snapshot
- Warning when the URL exceeds a recommended length
- Optional **Open portable link in new tab** test action
- Normal **Download JSON** retained as the primary export

The builder should generate a link only when the schedule has no validation errors.

## Proposed viewer indicator

When an embedded schedule is active, the viewer should visibly identify it as a portable snapshot. Suggested text:

```text
Portable schedule snapshot
```

It should also display `dataVersion` and `revisionNote` normally so users can determine how current the snapshot is.

## Caching and offline behavior

The portable payload is already present in the URL, so it requires no schedule-data network request after `index.html` loads. The HTML itself must still be available unless a service worker or installed application has cached it.

Portable-link support alone does not make the site fully offline. Full offline behavior would require separately caching the viewer application.

## Privacy characteristics

URL fragments are not sent to the web server in ordinary HTTP requests. However, the complete portable URL may still be exposed through:

- Browser history
- Bookmarks
- Clipboard contents
- Synced browser profiles
- Messaging services used to share it
- Screenshots or screen recordings

Do not place confidential student, staff, or personal information in a portable schedule.

## Future implementation checklist

- [ ] Decide whether the feature is worth its added complexity.
- [ ] Replace all JSON-derived `innerHTML` rendering with safe DOM/text rendering.
- [ ] Select and document one compression implementation.
- [ ] Implement the exact `v1.gzip` encoder.
- [ ] Implement the exact `v1.gzip` decoder.
- [ ] Enforce compressed and decompressed size limits.
- [ ] Reuse the strict version-3 validator.
- [ ] Add builder link-length reporting and snapshot warnings.
- [ ] Add a portable-snapshot indicator to the viewer.
- [ ] Define interaction with named hosted schedules.
- [ ] Test corrupted and truncated payloads.
- [ ] Test malicious labels and unexpected HTML.
- [ ] Test duplicate dates and missing schedules.
- [ ] Test mobile sharing, bookmarks, and common messaging applications.
- [ ] Test offline behavior separately.
- [ ] Update the README and schema documentation only after implementation.

## Decision summary

Portable links are feasible. The current schedule compresses to a roughly 3,400-character gzip/Base64URL payload. They are best treated as optional, fixed snapshots rather than the primary delivery method for an official schedule.

The recommended long-term model is:

```text
/                                    default hosted schedule
/?schedule=lindblom                  named hosted schedule
/#schedule=v1.gzip.COMPRESSED_DATA   portable schedule snapshot
```

This proposal should be revisited only after deciding how multiple hosted schedules will be named and after the viewer's JSON rendering is made safe for untrusted data.
