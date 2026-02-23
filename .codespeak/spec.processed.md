# Outlook MSG Converter Specification

The Outlook MSG converter transforms Microsoft Outlook `.msg` email files into structured markdown documents by parsing the OLE (Object Linking and Embedding) file format and extracting email metadata and content.

## Dependencies

Requires the `olefile` package for parsing OLE file structures. If the dependency is unavailable, converter operations raise `MissingDependencyException` with appropriate error context.

## File Acceptance Criteria

The converter accepts files based on a multi-tier validation approach:

1. **File Extension**: Files with `.msg` extension
2. **MIME Type**: Files with MIME types starting with `application/vnd.ms-outlook`
3. **OLE Format Validation**: Files that pass `olefile.isOleFile()` check
4. **Outlook-Specific Structure**: Files containing both `__properties_version1.0` and `__recip_version1.0_#00000000` streams in their table of contents

Validation preserves the original file stream position regardless of success or failure.

## Conversion Process

### Email Metadata Extraction

Extracts standard email headers from specific OLE streams:
- **From**: Stream `__substg1.0_0C1F001F`
- **To**: Stream `__substg1.0_0E04001F`
- **Cc**: Stream `__substg1.0_0E03001F`
- **Bcc**: Stream `__substg1.0_0E02001F`
- **Subject**: Stream `__substg1.0_0037001F`
- **Date**: Stream `__substg1.0_0039001F` (client submit time, human-readable format)

### Attachments

Enumerates attachment sub-storages (`__attach_version1.0_#XXXXXXXX`) and extracts metadata for each:
- **Filename**: Stream `__substg1.0_3707001F` (display name), falling back to `__substg1.0_3704001F` (filename)
- **Size**: Determined from the length of the attachment data stream `__substg1.0_37010102`

Attachment content is not decoded or included — only metadata is listed.

### Email Body Content

Extracts message body from stream `__substg1.0_1000001F`.

### Text Encoding

Applies cascading encoding detection for all extracted text:
1. UTF-16 Little Endian (primary encoding for MSG files)
2. UTF-8 (fallback)
3. UTF-8 with error ignoring (final fallback)

## Output Format

Generates markdown with the following structure:

```
# Email Message

**From:** [sender address]
**To:** [recipient address]
**Cc:** [cc addresses]
**Bcc:** [bcc addresses]
**Subject:** [email subject]
**Date:** [sent date/time]

## Content

[email body content]

## Attachments

- [filename] (size in human-readable format)
- ...
```

Only non-empty header fields are included in the output. The attachments section is omitted if there are no attachments. The document title is set to the email subject when available.

## Error Handling

- Missing dependencies trigger `MissingDependencyException` with preserved stack trace
- Stream extraction failures return `None` for individual fields
- Text decoding errors are handled gracefully with fallback encodings
- File parsing exceptions during acceptance testing are silently ignored
- OLE file resources are properly closed after processing

## Test inputs

- test_files/unicode.msg is expected to have a TIF attachment