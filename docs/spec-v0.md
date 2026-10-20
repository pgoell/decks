# decks v0

Status: draft, before the competitor scan. The scan (in `docs/research/`) answers the open questions at the end; then this spec is settled and turned into issues.

A slide editor: a 16:9 canvas like PowerPoint, typed blocks plus raw HTML, SVG and JavaScript blocks, themes and slide masters, MCP tools for agents, and export to HTML, PDF and pptx. It runs in the browser. The wider idea and later phases are in [idea.md](idea.md).

## Who it is for

One user first: Pascal, who gives talks at Deloitte and to clients. Today an agent builds each deck as HTML with the `presentations` plugin; the decks live in `~/Code/presentations`. He edits by hand when the agent gets a detail wrong, and wants to show the agent a layout by placing boxes rather than describing it. He wants every deck in one hosted place, open from any device, whichever agent built it: Claude Code, claude.ai or another MCP client. The local deck editor cannot do that, because it serves one file from one checkout to an agent in the same checkout.

## The test

v0 is done when one existing deck from `~/Code/presentations` is rebuilt in decks, partly by an agent through MCP and partly by hand, and the next talk is given from it, with presenter view, and the pptx export opens in PowerPoint with editable text.

## Scope

### 1. Canvas

- Slides of 1920 by 1080 px. A deck has one or more slides.
- Blocks are dragged, resized, deleted and locked, with corner and edge handles.
- Snap to the slide's margins and centre and to other blocks' edges and centres, with guide lines while dragging. Own guide lines and an optional grid.
- Multi-select, copy, paste, duplicate, align, distribute, bring forward, send back.
- Arrow keys move by 1 px, 10 with Shift.
- Undo and redo.
- A slide list beside the canvas: reorder, duplicate, delete, hide.
- Speaker notes per slide.

### 2. Blocks

| Block | Settings |
|---|---|
| Text | font, size, weight, italic, colour, alignment, line spacing; formatting on part of the text; bulleted and numbered lists with levels |
| Image | upload, crop, keep ratio |
| Shape | rectangle, rounded box, circle, line, arrow; fill, border, text inside |
| Table | rows, columns, text per cell, column widths, head row |
| Chart | bar, line, pie; data in the block |
| HTML | raw HTML and CSS, drawn inside the block's box |
| SVG | raw SVG, scaled to the box |
| Script | HTML plus JavaScript, run in a sandboxed frame inside the box |
| Placeholder | an empty box with a note, for sketching a layout before it is built |

### 3. Themes and masters

- A theme holds tokens: colours, fonts, font sizes, spacing.
- A master is a layout: placeholder boxes (title, body, image) and fixed blocks (logo, footer, page number, date).
- Each slide names one master. Changing the master changes every slide on it.
- A slide's block can override a theme token.
- A preset is a theme with its masters, saved and picked for a new deck. The existing presets in the `presentations` plugin (`extracting-presets`) are the first ones.

### 4. Animations

Part of the model from the first version, so no deck has to be rebuilt to gain them. The model follows PowerPoint's, so the pptx export keeps them as native animations.

- **Transitions** between slides: none, fade, push, wipe, zoom; a duration per slide.
- **Effects** on a block, of four kinds: `in` (appear, fade, fly in, zoom), `emphasis` (pulse, grow, colour change), `out` (the reverse of `in`), `path` (move along an SVG path).
- **Timing** as in PowerPoint: each effect starts on a `click`, `with` the effect before it, or `after` it, with a duration and a delay. A slide's clicks are its steps.
- **Parts of a block**: a text or list block plays `by` paragraph, one bullet per click. An effect on an SVG block can target an element inside it by `target` selector, so a diagram builds part by part.
- **Script blocks** get the slide's step as an event (`deck:step`, with the step number and the direction), so custom JavaScript animation runs in time with the clicks, forwards and backwards.
- **Editor**: an animation panel per slide lists the effects in order. Drag to reorder, set start, effect, duration and delay, play the slide from any step. Numbered badges on the canvas show which step each block appears on.
- **Present**: each click plays the next step; back undoes it. The presenter view's preview shows the next step, not only the next slide.

