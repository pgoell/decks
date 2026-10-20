# Synthesis Review (Iteration 1)

VERDICT: PASS

ISSUES:
  - id: structure-no-product-deep-dives
    severity: minor
    category: structure
    description: Brief asks for product-by-product deep dives first; synthesis is organized only by open question, so per-product facts (Keynote, reveal.js, Pitch, Gamma, tldraw, Google Slides) are scattered across OQ sections and the writer must regroup them without a product index.
    location: "Claims by Sub-Question" (whole section); no product-level section exists
  - id: logic-port-wording-in-thesis
    severity: minor
    category: logic
    description: Thesis lists "a python-pptx exporter ported from the plugin" as a kept core bet, while OQ5.1 notes and Outliers state there is no exporter code to port (recipes plus per-deck scripts only); thesis wording contradicts its own finding.
    location: Thesis; OQ5.1 plugin exporter claim Notes; Outliers "Port of the exporter"
  - id: logic-python-pptx-staleness-unweighed
    severity: minor
    category: logic
    description: python-pptx is recommended as core writer although the synthesis records no commits since 2024-08 and 541 open issues; moveable gets an unmaintained-dependency mitigation plan, python-pptx gets none, and power-pptx as a possible fork is not weighed in the recommendation.
    location: OQ5.1 python-pptx claim; OQ5.4 "Recommendation basis"; License table row python-pptx
  - id: evidence-preset-id-independence
    severity: minor
    category: evidence-gap
    description: Thesis pillar "preset ids that two independent importers agree on" (LibreOffice, Aspose FOSS) does not show the two are independent (Aspose FOSS is a 2026, 15-star project that may derive from LibreOffice's table); corroboration rests on one PowerPoint-saved Fly In sample and a COM-observed table, with no PowerPoint test run.
    location: Thesis; OQ6.1 preset id claim; Reconciled Contradictions 5
  - id: evidence-layouts-untested-in-powerpoint
    severity: minor
    category: evidence-gap
    description: The OQ2 recommendation (generated base file with real layouts, inherit via empty spPr) rests on XML reads only; no PowerPoint Change Layout or Reset test was run (acknowledged as gap-change-layout), so the writer should frame it as a hypothesis for the v0 test.
    location: OQ2.4 derived field list; Gaps 1
  - id: coverage-multiplayer-pitch-gslides-thin
    severity: minor
    category: coverage
    description: collab-crdt SQ3 asks how Pitch and Google Slides do multiplayer; Pitch rests on one git-tower blog and an HN comment, Google Slides on a 2010 Google Docs OT post and the API's requiredRevisionId, with no Slides-specific sync source.
    location: OQ8 "Server-ordered sync without a CRDT is the norm" claim
  - id: source-quality-pitch-vendor-only
    severity: minor
    category: source-quality
    description: Pitch, the spec's named DOM model, is covered almost entirely by vendor help pages plus one player DOM inspection; its AI treatment of hand-placed blocks and block-level comment anchors are absent (acknowledged in Gaps 5 and Outliers), so OQ4 and OQ7 answers about Pitch must be stated as unknown, not inferred.
    location: OQ2.3 Pitch; OQ4 Pitch claim; OQ7 Pitch claim; Gaps 5

SUMMARY:
Synthesis covers all 36 sub-questions across nine clusters, cites a source per claim, adjudicates 17 contradictions, carries a full license table read from LICENSE files, and keeps private-repo facts to unnamed aggregates (Deloitte appears only as already named in the public spec). Remaining issues are minor: no product-level grouping for the brief's deep dives, a thesis wording slip on "ported", and thin corroboration on the preset-id and layout pillars.

REFETCH:
  - url: https://arxiv.org/html/2608.24103v1
    claim: ACE beat a Claude-Code-on-HTML baseline on instruction following (4.23 vs 3.81, p = .010, 94 tasks), visual quality not significantly different
    mark: confirmed
    quote: "IF 4.23 vs. 3.81 on the full 94-task benchmark, paired p=.010" ... "VQ means are statistically indistinguishable."
  - url: https://arxiv.org/html/2608.07617
    claim: TeXFix-Bench compile rate Markdown 90.2%, LaTeX 74.2%, Typst 60.3% (repair task)
    mark: confirmed
    quote: "Markdown is easiest under CRitt (90.2%, N=2,789), then LaTeX (74.2%, N=30,814)"; Typst 60.3% (N=15,048); setting is "LLM-based full-source document repair"
  - url: https://arxiv.org/abs/2510.04717v1
    claim: JSON Whisperer: LLMs fail at array index arithmetic; stable-key dictionaries fix it; patches cut tokens 31% at near-equal quality
    mark: confirmed
    quote: "transforms arrays into dictionaries with stable keys, eliminating index arithmetic complexities" ... "reduces token usage by 31% while maintaining edit quality within 5% of full regeneration"
  - url: https://raw.githubusercontent.com/tldraw/tldraw/main/LICENSE.md
    claim: tldraw LICENSE.md permits use in Development Environments, forbids Production Environments and interfering with License Key enforcement, may collect usage data, Delaware law
    mark: confirmed
    quote: "Not to use the Software in Production Environments." ... "The Software may collect and transmit usage data to tldraw for license compliance purposes." ... "governed by the laws of Delaware"
  - url: https://cdn.jsdelivr.net/npm/y-utility@0.1.4/README.md
    claim: 100k ops on 10 keys: 524,985 B Y.Map vs 271 B YKeyValue
    mark: confirmed
    quote: "| 100k | 10 | 271 | 524985 | 121 |" (ops, keys, YKeyValue bytes, Y.Map bytes, JSON bytes)
