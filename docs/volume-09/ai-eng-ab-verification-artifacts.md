# AI-ENG-AB — Verification Artifacts - Auditability, Reproducibility & Evidence Trails

## **Doctrinal Foundations: The Evidence Architecture Paradigm**

In high-dimensional artificial intelligence systems, production reliability cannot be governed or monitored by traditional application performance monitoring (APM) paradigms.1 While legacy web architectures treat system health as a binary or scalar infrastructure state defined by hardware allocation, network latency, and HTTP status codes, modern multi-agent execution loops and probabilistic large language models (LLMs) break these assumptions.1 A system can return an infrastructure-level successful HTTP 200 payload that contains a complete structural schema failure, an unauthorized tool execution, a cross-tenant data leak, or an ungrounded factual confabulation.1 Conversely, optimal hardware utilization and high throughput can coexist with red behavioral and meaning-level system states.2 This discrepancy establishes the Green Dashboard Fallacy: the diagnostic error of assuming an AI system is operating correctly simply because its underlying servers and APIs are online and available.1

To resolve this visibility deficit, modern systems architectures isolate monitoring concerns across distinct operational layers.1 While strategic telemetry (AI-ENG-Z) functions as the sensor network capturing execution spans 1, and evaluation frameworks (AI-ENG-AA) act as the regression testing pipeline 1, this document defines the architecture of verification artifacts (AI-ENG-AB).1 Verification artifacts are the durable, privacy-governed, and tamper-evident evidence objects required to reconstruct, audit, and reproduce the behavior of probabilistic systems after the fact.3

This architecture is governed by the core doctrine: every consequential AI output or action must leave behind a replayable evidence trail.1 This trail must link the exact prompt template, model weights snapshot, retrieval context, tool payload, policy execution state, human modification, and final output in an unbroken, cryptographically signed chain of custody.1 The fundamental engineering question shifts from "did we record a log entry?" to "can the organization reconstruct the exact decision path from user input to final output or action, including the evidence, versions, state transitions, checks, edits, and approvals that shaped it?".1 Without this capability, compliance becomes guesswork, debugging degrades into digital archaeology, and the system cannot be safely governed.1

## **Logs-Traces-Artifacts-Evidence Taxonomy**

To establish a clear hierarchy, the following taxonomy contrasts these diagnostic layers across operational, technical, and evidentiary boundaries, proving why generic logs are insufficient for AI auditability:

| Layer | Technical Scope & Composition | Primary Operational Purpose | System Proof Capability | Architectural Limitations |
| :---- | :---- | :---- | :---- | :---- |
| **Event Log** | Flat structured or unstructured events emitted by applications, services, or workers. | Debugging, operational status, exception capture, and coarse audit hints. | Shows that a code path emitted an event at a time. | Does not prove full context, prompt state, evidence, policy checks, user edits, or side effects. |
| **Metric** | Aggregated numeric counters, gauges, histograms, and time-series measurements. | Capacity planning, alerting, scaling, and SLO monitoring. | Shows rates, volumes, durations, and resource pressure. | Blind to semantic correctness, grounding, authorization, and user-visible truth. |
| **Execution Trace** | Parent-child span graph linking distributed work across model calls, retrieval, tools, queues, and services. | Latency analysis, causal path reconstruction, and system debugging. | Shows the causal execution path and timing relationships. | Often redacted, sampled, or incomplete for bulky or sensitive payloads. |
| **Span** | Atomic observable unit of work such as inference, retrieval, rerank, parser, tool call, guardrail, or review step. | Local timing, status, error, and attribute capture. | Shows what a specific execution step attempted and how it ended. | Does not by itself prove global policy state, source lineage, or final user-facing transformation. |
| **Verification Artifact** | Schema-validated evidence packet containing hashes, references, versions, decisions, state checks, and selected payload evidence. | Auditing a specific decision boundary or execution step. | Proves integrity of recorded inputs, outputs, parameters, and decisions to the degree captured. | Cannot prove uncaptured state; raw payload capture must be governed by privacy policy. |
| **Manifest** | Declarative version record for prompts, models, tools, dependencies, policies, routes, and environments. | Pinning expected system state and baseline capability configuration. | Shows what configuration was intended or eligible for execution. | Does not prove which dynamic inputs arrived or whether runtime behavior matched intent. |
| **Ledger** | Append-only record of actions, approvals, state transitions, tool calls, retries, and commit/verification status. | Sequencing, accountability, and action-state audit. | Shows logical progression and final known state of operations. | Does not automatically preserve full prompts, retrieved evidence, or rendered UI output. |
| **Snapshot** | Point-in-time capture or reference to context, retrieval state, memory, index version, tool state, or UI state. | Preserving what information was available at execution time. | Shows the evidence/context surface available to the system. | Storage-intensive; may require deduplication, redaction, and secure references. |
| **Diff** | Structural, textual, semantic, or field-level comparison between artifact versions. | Tracking edits, redactions, transformations, overrides, and regressions. | Shows exactly what changed between two captured states. | Does not explain why the change occurred unless linked to trace, policy, or reviewer artifacts. |
| **Evidence Trail** | Hash-linked sequence of artifacts, manifests, ledgers, snapshots, and diffs with retention and access policy. | Forensic reconstruction, compliance proof, replay, and dispute resolution. | Shows an integrity-protected chain of recorded evidence across a consequential workflow. | Tamper-evident, not omniscient; evidence quality depends on capture coverage and governance. |

Legacy logs are fundamentally insufficient for auditing probabilistic architectures.1 Recording that an API returned a response does not show the prompt template that generated it, the vector database coordinates that informed it, the safety policies that validated it, or the user edits that overrode it before execution.1 If an administrator cannot inspect these intermediate states, they cannot defend automated decisions to regulators, resolve disputes, or debug system drift.1

## **Replay and Reproducibility Typology**

### **GPU Non-Determinism and the Concurrency Hypothesis**

Achieving exact, bitwise reproducibility in large language model inference is exceptionally difficult.1 It is a common misconception that setting a fixed random number seed guarantees identical outputs across successive runs.18 In parallel computing environments, GPUs execute matrix multiplications and attention calculations across thousands of concurrent cores.19  
Because floating-point addition is non-associative, the order of arithmetic reduction operations alters the least significant bits of the floating-point representations 18:  
(a + b) + c is not equal to a + (b + c)  
If the execution speed of concurrent GPU threads varies due to minor hardware latency fluctuations, the reduction order changes, introducing run-to-run non-determinism.19  
However, technical analysis reveals that this "concurrency and floating-point" hypothesis is not the primary driver of non-determinism in production inference endpoints.18 Individual operations (kernels) inside the model's forward pass are run-to-run deterministic when executed in isolation.20 The primary cause of output variance is dynamic batching.18 High-performance serving engines (such as vLLM) batch concurrent requests dynamically to maximize hardware utilization.20 Because system load changes constantly, an individual request is batched with varying sets of other user prompts.20  
To optimize performance for different batch sizes, the engine dynamically selects different parallelization strategies, such as switching from a standard data-parallel matrix multiplication to a Split-K matmul, or modifying reduction trees in Split-Reduction RMSNorm.19 This batch-dependent kernel selection alters the reduction arithmetic, producing different outputs for the identical prompt.20

To enforce true bit-exact determinism, the model must be executed on an inference stack utilizing batch-invariant kernels.19 This strategy parallelizes operations entirely within individual tensor cores (data-parallel reductions) and avoids Split-K optimizations, preserving the arithmetic order regardless of batch changes.19 This exact reproducibility introduces a measurable performance tradeoff:

In empirical testing over a Qwen3-8B model, executing 1,000 runs under standard vLLM produced 80 unique completions, while forcing batch-invariant operations yielded a single, consistent completion across all runs, accompanied by a 61.5% execution latency penalty.18

#### **The Replay and Reproducibility Typology**

