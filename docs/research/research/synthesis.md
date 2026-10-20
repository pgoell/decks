# Synthesis (Iteration 1)

Inputs: `plan.md` and the nine cluster files in `research/` (canvas-rendering, masters-layouts, text-forms, ai-editing, export-chart-libs, animations, comments-pointing, collab-crdt, local-prior-art). No gap-fill files exist. Cluster to open-question map: canvas-rendering = OQ1, masters-layouts = OQ2, text-forms = OQ3, ai-editing = OQ4, export-chart-libs = OQ5, animations = OQ6, comments-pointing = OQ7, collab-crdt = OQ8; local-prior-art feeds all eight. The plan's local-prior-art SQ2 says "open question 6" but its content (concurrency) bears on OQ8 and the MCP section; it is mapped to OQ8 here, as the researcher file itself notes.

PRIVACY NOTE FOR THE WRITER (binding). Use only aggregate, unnamed facts from the private deck repo; local-prior-art.md has been cut down to such aggregates. Never name a deck, client, slide or path from it.

## Thesis

decks should keep the spec's core bets (each block an HTML element on a 1920 by 1080 slide, one typed JSON document addressed by id, MCP tools as the agent's only write path, a python-pptx exporter ported from the plugin) and reject the three bigger alternatives the scan tested (tldraw, a two-way text form, a CRDT now), but it must tighten the model in five places before v0 code is written, because each is cheap now and costly to retrofit: split master from layout and give each slot an OOXML placeholder type and index; store order as fractional-index strings, not array places and integer `z`; keep rich text as structured runs; key the animation model to the PowerPoint preset ids that two independent importers agree on; and keep comments, selection and drawings in a store beside the deck, anchored to block ids with a fallback when the block goes.

## Argument Structure

