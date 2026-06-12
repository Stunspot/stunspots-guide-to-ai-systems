# AI-ENG-Q — Speech, Voice, and Real-Time Interaction Systems

Real-time voice systems represent a fundamental departure from traditional request-response text architectures. In a text-based system, information arrives as a complete, static artifact with explicit, deterministic boundaries. In contrast, spoken human interaction is a continuous, bi-directional, temporal stream where timing, pauses, overlaps, pitch, and physical acoustics carry critical semantic meaning.1 A real-time voice interaction system cannot be built merely by wrapping a text-based Large Language Model (LLM) in batch transcription and text-to-speech APIs.4 When voice interfaces are treated as text generation with audio input/output bolted on, the resulting systems fail under human conversational norms, manifesting high latency, false endpointing, fragile interruption mechanics, and unsafe tool execution on unstable transcripts.4  
The foundational architecture of a high-dimensional real-time voice system is therefore defined as a temporal state-coordination system. This architecture must process continuous streaming media under strict latency budgets, manage the conversational floor across overlapping speaker turns, filter background noise and echo to allow natural user barge-in, repair verbal misunderstandings without degrading progressivity, verify identity, and protect agentic tools from executing on partial or misheard intents.4  
This report establishes the system architecture, state models, and operational profiles required to build resilient, latency-bounded, and safe real-time voice systems.

## **Conceptual Glossary**

The following glossary defines the core terms and operational metrics governing high-dimensional speech, voice, and real-time interaction systems.

| Term | Technical Definition | Primary Operational Metric | Standard Production Target |
| :---- | :---- | :---- | :---- |
| **Speech-to-Text (STT)** | The conversion of continuous acoustic audio frames into structured text tokens or word hypotheses via streaming neural decoders.10 | Word Error Rate (WER) / Entity-specific WER 12 | < 5% English WER, < 2% Entity-WER 12 |
| **Text-to-Speech (TTS)** | The synthesis of structured text tokens into audible streaming waveforms with specified prosodic and expressive parameters.10 | Time-to-First-Audio (TTFA) / Mean Opinion Score (MOS) 4 | < 200 ms TTFA, > 4.2 MOS 4 |
| **Realtime Session** | A persistent, stateful, bi-directional network connection handling concurrent media streaming, event signaling, and state updates.15 | Jitter / Packet Loss / Session Reconnection Latency 14 | < 10 ms Jitter, 0% Packet Loss, < 30 s Reconnect 14 |
| **Streaming Transcription** | The ongoing emission of transcripts as audio frames arrive, returning both unstable intermediate guesses and locked historical frames.10 | Partial-to-Final Latency / Revision Rate 6 | < 150 ms Partial Latency, < 5% Revision Rate 12 |
| **Partial Transcript** | A speculative, volatile transcript hypothesis generated from incomplete audio context, subject to downstream correction by the STT engine.4 | Stability index / Token volatility 6 | Display-only; never sent to high-risk tools 4 |
| **Final Transcript** | An immutable, locked transcript segment emitted once the STT decoder establishes high confidence over the accumulated acoustic context.4 | Segment Commit Latency 4 | 300 ms to 500 ms post-utterance 4 |
| **Transcript Stability** | A probabilistic metric indicating the likelihood that currently generated partial transcript tokens will match the final emitted tokens.6 | Stability Confidence Score 6 | Threshold > 0.85 for speculative execution 6 |
| **Voice Activity Detection (VAD)** | A lightweight binary classification process identifying the presence or absence of human speech within short, sequential audio frames.1 | Frame-level Precision/Recall, False Trigger Rate 18 | Precision > 0.98, False Trigger < 2 per hour 1 |
| **Endpointing** | The decision logic determining whether a speaker has completed a conversational turn based on silence, prosodic cues, and semantic completeness.1 | Turn-End Detection Latency / Cutoff Error Rate 4 | < 300 ms Latency, < 1.5% Cutoff Error 4 |
| **Turn-Taking** | The protocol and state machine governing conversational floor transitions, preventing simultaneous speech while maintaining responsiveness.22 | Turn Gap Duration / Overtalk Overlap Time 22 | 200 ms to 450 ms Turn Gap 5 |
| **Backchannel** | Minimal vocal gestures (e.g., "uh-huh", "mm-hmm") emitted to indicate active listening and floor preservation without claiming the floor. | Backchannel Precision / False Interruption Rate | Precision > 0.90 under noise |
| **Barge-In** | The user action of speaking while the agent is playing audio, requiring the agent to immediately halt output and process the new input.5 | Interruption Detection Latency / Playback Flush Time 5 | < 150 ms from Speech Onset to TTS Flush 5 |
| **Interruption** | A conversational event where one speaker's active turn is terminated prematurely by the speech onset of another speaker.5 | Interruption False Alarm Rate | < 1.0% on environmental noise/laughter |
| **Floor Holding** | Acoustic and linguistic signals used by a speaker to retain the conversational floor during pauses, preventing premature agent endpointing.22 | Hold vs Shift Classification Accuracy 22 | Hold classification > 0.95 on trailing fillers |
| **Conversational Repair** | Interactive sequences where the system or user resolves speaking, hearing, or understanding breakdowns to restore mutual comprehension.7 | Repair Completion Rate / Conversation Abandonment Rate 26 | > 92% Repair Success, < 3% Abandonment 26 |
| **Speaker Diarization** | The process of partitioning an audio stream into distinct segments associated with individual speaker identities ("who spoke when").3 | Diarization Error Rate (DER) / Turn-Boundary DER 3 | DER < 10% in multi-speaker environments 3 |
| **Speaker Recognition** | The biometric verification or identification of a speaker based on the unique acoustic characteristics of their voice profile.3 | Equal Error Rate (EER) / False Acceptance Rate (FAR) 27 | EER < 0.1% under clean conditions 27 |
| **Voice Identity** | The unique, auditable representation of a speaker's vocal characteristics used for access control, styling, or authentication.3 | Identity Match Confidence / Authentication F1-score 27 | F1-score > 0.99 for gated mutations 27 |
| **Synthetic Voice** | Programmatically synthesized speech designed to sound like a specific human or styled persona.10 | Naturalness MOS / Persona Alignment Score 10 | MOS > 4.3 10 |
| **Latency Budget** | The end-to-end allocation of time segments across all processing components in a streaming interaction loop.4 | End-to-End Latency (T_streaming) 4 | Target < 700 ms, Hard Limit < 1000 ms 4 |
| **Voice-to-Action Gating** | The protective verification layers preventing unconfirmed, volatile, or unauthorized speech from triggering high-impact system mutations. | Gating Precision / Unintended Action Rate | 100% precision on irreversible tools |

## **Realtime Voice Interaction Pipeline**

A production-grade real-time voice system operates as a distributed, stream-aligned architecture. The end-to-end pipeline must coordinate multiple asynchronous systems, each running on distinct clocks, over low-latency media transport protocols.4

```
+--------------------------------------------------------------------------------------------------------+  
|                                  REALTIME VOICE INTERACTION PIPELINE                                   |  
+--------------------------------------------------------------------------------------------------------+  
|                                                                                                        |  
|                                                                                    |  
|  Microphone Capture ---> WebRTC AEC3 ---> Opus Encoder ---> WebRTC Transceiver Edge                    |  
|                                                                     |                                  |  
|                                                                     v                                  |  
|                                                                                 |  
|  WebRTC Termination/Decrypt (DTLS/SRTP) <--- Routing on ICE credentials (ufrag hint)                  |  
|          |                                                                                             |  
|          ├──────> Edge VAD (Wasm/Silero) ───> Instant Client Mute / Playback Buffer Flush              |  
|          │                                                                                             |  
|          └──────> Audio Stream (PCM Int16, 16kHz)                                                      |  
|                          |                                                                             |  
|                          v                                                                             |  
|                                                                            |  
|  Streaming STT (Deepgram/AssemblyAI)                                                                   |  
|          ├──────> Partial Hypotheses (is_final=False) ───> Prewarming / Speculative Planning           |  
|          └──────> Final Utterances (is_final=True)                                                     |  
|                          |                                                                             |  
|                          v                                                                             |  
|                                                                          |  
|  Turn-Taking Dialogue State Machine                                                                    |  
|          ├──────> Spoken Turn Completed (speech_final=True)                                            |  
|          ├──────> Think-Before-Speak Agent Loop                                                        |  
|          │               ├──────> Tool Gating & Execution (Confirmed Payloads Only)                    |  
|          │               └──────> Context / History Preservation                                       |  
|          │                                                                                             |  
|          ▼                                                                                             |  
|                                                                         |  
|  Streaming LLM Tokens ───> Sentence Buffer ───> Streaming TTS ───> Playback Jitter Buffer ───> Speaker |  
|                                                                                                        |  
+--------------------------------------------------------------------------------------------------------+
```

### **Media Transport Infrastructure and Session Edge Architecture**

Traditional request-response architectures terminate traffic using standard stateless HTTP endpoints. For real-time voice, bidirectional transport must bypass standard TCP handshakes to avoid the latency penalties of congestion control and packet retransmission.4 Production systems implement a hybrid architecture of WebRTC and WebSockets 31:

* **WebSockets**: Effective for control signaling, session setup, and low-concurrency streaming where packet-loss handling (TCP) does not introduce latency cliffs under clean network conditions.4  
* **WebRTC**: The standard for production-grade, high-scale bidirectional media transport.31 WebRTC utilizes UDP as its base transport, wrapping packets in Secure Real-time Transport Protocol (SRTP) for media and Datagram Transport Layer Security (DTLS) for key exchange.30

To avoid the overhead of multi-party Selective Forwarding Units (SFUs) where the AI acts as a participant in a group session, production systems for 1:1 interactions use a **Transceiver Edge Model**.30 In this model, a geographically distributed edge service terminates downstream WebRTC connections directly, handling Interactive Connectivity Establishment (ICE), DTLS handshakes, and SRTP decryption at the edge.30 The decrypted media is then converted into lightweight, internal protocols (such as raw gRPC or flat UDP streams) for upstream routing to model inference and orchestration clusters.30  
To route incoming UDP media packets to the correct stateful transceiver instance without an expensive database lookup on every packet, systems route on **ICE Credentials**.30 During Session Description Protocol (SDP) negotiation, the edge service generates a unique ICE username fragment (ufrag) containing encoded routing metadata.30 When the client sends its first STUN binding request to establish connectivity, the stateless network load balancers parse the ufrag from the STUN header and steer the UDP stream to the designated transceiver container.30 If a transceiver or relay restarts, the next incoming STUN packet rebuilds the path instantly based on the ufrag routing hint.30

### **Audio Evidence Provenance**

To maintain consistency and auditability, real-time voice systems must preserve a continuous trace mapping generated text tokens and tool calls back to the raw source audio segments. Every audio frame ingested by the transceiver edge is assigned a monotonically increasing timestamp and sequence identifier. The streaming STT engine maps word-level start and end times back to these frame offsets.4  
When the dialogue coordinator transitions its state or triggers an external tool contract, it logs the precise audio segment boundaries and transcript stability states that justified the action. This ensures that every mutation can be audited back to the raw physical signal, protecting against hallucinations or errors where a tool executes on ungrounded text.

### **Media Plane and Signal Plane Execution Pipeline**

The execution flow of the real-time voice pipeline translates physical acoustic pressure into state transformations and back into physical acoustics.4 The pipeline segments these responsibilities into distinct software layers.

