# decks v0: scope

Part of the [v0 spec](../spec-v0.md).

## 1. Canvas

- Slides of 1920 by 1080 px. A deck has one or more slides.
- Blocks are dragged, resized, deleted and locked, with corner and edge handles.
- Snap to the slide's margins and centre and to other blocks' edges and centres, with guide lines while dragging. Own guide lines and an optional grid.
- Multi-select, copy, paste, duplicate, align, distribute, bring forward, send back.
- Arrow keys move by 1 px, 10 with Shift.
- Undo and redo, for the user's edits and for each agent write.
- A slide list beside the canvas: reorder, duplicate, delete, hide.
- Speaker notes per slide.

## 2. Blocks

| Block | Settings |
|---|---|
| Text | font, size, weight, italic, colour, alignment, line spacing; formatting on part of the text; bulleted and numbered lists with levels. Stored as paragraphs of runs, never as one string |
| Image | upload, crop, keep ratio |
| Shape | rectangle, rounded box, circle, line, arrow; fill, border, text inside |
| Table | rows, columns, text per cell, column widths, head row |
| Chart | bar, line, pie; the data is a table (categories, series, values) in the block |
| HTML | raw HTML and CSS, drawn inside the block's box |
| SVG | raw SVG, scaled to the box |
| Script | HTML plus JavaScript, run in a sandboxed frame inside the box |
| Placeholder | an empty box with a note, for sketching a layout before it is built |

## 3. Themes, masters and layouts

Three tiers plus a theme, as in PowerPoint and Google Slides, so the pptx export gives real layouts.

- **Theme**: twelve colour slots (`dk1`, `lt1`, `dk2`, `lt2`, `accent1` to `accent6`, `hlink`, `folHlink`), a heading and a body font, plus font sizes and spacing. A preset's colour tokens by role map onto the slots.
- **Master**: text styles by level for title, body and other text; fixed blocks (logo, background); the footer trio of date, footer text and slide number.
- **Layout**: belongs to one master, has a name and a kind from PowerPoint's list of layout types (title, title and content, two content, section header, blank and the rest), and holds slots. A slot has a role (title, body, picture and the other PowerPoint placeholder types), an index, a box and a prompt.
- Each slide names one layout. A block with `slot` fills that slot and takes its box from the layout unless it sets its own. Changing the layout changes every slide on it.
- A slide's block can override a theme token.
- A preset is a theme with its masters and layouts, saved and picked for a new deck. The presets in the `presentations` plugin (`extracting-presets`) are the first ones.
- **Masters from and to PowerPoint**: a `.pptx` template comes in as a preset. Its theme becomes the theme's tokens, each master its text styles, fixed blocks and footer trio, and each layout its kind and slots, with the placeholder type and index PowerPoint gave them. decks keeps the template's own master, layout and theme parts, and the pptx export starts from them, so a client template goes back out as PowerPoint wrote it. A change made in decks to a master or layout replaces only what it changes in those parts. Today the `extracting-presets` skill does this import by hand.

## 4. Animations

Part of the model from the first version, so no deck has to be rebuilt to gain them. The model follows PowerPoint's, so the pptx export keeps them as native animations.

- **Transitions** between slides: none, fade, push, wipe, zoom; a duration in ms per slide.
- **Effects** on a block, of four kinds: `in`, `emphasis`, `out`, `path`. Each effect names one entry of a closed list, and the list holds PowerPoint's preset id for each:

  | Kind | Effect | Preset id |
  |---|---|---|
  | in, out | appear | 1 |
  | in, out | fly | 2 |
  | in, out | fade | 10 |
  | in, out | wipe | 22 |
  | in, out | zoom | 23 |
  | emphasis | fill colour | 1 |
  | emphasis | font colour | 3 |
  | emphasis | grow | 6 |
  | emphasis | spin | 8 |
  | emphasis | pulse | 26 |
  | path | along an SVG path | custom |

  The ids come from LibreOffice's import table and one PowerPoint-saved file; each is checked in PowerPoint before it ships.
