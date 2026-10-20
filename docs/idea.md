# decks: the idea

A slide editor that works like PowerPoint for people and like a typed document for agents. Blattomat does this for worksheets; decks does it for talks.

Status: idea, 2026-10-20. The scope of the first version is in [spec-v0.md](spec-v0.md). Everything here that is not in that spec comes later, but the data model must leave room for it.

## Why

Today an agent writes each deck as hand-made HTML and CSS (`~/Code/presentations`, built by the `presentations` plugin). The decks look good, and agents write HTML well. What hurts is editing by hand:

- An agent gets something wrong and a quick fix by hand takes longer than it should.
- Showing the agent what you want ("this box goes there") means describing it in words, because nothing can be dragged freely. The current deck editor only edits text, reorders siblings and shifts an element with `translate`.
- While drafting the storyline you already want to sketch how each slide is laid out, before any slide is built.

Fixing the editor alone does not solve it. Decks today are files in one checkout, edited through a local server, by an agent working in that checkout. The goal is one place that holds every deck, open from any device, and reachable by any agent that builds one (Claude Code, claude.ai, or another agent through MCP), with the same document whoever wrote it.

A personal asset first: every future talk is made here. Later maybe a tool Deloitte teams host for themselves.

## The wish list

### Editing

- A canvas like PowerPoint: drag, resize, snap, align, multi-select, undo.
- Typed blocks (text, bullets, image, table, chart, shape) and raw blocks (HTML, SVG, JavaScript), so nothing a hand-written deck can do is lost. A raw SVG block moves as one box in the editor while the agent rewrites what is inside.
- Rough layout while drafting: empty boxes with a note for the agent.
- Feedback the agent can read: select blocks, comment and draw, then copy everything as one prompt or let the agent read it through MCP.

### Look

- Themes: colours, fonts, spacing as tokens.
- Slide masters: layouts with placeholders, plus fixed parts such as logo, header, footer and page number.
- Presets: a theme with its masters, saved and reused; made from a client's PowerPoint template.
- Animations from the start: slide transitions, builds one click at a time, emphasis, exit and motion paths, timed as in PowerPoint and kept in the pptx export.
- Ideally everything PowerPoint can do here.

### Decks and slides

- Copy a slide from one deck into another; the slide takes on the target deck's master.
- A library to scroll through all decks and their slides.
- Search across every slide: keyword search and semantic (vector) search, with hits that show the slide, not only the deck.

### In and out

- Export to HTML, PDF and PowerPoint.
- Import PowerPoint, PDF and Excel: directly where it works, otherwise handed to an agent that rebuilds the deck through the MCP tools.
- An MCP server like Blattomat's, so any agent can build and change decks.

### People

- Self-hosted, as Blattomat is. The code is public; every deck stays private on the VPS, since many hold client material.
- Workspaces.
- Collaborative editing, with people and agents in the same deck at once.
- Version history and branches: an agent works on its own copy, a person accepts or rejects its changes slide by slide.

## Phases

1. **Personal core.** Document model, renderer, canvas editor, raw blocks, themes and masters, MCP, export to HTML, PDF and pptx. Replaces how decks are made today.
2. **Library.** Many decks, copy slides between them, search, import from pptx and PDF.
3. **Many people.** Accounts, workspaces, live collaboration (a CRDT such as Yjs), agent branches, hosting by others.

## Decisions taken now for later phases

- Every slide and block has a stable id. Nothing is addressed by its place in a list. Collaboration, agent branches and slide copy all need this.
- An agent changes a deck only through the same operations the editor uses, so it can later join a live session as one more peer.

## Open before phase 3

- Hosting for Deloitte: check the employment contract and Deloitte's policy on side projects and IP before building anything meant for Deloitte to host. That decides whose project it is.
