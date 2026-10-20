# pptx export and chart libraries and their licenses (open question 5)

Scope: open question 5 of `docs/spec-v0.md` ("Which libraries for pptx export and for charts fit, and what are their licenses?"), read against the spec's choices: FastAPI backend, TypeScript/React frontend, "pptx: Port of the exporter in the `presentations` plugin, from JSON instead of from HTML", Chart block "bar, line, pie; data in the block", pptx export where "typed blocks become native shapes and text boxes; masters become slide layouts; HTML, SVG and script blocks become images" [source: /home/pascal/Code/.worktrees/decks/research/docs/spec-v0.md | decks v0 spec | practitioner].

Snapshot date for all repository and registry figures: 2026-10-20 (pulled that day via `gh api` and the npm/PyPI APIs).

## Baseline: what the current exporter does

The current `exporting-presentations-to-pptx` skill drives python-pptx: "uv pip install --python /tmp/pptx-venv/bin/python python-pptx pillow pymupdf" [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/exporting-presentations-to-pptx/SKILL.md | Exporting Presentations to PPTX | practitioner].

It states "python-pptx cannot place SVGs, crop pictures to a circle, or set picture transparency. Pre-render PNGs first", and rasterizes SVGs with PyMuPDF "at the target display width times four" [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/exporting-presentations-to-pptx/SKILL.md | Exporting Presentations to PPTX | practitioner].

It already uses the same unit mapping as the spec: "1 stage px = 6350 EMU (12192000 / 1920)" and "1 stage px = 0.5 pt for font sizes" [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/exporting-presentations-to-pptx/SKILL.md | Exporting Presentations to PPTX | practitioner].

Charts: "build a native chart with `slide.shapes.add_chart(XL_CHART_TYPE.COLUMN_CLUSTERED, x, y, cx, cy, CategoryChartData())` so the client can edit the numbers"; "Waterfall: a stacked column chart with an invisible base series ... python-pptx has no native waterfall type"; "Picture fallback: when the data cannot be read back ... rasterize the SVG at 4x ... Tell the user which charts ship as pictures, since they are not editable" [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/exporting-presentations-to-pptx/references/python-pptx-recipes.md | python-pptx recipes for HTML deck exports | practitioner].

It records thirteen gotchas, among them: strip `<p:style>` from autoshapes (unwanted shadow), line spacing as exact points not `spcPct`, "LibreOffice ignores `wrap="none"`", native bullets via `marL`/`indent`/`buChar`, "Autofit off, size never reduced", and fonts: "Prefer widely installed faces over the deck's webfonts: Aptos for sans (M365 default), Consolas for mono" [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/exporting-presentations-to-pptx/references/python-pptx-recipes.md | python-pptx recipes for HTML deck exports | practitioner].

Verification renders the pptx through LibreOffice in a container (`soffice --headless --convert-to pdf`) and notes "the verification renderer substitutes fonts, so glyph differences are expected and harmless" [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/exporting-presentations-to-pptx/SKILL.md | Exporting Presentations to PPTX | practitioner].

## SQ1: pptx writing libraries for a TypeScript/Python stack

### PptxGenJS (JavaScript/TypeScript, browser and Node)

