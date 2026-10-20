# Research Plan

## Brief

Topic: Competitor and prior-art scan for decks, a new slide editor (PowerPoint-like canvas for people, typed JSON document and MCP tools for agents), to answer the eight open questions at the end of docs/spec-v0.md.
Scope: Products and libraries as they stand in 2026-10: tldraw, Gamma, Pitch, Slidev, Typst with Touying, reveal.js, Keynote, Figma (comments), Google Slides API, OOXML masters and layouts, Blattomat (~/Code/blattwerk), the presentations plugin (~/Code/pgoell-claude-tools/plugins/presentations), plus pptx export libraries, chart libraries and CRDTs (Yjs, Automerge) as the questions need. Every LICENSE file read directly.
Audience: Pascal plus the agents that turn the settled spec into issues; every claim names its source.
Purpose: Answer the eight open questions in docs/spec-v0.md so the spec can be settled and turned into issues.
Deliverable: Full report: product deep dives, one answer per open question with a recommendation, a license table. About 8,000+ words, English.
Deadline: none

## Clusters

### Cluster: canvas-rendering

Title: Canvas technology: HTML elements versus a canvas library such as tldraw (open question 1)
Sub-questions:

- SQ1: What is tldraw's license today? Read the LICENSE file in github.com/tldraw/tldraw (repo root and packages/tldraw) directly; what does it allow without a paid license key (watermark, production use, commercial use, self-hosting), and what does a business license cost?
  - Search angles: raw LICENSE.md on GitHub, tldraw.dev pricing and license pages, tldraw license change history and community reaction
  - Source types: primary repo files, vendor pages, practitioner discussion
- SQ2: How does tldraw render and model shapes (DOM/SVG vs canvas, its store and records, custom shapes, rich text, export to SVG/PNG), and has anyone built slides or presentations on it (tldraw slides example, "tldraw computer", Make Real)? What would it take to fit a fixed 1920x1080 slide model with masters?
  - Search angles: tldraw docs (shapes, store, editor API), tldraw slides example in repo, blog posts building presentation tools on tldraw
  - Source types: primary docs and code, practitioner blogs
- SQ3: How do browser slide editors draw slides: Pitch, Google Slides (canvas/SVG), Canva, Figma Slides, PowerPoint for the web? What do engineering blogs say about the trade-offs of DOM elements vs HTML canvas vs SVG for an editor whose output is also the presentation and the PDF?
  - Search angles: Pitch engineering blog, Google Slides rendering canvas switch, Figma Slides architecture, "DOM vs canvas editor" trade-offs
  - Source types: engineering blogs, journalism, practitioner
- SQ4: What are the licenses (read LICENSE files directly) and fitness of the DOM manipulation libraries the spec names, moveable and selecto (daybrush), and alternatives (react-moveable, interact.js, Konva, Fabric.js)? Are they maintained?
  - Search angles: GitHub LICENSE files, last release dates, issue trackers
  - Source types: primary repo files

### Cluster: masters-layouts

Title: Masters, layouts and placeholders in PowerPoint, Google Slides and Pitch (open question 2)
Sub-questions:

- SQ1: How does OOXML (ECMA-376 PresentationML) model slide masters, slide layouts, placeholders (p:ph type and idx), inheritance of position, size and text styles (bodyStyle, titleStyle, lstStyle levels), themes (a:theme colour scheme, font scheme), and what must a pptx contain for PowerPoint to show real layouts and let users switch layout?
  - Search angles: ECMA-376 spec part 1, Microsoft Open XML SDK docs on slide layouts and placeholders, python-pptx docs on placeholders and analysis pages
  - Source types: standards documents, vendor docs
- SQ2: How does the Google Slides API model masters, layouts and placeholders (Page types MASTER, LAYOUT, NOTES; placeholder type, index, parentObjectId; layoutReference in createSlide; placeholderIdMappings; inheritance of properties; theme colours)?
  - Search angles: developers.google.com/slides reference and guides, Pages concept page, createSlide request
  - Source types: vendor docs
- SQ3: How do Pitch, Keynote, Gamma and Figma Slides handle templates, layouts and masters, and what happens when the user changes the master or template after the slides exist? How do they export layouts to pptx?
  - Search angles: Pitch help center templates/brand kits, Pitch pptx export quality, Figma Slides templates, Gamma themes
  - Source types: vendor help docs, practitioner reviews
