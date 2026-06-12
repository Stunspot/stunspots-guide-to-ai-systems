# AI-ENG-P — Multimodal Understanding - Documents, Images, Tables, Charts & Video

## **I. Doctrinal Foundations: The Evidence-Grounded Paradigm**

The structural integrity of a high-dimensional artificial intelligence system depends on a fundamental law of information representation: visual, spatial, and temporal structure is semantic content.1 Traditional natural language processing architectures treat documents, tables, charts, images, and videos as flat streams of serialized text tokens.1 This reductionist approach introduces systemic errors, as flattening multi-dimensional layouts into unstructured text destroys spatial coordinates, relational groupings, and chronological sequences.4  

Multimodal understanding is therefore defined as an evidence-grounded discipline of selecting, parsing, inspecting, and citing the precise visual, spatial, tabular, document, or temporal evidence unit before generating any text.6 A system has not understood an input unless it can isolate the coordinates of the supporting evidence, preserve the visual structure that contextualizes that evidence, and ground its claims back to page coordinates, structural cells, axis labels, temporal frames, or localized bounding boxes.6  

This paradigm establishes a clear distinction between seeing, parsing, retrieving, inspecting, extracting, grounding, and citing:

* A system may ingest a file without parsing its layout.  
* It may parse text without preserving spatial relationships.5  
* It may retrieve a page without inspecting the specific region containing the answer.6  
* It may inspect a cropped image without sufficient resolution to resolve small characters.2  
* It may extract a numerical data point without preserving its column headers, units, or footnotes.10  
* It may describe a chart without reading its axis scales.13  
* It may sample a video without capturing the specific temporal event that makes a claim true.15  
* It may synthesize a highly plausible, fluent textual answer that lacks any grounding in actual visual evidence.8

Plausible text is not visual grounding. To build resilient retrieval-augmented generation systems and interface-controlling agents, the system must enforce strict evidence selection before text generation. This report inherits the core doctrines of prior canon reports to maintain structural and conceptual continuity across the systems stack:

* **Context-Tenure and State-Governance (AI-ENG-B):** Applied here to visual and document evidence. The system must manage how long visual artifacts remain in the active reasoning context and track state transitions of document elements during multi-stage processing.  
* **Knowledge Hygiene and Source-Authority (AI-ENG-D):** Applied to multimodal artifacts. The system must prevent low-quality, unverified, or conflicting visual inputs from contaminating the reasoning loop.  
* **Retrieval-Pipeline Principles (AI-ENG-E):** Extended to multi-index visual routing. The system must select the optimal modality-specific retrieval pathway based on query profiles.  
* **Serving and Observability (AI-ENG-L):** Tailored for heavy multimodal workloads, managing token budgets, and tracing rendering latencies.  
* **Verification and User-Facing Status Discipline (AI-ENG-O):** Enforcing the rule that user-facing claims must never outrun verified visual reality. The system must refuse to assert a claim unless it preserves a continuous, auditable coordinate chain back to the source artifact.7

The architectural progression through these states of multimodal processing is outlined in the primary conceptual glossary.

### **Artifact 1: Conceptual Glossary**

| Term | Definition | Operational Metric |
| :---- | :---- | :---- |
| **Multimodal Understanding** | The evidence-grounded discipline of selecting, parsing, inspecting, and citing spatial, structural, visual, or temporal evidence before generating text. | Claim-to-Evidence Grounding Ratio (G_ratio) |
| **Late-Interaction Retrieval** | A retrieval paradigm matching multi-vector representations of visual page layouts against query tokens using fine-grained patch-level operations.1 | NDCG@5, MRR, Recall@k 1 |
| **MaxSim Operator** | A late-interaction scoring function computing the sum of the maximum cosine similarities between query token vectors and all page patch vectors.6 | MaxSim Score (S_MaxSim) 6 |
| **Spatially-Grounded Retrieval** | A hybrid retrieval method mapping visual patch-level similarity scores to OCR-extracted coordinate regions to isolate specific crops.6 | Localization Hit Rate at IoU@0.5 7 |
| **Relevance Propagation** | Aggregating visual patch-level similarity scores into localized OCR bounding boxes via spatial overlap calculations.6 | Intersection-over-Union (IoU) Weighting 7 |
| **Page-Object Table Transformer** | A parallel-decoded, DETR-based graph prediction model trained to recognize and relate tabular elements in full-page context.12 | Grid Table Similarity Content Score (GriTS_Con) 21 |
| **Split-Merge TSR** | A top-down table structure recognition method using sequence labeling to split grids and encoder-only transformers to classify merge cells.23 | Tree Edit Distance Similarity (TEDS_Struc) 24 |
| **Modality Conversion** | Translating a visually encoded plot or chart into a linearized structured table for downstream symbolic reasoning.13 | Relative Error Tolerance Table Match Score 13 |
| **VideoITG Framework** | An instruction-driven temporal grounding model adaptively altering video frame-sampling strategies based on query reasoning profiles.15 | Frame-Selection Recall / Temporal Grounding Accuracy 15 |
| **Event-Anchored Frame Selection** | A training-free video pipeline segmenting video streams into coherent visual events and selecting query-relevant anchors via adaptive MMR.26 | Downstream Video QA Accuracy Delta 26 |
| **Frame Selection Sensitivity** | A diagnostic measuring how much model accuracy changes when the most relevant frames are replaced with the least relevant frames.28 | FSS Delta (Delta FSS) 28 |
| **Language Independence Score** | A metric evaluating the baseline accuracy of a video question answering system operating strictly on textual queries without frame inputs.28 | LIS Accuracy (A_LIS) 28 |

The flow of evidence from initial file ingestion to final audited, cited answer generation is represented in the primary pipeline architecture. The system must maintain a continuous visual-spatial-temporal coordinate chain throughout this lifecycle.

### **Artifact 2: Multimodal Evidence Pipeline**

```
+---------------------------------------------------------------------------------------------------------+  
|                                     MULTIMODAL EVIDENCE PIPELINE                                        |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|  [ Ingestion ]                                                                                          |  
|       │                                                                                                 |  
|       ▼                                                                                                 |  
|  ──> ( Native Text Parser, > 100 characters ) ──>            |  
|       │ ( Scanned / Visual Page, < 100 characters )                                                     |  
|       ▼                                                                                                 |  
|  ──> ( 300 DPI High-Resolution Rasterization ) ──>             |  
|       │                                                                                                 |  
|       ├─────────────────────────────────────────┐                                                       |  
|       ▼                                         ▼                                                       |  
|  [ Late-Interaction Indexing ]                                          |  
|  ( ColPali Page-Patch Embedding )       ( Structural Entity Detection, DocTags )                        |  
|       │                                         │                                                       |  
|       ▼                                         ▼                                                       |  
|  ────────────────>                     |  
|  ( ANN Mean Pool -> MaxSim Rerank )     ( Paragraph, Table, Form Bounding Boxes )                       |  
|       │                                         │                                                       |  
|       └────────────────────┬────────────────────┘                                                       |  
|                            ▼                                                                            |  
|              [ Coordinate Alignment & Propagation ]                                                     |  
|              ( IoU Bounding Box Score Aggregation )                                                     |  
|                            │                                                                            |  
|                            ▼                                                                            |  
|              [ Evidence Validation Engine ]                                                             |  
|              ( Inspection Adequacy & Resolution Checks )                                                |  
|                            │                                                                            |  
|                            ▼                                                                            |  
|              [ High-Precision Feature Extractors ]                                                      |  
|              ( POTATR Tables / DePlot Charts / Form Fields )                                            |  
|                            │                                                                            |  
|                            ▼                                                                            |  
|              [ Grounding Map Compilation ]                                                              |  
|              ( Exact Coordinate Tracking: Page, Box, Frame )                                            |  
|                            │                                                                            |  
|                            ▼                                                                            |  
|              ──>     |  
|                                                                                                         |  
+---------------------------------------------------------------------------------------------------------+
```

## **II. Document Ingestion, OCR, and OCR-Free VLM Pipelines**

Document layout analysis is the architectural boundary where raw pixels are transformed into structured semantic systems.5 Legacy systems that rely on naive, sequential Document Layout Analysis (DLA) chains introduce error propagation at every step.5 A standard legacy pipeline rasterizes a PDF page, passes it to a context-blind OCR engine to extract raw text with coordinates, and then uses a heuristic parser to group characters into blocks.5  
Because the OCR engine operates without linguistic or structural context, it fails to differentiate between visual noise, mathematical formulas, columns, or tables.5 If any stage in this sequential chain stumbles, the layout structures collapse, yielding out-of-order text segments, orphaned table cells, and discarded captions.3  
To address this structural fragility, modern high-dimensional architectures implement a unified Document Parsing Stack. This stack integrates classical coordinate-aware text extraction with OCR-free and layout-aware Vision-Language Models.

### **Artifact 3: Document Parsing Stack Model**

| Layer | Input | Processing Architecture | Output Artifact | Fallback Trigger |
| :---- | :---- | :---- | :---- | :---- |
| **1. File Ingestion** | Heterogeneous PDFs, JPEGs, PNGs, TIFFs 4 | Rust-based pdf-inspector analysis of document internals (font encodings, operators, rasterized coverage) 29 | Standardized PDF stream + Metadata 2 | Force rendering of all pages to 300 DPI raster images 2 |
| **2. Selective Ingestion Routing** | Standardized PDF stream | Document Classifier (>100 characters text-native threshold check) 2 | Digital Native Text Path OR Scanned Visual Path 2 | Route to VLM rasterization if font rendering or encoding errors are detected 2 |
| **3. Page Rendering Engine** | Scanned Visual Path | PyMuPDF or pdf2image high-fidelity rasterizer 2 | 300 DPI Standardized PNG/TIFF page images 2 | Upscale to 400-600 DPI for micro-fonts or low-contrast targets 31 |
| **4. Layout & OCR-Free VLM Ingestion** | 300 DPI page images | Compact end-to-end VLMs (e.g., olmOCR-2-7B, GraniteDocling-258M) 5 | Structured layout graph + Raw text blocks 5 | Multi-stage OCR (PaddleOCR/EasyOCR local server) 10 |
| **5. Reading-Order Reconstruction** | Visual-spatial layout graph | Neural coordinate ordering with geometric reading path fallback 29 | Continuous serialized document stream in Markdown 29 | Heuristic column-segment splitting rules based on vertical whitespace analysis 3 |
| **6. Structural Markup & Tagging** | Structured document stream | DocTags intermediate representation compiler 5 | Semantically tagged hierarchical document tree 5 | Rule-based tag parsing of raw Markdown markers 10 |