| Sequence Stage | Local Pipeline Step | Executing Infrastructure Layer | Local Protocol / Format | Critical Failure Risk |
| :---- | :---- | :---- | :---- | :---- |
| **1. Capture** | Microphone Audio Ingestion | Client-side App (Native SDK) | Linear PCM 16-bit, 16kHz, Mono 33 | Audio capture permission block 28 |
| **2. Preprocess** | WebRTC AEC3 Echo & Noise Filtering | Client AudioWorklet (WebAssembly) 34 | Float32 Internal -> PCM conversion 34 | Adaptive filter divergence on Android 35 |
| **3. Encode** | Audio Frame Compression | Client Opus Encoder | Opus packets over UDP (WebRTC) 34 | Packet dropped due to local thread lag 34 |
| **4. Transport** | Secure Packet Uplink Transit | Public/Transit IP Network | DTLS/SRTP via WebRTC Transceiver 30 | Network jitter, cross-region routing delay 4 |
| **5. Terminate** | WebRTC termination & Decryption | Server Edge (Transceiver Cluster) 30 | Decrypted PCM Int16 raw buffer 30 | Ephemeral port exhaustion under Kubernetes 30 |
| **6. Local VAD** | Client/Server Double VAD pass | Server Edge (Silero Wasm Thread) 36 | 10/20ms Frame classification 18 | High False Positives on street background noise 5 |
| **7. Stream STT** | Acoustic-to-Linguistic Decoding | STT Engine (WebSocket Pool) 11 | Bidirectional WebSocket JSON 11 | Model drift, phonetic spelling failure 10 |
| **8. Revise** | Partial Transcript update cycle | Dialogue Context Manager | Interim results collection (is_final=False) 6 | Premature execution on highly volatile phrases 6 |
| **9. Endpoint** | Spoken turn completion parsing | Semantic Turn Classifier | Semantic & Prosodic Feature Scoring | Cutting off users with assistive stutters |
| **10. Reason** | Think-Before-Speak execution | LLM Agent Execution Cluster 38 | In-memory token representation 38 | Context window token ceiling, high TTFT |
| **11. Gate** | Spoken action authorization check | Security & Policy Layer | Action schema-to-transcript mapping | Executing destructive tool on misheard entity 9 |
| **12. Synthesize** | Neural speech output compilation | Streaming TTS Cluster (Sonic Engine) 5 | Opus/PCM stream segment output 5 | Audio gaps, mechanical voice sounding 4 |
| **13. Playback** | Waveform output conversion | Client App Playback Engine | PCM 16-bit, 24kHz, Mono 33 | Glitching "chipmunk effect" on mismatch 33 |
| **14. Log & Trace** | Comprehensive Session auditing | Observability Storage Platform 28 | Structured OpenTelemetry session JSON 28 | Exposing raw audio leaks, PI data breach 28 |

## **Speech-to-Text and Streaming Transcription**

Real-time transcription is more than batch speech recognition optimized for speed; it is an active perception pipeline that processes incomplete, overlapping, and changing audio structures.

### **Streaming STT State Model**

The streaming STT decoder operates as a stateful transition engine over rolling audio frames.6 It translates continuous acoustic feature vectors into structured linguistic hypotheses, distinguishing between unstable partial guesses and locked historical frames.6

| Origin State | Transition Trigger Event | Destination State | Retained State Metadata | Downstream System Action |
| :---- | :---- | :---- | :---- | :---- |
| **Acoustic Idle** | Energy level crosses VAD threshold 1 | **Active Frame Ingestion** | Prefix buffer, timestamp onset 5 | Open active STT decoder socket 11 |
| **Active Frame Ingestion** | First frame sequence completed (> 100 ms) 40 | **Partial Hypothesis** (is_final=False) 6 | Speculative text array, confidence score 6 | Render real-time UI captions 34 |
| **Partial Hypothesis** | Subsequent audio frame arrival 6 | **Hypothesis Revision Loop** | Word-level coordinate shifts 32 | Refine speculative intent classification 1 |
| **Hypothesis Revision Loop** | Acoustic probability entropy decreases below threshold 6 | **Locked Segment** (is_final=True) 6 | Solidified token sequence, word timestamps 41 | Append immutable text to LLM context 4 |
| **Locked Segment** | VAD registers silence > threshold 39 | **Utterance Boundary** (speech_final=True) 6 | Final turn sequence number, semantic tag 6 | Trigger model turn reasoning thread 4 |
| **Utterance Boundary** | Core agent response generated 4 | **Acoustic Idle** | Historical dialogue logs, tool receipts 28 | Transition client state back to listening 5 |

### **Transcript Stability Policy**

Because interim results are volatile, acting on them prematurely introduces systemic risk.6 For instance, if a user says, "I want to cancel... my cancellation request," a system acting on the first partial transcript ("I want to cancel") might trigger a destructive account mutation before the corrective phrase ("my cancellation request") is ever processed.6 To mitigate this, real-time architectures enforce a strict **Transcript Stability Policy** that maps permitted system behaviors to transcript states.

| Transcript State | Stability Criteria | Permitted Actions | Prohibited Actions | Recovery Protocol |
| :---- | :---- | :---- | :---- | :---- |
| **Volatile Partial** (is_final: false, Stability < 0.70) 6 | Initial phoneme matching, highly subject to downstream linguistic revision.6 | Local UI caption rendering, speculative intent classification.6 | LLM context serialization, tool execution, external state mutations. | Silently discard the segment if updated frames diverge.6 |
| **Stable Speculative** (is_final: false, Stability >= 0.70) 6 | High acoustic confidence, but waiting for linguistic boundary resolution.11 | Pre-fetching read-only API data, pre-warming vector indexes.4 | Dispatching user replies, committing payments, updating external records. | Flush speculative state cache if final result differs.5 |
| **Locked Segment** (is_final: true, speech_final: false) 6 | Decoder has finalized the phoneme-to-text mapping.6 | Appending text to conversational history, routing to reasoning LLM.4 | Triggering irreversible tools without explicit spoken confirmation. | Wait for final utterance boundary or explicit pause.6 |
| **Utterance Complete** (is_final: true, speech_final: true) 6 | Segment is finalized and the user has executed a valid turn boundary.6 | Executing safe read-only queries, triggering standard dialog response.42 | Mutating critical user accounts or executing high-impact tools. | Validate turn completion against semantic VAD before speaking. |

### **Multilingual Dynamics and Keyterm Prompting**

Under multilingual and code-switching constraints, the STT engine must run dynamic, zero-latency language identification.12 If the speaker switches from English to Spanish mid-sentence, a monolingual decoder will hallucinate phonetic approximations, introducing transcription errors.12 Production systems employ parallel acoustic language-identification heads that dynamically adjust the decoder's language model matrix.12  
To maintain high accuracy for specialized, domain-specific vocabularies (such as product names, medical codes, or user-specific contact lists), the STT connection is initialized with a **Keyterm Prompting List**.12 This dynamically biases the decoder's decoding graph towards specified proper nouns or acronyms, reducing the Word Error Rate on critical nouns from over 18% to under 1.5%.12

## **Voice Activity Detection, Endpointing, and Turn Boundaries**

Voice Activity Detection (VAD) and Endpointing are distinct control systems within the real-time interaction loop.20 VAD is a low-level, frame-by-frame acoustic classifier that determines the presence or absence of human speech.1 Endpointing is a higher-level, context-aware decision process that determines whether the user has completed their conversational turn and yielded the floor to the assistant.1

### **Endpointing and Turn-Detection Model**

Standard energy-based VAD systems rely on simple volume thresholds: if the audio signal amplitude drops below a specified decibel level (typically -40 dBFS) for a static duration (e.g., 500 ms), the system assumes the user has stopped speaking.44 In production environments, this approach fails.1 Background noise (such as coffee-shop hum or traffic) can keep energy levels above the threshold, preventing the agent from ever responding.1 Conversely, when a user pauses mid-thought to compose a sentence or breathes heavily, energy-based VAD registers silence and prematurely endpoints the turn, cutting the user off mid-sentence.1  
Modern architectures deploy a **Hybrid Semantic-Acoustic Endpointing Model**. This system combines low-latency neural acoustic VAD (e.g., Silero VAD running at < 1 ms inference per 32 ms frame) with a lightweight, high-speed language classification model. This model analyzes the rolling partial transcript to evaluate three primary cues:

* **Linguistic Completeness**: Evaluating if the emitted token sequence represents a syntactically resolved clause.  
* **Prosodic Markers**: Processing pitch contours and trailing intonation to determine if the speaker’s voice is rising (indicating an unfinished thought or question) or falling (indicating completion).22  
* **Conversational Fillers**: Detecting floor-holding tokens such as "um", "uh", or trailing conjunctions like "and then...".

```
                +---------------------------------------+  
                |          Continuous Audio             |  
                +---------------------------------------+  
                                    |  
                                    v  
                +---------------------------------------+  
                |    Neural Acoustic VAD (Silero)       |  (Outputs frame speech probability p)   
                +---------------------------------------+  
                                    |  
                                    +-----------------------+  
                                    |                       | (p < 0.5) 45]  
                                    | (p >= 0.5) 45]    v  
                                    |             +-----------------------+  
                                    |             |  Silence Accumulator  |   
                                    |             +-----------------------+  
                                    |                       |  
                                    |                       | (Silence > min_turn_silence)   
                                    |                       v  
                                    |             +-----------------------+  
                                    |             | Semantic Classifier / |  
                                    |             |  Punctuation Check    |   
                                    |             +-----------------------+  
                                    |               |               |  
                                    |               | (Complete)    | (Incomplete / Fillers)  
                                    |               v               v  
                                    |         +-----------+   +---------------+  
                                    |         |   Emit    |   | Extend Silence| (Wait up to max_turn_silence)   
                                    |         | Endpoint  |   |   Threshold   | 20]  
                                    |         +-----------+   +---------------+  
                                    v               ^               |  
                        +-----------------------+   |               |  
                        |      Speech Onset     |───+               v  
                        |    Tracker / Reset    | <─────────────────+ (New speech detected)   
                        +-----------------------+
```

When silence is detected acoustically, the system calculates a dynamic hangover window:

* If the partial transcript is linguistically incomplete or ends with a filler ("I want to go to..."), the hangover threshold is automatically extended to 1500 ms to give the user time to speak.  
* If the transcript represents a structurally complete statement ("Book the flight."), the system triggers endpointing after a minimal 300 ms delay, achieving rapid response times without cutting off complex utterances.

To programmatically control this behavior, production platforms leverage configuration parameters across two primary models 39:

#### **OpenAI Realtime API VAD Properties**

* **threshold** (0.0 to 1.0): Sets the activation confidence for acoustic speech detection.39 Higher values prevent false triggers in noisy rooms.39  
* **prefix_padding_ms**: Controls the amount of audio buffered before the speech onset trigger, preserving early consonant bursts (attacks) for STT decoders.5  
* **silence_duration_ms**: In server_vad mode, sets the static silence duration required to trigger a turn completion.39  
* **eagerness** (low, auto, high): In semantic_vad mode, dynamically tunes the wait threshold based on linguistic context.39 low allows users to take their time and speak slowly, while high forces aggressive, low-latency turn-ending.39

#### **AssemblyAI Universal-3 Pro Punctuation-Based Turn Detection**

* **min_turn_silence**: The duration of silence required before the model runs a speculative punctuation-check.21 If terminal punctuation (e.g., ., ?, !) is detected in the speculative transcript at this point, end_of_turn: true is emitted immediately.21  
* **max_turn_silence**: If no terminal punctuation is found, the system holds the floor open until silence reaches this maximum threshold, at which point the turn is forced to close.21  
* **interruption_delay**: Controls how quickly the first partial transcript is emitted to downstream systems.21

### **Endpointing Policy Profiles**

A single endpointing configuration cannot satisfy all conversational contexts. A fast response is desirable for home automation commands, but causes frustration during complex cognitive tasks or for users with speech impairments.1 Architectures must implement specialized **Endpointing Policy Profiles**.1