- SQ4: What minimum must a JSON deck model hold (layout id, placeholder type/idx per block, inherited vs overridden props, theme colour and font references) for a pptx export to produce slides attached to real layouts with working placeholders? Evidence from exporters (PptxGenJS defineSlideMaster and placeholders, python-pptx) and from people who tried it.
  - Search angles: PptxGenJS masters docs and issues on placeholders, python-pptx placeholder behaviour, pptx exporters' known gaps
  - Source types: primary docs, issue trackers, practitioner

### Cluster: text-forms

Title: Text forms for agents: Slidev, Typst with Touying, Marp (open question 3)
Sub-questions:

- SQ1: How does Slidev let a person or agent write slides (Markdown with frontmatter, layouts, components, slots, themes, click animations, presenter mode, exports to PDF/PPTX/PNG) and how good is its pptx export (images or editable)? License (read LICENSE file).
  - Search angles: sli.dev docs (syntax, layouts, exporting), Slidev GitHub LICENSE, Slidev pptx export editable
  - Source types: primary docs and repo
- SQ2: How does Typst with Touying model slides (themes, slide functions, layouts, config, speaker notes via pdfpc, animations), how do agents fare writing Typst, and how does export work (PDF only? pptx?)? Licenses of Typst compiler and Touying (read LICENSE files).
  - Search angles: touying-typ.github.io docs, typst/typst LICENSE, touying LICENSE, LLM writing typst experiences
  - Source types: primary docs and repos, practitioner
