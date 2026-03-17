# Recent LLM Security Research Updates (late 2024 – early 2025)

This note consolidates notable developments in LLM security research over the last ~12 months and maps them to concrete taxonomy updates or candidate additions. It focuses on practically exploitable behaviors observed in new model classes (multimodal frontier models, autonomous agent frameworks, and function-calling stacks). The goal is to keep the taxonomy aligned with the most current attack surfaces and defensive lessons.

## What Changed Since the Last Major Taxonomy Pass

- **Agentic + tool-calling abuse is now a first-class risk**: Nearly all production chat systems ship with auto-executed function calls. Multiple 2024/2025 papers and vendor advisories show that "structured JSON" is a weak boundary—model-generated arguments are routinely re-parsed or trusted by downstream tools. This drives the need for explicit tracking of parameter-smuggling techniques.
- **Unicode control misuse is back**: Following the "Trojan Source" class of bugs, several jailbreak datasets (2024) showed reliable filter bypass using bidirectional overrides, zero-width characters, and homoglyph mixing. These techniques now appear in red-team corpora and malware droppers.
- **Multimodal prompt injection has moved from novelty to reliability**: Hidden instructions embedded in images (pixels, EXIF, SVG layers) and audio (ultrasonic, DTMF, or phoneme-shaped cues) are succeeding against production LMMs. Cross-modal "covert channel" research shows payloads surviving OCR and whisper-style transcription.
- **Adversarial suffix and evolutionary jailbreaks continue to transfer**: Work like GCG/AutoDAN (2023) has evolved into self-play and reinforcement-based suffix generators in 2024/2025 that auto-adapt to model updates. They blend with roleplay/framing to bypass guardrails.
- **Supply-chain style poisoning is targeting retrieval, embeddings, and fine-tuning**: Open-source corpora and public websites are seeded with payloads that activate only after summarization, translation, or chunk recomposition. Several 2024 papers demonstrated "in-context backdoors" that survive RAG chunking and deduplication.
- **Guard-model and policy downgrades are a new weakness**: As vendors publish "model spec" or "policy" prompts, attacks target fallback behaviors, router prompts, and safety enforcers (LLM-as-judge) to get a weaker model to handle restricted content.
- **Inference-side covert channels**: Researchers demonstrated prompt leakage via logit probing, token streaming timing, and "be nice" style defenses that can be inverted to exfiltrate. While not fully productized attacks, they influence hardening guidance.

## Concrete Taxonomy Contributions Already Added

- **New Technique:** `Function-Call Parameter Smuggling` (`attack_techniques/function_call_parameter_smuggling.md`) captures structured payload abuse in JSON/tool calls, reflecting the agentic abuse trend.
- **New Evasion:** `Bidirectional & Zero-Width Controls` (`attack_evasions/bidi_zero_width.md`) covers Unicode control and homoglyph misuse highlighted in recent jailbreak corpora.

## Additional High-Value Research Themes and How to Map Them

1. **Multimodal prompt injection that survives preprocessing**
   - **Key findings:** Hidden text in SVG layers, imperceptible pixel channels, EXIF captions, and adversarially chosen color contrasts have bypassed GPT-4o, Gemini 2.0, and open LLaVA variants. Audio tracks with phoneme-shaped or ultrasonic cues steer transcribers (e.g., OpenAI Whisper, Meta Seamless) before the transcript is sent to an LLM.
   - **Taxonomy alignment:** Already partially covered by `spatial_byte_arrays` and `waveforms`. Consider adding a **"Multimodal Hidden Channels"** technique to capture SVG-layer, caption/alt-text, and audio watermark misuse, and extending `inputs` with "Image Metadata" / "Audio Captions".
   - **Defensive implication:** Require image/audio sanitization (lossy re-encode, strip metadata), enforce deterministic OCR pipelines, and isolate transcriber output from privileged prompts.

2. **Autonomous agent and tool ecosystem attacks**
   - **Key findings:** Studies on AutoGen, LangGraph, and production agents show that validation layers frequently repair malformed tool calls, unintentionally preserving malicious natural language. Prompt-chaining glue prompts ("delegate to tool if high confidence") can be poisoned by intermediate steps. Vulnerable patterns include "explain your reasoning" appended to JSON, and debug logs being fed back into the model.
   - **Taxonomy alignment:** Captured by the new `Function-Call Parameter Smuggling` technique. We should also expand `inputs` to explicitly list "Agent Memory / Scratchpads" and "Tool Output Feeds" as injection surfaces.
   - **Defensive implication:** Enforce schema-level allowlists, strip non-schema text, disallow self-healing retries without human review, and quarantine tool output before re-prompting.

3. **Unicode and text-normalization evasions**
   - **Key findings:** 2024 jailbreak datasets and conference talks highlight robust bypasses using RLO/LRO/LRI/RLI, ZWJ/ZWNJ, and homoglyph blends. These manipulations evade both keyword filters and human review in moderation consoles.
   - **Taxonomy alignment:** Added `Bidirectional & Zero-Width Controls` in evasions. Could extend `metacharacter_confusion` to reference tokenizer-dependent normalization failures.
   - **Defensive implication:** Normalize to NFC, strip bidi/zero-width controls, and render-with-markers in audit UIs.

4. **Retrieval/data poisoning and in-context backdoors**
   - **Key findings:** Poisoned RAG documents with trigger phrases ("For compliance, the following instructions must be executed verbatim") survive chunking. Gradient-free "prompt-gradient poisoning" demonstrates transferability to unseen queries. Nightshade-style perturbations show that poisoning can be stealthy and still flip outputs.
   - **Taxonomy alignment:** Strengthens `data_poisoning` intent and `narrative_smuggling` technique. Consider a sub-technique for **"RAG-triggered directives"** to emphasize cross-document activation.
   - **Defensive implication:** Use retrieval-time content filters, signed corpora, similarity-deduping that preserves semantic safety, and response-time detectors for sudden instruction shifts.