| Profile Name | Target Use Case | Silence Threshold (T_silence) | VAD Sensitivity / Energy Threshold | Semantic Guarding Policy | Target User Segment / Context |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Command & Control** | Voice-driven utility execution (e.g., "turn on lights"). | 200 ms - 300 ms 19 | High sensitivity / Low energy threshold (aggressive).39 | Disabled; execute instantly on keyword detection.39 | Hands-busy environments, low cognitive overhead.46 |
| **Casual Conversation** | Natural, open-ended dialog.4 | 400 ms - 600 ms 19 | Moderate sensitivity (balance noise vs speech).18 | Enabled; verify clause resolution before yielding floor. | Standard customer service, informational voicebots.4 |
| **Dictation / Transcription** | Continuous spoken capture, long paragraphs.10 | 1200 ms - 2000 ms 21 | High sensitivity; long hangover padding.17 | Disabled; wait for explicit "period" keyword or long pause.21 | Legal, medical, or administrative text entry.10 |
| **High-Impact Action** | Financial mutations, secure database changes.28 | 800 ms - 1200 ms 19 | Low sensitivity (avoid false starts on noise).39 | Strict; require full grammatical resolution plus readback. | Banking, secure credential updates, account changes.28 |
| **Inclusive / Accessibility** | Support stutters, slow pacing, dysarthria.46 | 1500 ms - 3000 ms 19 | High tolerance; ignore rapid repetitive syllable bursts. | Strict; disable endpointing on repetitive sound patterns. | Users with neurological speech differences.46 |
| **Noisy Environment** | Cafes, open offices, industrial floors.17 | 600 ms - 1000 ms 19 | Low sensitivity / High energy threshold (WebRTC level 3).19 | Enabled; filter out sudden ambient sound spikes.1 | Mobile users, field operations, public transport.17 |
| **Group Conversation** | Multi-speaker business meetings, panels.3 | 800 ms - 1500 ms 3 | High sensitivity + Speaker Diarization mapping.3 | Strict; keep turn open if primary speaker pauses but holds floor.1 | Corporate boardrooms, multi-party calls.3 |

## **Turn-Taking and Dialogue State**

Turn-taking is managed by a state machine that tracks floor ownership. The system must prevent overtalk (where both speak simultaneously) and dead air (where neither speaks), transitioning states based on VAD, transcript stability, and semantic completions.22

### **Turn-Taking State Machine**

The following matrix defines the state transitions of the Turn-Taking State Machine.

| Source State | Input Event Trigger | Destination State | Triggering Mechanism | State Machine System Actions |
| :---- | :---- | :---- | :---- | :---- |
| **IDLE** | VAD Speech Started | **LISTENING** | Audio energy level crosses VAD threshold 1 | Open streaming ASR connection, start context buffering.4 |
| **LISTENING** | Partial Transcript Received | **PROVISIONAL_TRANSCRIPT** | Interim results payload parsed from socket 6 | Render real-time visual captions, pre-warm caches.4 |
| **PROVISIONAL_TRANSCRIPT** | VAD Silence > threshold | **ENDPOINT_CANDIDATE** | Local silence timer passes 300 ms window 19 | Halt active transcript appending, trigger turn completion model. |
| **ENDPOINT_CANDIDATE** | Semantic Resolution Completed | **USER_TURN_FINAL** | Turn completeness model returns probability > 0.85 | Lock finalized ASR transcript segment, compile prompt.6 |
| **ENDPOINT_CANDIDATE** | New Speech Detected | **LISTENING** | Audio energy level crosses threshold before timeout 1 | Reset local silence timer, continue transcript stream.1 |
| **ENDPOINT_CANDIDATE** | Timeout Exceeded | **USER_TURN_FINAL** | Silence duration passes forced floor timeout 21 | Force close the active speaking turn.21 |
| **USER_TURN_FINAL** | Dialogue Reasoning Invoked | **THINKING** | Prompt dispatched to core agent LLM 4 | Execute tool gating checks, initiate reasoning thread.9 |
| **THINKING** | API/Tool Execution Invoked | **TOOL_CHECKING** | LLM outputs speculative tool schema 4 | Suspend spoken generation, route parameters to tool runner.28 |
| **TOOL_CHECKING** | Tool Return Verified | **THINKING** | API response parsed and validated 4 | Append tool payload to context, resume linguistic path.4 |
| **THINKING** | First Linguistic Token Emitted | **SPEAKING** | LLM outputs structural response token | Launch streaming TTS pipeline, initialize client playback. |
| **SPEAKING** | LLM Stream Paused Mid-Turn | **BACKCHANNELING** | LLM outputs explicit wait or placeholder token | Play light acoustic feedback gesture ("uh-huh"). |
| **SPEAKING** | VAD Speech Started (Barge-In) | **INTERRUPTED** | User interrupts playing assistant stream 5 | Flush client-side audio buffer, cancel reasoning.5 |
| **SPEAKING** | Synthesis Complete | **IDLE** | Audio playback playhead reaches termination marker 1 | Reset dialogue state flags, return to passive listening.5 |
| **INTERRUPTED** | Interruption Processing Complete | **REPAIR** | Truncation playhead timestamp aligned 5 | Log partial response, load repair conversational patterns.5 |
| **REPAIR** | Misunderstanding Resolved | **THINKING** | User confirmation text committed 8 | Compile corrected prompt, restart generation thread.4 |
| **IDLE / LISTENING** | Critical Protocol Loss | **TERMINATED** | Media socket connection drops > 30 s 16 | Tear down WebRTC transceiver paths, log metrics.30 |

### **Conversational Dynamics**

To maintain natural dialogue flow, the dialogue coordinator must distinguish between different communicative actions:

* **User Turn**: The user has the floor and is transmitting semantic intent.4  
* **Assistant Turn**: The agent has the floor and is playing audio.  
* **Backchannel**: A non-intrusive vocal gesture (e.g., "uh-huh") emitted by the listener. When the user is speaking and emits a brief pause, the assistant may issue a backchannel to signal active listening. This does not trigger an assistant turn transition.  
* **Floor Holding**: The user pauses but emits acoustic filler signals ("uhhh") or grammatical continuity markers ("because...") indicating they intend to retain the floor.  
* **Turn Yield**: The active speaker completes their statement and explicitly yields the floor, marked by falling pitch contours and silence.22  
* **Barge-In / Overlap**: Both speakers speak simultaneously.5 The coordinator must classify this event: if the user's speech is a brief backchannel, the assistant continues speaking; if it is a structural correction or stop command, the assistant yields the floor immediately.  
* **Conversational Repair**: A sub-dialogue loop triggered when either party detects a communication breakdown, holding progressivity until mutual understanding is restored.7  
* **Confirmation Turn**: A dedicated turn where the system asks the user to confirm a high-risk entity or tool payload.  
* **Status Turn**: A temporary turn where the system informs the user of a background execution delay to prevent awkward silences.  
* **Handoff Turn**: An administrative turn transitioning floor control and context data to a human operator or fallback channel.