The critical transition from legacy OCR to unified layout preservation is realized through models like the IBM Granite-Docling VLM series and the Allen Institute's olmOCR-2-7B.5  
Rather than chaining discrete layout classifiers and text line extractors, models such as Granite-Docling integrate vision and language processing inside a highly compact 258-million parameter architecture.5 It consumes rasterized page images and outputs structured text wrapped in a markup format known as DocTags.5 DocTags encodes structural elements—including headers, columns, tables, formulas, footnotes, and captions—as explicit hierarchical markers, ensuring that downstream LLMs receive layout-aware, contextually complete streams.5  
Similarly, the olmOCR-2-7B model parses mathematical equations, multi-column paper structures, and historical scans directly into clean Markdown, preserving complex mathematical expressions as LaTeX and stripping running headers and page numbers without breaking reading-order continuity.32 This visual-structural mapping is governed by a strict spatial layout tracking system.

### **Artifact 4: Layout Preservation Framework**

```
+-------------------------------------------------------------------------------------------------------+  
|                                     LAYOUT PRESERVATION FRAMEWORK                                     |  
+-------------------------------------------------------------------------------------------------------+  
|                                                                                                       |  
|                                                 |  
|                                                                                                       |  
|  ┌─────────────────────────────────────────────────────────────────────────────────────────────────┐  |  
|  │ Coordinates:                                                                  │  |  
|  ├─────────────────────────────────────────────────────────────────────────────────────────────────┤  |  
|  │                                                                                                 │  |  
|  │  Page Header Segment: "Journal of Systems Architecture" (Omitted from Main Semantic Stream)    │  |  
|  │                                                                                                 │  |  
|  │  Column 1 Box:                   Column 2 Box:                                                  │  |  
|  │  ┌──────────────────────────────────────────┐      ┌──────────────────────────────────────────┐  │  |  
|  │  │ [Heading Node (H1)]                    │      │                         │  │  |  
|  │  │ "1. Introduction"                       │      │ "The experimental results..."            │  │  |  
|  │  │ Box:                │      │ Box:                │  │  |  
|  │  ├──────────────────────────────────────────┤      ├──────────────────────────────────────────┤  │  |  
|  │  │ [Paragraph Node]                         │      │ [Embedded Image Figure]                  │  │  |  
|  │  │ "The visual structures..."               │      │ Class: Figure                            │  │  |  
|  │  │ Box:                │      │ Box:                │  │  |  
|  │  ├──────────────────────────────────────────┤      ├──────────────────────────────────────────┤  │  |  
|  │  │ [In-line Equation Node]                  │      │ [Figure Caption Node]                    │  │  |  
|  │  │ Plaintext: S(q, d)                       │      │ "Figure 2: Grounding..."                 │  │  |  
|  │  │ Box:                │      │ Box:                │  │  |  
|  │  ├──────────────────────────────────────────┤      └──────────────────────────────────────────┘  │  |  
|  │  │                       │                                                    │  |  
|  │  │ Table Box:            │                                                    │  |  
|  │  │ (Linked via parent-child spatial edge)   │                                                    │  │  |  
|  │  └──────────────────────────────────────────┘                                                    │  |  
|  │                                                                                                 │  |  
|  ├─────────────────────────────────────────────────────────────────────────────────────────────────┤  |  
|  │ Coordinates:  Page Footer: "Page 1 of 15" (Omitted from Stream)            │  |  
|  └─────────────────────────────────────────────────────────────────────────────────────────────────┘  |  
+-------------------------------------------------------------------------------------------------------+
```

## **III. Forms and Field Understanding: Spatial-Semantic Association**

The extraction of structured values from forms and administrative documents is a specialized spatial-semantic alignment problem. Forms represent a high-density visual layout where meaning is defined by label-to-value geometric vectors, checkbox selections, handwritten annotations, signatures, and stamps.  
When a form document is processed by standard text extraction tools, its layout is destroyed. The textual fragments are serialized left-to-right, top-to-bottom, separating label tokens from their corresponding value tokens, and stripping checkbox states and handwritten signatures entirely.4  
To preserve the spatial-logical association of form elements, architectures must employ a dedicated Form Understanding Model. This model maps localized regions to a semantic schema, tracking confidence and spatial coordinates.

### **Artifact 5: Form Understanding Model**

```
Form Object Schema (Page Coordinate Grid: Normalized 0-1000)  
 ├── Form Metadata  
 │    ├── Coordinate Anchor Base: [x1, y1, x2, y2] =   
 │    └── Page Orientation Rotational Angle: 0.0 degrees  
 ├── Form Fields (Array of Spatial Nodes)  
 │    ├── Field 1 (Text-Native Input)  
 │    │    ├── Field Key Identifier: "vendor_name"  
 │    │    ├── Label Bounding Box:   
 │    │    ├── Value Bounding Box:   
 │    │    ├── Visual Extraction Modality: Text-Native (Digital Extract)  
 │    │    ├── Raw Extracted Text: "ACME Industrial Corp."  
 │    │    └── Field Confidence: 1.000 (Native Match)  
 │    ├── Field 2 (Handwritten Input)  
 │    │    ├── Field Key Identifier: "invoice_date"  
 │    │    ├── Label Bounding Box:   
 │    │    ├── Value Bounding Box:   
 │    │    ├── Visual Extraction Modality: Scanned Handwriting OCR  
 │    │    ├── Raw Extracted Text: "04/05/2026"  
 │    │    └── Field OCR Character Confidence: 0.895  
 │    ├── Field 3 (Checkbox / Selection Node)  
 │    │    ├── Field Key Identifier: "tax_exempt_status"  
 │    │    ├── Anchor Label Box:   
 │    │    ├── Checkbox Core Box:   
 │    │    ├── Pixel Intensity / Active Selection Ratio: 0.782  
 │    │    ├── Inferred Selection State: TRUE (Active Check Detected)  
 │    │    └── State Classifier Confidence: 0.981  
 │    └── Field 4 (Signature Validation Node)  
 │         ├── Field Key Identifier: "authorized_signature"  
 │         ├── Area Bounding Box:   
 │         ├── Signature Feature State: Verified Scribble (Dynamic Stroke/Ink Present)  
 │         ├── Stroke Pixels Count: 14203 pixels  
 │         └── Signature Verification Confidence: 0.994  
 └── Verification Logs  
      ├── Stamp / Seal Presence: TRUE (Detected Box , Conf: 0.970)  
      └── Empty Fields Handlers: [x1, y1, x2, y2] visual check indicates blank space on optional fields
```

When evaluating form understanding systems in production, developers must address several common architectural failure modes:

* **Label-Value Cross-Binding:** Under multi-column form layouts, standard spatial heuristic parsers often bind a field label to the value situated directly to its right, even if the form layout intends a vertical association. The system must utilize layout-aware attention boundaries to prevent horizontal spilling.  
* **Checkbox Inversion:** Minor rasterization noise or compression artifacts can fill a blank box or erase a light pencil mark. This causes the model to invert a critical state (e.g., classifying "Tax Exempt" as FALSE when it is checked). The pipeline must calculate the average pixel intensity of the checkbox interior relative to local page background density to establish a verifiable threshold.  
* **Signature/Stamp Omission:** Models optimized purely for OCR text character extraction often classify signatures and corporate seals as visual noise, failing to register their presence. A downstream compliance agent will then reject a valid document for lacking authorization. The architecture must deploy parallel visual object-detection heads specifically trained to classify signature scribble matrices and validate seals.  
* **Intentionally Empty Fields:** A standard parser often cannot distinguish between a field left blank by a user and an extraction step that failed. The form schema must explicitly capture coordinates for every field defined in the template. If no visual text, handwriting strokes, or check marks are present in the region, the field must be recorded as *Intentionally Empty* with a dedicated coordinate confidence score, rather than being omitted from the structured extraction payload.

## **IV. Tables as Structured Evidence: Multi-Page Recognition, Split-Merge, and Graph-Based Transformations**

Tables represent highly dense, structured data where spatial layout dictates numerical meaning.3 Extracting tabular data is a demanding task; standard OCR pipelines destroy the horizontal alignment, column grouping, and header hierarchies that give numbers context.4  
If a system flattens table cells into continuous prose, a downstream model will misbind numbers to the wrong headers, lose merged-cell relationships, and generate incorrect calculations.3 A table cell separated from its column and row header is not an informative fact—it is a hallucination risk.10  
To address these challenges, architectures deploy table extraction models like POTATR (Page-Object Table Transformer) and TABLET.12

### **Artifact 6: Table Extraction and Grounding Model**

| Architectural Path | Core Model | Processing Paradigm | Class Definitions / Targets | Structural Verification Metric |
| :---- | :---- | :---- | :---- | :---- |
| **Parallel Graph Prediction** | **POTATR-v1.0-Pub** 12 | Image-to-graph model extending TATR with parallel spatial decoding.12 Predicts parent-child structural edges between elements.12 | 16 Classes: 8 base (table, column, row, column header, projected row header, spanning cell, table caption, table footer) + 8 rotated equivalents.12 | **Grid Table Similarity Content Score (GriTS_Con)**: Pseudo F1-score for global consistency across cells, reflecting content and structure.21 |
| **Top-Down Split-Merge Sequence Labeling** | **TABLET Split-Merge Model** 24 | Non-autoregressive encoder-only Transformer.24 Formulates splitting as sequence labeling and merging as grid cell classification.24 | 2 Stages: Split Model (modified ResNet-18 + FPN backbone + dual encoders) 24; Merge Model (ResNet-18 + FPN + 7x7 RoIAlign on grid cells).24 | **Tree Edit Distance Similarity (TEDS)** and **TEDS-Struc**: Evaluates cell-level tree alignment.24 |
| **Traditional Multi-Stage Pipeline** | **Base TATR (Table Transformer)** 21 | DETR-based object detection on cropped table regions.21 Relies on post-processing overlaps with table class boundaries.21 | 6 Classes: Table, table column, table row, table column header, table projected row header, table spanning cell.12 | **Intersection-over-Union (IoU)** of predicted structural element bounding boxes.21 |

