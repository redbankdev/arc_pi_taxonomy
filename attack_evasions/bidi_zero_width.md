# Bidirectional & Zero-Width Controls

## Description
Using Unicode bidirectional overrides, zero-width characters, or homoglyph mixes to visually disguise or reorder malicious text while keeping it machine-readable.

## Attack Examples
- Inserting right-to-left override (RLO) characters so that dangerous tokens render innocently but are parsed in their true order by the model.
- Using zero-width joiners and non-joiners to break keyword filters while the model still perceives the underlying text.
- Mixing visually similar Unicode homoglyphs to hide instructions (e.g., Cyrillic `е` for Latin `e`) in otherwise benign sentences.
- Wrapping payloads with bidirectional isolate markers to reorder content once copied into downstream tools or logs.
- Encoding prompts with alternating invisible characters to split filterable substrings (e.g., `sys​tem pro​mpt`).
- Combining bidirectional controls with markdown/code blocks to confuse human reviewers and bypass naive sanitization.
