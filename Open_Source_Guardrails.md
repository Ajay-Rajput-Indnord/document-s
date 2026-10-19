# Open-Source Guardrails

Evaluation of 15 GitHub repositories and recommended architecture

Date: October 19, 2026 | Audience: senior review

## 1. Executive summary

We are building a product with a chatbot inside it that reviews and generates code in several programming languages, and the chatbot needs protection on both sides of the model. Before the model: detect harmful prompts, jailbreak and prompt-injection attempts, and personal data (PII). After the model: detect hallucination, harmful content, PII leakage and insecure generated code. The requirements are open-source code, self-hosting (no third-party vendor APIs), and compatibility with whichever model the chatbot uses.

We reviewed 15 open-source repositories. No single repository covers every need, so the proposal is a layered stack, not one tool.

## 2. Proposed architecture

**Input path (before the model)**

1. PII masking with Presidio, and a secrets scan of any submitted code (for example with Gitleaks).
2. Jailbreak and injection check with Prompt Guard.
3. Harmful-content check with Llama Guard (or Qwen3Guard for many languages).
4. NeMo Guardrails runs these as input rails and decides to allow, change or refuse.
5. Allowed request goes to the chatbot's model.

**Output path (after the model)**

1. Hallucination and groundedness check with Granite Guardian (optionally HHEM) against the source documents used for the answer.
2. Harmful-content check with Llama Guard.
3. Insecure-code scan with CodeShield on every generated code block.
4. PII scan with Presidio.
5. NeMo output rails block, rewrite or pass the answer.

**Before release and in CI (offline)**

- garak and promptfoo attack the system; Ragas and DeepEval score faithfulness and hallucination.

## 3. Repository reviews

### A. Pipeline and gateway tools

#### 1. NVIDIA NeMo Guardrails

Link: <https://github.com/NVIDIA-NeMo/Guardrails>

- **Deployment:** Installable Python framework and self-hostable. **API requirement:** No external API is mandatory; it can use vLLM, Ollama, Hugging Face Transformers or a hosted model API.

- **License and status:** Apache 2.0. One August 2026 review lists v0.23.0 released July 1, 2026. The repo moved to the NVIDIA-NeMo organization; the old NVIDIA/NeMo-Guardrails address redirects.
- **Why we chose it:** It is the most structured open framework, with rails for input, dialog, retrieval, execution and output. Its dialog rails can keep the chatbot to code review and generation, and it works with several model providers, so the guardrails stay the same if the model changes.
- **Input use:** Input rails run the jailbreak check and call Presidio, Prompt Guard and Llama Guard before the model sees the message.
- **Output use:** Output rails run the hallucination self-check and call Llama Guard and Presidio on the answer.
- **Watch-outs:** Policies are written in Colang, which has a learning curve. Some rails call an LLM to decide, which adds latency and token cost.

#### 2. Guardrails AI

Link: <https://github.com/guardrails-ai/guardrails>

- **Deployment:** Installable Python library and self-hostable. **API requirement:** Optional; some validators or Guardrails Hub features may call a model API or require a Hub key.

- **License and status:** Apache 2.0. One review lists v0.10.2 released June 4, 2026.
- **Why we chose it:** A hub of ready-made validators (toxicity, PII, profanity, hallucination, bias) that you combine into guards. It adds one check at a time and is strong at validating structured output.
- **Input use:** A guard with PII and jailbreak validators wraps the user message.
- **Output use:** A guard with hallucination and toxicity validators wraps the answer.
- **Watch-outs:** Protection depends on which validators we pick, and some call a model, so read each one. One comparison site says the Hub needs a free API key; confirm this fits our no-vendor rule. We should choose one pipeline framework (NeMo or this), not both.

#### 3. Portkey AI Gateway

Link: <https://github.com/Portkey-AI/gateway>

- **Deployment:** Installable/self-hostable gateway. **API requirement:** Usually routes requests to OpenAI, Anthropic or other model APIs; partner guardrails may call external services.

- **License and status:** Sources disagree (MIT in one, Apache 2.0 in another); read the LICENSE file. Palo Alto Networks announced its intent to acquire Portkey on April 30, 2026; one source says it closed May 29, 2026.
- **Why we chose it:** It is a self-hosted gateway with built-in guardrail checks, and it matches the guardrails catalogue already used in this project.
- **Input and output use:** Route chatbot traffic through the gateway and enable guardrail checks before the request and after the response.
- **Watch-outs:** Ownership change affects the roadmap. Some checks are partner integrations that call outside services, which break our self-hosted rule, so use only checks that run on our side.

#### 4. LiteLLM

Link: <https://github.com/BerriAI/litellm>

- **Deployment:** Installable/self-hostable proxy. **API requirement:** Optional; it can route to vLLM or Ollama internally, or to hosted model APIs such as OpenAI and Anthropic.