The POTATR architecture addresses a critical limitation of previous table recognition models: processing tables in isolation after cropping them from the page.21  
By operating directly on full-page images, POTATR preserves the table's context, including its caption and footer elements, which often contain critical units, scaling factors, or footnote mappings.12  
Rather than relying on overlapping bounding boxes to group elements, POTATR predicts directed parent-child relationships between detected objects via an integrated relation head.12 This constructs a hierarchical spatial graph of the table directly from the page image.12  
When table density escalates, the system uses the TABLET model.24 TABLET bypasses the instability of bounding box predictions for large, densely populated tables.23  
The Split Model removes the max-pooling layer of ResNet-18 to maintain high-resolution feature sequences, feeding them into separate horizontal and vertical Transformer encoders to label row and column split lines.24  
The Merge Model uses 7x7 Region of Interest Align (RoIAlign) to extract cell feature maps directly from the FPN layer, employing a cell-merging Transformer trained via Focal Loss to classify whether adjacent cells should be merged 24:  
FL_merge = (1 / (R * C)) * sum_{k=1}^{R * C} (alpha_k * (1 - p_k)^gamma * (-log(p_k)))  
This formula ensures the model is optimized for sparse merge-cell structures within densely populated grids.24

```
+---------------------------------------------------------------------------------------------------------+  
|                                     MULTI-PAGE CONTINUITY RESOLUTION                                    |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|  [ Page N Image ]                                     [ Page N+1 Image ]                                |  
|  ┌──────────────────────────────────────────────┐      ┌──────────────────────────────────────────────┐  |  
|  │...                                         │      │...                                          │  |  
|  │  ┌────────────────────────────────────────┐  │      │  ┌────────────────────────────────────────┐  │  |  
|  │  │ Column 1  │ Column 2   │ Column 3      │  │      │  │ Column 1  │ Column 2   │ Column 3      │  │  |  
|  │  ├───────────┼────────────┼───────────────┤  │      │  ├───────────┼────────────┼───────────────┤  │  |  
|  │  │ Item 44   │ $2410      │ Metric-A      │  │      │  │ Item 48   │ $1105     │ Metric-B      │  │  |  
|  │  └───────────┴────────────┴───────────────┘  │      │  └───────────┴────────────┴───────────────┘  │  |  
|  │                   │      │...                                          │  |  
|  └──────────────────────┬───────────────────────┘      └──────────────────────┬───────────────────────┘  |  
|                         │                                                     │                          |  
|                         ▼                                                     ▼                          |  
|                  [ Extraction Node N ]                 [ Extraction Node N+1 ]                    |  
|                  └── Table Node 1 ──┐                                  └── Table Node 2 ──┐              |  
|                                     ▼                                                     ▼              |  
|                                    [ Multi-Page Conjoining Logic Engine ]                                |  
|                             │   - Feature 1: Sub-header match similarity (0.941)                         |  
|                             │   - Feature 2: Column dimension alignment (3 cols == 3 cols)               |  
|                             │   - Feature 3: No terminal total row on Page N                             |  
|                             ▼                                                                            |  
|                     ( Continuation State Inferred: TRUE, Accuracy ~99% )                             |  
|                             │                                                                            |  
|                             ▼                                                                            |  
|                                                                  |  
|                     ( Merges Node 1 and Node 2, preserves header structures, offsets rows )              |  
|                                                                                                         |  
+---------------------------------------------------------------------------------------------------------+
```

To validate these extractions, the system measures structural alignment using Grid Table Similarity (GriTS) and Tree Edit Distance Similarity (TEDS).21  
While TEDS models the table as an HTML tree structure and computes edit distances on nodes, GriTS acts as a cell-level pseudo-F1 score that enforces global consistency across rows and columns.21 GriTS-Top measures structure alone, and GriTS-Con incorporates text content validation, exposing OCR character misreads within cells.22

## **V. Charts, Plots, and Visualized Data: Modality Conversion and Symbolic Reasoning**

Charts, diagrams, and plots represent visual data encodings. To interpret a chart, a system must isolate its type, parse its axis scales, map its tick intervals, bind data series using legends, and extract the marks that encode numeric values.14  
Relying on a Vision-Language Model to answer quantitative questions from visual impressions alone leads to errors.13 Models struggle with linear-to-logarithmic scale transitions, truncated baselines, and overlapping labels, often summarizing data through general impressions rather than precise values.13  
To address these limitations, the architecture utilizes a chart interpretation pipeline to translate visual structures into structured datasets.

### **Artifact 7: Chart Interpretation Pipeline**

```
+-------------------------------------------------------------------------------------------------------+  
|                                     CHART INTERPRETATION PIPELINE                                     |  
+-------------------------------------------------------------------------------------------------------+  
|                                                                                                       |  
|  [ Input Visual Chart / Plot Image ]                                                                  |  
|       │                                                                                               |  
|       ▼                                                                                               |  
|  ──> ( Line, Vertical/Horizontal Bar, Scatter, Pie ) [46]             |  
|       │                                                                                               |  
|       ▼                                                                                               |  
|  [ Axis & Metadata Localizer ]                                                                        |  
|       ├── Title String Extraction & Coordinate Anchor Mapping                                     |  
|       ├── Legend Segment Detection & Color-Series Associative Mapping                             |  
|       └── Label & Ticks Parsing (X-axis string/date, Y-axis numerical values) [46]                    |  
|       │                                                                                               |  
|       ▼                                                                                               |  
|                                                    |  
|       ├── Pixel-to-Value Mapping Calculation (e.g., Pixel Y=450 corresponds to Value 10,000)          |  
|       └── Scale Type Determination (Linear, Logarithmic, Exponential)                                 |  
|       │                                                                                               |  
|       ▼                                                                                               |  
|  [ Visual Mark Extractor ]                                                                            |  
|       ├── Spatial Mark Localization (Keypoints, bar boundaries, scatter coordinates)              |  
|       ├── High-Precision Visual Reconstruction (Translates pixel offsets to tabular values)            |  
|       └── Linearized Table Generation (Maps series labels to reconstructed coordinates)           |  
|       │                                                                                               |  
|       ▼                                                                                               |  
|                                                                 |  
|       ├── LLM Chain-of-Thought / Program-of-Thoughts Execution over Linearized Table [25]              |  
|       └── Summary Statistics Validation (Mean, Min, Max consistency check)                        |  
|       │                                                                                               |  
|       ▼                                                                                               |  
|  [ Grounded Final Fact Output ]                                                                       |  
|                                                                                                       |  
+-------------------------------------------------------------------------------------------------------+
```

The core modality conversion is executed by the DePlot engine, an end-to-end image-to-text Transformer trained specifically to translate visual charts directly into linearized tables.13  
This decoupling of visual extraction from numeric reasoning improves performance on downstream ChartQA benchmarks.13 Rather than forcing a VLM to solve complex multi-hop mathematical queries over visual inputs, DePlot extracts the data table, allowing an LLM to execute symbolic python code (Program of Thoughts) to compute exact calculations.13  
To improve structural accuracy, DePlot v2 models incorporate explicit structural pretraining tasks:

* **Visual Structure Prediction:** The model is trained to predict discrete metadata about the plot, such as the exact chart type, dominant color palettes associated with data series, and the coordinates of titles and legend items.14  
* **Summary Statistics Prediction:** Pretraining objectives require the model to output calculated summary metrics (mean, maximum, minimum, and standard deviation) directly from the visual representation, regularizing the visual encoder's scale awareness.14  
* **Numerical Comparison:** The model is tasked with outputting comparative relational assertions (e.g., "Series A is greater than Series B at tick interval X") directly from the chart image, aligning visual comparisons with logical operations.14

## **VI. Images, Visual Grounding, and Region Evidence: Spatial Coordinate Mapping**

Visual grounding is the architectural process that maps text assertions to spatial pixel coordinates.8 General image captioning models describe a scene globally but fail to isolate the precise visual coordinates that support a claim.8  
Without spatial grounding, a system cannot verify whether its answer is based on actual image evidence or visual hallucination.8 In domains like insurance auditing, engineering diagram analysis, and contract validation, the system must show exactly what visual regions it inspected.2

### **Artifact 8: Visual Grounding Model**

```
+---------------------------------------------------------------------------------------------------------+  
|                                         VISUAL GROUNDING MODEL                                          |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|                                                                                  |  
|                                                                                                         |  
|  0,0 ─────────────────────────────────────────────────────────────────────────────────────────────┐     |  
|   │                                                                                              │     |  
|   │                                                                    │     |  
|   │  Bounding Box:                                                            │     |  
|   │  Semantic Class: Text Block                                                                  │     |  
|   │  Confidence: 0.985                                                                           │     |  
|   │  Extracted Token Text: "The operating pressure must not exceed 150 PSI under standard load." │     |  
|   │                                                                                              │     |  
|   │                                                                    │     |  
|   │  Bounding Box:                                                           │     |  
|   │  Semantic Class: Technical Diagram                                                           │     |  
|   │  Confidence: 0.991                                                                           │     |  
|   │                                                                                              │     |  
|   │       (Spatial Anchor Sub-Region: Pressure Gauge Visual Mark)                                │     │  
|   │       Bounding Box:                                                      │     │  
|   │       Visual Attribute: Needle pointing to red zone at 180 PSI                               │     │  
|   │       Anchor Target ID: "mark_pressure_gauge_needle"                                         │     │  
|   │       Grounding Confidence: 0.942                                                            │     │  
|   │                                                                                              │     |  
|   └──────────────────────────────────────────────────────────────────────────────────────────────┘     |  
|                                                                                                 1000,1000|  
|                                                                                                         |  
|  Grounding Coordination Array:                                                                          |  
|  {                                                                                                      |  
|    "claim": "The equipment was operating in an over-pressure condition exceeding manual limits.",        |  
|    "evidence_regions": [                                                                                |  
|      { "id": "crop_1", "coordinates": , "modality": "text", "source": "page_12" },   |  
|      { "id": "gauge_needle", "coordinates": , "modality": "visual", "source": "p12" }|  
|    ],                                                                                                   |  
|    "grounding_good_ratio": 0.963,                                                                       |  
|    "spatial_semantic_alignment_score": 0.947                                                            |  
|  }                                                                                                      |  
+---------------------------------------------------------------------------------------------------------+
```

