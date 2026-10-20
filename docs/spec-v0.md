# decks v0

Status: settled after the competitor scan ([report](research/report.md)). Next step: turn it into issues. The open items at the end stay open until a test or a decision closes them.

A slide editor: a 16:9 canvas like PowerPoint, typed blocks plus raw HTML, SVG and JavaScript blocks, themes, slide masters and layouts, MCP tools for agents, and export to HTML, PDF and pptx. It runs in the browser. The wider idea and later phases are in [idea.md](idea.md).

## Who it is for

One user first: Pascal, who gives talks at Deloitte and to clients. Today an agent builds each deck as HTML with the `presentations` plugin; the decks live in `~/Code/presentations`. He edits by hand when the agent gets a detail wrong, and wants to show the agent a layout by placing boxes rather than describing it. He wants every deck in one hosted place, open from any device, whichever agent built it: Claude Code, claude.ai or another MCP client. The local deck editor cannot do that, because it serves one file from one checkout to an agent in the same checkout.

## The test

v0 is done when one existing deck from `~/Code/presentations` is rebuilt in decks, partly by an agent through MCP and partly by hand, and the next talk is given from it, with presenter view. Its pptx export opens in PowerPoint with editable text, and PowerPoint's Change Layout and Reset work on its slides.

The test rests on text, images, shapes, tables, SVG and notes, which cover nearly all of today's content. The existing decks use no PowerPoint timing, no builds, no native charts and almost no placeholders, so animation and chart export get checks of their own in PowerPoint and do not gate the test.

## Scope

The eleven parts of v0 (canvas, blocks, themes with masters and layouts, animations, feedback for agents, present, export, MCP, storage and accounts, import of the existing decks, privacy) are in [spec-v0/scope.md](spec-v0/scope.md). A section number below names a part there.

## Not in v0

- Live collaboration, workspaces, version branches (phase 3)
- Library, search across decks, copy slides between decks (phase 2)
- Import of slides from pptx, PDF or Excel (phase 2); masters and layouts from a pptx template come in from v0, see 3; plugin HTML decks from v0, see 10
- Drawing and bending a motion path by hand on the canvas: planned for v1. In v0 an agent writes the path through MCP and the editor shows and plays it
- Morph transitions
- A text form of the deck that can be edited and read back (the scan found no product that keeps one in step with a canvas); `outline` is read only
- Mobile editing

## Architecture

| Part | Choice | Why |
|---|---|---|
| Frontend | TypeScript, React, Vite | As Blattomat; most of the canvas code ports over |
| Canvas | Each block an HTML element placed in px on the 1920 by 1080 slide, scaled to fit | The same markup draws the editor, the presentation and the PDF. tldraw, Pitch and Google Slides draw DOM or SVG too |
| Drag, resize, select | `moveable` and `selecto` (MIT), kept behind one module of decks' own; vendored or forked | No release since 2023-12 and no answer from the author; Blattomat's editor already works around them in at least ten places |
| Text field | ProseMirror for the text being edited; text stored as runs it can load | As Blattomat |
| Raw blocks | HTML and SVG inline; Script blocks in a sandboxed `iframe` | A script must not reach the editor or the session |
| Backend | FastAPI | As Blattomat; python-pptx is Python |
| Writes | The server applies operations in order, checks locks, versions and cascades (deleting a block deletes its effects), then bumps the deck version | How Figma, tldraw sync and Linear order writes. No CRDT: a CRDT merges where decks must refuse |
| Storage | SQLite; a deck is one JSON document; comments in their own table | A few users; Postgres when it outgrows that |
| PDF and images | Playwright and Chromium, print URL with a signed short-lived token | As Blattomat; also rasterizes for pptx |
| pptx | python-pptx (MIT), pinned, plus decks' own OOXML writers | The mature writer, but no commits since 2024-08; decks owns the rest |
| Charts | ECharts (Apache-2.0; ship its NOTICE), SVG in the browser and in the PDF | Takes the block's table as it is; the same table makes the native pptx chart |

Not used: tldraw. Its LICENSE allows no production use without a paid key, and its model has no masters, layouts or placeholders.