License file, first lines: "The MIT License (MIT) / Copyright (c) 2015-2022 Brent Ely / Permission is hereby granted, free of charge, to any person obtaining a copy of this software ..." [source: https://github.com/gitbrent/PptxGenJS/blob/master/LICENSE | PptxGenJS LICENSE | vendor]. No non-standard clauses (21-line standard MIT text).

Maintenance: latest release v4.0.1 on 2025-06-26, v4.0.0 on 2025-05-04 [source: https://api.github.com/repos/gitbrent/PptxGenJS/releases | GitHub API releases | vendor]; 6,294 stars, 305 open issues, last push 2025-11-28 [source: https://api.github.com/repos/gitbrent/PptxGenJS | GitHub API repo | vendor]; last merged PR 2025-08-09 (#1412), 30 open PRs [source: https://github.com/gitbrent/PptxGenJS/pulls?q=is%3Amerged | PptxGenJS merged PRs | vendor].

Usage: 14,165,453 npm downloads 2026-09-19 to 2026-10-18 [source: https://api.npmjs.org/downloads/point/last-month/pptxgenjs | npm downloads API | vendor].

Maintenance concern raised by users: issue "Is this library still maintained?" (2024-03-29, closed); a commenter on 2024-12-19: "Afraid that this library is no longer maintained, I will try this: https://www.npmjs.com/package/pptx-automizer" [source: https://github.com/gitbrent/PptxGenJS/issues/1324 | Is this library still maintained? | practitioner].

Masters and placeholders: masters are made with `defineSlideMaster()`; "The defined Masters become first-class Layouts in the exported PowerPoint presentation" and "can be changed via View > Slide Master and will affect the Slides created using that layout". Placeholder types: `title`, `body`, `image`, `chart`, `table`, `media`; content targets a placeholder by name, e.g. `slide.addText(..., { placeholder: "body" })` [source: https://gitbrent.github.io/PptxGenJS/docs/masters/ | PptxGenJS Slide Masters | vendor].

Source detail: every `defineSlideMaster()` becomes a `ppt/slideLayouts/slideLayoutN.xml` under one single `ppt/slideMasters/slideMaster1.xml` (`zip.file('ppt/slideMasters/slideMaster1.xml', genXml.makeXmlMaster(this.masterSlide, this.slideLayouts))`) [source: https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/pptxgen.ts#L526-L537 | pptxgen.ts | vendor]. So a PptxGenJS "master" maps to an OOXML slide layout, not to an OOXML slide master.

Theme: `ThemeProps` has only `headFontFace` and `bodyFontFace` (defaults "Calibri Light" and "Calibri") [source: https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/core-interfaces.ts#L462-L474 | core-interfaces.ts | vendor]. Anthropic's pptx skill states the consequence: "pptxgenjs cannot write theme colors: until this runs, scheme colors resolve to Office's stock palette", and ships its own `apply_theme.js` post-processor [source: /home/pascal/.claude/skills/synced/fd3eaa6c-fd9b-44e1-9ac4-d446976eb04d_4247339b-e14f-4e59-9090-d60035112cef/pptx/SKILL.md | Anthropic pptx skill | practitioner].

Same skill on placeholder filling: "Only `title`, `body` and `chart` placeholders fill by name alone: `addImage({ placeholder })` takes the placeholder's x and y but not its size (a 1" × 1" picture)"; "A misspelt `placeholder` or `masterName` fails silently (a stray text box at 0,0; the blank layout)" [source: /home/pascal/.claude/skills/synced/fd3eaa6c-fd9b-44e1-9ac4-d446976eb04d_4247339b-e14f-4e59-9090-d60035112cef/pptx/SKILL.md | Anthropic pptx skill | practitioner].

Known defects with masters (user report, March 2026, PptxGenJS 3.x): "PptxGenJS registers one `<Override>` entry in `[Content_Types].xml` per slide rather than per actual slide master. A 55-slide deck with 3 defined masters produces 55 `slideMaster` override entries", which "trigger PowerPoint's repair process" [source: https://github.com/gitbrent/PptxGenJS/issues/1449 | Multiple errors in PowerPoint that were not in LibreOffice | practitioner]. The loop is still in current master source: `slides.forEach((slide, idx) => { strXml += \`<Override PartName="/ppt/slideMasters/slideMaster${idx + 1}.xml" ...` [source: https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/gen-xml.ts#L1405-L1412 | gen-xml.ts | vendor]. The same report: PowerPoint (macOS, Microsoft 365) showed "needs to be repaired" on every open while "LibreOffice rendered the same files without error" [source: https://github.com/gitbrent/PptxGenJS/issues/1449 | Multiple errors in PowerPoint that were not in LibreOffice | practitioner].

Other open repair-dialog bugs filed in 2026: #1441 shapes without text lacking `<p:txBody>`, #1442 background missing `<a:effectLst/>`, #1443 notesMaster placeholders removed by repair, #1444 phantom slideMaster entries, #1532 duplicate `p:cNvPr/@id` with a table, #1534 "addChart() writes an orphan `<c:axId>` for BAR/LINE charts" [source: https://github.com/gitbrent/PptxGenJS/issues?q=is%3Aopen+repair | PptxGenJS open issues "repair" | practitioner].

Charts: "Charts of almost any type can be added to Slides, including combo and 3D charts"; types area, bar, bar3D, bubble, bubble3D, doughnut, line, pie, radar, scatter; series data as `{ name, labels: [...], values: [...] }`; options `chartColors`, `showLabel`/`showValue`/`showPercent`, `dataLabelFormatCode`, axis min/max, gridlines, `valAxisLogScaleBase` [source: https://gitbrent.github.io/PptxGenJS/docs/api-charts/ | PptxGenJS Charts | vendor]. Enum confirms the ten types [source: https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/core-enums.ts#L697-L708 | core-enums.ts | vendor]. Charts carry an embedded workbook: `ppt/embeddings/Microsoft_Excel_Worksheet${chartObject.globalId}.xlsx` [source: https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/gen-charts.ts#L519-L527 | gen-charts.ts | vendor]. No waterfall in the enum.

Chart pitfalls (Anthropic pptx skill): "On a stacked bar or column chart, `dataLabelPosition` must be `ctr`, `inEnd`, or `inBase`. `outEnd` corrupts the file"; a combo with secondary axes "needs both `valAxes` and `catAxes` ... Without them pptxgenjs writes axis ids it never declares, and PowerPoint discards that chart"; chart text "is hard-coded Arial: give its `*FontFace` options `"+mn-lt"`" [source: /home/pascal/.claude/skills/synced/fd3eaa6c-fd9b-44e1-9ac4-d446976eb04d_4247339b-e14f-4e59-9090-d60035112cef/pptx/SKILL.md | Anthropic pptx skill | practitioner].

Tables, text, notes: `addTable(tableRows, options)`, `addText(text | TextProps[], options)` (an array of runs, each with its own options), `addNotes(notes)` [source: https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/slide.ts#L202-L241 | slide.ts | vendor]; table cells take `colspan`/`rowspan`, tables take `autoPage` options; text takes `bullet` and `indentLevel` [source: https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/core-interfaces.ts#L748-L844 | core-interfaces.ts | vendor].

SVG: an SVG image is written as a PNG `a:blip` with an `asvg:svgBlip` extension (`uri="{96DAC541-7B7A-43D3-8B79-37D633B846F1}"`, namespace `http://schemas.microsoft.com/office/drawing/2016/SVG/main`) [source: https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/gen-xml.ts#L576-L590 | gen-xml.ts | vendor]. The PNG fallback is drawn through a browser `<canvas>`; in Node it is replaced by a broken-image placeholder: `// console.log('Sorry, SVG is not supported in Node (more info: https://github.com/gitbrent/PptxGenJS/issues/401)') rel.data = IMG_BROKEN` [source: https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/gen-media.ts#L136-L152 | gen-media.ts | vendor].

Font embedding: no font embedding API in the source (grep for `embed`/`fntdata` finds only media and chart workbook embedding) [source: https://github.com/gitbrent/PptxGenJS/tree/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src | PptxGenJS src | vendor]. A separate MIT library, pptx-embed-fonts, adds it: "Support for TTF, EOT, OTF and WOFF format font file embedding" with "pptxgenjs Integration"; 9 stars, last push 2026-01-08 [source: https://github.com/liyao1520/pptx-embed-fonts | pptx-embed-fonts | practitioner]. Its LICENSE: "MIT License / Copyright (c) 2025 liyao1520" [source: https://github.com/liyao1520/pptx-embed-fonts/blob/main/LICENSE | pptx-embed-fonts LICENSE | practitioner].

### python-pptx (Python)

License file, first lines: "The MIT License (MIT) / Copyright (c) 2013 Steve Canny, https://github.com/scanny / Permission is hereby granted, free of charge ..." [source: https://github.com/scanny/python-pptx/blob/master/LICENSE | python-pptx LICENSE | vendor]. Standard MIT, no extra clauses.

Maintenance: last three releases 1.0.0 (2024-08-03), 1.0.1 (2024-08-05), 1.0.2 (2024-08-07); none since [source: https://pypi.org/pypi/python-pptx/json | PyPI JSON | vendor]. Last commit 2024-08-07 ("fix(enum): replace read-only enum values"); 3,558 stars; 541 open issues and PRs [source: https://api.github.com/repos/scanny/python-pptx | GitHub API | vendor]. Usage: 60,210,371 downloads in the last month [source: https://pypistats.org/api/packages/python-pptx/recent | pypistats | vendor]. A FreeBSD ports update in September 2025 still moved to 1.0.2 [source: https://reviews-dev.freebsd.org/R11:93bff23f327fa64ff6ed14f7824a9b643f85da10 | textproc/py-python-pptx: Update 0.6.23 → 1.0.2 | practitioner] (seen via search snippet only).

Masters and layouts: "there is currently no distinction between templates and presentations in python-pptx ... To use a "template" for a presentation you simply create a presentation with all the styles, logo, and layouts you want, delete all the slides ... and then load that as your starting place" [source: https://python-pptx.readthedocs.io/en/latest/user/concepts.html | python-pptx Concepts | vendor]. `SlideLayouts` supports indexed access, iteration and `remove()`, no add method [source: https://github.com/scanny/python-pptx/blob/278b47b1dedd5b46ee84c286e77cdfb0bf4594be/src/pptx/slide.py#L347-L406 | slide.py | vendor]. Inheritance: "A layout placeholder inherits from the master placeholder sharing the same type"; "A slide placeholder inherits from the layout placeholder having the same idx value" [source: https://python-pptx.readthedocs.io/en/latest/user/placeholders-understanding.html | Understanding placeholders | vendor].

Charts: "Most chart types other than 3D types are supported"; data through `CategoryChartData`, `XyChartData`, `BubbleChartData`; python-pptx "doesn't yet support creating multi-plot charts" [source: https://python-pptx.readthedocs.io/en/latest/user/charts.html | python-pptx Charts | vendor]. `XL_CHART_TYPE` has 116 members [source: https://github.com/scanny/python-pptx/blob/278b47b1dedd5b46ee84c286e77cdfb0bf4594be/src/pptx/enum/chart.py | enum/chart.py | vendor]. The embedded workbook is written with XlsxWriter (`from xlsxwriter import Workbook`; dependency `XlsxWriter>=0.5.7`) [source: https://github.com/scanny/python-pptx/blob/278b47b1dedd5b46ee84c286e77cdfb0bf4594be/src/pptx/chart/xlsx.py#L8 | chart/xlsx.py | vendor].

Tables, notes, text: `add_table`, `add_chart`, `add_picture` on the shape tree; `notes_slide` on a slide [source: https://github.com/scanny/python-pptx/blob/278b47b1dedd5b46ee84c286e77cdfb0bf4594be/src/pptx/shapes/shapetree.py#L236-L589 | shapetree.py | vendor]. Rich runs are supported, but "preserve formatting through `text_frame.text = "..."` (that collapses the paragraph to a single unstyled run — assign `run.text` instead)" [source: /home/pascal/.claude/skills/synced/fd3eaa6c-fd9b-44e1-9ac4-d446976eb04d_4247339b-e14f-4e59-9090-d60035112cef/pptx/SKILL.md | Anthropic pptx skill | practitioner].

SVG: package code says "---skip unknown/unsupported image types, like SVG---" [source: https://github.com/scanny/python-pptx/blob/278b47b1dedd5b46ee84c286e77cdfb0bf4594be/src/pptx/package.py#L164 | package.py | vendor]. Open issue "SlideShapes.add_picture() does not support SVG files" (#1112, 2026-03-15) and open PR "feat: add SVG image support" (#1115, 2026-03-21) [source: https://github.com/scanny/python-pptx/issues/1112 | issue 1112 | practitioner] [source: https://github.com/scanny/python-pptx/pull/1115 | PR 1115 | practitioner]. Anthropic's skill: python-pptx won't "read the SVG/EMF most template art uses (`add_picture` raises `UnidentifiedImageError`)" [source: /home/pascal/.claude/skills/synced/fd3eaa6c-fd9b-44e1-9ac4-d446976eb04d_4247339b-e14f-4e59-9090-d60035112cef/pptx/SKILL.md | Anthropic pptx skill | practitioner].

Fonts: python-pptx registers the `fntdata` content type (`("fntdata", CT.X_FONTDATA)`) but has no embedding API [source: https://github.com/scanny/python-pptx/blob/278b47b1dedd5b46ee84c286e77cdfb0bf4594be/src/pptx/opc/spec.py#L11 | opc/spec.py | vendor].

### power-pptx (python-pptx fork)

License file: "The MIT License (MIT) / Copyright (c) 2013 Steve Canny ... / Copyright (c) 2026 Daniel Halwell" [source: https://github.com/codehalwell/power-pptx/blob/master/LICENSE | power-pptx LICENSE | practitioner].

Self-description: "the actively-maintained fork of the excellent python-pptx library ... picking up where the upstream's 1.0.2 release left off"; imports as `power_pptx` [source: https://github.com/codehalwell/power-pptx | power-pptx README | vendor]. Claimed features: "Animations and transitions — preset entrance / exit / emphasis effects; motion-path presets; per-paragraph reveal ... transitions including Morph"; "Theme reader and writer ... write fresh `<a:srgbClr>` values into the clrScheme; apply a theme imported from a `.potx`"; "native SVG embedding with PNG fallback"; "JSON authoring — `power_pptx.compose.from_spec(...)`" [source: https://github.com/codehalwell/power-pptx | power-pptx README | vendor]. Not verified against its code.

Maintenance signals: repo created 2026-04-25; 13 stars; releases 2.12.0 (2026-08-24), 2.13.0 (2026-10-17), 2.14.0 (2026-10-19); 2,527 downloads last month [source: https://api.github.com/repos/codehalwell/power-pptx | GitHub API | vendor] [source: https://pypi.org/pypi/power-pptx/json | PyPI JSON | vendor] [source: https://pypistats.org/api/packages/power-pptx/recent | pypistats | vendor]. Contributor commit counts: scanny 463, claude 179, CodeHalwell 39, karthik1024 11 [source: https://api.github.com/repos/codehalwell/power-pptx/contributors | GitHub API contributors | vendor]. Another fork, python-pptx-ng, last released 0.7.0 on 2024-01-17 [source: https://pypi.org/pypi/python-pptx-ng/json | PyPI JSON | vendor].

### pptx-automizer (Node, template-based)

License file: "MIT License / Copyright (c) 2021 Thomas Singer" [source: https://github.com/singerla/pptx-automizer/blob/main/LICENSE | pptx-automizer LICENSE | vendor].

Maintenance: v0.9.4 on npm 2026-09-28; last push 2026-10-14; 251 stars; 1 open issue; 407,136 npm downloads in the last month [source: https://registry.npmjs.org/pptx-automizer | npm registry | vendor] [source: https://api.github.com/repos/singerla/pptx-automizer | GitHub API | vendor] [source: https://api.npmjs.org/downloads/point/last-month/pptx-automizer | npm downloads API | vendor].

Model: "a Node.js-based PowerPoint (.pptx) generator that automates the manipulation of existing .pptx files"; any "existing slide or even a single element can serve as a data-driven template"; for new elements "pptx-automizer wraps around PptxGenJS" [source: https://github.com/singerla/pptx-automizer | pptx-automizer README | vendor]. Depends on `"pptxgenjs": "^3.12.0"` [source: https://github.com/singerla/pptx-automizer/blob/main/package.json | package.json | vendor].

Limits: "This generator can only be used on the server-side and requires a Node.js environment"; "Animations are currently out of scope ... You might get errors on opening an output .pptx when there are added or removed shapes"; "Slide masters and their layouts can be imported, but individual slide layouts cannot be added, modified or removed directly, and layouts must not carry complex content like charts and images"; "All testing focuses on PowerPoint 2019 .pptx file format" [source: https://github.com/singerla/pptx-automizer/blob/main/docs/limitations.md | Requirements and Limitations | vendor]. The README does not mention SVG or speaker notes [source: https://github.com/singerla/pptx-automizer | pptx-automizer README | vendor]; its docs list text, tables, charts, images and masters/layouts pages (`docs/text.md`, `docs/tables.md`, `docs/charts.md`, `docs/images.md`, `docs/masters-layouts.md`), which were not read in depth [source: https://github.com/singerla/pptx-automizer/tree/main/docs | pptx-automizer docs | vendor]. "Individual layouts cannot be imported on their own — a layout always comes with its master" [source: https://github.com/singerla/pptx-automizer/blob/main/docs/masters-layouts.md | Slide Masters and Layouts | vendor].

### officegen (Node)

License file: "MIT License / Copyright (c) 2013 Ziv Barber" [source: https://github.com/Ziv-Barber/officegen/blob/master/LICENSE | officegen LICENSE | vendor]. Last release v0.6.5 on 2021-03-06; last push 2024-04-30; 199 open issues; 79,813 npm downloads in the last month [source: https://api.github.com/repos/Ziv-Barber/officegen | GitHub API | vendor] [source: https://api.npmjs.org/downloads/point/last-month/officegen | npm downloads API | vendor]. No feature deep dive done: last release five years old.

### Open XML SDK (.NET)

License file: "The MIT License (MIT) / Copyright (c) .NET Foundation and Contributors" [source: https://github.com/dotnet/Open-XML-SDK/blob/main/LICENSE | Open XML SDK LICENSE | vendor]. Listed for completeness; it is .NET, outside the TypeScript/Python stack.

### Aspose.Slides (paid comparison)

Prices: Developer Small Business "1 Developer And 1 Deployment Location" $1,399; Developer OEM $4,197; Site Small Business (up to 10 developers, 10 locations) $6,995; Metered from $1,999/month [source: https://purchase.aspose.com/pricing/slides/family/ | Aspose.Slides pricing | vendor].

Layouts: "To create a layout, call the MasterLayoutSlideCollection.add method on the target master's layout collection"; placeholders via `LayoutSlide.placeholder_manager` (`add_content_placeholder(x, y, width, height)`); "A newly added layout placeholder is not backfilled into existing normal slides" [source: https://docs.aspose.com/slides/python-net/slide-layout/ | Apply or Change Slide Layouts in Python | vendor].

Fonts: `fonts_manager.add_embedded_font` with `EmbedFontCharacters.ALL` or `ONLY_USED`; "The fonts to add must be available on the machine running the code"; `RESTRICTED` fonts "generally cannot be embedded without the legal owner's permission" [source: https://docs.aspose.com/slides/python-net/embedded-font/ | Embed Fonts in Presentations with Python | vendor].

### Syncfusion Presentation (paid comparison)

Community License: "Companies and individuals with less than $1 million USD in annual gross revenue, 5 or fewer developers" and "10 or fewer total employees"; "An entity or organization may not have ever received more than $3,000,000 USD in capital from an outside source"; "Any government-related organization ... is not eligible"; client work requires the client's own license [source: https://www.syncfusion.com/products/communitylicense | Syncfusion Community License | vendor]. The NuGet package states it is a commercial product requiring a paid license [source: https://www.NuGet.org/packages/Syncfusion.Presentation.AspNet | Syncfusion.Presentation.AspNet | vendor] (search snippet only). Syncfusion's library is .NET [source: https://www.syncfusion.com/powerpoint-framework/net-core | Syncfusion .NET Core PowerPoint library | vendor] (search snippet only). No feature deep dive done.

### Anthropic pptx skill (prior art, not a library)

Its creation path: "Write a `pptxgenjs` script"; "Keep charts native. Use `addChart()` for everything PowerPoint can chart ... Only chart types PowerPoint has no native form for (Sankey, network, chord) go in as images"; "One layout per slide frame (`defineSlideMaster()`)"; "Footer text, logo and `slideNumber` go on the layout, never on a slide" [source: /home/pascal/.claude/skills/synced/fd3eaa6c-fd9b-44e1-9ac4-d446976eb04d_4247339b-e14f-4e59-9090-d60035112cef/pptx/SKILL.md | Anthropic pptx skill | practitioner].

License: "© 2025 Anthropic, PBC. All rights reserved. LICENSE: Use of these materials ... is governed by your agreement with Anthropic regarding use of Anthropic's services" [source: https://github.com/anthropics/skills/blob/main/skills/pptx/LICENSE.txt | Anthropic skills pptx LICENSE | vendor]. Proprietary; not an open source license.

## SQ2: chart libraries for a block rendered in the browser, printed by Chromium, exported to pptx

### Apache ECharts

License file: "Apache License / Version 2.0, January 2004"; appendix: "Apache ECharts Subcomponents: ... BSD 3-Clause (d3.js): The following files embed d3.js BSD 3-Clause: `/src/chart/treemap/treemapLayout.ts`, `/src/chart/tree/layoutHelper.ts`, `/src/chart/graph/forceHelper.ts`, `/src/util/number.ts`" [source: https://github.com/apache/echarts/blob/master/LICENSE | ECharts LICENSE | vendor]. NOTICE file: "Apache ECharts / Copyright 2017-2026 The Apache Software Foundation / This product includes software developed at The Apache Software Foundation" [source: https://github.com/apache/echarts/blob/master/NOTICE | ECharts NOTICE | vendor]. Apache-2.0 section 4(d) requires redistributions to include the NOTICE text [source: https://github.com/apache/echarts/blob/master/LICENSE | ECharts LICENSE | vendor].

Runtime dependency zrender, license file: "BSD 3-Clause License / Copyright (c) 2017, Baidu Inc." [source: https://github.com/ecomfe/zrender/blob/master/LICENSE | zrender LICENSE | vendor].

Maintenance: 6.1.0 released 2026-05-19; last push 2026-10-13; 67,552 stars; 23,453,063 npm downloads last month [source: https://api.github.com/repos/apache/echarts/releases | GitHub API | vendor] [source: https://api.npmjs.org/downloads/point/last-month/echarts | npm downloads API | vendor].

SVG and server rendering: ECharts "introduced a new zero-dependency server-side string based SVG rendering solution" in 5.3.0; create the chart with `ssr: true` and `renderer: 'svg'`, then "we use `renderToSVGString` to render the current chart to an SVG string"; container may be `null`; `ssr: true` "disable[s] the animation loop and event modules"; initial animation is "achieved by embedding CSS animations in the output SVG string"; server output cannot do tooltips, legend toggling or dynamic data [source: https://echarts.apache.org/handbook/en/how-to/cross-platform/server/ | Server Side Rendering | vendor].

Renderer trade-offs: "For larger amounts of data (>1k is an experience value), canvas renderer is always recommended"; SVG "has less memory usage" and "won't be blurry when zooming in"; since v5.3.0 the SVG renderer uses a virtual DOM and "the performance got improved by 2-10 times"; "some special effects still relies on Canvas", e.g. the line trail effect and the blended heatmap [source: https://echarts.apache.org/handbook/en/best-practices/canvas-vs-svg/ | Render with SVG or Canvas | vendor].

Counter-evidence (open SVG-renderer bugs): #21465 "RTL text issues and inconsistencies between Canvas and SVG renderers" (2026-01-12); #21744 legend hover "unresponsive on line series above ~2,000-3,000 points (SVG renderer)" (2026-09-02); #20261 "SVG Renderer in ECharts Fails to Display Base64 Encoded Data URI Image"; #20390 SVG glitches in a CSS container in Safari [source: https://github.com/apache/echarts/issues?q=is%3Aopen+%22svg+renderer%22 | ECharts open issues "svg renderer" | practitioner]. No issue found specific to `renderToSVGString` output.

Data model: `dataset.source` accepts "a 2D array, array of classes, etc."; the first row can hold dimension names; `seriesLayoutBy` maps series to columns (default) or rows; `encode` maps dimensions to axes [source: https://echarts.apache.org/handbook/en/concepts/dataset/ | Dataset | vendor].

### Vega-Lite (and Vega)

Vega-Lite license file: "Copyright (c) 2015, University of Washington Interactive Data Lab. All rights reserved. Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met"; clause 3: "Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission" [source: https://github.com/vega/vega-lite/blob/main/LICENSE | Vega-Lite LICENSE | vendor]. Vega: same BSD-3 text, "Copyright (c) 2015-2023, University of Washington Interactive Data Lab" [source: https://github.com/vega/vega/blob/main/LICENSE | Vega LICENSE | vendor].

Maintenance: Vega-Lite v6.5.0 on 2026-10-08; 4,584,933 npm downloads last month [source: https://api.github.com/repos/vega/vega-lite/releases | GitHub API | vendor] [source: https://api.npmjs.org/downloads/point/last-month/vega-lite | npm downloads API | vendor].

SVG: `view.toSVG([scaleFactor])` "Returns a Promise that resolves to an SVG string, providing a vector graphics image of the view" [source: https://github.com/vega/vega/blob/main/docs/docs/api/view.md | Vega View API | vendor]. CLI scripts `vl2vg`, `vl2png`, `vl2svg`, `vl2pdf`; canvas errors need "the `canvas` package (or install vega-cli)" [source: https://vega.github.io/vega-lite/usage/compile.html | Compiling Vega-Lite | vendor].

Python-side rendering: vl-convert converts Vega-Lite to SVG/PNG without Node (`vegalite_to_svg`, `vegalite_to_png`) [source: https://pypi.org/project/vl-convert-python/0.13.1 | vl-convert-python | vendor]; license file "Copyright 2023 Jon Mease / Redistribution and use in source and binary forms ..." (BSD-3); latest tag v2.0.0-rc7 on 2026-09-14 [source: https://github.com/vega/vl-convert/blob/main/LICENSE | vl-convert LICENSE | vendor] [source: https://api.github.com/repos/vega/vl-convert/releases | GitHub API | vendor].

### Chart.js

License file: "The MIT License (MIT) / Copyright (c) 2014-2024 Chart.js Contributors" [source: https://github.com/chartjs/Chart.js/blob/master/LICENSE.md | Chart.js LICENSE.md | vendor].

Maintenance: v4.5.1 on 2025-10-13; 51,937,298 npm downloads last month [source: https://api.github.com/repos/chartjs/Chart.js/releases | GitHub API | vendor] [source: https://api.npmjs.org/downloads/point/last-month/chart.js | npm downloads API | vendor].

Rendering: "Chart.js renders chart elements on an HTML5 canvas unlike several others, mostly D3.js-based, charting libraries that render as SVG ... canvas rendering disallows CSS styling" [source: https://github.com/chartjs/Chart.js/blob/master/docs/index.md | Chart.js docs index | vendor]. Server use needs "node-canvas or skia-canvas", sample output is `canvas.toBuffer('png', ...)` [source: https://github.com/chartjs/Chart.js/blob/master/docs/getting-started/using-from-node-js.md | Using from Node.js | vendor]. For print: "For applications where a chart will be converted to a bitmap, or printed to a higher DPI medium, it can be desirable to render the chart at a higher resolution than the default" via `devicePixelRatio` [source: https://github.com/chartjs/Chart.js/blob/master/docs/configuration/device-pixel-ratio.md | Device Pixel Ratio | vendor].

### Observable Plot

License file (ISC): "Copyright 2020-2025 Observable, Inc. / Permission to use, copy, modify, and/or distribute this software for any purpose with or without fee is hereby granted, provided that the above copyright notice and this permission notice appear in all copies" [source: https://github.com/observablehq/plot/blob/main/LICENSE | Observable Plot LICENSE | vendor].

Maintenance: last release v0.6.17 on 2025-02-14; last push 2026-09-01; 3,303,739 npm downloads last month [source: https://api.github.com/repos/observablehq/plot/releases | GitHub API | vendor] [source: https://api.npmjs.org/downloads/point/last-month/@observablehq/plot | npm downloads API | vendor].

SSR: use "the **document** plot option to tell Plot to render with React's virtual DOM" (`Plot.plot({...options, document: new Document()}).toHyperScript()`); "Server-side rendering is only practical for simple plots of small data; complex plots ... are better rendered on the client because the serialized SVG is large" [source: https://github.com/observablehq/plot/blob/main/docs/getting-started.md | Plot Getting started | vendor].

### Recharts

License file: "The MIT License (MIT) / Copyright (c) 2015-present recharts" [source: https://github.com/recharts/recharts/blob/main/LICENSE | Recharts LICENSE | vendor].

Maintenance: latest stable 3.10.1 (2026-07-25), canary 3.11.0-canary.6 (2026-10-13); 239,787,943 npm downloads last month [source: https://registry.npmjs.org/recharts | npm registry | vendor] [source: https://api.npmjs.org/downloads/point/last-month/recharts | npm downloads API | vendor].

SSR: open issue "Unable to render chart on the server using 3.0.0" with `renderToStaticMarkup` [source: https://github.com/recharts/recharts/issues/5997 | issue 5997 | practitioner]. Maintainer reply on the duplicate: "recharts is a client side library and uses APIs that require a browser. This was the case in 2.x as well, server side rendering has never been explicitly supported"; the duplicate reported "Charts produce minimal HTML (~100-150 characters) with no SVG content" in 3.x [source: https://github.com/recharts/recharts/issues/6139 | SSR Regression: Charts render empty in Recharts 3.x | practitioner]. A maintainer also wrote "there's about ~35 references to `useEffect` in recharts code at this point which were all added to synchronize state across components" [source: https://github.com/recharts/recharts/issues/5997 | Unable to render chart on the server using 3.0.0 | practitioner].

### D3

License file (ISC): "Copyright 2010-2023 Mike Bostock / Permission to use, copy, modify, and/or distribute this software for any purpose with or without fee is hereby granted ..." [source: https://github.com/d3/d3/blob/main/LICENSE | D3 LICENSE | vendor]. D3 is the low-level layer under Plot; no chart spec of its own. Note: ECharts' LICENSE describes the d3 code it embeds as "BSD 3-Clause" [source: https://github.com/apache/echarts/blob/master/LICENSE | ECharts LICENSE | vendor], while the current d3 repo LICENSE is ISC [source: https://github.com/d3/d3/blob/main/LICENSE | D3 LICENSE | vendor]; d3 changed license over time (no source read for the change date).

### Spec-to-native-pptx-chart mapping

No maintained library found that turns an ECharts option or a Vega-Lite spec into a native PowerPoint chart; queries tried: "convert ECharts or Vega-Lite chart spec to native editable PowerPoint chart pptxgenjs" (results were vl-convert image renderers, flint-chart targeting Excel through Office.js, and the tutorial below) [author estimate: absence of evidence from one search round plus registry checks, not proof]. A tutorial on Oracle APEX charts maps a chart's series names, labels and values into PptxGenJS's `{name, labels, values}` shape by hand: "PptxGenJS expect data with this definition so we need to retrieve APEX chart data with the same structure" [source: https://codix.ai/export-apex-charts-as-powerpoint-charts/ | Export APEX charts as PowerPoint Charts | practitioner] (search snippet only).

The two pptx libraries take category data in near-identical shapes: PptxGenJS `{ name, labels, values }` per series [source: https://gitbrent.github.io/PptxGenJS/docs/api-charts/ | PptxGenJS Charts | vendor]; python-pptx `CategoryChartData` with `categories` and `add_series(name, values)` [source: https://python-pptx.readthedocs.io/en/latest/user/charts.html | python-pptx Charts | vendor]. ECharts `dataset.source` as a 2D array with a header row maps columns to series by default [source: https://echarts.apache.org/handbook/en/concepts/dataset/ | Dataset | vendor].

Google Slides takes charts only from Sheets: "must first create the chart in Google Sheets"; `LinkingMode` `LINKED` or `NOT_LINKED_IMAGE` [source: https://developers.google.com/workspace/slides/api/guides/add-chart | Add charts to your slides | vendor].

think-cell (the consulting standard add-in): "The thinkcell elements appear as regular PowerPoint shapes such as textboxes and PowerPoint chart objects"; files are "100% compatible with PowerPoint" for people without the add-in [source: https://www.thinkcell.com/en/resources/manual/compatibility | think-cell manual: Exchanging files with PowerPoint | vendor].

## SQ3: how others export HTML or JSON slides to pptx, and the fidelity reached

### Slidev

Standard export: "all the slides in the PPTX file will be exported as images, so the text will not be selectable"; "Presenter notes will be conveyed into the PPTX file on a per-slide basis" [source: https://sli.dev/guide/exporting | Slidev Exporting | vendor].

New `--format pptx-editable`: slides rebuilt as native shapes, "text is selectable and editable"; SVG (including Mermaid and icons), canvas, iframe, video, KaTeX and CSS gradients stay pictures; slides that cannot be rebuilt fall back to images; fonts referenced, not embedded; text may wrap differently [source: https://sli.dev/guide/exporting | Slidev Exporting | vendor].

Implementation: PR #2722 "feat(export): editable PPTX export with native shapes and text", merged 2026-09-16, first in tag v52.20.0 [source: https://github.com/slidevjs/slidev/pull/2722 | Slidev PR 2722 | vendor]. It "walks the rendered DOM at export time and emits native shapes and text boxes rather than one picture per slide"; on `demo/starter`, `slide1.xml` grows from 770 to 5,134 bytes, 0 to 640 shapes, 0 to 1,554 text runs; "No template or `.potx` loading, no slide master synthesis, no font embedding"; "No gradients: `pptxgenjs` has none, so they rasterize"; "A long paragraph can wrap onto a different number of lines, because PowerPoint's text metrics are not Chromium's"; "`--format pptx` is untouched. The two ship side by side because `pptx` is pixel-exact on every theme and this cannot be" [source: https://github.com/slidevjs/slidev/pull/2722 | Slidev PR 2722 | vendor]. Slidev depends on `pptxgenjs` in both `packages/slidev` and `packages/client` [source: https://github.com/slidevjs/slidev/blob/main/packages/slidev/package.json | Slidev package.json | vendor].

Slidev license file: "MIT License / Copyright (c) 2020-PRESENT Anthony Fu" [source: https://github.com/slidevjs/slidev/blob/main/LICENSE | Slidev LICENSE | vendor].

### Marp

Marp CLI: "A created PPTX includes rendered Marp slide pages and the support of Marpit presenter notes"; "The experimental `--pptx-editable` option requires installing both of the browser and LibreOffice Impress"; it "may throw an error or output the incomplete result"; it "results in lower slide reproducibility compared to the conversion into regular PPTX. Additionally, presenter notes are not supported"; "We do not recommend to export the editable PPTX if maintaining the slide's appearance is important" [source: https://github.com/marp-team/marp-cli | Marp CLI README | vendor].

Marp CLI license file: "MIT License / Copyright (c) 2018 Marp team (marp-team@marp.app)" [source: https://github.com/marp-team/marp-cli/blob/main/LICENSE | Marp CLI LICENSE | vendor]. Marp Core: same text [source: https://github.com/marp-team/marp-core/blob/main/LICENSE | Marp Core LICENSE | vendor].

### Gamma

Help center: "Charts and diagrams render at their proper size in PDF, PPTX, and Google Slides exports"; "Exports reflect Present Mode, not Edit Mode, so some visual differences are expected"; "Google Slides ignores fonts embedded in an uploaded PowerPoint file" [source: https://help.gamma.app/en/articles/15939201-why-doesnt-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma | Why doesn't my exported PDF or PowerPoint match | vendor].

Feature request "Native powerpoint chart exports (don't exported as images)": "it's really frustrating that the charts (such as pie charts, bar charts, and some others) ... they're automatically exported as images"; Gamma staff: "Native, editable charts in PPT would be a big usability win. I'll pass this along to the team" and "that's a pretty painful workflow in consulting when you need to tweak charts last minute in PowerPoint" [source: https://ideas.gamma.app/ideas/p/native-powerpoint-chart-exports-dont-exported-as-images | Gamma ideas board | practitioner]. Search snippet: tables export as editable PowerPoint tables, with sharp corners [source: https://help.gamma.app/en/articles/8022861-how-do-i-export-my-gamma | How do I export my gamma | vendor] (search snippet only; page not fetched).

### Pitch

Supported: slides, images, text, shapes, tables, speaker notes, embedded Vimeo/YouTube videos; "Charts (these may look different in the export based on PowerPoint's formatting)"; "Fonts will be included in the export", but "we cannot always embed the font into the PowerPoint file", falling back to Calibri in some cases; unsupported: "Uploaded videos," "Recordings," "Animations (enter/exit effects, transitions, continuity)" [source: https://help.pitch.com/en/articles/6713988-export-a-presentation-to-power-point | Export a presentation to PowerPoint | vendor]. The page does not say whether charts are native.

### DOM-walking converters

dom-to-pptx: "a Coordinate Scraper & Style Engine that traverses your DOM, calculates the exact computed styles of every element ... and mathematically maps them to native PowerPoint shapes and text boxes"; "Smart Font Embedding: The library automatically detects the fonts used in your HTML, finds their URLs in your CSS, and embeds them into the PPTX"; `svgAsVector: true` keeps SVGs as vectors, then "right-click the SVG image and select "Convert to Shape""; `template: './corporate-template.pptx'` to "inherit a real PowerPoint slide layout/master background" [source: https://github.com/atharva9167j/dom-to-pptx | dom-to-pptx README | vendor]. Depends on `pptxgenjs ^4.0.1`, `puppeteer`, `html2canvas`, `opentype.js`, `fonteditor-core`; v2.1.2 on 2026-09-14; 380 stars; 209,322 npm downloads last month [source: https://registry.npmjs.org/dom-to-pptx | npm registry | vendor] [source: https://api.npmjs.org/downloads/point/last-month/dom-to-pptx | npm downloads API | vendor]. License file: "MIT License / Copyright (c) 2025 Atharva Dharmendra Jagtap and dom-to-pptx contributors" [source: https://github.com/atharva9167j/dom-to-pptx/blob/master/LICENSE | dom-to-pptx LICENSE | vendor].

html2pptx (earlier Anthropic pptx skill, read via a third-party npm mirror): takes an HTML file and a PptxGenJS presentation, returns a slide plus placeholder positions "calculated from rendered layout"; "Text in `<div>` or `<span>` without a text tag will be silently ignored"; "Never use CSS gradients"; charts go into reserved placeholders with `slide.addChart(...)` [source: https://cdn.jsdelivr.net/npm/stableclaw@2026.5.15/skills/pptx/html2pptx.md | HTML to PowerPoint Guide (mirror) | practitioner]. The current Anthropic skill dropped html2pptx and writes PptxGenJS scripts directly [source: /home/pascal/.claude/skills/synced/fd3eaa6c-fd9b-44e1-9ac4-d446976eb04d_4247339b-e14f-4e59-9090-d60035112cef/pptx/SKILL.md | Anthropic pptx skill | practitioner].

## SQ4: fonts and SVG in pptx exports

### Fonts

PowerPoint embedding: Windows (Microsoft 365, 2024, 2021, 2019, 2016) and Mac ("PowerPoint for Microsoft 365 for Mac, PowerPoint 2024 for Mac, and PowerPoint 2021 for Mac") can embed fonts; "Use OpenType (.OTF) or TrueType (.TTF), if possible"; "Not all TrueType fonts can be embedded" (embeddability set by the font: Non-embeddable, Preview/Print, Editable, Installable); embedding only used characters "limits editing" [source: https://support.microsoft.com/en-us/office/benefits-of-embedding-custom-fonts-cb3982aa-ea76-4323-b008-86670f222dbc | Benefits of embedding custom fonts | vendor]. The page says nothing about PowerPoint for the web or mobile. No primary source found for how PowerPoint for the web handles embedded fonts; one secondary page claims the web version cannot embed [source: https://pdf.it.com/learn/pptx-to-pdf-missing-fonts | PowerPoint to PDF missing fonts | practitioner] (search snippet only, low quality).

File format: PowerPoint 2007 and 2010 "generate .eot files with the '.fntdata' extension when fonts are selected to be embedded" [source: https://en.wikipedia.org/wiki/Embedded_OpenType | Embedded OpenType | independent] (search snippet only).

Library support: none in PptxGenJS or python-pptx (see SQ1); pptx-embed-fonts adds it to PptxGenJS output; dom-to-pptx embeds detected web fonts; Aspose embeds via `fonts_manager` (see SQ1). LibreOffice 25.8: "LibreOffice now supports font embedding, so slides keep their intended look even on systems without those fonts installed"; "only the fonts actually in use are embedded" [source: https://www.collaboraonline.com/blog/collabora-productivity-contributions-in-libreoffice-25-8/ | Collabora contributions in LibreOffice 25.8 | vendor].

Who embeds: Slidev editable export does not ("A `.pptx` names fonts rather than carrying them ... The export prints which families it referenced") [source: https://github.com/slidevjs/slidev/pull/2722 | Slidev PR 2722 | vendor]; Pitch tries and sometimes falls back to Calibri [source: https://help.pitch.com/en/articles/6713988-export-a-presentation-to-power-point | Pitch help | vendor]; Google Slides ignores embedded fonts on import [source: https://help.gamma.app/en/articles/15939201-why-doesnt-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma | Gamma help | vendor].

Theme fonts: PptxGenJS writes only the heading and body faces into the theme (`headFontFace`, `bodyFontFace`) [source: https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/core-interfaces.ts#L462-L474 | core-interfaces.ts | vendor]; text referencing `+mn-lt` / `+mj-lt` follows the theme body / heading font [source: /home/pascal/.claude/skills/synced/fd3eaa6c-fd9b-44e1-9ac4-d446976eb04d_4247339b-e14f-4e59-9090-d60035112cef/pptx/SKILL.md | Anthropic pptx skill | practitioner]. The current exporter's fallback rule: Aptos for sans, Consolas for mono, or the preset's corporate font when recipients have it [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/exporting-presentations-to-pptx/references/python-pptx-recipes.md | python-pptx recipes | practitioner].

### SVG

Office support: Microsoft 365 apps "support inserting and editing scalable vector graphics (.SVG) files"; Applies To lists PowerPoint for Microsoft 365, 2024, 2021, 2019, Mac and mobile; "you can disassemble an SVG file and customize individual parts of it by using the Convert to Shape option"; on iOS one can only edit SVGs inserted elsewhere [source: https://support.microsoft.com/en-us/office/edit-svg-images-in-microsoft-office-69f29d39-194a-4072-8c35-dbe5e7ea528c | Edit SVG images in Microsoft Office | vendor]. A 2017 post: all versions of Office 365 and PowerPoint 2016 supported .svg at that point [source: https://thepowerpointblog.com/2017/01/ | The PowerPoint Blog, January 2017 | practitioner] (search snippet only). Templafy: inserting SVG into Office without SVG support converts it to PNG [source: https://templafyone.templafy.com/hc/en-us/articles/8375112582429-Support-for-svg-files | Support for svg files | vendor] (search snippet only). The exact first build with SVG support is unconfirmed.

File structure: the PNG sits in `a:blip r:embed`, the SVG in an `a:extLst` extension `asvg:svgBlip` (uri `{96DAC541-7B7A-43D3-8B79-37D633B846F1}`) [source: https://github.com/gitbrent/PptxGenJS/blob/3c9ec1b687c174952166f6a34b5e87ebf69fa469/src/gen-xml.ts#L576-L590 | PptxGenJS gen-xml.ts | vendor]. FlexCel docs: "If the blip contains an SVG image, then the actual data of the SVG image is here. In this case, PictureData contains a PNG render of the SVG image" [source: https://doc.tmssoftware.com/flexcel/net/api/FlexCel.Core/TBlip/SVGBlip.html | TBlip.SVGBlip Property | vendor] (search snippet only). The MS-ODRAWXML spec that defines `svgBlip` was not read: [inaccessible] learn.microsoft.com returned 404 and msopenspecs.microsoft.com failed with a certificate error [source: https://msopenspecs.microsoft.com/files/MS-ODRAWXML/%5bMS-ODRAWXML%5d-210216.docx | MS-ODRAWXML | vendor] [inaccessible].

Library support: PptxGenJS writes svgBlip plus a canvas-made PNG in the browser, a broken placeholder PNG in Node (see SQ1); python-pptx has no SVG (see SQ1); power-pptx claims "native SVG embedding with PNG fallback" (see SQ1); Slidev's editable export and the current plugin both rasterize SVG (see SQ3 and Baseline); dom-to-pptx offers `svgAsVector: true` (see SQ3).

## License table (from LICENSE files read 2026-10-20)

| Library | License (first lines of LICENSE) | Non-standard parts | File |
|---|---|---|---|
| PptxGenJS | "The MIT License (MIT) / Copyright (c) 2015-2022 Brent Ely" | none | https://github.com/gitbrent/PptxGenJS/blob/master/LICENSE |
| python-pptx | "The MIT License (MIT) / Copyright (c) 2013 Steve Canny" | none | https://github.com/scanny/python-pptx/blob/master/LICENSE |
| power-pptx | MIT, "Copyright (c) 2013 Steve Canny ... Copyright (c) 2026 Daniel Halwell" | none | https://github.com/codehalwell/power-pptx/blob/master/LICENSE |
| pptx-automizer | "MIT License / Copyright (c) 2021 Thomas Singer" | none | https://github.com/singerla/pptx-automizer/blob/main/LICENSE |
| officegen | "MIT License / Copyright (c) 2013 Ziv Barber" | none | https://github.com/Ziv-Barber/officegen/blob/master/LICENSE |
| pptx-embed-fonts | "MIT License / Copyright (c) 2025 liyao1520" | none | https://github.com/liyao1520/pptx-embed-fonts/blob/main/LICENSE |
| dom-to-pptx | "MIT License / Copyright (c) 2025 Atharva Dharmendra Jagtap and dom-to-pptx contributors" | none | https://github.com/atharva9167j/dom-to-pptx/blob/master/LICENSE |
| Open XML SDK | "The MIT License (MIT) / Copyright (c) .NET Foundation and Contributors" | none | https://github.com/dotnet/Open-XML-SDK/blob/main/LICENSE |
| Apache ECharts | "Apache License / Version 2.0, January 2004" | NOTICE file must travel with redistributions; four files embed d3 code under BSD-3 | https://github.com/apache/echarts/blob/master/LICENSE |
| zrender | "BSD 3-Clause License / Copyright (c) 2017, Baidu Inc." | no-endorsement clause | https://github.com/ecomfe/zrender/blob/master/LICENSE |
| Vega-Lite | BSD-3, "Copyright (c) 2015, University of Washington Interactive Data Lab" | no-endorsement clause | https://github.com/vega/vega-lite/blob/main/LICENSE |
| Vega | BSD-3, "Copyright (c) 2015-2023, University of Washington Interactive Data Lab" | no-endorsement clause | https://github.com/vega/vega/blob/main/LICENSE |
| vl-convert | BSD-3, "Copyright 2023 Jon Mease" | no-endorsement clause | https://github.com/vega/vl-convert/blob/main/LICENSE |
| Chart.js | "The MIT License (MIT) / Copyright (c) 2014-2024 Chart.js Contributors" | none | https://github.com/chartjs/Chart.js/blob/master/LICENSE.md |
| Observable Plot | ISC, "Copyright 2020-2025 Observable, Inc." | none | https://github.com/observablehq/plot/blob/main/LICENSE |
| Recharts | "The MIT License (MIT) / Copyright (c) 2015-present recharts" | none | https://github.com/recharts/recharts/blob/main/LICENSE |
| D3 | ISC, "Copyright 2010-2023 Mike Bostock" | none | https://github.com/d3/d3/blob/main/LICENSE |
| Slidev | "MIT License / Copyright (c) 2020-PRESENT Anthony Fu" | none | https://github.com/slidevjs/slidev/blob/main/LICENSE |
| Marp CLI / Marp Core | "MIT License / Copyright (c) 2018 Marp team" | none | https://github.com/marp-team/marp-cli/blob/main/LICENSE |
| Anthropic pptx skill | "© 2025 Anthropic, PBC. All rights reserved." | proprietary, governed by Anthropic terms | https://github.com/anthropics/skills/blob/main/skills/pptx/LICENSE.txt |
| Aspose.Slides | commercial, per developer and deployment location | paid ($1,399 and up) | https://purchase.aspose.com/pricing/slides/family/ |
| Syncfusion Presentation | commercial; free Community License under $1M revenue, 5 or fewer developers, 10 or fewer employees | paid otherwise | https://www.syncfusion.com/products/communitylicense |

All LICENSE texts were fetched through `gh api repos/<owner>/<repo>/license` and decoded; the GitHub SPDX labels were not used as evidence.

## Gaps

- MS-ODRAWXML (`svgBlip` definition) [inaccessible]; structure taken from PptxGenJS source instead.
- PowerPoint for the web and mobile handling of embedded fonts: no primary source found.
- Gamma table export and Syncfusion feature details: search snippets only, pages not read.
- power-pptx feature claims not checked against its code.