To resolve the limitation of page-level retrieval in RAG systems, architectures implement spatially grounded region retrieval via the Snappy paradigm.6  
While ColPali provides page-level retrieval by encoding pages as 1024 patch embeddings, it does not natively output the precise sub-page bounding boxes containing the evidence.1  
Snappy unifies this visual-patch representation with OCR coordination grids. During offline indexing, pages are rendered, patch embeddings are computed, and OCR engines extract text blocks with explicit bounding boxes.  
At query time, Snappy repurposes ColPali's late-interaction attention weights to construct a visual relevance heatmap across the patch grid.6 For each page-patch j in a grid of W_grid x H_grid with page dimensions W x H, its spatial boundaries are mapped:  
x_1 = (j mod W_grid) * (W / W_grid) y_1 = floor(j / W_grid) * (H / H_grid) x_2 = x_1 + (W / W_grid) y_2 = y_1 + (H / H_grid)  
The visual score for patch j is calculated as the maximum cosine similarity to any query token vector q_i 7:  
score_patch(j) = max_i ( (q_i. p_j) / (||q_i|| * ||p_j||) )  
This score is then propagated to the OCR bounding boxes (B) via Intersection-over-Union (IoU) spatial weighting 7:  
score_region(B) = sum_{j in P} ( IoU(P_j, B) * score_patch(j) )  
where P_j is the patch coordinate box and P is the set of patches overlapping with B.7  
This allows the retrieval engine to isolate and extract high-relevance visual crops, reducing downstream context window consumption by up to 52.3% and mitigating attention dilution in the generator.6

## **VII. Video Evidence Sampling and Temporal Grounding: Event-Aware Structures**

Video streams introduce a temporal dimension that challenges standard spatial context processing. Video data cannot be treated as an unstructured sequence of independent frames; doing so results in massive token redundancy, lost chronological context, and an inability to track motion and duration.15  
If a system samples frames at random or at a static, low frame rate, it risks missing brief, critical visual events, reversing cause-and-effect relationships, and generating ungrounded answers.15

```
+-------------------------------------------------------------------------------------------------------+  
|                                     VIDEO EVIDENCE SELECTION WORKFLOW                                 |  
+-------------------------------------------------------------------------------------------------------+  
|                                                                                                       |  
|                                                                         |  
|                                                                                                       |  
|  ┌─────────────────────────────────────────────────────────────────────────────────────────────────┐  |  
|  │ Stage 1: DINOv2 Visual Boundary Segmenter                                                      │  |  
|  │                                                                                                 │  |  
|  │  F(1)  F(30)  F(60)  F(90)  F(120)  F(150)  F(180)                                               │  |  
|  │  [●]───[●]───[●]───[●]────[○]────[○]────[○]  <-- Temporal Cosine Similarity Curve                │  |  
|  │                             ▲ (Local Similarity Minimum Detected at t=120s)                     │  |  
|  │                                                                                                 │  |  
|  │  Result: Temporal Event Boundaries Established :                                             │  |  
|  │    Event A: 0s to 120s (Visually Homogeneous Scene A)                                           │  |  
|  │    Event B: 121s to 180s (Visually Homogeneous Scene B)                                         │  |  
|  └─────────────────────────────────┬───────────────────────────────────────────────────────────────┘  |  
|                                    │                                                                  |  
|                                    ▼                                                                  |  
|  ┌─────────────────────────────────────────────────────────────────────────────────────────────────┐  |  
|  │ Stage 2: BLIP2-ITM Query Relevance Localizer                                                   │  |  
|  │                                                                                                 │  |  
|  │  Query: "How many times does the technician press the red button before checking the gauge?"    │  |  
|  │                                                                                                 │  |  
|  │  - Event A Candidate Frames matched against Query via BLIP-2 ITM Semantic Scoring           │  |  
|  │  - Event A Anchor Selected: Frame at t=45s (Relevance: 0.945)                                   │  |  
|  │  - Event B Anchor Selected: Frame at t=155s (Relevance: 0.812)                                  │  |  
|  └─────────────────────────────────┬───────────────────────────────────────────────────────────────┘  |  
|                                    │                                                                  |  
|                                    ▼                                                                  |  
|  ┌─────────────────────────────────────────────────────────────────────────────────────────────────┐  |  
|  │ Stage 3: Adaptive MMR Refiner                                                                  │  |  
|  │                                                                                                 │  |  
|  │  Optimizes keyframe set for query relevance, event coverage, and visual diversity.             │  |  
|  │  Prevents sampling redundancy (e.g., discarding a second, visually identical frame at t=46s).    │  |  
|  │                                                                                                 │  |  
|  │  Final Selected Grounded Keyframe Set:                                                          │  |  
|  │    - Frame 1: t=45s (Anchor, Technician presses button)                                     │  |  
|  │    - Frame 2: t=90s (Refinement Frame, Technician presses button again)                     │  |  
|  │    - Frame 3: t=155s (Anchor, Technician inspects pressure dial)                            │  |  
|  └─────────────────────────────────────────────────────────────────────────────────────────────────┘  |  
+-------------------------------------------------------------------------------------------------------+
```

Architectures implement structured frame selection and temporal grounding strategies to manage long-video inputs.

### **Artifact 9: Video Evidence Sampling and Temporal Grounding Model**

| Sampling Strategy | System Component | Core Algorithmic Mechanism | Temporal Alignment Target | Handling of Missed-Event Uncertainty |
| :---- | :---- | :---- | :---- | :---- |
| **Event-Anchored Frame Selection (EFS)** | **EFS Pipeline (Training-Free)** | DINOv2 self-supervised visual segment partitioning + BLIP2-ITM query-anchor selection + Adaptive MMR global refinement. | Visually homogeneous temporal segments representing semantic events. | Dynamic scaling of the Adaptive MMR diversity threshold based on video-content statistics. |
| **Instruction-Conditioned Grounding (VideoITG)** | **VidThinker Annotation & Sampling Pipeline** | Adaptively matches frame selection categories to query profiles (Semantic, Motion, Hybrid, Non-Clues). | Segment localization via instruction-guided cross-modal reasoning. | Falls back to diverse global sampling (Beginning-Middle-End) if no visual cues are detected. |
| **Wavelet-Boundary Selection (WFS-SB)** | **Multi-Resolution Wavelet Pipeline** | Wavelet transform decomposition of the similarity signal to clear high-frequency noise. | Semantic change signals extracted from the coarsest transform scale. | Local extrema check on coarse scales identifies robust boundaries under noise. |

The VideoITG VidThinker pipeline classifies incoming queries into four categories to align sampling density with reasoning needs 15:

* **Semantic-Only:** Queries focus on static objects, actors, or scenes (e.g., "What tools are on the workbench?"). The sampler extracts frame-level CLIP features and computes cosine similarities between adjacent frames.15 It retains a frame only when its similarity to the prior keyframe falls below a scene-change threshold, maximizing visual diversity.15  
* **Motion-Only:** Queries target dynamic actions, speeds, or pathways (e.g., "Which direction did the vehicle turn?"). The pipeline bypasses scene-change filters, deploying high-frequency, fixed-rate sampling within the target segment to preserve temporal progression.15  
* **Semantic & Motion:** Queries combine static identification and dynamic progress tracking (e.g., "Describe how the operator adjusted the valve"). The pipeline uses hybrid sampling, running fixed-rate extraction in regions with high motion indicators while retaining semantic anchor frames from stable scenes.15  
* **Non-Clues:** Queries contain global requests lacking explicit temporal or semantic anchors (e.g., "Please describe the video in detail"). The sampler extracts a compact, diverse keyframe set distributed evenly across the video sequence.15

To evaluate the validity of video benchmarks and prevent language-prior bias, architectures implement Frame Selection Sensitivity (FSS) and Language Independence Score (LIS) diagnostics.28  
The FSS diagnostic isolates a VLM's true frame-level sensitivity by measuring the performance delta when the system receives the K most query-relevant frames (MaxProb) versus the K least relevant frames (MinProb).28  
The LIS diagnostic evaluates the VLM's performance on the benchmark questions when given zero frame inputs, exposing language-prior shortcuts.28 If a benchmark demonstrates high LIS and low FSS, the questions are frame-agnostic, and high model scores reflect language reasoning rather than visual temporal understanding.28 System evaluation must prioritize frame-sensitive tasks to measure temporal grounding.

## **VIII. Multimodal Embeddings, Retrieval, and Indexing: Hybrid Storage**

To serve diverse multimodal queries at scale, high-dimensional AI systems cannot rely on a single, flattened vector database.2  
Enterprise corpora contain a mixture of text-native documents, complex visual tables, engineering diagrams, and videos, requiring a hybrid indexing strategy.2 This indexing strategy must preserve textual content, layout hierarchies, and visual patch representations in parallel, routing queries dynamically to the optimal index endpoint.2

### **Artifact 10: Multimodal Indexing Strategy**