If the turn-taking state machine fails, several user-visible pathologies occur: **Overtalk** (the agent speaks over the user, causing frustration), **Dead Air** (the agent hesitates, leaving the user unsure if the connection is active), and **Clipped Speech** (the agent speaks too quickly, cutting off the end of the user's words).22

## **Latency Budgets and Realtime System Margins**

Human conversational turn-taking operates on a 200 ms to 500 ms rhythm.22 If a voice agent exceeds a 700 ms response delay, the interaction feels lagging or unresponsive.4 If it exceeds 1000 ms, the conversation breaks down as users assume a system error and repeat themselves, triggering race conditions.31

### **Realtime Latency Budget Model**

The end-to-end latency (T_streaming) of a real-time voice system is the cumulative sum of capture, transport, processing, and playback steps.4

```
+---------------------------------------------------------------------------------------------------------+  
|                                     REALTIME LATENCY BUDGET TIMELINE                                    |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|   0ms                   150ms                 350ms                 550ms                 700ms         |  
|   +---------------------+---------------------+---------------------+---------------------+         |  
|   |   Media Capture     |   Streaming STT     |   LLM First-Token   |   Streaming TTS     |         |  
|   |   & Transport       |   Processing        |   & Reasoning       |   Synthesis         |         |  
|   |   (80ms) | (200ms)    |   (200ms)     |   (120ms)     |         |  
|   +---------------------+---------------------+---------------------+---------------------+         |  
|                                                                                           |         |  
|                                                                                           v         |  
|                                                                                     Speaker Output  |  
|                                                                                     (< 700ms Target) |  
+---------------------------------------------------------------------------------------------------------+
```

To maintain this budget under real-world network conditions, latency margins must be allocated across the system components 4:

| Processing Component | Technical Processing Step | Latency Margin (P50) | Latency Margin (P95) | Architectural Optimization Strategy |
| :---- | :---- | :---- | :---- | :---- |
| **Media Capture** | Microphone hardware capture, client buffering | 10 ms | 20 ms | Bind audio capture to native client thread; restrict client-side buffer frame size to 20 ms.4 |
| **Acoustic Preprocessing** | WebRTC AEC3, spatial noise suppression | 15 ms | 30 ms | Compile AEC3 filters to native WebAssembly on client browser.34 |
| **Uplink Transport** | Opus encoding, DTLS/SRTP transit | 40 ms | 110 ms | Route media via Global Relay ingress points, avoiding multi-hop public IP transit.30 |
| **VAD & Endpointing** | Silero frame classification + semantic check | 35 ms | 75 ms | Execute neural VAD on server edge, caching prior tokens.20 |
| **Streaming STT** | Word lattice path decoding (Nova-3) | 210 ms | 320 ms | Keep persistent WebSocket connection open 4; avoid batch transcription APIs.40 |
| **Dialogue State Update** | Context assembly, prompt formatting | 5 ms | 12 ms | Keep dialogue context in-memory on state container.4 |
| **Model Reasoning** | First token execution on Groq/custom GPU | 120 ms | 220 ms | Prioritize model execution pathways to reduce TTFT. |
| **Tool Execution** | Parallel lookup database query execution | 180 ms | 380 ms | Run non-essential tools speculatively; pre-warm connections on partials.4 |
| **Streaming TTS** | Waveform synthesis on GPU clusters | 90 ms | 150 ms | Stream first output chunk (TTFA) before full sentence completes. |
| **Downlink Transport** | Audio frame routing back to client | 35 ms | 95 ms | Match downstream packet sizes to local audio output frames.34 |
| **Playback Buffer** | Web Audio API buffering, DAC output | 30 ms | 70 ms | Maintain minimal client jitter buffer size (< 60 ms).5 |
| **Interruption Stop** | Client-side playback mute on VAD trigger | 12 ms | 25 ms | Local mute inside AudioWorklet on user voice detection.34 |
| **Recovery Latency** | Dialogue state rollback, new path launch | 65 ms | 140 ms | Keep alternate linguistic pathways pre-compiled in memory.5 |

### **Pipeline Concurrency and Stream-Pipelining**

The critical path of a voice agent is optimized through **Stream-Pipelining**.4 In a sequential pipeline, the user waits for each stage to complete:

T_turn-based = T_STT + T_LLM + T_TTS ~ 400 ms + 800 ms + 400 ms = 1600 ms

In a streaming pipeline, processing stages overlap concurrently :

1. The STT engine streams partial and finalized segments continuously.6  
2. The dialogue orchestrator feeds completed words directly into the LLM context.4  
3. The LLM streams generated tokens token-by-token.4  
4. A **Sentence Buffer** (such as the SentenceAggregator) accumulates generated tokens until a clause or sentence boundary is detected (e.g., matching a punctuation mark).  
5. This completed sentence is dispatched to the TTS engine instantly, while the LLM continues generating subsequent sentences.  
6. The TTS engine streams audio chunks back to the client playback buffer while synthesizing the remainder of the response.

This overlaps generation and synthesis, reducing perceived latency to the first sentence:

T_streaming = T_STT + T_LLM-first-sentence + T_TTS-TTFB ~ 400 ms + 300 ms + 200 ms = 900 ms

The user hears the start of the response while the system is still computing the end of the sentence.

## **Streaming Generation and Speech Output Policy**

When an agent begins playing back audio, it commits to a public trajectory. If the language model subsequently changes its reasoning or encounters a tool error, the agent cannot retroactively edit its spoken output without a jarring verbal correction. Systems require a strict **Streaming Response Policy** to balance generation speed with truthfulness and action-safety boundaries.

### **Streaming Response Policy**

The dialogue engine evaluates the risk and dependencies of the requested turn before generating speech:

```
                  +-----------------------------------------+  
                  |           Committed Spoken Turn         |  
                  +-----------------------------------------+  
                                       |  
                                       v  
                  +-----------------------------------------+  
                  |       Analyze Action Dependencies       |  
                  +-----------------------------------------+  
                                       |  
                     ├─────────────────┼─────────────────┐  
                     | (Low Risk /     | (Read-Only      | (Mutation Tool /  
                     |  Chitchat)      |  Tool Call)     |  Irreversible Action)   
                     v                 v                 v  
          +------------------+ +------------------+ +------------------+  
          | Stream Speech    | | Stream Filler/   | | Wait Silently /  |  
          | Immediately      | | speculative turn | | Resolve Tool     |  
          | (No Buffer)      | | "Let me check..."| | (Verify Payload) |  
          +------------------+ +------------------+ +------------------+  
                     |                 |                 |  
                     |                 v                 |  
                     |         +------------------+      |  
                     |         |  Execute Tool    |      |  
                     |         |  Asynchronously  |      |  
                     |         +------------------+      |  
                     |                 |                 |  
                     v                 v                 v  
          +------------------------------------------------------------+  
          |                  Synthesize Verbal Output                  |  
          +------------------------------------------------------------+
```

To programmatically enforce these paths, the dialogue coordinator executes based on defined policy rules:

* **Low-Risk conversational path**: When the response requires no external tools or sensitive data lookup, the system streams tokens to TTS instantly.4  
* **Read-Only tool path**: When the query requires checking an external resource (e.g., checking an order status or flight time), the system streams a low-latency **Acoustic Filler or Verbal Preamble** (e.g., "Sure, let me look up that order for you..."). This filler plays while the system executes the tool asynchronously, masking execution latency. If the tool returns successfully, the system transitions to speaking the verified result.  
* **Mutation tool path**: When the requested action is destructive or irreversible (e.g., transferring funds, deleting an entry, or sending a message), **zero speech may be generated until the action is validated and committed**. The system must remain silent or provide a clear status update ("Processing your payment now...") and verify the state change before asserting that the action is complete. The spoken claim must never outrun verified reality.9

### **Speech Output UX Model**

Text formatted for visual layout is often unsuited for speech. Reading long code blocks, nested tables, markdown links, or dense raw numbers aloud increases cognitive load and causes conversational abandonment.17  
The speech generation engine must pass raw text through a **Speech Output Normalization Layer** before synthesis:

* **Numerical Normalization**: Translating raw digits into spoken representations based on context.10 For example, "Your account balance is $1520.50" is normalized to "Your account balance is fifteen twenty dollars and fifty cents," whereas "Your account number is 1520" is normalized to "Your account number is one five two zero".10  
* **Content Filtering**: Suppressing structural metadata.30 Markdown markers, image URLs, raw HTML, and complex terminal outputs are filtered out.10  
* **Alternative Presentation**: If the system must present dense multi-column tables, long bulleted lists, or sensitive personal data (such as passwords or full credit card numbers), the agent must redirect the content to a visual display or private text channel instead of reading it aloud.46

## **Text-to-Speech, Prosody, and Voice Behavior**

Text-to-Speech (TTS) models determine the vocal personality, warmth, and authority of the voice agent. Prosody (the rhythm, melody, and intonation of speech) carries emotional and syntactic weight, shaping user trust and understanding.10

### **TTS and Prosody Control Model**

To control synthetic voice output, the synthesis engine structures its commands using specialized parameters 10:

| Parameter Target | Control Value / Range | Algorithmic Mechanism | Target Output Behavior |
| :---- | :---- | :---- | :---- |
| **Voice Selection** | Corporate Profile Clones 27 | Neural speaker embedding mapping | Anchor brand personality.27 |
| **Speaking Rate** | 130 - 180 words/min 17 | Linear time-scaling DSP filters | Slow for numbers, fast for summaries.46 |
| **Pitch Contour** | -12 to +12 semitones 36 | F0 frequency-domain pitch shifting | Dynamic intonation peaks. |
| **Intonation Style** | Rising / Falling contours | Prosodic pitch trajectory modulation | Signal question vs firm statement.44 |
| **Pause Insertion** | 100 ms - 1200 ms 36 | Frame zero-padding injection 36 | Mimic breathing, separate sentences.36 |
| **Emphasis Index** | 1.0 to 1.5 scale factor | Dynamic amplitude & pitch scaling | Highlight critical dates or terms.25 |
| **Pronunciation** | Lexicon/IPA mappings 11 | Grapheme-to-phoneme override dictionaries | Prevent misreading industry jargon.11 |
| **Spelling Mode** | Character-by-character spelling 6 | Interleaved spelling character strings | Speak codes/names letter-by-letter.6 |
| **Chunking Logic** | Sentence boundary locks | Sentence-buffer tokenizer filters | Interruption-safe streaming audio. |
| **Emotional Tone** | empathetic, serious, excited 10 | Multi-style style-space embeddings 10 | Match caller sentiment.10 |
| **Multilingual Voice** | Native language profiles 11 | Dynamic cross-lingual phoneme sharing | Code-switch without accent drift.11 |
| **Audio Quality** | 24 kHz PCM mono 33 | High-fidelity sample rate rendering 33 | Studio clarity over WebRTC.34 |
| **Fallback Voice** | On-device SFSpeech synthesizers 36 | Dynamic on-client system switch | Retain function under network loss.36 |

### **Identity Transparency and Behavioral Boundaries**

Human-like synthetic voices can lead to overtrust, where users assume an agent has cognitive, emotional, or moral capabilities it does not possess.48 To maintain safety, systems must enforce **Identity Transparency Boundaries**:

* **Explicit Disclosure**: The agent must identify itself as an artificial system at the beginning of the interaction or whenever asked.  
* **Consent and Cloned Voices**: Voice cloning is prohibited unless the target individual has provided explicit, cryptographically verifiable authorization.27  
* **Public Impersonation Bans**: The system must block the synthesis of celebrity, public official, or unauthorized third-party voices, enforcing safety filters at the model level to reject unauthorized voice models.27

## **Interruption, Barge-In, and Cancellation**

A natural voice system must allow the user to interrupt the agent at any point during playback.5 If the user has to wait for the agent to finish reading a long response before they can correct an error, the interface becomes unusable.5

### **Interruption and Barge-In Model**

Managing interruption is difficult because the system must capture the user's speech while playing its own audio through the device's speakers.5 This creates acoustic echo: the agent's output travels through the air and enters the microphone, which the system can mistake for a user interruption.  
To solve this, architectures implement a **Hybrid Client-Server Interruption Pipeline** 34:

```
+---------------------------------------------------------------------------------------------------------+  
|                                    HYBRID INTERRUPTION COORDINATION                                     |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|                                                                      |  
|  Microphone Capture ---> WebRTC AEC3 (Eco Suppression) ---> Client VAD (Wasm)                           |  
|                                                                    |                                    |  
|                                                                    | (Sustained Speech > 100ms)  |  
|                                                                    v                                    |  
|                                                                                |  
|                                                       1. Mute local playback speaker instantly  |  
|                                                       2. Send high-priority truncate message    |  
|                                                                    |                                    |  
|                                                                    v                                    |  
|                                                                                  |  
|  Receive Truncate Message ─────────────────────────────────────────┘                                    |  
|         │                                                                                               |  
|         ├──────> Halt Streaming TTS (Close WebSocket)                                            |  
|         ├──────> Cancel In-Flight LLM Generation                                                 |  
|         └──────> Reconstruct Interrupted Context (Preserve playhead timestamp)            |  
|                                                                                                         |  
+---------------------------------------------------------------------------------------------------------+
```

1. **Acoustic Echo Cancellation (AEC)**: The client or server runs a partitioned block frequency-domain adaptive filter (e.g., WebRTC AEC3).35 AEC3 cross-correlates the playback speaker reference signal with the microphone capture signal, modeling the room's reverberation tail (100 ms to 300 ms) to isolate and cancel the agent's own voice from the incoming stream.35  
2. **Client-Side Edge VAD**: To eliminate network round-trip delays (50 ms to 150 ms), the client application runs a lightweight neural VAD (compiled to WebAssembly) inside a browser AudioWorklet or mobile audio background thread.34  
3. **Truncation Trigger**: When the client-side VAD detects speech with high confidence for a sustained window (e.g., 100 ms to 200 ms of continuous speech, filtering out rapid clicks or coughs), it triggers barge-in.5  
4. **Immediate Local Mute**: The client app immediately mutes local audio playback, dropping its jitter buffers.34 To the user, the agent stops talking the millisecond they interrupt.34  
5. **Upstream Signaling**: The client sends a high-priority "truncate" or "barge-in" control message over the WebRTC data channel or WebSocket control channel to the backend.34  
6. **Server Cancellation**: Upon receipt, the dialogue coordinator immediately terminates the in-flight LLM token generation and closes the active streaming TTS WebSocket, halting upstream billing and processing.5  
7. **Playhead Alignment**: To understand what the user was reacting to, the system maps the exact millisecond the playback was interrupted back to the synthesized text markers.1 If the agent was reading a list of options and the user interrupted at second 4.2, the system correlates the playhead timestamp to identify the exact option the user selected, ensuring correct dialogue state tracking.1

### **Interruption Classification and Dialogue Recovery**

Not all incoming speech during agent playback represents an intent to seize the conversational floor. The system must classify the interruption to determine the appropriate recovery state.

| Interruption Category | Audio/Text Pattern | Classification Marker | State Machine Action | Playback Recovery Action |
| :---- | :---- | :---- | :---- | :---- |
| **Correction** | "No, Tuesday." 1 | High confidence word mismatch | Rollback, load repair prompt | Purge audio buffer; synthesize alternate path.5 |
| **Stop Request** | "Stop.", "Hold on." 1 | Critical intent classification | Transition to **PAUSED** state | Cessation of all audio output; wait for new trigger.5 |
| **New Task** | "Actually, book Wednesday." | Semantic intent shift 1 | Purge current session cache | Terminate running model threads, start new execution.5 |
| **Clarification** | "Wait, what?" 1 | Question pattern detection | Transition to **REPAIR** state | Generate explanatory sidebar text immediately.25 |
| **Backchannel** | "mm-hmm", "okay" 1 | Short acoustic/duration signature | Ignore interruption; keep floor | Maintain current playback; do not halt audio buffer.1 |
| **Ambient Noise** | Cough], | Low speech probability from VAD | Reject classification 1 | Maintain current playback.1 |
| **Assistant Echo** | Agent's own words leaking | AEC linear filter correlation | Reject classification 35 | Unmute client playback if muted speculatively.34 |
| **Bystander Speech** | Multi-speaker chatter 3 | Diarization ID mismatch 3 | Ignore segment 3 | Maintain current playback.3 |

### **Cancellation Semantics**

When an interruption occurs, the dialogue coordinator must evaluate the state of pending tool calls:

* If the agent was interrupted while simply speaking, the system discards the remainder of the generated text and updates its context to reflect the user's interruption.5  
* If a tool call has been dispatched to the execution environment, the system must consult the tool contract.4 If the tool is **Idempotent / Read-Only** (e.g., a search query), the coordinator cancels the execution thread immediately.42 If the tool is **Non-Idempotent / Side-Effecting** (e.g., a financial ledger write or database insertion) and execution has already started, **the operation cannot be cancelled**.9 The system must allow the database transaction to resolve, capture the resulting state, and present a verbal status update to the user, ensuring the agent's spoken statements align with the database state.9

## **Conversational Repair and Misunderstanding Recovery**

Because acoustic environments are imperfect and spoken language is often ambiguous, speech systems will frequently mishear entities, names, numbers, or intents.26 A production-ready voice agent must be capable of identifying communication breakdowns and resolving them through conversational repair without forcing the user to repeat the entire interaction.7

### **Conversational Repair Framework**

Conversational analysis classifies repairs into two categories based on who initiates the process and who resolves the issue 8:

* **Self-Initiated Repair**: The speaker notices their own error and corrects it mid-sentence (e.g., "Please book the flight to London... sorry, I meant Paris").8  
* **Other-Initiated Repair**: The listener flags a comprehension breakdown and requests clarification.8

In human dialogue, these are managed using specific, structured patterns 25:

```
                  +-----------------------------------------+  
                  |      Comprehension / Confidence Gate    |  
                  +-----------------------------------------+  
                                       |  
                   ┌───────────────────┼───────────────────┐  
                   | (High Confidence) | (Low Confidence/  | (Ambiguous  
                   |                   |  Entity Error)    |  Resolution)  
                   v                   v                   v  
          +------------------+ +------------------+ +------------------+  
          | Proceed to Turn  | | Explicit Request | | Restricted Offer |  
          | Completion       | | "Can you spell?" | | "Do you mean A?"   
          +------------------+ +------------------+ +------------------+  
                                       |                   |  
                                       v                   v  
                               +---------------------------------------+  
                               |     Process User Correction / Response|  
                               +---------------------------------------+  
                                                   |  
                                                   v  
                               +---------------------------------------+  
                               |      Update Active Dialogue State     |  
                               +---------------------------------------+
```

To implement these repair dynamics, production architectures run three primary repair patterns based on confidence scoring and entity classification 25:

| Repair Modality | Triggering Condition | Algorithmic Mechanism | Verbal Prompts / Patterns | Escalation Threshold |
| :---- | :---- | :---- | :---- | :---- |
| **Confidence Clarification** | Overall STT confidence drops below threshold (0.50 - 0.70) 41 | Open request prompting for repetition | "I'm sorry, I didn't quite catch that. Could you please repeat what you said?" 25 | Strike 1: Prompt again. Strike 2: Slow down output tempo.46 |
| **Explicit Readback** | Critical alphanumeric capture completed 6 | Repeat exact parameter with confirm request | "I heard your account number is four five six seven. Is that correct?" 9 | Target parameter rejection resets input field buffer.9 |
| **Spelling Mode** | Phonetic spelling mismatch on proper nouns 8 | Character-by-character grapheme collector | "Could you please spell your last name letter-by-letter?" 8 | Trigger automatic lookup matching phonetic dictionaries.11 |
| **Digit-by-Digit Mode** | Alphanumeric string segmentation fail 10 | Disfluent digit capture filter | "Please repeat the routing code, saying each number slowly and separately." | Limit field capture size, fallback to DTMF/Keypad.46 |
| **Entity Disambiguation** | Multiple phonetic matches detected 25 | Restricted binary selection offering | "Did you mean June twelfth or July twelfth?" 25 | Strike 3: Divert to human specialist queue. |
| **Paraphrase Confirmation** | Multi-step logical intent inferred | Semantic summarization comparison | "Just to make sure we're on the same page, you want me to transfer five hundred dollars to John?" 9 | Rejection restarts planning loop; does not execute tool.9 |

## **Voice-to-Action Gating**

When voice interfaces are connected to agentic tool execution environments (e.g., databases, CRM systems, or transaction APIs), voice uncertainty can translate into unsafe action execution.9 An unstable, intermediate partial transcript, or an unconfirmed name parameter, must never be allowed to trigger a permanent mutation.4

### **Voice-to-Action Gating Model**

Architectures enforce a **Voice-to-Action Gate** that classifies every requested tool invocation by risk and binds its execution to strict transcript and confidence metrics.

```
                  +-----------------------------------------+  
                  |         Proposed Spoken Command         |  
                  +-----------------------------------------+  
                                       |  
                                       v  
                  +-----------------------------------------+  
                  |   Classify Tool Action Risk Profile     |  
                  +-----------------------------------------+  
                                       |  
            ┌──────────────────────────┼──────────────────────────┐  
            | (Read-Only)              | (Low-Risk Mutation)      | (High-Risk/Irreversible)   
            v                          v                          v  
+-----------------------+  +-----------------------+  +-----------------------+  
|  Transcript Final &   |  |   Transcript Final,   |  |   Transcript Final,   |  
|   Confidence > 0.80   |  |   Confidence > 0.90   |  |   Confidence > 0.98   | 41, 9]  
+-----------------------+  +-----------------------+  +-----------------------+  
            |                          |                          |  
            | (Criteria Met)           | (Criteria Met)           | (Requires explicit spoken readback)  
            v                          v                          v  
+-----------------------+  +-----------------------+  +-----------------------+  
|  Execute Instantly    |  | Lightweight Verbal    |  | "Confirm payload:     |  
|  (No spoken confirm)  |  | "Done."        |  | Transfer $500?" |  
+-----------------------+  +-----------------------+  +-----------------------+
```

To implement this gating architecture, the system maps execution pathways to five primary action risk classes:

| Action Risk Class | Target Operations | Transcript Stability Requirement | Minimum STT Confidence | Spoken Confirmation Policy | Visual/Multimodal Verification |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Read-Only / Informational** | Querying weather, checking flight time, reading account balances. | is_final: true | >= 0.80 | **None**: Execute instantly and speak results. | Optional; show data cards on display if available. |
| **Low-Risk Local Mutation** | Creating a checklist item, adding calendar events. | is_final: true | >= 0.90 | **Lightweight**: State completion after execution ("Added."). | Render entry on screen for confirmation. |
| **Low-Impact External Action** | Sending a standard Slack or team message. | is_final: true | >= 0.95 | **Explicit Intent Confirmation**: "Do you want me to send that message?" | Display draft message with recipient name. |
| **High-Risk / Irreversible** | Financial wire transfers, system deployments, deleting files. | is_final: true | >= 0.98 | **Strict Payload Readback**: Read back the exact target, amount, and recipient before executing. | Mandate explicit PIN entry or bio-verification. |
| **Critical Mutation** | Executing large refunds, modifying security access levels. | is_final: true | 1.00 (Absolute certainty) | **Dual-Control Gating**: Voice interface registers intent, but requires manual administrator authorization. | Block voice execution; send confirmation link to registered device. |

### **Think-Before-Speak Reasoning Paradigms**

To avoid executing tools on conversational filler sentences, production architectures employ a **Think-Before-Speak Mechanism** (e.g., VoxMind architecture).38 Before generating any synthesized audio, the agentic model is trained to emit a structured internal reasoning trajectory.38 This internal thought token sequence is evaluated strictly inside a hidden context, allowing the model to interpret tool schemas, plan multi-turn dependent actions, and format parameters before committing to a spoken phrase.  
This process is augmented using self-reflection control tokens (such as the Speech-Hands framework). During the inference pass, the agent generates control tokens from a learned action set, directing its cognitive flow:

* **<internal>**: The system is highly confident in its phonetic understanding and executes direct inference.9  
* **<external>**: The system detects high local acoustic ambiguity and triggers external verification tools.9  
* **<rewrite>**: The system flags a logical mismatch between its transcription and downstream API schemas, triggering an internal dialogue state correction and re-evaluating the user's intent before acting.9

## **Voice Identity, Consent, and Security**

Voice interaction surfaces are vulnerable to identity spoofing, biometric replication, and unauthorized execution attacks.27 A robust real-time voice system must distinguish between speech recognition ("what is being said") and speaker recognition ("who is saying it"), protecting both channels with active defense-in-depth measures.3

### **Biometric Security and Cloning Prevention**

As voice synthesis models improve, neural voice clones can easily bypass standard, legacy voice biometrics. To defend against biometric identity theft and replay attacks, security architectures implement three layers of physical and cryptographic controls :

#### **1. Real-time Liveness Detection & Anti-Spoofing**