Because forcing bit-exact determinism across third-party cloud providers, variable GPU clusters, and dynamic databases is often impractical, systems must design for graded levels of verification 1:

| Replay Mode | Target Objective | Execution Constraints | State Preservation Requirement | System Verifications |
| :---- | :---- | :---- | :---- | :---- |
| **Exact Replay** | Recreate bitwise identical outputs for debugging.1 | Requires identical hardware, batch-invariant kernels, pinned model snapshots, and matching seeds.20 | Bit-exact saving of weights, inputs, and attention states.20 | Recomputes forward passes to verify exact token matches.20 |
| **Constrained Replay** | Recreate comparable completions on standard engines.1 | Executes on matching model versions with fixed sampling parameters and temperatures.5 | Saves the compiled prompt and exact decoding configurations.2 | Compares output tokens to ensure they fall within acceptable semantic ranges.1 |
| **Semantic Replay** | Verify that a rerun satisfies equivalent logical constraints.1 | Allows the model, prompt, or database to vary within defined boundaries.1 | Saves the evaluation rubric, task schema, and input goals.1 | Evaluates whether the new completion satisfies identical Pydantic rules and NLI grounding.1 |
| **Counterfactual Replay** | Compare how a change to a single variable alters behavior.1 | Modifies one parameter (e.g., prompt version, retrieved chunk) while keeping others constant.1 | Saves the base run as a template and isolates the variable under test.1 | Maps behavior changes to identify regressions or evaluate improvements.1 |
| **Forensic Replay** | Reconstruct the cause of a failure when re-execution is impossible.1 | Does not execute the active model; relies entirely on recorded logs and states.1 | Saves the complete execution trace, tool arguments, and environment hashes.3 | Walks the recorded sequence to pinpoint the exact failure location.3 |
| **Audit-Only** | Prove compliance with policies without exposing raw data.22 | Utilizes cryptographic hashes and signatures to verify states.1 | Saves the metadata headers, content hashes, and digital signatures.4 | Validates the signatures against root authorities to prove the run occurred as logged.15 |

### **Determinism Decay and Environment Drift**

A primary engineering challenge is that reproducibility degrades over time due to external dependencies.1 Even with fixed seeds and invariant kernels, the surrounding execution environment shifts silently, causing "determinism decay".1  
The primary operational drivers of this decay are categorized below:

* **Provider Model Updates:** Third-party API model weights are modified without changing the public model identifier, causing divergent semantic outputs.1  
* **Kernel and Library Drift:** Minor updates to deep learning frameworks (e.g., PyTorch, CUDA libraries) change internal mathematical approximations, mutating low-level floats.2  
* **Expired Caches and Rebuilt Indexes:** Changes to vector database structures or the expiration of semantic caches alter the exact retrieved context.1  
* **Time-Dependent Inputs:** System prompts injecting dynamic variables (e.g., current_time = "2026-06-11T23:19:00Z") force models into divergent logical paths.1  
* **Feature Flag Modifications:** Silent deployment updates change the active routing configurations or context limits mid-session.1

To address this, platforms must avoid false promises of bit-exact determinism, defining instead strict "reproducibility envelopes" that capture the complete state of the execution context.1

## **Verification Artifact Bundle Schema**

The central evidence container of this architecture is the **Verification Artifact Bundle**. It is a structured, schema-validated evidence object that records the minimum sufficient information required to audit, replay, or explain a consequential AI output or action.

The bundle should support JSON or CBOR serialization depending on platform tooling. It may be wrapped in an in-toto / DSSE-style envelope or another signed attestation format when the artifact class requires cryptographic proof. The important architectural rule is not the file format fetish—tempting though that altar is—but the preservation of signed, versioned, replayable evidence.

| Block Name | Example Field Paths | Capture Rule | Purpose |
| :---- | :---- | :---- | :---- |
| **Identity & Scope** | `identity.request_id`, `identity.trace_id`, `identity.session_hash`, `identity.tenant_hash`, `identity.subject_hash`, `identity.workflow_id`, `identity.risk_class` | Store opaque IDs or scoped hashes by default; raw identifiers only in restricted audit stores. | Correlates the bundle with traces, sessions, tenants, subjects, workflows, and risk policy. |
| **Prompt Lineage** | `prompt.template_id`, `prompt.template_version`, `prompt.compiler_hash`, `prompt.compiled_hash`, `prompt.payload_ref`, `prompt.token_count` | Store hash and secure reference for compiled prompt; raw prompt only under access-controlled registry. | Proves which prompt template and compiled prompt shaped execution. |
| **Model / Route Manifest** | `model.requested_route`, `model.selected_route`, `model.provider`, `model.model_version_ref`, `model.decoding_settings_hash`, `model.routing_reason` | Store route/model manifest and provider-visible version; note when hosted provider cannot provide weight hash. | Documents model selection, fallback, decoding, and route changes. |
| **Retrieval Snapshot** | `retrieval.index_id`, `retrieval.index_version`, `retrieval.query_hash`, `retrieval.rewrite_hashes`, `retrieval.chunk_ids`, `retrieval.source_version_hashes`, `retrieval.evidence_refs` | Store document IDs, chunk IDs, source hashes, scores, coordinates, and secure refs; avoid raw text in general bundle. | Proves what evidence was searched, selected, omitted, cited, or unavailable. |
| **Tool Execution** | `tool.name`, `tool.schema_version`, `tool.action_class`, `tool.payload_hash`, `tool.idempotency_key_hash`, `tool.pre_state_hash`, `tool.post_state_hash`, `tool.verification_status` | Store payload hashes and redacted summaries; sensitive request/response bodies by secure reference. | Proves tool intent, arguments, authorization, side-effect status, and verification outcome. |
| **State Ledger** | `state.plan_id`, `state.step_id`, `state.transition_log`, `state.loop_count`, `state.no_progress_count`, `state.fallback_state`, `state.termination_reason` | Store state transitions and hashes; never require hidden chain-of-thought capture. | Reconstructs the workflow trajectory without exposing private reasoning. |
| **User / Reviewer Edit** | `edit.draft_hash`, `edit.final_hash`, `edit.diff_ref`, `edit.editor_hash`, `edit.edit_class`, `edit.approval_state` | Store diffs, hashes, and secure refs; redact sensitive text as required. | Shows how user or reviewer modifications changed the artifact. |
| **Output Lineage** | `output.raw_hash`, `output.post_processed_hash`, `output.policy_filtered_hash`, `output.redacted_hash`, `output.rendered_hash`, `output.final_hash` | Store transformation hashes and secure refs; raw outputs only under policy. | Tracks every transformation between model output and final displayed/submitted output. |
| **Policy Enforcement** | `policy.bundle_version`, `policy.rule_ids`, `policy.decisions`, `policy.enforcement_points`, `policy.override_state`, `policy.reviewer_hash` | Store deterministic rule results separately from classifier scores. | Proves which policy checks ran, where, and with what outcome. |
| **Human Approval** | `approval.request_id`, `approval.status`, `approval.maker_hash`, `approval.checker_hash`, `approval.payload_hash`, `approval.expires_at`, `approval.signature_ref` | Store signed approval metadata and payload hash. | Proves approval state, separation of duties, and freshness of authorization. |
| **Audit & Integrity** | `audit.bundle_hash`, `audit.parent_hash`, `audit.signature_ref`, `audit.created_at`, `audit.retention_class`, `audit.legal_hold`, `audit.redaction_class` | Sign bundle hash or envelope; store chain metadata and retention policy. | Makes the evidence record tamper-evident and governed. |

### **Risk-Class Tiering Policy**

The bundle should adapt to risk class:

| Risk Class | Capture Profile |
| :---- | :---- |
| **Low** | Metadata, hashes, route IDs, status, and short-retention trace links. |
| **Medium** | Metadata plus selected redacted snippets, secure refs, policy decisions, and tool hashes. |
| **High** | Complete-enough evidence trail with secure references, signatures, approval records, state verification, and stricter retention. |
| **Critical / Regulated** | High-risk profile plus legal hold support, stronger signing, replay package linkage, reviewer identity controls, and access-audited payload vaults. |

High risk does **not** mean “store everything raw forever.” It means preserve enough evidence to audit and replay the decision while minimizing raw sensitive payloads, enforcing access controls, and recording when data was intentionally not captured.

## **Reproducibility Envelope Model**

Every consequential AI execution should run inside a **Reproducibility Envelope**. The envelope records the system state needed to recompile, inspect, simulate, or replay the run. The reproducibility unit is not “the model” alone; it is the model route, prompt compiler, retrieval surface, policy bundle, tool contracts, feature flags, runtime configuration, and evidence state.

```
REPRODUCIBILITY ENVELOPE

+----------------------------------------------------------+
| Prompt Assembly                                          
| template id/version | compiler hash | context policy     
+----------------------------------------------------------+
| Inference Route                                          
| provider route | model ref | tokenizer | decoding config 
+----------------------------------------------------------+
| Retrieval Surface                                        
| corpus version | index hash | embedding model | filters  
+----------------------------------------------------------+
| Tool and Action Surface                                  
| tool schemas | mock mode | credential policy | idempotency
+----------------------------------------------------------+
| Policy and Governance                                    
| policy bundle | approval flow | safety filters | retention
+----------------------------------------------------------+
| Runtime Environment                                      
| container image | dependency lock | feature flags | region 
+----------------------------------------------------------+
| Replay Constraints                                       
| exact | constrained | semantic | counterfactual | forensic
+----------------------------------------------------------+
```

| Parameter Category | Example Field Paths | Validation Source Registry |
| :---- | :---- | :---- |
| **Prompt Assembly** | `env.prompt.template_id`, `env.prompt.template_version`, `env.prompt.compiler_hash`, `env.prompt.context_assembler_hash`, `env.prompt.memory_policy_version` | Prompt registry, compiler release manifest, context policy registry. |
| **Inference Runtime** | `env.model.requested_route`, `env.model.selected_route`, `env.model.provider_api_version`, `env.model.snapshot_ref`, `env.model.tokenizer_ref`, `env.model.decoding_settings_hash` | Model registry, provider manifest, gateway route manifest. |
| **Serving Environment** | `env.serving.container_digest`, `env.serving.runtime_version`, `env.serving.cuda_or_accelerator_version`, `env.serving.batch_policy`, `env.serving.determinism_mode` | Container registry, deployment manifest, serving-engine config. |
| **Retrieval Engine** | `env.retrieval.index_id`, `env.retrieval.index_version`, `env.retrieval.corpus_version`, `env.retrieval.embedding_model_ref`, `env.retrieval.chunker_version`, `env.retrieval.reranker_ref`, `env.retrieval.permission_policy_version` | Vector/index registry, corpus manifest, ingestion pipeline records. |
| **Tool Integration** | `env.tool.manifest_hash`, `env.tool.schema_versions`, `env.tool.server_digest`, `env.tool.mock_mode`, `env.tool.side_effect_mode`, `env.tool.credential_policy_version` | Tool registry, container registry, credential broker policy. |
| **Policy and Safety** | `env.policy.bundle_hash`, `env.policy.safety_filter_version`, `env.policy.redaction_policy_version`, `env.policy.approval_flow_version`, `env.policy.retention_class` | Policy registry, governance manifest, approval workflow registry. |
| **System and UI** | `env.ui.version`, `env.ui.renderer_hash`, `env.feature_flags.hash`, `env.deployment.hash`, `env.region`, `env.cache_state`, `env.degraded_mode_state` | Web client registry, feature-flag snapshot, deployment manifest. |
| **Replay Controls** | `env.replay.mode`, `env.replay.egress_policy`, `env.replay.payload_capture_class`, `env.replay.tool_response_source`, `env.replay.allowed_counterfactuals` | Replay policy registry and audit configuration. |

This envelope does not guarantee bit-exact reproduction in every environment. It defines what must be pinned, captured, or declared unknown so auditors can distinguish exact replay, constrained replay, semantic replay, counterfactual replay, forensic replay, and audit-only verification.30

## **Specialized Evidence Records**

### **The Prompt Lineage Record**

To determine whether behavior changed because of template edits, context assembly, memory inclusion, or dynamic variable injection, the system preserves a Prompt Lineage Record.

| Field Name | Physical Data Type | Capture Rule | Operational Purpose |
| :---- | :---- | :---- | :---- |
| `prompt.template_id` | String | Metadata. | Identifies the registered template. |
| `prompt.template_version` | SemVer / release ID | Metadata. | Matches the active template registry version. |
| `prompt.template_hash` | SHA-256 hash | Hash. | Verifies template integrity without exposing text. |
| `prompt.template_ref` | Secure reference | Restricted. | Points to raw template in controlled registry if access is allowed. |
| `prompt.compiler_version` | Version / commit SHA | Metadata. | Identifies prompt compiler behavior. |
| `prompt.context_assembler_version` | Version / commit SHA | Metadata. | Identifies context assembly logic. |
| `prompt.variable_manifest` | JSON object / hash | Redacted or secure reference. | Records which variables were injected and their capture classes. |
| `prompt.memory_refs` | Array of hashes/refs | Hash/reference. | Identifies memory items included in context. |
| `prompt.evidence_refs` | Array of chunk/source refs | Hash/reference. | Identifies retrieval chunks inserted into context. |
| `prompt.tool_schema_refs` | Array of schema hashes | Metadata/hash. | Records tool definitions available to the model. |
| `prompt.policy_refs` | Array of policy hashes | Metadata/hash. | Records dynamic safety/governance instructions. |
| `prompt.message_boundaries` | Structured offsets / roles | Metadata. | Maps system, developer, user, tool, and context scopes. |
| `prompt.compiled_hash` | SHA-256 hash | Hash. | Verifies exact compiled prompt. |
| `prompt.compiled_ref` | Secure reference | Restricted. | Allows controlled replay of compiled prompt when permitted. |
| `prompt.token_count` | Integer | Metadata. | Records input size for cost and replay. |
| `prompt.version_diff_ref` | Secure diff reference | Restricted. | Links to template diff without dumping sensitive prompt text into logs. |

Saving only raw templates is insufficient because compiled prompts include dynamic variables, memories, retrieved evidence, tool schemas, policies, and message boundaries. The lineage record proves what was assembled without requiring every observer to see the full prompt text.

### **The Model and Route Manifest**

Model route decisions are critical audit artifacts.1 A silent model downgrade, failover routing, or cache bypass changes the quality, safety, and cost of a response, and must be documented explicitly 1:

| Field Name | Physical Data Type | Validation Range | Description / Key Purpose |
| :---- | :---- | :---- | :---- |
| route.provider_name | String | openai | anthropic | local | Identifies the physical host handling the query.1 |
| route.requested_model | String | Standard model names | Documents the client's intended capability baseline.1 |
| route.selected_model | String | Standard model names | Records the actual model executed for inference.1 |
| route.model_version | String | Vendor snapshot tag | Pinpoints the exact model snapshot used.1 |
| route.routing_reason | Enum | nominal | failover | cost | Explains why the gateway deviated from requested models.1 |
| route.fallback_path | String | URI string | Documents the fallback route triggered on failure.1 |
| route.cached_response | Boolean | True | False | Flags whether the gateway served a semantic cache hit.1 |
| route.router_policy | String | SemVer format | Tracks the active gateway model selection policy version.1 |
| route.quality_floor | String | Class identifier | Verifies fallback models met capability thresholds.1 |
| route.safety_floor | String | Class identifier | Verifies fallback models contained necessary filters.1 |
| route.cost_estimate | Double | Currency (> 0.00) | Tracks preflight cost calculations.1 |
| route.latency_estimate | Double | Latency (> 0.0) | Tracks expected response durations.1 |
| route.context_headroom | Integer | Token count (> 0) | Verifies model possessed necessary context size.1 |
| route.tool_requirement | Boolean | True | False | Flags if the task required native tool capabilities.1 |
| route.modality | Enum | text | image | video | Records the input modalities processed.1 |
| route.tenant_tier | Enum | free | standard | enterprise | Links execution priorities to B2B subscription metrics.1 |
| route.budget_state | Double | Currency (> 0.00) | Confirms tenant credit balances were verified.1 |
| route.decoding_settings | JSON String | JSON map | Documents temperature, top_p, and frequency penalty.1 |
| route.stop_reason | Enum | stop | length | content_filter | Records why token generation terminated.1 |
| route.user_disclosed | Boolean | True | False | Proves the user was notified of model changes.1 |

