# How to Use This Canon

*Stunspot’s Guide to AI Systems* is designed to work in two ways:

1. as a human-readable field manual for practical AI systems design;
2. as model-readable reference material for AI Projects, RAG systems, long-context workspaces, and agentic design workflows.

The canonical source reports live in [`/docs/`](./).

Bundled upload formats live in [`/knowledge-packs/`](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs).

---

## Quick Start

Use the Guide in one of four ways:

| Use Case | Best Format |
|---|---|
| Reading and browsing the canon | [`/docs/`](./) |
| Uploading to an AI Project or RAG workspace | [`/knowledge-packs/by-part/`](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/by-part) |
| Uploading when smaller files or cleaner retrieval boundaries are preferred | [`/knowledge-packs/by-volume/`](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/by-volume) |
| Local search, archival use, or full-corpus import | [`/knowledge-packs/omnibus/`](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/omnibus) |

Most users should start with the [**By Part** pack](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/by-part). It gives broad coverage with low upload friction while avoiding both extremes: one giant file or dozens of separate reports.

---

## Use It as Project Knowledge

For ChatGPT Projects, Claude Projects, NotebookLM-style tools, or other AI workspaces, upload the relevant knowledge pack.

Recommended order:

1. Start with [`/knowledge-packs/by-part/`](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/by-part).
2. Upload only the parts relevant to your task when possible.
3. Use [`/knowledge-packs/by-volume/`](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/by-volume) when you want smaller files or more precise retrieval boundaries.
4. Use [`/knowledge-packs/omnibus/`](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/omnibus) only when the platform handles large single-file knowledge sources well.

The goal is to give the model enough canon context to reason well without forcing the retrieval system to chew one giant undifferentiated brick.

---

## Use It as RAG Substrate

For custom RAG systems, the best source is usually the canonical report structure in [`/docs/`](./).

Use individual reports when you want:

- precise chunking;
- better source attribution;
- cleaner retrieval boundaries;
- easier updates;
- report-level metadata;
- stronger citation and traceability.

Use [bundled packs](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs) when your platform is simpler and expects fewer files.

---

## Use It as an Architecture Reviewer

The Guide is especially useful when asking an AI system to review, critique, or improve an AI design.

Use it to evaluate:

- prompt systems;
- RAG pipelines;
- model-routing plans;
- agent workflows;
- tool/action boundaries;
- eval plans;
- governance models;
- product architecture;
- adoption plans;
- vendor strategy;
- operational readiness.

---

## Minimal Loading Strategy

Do not load the whole canon by default.

Load the smallest useful unit:

| Task | Suggested Loading |
|---|---|
| Prompting / model steering | [Volume 1](./volume-01/) |
| RAG / knowledge systems | [Volumes 1–2](./canon-map.md#volume-1--the-informationalepistemic-layer) |
| Model selection or serving | [Volumes 1](./volume-01/), [3](./volume-03/), [4](./volume-04/) |
| Agents and tool use | [Volumes 1](./volume-01/), [5](./volume-05/), [7](./volume-07/), [9](./volume-09/) |
| Multimodal systems | [Volumes 6](./volume-06/), [8](./volume-08/), [9](./volume-09/) |
| Security and failure analysis | [Volumes 7](./volume-07/), [9](./volume-09/), [10](./volume-10/) |
| Product and adoption | [Volumes 8](./volume-08/) and [11](./volume-11/) |
| Governance and operations | [Volumes 9–10](./canon-map.md#volume-9--observability-evaluation-and-verification) |
| System doctrine / design review | [Volume 12](./volume-12/) |
| Broad AI architecture review | [By Part pack](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/by-part) or [By Volume pack](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/by-volume) |

Load more only when the task actually needs it.

---

## Choosing Between Pack Formats

### [Source Reports](./canon-map.md)

Use [`/docs/`](./) when cloning the repo, building a serious RAG index, citing specific reports, or editing the canon.

### [By Part](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/by-part)

Use [`/knowledge-packs/by-part/`](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/by-part) as the recommended default for most AI Projects and RAG workspaces. It preserves major canon structure while keeping file count low.

### [By Volume](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/by-volume)

Use [`/knowledge-packs/by-volume/`](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/by-volume) when the By Part files are too large, when you want smaller upload units, or when cleaner retrieval boundaries matter more than file-count simplicity.

### [Omnibus](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/omnibus)

Use [`/knowledge-packs/omnibus/`](https://github.com/Stunspot/stunspots-guide-to-ai-systems/tree/main/knowledge-packs/omnibus) when you need the entire canon as one file for local search, archival use, or a platform that handles large single-file knowledge sources well.

For most hosted RAG and Project systems, the omnibus is not the best default.

Special note: NotebookLM has a powerful RAG/knowledge-graph infrastructure and can usually handle the omnibus file well.

---

## Suggested Instruction

When loading the Guide into an AI system, give the model a clear usage frame:

> Analyze, design, critique, or improve the requested AI system using *Stunspot’s Guide to AI Systems* as governing reference material, not decorative background reading. Retrieve and apply the Guide’s vocabulary, doctrine, design patterns, failure modes, interface logic, evaluation standards, and operational assumptions as the frame through which the system is understood. Do not merely summarize the Guide. Use it to improve the precision, realism, safety, and build-awareness of the work.

---

## Reading Paths

For human reading paths, start with the [Canon Map](./canon-map.md).

For upload formats, start with the [Knowledge Packs](./knowledge-packs.md) page.