### 5. Feedback for agents

The fastest way to tell an agent what to change is to point at it. Modelled on the comments of the current deck editor (`editing-presentations`), but tied to ids instead of source lines, so a comment still finds its block after edits.

- **Comment**: select one or more blocks (or nothing, for the whole slide), type a note. A comment can also name an animation step.
- **Draw**: a pen on the slide for circles and arrows. A drawing belongs to its comment and never enters the deck.
- **Threads**: the agent answers under a comment and marks it done; the user reopens it if the change is wrong.
- **Selection**: what the user has selected right now is readable by the agent, so "make this bigger" needs no comment at all.
- **Copy prompt**: all open comments as one markdown prompt on the clipboard, per slide a screenshot path with each commented block boxed and numbered, and per comment the note, the block ids, their type and props. For agents without MCP.
- **MCP**: `get_selection`, `list_comments` (open ones by default, with a screenshot per slide), `reply_comment`, `resolve_comment`. An agent works through the open comments and resolves each with a reply on what it changed.

### 6. Present

- Full-screen presenting in the browser.
- Presenter view with notes, next slide and timer, kept in sync with the audience window, as `presenter.js` does today.

### 7. Export

- **HTML**: one self-contained file per deck; effects play through the Web Animations API.
- **PDF**: headless Chromium prints the same markup the screen shows; each slide in its final state, or one page per step.
- **pptx**: typed blocks become native shapes and text boxes; masters become slide layouts; HTML, SVG and script blocks become images. Transitions and effects on typed blocks become native PowerPoint ones (`<p:transition>`, `<p:timing>`). An effect that targets the inside of an SVG block, and a Script block's own motion, are lost; the export lists each one it drops.

### 8. MCP

- Remote MCP server with OAuth, as Blattomat has.
- Tools for decks, slides, blocks, animations, themes, masters and comments, all by id.
- `render_slide` takes a step number, so the agent sees each step of a build.
- `render_slide` returns a PNG so the agent sees what it built.
- Each write answers with the problems it finds: overflowing text, overlapping text, text below the type floor (the hard gates H1, H2 and H7 from the `presentations` plugin).
- Agent docs in the repo, in the OKF format Blattomat uses (`docs/okf/`).

### 9. Storage and accounts

- Decks saved on the server, automatically after each change.
- A list of decks with title, date and a thumbnail.
- Login and invite-only sign-up, as in Blattomat.

## Not in v0

- Live collaboration, workspaces, version branches (phase 3)
- Library, search across decks, copy slides between decks (phase 2)
- Import from pptx, PDF or Excel (phase 2)
- Drawing and bending a motion path by hand on the canvas: planned for v1. In v0 an agent writes the path through MCP and the editor shows and plays it
- Mobile editing

## Architecture

| Part | Choice | Why |
|---|---|---|
| Frontend | TypeScript, React, Vite | As Blattomat; most of the canvas code ports over |
| Canvas | Each block an HTML element placed in px on the 1920 by 1080 slide, scaled to fit; `moveable` and `selecto` for drag, resize and select | The same markup draws the editor, the presentation and the PDF |
| Text field | ProseMirror for the text being edited | As Blattomat |
| Raw blocks | HTML and SVG inline; Script blocks in a sandboxed `iframe` | A script must not reach the editor or the session |
| Backend | FastAPI | As Blattomat |
| Storage | SQLite; a deck is one JSON document | A few users; Postgres when it outgrows that |
| PDF | Playwright and Chromium, print URL with a signed short-lived token | As Blattomat |
| pptx | Port of the exporter in the `presentations` plugin, from JSON instead of from HTML | JSON blocks map almost one to one onto pptx shapes |

