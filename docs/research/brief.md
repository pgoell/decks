# Research Brief

Status: approved

Topic: Competitor and prior-art scan for decks, a new slide editor (PowerPoint-like canvas for people, typed JSON document and MCP tools for agents). The scan exists to answer the eight open questions at the end of docs/spec-v0.md (read it and docs/idea.md first).
Scope: Current state of the products and libraries (as of 2026-10). Cover at least: tldraw, Gamma, Pitch, Slidev, Typst with Touying, reveal.js, Keynote, Figma (comments), Google Slides API (and its master/layout/placeholder model), PowerPoint's own model (OOXML slide masters and layouts), Blattomat itself (~/Code/blattwerk, its docs/spec-v0.md and docs/okf/), and the current presentations plugin (~/Code/pgoell-claude-tools/plugins/presentations). Also pptx export libraries, chart libraries and CRDTs (Yjs, Automerge) as far as the open questions need them. For every library or product we might build on, read its LICENSE file directly, never a README badge or summary. Out: anything the eight open questions do not need.
Audience: Pascal plus the agents that turn the settled spec into issues; every claim names its source (URL or file path).
Purpose: Answer the eight open questions in docs/spec-v0.md so the spec can be settled and turned into issues.
Deliverable: Full report: product by product deep dives, then one answer per open question with a recommendation, plus a license table. About 8,000+ words, English.
Format: html
Deadline: none
Output path: docs/research/
After the report: commit on branch research/competitor-scan, push, open a PR into master (no AI attribution lines).
