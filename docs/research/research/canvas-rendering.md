# Canvas technology: HTML elements versus a canvas library such as tldraw (open question 1)

Scope: open question 1 of `docs/spec-v0.md` ("Draw the canvas as HTML elements (Blattomat, Pitch) or on a canvas library such as tldraw? Check tldraw's license by reading its LICENSE file"). Research date 2026-10-20. Notes only; no conclusions.

Method notes:
- LICENSE files were read raw from GitHub (`raw.githubusercontent.com`) and through `gh api`, never from badges.
- "First-hand observation" means I loaded the page in headless Chrome (Chrome 155, isolated browser context, 2026-10-20) and read the DOM with a script. These are tagged `[practitioner]` because they are my own inspection, not a vendor statement.

---

## SQ1: What is tldraw's license today, what does it allow without a paid key, and what does a business license cost?

### The LICENSE file as it stands (main branch, read 2026-10-20)

The repo root `LICENSE.md` is titled "tldraw license" and is a proprietary, source-available license, not an OSI license [source: https://raw.githubusercontent.com/tldraw/tldraw/main/LICENSE.md | tldraw license (LICENSE.md, main) | vendor].

Definitions, quoted verbatim:
- "\"Production Environment\" means any production deployment of the Software that operates on servers, cloud platforms, web applications, or where the software is used to provide functionality to end users, customers, or the public. Production Environment excludes internal development." [source: https://raw.githubusercontent.com/tldraw/tldraw/main/LICENSE.md | tldraw license | vendor]
- "\"Development Environment\" means any internal hosting or deployment of the Software for development, testing, or staging purposes, operated by your organization and not accessible to end users, customers, or the public." [source: same | tldraw license | vendor]
- "\"License Key\" means the programmatically generated key that controls Software functionality and usage restrictions." [source: same | tldraw license | vendor]

Permissions, quoted: "Use the Software in Development Environments. Modify the Software to suit your needs. Bundle the Software with your own projects. Submit modifications of the Software to tldraw." [source: same | tldraw license | vendor]

Conditions, quoted: "Not to use the Software in Production Environments." "Not to disable, change, or interfere with the Software's License Key enforcement." "Not to remove any copyright or other notices from the Software." "Not to distribute the Software or modifications of the Software as a standalone product, but only as part of another application." "To include a verbatim copy of this License in any distribution of the Software." "To comply with tldraw's trademark policy." [source: same | tldraw license | vendor]

Trial clause, quoted: "In the case that tldraw makes the Software available to you on a trial basis, you are permitted to use the Software in Production Environments for the specified period beginning from the date of the trial's License Key issuance. Business units are limited to one trial period unless otherwise approved by tldraw in writing." [source: same | tldraw license | vendor]

Commercial clause, quoted: "In the case that tldraw makes the Software available to you under the terms of a separate commercial agreement, you are permitted to use the Software in Production Environments for the specified period beginning from the date of the agreement's License Key issuance." [source: same | tldraw license | vendor]

Technical enforcement clause, quoted: "The Software includes technical measures to verify License Key validity, detect deployment environments, enforce usage restrictions based on license type, and ensure proper watermark display. The Software may collect and transmit usage data to tldraw for license compliance purposes." [source: same | tldraw license | vendor]

Other clauses: governed by Delaware law with exclusive Delaware jurisdiction; termination on breach or on a copyright, trade secret or patent claim against tldraw; "tldraw retains all right, title, and interest in the Software" [source: same | tldraw license | vendor].

`packages/tldraw/LICENSE.md` and `packages/editor/LICENSE.md` each hold one line: "This code is licensed under the [tldraw license](https://github.com/tldraw/tldraw/blob/main/LICENSE.md)" [source: https://raw.githubusercontent.com/tldraw/tldraw/main/packages/tldraw/LICENSE.md | packages/tldraw/LICENSE.md | vendor] [source: https://raw.githubusercontent.com/tldraw/tldraw/main/packages/editor/LICENSE.md | packages/editor/LICENSE.md | vendor].

GitHub's license detector reports `NOASSERTION` for the repo (it does not recognise the custom text) [source: `gh api repos/tldraw/tldraw` (.license.spdx_id), read 2026-10-20 | GitHub API | vendor].

### Per-package license split (each package's LICENSE.md and package.json read directly)

MIT ("MIT License, Copyright (c) 2024 tldraw Inc."): `@tldraw/store`, `@tldraw/state`, `@tldraw/state-react`, `@tldraw/utils`, `@tldraw/validate`, `@tldraw/tlschema`; each package.json says `"license": "MIT"` [source: https://raw.githubusercontent.com/tldraw/tldraw/main/packages/store/LICENSE.md and the same path for state, state-react, utils, validate, tlschema | tldraw package LICENSE files | vendor].

tldraw license ("SEE LICENSE IN LICENSE.md", one-line pointer to the root license): `tldraw`, `@tldraw/editor`, `@tldraw/sync`, `@tldraw/sync-core`, `@tldraw/commenting`, `@tldraw/collaboration`, `@tldraw/mentions`, `@tldraw/driver`, `@tldraw/mermaid` [source: https://raw.githubusercontent.com/tldraw/tldraw/main/packages/sync/LICENSE.md and the same path for sync-core, commenting, collaboration, mentions, driver, mermaid | tldraw package LICENSE files | vendor].

The MIT package descriptions: store is "tldraw infinite canvas SDK (store)", tlschema is "(schema)", state is "(state)", validate is "A runtime validation library by tldraw" [source: packages/*/package.json in github.com/tldraw/tldraw main | package.json | vendor]. The editor, shape utils, UI and rendering are in `tldraw` and `@tldraw/editor`, both under the tldraw license [source: same].

### History of the license (commit log of LICENSE.md)

Commits touching `LICENSE.md` [source: `gh api 'repos/tldraw/tldraw/commits?path=LICENSE.md'`, read 2026-10-20 | GitHub API | vendor]:
- 2023-12-19 `3cf4dae34d` "Change licenses to tldraw (#2167)".
- 2024-07-22 `9cb6da32b2` "license: add example; fix up example dialog (#4163)".
- 2024-09-13 `f26b077c73` "3.0 megabus (#4494)".
- 2024-10-24 `7bf11455fa` "Fix license link (#4770)".
- 2025-09-17 `e455ab838b` "License change for 4.0 (#6780)". The PR body says only "This PR updates the language of the tldraw license" [source: https://github.com/tldraw/tldraw/pull/6780 | chore: update license for 4.0 | vendor].

The text introduced on 2023-12-19 was a non-commercial license: "You may use this Software for any non-commercial purpose ... You may not use or distribute this Software or any derivative works in any form for commercial purposes. Examples of commercial purposes would be running business operations, ... for internal products within commercial entities" [source: https://raw.githubusercontent.com/tldraw/tldraw/3cf4dae34d/LICENSE.md | tldraw License (2023-12-19) | vendor].

The text just before 4.0 (3.x line) allowed commercial use with a watermark: "Use the Software in your commercial or non-commercial projects." with conditions "Not to disable, hide, remove, or alter the Watermark." and "Not to disable, change, or interfere with the license key validation process that governs the display of the Watermark." and "If you wish to use the tldraw SDK without displaying the Watermark, please visit https://tldraw.dev/#pricing to purchase a Business License." [source: https://raw.githubusercontent.com/tldraw/tldraw/e455ab838b~1/LICENSE.md | tldraw license (pre-4.0) | vendor].

The 4.0 announcement: "the tldraw SDK is only permitted to be used in development environments." "This policy is enforced through the license keys that come with each license." "The SDK will only work in production when it has a valid license key." Production needs "either a trial license, a commercial license, or a hobby license." [source: https://tldraw.dev/blog/tldraw-sdk-4-0 | Announcing tldraw SDK 4.0 | vendor]. The 4.0 post offered to discount a one-year commercial agreement signed before the end of 2025 by the time left on the trial [source: same].

Steve Ruiz on HN about the 4.0 change: "You can stay on 3.x." and commercial licenses "will still work for 3.x too." [source: https://news.ycombinator.com/item?id=45293839 | Tldraw SDK 4.0 (HN thread) | practitioner].

Latest release at research time: v5.5.2, published 2026-10-02 [source: `gh api repos/tldraw/tldraw/releases/latest` | GitHub API | vendor]. SDK 5.0 was announced 2026-05-07 and the announcement mentions no license change [source: https://tldraw.dev/blog/tldraw-sdk-5-0 | tldraw SDK 5.0 | vendor].

### What works without a paid key today (vendor pages)

- "The SDK will work in production only when provided with a valid and active license key." "License keys are validated on the client." "The tldraw SDK will not work in production without a valid license key." [source: https://tldraw.dev/community/license | tldraw license page | vendor]
- Trial: "You can get a free 100-day trial license"; "The license key will stop working when the trial ends." [source: same | tldraw license page | vendor]
- Hobby: "a discretionary hobby license" for non-commercial projects; "When using the tldraw SDK under a hobby license, the \"made with tldraw\" watermark must be shown on the canvas." [source: same | tldraw license page | vendor]
- Ruiz, 2026-06-12: "Production use of the tldraw SDK requires a license key." "The 'made with tldraw' watermark appears with the free hobby license." The watermark goes away with a commercial license set through the `licenseKey` prop [source: https://tldraw.dev/blog/20-things-i-wish-ai-chatbots-knew-about-tldraw | 20 things I wish AI chatbots knew about tldraw | vendor].
- Commenting is licensed separately: "Commenting is a licensed feature. It runs in development without a key. In production it needs a tldraw license that includes commenting." [source: https://tldraw.dev/llms-full.txt (section "Commenting") | tldraw docs, full text | vendor]

### Price: three dated snapshots

- Archived pricing page, 2025-10-10: "Startup $6,000/yr For businesses with 10 employees or fewer. Full SDK features. Host anywhere. 2 production domains. Community support." and "Business Custom Growing and larger companies. ... Unlimited production domains." plus a 100-day trial at $0 and "Apply for a hobby license" [source: http://web.archive.org/web/20251010182142/https://tldraw.dev/pricing | tldraw pricing (archived 2025-10-10) | vendor].
- Archived pricing page, 2026-08-28: no figure; "Need to go to production? Start with a free trial or apply for an annual license. The SDK License ... Multiplayer sync included", "Apply for startup pricing", "Get a free hobby license" [source: http://web.archive.org/web/20260828114609/https://tldraw.dev/pricing | tldraw pricing (archived 2026-08-28) | vendor].
- Live pricing page, 2026-10-20: "Value-based pricing"; SDK License is a "Commercial license for production use" with a 100-day trial, "No credit card required"; "Discounted pricing is available for new companies"; hobby license is free on application; "Premium modules" are "in development" [source: https://tldraw.dev/pricing | tldraw pricing | vendor].

Community reaction to the 4.0 terms and the $6,000 figure (HN, September 2025): phgn "How is $6000 per year (upfront) affordable?"; abxyz "I read $6,000/year as 'we don't want your business'" and calls it "a big gift to Excalidraw (and others)"; colesantiago "$6K a year and getting only community support is a huge risk for some SMEs"; max1990 "no visibility into scale pricing is a risk"; in defence matt-p "$6000 feels quite reasonable" and florians "Peanuts if you build a business on top of the SDK" [source: https://news.ycombinator.com/item?id=45293839 | Tldraw SDK 4.0 (HN thread) | practitioner].

Ruiz in the same thread on how the price was set: when asked, teams said "nothing because we don't have any money yet" or "a number between $5,000 and $10,000"; "Small teams are so hard to price for."; "Pricing is such whackamole."; "I expect we'll do extended trial licenses for teams that are serious but just getting started." [source: same | HN thread | practitioner].

Earlier pricing idea from Ruiz (2023-11): "$1 / user / year? ... we were going to have a flat fee around 10k for \"small\" teams <10k users" [source: https://x.com/steveruizok/status/1729410167983263827 | Steve Ruiz on X | practitioner]. Secondary press repeats "$6,000 USD annually per team" [source: https://biggo.com/news/202509190115_tldraw_SDK_4.0_Licensing_Debate | tldraw SDK 4.0 Licensing Changes Spark Community Debate | journalism].

### MIT alternative in the same space

Excalidraw's LICENSE: "MIT License, Copyright (c) 2020 Excalidraw"; GitHub detects MIT; last push 2026-10-20 [source: https://raw.githubusercontent.com/excalidraw/excalidraw/master/LICENSE | Excalidraw LICENSE | vendor]. Excalidraw draws its scene to a 2D canvas (`CanvasRenderingContext2D` in `renderer/staticScene.ts`, plus a separate `staticSvgScene.ts` for SVG) [source: https://github.com/excalidraw/excalidraw/tree/master/packages/excalidraw/renderer | Excalidraw renderer folder | vendor] and edits text through a textarea overlay: `const editable = ownerDocument.createElement("textarea");` with `position: "absolute"` placed from the element's viewport coordinates [source: https://github.com/excalidraw/excalidraw/blob/master/packages/excalidraw/wysiwyg/textWysiwyg.tsx | textWysiwyg.tsx | vendor].

---

## SQ2: How does tldraw render and model shapes, has anyone built slides on it, and what would a fixed 1920x1080 slide model with masters take?

### Rendering

- "Shapes on the tldraw canvas are HTML and CSS in the DOM, rendered by React, not drawn to a 2D `<canvas>` element." "Canvas elements are used only for small things like selection indicators and the minimap." Because shapes are DOM elements, "you can put working web content directly on the canvas: buttons, videos, iframes, embedded documents, even other tldraws." [source: https://tldraw.dev/blog/20-things-i-wish-ai-chatbots-knew-about-tldraw | 20 things I wish AI chatbots knew about tldraw (Steve Ruiz, 2026-06-12) | vendor]
- SDK 5.0 (2026-05-07) moved indicators, handles and other overlays from individual React SVGs into one HTML canvas: "these ephemeral overlays are now rendered more efficiently into a single HTML canvas." [source: https://tldraw.dev/blog/tldraw-sdk-5-0 | tldraw SDK 5.0 | vendor]
- Overlay utils "draw directly to a `<canvas>` element using the Canvas 2D API": selection handles, brush rectangles, snap lines, scribbles, shape handles [source: https://tldraw.dev/llms-full.txt (section "Overlay utils") | tldraw docs | vendor].
- Culling: "Culled shapes stay in the DOM with `display: none` ... and they stay in the store, so they can still be selected, hit-tested, and exported." [source: https://tldraw.dev/llms-full.txt (section "Culling") | tldraw docs | vendor]. "a canvas with 10,000 shapes might only render 50 if the rest are out of view." [source: https://tldraw.dev/llms-full.txt (section "Performance") | tldraw docs | vendor]
- Reactive signals: "When a shape's props change, only that shape's component re-renders, not the entire canvas." [source: same, section "Performance" | tldraw docs | vendor]
- Text is measured with "a hidden DOM element" (`TextManager`), and `FontManager` loads fonts before measuring [source: https://tldraw.dev/llms-full.txt (TextManager/FontManager passage) | tldraw docs | vendor].
- 2022 podcast notes: tldraw "uses DOM and SVG instead of Canvas for easier custom components"; canvas "is painful for overlays and animations" [source: https://syntax.fm/505 | Syntax 505, Supper Club with Steve Ruiz (show notes as summarised by search; transcript not read in full) | practitioner].

### Store and records

- "The store is tldraw's reactive database. It holds all shapes, pages, bindings, assets, and other records ... It validates all records against a schema and tracks every change for undo/redo, persistence, and synchronization." [source: https://tldraw.dev/llms-full.txt (section "Store") | tldraw docs | vendor]
- Record shape: `{ id: 'shape:abc123', typeName: 'shape', type: 'geo', x: 100, y: 200, props: { geo: 'rectangle', w: 300, h: 150, color: 'blue' } }`; ids are "branded" strings with a type prefix (`shape:`, `page:`, `binding:`) [source: same | tldraw docs | vendor].
- `TLBaseShape` carries "a unique identifier, position and rotation, z-ordering index, parent reference, lock state, opacity, and a `props` field"; a `meta` field holds "your own application data, which tldraw stores but doesn't use itself" [source: https://tldraw.dev/llms-full.txt (section "Shapes") | tldraw docs | vendor].
- Default shape types: `geo`, `text`, `note`, `draw`, `line`, `highlight`, `image`, `video`, `bookmark`, `embed`, `frame`, `group`, `arrow` [source: same | tldraw docs | vendor].
- Changes run through `editor.createShapes`, `editor.updateShapes`, `editor.run(...)`; batched changes "become a single undo step"; `history: 'ignore'` leaves undo untouched [source: https://tldraw.dev/docs/editor | Editor | vendor].

### Custom shapes

- A ShapeUtil defines "how it renders, its geometry for hit testing" and interactions; required methods are "getDefaultProps, getGeometry, component, and getIndicatorPath"; the example component returns `<HTMLContainer>` [source: https://tldraw.dev/docs/shapes | Shapes | vendor].

### Rich text

- "tldraw uses TipTap (a headless editor toolkit built on ProseMirror) as the rich text engine. Text is stored as structured JSON rather than plain strings"; text, note, geo and arrow label shapes support it; the doc is a `TLRichText` tree of `doc` > `paragraph` > `text` with `marks` such as `bold` [source: https://tldraw.dev/llms-full.txt (section "Rich text") | tldraw docs | vendor].

### Export to SVG and PNG

- "Export has two stages: SVG generation and optional raster conversion." The editor "creates a React tree representing the SVG, renders it into a temporary DOM element, and processes it to be self-contained." If a ShapeUtil implements `toSvg`, that is used; "Otherwise the editor renders the shape's normal HTML inside an SVG `<foreignObject>` element." [source: https://tldraw.dev/llms-full.txt (section "Image export") | tldraw docs | vendor]
- Fonts are inlined as data URLs by `FontEmbedder`; computed styles inside `<foreignObject>` are written inline by `StyleEmbedder`; images become data URLs, videos "a single captured frame" [source: same | tldraw docs | vendor].
- Raster: the SVG is loaded into an `Image`, drawn to a canvas, exported as a blob; `pixelRatio` defaults to 2; options include `bounds`, `scale`, `padding`, `background`, `darkMode`; formats png, jpeg, webp, svg [source: same | tldraw docs | vendor].
- The PDF editor example renders each PDF page with `pdfjs-dist` as a locked image shape, confines the camera with `setCameraOptions` (`contain`), and exports with `editor.toImage` per page `bounds`, stamped onto the original PDF with `pdf-lib` [source: https://tldraw.dev/llms-full.txt (section "PDF editor") | tldraw docs | vendor]. That is, the annotations go into the PDF as raster images from `toImage` (as the example describes it).

### Slides built on tldraw

- Repo example "Slideshow (free camera)": "A `slide` custom shape is an unfilled dashed rectangle you can draw with the slide tool (`S`), place other shapes on, and move around freely." Clicking a slide or pressing arrow keys "calls `editor.zoomToBounds` to animate the camera to that slide." Default slide props `w: 720, h: 480` [source: https://github.com/tldraw/tldraw/tree/main/apps/examples/src/examples/use-cases/slides (README.md, SlideShapeUtil.tsx) | Slideshow (free camera) example | vendor].
- Repo example "Slideshow (fixed camera)": "Slides are frame shapes laid out in a row." On slide change it "calls `editor.setCameraOptions` with `bounds` set to that frame and `behavior: 'contain'`"; "Side effects keep the slide frames from being moved, selected, or hovered" [source: https://github.com/tldraw/tldraw/tree/main/apps/examples/src/examples/use-cases/slideshow (README.md) | Slideshow (fixed camera) example | vendor].
- tldraw offline presentation mode, 2026-07-22: a tldraw PM "turned tldraw offline into a slide deck tool"; "camera controls, frames, and slide content are all from built-in pieces of the tldraw SDK"; "it was easy to add in a few react components to handle the slide menu"; the post suggests extending it to "support common presentation tool options like templates, themes, or transitions" [source: https://tldraw.dev/blog/building-with-tldraw-offline | Building with tldraw offline | vendor].
- The tldraw showcase lists an unnamed "AI presentation tool with smart blocks"; I could not identify the product [source: https://tldraw.dev/showcase | Made with tldraw (as summarised by search) | vendor].
- Make Real turns sketches into working websites; tldraw computer (with a Google AI team) builds natural-language workflows; neither is described as a slide tool [source: https://tldraw.dev/blog/make-real-the-story-so-far | make real the story so far | vendor] [source: https://ai.engineer/talks/tldraw-computer | tldraw computer talk page | vendor].
- The agent starter kit replaced the removed `@tldraw/ai` module; it has "a mode system for controlling agent capabilities per mode, action schema registries, prompt part definitions, canvas linting" [source: https://tldraw.dev/llms-full.txt (release notes, PR #7640) | tldraw docs | vendor].

### Pieces relevant to a fixed 1920x1080 slide with masters

- Camera constraints: `TLCameraOptions` with "constraints that keep the camera inside a bounded area with a chosen fit behavior, padding, and origin"; the docs example switches bounds between "A4 page" and "landscape" [source: https://tldraw.dev/llms-full.txt (section "Camera options") | tldraw docs | vendor].
- Snapping: "Bounds snapping aligns edges, centers, and corners during movement and resizing, and also keeps gaps between shapes consistent." "Snapping is off by default. Users hold Ctrl (Cmd on Mac) while translating, resizing, or dragging a handle to snap." Snap mode inverts this; grid snapping is separate (`isGridMode`) [source: https://tldraw.dev/llms-full.txt (section "Snapping") | tldraw docs | vendor].
- Locked shapes, readonly mode, pages, shape clipping and side effects are documented features [source: https://tldraw.dev/llms-full.txt (sections "Locked shapes", "Readonly mode", "Pages", "Shape clipping") | tldraw docs | vendor].
- A grep of the 2.5 MB full docs for "slide master", "master slide", "placeholder shape" and "layout template" found no master or layout concept; the one "placeholder" hit is about an `ErrorShape` indicator [source: https://tldraw.dev/llms-full.txt, grep run 2026-10-20 | tldraw docs | vendor]. Masters, layouts and placeholders would therefore be custom code on top of the SDK [author estimate: based on the absence of any such concept in the docs and both slide examples using plain frames or a custom shape].
- Comments in `@tldraw/commenting`: a thread's `anchor` is a union of point, shape, region and `page`; "Shape-anchored threads outlive their shape. Delete the shape and the thread converts to a `point` anchor where its pin last sat"; precise and imprecise shape anchors "both track the shape as it moves and resizes" [source: https://tldraw.dev/llms-full.txt (section "Commenting") | tldraw docs | vendor]. The package is under the tldraw license and needs a key that includes commenting in production (SQ1).

---

## SQ3: How do browser slide editors draw slides, and what do engineering sources say about DOM vs canvas vs SVG?

### Pitch: DOM (first-hand observation of the public player)

Page inspected: https://pitch.com/presentations/Pitch-Product-Highlights--June-2026-0EC3YQ4kjWws4gzyps22EY2v (public presentation player, not the logged-in editor), 2026-10-20 [source: first-hand DOM inspection, headless Chrome | Pitch public player | practitioner]:
- No `<canvas>` and no `<iframe>` on the page.
- The slide is `div.slide` with inline `width: 1920px; position: relative; font-size: 40px`; its parent has `width: 1920px; transform-origin: left top; transform: scale(0.345426)`. That is, a 1920 px wide slide in DOM, scaled to fit.
- Each block is a `div.position` with `transform: translate(Xpx, Ypx) rotate(0deg); position: absolute; z-index: N`.
- Shapes are inline `<svg>` with a `<rect>`; text sits in `div.text-in-shape` > `div.ql-editor block text read-only-text paragraph`. The `ql-editor` class is Quill's editor class, here in read-only mode [author estimate: class name matches the Quill rich text editor's convention; not confirmed by Pitch].
- Images are `<picture>`/`<img>` inside wrappers; video clips are `<video>` with a poster image.

Stack signals: Pitch uses UIx (ClojureScript interface to React) in production [source: https://building.nubank.com/building-elegant-interfaces-with-clojurescript-react-and-uix/ | Building elegant interfaces with ClojureScript, React and UIx | practitioner] [source: https://github.com/pitch-io/uix | pitch-io/uix | vendor]. A Pitch job ad asks for knowledge of "React (Reagent), Re-frame or similar reactive systems" and keeping rendering at 60 fps [source: https://startup.jobs/frontend-performance-engineer-mfd-at-pitch | Frontend Performance Engineer at Pitch | vendor].

No Pitch engineering blog post on rendering found after two searches [No usable sources for the "Pitch engineering blog" angle].

### Google Slides: SVG (first-hand observation of the editor)

Page inspected: https://docs.google.com/presentation/d/1EAYk18WDjIG-zp_0vLm3CsfQh_i8eXc67Jo2O9C6Vuc/edit (public sample deck "Baby album" used in Slides API docs), anonymous view, 2026-10-20 [source: first-hand DOM inspection, headless Chrome | Google Slides editor | practitioner]:
- No `<canvas>` elements.
- The slide is an `<svg>` 1060x733 CSS px on screen with `viewBox="0 0 403860 279273"` inside a container of class `pages`; a second SVG of the same viewBox sits in `workspace`.
- Inside: `g` x55, `path` x10, `rect` x5, `clipPath` x3, `image` x3, `text` x4. Group ids follow the API object ids: `g#editor-ge63a4b4_1_0`, `g#editor-ge63a4b4_1_0-bg`, `g#editor-ge63a4b4_1_6-paragraph-0`.
- Text is SVG `<text>` with inline style, for example `font-family:verdana;font-size:64px;fill:#61664F;white-space:pre;` and `text-rendering="geometricPrecision"`, with explicit `x`.
- Keyboard input goes through hidden iframes: `iframe.docs-texteventtarget-iframe docs-offscreen-z-index` and `iframe.docs-domreader-iframe`.

Google Docs (not Slides) moved from HTML to canvas rendering in 2021 "to improve performance and improve consistency in how content appears across different platforms", warning that extensions relying on the HTML structure "may no longer work as intended" [source: https://workspaceupdates.googleblog.com/2021/05/Google-Docs-Canvas-Based-Rendering-Update.html | Google Docs Canvas Based Rendering Update | vendor] [source: https://thenewstack.io/google-docs-switches-to-canvas-rendering-sidelining-the-dom/ | Google Docs switches to canvas rendering, sidelining the DOM | journalism]. No announcement of a Slides canvas switch was found, and the 2026-10-20 inspection shows SVG.

A WebKit bug notes a Google Slides deck rendering wrongly in Safari (misplaced SVG arrows) while correct in Chrome and Firefox [source: https://bugs.webkit.org/show_bug.cgi?id=241781 | WebKit Bugzilla 241781 (as summarised by search) | practitioner].

### Figma Slides: C++/WASM renderer to `<canvas>` (WebGL or WebGPU)

- "Figma's core editors use a C++ codebase and custom renderer outputting to a <canvas> element via WebGL or WebGPU." Single slide view "operates by zooming into the infinite canvas and hiding elements, maintaining the 2D space"; it is a "zoomed in version of the grid view". Custom text rendering is used for "consistency across browsers and operating systems", so features such as spellcheck must be built in-house. Episode dated 2025-03-26; Figma Slides public launch 2025-03-19 [source: https://newsletter.pragmaticengineer.com/p/building-figma-slides-with-noah-finer | Building Figma Slides with Noah Finer and Jonathan Kaufman | journalism].
- Figma's renderer gained WebGPU while keeping WebGL compatibility [source: https://www.figma.com/blog/figma-rendering-powered-by-webgpu/ | Figma rendering: Powered by WebGPU | vendor].

### Canva: not determined

- Canva's template pages returned a Cloudflare challenge in headless Chrome; no DOM inspection possible [inaccessible: https://www.canva.com/presentations/templates/].
- Job ads describe the "Lightspeed Renderer" as "Canva's next-generation, general-purpose renderer" built with "Rust alongside wGPU, Data Oriented Design, and WebAssembly", aiming at "dramatically faster first paint ... and consistent fidelity across browsers and OSes" and at "making rendering dramatically cheaper for Export and AI workloads" [source: https://jobs.blackbird.vc/companies/canva/jobs/72657275-lightspeed-renderer-senior-software-engineer | Lightspeed Renderer, Senior Software Engineer (Canva) | vendor]. The ads do not say what the current editor renders with.
- A third-party interview guide claims the Canva editor is "built on a custom WebGL rendering stack" [source: https://www.techinterview.org/companies/canva/ | Canva Interview Guide 2026 (as summarised by search; not fetched) | practitioner]. Unverified.

### PowerPoint for the web: not determined

- The Office viewer (view.officeapps.live.com) loads the slide inside a cross-origin iframe `de5-powerpoint.officeapps.live.com/p/PowerPointFrame.aspx?PowerPointView=ReadingView`; its DOM cannot be read from the outer page. The frame's initial HTML (120,688 bytes) holds one `<svg` tag and no `<canvas` tag, so the slide is built later by script [source: first-hand network inspection, headless Chrome, 2026-10-20 | PowerPoint web viewer | practitioner]. The viewer's CSP lists trusted-type policies named `pptedit#webpack`, `pptviewer#webpack` and `html2canvas` [source: same response headers | PowerPoint web viewer | practitioner]. No rendering evidence for the editor itself. Two searches found no Microsoft write-up [No usable sources for the "PowerPoint for the web rendering" angle].

### The presentations plugin and Blattomat (local sources)

- `deck-stage.js`: "inner canvas is a fixed design size (default 1920×1080) scaled with `transform: scale()` to fit the viewport, letterboxed. Set the `noscale` attribute to render at authored size (1:1), the PPTX exporter sets this so its DOM capture sees unscaled geometry." Slides are hidden, not unmounted, "so their state (videos, iframes, form inputs, React trees) is preserved" [source: ~/Code/pgoell-claude-tools/plugins/presentations/skills/creating-presentations/assets/deck-stage.js lines 1-30 (repo at ad18ab2, 2026-10-20) | deck-stage.js | practitioner].
- Blattomat spec: "Each block is an HTML element placed in mm on an A4 page; drag, resize, snap and multi-select from `moveable` and `selecto`" because of "Sharp text in the PDF, the same markup renders the PDF" [source: ~/Code/blattwerk/docs/spec-v0.md line 148 | Blattomat spec v0 | practitioner].

### Trade-off sources (DOM vs canvas vs SVG)

For DOM or a DOM text layer:
- 2010 W3C canvas-API thread: rebuilding a text editor on canvas means re-implementing selection, clipboard, keyboard handling, spell check, IME and autofill [source: https://lists.w3.org/Archives/Public/public-canvas-api/2010JulSep/0028.html | RE: html5 editor responds to Canvas accessibility related bugs (as summarised by search) | practitioner].
- Konva's own docs: "Users cannot edit `Konva.Text` content directly"; the demo hides the node and overlays a styled `textarea`. Listed limits: the textarea "doesn't re-wrap while the box is resized"; late web fonts make "cached measurements and caret position stale"; `Konva.Text` "applies one style to the whole node, so mixed formatting within a paragraph requires multiple nodes and custom layout"; keystrokes must join the canvas undo stack "or undo will run on two separate timelines"; "Exports are drawn by the canvas, not the textarea" [source: https://konvajs.org/docs/sandbox/Editable_Text.html | Editable Text (Konva docs) | vendor].
- Konva's PDF demo puts the stage into jsPDF as an image (raster); "text rendering in PDF differs from Konva's" [source: https://konvajs.org/docs/sandbox/Canvas_to_PDF.html | Canvas to PDF (Konva docs, as summarised by search) | vendor].
- Figma had to build spellcheck itself because its text is not DOM text [source: https://newsletter.pragmaticengineer.com/p/building-figma-slides-with-noah-finer | Building Figma Slides | journalism].
- David Bushell (2026-09) compares canvas, contenteditable and textarea for an editor: canvas gives full control but no native selection, undo history or multi-line paste [source: https://web-standards.dev/news/2026/09/build-my-own-text-editor/ | Fine, I'll build my own text editor! (as summarised by search) | practitioner].

For canvas:
- GeekyAnts (2026-09-10), building a Canva-like editor: DOM performance "degraded significantly as documents became more complex"; DOM repaints slowed dragging at "50+ elements with rotations, drop shadows, and masks"; "Rotation and corner resize handle math also glitch across different web browsers"; Konva gave "smooth 60 FPS" with "250+ canvas objects". Their canvas downsides: "some browser features simply work better in the DOM" (caret, spell check, copy and paste, selection), solved with temporary HTML overlays; blurry canvas text on high-DPI screens; headless PNG/PDF export had to wait for fonts and images [source: https://geekyants.com/blog/building-a-production-ready-canva-like-editor-with-konva-js-react-19-and-next-js-15 | Building a Production-Ready Canva-like Editor with Konva.js | practitioner].
- Google Docs' stated reasons for canvas: performance and cross-platform consistency (above).

Contradiction: tldraw's docs claim a DOM canvas "with 10,000 shapes might only render 50" through culling [source: https://tldraw.dev/llms-full.txt (Performance) | tldraw docs | vendor], while GeekyAnts reports DOM lag at 50+ elements with rotations, shadows and masks [source: GeekyAnts article above | practitioner]. The two measure different things (off-screen culling vs on-screen effects); neither publishes a benchmark method.

Slide counts for context: a 16:9 slide in the current decks rarely holds hundreds of blocks [author estimate: not measured; the spec's example slide has two blocks, `docs/spec-v0.md` "Deck document"].

---

## SQ4: Licenses and fitness of moveable, selecto and alternatives

All LICENSE files below read raw on 2026-10-20; repo metadata from `gh api repos/<owner>/<repo>` and npm from `npm view` the same day.

### moveable and react-moveable (daybrush)

- LICENSE: "MIT License, Copyright (c) 2019 Daybrush" (repo root and `packages/react-moveable/LICENSE`) [source: https://raw.githubusercontent.com/daybrush/moveable/master/LICENSE | moveable LICENSE | vendor] [source: https://raw.githubusercontent.com/daybrush/moveable/master/packages/react-moveable/LICENSE | react-moveable LICENSE | vendor].
- Last release 0.53.0 on 2023-12-03; last commit 2023-12-03 "chore: Release 0.53.0"; npm `moveable` 0.53.0 and `react-moveable` 0.56.0, both last modified 2023-12-03; 10,767 stars; 454 open issues; 4 open PRs; not archived [source: `gh api repos/daybrush/moveable`, `npm view moveable`, `npm view react-moveable` | GitHub API, npm | vendor].
- `react-moveable` declares no `peerDependencies` [source: `npm view react-moveable peerDependencies` (empty) | npm | vendor]. Blattomat runs it on React `^19.3.1` [source: ~/Code/blattwerk/frontend/package.json | Blattomat package.json | practitioner].
- Issue #1137 (2024-11-20) "Is this project still active?": "no updates, commits, or issue reactions from the maintainer for over a year"; replies "@daybrush any chance for an official status update?" (2025-09-19) and "I guess the project is dead" (2025-09-19) [source: https://github.com/daybrush/moveable/issues/1137 | Is this project still active? | practitioner].
- Issue #1156 (2026-09-16) "Maintainership: offering to help": last commits on master "moveable 2023-12-03, selecto 2023-12-03, gesto 2023-12-01, keycon 2024-07-29"; "`react-moveable` is doing around 412k npm downloads a week" (issue author's figure); "@daybrush's focus has moved to egjs"; no reply at research time (0 comments) [source: https://github.com/daybrush/moveable/issues/1156 | Maintainership: offering to help, and a question about the project's future | practitioner]. The author said on #1137 that without a reply "in about a month I'll publish separately" [source: https://github.com/daybrush/moveable/issues/1137 (comment by colinli007, 2026-09-16) | practitioner].
- A new MIT alternative, `frameable` 0.2.1 (created 2026-09-07), "Headless React hooks for dragging, resizing and rotating frames on a 2D surface"; its author says groups are "planned for 0.2" and are "the part of the design I am least sure about" [source: `npm view frameable` | npm | vendor] [source: https://github.com/daybrush/moveable/issues/1137 (comments by KevinArce98, 2026-09-07 and 2026-09-21) | practitioner].

Blattomat's experience with moveable (local code, `~/Code/blattwerk/frontend/src/pages/Editor.tsx`, 5,874 lines, 53 mentions of "Moveable", repo at 7e4a66b 2026-10-20) [source: ~/Code/blattwerk/frontend/src/pages/Editor.tsx | Blattomat editor | practitioner]:
- line 500: "Moveable reads a selector as its first match only, so it gets the elements."
- line 946: "Moveable cannot resize a box with no height, so lines get no corner handles."
- line 954: "Moveable collapses a selection that holds a flat line, so one with a line gets its own corner handles."
- line 1009: "Moveable needs the elements, and they exist only after the blocks render."
- line 1484: "Moveable ends its drag with no event."; line 1488: "Moveable sends no end of the drag".
- line 4004: "Level lines in a row leave Moveable's group area no height".
- line 4894: "Moveable keeps a click on a selection it cannot drag to itself, so the block under its area is looked up on the way down."
- No `patches/` folder and no patch tooling in `frontend/package.json` [source: ~/Code/blattwerk/frontend (ls, grep) | Blattomat | practitioner].

### selecto and react-selecto (daybrush)

- LICENSE: "MIT License, Copyright (c) 2020 Daybrush" [source: https://raw.githubusercontent.com/daybrush/selecto/master/LICENSE | selecto LICENSE | vendor].
- Last release 1.26.3 on 2023-12-03; last commit 2023-12-03; npm 1.26.3; 2,234 stars; 94 open issues [source: `gh api repos/daybrush/selecto`, `npm view selecto` | GitHub API, npm | vendor]. Blattomat uses `react-selecto ^1.26.3` [source: ~/Code/blattwerk/frontend/package.json | practitioner].

### interact.js

- LICENSE: "Copyright (c) 2012-present Taye Adeyemi" followed by the MIT permission text ("Permission is hereby granted, free of charge, to any person obtaining a copy ..."); GitHub detects MIT [source: https://raw.githubusercontent.com/taye/interact.js/main/LICENSE | interact.js LICENSE | vendor].
- npm `interactjs` 1.10.28 last modified 2026-08-01; last commit 2026-08-01 "chore: remove publish workflow dependency on test"; 12,939 stars; 95 open issues [source: `gh api repos/taye/interact.js`, `npm view interactjs` | GitHub API, npm | vendor]. interact.js gives drag, resize and gesture events; it has no selection box, handles or snap guide lines of its own [author estimate: from its role as a pointer-interaction library; its docs were not read in this round].

### Konva and react-konva (canvas)

- LICENSE: "MIT License. Original work Copyright (C) 2011 - 2013 by Eric Rowell (KineticJS). Modified work Copyright (C) 2014 - present by Anton Lavrenov (Konva)" followed by the MIT text [source: https://raw.githubusercontent.com/konvajs/konva/master/LICENSE | Konva LICENSE | vendor]. GitHub reports `NOASSERTION`, which I read as the detector tripping on the two copyright lines; npm says `MIT` [source: `gh api repos/konvajs/konva`, `npm view konva license` | GitHub API, npm | vendor].
- Konva 10.9.0 released 2026-10-15; 14,878 stars; 2 open issues. react-konva LICENSE "MIT License, Copyright (c) 2017 Anton Lavrenov", v19.3.2 on 2026-10-15, 0 open issues [source: `gh api repos/konvajs/konva`, `gh api repos/konvajs/react-konva`; https://raw.githubusercontent.com/konvajs/react-konva/master/LICENSE | GitHub API, react-konva LICENSE | vendor].
- Konva core has no SVG export: a code search for `toSVG` in the repo returns 0 hits, and `src/Node.ts` has no "svg" string; export goes through `toDataURL`/`toCanvas` [source: `gh api 'search/code?q=repo:konvajs/konva+toSVG'` and a shallow clone grep, 2026-10-20 | Konva source | vendor]. SVG export exists only in third-party packages such as `react-konva-to-svg` [source: https://github.com/dendrofen/react-konva-to-svg | react-konva-to-svg | practitioner].
- Text editing limits: see Konva Editable Text docs under SQ3.

### Fabric.js (canvas)

- LICENSE: "MIT License. Copyright (c) 2008-2015 Printio (Juriy Zaytsev, Maxim Chernyak). Copyright (c) 2016-present Andrea Bogazzi, Shachar Nen and Fabric.js contributors" [source: https://raw.githubusercontent.com/fabricjs/fabric.js/master/LICENSE | Fabric.js LICENSE | vendor].
- Release v7.4.0 on 2026-05-18 (package.json `"version": "7.4.0"`); last commit 2026-10-14 (a Dependabot bump); 31,506 stars; 476 open issues [source: `gh api repos/fabricjs/fabric.js`, `npm view fabric` | GitHub API, npm | vendor].
- Fabric has vector export: `StaticCanvas.toSVG()` in `packages/core/src/canvas/StaticCanvas.ts` (line 931 doc example `var svg = canvas.toSVG();`). Its editable text (`IText`) uses a `hiddenTextarea` for keyboard input (`packages/core/src/shapes/IText/IText.ts`, `ITextKeyBehavior.ts`) [source: https://github.com/fabricjs/fabric.js/tree/master/packages/core/src | Fabric.js source (shallow clone grep, 2026-10-20) | vendor].

### Summary table (facts only)

| Library | Draws with | LICENSE file says | Last release | Open issues |
|---|---|---|---|---|
| tldraw / @tldraw/editor | DOM (React), overlays on 2D canvas | tldraw license (production needs key) | v5.5.2, 2026-10-02 | n/a |
| @tldraw/store, state, tlschema, validate, utils | (data layer) | MIT | same monorepo | n/a |
| moveable / react-moveable | DOM handles over your elements | MIT | 0.53.0 / 0.56.0, 2023-12-03 | 454 |
| selecto / react-selecto | DOM | MIT | 1.26.3, 2023-12-03 | 94 |
| interact.js | DOM events | MIT | 1.10.28 (npm modified 2026-08-01) | 95 |
| Konva / react-konva | 2D canvas | MIT | 10.9.0 / v19.3.2, 2026-10-15 | 2 / 0 |
| Fabric.js | 2D canvas, `toSVG` | MIT | v7.4.0, 2026-05-18 | 476 |
| Excalidraw | 2D canvas, SVG export scene | MIT | (push 2026-10-20) | n/a |

Sources for each row are given in the sections above.

### Gaps

- PowerPoint for the web and Canva editors: rendering technology not determined (cross-origin frame; Cloudflare challenge). Marked inline.
- Pitch: only the public player was inspected, not the logged-in editor.
- Syntax 505 transcript, W3C 2010 thread, Bushell post and techinterview.org Canva guide were used through search summaries only, as tagged inline.