- **Direction**: `from` is top, right, bottom or left (PowerPoint's subtypes 1, 2, 4, 8), or in and out for zoom (16, 32).
- **Timing** as in PowerPoint: each effect starts on a `click`, `with` the effect before it, or `after` it, with a duration and a delay. A slide's clicks are its steps.
- **Parts of a block**: a text block plays `by: "paragraph"`, one paragraph per click, down to a given list level. An effect on an SVG block can target an element inside it by `target` selector, so a diagram builds part by part.
- **Playback** is a function of the step number, as in reveal.js and Slidev: the player computes each block's state for step n, so going back is as exact as going forward.
- **Script blocks** get the slide's step as an event (`deck:step`, with the step number and the direction), so custom JavaScript animation runs in time with the clicks, forwards and backwards.
- **Editor**: an animation panel per slide lists the effects in order. Drag to reorder, set start, effect, direction, duration and delay, play the slide from any step. Numbered badges on the canvas show which step each block appears on.
- **Present**: each click plays the next step; back undoes it. The presenter view's preview shows the next step, not only the next slide.

## 5. Feedback for agents

The fastest way to tell an agent what to change is to point at it. Modelled on the comments of the current deck editor (`editing-presentations`), but tied to ids instead of source lines, so a comment still finds its block after edits.

- **Comment**: select one or more blocks (or nothing, for the whole slide), type a note. The anchor is the slide id, the block ids, a pin point in fractions of the slide's width and height, and optionally an animation step.
- **Block deleted**: the thread stays, anchored to its slide at the last pin point.
- **Draw**: a pen on the slide for circles and arrows. A drawing is vector strokes owned by its comment, shown only in the editor and in screenshots, never in the deck.
- **Threads**: the agent answers under a comment and marks it done; the user reopens it if the change is wrong.
- **Selection**: what the user has selected right now is readable by the agent, so "make this bigger" needs no comment at all.
- **Copy prompt**: all open comments as one markdown prompt on the clipboard, per slide a screenshot path with each commented block boxed and numbered, and per comment the note, the block ids, their type and props. For agents without MCP.
- **MCP**: `get_selection` (ids, types, props and a screenshot with the selection boxed), `list_comments` (open ones by default, with a screenshot per slide), `reply_comment`, `resolve_comment`, `reopen_comment`.
- Comments, drawings and selection live in their own store beside the deck, never in it, and no export carries them.

## 6. Present

- Full-screen presenting in the browser.
- Presenter view with notes, next step, next slide and timer, kept in sync with the audience window, as `presenter.js` does today.

## 7. Export

- **HTML**: one self-contained file per deck; effects play through the Web Animations API.
- **PDF**: headless Chromium prints the same markup the screen shows; each slide in its final state, or one page per step.
- **pptx**: a new exporter, written from the plugin's python-pptx recipes (the plugin has recipes, not exporter code). It starts from a base file that holds the deck's masters and layouts (the imported template's own parts, or parts decks generates for a preset made in decks), then adds slides with python-pptx. A block in a slot writes an empty shape box, so it inherits its place from the layout.
  - Typed blocks become native text boxes, shapes, pictures and tables; a Chart block becomes a native chart from its table.
  - decks writes its own OOXML for what python-pptx cannot: the layouts in the base file, the theme's colour scheme, SVG (a Chromium-made PNG with the SVG beside it, as PowerPoint 2019 and later read it), transitions (ms duration with a speed fallback) and effects (`<p:timing>`).
  - HTML and Script blocks become images. An effect that targets the inside of an SVG block, and a Script block's own motion, are lost; the export lists each one it drops.
  - Chromium (Playwright) does every rasterizing. No PyMuPDF (AGPL) and no CairoSVG (LGPL), which today's plugin exporter uses.
  - Layouts and animations are checked in PowerPoint, never in LibreOffice: LibreOffice reads placeholders differently and has 27 open animation bugs.

## 8. MCP

- Remote MCP server with OAuth, as Blattomat has. MCP is the agent's only write path.
- Tools for decks, slides, blocks, animations, themes, masters, layouts and comments. Every write is an operation that names its target by id and sets single properties; no tool takes a list position.
- **Locks**: a lock asks agents to leave a block alone, and an agent may still lift it. `unlock_block` unlocks one block or every block on a slide, as an operation of its own. A write that changes a locked block without that step first is refused with a hint to unlock, also when it comes as a whole-slide or whole-deck write. The write summary names every unlock, and undo locks the block again. So an agent never moves a locked block by accident, and never has to wait for the user.
- **Versions**: every deck has a version number. A write names the version it read, and a write on an older version is refused with the current one.
- **Undo**: each agent write is one undo step for the user, and its answer carries a short summary of what changed. The server keeps a log of operations.
- `outline` returns, per slide, the title, the text and each block's id and type: a cheap read for agents, read only.
- `render_slide` returns a PNG of a slide at a given step, so the agent sees what it built.
- Each write answers with the problems it finds: overflowing text, overlapping text, text below the type floor (the hard gates H1, H2 and H7 from the `presentations` plugin), and a slide that is mostly raw blocks.
- Agent docs in the repo, in the OKF format Blattomat uses (`docs/okf/`).

## 9. Storage and accounts

- Decks saved on the server after each operation.
- A list of decks with title, date and a thumbnail.
- Login and invite-only sign-up, as in Blattomat.

## 10. Import of the existing decks

The decks the `presentations` plugin built (`~/Code/presentations`) come in from v0. v0 carries the wrap; the rebuild is an agent's job, through the MCP tools v0 already has.

- **Wrap** (v0 code): each slide of a plugin deck becomes one HTML block filling the slide, with the deck's stylesheet, assets and speaker notes. Lossless: the slide looks as it did. It can be presented, commented and searched at once, but not dragged.
- **Rebuild** (an agent, no v0 code): when the user wants to edit a wrapped slide, an agent turns it into typed blocks through MCP. An OKF recipe in the agent docs says how. Undo brings the wrapped slide back.

No converter code turns arbitrary flex or grid layout into boxes.

New decks use typed blocks first. Each write answers with a warning when a slide is mostly raw HTML, SVG or Script blocks.

## 11. Privacy

The code is public; the decks are not. Many decks hold client material.

- Every deck, upload, theme, master, comment and export lives on the VPS data volume only, as in Blattomat. Nothing of it enters this repo, a GitHub issue, a pull request, a CI log or a container image.
- Tests and fixtures use made-up decks only. No real deck, not even a slide of one, becomes a fixture or a screenshot in the repo. Research notes name no deck and no client; counts across decks are fine.
- Every page and file sits behind the login; a request for someone else's deck gets a 404. Exports and uploads are served through the app after an owner check, never from a public path.
- Feedback about the tool goes through the same privacy check as Blattomat's feedback job before anything reaches a public issue.
- Backups go with the VPS's other data (server-infra `backup/`), not to GitHub.
- Content does leave the VPS when an agent reads a deck through MCP: it goes to that agent's model provider. Which agents may connect is the user's choice per OAuth grant.

