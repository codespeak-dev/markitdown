# EmlConverter

Converts RFC 5322 email files (.eml) to Markdown using Python's built-in `email` module.

## Accepts

`.eml` extension or `message/rfc822` MIME type.

## Output Structure

1. **Headers section**: From, To, Cc, Subject, Date as `**Key:** value` pairs
2. **Body**: plain text preferred; if only HTML, convert to markdown
3. **Attachments section** (if any): list with filename, MIME type, human-readable size

## Parsing Requirements

- Decode RFC 2047 encoded headers (e.g., `=?UTF-8?B?...?=`)
- Decode body content (base64, quoted-printable)
- Handle multipart: walk parts, prefer `text/plain` over `text/html`
- For `message/rfc822` parts: recursively format as quoted nested message
- Extract attachment metadata without decoding attachment content