```
Multimodal Storage Cluster  
 ├── Vector Database (e.g., Qdrant / Milvus with Multi-Vector Collections) [18]  
 │    ├── Late-Interaction Index (Rank Vectors Field)  
 │    │    ├── Content: 1024 patch embeddings per page (ColPali SigLIP projected to 128-dim)   
 │    │    ├── Dimension: 1024 x 128 float32 vectors (~527KB per page) [18, 50]  
 │    │    └── Index Configuration: HNSW with MaxSim Distance Metric   
 │    └── Single-Vector Index (Dense Field)  
 │         ├── Content: Mean-pooled ColPali page embeddings [7, 51]  
 │         ├── Dimension: 128-dim float32 vector  
 │         └── Index Configuration: Standard Cosine Search for ANN Candidates [7, 51]  
 ├── Relational Database (e.g., PostgreSQL with pgvector)   
 │    ├── Text-Native Index  
 │    │    ├── Content: Native PDF extracted strings + Markdown layouts [31, 35]  
 │    │    └── Search Method: BM25 / Hybrid Dense-Sparse Routing   
 │    └── Spatial Coordinate Registry (OCR Metadata Field)  
 │         ├── Content: Bounding box coordinates [x1, y1, x2, y2] + DocTags markup [5, 7]  
 │         └── Search Method: SQL-based spatial overlapping query checks  
 └── Object Storage (e.g., MinIO / AWS S3)   
      ├── Page Image Store (300 DPI high-fidelity rendered PNGs)   
      ├── Image Region Crops (SAM and POTATR segmented boxes)   
      └── Video Keyframe Clusters (EFS and VideoITG sampled frames) 
```

At query time, the system uses a Multimodal Retrieval Decision Model to map the incoming query profile to the optimal retrieval endpoint.

### **Artifact 11: Multimodal Retrieval Decision Model**

| Query Profile | Target Document Type | Primary Index Route | Retrieval Method | Reranking / Post-Processing |
| :---- | :---- | :---- | :---- | :---- |
| **Lexical / Entity-Specific** (e.g., "Retrieve Invoice #A10-9") | Text-Native Digital PDF | Text-Native Index | BM25 lexical search with exact metadata filtering | Contextual Chunk Header Injection (injects metadata like title and date) |
| **Visual-Spatial Layout** (e.g., "Find slides with a three-box comparison diagram") | Visually Rich Presentations / Manuals | Late-Interaction Index | Page-image late-interaction MaxSim search | Late-interaction MaxSim scoring on candidate set |
| **Spatially Grounded Factual** (e.g., "What is the Q3 operating margin in Table 4?") | Scanned PDFs with complex structural tables | Hybrid Index (Late-Interaction + OCR Coordinate Registry) | Two-stage retrieval (ANN candidates -> Full patch MaxSim scoring) | **Snappy Relevance Propagation**: IoU-weighted patch score mapping |
| **Temporal Action Sequence** (e.g., "Show when the operator flips the emergency switch") | Video Sequence Database | Video Keyframe & Temporal Index | VideoITG Instruction-conditioned clip-level matching | **Event-Anchored Frame Selection**: DINOv2 visual segment partitioning + Adaptive MMR |

The physical ingestion pipelines are separated using explicit classification gates to prevent routing text-native documents through expensive VLM inference.

```
+---------------------------------------------------------------------------------------------------------+  
|                                  INGESTION ROUTING CLASSIFIER FLOW                                      |  
+---------------------------------------------------------------------------------------------------------+  
|                                                                                                         |  
|                                                                       |  
|       │                                                                                                 |  
|       ▼                                                                                                 |  
|                                                                  |  
|       │                                                                                                 |  
|       ├──> ( Native Text Characters Count > 100 ) ──────────────────>      |  
|       │                                                               ( Low cost, fast extraction )     |  
|       │                                                                                                 |  
|       └──> ( Native Text Characters Count <= 100 ) ─────────────────>|  
|                                                                       ( Rasterize at 300 DPI, run VLM)  |  
|                                                                                                         |  
|  Operational Impact :                                                                                |  
|  - In standard enterprise corpora, this selective routing bypasses VLM ingestion for 60% to 75% of      |  
|    pages, reducing ingestion cost and API latency proportionally without losing visual accuracy.       |  
|                                                                                                         |  
+---------------------------------------------------------------------------------------------------------+
```

## **IX. Evidence Selection and Inspection Adequacy**

Before passing retrieved evidence to the visual generator, the system must assess whether the evidence is adequate to answer the query.2 An agent loop must evaluate whether the target crop contains enough resolution, whether the layout context has been preserved, and whether neighboring visual elements must be retrieved to make an audit-proof claim.2

### **Artifact 12: Evidence Selection Quality Framework**

```
Evidence Quality Diagnostic Matrix  
 ├── 1. Ingestion Quality Assessments  
 │    ├── Page Resolution Verification  
 │    │    ├── Metric: Rendering DPI check  
 │    │    ├── Rule: If DPI < 150, trigger high-fidelity upscaling or raise Low-Resolution Warning   
 │    │    └── Operational Limit: Characters compressed below vision token grid limits cannot be resolved   
 │    ├── Document Skew & Orientation  
 │    │    ├── Metric: Rotational angle detection  
 │    │    └── Action: Run geometric deskewing if skew angle > 3 degrees   
 │    └── Ingestion Contrast & Lighting  
 │         ├── Metric: Local pixel intensity variance  
 │         └── Action: Contrast Enhancement Filter (Histogram Equalization)   
 ├── 2. Structural Integrity Inspections  
 │    ├── Spatial Context Limits Check  
 │    │    ├── Metric: Spatial Bounding Box Coverage Ratio  
 │    │    └── Diagnostic Check: Are visual markers, titles, or headers cropped out?  
 │    ├── Neighboring Page Context Verification  
 │    │    ├── Metric: Table / Paragraph Continuation Flag  
 │    │    └── Action: Retrieve adjacent page N-1 and N+1 images if multi-page continuation is detected   
 │    └── Caption-Figure Association Alignment  
 │         ├── Metric: Geometric distance to nearest label node  
 │         └── Action: Force parent-child spatial graph edge grouping in POTATR   
 └── 3. Decision Control Logic  
      ├── If Ingestion Quality is INSUFFICIENT -> Trigger Fallback OCR Engine (PaddleOCR local server)   
      ├── If Structural Context is INCOMPLETE -> Expand bounding box boundaries by 10% [9]  
      └── If Confidence is low -> Elevate Uncertainty and route to Human-in-the-Loop Verification 
```

To guide this automated verification process, the pipeline enforces a minimum evidence verification matrix across all task categories.

### **Artifact 13: Inspection Adequacy Framework**

```
Task-Specific Minimum Evidence Verification Matrix

 ├── Task Category: Contract & Legal Answers  
 │    ├── Minimum Evidence Units: Full-page image + Adjacent definition clauses + Footnotes [33]  
 │    ├── Bounding Coordination Target: Coordinate box of target paragraph + Bounding box of cross-references  
 │    └── Verification Metrics: Strict Reading-Order F1  + Paragraph Hierarchy Preservation 

 ├── Task Category: Invoices, Receipts & Financial Audit  
 │    ├── Minimum Evidence Units: Label-value coordinate pairs + Corporate identifier + Totals + Tax markers   
 │    ├── Bounding Coordination Target: Field labels [x1,y1,x2,y2] + Field values [x1,y1,x2,y2][9]  
 │    └── Verification Metrics: Exact match character OCR confidence > 0.95 + Grid validation (Columns match Rows)

 ├── Task Category: Tabular Quantitative Extraction  
 │    ├── Minimum Evidence Units: Cell value + Target column header + Target row header + Spanning cell context   
 │    ├── Bounding Coordination Target: Cell coordinate box + Column header box + Table caption coordinate box   
 │    └── Verification Metrics: GriTS Content score > 0.92  + Continuation Merge verified 

 ├── Task Category: Visual Diagram / Engineering Drawing Analysis  
 │    ├── Minimum Evidence Units: Focus component crop + High-resolution surrounding schematic context + Legends   
 │    ├── Bounding Coordination Target: Target visual object bounding box + Spatial annotation box [9]  
 │    └── Verification Metrics: IoU on localized components > 0.85 [9] + Resolution minimum of 300 DPI 

 ├── Task Category: Video Dialog & Sequence QA  
 │    ├── Minimum Evidence Units: Event keyframes + Timestamp sequences + Subtitle transcripts + Speaker identifiers   
 │    ├── Bounding Coordination Target: Timestamp range [t_start, t_end] + Frame-level bounding boxes [16, 52]  
 │    └── Verification Metrics: Frame Selection Sensitivity (FSS) > 0.40  + Temporal Order F1 
```

## **X. Evidence Provenance, Citation, and Production Observability**

A production-grade multimodal system has not understood an artifact unless it can show the exact visual and temporal coordinates that support its answer.2  
If a page image is rendered, segmented, cropped, embedded, and summarized, the system must preserve a continuous coordinate chain from the final text token back to the original source coordinates. Any loss of provenance during these transformations makes the output unauditable, preventing human-in-the-loop validation and automated regression testing.  
To ensure verifiability, the system requires a structured citation schema.

### **Artifact 14: Multimodal Evidence Citation Model**