- **License and status:** MIT according to its own website, which was checked on October 12, 2026.
- **Why we chose it:** A self-hosted gateway with one OpenAI-style interface to 100+ providers. Guardrails sit on the gateway, so they apply whichever model is behind it.
- **Input and output use:** Presidio PII masking and prompt-injection guardrails run on requests and responses passing through the proxy.
- **Watch-outs:** A competing project reports open bugs in the built-in Presidio guardrail and says it does not scan tool-call arguments; we have not confirmed this. A comparison article says per-key guardrails are in the enterprise tier. Confirm which guardrail features are in the free version.

#### 5. Bifrost

Link: <https://github.com/maximhq/bifrost>

- **Deployment:** Installable/self-hostable gateway, commonly deployed with Docker. **API requirement:** Usually routes to provider APIs; private endpoints may be possible.

- **License and status:** Apache 2.0. Built by Maxim AI, a vendor.
- **Why we chose it:** A fast Go-based gateway that lists several providers and advertises PII and content-moderation guardrails.
- **Input and output use:** Same gateway pattern as LiteLLM.
- **Watch-outs:** Vendor pages disagree on whether guardrails are free or enterprise-only. This is our lowest-priority option.

### B. Classifier and guard models

#### 6. Meta PurpleLlama (Llama Guard, Prompt Guard, CodeShield)

Link: <https://github.com/meta-llama/PurpleLlama>

- **Deployment:** Installable local tools and gated model weights. **API requirement:** No external API is mandatory; Prompt Guard 2 and Llama Guard can run with Transformers or vLLM, and CodeShield uses local Semgrep.