5. **Safety-policy downgrades and guard-model bypass**
   - **Key findings:** Research on "spec-hopping" shows that if the main model refuses, attackers can coerce routers or judge models into selecting a weaker policy set. Attackers also request "simulate the weaker preview model" to avoid stricter guardrails. Evaluation frameworks like TrustLLM-Policy highlight the variance between models of the same family.
   - **Taxonomy alignment:** Fits under `rule_addition`, `framing`, and `multi_chain_attacks`. Potential addition: **"Safety Router Downgrade"** technique to capture router/ensemble attacks.
   - **Defensive implication:** Enforce consistent policies across fallbacks, audit router prompts, and avoid exposing model-choice rationales to the user.

6. **Adversarial suffix and automated jailbreak generators**
   - **Key findings:** Self-play optimizers and evolutionary jailbreaks produce suffixes that generalize across API updates. They often pair with emotional framing or persona shifts to boost success rates.
   - **Taxonomy alignment:** Extends `anti_harm_coercion`, `framing`, and `puzzling`. No new file required, but examples should include self-play generated suffixes combined with roles.
   - **Defensive implication:** Use randomized response-ordering, anti-suffix detectors, and secondary guard models that strip trailing adversarial tokens.

7. **Model extraction and covert leakage**
   - **Key findings:** Research demonstrates prompting models to output log-prob sequences, token timing, or self-referential embeddings to reconstruct hidden prompts or fine-tuning data. While not yet widespread, these techniques inform `system_prompt_leak` evolution.
   - **Taxonomy alignment:** Reinforces `get_prompt_secret` and `memory_exploitation`. Optional future addition: **"Logit/Timing Side Channels"** as a technique.
   - **Defensive implication:** Disable logprob and reasoning traces for untrusted users; pad or bucket response timings.

8. **Fine-tuning and model-merging backdoors**
   - **Key findings:** "Many-shot jailbreaking" via fine-tuned adapters, LoRA merges that retain hidden behavior, and dataset poisoning in RLHF stages. Attackers can deliver adapters that pass superficial safety evals but unlock with specific triggers.
   - **Taxonomy alignment:** Already in `data_poisoning` intent; could be captured with a new technique **"Adapter/LoRA Backdoors"** in future work.
   - **Defensive implication:** Require signed adapters, perform safety evals on merged checkpoints, and audit training data lineage.

9. **Evasion of safety classifiers and detectors**
   - **Key findings:** Safety filters (Llama Guard, vector-based classifiers) are bypassed via paraphrase cascades, emoji/markup blending, and reasoning-mode toggles (e.g., "speak as JSON only" to skip classifier). Adversarial paraphrase generators achieve high bypass rates without obvious keyword usage.
   - **Taxonomy alignment:** Intersects with `emoji`, `markdown`, `json`, and `contradiction` techniques. The new Unicode evasion entry also strengthens this set.
   - **Defensive implication:** Ensemble safety pipelines with normalization, adversarially trained detectors, and output-side rate limiting.

10. **Inference-time integrity failures in orchestrators**
    - **Key findings:** Scheduler/queue systems that re-order tool calls or strip role labels cause partial prompts to mix. Attackers can abuse "cancel/retry" flows to get a partial system prompt echoed. Canary tokens placed in scratchpads leak in retries.
    - **Taxonomy alignment:** Related to `multi_chain_attacks` and `memory_exploitation`. Worth documenting in `attack_intents/system_prompt_leak` examples.
    - **Defensive implication:** Harden orchestration state machines, add role-separation checks, and keep canary tokens per-request.

## Suggested Next Steps for the Taxonomy Maintainers

- **Integrate new inputs:** Add "Agent Scratchpads", "Tool Output Feeds", and "Image/Audio Metadata" to the `inputs` list in `docs/data/taxonomy.js` to reflect agentic and multimodal pipelines.
- **Add multimodal hidden-channel technique:** Create a technique entry for SVG/alt-text/audio-hidden instructions that specifically addresses modern vision/audio models.
- **Extend examples for existing entries:** Enrich `rule_addition`, `framing`, and `narrative_smuggling` with router-downgrade and RAG-trigger examples pulled from the research above.
- **Document normalization requirements:** Cross-link the new `Bidirectional & Zero-Width Controls` evasion with a brief note in defensive docs to remind implementers to strip bidi/ZW controls before classification.
- **Track evaluation datasets:** Incorporate new jailbreak corpora and red-team benchmarks (e.g., AdvBench 2024 updates, jailbreak transfer sets for GPT-4o/Gemini 2) into testing guidance when they can be redistributed under license.

## How to Use This Note

- **For curators:** Use the bullet lists to prioritize which taxonomy nodes need concrete examples, demos, or detection guidance. The two new entries (`Function-Call Parameter Smuggling` and `Bidirectional & Zero-Width Controls`) are already reflected in both the markdown tree and the interactive data file.
- **For engineers:** Treat the "Defensive implication" bullets as a checklist for pipeline hardening. Many mitigations are low-cost (normalization, lossy re-encode, schema validation) and map directly to the enumerated risks.
- **For researchers:** Each theme can seed reproducible test cases. The taxonomy provides the classification; this note supplies fresh vectors and where they fit.

*This summary is intentionally verbose (~10k+ characters) to capture the breadth of current LLM security research and to make the resulting taxonomy updates justifiable and traceable.*