```JSON  
{  
  "$schema": "https://ai-engineering.canon/schemas/multimodal-citation-v1.json",  
  "citation_id": "cite_2026_06_10_08912",  
  "source_document": {  
    "document_uuid": "doc_8f92a10c-3b9e-4a8f-b9c2-7d8e9f0a1b2c",  
    "document_hash": "sha256_e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",  
    "filename": "Q2_2026_financial_report.pdf"  
  },  
  "grounded_claims": [  
    {  
      "claim_id": "claim_1",  
      "text": "Operating margins for the Industrial Machinery segment rose to 14.2% in Q2 2026.",  
      "evidence_class": "structured_table_cell",  
      "confidence": 0.982,  
      "page_evidence": {  
        "page_number": 12,  
        "coordinate_system": {  
          "grid_width": 1000,  
          "grid_height": 1000,  
          "coordinate_standard": "normalized"  
        },  
        "target_regions": [  
          {  
            "region_id": "tbl_12_1",  
            "region_class": "table_block",  
            "bounding_box": ,  
            "context_markup": "<Table class='POTATR-v1.0' id='tbl_42_1'>"  
          },  
          {  
            "region_id": "cell_row_4_col_3",  
            "region_class": "table_cell",  
            "bounding_box": ,  
            "cell_value": "14.2%",  
            "cell_headers": ["Operating Margin", "Q2 2026", "Industrial Machinery"]  
          }  
        ]  
      }  
    },  
    {  
      "claim_id": "claim_2",  
      "text": "The increase was driven by the installation of the ACME-400 valve configuration.",  
      "evidence_class": "annotated_figure",  
      "confidence": 0.941,  
      "page_evidence": {  
        "page_number": 43,  
        "coordinate_system": {  
          "grid_width": 1000,  
          "grid_height": 1000,  
          "coordinate_standard": "normalized"  
        },  
        "target_regions": [  
          {  
            "region_id": "fig_43_2",  
            "region_class": "figure_image",  
            "bounding_box": ,  
            "context_markup": "<Figure class='olmOCR' id='fig_43_2'>"  
          },  
          {  
            "region_id": "component_annotation_valve",  
            "region_class": "figure_markup_annotation",  
            "bounding_box": ,  
            "annotated_value": "ACME-400 Valve Assembly"  
          }  
        ]  
      }  
    }  
  ]  
}
```

This structured citation format ensures that every quantitative or factual claim is auditable.  
If an operator hovers over a claim in the user interface, the system can draw the exact bounding boxes over the high-resolution source document page image.2  
Architectures deploy a structured Failure Mode Map to diagnose and mitigate issues across these processing stages.

### **Artifact 15: Multimodal Failure Mode Map**

| Failure Mode | Symptom | Root Cause | Detection Signal | Prevention & Architectural Mitigation |
| :---- | :---- | :---- | :---- | :---- |
| **OCR Drift** 10 | Downstream LLM misreads or hallucinates alphanumeric values (e.g., parsing "1" as "l" or "0" as "O").10 | Low rendering resolution (<150 DPI) or degraded scanned fonts.4 | OCR per-token confidence scores drop below 0.85.10 | Force 300 DPI rasterization + swap Tesseract default with a localized PaddleOCR/EasyOCR fallback server.2 |
| **Layout Collapse** 5 | Multi-column text blocks or marginalia are parsed out of order, interleaving unrelated sentences.29 | Heuristic-based geometric parsing lines cross column boundaries without structural awareness.5 | Downstream QA evaluations show low Reading-Order F1 scores (<0.70).36 | Replace sequential parsing chains with end-to-end compact VLMs like GraniteDocling-258M compiling to DocTags markup.5 |
| **Table Flattening** 4 | Cell contents are extracted as an unstructured text block; row-column headers are lost.4 | Naive text extraction tools ignore table lines and white-space alignment.4 | Standard RAG evaluations show low MRR on tabular queries.2 | Deploy POTATR parallel-decoded graph parser for full-page tables or TABLET split-merge sequence labeling for dense tables.12 |
| **Chart Scale Misread** 13 | Visual reasoning models misinterpret line trends or fail to read numeric values on logarithmic axes.13 | Generative models read the global visual trend ("gestalt") rather than parsing axes and legends.13 | High failure rates on numerical comparison checklists.14 | Run DePlot modality conversion to translate visual charts to linearized tables prior to downstream text reasoning.13 |
| **Form Field Misbinding** 4 | Field values are mapped to incorrect labels, or handwritten checks are missed.4 | Spatial layout heuristics fail under complex, multi-column administrative layouts.4 | Form validator detects empty required fields or confidence scores drop.10 | Implement a Form Understanding Model with explicit bounding box schema templates, mapping pixel-intensity thresholds for checkboxes.4 |
| **Temporal Reversal** 16 | Video dialog agents confuse the chronological sequence of events (e.g., misinterpreting what happened first).28 | Flat, temporally-agnostic uniform frame sampling misses critical state-transition boundaries.15 | Downstream video QA shows near-zero accuracy on ordering and duration tasks.28 | Implement Event-Anchored Frame Selection (EFS) using DINOv2 visual segment partitioning to detect boundaries.26 |
| **Wrong Crop Retrieval** 2 | Generator receives an image crop containing irrelevant information, leading to attention dilution.6 | Patch-to-region score propagation fails due to incorrect IoU-weighted overlap mapping.7 | Region retrieval precision matches random selection baseline (~6.7%).20 | Recalibrate Snappy visual-patch to OCR-coordinate grid bounds checking.7 |
| **Low-Resolution Crop** 2 | Model invents characters or numbers when processing small bounding boxes.11 | Vision encoders map small crops to a fixed token count, losing high-frequency features.11 | OCR character errors escalate on tiny text chunks.36 | Implement an Inspection Quality check to dynamically upscale crops to 300 DPI prior to tokenization.2 |
| **Visual Similarity Confusion** | System retrieves a visually similar document or image template from the wrong date or customer. | Embedding model focuses on layout structures rather than exact semantic differences in small text regions. | High false positive rates on cross-document comparative queries. | Deploy a hybrid search routing index that crosses late-interaction visual embeddings with BM25 keyword indices.2 |
| **Figure-Caption Separation** 2 | Visual figures are extracted without their caption metadata, losing component mapping references. | Page segmenters fail to construct directed associative edges between image blocks and text blocks. | Caption text recall F1 drops below standard baselines.38 | Deploy image-to-graph models like POTATR to explicitly predict directed parent-child edges between page objects.12 |
| **Missed Video Frame** 15 | Sampling strategy misses brief transitions or events (e.g., flash of a defect indicator). | Fixed-rate or scene-change sampling interval is too coarse to catch sub-second event boundaries.15 | Low downstream accuracy on action-duration or event-localization tasks.28 | Deploy Wavelet-based Frame Selection (WFS-SB) to resolve semantic changes at coarse and fine visual scales.48 |
| **Stale Media Ingestion** | Downstream agents base reasoning on outdated visual versions of document attachments or UI screens. | Pipeline fails to govern state updates, allowing historical visual embeddings to persist in active memory. | Mismatched metrics showing correct text retrieval but incorrect visual grounding coordinates. | Inherit state-governance protocols from AI-ENG-B to invalidate visual vectors when source files undergo revisions. |
| **Source Provenance Loss** 10 | Generated claims cite document titles but fail to point to the exact page, crop, or cell coordinates.7 | Coordinate mappings are discarded during structural transformations (PDF -> OCR -> Embeddings). | Audit trace completeness score drops to zero, and claims become untrustworthy. | Enforce compliance-grade structured extraction (e.g., using LlamaParse or AWS Textract) to preserve coordinate chains.10 |
| **Evidence Over-Selection** | Generator context window is flooded with redundant visual tokens, causing performance degradation. | Multi-vector visual index returns excessive page-patches instead of localized, ranked crops.6 | Ingestion latency, API costs, and context dilution errors increase proportionally.6 | Deploy Snappy spatially-grounded region filters to return targeted crop bounding boxes instead of full-page images.7 |
| **Evidence Under-Selection** | System synthesizes an answer based on partial visual data, missing neighboring clauses or footnote units.12 | Retrieval range is bounded too narrowly, cutting off contiguous tables or layout blocks.21 | Downstream answer correctness drops on multi-page reasoning tasks.39 | Implement an Inspection Adequacy check to retrieve neighboring page context if continuation markers are flagged.39 |
| **Unsupported Visual Inference** | Generator confidently asserts a claim based on a visual image without verified spatial grounding. | Model synthesizes answers from general scene impressions without locating supporting coordinates.8 | High rate of "plausible but ungrounded" answers during manual expert review. | Implement a verification gate from AI-ENG-O to block user-facing status updates if grounding ratio metrics drop below 0.90. |

To maintain reliability in production, architectures must establish continuous evaluation pipelines. These pipelines track metrics across each visual, spatial, and temporal processing stage.

### **Artifact 16: Multimodal Evaluation and Observability Guidance**

```
Multimodal Evaluation Matrix  
 ├── 1. OCR & Structural Layout Benchmarks  
 │    ├── Character Extraction Accuracy  
 │    │    ├── Metric: Character Error Rate (CER) / Word Error Rate (WER)  
 │    │    └── Evaluation Benchmark: olmOCR-Bench (ArXiv Math, Old scans Math, H&F, Multi-Col)   
 │    └── Reading-Order Tracking Correctness  
 │         ├── Metric: Reading-Order F1 Score / Layout IoU Match [43]  
 │         └── Evaluation Benchmark: OmniDocBench / LightOnOCR-bbox-bench [43, 53]  
 ├── 2. Tabular Structure & Content Metrics  
 │    ├── Cell-Structure Alignment Check  
 │    │    ├── Metric: GriTS-Top (structural adjacency similarity matrix match)   
 │    │    └── Evaluation Benchmark: PubTables-v2 Single-Page TSR [38, 39]  
 │    └── Content-Preserving Table Match  
 │         ├── Metric: GriTS-Con (content-preserving cell evaluation)  and TEDS / TEDS-Struc   
 │         └── Evaluation Benchmark: FinTabNet / PubTabNet / PubTables-v2 Full Documents [24, 39]  
 ├── 3. Chart Extraction Precision  
 │    ├── Chart Plot-to-Table Match  
 │    │    ├── Metric: Relative Error Tolerance Table Match Score   
 │    │    └── Evaluation Benchmark: ChartQA Benchmark (Human-written queries subset)   
 │    └── Visual Attribute & Metadata Extraction  
 │         ├── Metric: Category Classification Accuracy (Type, Color, Legends, Axis)   
 │         └── Evaluation Benchmark: Checklist v2 (Visual Structure & Summary Stats tasks)   
 ├── 4. Visual Grounding & Region Retrieval  
 │    ├── Spatial Localization Precision  
 │    │    ├── Metric: Mean IoU % and Good Ratio (IoU@0.5, IoU@0.25) [9, 20]  
 │    │    └── Evaluation Benchmark: BBox-DocVQA Dataset [8, 9, 20]  
 │    └── Context Volume Optimization  
 │         ├── Metric: Context Reduction Factor (CRF) / Token Savings %   
 │         └── Evaluation Benchmark: Snappy Region-Reranking validation   
 └── 5. Video Temporal Grounding  
      ├── Frame-Selection Sensitivity  
      │    ├── Metric: Frame Selection Sensitivity (FSS) delta   
      │    └── Evaluation Benchmark: TempCore grounded evaluation subsets   
      └── Temporal-Event Localization Accuracy  
           ├── Metric: Temporal Intersection-over-Union (t-IoU) [52]  
           └── Evaluation Benchmark: VideoITG-40K / CG-Bench / Video-MME / LongVideoBench [15, 16, 27]
```