### **The Retrieval Evidence Snapshot**

GRC verification requires proving the exact context available to the system before generation, ensuring models do not perform "citation laundering" (citing files that were never actually retrieved) 1:

| Field Name | Physical Data Type | Validation Range | Description / Key Purpose |
| :---- | :---- | :---- | :---- |
| retrieval.raw_query | String | Unicode text | The raw query submitted to the retrieval engine.1 |
| retrieval.rewritten | Array | String Array | Logs query expansion and reformulation steps.1 |
| retrieval.embedding | String | Model ID + Hash | Records the model used to vectorize the query.1 |
| retrieval.index_id | String | UUIDv4 format | Identifies the physical database partition searched.1 |
| retrieval.index_version | String | SemVer format | Tracks database migrations and schema updates.1 |
| retrieval.corpus_version | String | SemVer format | Tracks knowledge base ingestion updates.1 |
| retrieval.tenant_scope | String | UUIDv4 format | Confirms search was isolated to the active tenant partition.1 |
| retrieval.permissions | Array | Role principal strings | Records active RBAC filters applied during search.1 |
| retrieval.metadata | JSON String | JSON map | Logs structural metadata filters applied.1 |
| retrieval.candidates | Array | Object Array | Logs the top-100 initial retrieved candidate IDs and scores.1 |
| retrieval.selected | Array | Object Array | Logs the subset of chunks loaded into active context.1 |
| retrieval.omitted | Array | Object Array | Logs relevant chunks bypassed during context pruning.1 |
| retrieval.rerank_scores | Array | Double Array | Records cross-encoder relevance scores for candidates.1 |
| retrieval.authority | Double | Range [0.0, 1.0] | Documents the trust weight of the retrieved document.1 |
| retrieval.freshness | String | Timestamp (ISO-8601) | Compares document modification dates against database states.1 |
| retrieval.doc_version | String | SHA-256 Hex | Cryptographic digest of the parent document source.1 |
| retrieval.chunk_ids | Array | UUIDv4 Array | Unique identifiers of the exact retrieved segments.1 |
| retrieval.secure_ref | String | Secure URI string | Redirection pointer to raw text (for data protection).1 |
| retrieval.coordinates | Array | Bounding Box Coordinate Array | Page, line, and offset coordinates of retrieved data.1 |
| retrieval.citation_map | JSON String | JSON mapping object | Binds generated sentences to retrieved coordinates.1 |
| retrieval.conflict_signal | Boolean | True | False | Flags if different retrieved sources contradicted each other.1 |
| retrieval.stale_flag | Boolean | True | False | Flags if retrieved document had exceeded its TTL.1 |
| retrieval.inaccessible | Boolean | True | False | Flags if source was deleted but remained in vector index.1 |
| retrieval.claim_links | Array | Object Array | Maps specific model-generated assertions to chunk hashes.1 |

This snapshot provides bounded verifiability when capture coverage, source versioning, and permission metadata are complete: if a model generates a factual error, the snapshot proves whether the error was caused by retrieval failure or model hallucination.1

### **The Tool Execution Record**

A generated text statement claiming an action succeeded (e.g., "invoice sent") is not evidence of success.1 Auditing requires verifying the physical database transactions and tool invocations 1:

| Field Name | Physical Data Type | Validation Constraint | Description / Key Purpose |
| :---- | :---- | :---- | :---- |
| tool.planned_name | String | Match approved tool schemas | The tool the model intended to invoke.1 |
| tool.executed_name | String | Match approved tool schemas | The tool actually executed by the coordinator.1 |
| tool.schema_version | String | SemVer format | Tracks modifications to API specifications.1 |
| tool.action_class | Enum | idempotent | mutation | Identifies state-changing operations.1 |
| tool.permissions | Array | Role principal strings | Confirms active RBAC credentials before execution.1 |
| tool.credential | String | Token reference URI | Records the short-lived OAuth credentials utilized.1 |
| tool.argument_object | JSON String | JSON schema compliant | The exact inputs generated by the model.1 |
| tool.argument_hash | String | SHA-256 Hex | Cryptographic digest of the argument object.1 |
| tool.user_edit_diff | String | Unified character diff | Records manual overrides made by the user.1 |
| tool.idempotency_key | String | Unique UUIDv4 | Prevents duplicate actions during network retries.1 |
| tool.request_payload | String | Serialized payload string | The raw request dispatched to the external system.1 |
| tool.response_payload | String | Serialized payload string | The raw response returned by the external system.1 |
| tool.error_status | Enum | success | timeout | error | Captures API-level exceptions and system errors.1 |
| tool.retry_decision | Enum | backoff | abort | Documents retry actions triggered on failure.1 |
| tool.quota_consumed | Double | Resource metric | Tracks external system rate limits and credits.1 |
| tool.side_effects | Array | Object Array | Documents downstream mutations triggered by the tool.1 |
| tool.state_pre_hash | String | SHA-256 Hex | Proves database state before action execution.1 |
| tool.state_post_hash | String | SHA-256 Hex | Proves database state after action execution.1 |
| tool.system_verified | Boolean | Must be True | Proves the state change was verified in the database.1 |
| tool.compensation | String | URI Endpoint | Specifies the rollback action to trigger on failure.1 |
| tool.user_confirmed | Boolean | True | False | Proves explicit user consent before execution.1 |

### **The Intermediate State Ledger**

To trace execution trajectories without exposing raw model reasoning, the system preserves the Intermediate State Ledger 1:

| Field Name | Physical Data Type | Validation Constraint | Description / Key Purpose |
| :---- | :---- | :---- | :---- |
| state.plan_version | String | SemVer format | Tracks modifications to the agentic planner.1 |
| state.subtask_graph | JSON String | JSON DAG representation | Maps dependencies across planned agent tasks.1 |
| state.transitions | Array | Object Array | Logs state-machine transitions and events.1 |
| state.loop_counter | Integer | Positive Integer (<= 10) | Prevents runaway costs and infinite tool loops.1 |
| state.memory_reads | Array | String Array (hashes) | Logs profile and memory retrievals during the turn.1 |
| state.memory_writes | Array | String Array (hashes) | Logs updates committed to long-term storage.1 |
| state.retry_branches | Array | Object Array | Logs sub-agent retries and execution paths.1 |
| state.repair_attempts | Array | Object Array | Logs schema corrections and output modifications.1 |
| state.validation_fails | Array | Object Array | Logs Pydantic and JSON validation exceptions.1 |
| state.no_progress | Integer | Positive Integer (<= 2) | Tracks sequential turns with zero logical progress.1 |
| state.tool_results | Array | String Array (hashes) | Correlates execution results with active steps.1 |
| state.policy_blocks | Array | String Array (rule IDs) | Logs safety and compliance rules triggered.1 |
| state.fallback | Enum | nominal | degraded_model | Logs gateway fallback transitions triggered.1 |
| state.degraded_state | Enum | nominal | cached | partial | Tracks system behavior under resource pressure.1 |
| state.termination | Enum | success | max_turns | abort | Records why the execution sequence terminated.1 |

