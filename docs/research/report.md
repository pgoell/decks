# decks: competitor and prior-art scan

_Date: 2026-10-20_

## Executive Summary

decks should keep the spec's core bets and tighten the model in five places before any v0 code is written. The core bets are these: each block is an HTML element on a 1920 by 1080 slide; the deck is one typed JSON document addressed by id; MCP tools are the agent's only write path; and the pptx export is a python-pptx program built from the plugin's recipes. The scan tested three bigger alternatives (tldraw as the canvas, a two-way text form beside the JSON, a CRDT now) and decks should reject all three. The five places to tighten are cheap now and costly to retrofit: split master from layout and give each slot an OOXML placeholder type and index; store order as fractional-index strings, not array places and integer `z`; keep rich text as structured runs; key the animation model to the PowerPoint preset ids that two importers agree on; and keep comments, selection and drawings in a store beside the deck, anchored to block ids, with a fallback when the block goes.

The case against tldraw is not about DOM against canvas. tldraw itself draws its shapes as DOM, as do Pitch and (through SVG) Google Slides; only Figma, and probably Canva, draw to a `<canvas>` with their own text engine. What rules tldraw out is its license, which forbids production use without a key (a 100-day trial or a paid one), and its model, which has no master, layout or placeholder. On the DOM path the real risk is smaller and nearer: the `moveable` and `selecto` libraries that Blattomat's canvas uses have had no release since 2023-12.

Real pptx layouts need more in the JSON than the spec holds. OOXML and Google Slides both separate the master (text styles, colour map, fixed shapes) from the layout (where placeholders sit), and a slide finds its placeholder on the layout by index, while the layout finds its placeholder on the master by type. The spec's "A master is a layout" and a bare `slot` name cannot carry that. Neither pptx library gives it for free, and neither writes animations, theme colours or SVG; decks will write that OOXML itself whichever library it picks. For agents, the evidence from four studies and the shipping MCP servers points one way: structured data addressed by stable id through semantic tools beats raw HTML, absolute-coordinate OOXML and hand-kept text forms. No shipping AI deck tool protects hand-placed work with a hard rule, so decks needs its own guards: per-operation writes by id, locks that the tools refuse to break, version checks, and undo that holds agent writes. Comments anchor best to object ids with a normalized offset and live outside the document. Live collaboration needs no CRDT, only the model choices that Figma, tldraw and Linear made: client-made ids, fractional order, per-property writes and a server that orders operations.

One caveat cuts against priority, not design. The real decks hold no PowerPoint timing, no builds, no native charts and one placeholder in total (private deck repo, aggregate count). Animations and native charts are bets on future use, not ports of current use; the model should hold them now so no deck needs a rebuild later, but the v0 test should not hang on them.

## Table of Contents

1. Introduction
2. Methodology
3. What Matters Most: the eight open questions answered
4. Supporting Evidence: product by product
5. Analysis and Insights
6. Limitations and Open Problems
7. Future Outlook
8. Conclusions and Practical Starting Point
9. License Table
10. References

## Introduction

decks is a slide editor with two front doors. People get a PowerPoint-like canvas in the browser; agents get a typed JSON document and MCP tools. Its first user is Pascal, who gives talks for work and to clients and today has an agent build each deck as HTML with the `presentations` plugin. The draft spec (`docs/spec-v0.md`) sets the v0 test: rebuild one existing deck in decks, partly by an agent through MCP and partly by hand, give the next talk from it with presenter view, and export a pptx that opens in PowerPoint with editable text.

The spec ends with eight open questions, and this scan exists to answer them so the spec can be settled and turned into issues. The questions cover how to draw the canvas, how masters and layouts work, whether a text form of the deck pays, how AI tools edit decks without wrecking hand work, which export and chart libraries fit, how animations are modelled, how comments anchor, and what live collaboration needs from the model today.

The scan covers the current state (2026-10) of tldraw, Gamma, Pitch, Slidev, Typst with Touying, reveal.js, Keynote, Figma, the Google Slides API, PowerPoint's own OOXML model, and two local sources of prior art: Blattomat (the worksheet editor in the `blattwerk` repo, which already has a DOM canvas and an MCP server) and the `presentations` plugin. It reads pptx export libraries, chart libraries and CRDTs only as far as the eight questions need them, and for every library decks might build on it reads the LICENSE file itself.

The report argues one thesis. decks should keep its core bets (DOM blocks, one JSON document by id, MCP as the only agent write path, a python-pptx exporter), reject tldraw, a two-way text form and a CRDT, and tighten the model in five places now: master apart from layout with OOXML placeholder type and index; fractional-index order; structured rich text; animation keyed to verified PowerPoint preset ids; and comments, selection and drawings kept beside the deck.

## Methodology

The research ran as nine clusters, one per open question plus a cluster on local prior art that fed all eight. Each cluster's researcher read primary sources where they existed: the ECMA-376 standard (through the docx4java mirror), Microsoft's Open XML and [MS-PPTX] pages, the Google Slides API reference, vendor help centres, GitHub source code, issue trackers and LICENSE files. A synthesis pass then merged the clusters, settled seventeen contradictions, and listed twenty-two gaps. A reviewer passed the synthesis with minor notes, and re-fetched five load-bearing claims (the ACE result, the TeXFix-Bench rates, the JSON Whisperer result, the tldraw license text, and the Yjs map size figures), all of which held.

Every claim below carries a credibility tag. **Independent** means standards bodies, bug trackers of independent projects such as LibreOffice, MDN, and academic papers. **Vendor** means the maker's own docs, blog, code or LICENSE file; it is reliable on what the product is meant to do and weaker on how well it does it. **Practitioner** means user reports, forum threads, independent blogs, first-hand inspection during the scan, and the local repos. **Journalism** means press. A few sources are competitors writing about rivals, tagged "vendor (competitor)". Where a claim rests only on a search summary, because the page itself could not be read, the text says so.

The scan has limits of its own, set out in full under Limitations. The largest: no one opened a generated pptx in PowerPoint during the scan; every pptx claim comes from reading XML, specs and code. Pitch's editor, as opposed to its public player, was not inspected. The private deck repo was read only for unnamed aggregates, under a privacy rule, and those counts appear here as "private deck repo, aggregate count".

## What Matters Most: the eight open questions answered

The answers follow the order of the thesis, which is also the spec's order. Each states the question as the spec asks it, gives the answer, the evidence that carries it, and a recommendation the spec can adopt.

### 1. Draw the canvas as HTML elements or on a canvas library such as tldraw?

> Draw the canvas as HTML elements (Blattomat, Pitch) or on a canvas library such as tldraw? Check tldraw's license by reading its LICENSE file, not a badge or a summary.

**Answer: HTML elements. Do not adopt tldraw. Plan for `moveable` as an unmaintained dependency.**

