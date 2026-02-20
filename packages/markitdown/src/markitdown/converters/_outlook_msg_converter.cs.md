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
- **Subject**: Stream `__substg1.0_0037001F`

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
**Subject:** [email subject]

## Content

[email body content]
```

Only non-empty header fields are included in the output. The document title is set to the email subject when available.

## Error Handling

- Missing dependencies trigger `MissingDependencyException` with preserved stack trace
- Stream extraction failures return `None` for individual fields
- Text decoding errors are handled gracefully with fallback encodings
- File parsing exceptions during acceptance testing are silently ignored
- OLE file resources are properly closed after processing