### **The User Edit and Output Diff Record**

The final answer displayed on a user's screen is simply the last artifact in a chain of output transformations.1 The system documents each transformation step:

| Field Name | Physical Data Type | Validation Constraint | Description / Key Purpose |
| :---- | :---- | :---- | :---- |
| edit.original_draft | String | Raw output text | The unmodified token stream generated by the model.1 |
| edit.user_edits | String | Unicode text | The finalized text after human modification.1 |
| edit.editor_id | String | User principal string | Identifies the operator executing changes.1 |
| edit.edit_class | Enum | factual | stylistic | code | Categorizes the modification type.1 |
| edit.timestamp | String | Timestamp (ISO-8601) | Pinpoints when the override occurred.1 |
| edit.target_field | String | Schema property name | Identifies the specific input field altered.1 |
| edit.status | Enum | accepted | rejected | Documents whether edits were committed.1 |
| edit.diff_view | String | Standard Unified Diff | Unified character diff of the user modification.1 |
| edit.updated_memory | Boolean | True | False | Logs if edits updated local conversation memory.1 |
| edit.entered_evals | Boolean | True | False | Flags if edited runs were promoted to golden sets.1 |
| edit.changed_args | Boolean | True | False | Flags if edits updated tool parameters.1 |
| edit.preserved_output | Boolean | True | False | Flags if the edit was retained in final display.1 |
| edit.overwritten | Boolean | True | False | Flags if the model later discarded user changes.1 |

To ensure complete output lineage, the platform captures every intermediate transformation stage, recording the SHA-256 hash and a character diff for each:

1. output.raw_model -> output.post_processed (e.g., stripping Markdown blocks, parsing syntax).1  
2. output.post_processed -> output.policy_filtered (e.g., blocking toxic content, enforcing compliance rules).1  
3. output.policy_filtered -> output.redacted (e.g., masking SSNs, credentials, PII via ARGUS).1  
4. output.redacted -> output.citation_injected (e.g., binding references to coordinate-level chunks).1  
5. output.citation_injected -> output.rendered_ui (e.g., compiling text to structured UI cards or speech).1  
6. output.rendered_ui -> output.human_edited (e.g., capturing operator overrides and updates).1  
7. output.human_edited -> output.final_submitted (e.g., the finalized output written to database).1

### **The Policy Check Record**

Policy checks must function as executable verification artifacts, rather than passive, unstructured annotations 1:

| Field Name | Physical Data Type | Validation Constraint | Description / Key Purpose |
| :---- | :---- | :---- | :---- |
| policy.bundle_version | String | SemVer format | Tracks the active regulatory and safety rules.1 |
| policy.rule_ids | Array | String Array | Identifies the specific rules executed.1 |
| policy.source_auth | String | Authority principal string | Documents who authorized and signed the rule.1 |
| policy.input_evaluated | String | SHA-256 Hex | Verifies the input data evaluated by the rule.1 |
| policy.decision | Enum | allow | block | escalate | The action completed by the policy check.1 |
| policy.threshold | Double | Range [0.0, 1.0] | The sensitivity cutoff of the policy filter.1 |
| policy.confidence | Double | Range [0.0, 1.0] | The classifier confidence of the safety model.1 |
| policy.enforce_point | Enum | ingress | egress | gateway | Pinpoints where the policy filter was applied.1 |
| policy.pass_fail | Enum | pass | fail | Records the binary rule check status.1 |
| policy.override_status | Enum | nominal | authorized_override | Flags if the block was overridden.1 |
| policy.exception | String | ASCII Error String | Logs system and parsing exceptions.1 |
| policy.reviewer | String | Reviewer principal string | Identifies the officer authorizing overrides.1 |
| policy.approval_path | String | URI string | Documents the digital approval trail.1 |
| policy.escalation | String | URI string | Links the check to human-in-the-loop queues.1 |
| policy.action_state | Enum | completed | blocked | Records the final system state post-check.1 |

## **Replay Package Model and Execution Safety**

To audit, debug, or evaluate a transaction safely, the system compiles a Replay Package.1 It is an executable, sandboxed environment that allows developers and compliance teams to rerun or simulate runs without risking side effects.1  
The Replay Package consists of several core components:

* **Trace Graph:** The parent-child span hierarchy representing the original causal execution path.1  
* **Verification Bundle:** The cryptographically signed, complete evidence record of the transaction.1  
* **Dependency Manifest:** Pinned package lockfiles and container image digests.1  
* **Tool Mocks:** Pre-recorded, signed tool response payloads, mapped directly to caller keys.1  
* **Environment Variables:** Locked configuration parameters and system feature flag states.1  
* **Policy Bundle:** The exact policy engine rules and thresholds that evaluated the run.1  
* **Evaluation Rubric:** The objective performance benchmarks used to score completions.1

To prevent actions from escaping containment and mutating live production systems, the platform enforces several execution constraints 1:

```
+-----------------------------------------------------------------------------------+  
|                              REPLAY PACKAGE CONTAINER                             |  
+-----------------------------------------------------------------------------------+  
|    |                                                                              |  
|    +----> Simulated LLM Inference  ----> (Mocked via recorded output tokens)      |  
|    +----> Simulated DB Similarity  ----> (Mocked via retrieval snapshot chunks)   |  
|    +----> Simulated Tool API Calls ----> (Blocked at loopback proxy)              |  
+-----------------------------------------------------------------------------------+
```

These execution bounds are mapped across six discrete replay modes:

| Replay Mode | Egress Policy | Input Constraints | Execution Layer | Target Verification | Side-Effect Prevention |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Read-Only Replay** | Blocked entirely.1 | Inputs locked to original values.1 | Re-compiles layout structures only.1 | Verifies text rendering and display formats.1 | Read-only execution prevents database write actions.1 |
| **Mocked-Tool Replay** | Blocked entirely.1 | Inputs locked to original values.1 | Re-executes model inference; mocks tools.1 | Evaluates output variance under fixed templates.1 | API gateway loopback intercepts and mocks all writes.1 |
| **Sandbox Replay** | Isolated sandbox subnet.31 | Inputs locked; dynamic tools active.31 | Re-executes the run within a isolated test database.31 | Evaluates end-to-end integration safety.1 | Local Docker environments isolate tool containers.31 |
| **Counterfactual Replay** | Blocked entirely.1 | Allows single parameter modifications.1 | Re-runs template compilation over model.1 | Pinpoints regressions and evaluates changes.1 | Mocks tool response arrays using similarity indexes.1 |
| **Semantic Replay** | Blocked entirely.1 | Inputs locked to original values.1 | Evaluates logical constraints on standard stack.1 | Verifies semantic and grounding equivalence.1 | Compares output tokens to ensure compliance.1 |
| **Forensic Replay** | Blocked entirely.1 | No model execution; uses recorded states.3 | Timeline analysis of pre-recorded traces.3 | Pinpoints failure locations and identifies causes.3 | Complete execution decoupling; no code is run.3 |

## **Evidence Chain and Tamper-Evidence Infrastructure**

To guarantee that recorded verification artifacts are not silently modified or deleted after the fact, the platform implements a cryptographic hash chain.1 Each transaction's Verification Artifact Bundle (sigma_i) is cryptographically linked to its predecessor, forming an append-only chain 1:  
h_i = H(h_{i-1} |  
| canonical(sigma_i))  
where h_0 = 0^64, H is a collision-resistant SHA-256 function, and canonical represents a deterministic JSON serialization format (with sorted keys and stripped whitespace).33  
To establish an institutional root of trust, the platform integrates with the OpenSSF Model Signing (OMS) and Sigstore standards, keyless signing pipelines, and C2PA metadata frameworks.34  
The integration of these standards operates through three distinct layers:

1. **Identity Attestation (Fulcio CA):** The signing engine (e.g., an agent runtime or a CI/CD pipeline) generates an ephemeral public/private keypair in-memory.35 It authenticates with a trusted OpenID Connect (OIDC) provider (e.g., GitHub, Google) to obtain an identity token.35 Fulcio validates this token and issues a short-lived X.509 certificate (typically valid for 10 minutes) binding the verified identity to the ephemeral public key.35  
2. **Tamper-Evident Ledger (Rekor Log):** The signature, the X.509 certificate, the artifact digest, and the parent hash chain are submitted to Rekor, an append-only, public transparency log.35 Rekor records the entry in its Merkle tree and returns a signed timestamp token proving the signature was generated during the certificate's 10-minute validity window.35 The ephemeral private key is then permanently discarded, eliminating the security risks of long-lived keys.35  
3. **Media and Text Provenance (C2PA standard):** For visual outputs, charts, or video, the signing engine embeds a C2PA manifest within a JUMBF metadata box directly inside the output file.12 For text-only or binary datasets, the engine generates a sidecar C2PA manifest utilizing an Asset Reference Assertion to cryptographically bind the signature to the external file hash.38

```
+---------------------------------------------------------------------------------+  
|                            TAMPER-EVIDENT EVIDENCE CHAIN                        |  
+---------------------------------------------------------------------------------+  
|  [ Ingest Artifact ] (SHA-256 Hash)                                             |  
|        |                                                                        |  
|        +----> Sign with short-lived X.509 Certificate (Fulcio)                  |  
|        +----> Record Signature & Certificate in Append-Only Ledger (Rekor)      |  
|        +----> Embed CBOR-serialized Manifest in JUMBF Container (C2PA)          |  
+---------------------------------------------------------------------------------+
```

The specific schema mapping for the Evidence Chain is detailed below:

| Field Name | JSON Field Key Path | Data Type | Validation Standard |
| :---- | :---- | :---- | :---- |
| **Artifact ID** | chain.artifact_id | String | Regex UUIDv4.1 |
| **Artifact Type** | chain.artifact_type | Enum | verification_bundle | repro_env.1 |
| **Content Hash** | chain.content_hash | String | SHA-256 Hex.1 |
| **Parent Hash** | chain.parent_hash | String | SHA-256 Hex (matches h_{i-1}).1 |
| **Created By** | chain.created_by | String | OIDC Workload Identity.1 |
| **Created At** | chain.created_at | String | Timestamp (ISO-8601 with ms).1 |
| **Signed By** | chain.signed_by | String | Fulcio X.509 Subject.1 |
| **Signing Key** | chain.signing_key | String | Ephemeral Public Key.1 |
| **Storage Location** | chain.storage_path | String | S3 URI with region.1 |
| **Retention Class** | chain.retention_class | Enum | hot | warm | cold.1 |
| **Access Policy** | chain.access_policy | String | IAM Role Identifier.1 |
| **Redaction Status** | chain.redacted | Enum | un_redacted | partially_redacted.1 |
| **Deletion Eligibility** | chain.deletion_date | String | Timestamp or permanent.1 |
| **Legal Hold** | chain.legal_hold | Boolean | True | False.1 |
| **Verification** | chain.verification | Enum | verified | unverified.1 |

This structure prevents "blockchain theater," using practical, industry-standard cryptographic tools to ensure that if an attacker alters a single historical byte, subsequent hash-chain comparisons fail immediately, alert monitors trigger, and the compromised environment is quarantined.1

## **Artifact Privacy, Redaction, and Retention Model**

Verification artifacts contain sensitive operational data, including proprietary documents, customer records, payment details, and credentials.1 To prevent the evidence trail from acting as a data leak repository, the system implements a strict Data Minimization Model.1  
This is enforced using seven distinct Capture Classes that categorize and redact data prior to storage:

```
+-----------------------------------------------------------------------------+  
|                            PRIVACY REDACTION WORKFLOW                       |  
+-----------------------------------------------------------------------------+  
|  ----> ARGUS Regex/NER Scanner                                              |  
|        |                                                                    |  
|        +----> Raw SSN/Passwords  ----> Class 0: Never Capture (Dropped)     |  
|        +----> Large Documents    ----> Class 4: Reference Only (Secure S3)  |  
|        +----> Conversation Logs  ----> Class 3: Redacted Snippet (Masked)   |  
+-----------------------------------------------------------------------------+
```

The specific properties and processing rules of these capture classes are mapped below:

| Capture Class | Technical Processing Rule | Physical Storage Substrate | Access Control Level | Typical Data Target |
| :---- | :---- | :---- | :---- | :---- |
| **0. Never Capture** | Strip elements immediately at the gateway before writing any records.2 | No storage footprint.31 | None.31 | Passwords, API tokens, credit card CVVs.31 |
| **1. Hash Only** | Calculate the SHA-256 digest of the payload; discard the raw characters.1 | Relational Database metadata columns.10 | Public / System audit.10 | File attachments, large database structures.5 |
| **2. Metadata Only** | Preserve size, schema names, and response status; strip content.2 | Relational Database indexes.10 | Standard Auditor | PDF layout trees, API schemas.5 |
| **3. Redacted Snippet** | Run the ARGUS engine to replace sensitive variables with placeholders.2 | Ephemeral Redis cache partitions.5 | Standard Operator | Conversation histories containing PII.2 |
| **4. Reference-Only** | Store the raw payload in a secure, role-gated bucket; save only the URI in traces.1 | Encrypted Object Storage (S3 with KMS encryption).2 | Role-Gated Administrator | Original customer PDFs, full invoice scans.2 |
| **5. Full Payload** | Retain complete data under strict encryption and access logging.1 | Decoupled Secure S3 Bucket (restricted region).31 | Decoupled Security Admin | High-risk financial ledger writes.1 |
| **6. Incident Mode** | Temporarily escalate capture settings during active outages or attacks.1 | Ephemeral Forensic Vault (short TTL).31 | Escalated Security Analyst | Running agent trace dumps during failure runs.1 |

This model ensures that the evidence trail remains a safe GRC capability rather than a target for data harvest attacks.1

## **GRC Audit Query Playbook**

To support systematic audits and regulatory inspections, the verification framework resolves standard queries through precise mapping to required artifacts, trace fields, and replay modes 1:

| Audit Question | Required Verification Artifacts | Target Trace Fields / Metadata | Optimal Replay Mode | Technical Validation Path |
| :---- | :---- | :---- | :---- | :---- |
| **Why did the model cite this source?** | Retrieval Snapshot, Model Manifest.1 | retrieval.chunks, model.selected_model.1 | Forensic Replay.1 | Checks that the cited text has a high NLI entailment score against the source document coordinates.1 |
| **Was the source document actually retrieved?** | Retrieval Snapshot.1 | retrieval.index_id, retrieval.chunks.id.1 | Audit-Only.1 | Matches retrieved chunk IDs against the database access log.1 |
| **Was the source version current?** | Retrieval Snapshot, Model Manifest.1 | retrieval.chunks.version, model.snapshot.1 | Audit-Only.1 | Compares retrieved version hashes against the master registry.1 |
| **Which prompt version produced this answer?** | Prompt Lineage Record.1 | prompt.template_id, prompt.template_version.1 | Constrained Replay.1 | Re-compiles the template to confirm it outputs matching token hashes.1 |
| **Did routing silently downgrade the model?** | Model Selection Manifest.1 | model.requested_model, model.selected_model.1 | Audit-Only.1 | Detects discrepancies between requested and executed models.2 |
| **Which tool call failed, and did it succeed later?** | Tool Execution Record.1 | tool.verification_status, tool.logical_epoch.1 | Semantic Replay.1 | Audits the epoch timeline to identify the failure point.1 |
| **Did the tool actually execute, or did the model hallucinate success?** | Tool Execution Record.1 | tool.state_post_hash, tool.verification_status.1 | Forensic Replay.1 | Verifies that the post-action state hash matches the database record.1 |
| **Was a confirmation shown before transaction execution?** | Tool Execution Record, Policy Record.1 | tool.idempotency_key, policy.fired_rules.1 | Forensic Replay.1 | Verifies that a signed permission token was generated before the epoch write.1 |
| **Did the user edit the value?** | User Edit Record.1 | edit.character_diff, edit.user_id.1 | Counterfactual Replay.1 | Compares the generated draft with the final display text.1 |
| **Did the model overwrite the correction?** | User Edit Record, Intermediate State Ledger.1 | edit.character_diff, ledger.current_step.1 | Counterfactual Replay.1 | Compares subsequent generation blocks with the edited state.1 |
| **Which policy rule fired?** | Policy Check Record.1 | policy.fired_rules, policy.bundle_version.1 | Audit-Only.1 | Verifies rule execution parameters against rule assertions.1 |
| **Who approved the override?** | Policy Check Record.1 | policy.override_signer, policy.fired_rules.1 | Audit-Only.1 | Checks the X.509 signature against root access directories.1 |
| **Did the final UI hide a warning?** | Output Transformation Record, Policy Record.1 | transform.final_display, policy.fired_rules.1 | Forensic Replay.1 | Confirms that the rendered output contains the warning blocks.5 |
| **Would a different model have behaved differently?** | Verification Bundle, Reproducibility Envelope.1 | model.model_id, repro_env.transformers_version.1 | Counterfactual Replay.1 | Re-runs the compiled prompt over alternative model versions.1 |

## **Artifact Lifecycle and Storage Architecture**

To manage costs and comply with regulatory retention policies, the platform organizes evidence across three storage tiers 4:

```
+-----------------------------------------------------------------------------------+  
|                            EVIDENCE STORAGE LIFECYCLE                             |  
+-----------------------------------------------------------------------------------+  
|   -> 0-90 days, 4-hour retrieval, Multi-tenant Redis/PostgreSQL      |  
|       |                                                                           |  
|       v (Age > 90 days)                                                           |  
|  -> 91 days - 2 years, 24-hour retrieval, S3 CAS with RLS           |  
|       |                                                                           |  
|       v (Age > 2 years & REGULATORY_7YR tag)                                      |  
|  -> 2-7 years, 72-hour retrieval, WORM Archive (Glacier Object Lock)|  
+-----------------------------------------------------------------------------------+
```

The operational profiles, retrieval SLAs, and redundancy targets of these storage tiers are specified as follows:

| Storage Tier | Data Age Window | Retrieval SLA | Storage Redundancy Target | Physical Technology | Encryption Profile |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Hot Debugging** | 0 to 90 days.4 | 4 hours.4 | Minimum of 3 active nodes (N + 2).4 | PostgreSQL with pgvector indexes; multi-tenant Redis partitions.2 | AES-256 with tenant-scoped KMS keys.2 |
| **Warm Audit** | 91 days to 2 years.4 | 24 hours.4 | Minimum of 3 active nodes (N + 2).4 | Content-Addressed Object Storage (S3 with content-hash indexing).4 | AES-256 with tenant-scoped KMS keys.2 |
| **Cold Compliance** | 2 to 7 years.4 | 72 hours.4 | Minimum of 2 active nodes (N + 1).4 | Write-Once-Read-Many (WORM) storage (AWS S3 Glacier Object Lock).4 | Corporate-managed KMS keys.4 |

To support discoverability, evidence indexes must be searchable across multiple keys—including trace_id, session_id, tenant_id, model_version, prompt_version, document_id, tool_call_id, and final_output_hash.1 These indexes are updated dynamically via event-driven ingestion workers, ensuring compliance officers can locate specific transactions in minutes.4

## **Cross-Canon Handoff Map**

The Verification Artifacts architecture integrates with adjacent disciplines across the AI Systems Engineering Canon 1:

| Target Report ID | Target Report Domain | Operational Handoff Metric | Dependency / Engineering Integration Rules |
| :---- | :---- | :---- | :---- |
| **AI-ENG-AC** | Incident Response & Remediation.1 | Isolation Containment Delay (< 15 seconds).31 | Leverages the Verification Artifact Bundle to run forensic postmortems and isolate compromised user accounts.1 |
| **AI-ENG-AD** | Operational Governance & Compliance. | Gating Compliance Ratio (100.0%).1 | Uses Evidence Trails and in-toto attestations to satisfy SOC2 Type II and ISO/IEC 42001 audits.13 |
| **AI-ENG-AJ** | Reference Architectures. | Schema Validation Fidelity (100.0%).2 | Integrates the Verification Bundle schema into B2B multi-tenant reference blueprints.2 |
| **AI-ENG-Z** | Strategic Telemetry.1 | OpenTelemetry Compliance Score (100.0%).2 | Inherits the trace-id and span-id propagation rules to correlate physical telemetry with logical evidence.2 |
| **AI-ENG-AA** | Evals Architecture.1 | Golden Set Relevance Density (100.0%).1 | Feeds production failure traces and user corrections directly into versioned golden test sets.1 |
| **AI-ENG-Y** | Human-in-the-Loop Governance.5 | Escalation Transition Delay (< 45 seconds).5 | Uses the Escalation Packaging Model to deliver rich, redacted evidence histories to manual review queues.5 |

#### **Works cited**