The question as posed assumes a choice between DOM and canvas, but tldraw is not on the canvas side. Its own lead writes that tldraw shapes are "HTML and CSS in the DOM, rendered by React, not drawn to a 2D `<canvas>`"; since SDK 5.0 a single HTML canvas draws only overlays such as handles, snap lines and the selection brush ([20 things I wish AI chatbots knew about tldraw](https://tldraw.dev/blog/20-things-i-wish-ai-chatbots-knew-about-tldraw), vendor; [tldraw SDK 5.0](https://tldraw.dev/blog/tldraw-sdk-5-0), vendor). Pitch's public player draws DOM too: a `div.slide` 1920 px wide scaled with `transform`, blocks placed absolutely, shapes as inline SVG, no `<canvas>` (first-hand DOM inspection of the Pitch public player, 2026-10-20, practitioner; player only, not the editor). The Google Slides editor draws an SVG slide whose group ids follow the API's object ids (first-hand DOM inspection, 2026-10-20, practitioner). Only Figma Slides draws to `<canvas>`, through a C++ and WebAssembly renderer with its own text engine, which is why Figma had to build its own spellcheck ([Building Figma Slides](https://newsletter.pragmaticengineer.com/p/building-figma-slides-with-noah-finer), journalism). Canva's and PowerPoint for the web's rendering could not be determined.

So the live question is license and model fit, and both point away from tldraw. Its root `LICENSE.md`, read directly, is a proprietary source-available license. It permits use "in Development Environments" and requires "Not to use the Software in Production Environments" and "Not to disable, change, or interfere with the Software's License Key enforcement". Production is allowed only under a trial key or "a separate commercial agreement"; the software may "collect and transmit usage data to tldraw for license compliance purposes"; Delaware law governs ([tldraw LICENSE.md](https://raw.githubusercontent.com/tldraw/tldraw/main/LICENSE.md), vendor). The rendering, shapes and UI all sit in the licensed packages (`tldraw`, `@tldraw/editor`, `sync`, `commenting` and others); only `@tldraw/store`, `state`, `tlschema`, `validate` and `utils` are MIT. The license has tightened in steps: non-commercial in 2023-12, commercial with a watermark in 3.x, development-only without a key since 4.0 on 2025-09-17, unchanged in 5.0 and v5.5.2 ([Announcing tldraw SDK 4.0](https://tldraw.dev/blog/tldraw-sdk-4-0), vendor). Vendor pages also describe a free "hobby license" with a watermark; the LICENSE file names only trial and commercial grants. Both are true, since the hobby license is a separate, discretionary, non-commercial grant ([tldraw license page](https://tldraw.dev/community/license), vendor). Pascal's talks are commercial use, so decks would need a paid key. The 2025-10 pricing page showed $6,000 a year for small startups; the 2026-08 and 2026-10 pages show no figure, only "value-based pricing" ([archived pricing, 2025-10-10](http://web.archive.org/web/20251010182142/https://tldraw.dev/pricing), vendor). The current price is unknown.

The model fit is no better. tldraw's slide examples build slides from frames and camera limits, and its 2.5 MB of docs contain no master, layout or placeholder concept ([tldraw docs full text](https://tldraw.dev/llms-full.txt), vendor). Masters would be custom code either way [author estimate].

The DOM path has its own costs. Canvas editors must rebuild selection, clipboard, input methods and spellcheck; Konva's own docs list the limits of its textarea overlay (no rewrap on resize, stale metrics when fonts load late, one style per node, two undo timelines) ([Konva editable text](https://konvajs.org/docs/sandbox/Editable_Text.html), vendor). The one report of DOM drag lag at 50 or more effect-heavy elements comes from a single practitioner blog ([GeekyAnts, Canva-like editor with Konva](https://geekyants.com/blog/building-a-production-ready-canva-like-editor-with-konva-js-react-19-and-next-js-15), practitioner); slides rarely hold that many blocks [author estimate, not measured].

The real risk sits in the drag and resize layer. `moveable`, `react-moveable` and `selecto` are MIT, read from their LICENSE files, but their last releases all date from 2023-12-03. Issue #1137 ("Is this project still active?") and issue #1156 (2026-09-16, an offer to take over maintenance) have no answer from the author, whose work has moved to other projects; an outsider plans to publish a fork ([moveable #1137](https://github.com/daybrush/moveable/issues/1137), practitioner; [moveable #1156](https://github.com/daybrush/moveable/issues/1156), practitioner). Blattomat's `Editor.tsx` (5,874 lines) already carries at least ten documented workarounds for moveable, and one open issue (#732) and one closed (#201) trace back to it (blattwerk: `frontend/src/pages/Editor.tsx`, practitioner).

**Recommendation.** Keep DOM blocks and port Blattomat's canvas. Treat moveable as unmaintained: vendor it or fork it, keep every use behind one module, and watch the fork proposed in #1156. The MIT `@tldraw/store` and `tlschema` packages are fair references for a typed record store; they carry none of the license.

### 2. Masters, layouts and placeholders: what must the JSON hold?

> How do Google Slides, Pitch and PowerPoint model masters, layouts and placeholders, and what of that must the JSON hold for the pptx export to give real layouts?

**Answer: two tiers, not one. The JSON must split master from layout and give every slot an OOXML placeholder type and index.**

OOXML has three tiers plus a theme. The master holds fixed shapes, text styles (`titleStyle`, `bodyStyle`, `otherStyle`, nine levels each) and a colour map. The layout is a "template slide design that can be applied to any existing slide", with a `type` drawn from a fixed list. A slide reaches its master only through its layout ([ECMA-376 sldMaster](https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/sldMaster.html), independent; [ECMA-376 sldLayout](https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/sldLayout.html), independent). A placeholder (`p:ph`) carries a `type` and an `idx`, and the two links run on different keys: a layout inherits from its master by `type` only, a slide from its layout by `idx` only ([ECMA-376 ph](https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/ph.html), independent; [python-pptx: understanding placeholders](https://python-pptx.readthedocs.io/en/latest/user/placeholders-understanding.html), practitioner). Inheriting means writing nothing: a slide placeholder with an empty `<p:spPr/>` takes position, size and shape from its layout, and any `<a:xfrm>` on the slide pins it there. The theme holds twelve colour slots (`dk1`, `lt1`, `dk2`, `lt2`, `accent1` to `accent6`, `hlink`, `folHlink`) and a major and minor font.

Google Slides follows the same shape. Its pages are SLIDE, MASTER, LAYOUT and the notes pair; a slide names both its layout and its master; a placeholder is `{type, index, parentObjectId}`, with types that mirror OOXML's, and leaving a property unset inherits it ([Slides API pages](https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages), vendor). The API cannot create placeholders, layouts or masters, nor change a slide's layout. Keynote's slide layouts tag text, media and object placeholders, and those tags route content when the layout changes ([Keynote: add and edit master slides](https://support.apple.com/guide/keynote/add-and-edit-master-slides-tan7a2b69972/mac), vendor). Pitch takes another road: slide styles as tokens plus a fixed gallery of layouts that "dynamically matches the style", with no placeholders documented ([Pitch: work with slide styles](https://help.pitch.com/en/articles/4063049-work-with-slide-styles), vendor).

Placeholders matter beyond layout: PowerPoint's Outline view and screen readers see placeholder text, not free text boxes ([Elgin: slide layouts](https://elgin.edu/accessibility/accessibility-in-presentations/slide-layouts/), independent). One WebAIM voice calls this "best practice", not a requirement (search summary, practitioner).

The spec's "A master is a layout" folds two tiers into one, and a bare `slot` string per block gives the exporter neither `ph@type` nor `ph@idx`, nor a layout `type` to match on. Neither library fills the gap. python-pptx cannot add layouts or masters (issue #413, open since 2018); the route is a starting pptx that already holds them ([python-pptx #413](https://github.com/scanny/python-pptx/issues/413), practitioner). PptxGenJS's docs say "The defined Masters become first-class Layouts" ([PptxGenJS masters](https://gitbrent.github.io/PptxGenJS/docs/masters/), vendor), but its XML says otherwise: the layouts lack `type`, slide placeholders write their own geometry and font size, so nothing is inherited, and several masters trigger PowerPoint's repair dialog (issue #1449, still in current source) (scratchpad PptxGenJS 4.0.1 experiment, practitioner). The scan sides with the XML.

Today none of this is used. The plugin's exporter mentions no layouts or placeholders, all seven per-deck generators start from the Blank layout, and across the ten shipped pptx files `<p:ph` occurs once in total (private deck repo, aggregate count; plugin `exporting-presentations-to-pptx` SKILL.md, practitioner).

**Recommendation.** Give the JSON, per layout: id, name, layout kind (the OOXML `ST_SlideLayoutType`), owning master; per slot: role (`ph@type`), index (`ph@idx`), box, prompt; per block with a `slot`: inherit unless the block sets its own box (written as an empty `spPr`); per master: text styles by level (title, body, other), fixed shapes, the footer trio (date, footer, slide number) and background; per theme: twelve colour slots plus major and minor fonts, with the preset's role-named tokens mapped onto the slots. Export by generating a base pptx that holds these layouts, then adding slides with python-pptx. Treat this as a hypothesis: no one tested PowerPoint's Change Layout or Reset on such a file during the scan, so the v0 test must.

### 3. Slidev and a text form beside the JSON

> How does Slidev (Markdown) let an agent write slides, and is a text form of the deck worth having beside the JSON?

**Answer: no two-way text form in v0. Add a read-only outline tool beside JSON operations by id.**

Slidev (MIT, v53.0.0 on 2026-09-16) writes slides as Markdown split by `---`, with YAML frontmatter, notes as trailing HTML comments, Vue components and 19 built-in layouts ([Slidev syntax](https://sli.dev/guide/syntax), vendor). Agents get an official skill, VS Code tools, and since 2026-07 a built-in MCP server whose tools list, get, update, insert, remove and move slides addressed by 1-based number, with no screenshot tool ([Slidev MCP](https://sli.dev/features/mcp), vendor). Free positioning comes from `v-drag`, which writes back into the Markdown by regular expression; the docs warn about it and two bugs trace to it ([Slidev draggable](https://sli.dev/features/draggable), vendor; [Slidev #1959](https://github.com/slidevjs/slidev/issues/1959), practitioner).

The research on agent editing points away from text as the write path. JSON Whisperer found that models fail at array index arithmetic, that dictionaries with stable keys fix it, and that patches cut tokens by 31% at near-equal quality ([JSON Whisperer](https://arxiv.org/abs/2510.04717v1), independent). ACE, a scene graph behind 98 semantic tools, beat a Claude-Code-on-HTML baseline on instruction following (4.23 against 3.81, p = .010, 94 tasks) with no significant gap in visual quality; in its ablation the scene graph beat OpenXML, absolute coordinates led to "coordinate-pushing" and spatial errors, and HTML edits "over-applied to whole rows" ([ACE](https://arxiv.org/html/2608.24103v1), independent, preprint; one author's affiliation is not described in the paper). PPTArena found XML-only edits worst and python-pptx with a refinement loop best ([PPTArena](https://arxiv.org/html/2512.03042), independent); Talk to Your Slides found structured editing beat GUI agents ([Talk to Your Slides](https://arxiv.org/abs/2505.11604), independent).

Two sources favour text for generation, and the report weighs them. Aider found four models wrote code slightly worse inside JSON than in Markdown ([Aider: code in JSON](https://aider.chat/2024/08/14/code-in-json.html), practitioner), and one author testing one deck found a Quarto text form the practical choice ([Brosse, LLM slides](https://nbrosse.github.io/posts/llm-slides/llm-slides.html), practitioner, single source). Both are about generating, one is about code, and the other rests on one deck; ACE, PPTArena and JSON Whisperer study editing structured decks and point the other way. Whether format limits hurt reasoning in general stays disputed ("Let Me Speak Freely?", independent, against a rerun by .txt, vendor) and does not decide this.

No product keeps a JSON source with an editable two-way text projection, and the nearest attempt shows why. Slidev Studio (MIT, 2026-08) keeps Markdown as the source, writes drags back as `v-drag`, allows canvas edits only where the markup round-trips exactly, breaks undo on renumbering and cannot reach frontmatter text ([slidev-addon-studio](https://github.com/BobTheShoplifter/slidev-addon-studio), vendor). Read-only projections, by contrast, are the norm: Gamma's `read_gamma` returns text "not raw HTML or JSON", Pitch's MCP reads Markdown, Anthropic's pptx skill reads through `markitdown`, Google's Slides MCP returns "verbalized" structures ([Gamma API docs](https://developers.gamma.app/llms-full.txt), vendor; [Pitch MCP overview](https://help.pitch.com/en/articles/16849368-an-overview-of-pitch-s-mcp), vendor). A rough token count of the spec's one-slide example gives about 255 tokens as compact JSON against about 141 for an invented Markdown-like sketch, near 1.8 times [author estimate: one slide, OpenAI tokenizer, invented format].

Typst with Touying is weaker still for agents. Typst is Apache-2.0 and Touying MIT, output is native PDF with HTML experimental, and pptx exists only as images or through a small "vibe coding" project ([touying-exporter](https://github.com/touying-typ/touying-exporter), vendor; [typ2pptx](https://github.com/touying-typ/typ2pptx), vendor). On TeXFix-Bench, models repairing documents compiled 90.2% of Markdown, 74.2% of LaTeX and 60.3% of Typst ([TeXFix-Bench](https://arxiv.org/html/2608.07617), independent). That is one sizable study, and it measures repair, not authoring; no benchmark of writing Touying decks exists.

**Recommendation.** No two-way text form in v0. Add a read-only outline tool that returns, per slide, the title, the text, and each block's id and type, beside the JSON operations by id. Keep the plugin's `deck.md` as a planning input that an agent turns into operations.

### 4. How AI tools change a deck, and what happens to hand-placed blocks

> How do Gamma and Pitch let an AI change a deck: whole deck, one slide, one block? What does it do to blocks the user placed by hand?

**Answer: every product scopes edits to deck, slide or element; none protects hand-placed work with a rule the agent must obey. decks should build that guard into its tools.**

Gamma 5 (2026-10-06) offers three scopes: the whole deck through chat, one slide through the sparkle button, and one element through AI editing mode, which shows the selected elements in chat. It previews before and after, but accepts or rejects only the whole request; reject restores the prior version ([Gamma: edit content using AI](https://help.gamma.app/en/articles/8033284-how-to-edit-content-using-ai), vendor). Keeping text unchanged means saying so in the prompt. Gamma's help says targeted edits keep "the rest of your gamma intact"; one user reports a single AI action "undid a whole bunch of my own manual edits on multiple other unrelated cards", which version history restored ([Gamma ideas: AI clobbered my edits](https://ideas.gamma.app/ideas/p/gamma-ai-clobbered-my-edits), practitioner). That is one anecdote; Gamma staff asked for detail and no fix is stated, so the contradiction stays open. Gamma's API edits one page by prompt, returns 409 on a concurrent edit, may apply a change in part on failure, and has no object-level addressing ([Gamma API docs](https://developers.gamma.app/llms-full.txt), vendor).

Pitch offers AI actions on a text block, whole-slide edits, Pitch Agent (2026-05) for the deck or a selected block, and "Ask Agent to edit the slide you're on" (2026-08); it says output is "fully editable" and "you're expected to review it" ([Pitch Agent](https://help.pitch.com/en/articles/14981091-pitch-agent), vendor). How Pitch treats hand-placed blocks, and whether it previews, is unknown: no source covers it.

The rest of the field looks the same. Copilot in PowerPoint promises "Formatting, structure, and branding will be preserved" and rewrites only whole text boxes; its per-slide note "Don't modify this slide." is guidance, and its strict Brand Kit "won't add or remove placeholders or invent new layouts" ([Copilot: keep on brand](https://support.microsoft.com/en-us/powerpoint/copilot/keep-your-presentation-on-brand-with-copilot), vendor). Gemini in Slides works one slide at a time with Replace, Insert or Close ([Gemini in Slides help](https://support.google.com/docs/answer/16961475?hl=en), vendor). Plus AI's Remix rewrites text to fit by default and advises "duplicate the slide before remixing" ([Plus AI Remix](https://guide.plusai.com/ai-for-presentations/remix-slides), vendor). Figma Make stages changes until Apply, and its users ask for section locks ([Figma forum: lock sections in Make](https://forum.figma.com/suggest-a-feature-11/is-there-a-way-to-lock-sections-in-figma-make-to-ensure-they-aren-t-removed-or-revised-with-an-unrelated-prompt-to-another-section-of-the-project-50240), practitioner). Canva has Brand Template locks, but whether its AI honours them could not be read (the page returned 403).

Blattomat already has the guard the others lack. Its `delete_block`, `move_block` and `update_block` tools refuse locked blocks, and every write refuses a stale version with "Nothing was written" (blattwerk: `docs/okf/tools/errors-and-limits.md`, `docs/okf/versions.md`, practitioner). Two holes remain. `replace_doc` documents no lock refusal, so a whole-document write could bypass locks; and the lock refusal was traced through docs and tests, not code. Blattomat's undo, too, falls short of what the spec promises: it is a stack of whole-document snapshots in the canvas, and a document loaded from the server remounts the canvas and empties history, so agent writes never enter the user's undo (blattwerk: `frontend/src/history.ts`, `Editor.tsx:388, 402, 414`, practitioner). The spec's "Undo brings the wrapped slide back" fails on the code it would port.

**Recommendation.** Scope agent edits to deck, slide and block, all through operations addressed by id. Make locks a tool refusal that also covers whole-slide and whole-deck writes. Version every agent write, make each one a single undo step for the user, and return a summary. Defer preview and branch review to phase 3, but keep an operation log that allows them.

### 5. Libraries for pptx export and charts

> Which libraries for pptx export and for charts fit, and what are their licenses?

**Answer: python-pptx as the core writer, plus small OOXML writers of decks' own; one table-shaped Chart block drawn by ECharts (or Vega-Lite) in the browser and as a native chart in pptx.**

The first finding corrects the spec's wording. "Port of the exporter" assumes code to port, and there is none. The plugin's exporter is a procedure, a SKILL.md plus recipes, and each deck gets a fresh python-pptx script: seven generators, each hundreds to thousands of lines, a greedy wrap estimator, thirteen gotchas, SVG rasterized with PyMuPDF, and a LibreOffice render-and-check loop of two to four rounds (plugin `exporting-presentations-to-pptx/SKILL.md` and `references/python-pptx-recipes.md`, practitioner; private deck repo, aggregate count). A native-chart recipe exists, but no generator ever called `add_chart`. decks writes a new program from these recipes.

python-pptx is MIT, read from its LICENSE file. Its last release is 1.0.2 (2024-08-07), with no commits since and 541 open issues, yet about 60 million downloads a month ([python-pptx LICENSE](https://github.com/scanny/python-pptx/blob/master/LICENSE), vendor; [PyPI metadata](https://pypi.org/pypi/python-pptx/json), vendor). It writes native charts (most 2D types, with an embedded workbook), tables, text runs and notes; it cannot create layouts, skips SVG, and has no API for embedding fonts or for animation ([python-pptx charts](https://python-pptx.readthedocs.io/en/latest/user/charts.html), vendor). PptxGenJS is MIT too, v4.0.1 (2025-06-26), about 14 million downloads a month. Its layouts fail as described under question 2; it writes charts (ten types, no waterfall), tables and notes; in Node its SVG becomes a broken-image PNG; it has no theme colours, no font embedding and "no plans" for animations; and it has open 2026 bugs that trigger PowerPoint's repair dialog, including orphan `c:axId` on bar and line charts ([PptxGenJS LICENSE](https://github.com/gitbrent/PptxGenJS/blob/master/LICENSE), vendor; [PptxGenJS repair issues](https://github.com/gitbrent/PptxGenJS/issues?q=is%3Aopen+repair), practitioner).

The others do not change the choice. power-pptx is an MIT fork created in 2026-04 with 13 stars and frequent releases, partly committed by "claude"; it claims animations, a theme writer, SVG with PNG fallback and JSON authoring, and none of it was checked ([power-pptx](https://github.com/codehalwell/power-pptx), vendor). pptx-automizer is MIT, template-based, Node only, and cannot add layouts. officegen is dead since 2021. The Open XML SDK is MIT but .NET. Aspose.Slides can add layouts and embed fonts but costs from $1,399; its MIT "FOSS for Python" sibling (2026-03, 15 stars) claims animations and 57 transitions, unchecked.

Both main libraries share the gap that matters: neither writes animations, transitions, theme colours or SVG. The spec's "native PowerPoint ones" means decks writes that OOXML itself, whichever library it picks. SVG in PowerPoint is a PNG `a:blip` with an `asvg:svgBlip` extension beside it, supported in Microsoft 365, 2019 and later, and on Mac ([Microsoft: edit SVG images](https://support.microsoft.com/en-us/office/edit-svg-images-in-microsoft-office-69f29d39-194a-4072-8c35-dbe5e7ea528c), vendor). Fonts are uneven everywhere: PowerPoint on Windows and Mac embeds TTF and OTF within the fonts' embedding flags, LibreOffice 25.8 embeds, Google Slides ignores embedded fonts, and Pitch falls back to Calibri ([Microsoft: benefits of embedding custom fonts](https://support.microsoft.com/en-us/office/benefits-of-embedding-custom-fonts-cb3982aa-ea76-4323-b008-86670f222dbc), vendor; [Collabora on LibreOffice 25.8](https://www.collaboraonline.com/blog/collabora-productivity-contributions-in-libreoffice-25-8/), vendor; [Gamma: why export does not match](https://help.gamma.app/en/articles/15939201-why-doesnt-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma), vendor). The plugin sidesteps this with widely installed faces (Aptos, Consolas) or the client's corporate font (plugin `python-pptx-recipes.md`, gotcha 11, practitioner).

Charts need a library that renders SVG in the browser and in the Chromium PDF, from a plain 2D table that maps onto native pptx chart data. Two fit. ECharts (Apache-2.0 with a NOTICE file; its zrender dependency BSD-3) renders to an SVG string on the server with `ssr: true`, is fine below about a thousand points, and takes a `dataset.source` table whose columns map to series; it has open SVG-renderer bugs (right-to-left text, hover at 2,000 to 3,000 points, data-URI images) ([ECharts server-side rendering](https://echarts.apache.org/handbook/en/how-to/cross-platform/server/), vendor; [ECharts NOTICE](https://github.com/apache/echarts/blob/master/NOTICE), vendor). Vega-Lite and Vega (BSD-3) render SVG through `view.toSVG()`, and vl-convert renders from Python without Node ([vl-convert LICENSE](https://github.com/vega/vl-convert/blob/main/LICENSE), vendor). The rest fail on a specific point: Chart.js is canvas only, so a bitmap in the PDF; Recharts has "never been explicitly supported" server rendering and gives empty output in 3.x ([Recharts #5997](https://github.com/recharts/recharts/issues/5997), practitioner); Observable Plot's server rendering is "only practical for simple plots"; D3 has no chart spec.

No maintained library maps an ECharts or Vega-Lite spec to a native pptx chart [author estimate, one search round]. But both pptx libraries take near-identical category data (`{name, labels, values}` in PptxGenJS; `categories` plus `add_series` in python-pptx), so a block that holds a plain table (categories, series names, values, chart kind) maps onto both renderers with small hand-written code. Users ask for exactly this: Gamma staff call image-only chart export a "painful workflow in consulting" ([Gamma ideas: native PowerPoint chart exports](https://ideas.gamma.app/ideas/p/native-powerpoint-chart-exports-dont-exported-as-images), practitioner), and think-cell's output, the consulting norm, is plain PowerPoint charts ([think-cell compatibility](https://www.thinkcell.com/en/resources/manual/compatibility), vendor).

The license reads raise flags on the current export path. PyMuPDF is AGPL-3.0 or commercial and CairoSVG is LGPL-3.0, both in today's exporter; Chromium through Playwright, already in the stack for the PDF, can rasterize instead.

**Recommendation.** python-pptx (MIT, it fits the FastAPI backend, and the recipes exist) as the core writer, plus decks' own small OOXML writers for layouts in a generated base file, the theme `clrScheme`, `svgBlip` with a Chromium-rendered PNG, `p:transition` and `p:timing`. Replace PyMuPDF and CairoSVG with Playwright's Chromium for rasterizing. One table-shaped Chart block: ECharts SVG in browser and PDF, a native pptx chart from the same table. Ship ECharts' NOTICE.

### 6. Animation models, and what survives the trip to pptx

> How do PowerPoint (`<p:timing>`), Keynote, Google Slides, reveal.js and Slidev model animations and builds, and which effects survive the trip to pptx and back?

**Answer: the spec's model already matches PowerPoint's. It lacks a verified preset-id table, a per-paragraph build target, and a plan for transitions' millisecond duration. Target decks' own HTML runtime and PowerPoint, and nothing else.**

PowerPoint stores one `p:timing` per slide. A main sequence holds click groups, each click group holds effects that start on click, with the previous effect, or after it, and each effect carries a preset class (entrance, exit, emphasis, path), a `presetID`, a `presetSubtype` bit flag for direction, and durations in milliseconds ([ECMA-376 seq](https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/seq.html), independent; [ECMA-376 cTn](https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/cTn.html), independent). A PowerPoint-saved Fly In shows the nesting in practice: an outer `par` waiting on a click, an inner `par`, then the effect with `presetID="2" presetClass="entr" presetSubtype="4"`, and a build entry `<p:bldP spid="4" grpId="0"/>` ([PowerPoint-saved sample, python-pptx #954](https://github.com/scanny/python-pptx/files/14556139/file_with_bounce.pptx), practitioner). When a p14 feature is used, PowerPoint wraps the whole `p:timing` in `mc:AlternateContent` with a fallback copy. Text builds by paragraph through `bldP build="p"` with a level, and each paragraph's effect targets a range of paragraphs; a mismatched `grpId` fails without warning.

The standard leaves `presetID` as a plain integer with no list, so the ids must come from somewhere else. LibreOffice's import table gives: entrance appear 1, fly in 2, fade 10, wipe 22, zoom 23; exit reuses the same ids; emphasis fill colour 1, font colour 3, grow and shrink 6, spin 8, pulse 26, teeter 32; subtypes 1 top, 2 right, 4 bottom, 8 left, 16 in, 32 out ([LibreOffice commontimenodecontext.cxx](https://github.com/LibreOffice/core/blob/master/oox/source/ppt/commontimenodecontext.cxx), independent). Aspose's FOSS mapping agrees, a COM-observed table agrees on pulse as 26, and the PowerPoint-saved Fly In agrees on 2 and 4. Three writers disagree, each in a different direction: dom-to-pptx calls entrance 23 "Grow & Turn", svg2ooxml uses emphasis 32 for grow and shrink, and Microsoft's own sample MCP server, Resource2Skill, lists spin as 2 and pulse as 8 ([Resource2Skill server.py](https://github.com/microsoft/Resource2Skill/blob/main/domains/ppt/mcp_server/server.py), vendor). The scan sides with the importers. The VBA `MsoAnimEffect` numbers differ again for emphasis and paths and must not be used ([MsoAnimEffect](https://learn.microsoft.com/en-us/office/vba/api/powerpoint.msoanimeffect), vendor).

Transitions sit on the incoming slide as `p:transition`. ECMA gives only speed keywords; a millisecond duration is `p14:dur` inside `mc:AlternateContent` with a fallback. Morph is `p159:morph`, matched by "!!" names, and LibreOffice has had no Morph since a bug opened in 2018 ([LibreOffice bug 116736](https://bugs.documentfoundation.org/show_bug.cgi?id=116736), independent). Whether Morph needs the same wrapper stays unresolved, since [MS-PPTX] could not be read; Morph is not in the spec's v0 list.

The other tools model the same ideas with less. Keynote has build in and out with delivery by bullet group, action builds (opacity, rotate, scale, move along a path), and the same three start triggers; Magic Move matches by object, word or character ([Keynote: animate objects](https://support.apple.com/en-sa/guide/keynote/tan72234bb6/mac), vendor). Google Slides has the three triggers, by-paragraph builds and a short effect list, and no animation or transition field in its API ([Google Slides: animate](https://support.google.com/docs/answer/1689475?hl=en), vendor). reveal.js fragments and Slidev clicks both derive state from the current step index, so stepping back is just a function of the index ([reveal.js fragments](https://revealjs.com/fragments/), vendor; [Slidev animations](https://sli.dev/guide/animations), vendor).

Little survives a round trip. Neither pptx library writes animations, and hand-written XML is "silently" dropped on small mistakes (python-pptx and PptxGenJS maintainers, vendor). Pptx to Google Slides loses delays, rounds timing, and drops motion paths and Morph; the two practitioner sources disagree on whether unknown presets vanish or flatten to a single appear, but agree on loss ([BrightCarbon: convert PowerPoint to Google Slides](https://www.brightcarbon.com/blog/convert-powerpoint-google-slides/), practitioner). Keynote's pptx export of builds is undocumented; Apple Community says only video guarantees them ([Apple Discussions 250144385](https://discussions.apple.com/thread/250144385), practitioner). LibreOffice has a meta bug with 27 open animation dependents ([LibreOffice bug 139897](https://bugs.documentfoundation.org/show_bug.cgi?id=139897), independent), so the plugin's LibreOffice check loop cannot verify animations. In the browser, the Web Animations API supplies `reverse()`, `finish()`, `cancel()` and `currentTime`, and CSS `offset-path: path()` takes SVG path data, widely available since 2022 ([MDN offset-path](https://developer.mozilla.org/en-US/docs/Web/CSS/offset-path), independent).

None of this is in use yet: the shipped pptx files hold no `<p:timing>` and one `<p:transition>` in total, and the plugin's build steps appear in 13 gallery slides and no built deck (private deck repo, aggregate count; plugin presets, practitioner).

**Recommendation.** Keep the spec's model. Add `by: "paragraph"` with an optional level; a direction field mapped to `presetSubtype`; name each effect from a closed list mapped to the LibreOffice and Aspose id table; and state playback as a function of the step index, in the reveal.js and Slidev way, so going back works. Write transition milliseconds as `p14:dur` with a fallback `spd`. Verify animation export in PowerPoint, never LibreOffice.

### 7. Comments and pointing the AI at an element

> How do Figma, Google Slides and Pitch anchor comments to objects, and how do Gamma and Pitch let a user point their AI at one element?

**Answer: anchor comments to object ids with a normalized offset and a defined fallback on delete; keep them outside the document; let agents list, reply and resolve.**

Figma's REST API anchors a comment either to a canvas point or region, or to a frame by `node_id` plus a pixel offset from its top-left corner; replies are flat by `parent_id`, the REST API cannot resolve, and pins attach to top-level frames only ([Figma comments types](https://developers.figma.com/docs/rest-api/comments-types), vendor). Users' reports that comments do and do not move with frames agree once read that way. Google's Drive API anchors are opaque and editors do not render them, but the Slides API's developer preview (2026-08-31, MCP comments since 2026-10-01) anchors threads to slide or element ids, optionally a text range or table cell, through a separate anchor list, with reply, resolve, reopen and assignee ([Slides API comments](https://developers.google.com/workspace/slides/api/guides/comments), vendor). PowerPoint's modern comments anchor to exactly one of a slide, a drawing element (by id and creation GUID) or a text range, with position relative to the anchored object, and since 2025 they follow cut-and-pasted objects ([MS-PPTX CT_Comment](https://learn.microsoft.com/en-us/openspecs/office_standards/ms-pptx/161bc2c9-98fc-46b7-852b-ba7ee77e2e54), vendor; [Office Watch: comments stick like glue](https://office-watch.com/2025/powerpoints-comments-stick-like-glue/), journalism). Pitch documents comments per slide, with a panel, mentions, resolve and a Commenter role; whether it anchors comments to blocks is unknown ([Pitch: collaborate with comments](https://help.pitch.com/en/articles/4318672-collaborate-with-comments), vendor).

tldraw has the best-documented design. A comment anchors to a shape (with a normalized 0 to 1 offset and a precise flag), a point, a region or a page; when the shape is deleted the thread becomes a point anchor where the pin sat and re-attaches on undo unless moved; threads stay out of snapshots and exports and sync in a separate lane ([A comment is not a shape](https://tldraw.dev/blog/a-comment-is-not-a-shape), vendor; [anchor-lifecycle.ts](https://github.com/tldraw/tldraw/blob/main/packages/commenting/src/canvas/anchor-lifecycle.ts), vendor). The release notes for 5.3.0 list a text-range anchor; the blog says it was cut and the source has no such variant, so the scan takes the source. `@tldraw/commenting` is under the tldraw license: decks can copy the design, not the code. What other tools do when a commented object is deleted is undocumented for Figma, Google Slides and PowerPoint; for Lucid, a search summary of an unreadable page says the comment is orphaned and comes back with the revision (practitioner, single source).

Pointing works the same way in every product: selection scopes the request. Gamma's AI editing mode shows clicked elements in chat; Pitch Agent offers Quick actions on a selected block; Canva lets users @mention its AI in a comment on a selected element; Figma's selection scopes its agent, and Make has a point-and-edit tool with staged Apply ([Figma prompt to edit](https://www.figma.com/solutions/prompt-to-edit/), vendor). One Figma Make bug shows the failure mode, where the selection never reached the prompt ([Figma forum: selected element not passed](https://forum.figma.com/report-a-problem-6/selected-element-not-passed-to-figma-make-ai-prompt-51238), practitioner). A study of editing 3D scenes with an LLM found pointing gives precision and speech gives speed, and that ambiguous references hit every object ("changed all the books") ([LLM scene editing with pointing](https://arxiv.org/html/2410.22177), independent).

For agents reading comments, the working patterns come from code review. GitHub's MCP server lists, replies to and resolves review threads by `threadId`; Vercel's `comments` CLI lists unresolved threads by default and returns JSON for agents ([github-mcp-server README](https://github.com/github/github-mcp-server/blob/main/README.md), vendor; [Vercel CLI comments](https://vercel.com/docs/cli/comments), vendor). Figma's MCP reads selection only on the desktop server and has no comment tools; Pitch's MCP has neither. No product ties freehand pen strokes to a comment thread. The plugin's deck editor keeps pen drawings in the editor and passes them to the agent as red marks in a screenshot, and its known weakness is line numbers that go stale after the first edit, which the spec's switch to ids targets (plugin `editing-presentations` SKILL.md, practitioner).

**Recommendation.** Keep comments in their own table, never in the deck. An anchor is a slide id plus optional block ids, with a normalized pin offset and an optional build step. When a block is deleted, keep the thread with a slide-point anchor at the last pin, as tldraw does. Store drawings as vector strokes owned by the comment, drawn only in the editor and in screenshots. MCP tools: list (open threads by default), reply, resolve, reopen, plus `get_selection`, which returns ids, types, props and a screenshot with boxes drawn on it.

### 8. What live collaboration needs from the model now

> What would live collaboration need from the model now (Yjs, Automerge), so v0 does not block it?

**Answer: no CRDT. It needs stable client-made ids, order as fractional indexes, per-property last-writer-wins, rich text as structured data, no derived data in the document, and presence and comments outside it.**

The products that already ship live editing on objects do it without a CRDT. Figma runs one process per file with a journal and per-property last-writer-wins on `Map<id, Map<prop, value>>` ([How Figma's multiplayer technology works](https://www.figma.com/blog/how-figmas-multiplayer-technology-works/), vendor). tldraw sync is "deliberately not a CRDT": it commits, discards and rebases with diffs keyed by record id, keeps a document clock and tombstones, and runs one instance per room ([tldraw sync-core SPEC.md](https://github.com/tldraw/tldraw/blob/main/packages/sync-core/SPEC.md), vendor). Linear orders transactions on the server ([reverse-linear-sync-engine](https://github.com/wzhudev/reverse-linear-sync-engine), practitioner, unaffiliated). Google Docs has run on an operation-transform revision log since 2010, and the Slides API's `requiredRevisionId` refuses stale writes ([Slides API batchUpdate](https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/batchUpdate), vendor). The evidence on how Pitch and Google Slides themselves sync is thin: one vendor blog and one forum comment for Pitch, and no Slides-specific sync source at all.

The CRDT libraries themselves are all MIT (Yjs, Yrs, pycrdt, Hocuspocus, Automerge, Loro), so license does not decide this. Fit does. Yjs maps keep history per key and grow under alternating writes: 100,000 operations on 10 keys came to 524,985 bytes as a `Y.Map` against 271 bytes as a `YKeyValue` ([y-utility README](https://cdn.jsdelivr.net/npm/y-utility@0.1.4/README.md), vendor); one field report of 30 MB a day of growth ended in a move to last-writer-wins outside Yjs ([discuss.yjs.dev: document growing unbounded](https://discuss.yjs.dev/t/document-growing-unbounded/3729), practitioner). Yjs 14 is still a release candidate. Automerge keeps full history, has no list move, and makes schema migrations hard ([Automerge: modeling data](https://github.com/automerge/automerge.github.io/blob/main/content/docs/cookbook/modeling-data.md), vendor). Loro's fork and branch APIs are still changing (Loro docs, vendor). The one benchmark suite is run by the Yjs author on old Automerge and Loro versions, and no benchmark of a deck-shaped document exists ([crdt-benchmarks](https://github.com/dmonad/crdt-benchmarks), practitioner). Above all, "CRDTs merge; they do not reject" ([Loro: when not to use a CRDT](https://github.com/loro-dev/loro-docs/blob/main/pages/docs/concepts/when_not_crdt.mdx), vendor), and the spec has rules that must reject, such as "deleting a block deletes its animations". A server can check those.

What matters is the model, and here the spec has one gap that ids alone do not close. It stores order as array places and stacking as integer `z`, and the animation timeline as an ordered array. Figma, tldraw and Excalidraw all store order as fractional-index strings per object, with a random suffix against interleaving and a server that removes duplicates; Excalidraw retrofitted this in 2024 and was still patching it in 2026 ([Figma: realtime editing of ordered sequences](https://www.figma.com/blog/realtime-editing-of-ordered-sequences/), vendor; [Excalidraw PR 7359](https://github.com/excalidraw/excalidraw/pull/7359), vendor). The spec's "every operation names them by id" is necessary but not enough while order is still positional. JSON Patch addresses arrays by index too, which is fragile for the same reason ([RFC 6902](https://www.rfc-editor.org/rfc/rfc6902), independent). Rich text needs its own sequence type, since last-writer-wins on a string loses concurrent edits. Client-made ids matter because Figma "can't" have the server assign them.

Blattomat is a sound v0 base for this. It has no operation log, but it saves the whole document with a version, compares and sets in SQL, returns 409 on a clash and offers overwrite, keep as new or discard; agent tools take the version and refuse on mismatch (blattwerk: `src/blattwerk/sheets.py`, `frontend/src/saves.ts`, `docs/okf/versions.md`, practitioner). It has no server push. One more gap: idea.md says the agent changes a deck "only through the same operations the editor uses", but Blattomat's agent tools are a separate module that sends the whole document back (blattwerk: `frontend/mcp/tools.ts`, practitioner).

**Recommendation.** No CRDT in v0. Keep ids client-made. Replace array order and integer `z` with fractional-index strings (an `order` field on slides, blocks and animations; rocicorp's `fractional-indexing` is CC0). Store text as ProseMirror-compatible runs. Make operations target ids and write single properties. Have the server apply operations in order, check invariants and cascades, and bump a deck version. Keep comments and presence beside the deck, and add a schema version field for migrations. That leaves both a tldraw-sync-style server rebase and a later Yjs or Automerge mapping open.

## Supporting Evidence: product by product

The answers above cut across products. This section turns it around: one profile per product or library, with what the scan found about it and what decks should take from it. Facts already argued above appear here in short form.

### tldraw

tldraw is the strongest product in the scan and the one decks should not build on. Its store holds typed records with branded ids, `props`, `meta`, an index and a lock flag; edits go through `editor.createShapes`, `updateShapes` and `run`; rich text is TipTap (ProseMirror) JSON; export is SVG through `toSvg` or `<foreignObject>`, with fonts inlined ([tldraw editor docs](https://tldraw.dev/docs/editor), vendor). Snapping is off by default (hold Ctrl), with a separate grid, camera limits and locked shapes. Its agent starter kit, which replaced `@tldraw/ai`, gives the model a simplified JSON view of the page (focused shapes in full, nearby shapes blurred, distant ones as clusters) plus a screenshot, and cleans up ids the model invents ([tldraw agent starter kit](https://tldraw.dev/starter-kits/agent), vendor). That design, a reduced JSON view plus an image, matches the read-only outline decks should offer. tldraw's commenting and sync designs (questions 7 and 8) are the best documented in the scan. All of it sits behind a license that forbids production use without a key (question 1), with a price that was $6,000 a year for small startups in 2025-10 and is now unpublished. The Hacker News reaction to the 4.0 change split between "we don't want your business" and "quite reasonable" ([HN thread](https://news.ycombinator.com/item?id=45293839), practitioner). decks takes tldraw's designs and leaves its code.

### Pitch

Pitch is the product the spec names as its DOM model, and the scan learned less about it than about any other. Its public player is DOM: a 1920 px `div.slide` scaled with `transform`, absolute blocks, inline SVG shapes, and text in what look like read-only Quill editors [class reading is author estimate]; the stack is ClojureScript with UIx and React (first-hand inspection, practitioner; [Nubank on UIx](https://building.nubank.com/building-elegant-interfaces-with-clojurescript-react-and-uix/), practitioner). The editor was not inspected. Its styling model is slide styles (tokens for colours, fonts, background, text styles and chart palette, where a colour edit updates every element) plus a fixed, workspace-wide layout gallery; no placeholders are documented, and the pptx export lists animations as unsupported and says nothing about layouts ([Pitch: export to PowerPoint](https://help.pitch.com/en/articles/6713988-export-a-presentation-to-power-point), vendor). Its AI runs from text-block actions through Pitch Agent (2026-05) to the Anything block (2026-10), drawing on "150+ additional layouts" and models from Anthropic, OpenAI and Google ([Pitch: what's new](https://pitch.com/whats-new), vendor). Its MCP fills template variables or prompts the agent, reads Markdown, and has no element operations. Comments are per slide. Pitch's AI treatment of hand-placed blocks, its previews, its block-level comment anchors, its sync design and the layout structure of its pptx are all unknown. Nearly every Pitch fact here is vendor-sourced, and the report does not infer what the sources leave out.

### Gamma

Gamma is the clearest case of an AI-first deck tool, and of what that costs. Its cards are fluid by default, with fixed 16:9 optional; the API's dimensions include `fluid` and `pageless` ([Gamma: cards](https://help.gamma.app/en/articles/11016396-what-are-cards-in-gamma-and-how-do-they-work), vendor). Themes restyle a deck and keep content ([Gamma: change theme](https://help.gamma.app/en/articles/10262646-how-do-i-change-my-gamma-theme), vendor). Its pptx export sometimes turns a slide into one locked image and breaks text line by line ([Gamma: why export does not match](https://help.gamma.app/en/articles/15939201-why-doesnt-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma), vendor); a competitor's blog adds that the slide master may be blank ([WPS on Gamma exports](https://www.wps.com/blog/gamma-ppt-fix-exported-decks-in-wps-presentation/), vendor (competitor), search summary). Gamma 5's three edit scopes, whole-request acceptance and the clobbered-edits report are set out under question 4. Its MCP began as generate-only, which Gamma staff called "pretty painful" for iteration ([Gamma ideas: add edit support to MCP](https://ideas.gamma.app/ideas/p/add-update-edit-support-to-gamma-mcp-not-just-generation), practitioner). Reddit users saw Gamma 5 as a loss of fine control; one import test kept text and table but swapped fonts and moved two titles ([r/GammaApp](https://reddit.com/r/GammaApp/comments/1x6a2yi/), practitioner). One thread of praise carries less weight because of an unverified sponsorship claim (practitioner, weight lowered). The lesson for decks: an agent that cannot address objects ends up rewriting more than the user asked.

### Google Slides

Google Slides is the closest public match to the model decks needs. The editor draws SVG with group ids that follow API object ids; text is SVG `<text>`; input runs through hidden iframes; no canvas (first-hand inspection, practitioner). Google Docs moved to canvas in 2021; Slides did not ([Workspace Updates: Docs canvas rendering](https://workspaceupdates.googleblog.com/2021/05/Google-Docs-Canvas-Based-Rendering-Update.html), vendor). The API models master, layout and placeholder in OOXML's shape (question 2), its editor puts a Theme Builder above layouts and applies layouts per slide, and it edits only the master's twelve-slot colour scheme, with colour names that mirror OOXML's. Its `batchUpdate` is atomic, addressed by object id, and refuses stale writes through `requiredRevisionId`; its official MCP server is in developer preview. Charts come only from Sheets. The 2026 comments API anchors threads to object ids (question 7). There is no animation or transition field in the API, and pptx imports lose delays, motion paths and Morph. What Slides' own pptx download keeps of layouts and animations was not tested.

### PowerPoint and OOXML

PowerPoint's file format is the target decks must meet, and its own behaviour sets the bar. Master edits reach every slide; edited layouts must be reapplied to existing slides; on a layout switch, content goes to placeholders in creation order, not by name, and orphans land at the slide edge with "no out-of-the-box way" to fix it ([Microsoft Q&A on placeholder order](https://learn.microsoft.com/en-us/answers/a/11388916), practitioner). PowerPoint does not clone the date, footer and slide-number placeholders onto new slides, adds `<a:spLocks noGrp="1"/>`, and drops `hasCustomPrompt` ([python-pptx: placeholders in new slide](https://python-pptx.readthedocs.io/en/stable/dev/analysis/placeholders/slide-placeholders/placeholders-in-new-slide.html), practitioner). Master and layout ids are unsigned integers of 2147483648 and above. Renderers disagree on the details: LibreOffice inherits where PowerPoint does not for `idx` 4294967295 (bug 104202, independent). Copilot reads placeholder type and position but not text, has mistaken decorative shapes for placeholders unless layouts are named, and in earlier field reports reused one layout throughout ([PrimeDocs on Copilot](https://docs.primedocs.io/en/docs/4.0/layouter/best-practices/microsoft-365-copilot), vendor; [Microsoft Q&A 1908366](https://learn.microsoft.com/en-us/answers/a/1908366), practitioner). Office.js addresses shapes by id, in points ([Office.js PowerPoint shapes](https://learn.microsoft.com/en-us/office/dev/add-ins/powerpoint/shapes), vendor). Whether Copilot passes a selected shape to its model has no developer source. PowerPoint shows a revisited slide in its final state, and its file holds hooks for stepping back, but no Microsoft source covers stepping back within a slide.

### Keynote

Keynote shares PowerPoint's ideas under other names. Slide layouts hold tagged text, media and one object placeholder; objects that are not placeholders become uneditable background; layout edits reach all slides; tags route content on a layout change; and reapplying a layout resets style and position but keeps content ([Keynote: change layout or theme](https://support.apple.com/guide/keynote/change-the-slide-layout-or-theme-tan584189747/mac), vendor). Its builds and Magic Move are described under question 6. No source says how Keynote maps builds, Magic Move or layouts into pptx; one third-party page claims Magic Move becomes a fade (unverified, practitioner). Keynote is a reference for decks, not a target.

### Figma and Figma Slides

Figma Slides draws to canvas with WebGL or WebGPU and its own text and spellcheck. It cannot change a slide's layout after creation, and template changes have not reached existing decks from 2024 to 2026 ([Figma forum: change layout](https://forum.figma.com/ask-the-community-7/how-do-i-change-the-layout-of-an-existing-slide-in-figma-slides-46971), vendor). Its pptx export swaps fonts, turns gradients solid and code blocks into images. A plugin, Pitchdeck, builds PowerPoint masters from Figma components ([Hypermatic Pitchdeck](https://docs.hypermatic.com/pitchdeck/tutorials/add-master-slides-to-powerpoint-exports-from-figma.md), vendor). Figma's comment model (frame id plus pixel offset, top-level frames only) and its MCP's limits are under question 7. Dev Mode annotations live on the node and follow it, which is the closest Figma comes to the anchor decks wants.

### Slidev

Slidev is the reference for Markdown decks with agents. Beyond its syntax and MCP (question 3), its export story moved in 2026: images plus notes since 2024-05, and `--format pptx-editable` since 2026-09-16, which walks the rendered DOM and writes native shapes through PptxGenJS, falls back to an image per slide, and has no masters, font embedding or gradients; SVG stays a picture and each click step becomes a slide ([Slidev PR 2722](https://github.com/slidevjs/slidev/pull/2722), vendor). One guide's claim that pptx "is not the core use case" held before that date ([pkgpulse](https://www.pkgpulse.com/guides/slidev-vs-marp-vs-revealjs-code-first-presentations-2026), practitioner). Whether `--with-clicks` should be the default is disputed in an open PR. Slidev addresses slides by number; marp-mcp, by contrast, embeds stable slide ids in Markdown comments and offers typed layout schemas ([marp-mcp slide-id.ts](https://github.com/masaki39/marp-mcp/blob/main/src/utils/slide-id.ts), vendor). Marp's own `--pptx-editable` goes through LibreOffice and its authors "not recommend" it.

### Typst with Touying

Touying makes slides from headings, subslides from `#pause` and `#meanwhile`, and notes for pdfpc; v0.8.0 (2026-09-17) needs Typst 0.15 and broke compatibility ([Touying README](https://github.com/touying-typ/touying/blob/main/README.md), vendor). The only WYSIWYG editor, Tylina, is proprietary ("All rights reserved") and keeps `.typ` as the source. Opinions on models writing Typst split from "pretty good" to "terrible" (Typst forum and HN, practitioner); the one sizable independent study, TeXFix-Bench, leans against it, with the caveat that it measures repair. Without an editable pptx path, Touying does not fit decks' v0 test.

### reveal.js

reveal.js (MIT) contributes two ideas. Fragments set visibility by index, can share an index, and fire events; auto-animate matches elements by content or `data-id` ([reveal.js auto-animate](https://revealjs.com/auto-animate/), vendor). Because state follows from the step index, going back is free, which is the model decks' runtime should copy. reveal.js practitioners who animate parts of an SVG inline it and put classes on the parts, and call the CSS messy ([Sacha Chua on animating SVG with reveal.js](https://sachachua.com/blog/2024/01/animating-svg-topic-maps-with-inkscape-emacs-ffmpeg-and-reveal-js/), practitioner). In pptx such effects have no target once the SVG becomes an image, which the spec already accepts.

### pptx export libraries

The comparison is under question 5. Two other references are worth keeping. dom-to-pptx (MIT, about 209,000 downloads a month) walks the DOM, embeds web fonts, writes optional vector SVG, starts from a template pptx, writes `p:timing`, and previews its animations in the browser ([dom-to-pptx](https://github.com/atharva9167j/dom-to-pptx), vendor); its timing templates are a useful XML reference, though its preset names are wrong in places. Anthropic's old `html2pptx` was dropped for direct PptxGenJS scripts, and its pptx skill is proprietary and must not be copied ([Anthropic pptx skill LICENSE](https://github.com/anthropics/skills/blob/main/skills/pptx/LICENSE.txt), vendor). Working raw animation writers exist under permissive licenses (dom-to-pptx, OfficeCLI under Apache-2.0, Aspose FOSS under MIT); svg2ooxml is AGPL or commercial, to be avoided, though its notes are useful (discrete colour changes dropped, `additive="sum"` on motion broken, `p:set` flipbooks work) ([svg2ooxml animation fidelity](https://github.com/BramAlkema/svg2ooxml/blob/main/docs/specs/animation-fidelity.md), practitioner).

### Chart libraries

ECharts and Vega-Lite pass; Chart.js, Recharts, Observable Plot and D3 each fail on one point (question 5). ECharts' SVG output carries a CSS-embedded entry animation, which suits decks' HTML runtime. Its LICENSE notes four files with D3 code under BSD-3, while D3's own repo is now ISC; this is historical relicensing, both permissive, and only means decks ships ECharts' NOTICE.

### CRDTs: Yjs, Automerge, Loro

The fit and license findings are under question 8. Two further points matter for later phases. Hocuspocus's `openDirectConnection` lets a server-side agent write straight into a live Yjs document ([Hocuspocus DirectConnection.ts](https://github.com/ueberdosis/hocuspocus/blob/main/packages/server/src/DirectConnection.ts), vendor), and Automerge's `changeAt`, `view`, `clone`, `merge` and `diff` support branch-style agent edits. Ink & Switch's Patchwork shows bots editing on branches that the user merges in part, with attribution, but domain-specific diff views and comments needed custom work ([Patchwork notebook](https://www.inkandswitch.com/patchwork/notebook/2024-version-control/), independent). Automerge 3 cut memory more than tenfold; Loro offers a last-writer-wins map, a movable list and a movable tree with fractional indexes. None of this is needed in v0, and the model choices under question 8 keep all of it open.

### Blattomat

Blattomat already has most of what spec section 8 asks for. It runs a remote MCP server with OAuth 2.1 (PKCE, refresh, dynamic registration), a Node MCP child process that calls the REST API, 16 tools, warnings on each write (overlap, printable area), a `render_preview` PNG with magenta boxes around block ids, Playwright PDF behind a 60-second HMAC token, OKF docs served as MCP instructions and resources, generated block pages, `okf_check.py` and an eval harness (blattwerk: `src/blattwerk/remote.py`, `oauth.py`, `pdf.py`, `frontend/mcp/tools.ts`, `docs/okf/index.md`, `scripts/eval.py`, practitioner). It chose DOM with moveable and selecto for "Sharp text in the PDF, the same markup renders the PDF". Its weak points for decks: pages are addressed by number, not id; its undo drops agent writes; its agent tools send the whole document, not the editor's operations; `replace_doc` documents no lock check; it has no server push; its tests show WebKit flakes (#225, #932); and its suite carries 27 xfail or skip lines across 19 test files. It also has no LICENSE file (issue #889), which must be settled before its code moves into the public decks repo.

### The presentations plugin

The plugin (MIT, read from its LICENSE file) supplies the stage, the gates and the recipes. `deck-stage.js` already uses the scaled 1920 by 1080 DOM stage, with a `noscale` mode for the exporter. Its hard gates fail text past the slide edge by more than 2 px (H1), text boxes overlapping by more than 4 px (H2), and text below role floors of 36 px for body, 28 for captions and 24 for footers (H7); the floors are "starting points ... not sourced numbers", the probes run in Chromium, and the deck editor reruns them after each edit and lists only new failures (plugin `creating-presentations/references/hard-gates.md`, practitioner). Its presets hold CSS tokens named by role, guideline prose and HTML example slides, with no structured master or layout file, though `extracting-presets` already walks slide to layout to master to theme and reads `txStyles`, placeholder frames and `clrMap`. The real deck corpus shows what decks must carry on day one: 18 deck folders and 43 deck HTML files, inline SVG in 38 of the 43, tables in 13, speaker notes in 35, flex and grid in every deck, shared assets outside each deck folder, and no iframe, video or canvas anywhere (private deck repo, aggregate count). Text, Image, Shape, Table, SVG and Placeholder blocks plus speaker notes cover nearly all of it; the Script and Chart blocks serve future decks, and the import of wrapped decks must carry the shared assets.

## Analysis and Insights

Read together, the eight answers say something none of them says alone: the spec's bets are right, and its data model is a few fields short in places where every mature product learned the same lesson.

**The five tightenings share one cause.** Each comes from a place where the spec stores something by position or by shorthand, and where a second consumer (PowerPoint, a second editor, an agent, a deleted block) later needs it by identity. "A master is a layout" and a bare `slot` name are shorthand; PowerPoint needs `type` and `idx`. Array places and integer `z` are positions; Figma, tldraw and Excalidraw needed fractional keys, and Excalidraw paid for retrofitting them. A string of text is shorthand for runs; concurrent edits and the pptx writer both need the runs. An effect named in prose is shorthand for a `presetID`; three public writers got the numbers wrong, including Microsoft's own sample. A comment pinned to a place on the slide is a position; tldraw, PowerPoint and Google all anchor to object ids. In every case the fix costs a field or two now and a migration later. That is why the report puts them before v0 code, not after.

**The three rejections share a cause too: each would hand control of the model to someone else.** tldraw would hand it to a vendor license and a model with no masters. A two-way text form would make two sources of truth and spend effort keeping them in step, which Slidev Studio shows breaks undo and drops what the text cannot express. A CRDT would merge where decks needs to refuse, since the spec's cascade rules ("deleting a block deletes its animations") are the kind of invariant "CRDTs merge; they do not reject" cannot hold. In each case the alternative's strength (a polished canvas, a readable text, offline merge) is something decks can get more cheaply another way: DOM blocks it already has, a read-only outline tool, and a server that orders operations.

**No shipping product guarantees anything to the user about hand work.** This is the most surprising finding, and the one that most shapes decks' MCP design. Gamma, Pitch, Copilot, Gemini, Plus AI, Figma and Canva all scope edits, and some preview, but acceptance is per request at best, locks are guidance (Copilot) or unverified (Canva), and the one documented clobbering (Gamma, a single report) went unexplained. Users work around it by duplicating slides first (Plus AI's own advice) or by asking in the prompt. decks can do better cheaply because it controls the only write path: a tool that refuses to touch a locked block is a rule, not a request. Blattomat already does this for single blocks; the gap is whole-document writes and undo.

**Everyone who ships live editing on objects runs it through a server.** Figma, tldraw sync, Linear and Google's revision checks all order writes centrally; the CRDT libraries are better suited to text and offline-first apps than to a deck of a few hundred small objects with rules. The conventional view that collaboration means a CRDT gets this wrong for decks. What collaboration needs from v0 is model hygiene, which is the same list as the tightenings.

**Agents want what collaboration wants.** Stable ids, per-property writes, no positional order and no derived data in the document are the same requirements whether the second writer is a person in another tab or a model through MCP. JSON Whisperer's finding that models fail at index arithmetic and ACE's finding that semantic tools by id beat raw HTML are the agent-side form of Figma's "can't have the server assign ids". One model serves both.

**The real decks argue for order of work, not for a smaller model.** The private deck corpus has no timing, no builds, no native charts and one placeholder (private deck repo, aggregate count). That could read as a case for cutting animations, charts and layouts from v0. The scan does not support that reading: the spec puts them in so "no deck has to be rebuilt to gain them", and each costs little in the model. It does argue that the v0 test should rest on text, images, shapes, tables, SVG and notes, which cover nearly all real content, and that animation and chart export get their own checks in PowerPoint.

**For a decision-maker with five minutes:** keep DOM blocks, JSON by id and MCP; drop tldraw, text round trips and CRDTs; before code, add placeholder type and index, fractional order, text runs, a preset-id table and a separate comment store; and close Blattomat's two gaps (locks on whole-document writes, undo for agent writes) when porting it.

## Limitations and Open Problems

The scan has gaps that the spec should carry as open items, not as settled facts.

**Nothing was opened in PowerPoint.** Every pptx claim rests on XML, specs and code. The layout design under question 2 (a generated base file, placeholders inherited through empty `spPr`) is therefore a hypothesis. No one tested Change Layout or Reset on a file from python-pptx or PptxGenJS, and the v0 test ("opens in PowerPoint with editable text") must include that check. The same holds for per-paragraph builds: no PowerPoint-saved `bldP build="p"` sample was found, so that XML rests on ECMA examples and dom-to-pptx. [MS-PPTX] (Morph, `p14:dur`) and MS-ODRAWXML (`svgBlip`) could not be read; the evidence is code that writes them.

**The preset-id table is less certain than "two importers agree" suggests.** LibreOffice's table and Aspose FOSS's mapping agree, but Aspose FOSS is a 2026 project with 15 stars, and the scan did not establish that it built its table independently of LibreOffice's. The firmer corroboration is one PowerPoint-saved Fly In and one COM-observed table. The ids must be tested in PowerPoint before they ship.

**python-pptx is stale, and the scan made no plan for that.** It has had no commits since 2024-08 and has 541 open issues. The report gives moveable an unmaintained-dependency plan but leaves python-pptx's risk unweighed, and power-pptx, the one active fork, is young and unchecked. The spec should decide whether to pin, vendor or fork python-pptx, as it does for moveable.

**Pitch is mostly unknown.** Its editor internals, sync design, AI treatment of hand-placed blocks, previews, block-level comment anchors and pptx layout structure all have no source. Answers to questions 4 and 7 therefore rest on Gamma, Google, Microsoft, Figma and tldraw. Canva's AI and lock behaviour could not be read (403); its claims stay unverified. Copilot's edit mechanics and whether a selected shape reaches the model have no developer source. Canva's and PowerPoint for the web's rendering stayed undetermined.

**Other open points.** No source covers what happens to comments when the commented object is deleted in Figma, Google Slides or PowerPoint; Lucid's behaviour rests on one search summary. No source covers stepping back within a slide in PowerPoint, nor Keynote's pptx mapping of builds, Magic Move or layouts, nor what Google Slides' pptx download keeps. No benchmark of a deck-shaped document exists for CRDTs, and the one suite is run by the Yjs author on old versions; Yjs 14 is not final. No benchmark of models writing Touying or Slidev decks exists. The full text-style lookup order in PresentationML was not found. The JSON-against-text token figure is one slide, one tokenizer and an invented format. Several claims rest on search summaries only, and each is marked where it appears. LibreOffice's license and the commercial Spire and Aspose licenses were not read. Blattomat's lock refusal was traced through docs and tests, not code. Two papers on LLM agents over CRDTs could not be retrieved and were not used.

**What the scan cannot solve.** It cannot say how Pascal's next deck will use animations or charts, since none of the existing ones do; it cannot settle Blattomat's license, which is Pascal's call; and and it cannot make LibreOffice stand in for PowerPoint on layouts or animations: LibreOffice disagrees with PowerPoint on placeholder inheritance, has 27 open animation bugs, and the scan ran no PowerPoint check.

## Future Outlook

These are the report's own predictions [author estimate], each tied to a fact in the scan and each checkable.

1. By 2027-04, daybrush will have published no new `moveable` release, so the last one will still date from 2023-12-03. Issue #1156 offered maintenance on 2026-09-16 with no answer; if a community fork ships instead, decks should judge it then, and otherwise treat its vendored copy as its own code.
2. By 2027-10, python-pptx will still have no release after 1.0.2, given no commits since 2024-08 and a steady 60 million downloads a month that have not prompted one. If this holds, the question of pinning or forking becomes urgent.
3. Within 2027, the Google Slides comments API will leave developer preview with object-id anchors intact, since its MCP comment tools already shipped on 2026-10-01 on top of it. That would make object-id anchoring the shared pattern of Google, Microsoft and tldraw, and decks' design would match it.
4. By 2027-06, Yjs 14 will have a final release, as it has already run to rc.29. This does not change the v0 advice, which leaves a Yjs mapping open either way.

## Conclusions and Practical Starting Point

The scan supports the spec's core and asks for five changes. Keep each block as an HTML element on a 1920 by 1080 slide, keep one typed JSON document addressed by id, keep MCP as the agent's only write path, and build the pptx exporter as a python-pptx program from the plugin's recipes. Reject tldraw (license and model), a two-way text form (no product makes it work), and a CRDT now (server-ordered sync is the norm and decks has rules a CRDT cannot enforce). Before v0 code: split master from layout with placeholder `type` and `idx`; replace array order and integer `z` with fractional-index strings; store text as structured runs; key animations to the verified preset-id table, with per-paragraph builds and `p14:dur` transitions; and keep comments, selection and drawings beside the deck, anchored to block ids with a fallback on delete.

By impact, the actions are:

1. **Model changes in the spec** (questions 2, 6, 8): layout and slot fields, `order` strings, text runs, a closed effect list with preset ids, a direction field, `by: "paragraph"`, a schema version field. Cheap now, costly later.
2. **Guards in the MCP tools** (question 4): lock refusal on every write including whole-slide and whole-deck ones, version checks, every agent write a single undo step with a summary.
3. **Comment store and tools** (question 7): own table, slide-plus-block anchors with normalized offset, delete fallback, list, reply, resolve, reopen, `get_selection`.
4. **Export plan** (question 5): python-pptx plus decks' own OOXML writers, Chromium for rasterizing in place of PyMuPDF and CairoSVG, one table-shaped Chart block with ECharts.
5. **Port hygiene** (questions 1 and 8): moveable behind one module, Blattomat's undo and `replace_doc` gaps closed, Blattomat's license settled.

### Starting from zero: the first spec edits and issues

The template's usual scenario (an organization with pilots and no baselines) does not fit a one-person project; the realistic start here is a draft spec and two codebases to port from. The first steps, in order:

1. Edit `docs/spec-v0.md`: replace "A master is a layout" with the master, layout and slot fields from question 2; replace array order and `z` with `order` strings; state that text is stored as runs; add the effect table, direction and `by: "paragraph"`; move comments out of the deck.
2. Settle Blattomat's license (blattwerk issue #889) before any of its code moves.
3. Open the first build issues in the order the v0 test needs them: the canvas port with moveable behind one module; the JSON schema with ids and `order`; MCP tools with lock refusal and versions; the read-only outline tool; the pptx exporter for text, images, shapes, tables, SVG and notes.
4. Open one test issue: generate a pptx with a base file of real layouts, open it in PowerPoint, and try Change Layout and Reset. Its result confirms or breaks the question 2 design.
5. Leave animation export, native charts and the comment store as later v0 issues, each with its own PowerPoint check, since no existing deck depends on them.

## License Table

Every row comes from reading the LICENSE (or COPYING) file directly, except the two commercial products, whose terms come from their pricing pages. "Flag" says what decks must do or watch.

| Component | Role for decks | License as read | Flag | Source |
|---|---|---|---|---|
| tldraw, @tldraw/editor, sync, sync-core, commenting, collaboration, mentions | rejected canvas option | tldraw license: development only; production needs a trial or commercial key; usage telemetry | proprietary | https://raw.githubusercontent.com/tldraw/tldraw/main/LICENSE.md and packages/*/LICENSE.md (vendor) |
| @tldraw/store, state, tlschema, validate, utils | reference only | MIT (2024 tldraw Inc.) | none | tldraw packages/*/LICENSE.md (vendor) |
| moveable, react-moveable | canvas (port) | MIT (2019 Daybrush) | no release since 2023-12 | https://raw.githubusercontent.com/daybrush/moveable/master/LICENSE (vendor) |
| selecto, react-selecto | canvas (port) | MIT (2020 Daybrush) | no release since 2023-12 | https://raw.githubusercontent.com/daybrush/selecto/master/LICENSE (vendor) |
| interact.js | alternative | MIT (Taye Adeyemi) | none | https://raw.githubusercontent.com/taye/interact.js/main/LICENSE (vendor) |
| Konva, react-konva | alternative (canvas) | MIT | none | https://raw.githubusercontent.com/konvajs/konva/master/LICENSE (vendor) |
| Fabric.js | alternative (canvas) | MIT | none | https://raw.githubusercontent.com/fabricjs/fabric.js/master/LICENSE (vendor) |
| Excalidraw | reference | MIT (2020) | none | https://raw.githubusercontent.com/excalidraw/excalidraw/master/LICENSE (vendor) |
| ProseMirror (model, state, view, commands, keymap) | text field (port) | MIT (Marijn Haverbeke) | none | https://raw.githubusercontent.com/ProseMirror/prosemirror-view/master/LICENSE (vendor) |
| React, react-router | frontend | MIT | none | local license read (vendor) |
| lucide | icons | ISC; Feather icons MIT | none | https://raw.githubusercontent.com/lucide-icons/lucide/main/LICENSE (vendor) |
| modern-screenshot | DOM to image | MIT | none | local license read (vendor) |
| MCP TypeScript SDK, mcpb | MCP server | moving from MIT to Apache-2.0 | note both | https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/LICENSE (vendor) |
| MCP Python SDK | MCP | MIT (2024 Anthropic) | none | local license read (vendor) |
| FastAPI | backend | MIT | none | local license read (vendor) |
| uvicorn, httpx, pypdf, Python-Markdown | backend | BSD-3-style | no-endorsement clause | local license read (vendor) |
| playwright-python | PDF, rendering | Apache-2.0 | NOTICE duties | local license read (vendor) |
| argon2-cffi, jsonschema | backend | MIT | none | local license read (vendor) |
| python-pptx | pptx export | MIT (2013 Steve Canny) | no commits since 2024-08 | https://github.com/scanny/python-pptx/blob/master/LICENSE (vendor) |
| power-pptx | candidate fork | MIT | young, claims unchecked | https://github.com/codehalwell/power-pptx/blob/master/LICENSE (practitioner) |
| PptxGenJS | alternative | MIT (Brent Ely) | none | https://github.com/gitbrent/PptxGenJS/blob/master/LICENSE (vendor) |
| pptx-automizer | alternative | MIT | none | https://github.com/singerla/pptx-automizer/blob/main/LICENSE (vendor) |
| officegen | rejected | MIT | dead since 2021 | https://github.com/Ziv-Barber/officegen/blob/master/LICENSE (vendor) |
| pptx-embed-fonts | fonts | MIT | tiny project | https://github.com/liyao1520/pptx-embed-fonts/blob/main/LICENSE (practitioner) |
| dom-to-pptx | reference (timing XML) | MIT | none | https://github.com/atharva9167j/dom-to-pptx/blob/master/LICENSE (vendor) |
| Aspose.Slides FOSS for Python | reference (preset ids) | MIT (2026 Aspose) | none | https://github.com/aspose-slides-foss/Aspose.Slides-FOSS-for-Python/blob/main/LICENSE (vendor) |
| OfficeCLI, ChristopherVR/ooxml | reference | Apache-2.0 | NOTICE if copied | repo LICENSE files (vendor) |
| svg2ooxml | avoid | AGPL-3.0 or commercial | copyleft | https://github.com/BramAlkema/svg2ooxml/blob/main/LICENSE (vendor) |
| Resource2Skill | reference only | MIT (Microsoft) | preset ids wrong | https://github.com/microsoft/Resource2Skill/blob/main/LICENSE (vendor) |
| Open XML SDK | not in stack | MIT | .NET | https://github.com/dotnet/Open-XML-SDK/blob/main/LICENSE (vendor) |
| Aspose.Slides | paid comparison | commercial, from $1,399 | paid | https://purchase.aspose.com/pricing/slides/family/ (vendor) |
| Syncfusion Presentation | paid comparison | commercial; Community License limits | paid | https://www.syncfusion.com/products/communitylicense (vendor) |
| Anthropic pptx skill | do not copy | proprietary, all rights reserved | proprietary | https://github.com/anthropics/skills/blob/main/skills/pptx/LICENSE.txt (vendor) |
| PyMuPDF | current exporter's rasterizer | AGPL-3.0 or commercial (Artifex) | copyleft; replace | https://raw.githubusercontent.com/pymupdf/PyMuPDF/main/COPYING (vendor) |
| CairoSVG | current exporter's fallback | LGPL-3.0 | weak copyleft; replace | https://raw.githubusercontent.com/Kozea/CairoSVG/main/LICENSE (vendor) |
| Pillow | exporter | MIT-CMU | none | https://raw.githubusercontent.com/python-pillow/Pillow/main/LICENSE (vendor) |
| Apache ECharts | Chart block | Apache-2.0 with NOTICE; D3 parts BSD-3 | ship NOTICE | https://github.com/apache/echarts/blob/master/LICENSE and NOTICE (vendor) |
| zrender | ECharts dependency | BSD-3 (Baidu) | no-endorsement clause | https://github.com/ecomfe/zrender/blob/master/LICENSE (vendor) |
| Vega-Lite, Vega, vl-convert | chart alternative | BSD-3 | no-endorsement clause | vega repos LICENSE (vendor) |
| Chart.js | rejected (canvas only) | MIT | none | https://github.com/chartjs/Chart.js/blob/master/LICENSE.md (vendor) |
| Observable Plot, D3 | rejected or low level | ISC | none | observablehq/plot and d3/d3 LICENSE (vendor) |
| Recharts | rejected (no server rendering) | MIT | none | https://github.com/recharts/recharts/blob/main/LICENSE (vendor) |
| Slidev, slidev-addon-studio, @vueuse/motion | reference | MIT | none | slidevjs/slidev, BobTheShoplifter/slidev-addon-studio, vueuse/motion LICENSE (vendor) |
| Marp CLI, Marp Core, Marpit, marp-mcp | reference | MIT | none | marp-team repos, masaki39/marp-mcp LICENSE (vendor) |
| reveal.js | reference | MIT | none | https://github.com/hakimel/reveal.js/blob/master/LICENSE (vendor) |
| Quarto CLI | reference | MIT (COPYING.md) | GitHub shows NOASSERTION | https://github.com/quarto-dev/quarto-cli/blob/main/COPYING.md (vendor) |
| Spectacle | reference | MIT | none | https://github.com/FormidableLabs/spectacle/blob/main/LICENSE (vendor) |
| Typst | rejected text form | Apache-2.0 | none | https://github.com/typst/typst/blob/main/LICENSE (vendor) |
| Touying, touying-exporter, typ2pptx, SeaSlides | rejected text form | MIT | none | touying-typ repos LICENSE (vendor) |
| Tylina | reference | proprietary, all rights reserved | proprietary | https://github.com/tylina/tylina.github.io/blob/main/LICENSE (vendor) |
| Yjs, y-prosemirror, y-websocket, y-utility, Yrs, pycrdt, pycrdt-websocket | later collaboration | MIT | none | https://github.com/yjs/yjs/blob/main/LICENSE and sibling repos (vendor) |
| Hocuspocus, y-sweet | later collaboration | MIT | none | repo LICENSE files (vendor) |
| Automerge, automerge-repo, automerge-py | later collaboration | MIT | none | https://github.com/automerge/automerge/blob/main/LICENSE (vendor) |
| Loro | later collaboration | MIT | none | https://github.com/loro-dev/loro/blob/main/LICENSE (vendor) |
| fractional-indexing (rocicorp) | order keys, now | CC0 1.0 | none | https://github.com/rocicorp/fractional-indexing/blob/main/LICENSE (vendor) |
| react-grab | reference | MIT | none | https://github.com/aidenybai/react-grab/blob/main/LICENSE (vendor) |
| presentations plugin (pgoell-claude-tools) | port source | MIT (2026 Pascal Göllner) | none | pgoell-claude-tools: LICENSE (practitioner) |
| Blattomat | port source | no LICENSE file | settle before porting (issue #889) | https://github.com/pgoell/blattwerk/issues/889 (practitioner) |

Several of these repos show `NOASSERTION` on GitHub only because of file headers; the text read is as stated.

## References

All sources accessed 2026-10-20. Tags: independent, vendor, practitioner, journalism; "vendor (competitor)" for a rival writing about a product. Local sources are named by repository and path.

### tldraw

| Title | URL | Tag | Accessed |
|---|---|---|---|
| tldraw license (LICENSE.md, main) | https://raw.githubusercontent.com/tldraw/tldraw/main/LICENSE.md | vendor | 2026-10-20 |
| tldraw package LICENSE files | https://raw.githubusercontent.com/tldraw/tldraw/main/packages/store/LICENSE.md (and siblings) | vendor | 2026-10-20 |
| tldraw license, 2023-12-19 | https://raw.githubusercontent.com/tldraw/tldraw/3cf4dae34d/LICENSE.md | vendor | 2026-10-20 |
| tldraw license before 4.0 | https://raw.githubusercontent.com/tldraw/tldraw/e455ab838b~1/LICENSE.md | vendor | 2026-10-20 |
| Announcing tldraw SDK 4.0 | https://tldraw.dev/blog/tldraw-sdk-4-0 | vendor | 2026-10-20 |
| tldraw SDK 5.0 | https://tldraw.dev/blog/tldraw-sdk-5-0 | vendor | 2026-10-20 |
| tldraw license page | https://tldraw.dev/community/license | vendor | 2026-10-20 |
| tldraw pricing (live) | https://tldraw.dev/pricing | vendor | 2026-10-20 |
| tldraw pricing, archived 2025-10-10 | http://web.archive.org/web/20251010182142/https://tldraw.dev/pricing | vendor | 2026-10-20 |
| tldraw pricing, archived 2026-08-28 | http://web.archive.org/web/20260828114609/https://tldraw.dev/pricing | vendor | 2026-10-20 |
| 20 things I wish AI chatbots knew about tldraw (Ruiz) | https://tldraw.dev/blog/20-things-i-wish-ai-chatbots-knew-about-tldraw | vendor | 2026-10-20 |
| tldraw docs, full text | https://tldraw.dev/llms-full.txt | vendor | 2026-10-20 |
| tldraw editor docs | https://tldraw.dev/docs/editor | vendor | 2026-10-20 |
| tldraw shapes docs | https://tldraw.dev/docs/shapes | vendor | 2026-10-20 |
| tldraw slides example | https://github.com/tldraw/tldraw/tree/main/apps/examples/src/examples/use-cases/slides | vendor | 2026-10-20 |
| Building with tldraw offline | https://tldraw.dev/blog/building-with-tldraw-offline | vendor | 2026-10-20 |
| tldraw AI docs | https://tldraw.dev/docs/ai | vendor | 2026-10-20 |
| tldraw agent starter kit | https://tldraw.dev/starter-kits/agent | vendor | 2026-10-20 |
| tldraw agent format | https://github.com/tldraw/tldraw/tree/main/templates/agent/shared/format | vendor | 2026-10-20 |
| A comment is not a shape | https://tldraw.dev/blog/a-comment-is-not-a-shape | vendor | 2026-10-20 |
| TLComment.ts | https://github.com/tldraw/tldraw/blob/main/packages/tlschema/src/records/TLComment.ts | vendor | 2026-10-20 |
| anchor-lifecycle.ts | https://github.com/tldraw/tldraw/blob/main/packages/commenting/src/canvas/anchor-lifecycle.ts | vendor | 2026-10-20 |
| tldraw comment anchors example | https://tldraw.dev/examples/comment-anchors | vendor | 2026-10-20 |
| tldraw v5.3.0 release | https://tldraw.dev/releases/v5.3.0 | vendor | 2026-10-20 |
| tldraw commenting LICENSE | https://github.com/tldraw/tldraw/blob/main/packages/commenting/LICENSE.md | vendor | 2026-10-20 |
| Agents can't point | https://tldraw.dev/blog/agents-cant-point | vendor | 2026-10-20 |
| tldraw sync-core SPEC.md | https://github.com/tldraw/tldraw/blob/main/packages/sync-core/SPEC.md | vendor | 2026-10-20 |
| tldraw sync docs | https://tldraw.dev/docs/sync | vendor | 2026-10-20 |
| Announcing tldraw sync | https://tldraw.dev/blog/announcing-tldraw-sync | vendor | 2026-10-20 |
| tldraw shape indexing (search snippet) | https://tldraw.dev/sdk-features/shape-indexing | vendor | 2026-10-20 |
| Tldraw SDK 4.0, Hacker News thread | https://news.ycombinator.com/item?id=45293839 | practitioner | 2026-10-20 |
| tldraw SDK 4.0 licensing debate | https://biggo.com/news/202509190115_tldraw_SDK_4.0_Licensing_Debate | journalism | 2026-10-20 |
| Syntax 505 (search summary) | https://syntax.fm/505 | practitioner | 2026-10-20 |

### Canvas rendering and drag libraries

| Title | URL | Tag | Accessed |
|---|---|---|---|
| First-hand DOM and network inspection: Pitch player, Google Slides, PowerPoint web viewer | (own inspection, 2026-10-20) | practitioner | 2026-10-20 |
| Building elegant interfaces with ClojureScript, React and UIx | https://building.nubank.com/building-elegant-interfaces-with-clojurescript-react-and-uix/ | practitioner | 2026-10-20 |
| UIx | https://github.com/pitch-io/uix | vendor | 2026-10-20 |
| Pitch frontend performance engineer job ad | https://startup.jobs/frontend-performance-engineer-mfd-at-pitch | vendor | 2026-10-20 |
| Google Docs canvas-based rendering update | https://workspaceupdates.googleblog.com/2021/05/Google-Docs-Canvas-Based-Rendering-Update.html | vendor | 2026-10-20 |
| Google Docs switches to canvas rendering | https://thenewstack.io/google-docs-switches-to-canvas-rendering-sidelining-the-dom/ | journalism | 2026-10-20 |
| Building Figma Slides with Noah Finer | https://newsletter.pragmaticengineer.com/p/building-figma-slides-with-noah-finer | journalism | 2026-10-20 |
| Figma rendering powered by WebGPU | https://www.figma.com/blog/figma-rendering-powered-by-webgpu/ | vendor | 2026-10-20 |
| Canva Lightspeed renderer job ad | https://jobs.blackbird.vc/companies/canva/jobs/72657275-lightspeed-renderer-senior-software-engineer | vendor | 2026-10-20 |
| Canva interview guide (search summary) | https://www.techinterview.org/companies/canva/ | practitioner | 2026-10-20 |
| Konva editable text | https://konvajs.org/docs/sandbox/Editable_Text.html | vendor | 2026-10-20 |
| Konva canvas to PDF (search summary) | https://konvajs.org/docs/sandbox/Canvas_to_PDF.html | vendor | 2026-10-20 |
| W3C public-canvas-api thread, 2010 (search summary) | https://lists.w3.org/Archives/Public/public-canvas-api/2010JulSep/0028.html | practitioner | 2026-10-20 |
| Build my own text editor (search summary) | https://web-standards.dev/news/2026/09/build-my-own-text-editor/ | practitioner | 2026-10-20 |
| Building a production-ready Canva-like editor with Konva | https://geekyants.com/blog/building-a-production-ready-canva-like-editor-with-konva-js-react-19-and-next-js-15 | practitioner | 2026-10-20 |
| moveable LICENSE | https://raw.githubusercontent.com/daybrush/moveable/master/LICENSE | vendor | 2026-10-20 |
| selecto LICENSE | https://raw.githubusercontent.com/daybrush/selecto/master/LICENSE | vendor | 2026-10-20 |
| moveable #1137, Is this project still active? | https://github.com/daybrush/moveable/issues/1137 | practitioner | 2026-10-20 |
| moveable #1156, maintainership offer | https://github.com/daybrush/moveable/issues/1156 | practitioner | 2026-10-20 |
| interact.js LICENSE | https://raw.githubusercontent.com/taye/interact.js/main/LICENSE | vendor | 2026-10-20 |
| Konva LICENSE | https://raw.githubusercontent.com/konvajs/konva/master/LICENSE | vendor | 2026-10-20 |
| react-konva LICENSE | https://raw.githubusercontent.com/konvajs/react-konva/master/LICENSE | vendor | 2026-10-20 |
| Fabric.js LICENSE | https://raw.githubusercontent.com/fabricjs/fabric.js/master/LICENSE | vendor | 2026-10-20 |
| Excalidraw LICENSE | https://raw.githubusercontent.com/excalidraw/excalidraw/master/LICENSE | vendor | 2026-10-20 |
| npm, PyPI and GitHub API registry metadata (moveable, selecto, frameable, python-pptx, PptxGenJS and others) | (registry queries) | vendor | 2026-10-20 |

### Masters, layouts and OOXML

| Title | URL | Tag | Accessed |
|---|---|---|---|
| ECMA-376 sldMaster | https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/sldMaster.html | independent | 2026-10-20 |
| ECMA-376 sldLayout | https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/sldLayout.html | independent | 2026-10-20 |
| ECMA-376 ST_SlideLayoutType | https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/ST_SlideLayoutType.html | independent | 2026-10-20 |
| ECMA-376 ph | https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/ph.html | independent | 2026-10-20 |
| ECMA-376 txStyles, bodyStyle, ST_SlideLayoutId, ST_SlideMasterId, clrMap | https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/txStyles.html (and siblings) | independent | 2026-10-20 |
| ECMA-376 clrScheme | https://webapp.docx4java.org/OnlineDemo/ecma376/DrawingML/clrScheme.html | independent | 2026-10-20 |
| Working with slide masters (Open XML SDK) | https://learn.microsoft.com/en-us/office/open-xml/presentation/working-with-slide-masters | vendor | 2026-10-20 |
| Working with slide layouts (Open XML SDK) | https://learn.microsoft.com/en-us/office/open-xml/presentation/working-with-slide-layouts | vendor | 2026-10-20 |
| python-pptx: understanding placeholders | https://python-pptx.readthedocs.io/en/latest/user/placeholders-understanding.html | practitioner | 2026-10-20 |
| python-pptx: layout placeholders analysis | https://python-pptx.readthedocs.io/en/latest/dev/analysis/placeholders/layout-placeholders.html | practitioner | 2026-10-20 |
| python-pptx: slide placeholders analysis | https://python-pptx.readthedocs.io/en/latest/dev/analysis/placeholders/slide-placeholders/index.html | practitioner | 2026-10-20 |
| python-pptx: placeholders in new slide | https://python-pptx.readthedocs.io/en/stable/dev/analysis/placeholders/slide-placeholders/placeholders-in-new-slide.html | practitioner | 2026-10-20 |
| python-pptx: concepts | https://python-pptx.readthedocs.io/en/latest/user/concepts.html | vendor | 2026-10-20 |
| python-pptx #413, add layouts | https://github.com/scanny/python-pptx/issues/413 | practitioner | 2026-10-20 |
| python-pptx #1109 | https://github.com/scanny/python-pptx/issues/1109 | practitioner | 2026-10-20 |
| python-pptx #175 | https://github.com/scanny/python-pptx/issues/175 | practitioner | 2026-10-20 |
| python-pptx #864 | https://github.com/scanny/python-pptx/issues/864 | practitioner | 2026-10-20 |
| What is a slide master? | https://support.microsoft.com/en-us/office/what-is-a-slide-master-b9abb2a0-7aef-4257-a14e-4329c904da54 | vendor | 2026-10-20 |
| Reset slide (Indezine) | https://www.indezine.com/products/powerpoint/learn/interface/2013/reset-slide.html | practitioner | 2026-10-20 |
| Placeholder order across layouts (Microsoft Q&A) | https://learn.microsoft.com/en-us/answers/a/11388916 | practitioner | 2026-10-20 |
| LibreOffice bug 104202 | https://bugs.documentfoundation.org/rest/bug/104202/comment | independent | 2026-10-20 |
| LibreOffice bug 128547 | https://bugs.documentfoundation.org/rest/bug/128547/comment | independent | 2026-10-20 |
| Accessibility in presentations: slide layouts (Elgin) | https://elgin.edu/accessibility/accessibility-in-presentations/slide-layouts/ | independent | 2026-10-20 |
| Text placeholders vs text boxes (Indezine) | https://www.indezine.com/products/powerpoint/learn/interface/365/text-placeholders-vs-boxes.html | practitioner | 2026-10-20 |
| WebAIM discussion (search summary) | https://webaim.org/discussion/mail_message?id=41715 | practitioner | 2026-10-20 |
| Scratchpad python-pptx and PptxGenJS 4.0.1 experiments (XML reads) | (local experiment) | practitioner | 2026-10-20 |

### Google Slides

| Title | URL | Tag | Accessed |
|---|---|---|---|
| Slides API: presentations.pages | https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages | vendor | 2026-10-20 |
| Slides API: pages, other | https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages/other | vendor | 2026-10-20 |
| Slides API: pages, shapes | https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages/shapes | vendor | 2026-10-20 |
| Slides API: page elements | https://developers.google.com/workspace/slides/api/concepts/page-elements | vendor | 2026-10-20 |
| Slides API: requests | https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/request | vendor | 2026-10-20 |
| Slides API: create a slide | https://developers.google.com/workspace/slides/api/guides/create-slide | vendor | 2026-10-20 |
| Slides API: batch | https://developers.google.com/workspace/slides/api/guides/batch | vendor | 2026-10-20 |
| Slides API: batchUpdate | https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/batchUpdate | vendor | 2026-10-20 |
| Slides API: add a chart | https://developers.google.com/workspace/slides/api/guides/add-chart | vendor | 2026-10-20 |
| Slides API: configure MCP server | https://developers.google.com/workspace/slides/api/guides/configure-mcp-server | vendor | 2026-10-20 |
| Slides API: comments | https://developers.google.com/workspace/slides/api/guides/comments | vendor | 2026-10-20 |
| Slides API: CommentThread | https://developers.google.com/workspace/slides/api/reference/rest/v1/CommentThread | vendor | 2026-10-20 |
| Apps Script: Slide | https://developers.google.com/apps-script/reference/slides/slide | vendor | 2026-10-20 |
| Apps Script: Presentation | https://developers.google.com/apps-script/reference/slides/presentation | vendor | 2026-10-20 |
| Edit themes in Google Slides | https://support.google.com/docs/answer/1705254 | vendor | 2026-10-20 |
| Editing themes and layouts in Google Slides (BrightCarbon) | https://www.brightcarbon.com/blog/editing-themes-and-layouts-in-google-slides/ | practitioner | 2026-10-20 |
| Animate in Google Slides | https://support.google.com/docs/answer/1689475?hl=en | vendor | 2026-10-20 |
| How to add animation to Google Slides | https://helpdeskgeek.com/how-to-add-animation-to-google-slides/ | practitioner | 2026-10-20 |
| Creator Studio (Workspace Marketplace) | https://workspace.google.com/marketplace/app/creator_studio/509621243108 | vendor | 2026-10-20 |
| Drive API: manage comments | https://developers.google.com/workspace/drive/api/guides/manage-comments | vendor | 2026-10-20 |
| Drive API: comments resource | https://developers.google.com/workspace/drive/api/reference/rest/v3/comments | vendor | 2026-10-20 |
| Programmatic comment and suggestion (Workspace Updates, 2026-09) | https://workspaceupdates.googleblog.com/2026/09/programmatic-comment-and-suggestion.html | vendor | 2026-10-20 |
| releases.sh tracker | https://releases.sh/sources/src_7aLBPByE36fG9UEb8vh_n | practitioner | 2026-10-20 |
| Comments in Google Docs help | https://support.google.com/docs/answer/65129 | vendor | 2026-10-20 |
| Gemini in Google Slides announcement | https://workspace.google.com/blog/product-announcements/create-on-brand-presentations-in-minutes-with-gemini-in-google-slides | vendor | 2026-10-20 |
| Gemini in Slides help | https://support.google.com/docs/answer/16961475?hl=en | vendor | 2026-10-20 |
| What's different about the new Google Docs (2010) | https://drive.googleblog.com/2010/09/whats-different-about-new-google-docs_22.html | vendor | 2026-10-20 |

### Pitch

| Title | URL | Tag | Accessed |
|---|---|---|---|
| Create your own slide style | https://help.pitch.com/en/articles/4059534-create-your-own-slide-style | vendor | 2026-10-20 |
| Work with slide styles | https://help.pitch.com/en/articles/4063049-work-with-slide-styles | vendor | 2026-10-20 |
| Add slides using layouts | https://help.pitch.com/en/articles/7003075-add-slides-using-layouts | vendor | 2026-10-20 |
| Find and use templates | https://help.pitch.com/en/articles/4056913-find-and-use-templates | vendor | 2026-10-20 |
| Export a presentation to PowerPoint | https://help.pitch.com/en/articles/6713988-export-a-presentation-to-power-point | vendor | 2026-10-20 |
| Pitch: what's new | https://pitch.com/whats-new | vendor | 2026-10-20 |
| Pitch's AI actions (archived 2026-08-11) | https://web.archive.org/web/20260811205908/https://help.pitch.com/en/articles/12771746-pitch-s-ai-actions | vendor | 2026-10-20 |
| Pitch Agent | https://help.pitch.com/en/articles/14981091-pitch-agent | vendor | 2026-10-20 |
| AI in Pitch | https://pitch.com/ai-in-pitch | vendor | 2026-10-20 |
| Introducing Pitch Agent | https://pitch.com/blog/introducing-pitch-agent | vendor | 2026-10-20 |
| Create a new presentation | https://help.pitch.com/en/articles/14844265-create-a-new-presentation | vendor | 2026-10-20 |
| AI presentation maker | https://pitch.com/use-cases/ai-presentation-maker | vendor | 2026-10-20 |
| Add and edit images with AI | https://help.pitch.com/en/articles/15197157-add-and-edit-images-with-ai | vendor | 2026-10-20 |
| An overview of Pitch's MCP | https://help.pitch.com/en/articles/16849368-an-overview-of-pitch-s-mcp | vendor | 2026-10-20 |
| What's possible with Pitch and Claude | https://help.pitch.com/en/articles/16221995-what-s-possible-with-pitch-and-claude | vendor | 2026-10-20 |
| Collaborate with comments | https://help.pitch.com/en/articles/4318672-collaborate-with-comments | vendor | 2026-10-20 |
| Review presentations on mobile | https://help.pitch.com/en/articles/5695595-review-presentations-on-mobile | vendor | 2026-10-20 |
| Set a status and assignees for a slide | https://help.pitch.com/en/articles/4318642-set-a-status-and-assignees-for-a-slide | vendor | 2026-10-20 |
| About the Commenter role | https://help.pitch.com/en/articles/8102572-about-the-commenter-role | vendor | 2026-10-20 |
| Pitch help sitemap | https://help.pitch.com/sitemap.xml | vendor | 2026-10-20 |
| Developing for the desktop: Pitch | https://www.git-tower.com/blog/developing-for-the-desktop-pitch/ | vendor | 2026-10-20 |
| Hacker News comment on Pitch | https://news.ycombinator.com/item?id=24836676 | practitioner | 2026-10-20 |
| Pitch app review (Plus AI) | https://plusai.com/blog/pitch-app-review | vendor (competitor) | 2026-10-20 |

### Gamma

| Title | URL | Tag | Accessed |
|---|---|---|---|
| Introducing Gamma 5 (archived 2026-10-10) | https://web.archive.org/web/20261010014412/https://gamma.app/insights/introducing-gamma-5-your-story-your-style-your-tools | vendor | 2026-10-20 |
| How to edit content using AI | https://help.gamma.app/en/articles/8033284-how-to-edit-content-using-ai | vendor | 2026-10-20 |
| Can I edit my content using AI? | https://help.gamma.app/en/articles/8033284-can-i-edit-my-content-using-ai | vendor | 2026-10-20 |
| Can I undo a change or restore a previous version? | https://help.gamma.app/en/articles/11048579-can-i-undo-a-change-or-restore-a-previous-version-in-gamma | vendor | 2026-10-20 |
| What are cards in Gamma? | https://help.gamma.app/en/articles/11016396-what-are-cards-in-gamma-and-how-do-they-work | vendor | 2026-10-20 |
| How do I change my Gamma theme? | https://help.gamma.app/en/articles/10262646-how-do-i-change-my-gamma-theme | vendor | 2026-10-20 |
| Why doesn't my exported PDF or PowerPoint match? | https://help.gamma.app/en/articles/15939201-why-doesnt-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma | vendor | 2026-10-20 |
| Gamma developer docs, full text | https://developers.gamma.app/llms-full.txt | vendor | 2026-10-20 |
| Gamma generate API parameters | https://developers.gamma.app/docs/generate-api-parameters-explained | vendor | 2026-10-20 |
| Gamma common feature requests | https://developers.gamma.app/reference/common-feature-requests | vendor | 2026-10-20 |
| Gamma ideas: AI clobbered my edits | https://ideas.gamma.app/ideas/p/gamma-ai-clobbered-my-edits | practitioner | 2026-10-20 |
| Gamma ideas: add edit support to MCP | https://ideas.gamma.app/ideas/p/add-update-edit-support-to-gamma-mcp-not-just-generation | practitioner | 2026-10-20 |
| Gamma ideas: fully editable PowerPoint export | https://ideas.gamma.app/ideas/p/fully-editable-powerpoint-export-title-subtitle-all-text-graphics-speaker-notes | practitioner | 2026-10-20 |
| Gamma ideas: native PowerPoint chart exports | https://ideas.gamma.app/ideas/p/native-powerpoint-chart-exports-dont-exported-as-images | practitioner | 2026-10-20 |
| r/GammaApp thread 1x6a2yi | https://reddit.com/r/GammaApp/comments/1x6a2yi/ | practitioner | 2026-10-20 |
| r/GammaApp thread 1x4xmqb | https://reddit.com/r/GammaApp/comments/1x4xmqb/ | practitioner | 2026-10-20 |
| r/GammaApp thread 1wpmgsm | https://reddit.com/r/GammaApp/comments/1wpmgsm/ | practitioner (weight lowered: unverified sponsorship claim) | 2026-10-20 |
| r/powerpoint thread 1uyp4w8 | https://reddit.com/r/powerpoint/comments/1uyp4w8/ | practitioner | 2026-10-20 |
| Gamma 5 (BornCity) | https://borncity.com/news/gamma-5-gamma-tech-stellt-den-groessten-umbau-seiner-praesentationsplattform-vor/ | journalism | 2026-10-20 |
| Gamma 5 rebuild guide | https://rar.design/posts/gamma-5-ai-presentation-rebuild-guide | practitioner | 2026-10-20 |
| Gamma 5: what changed | https://explainx.ai/blog/gamma-5-agent-driven-presentations-ai-smell-what-changed-2026 | practitioner | 2026-10-20 |
| Fix Gamma exported decks in WPS | https://www.wps.com/blog/gamma-ppt-fix-exported-decks-in-wps-presentation/ | vendor (competitor) | 2026-10-20 |

### Keynote, Figma, Copilot, other AI tools

| Title | URL | Tag | Accessed |
|---|---|---|---|
| Keynote: add and edit master slides | https://support.apple.com/guide/keynote/add-and-edit-master-slides-tan7a2b69972/mac | vendor | 2026-10-20 |
| Keynote: change the slide layout or theme | https://support.apple.com/guide/keynote/change-the-slide-layout-or-theme-tan584189747/mac | vendor | 2026-10-20 |
| Keynote 9 help (search summary) | https://help.apple.com/keynote/mac/9.0/en.lproj/tan584189747.html | vendor | 2026-10-20 |
| Keynote: animate objects | https://support.apple.com/en-sa/guide/keynote/tan72234bb6/mac | vendor | 2026-10-20 |
| Keynote: builds | https://support.apple.com/guide/keynote/tanf96d92cb6/mac | vendor | 2026-10-20 |
| Keynote: action builds | https://support.apple.com/guide/keynote/tan3ad5f8d82/mac | vendor | 2026-10-20 |
| Keynote: Magic Move | https://support.apple.com/en-ph/guide/keynote/tanff5ae749e/mac | vendor | 2026-10-20 |
| Apple Discussions thread 254026886 | https://origin-discussions2-us-dr-prz.apple.com/thread/254026886 | practitioner | 2026-10-20 |
| Keynote: export to other formats | https://support.apple.com/guide/keynote/tana0d19882a/mac | vendor | 2026-10-20 |
| Apple HT5591 (archived) | http://web.archive.org/web/20141016150659/http://support.apple.com/kb/HT5591 | vendor | 2026-10-20 |
| Keynote export format guide | https://keynote.org.cn/blog/keynote-export-format-guide-en/ | practitioner | 2026-10-20 |
| Apple Discussions thread 250144385 | https://discussions.apple.com/thread/250144385 | practitioner | 2026-10-20 |
| iWork '09: The Missing Manual, ch. 15 | https://www.oreilly.com/library/view/iwork-09-the/9780596802073/ch15s06.html | journalism (dated) | 2026-10-20 |
| Figma forum: change layout of an existing slide | https://forum.figma.com/ask-the-community-7/how-do-i-change-the-layout-of-an-existing-slide-in-figma-slides-46971 | vendor | 2026-10-20 |
| Figma forum: template changes should apply to existing decks | https://forum.figma.com/suggest-a-feature-11/figma-slides-publishing-template-changes-should-auto-apply-to-existing-slide-decks-23115 | practitioner | 2026-10-20 |
| Figma forum: slide layouts and template styles | https://forum.figma.com/ask-the-community-7/slides-template-creating-editing-slide-layouts-template-styles-40621 | practitioner | 2026-10-20 |
| Figma Slides export help | https://help.figma.com/hc/en-us/articles/24848334599447 | vendor | 2026-10-20 |
| Figma forum: issues exporting slides to pptx | https://forum.figma.com/report-a-problem-6/issues-exporting-slides-to-pptx-42186 | practitioner | 2026-10-20 |
| Pitchdeck: master slides in PowerPoint exports from Figma | https://docs.hypermatic.com/pitchdeck/tutorials/add-master-slides-to-powerpoint-exports-from-figma.md | vendor | 2026-10-20 |
| Figma prompt to edit | https://www.figma.com/solutions/prompt-to-edit/ | vendor | 2026-10-20 |
| Figma Make help | https://help.figma.com/hc/en-us/articles/42009840449175 | vendor | 2026-10-20 |
| Figma forum: lock sections in Figma Make | https://forum.figma.com/suggest-a-feature-11/is-there-a-way-to-lock-sections-in-figma-make-to-ensure-they-aren-t-removed-or-revised-with-an-unrelated-prompt-to-another-section-of-the-project-50240 | practitioner | 2026-10-20 |
| Figma forum: selected element not passed to Make prompt | https://forum.figma.com/report-a-problem-6/selected-element-not-passed-to-figma-make-ai-prompt-51238 | practitioner | 2026-10-20 |
| Figma REST API: comments endpoints | https://developers.figma.com/docs/rest-api/comments-endpoints | vendor | 2026-10-20 |
| Figma REST API: comments types | https://developers.figma.com/docs/rest-api/comments-types | vendor | 2026-10-20 |
| Figma REST API: comments property types | https://developers.figma.com/docs/rest-api/comments-property-types | vendor | 2026-10-20 |
| Figma help: comments | https://help.figma.com/hc/en-us/articles/360041547853 | vendor | 2026-10-20 |
| Figma forum threads 17753 and 4632 on comments following frames | (forum.figma.com) | practitioner | 2026-10-20 |
| Dev Mode GA | https://www.figma.com/blog/dev-mode-ga/ | vendor | 2026-10-20 |
| Figma Plugin API: Annotation | https://developers.figma.com/docs/plugins/api/Annotation | vendor | 2026-10-20 |
| Figma MCP server: tools and prompts | https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/ | vendor | 2026-10-20 |
| Figma forum: MCP support for annotations | https://forum.figma.com/suggest-a-feature-11/mcp-support-for-annotations-53685 | practitioner | 2026-10-20 |
| figma-comments-mcp (search summary) | https://glama.ai/mcp/servers/junsimon/figma-comments-mcp | practitioner | 2026-10-20 |
| Microsoft message center RM548646 | https://mc.merill.net/message/RM548646 | vendor | 2026-10-20 |
| Microsoft message center MC1219792 | https://www.itelio.com/en/microsoft-message-center/MC1219792 | vendor | 2026-10-20 |
| Microsoft message center MC1243550 | https://www.itelio.com/en/microsoft-message-center/MC1243550 | vendor | 2026-10-20 |
| Microsoft message center MC1243550 (merill mirror) | https://mc.merill.net/message/MC1243550 | vendor | 2026-10-20 |
| Microsoft message center MC1473166 | https://mc.merill.net/message/MC1473166 | vendor | 2026-10-20 |
| Agent mode (Frontier) in PowerPoint | https://support.microsoft.com/en-au/topic/agent-mode-frontier-in-powerpoint-008f17aa-8e5f-4cda-ba6c-0588000bdad7 | vendor | 2026-10-20 |
| Keep your presentation on brand with Copilot | https://support.microsoft.com/en-us/powerpoint/copilot/keep-your-presentation-on-brand-with-copilot | vendor | 2026-10-20 |
| Manage Brand Kit template settings | https://support.microsoft.com/en-us/powerpoint/copilot/manage-brand-kit-template-settings-in-powerpoint | vendor | 2026-10-20 |
| Microsoft Q&A 1908366 | https://learn.microsoft.com/en-us/answers/a/1908366 | practitioner | 2026-10-20 |
| Microsoft Q&A 12080966 | https://learn.microsoft.com/en-us/answers/a/12080966 | practitioner | 2026-10-20 |
| PrimeDocs: Microsoft 365 Copilot best practices | https://docs.primedocs.io/en/docs/4.0/layouter/best-practices/microsoft-365-copilot | vendor | 2026-10-20 |
| Copilot updates: on-canvas features and agent mode (search snippets) | https://m365admin.handsontek.net/microsoft-365-copilot-updates-powerpoint-on%E2%80%91canvas-features-agent-mode/ | practitioner | 2026-10-20 |
| Office.js PowerPoint shapes | https://learn.microsoft.com/en-us/office/dev/add-ins/powerpoint/shapes | vendor | 2026-10-20 |
| Plus AI: remix slides | https://guide.plusai.com/ai-for-presentations/remix-slides | vendor | 2026-10-20 |
| Beautiful.ai: introducing Classic slides (archived) | https://web.archive.org/web/20260306150742/https://www.beautiful.ai/blog/introducing-classic-slides | vendor | 2026-10-20 |
| Beautiful.ai review 2026 | https://aumiqx.com/ai-tools/beautiful-ai-review-presentation-maker-2026/ | practitioner | 2026-10-20 |
| Canva AI 2.0 adds agentic design tools | https://www.cmswire.com/digital-experience/canva-ai-20-adds-agentic-design-tools/ | journalism | 2026-10-20 |
| Canva brand template locks (archived) | https://web.archive.org/web/20260117105657/https://www.canva.com/help/brand-template-locks/ | vendor | 2026-10-20 |
| Canva: ask Canva in comments (archived) | https://web.archive.org/web/20251029103802/https://www.canva.com/help/ask-canva-comments/ | vendor | 2026-10-20 |
| Canva's AI fix-it button | https://fastcompany.co.za/co-design/2025-11-02-canva-just-introduced-an-ai-fix-it-button/ | journalism | 2026-10-20 |
| Canva MCP: commit editing transaction | https://www.canva.dev/docs/mcp/tools/commit-editing-transaction/ | vendor | 2026-10-20 |
| Canva MCP: perform editing operations | https://www.canva.dev/docs/mcp/tools/perform-editing-operations/ | vendor | 2026-10-20 |
| Tome's founders ditch viral presentation app | https://venturebeat.com/ai/tomes-founders-ditch-viral-presentation-app-with-20m-users-to-build-ai | journalism | 2026-10-20 |
| Tome AI shut down | https://www.chatslide.ai/articles/tome-ai-shut-down-what-happened | vendor (competitor) | 2026-10-20 |
| Claude Design (Anthropic Labs) | https://anthropic.com/news/claude-design-anthropic-labs | vendor | 2026-10-20 |

### Text forms, agent editing and research papers

| Title | URL | Tag | Accessed |
|---|---|---|---|
| Slidev LICENSE | https://github.com/slidevjs/slidev/blob/main/LICENSE | vendor | 2026-10-20 |
| Slidev releases | https://github.com/slidevjs/slidev/releases | vendor | 2026-10-20 |
| Slidev syntax | https://sli.dev/guide/syntax | vendor | 2026-10-20 |
| Slidev layouts | https://sli.dev/guide/layout | vendor | 2026-10-20 |
| Slidev built-in layouts | https://sli.dev/builtin/layouts | vendor | 2026-10-20 |
| Slidev slot sugar | https://sli.dev/features/slot-sugar | vendor | 2026-10-20 |
| Slidev animations | https://sli.dev/guide/animations | vendor | 2026-10-20 |
| Slidev animations source doc | https://github.com/slidevjs/slidev/blob/main/docs/guide/animations.md | vendor | 2026-10-20 |
| Slidev useNav.ts | https://github.com/slidevjs/slidev/blob/main/packages/client/composables/useNav.ts | vendor | 2026-10-20 |
| Slidev UI | https://sli.dev/guide/ui | vendor | 2026-10-20 |
| Slidev draggable | https://sli.dev/features/draggable | vendor | 2026-10-20 |
| Slidev #1959 | https://github.com/slidevjs/slidev/issues/1959 | practitioner | 2026-10-20 |
| Slidev #2365 | https://github.com/slidevjs/slidev/issues/2365 | practitioner | 2026-10-20 |
| Slidev exporting | https://sli.dev/guide/exporting | vendor | 2026-10-20 |
| Slidev PR 1603 (pptx export) | https://github.com/slidevjs/slidev/pull/1603 | vendor | 2026-10-20 |
| Slidev PR 2722 (editable pptx) | https://github.com/slidevjs/slidev/pull/2722 | vendor | 2026-10-20 |
| Slidev PR 2758 | https://github.com/slidevjs/slidev/pull/2758 | vendor | 2026-10-20 |
| Slidev PR 2709 | https://github.com/slidevjs/slidev/pull/2709 | vendor | 2026-10-20 |
| Slidev: work with AI | https://sli.dev/guide/work-with-ai | vendor | 2026-10-20 |
| Slidev skill | https://github.com/slidevjs/slidev/blob/main/skills/slidev/SKILL.md | vendor | 2026-10-20 |
| Slidev MCP | https://sli.dev/features/mcp | vendor | 2026-10-20 |
| Slidev MCP server.ts | https://github.com/slidevjs/slidev/blob/main/packages/slidev/node/mcp/server.ts | vendor | 2026-10-20 |
| Slidev vs Marp vs reveal.js (pkgpulse) | https://www.pkgpulse.com/guides/slidev-vs-marp-vs-revealjs-code-first-presentations-2026 | practitioner | 2026-10-20 |
| slidev-addon-studio | https://github.com/BobTheShoplifter/slidev-addon-studio | vendor | 2026-10-20 |
| slidev-addon-studio LICENSE | https://github.com/BobTheShoplifter/slidev-addon-studio/blob/main/LICENSE | vendor | 2026-10-20 |
| Typst LICENSE | https://github.com/typst/typst/blob/main/LICENSE | vendor | 2026-10-20 |
| Typst HTML export | https://typst.app/docs/reference/html/ | vendor | 2026-10-20 |
| Touying LICENSE | https://github.com/touying-typ/touying/blob/main/LICENSE | vendor | 2026-10-20 |
| Touying README | https://github.com/touying-typ/touying/blob/main/README.md | vendor | 2026-10-20 |
| Touying docs | https://touying-typ.github.io/docs/intro | vendor | 2026-10-20 |
| touying-exporter | https://github.com/touying-typ/touying-exporter | vendor | 2026-10-20 |
| typ2pptx | https://github.com/touying-typ/typ2pptx | vendor | 2026-10-20 |
| SeaSlides | https://github.com/touying-typ/seaslides | vendor | 2026-10-20 |
| Tylina | https://tylina.github.io/ | vendor | 2026-10-20 |
| Tylina LICENSE | https://github.com/tylina/tylina.github.io/blob/main/LICENSE | vendor | 2026-10-20 |
| TeXFix-Bench | https://arxiv.org/html/2608.07617 | independent | 2026-10-20 |
| Benchmarking LLMs on Typst (forum) | https://forum.typst.app/t/benchmarking-llms-on-typst/4085 | practitioner | 2026-10-20 |
| LLM code generation (Typst forum) | https://forum.typst.app/t/llm-code-generation/1331 | practitioner | 2026-10-20 |
| HN mirror 45693978 | https://hn.svelte.dev/item/45693978 | practitioner | 2026-10-20 |
| typst-writer skill | https://skills.sh/ypares/agent-skills/typst-writer | practitioner | 2026-10-20 |
| Aider: LLMs are bad at returning code in JSON | https://aider.chat/2024/08/14/code-in-json.html | practitioner | 2026-10-20 |
| Let Me Speak Freely? | https://arxiv.org/abs/2408.02442 | independent | 2026-10-20 |
| Say What You Mean (.txt) | https://blog.dottxt.ai/say-what-you-mean.html | vendor | 2026-10-20 |
| JSON Whisperer | https://arxiv.org/abs/2510.04717v1 | independent | 2026-10-20 |
| ACE | https://arxiv.org/html/2608.24103v1 | independent (preprint) | 2026-10-20 |
| ACE (Hugging Face papers) | https://huggingface.co/papers/2608.24103.md | independent (preprint) | 2026-10-20 |
| PPTArena | https://arxiv.org/html/2512.03042 | independent | 2026-10-20 |
| Talk to Your Slides | https://arxiv.org/abs/2505.11604 | independent | 2026-10-20 |
| PPTC | https://arxiv.org/abs/2311.01767 | independent | 2026-10-20 |
| LLM slides formats (Brosse) | https://nbrosse.github.io/posts/llm-slides/llm-slides.html | practitioner | 2026-10-20 |
| Quarto visual editor: Markdown | https://quarto.org/docs/visual-editor/markdown.html | vendor | 2026-10-20 |
| marp-mcp | https://github.com/masaki39/marp-mcp | vendor | 2026-10-20 |
| marp-mcp slide-id.ts | https://github.com/masaki39/marp-mcp/blob/main/src/utils/slide-id.ts | vendor | 2026-10-20 |
| Marp CLI | https://github.com/marp-team/marp-cli | vendor | 2026-10-20 |
| Anthropic pptx skill | https://github.com/anthropics/skills/blob/main/skills/pptx/SKILL.md | vendor | 2026-10-20 |
| Office-PowerPoint-MCP-Server | https://github.com/GongRzhe/Office-PowerPoint-MCP-Server | vendor | 2026-10-20 |
| Tiptap: review changes | https://tiptap.dev/docs/ai/ai-toolkit/client/agents/review-changes | vendor | 2026-10-20 |
| Diff patch review (AI UX Playground) | https://aiuxplayground.com/glossary/diff-patch-review | practitioner | 2026-10-20 |
| Microsoft HAX toolkit G9-C: undo automated actions | https://www.microsoft.com/en-us/haxtoolkit/pattern/g9-c-undo-automated-actions/ | vendor (research-backed) | 2026-10-20 |
| Patchwork notebook 07 | https://www.inkandswitch.com/patchwork/notebook/2024-version-control/07/ | independent | 2026-10-20 |
| Patchwork notebook (version control) | https://www.inkandswitch.com/patchwork/notebook/2024-version-control/ | independent | 2026-10-20 |

### pptx export, fonts, SVG and charts

| Title | URL | Tag | Accessed |
|---|---|---|---|
| python-pptx LICENSE | https://github.com/scanny/python-pptx/blob/master/LICENSE | vendor | 2026-10-20 |
| python-pptx on PyPI (JSON) | https://pypi.org/pypi/python-pptx/json | vendor | 2026-10-20 |
| python-pptx GitHub API | https://api.github.com/repos/scanny/python-pptx | vendor | 2026-10-20 |
| python-pptx download stats | https://pypistats.org/api/packages/python-pptx/recent | vendor | 2026-10-20 |
| python-pptx charts | https://python-pptx.readthedocs.io/en/latest/user/charts.html | vendor | 2026-10-20 |
| python-pptx #1112 | https://github.com/scanny/python-pptx/issues/1112 | practitioner | 2026-10-20 |
| python-pptx PR 1115 | https://github.com/scanny/python-pptx/pull/1115 | practitioner | 2026-10-20 |
| PptxGenJS LICENSE | https://github.com/gitbrent/PptxGenJS/blob/master/LICENSE | vendor | 2026-10-20 |
| PptxGenJS repo | https://github.com/gitbrent/PptxGenJS | vendor | 2026-10-20 |
| PptxGenJS releases | https://api.github.com/repos/gitbrent/PptxGenJS/releases | vendor | 2026-10-20 |
| PptxGenJS npm downloads | https://api.npmjs.org/downloads/point/last-month/pptxgenjs | vendor | 2026-10-20 |
| PptxGenJS masters docs | https://gitbrent.github.io/PptxGenJS/docs/masters/ | vendor | 2026-10-20 |
| PptxGenJS charts docs | https://gitbrent.github.io/PptxGenJS/docs/api-charts/ | vendor | 2026-10-20 |
| PptxGenJS pptxgen.ts at 3c9ec1b | https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/pptxgen.ts#L526-L537 | vendor | 2026-10-20 |
| PptxGenJS gen-xml.ts at 3c9ec1b | https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/gen-xml.ts#L1405-L1412 | vendor | 2026-10-20 |
| PptxGenJS issues 1453, 1208, 856, 1422, 1443, 1449, 604, 1324 | https://github.com/gitbrent/PptxGenJS/issues/1453 (and siblings) | practitioner | 2026-10-20 |
| PptxGenJS issues 327, 704 (animations) | https://github.com/gitbrent/PptxGenJS/issues/327 | vendor | 2026-10-20 |
| PptxGenJS open repair issues | https://github.com/gitbrent/PptxGenJS/issues?q=is%3Aopen+repair | practitioner | 2026-10-20 |
| pptx-embed-fonts | https://github.com/liyao1520/pptx-embed-fonts | practitioner | 2026-10-20 |
| power-pptx | https://github.com/codehalwell/power-pptx | vendor | 2026-10-20 |
| power-pptx contributors | https://api.github.com/repos/codehalwell/power-pptx/contributors | vendor | 2026-10-20 |
| pptx-automizer limitations | https://github.com/singerla/pptx-automizer/blob/main/docs/limitations.md | vendor | 2026-10-20 |
| Aspose.Slides pricing | https://purchase.aspose.com/pricing/slides/family/ | vendor | 2026-10-20 |
| Aspose.Slides: slide layout | https://docs.aspose.com/slides/python-net/slide-layout/ | vendor | 2026-10-20 |
| Aspose.Slides: embedded font | https://docs.aspose.com/slides/python-net/embedded-font/ | vendor | 2026-10-20 |
| Aspose.Slides FOSS for Python | https://github.com/aspose-slides-foss/Aspose.Slides-FOSS-for-Python | vendor | 2026-10-20 |
| Syncfusion Community License | https://www.syncfusion.com/products/communitylicense | vendor | 2026-10-20 |
| dom-to-pptx | https://github.com/atharva9167j/dom-to-pptx | vendor | 2026-10-20 |
| html2pptx (mirror) | https://cdn.jsdelivr.net/npm/stableclaw@2026.5.15/skills/pptx/html2pptx.md | practitioner | 2026-10-20 |
| Benefits of embedding custom fonts | https://support.microsoft.com/en-us/office/benefits-of-embedding-custom-fonts-cb3982aa-ea76-4323-b008-86670f222dbc | vendor | 2026-10-20 |
| PPTX to PDF missing fonts (search snippet) | https://pdf.it.com/learn/pptx-to-pdf-missing-fonts | practitioner | 2026-10-20 |
| Collabora contributions in LibreOffice 25.8 | https://www.collaboraonline.com/blog/collabora-productivity-contributions-in-libreoffice-25-8/ | vendor | 2026-10-20 |
| Edit SVG images in Microsoft Office | https://support.microsoft.com/en-us/office/edit-svg-images-in-microsoft-office-69f29d39-194a-4072-8c35-dbe5e7ea528c | vendor | 2026-10-20 |
| Edit SVG images in Microsoft 365 | https://support.microsoft.com/en-us/office/edit-svg-images-in-microsoft-365-69f29d39-194a-4072-8c35-dbe5e7ea528c | vendor | 2026-10-20 |
| FlexCel SVGBlip (search snippet) | https://doc.tmssoftware.com/flexcel/net/api/FlexCel.Core/TBlip/SVGBlip.html | vendor | 2026-10-20 |
| ECharts LICENSE | https://github.com/apache/echarts/blob/master/LICENSE | vendor | 2026-10-20 |
| ECharts NOTICE | https://github.com/apache/echarts/blob/master/NOTICE | vendor | 2026-10-20 |
| zrender LICENSE | https://github.com/ecomfe/zrender/blob/master/LICENSE | vendor | 2026-10-20 |
| ECharts server-side rendering | https://echarts.apache.org/handbook/en/how-to/cross-platform/server/ | vendor | 2026-10-20 |
| ECharts canvas vs SVG | https://echarts.apache.org/handbook/en/best-practices/canvas-vs-svg/ | vendor | 2026-10-20 |
| ECharts open SVG renderer issues | https://github.com/apache/echarts/issues?q=is%3Aopen+%22svg+renderer%22 | practitioner | 2026-10-20 |
| ECharts dataset | https://echarts.apache.org/handbook/en/concepts/dataset/ | vendor | 2026-10-20 |
| Vega-Lite LICENSE | https://github.com/vega/vega-lite/blob/main/LICENSE | vendor | 2026-10-20 |
| Vega LICENSE | https://github.com/vega/vega/blob/main/LICENSE | vendor | 2026-10-20 |
| Vega view API | https://github.com/vega/vega/blob/main/docs/docs/api/view.md | vendor | 2026-10-20 |
| Vega-Lite compile | https://vega.github.io/vega-lite/usage/compile.html | vendor | 2026-10-20 |
| vl-convert LICENSE | https://github.com/vega/vl-convert/blob/main/LICENSE | vendor | 2026-10-20 |
| vl-convert-python | https://pypi.org/project/vl-convert-python/0.13.1 | vendor | 2026-10-20 |
| Chart.js LICENSE and docs (Node, device pixel ratio) | https://github.com/chartjs/Chart.js/blob/master/LICENSE.md | vendor | 2026-10-20 |
| Observable Plot LICENSE | https://github.com/observablehq/plot/blob/main/LICENSE | vendor | 2026-10-20 |
| Observable Plot getting started | https://github.com/observablehq/plot/blob/main/docs/getting-started.md | vendor | 2026-10-20 |
| Recharts LICENSE | https://github.com/recharts/recharts/blob/main/LICENSE | vendor | 2026-10-20 |
| Recharts #5997 | https://github.com/recharts/recharts/issues/5997 | practitioner | 2026-10-20 |
| Recharts #6139 | https://github.com/recharts/recharts/issues/6139 | practitioner | 2026-10-20 |
| D3 LICENSE | https://github.com/d3/d3/blob/main/LICENSE | vendor | 2026-10-20 |
| Export Apex charts as PowerPoint charts (search snippet) | https://codix.ai/export-apex-charts-as-powerpoint-charts/ | practitioner | 2026-10-20 |
| think-cell compatibility | https://www.thinkcell.com/en/resources/manual/compatibility | vendor | 2026-10-20 |

### Animations

| Title | URL | Tag | Accessed |
|---|---|---|---|
| python-pptx oxml/slide.py | https://github.com/scanny/python-pptx/blob/master/src/pptx/oxml/slide.py | vendor | 2026-10-20 |
| ECMA-376 seq | https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/seq.html | independent | 2026-10-20 |
| ECMA-376 ST_TLTimeNodeType, ST_TLTimeNodePresetClassType, cTn, stCondLst, cond, ST_TLPreviousActionType | https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/cTn.html (and siblings) | independent | 2026-10-20 |
| ECMA-376 animEffect, animScale, animClr, animMotion, tgtEl and motion types | https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/animMotion.html (and siblings) | independent | 2026-10-20 |
| ECMA-376 bldP, ST_TLParaBuildType, txEl | https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/bldP.html (and siblings) | independent | 2026-10-20 |
| ECMA-376 transition | https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/transition.html | independent | 2026-10-20 |
| PowerPoint-saved sample file_with_bounce.pptx | https://github.com/scanny/python-pptx/files/14556139/file_with_bounce.pptx | practitioner | 2026-10-20 |
| python-pptx #954 | https://github.com/scanny/python-pptx/issues/954 | practitioner | 2026-10-20 |
| python-pptx #1106 | https://github.com/scanny/python-pptx/issues/1106 | practitioner | 2026-10-20 |
| python-pptx #264 | https://github.com/scanny/python-pptx/issues/264 | vendor | 2026-10-20 |
| dom-to-pptx xml-templates.js | https://github.com/atharva9167j/dom-to-pptx/blob/master/src/animations/xml-templates.js | vendor | 2026-10-20 |
| OfficeCLI PowerPointHandler.Animations.cs | https://github.com/iOfficeAI/OfficeCLI/blob/main/src/officecli/Handlers/Pptx/PowerPointHandler.Animations.cs | vendor | 2026-10-20 |
| OfficeCLI | https://github.com/iOfficeAI/OfficeCLI | vendor | 2026-10-20 |
| svg2ooxml animation fidelity | https://github.com/BramAlkema/svg2ooxml/blob/main/docs/specs/animation-fidelity.md | practitioner | 2026-10-20 |
| LibreOffice commontimenodecontext.cxx | https://github.com/LibreOffice/core/blob/master/oox/source/ppt/commontimenodecontext.cxx | independent | 2026-10-20 |
| Aspose FOSS animation_mappings.py | https://github.com/aspose-slides-foss/Aspose.Slides-FOSS-for-Python/blob/main/aspose/slides_foss/_internal/pptx/animation_mappings.py | vendor | 2026-10-20 |
| ChristopherVR/ooxml emphasis mappings | https://github.com/ChristopherVR/ooxml/blob/main/src/core/pptx/core/services/animation-write-mappings-emphasis.ts | vendor | 2026-10-20 |
| MsoAnimEffect (VBA) | https://learn.microsoft.com/en-us/office/vba/api/powerpoint.msoanimeffect | vendor | 2026-10-20 |
| LibreOffice bug 169343 | https://bugs.documentfoundation.org/show_bug.cgi?id=169343 | independent | 2026-10-20 |
| Resource2Skill PPT MCP server | https://github.com/microsoft/Resource2Skill/blob/main/domains/ppt/mcp_server/server.py | vendor | 2026-10-20 |
| Aspose TransitionMorphType (search excerpt) | https://apireference.aspose.com/slides/java/com.aspose.slides/transitionmorphtype | vendor | 2026-10-20 |
| Morph transition tips and tricks | https://support.microsoft.com/en-us/office/morph-transition-tips-and-tricks-bc7f48ff-f152-4ee8-9081-d3121788024f | vendor | 2026-10-20 |
| LibreOffice bug 116736 (Morph) | https://bugs.documentfoundation.org/show_bug.cgi?id=116736 | independent | 2026-10-20 |
| Animate words one line at a time (search excerpt) | https://support.microsoft.com/en-us/PowerPoint/animate-or-make-words-appear-one-line-at-a-time-in-powerpoint | vendor | 2026-10-20 |
| Animating text (Indezine, search excerpt) | https://www.indezine.com/products/powerpoint/learn/animationsandtransitions/2013/animating-text.html | practitioner | 2026-10-20 |
| reveal.js fragments | https://revealjs.com/fragments/ | vendor | 2026-10-20 |
| reveal.js fragments.js | https://github.com/hakimel/reveal.js/blob/master/js/controllers/fragments.js | vendor | 2026-10-20 |
| reveal.js auto-animate | https://revealjs.com/auto-animate/ | vendor | 2026-10-20 |
| reveal.js source | https://github.com/hakimel/reveal.js/blob/master/js/reveal.js | vendor | 2026-10-20 |
| Convert PowerPoint to Google Slides (BrightCarbon) | https://www.brightcarbon.com/blog/convert-powerpoint-google-slides/ | practitioner | 2026-10-20 |
| Fix compatibility PowerPoint and Google Slides (SlideModel, search excerpt) | https://slidemodel.com/fix-compatibility-powerpoint-google-slides/ | practitioner | 2026-10-20 |
| LibreOffice meta bug 139897 | https://bugs.documentfoundation.org/show_bug.cgi?id=139897 | independent | 2026-10-20 |
| LibreOffice bug 128821 | https://bugs.documentfoundation.org/show_bug.cgi?id=128821 | independent | 2026-10-20 |
| LibreOffice bug 34407 | https://bugs.documentfoundation.org/show_bug.cgi?id=34407 | independent | 2026-10-20 |
| MDN: Animation.reverse() | https://developer.mozilla.org/en-US/docs/Web/API/Animation/reverse | independent | 2026-10-20 |
| MDN: using the Web Animations API | https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API/Using_the_Web_Animations_API | independent | 2026-10-20 |
| MDN: offset-path | https://developer.mozilla.org/en-US/docs/Web/CSS/offset-path | independent | 2026-10-20 |
| Animations don't play when I go back to a slide (PPT FAQ) | https://www.rdpslides.com/pptfaq/FAQ00598_Animations_don-t_play_when_I_go_back_to_a_slide.htm | practitioner | 2026-10-20 |
| Berger weblog, SVG tag | https://www-public.telecom-sudparis.eu/~berger_o/weblog/tag/svg/index.html | practitioner | 2026-10-20 |
| Animating SVG topic maps with reveal.js (Sacha Chua) | https://sachachua.com/blog/2024/01/animating-svg-topic-maps-with-inkscape-emacs-ffmpeg-and-reveal-js/ | practitioner | 2026-10-20 |
| reveal.js animate plugin README (search excerpt) | https://people.willamette.edu/~jjrembold/class_files/data275/Slides/html_srcs/plugin/animate/README.md | practitioner | 2026-10-20 |

### Comments and agent loops

| Title | URL | Tag | Accessed |
|---|---|---|---|
| MS-PPTX CT_Comment | https://learn.microsoft.com/en-us/openspecs/office_standards/ms-pptx/161bc2c9-98fc-46b7-852b-ba7ee77e2e54 | vendor | 2026-10-20 |
| MS-PPTX ST_CommentStatus | https://learn.microsoft.com/en-us/openspecs/office_standards/ms-pptx/ (ST_CommentStatus page) | vendor | 2026-10-20 |
| MS-ODRAWXML monikers | https://learn.microsoft.com/en-us/openspecs/office_standards/ms-odrawxml/145d263b-da8a-4cad-b076-1e51a9d149bf | vendor | 2026-10-20 |
| Modern comments in PowerPoint | https://support.microsoft.com/en-us/powerpoint/modern-comments-in-powerpoint | vendor | 2026-10-20 |
| PowerPoint's comments stick like glue | https://office-watch.com/2025/powerpoints-comments-stick-like-glue/ | journalism | 2026-10-20 |
| Do more with modern comments (search snippet) | https://insider.microsoft365.com/blog/do-more-with-modern-comments-in-powerpoint | vendor | 2026-10-20 |
| Lucid community: shape was deleted error (search summary, page inaccessible) | https://community.lucid.co/admin-questions-2/receiving-the-shape-was-deleted-error-within-comments-tasks-12546 | practitioner | 2026-10-20 |
| LLM 3D scene editing with pointing | https://arxiv.org/html/2410.22177 | independent | 2026-10-20 |
| github-mcp-server README | https://github.com/github/github-mcp-server/blob/main/README.md | vendor | 2026-10-20 |
| Review Copilot PRs | https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/review-copilot-prs | vendor | 2026-10-20 |
| Vercel CLI comments | https://vercel.com/docs/cli/comments | vendor | 2026-10-20 |
| Manage Vercel toolbar comments from the CLI | https://vercel.com/changelog/manage-vercel-toolbar-comments-from-the-cli | vendor | 2026-10-20 |
| Using Vercel comments | https://vercel.com/docs/comments/using-comments | vendor | 2026-10-20 |
| React Grab | https://github.com/aidenybai/react-grab | vendor | 2026-10-20 |
| PR-Review-Resolve-MCP reply_and_resolve | https://glama.ai/mcp/servers/UtakataKyosui/PR-Review-Resolve-MCP/tools/reply_and_resolve | practitioner | 2026-10-20 |

### Collaboration and CRDTs

| Title | URL | Tag | Accessed |
|---|---|---|---|
| Yjs LICENSE | https://github.com/yjs/yjs/blob/main/LICENSE | vendor | 2026-10-20 |
| Automerge LICENSE | https://github.com/automerge/automerge/blob/main/LICENSE | vendor | 2026-10-20 |
| Loro LICENSE | https://github.com/loro-dev/loro/blob/main/LICENSE | vendor | 2026-10-20 |
| fractional-indexing LICENSE | https://github.com/rocicorp/fractional-indexing/blob/main/LICENSE | vendor | 2026-10-20 |
| Yjs INTERNALS | https://github.com/yjs/yjs/blob/main/INTERNALS.md | vendor | 2026-10-20 |
| y-utility README | https://cdn.jsdelivr.net/npm/y-utility@0.1.4/README.md | vendor | 2026-10-20 |
| Yjs best practices (Liveblocks) | https://liveblocks.io/docs/guides/yjs-best-practices-and-tips | vendor | 2026-10-20 |
| discuss.yjs.dev: map metadata overhead | https://discuss.yjs.dev/t/map-metadata-overhead/492 | practitioner | 2026-10-20 |
| discuss.yjs.dev: document growing unbounded | https://discuss.yjs.dev/t/document-growing-unbounded/3729 | practitioner | 2026-10-20 |
| Yjs 14.0.0-0 release | https://newreleases.io/project/yarn/yjs/release/14.0.0-0 | vendor | 2026-10-20 |
| BlockNote, Yjs and ProseMirror (FOSDEM) | https://fosdem.org/schedule/event/8VKQXR-blocknote-yjs-prosemirror | practitioner | 2026-10-20 |
| Automerge merge rules | https://github.com/automerge/automerge.github.io/blob/main/content/docs/reference/under-the-hood/merge-rules.md | vendor | 2026-10-20 |
| Automerge conflicts | https://automerge.org/docs/reference/documents/conflicts/ | vendor | 2026-10-20 |
| Automerge modeling data | https://github.com/automerge/automerge.github.io/blob/main/content/docs/cookbook/modeling-data.md | vendor | 2026-10-20 |
| Automerge 3 | https://automerge.org/blog/automerge-3/ | vendor | 2026-10-20 |
| Automerge implementation.ts | https://github.com/automerge/automerge/blob/main/javascript/src/implementation.ts | vendor | 2026-10-20 |
| Loro docs (list, map, tree, v1.0, time travel) | https://github.com/loro-dev/loro-docs (pages) | vendor | 2026-10-20 |
| Loro: when not to use a CRDT | https://github.com/loro-dev/loro-docs/blob/main/pages/docs/concepts/when_not_crdt.mdx | vendor | 2026-10-20 |
| crdt-benchmarks | https://github.com/dmonad/crdt-benchmarks | practitioner (run by the Yjs author; pre-Automerge 3, pre-Loro 1.0) | 2026-10-20 |
| How Figma's multiplayer technology works | https://www.figma.com/blog/how-figmas-multiplayer-technology-works/ | vendor | 2026-10-20 |
| Realtime editing of ordered sequences | https://www.figma.com/blog/realtime-editing-of-ordered-sequences/ | vendor | 2026-10-20 |
| Making multiplayer more reliable | https://www.figma.com/blog/making-multiplayer-more-reliable/ | vendor | 2026-10-20 |
| CRDT fractional indexing (Wallace) | https://madebyevan.com/algos/crdt-fractional-indexing/ | practitioner | 2026-10-20 |
| Excalidraw PR 7359 | https://github.com/excalidraw/excalidraw/pull/7359 | vendor | 2026-10-20 |
| Excalidraw fractional PRs (7348, 11258, 11934) | https://github.com/excalidraw/excalidraw/pulls?q=fractional | vendor | 2026-10-20 |
| Building Excalidraw's P2P collaboration | https://plus.excalidraw.com/blog/building-excalidraw-p2p-collaboration-feature | vendor | 2026-10-20 |
| Moving elements in list CRDTs (Kleppmann) | https://martin.kleppmann.com/papers/list-move-papoc20.pdf | independent | 2026-10-20 |
| You might not need a CRDT (Jamsocket, Wayback) | https://web.archive.org/web/2024/https://jamsocket.com/blog/you-might-not-need-a-crdt | vendor | 2026-10-20 |
| Cambria | https://www.inkandswitch.com/cambria/ | independent | 2026-10-20 |
| Live and local schema change (search snippet) | https://arxiv.org/pdf/2309.11406 | independent | 2026-10-20 |
| reverse-linear-sync-engine | https://github.com/wzhudev/reverse-linear-sync-engine | practitioner (unaffiliated) | 2026-10-20 |
| RFC 6902, JSON Patch | https://www.rfc-editor.org/rfc/rfc6902 | independent | 2026-10-20 |
| Hocuspocus DirectConnection.ts | https://github.com/ueberdosis/hocuspocus/blob/main/packages/server/src/DirectConnection.ts | vendor | 2026-10-20 |
| Tiptap server AI toolkit (search snippet) | https://tiptap.dev/docs/content-ai/capabilities/server-ai-toolkit/overview | vendor | 2026-10-20 |

### Local prior art

| Title | Location | Tag | Accessed |
|---|---|---|---|
| decks spec and idea | decks: `docs/spec-v0.md`, `docs/idea.md` | practitioner | 2026-10-20 |
| Blattomat spec | blattwerk: `docs/spec-v0.md` | practitioner | 2026-10-20 |
| Blattomat editor, history, saves | blattwerk: `frontend/src/pages/Editor.tsx`, `frontend/src/history.ts`, `frontend/src/saves.ts` | practitioner | 2026-10-20 |
| Blattomat server and MCP | blattwerk: `src/blattwerk/sheets.py`, `remote.py`, `oauth.py`, `pdf.py`, `sheet.schema.json`; `frontend/mcp/tools.ts`, `frontend/mcp/docs.ts` | practitioner | 2026-10-20 |
| Blattomat OKF docs | blattwerk: `docs/okf/index.md`, `teacher-does.md`, `versions.md`, `page-model.md`, `tools/errors-and-limits.md` | practitioner | 2026-10-20 |
| Blattomat scripts and tests | blattwerk: `scripts/okf_check.py`, `scripts/eval.py`, `tests/` | practitioner | 2026-10-20 |
| Blattomat issue #201 | https://github.com/pgoell/blattwerk/issues/201 | practitioner | 2026-10-20 |
| Blattomat issue #732 | https://github.com/pgoell/blattwerk/issues/732 | practitioner | 2026-10-20 |
| Blattomat issue #889 (no LICENSE) | https://github.com/pgoell/blattwerk/issues/889 | practitioner | 2026-10-20 |
| presentations plugin: deck-stage.js | pgoell-claude-tools: `plugins/presentations/skills/creating-presentations/assets/deck-stage.js` | practitioner | 2026-10-20 |
| presentations plugin: creating-presentations SKILL.md, hard-gates.md, layouts-and-visuals.md | pgoell-claude-tools: `plugins/presentations/skills/creating-presentations/` | practitioner | 2026-10-20 |
| presentations plugin: exporting-presentations-to-pptx SKILL.md and python-pptx-recipes.md | pgoell-claude-tools: `plugins/presentations/skills/exporting-presentations-to-pptx/` | practitioner | 2026-10-20 |
| presentations plugin: editing-presentations SKILL.md and overlay-gates.js | pgoell-claude-tools: `plugins/presentations/skills/editing-presentations/` | practitioner | 2026-10-20 |
| presentations plugin: designing-presentations SKILL.md | pgoell-claude-tools: `plugins/presentations/skills/designing-presentations/` | practitioner | 2026-10-20 |
| presentations plugin: extracting-presets references (slide-masters.md, template-libraries.md) | pgoell-claude-tools: `plugins/presentations/skills/extracting-presets/references/` | practitioner | 2026-10-20 |
| presentations plugin: presets (README, default colors.css, example slides) | pgoell-claude-tools: `plugins/presentations/presets/` | practitioner | 2026-10-20 |
| presentations plugin LICENSE | pgoell-claude-tools: `LICENSE` | practitioner | 2026-10-20 |
| Private deck repo | aggregate counts only, no paths or names | practitioner | 2026-10-20 |
| Local dependency license reads (React, FastAPI, Playwright, PyMuPDF, CairoSVG, Pillow and others) | see License Table | vendor | 2026-10-20 |