The system processes the incoming raw audio stream using deep neural networks trained to detect synthetic audio signatures (such as Resemble's DETECT-2B or Mamba-SSM architecture).27 These models analyze acoustic artifacts, phase-reconstruction anomalies, and high-frequency spectral patterns that are imperceptible to humans but present in AI-generated waveforms.27 If synthetic artifacts are detected, the biometric match is rejected, and the system prompts the user for multi-factor authentication (MFA).46

#### **2. Cryptographic Content Watermarking**

All synthetic audio synthesized by the system must embed a tamper-resistant, imperceptible digital watermark (e.g., Resemble's PerTH watermarking scheme).27 This watermark modifies the spectral domain of the synthesized waveform so that it carries a statistically detectable pattern that survives compressed audio transport (such as cell-phone codecs or WebRTC channels) over the network without degrading audio quality.19 Any audio capture presented to the system as input is checked for this watermark; if the watermark is present, the system immediately recognizes the audio as machine-generated, blocking replay or spoofing attempts.27

#### **3. Cryptographic Provenance Standards (C2PA)**

To establish an unbroken chain of custody, real-time voice assets are bound to cryptographic metadata manifests in accordance with the **Coalition for Content Provenance and Authenticity (C2PA)** and Adobe Content Authenticity Initiative (CAI) specifications.19

```
+---------------------------------------------------------------------------------------------------------+  
|                                        C2PA CRYPTOGRAPHIC MANIFEST                                      |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|                                                                       |  
|  Brand Private Key (KMS/HSM) ---> Sign Manifest Hash ───> Tamper-evident C2PA Manifest    |  
|                                                                                                         |  
|                                                                                  |  
|  {                                                                                                      |  
|    "assertions": ... ],                                                                               |  
|    "signature": "COSE_Sign1_Signature_Bytes_Cryptographically_Linked_to_Audio_Hash"                     |  
|  }                                                                                                      |  
|                                                                                                         |  
+---------------------------------------------------------------------------------------------------------+
```

When the voice agent synthesizes a stream, it hashes the output audio frames, binds them to a tamper-evident C2PA manifest asserting AI involvement and corporate origin, and cryptographically signs the manifest using a private key secured in a Hardware Security Module (HSM) or cloud Key Management Service (KMS).19 Downstream consumer applications can verify this signature against trusted root authorities, instantly validating origin and proving the "receipts" of the AI's statements.19

### **Voice Identity and Consent Model**

Securing real-time interaction systems requires separating phonetic parsing from speaker biometric authorization, establishing explicit constraints around voice usage.3

| Governance Parameter | Operational Mechanism | Cryptographic Control | Target Security Boundary |
| :---- | :---- | :---- | :---- |
| **Speech vs Speaker Recognition** | Separate phonetic STT arrays from Speaker Biometric verification algorithms.3 | Isolated execution runtime sandboxes | Prevent transcription leaks from triggering biometric key creation.28 |
| **Synthetic Voice Disclosure** | Inject high-frequency markers or explicit verbal preamble disclosures.48 | C2PA assertion metadata binding 19 | Compliance under state consumer disclosure rules.48 |
| **Consent & Licensing** | Record user signing certificates to verify cloning authorization.27 | HSM/KMS Private Key generation matching | Prevent unauthorized replica generation of employees.27 |
| **Anti-Spoofing Filters** | Real-time phase, spectral, and dynamic waveform checking.27 | DETECT-2B Mamba-SSM classifier filters 27 | Block synthesized voices bypassing customer service.27 |
| **Watermark Durability** | Imperceptible frequency-domain watermark embedding.19 | PerTH algorithm spectral modulation 27 | Retain verification after G.711 telephony compression.2 |
| **Biometric Privacy** | Local hashing of vocal characteristic vectors; delete raw sound files.36 | Anonymized vector storage schemas 36 | Conform to regional biometric privacy regulations (GDPR/CCPA). |
| **Corporate Governance** | Strict restriction of voice models to authorized brand pipelines.27 | Role-Based Access Control (RBAC) keys | Prevent corporate brand voice hijack or leak.27 |

## **Accessibility and Inclusive Voice Interaction**

Voice-first design must never mean voice-only.46 While real-time speech interfaces provide unprecedented accessibility for individuals with visual or motor impairments, they introduce barriers for individuals with stutters, aphasia, dysarthria, or hearing impairments.46

### **Inclusive Design Adaptations**

To ensure universal accessibility, real-time voice architectures must integrate inclusive-speech layers:

```
                  +-----------------------------------------+  
                  |         Incoming User Audio             |  
                  +-----------------------------------------+  
                                       |  
                                       v  
                  +-----------------------------------------+  
                  |        Speech Classifier Mode           |  
                  +-----------------------------------------+  
                                       |  
                     ├─────────────────┼─────────────────┐  
                     | (Standard       | (Stutter /      | (Non-Verbal /  
                     |  Speech)        |  Dysarthria)    |  AAC Input)  
                     v                 v                 v  
          +------------------+ +------------------+ +------------------+  
          | Standard VAD /   | | Extend Silence   | | Redirect to      |  
          | Fast Turn-Taking | | Hangover (1500ms)| | Text Chat /      |  
          | (400ms Silence)  | | Bypass Repeats   | | Visual UI Form   |   
          +------------------+ +------------------+ +------------------+
```

* **Stutter-Aware Endpointing**: Users who stutter experience involuntary sound repetitions, prolongations, or silent blocks. Standard endpointing cutoffs will segment their sentences prematurely. The dialogue coordinator must detect repetitive syllable patterns or acoustic blocks and automatically extend the max silence threshold to 2500 ms - 4000 ms, preventing premature interruption and allowing the speaker to finish their thought at their own pace.  
* **Atypical Speech Models**: Classic STT decoders trained on standard pronunciation frequently misidentify slurred, strained, or atypically paced speech caused by dysarthria, cerebral palsy, or Parkinson's disease. Production pipelines route atypical audio through specialized speech recognition models (e.g., Voiceitt's atypical speech API) that adapt dynamically to individual vocal patterns, converting disjointed acoustics into clear, reconstructed text.  
* **Multimodal Fallbacks**: For hearing-impaired users or non-verbal individuals, the interface must present synchronized visual captions, interactive text-entry alternatives, and support for Augmentative and Alternative Communication (AAC) device speech output. The system must allow users to transition between voice, touch, and text typing at any point during the session without losing conversation state or history.

### **Accessibility and Inclusive Voice Interaction Model**

Implementing accessible interfaces requires programmatic constraints to support diverse speech and language capabilities.46

| Target Disability Category | Specific Acoustic Challenge | Technical Mitigation Strategy | System Fallback Mechanism |
| :---- | :---- | :---- | :---- |
| **Motor & Physical** | Inability to press physical keys or hold devices. | Native voice wake-word + local VAD processing.39 | Dynamic push-to-talk toggling.54 |
| **Low Vision / Blind** | Inability to inspect screens or visual cards.46 | Strict verbal descriptive summaries of visual state.46 | Screen-reader compliant LaTeX / HTML output. |
| **Dyslexia / Cognitive** | Working memory load limits on long spoken sentences.46 | Clamp speaking rate below 140 words/min.46 | Push persistent summary transcripts to the client visual display.46 |
| **Hearing Impairments** | Inability to hear synthesized audio output.46 | Stream real-time visual closed-caption arrays.46 | Full download of session text transcript.46 |
| **Stutter / Stammer** | Involuntary blocks, prolongations, repetitions.51 | Extended semantic VAD thresholds, ignore syllable loops. | Toggle conversational pace setting to "Slow".39 |
| **Dysarthria / Slurred** | Muscular weakness altering consonant articulation.46 | Specialized acoustic training adapters (Voiceitt databases).55 | Refine input via local LLM grammar reconstruction.47 |
| **Aphasia** | Long pauses, word-retrieval struggle, fragments.46 | Extend silence hangover to > 3.0 s dynamically.46 | Use predictive word bank suggestions on visual displays.46 |
| **Temporary / Environmental** | Noisy cafés, laryngitis, dental recovery.46 | Noise-aware filtering + WebRTC AGC adjustments.35 | Switch dynamically to non-verbal text-chat messaging.46 |

## **Voice Privacy and Environment Model**

Because speech is an active acoustic waveform projected into physical space, voice systems interact directly with the user’s social and environmental surroundings.28 The system must respect privacy boundaries and adapt to the context of the user’s immediate physical space.30

| Operational Environment | Privacy Hazard | Technical Defense Protocol | Fallback / Safe State |
| :---- | :---- | :---- | :---- |
| **Public Place / Transport** | Shoulder-surfing acoustics, bystander overhearing.19 | Private Content Filter: Restrict financial/personal parameter readbacks.46 | Dynamic redirect to private smartphone display cards.46 |
| **Noisy Home / Office** | Accidental capture of background conversations or television.2 | Neural speaker-separation + diarization classification.3 | Ignore audio frames mapping to non-primary diarization IDs.3 |
| **Shared Workplace** | Capturing private background speech from coworkers.36 | Local wake-word validation + near-field microphone limits.39 | Shut down media plane if signal-to-noise ratio drops below 10 dB. |
| **Static Secure Room** | Eavesdropping via always-listening cloud connections.36 | Hardware Push-to-Talk (PTT) / physical mute triggers.36 | Terminate media stream socket; run local-only VAD.34 |
| **Compliance Monitored** | Accidental capture of PCI/PHI metadata.36 | Real-time regex and entity redaction on STT pipeline. | Quarantine target transcript blocks; raise security flag. |

## **Realtime Voice Failure Mode Map**

Real-time voice systems are vulnerable to failure modes that do not exist in text-based environments.1 This failure mode map outlines the typical symptoms, underlying causes, and mitigation strategies for production-grade systems.4

| Failure Mode | User-Visible Symptom | Technical Root Cause | Detection Signal / Trigger Metric | Mitigation & Architectural Prevention |
| :---- | :---- | :---- | :---- | :---- |
| **False Endpoint** | Agent cuts off the user mid-sentence or mid-thought, speaking over them.1 | Silence thresholds are too short or VAD registers silence during thoughtful pauses.1 | Post-interruption user correction rate / Overlap duration.1 | Deploy Hybrid Semantic-Acoustic VAD; extend pause thresholds on trailing conjunctions.20 |
| **Late Endpoint** | The agent takes 1.5 s - 2 s to respond, creating dead air.1 | Silence thresholds are set too high or wait for forced timeout.38 | Turn Latency / User repeat prompt rate.4 | Use punctuation-based turn detection or eager semantic classification.39 |
| **Clipped Speech** | The STT engine misses the last word of the user's sentence.1 | VAD closes the capture buffer immediately upon silence, chopping trailing phonemes.1 | Under-transcribed word error rate / Speech-stopped timestamp overlap. | Apply a minimum trailing padding (150 ms - 200 ms) after VAD silence detection.5 |
| **Overtalk / Overlap** | Both user and agent speak simultaneously, creating chaotic crosstalk.22 | State machine fails to yield floor or slow network delay.2 | Double-talk duration > 300 ms.2 | Calibrate WebRTC AEC3; enforce strict floor yield rules.5 |
| **Dead Air** | Conversation halts, neither party speaks, session feels broken.22 | Silence timeout fails to trigger or LLM reasoning hangs.4 | Continuous silence duration > 3.5 s.4 | Play comforting background hum or dispatch verbal status updates. |
| **False Barge-In** | Agent stops speaking prematurely on brief background noises (cough, click, door slam).5 | High-sensitivity VAD threshold with no duration guardrails.5 | Interruption rate / False interruption count.5 | Implement a minimum sustained-speech duration guard (150 ms - 250 ms) before triggering truncation.5 |
| **Missed Barge-In** | User says "stop" or "no", but the agent continues speaking over them.5 | Poor acoustic echo cancellation or high local client jitter buffer.5 | High overlap duration / User speech amplitude spike during output. | Calibrate WebRTC AEC3; implement immediate local speaker muting on client-side VAD speech onset.34 |
| **Echo Interruption** | The agent interrupts itself as soon as it begins speaking. | The agent's output leaks into the microphone and triggers the VAD.5 | Autocorrelation match between output and input streams.35 | Route speaker reference audio back to WebRTC AEC3; calibrate delay estimation parameters.35 |
| **Partial Action Mutation** | An API database change is executed on a sentence before it is finished.6 | Dialogue coordinator acts on an unstable partial transcript.6 | Database rollback count / API execution timestamp preceding speech_final.6 | Restrict mutation tool access to finalized, stable transcripts with explicit confirm gates. |
| **Transcript Revision** | The system processes a parameter (e.g., "$15" vs "$50") but the STT later revises the text after execution. | Over-optimistic parsing of interim results.6 | Post-execution revision count / Transcript mismatch rate.6 | Buffer state execution until is_final: true is returned by the decoder.6 |
| **Entity Miscapture** | The agent mishears proper nouns, names, or addresses, causing task failure.1 | Out-of-vocabulary phoneme mapping on specialized terms.11 | Downstream validation failures / Specific Entity WER.1 | Initialize STT connections with keyterm prompts and domain-specific pronunciation dictionaries.11 |
| **Wrong Speaker** | Commands from a background bystander are processed as user intent.3 | Lack of speaker diarization or biometric verification.3 | Speaker embedding distance > threshold on active turn.3 | Bind the active session context strictly to the authenticated user's voice footprint.3 |
| **Speaker Spoofing** | Unauthorized user gains access using a recorded or synthesized voice.27 | Vulnerable biometric voice verification system lacking anti-spoofing controls.27 | Verification security alert rate.27 | Run active liveness detection; verify synthetic watermarks on inputs; enforce multi-factor authentication.27 |
| **Accent Failure** | Accuracy drops dramatically for non-standard English accents.53 | Training data under-represents regional pronunciations.53 | Segment-level WER > 15% on accented accounts.53 | Implement multi-scale acoustic feature recalibration networks.43 |
| **Stutter Cutoff** | User with a stutter is cut off after repeating initial syllables.47 | Rapid syllable breaks are registered by VAD as silent turn-yielding pauses. | High session abandonment rate for accessibility segments.47 | Dynamically switch to the Inclusive/Accessibility profile; extend max turn silence.38 |
| **Noisy Room Fail** | System loops constantly, unable to process clean speech.36 | Environmental noise overrides energy thresholds.38 | Frame classification entropy > 0.8.38 | Calibrate adaptive noise filters; switch client settings to WebRTC level 3.35 |
| **Latency Cliff** | Conversations drop below human conversational pace (> 1.2 s).4 | Lack of stream-pipelining; slow model inference; geographic server-client distance.4 | End-to-End Latency distribution (P95 > 1000 ms).4 | Enforce concurrent sentence-buffered TTS streaming; peer transceivers close to major network hubs.30 |
| **TTS delay** | Long gap between LLM first token and audio playback onset. | Sentence buffer is set too large or TTS synthesis is too slow.4 | TTS Synthesis time-to-first-audio (TTFA) > 300 ms.4 | Utilize low-latency synthetic voices; stream first chunk before completing sentence.4 |
| **Streaming Contradiction** | Agent begins speaking a claim, then halts and corrects itself mid-sentence. | The streaming LLM changed its reasoning direction after partial tokens were already sent to synthesis. | Self-correction count in active speaker streams.8 | Implement a minimum lookahead token buffer before dispatching to TTS; enforce semantic sentence buffering.11 |
| **Tool Result Outrun** | Agent says "Your money has been sent" but the payment API fails 2 s later. | Claim generated before tool execution state was verified. | Claim-to-Reality validation failures. | Restrict vocalized completions until the execution engine returns a verified success payload.9 |
| **Under-Repair** | System repeatedly misinterprets input without launching clarification loops.8 | Lacks confidence threshold filters.1 | Repetitive intent correction loops.8 | Execute Targeted Restricted Requests on any entity confidence drops.25 |
| **Over-Confirmation** | Agent asks "Is that correct?" for basic, low-risk statements, frustrating the user.9 | Confirm gates applied globally without risk classification.9 | Task Completion Time (TCT) rises unnecessarily.2 | Align confirm prompts to action risk profiles.9 |
| **Creepy Voice** | User experiences discomfort or anxiety regarding the agent's emotional tone.48 | Excessive, inappropriate synthetic emotional matching or fake human claims.48 | Perceived creepy score in post-call evaluations. | Mandate identity transparency disclosures; clamp emotional synthesis parameters to appropriate ranges.48 |
| **Voice Impersonation** | Brand voice replica is compiled and posted to public domains.27 | Lack of server-side watermarking or signing.19 | Unauthorized brand asset detection alerts.48 | Embed permanent, spectral PerTH watermarks and C2PA manifests into all generated audio.19 |
| **Privacy Leak** | Agent reads highly private data aloud in a public, crowded space.1 | Failure to evaluate physical or environmental context. | Post-call privacy incident logs.1 | Restrict sensitive readbacks; redirect private parameters to visual display cards.46 |
| **Inaccessible Fallback** | Non-verbal user is locked out because the system requires voice confirmation. | System has no multimodal interfaces.46 | Accessibility failure report count.46 | Support parallel text typing, DTMF input, and non-voice alternatives.46 |

## **Evaluation and Observability**

A real-time voice system cannot be evaluated using offline Word Error Rate (WER) alone.1 While transcription accuracy is a key metric, the conversational success of an agent is determined by turn transitions, interruption responsiveness, and safety boundaries.5

### **Voice Evaluation and Observability Guidance**

Establishing continuous observability over live voice sessions requires tracking metrics across both the perception plane and the temporal interaction plane.1

| Metric Category | Technical Metric Identifier | Diagnostic Target | Production Target Threshold |
| :---- | :---- | :---- | :---- |
| **Speech Accuracy** | Word Error Rate (WER) 1 | General phonetic transcription accuracy | < 5.0% under standard conditions.11 |
| **Speech Accuracy** | Entity-specific WER 1 | Alphanumeric and proper noun accuracy | < 1.5% on prioritized codes.10 |
| **Transcription** | Transcript Stability Score 6 | Partial token volatility index | Stability confidence > 0.85.6 |
| **Transcription** | Partial-to-Final Revision Rate | Frequency of finalized text corrections | < 4.0% post-emission revision rate.6 |
| **Endpointing** | Turn-End Detection Latency 4 | Duration from speech stop to committed turn | 250 ms - 450 ms average turn gap.1 |
| **Endpointing** | Cutoff Error Rate 1 | Percentage of turns prematurely truncated | < 1.5% on conversational sessions.1 |
| **Endpointing** | False Endpoint Rate 1 | VAD silence false-positive triggers | < 2.0 instances per hour.1 |
| **Endpointing** | VAD False-Negative Rate 1 | Missed user speech frames | < 1.0% under noisy cafes.1 |
| **Barge-In** | Barge-In Detection Latency 5 | Duration to register user interruption | < 120 ms post-speech onset.5 |
| **Barge-In** | False Barge-In Rate | Agent stops on noise/coughs/clicks | < 1.0% on environmental noise. |
| **Barge-In** | Missed Barge-In Rate 5 | Failure of agent to yield floor | < 1.5% on explicit interruptions.5 |
| **Barge-In** | Interruption Recovery Time | Time to synthesize new path post-stop | < 150 ms total recovery window.5 |
| **Model Serving** | Model First-Token Latency | LLM Time-to-First-Token (TTFT) | < 150 ms from committed turn. |
| **Voice Synthesis** | TTS First-Audio Latency 4 | Synthesis Time-to-First-Audio (TTFA) | < 120 ms from sentence buffer.4 |
| **Media Playback** | Playback Jitter Buffer Delay 5 | Downlink client buffering lag | Keep dynamic size < 60 ms.5 |
| **E2E Dynamic** | Turn Latency (T_streaming) 16 | End-of-speech to start-of-audio | < 700 ms average conversational.4 |
| **Action Gating** | Tool-Gating Precision | Safe blocking of unauthorized tools | 100% precision on irreversible API mutations.9 |
| **Dialogue Repair** | Confirmation Success Rate 9 | Explicit readback validation match | > 98% confirm loop success.9 |
| **Dialogue Repair** | Repair Success Rate 8 | Resolution of phonetic misunderstandings | > 92% of clarification prompts resolved.8 |
| **Dialogue Repair** | User Correction Rate 8 | Frequency of user repeating commands | < 5.0% of active conversational turns.8 |
| **Dialogue Repair** | Repeat Request Rate | User says "what did you say?" | < 2.0% of active sessions. |
| **Accessibility** | Accessibility Success Rate 46 | Task completion for impaired speech | > 90% completion across atypical profiles.47 |
| **System Security** | Privacy Incident Rate 1 | Public data leaks / bystander capture | 0 verified leaks under public context.1 |
| **Conversational** | Abandonment Rate 26 | Call drop during breakdown sequences | < 3.0% of active calling sessions.26 |
| **Conversational** | User Satisfaction (MOS) 44 | Perceived naturalness / helpfulness | Mean Opinion Score > 4.2 out of 5. |

## **Cross-Canon Handoff Map**

The systems architecture of AI-ENG-Q interfaces directly with upstream and downstream reports across the AI Systems Engineering Canon, ensuring consistent temporal state preservation and tool safety.

| Target Report ID | Target Report Domain | Operational Handoff Metric | Dependency / Engineering Integration Rules |
| :---- | :---- | :---- | :---- |
| **AI-ENG-B** | Context-Tenure & State Governance | Session vector decay rate | Inherit memory-clearing rules for audio frames, transcoded buffers, and biometric maps.1 |
| **AI-ENG-C** | Latency-Margin Management | Jitter queue delay budgets | Coordinate peak serving costs with streaming inference queues. |
| **AI-ENG-L** | Servicing Mechanics | Bidirectional serving backpressure | Terminate WebRTC packets at dynamic transceiver edge proxy containers.30 |
| **AI-ENG-M** | Autonomy Boundaries | Dynamic terminal state timeout | Stop active execution threads immediately when user emits "stop".5 |
| **AI-ENG-N** | Tool Contracts | Schema verification F1-score | Require strict JSON parameter validation against extracted audio entities before API call. |
| **AI-ENG-O** | Action-Verification Discipline | Grounding validation index | Spoken agent claims must never outrun verified database transactional commits. |
| **AI-ENG-P** | Multimodal Understanding | Coordinate mapping accuracy | Ground audio transcripts and conversational state directly back to physical audio frames. |
| **AI-ENG-S** | Production Pathologies | Telemetry trace latency variance | Log and trace cumulative turn latency across each pipeline component. |
| **AI-ENG-T** | Prompt Injection Safeguards | Injection payload block rate | Sanitize speech transcripts for vocal override commands before LLM processing.19 |
| **AI-ENG-U** | Dependency and Tool Risks | Sandbox escape isolation rate | Sandbox third-party acoustic VAD and speech-recognition libraries.36 |
| **AI-ENG-V** | Resource and Voice Abuse | Command execution authorization | Enforce biometric liveness verification for high-risk voice commands.27 |
| **AI-ENG-W** | Fallback & Degraded Modes | Failover latency penalty | Switch to DTMF keypad fallback if STT or network latency exceeds bounds.46 |
| **AI-ENG-X** | User Trust and Control | Citation coordinates match rate | Highlight exact speech segments and transcribed words on user display. |
| **AI-ENG-Y** | Human Review | Escalation transition window | Divert to human operators when speech repair loops fail twice. |
| **AI-ENG-Z** | Telemetry and Metrics | OpenTelemetry parsing success | Log event timestamps for voice onset, STT stops, and speaker transitions.28 |
| **AI-ENG-AA** | Speech & Voice Evaluations | Benchmark precision/recall | Evaluate agent dynamics against TempCore and VoiceAgentBench.47 |
| **AI-ENG-AB** | Audit and Replay Mechanics | Replay alignment success | Store cryptographically signed manifests (C2PA) alongside conversation audio logs.19 |
| **AI-ENG-AC** | Incident Response Protocols | Isolation containment delay | Quarantine sessions that trigger biometric spoofing alerts or acoustic failures.27 |
| **AI-ENG-AJ** | Reference Architectures | Blueprints integration success | Use modular WebRTC transceiver proxies and streaming sentence buffers.4 |

## **Durable Principles of Real-Time Interaction Systems**

### **1. Timing is Semantic Content**

In real-time voice, pauses, turn transitions, intonation, and response latency carry equal semantic weight to the textual tokens themselves.1 A real-time voice system must treat conversational pace as a first-class optimization metric.4

### **2. Stream-Pipelining is Non-Negotiable**

To operate within human turn-taking expectations (200 ms - 500 ms), a voice agent cannot run sequentially.22 Every stage of the pipeline—from client-side audio capture, streaming transcription, token generation, clause-boundary sentence buffering, to TTS synthesis—must execute concurrently and stream outputs continuously to the next stage.4

### **3. Silence is Not the Same as an Endpoint**

Energy-based silence detection fails in real-world environments.1 Highly accurate turn-taking requires a hybrid semantic-acoustic model that combines frame-level acoustic VAD with real-time evaluation of linguistic completeness, prosodic contours, and conversational filler tokens, ensuring users are never prematurely cut off mid-thought.1

### **4. Spoken Claims Must Never Outrun Verified Reality**

When a voice interface is connected to external systems, unconfirmed spoken statements can create false user expectations. All non-idempotent or high-impact actions must be validated, and their execution state verified, before the agent vocalizes completion, enforcing a strict confirm-gate on unstable transcripts.

### **5. Voice-First Must Never Mean Voice-Only**

Speech is biologically and socially variable.46 Real-time voice systems must incorporate inclusive-design adaptations to support speech differences (stutters, dysarthria, aphasia) and mandate alternative visual captions, manual touch fallbacks, and text alternative paths to ensure accessibility under all environmental and physical conditions.46

#### **Works cited**

1. Tackling Turn Detection in Voice AI: Overcoming Noise and Interruption Challenges - Notch, accessed June 10, 2026, [https://www.notch.cx/post/turn-detection-in-voice-ai](https://www.notch.cx/post/turn-detection-in-voice-ai)  
2. Human Latency Conversational Turns for Spoken Avatar Systems - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2404.16053v1](https://arxiv.org/html/2404.16053v1)  
3. Streaming Speaker Diarization in Real Time: Guide - AssemblyAI, accessed June 10, 2026, [https://www.assemblyai.com/blog/streaming-speaker-diarization](https://www.assemblyai.com/blog/streaming-speaker-diarization)  
4. Building Enterprise Realtime Voice Agents from Scratch: A Technical Tutorial - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2603.05413v1](https://arxiv.org/html/2603.05413v1)  
5. Voice AI Barge-In and Turn-Taking: A 2026 Implementation Guide - Future AGI, accessed June 10, 2026, [https://futureagi.com/blog/voice-ai-barge-in-turn-taking-2026/](https://futureagi.com/blog/voice-ai-barge-in-turn-taking-2026/)  
6. Configure Endpointing and Interim Results - Deepgram's Docs, accessed June 10, 2026, [https://developers.deepgram.com/docs/understand-endpointing-interim-results](https://developers.deepgram.com/docs/understand-endpointing-interim-results)  
7. Repair: The Interface Between Interaction and Cognition - PMC, accessed June 10, 2026, [https://pmc.ncbi.nlm.nih.gov/articles/PMC6849777/](https://pmc.ncbi.nlm.nih.gov/articles/PMC6849777/)  
8. System and User Strategies to Repair Conversational Breakdowns of Spoken Dialogue Systems: A Scoping Review | Request PDF - ResearchGate, accessed June 10, 2026, [https://www.researchgate.net/publication/382076125_System_and_User_Strategies_to_Repair_Conversational_Breakdowns_of_Spoken_Dialogue_Systems_A_Scoping_Review](https://www.researchgate.net/publication/382076125_System_and_User_Strategies_to_Repair_Conversational_Breakdowns_of_Spoken_Dialogue_Systems_A_Scoping_Review)  
9. A Self-Reflection Voice Agentic Approach to Speech Recognition and Audio Reasoning with Omni Perception - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2601.09413v2](https://arxiv.org/html/2601.09413v2)  
10. Edge-Optimized Speech Workflows: Combining Deepgram Nova-3 STT with Fish Speech V1.5 TTS - GetStream.io, accessed June 10, 2026, [https://getstream.io/blog/edge-speech-deepgram-fish/](https://getstream.io/blog/edge-speech-deepgram-fish/)  
11. Live Audio - Deepgram's Docs, accessed June 10, 2026, [https://developers.deepgram.com/reference/speech-to-text/listen-streaming](https://developers.deepgram.com/reference/speech-to-text/listen-streaming)  
12. Best Speech-to-Text APIs for Developers Building Real-Time Voice AI in 2026, accessed June 10, 2026, [https://inworld.ai/resources/best-speech-to-text-apis](https://inworld.ai/resources/best-speech-to-text-apis)  
13. 10 Best speech-to text api You Should Know - Vatis Tech, accessed June 10, 2026, [https://vatis.tech/blog/best-speech-to-text-api-a00ab](https://vatis.tech/blog/best-speech-to-text-api-a00ab)  
14. Voice Agent API - AssemblyAI, accessed June 10, 2026, [https://www.assemblyai.com/products/voice-agent-api](https://www.assemblyai.com/products/voice-agent-api)  
15. How to Build a Voice Agent with Real-Time Translation Using OpenAI GPT Realtime 2, accessed June 10, 2026, [https://www.mindstudio.ai/blog/build-voice-agent-real-time-translation-gpt-realtime-2](https://www.mindstudio.ai/blog/build-voice-agent-real-time-translation-gpt-realtime-2)  
16. Real-time transcription in Python with Universal-Streaming - AssemblyAI, accessed June 10, 2026, [https://www.assemblyai.com/blog/real-time-transcription-in-python](https://www.assemblyai.com/blog/real-time-transcription-in-python)  
17. Voice Activity Detection: How VAD Powers AI Agents in 2026 - Parloa, accessed June 10, 2026, [https://www.parloa.com/blog/voice-activity-detection-vad/](https://www.parloa.com/blog/voice-activity-detection-vad/)  
18. VAD voice activity detection for clearer agent calls - Teammates.ai, accessed June 10, 2026, [https://teammates.ai/blog/vad-voice-activity-detection-for-clearer-agent-calls](https://teammates.ai/blog/vad-voice-activity-detection-for-clearer-agent-calls)  
19. Why VAD End-of-Speech Detection Is the Hardest Problem in Production Voice Agents, accessed June 10, 2026, [https://altersquare.medium.com/why-vad-end-of-speech-detection-is-the-hardest-problem-in-production-voice-agents-fee308e38cfc](https://altersquare.medium.com/why-vad-end-of-speech-detection-is-the-hardest-problem-in-production-voice-agents-fee308e38cfc)  
20. What Is Semantic VAD? (And Why It Matters for Voice Agents) - Inworld AI, accessed June 10, 2026, [https://inworld.ai/resources/what-is-semantic-vad](https://inworld.ai/resources/what-is-semantic-vad)  
21. Universal-3 Pro Streaming | AssemblyAI | Documentation, accessed June 10, 2026, [https://www.assemblyai.com/docs/streaming/universal-3-pro](https://www.assemblyai.com/docs/streaming/universal-3-pro)  
22. Turn-Taking Modelling in Conversational Systems: A Review of Recent Advances - MDPI, accessed June 10, 2026, [https://www.mdpi.com/2227-7080/13/12/591](https://www.mdpi.com/2227-7080/13/12/591)  
23. Barge-In – End-to-End Interruption Metrics Across ASR & TTS - Cekura, accessed June 10, 2026, [https://www.cekura.ai/discover/voice-ai-barge-in-testing-asr-latency-tts-overrun](https://www.cekura.ai/discover/voice-ai-barge-in-testing-asr-latency-tts-overrun)  
24. Turn Detection for Voice Agents: VAD, Endpointing, and Model-Based Detection | LiveKit, accessed June 10, 2026, [https://livekit.com/blog/turn-detection-voice-agents-vad-endpointing-model-based-detection](https://livekit.com/blog/turn-detection-voice-agents-vad-endpointing-model-based-detection)  
25. Conversational Repair and the Acquisition of Language - Stanford University, accessed June 10, 2026, [https://web.stanford.edu/class/cs379c/class_messages_listing/curriculum/Annotated_Readings/ClarkDISCOURSE-PROCESSES-20_Unannotated.pdf](https://web.stanford.edu/class/cs379c/class_messages_listing/curriculum/Annotated_Readings/ClarkDISCOURSE-PROCESSES-20_Unannotated.pdf)  
26. Analyzing Patterns of Conversational Breakdown in Human-Chatbot Customer Service Conversations - UU Research Portal, accessed June 10, 2026, [https://research-portal.uu.nl/ws/files/263067946/978-3-031-88045-2_1.pdf](https://research-portal.uu.nl/ws/files/263067946/978-3-031-88045-2_1.pdf)  
27. Top 10 Deepfake Audio Detection Tools for 2025 | Resemble AI, accessed June 10, 2026, [https://www.resemble.ai/resources/audio-deepfake-detection-tools](https://www.resemble.ai/resources/audio-deepfake-detection-tools)  
28. How to Build a Production-Ready Voice Agent Architecture with WebRTC - freeCodeCamp, accessed June 10, 2026, [https://www.freecodecamp.org/news/how-to-build-production-ready-voice-agents/](https://www.freecodecamp.org/news/how-to-build-production-ready-voice-agents/)  
29. How Real-Time Voice AI Actually Works (STT → LLM → TTS, Explained) | Retell AI, accessed June 10, 2026, [https://www.retellai.com/blog/how-real-time-voice-ai-works-stt-llm-tts](https://www.retellai.com/blog/how-real-time-voice-ai-works-stt-llm-tts)  
30. How OpenAI delivers low-latency voice AI at scale | OpenAI, accessed June 10, 2026, [https://openai.com/index/delivering-low-latency-voice-ai-at-scale/](https://openai.com/index/delivering-low-latency-voice-ai-at-scale/)  
31. How Real-Time Voice Agents Work: Architecture and Latency, accessed June 10, 2026, [https://gokuljs.com/blogs/real-time-voice-agent-infrastructure](https://gokuljs.com/blogs/real-time-voice-agent-infrastructure)  
32. Getting Started - Deepgram's Docs, accessed June 10, 2026, [https://developers.deepgram.com/docs/live-streaming-audio](https://developers.deepgram.com/docs/live-streaming-audio)  
33. Gemini Live API Proactive, in Next.js and React Native Expo | by Felipe Lujan - Medium, accessed June 10, 2026, [https://felipelujan.medium.com/gemini-live-api-proactive-in-next-js-and-react-native-expo-26d070dafff9?postPublishedType=initial](https://felipelujan.medium.com/gemini-live-api-proactive-in-next-js-and-react-native-expo-26d070dafff9?postPublishedType=initial)  
34. The Art of Interruption: VAD Strategies for Fluid AI Conversations - DEV Community, accessed June 10, 2026, [https://dev.to/deepak_mishra_35863517037/the-art-of-interruption-vad-strategies-for-fluid-ai-conversations-15bh](https://dev.to/deepak_mishra_35863517037/the-art-of-interruption-vad-strategies-for-fluid-ai-conversations-15bh)  
35. Acoustic Echo Cancellation: How WebRTC AEC3 Works | Switchboard Audio SDK, accessed June 10, 2026, [https://switchboard.audio/hub/how-webrtc-aec3-works/](https://switchboard.audio/hub/how-webrtc-aec3-works/)  
36. How to Implement Silence Trimming in Your iOS App (Swift + AVAudioEngine, 2026 Guide), accessed June 10, 2026, [https://www.forasoft.com/blog/article/how-to-implement-silence-trimming-feature-to-your-ios-app-1720](https://www.forasoft.com/blog/article/how-to-implement-silence-trimming-feature-to-your-ios-app-1720)  
37. Real-Time Voicemail Detection in Telephony Audio Using Temporal Speech Activity Features - arXiv, accessed June 10, 2026, [https://arxiv.org/pdf/2604.09675](https://arxiv.org/pdf/2604.09675)  
38. VoxMind: An End-to-End Agentic Spoken Dialogue System - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2604.15710v1](https://arxiv.org/html/2604.15710v1)  
39. Voice activity detection (VAD) | OpenAI API, accessed June 10, 2026, [https://developers.openai.com/api/docs/guides/realtime-vad](https://developers.openai.com/api/docs/guides/realtime-vad)  
40. Streaming Speech Recognition API for Real-Time Transcription - Deepgram, accessed June 10, 2026, [https://deepgram.com/learn/streaming-speech-recognition-api](https://deepgram.com/learn/streaming-speech-recognition-api)  
41. Speech-to-Text WebSocket messages - Telnyx, accessed June 10, 2026, [https://developers.telnyx.com/docs/voice/stt/websocket-streaming/responses](https://developers.telnyx.com/docs/voice/stt/websocket-streaming/responses)  
42. Build a voice agent with function calling - AssemblyAI, accessed June 10, 2026, [https://www.assemblyai.com/blog/build-voice-agent-function-calling](https://www.assemblyai.com/blog/build-voice-agent-function-calling)  
43. Artificial Intelligence for Speech Classification and Enhancement of Speech and Language Disorders: Techniques, Applications, an - IEEE Xplore, accessed June 10, 2026, [https://ieeexplore.ieee.org/iel8/6287639/10820123/11199322.pdf](https://ieeexplore.ieee.org/iel8/6287639/10820123/11199322.pdf)  
44. VAD vs. Turn-Taking Endpoints in Conversational AI | Retell AI, accessed June 10, 2026, [https://www.retellai.com/blog/vad-vs-turn-taking-end-point-in-conversational-ai](https://www.retellai.com/blog/vad-vs-turn-taking-end-point-in-conversational-ai)  
45. Real-Time Voicemail Detection in Telephony Audio Using Temporal Speech Activity Features - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2604.09675v1](https://arxiv.org/html/2604.09675v1)  
46. Speech Disabilities & Digital Accessibility - ExceedAbility, accessed June 10, 2026, [https://exceedability.com/speech-disabilities.html](https://exceedability.com/speech-disabilities.html)  
47. SpeechAgent: An End-to-End Mobile Infrastructure for Speech Impairment Assistance - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2510.20113v1](https://arxiv.org/html/2510.20113v1)  
48. C2PA Implementation Guidance, accessed June 10, 2026, [https://spec.c2pa.org/specifications/specifications/1.0/guidance/Guidance.html](https://spec.c2pa.org/specifications/specifications/1.0/guidance/Guidance.html)  
49. An Analysis of Dialogue Repair in Voice Assistants - arXiv, accessed June 10, 2026, [https://arxiv.org/pdf/2311.03952](https://arxiv.org/pdf/2311.03952)  
50. VoiceAgentBench: Are Voice Assistants ready for agentic tasks? - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2510.07978v1](https://arxiv.org/html/2510.07978v1)  
51. How Speech and Language Disabilities Affect Online Experiences & Accessibility Best Practices - AudioEye, accessed June 10, 2026, [https://www.audioeye.com/post/speech-and-language-disabilities/](https://www.audioeye.com/post/speech-and-language-disabilities/)  
52. Listener Ratings of Stuttering: Evaluating Two Auditory–Perceptual Scales - PMC, accessed June 10, 2026, [https://pmc.ncbi.nlm.nih.gov/articles/PMC12806039/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12806039/)  
53. Inclusive AI Starts with Ethical Voice Data for Speech-Impaired and Atypical Speakers, accessed June 10, 2026, [https://blog.andovar.com/inclusive-ai-starts-with-ethical-voice-data-for-speech-impaired-and-atypical-speakers](https://blog.andovar.com/inclusive-ai-starts-with-ethical-voice-data-for-speech-impaired-and-atypical-speakers)  
54. Voice capability (TTS / STT / real-time audio) · Issue #116 - GitHub, accessed June 10, 2026, [https://github.com/pydantic/pydantic-ai-harness/issues/116](https://github.com/pydantic/pydantic-ai-harness/issues/116)  
55. Voiceitt - Inclusive Voice AI, accessed June 10, 2026, [https://www.voiceitt.com/](https://www.voiceitt.com/)

---

[← Back to Canon Map](../canon-map.md)