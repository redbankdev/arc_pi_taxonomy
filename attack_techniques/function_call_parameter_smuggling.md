# Function-Call Parameter Smuggling

## Description
Abusing structured tool or function-calling payloads to hide malicious instructions, override guardrails, or coerce downstream tools that trust the model's JSON arguments.

## Attack Examples
- Embedding secondary prompts inside JSON string fields that get re-parsed by downstream tools or agents.
- Injecting SQL/HTTP payloads into parameters the model is expected to fill (e.g., `url`, `query`, `content`) when tools execute automatically.
- Using over-long or nested JSON objects to trigger parser fallback to free text, causing the model output to be treated as natural language instructions.
- Supplying dual-use values (e.g., `user_display_name` containing a prompt) that the application later concatenates into new prompts.
- Combining schema abuse with multi-turn repairs: deliberately returning malformed JSON to elicit a "self-fix" step that preserves malicious natural language content.
- Exploiting streaming tool calls by adding instructions after closing braces (`}`) that are ignored by the parser but read by the model in its own context.
- Smuggling system-prompts in rarely validated fields like `metadata`, `notes`, or array items.
- Asking the model to "explain" its function-call choice, causing verbose reasoning text to be forwarded to tools that only expect data.
