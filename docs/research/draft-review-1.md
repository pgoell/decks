# Report Review (Iteration 1)

VERDICT: PASS

ISSUES:
  - id: typo-and-and
    severity: minor
    category: prose
    description: Doubled word "and and it cannot make LibreOffice stand in".
    location: Limitations and Open Problems, "What the scan cannot solve" paragraph (draft.md line 290)
  - id: untagged-inline-claims
    severity: minor
    category: prose
    description: Brief says every claim names its source, but several claims carry no inline source or credibility tag (pptx-automizer, officegen, Open XML SDK, Aspose prices; Observable Plot quote; Aspose FOSS and COM-observed table agreement; Canva @mention; Yjs 14 RC; master/layout id range; Marp "not recommend" and the --with-clicks PR; Tylina license; ECharts LICENSE D3 note; Automerge 3 memory; Blattomat "Sharp text" quote and WebKit flakes #225/#932).
    location: draft.md lines 130, 134, 150, 172, 186, 216, 228, 232, 244, 248, 252
  - id: pitch-calibri-unsourced
    severity: minor
    category: accuracy
    description: "Pitch falls back to Calibri" is cited only to Microsoft, Collabora and Gamma pages; no Pitch source sits behind it (the synthesis has the same hole).
    location: What Matters Most, question 5, fonts paragraph (line 132)
  - id: misquote-recharts
    severity: minor
    category: accuracy
    description: Quote reads "never explicitly supported"; the synthesis quote is "never been explicitly supported".
    location: Question 5, chart libraries paragraph (line 134)
  - id: misquote-figma-ids
    severity: minor
    category: accuracy
    description: Puts "can't have the server assign ids" in quotes as Figma's words; the synthesis quotes only "can't".
    location: Analysis and Insights, "Agents want what collaboration wants" (line 270)
  - id: block-count-shift
    severity: minor
    category: accuracy
    description: "slides rarely hold that many blocks" now refers to 50; the synthesis's author estimate is that slides rarely hold hundreds of blocks.
    location: Question 1, DOM costs paragraph (line 62)
  - id: crdt-claim-overreach
    severity: minor
    category: accuracy
    description: "Everyone who ships live editing on objects runs it through a server" and "CRDT libraries are better suited to text and offline-first apps" go past the synthesis, which calls the Pitch and Google Slides sync evidence thin (the draft says so itself at line 184) and makes no such suitability claim.
    location: Analysis and Insights, fourth bold paragraph (line 268)
  - id: section-order-vs-brief
    severity: minor
    category: format
    description: Brief asks for product deep dives first, then one answer per question; the draft follows the template and puts the answers (What Matters Most) before the product profiles. The order can be defended, but it differs from the Deliverable line.
    location: Sections "What Matters Most" (line 46) and "Supporting Evidence: product by product" (line 194)
  - id: length-well-above-target
    severity: minor
    category: format
    description: Prose runs to about 11,300 words before the license table and references, against "about 8,000+". The plus sign allows this, but some of the excess comes from the product profiles repeating the answers.
    location: Whole draft; Supporting Evidence section (lines 194 to 256)
  - id: thesis-restated-verbatim
    severity: minor
    category: flow
    description: The same keep/reject/five-tightenings list appears almost word for word in the Executive Summary paragraph 1, the Introduction's last paragraph, the Conclusions' first paragraph and the five-minute summary.
    location: lines 7, 36, 274, 303
  - id: template-meta-reference
    severity: minor
    category: flow
    description: "The template's usual scenario (an organization with pilots and no baselines)" refers to the report template, which the reader has never seen.
    location: Starting from zero, first paragraph (line 315)
  - id: python-pptx-risk-self-reference
    severity: minor
    category: content-gap-suspected
    description: The Limitations section says "The report ... leaves python-pptx's risk unweighed" and passes the call to the spec, while the question 5 recommendation still names python-pptx as core writer with no pin, vendor or fork advice. The synthesis likely lacks a basis for that mitigation.
    location: Limitations, python-pptx paragraph (line 284); question 5 Recommendation (line 140)
  - id: stock-figures-of-speech
    severity: minor
    category: prose
    description: A few stock phrases slip past Orwell rule 1: "two front doors", "points one way", "cuts against priority", "turns it around".
    location: lines 30, 11, 13, 196

SUMMARY:
The draft represents the synthesis faithfully. Thesis, adjudications, recommendations and the 58-row license table all match, the privacy rule holds (unnamed aggregates only) and the grep found no banned punctuation. All template sections are present. The open points are minor: a typo, some claims with no inline source, two slight misquotes, one overstated generalization, and a thesis repeated across sections.