- **License and status:** Meta's Llama community license (weights are a gated download), not MIT or Apache. Prompt Guard 2 comes in 86M and 22M sizes.
- **Why we chose it:** Trained classifiers that run locally. Llama Guard moderates both prompts and responses; Prompt Guard targets injection and jailbreaks. CodeShield, part of LlamaFirewall, is a static-analysis engine that scans LLM-generated code with Semgrep and regex rules across 8 programming languages and 50+ weakness types (CWEs). Documentation: [CodeShield](https://meta-llama.github.io/PurpleLlama/LlamaFirewall/docs/documentation/scanners/code-shield).
- **Input use:** Prompt Guard on every user message; Llama Guard for harmful content.
- **Output use:** Llama Guard on every answer; CodeShield on every generated code block before it is shown, with block, warn or rewrite when it finds a problem.
- **Watch-outs:** Meta's model card reports .998 AUC and 97.5 percent recall at a 1 percent false-positive rate on English, but only 81.2 percent real-world attack prevention at a 3 percent utility cost. Prompt Guard has a 512-token window, so long inputs need chunking. CodeShield covers 8 languages, so we must check this against the languages our chatbot generates, and it finds insecure patterns, not every logic flaw. Legal should review the license.

#### 7. IBM Granite Guardian

Link: <https://github.com/ibm-granite/granite-guardian>

- **Deployment:** Downloadable and self-hostable model. **API requirement:** No external API is mandatory; run `granite-guardian-4.1-8b` or another Granite Guardian variant with Transformers, vLLM or Ollama.

- **License and status:** Apache 2.0 on the model cards. IBM's GGUF list includes a granite-guardian-4.1-8b release; model sizes range from about 2B to 8B parameters.
- **Why we chose it:** It detects risk in prompts and in responses, including RAG use cases, and covers groundedness. It is the strongest open option we found for hallucination.
- **Input use:** Optional harm and jailbreak check on the prompt.
- **Output use:** Check the answer against the retrieved documents for groundedness.
- **Watch-outs:** It needs GPU-class compute at the larger sizes. It checks against supplied context, so a RAG setup is required.

#### 8. Qwen3Guard

Link: <https://github.com/QwenLM/Qwen3Guard>

- **Deployment:** Downloadable and self-hostable model. **API requirement:** No external API is mandatory; run Qwen3Guard 0.6B, 4B or 8B with Transformers, vLLM or SGLang where supported.

- **License and status:** Apache 2.0. A young project (about two months old when the listing I read was updated).
- **Why we chose it:** Safety moderation across 119 languages, three severity levels (safe, controversial, unsafe) and nine categories. Qwen3Guard-Gen comes in 0.6B, 4B and 8B sizes; a Stream variant checks text while it is generated.
- **Input and output use:** An alternative to Llama Guard on prompts and responses, useful if users write in many languages.
- **Watch-outs:** One listing says vLLM and SGLang support for the Stream variant is still coming. Test on our languages before relying on it.

#### 9. Vectara HHEM-2.1-Open (Hugging Face)

Link: <https://huggingface.co/vectara/hallucination_evaluation_model>

- **Deployment:** Downloadable Hugging Face model and self-hostable classifier. **API requirement:** No external API is mandatory; run HHEM-2.1-Open with Python/PyTorch.

- **License and status:** Apache 2.0. A Hugging Face model, not a GitHub repository.
- **Why we chose it:** A small classifier (not an LLM judge) that scores whether an answer is consistent with a source text. HHEM-2.1 removes the old 512-token limit, which helps for RAG.
- **Output use:** Score each answer against the documents it was based on; block or flag low scores.
- **Watch-outs:** It needs source text. The newer HHEM-2.3 is commercial-only. Vectara's evaluation sets were English-only.

### C. PII

#### 10. Presidio

Link: <https://github.com/data-privacy-stack/presidio>

- **Deployment:** Installable Python library/service and self-hostable. **API requirement:** No external API is mandatory; use Presidio AnalyzerEngine with configured recognizers, spaCy `en_core_web_lg` or Stanza.

- **License and status:** MIT. Version 2.2.364 was released July 22, 2026 (per one review). The project moved from microsoft/presidio to a community-owned organization; container images moved to ghcr.io/data-privacy-stack.
- **Why we chose it:** It combines named-entity models, patterns and checksums, so it finds names and places that regex cannot. It runs locally and is already used by NeMo and LiteLLM.
- **Input use:** Mask PII before the prompt is logged or sent to the model.
- **Output use:** Scan answers for leaked PII.
- **Watch-outs:** On default settings an independent test produced false positives (for example, the word SSN tagged as an organization). Set an explicit entity list and score threshold. Microsoft warns that automated detection cannot guarantee it finds all sensitive data.

### D. Agent and tool-call guardrails (only if the chatbot calls tools)

#### 11. Invariant Guardrails

Link: <https://github.com/invariantlabs-ai/invariant>

- **Deployment:** Installable local library and self-hostable. **API requirement:** No external API is mandatory; its optional cloud API should be disabled for self-hosting.

- **License and status:** Apache 2.0. We could not confirm recent activity.
- **Why we chose it:** A rule-based layer that can block or flag tool calls and agent actions. It can run fully locally.
- **Use:** Rules on tool arguments and results, only if the chatbot takes actions.
- **Watch-outs:** Skip for a plain chatbot. Do not use its optional cloud API, which would break our self-hosted rule.

### E. Testing and evaluation

#### 12. NVIDIA garak

Link: <https://github.com/NVIDIA/garak>

- **Deployment:** Installable local testing tool. **API requirement:** Optional; it can test Ollama, vLLM, Transformers or hosted model APIs.

- **License and status:** Apache 2.0, maintained by NVIDIA. Lists 23 generator backends.
- **Why we chose it:** A vulnerability scanner with 50+ attack probes (injection, jailbreaks, leakage, hallucination).
- **Use:** Pre-release and periodic scans of the model and the guarded endpoint; track what gets through.
- **Watch-outs:** It does not map findings to compliance frameworks such as the OWASP list; we would do that translation.

#### 13. promptfoo

Link: <https://github.com/promptfoo/promptfoo>

- **Deployment:** Installable local CLI and self-hostable. **API requirement:** Optional; it can test local NeMo, Ollama or vLLM endpoints, or hosted provider APIs.

- **License and status:** MIT core. OpenAI announced on March 9, 2026 that it is acquiring promptfoo, and its site now says it is part of OpenAI.
- **Why we chose it:** Red-team testing with 50+ vulnerability types. It works with many model providers, and its docs show how to test NeMo Guardrails.
- **Use:** Run attack suites in CI against the chatbot with guardrails turned on.
- **Watch-outs:** The tool runs on our side, but ownership is OpenAI, which matters if our policy is no third-party vendors. Its adaptive guardrails feature is an Enterprise product, not open source.

#### 14. Ragas

Link: <https://github.com/explodinggradients/ragas>

- **Deployment:** Installable Python evaluation library and self-hostable. **API requirement:** Optional; use an internal judge such as Llama 3.1 8B Instruct or Granite 3.3 8B Instruct, or configure an external LLM/embedding API.

- **License and status:** Apache 2.0.
- **Why we chose it:** Metrics for RAG quality, including faithfulness, answer relevancy, context precision and context recall. Low faithfulness means the model is hallucinating.
- **Use:** Measure hallucination on a test set after each change; fail the build if scores drop.
- **Watch-outs:** It uses an LLM as the judge, so we must decide which model acts as judge and where it runs.

#### 15. DeepEval

Link: <https://github.com/confident-ai/deepeval>

- **Deployment:** Installable Python testing library and self-hostable. **API requirement:** Optional; use Llama 3.1 8B Instruct or Granite 3.3 8B Instruct internally, and disable the hosted Confident AI dashboard.

- **License and status:** Apache 2.0.
- **Why we chose it:** Pytest-style tests with hallucination, bias, toxicity and faithfulness metrics.
- **Use:** Unit tests for answer quality that run in CI.
- **Watch-outs:** It has an optional hosted dashboard (Confident AI). Use it offline to stay self-hosted.

 ## 4. Recommendation: the top six repositories

From the 15 repositories reviewed, these six give us the strongest first-release coverage for a self-hosted chatbot that reviews and generates code. They are complementary: one orchestrates the request, three enforce the main safety controls, and two make the system testable.

**1. NVIDIA NeMo Guardrails - Use as the pipeline framework.** It provides input, dialog, retrieval, execution and output rails, so policy is applied across the full request lifecycle. It also keeps the design portable if we change model providers.

**2. Meta PurpleLlama (Prompt Guard, Llama Guard and CodeShield) - Use as the main classifier and code-security layer.** Prompt Guard targets jailbreaks and prompt injection, Llama Guard checks harmful content on prompts and responses, and CodeShield scans generated code across eight languages. This is the broadest direct match to our threat model, although its community license needs legal review.

**3. IBM Granite Guardian - Use for groundedness and response-risk checks.** It can assess prompts and answers, including whether an answer is supported by retrieved context. That makes it the best fit in this shortlist for RAG-based hallucination control.

**4. Presidio - Use for PII detection and masking.** It combines recognizers, patterns and checksums and runs locally. It can protect both sides of the model boundary by masking submitted data and scanning responses for leakage.

**5. NVIDIA garak - Use for adversarial security testing.** Its attack probes cover injection, jailbreaks, leakage and hallucination. It gives us evidence about what the layered controls actually block before release and during regression testing.

**6. promptfoo - Use for repeatable red-team and CI tests.** It supports many providers, documents testing with NeMo Guardrails, and can run attack suites against the guarded endpoint. This turns the security requirements into repeatable tests rather than one-time manual checks.

Defer the other repositories for now. LiteLLM is useful only if we need a shared multi-model gateway; Ragas and DeepEval are valuable quality metrics but are secondary to the six above; HHEM can be added as a second groundedness check; Qwen3Guard is an alternative when multilingual coverage is a priority; and Guardrails AI remains a fallback if the team prefers Python validators over NeMo's rail model.

## 5. Risks and limitations

No guardrail blocks every attack, so NeMo, PurpleLlama, garak and promptfoo must work in layers.

NeMo Guardrails can be complex to configure, and its model-based rails may add latency, compute use and cost.

PurpleLlama may miss long or new attacks, and CodeShield does not find every logic flaw, dependency risk or programming language issue.

PurpleLlama's community license and gated model weights need legal and deployment review.

Granite Guardian needs source documents for reliable groundedness checks; it cannot verify answers based only on model knowledge.

Presidio can create false positives or miss unusual, encoded or unconfigured PII, so it cannot guarantee complete privacy protection.

garak and promptfoo test security but do not replace runtime guardrails; their probes cannot cover every attack, language or policy.

Generated code must still be reviewed and sandboxed before execution because static scanning cannot guarantee safe code.

## 6. Sources and verification

Information was gathered on October 15, 2026 from repository pages, project documentation and third-party reviews. Items from third-party articles are marked in the text. Nothing was run or benchmarked on our data.

- LLM Guard archive notice: <https://github.com/protectai/llm-guard> and review <https://app.stationx.net/articles/llm-guard>
- Open-source guardrails comparison (August 2026): <https://devtoollab.com/blog/best-llm-guardrails-tools>
- NeMo Guardrails model configuration: <https://docs.nvidia.com/nemo/guardrails/configure-guardrails/yaml-schema/model-configuration>
- Meta Prompt Guard: <https://dev.meta.ai/llama/llama-protections
- IBM Granite Guardian docs: <https://www.ibm.com/granite/docs/models/guardian>
- Qwen3Guard technical report: <https://arxiv.org/abs/2510.14276>
- Vectara HHEM: <https://huggingface.co/vectara/hallucination_evaluation_model>
- promptfoo guardrail testing: <https://promptfoo.dev/docs/guides/testing-guardrails>
- LiteLLM: <https://www.litellm.ai/>
- Portkey and Palo Alto: <https://tetrate.io/learn/ai/portkey-palo-alto-acquisition> and <https://neuraltrust.ai/blog/neuraltrust-vs-portkey>
- Bifrost: <https://github.com/maximhq/bifrost>
- garak: <https://github.com/NVIDIA/garak>
- Code checks: [CodeShield documentation](https://meta-llama.github.io/PurpleLlama/LlamaFirewall/docs/documentation/scanners/code-shield) and [Gitleaks](https://github.com/gitleaks/gitleaks)