1. AI-ENG-AA — Evals Architecture: Ground Truth, Golden Sets & Regression Tests  
2. AI-ENG-Z — Strategic Telemetry - Traces, Tokens, Latency & Semantic Drift  
3. Recon.AI — Trust Accounting Platform for AI systems, accessed June 11, 2026, [https://reconai.net/](https://reconai.net/)  
4. Content-Addressed Evidence Storage & Preservation ... - SEC.gov, accessed June 11, 2026, [https://www.sec.gov/files/ctf-written-fcck-pilot-evidence-02-16-2026.pdf](https://www.sec.gov/files/ctf-written-fcck-pilot-evidence-02-16-2026.pdf)  
5. [KNOWLEDGE] - AI Engineering - Volume 8. W-Y Resilience, Degraded Modes, and Human Trust  
6. Provenance only gets attention after a messy document case : r/AI_Agents - Reddit, accessed June 11, 2026, [https://www.reddit.com/r/AI_Agents/comments/1sco94r/provenance_only_gets_attention_after_a_messy/](https://www.reddit.com/r/AI_Agents/comments/1sco94r/provenance_only_gets_attention_after_a_messy/)  
7. Model Lineage and Reproducibility: Tracking Provenance Across the ML Lifecycle, accessed June 11, 2026, [https://agility-at-scale.com/ai/governance/model-lineage-and-reproducibility/](https://agility-at-scale.com/ai/governance/model-lineage-and-reproducibility/)  
8. Trace concepts - Azure Databricks | Microsoft Learn, accessed June 11, 2026, [https://learn.microsoft.com/en-us/azure/databricks/mlflow3/genai/tracing/tracing-101](https://learn.microsoft.com/en-us/azure/databricks/mlflow3/genai/tracing/tracing-101)  
9. OpenInference Specification - GitHub Pages, accessed June 11, 2026, [https://arize-ai.github.io/openinference/spec/](https://arize-ai.github.io/openinference/spec/)  
10. Trace Concepts | MLflow AI Platform, accessed June 11, 2026, [https://mlflow.org/docs/latest/genai/concepts/trace/](https://mlflow.org/docs/latest/genai/concepts/trace/)  
11. Propagation - Python - OpenTelemetry, accessed June 11, 2026, [https://opentelemetry.io/docs/languages/python/propagation/](https://opentelemetry.io/docs/languages/python/propagation/)  
12. What is a C2PA Manifest? Structure, Assertions, and Verification, accessed June 11, 2026, [https://c2paviewer.com/articles/what-is-c2pa-manifest](https://c2paviewer.com/articles/what-is-c2pa-manifest)  
13. In-toto Attestations in Secure Pipelines - Emergent Mind, accessed June 11, 2026, [https://www.emergentmind.com/topics/in-toto-attestations](https://www.emergentmind.com/topics/in-toto-attestations)  
14. Provenance Driven Identity Trust Architecture, accessed June 11, 2026, [https://identitymanagementinstitute.org/provenance-driven-identity-trust-architecture/](https://identitymanagementinstitute.org/provenance-driven-identity-trust-architecture/)  
15. Technology - Ar.io, accessed June 11, 2026, [https://ar.io/technology/](https://ar.io/technology/)  
16. C2PA Implementation Guidance, accessed June 11, 2026, [https://spec.c2pa.org/specifications/specifications/2.4/guidance/Guidance.html](https://spec.c2pa.org/specifications/specifications/2.4/guidance/Guidance.html)  
17. Blockchain-enforced data lineage architectures with formal verification workflows, accessed June 11, 2026, [https://ijsra.net/sites/default/files/fulltext_pdf/IJSRA-2023-0559.pdf](https://ijsra.net/sites/default/files/fulltext_pdf/IJSRA-2023-0559.pdf)  
18. Defeating Nondeterminism in LLM Inference - Simon Willison's Weblog, accessed June 11, 2026, [https://simonwillison.net/2025/Sep/11/defeating-nondeterminism/](https://simonwillison.net/2025/Sep/11/defeating-nondeterminism/)  
19. Defeating Nondeterminism in LLM Inference - Thinking Machines Lab, accessed June 11, 2026, [https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/](https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/)  
20. Defeating Nondeterminism in LLM Inference - OpenAI Developer Community, accessed June 11, 2026, [https://community.openai.com/t/defeating-nondeterminism-in-llm-inference/1358623](https://community.openai.com/t/defeating-nondeterminism-in-llm-inference/1358623)  
21. Openinference Semantic Conventions - Arize AX Docs, accessed June 11, 2026, [https://arize.com/docs/ax/observe/tracing-concepts/openinference-semantic-conventions](https://arize.com/docs/ax/observe/tracing-concepts/openinference-semantic-conventions)  
22. Poisoning Attacks in Federated Learning: An Accountability-Oriented Survey with Centralized Learning as Baseline - Preprints.org, accessed June 11, 2026, [https://www.preprints.org/manuscript/202605.1674](https://www.preprints.org/manuscript/202605.1674)  
23. C2PA for Developers: Implementation Guide, accessed June 11, 2026, [https://c2pa.ai/for-developers](https://c2pa.ai/for-developers)  
24. An 84-Format Numeric Catalog with Bit-Exact Conformance Vectors: A Vendor-Neutral Reference for FP8, BF16, MXFP4, and Microscaling Formats - arXiv, accessed June 11, 2026, [https://arxiv.org/html/2606.09686v1](https://arxiv.org/html/2606.09686v1)  
25. Fighting Misinformation with Authenticated C2PA Provenance Metadata - IPTC, accessed June 11, 2026, [https://iptc.org/std/MediaProvenance/Documents/NAB%20Paper%20-%20Formatted%20Version.pdf](https://iptc.org/std/MediaProvenance/Documents/NAB%20Paper%20-%20Formatted%20Version.pdf)  
26. envelope.md - in-toto/attestation - GitHub, accessed June 11, 2026, [https://github.com/in-toto/attestation/blob/main/spec/v1/envelope.md](https://github.com/in-toto/attestation/blob/main/spec/v1/envelope.md)  
27. mlflow.tracing, accessed June 11, 2026, [https://mlflow.org/docs/latest/api_reference/python_api/mlflow.tracing.html](https://mlflow.org/docs/latest/api_reference/python_api/mlflow.tracing.html)  
28. Attribute Mapping Reference | MLflow AI Platform, accessed June 11, 2026, [https://mlflow.org/docs/latest/genai/tracing/opentelemetry/attribute-mapping/](https://mlflow.org/docs/latest/genai/tracing/opentelemetry/attribute-mapping/)  
29. Agent Trace Evaluation with TruLens Scorers in MLflow, accessed June 11, 2026, [https://mlflow.org/blog/mlflow-trulens-evaluation/](https://mlflow.org/blog/mlflow-trulens-evaluation/)  
30. From Attack Simulation to SIEM Rule: Deterministic Detection-as-Code Synthesis with Probe-Level Traceability - arXiv, accessed June 11, 2026, [https://arxiv.org/pdf/2606.05252](https://arxiv.org/pdf/2606.05252)  
31. [KNOWLEDGE] - AI Engineering - Volume 7. S-V Failure, Security, and Hostile Environments  
32. What do you use for tamper-evident audit logs? Looking for approaches beyond "ship to S3", accessed June 11, 2026, [https://www.reddit.com/r/Observability/comments/1rwm9vh/what_do_you_use_for_tamperevident_audit_logs/](https://www.reddit.com/r/Observability/comments/1rwm9vh/what_do_you_use_for_tamperevident_audit_logs/)  
33. QCIVET: A Quantum–Classical Pipeline Integrity Framework with Contract-Based Subtype Verification and Hash-Chained Audit Traces - arXiv, accessed June 11, 2026, [https://arxiv.org/html/2605.13109v1](https://arxiv.org/html/2605.13109v1)  
34. Cosign Projects - AI Tinkerers - Bogotá, accessed June 11, 2026, [https://bogota.aitinkerers.org/technologies/cosign](https://bogota.aitinkerers.org/technologies/cosign)  
35. What Is Sigstore? Keyless Signing for the Software Supply Chain - Sbomify, accessed June 11, 2026, [https://sbomify.com/2024/08/12/what-is-sigstore/](https://sbomify.com/2024/08/12/what-is-sigstore/)  
36. New SIG proposal: E2E Model Provenance · Issue #40 · ossf/ai-ml-security - GitHub, accessed June 11, 2026, [https://github.com/ossf/ai-ml-security/issues/40](https://github.com/ossf/ai-ml-security/issues/40)  
37. Docker Image signing and attestation with Cosign - AugmentedMind.de, accessed June 11, 2026, [https://www.augmentedmind.de/2025/03/02/docker-image-signing-with-cosign/](https://www.augmentedmind.de/2025/03/02/docker-image-signing-with-cosign/)  
38. Guidance for Artificial Intelligence and Machine Learning - C2PA Specifications, accessed June 11, 2026, [https://spec.c2pa.org/specifications/specifications/2.4/ai-ml/ai_ml.html](https://spec.c2pa.org/specifications/specifications/2.4/ai-ml/ai_ml.html)  
39. A Study on Broker-Assisted Blockchain Trust Chains for Provenance and Integrity Verification of Generative Media Using Watermarking, Semantic Fingerprinting, and C2PA - MDPI, accessed June 11, 2026, [https://www.mdpi.com/2076-3417/16/7/3391](https://www.mdpi.com/2076-3417/16/7/3391)  
40. Evidentia™ T1 — Institutional Evidence Infrastructure, accessed June 11, 2026, [https://evidentiat1.com/](https://evidentiat1.com/)  
41. (PDF) From Policy to Pipeline: A Governance Framework for AI Development and Operations Pipelines - ResearchGate, accessed June 11, 2026, [https://www.researchgate.net/publication/399025393_From_Policy_to_Pipeline_A_Governance_Framework_for_AI_Development_and_Operations_Pipelines](https://www.researchgate.net/publication/399025393_From_Policy_to_Pipeline_A_Governance_Framework_for_AI_Development_and_Operations_Pipelines)

---

[← Back to Canon Map](../canon-map.md)