1. **The canvas question is not DOM vs canvas; it is license and model fit, and both point away from tldraw.** tldraw itself draws shapes as DOM; Pitch draws DOM; Google Slides draws SVG; only Figma (and probably Canva) draw to `<canvas>` with a custom text engine. tldraw's editor needs a paid production key (Pascal's talks are commercial use) and has no master, layout or placeholder concept. The real risk on the DOM path is that `moveable`/`selecto` have had no release since 2023-12. (OQ1)
2. **Real pptx layouts need more in the JSON than the spec holds today.** OOXML and Google Slides both separate master (text styles, colour map, fixed shapes) from layout (placeholder arrangement); slide-to-layout inheritance runs on placeholder `idx`, layout-to-master on `type`; inherit means an empty `<p:spPr/>`. The spec's "A master is a layout" and a bare `slot` name are not enough. Neither library gives this for free: python-pptx cannot add layouts, PptxGenJS writes layouts whose slide placeholders pin their own geometry and trigger repair dialogs with several masters. (OQ2, OQ5)
3. **Agents do best on structured data addressed by stable id through semantic tools, not on raw HTML, absolute-coordinate OOXML, or a hand-kept two-way text form.** ACE, PPTArena, Talk to Your Slides and JSON Whisperer point the same way; read-only Markdown projections are the norm in shipping MCP servers (Gamma, Pitch, Anthropic's pptx skill); no product keeps a JSON source with an editable two-way text projection, and Slidev Studio documents what such round trips lose. Typst/Touying is weaker for agents and has no solid editable pptx. (OQ3)
4. **No shipping AI deck tool protects hand-placed work with a hard rule, and the evidence of clobbered edits argues for decks' own guards: per-op writes by id, tool-level lock refusal, version checks, and undo that holds agent writes.** Gamma, Pitch, Copilot, Gemini and Plus AI scope edits to deck, slide or element; acceptance is per request at best; locks are guidance (Copilot) or unverified (Canva). Blattomat already refuses locked blocks and stale versions. (OQ4)
5. **Export libraries cover shapes, text, tables, notes and charts, but not animations, theme colours or SVG; those are hand-written OOXML whichever library is chosen.** Charts need a library that renders SVG for the browser and Chromium PDF from a plain 2D table that maps onto native pptx chart data: ECharts or Vega-Lite. (OQ5)
6. **The spec's animation model already matches PowerPoint's; what it lacks is a verified preset-id table, a per-paragraph build target, and a plan for transitions' millisecond duration (a p14 extension).** Survival across tools is poor everywhere (Google Slides drops delays and paths; Keynote export unknown; LibreOffice has 27 open animation bugs), so decks should treat its own HTML runtime and PowerPoint as the two targets and nothing else. (OQ6)
7. **Comments anchor best to object ids with a normalized offset and a defined fallback on delete (tldraw, PowerPoint modern comments, Google Slides API 2026); they live outside the document; agents work them as list, reply, resolve.** (OQ7)
8. **Live collaboration does not need a CRDT; it needs stable client-made ids, order as fractional indexes, per-property last-writer-wins, rich text as structured data, no derived data in the document, and presence and comments outside it.** Figma, tldraw sync and Linear all run server-ordered without a CRDT; Blattomat's version-and-409 rule is the right v0 base. (OQ8)

Caveats that cut against priority, not design: the real decks hold 0 `<p:timing>`, 0 builds, 0 native charts and 1 `<p:ph` in total; so animations and native charts are bets on future use, not ports of current use.

## Claims by Sub-Question

### OQ1: Draw the canvas as HTML elements or on a canvas library such as tldraw? (cluster canvas-rendering, plus local-prior-art SQ1)

#### OQ1.1 tldraw license (read from LICENSE.md)

- Claim: tldraw's root `LICENSE.md` is a proprietary source-available "tldraw license": it permits use "in Development Environments", modifying and bundling, and requires "Not to use the Software in Production Environments" and "Not to disable, change, or interfere with the Software's License Key enforcement"; production is allowed only under a trial key or "a separate commercial agreement"; the software may "collect and transmit usage data to tldraw for license compliance purposes"; Delaware law.
  - Sources: [https://raw.githubusercontent.com/tldraw/tldraw/main/LICENSE.md | tldraw license (LICENSE.md, main) | vendor]; [`gh api repos/tldraw/tldraw` spdx NOASSERTION | GitHub API | vendor]
- Claim: The split by package: `@tldraw/store`, `state`, `state-react`, `utils`, `validate`, `tlschema` are MIT; `tldraw`, `@tldraw/editor`, `sync`, `sync-core`, `commenting`, `collaboration`, `mentions`, `driver`, `mermaid` are under the tldraw license. Rendering, shapes and UI sit in the licensed packages.
  - Sources: [https://raw.githubusercontent.com/tldraw/tldraw/main/packages/store/LICENSE.md and siblings | tldraw package LICENSE files | vendor]; [packages/*/package.json | vendor]
- Claim: License history: non-commercial (2023-12), commercial with watermark (3.x), development-only without a key since 4.0 (2025-09-17); SDK 5.0 (2026-05) and v5.5.2 (2026-10-02) made no further change. Ruiz: "You can stay on 3.x."
  - Sources: [https://raw.githubusercontent.com/tldraw/tldraw/3cf4dae34d/LICENSE.md | tldraw License 2023-12-19 | vendor]; [https://raw.githubusercontent.com/tldraw/tldraw/e455ab838b~1/LICENSE.md | pre-4.0 | vendor]; [https://tldraw.dev/blog/tldraw-sdk-4-0 | Announcing tldraw SDK 4.0 | vendor]; [https://news.ycombinator.com/item?id=45293839 | HN thread | practitioner]; [https://tldraw.dev/blog/tldraw-sdk-5-0 | SDK 5.0 | vendor]
- Claim: Without a paid key: 100-day free trial (key stops working after), a "discretionary hobby license" for non-commercial projects with mandatory "made with tldraw" watermark; commenting needs a license that includes commenting.
  - Sources: [https://tldraw.dev/community/license | tldraw license page | vendor]; [https://tldraw.dev/blog/20-things-i-wish-ai-chatbots-knew-about-tldraw | 20 things (Ruiz, 2026-06-12) | vendor]; [https://tldraw.dev/llms-full.txt Commenting | tldraw docs | vendor]
- Claim: Price: $6,000/yr "Startup" (10 employees or fewer, 2 domains) on the 2025-10 archived page; no figure on 2026-08 and 2026-10 pages ("Value-based pricing", startup discounts). HN reaction split ("we don't want your business" vs "quite reasonable").
  - Sources: [http://web.archive.org/web/20251010182142/https://tldraw.dev/pricing | archived 2025-10-10 | vendor]; [http://web.archive.org/web/20260828114609/https://tldraw.dev/pricing | archived 2026-08-28 | vendor]; [https://tldraw.dev/pricing | live 2026-10-20 | vendor]; [https://news.ycombinator.com/item?id=45293839 | HN | practitioner]; [https://biggo.com/news/202509190115_tldraw_SDK_4.0_Licensing_Debate | biggo | journalism]
  - Notes: current price unknown; "value-based".
- Contradictions: hobby license on vendor pages vs LICENSE.md naming only trial and commercial. See Reconciled Contradictions 1.

#### OQ1.2 tldraw rendering, model, slides fit

- Claim: tldraw shapes are "HTML and CSS in the DOM, rendered by React, not drawn to a 2D `<canvas>`"; canvas only for overlays (since 5.0 one HTML canvas for handles, snap lines, brush).
  - Sources: [https://tldraw.dev/blog/20-things-i-wish-ai-chatbots-knew-about-tldraw | Ruiz | vendor]; [https://tldraw.dev/blog/tldraw-sdk-5-0 | vendor]; [https://tldraw.dev/llms-full.txt Overlay utils | vendor]; [https://syntax.fm/505 | Syntax 505 (search summary) | practitioner]
- Claim: Store of typed records with branded ids, `props`, `meta`, index, lock; edits through `editor.createShapes/updateShapes/run`; rich text via TipTap (ProseMirror) as JSON; export SVG via `toSvg` or `<foreignObject>`, fonts inlined, raster at 2x.
  - Sources: [https://tldraw.dev/llms-full.txt Store, Shapes, Rich text, Image export | vendor]; [https://tldraw.dev/docs/editor | vendor]; [https://tldraw.dev/docs/shapes | vendor]
- Claim: Two repo slide examples (free camera custom `slide` shape, 720x480; fixed camera with frames and `setCameraOptions` contain); tldraw offline presentation mode (2026-07) built from frames; no master, layout or placeholder concept anywhere in the 2.5 MB docs. Masters would be custom code [author estimate].
  - Sources: [https://github.com/tldraw/tldraw/tree/main/apps/examples/src/examples/use-cases/slides | vendor]; [.../use-cases/slideshow | vendor]; [https://tldraw.dev/blog/building-with-tldraw-offline | vendor]; [https://tldraw.dev/llms-full.txt grep | vendor]
- Claim: Snapping is off by default (Ctrl to snap), grid separate; camera constraints, locked shapes exist.
  - Sources: [https://tldraw.dev/llms-full.txt Snapping, Camera options | vendor]
- Claim: The agent starter kit (replacing `@tldraw/ai`) gives the model a simplified JSON projection (`FocusedShape`, `BlurryShape`, `PeripheralShapeCluster`) plus a screenshot, and sanitizes ids.
  - Sources: [https://github.com/tldraw/tldraw/tree/main/templates/agent/shared/format | vendor]; [https://tldraw.dev/docs/ai | vendor]; [https://tldraw.dev/starter-kits/agent | vendor]

#### OQ1.3 How other editors draw

- Claim: Pitch public player: DOM, `div.slide` at 1920 px with `transform: scale()`, blocks `position: absolute` with translate, shapes inline SVG, text in Quill-class read-only editors [class reading is author estimate]; no `<canvas>`. Stack ClojureScript/UIx/React.
  - Sources: [first-hand DOM inspection, Pitch public player, 2026-10-20 | practitioner]; [https://building.nubank.com/building-elegant-interfaces-with-clojurescript-react-and-uix/ | practitioner]; [https://github.com/pitch-io/uix | vendor]; [https://startup.jobs/frontend-performance-engineer-mfd-at-pitch | vendor]
  - Notes: player only, not the logged-in editor.
- Claim: Google Slides editor: SVG slide whose group ids follow API object ids; text as SVG `<text>`; input via hidden iframes; no canvas. Google Docs (not Slides) moved to canvas in 2021.
  - Sources: [first-hand DOM inspection, Google Slides, 2026-10-20 | practitioner]; [https://workspaceupdates.googleblog.com/2021/05/Google-Docs-Canvas-Based-Rendering-Update.html | vendor]; [https://thenewstack.io/google-docs-switches-to-canvas-rendering-sidelining-the-dom/ | journalism]
- Claim: Figma Slides: C++/WASM renderer to `<canvas>` via WebGL/WebGPU, custom text, so spellcheck built in-house.
  - Sources: [https://newsletter.pragmaticengineer.com/p/building-figma-slides-with-noah-finer | journalism]; [https://www.figma.com/blog/figma-rendering-powered-by-webgpu/ | vendor]
- Claim: Canva and PowerPoint for the web: not determined (Cloudflare; cross-origin frame). Canva job ads describe a Rust/wGPU/WASM next-gen renderer.
  - Sources: [https://jobs.blackbird.vc/companies/canva/jobs/72657275-lightspeed-renderer-senior-software-engineer | vendor]; [https://www.techinterview.org/companies/canva/ | search summary | practitioner]; [first-hand network inspection, PowerPoint web viewer | practitioner]
- Claim: plugin `deck-stage.js` uses the same scaled 1920x1080 DOM stage with a `noscale` mode for the exporter; Blattomat chose DOM + moveable/selecto for "Sharp text in the PDF, the same markup renders the PDF".
  - Sources: [~/Code/pgoell-claude-tools/plugins/presentations/skills/creating-presentations/assets/deck-stage.js | practitioner]; [~/Code/blattwerk/docs/spec-v0.md "Architecture" | practitioner]

#### OQ1.4 Trade-offs DOM vs canvas

- Claim (for DOM text): canvas editors must rebuild selection, clipboard, IME, spellcheck; Konva's own docs list textarea-overlay limits (no rewrap on resize, stale metrics on late fonts, one style per node, two undo timelines, export drawn by canvas); Konva PDF is raster; Figma built its own spellcheck.
  - Sources: [https://konvajs.org/docs/sandbox/Editable_Text.html | vendor]; [https://konvajs.org/docs/sandbox/Canvas_to_PDF.html | search summary | vendor]; [https://lists.w3.org/Archives/Public/public-canvas-api/2010JulSep/0028.html | search summary | practitioner]; [https://web-standards.dev/news/2026/09/build-my-own-text-editor/ | search summary | practitioner]; [Pragmatic Engineer | journalism]
- Claim (for canvas): GeekyAnts saw DOM drag lag at 50+ elements with rotations, shadows, masks; Konva at 60 FPS with 250+ objects; but had to overlay HTML for caret and selection and wait for fonts in headless export.
  - Sources: [https://geekyants.com/blog/building-a-production-ready-canva-like-editor-with-konva-js-react-19-and-next-js-15 | practitioner]
- Claim: slides rarely hold hundreds of blocks [author estimate; not measured].
- Contradictions: tldraw "10,000 shapes might only render 50" vs GeekyAnts lag at 50+. See Reconciled 3.

#### OQ1.5 Drag/resize/select libraries (licenses read)

- Claim: moveable/react-moveable MIT (2019 Daybrush), selecto MIT (2020 Daybrush). Last release moveable 0.53.0 / react-moveable 0.56.0 / selecto 1.26.3, all 2023-12-03; 454 and 94 open issues; issue #1137 "Is this project still active?" and #1156 (2026-09-16) maintainership offer unanswered; daybrush's focus moved to egjs; an outsider plans to publish a fork. New MIT `frameable` 0.2.1 (2026-09) is immature (groups uncertain).
  - Sources: [https://raw.githubusercontent.com/daybrush/moveable/master/LICENSE | vendor]; [https://raw.githubusercontent.com/daybrush/selecto/master/LICENSE | vendor]; [`gh api`, `npm view` | vendor]; [https://github.com/daybrush/moveable/issues/1137 | practitioner]; [https://github.com/daybrush/moveable/issues/1156 | practitioner]; [`npm view frameable` | vendor]
- Claim: Blattomat's Editor.tsx (5,874 lines) carries at least ten documented moveable workarounds (selector first-match, no-height lines, collapsed flat-line selections, missing drag-end event, whole-pixel snapping off by two, re-measure each step, swallowed taps), open issue #732 (`onResizeGroup` during React commit), closed #201 (server block during drag ends drag; fix: hold server changes while pointer down), WebKit flakes #225/#932; 27 xfail/skip lines across 19 test files.
  - Sources: [~/Code/blattwerk/frontend/src/pages/Editor.tsx lines 500, 946, 954, 1484-1488, 3191, 3200, 3433, 5230, 2677-2698 | practitioner]; [https://github.com/pgoell/blattwerk/issues/732 | practitioner]; [https://github.com/pgoell/blattwerk/issues/201 | practitioner]; [~/Code/blattwerk/tests/ | practitioner]
- Claim: Alternatives (all MIT): interact.js (active, 2026-08; events only, no handles or guides [author estimate]); Konva/react-konva (active 2026-10-15; canvas; no SVG export in core); Fabric.js (v7.4.0 2026-05; canvas with `toSVG`; hidden textarea for text); Excalidraw (canvas 2D, textarea overlay).
  - Sources: [https://raw.githubusercontent.com/taye/interact.js/main/LICENSE | vendor]; [https://raw.githubusercontent.com/konvajs/konva/master/LICENSE | vendor]; [https://raw.githubusercontent.com/konvajs/react-konva/master/LICENSE | vendor]; [https://raw.githubusercontent.com/fabricjs/fabric.js/master/LICENSE | vendor]; [https://raw.githubusercontent.com/excalidraw/excalidraw/master/LICENSE | vendor]; [Konva and Fabric source greps | vendor]
- Contradictions: moveable last activity dates differ between clusters. See Reconciled 2.
- Recommendation basis: keep DOM blocks and port Blattomat's canvas; do not adopt tldraw; plan for moveable as an unmaintained dependency (vendor it or fork it, keep its use behind one module, watch the #1156 fork). The MIT `@tldraw/store`/`tlschema` packages are usable as references for a record store without the license.

### OQ2: How do Google Slides, Pitch and PowerPoint model masters, layouts and placeholders, and what must the JSON hold? (cluster masters-layouts, plus export-chart-libs SQ1, local-prior-art SQ3/SQ4)

#### OQ2.1 OOXML

- Claim: Three tiers plus theme: master (`cSld` shapes, `txStyles` with `titleStyle`/`bodyStyle`/`otherStyle`, nine levels each, `clrMap`, `sldLayoutIdLst`, timing, transition, hf); layout ("template slide design that can be applied to any existing slide", attrs `type` ST_SlideLayoutType, `matchingName`, `preserve`, `showMasterSp`); slide reaches master only through its layout relationship. Ids for masters and layouts are unsignedInt >= 2147483648.
  - Sources: [https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/sldMaster.html | ECMA-376 mirror | independent]; [https://learn.microsoft.com/en-us/office/open-xml/presentation/working-with-slide-masters | vendor]; [https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/sldLayout.html | independent]; [.../ST_SlideLayoutType.html | independent]; [.../txStyles.html, bodyStyle.html | independent]; [.../ST_SlideLayoutId.html, ST_SlideMasterId.html | independent]; [scratchpad python-pptx experiment XML | local experiment | practitioner]
- Claim: Placeholder `p:ph` has `type` (default `obj`), `idx` (default 0; "used when applying templates or changing layouts to match a placeholder"), `sz`, `orient`, `hasCustomPrompt`. Layout inherits from master by `type` only; slide inherits from layout by `idx` only. Footer/date/slide number carry their own idx (10, 11, 12 in the default template). Placeholders can be autoshapes, pictures, graphic frames (table, chart), not groups or connectors.
  - Sources: [https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/ph.html | independent]; [https://python-pptx.readthedocs.io/en/latest/user/placeholders-understanding.html | practitioner]; [https://python-pptx.readthedocs.io/en/latest/dev/analysis/placeholders/layout-placeholders.html | practitioner]; [https://python-pptx.readthedocs.io/en/latest/dev/analysis/placeholders/slide-placeholders/index.html | practitioner]
- Claim: Inheritance in XML: a slide placeholder with empty `<p:spPr/>` inherits position, size, geometry; any `<a:xfrm>` on the slide pins it; reapplying the layout removes direct size and position (not colour or font). PowerPoint does not clone `dt`, `ftr`, `sldNum` onto new slides; it adds `<a:spLocks noGrp="1"/>`, drops `hasCustomPrompt`.
  - Sources: [python-pptx dev analysis pages above | practitioner]; [https://python-pptx.readthedocs.io/en/stable/dev/analysis/placeholders/slide-placeholders/placeholders-in-new-slide.html | practitioner]; [scratchpad XML | practitioner]
- Claim: Theme: `clrScheme` of 12 required slots (dk1 lt1 dk2 lt2 accent1-6 hlink folHlink); master `clrMap` maps bg1/tx1/bg2/tx2 onto them; layouts/slides use `clrMapOvr`; fonts as major/minor (`+mj-lt`, `+mn-lt`) with per-script fallbacks.
  - Sources: [https://webapp.docx4java.org/OnlineDemo/ecma376/DrawingML/clrScheme.html | independent]; [.../PresentationML/clrMap.html | independent]; [https://learn.microsoft.com/en-us/office/open-xml/presentation/working-with-slide-layouts | vendor]; [scratchpad theme1.xml | practitioner]
- Claim: PowerPoint behaviour: master edits reach slides; edited layouts need reapply on existing slides; content placement on layout switch follows placeholder creation order, not name; orphans land at the slide edge ("no out-of-the-box way").
  - Sources: [https://support.microsoft.com/en-us/office/what-is-a-slide-master-b9abb2a0-7aef-4257-a14e-4329c904da54 | vendor]; [https://www.indezine.com/products/powerpoint/learn/interface/2013/reset-slide.html | practitioner]; [https://learn.microsoft.com/en-us/answers/a/11388916 | Microsoft Q&A, Korchok | practitioner]
- Claim: Renderers disagree: LibreOffice bug 104202 (idx 4294967295 means no inheritance in PowerPoint, Impress inherited anyway); Keynote-for-iCloud pptx misread in Impress (128547). The plugin verifies exports in LibreOffice.
  - Sources: [https://bugs.documentfoundation.org/rest/bug/104202/comment | independent]; [https://bugs.documentfoundation.org/rest/bug/128547/comment | independent]; [exporting-presentations-to-pptx SKILL.md | practitioner]
- Claim: Placeholders matter for accessibility: Outline view and screen readers see placeholder text, not free text boxes; one WebAIM voice calls it "best practice", not a requirement.
  - Sources: [https://elgin.edu/accessibility/accessibility-in-presentations/slide-layouts/ | independent]; [https://www.indezine.com/products/powerpoint/learn/interface/365/text-placeholders-vs-boxes.html | practitioner]; [https://webaim.org/discussion/mail_message?id=41715 | search summary | practitioner]
- Gap noted: no source states the full text-style lookup order (Okapi page 404).

#### OQ2.2 Google Slides API

- Claim: Page types SLIDE, MASTER, LAYOUT, NOTES, NOTES_MASTER; a slide names both `layoutObjectId` and `masterObjectId` (read-only). Placeholder `{type, index, parentObjectId}`, types mirror ST_PlaceholderType; inheritance by leaving a property unset (`INHERIT`). Only shapes can be placeholders; API cannot create placeholders, layouts or masters, nor change a slide's layout. `createSlide` takes a layout reference and `placeholderIdMappings` (type+index or object id). Only the master's 12-slot colour scheme is editable; `ThemeColorType` mirrors OOXML slots and clrMap names.
  - Sources: [https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages | vendor]; [.../presentations.pages/other | vendor]; [.../presentations.pages/shapes | vendor]; [https://developers.google.com/workspace/slides/api/concepts/page-elements | vendor]; [https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/request | vendor]; [https://developers.google.com/workspace/slides/api/guides/create-slide | vendor]; [https://developers.google.com/apps-script/reference/slides/slide | vendor]; [https://developers.google.com/apps-script/reference/slides/presentation | vendor]
- Claim: Editor: Theme Builder (formerly Master) above layouts; Apply layout per slide.
  - Sources: [https://support.google.com/docs/answer/1705254 | vendor]; [https://www.brightcarbon.com/blog/editing-themes-and-layouts-in-google-slides/ | practitioner]; [https://github.com/scanny/python-pptx/issues/864 | practitioner]

#### OQ2.3 Pitch, Keynote, Gamma, Figma Slides

- Claim: Pitch uses slide styles (tokens: colours, fonts, background, text styles, chart palette; colour edits update all elements; Update/Reset per text style) plus a fixed workspace-wide layout gallery that "dynamically matches the style"; no placeholders documented; pptx export lists animations unsupported and says nothing on layouts.
  - Sources: [https://help.pitch.com/en/articles/4059534-create-your-own-slide-style | vendor]; [https://help.pitch.com/en/articles/4063049-work-with-slide-styles | vendor]; [https://help.pitch.com/en/articles/7003075-add-slides-using-layouts | vendor]; [https://help.pitch.com/en/articles/4056913-find-and-use-templates | vendor]; [https://help.pitch.com/en/articles/6713988-export-a-presentation-to-power-point | vendor]
- Claim: Keynote: slide layouts with tagged text, media and one object placeholder; non-placeholder objects become uneditable background; layout edits reach all slides; tags route content on layout change; reapply resets style and position, keeps content.
  - Sources: [https://support.apple.com/guide/keynote/add-and-edit-master-slides-tan7a2b69972/mac | vendor]; [https://support.apple.com/guide/keynote/change-the-slide-layout-or-theme-tan584189747/mac | vendor]; [https://help.apple.com/keynote/mac/9.0/en.lproj/tan584189747.html | search summary | vendor]
- Claim: Gamma: themes restyle and keep content; pptx export sometimes a locked image per slide, line-by-line text; possible blank slide master (WPS, competitor, search summary).
  - Sources: [https://help.gamma.app/en/articles/10262646-how-do-i-change-my-gamma-theme | vendor]; [https://help.gamma.app/en/articles/15939201-why-doesnt-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma | vendor]; [https://ideas.gamma.app/ideas/p/fully-editable-powerpoint-export-title-subtitle-all-text-graphics-speaker-notes | practitioner]; [https://www.wps.com/blog/gamma-ppt-fix-exported-decks-in-wps-presentation/ | vendor (competitor)]
- Claim: Figma Slides: cannot change a slide's layout after creation; template changes do not reach existing decks (2024 to 2026); pptx export swaps fonts, gradients to solid, code blocks to images. Pitchdeck plugin makes PowerPoint masters from Figma components.
  - Sources: [https://forum.figma.com/ask-the-community-7/how-do-i-change-the-layout-of-an-existing-slide-in-figma-slides-46971 | vendor]; [https://forum.figma.com/suggest-a-feature-11/figma-slides-publishing-template-changes-should-auto-apply-to-existing-slide-decks-23115 | practitioner]; [https://forum.figma.com/ask-the-community-7/slides-template-creating-editing-slide-layouts-template-styles-40621 | practitioner]; [https://help.figma.com/hc/en-us/articles/24848334599447 | vendor]; [https://forum.figma.com/report-a-problem-6/issues-exporting-slides-to-pptx-42186 | practitioner]; [https://docs.hypermatic.com/pitchdeck/tutorials/add-master-slides-to-powerpoint-exports-from-figma.md | vendor]

#### OQ2.4 What the JSON must hold

- Claim: Current exporter and real output: the plugin's exporter mentions no layouts or placeholders; all seven per-deck generators start from `Presentation()` and use the Blank layout; across the ten shipped pptx files `<p:ph` occurs once in total and no file holds a chart part.
  - Sources: [exporting-presentations-to-pptx SKILL.md and references/python-pptx-recipes.md | practitioner]; [private deck repo, aggregate count | local inspection | practitioner]
- Claim: Presets hold CSS tokens, guideline prose and HTML example slides; no structured master/layout file; extracting-presets already walks slide to layout to master to theme and reads `txStyles`, placeholder frames and `clrMap`; colour tokens are named by role, not by OOXML slot.
  - Sources: [presets/README.md "Directory shape" | practitioner]; [extracting-presets/references/slide-masters.md | practitioner]; [extracting-presets/references/template-libraries.md | practitioner]; [presets/default/colors.css | practitioner]
- Claim: python-pptx cannot add layouts or masters (issue #413 open since 2018); route is a starting pptx holding the layouts. Default `Presentation()` is 4:3, unlike the spec's 12192000 x 6858000.
  - Sources: [https://github.com/scanny/python-pptx/issues/413 | practitioner]; [https://github.com/scanny/python-pptx/issues/1109 | practitioner]; [https://github.com/scanny/python-pptx/issues/175 | practitioner]; [https://python-pptx.readthedocs.io/en/latest/user/concepts.html | vendor]; [scratchpad python-pptx experiment | practitioner]
- Claim: PptxGenJS `defineSlideMaster` makes one OOXML layout per call under one shared master; layouts lack `type`/`matchingName`; slide placeholders write their own `a:xfrm` and font size (not inherited) and keep `hasCustomPrompt`; idx starts at 102; theme only head/body font, scheme colours stay stock Office; several open master bugs (#1453 duplicate placeholder content, #1208, #856 tables, #1422, #1443) and #1449 one `<Override>` per slide, repair dialog with several masters (still in current source).
  - Sources: [https://gitbrent.github.io/PptxGenJS/docs/masters/ | vendor]; [https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/pptxgen.ts#L526-L537 | vendor]; [.../src/gen-xml.ts#L1405-L1412 | vendor]; [scratchpad PptxGenJS 4.0.1 experiment | practitioner]; [https://github.com/gitbrent/PptxGenJS/issues/1453, 1208, 856, 1422, 1443, 1449, 604 | practitioner]; [Anthropic pptx skill (local synced copy) | practitioner]
- Claim (derived field list, from evidence): per layout: id, name, layout kind (ST_SlideLayoutType), owning master; per slot: role (`ph@type`) and index (`ph@idx`), box, prompt; per block with `slot`: inherit unless own box set (empty `spPr`); master: text styles by level (title, body, other), fixed shapes, footer trio, background; theme: 12 colour slots plus major/minor fonts; tokens mapped onto slots.
  - Sources: [masters-layouts SQ4 synthesis of ECMA, python-pptx, Google sources above]; [/home/pascal/Code/.worktrees/decks/research/docs/spec-v0.md "Deck document" | practitioner]
- Contradictions: PptxGenJS "first-class Layouts" vs its XML. See Reconciled 4.
- Outlier: spec's "A master is a layout" conflates two tiers. See Outliers.

### OQ3: How does Slidev let an agent write slides, and is a text form worth having beside the JSON? (cluster text-forms)

#### OQ3.1 Slidev

- Claim: MIT (2020-PRESENT Anthony Fu); v53.0.0 2026-09-16; ~49k stars. Markdown split by `---`, YAML headmatter/frontmatter, notes as trailing HTML comments, Vue components, layouts as Vue components with `::slot::` sugar; 19 built-in layouts; clicks via `v-click`/`v-after`/`v-clicks`/`v-motion`; transitions per direction; presenter view.
  - Sources: [https://github.com/slidevjs/slidev/blob/main/LICENSE | vendor]; [https://github.com/slidevjs/slidev/releases | vendor]; [https://sli.dev/guide/syntax | vendor]; [https://sli.dev/guide/layout | vendor]; [https://sli.dev/builtin/layouts | vendor]; [https://sli.dev/features/slot-sugar | vendor]; [https://sli.dev/guide/animations | vendor]; [https://sli.dev/guide/ui | vendor]
- Claim: Free positioning via `v-drag`, written back into Markdown by regex (docs warn; bugs 1959, 2365).
  - Sources: [https://sli.dev/features/draggable | vendor]; [https://github.com/slidevjs/slidev/issues/1959 | practitioner]; [https://github.com/slidevjs/slidev/issues/2365 | practitioner]
- Claim: Export: PDF/PNG; `--format pptx` = images plus notes; `--format pptx-editable` (PR 2722, 2026-09-16) walks the rendered DOM and writes native shapes through pptxgenjs; no masters, no font embedding, no gradients, SVG stays picture, wraps may differ; per-slide fallback to image; one slide per click step, no native animations.
  - Sources: [https://sli.dev/guide/exporting | vendor]; [https://github.com/slidevjs/slidev/pull/1603 | vendor]; [https://github.com/slidevjs/slidev/pull/2722 | vendor]; [https://github.com/slidevjs/slidev/pull/2758 | vendor]
- Claim: Agent surface: official skill generated from docs; VS Code LM tools; built-in MCP (2026-07) with list/get/update/insert/remove/move/goto slide, addressed by 1-based number, frontmatter patch, no screenshot tool.
  - Sources: [https://sli.dev/guide/work-with-ai | vendor]; [https://github.com/slidevjs/slidev/blob/main/skills/slidev/SKILL.md | vendor]; [https://sli.dev/features/mcp | vendor]; [https://github.com/slidevjs/slidev/blob/main/packages/slidev/node/mcp/server.ts | vendor]
- Contradictions: pkgpulse says pptx is not core; docs/PRs show it shipped. Reconciled 9.

#### OQ3.2 Typst with Touying

- Claim: Typst Apache-2.0; Touying MIT (0.8.0, 2026-09-17, needs Typst 0.15); headings make slides, `#pause`/`#meanwhile` subslides, pdfpc notes; output PDF native, HTML experimental; pptx only as images (touying-exporter, MIT) or "vibe coding" typ2pptx (MIT, 50 stars); Tylina WYSIWYG editor is proprietary ("All rights reserved").
  - Sources: [https://github.com/typst/typst/blob/main/LICENSE | vendor]; [https://github.com/touying-typ/touying/blob/main/LICENSE | vendor]; [https://github.com/touying-typ/touying/blob/main/README.md | vendor]; [https://touying-typ.github.io/docs/intro | vendor]; [https://typst.app/docs/reference/html/ | vendor]; [https://github.com/touying-typ/touying-exporter | vendor]; [https://github.com/touying-typ/typ2pptx | vendor]; [https://github.com/tylina/tylina.github.io/blob/main/LICENSE | vendor]; [https://tylina.github.io/ | vendor]; [https://github.com/touying-typ/seaslides | vendor]
- Claim: LLMs are weaker at Typst: TeXFix-Bench compile rate Markdown 90.2%, LaTeX 74.2%, Typst 60.3% (repair task); TypstBench 23 tasks, disputed scoring; forum and HN split; Touying 0.8 breaking changes.
  - Sources: [https://arxiv.org/html/2608.07617 | TeXFix-Bench | independent]; [https://forum.typst.app/t/benchmarking-llms-on-typst/4085 | practitioner]; [https://forum.typst.app/t/llm-code-generation/1331 | practitioner]; [https://hn.svelte.dev/item/45693978 | practitioner]; [https://skills.sh/ypares/agent-skills/typst-writer | practitioner]
- Contradictions: Reconciled 10.

#### OQ3.3 Text vs JSON for agents; round-trip prior art

- Claim: Aider: four models scored lower writing code inside JSON than Markdown; strict JSON mode did not help. "Let Me Speak Freely?" finds format restrictions hurt reasoning; .txt's rerun disputes it.
  - Sources: [https://aider.chat/2024/08/14/code-in-json.html | practitioner]; [https://arxiv.org/abs/2408.02442 | independent]; [https://blog.dottxt.ai/say-what-you-mean.html | vendor]
- Claim: JSON Whisperer: LLMs fail at array index arithmetic; stable-key dictionaries fix it; patches cut tokens 31% at near-equal quality.
  - Sources: [https://arxiv.org/abs/2510.04717v1 | independent]
- Claim: ACE (scene graph, 98 semantic tools) beat a Claude-Code-on-HTML baseline on instruction following (4.23 vs 3.81, p = .010, 94 tasks), visual quality not significantly different; ablation scene graph beat OpenXML; absolute coordinates produce a "coordinate-pushing" regime with spatial hallucinations; HTML "over-applies to whole rows". PPTArena: XML-only edits worst, python-pptx plus refinement loop best. Talk to Your Slides: structured editing beat GUI agents. PPTC: GPT-4 6% session accuracy, errors accumulate over turns.
  - Sources: [https://arxiv.org/html/2608.24103v1 | ACE | independent]; [https://huggingface.co/papers/2608.24103.md | ACE | independent]; [https://arxiv.org/html/2512.03042 | PPTArena | independent]; [https://arxiv.org/abs/2505.11604 | Talk to Your Slides | independent]; [https://arxiv.org/abs/2311.01767 | PPTC | independent]
  - Notes: ACE is a preprint; one author affiliation (Miridih) is not described in the paper.
- Claim: Brosse (one author, one deck): Quarto/reveal.js text practical, API formats yield "a program that produces the deck", python-pptx defects silent.
  - Sources: [https://nbrosse.github.io/posts/llm-slides/llm-slides.html | practitioner]
- Claim: Token size estimate: the spec's one-slide example is ~255 tokens compact JSON vs ~141 in an invented Markdown-like sketch (about 1.8x), OpenAI tokenizer, one slide.
  - Sources: [author estimate, text-forms SQ3]
- Claim: No product keeps a JSON source with an editable two-way text projection. Slidev Studio (MIT, 2026-08) keeps Markdown canonical and writes drags as `v-drag`; edits on canvas only where markup round-trips exactly; undo breaks on renumbering; frontmatter text unreachable. Quarto visual editor normalizes Markdown. Tylina keeps `.typ` canonical. PowerPoint Outline view shows only placeholder text.
  - Sources: [https://github.com/BobTheShoplifter/slidev-addon-studio | vendor]; [https://github.com/BobTheShoplifter/slidev-addon-studio/blob/main/LICENSE | vendor]; [https://github.com/slidevjs/slidev/pull/2709 | vendor]; [https://quarto.org/docs/visual-editor/markdown.html | vendor]; [https://tylina.github.io/ | vendor]; [https://www.indezine.com/products/powerpoint/learn/interface/365/text-placeholders-vs-boxes.html | practitioner]
- Claim: Read-only text projections are common: Gamma `read_gamma` ("not raw HTML or JSON"), Pitch MCP Markdown, Anthropic pptx skill via `markitdown`, Google Slides MCP "verbalized" structures; marp-mcp embeds stable slide ids in Markdown comments; the plugin already has `deck.md` as content storyboard.
  - Sources: [https://developers.gamma.app/llms-full.txt | vendor]; [https://help.pitch.com/en/articles/16849368-an-overview-of-pitch-s-mcp | vendor]; [https://github.com/anthropics/skills/blob/main/skills/pptx/SKILL.md | vendor]; [https://developers.google.com/workspace/slides/api/guides/configure-mcp-server | vendor]; [https://github.com/masaki39/marp-mcp/blob/main/src/utils/slide-id.ts | vendor]; [designing-presentations SKILL.md | practitioner]

#### OQ3.4 How agent slide tools expose decks

- Claim: Spread of interfaces: Slidev MCP (Markdown per slide by number); marp-mcp (Markdown by stable id; typed layout schemas); Google Slides (JSON batchUpdate by objectId, atomic; official MCP in developer preview); Gamma (natural-language edit per page, no object addressing, text reads); Pitch (template-variable fill or agent prompt; Markdown reads; no element ops); Canva MCP (element ids inside transactions that fail on concurrent edits); Office.js (shapes by id, points); Copilot (chat; thin evidence); Office-PowerPoint-MCP-Server (index-addressed python-pptx tools); Anthropic pptx skill (pptxgenjs scripts, raw XML edits; proprietary license).
  - Sources: [https://sli.dev/features/mcp | vendor]; [https://github.com/masaki39/marp-mcp | vendor]; [https://developers.google.com/workspace/slides/api/guides/batch | vendor]; [https://developers.gamma.app/reference/common-feature-requests | vendor]; [https://help.pitch.com/en/articles/16849368-an-overview-of-pitch-s-mcp | vendor]; [https://www.canva.dev/docs/mcp/tools/commit-editing-transaction/ | vendor]; [https://www.canva.dev/docs/mcp/tools/perform-editing-operations/ | vendor]; [https://learn.microsoft.com/en-us/office/dev/add-ins/powerpoint/shapes | vendor]; [https://m365admin.handsontek.net/microsoft-365-copilot-updates-powerpoint-on%E2%80%91canvas-features-agent-mode/ | search snippets | practitioner]; [https://github.com/GongRzhe/Office-PowerPoint-MCP-Server | vendor]; [https://github.com/anthropics/skills/blob/main/skills/pptx/LICENSE.txt | vendor]
- Claim: Blattomat exposes only JSON operations to agents and teaches them with OKF Markdown docs.
  - Sources: [/home/pascal/Code/blattwerk/docs/okf/index.md | practitioner]
- Recommendation basis: no two-way text form in v0; add a read-only outline tool (per slide: title, text, block ids and types) beside JSON ops by id; keep `deck.md` as a planning input an agent turns into ops.

### OQ4: How do Gamma and Pitch let an AI change a deck, and what happens to hand-placed blocks? (cluster ai-editing, comments-pointing SQ3, local-prior-art)

- Claim: Gamma 5 (2026-10-06) mixes agent edits with a freeform canvas; scopes: whole deck via chat, one slide via sparkle, one element via AI editing mode (selected elements shown in chat); before/after preview; acceptance per whole request only; reject restores the prior version; CMD+Z and version history (restore replaces whole gamma). Keeping text requires saying so in the prompt.
  - Sources: [https://web.archive.org/web/20261010014412/https://gamma.app/insights/introducing-gamma-5-your-story-your-style-your-tools | vendor]; [https://help.gamma.app/en/articles/8033284-how-to-edit-content-using-ai | vendor]; [https://help.gamma.app/en/articles/8033284-can-i-edit-my-content-using-ai | vendor]; [https://help.gamma.app/en/articles/11048579-can-i-undo-a-change-or-restore-a-previous-version-in-gamma | vendor]; [https://borncity.com/news/gamma-5-gamma-tech-stellt-den-groessten-umbau-seiner-praesentationsplattform-vor/ | journalism]; [https://rar.design/posts/gamma-5-ai-presentation-rebuild-guide | practitioner]; [https://explainx.ai/blog/gamma-5-agent-driven-presentations-ai-smell-what-changed-2026 | practitioner]
- Claim: Gamma cards are fluid by default, fixed 16:9 optional; API dimensions include `fluid`, `pageless`.
  - Sources: [https://help.gamma.app/en/articles/11016396-what-are-cards-in-gamma-and-how-do-they-work | vendor]; [https://developers.gamma.app/llms-full.txt | vendor]
- Claim: Gamma API/MCP: generate (agent and classic), edit one page by prompt (409 on concurrent edit, cannot add pages, may be partially applied on failure, returns a summary), no object-level addressing, read as text, comments readable, no webhooks.
  - Sources: [https://developers.gamma.app/llms-full.txt | vendor]; [https://developers.gamma.app/docs/generate-api-parameters-explained | vendor]
- Claim: User reports: one targeted AI action "undid a whole bunch of my own manual edits on multiple other unrelated cards" (restored via history; Gamma staff asked for detail); MCP "generate-only" made iteration "pretty painful" (Gamma staff); Gamma 5 seen as loss of fine control; import test kept text and table, swapped fonts, realigned two titles.
  - Sources: [https://ideas.gamma.app/ideas/p/gamma-ai-clobbered-my-edits | practitioner]; [https://ideas.gamma.app/ideas/p/add-update-edit-support-to-gamma-mcp-not-just-generation | practitioner]; [https://reddit.com/r/GammaApp/comments/1x6a2yi/ | practitioner]; [https://reddit.com/r/GammaApp/comments/1x4xmqb/ | practitioner]; [https://reddit.com/r/GammaApp/comments/1wpmgsm/ | practitioner, weight lowered by unverified sponsorship claim]; [https://reddit.com/r/powerpoint/comments/1uyp4w8/ | sponsorship claim | practitioner]
- Claim: Pitch: AI actions on a text block, whole-slide edits from a quick menu, Pitch Agent (2026-05) for deck and selection (select a block for Quick actions), "Ask Agent to edit the slide you're on" (2026-08), Anything block (2026-10); agent builds from template slides and "150+ additional layouts"; output "fully editable", "you're expected to review it"; models from Anthropic, OpenAI, Google. No source on how Pitch treats hand-placed blocks or whether it previews.
  - Sources: [https://pitch.com/whats-new | vendor]; [https://web.archive.org/web/20260811205908/https://help.pitch.com/en/articles/12771746-pitch-s-ai-actions | vendor]; [https://help.pitch.com/en/articles/14981091-pitch-agent | vendor]; [https://pitch.com/ai-in-pitch | vendor]; [https://pitch.com/blog/introducing-pitch-agent | vendor]; [https://help.pitch.com/en/articles/14844265-create-a-new-presentation | vendor]; [https://pitch.com/use-cases/ai-presentation-maker | vendor]; [https://plusai.com/blog/pitch-app-review | vendor (competitor)]
- Claim: Copilot in PowerPoint: chat edits (GA 2026); "Formatting, structure, and branding will be preserved" (Microsoft); rewrite only whole text boxes, no tables/charts; save a copy before translate; reads placeholder type/position, not text; Brand Kit strict adherence ("won't add or remove placeholders or invent new layouts", owner-set); per-slide note steering ("Don't modify this slide.") as guidance; earlier field reports of one layout reused; decorative shapes mistaken for placeholders unless layouts named.
  - Sources: [https://mc.merill.net/message/RM548646 | vendor]; [https://www.itelio.com/en/microsoft-message-center/MC1219792 | vendor]; [https://www.itelio.com/en/microsoft-message-center/MC1243550 | vendor]; [https://support.microsoft.com/en-au/topic/agent-mode-frontier-in-powerpoint-008f17aa-8e5f-4cda-ba6c-0588000bdad7 | vendor]; [https://support.microsoft.com/en-us/powerpoint/copilot/keep-your-presentation-on-brand-with-copilot | vendor]; [https://support.microsoft.com/en-us/powerpoint/copilot/manage-brand-kit-template-settings-in-powerpoint | vendor]; [https://mc.merill.net/message/MC1473166 | vendor]; [https://learn.microsoft.com/en-us/answers/a/1908366 | practitioner]; [https://learn.microsoft.com/en-us/answers/a/12080966 | practitioner]; [https://docs.primedocs.io/en/docs/4.0/layouter/best-practices/microsoft-365-copilot | vendor]
- Claim: Gemini in Slides: one slide at a time, preview, Replace / Insert / Close; full-deck outline approved before build. Plus AI Remix: per slide, default rewrites text to fit, strict mode keeps it, "duplicate the slide before remixing". Beautiful.ai: constraint engine, one-way escape to Classic slides. Figma: selection scopes the agent; Make stages changes until Apply; Make users ask for section locks. Canva AI 2.0: claims element-level edits; Brand Template locks exist (full, position-only, page, background); whether AI honours them unread. Tome shut its slide product 2025-04-30.
  - Sources: [https://workspace.google.com/blog/product-announcements/create-on-brand-presentations-in-minutes-with-gemini-in-google-slides | vendor]; [https://support.google.com/docs/answer/16961475?hl=en | vendor]; [https://guide.plusai.com/ai-for-presentations/remix-slides | vendor]; [https://web.archive.org/web/20260306150742/https://www.beautiful.ai/blog/introducing-classic-slides | vendor]; [https://aumiqx.com/ai-tools/beautiful-ai-review-presentation-maker-2026/ | practitioner]; [https://www.figma.com/solutions/prompt-to-edit/ | vendor]; [https://help.figma.com/hc/en-us/articles/42009840449175 | vendor]; [https://forum.figma.com/suggest-a-feature-11/is-there-a-way-to-lock-sections-in-figma-make-to-ensure-they-aren-t-removed-or-revised-with-an-unrelated-prompt-to-another-section-of-the-project-50240 | practitioner]; [https://www.cmswire.com/digital-experience/canva-ai-20-adds-agentic-design-tools/ | journalism]; [https://web.archive.org/web/20260117105657/https://www.canva.com/help/brand-template-locks/ | vendor]; [https://venturebeat.com/ai/tomes-founders-ditch-viral-presentation-app-with-20m-users-to-build-ai | journalism]; [https://www.chatslide.ai/articles/tome-ai-shut-down-what-happened | vendor (competitor)]
- Claim: Review patterns: Tiptap preview vs review modes with per-suggestion accept; diff review mistakes ("Apply-all with no per-hunk control", "No undo after accept"); Microsoft HAX 9C undo automated actions; Ink & Switch Patchwork bots edit on branches you merge partly.
  - Sources: [https://tiptap.dev/docs/ai/ai-toolkit/client/agents/review-changes | vendor]; [https://aiuxplayground.com/glossary/diff-patch-review | practitioner]; [https://www.microsoft.com/en-us/haxtoolkit/pattern/g9-c-undo-automated-actions/ | vendor (research-backed)]; [https://www.inkandswitch.com/patchwork/notebook/2024-version-control/07/ | independent]
- Claim: Local prior art: Blattomat tools refuse locked blocks (`delete_block`, `move_block`, `update_block`; `replace_doc` lists no lock refusal) and stale versions ("Nothing was written"); the plugin editor tells the agent to re-read after hand edits.
  - Sources: [/home/pascal/Code/blattwerk/docs/okf/teacher-does.md | practitioner]; [/home/pascal/Code/blattwerk/docs/okf/tools/errors-and-limits.md | practitioner]; [/home/pascal/Code/blattwerk/docs/okf/versions.md | practitioner]; [/home/pascal/Code/blattwerk/src/blattwerk/sheet.schema.json | practitioner]; [editing-presentations SKILL.md | practitioner]
  - Notes: implementation of the lock refusal was not traced in code; docs and tests are the evidence. The `replace_doc` gap matters: a whole-doc write could bypass locks.
- Contradictions: Gamma "intact" vs clobbered report (Reconciled 11); Copilot licensing (Reconciled 12).
- Recommendation basis: scope = deck, slide, block, all through id-addressed ops; lock = tool refusal including whole-slide/whole-deck writes; every agent write versioned, one undo step, with a summary; preview/branch review deferred to phase 3 but the op log should allow it.

### OQ5: Which libraries for pptx export and charts fit, and what are their licenses? (cluster export-chart-libs, plus masters-layouts SQ4, local-prior-art SQ4)

#### OQ5.1 pptx writers

- Claim: python-pptx: MIT; last release 1.0.2 (2024-08-07), no commits since, 541 open issues, ~60M downloads/month; no layout creation, no SVG (skips it), no font embedding API, no animation API; native charts (most 2D types, data via `CategoryChartData`, embedded workbook via XlsxWriter), tables, runs, notes.
  - Sources: [https://github.com/scanny/python-pptx/blob/master/LICENSE | vendor]; [https://pypi.org/pypi/python-pptx/json | vendor]; [https://api.github.com/repos/scanny/python-pptx | vendor]; [https://pypistats.org/api/packages/python-pptx/recent | vendor]; [https://python-pptx.readthedocs.io/en/latest/user/charts.html | vendor]; [package.py L164, opc/spec.py, chart/xlsx.py | vendor]; [https://github.com/scanny/python-pptx/issues/1112 | practitioner]; [https://github.com/scanny/python-pptx/pull/1115 | practitioner]
- Claim: PptxGenJS: MIT; v4.0.1 (2025-06-26), last push 2025-11, 305 open issues, ~14M downloads/month; layouts (with issues in OQ2.4); charts (10 types, no waterfall), tables, runs, notes; SVG as svgBlip with canvas PNG in the browser but a broken-image PNG in Node; no font embedding (pptx-embed-fonts, MIT, 9 stars adds it); no theme colours; no animations ("no plans"); 2026 repair-dialog bugs (#1441-1444, #1532, #1534 orphan `c:axId` on BAR/LINE charts).
  - Sources: [https://github.com/gitbrent/PptxGenJS/blob/master/LICENSE | vendor]; [https://api.github.com/repos/gitbrent/PptxGenJS/releases | vendor]; [https://api.npmjs.org/downloads/point/last-month/pptxgenjs | vendor]; [https://github.com/gitbrent/PptxGenJS/issues/1324 | practitioner]; [https://gitbrent.github.io/PptxGenJS/docs/api-charts/ | vendor]; [src/gen-xml.ts L576-590, src/gen-media.ts L136-152, src/core-interfaces.ts | vendor]; [https://github.com/gitbrent/PptxGenJS/issues?q=is%3Aopen+repair | practitioner]; [https://github.com/liyao1520/pptx-embed-fonts | practitioner]; [https://github.com/gitbrent/PptxGenJS/issues/327, 704 | vendor]
- Claim: Others: power-pptx (MIT fork, created 2026-04, 13 stars, frequent releases, commits partly by "claude"; claims animations, theme writer, SVG with PNG fallback, JSON authoring; unverified); pptx-automizer (MIT, template-based, Node only, wraps PptxGenJS 3, animations out of scope, cannot add layouts); officegen (MIT, last release 2021); Open XML SDK (MIT, .NET); Aspose.Slides (commercial from $1,399; can add layouts and embed fonts); Aspose.Slides FOSS for Python (MIT, 2026-03, 15 stars; claims animations and 57 transitions); Syncfusion (.NET, commercial or Community License limits).
  - Sources: [https://github.com/codehalwell/power-pptx | vendor]; [https://github.com/codehalwell/power-pptx/blob/master/LICENSE | practitioner]; [https://api.github.com/repos/codehalwell/power-pptx/contributors | vendor]; [https://github.com/singerla/pptx-automizer/blob/main/docs/limitations.md | vendor]; [https://github.com/singerla/pptx-automizer/blob/main/LICENSE | vendor]; [https://github.com/Ziv-Barber/officegen/blob/master/LICENSE | vendor]; [https://github.com/dotnet/Open-XML-SDK/blob/main/LICENSE | vendor]; [https://purchase.aspose.com/pricing/slides/family/ | vendor]; [https://docs.aspose.com/slides/python-net/slide-layout/ | vendor]; [https://docs.aspose.com/slides/python-net/embedded-font/ | vendor]; [https://github.com/aspose-slides-foss/Aspose.Slides-FOSS-for-Python | vendor]; [https://www.syncfusion.com/products/communitylicense | vendor]
- Claim: Plugin exporter today: a procedure (SKILL.md plus recipes), not a program; each deck gets a fresh python-pptx script (seven generators, hundreds to thousands of lines); 1 px = 6350 EMU, 0.5 pt per px; greedy wrap estimator; 13 gotchas; SVG rasterized with PyMuPDF; LibreOffice render-verify loop of 2 to 4 cycles; native-chart recipe exists but no generator ever called `add_chart`.
  - Sources: [exporting-presentations-to-pptx/SKILL.md | practitioner]; [references/python-pptx-recipes.md | practitioner]; [private deck repo, aggregate count | local inspection | practitioner]
  - Notes: "Port of the exporter" in the spec means writing a new program from these recipes; there is no code to port.

#### OQ5.2 Chart libraries

- Claim: ECharts (Apache-2.0 + NOTICE; zrender BSD-3; four files with d3 code under BSD-3): `renderToSVGString` server-side with `ssr: true`, CSS-embedded entry animation; SVG renderer fine below ~1k points; open SVG-renderer bugs (RTL, hover at 2-3k points, data-URI images, Safari CSS container); `dataset.source` 2D array with header row maps columns to series.
  - Sources: [https://github.com/apache/echarts/blob/master/LICENSE | vendor]; [https://github.com/apache/echarts/blob/master/NOTICE | vendor]; [https://github.com/ecomfe/zrender/blob/master/LICENSE | vendor]; [https://echarts.apache.org/handbook/en/how-to/cross-platform/server/ | vendor]; [https://echarts.apache.org/handbook/en/best-practices/canvas-vs-svg/ | vendor]; [https://github.com/apache/echarts/issues?q=is%3Aopen+%22svg+renderer%22 | practitioner]; [https://echarts.apache.org/handbook/en/concepts/dataset/ | vendor]
- Claim: Vega-Lite and Vega (BSD-3): `view.toSVG()`, CLI `vl2svg`; vl-convert (BSD-3) renders to SVG/PNG from Python without Node (v2.0.0-rc7).
  - Sources: [https://github.com/vega/vega-lite/blob/main/LICENSE | vendor]; [https://github.com/vega/vega/blob/main/LICENSE | vendor]; [https://github.com/vega/vega/blob/main/docs/docs/api/view.md | vendor]; [https://vega.github.io/vega-lite/usage/compile.html | vendor]; [https://github.com/vega/vl-convert/blob/main/LICENSE | vendor]; [https://pypi.org/project/vl-convert-python/0.13.1 | vendor]
- Claim: Chart.js MIT but canvas only (bitmap in PDF, `devicePixelRatio` to sharpen); Observable Plot ISC, SSR "only practical for simple plots", last release 2025-02; Recharts MIT but no SSR ("never been explicitly supported", empty output in 3.x); D3 ISC, no chart spec.
  - Sources: [https://github.com/chartjs/Chart.js/blob/master/LICENSE.md | vendor]; [Chart.js docs index, using-from-node-js, device-pixel-ratio | vendor]; [https://github.com/observablehq/plot/blob/main/LICENSE | vendor]; [https://github.com/observablehq/plot/blob/main/docs/getting-started.md | vendor]; [https://github.com/recharts/recharts/blob/main/LICENSE | vendor]; [https://github.com/recharts/recharts/issues/5997 | practitioner]; [https://github.com/recharts/recharts/issues/6139 | practitioner]; [https://github.com/d3/d3/blob/main/LICENSE | vendor]
- Claim: No maintained library maps an ECharts or Vega-Lite spec to a native pptx chart; both pptx libraries take near-identical category data (`{name, labels, values}`; `categories` + `add_series`), so a block holding a plain table (categories, series names, values, chart kind) maps onto both renderers by small hand code. think-cell output is plain PowerPoint charts (consulting norm). Google Slides takes charts only from Sheets.
  - Sources: [author estimate from one search round, export-chart-libs SQ2]; [https://codix.ai/export-apex-charts-as-powerpoint-charts/ | search snippet | practitioner]; [https://gitbrent.github.io/PptxGenJS/docs/api-charts/ | vendor]; [https://python-pptx.readthedocs.io/en/latest/user/charts.html | vendor]; [https://www.thinkcell.com/en/resources/manual/compatibility | vendor]; [https://developers.google.com/workspace/slides/api/guides/add-chart | vendor]
- Claim: Gamma users ask for native charts in pptx export ("painful workflow in consulting", Gamma staff); Pitch charts "may look different".
  - Sources: [https://ideas.gamma.app/ideas/p/native-powerpoint-chart-exports-dont-exported-as-images | practitioner]; [https://help.pitch.com/en/articles/6713988-export-a-presentation-to-power-point | vendor]

#### OQ5.3 Others' HTML/JSON to pptx

- Claim: Slidev editable (DOM walk, pptxgenjs, image fallback per slide); Marp `--pptx-editable` via LibreOffice, "not recommend"; Gamma, Pitch, Figma partial; dom-to-pptx (MIT, ~209k downloads/month) walks the DOM, embeds web fonts, optional vector SVG, template pptx, and writes `p:timing`; old Anthropic html2pptx dropped for direct pptxgenjs scripts.
  - Sources: [https://github.com/slidevjs/slidev/pull/2722 | vendor]; [https://github.com/marp-team/marp-cli | vendor]; [https://github.com/atharva9167j/dom-to-pptx | vendor]; [https://github.com/atharva9167j/dom-to-pptx/blob/master/LICENSE | vendor]; [https://cdn.jsdelivr.net/npm/stableclaw@2026.5.15/skills/pptx/html2pptx.md | mirror | practitioner]

#### OQ5.4 Fonts and SVG

- Claim: PowerPoint Windows/Mac embed TTF/OTF subject to font embeddability flags; web/mobile handling undocumented (one low-quality claim it cannot); LibreOffice 25.8 embeds fonts; Google Slides ignores embedded fonts; Slidev does not embed; Pitch tries, falls back to Calibri; plugin uses widely installed faces (Aptos, Consolas) or the client's corporate font.
  - Sources: [https://support.microsoft.com/en-us/office/benefits-of-embedding-custom-fonts-cb3982aa-ea76-4323-b008-86670f222dbc | vendor]; [https://pdf.it.com/learn/pptx-to-pdf-missing-fonts | search snippet | practitioner]; [https://www.collaboraonline.com/blog/collabora-productivity-contributions-in-libreoffice-25-8/ | vendor]; [https://help.gamma.app/en/articles/15939201-why-doesnt-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma | vendor]; [python-pptx-recipes.md gotcha 11 | practitioner]
- Claim: SVG in PowerPoint: Microsoft 365, 2019+ and Mac support SVG with Convert to Shape; file holds PNG `a:blip` plus `asvg:svgBlip` extension; spec (MS-ODRAWXML) inaccessible.
  - Sources: [https://support.microsoft.com/en-us/office/edit-svg-images-in-microsoft-office-69f29d39-194a-4072-8c35-dbe5e7ea528c | vendor]; [PptxGenJS gen-xml.ts | vendor]; [https://doc.tmssoftware.com/flexcel/net/api/FlexCel.Core/TBlip/SVGBlip.html | search snippet | vendor]
- Recommendation basis: python-pptx (MIT, matches FastAPI, the recipes exist) as the core writer, plus decks' own small OOXML writers for layouts in a generated base file, theme `clrScheme`, `svgBlip` with a Chromium-rendered PNG, `p:transition` and `p:timing`; replace PyMuPDF/CairoSVG with the Playwright Chromium already in the stack for rasterizing (license, see OQ5 license flags). Charts: one table-shaped Chart block, ECharts SVG in browser and PDF, native pptx chart from the same table.

### OQ6: How do PowerPoint, Keynote, Google Slides, reveal.js and Slidev model animations, and what survives pptx? (cluster animations, plus local-prior-art SQ3)

#### OQ6.1 PresentationML

- Claim: One `p:timing` per slide (`tnLst`, `bldLst`), root `tmRoot` with a `mainSeq` `p:seq` (`concurrent`, `nextAc="seek"`, `prevAc`); node types `clickEffect`, `withEffect`, `afterEffect` (and groups); preset classes `entr`, `exit`, `emph`, `path`; `presetID` is plain int with no enumeration in the standard; `presetSubtype` bitflag for direction; `dur`, `accel`, `decel`, `repeatCount` in thousandths, `grpId`, `fill`.
  - Sources: [https://github.com/scanny/python-pptx/blob/master/src/pptx/oxml/slide.py | vendor]; [https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/seq.html | independent]; [.../ST_TLTimeNodeType.html | independent]; [.../ST_TLTimeNodePresetClassType.html | independent]; [.../cTn.html | independent]; [.../stCondLst.html, cond.html, ST_TLPreviousActionType.html | independent]
- Claim: A PowerPoint-saved Fly In (Mac PowerPoint 16) shows the click-group nesting (outer `par` with `delay="indefinite"`, inner `par` delay 0, effect `par` `presetID="2" presetClass="entr" presetSubtype="4"`), `p:set` visibility then two `p:anim` on `ppt_x`/`ppt_y`, `prevCondLst`/`nextCondLst` on `sldTgt`, and `<p:bldP spid="4" grpId="0"/>`; with a p14 feature PowerPoint wraps the whole `p:timing` in `mc:AlternateContent` with a fallback copy.
  - Sources: [https://github.com/scanny/python-pptx/files/14556139/file_with_bounce.pptx | PowerPoint-saved sample | practitioner]; [https://github.com/scanny/python-pptx/issues/954 | practitioner]; [https://github.com/atharva9167j/dom-to-pptx/blob/master/src/animations/xml-templates.js | vendor]
- Claim: Behaviours: `animEffect` (filters such as `fade`, `wipe(left)`), `animScale`, `animClr`, `animMotion` with path letters {M,L,C,Z,E} absolute/relative (subset of SVG path; no H, V, S, Q, T, A [author estimate]), `origin`, `pathEditMode`.
  - Sources: [.../animEffect.html, animScale.html, animClr.html, animMotion.html, ST_TLAnimateMotionPathEditMode.html, ST_TLAnimateMotionBehaviorOrigin.html, tgtEl.html | independent]
- Claim: Builds by paragraph: `bldP build="p"` with `bldLvl`, `rev`, `grpId` per shape; each paragraph effect targets `p:txEl/p:pRg st end`; charts use `p:bldGraphic`; mismatched `grpId` fails silently.
  - Sources: [.../bldP.html, ST_TLParaBuildType.html, txEl.html | independent]; [dom-to-pptx xml-templates.js | vendor]; [https://github.com/iOfficeAI/OfficeCLI/blob/main/src/officecli/Handlers/Pptx/PowerPointHandler.Animations.cs | vendor]; [https://github.com/BramAlkema/svg2ooxml/blob/main/docs/specs/animation-fidelity.md | practitioner]
- Claim: Preset ids for the spec's effects (LibreOffice import table, confirmed by Aspose FOSS): entr appear 1, fly in 2, fade 10, wipe 22, zoom 23; exit reuses the same ids; emph fill colour 1, font colour 3, grow/shrink 6, spin 8, pulse 26 (= flash bulb, COM-observed), teeter 32; path presets own id space, custom path 0; subtypes 1 top, 2 right, 4 bottom, 8 left, 16 in, 32 out. VBA `MsoAnimEffect` numbers differ for emphasis and paths.
  - Sources: [https://github.com/LibreOffice/core/blob/master/oox/source/ppt/commontimenodecontext.cxx | independent]; [https://github.com/aspose-slides-foss/Aspose.Slides-FOSS-for-Python/blob/main/aspose/slides_foss/_internal/pptx/animation_mappings.py | vendor]; [https://github.com/ChristopherVR/ooxml/blob/main/src/core/pptx/core/services/animation-write-mappings-emphasis.ts | vendor]; [https://learn.microsoft.com/en-us/office/vba/api/powerpoint.msoanimeffect | vendor]; [https://bugs.documentfoundation.org/show_bug.cgi?id=169343 | independent]
- Claim: Transitions: `p:transition` on the incoming slide; ECMA choices include fade, push, wipe, zoom, cut; ECMA has only `spd` keywords; a millisecond duration is `p14:dur` inside `mc:AlternateContent` with a fallback; Morph is `p159:morph` (byObject/byWord/byChar), matched by "!!" names; LibreOffice has no Morph (bug open since 2018).
  - Sources: [.../transition.html | independent]; [OfficeCLI PowerPointHandler.Animations.cs | vendor]; [https://github.com/microsoft/Resource2Skill/blob/main/domains/ppt/mcp_server/server.py | vendor]; [https://github.com/scanny/python-pptx/issues/1106 | practitioner]; [https://apireference.aspose.com/slides/java/com.aspose.slides/transitionmorphtype | search excerpt | vendor]; [https://support.microsoft.com/en-us/office/morph-transition-tips-and-tricks-bc7f48ff-f152-4ee8-9081-d3121788024f | vendor]; [https://bugs.documentfoundation.org/show_bug.cgi?id=116736 | independent]
- Claim: User model: Effect Options By Paragraph, one paragraph per click by default; "By 1st Level Paragraphs" default (Indezine).
  - Sources: [https://support.microsoft.com/en-us/PowerPoint/animate-or-make-words-appear-one-line-at-a-time-in-powerpoint | search excerpt | vendor]; [https://www.indezine.com/products/powerpoint/learn/animationsandtransitions/2013/animating-text.html | search excerpt | practitioner]

#### OQ6.2 Keynote, Google Slides, reveal.js, Slidev

- Claim: Keynote: build in/out with Delivery (by bullet group, chart wedge), action builds (opacity, rotate, scale, move along a draggable path), start On Click / With Build / After Build / After Transition with delay; Magic Move matches by object, word, character (no "!!" naming).
  - Sources: [https://support.apple.com/en-sa/guide/keynote/tan72234bb6/mac | vendor]; [https://support.apple.com/guide/keynote/tanf96d92cb6/mac | vendor]; [https://support.apple.com/guide/keynote/tan3ad5f8d82/mac | vendor]; [https://support.apple.com/en-ph/guide/keynote/tanff5ae749e/mac | vendor]; [https://origin-discussions2-us-dr-prz.apple.com/thread/254026886 | practitioner]
- Claim: Google Slides: On click / With previous / After previous, By paragraph, a short effect list; no animation or transition field in the API.
  - Sources: [https://support.google.com/docs/answer/1689475?hl=en | vendor]; [https://helpdeskgeek.com/how-to-add-animation-to-google-slides/ | practitioner]; [https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages | vendor]; [https://workspace.google.com/marketplace/app/creator_studio/509621243108 | vendor]
- Claim: reveal.js fragments (index-based visibility, shared `data-fragment-index`, events), auto-animate by content or `data-id`; Slidev clicks as the step unit, enter/leave ranges, motion variants by click count. Both derive state from the current step index, so going back is a function of the index.
  - Sources: [https://revealjs.com/fragments/ | vendor]; [https://github.com/hakimel/reveal.js/blob/master/js/controllers/fragments.js | vendor]; [https://revealjs.com/auto-animate/ | vendor]; [https://github.com/slidevjs/slidev/blob/main/docs/guide/animations.md | vendor]; [https://github.com/slidevjs/slidev/blob/main/packages/client/composables/useNav.ts | vendor]; [https://github.com/hakimel/reveal.js/blob/master/js/reveal.js | vendor]
- Claim: Plugin and Blattomat: no animation model; plugin builds are copies of a slide (`data-build-step`), used in 13 gallery slides, 0 times in the built decks; all ten shipped pptx hold 0 `<p:timing>` and one `<p:transition>` in total; a preset built from a client template did not carry its animations.
  - Sources: [creating-presentations/references/layouts-and-visuals.md:42 | practitioner]; [presets/*/slides/*.html grep | practitioner]; [private deck repo, aggregate count | local inspection | practitioner]; [/home/pascal/Code/blattwerk/docs/spec-v0.md grep | practitioner]

#### OQ6.3 What survives

- Claim: Writers: PptxGenJS and python-pptx write no animations (maintainers say so; python-pptx may wipe complex timing when adding video); hand XML "silently" dropped on small mistakes; working raw writers exist (dom-to-pptx MIT, OfficeCLI Apache-2.0, Aspose FOSS MIT; svg2ooxml AGPL/commercial with empirical notes: discrete colour dropped, `additive="sum"` on motion broken, `p:set` flipbook works).
  - Sources: [https://github.com/gitbrent/PptxGenJS | vendor]; [https://github.com/gitbrent/PptxGenJS/issues/327 | vendor]; [https://github.com/scanny/python-pptx/issues/264 | vendor]; [https://github.com/scanny/python-pptx/issues/1106 | practitioner]; [https://github.com/atharva9167j/dom-to-pptx | vendor]; [https://github.com/iOfficeAI/OfficeCLI | vendor]; [https://github.com/aspose-slides-foss/Aspose.Slides-FOSS-for-Python | vendor]; [https://github.com/BramAlkema/svg2ooxml/blob/main/docs/specs/animation-fidelity.md | practitioner]
- Claim: Keynote to pptx: no Apple mapping; third-party claims Magic Move becomes fade (unverified); Apple Community: only video guarantees animations.
  - Sources: [https://support.apple.com/guide/keynote/tana0d19882a/mac | vendor]; [http://web.archive.org/web/20141016150659/http://support.apple.com/kb/HT5591 | vendor]; [https://keynote.org.cn/blog/keynote-export-format-guide-en/ | practitioner]; [https://discussions.apple.com/thread/250144385 | practitioner]; [https://www.oreilly.com/library/view/iwork-09-the/9780596802073/ch15s06.html | journalism, dated]
- Claim: pptx to Google Slides: delays lost, timing rounded, motion paths and Morph lost, unknown presets removed or flattened.
  - Sources: [https://www.brightcarbon.com/blog/convert-powerpoint-google-slides/ | practitioner]; [https://slidemodel.com/fix-compatibility-powerpoint-google-slides/ | practitioner]
- Claim: LibreOffice: meta bug 139897 with 27 dependents (master animations not imported, first bullet visible from start on save, teeter random position, custom path id ignored). So the plugin's LibreOffice verify loop cannot check animations.
  - Sources: [https://bugs.documentfoundation.org/show_bug.cgi?id=139897 | independent]; [https://bugs.documentfoundation.org/show_bug.cgi?id=128821 | independent]
- Claim: Effect on an SVG internal by selector has no PowerPoint target once the SVG becomes an image (spec already says it is dropped).
  - Sources: [https://support.microsoft.com/en-us/office/edit-svg-images-in-microsoft-365-69f29d39-194a-4072-8c35-dbe5e7ea528c | vendor]; [spec-v0.md section 7 | practitioner]

#### OQ6.4 Web runtimes and going back

- Claim: Web Animations API: `reverse()`, `finish()`, `cancel()`, `currentTime`, `commitStyles()`; CSS `offset-path: path()` takes SVG path data (widely available since 2022).
  - Sources: [https://developer.mozilla.org/en-US/docs/Web/API/Animation/reverse | independent]; [https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API/Using_the_Web_Animations_API | independent]; [https://developer.mozilla.org/en-US/docs/Web/CSS/offset-path | independent]
- Claim: PowerPoint shows a revisited slide in its final state; file holds `prevAc`/`onPrev` hooks; no Microsoft source on back within a slide; transitions reverse since PowerPoint 2010 (forum quote via LibreOffice bug).
  - Sources: [https://www.rdpslides.com/pptfaq/FAQ00598_Animations_don-t_play_when_I_go_back_to_a_slide.htm | practitioner]; [https://bugs.documentfoundation.org/show_bug.cgi?id=34407 | independent]
- Claim: HTML decks animate SVG parts by inlining the SVG and putting classes/ids on parts (reveal.js practitioners call the CSS messy); dom-to-pptx previews its pptx animations in the browser.
  - Sources: [https://www-public.telecom-sudparis.eu/~berger_o/weblog/tag/svg/index.html | practitioner]; [https://sachachua.com/blog/2024/01/animating-svg-topic-maps-with-inkscape-emacs-ffmpeg-and-reveal-js/ | practitioner]; [https://people.willamette.edu/~jjrembold/class_files/data275/Slides/html_srcs/plugin/animate/README.md | search excerpt | practitioner]
- Contradictions: preset ids (Reconciled 5), animBg (6), Morph wrapper (7), Google import (8).
- Recommendation basis: keep the spec's model; add `by: "paragraph"` with optional level, a direction field mapped to `presetSubtype`, name each effect from a closed list mapped to the LibreOffice/Aspose id table, state play as a function of the step index (reveal/Slidev style) for back; transitions ms written as `p14:dur` with fallback `spd`; verify animation export in PowerPoint, not LibreOffice.

### OQ7: How do Figma, Google Slides and Pitch anchor comments, and how do Gamma and Pitch let a user point the AI at one element? (cluster comments-pointing)

- Claim: Figma REST: comment `client_meta` is canvas Vector/Region or `FrameOffset{node_id, node_offset}` (pixels from frame top-left) or FrameOffsetRegion; flat replies by `parent_id`; no REST resolve; pins to top-level frames only; cut/paste disconnects; Move to page carries comments; duplicating a file drops them; deletion behaviour undocumented. Dev Mode annotations live on the node and follow it.
  - Sources: [https://developers.figma.com/docs/rest-api/comments-endpoints | vendor]; [https://developers.figma.com/docs/rest-api/comments-types | vendor]; [https://developers.figma.com/docs/rest-api/comments-property-types | vendor]; [https://help.figma.com/hc/en-us/articles/360041547853 | vendor]; [Figma forum threads 17753 index14/index16 and 4632 page 8 | practitioner]; [https://www.figma.com/blog/dev-mode-ga/ | vendor]; [https://developers.figma.com/docs/plugins/api/Annotation | vendor]; [https://glama.ai/mcp/servers/junsimon/figma-comments-mcp | search summary | practitioner]
- Claim: Google Drive API anchors are opaque and not rendered by editors. The Slides API (developer preview 2026-08-31, MCP comments 2026-10-01) anchors comment threads to object ids (slide or element), optionally text ranges or table cells, via a separate `CommentAnchor` list; `insertComment`, `addCommentReply` with `RESOLVE`/`REOPEN`, assignee; `commentUpdateState` reports partial failure. Behaviour on object delete undocumented.
  - Sources: [https://developers.google.com/workspace/drive/api/guides/manage-comments | vendor]; [https://developers.google.com/workspace/drive/api/reference/rest/v3/comments | vendor]; [https://developers.google.com/workspace/slides/api/guides/comments | vendor]; [https://developers.google.com/workspace/slides/api/reference/rest/v1/CommentThread | vendor]; [https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages | vendor]; [https://workspaceupdates.googleblog.com/2026/09/programmatic-comment-and-suggestion.html | vendor]; [https://releases.sh/sources/src_7aLBPByE36fG9UEb8vh_n | practitioner]; [https://support.google.com/docs/answer/65129 | vendor]
- Claim: Pitch documents comments per slide (panel, @mentions, resolve keeps them, lost on workspace move), slide status and assignees, a Commenter role; no block-level anchor documented.
  - Sources: [https://help.pitch.com/en/articles/4318672-collaborate-with-comments | vendor]; [https://help.pitch.com/en/articles/5695595-review-presentations-on-mobile | vendor]; [https://help.pitch.com/en/articles/4318642-set-a-status-and-assignees-for-a-slide | vendor]; [https://help.pitch.com/en/articles/8102572-about-the-commenter-role | vendor]; [https://help.pitch.com/sitemap.xml | vendor]
- Claim: PowerPoint modern comments anchor to exactly one of slide, drawing element (moniker with id and creation GUID) or text range; position relative to the anchored object; status active/resolved/closed; comments follow moved and (since 2025) cut-and-pasted objects; text deletion falls back to the containing object (search snippet).
  - Sources: [https://learn.microsoft.com/en-us/openspecs/office_standards/ms-pptx/161bc2c9-98fc-46b7-852b-ba7ee77e2e54 | vendor]; [.../bd806e3a-... ST_CommentStatus | vendor]; [https://learn.microsoft.com/en-us/openspecs/office_standards/ms-odrawxml/145d263b-da8a-4cad-b076-1e51a9d149bf | vendor]; [.../496f12c8-... CT_ShapeMoniker | vendor]; [https://support.microsoft.com/en-us/powerpoint/modern-comments-in-powerpoint | vendor]; [https://office-watch.com/2025/powerpoints-comments-stick-like-glue/ | journalism]; [https://insider.microsoft365.com/blog/do-more-with-modern-comments-in-powerpoint | search snippet | vendor]
- Claim: tldraw `@tldraw/commenting` (tldraw license, key needed in production): anchor union shape (normalized 0-1 offset, precise flag), point, region, page; text range cut; on shape delete the thread becomes a point anchor where the pin sat, re-attaches on undo unless moved; threads excluded from snapshots and exports, synced in a separate lane, "can comment but not edit"; comment mutations not undoable; flat threads.
  - Sources: [https://tldraw.dev/blog/a-comment-is-not-a-shape | vendor]; [https://github.com/tldraw/tldraw/blob/main/packages/tlschema/src/records/TLComment.ts | vendor]; [https://github.com/tldraw/tldraw/blob/main/packages/commenting/src/canvas/anchor-lifecycle.ts | vendor]; [https://tldraw.dev/examples/comment-anchors | vendor]; [https://tldraw.dev/releases/v5.3.0 | vendor]; [https://github.com/tldraw/tldraw/blob/main/packages/commenting/LICENSE.md | vendor]
  - Notes: decks can copy the design, not the code (license).
- Claim: Lucid: deleting an object orphans its comment; restoring a revision brings both back (search summary).
  - Sources: [https://community.lucid.co/admin-questions-2/receiving-the-shape-was-deleted-error-within-comments-tasks-12546 | practitioner, inaccessible page]
- Claim: Pointing the AI: Gamma AI editing mode (click elements, shown in chat; sparkle on text; CMD+E); Pitch Agent Quick actions on a selected block; Copilot targets a text box, slide or deck (no source that a selected shape is passed); Canva @Canva in comments on a selected element or page with selection-specific prompts ("select one element at a time"); Figma selection scopes the agent, Make has a point-and-edit tool with staged Apply and annotations sent together as one prompt; a Figma Make bug where selection did not reach the prompt. A VR study: pointing gives precision, speech gives speed, ambiguous references hit all objects ("changed all the books").
  - Sources: [https://help.gamma.app/en/articles/8033284-how-to-edit-content-using-ai | vendor]; [https://help.pitch.com/en/articles/14981091-pitch-agent | vendor]; [https://help.pitch.com/en/articles/15197157-add-and-edit-images-with-ai | vendor]; [https://mc.merill.net/message/MC1243550 | vendor]; [https://web.archive.org/web/20251029103802/https://www.canva.com/help/ask-canva-comments/ | vendor]; [https://fastcompany.co.za/co-design/2025-11-02-canva-just-introduced-an-ai-fix-it-button/ | journalism]; [https://www.figma.com/solutions/prompt-to-edit/ | vendor]; [https://help.figma.com/hc/en-us/articles/42009840449175 | vendor]; [https://forum.figma.com/report-a-problem-6/selected-element-not-passed-to-figma-make-ai-prompt-51238 | practitioner]; [https://arxiv.org/html/2410.22177 | independent]
- Claim: Agents reading comments and selection: Figma MCP reads selection only on the desktop server (remote needs a link), lists ids, sizes, screenshot; annotations now via `get_design_context`; no comment tools. Google Docs/Sheets/Slides MCP read comments (preview). Pitch MCP has neither. GitHub MCP review threads (list, reply, resolve by `threadId`, outdated coordinates nullable); Vercel `comments` CLI lists unresolved by default, resolve with a closing reply, JSON for agents; React Grab copies element plus source location. tldraw lets agents answer in comment threads ("Agents can't point").
  - Sources: [https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/ | vendor]; [https://forum.figma.com/suggest-a-feature-11/mcp-support-for-annotations-53685 | practitioner]; [https://github.com/github/github-mcp-server/blob/main/README.md | vendor]; [https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/review-copilot-prs | vendor]; [https://vercel.com/docs/cli/comments | vendor]; [https://vercel.com/changelog/manage-vercel-toolbar-comments-from-the-cli | vendor]; [https://vercel.com/docs/comments/using-comments | vendor]; [https://github.com/aidenybai/react-grab | vendor]; [https://tldraw.dev/blog/agents-cant-point | vendor]; [https://help.pitch.com/en/articles/16221995-what-s-possible-with-pitch-and-claude | vendor]; [https://glama.ai/mcp/servers/UtakataKyosui/PR-Review-Resolve-MCP/tools/reply_and_resolve | practitioner]
- Claim: Drawings kept out of the document: the plugin editor (pen drawings live in the editor only, reach the agent as red marks in a screenshot); Figma Region rectangles; Vercel drag-to-screenshot; tldraw region anchors. No product ties freehand pen strokes to a comment thread.
  - Sources: [editing-presentations SKILL.md | practitioner]; [Figma comments property types | vendor]; [Vercel using-comments | vendor]; [tldraw comment-anchors example | vendor]
- Claim: The plugin editor's known weakness: line numbers go stale after the first edit; the spec's switch to ids targets exactly this.
  - Sources: [editing-presentations SKILL.md | practitioner]
- Contradictions: Figma follow (Reconciled 13); tldraw text range (14).
- Recommendation basis: comments in their own table, never in the deck; anchor = slide id plus optional block ids with normalized pin offset, optional step; on block delete keep the thread with a slide-point anchor at the last pin (tldraw); drawings as vector strokes owned by the comment, rendered only in editor and screenshots; MCP list (open by default) / reply / resolve / reopen, plus `get_selection` returning ids, types, props and a screenshot with boxes.

### OQ8: What would live collaboration need from the model now? (cluster collab-crdt, plus local-prior-art SQ2)

- Claim: Licenses: Yjs, y-prosemirror, y-websocket, y-utility, Yrs, pycrdt, pycrdt-websocket, Hocuspocus, y-sweet, Automerge, automerge-repo, automerge-py, Loro all MIT (several flagged NOASSERTION by GitHub only because of headers); rocicorp fractional-indexing CC0; tldraw sync under the tldraw license; @tldraw/store MIT.
  - Sources: [LICENSE files listed in collab-crdt SQ1 table, e.g. https://github.com/yjs/yjs/blob/main/LICENSE, https://github.com/automerge/automerge/blob/main/LICENSE, https://github.com/loro-dev/loro/blob/main/LICENSE, https://github.com/rocicorp/fractional-indexing/blob/main/LICENSE | vendor]
- Claim: Yjs maps resolve last-insert-wins; Y.Map keeps history per key and grows under alternating writes (100k ops on 10 keys: 524,985 B Y.Map vs 271 B YKeyValue); the author says Y.Map suits bounded domain objects, not collections; field report of 30 MB/day growth moved to LWW outside Yjs. Yjs 14 still release candidate.
  - Sources: [https://github.com/yjs/yjs/blob/main/INTERNALS.md | vendor]; [https://cdn.jsdelivr.net/npm/y-utility@0.1.4/README.md | vendor]; [https://liveblocks.io/docs/guides/yjs-best-practices-and-tips | vendor]; [https://discuss.yjs.dev/t/map-metadata-overhead/492 | practitioner]; [https://discuss.yjs.dev/t/document-growing-unbounded/3729 | practitioner]; [https://newreleases.io/project/yarn/yjs/release/14.0.0-0 | vendor]; [https://fosdem.org/schedule/event/8VKQXR-blocknote-yjs-prosemirror | practitioner]
- Claim: Automerge keeps full history, losing values readable via `getConflicts`, no list move; suits small groups; schema migrations hard (no shared ancestry; Cambria not implemented); Automerge 3 cut memory over 10x. Loro: LWW map, MovableList (slower), movable tree with fractional index, fork/branch APIs still evolving.
  - Sources: [https://github.com/automerge/automerge.github.io/blob/main/content/docs/reference/under-the-hood/merge-rules.md | vendor]; [https://automerge.org/docs/reference/documents/conflicts/ | vendor]; [https://github.com/automerge/automerge.github.io/blob/main/content/docs/cookbook/modeling-data.md | vendor]; [https://automerge.org/blog/automerge-3/ | vendor]; [Loro docs list.mdx, map.mdx, tree.mdx, v1.0.mdx, time_travel.mdx | vendor]
- Claim: Benchmarks (dmonad/crdt-benchmarks) use old Automerge 2.1 and Loro 0.10 and are run by the Yjs author; no deck-shaped benchmark exists.
  - Sources: [https://github.com/dmonad/crdt-benchmarks | practitioner, maintained by Yjs author, pre-Automerge 3 / pre-Loro 1.0]
- Claim: Model choices that ease later sync: client-generated unique ids (Figma "can't" have the server assign them); objects as `Map<id, Map<prop, value>>` with per-property LWW (Figma); network diffs keyed by record id (tldraw); order as fractional-index strings per object, with random suffix against interleaving and server dedup (Figma, tldraw, Excalidraw retrofit 2024 still patched in 2026); parent link and position stored atomically; move as LWW position register (Kleppmann); rich text as its own sequence type, since string LWW loses concurrent edits (tldraw append-or-put); presence outside the document; tombstones; schema migrations by version.
  - Sources: [https://www.figma.com/blog/how-figmas-multiplayer-technology-works/ | vendor]; [https://www.figma.com/blog/realtime-editing-of-ordered-sequences/ | vendor]; [https://madebyevan.com/algos/crdt-fractional-indexing/ | practitioner]; [https://github.com/tldraw/tldraw/blob/main/packages/sync-core/SPEC.md | vendor]; [https://tldraw.dev/sdk-features/shape-indexing | search snippet | vendor]; [https://github.com/excalidraw/excalidraw/pull/7359 | vendor]; [https://github.com/excalidraw/excalidraw/pulls?q=fractional | vendor]; [https://plus.excalidraw.com/blog/building-excalidraw-p2p-collaboration-feature | vendor]; [https://martin.kleppmann.com/papers/list-move-papoc20.pdf | independent]; [https://web.archive.org/web/2024/https://jamsocket.com/blog/you-might-not-need-a-crdt | vendor]; [https://drive.googleblog.com/2010/09/whats-different-about-new-google-docs_22.html | vendor]; [https://tldraw.dev/docs/sync | vendor]; [https://www.inkandswitch.com/cambria/ | independent]; [https://arxiv.org/pdf/2309.11406 | search snippet | independent]
- Claim: Invariants and cascades are poor CRDT fits ("CRDTs merge; they do not reject"); a server can check them. The spec's "deleting a block deletes its animations" is such a cascade.
  - Sources: [https://github.com/loro-dev/loro-docs/blob/main/pages/docs/concepts/when_not_crdt.mdx | vendor]; [Jamsocket | vendor]; [spec-v0.md | practitioner]
- Claim: Server-ordered sync without a CRDT is the norm: Figma (one process per file, journal, 33 ms updates), tldraw sync ("deliberately not a CRDT", commit/discard/rebaseWithDiff, document clock, tombstones, SQLite storage, one instance per room), Linear (server transactions), Google Docs (OT revision log); Google Slides API `requiredRevisionId` refuses stale writes. Limits: single authority per document; text needs OT or a CRDT.
  - Sources: [Figma multiplayer blogs | vendor]; [https://www.figma.com/blog/making-multiplayer-more-reliable/ | vendor]; [https://tldraw.dev/blog/20-things-i-wish-ai-chatbots-knew-about-tldraw | vendor]; [https://tldraw.dev/blog/announcing-tldraw-sync | vendor]; [https://tldraw.dev/docs/sync | vendor]; [tldraw sync-core SPEC.md | vendor]; [https://github.com/wzhudev/reverse-linear-sync-engine | practitioner, unaffiliated]; [Google Docs 2010 blog | vendor]; [https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/batchUpdate | vendor]; [https://www.git-tower.com/blog/developing-for-the-desktop-pitch/ | vendor]; [https://news.ycombinator.com/item?id=24836676 | practitioner]
- Claim: Blattomat: no op log; whole-document PATCH with a version, SQL compare-and-set, 409; editor saves 2 s after change; clash UI (overwrite, keep as new, discard); no server push (focus and cross-tab events only); agent tools take the version, refuse on mismatch; `replace_doc` retries once when only the version moved.
  - Sources: [/home/pascal/Code/blattwerk/src/blattwerk/sheets.py | practitioner]; [/home/pascal/Code/blattwerk/frontend/src/saves.ts | practitioner]; [/home/pascal/Code/blattwerk/docs/okf/versions.md | practitioner]; [/home/pascal/Code/blattwerk/frontend/mcp/tools.ts | practitioner]; [/home/pascal/Code/blattwerk/frontend/src/pages/Editor.tsx:484-489 | practitioner]
- Claim: Agent edits: JSON Patch addresses arrays by index (fragile); id-keyed diffs and object ids (tldraw, Google) are robust; Automerge `changeAt`, `view`, `clone`, `merge`, `diff`; Loro fork; Patchwork bot branches with partial merge and attribution, but domain diff views and comments need custom work; Hocuspocus `openDirectConnection` lets a server agent write into a live Yjs doc; Tiptap server AI toolkit.
  - Sources: [https://www.rfc-editor.org/rfc/rfc6902 | independent]; [https://github.com/automerge/automerge/blob/main/javascript/src/implementation.ts | vendor]; [https://www.inkandswitch.com/patchwork/notebook/2024-version-control/ | independent]; [https://github.com/ueberdosis/hocuspocus/blob/main/packages/server/src/DirectConnection.ts | vendor]; [https://tiptap.dev/docs/content-ai/capabilities/server-ai-toolkit/overview | search snippet | vendor]
- Claim: Convergence is not correctness (Jamsocket, Loro).
  - Sources: [Jamsocket | vendor]; [Loro when_not_crdt | vendor]
- Recommendation basis: no CRDT in v0. Keep ids client-made; replace array order and integer `z` with fractional-index strings (`order` on slides, blocks, animations); text as ProseMirror-compatible runs; ops by id with per-property writes; server applies ops in order, checks invariants and cascades, bumps a deck version; comments and presence beside the deck; a schema version field for migrations. That leaves both tldraw-sync-style server rebase and a later Yjs/Automerge mapping open.

### Cross-cutting: Blattomat and the plugin as port sources (local-prior-art)

- Claim: Blattomat already has what spec section 8 asks: remote MCP with OAuth 2.1 (PKCE, refresh, dynamic registration), a Node MCP child process using the REST API, 16 tools, warnings on each write (overlap, printable area), `render_preview` PNG with magenta id boxes, Playwright PDF with a 60 s HMAC token, OKF docs served as MCP instructions and resources, generated block pages, `okf_check.py`, an eval harness. Pages are addressed by number, not id.
  - Sources: [/home/pascal/Code/blattwerk/src/blattwerk/remote.py | practitioner]; [/home/pascal/Code/blattwerk/src/blattwerk/oauth.py | practitioner]; [frontend/mcp/tools.ts | practitioner]; [src/blattwerk/pdf.py | practitioner]; [docs/okf/index.md; frontend/mcp/docs.ts | practitioner]; [scripts/okf_check.py | practitioner]; [scripts/eval.py | practitioner]; [docs/okf/page-model.md | practitioner]
- Claim: Blattomat undo is whole-document snapshots in Canvas state; a server-loaded sheet remounts Canvas and empties history (reading of code).
  - Sources: [/home/pascal/Code/blattwerk/frontend/src/history.ts | practitioner]; [Editor.tsx:388, 402, 414 | practitioner]
- Claim: Hard gates H1 (text past slide edge, >2 px), H2 (text rects overlap >4 px), H7 (floors body 36 px, caption 28, footer 24, by role); floors are "starting points ... not sourced numbers"; probes run in Chromium; the deck editor reruns them after each edit and lists only new failures.
  - Sources: [creating-presentations/references/hard-gates.md | practitioner]; [creating-presentations/SKILL.md:43 | practitioner]; [editing-presentations/assets/overlay-gates.js | practitioner]
- Claim: Real deck corpus (aggregates): 18 deck folders, 43 deck HTML files, 9 to 70 slides in the larger decks; inline SVG in 38 of 43 files; tables in 13; no iframe, video or canvas anywhere; speaker notes in 35; flex and grid in every deck; decks depend on shared assets by relative path; 10 pptx files, 1 to 70 slides, 0 `<p:timing>`, 1 `<p:transition>`, 1 `<p:ph`, 0 chart parts; 0 builds, 0 `data-chart`.
  - Sources: [private deck repo, aggregate count | local inspection | practitioner]
  - Notes: implies Text, Image, Shape, Table, SVG and Placeholder blocks plus speaker notes cover nearly all of today's content; Script and Chart blocks serve future decks; the wrap import must carry shared assets.

## Reconciled Contradictions

1. tldraw hobby license
   - Side A: vendor pages and Ruiz's blog describe a free hobby license (watermark) [https://tldraw.dev/community/license | vendor]; [https://tldraw.dev/blog/20-things-i-wish-ai-chatbots-knew-about-tldraw | vendor]
   - Side B: LICENSE.md names only trial and commercial agreements for production [https://raw.githubusercontent.com/tldraw/tldraw/main/LICENSE.md | vendor]
   - Adjudication: both true; the hobby license is a discretionary separate grant, non-commercial only. For decks, used for Pascal's Deloitte and client talks, production use needs a commercial key. The license text governs.
2. moveable last activity
   - Side A: last release and master commit 2023-12-03 [canvas-rendering, `gh api`, `npm view` | vendor]
   - Side B: repo last pushed 2024-06-03 [local-prior-art, GitHub API pushed_at | vendor]
   - Adjudication: compatible (push timestamp covers non-release branch pushes). Report as "no release since 2023-12, no maintainer answer to #1137 or #1156".
3. DOM performance
   - Side A: tldraw culling, 10,000 shapes render ~50 [https://tldraw.dev/llms-full.txt | vendor]
   - Side B: DOM lag at 50+ effect-heavy elements [GeekyAnts | practitioner]
   - Adjudication: different measures (off-screen culling vs on-screen effects), no method published either side. Low relevance: slides hold few blocks [author estimate]. Unresolved but immaterial.
4. PptxGenJS layouts
   - Side A: "The defined Masters become first-class Layouts" [https://gitbrent.github.io/PptxGenJS/docs/masters/ | vendor]
   - Side B: layouts exist but lack `type`; slide placeholders write their own geometry and size; issues #1453, #1208, #1449 [scratchpad XML | practitioner]; [GitHub issues | practitioner]; [gen-xml.ts source | vendor]
   - Adjudication: Side B. They are real `sldLayout` parts, so Change Layout appears, but slides do not inherit, and several masters trigger repair. Not "real layouts" in the spec's sense without post-processing.
5. Animation preset ids
   - Side A: LibreOffice import table, Aspose FOSS mapping, ChristopherVR COM/UI-observed ids (pulse = emph 26), PowerPoint-saved Fly In (entr 2, subtype 4) [LibreOffice commontimenodecontext.cxx | independent]; [Aspose FOSS animation_mappings.py | vendor]; [ChristopherVR ooxml | vendor]; [file_with_bounce.pptx | practitioner]
   - Side B: dom-to-pptx calls entr 23 "Grow & Turn"; svg2ooxml uses emph 32 for grow/shrink; Resource2Skill lists emph 2 spin, 8 pulse, 20 teeter, 31 grow_shrink [dom-to-pptx | vendor]; [svg2ooxml | practitioner]; [Resource2Skill | vendor]
   - Adjudication: Side A. Two independent importers agree, a COM-observed source and a PowerPoint-saved file agree with them; the Side B writers each disagree in a different direction. Even Microsoft's own sample MCP server is wrong here, so ids must be tested in PowerPoint.
6. `animBg` in `bldP`
   - Side A: dom-to-pptx "bldLst entries MUST include animBg="1"" [vendor]
   - Side B: PowerPoint-saved sample has `<p:bldP spid="4" grpId="0"/>` [practitioner]
   - Adjudication: Side B (primary output). `animBg` is optional; it is a convention of that writer.
7. Morph wrapper
   - Side A: OfficeCLI wraps `p159:morph` in `mc:AlternateContent` with a fade fallback [vendor]
   - Side B: Resource2Skill writes it unwrapped [vendor]
   - Adjudication: unresolved ([MS-PPTX] inaccessible). Leaning to Side A because PowerPoint itself wraps p14 features in AlternateContent (sample). Morph is not in the spec's v0 list, so it does not block v0.
8. pptx to Google Slides animations
   - Side A: unknown presets removed, their click lost [https://www.brightcarbon.com/blog/convert-powerpoint-google-slides/ | practitioner]
   - Side B: complex chains and paths "flatten to a single appear effect" [https://slidemodel.com/fix-compatibility-powerpoint-google-slides/ | practitioner, search excerpt]
   - Adjudication: unresolved; both agree on loss. Google Slides is not a target for decks' animations.
9. Slidev pptx
   - Side A: "PPTX is not the core use case" [https://www.pkgpulse.com/guides/slidev-vs-marp-vs-revealjs-code-first-presentations-2026 | practitioner]
   - Side B: image pptx since 2024-05, editable since 2026-09-16 [Slidev PRs 1603, 2722 | vendor]
   - Adjudication: Side B for facts; Side A's judgment held before 2026-09. Minor: `--with-clicks` default disputed by open PR 2758 [vendor], unresolved, immaterial.
10. LLMs on Typst
    - Side A: "pretty good" (HN), Claude needs "minor adjustments" (forum) [practitioner]
    - Side B: "terrible" (HN); TeXFix-Bench compile 60.3% vs Markdown 90.2% [practitioner]; [https://arxiv.org/html/2608.07617 | independent]
    - Adjudication: lean Side B on the one sizable independent study, with its caveat that it measures repair, not authoring; no Touying-authoring benchmark exists.
11. Gamma keeps the rest intact
    - Side A: targeted AI edits keep "the rest of your gamma intact" [Gamma help | vendor]
    - Side B: one AI card creation undid manual edits on other cards [https://ideas.gamma.app/ideas/p/gamma-ai-clobbered-my-edits | practitioner]
    - Adjudication: unresolved (one anecdote; Gamma staff investigated, no stated fix). The decision-relevant finding stands either way: no product documents a hard guarantee an agent honours; Copilot's note steering is "guidance", Canva's AI-and-locks page was unreadable.
12. Copilot licensing
    - Side A: available to Copilot Chat users without a Microsoft 365 Copilot license [RM548646 mirror | vendor]
    - Side B: affects holders of Microsoft 365 Copilot licenses [MC1219792 mirror | vendor]
    - Adjudication: unresolved; irrelevant to decks.
13. Figma comments follow frames
    - Side A: support "My comments are moving with the frames" (2023) [forum | practitioner]
    - Side B: users since 2021: pinned comments do not move with nested elements; support 2024: pins only on top-level frames [forum | practitioner]
    - Adjudication: consistent once read as "follow top-level frames only", matching `FrameOffset.node_id` + pixel offset in the API [vendor].
14. tldraw text-range anchor
    - Side A: v5.3.0 release notes list a text-range anchor [https://tldraw.dev/releases/v5.3.0 | vendor]
    - Side B: blog says it was cut; source union has no such variant [blog | vendor]; [TLComment.ts | vendor]
    - Adjudication: Side B (source code).
15. Format restriction and reasoning
    - Side A: "Let Me Speak Freely?" format limits hurt reasoning [independent]
    - Side B: .txt rerun finds no loss [vendor]
    - Adjudication: unresolved on reasoning. Decision rests on more specific evidence: JSON Whisperer (index arithmetic fails, stable keys fix) [independent], Aider (code inside JSON slightly worse) [practitioner], ACE (id-addressed semantic tools beat HTML) [independent, preprint].
16. Gamma API limits (750k chars vs 100k tokens; numCards 60/75/100) and Tome user counts (20M vs 25M): internal inconsistencies in vendor and press numbers; immaterial.
17. d3 licence in ECharts' LICENSE (BSD-3) vs d3 repo (ISC): historical relicensing; both permissive; immaterial, but keep ECharts' NOTICE.

## Gaps and Thin Evidence

1. gap-change-layout: no test of PowerPoint's Change Layout or Reset on a pptx generated by python-pptx or PptxGenJS; no PowerPoint or render check was run in the scan at all (XML reads only). The v0 test ("opens in PowerPoint with editable text") needs one.
2. gap-build-p-sample: no PowerPoint-saved `bldP build="p"` sample; per-paragraph build XML rests on ECMA examples and dom-to-pptx.
3. gap-ms-pptx: [MS-PPTX] (Morph, `p14:dur`) and MS-ODRAWXML (`svgBlip`) unread; evidence from code that writes them.
4. gap-back-step: no Microsoft source on the back arrow within a slide's steps.
5. gap-pitch: Pitch editor internals (only the player inspected), sync architecture, AI treatment of hand-placed blocks, previews, block-level comment anchors, pptx layout structure: all absent.
6. gap-renderers: PowerPoint for the web and Canva editor rendering undetermined.
7. gap-keynote-export: no source on Keynote builds or Magic Move mapping in pptx export; no source on Keynote layout mapping to `sldLayout`.
8. gap-google-export: no tested source on Google Slides' pptx download keeping layouts or animations.
9. gap-delete-anchor: behaviour on deleting a commented object undocumented for Figma, Google Slides, PowerPoint; Lucid only by search summary.
10. gap-copilot: no Microsoft developer source on how Copilot edits slides or whether a selected shape reaches the model.
11. gap-canva: Canva AI and locks page unreadable (403); Canva claims unverified.
12. gap-crdt-bench: no benchmark of a deck-shaped document (hundreds of small objects, frequent property writes); the one benchmark uses old Automerge and Loro and is run by the Yjs author.
13. gap-unverified-libs: power-pptx and Aspose FOSS feature claims (animations, themes, SVG) not checked against code; pptx-automizer docs not read in depth.
14. gap-yjs14: Yjs 14 not final (rc.29).
15. gap-licenses-unread: LibreOffice license not read (runs as a separate program); Spire and commercial Aspose licenses not read; Syncfusion and Gamma table export only via snippets.
16. gap-text-style-order: full PresentationML text-style lookup order not found.
17. gap-token-estimate: the JSON-vs-text token figure is one slide, OpenAI tokenizer, invented sketch format [author estimate].
18. gap-typst-authoring: no benchmark of LLMs writing Touying or Slidev decks.
19. gap-search-summaries: several claims rest on search summaries only (Syntax 505, W3C 2010 thread, Bushell, Canva interview guide, Lucid, PowerPoint text-anchor deletion, Gamma table export, Templafy, tldraw shape indexing); each is tagged inline.
20. gap-private-figures: further per-deck numbers exist in `local-prior-art.md` but are withheld under the privacy rule; only repo-wide totals and ranges are used.
21. gap-lock-impl: Blattomat's lock refusal not traced into code; `replace_doc` documents no lock check.
22. gap-agentroom: two papers on LLM agents over CRDTs (AgentRoom, CodeCRDT) unread (arXiv returned no record); not used.

## Outliers / Counter-Evidence

- Spec says "A master is a layout". OOXML and Google Slides separate master (text styles by level, colour map, fixed shapes, background) from layout (placeholder arrangement, `type`, `idx`); slide-to-layout matching runs on `idx`, layout-to-master on `type`. A bare `slot` string per block does not give the exporter `ph@type`/`ph@idx`, nor a layout `type` for matching. [masters-layouts SQ1, SQ2 sources above]
- Spec stores order as array places and stacking as integer `z`, and the animation timeline as an ordered array. Figma, tldraw and Excalidraw (retrofitted in 2024, still fixing it in 2026) store order as fractional-index strings per object. The spec's "every operation names them by id" is necessary but not enough: the order itself is still positional. [collab-crdt SQ2 sources]
- Spec says agent rebuild of a wrapped slide can be undone ("Undo brings the wrapped slide back"); Blattomat, the port source, remounts the canvas on a server-loaded document and drops the undo stack, so agent writes never enter the user's undo. A real design gap in the code being ported. [Editor.tsx:388, 402, 414 | practitioner]
- idea.md says the agent changes a deck "only through the same operations the editor uses"; Blattomat's agent tools are a separate module that sends the whole document back, not the editor's operations. [frontend/mcp/tools.ts | practitioner]
- Real decks hold 0 `<p:timing>`, 0 build steps, 0 `data-chart`, 0 chart parts, 1 placeholder in total; the spec puts animations, native charts and real layouts in v0. This weighs on priority and test scope, not on model design (the spec's reason is "so no deck has to be rebuilt to gain them"). [private deck repo, aggregate count | local inspection | practitioner]
- "Port of the exporter" assumes code to port; the plugin ships recipes and per-deck scripts, no reusable exporter. The port is a new program. [local-prior-art SQ4 | practitioner]
- Neither PptxGenJS nor python-pptx writes animations, transitions, theme colours or SVG (PptxGenJS SVG becomes a broken image in Node); the spec's "native PowerPoint ones" means decks writes that OOXML itself whichever library it picks. [export-chart-libs, animations sources]
- LibreOffice, the plugin's verification renderer, has 27 open pptx animation bugs and no Morph, and disagrees with PowerPoint on placeholder inheritance; it cannot verify animation or layout export. [LibreOffice bugs 139897, 104202 | independent]
- License flags on the port path: PyMuPDF is AGPL-3.0 (commercial license from Artifex) and CairoSVG LGPL-3.0, both in the current exporter; svg2ooxml is AGPL/commercial; the MCP TypeScript SDK and mcpb are in an MIT-to-Apache-2.0 transition; ECharts requires shipping its NOTICE; Blattomat has no LICENSE file (issue #889), which matters when porting its code into the public decks repo; Anthropic's pptx skill is proprietary and must not be copied. [local-prior-art license table; export-chart-libs license table; animations license table | vendor]
- Text-form counter-evidence: Brosse found a text form (Quarto) the practical choice for one LLM and Aider found JSON-wrapped code slightly worse; these favour text for generation, against the JSON-ops thesis. They are single-author or code-centric; ACE, PPTArena and JSON Whisperer address editing structured decks and point the other way. Keep a read-only text projection to capture the reading benefit.
- Comments: tldraw, the best-documented anchor design, is under the tldraw license; decks can copy the design but not the package.
- Pitch, the product the spec names as the DOM model, documents almost nothing about AI treatment of hand edits or comment anchoring; much of OQ4 and OQ7 rests on Gamma, Google, Microsoft, Figma and tldraw instead.

## Consolidated License Table (LICENSE files read directly; for the report's license table)

| Component | Role for decks | License as read | Flag | Source |
|---|---|---|---|---|
| tldraw, @tldraw/editor, sync, sync-core, commenting, collaboration, mentions | rejected canvas option | tldraw license: dev only; production needs trial or commercial key; usage telemetry | proprietary | https://raw.githubusercontent.com/tldraw/tldraw/main/LICENSE.md, packages/*/LICENSE.md [vendor] |
| @tldraw/store, state, tlschema, validate, utils | reference only | MIT (2024 tldraw Inc.) | none | packages/*/LICENSE.md [vendor] |
| moveable / react-moveable | canvas (port) | MIT (2019 Daybrush) | unmaintained since 2023-12 | https://raw.githubusercontent.com/daybrush/moveable/master/LICENSE [vendor] |
| selecto / react-selecto | canvas (port) | MIT (2020 Daybrush) | unmaintained since 2023-12 | https://raw.githubusercontent.com/daybrush/selecto/master/LICENSE [vendor] |
| interact.js | alternative | MIT (Taye Adeyemi) | none | https://raw.githubusercontent.com/taye/interact.js/main/LICENSE [vendor] |
| Konva / react-konva | alternative (canvas) | MIT | none | https://raw.githubusercontent.com/konvajs/konva/master/LICENSE [vendor] |
| Fabric.js | alternative (canvas) | MIT | none | https://raw.githubusercontent.com/fabricjs/fabric.js/master/LICENSE [vendor] |
| Excalidraw | reference | MIT (2020) | none | https://raw.githubusercontent.com/excalidraw/excalidraw/master/LICENSE [vendor] |
| ProseMirror (model, state, view, commands, keymap) | text field (port) | MIT (Marijn Haverbeke) | none | https://raw.githubusercontent.com/ProseMirror/prosemirror-view/master/LICENSE [vendor] |
| React, react-router | frontend | MIT | none | local-prior-art table [vendor] |
| lucide | icons | ISC, Feather icons MIT | none | https://raw.githubusercontent.com/lucide-icons/lucide/main/LICENSE [vendor] |
| modern-screenshot | DOM to image | MIT | none | local-prior-art table [vendor] |
| MCP TypeScript SDK, mcpb | MCP server | MIT to Apache-2.0 transition | note both | https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/LICENSE [vendor] |
| MCP Python SDK | MCP | MIT (2024 Anthropic) | none | local-prior-art table [vendor] |
| FastAPI | backend | MIT | none | local-prior-art table [vendor] |
| uvicorn, httpx, pypdf, Python-Markdown | backend | BSD-3-style | no-endorsement | local-prior-art table [vendor] |
| playwright-python | PDF, render | Apache-2.0 | NOTICE duties | local-prior-art table [vendor] |
| argon2-cffi, jsonschema | backend | MIT | none | local-prior-art table [vendor] |
| python-pptx | pptx export | MIT (2013 Steve Canny) | stale since 2024-08 | https://github.com/scanny/python-pptx/blob/master/LICENSE [vendor] |
| power-pptx | candidate fork | MIT | young, unverified | https://github.com/codehalwell/power-pptx/blob/master/LICENSE [practitioner] |
| PptxGenJS | alternative | MIT (Brent Ely) | none | https://github.com/gitbrent/PptxGenJS/blob/master/LICENSE [vendor] |
| pptx-automizer | alternative | MIT | none | https://github.com/singerla/pptx-automizer/blob/main/LICENSE [vendor] |
| officegen | rejected | MIT | dead since 2021 | https://github.com/Ziv-Barber/officegen/blob/master/LICENSE [vendor] |
| pptx-embed-fonts | fonts | MIT | tiny project | https://github.com/liyao1520/pptx-embed-fonts/blob/main/LICENSE [practitioner] |
| dom-to-pptx | reference (timing XML) | MIT | none | https://github.com/atharva9167j/dom-to-pptx/blob/master/LICENSE [vendor] |
| Aspose.Slides FOSS for Python | reference (preset ids) | MIT (2026 Aspose) | none | https://github.com/aspose-slides-foss/Aspose.Slides-FOSS-for-Python/blob/main/LICENSE [vendor] |
| OfficeCLI, ChristopherVR/ooxml | reference | Apache-2.0 | NOTICE if copied | animations license table [vendor] |
| svg2ooxml | avoid | AGPL-3.0 or commercial | copyleft | https://github.com/BramAlkema/svg2ooxml/blob/main/LICENSE [vendor] |
| Resource2Skill | reference only | MIT (Microsoft) | ids wrong | https://github.com/microsoft/Resource2Skill/blob/main/LICENSE [vendor] |
| Open XML SDK | not in stack | MIT | .NET | https://github.com/dotnet/Open-XML-SDK/blob/main/LICENSE [vendor] |
| Aspose.Slides | paid comparison | commercial, from $1,399 | paid | https://purchase.aspose.com/pricing/slides/family/ [vendor] |
| Syncfusion Presentation | paid comparison | commercial; Community License limits | paid | https://www.syncfusion.com/products/communitylicense [vendor] |
| Anthropic pptx skill | do not copy | proprietary, all rights reserved | proprietary | https://github.com/anthropics/skills/blob/main/skills/pptx/LICENSE.txt [vendor] |
| PyMuPDF | current exporter rasterizer | AGPL-3.0 or commercial (Artifex) | copyleft | https://raw.githubusercontent.com/pymupdf/PyMuPDF/main/COPYING [vendor] |
| CairoSVG | current exporter fallback | LGPL-3.0 | copyleft (weak) | https://raw.githubusercontent.com/Kozea/CairoSVG/main/LICENSE [vendor] |
| Pillow | exporter | MIT-CMU | none | https://raw.githubusercontent.com/python-pillow/Pillow/main/LICENSE [vendor] |
| Apache ECharts | chart block | Apache-2.0 + NOTICE; d3 parts BSD-3 | ship NOTICE | https://github.com/apache/echarts/blob/master/LICENSE, NOTICE [vendor] |
| zrender | ECharts dependency | BSD-3 (Baidu) | no-endorsement | https://github.com/ecomfe/zrender/blob/master/LICENSE [vendor] |
| Vega-Lite, Vega, vl-convert | chart alternative | BSD-3 | no-endorsement | vega repos LICENSE [vendor] |
| Chart.js | rejected (canvas) | MIT | none | https://github.com/chartjs/Chart.js/blob/master/LICENSE.md [vendor] |
| Observable Plot, D3 | rejected / low level | ISC | none | observablehq/plot, d3/d3 LICENSE [vendor] |
| Recharts | rejected (no SSR) | MIT | none | https://github.com/recharts/recharts/blob/main/LICENSE [vendor] |
| Slidev, slidev-addon-studio, @vueuse/motion | reference | MIT | none | slidevjs/slidev, BobTheShoplifter/slidev-addon-studio, vueuse/motion LICENSE [vendor] |
| Marp CLI, Marp Core, Marpit, marp-mcp | reference | MIT | none | marp-team repos, masaki39/marp-mcp LICENSE [vendor] |
| reveal.js | reference | MIT | none | https://github.com/hakimel/reveal.js/blob/master/LICENSE [vendor] |
| Quarto CLI | reference | MIT (COPYING.md) | GitHub says NOASSERTION | https://github.com/quarto-dev/quarto-cli/blob/main/COPYING.md [vendor] |
| Spectacle | reference | MIT | none | https://github.com/FormidableLabs/spectacle/blob/main/LICENSE [vendor] |
| Typst | rejected text form | Apache-2.0 | none | https://github.com/typst/typst/blob/main/LICENSE [vendor] |
| Touying, touying-exporter, typ2pptx, SeaSlides | rejected text form | MIT | none | touying-typ repos LICENSE [vendor] |
| Tylina | reference | proprietary, all rights reserved | proprietary | https://github.com/tylina/tylina.github.io/blob/main/LICENSE [vendor] |
| Yjs, y-prosemirror, y-websocket, y-utility, Yrs, pycrdt, pycrdt-websocket | later collaboration | MIT | none | collab-crdt license table [vendor] |
| Hocuspocus, y-sweet | later collaboration | MIT | none | collab-crdt license table [vendor] |
| Automerge, automerge-repo, automerge-py | later collaboration | MIT | none | collab-crdt license table [vendor] |
| Loro | later collaboration | MIT | none | https://github.com/loro-dev/loro/blob/main/LICENSE [vendor] |
| fractional-indexing (rocicorp) | order keys now | CC0 1.0 | none | https://github.com/rocicorp/fractional-indexing/blob/main/LICENSE [vendor] |
| react-grab | reference | MIT | none | https://github.com/aidenybai/react-grab/blob/main/LICENSE [vendor] |
| presentations plugin (pgoell-claude-tools) | port source | MIT (2026 Pascal Göllner) | none | /home/pascal/Code/pgoell-claude-tools/LICENSE [practitioner] |
| Blattomat | port source | no LICENSE file (issue #889) | settle before porting | https://github.com/pgoell/blattwerk/issues/889 [practitioner] |

## Source Inventory

| Source | Title | Tag | Used in claims |
|--------|-------|-----|----------------|
| https://raw.githubusercontent.com/tldraw/tldraw/main/LICENSE.md | tldraw license | vendor | OQ1.1, OQ7, License table, Reconciled 1 |
| tldraw packages/*/LICENSE.md, package.json | tldraw package licenses | vendor | OQ1.1, OQ8, License table |
| https://tldraw.dev/blog/tldraw-sdk-4-0 ; tldraw-sdk-5-0 | SDK 4.0 / 5.0 announcements | vendor | OQ1.1, OQ1.2 |
| https://tldraw.dev/community/license ; https://tldraw.dev/pricing (+ archives 2025-10-10, 2026-08-28) | license page, pricing | vendor | OQ1.1, Reconciled 1 |
| https://tldraw.dev/blog/20-things-i-wish-ai-chatbots-knew-about-tldraw | 20 things (Ruiz) | vendor | OQ1.1, OQ1.2, OQ8 |
| https://tldraw.dev/llms-full.txt | tldraw docs full text | vendor | OQ1.2, OQ1.4, Reconciled 3 |
| https://news.ycombinator.com/item?id=45293839 | Tldraw SDK 4.0 HN thread | practitioner | OQ1.1 |
| tldraw examples slides/slideshow; blog building-with-tldraw-offline | slide examples | vendor | OQ1.2 |
| https://tldraw.dev/docs/ai ; starter-kits/agent ; templates/agent/shared/format | agent kit | vendor | OQ1.2, OQ3.3, OQ4 |
| https://tldraw.dev/blog/a-comment-is-not-a-shape ; TLComment.ts ; anchor-lifecycle.ts ; releases/v5.3.0 ; examples/comment-anchors | tldraw commenting | vendor | OQ7, Reconciled 14 |
| https://tldraw.dev/blog/agents-cant-point | Agents can't point | vendor | OQ7 |
| tldraw sync-core SPEC.md ; tldraw.dev/docs/sync ; announcing-tldraw-sync | tldraw sync | vendor | OQ8 |
| first-hand DOM inspection (Pitch player, Google Slides, PowerPoint web) | own inspection 2026-10-20 | practitioner | OQ1.3 |
| https://newsletter.pragmaticengineer.com/p/building-figma-slides-with-noah-finer | Building Figma Slides | journalism | OQ1.3, OQ1.4 |
| https://konvajs.org/docs/sandbox/Editable_Text.html | Konva editable text | vendor | OQ1.4 |
| https://geekyants.com/blog/building-a-production-ready-canva-like-editor-with-konva-js-react-19-and-next-js-15 | Canva-like editor with Konva | practitioner | OQ1.4, Reconciled 3 |
| daybrush moveable/selecto LICENSE, issues 1137, 1156 | moveable, selecto | vendor / practitioner | OQ1.5, Reconciled 2 |
| ~/Code/blattwerk (Editor.tsx, history.ts, saves.ts, sheets.py, mcp/tools.ts, docs.ts, okf/*, oauth.py, remote.py, pdf.py, NOTICE, issues 201, 732, 889) | Blattomat | practitioner | OQ1.5, OQ4, OQ8, Cross-cutting, Outliers |
| ~/Code/pgoell-claude-tools/plugins/presentations (deck-stage.js, SKILL.md files, recipes, hard-gates.md, presets, editing-presentations assets) | presentations plugin | practitioner | OQ1.3, OQ2.4, OQ5.1, OQ6.2, OQ7, Cross-cutting |
| private deck repo | aggregate counts only | practitioner | OQ2.4, OQ5.1, OQ6.2, Cross-cutting, Outliers |
| webapp.docx4java.org ECMA-376 mirror pages (sldMaster, sldLayout, ph, txStyles, clrScheme, clrMap, seq, cTn, bldP, txEl, animMotion, transition ...) | ECMA-376 Part 1 | independent | OQ2.1, OQ6.1 |
| learn.microsoft.com Open XML SDK slide masters / layouts | Open XML SDK docs | vendor | OQ2.1 |
| python-pptx docs (placeholders-understanding, dev analysis pages, concepts, charts) | python-pptx docs | practitioner / vendor | OQ2.1, OQ2.4, OQ5.1 |
| python-pptx issues 413, 1109, 175, 864, 954, 1106, 264, 1112; PR 1115; file_with_bounce.pptx | python-pptx issues | practitioner | OQ2, OQ5, OQ6, Reconciled 5-6 |
| https://learn.microsoft.com/en-us/answers/a/11388916 | placeholder order across layouts | practitioner | OQ2.1 |
| LibreOffice bugs 104202, 128547, 139897, 128821, 169343, 116736, 34407; commontimenodecontext.cxx | LibreOffice | independent | OQ2.1, OQ6, Reconciled 5 |
| developers.google.com Slides API (pages, request, page-elements, create-slide, batch, batchUpdate, comments, CommentThread, add-chart, configure-mcp-server) | Google Slides API | vendor | OQ2.2, OQ3.4, OQ5.2, OQ6.2, OQ7, OQ8 |
| Pitch help articles (slide styles, layouts, templates, export, agent, AI actions, comments, status, MCP) ; pitch.com/whats-new ; blog | Pitch | vendor | OQ2.3, OQ3.4, OQ4, OQ7 |
| Apple Keynote user guide pages; HT5591 archive | Keynote | vendor | OQ2.3, OQ6.2, OQ6.3 |
| Gamma help, developers.gamma.app/llms-full.txt, ideas board, Gamma 5 archive | Gamma | vendor / practitioner | OQ2.3, OQ3.4, OQ4, OQ5.2, OQ7, Reconciled 11 |
| Figma help, forum, REST API comments docs, Plugin Annotation, MCP server docs | Figma | vendor / practitioner | OQ2.3, OQ4, OQ7, Reconciled 13 |
| gitbrent PptxGenJS docs, source at 3c9ec1b, issues 1324, 1449, 1453, 1208, 856, 1422, 1443, 604, 327, 704 | PptxGenJS | vendor / practitioner | OQ2.4, OQ5.1, OQ6.3, Reconciled 4 |
| Anthropic pptx SKILL.md and LICENSE.txt | Anthropic pptx skill | vendor / practitioner | OQ2.4, OQ3.4, OQ5.1 |
| sli.dev guides; slidevjs/slidev PRs 1603, 2722, 2758, 2709; issues 1959, 2365; MCP server.ts | Slidev | vendor / practitioner | OQ3.1, OQ5.3, OQ6, Reconciled 9 |
| https://www.pkgpulse.com/guides/slidev-vs-marp-vs-revealjs-code-first-presentations-2026 | Slidev vs Marp vs reveal.js | practitioner | Reconciled 9 |
| typst, touying-typ repos, tylina | Typst, Touying, Tylina | vendor | OQ3.2 |
| https://arxiv.org/html/2608.07617 | TeXFix-Bench | independent | OQ3.2, Reconciled 10 |
| Typst forum threads 4085, 1331; HN mirror 45693978 | LLMs on Typst | practitioner | OQ3.2, Reconciled 10 |
| https://aider.chat/2024/08/14/code-in-json.html | code in JSON | practitioner | OQ3.3, Reconciled 15 |
| https://arxiv.org/abs/2408.02442 ; https://blog.dottxt.ai/say-what-you-mean.html | Let Me Speak Freely; Say What You Mean | independent / vendor | OQ3.3, Reconciled 15 |
| https://arxiv.org/abs/2510.04717v1 | JSON Whisperer | independent | OQ3.3, Reconciled 15 |
| https://arxiv.org/html/2608.24103v1 ; huggingface papers 2608.24103 | ACE | independent (preprint) | OQ3.3, OQ4 |
| https://arxiv.org/html/2512.03042 | PPTArena | independent | OQ3.3 |
| https://arxiv.org/abs/2505.11604 | Talk to Your Slides | independent | OQ3.3, OQ4 |
| https://arxiv.org/abs/2311.01767 | PPTC | independent | OQ3.3, OQ4 |
| https://nbrosse.github.io/posts/llm-slides/llm-slides.html | LLM slide formats comparison | practitioner | OQ3.3, Outliers |
| slidev-addon-studio README, LICENSE | Slidev Studio | vendor | OQ3.3 |
| quarto.org visual editor, revealjs advanced, powerpoint | Quarto | vendor | OQ3.3 |
| revealjs.com markdown, fragments, auto-animate; reveal.js source | reveal.js | vendor | OQ3.3, OQ6 |
| marp-team marp-cli, marpit; masaki39/marp-mcp | Marp | vendor | OQ3.3, OQ3.4, OQ5.3 |
| canva.dev MCP docs; Canva help archives; cmswire; fastcompany.co.za | Canva | vendor / journalism | OQ3.4, OQ4, OQ7 |
| Microsoft message center mirrors (RM548646, MC1219792, MC1243550, MC1473166, MC1187796); support pages Copilot | Copilot in PowerPoint | vendor | OQ4, OQ7, Reconciled 12 |
| learn.microsoft.com answers 1908366, 12080966 | Copilot layout field reports | practitioner | OQ4 |
| https://docs.primedocs.io/en/docs/4.0/layouter/best-practices/microsoft-365-copilot | Copilot best practices | vendor | OQ4 |
| Gemini in Slides blog and help 16961475 | Gemini in Slides | vendor | OQ4 |
| Plus AI guide; plusai.com Pitch review | Plus AI | vendor / vendor (competitor) | OQ4 |
| Beautiful.ai classic slides archive; aumiqx; dupple | Beautiful.ai | vendor / practitioner | OQ4 |
| venturebeat Tome; chatslide Tome | Tome | journalism / vendor (competitor) | OQ4 |
| reddit r/GammaApp, r/powerpoint, r/GoogleSlides (via arctic-shift) | user reports | practitioner (Gamma praise weighted down: sponsorship claim) | OQ4 |
| tiptap review-changes; aiuxplayground; Microsoft HAX 9C | review patterns | vendor / practitioner | OQ4 |
| https://www.inkandswitch.com/patchwork/notebook/2024-version-control/ (07 and index) | Patchwork | independent | OQ4, OQ8 |
| https://anthropic.com/news/claude-design-anthropic-labs | Claude Design | vendor | OQ4 |
| pypi/npm/GitHub API metadata for libraries | registry metadata | vendor | OQ1.5, OQ5 |
| power-pptx, pptx-automizer, officegen, Open XML SDK, Aspose, Syncfusion, pptx-embed-fonts, dom-to-pptx | pptx libraries | vendor / practitioner | OQ5.1, OQ5.3 |
| ECharts handbook, LICENSE, NOTICE, issues; zrender | ECharts | vendor / practitioner | OQ5.2 |
| Vega, Vega-Lite, vl-convert docs and LICENSE | Vega | vendor | OQ5.2 |
| Chart.js, Observable Plot, Recharts (issues 5997, 6139), D3 | chart libraries | vendor / practitioner | OQ5.2 |
| https://www.thinkcell.com/en/resources/manual/compatibility | think-cell compatibility | vendor | OQ5.2 |
| Microsoft support: embedding fonts, SVG in Office, Morph tips, modern comments | Microsoft support | vendor | OQ5.4, OQ6, OQ7 |
| Collabora LibreOffice 25.8 | LibreOffice font embedding | vendor | OQ5.4 |
| OfficeCLI; ChristopherVR/ooxml; Aspose FOSS mappings; Resource2Skill; svg2ooxml spec | animation writers | vendor / practitioner | OQ6, Reconciled 5-7 |
| MDN Web Animations API, reverse, offset-path | Web Animations | independent | OQ6.4 |
| rdpslides PPT FAQ 598 | animations on going back | practitioner | OQ6.4 |
| BrightCarbon convert to Google Slides; slidemodel | pptx to Google | practitioner | OQ6.3, Reconciled 8 |
| keynote.org.cn; discussions.apple.com 250144385; O'Reilly iWork '09 | Keynote export | practitioner / journalism | OQ6.3 |
| Berger, Sacha Chua, reveal animate plugin | SVG part animation | practitioner | OQ6.4 |
| MS-PPTX CT_Comment, ST_CommentStatus; MS-ODRAWXML monikers | PowerPoint comments spec | vendor | OQ7 |
| office-watch.com comments | PowerPoint comments follow cut/paste | journalism | OQ7 |
| Drive API manage-comments, comments resource; workspaceupdates 2026-09; releases.sh tracker | Google comments | vendor / practitioner | OQ7 |
| github-mcp-server README; GitHub Copilot docs; Vercel comments docs and CLI; react-grab | agent comment loops | vendor | OQ7 |
| https://arxiv.org/html/2410.22177 | LLM 3D scene editing with pointing | independent | OQ7 |
| Yjs INTERNALS, docs, y-utility README, discuss.yjs.dev 492, 3729, y-prosemirror README, Liveblocks guide and example | Yjs | vendor / practitioner | OQ8 |
| Automerge docs (merge rules, conflicts, modeling data, refs, implementation.ts, blog Automerge 3) | Automerge | vendor | OQ8 |
| loro-docs pages; Loro README | Loro | vendor | OQ8 |
| https://github.com/dmonad/crdt-benchmarks | CRDT benchmarks (by Yjs author, old versions) | practitioner | OQ8 |
| Figma multiplayer, ordered sequences, reliability blogs | Figma multiplayer | vendor | OQ8 |
| https://madebyevan.com/algos/crdt-fractional-indexing/ | fractional indexing (Wallace) | practitioner | OQ8 |
| Excalidraw P2P blog, PRs 7359, 7348, 11258, 11934 | Excalidraw | vendor | OQ8 |
| https://martin.kleppmann.com/papers/list-move-papoc20.pdf | Moving Elements in List CRDTs | independent | OQ8 |
| Jamsocket "You might not need a CRDT" (Wayback) | Jamsocket | vendor | OQ8 |
| Google Docs 2010 conflict resolution blog | Google Docs OT | vendor | OQ8 |
| https://github.com/wzhudev/reverse-linear-sync-engine | Reverse Linear sync | practitioner (unaffiliated) | OQ8 |
| Cambria; Live & Local Schema Change (arXiv 2309.11406) | schema evolution | independent | OQ8 |
| RFC 6902 | JSON Patch | independent | OQ8 |
| Hocuspocus source; Tiptap server AI toolkit | server agent writes | vendor | OQ8 |
| local license reads in local-prior-art table (React, FastAPI, Playwright, PyMuPDF, CairoSVG, Pillow, etc.) | dependency licenses | vendor | License table, Outliers |