### Units

Lengths in px on a 1920 by 1080 slide, font sizes in px. A 16:9 pptx slide is 12192000 by 6858000 EMU, so 1 px is exactly 6350 EMU and the export rounds nothing.

### Deck document

```json
{
  "schema": 1,
  "id": "d_7Kq2",
  "version": 42,
  "title": "Example talk",
  "theme": "t_example",
  "slides": {
    "s_a91F": {
      "order": "a0",
      "layout": "l_title_body",
      "notes": "Open with the question, not the answer.",
      "hidden": false,
      "transition": { "effect": "fade", "ms": 400 },
      "blocks": {
        "b_x3": { "type": "text", "slot": "title", "order": "a0",
          "x": 96, "y": 80, "w": 1728, "h": 120, "locked": false,
          "props": { "paragraphs": [{ "runs": [{ "text": "Why decks as code" }] }] } },
        "b_y8": { "type": "svg", "order": "a1",
          "x": 960, "y": 260, "w": 864, "h": 700, "locked": false,
          "props": { "svg": "<svg viewBox=\"0 0 864 700\">…</svg>" } }
      },
      "animations": {
        "a_1": { "order": "a0", "block": "b_y8", "kind": "in", "effect": "fade",
          "start": "click", "ms": 500, "delay": 0 },
        "a_2": { "order": "a1", "block": "b_y8", "target": "#arrow-2", "kind": "in",
          "effect": "wipe", "from": "left", "start": "after", "ms": 300, "delay": 0 }
      }
    }
  }
}
```

Slides, blocks and animations are maps keyed by id. Their order is the `order` string, a fractional index (rocicorp's `fractional-indexing`, CC0): the slide list for slides, stacking for blocks (later lies on top), play order for animations. Inserting between two items makes a key between theirs, so no other item changes. The client makes ids and keys; the server removes a duplicate key. Nothing in the document is derived: thumbnails, warnings and computed text heights live outside it.

`schema` is the document format's version, for migrations; `version` counts the writes.

Themes, masters and layouts are documents of their own, named by id, so many decks share one. Comments live in their own table.

## What to port

| From | What |
|---|---|
| Blattomat (`~/Code/blattwerk`) | canvas: drag, resize, snap, guides, multi-select, undo (`frontend/src/`), with moveable moved behind one module on the way; ProseMirror text field; Playwright PDF (`src/blattwerk/pdf.py`); invite-only accounts (`auth.py`); remote MCP with OAuth and its lock refusal (`remote.py`, `oauth.py`); OKF agent docs (`docs/okf/`); feedback job (`scripts/feedback-*`); deploy (`.github/workflows/deploy.yml`, server-infra `deploy/`) |
| Blattomat gaps to close in the port | its lock refusal does not cover whole-document writes (`replace_doc`); agent writes are not undo steps for the user |
| `presentations` plugin (`~/Code/pgoell-claude-tools/plugins/presentations`) | `deck-stage.js`, presenter view (`presenter.js`, `presenter.html`); the python-pptx recipes and gotchas (`exporting-presentations-to-pptx/references/python-pptx-recipes.md`) as the basis of a new exporter; presets and `extracting-presets`; hard gates H1, H2, H7 |

## Open items

1. **Layouts in PowerPoint**: import a template, export a deck built on it, open the file in PowerPoint, try Change Layout and Reset, and compare its masters with the template's. The design in [scope section 3](spec-v0/scope.md#3-themes-masters-and-layouts) is a hypothesis until this passes. First test issue. The template in the repo is a made-up one; a real client template is tried on the VPS only.
2. **Preset ids**: check each effect of [scope section 4](spec-v0/scope.md#4-animations) in PowerPoint before it ships.
3. **python-pptx**: pinned for now; decide to vendor or fork once a bug in it blocks decks.
4. **Licences**: neither this repo nor Blattomat has a LICENSE file. Choose one for decks, and settle Blattomat's (blattwerk issue #889) before its code moves here.
5. **Deloitte**: check the employment contract and Deloitte's IP policy before anything is built for Deloitte to host (see [idea.md](idea.md)).