### Units

Lengths in px on a 1920 by 1080 slide, font sizes in px. A 16:9 pptx slide is 12192000 by 6858000 EMU, so 1 px is exactly 6350 EMU and the export rounds nothing.

### Deck document

```json
{
  "id": "d_7Kq2",
  "title": "Presentations as code",
  "theme": "t_deloitte",
  "slides": [
    {
      "id": "s_a91F",
      "master": "m_title_body",
      "notes": "Open with the question, not the answer.",
      "hidden": false,
      "blocks": [
        { "id": "b_x3", "type": "text", "slot": "title",
          "x": 96, "y": 80, "w": 1728, "h": 120, "z": 1, "locked": false,
          "props": { "text": "Why decks as code" } },
        { "id": "b_y8", "type": "svg",
          "x": 960, "y": 260, "w": 864, "h": 700, "z": 2, "locked": false,
          "props": { "svg": "<svg viewBox=\"0 0 864 700\">…</svg>" } }
      ],
      "transition": { "effect": "fade", "ms": 400 },
      "animations": [
        { "id": "a_1", "block": "b_y8", "kind": "in", "effect": "fade",
          "start": "click", "ms": 500, "delay": 0 },
        { "id": "a_2", "block": "b_y8", "target": "#arrow-2", "kind": "in", "effect": "wipe",
          "from": "left", "start": "after", "ms": 300, "delay": 0 }
      ]
    }
  ]
}
```

`animations` is the slide's timeline, in play order. Each effect names its block by id, so moving or restyling a block keeps its effects; deleting a block deletes them.

A block with `slot` fills that placeholder of the slide's master and takes its box from the master unless it sets its own. Comments live beside the deck, not in it: each names its slide and block ids, and an export never carries them. Themes and masters are documents of their own, named by id, so many decks share one.

Slides and blocks are a list in the JSON, but every operation names them by `id`, never by their place in the list. This keeps live collaboration, agent branches and slide copy open for later.

## What to port

| From | What |
|---|---|
| Blattomat (`~/Code/blattwerk`) | canvas: drag, resize, snap, guides, multi-select, undo (`frontend/src/`); ProseMirror text field; Playwright PDF (`src/blattwerk/pdf.py`); invite-only accounts (`auth.py`); remote MCP with OAuth (`remote.py`, `oauth.py`); OKF agent docs (`docs/okf/`); feedback job (`scripts/feedback-*`); deploy (`.github/workflows/deploy.yml`, server-infra `deploy/`) |
| `presentations` plugin (`~/Code/pgoell-claude-tools/plugins/presentations`) | `deck-stage.js`, presenter view (`presenter.js`, `presenter.html`); pptx exporter (`exporting-presentations-to-pptx`); presets and `extracting-presets`; hard gates H1, H2, H7 |

## Open questions for the competitor scan

1. Draw the canvas as HTML elements (Blattomat, Pitch) or on a canvas library such as tldraw? Check tldraw's license by reading its LICENSE file, not a badge or a summary.
2. How do Google Slides, Pitch and PowerPoint model masters, layouts and placeholders, and what of that must the JSON hold for the pptx export to give real layouts?
3. How does Slidev (Markdown) let an agent write slides, and is a text form of the deck worth having beside the JSON?
4. How do Gamma and Pitch let an AI change a deck: whole deck, one slide, one block? What does it do to blocks the user placed by hand?
5. Which libraries for pptx export and for charts fit, and what are their licenses?
6. How do PowerPoint (`<p:timing>`), Keynote, Google Slides, reveal.js and Slidev model animations and builds, and which effects survive the trip to pptx and back?
7. How do Figma, Google Slides and Pitch anchor comments to objects, and how do Gamma and Pitch let a user point their AI at one element?
8. What would live collaboration need from the model now (Yjs, Automerge), so v0 does not block it?