- SQ3: What do agents and LLM tools do better with text forms (Markdown, Typst, HTML) than with JSON trees, and what is known about round-tripping a text form with a positioned/canvas model (Slidev's visual editing, Marp, reveal.js, Quarto revealjs, Spectacle)? Any prior art for a deck that has both a JSON source of truth and a text projection?
  - Search angles: LLM structured output JSON vs markdown token efficiency, Slidev drag-and-drop v-drag feature, two-way sync markdown visual editor
  - Source types: practitioner, independent studies, primary docs
- SQ4: How do existing agent/MCP slide tools expose decks (Slidev MCP, Marp MCP, Google Slides MCP servers, Gamma API, Microsoft PowerPoint Copilot agent APIs, Anthropic pptx skill): JSON ops, text, or HTML?
  - Search angles: MCP server registries for slides/presentations, Gamma generate API docs, "slides MCP server"
  - Source types: primary repos and docs

### Cluster: ai-editing

Title: How Gamma, Pitch and others let an AI change a deck (open question 4)
Sub-questions:

- SQ1: How does Gamma's AI edit: whole-deck generation, per-card edits, "edit with AI" on a selection, theme changes, and its card model (flowing cards vs fixed slides)? What happens to content the user edited by hand? Gamma API (generate endpoint) shape.
  - Search angles: Gamma help center AI editing, Gamma API docs, reviews of Gamma editing control
  - Source types: vendor docs, practitioner reviews
- SQ2: How does Pitch's AI work (AI presentation generator, rewrite text, AI on single slides or blocks) and how does it treat hand-placed blocks and the template?
  - Search angles: Pitch AI help center, Pitch blog AI launch, reviews
  - Source types: vendor docs, practitioner reviews
- SQ3: How do Microsoft Copilot in PowerPoint, Google Gemini in Slides, Canva Magic Design, Beautiful.ai, Tome (and its pivot), Figma Slides AI and Plus AI scope AI edits (deck, slide, element), and how do they keep or overwrite manual layout? Any notion of locking or protected blocks?
  - Search angles: Copilot PowerPoint edit slide features, Gemini Slides, Beautiful.ai smart slides constraint layout, Tome shutdown
  - Source types: vendor docs, journalism, practitioner
- SQ4: What complaints and failure modes do users report about AI deck editing (lost manual edits, layout drift, regeneration wiping changes), and what design patterns (diff/preview, accept/reject, per-element locks, versioning) are recommended?
  - Search angles: reddit/HN complaints about Gamma/Copilot overwriting, UX writing on AI edit review patterns
  - Source types: practitioner, journalism

### Cluster: export-chart-libs

Title: pptx export and chart libraries and their licenses (open question 5)
Sub-questions:

- SQ1: Which pptx writing libraries fit a TypeScript/Python stack (PptxGenJS, python-pptx, officegen, docx/pptx-automizer, Aspose and Syncfusion as paid comparisons)? For each: license read from the LICENSE file, maintenance, support for masters/layouts with placeholders, native charts, tables, rich text runs, speaker notes, images, SVG.
  - Search angles: GitHub LICENSE files, release history, docs on masters and charts
  - Source types: primary repos and docs
- SQ2: Which chart libraries fit a block that must render in the browser, print to PDF via Chromium, and export to pptx (ideally as native editable PowerPoint charts): ECharts, Vega-Lite, Chart.js, Observable Plot, Recharts, D3? Licenses from LICENSE files; SVG output support; a declarative spec that maps to PptxGenJS/python-pptx chart data.
  - Search angles: LICENSE files, SSR/SVG rendering docs, pptx native chart from chart spec
  - Source types: primary repos and docs
- SQ3: How do others export HTML or JSON slides to pptx (Slidev, Marp's editable pptx via LibreOffice, Gamma, Pitch, html2pptx approaches) and what fidelity do they reach (editable text, fonts, native charts)?
  - Search angles: Marp pptx editable experimental, Gamma pptx export quality, html to pptx converters
  - Source types: primary docs, practitioner reviews
- SQ4: How do pptx exports handle fonts (embedding, theme fonts, fallbacks) and SVG images (PowerPoint SVG support with PNG fallback)?
  - Search angles: PptxGenJS SVG images, PowerPoint svgBlip fallback, font embedding pptx libraries
  - Source types: primary docs, issue trackers

### Cluster: collab-crdt

Title: What live collaboration needs from the model now (open question 8)
Sub-questions:

- SQ1: How do Yjs and Automerge (and Loro as an alternative) model JSON documents (maps, arrays/lists, text), what are their licenses (read LICENSE files), and what are the known costs (doc size, history, performance with many small objects)?
  - Search angles: Yjs docs and LICENSE, Automerge 2/3 docs and LICENSE, Loro docs and LICENSE, CRDT benchmarks
  - Source types: primary repos, independent benchmarks
- SQ2: What model choices make later CRDT adoption easy or hard: stable ids, maps keyed by id vs ordered arrays, fractional indexing for order (z and slide order), last-writer-wins per property, rich text as a CRDT type (y-prosemirror), avoiding derived data in the document?
  - Search angles: Figma multiplayer blog, Linear sync engine, fractional indexing, tldraw sync store records, Excalidraw collaboration reconciliation
  - Source types: engineering blogs, primary docs
- SQ3: How do tldraw (tldraw sync), Excalidraw, Figma, Pitch and Google Slides do multiplayer, and what does each require of the document model? Is a server-authoritative operation log (like Blattomat's, if any) a viable path without a CRDT?
  - Search angles: tldraw sync docs, Excalidraw P2P collab blog, Figma "how multiplayer works", Pitch collaboration engineering
  - Source types: engineering blogs, primary docs
- SQ4: How do agent edits (MCP tool calls, branches) fit with CRDTs or op logs: patches by id, versioned writes, optimistic concurrency, Automerge branches/forks?
  - Search angles: Automerge branching, Ink & Switch Patchwork, version control for CRDT docs, JSON Patch with ids
  - Source types: independent research (Ink & Switch), primary docs

### Cluster: animations

Title: How slide tools model animations and builds, and what survives pptx (open question 6)
Sub-questions:

- SQ1: How does PresentationML model animations and transitions: p:timing (p:tnLst, p:par, p:seq mainSeq, clickEffect / withEffect / afterEffect, presetClass entr/emph/exit/path, presetID and presetSubtype, p:bldLst and p:bldP build by paragraph, p:animMotion path syntax), p:transition and the p14/p15 extensions (morph)? Which presets map to fade, fly in, zoom, wipe, pulse, grow, colour change?
  - Search angles: ECMA-376 part 1 animation chapter, Microsoft Open XML SDK animation docs, blog posts reverse-engineering p:timing XML, python-pptx animation support issues
  - Source types: standards documents, vendor docs, practitioner
- SQ2: How do Keynote (builds in/out, action builds, magic move), Google Slides (animations panel, API support or lack of it for animations), reveal.js (fragments, auto-animate, data-fragment-index) and Slidev (v-click, v-after, v-motion, click counters) model animations and steps?
  - Search angles: Apple Keynote user guide builds, Google Slides API animation support, revealjs.com fragments and auto-animate, sli.dev animations guide
  - Source types: vendor docs, primary docs
- SQ3: Which effects survive the round trip to pptx and back: Keynote export to pptx, Google Slides import/export of pptx animations, PptxGenJS and python-pptx animation support (or writing raw p:timing XML), LibreOffice Impress handling? Which effects are lost?
  - Search angles: Keynote export PowerPoint animations lost, Google Slides import pptx animations, PptxGenJS animation issue, Aspose animation docs
  - Source types: vendor docs, issue trackers, practitioner
- SQ4: How do web runtimes play steps (Web Animations API, reveal.js fragment events, Slidev click state), how do they handle going backwards, and how do HTML exports of decks animate SVG internals by selector?
  - Search angles: MDN Web Animations API, reveal.js fragmentshown events, Slidev click navigation backwards, SVG part animation in decks
  - Source types: primary docs

### Cluster: comments-pointing

Title: Comments anchored to objects, and pointing an AI at one element (open question 7)
Sub-questions:

- SQ1: How does Figma anchor comments (canvas coordinates vs node ids, pinned to frames, what happens when the node moves or is deleted, REST API comment model with client_meta node_id and node_offset)?
  - Search angles: Figma REST API comments endpoint, Figma help comments attached to frames, Figma dev mode annotations
  - Source types: vendor docs
- SQ2: How do Google Slides (Drive API comments with anchor, object-level comments, what the API exposes) and Pitch (comments on slides and blocks, mentions, resolve) anchor comments to objects, and what happens to comments when the object changes?
  - Search angles: Google Drive API comments anchor Slides, Google Slides comment on object, Pitch help comments
  - Source types: vendor docs, practitioner
- SQ3: How do Gamma, Pitch, Microsoft Copilot in PowerPoint, Canva and Figma (Figma AI, Figma Make) let a user point the AI at one element or a selection (select then prompt, "edit with AI" on a block, @-mention objects), and what context do they pass to the model?
  - Search angles: Gamma edit with AI selection, Pitch AI on block, Copilot PowerPoint selected text, Figma first draft selection
  - Source types: vendor docs, practitioner
- SQ4: Prior art for agents reading comments and selection through APIs or MCP (Figma MCP server get selection, Google Docs comments for agents, coding-agent review comment loops), and for drawing annotations on a slide that are kept out of the document.
  - Search angles: Figma dev mode MCP server selection, MCP comments tools, annotation layers in design tools
  - Source types: primary docs, practitioner

### Cluster: local-prior-art

Title: Blattomat and the presentations plugin as prior art (all questions)
Sub-questions:

- SQ1: What does Blattomat (~/Code/blattwerk) do and how: page and block model (docs/spec-v0.md, docs/okf/page-model.md, docs/okf/blocks/), canvas code in frontend/src (DOM elements? which drag/resize/snap libraries, from package.json), ProseMirror text, undo, how the agent edits through MCP (docs/okf/tools/, versions.md, server-computes.md), and what licenses its dependencies carry?
  - Search angles: read the repo files directly, package.json and pyproject dependencies, git log for design decisions
  - Source types: local repository files
- SQ2: How does Blattomat handle concurrent edits by teacher and agent (versions, conflict handling, ids), and how are its OKF agent docs structured? What of that maps onto decks' open question 6 and the MCP section?
  - Search angles: docs/okf/versions.md, src/blattwerk modules for versioning, remote.py, oauth.py
  - Source types: local repository files
- SQ3: What does the presentations plugin (~/Code/pgoell-claude-tools/plugins/presentations) do: deck-stage.js and presenter view, how slides and layouts are written in HTML, presets and extracting-presets (how a client pptx template's masters are mapped), the hard gates H1, H2 and H7, the editing-presentations deck editor and how its comments and selection reach the agent, and how existing decks animate steps (deck-stage.js)?
  - Search angles: read skills/*/SKILL.md and references, presets folder, scripts
  - Source types: local repository files
- SQ4: How does the plugin's pptx exporter (exporting-presentations-to-pptx) work: which library (python-pptx or PptxGenJS, versions), how it maps HTML elements to shapes, whether it uses slide layouts and placeholders or absolute shapes, how charts, SVG and fonts are handled, and its known limits? What blocks do the existing decks (in the private deck repo) use (to size the typed block set)?
  - Search angles: read exporter scripts and docs, list decks and sample their HTML
  - Source types: local repository files