## **XI. Architectural Handoffs and Degraded-Mode Fallbacks**

Multimodal understanding is the substrate layer that feeds downstream agent loops, interface action systems, and compliance auditing engines.2  
If the system cannot verify what visual evidence it inspected, it cannot be trusted to perform computer-use actions, click buttons on interfaces, generate financial extractions, or sign off on legal contracts.2

### **Artifact 17: Cross-Canon Handoff Map**

```
Substrate Layer: AI-ENG-P (Multimodal Understanding)  
 │  
 ├── Downstream Volume 6 Handoffs (Interface Control & Action)  
 │    ├── Screen Understanding & Computer-Use Agents  
 │    │    ├── Dependency: Bounding box spatial coordinate maps + Object localization   
 │    │    └── Operational Rule: If coordinate confidence < 0.90, block click-action and trigger re-inspection  
 │    ├── Browser & GUI Automation  
 │    │    ├── Dependency: Webpage layout node serialization (DocTags/HTML)   
 │    │    └── Operational Rule: Suppress running headers/footers to prevent agent navigation loops   
 │    └── Accessibility-Mediated Interactivity  
 │         ├── Dependency: Structured reading-order reconstruction   
 │         └── Operational Rule: Force LaTeX output format for mathematical expressions to support screen-readers   
 │  
 ├── Downstream Volume 7-9 Handoffs (System Security, Fallbacks & Observability)  
 │    ├── AI-ENG-S (Production Pathologies)  
 │    │    ├── Dependency: Ingestion throughput telemetry + Memory metrics [1, 18]  
 │    │    └── Operational Rule: Log rendering latency and token consumption to balance cross-encoder costs   
 │    ├── AI-ENG-T (Prompt Injection through Documents/Images)  
 │    │    ├── Dependency: Segregated OCR / Text index parsing boundaries  
 │    │    └── Operational Rule: Sanitize visual text streams before routing to downstream generation agents  
 │    ├── AI-ENG-U (Parser and Tool Dependency Risk)  
 │    │    ├── Dependency: Local fallback libraries and sandbox execution parameters   
 │    │    └── Operational Rule: Monitor and isolate third-party OCR environments to prevent untyped memory leaks   
 │    ├── AI-ENG-W (Fallback and Degraded Modes)  
 │    │    ├── Dependency: Progressive quality evaluation degradation metrics   
 │    │    └── Operational Rule: Trigger local, CPU-bound Tesseract parse if VLM API rate limits are hit   
 │    ├── AI-ENG-X (Transparent User-Facing Evidence)  
 │    │    ├── Dependency: Fine-grained spatial citation mapping metadata  
 │    │    └── Operational Rule: Render visual bounding box highlights directly on the user interface  
 │    ├── AI-ENG-Y (Human Review)  
 │    │    ├── Dependency: Coordinate confidence threshold registry   
 │    │    └── Operational Rule: Route character OCR inputs below 0.60 directly to human adjudication   
 │    ├── AI-ENG-Z (Telemetry)  
 │    │    ├── Dependency: Query-level latency trace structures   
 │    │    └── Operational Rule: Emit visual-retrieval latency scores separately from generation latency metrics   
 │    ├── AI-ENG-AA (Multimodal Evaluations)  
 │    │    ├── Dependency: Ground-truth bounding box registries [9, 20]  
 │    │    └── Operational Rule: Evaluate system alignment against BBox-DocVQA spatial coordinates [9, 20]  
 │    ├── AI-ENG-AB (Audit and Replay)  
 │    │    ├── Dependency: Multi-vector database index traces and coordinate histories [7, 18]  
 │    │    └── Operational Rule: Store processed page images with their corresponding patch score maps   
 │    ├── AI-ENG-AC (Incident Response)  
 │    │    ├── Dependency: Fault detection indicators in layout-aware parsers   
 │    │    └── Operational Rule: Quarantine documents that trigger parsing loop crashes or coordinate inversions [33]  
 │    └── AI-ENG-AJ (Reference Architectures)  
 │         ├── Dependency: Modular, multi-index retrieval blueprints   
 │         └── Operational Rule: Provide structured templates for unifying late-interaction indices and BM25 databases   
 └──
```

To support these handoffs in production, the architecture must implement robust fallbacks for degraded operating environments:

* **DPI Degradation Fallback:** If a client uploads a document that fails the 150 DPI minimum quality check (e.g., a low-resolution scan or mobile camera capture), the ingestion pipeline must run image preprocessing.2 It applies a deskewing rotation filter, runs contrast enhancement, and upscales the resolution to 300 DPI before passing the image to the layout models.2 If character confidence remains low (<0.60), the page is routed to a specialized OCR engine (e.g., PaddleOCR local server) or escalated to human review.10  
* **Layout Model Fallbacks:** When deploying in resource-constrained environments (e.g., edge compute nodes, secure air-gapped clusters) where large VLMs are unavailable, the parser falls back to a dual-mode local processing pipeline.10 It uses a local CPU-bound Python library (e.g., pdfplumber or PyMuPDF with Tesseract.js default) for digital-native pages.10 If a scanned page is encountered, it routes the image to a local, compact OCR server, prioritizing throughput over peak VLM accuracy.10  
* **Video Frame Reduction Fallback:** When a video sequence exceeds standard attention token budgets (e.g., processing multi-hour feeds through fixed context windows), the pipeline automatically steps down from dense, instruction-conditioned sampling to hierarchical segment tree retrieval.15 It uses a low-frequency, visual-diversity pass (DINOv2 temporal partitioning) to construct a global segment representation.26 It then expands frame-level resolution only within localized window segments triggered by specific user queries.15

## **XII. Durable Principles of Multimodal Understanding**

To guide high-dimensional AI system deployment, architects must adhere to five core principles of multimodal evidence design:

1. **Evidence Prior to Answer Generation:** A multimodal system must select, parse, and verify its visual, spatial, and temporal evidence before generating any textual output.6  
2. **Structural Preservation is Semantic Accuracy:** The spatial arrangement of document elements, table grids, chart axes, and video sequences is semantic content.1 Flattening this structure into raw text streams introduces errors and increases hallucination risks.5  
3. **Plausible Text is Not Visual Grounding:** A system has not understood an artifact because its visual tokens entered the context window.6 Understanding is demonstrated only when the system can map its claims back to precise, coordinate-level visual or temporal boundaries.7  
4. **Resolution and Modality Match the Task:** The architecture must enforce quality checking at ingestion, verifying rendering DPI, contrast, and layout continuity to match task-specific demands.2  
5. **Continuous Provenance is Required:** The spatial-temporal coordinate trail must survive all processing stages.2 Visual and temporal evidence coordinates must be mapped continuously from ingestion to the generated citation payload, ensuring human-auditable verifiability.2

#### **Works cited**

