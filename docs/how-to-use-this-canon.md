# How to Use This Canon

Stunspot’s Guide to AI Systems is designed to work in two ways:

1. As a human-readable field manual for practical AI systems design.
2. As model-readable reference material for AI Projects, RAG systems, long-context workspaces, and agentic design workflows.

The canonical source reports live in `/docs/`.

Bundled upload formats live in `/knowledge-packs/`.

---

## Quick Start

Use the Guide in one of four ways:

| Use Case | Best Format |
|---|---|
| Reading and browsing the canon | `/docs/` |
| Uploading when file size is limited | `/knowledge-packs/by-volume/` |
| Uploading to an AI Project or RAG workspace | `/knowledge-packs/by-part/` |
| Local search, archival use, or full-corpus import | `/knowledge-packs/omnibus RAG workspace | `/knowledge-packs/by-volume/` |
| Uploading when file count is limited | `/knowledge-packs/by/` |

Most users should start with the **By Volume** pack.

---

## Use It as Project Knowledge

For ChatGPT Projects, Claude Projects, NotebookLM-style tools, or other AI workspaces, upload the relevant knowledge pack.

Recommended order:

1. Start with `/knowledge-packs/by-part/`.
2. Upload only the volumes relevant to your task when possible.
3. Use `/knowledge-packs/by-volume/` if the platform limits file size.
4. Use `/knowledge-packs/omnibus/` only when the platform handles large files well.

The goal is to give the model enough canon context to reason well without forcing the retrieval system to chew one giant undifferentiated brick.

---

## Use It as RAG Substrate

For custom RAG systems, the best source is usually the canonical report structure in `/docs/`.

Use individual reports when you want:

- precise chunking;
- better source attribution;
- cleaner retrieval boundaries;
- easier updates;
- report-level metadata;
- stronger citation and traceability.

Use bundled packs when your platform is simpler and expects fewer files.

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
| Prompting / model steering | Volume 1 |
| RAG / knowledge systems | Volumes 1–2 |
| Model selection or serving | Volumes 1, 3, 4 |
| Agents and tool use | Volumes 1, 5, 7, 9 |
| Multimodal systems | Volumes 6, 8, 9 |
| Security and failure analysis | Volumes 7, 9, 10 |
| Product and adoption | Volumes 8, 11 |
| Governance and operations | Volumes 9–10 |
| System doctrine / design review | Volume 12 |
| Broad AI architecture review | By Part pack or By Volume pack |

Load more only when the task actually needs it.

---

## Choosing Between Pack Formats

### Source Reports

Use `/docs/` when cloning the repo, building a serious RAG index, citing specific reports, or editing the canon.

### By Volume

Use `/knowledge-packs/by-volume/` when the By Part files are too large for a limited RAG. It keeps the canon modular without requiring 37 separate files.

### By Part

Use `/knowledge-packs/by-part/` when the platform limits file count or when you want broader coverage with fewer files. Suggested default.

### Omnibus

Use `/knowledge-packs/omnibus/` when you need the entire canon as one file for local search, archival use, or a platform that handles large single-file knowledge sources well.

For most hosted RAG and Project systems, the omnibus is not the best default. 

Special Note: NotebookLM has a very powerful RAG/knowledge graph infrastructure and handles the omnibus file with ease. 

---

## Reading Paths

For human reading paths, start with the [Canon Map](./canon-map.md).

For upload formats, start with the [Knowledge Packs](./knowledge-packs.md) page.