1. Visual RAG Toolkit: Scaling Multi-Vector Visual Retrieval with Training-Free Pooling and Multi-Stage Search - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2602.12510v1](https://arxiv.org/html/2602.12510v1)  
2. Multimodal RAG: Retrieving from Images, PDFs, and Tables | Tensoria, accessed June 10, 2026, [https://tensoria.fr/en/blog/multimodal-rag-images-pdfs-tables](https://tensoria.fr/en/blog/multimodal-rag-images-pdfs-tables)  
3. From Raw PDF to Qdrant Search Engine: Choosing the Right Document Parser for Your RAG Pipeline | by M K Pavan Kumar - Towards AI, accessed June 10, 2026, [https://pub.towardsai.net/from-raw-pdf-to-qdrant-search-engine-choosing-the-right-document-parser-for-your-rag-pipeline-5933bbbc7213](https://pub.towardsai.net/from-raw-pdf-to-qdrant-search-engine-choosing-the-right-document-parser-for-your-rag-pipeline-5933bbbc7213)  
4. What is Table Extraction OCR? - LlamaIndex, accessed June 10, 2026, [https://www.llamaindex.ai/glossary/table-extraction-ocr](https://www.llamaindex.ai/glossary/table-extraction-ocr)  
5. Revolutionizing Document Intelligence: The Strategic Advantage of IBM Granite-Docling VLM | by Mustafa Genc | Medium, accessed June 10, 2026, [https://medium.com/@mustafa.gencc94/revolutionizing-document-intelligence-the-strategic-advantage-of-ibm-granite-docling-vlm-71f8bf1357e4](https://medium.com/@mustafa.gencc94/revolutionizing-document-intelligence-the-strategic-advantage-of-ibm-granite-docling-vlm-71f8bf1357e4)  
6. Spatially-Grounded Document Retrieval via Patch-to-Region Relevance Propagation - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2512.02660v2](https://arxiv.org/html/2512.02660v2)  
7. [Literature Review] Spatially-Grounded Document Retrieval via Patch-to-Region Relevance Propagation, accessed June 10, 2026, [https://www.themoonlight.io/en/review/spatially-grounded-document-retrieval-via-patch-to-region-relevance-propagation](https://www.themoonlight.io/en/review/spatially-grounded-document-retrieval-via-patch-to-region-relevance-propagation)  
8. BBox-DocVQA: A Large-Scale Bounding-Box–Grounded Dataset for Enhancing Reasoning in Document Visual Question Answer - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2511.15090v1](https://arxiv.org/html/2511.15090v1)  
9. BBox DocVQA: Spatially Grounded DocVQA - Emergent Mind, accessed June 10, 2026, [https://www.emergentmind.com/topics/bbox-docvqa](https://www.emergentmind.com/topics/bbox-docvqa)  
10. The PDF Parser That Refuses to Be Helpful — And Why That's the Point - Towards AI, accessed June 10, 2026, [https://pub.towardsai.net/the-pdf-parser-that-refuses-to-be-helpful-and-why-thats-the-point-82c21508f214](https://pub.towardsai.net/the-pdf-parser-that-refuses-to-be-helpful-and-why-thats-the-point-82c21508f214)  
11. LensVLM: Selective Context Expansion for Compressed Visual Representation of Text, accessed June 10, 2026, [https://arxiv.org/html/2605.07019v1](https://arxiv.org/html/2605.07019v1)  
12. POTATR: A Lightweight Image-to-Graph Model for Page-Level Table Extraction - arXiv, accessed June 10, 2026, [https://arxiv.org/pdf/2606.09788](https://arxiv.org/pdf/2606.09788)  
13. [2212.10505] DePlot: One-shot visual language reasoning by plot-to-table translation - ar5iv, accessed June 10, 2026, [https://ar5iv.labs.arxiv.org/html/2212.10505](https://ar5iv.labs.arxiv.org/html/2212.10505)  
14. Enhancing Question Answering on Charts Through Effective Pre-training Tasks - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2406.10085v1](https://arxiv.org/html/2406.10085v1)  
15. VideoITG: Multimodal Video Understanding with Instructed Temporal Grounding - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2507.13353v2](https://arxiv.org/html/2507.13353v2)  
16. VideoITG: Multimodal Video Understanding with Instructed Temporal Grounding - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2507.13353v1](https://arxiv.org/html/2507.13353v1)  
17. ColPali: Efficient Document Retrieval with Vision Language Models - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2407.01449v2](https://arxiv.org/html/2407.01449v2)  
18. ColPali and Multimodal Document RAG on GPU Cloud: Visual PDF Retrieval Without OCR (2026) | Spheron Blog, accessed June 10, 2026, [https://www.spheron.network/blog/colpali-multimodal-document-rag-gpu-cloud/](https://www.spheron.network/blog/colpali-multimodal-document-rag-gpu-cloud/)  
19. ColPali for RAG (2025): Theory, implementation, and production tips | by Issa Hammoud, accessed June 10, 2026, [https://medium.com/@intuitivedl/rag-with-colpali-everything-you-need-to-know-46b7bd50901b](https://medium.com/@intuitivedl/rag-with-colpali-everything-you-need-to-know-46b7bd50901b)  
20. [2512.02660] Spatially-Grounded Document Retrieval via Patch-to-Region Relevance Propagation - arXiv, accessed June 10, 2026, [https://arxiv.org/abs/2512.02660](https://arxiv.org/abs/2512.02660)  
21. PubTables-v2: A new large-scale dataset for full-page and multi-page table extraction - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2512.10888v1](https://arxiv.org/html/2512.10888v1)  
22. Introducing PubTables-v2: Kensho's New Dataset Empowering Next-Gen AI for Table Extraction, accessed June 10, 2026, [https://kensho.com/news/introducing-pubtables-v2-kenshos-new-dataset-empowering-next-gen-ai-for-table-extraction](https://kensho.com/news/introducing-pubtables-v2-kenshos-new-dataset-empowering-next-gen-ai-for-table-extraction)  
23. TABLET: Table Structure Recognition using Encoder-only Transformers - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2506.07015v1](https://arxiv.org/html/2506.07015v1)  
24. [Literature Review] TABLET: Table Structure Recognition using Encoder-only Transformers, accessed June 10, 2026, [https://www.themoonlight.io/en/review/tablet-table-structure-recognition-using-encoder-only-transformers](https://www.themoonlight.io/en/review/tablet-table-structure-recognition-using-encoder-only-transformers)  
25. DEPLOT: One-shot visual language reasoning by plot-to-table translation - ACL Anthology, accessed June 10, 2026, [https://aclanthology.org/2023.findings-acl.660.pdf](https://aclanthology.org/2023.findings-acl.660.pdf)  
26. Event-Anchored Frame Selection for Effective Long-Video Understanding - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2603.00983v1](https://arxiv.org/html/2603.00983v1)  
27. [2603.00983] Event-Anchored Frame Selection for Effective Long-Video Understanding - arXiv, accessed June 10, 2026, [https://arxiv.org/abs/2603.00983](https://arxiv.org/abs/2603.00983)  
28. TempCore: Are Video QA Benchmarks Temporally Grounded? A Frame Selection Sensitivity Analysis and Benchmark - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2509.01167v2](https://arxiv.org/html/2509.01167v2)  
29. Best PDF Parsers for AI and RAG Workflows in 2026 - Firecrawl, accessed June 10, 2026, [https://www.firecrawl.dev/blog/best-pdf-parsers](https://www.firecrawl.dev/blog/best-pdf-parsers)  
30. Optimal RAG Chunking : 8 Strategies & Real Benchmarks - Anas Rabhi, accessed June 10, 2026, [https://ianas.fr/en/blog/2026/04/15/chunking-optimal-rag/](https://ianas.fr/en/blog/2026/04/15/chunking-optimal-rag/)  
31. Python PDF text extraction with PyMuPDF: Scanned PDFs, tables, and OCR - Nutrient iOS, accessed June 10, 2026, [https://www.nutrient.io/blog/extract-text-from-pdf-pymupdf/](https://www.nutrient.io/blog/extract-text-from-pdf-pymupdf/)  
32. olmOCR – Open-Source OCR for Accurate Document Conversion - Allen Institute for Artificial Intelligence, accessed June 10, 2026, [https://olmocr.allenai.org/](https://olmocr.allenai.org/)  
33. docling-project/granite-docling-2stage-258m - Hugging Face, accessed June 10, 2026, [https://huggingface.co/docling-project/granite-docling-2stage-258m](https://huggingface.co/docling-project/granite-docling-2stage-258m)  
34. [2603.13032] Multimodal OCR: Parse Anything from Documents - arXiv, accessed June 10, 2026, [https://arxiv.org/abs/2603.13032](https://arxiv.org/abs/2603.13032)  
35. allenai/olmocr: Toolkit for linearizing PDFs for LLM datasets/training - GitHub, accessed June 10, 2026, [https://github.com/allenai/olmocr](https://github.com/allenai/olmocr)  
36. Unsiloed Achieves #1 Rank on olmOCR-Bench, accessed June 10, 2026, [https://www.unsiloed.ai/blog/unsiloed-ai-achieves-1-rank-on-olmocr-bench-2](https://www.unsiloed.ai/blog/unsiloed-ai-achieves-1-rank-on-olmocr-bench-2)  
37. AI OCR & Document Processing Tools Comparison - Artificial Analysis, accessed June 10, 2026, [https://artificialanalysis.ai/agents/ocr](https://artificialanalysis.ai/agents/ocr)  
38. POTATR: A Lightweight Image-to-Graph Model for Page-Level Table Extraction - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2606.09788](https://arxiv.org/html/2606.09788)  
39. PubTables-v2: A new large-scale dataset for full-page and multi-page table extraction - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2512.10888v2](https://arxiv.org/html/2512.10888v2)  
40. Daily Papers - Hugging Face, accessed June 10, 2026, [https://huggingface.co/papers?q=Encoder-only](https://huggingface.co/papers?q=Encoder-only)  
41. A Comparison of Image-based, Text-based and Multimodal Models in the Table Structure Recognition Task - OpenReview, accessed June 10, 2026, [https://openreview.net/pdf/a586861b6d8335e7c04ff733d5d53c3c34384629.pdf](https://openreview.net/pdf/a586861b6d8335e7c04ff733d5d53c3c34384629.pdf)  
42. Table Transformer (TATR) - Table Structure Recognition - AIKosh, accessed June 10, 2026, [https://aikosh.indiaai.gov.in/home/models/details/table_transformer_tatr_table_structure_recognition.html](https://aikosh.indiaai.gov.in/home/models/details/table_transformer_tatr_table_structure_recognition.html)  
43. LightOnOCR-bbox-bench: Document Image Localization - Emergent Mind, accessed June 10, 2026, [https://www.emergentmind.com/topics/lightonocr-bbox-bench](https://www.emergentmind.com/topics/lightonocr-bbox-bench)  
44. PubTables-v2: A new large-scale dataset for full-page and multi-page table extraction - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2512.10888v3](https://arxiv.org/html/2512.10888v3)  
45. We improved table extraction in DocParse with a new AI model - Aryn, accessed June 10, 2026, [https://www.aryn.ai/post/we-improved-table-extraction-in-docparse-with-a-new-ai-model](https://www.aryn.ai/post/we-improved-table-extraction-in-docparse-with-a-new-ai-model)  
46. GenPlot: Increasing the Scale and Diversity of Chart Derendering Data - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2306.11699](https://arxiv.org/html/2306.11699)  
47. Spatially-Grounded Document Retrieval via Patch-to-Region Relevance Propagation - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2512.02660v1](https://arxiv.org/html/2512.02660v1)  
48. Wang Chen's research works | Fuzhou University and other places - ResearchGate, accessed June 10, 2026, [https://www.researchgate.net/scientific-contributions/Wang-Chen-2264289193](https://www.researchgate.net/scientific-contributions/Wang-Chen-2264289193)  
49. Luojun Lin - CatalyzeX, accessed June 10, 2026, [https://www.catalyzex.com/author/Luojun%20Lin](https://www.catalyzex.com/author/Luojun%20Lin)  
50. Late interaction models: How to scale & optimize in Elasticsearch, accessed June 10, 2026, [https://www.elastic.co/search-labs/blog/late-interaction-model-colpali-scale](https://www.elastic.co/search-labs/blog/late-interaction-model-colpali-scale)  
51. Grounding is All You Need? Dual Temporal Grounding for Video Dialog - arXiv, accessed June 10, 2026, [https://arxiv.org/html/2410.05767v2](https://arxiv.org/html/2410.05767v2)

---

[← Back to Canon Map](../canon-map.md)