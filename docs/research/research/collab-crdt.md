# What live collaboration needs from the model now (open question 8)

Scope: open question 8 of `docs/spec-v0.md` ("What would live collaboration need from the model now (Yjs, Automerge), so v0 does not block it?"). The spec puts live collaboration in phase 3 and states: "Slides and blocks are a list in the JSON, but every operation names them by `id`, never by their place in the list" [source: /home/pascal/Code/.worktrees/decks/research/docs/spec-v0.md | decks v0 spec | practitioner]. `docs/idea.md` adds "Every slide and block has a stable id. Nothing is addressed by its place in a list" and "An agent changes a deck only through the same operations the editor uses, so it can later join a live session as one more peer" [source: /home/pascal/Code/.worktrees/decks/research/docs/idea.md | decks idea | practitioner].

Model facts from the spec that later sections set evidence against: slides and blocks are JSON arrays with `id`; each block has an integer `z` (`"z": 1`, `"z": 2`); `animations` is an array "in play order" whose items name a block by id; text is `props.text`; themes and masters are separate documents named by id; comments live beside the deck [source: /home/pascal/Code/.worktrees/decks/research/docs/spec-v0.md | decks v0 spec | practitioner].

Release state checked on 2026-10-20 through the GitHub releases API: Yjs `v14.0.0-rc.29` (2026-10-15) and `v13.6.33` (2026-09-23); Automerge `js/automerge-3.5.0` (2026-09-16); Loro `loro-crdt@1.16.4` (2026-09-30) [source: https://api.github.com/repos/yjs/yjs/releases , https://api.github.com/repos/automerge/automerge/releases , https://api.github.com/repos/loro-dev/loro/releases | GitHub releases API | vendor].

---

## SQ1: How Yjs, Automerge and Loro model JSON documents, their licenses, and their known costs

### Yjs

Data model. Yjs offers shared types (Y.Map, Y.Array, Y.Text, Y.XmlFragment) that nest: "Nested types can be included as content into any other shared type." A Y.Map value "can be a shared type, an Uint8Array, or anything JSON-encodable" [source: https://docs.yjs.dev/api/shared-types/y.map | Y.Map docs | vendor]. The Y.Map page does not state how concurrent sets on one key resolve [source: same | vendor].

Map conflict rule, from the internals document: "Maps are lists of entries. The last inserted entry for each key is used, and all other duplicates for each key are flagged as deleted." Each client gets "a random 53-bit integer" clientID [source: https://github.com/yjs/yjs/blob/main/INTERNALS.md | Yjs INTERNALS.md | vendor].

Algorithm. Yjs is "an adaptation of the YATA CRDT with improved runtime performance" [source: https://docs.yjs.dev/api/internals | Yjs internals page | vendor].

Deletes and garbage collection: "Yjs does not record metadata about a deletion: No data is kept on *when* an item was deleted, or which user deleted it." "If garbage collection is enabled in Yjs, when an object is deleted its content is discarded. If a deleted object contains children ... the content is replaced with a `GC` object ... it only stores the length of the removed content." On delete-set size: "The real world editing trace from the B4 benchmark document contains 182k inserts and 77k deleted characters. The deleted set size in a snapshot is only 4.5Kb" [source: https://github.com/yjs/yjs/blob/main/INTERNALS.md | Yjs INTERNALS.md | vendor].

Known cost: Y.Map with many keys or alternating writes. The y-utility README (Yjs org): "Y.Map doesn't make use of Yjs' optimizations if you write key-value entries in alternating order. Always writing the same entry does't significantly increase the size of the document. But writing key1, then key2, then key1, then key2 (alternating order) breaks Yjs' optimization." "Y.Map needs to retain all key values that were created in history to resolve potential conflicts. This makes Y.Map unsuitable as a key-value store." Benchmark table in the same README: 100k operations on 10 keys give 271 bytes with YKeyValue against 524,985 bytes with Y.Map (JSON size 121 bytes); 100k operations on 1000 keys give 30,017 against 593,834 bytes; 500k operations on 10 keys give 329 against 2,684,482 bytes [source: https://cdn.jsdelivr.net/npm/y-utility@0.1.4/README.md | y-utility README, YKeyValue | vendor]. Liveblocks repeats the 524KB to 271 bytes figure and recommends YKeyValue for "frequently updating alternating entries" [source: https://liveblocks.io/docs/guides/yjs-best-practices-and-tips | Yjs best practices and tips | vendor].

Kevin Jahns (dmonad, Yjs author) on a map that gains and loses keys: "The metadata only grows because you add a new key in each iteration. The current garbage collection approach doesn't work in this case ... Rather, you should store each entry in a Y.Array. E.g. yarray.insert(i, [{ .. }]) instead of ymap.set('key-${i}', {..})." User braden in the same thread: "YMaps are better suited to modeling domain objects with an upper bound on number of keys, rather than collections." (2021-04-22) [source: https://discuss.yjs.dev/t/map-metadata-overhead/492 | Map metadata overhead | practitioner].

Field report: a map "receives key updates every second or so and it's rapidly becoming a 6mb document"; later "like 30MB per day"; the poster moved that data "to a last-write-wins approach outside Yjs" (tobowers, February 2025). No maintainer replied in the thread [source: https://discuss.yjs.dev/t/document-growing-unbounded/3729 | Document growing unbounded | practitioner].

Yjs 14 (pre-release). A FOSDEM 2026 talk names "changesets and attributions" for versioning and track changes as Yjs 14 features [source: https://fosdem.org/schedule/event/8VKQXR-blocknote-yjs-prosemirror | BlockNote, Prosemirror and Yjs 14 | practitioner]. A 14.0.0 pre-release note says it enables single-range moves in Y.Array and that "v14 can read v13 updates, but v13 can't read v14 updates (if content has ever been moved)" [source: https://newreleases.io/project/yarn/yjs/release/14.0.0-0 | yjs 14.0.0-0 release | vendor]. Not checked against a final 14.0 changelog: none exists as of 2026-10-20 (latest tag is rc.29).

Rich text binding. y-prosemirror 1.x: "This binding maps a Y.XmlFragment to the ProseMirror state"; the README example takes `ydocument.get('prosemirror', Y.XmlFragment)` and passes it to `ySyncPlugin(type)`; it also ships converters `prosemirrorJSONToYDoc`, `yDocToProsemirrorJSON` [source: https://github.com/yjs/y-prosemirror/blob/v1.3.7/README.md | y-prosemirror README v1.3.7 | vendor]. The master README documents the unstable `@y/prosemirror` for Yjs 14, bound to a `Y.Node`: "The Yjs type is the source of truth"; "Undo and redo are scoped to the local client, so one peer won't undo another's changes"; "Create one `syncPlugin()` per editor" [source: https://github.com/yjs/y-prosemirror | y-prosemirror README (master) | vendor]. The README does not address one fragment per text block in a canvas of many blocks.

Python side. pycrdt "provides bindings for Yrs, the Rust port of the Yjs framework" [source: https://github.com/y-crdt/pycrdt/blob/main/docs/index.md | pycrdt docs | vendor]; its package exports `Doc, Map, Array, Text, XmlFragment, XmlElement, UndoManager, Awareness` [source: https://github.com/y-crdt/pycrdt/blob/main/python/pycrdt/__init__.py | pycrdt __init__.py | vendor].

### Automerge

Data model: "Automerge documents are composed of nested maps and lists or simple values or text sequences." Map rules include: "If `A` deletes key x and `B` sets x to a new value then set the value of x to the new value `B` set." List rules: "Every element in a list has an ID and operations on the list reference these IDs ... When you _insert_ elements into a list the insert operation references the ID of the element you are inserting after." Text "merged using the same logic as lists", marks per Peritext [source: https://github.com/automerge/automerge.github.io/blob/main/content/docs/reference/under-the-hood/merge-rules.md | Automerge merge rules | vendor]. (The rendered page at automerge.org/docs/reference/under-the-hood/merge_rules/ returned 404; the content was read from the docs repo.)

Conflicts: "Automerge picks one of the concurrently written values as the 'winner' ... the other values are not lost. They are merely relegated to a conflicts object", readable through `Automerge.getConflicts(doc, 'foo')`; "'last writer wins' here is based on the internal ID of the operation, not a wall clock time"; ordered "based on the counter first", actorId breaks ties [source: https://automerge.org/docs/reference/documents/conflicts/ | Automerge conflicts | vendor].

Move: the Automerge lists reference page has no move operation (a search of `reference/documents/lists.md` for "move" returns nothing) [source: https://github.com/automerge/automerge.github.io/blob/main/content/docs/reference/documents/lists.md | Automerge lists reference | vendor].

History and size: "Automerge documents hold their entire change histories ... we strongly suggest you do your own measurements." On truncating history: Kleppmann's experiments show "The savings are not all that great, which is why we haven't prioritised history truncation so far." On granularity: "an Automerge document is best suited to being a unit of collaboration between two people or a small group"; PushPin used very granular documents and "the overhead of syncing many thousands of documents was high" [source: https://github.com/automerge/automerge.github.io/blob/main/content/docs/cookbook/modeling-data.md | Automerge: Modeling Data | vendor].

Schema migration: two devices each running `Automerge.change()` to set up a schema "will not work, because you now have two different documents with no shared ancestry"; migrations are "more difficult than migrations in a centralized relational database, because it could happen that two users independently perform the same migration"; the suggested workaround is to hard-code the initial change and every migration as byte arrays [source: same | vendor].

Automerge 3 (July 2025): "we've cut that down memory usage by over 10x, sometimes dramatically more"; "pasting Moby Dick into an Automerge 2 document consumes 700Mb of memory, in Automerge 3 it only consumes 1.3Mb"; "a document which hadn't loaded after 17 hours loading in 9 seconds"; the library now "uses the compressed representation at runtime"; file format unchanged from Automerge 2 [source: https://automerge.org/blog/automerge-3/ | Automerge 3.0 | vendor].

Python side: automerge-py exists, copyright "2024 Salesforce, Inc.", MIT [source: https://github.com/automerge/automerge-py/blob/main/LICENSE | automerge-py LICENSE | vendor].

### Loro

Containers: Text (Fugue), rich text, "Moveable Tree", "Moveable List", "Last-Write-Wins Map", counter; "Fast Time Travel Through History"; "Version Control with Real-Time Collaboration"; "Shallow Snapshot that Works like Git Shallow Clone" [source: https://github.com/loro-dev/loro/blob/main/README.md | Loro README | vendor]. (loro.dev pages returned HTTP 403 to WebFetch [inaccessible]; the docs were read from the `loro-dev/loro-docs` repo below.)

Map: "Loro's Map uses LWW (Last-Write-Wins) semantics. When concurrent edits conflict, it compares Lamport logic timestamps to determine the winner." A child container created at the same key by two peers can become "two different children" unless a "mergeable child container" is used [source: https://github.com/loro-dev/loro-docs/blob/main/pages/docs/tutorial/map.mdx | Loro Map | vendor].

List vs MovableList: "if the same element is set or moved concurrently, the simulation would result in the deletion of the original element and the insertion of two new elements"; MovableList fixes this but "when performing only insertions/deletions, the `MovableList` is approximately 80% slower in encode/decode and consumes about 50% more memory compared to the `List`." MovableList uses the algorithm of Kleppmann's "Moving Elements in List CRDTs" [source: https://github.com/loro-dev/loro-docs/blob/main/pages/docs/tutorial/list.mdx | Loro List and Movable List | vendor].

Tree: Loro's tree follows Kleppmann et al., "A highly-available move operation for replicated trees", and orders siblings with a fractional index; "Fractional Index has an interleaving issue, but we believe this is acceptable for tree structures" [source: https://github.com/loro-dev/loro-docs/blob/main/pages/docs/tutorial/tree.mdx | Loro Tree | vendor].

Loro 1.0: "10-100x faster document import"; "reduced the loading time for a document with millions of operations from 16ms to 1ms", 0.37ms with a shallow snapshot [source: https://github.com/loro-dev/loro-docs/blob/main/pages/blog/v1.0.mdx | Loro 1.0 announcement | vendor].

### Independent benchmark (dated)

dmonad/crdt-benchmarks tables use Yjs 13.6.11, ywasm 0.9.3, Loro 0.10.1, Automerge 2.1.10, on an Intel i5-8400 with Node 20.5.0; no run date or repetition count given [source: https://github.com/dmonad/crdt-benchmarks | crdt-benchmarks | practitioner]. These versions predate Automerge 3 (July 2025) and Loro 1.0, so the Automerge and Loro numbers below describe older engines. The maintainer of the benchmark is the Yjs author.

- B4 real-world editing trace, docSize / parseTime: Yjs 159,929 B / 39 ms; ywasm 159,929 B / 16 ms; Loro 258,228 B / 13 ms; Automerge 129,116 B / 1,805 ms [source: https://github.com/dmonad/crdt-benchmarks | crdt-benchmarks | practitioner].
- B3 maps with 20√N concurrent clients, docSize: "Set number in Map" Yjs 32,225 B, Loro 21,506 B, Automerge 86,167 B; "Set Object in Map" Yjs 32,235 B, Loro 40,494 B, Automerge 112,570 B; "Set String in Map" Yjs 38,357 B, Loro 7,798,572 B, Automerge 98,047 B [source: same | practitioner].
- Caveats stated by the page: `memUsed` excludes wasm memory; Loro snapshots are on by default, which lowers its parse time; parse time "is significantly higher for Automerge and Loro when the initial document is not empty"; B4 applies all edits in one transaction because Automerge otherwise runs out of memory [source: same | practitioner].

No benchmark found that models a slide deck (hundreds of small objects with frequent property writes during drag). [author estimate: a deck of 30 slides with 10 to 20 blocks each holds roughly 300 to 600 block objects, so the "many keys" and "alternating writes" Yjs findings above are the relevant ones, not the text traces.]

### Licenses (read from the LICENSE files via `gh api repos/<o>/<r>/license`)

| Project | File | Operative text |
|---|---|---|
| Yjs | https://github.com/yjs/yjs/blob/main/LICENSE | "The MIT License (MIT) Copyright (c) 2023 - Kevin Jahns ... - Chair of Computer Science 5 (Databases & Information Systems), RWTH Aachen University, Germany. Permission is hereby granted, free of charge ... to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies". GitHub reports `NOASSERTION` because of the two-holder header and a repeated warranty paragraph; the text is MIT. |
| y-prosemirror | https://github.com/yjs/y-prosemirror/blob/master/LICENSE | "The MIT License (MIT) Copyright (c) 2019 Kevin Jahns" |
| y-websocket | https://github.com/yjs/y-websocket/blob/master/LICENSE | "The MIT License (MIT) Copyright (c) 2025 Kevin Jahns" |
| y-utility (YKeyValue) | https://github.com/yjs/y-utility/blob/main/LICENSE | "The MIT License (MIT) Copyright (c) 2022 Kevin Jahns" |
| y-crdt (Yrs) | https://github.com/y-crdt/y-crdt/blob/main/LICENSE | "The MIT License (MIT) Copyright (c) 2020 - Bartosz Sypytkowski" (GitHub: `NOASSERTION`, text is MIT) |
| pycrdt | https://github.com/y-crdt/pycrdt/blob/main/LICENSE | "The MIT License (MIT) Copyright (c) 2023 David Brochart" |
| pycrdt-websocket | https://github.com/y-crdt/pycrdt-websocket/blob/main/LICENSE | "MIT License Copyright (c) 2022 David Brochart" |
| Hocuspocus | https://github.com/ueberdosis/hocuspocus/blob/main/LICENSE.md | "MIT License Copyright (c) 2023, Tiptap GmbH" |
| y-sweet | https://github.com/jamsocket/y-sweet/blob/main/LICENSE | "MIT License Copyright (c) 2023 Drifting in Space Corp." plus "Additionally, this repository contains code adapted from y-crdt, which is licensed as follows: The MIT License (MIT)" (GitHub: `NOASSERTION`) |
| Automerge | https://github.com/automerge/automerge/blob/main/LICENSE | "Copyright (c) 2019-2021 the Automerge contributors. Permission is hereby granted, free of charge ..." (MIT text) |
| automerge-repo | https://github.com/automerge/automerge-repo/blob/main/LICENSE | "Copyright (c) 2019-2023, Ink & Switch LLC. Permission is hereby granted, free of charge ..." (MIT text) |
| automerge-py | https://github.com/automerge/automerge-py/blob/main/LICENSE | "Copyright (c) 2024 Salesforce, Inc. Permission is hereby granted, free of charge ..." (MIT text) |
| Loro | https://github.com/loro-dev/loro/blob/main/LICENSE | "MIT License Copyright (c) 2023 Loro" |
| fractional-indexing (rocicorp) | https://github.com/rocicorp/fractional-indexing/blob/main/LICENSE | "Creative Commons Legal Code CC0 1.0 Universal" |
| Excalidraw | https://github.com/excalidraw/excalidraw/blob/master/LICENSE | "MIT License Copyright (c) 2020 Excalidraw" |
| @tldraw/store | https://github.com/tldraw/tldraw/blob/main/packages/store/LICENSE.md | "MIT License Copyright (c) 2024 tldraw Inc."; package.json `"license": "MIT"` |
| @tldraw/sync-core, @tldraw/sync | https://github.com/tldraw/tldraw/blob/main/packages/sync-core/LICENSE.md | "This code is licensed under the [tldraw license]"; package.json `"license": "SEE LICENSE IN LICENSE.md"` |
| tldraw (root) | https://github.com/tldraw/tldraw/blob/main/LICENSE.md | "Use the Software in Development Environments"; conditions: "Not to use the Software in Production Environments." "Not to disable, change, or interfere with the Software's License Key enforcement." "The Software may collect and transmit usage data to tldraw for license compliance purposes." |

tldraw's own blog on the license: "Since tldraw SDK 4.0 (September 2025), the SDK ships under the tldraw license." "Production use of the tldraw SDK requires a license key", either "a commercial license for commercial use, or a free hobby license for non-commercial use"; the watermark "appears with the free hobby license" [source: https://tldraw.dev/blog/20-things-i-wish-ai-chatbots-knew-about-tldraw | 20 things I wish AI chatbots knew about tldraw | vendor]. The root LICENSE.md itself mentions only trial and commercial licenses, not a hobby license [source: https://github.com/tldraw/tldraw/blob/main/LICENSE.md | tldraw license | vendor].

---

## SQ2: Model choices that make later CRDT adoption easy or hard

### Stable ids and objects in a map keyed by id

Figma: "Each object has an ID and a collection of properties with values", a "two-level map: `Map<ObjectID, Map<Property, Value>>`". Ids are made by clients: Figma "relies on clients being able to generate new object IDs that are guaranteed to be unique"; "we can't solve this by having the server assign IDs to newly-created objects" [source: https://www.figma.com/blog/how-figmas-multiplayer-technology-works/ | How Figma's multiplayer technology works | vendor].

Jamsocket (Paul Butler, 2022-12-05), on index-based operations: "'item 2' refers to the item you just added, not the one I actually checked. Our lists have diverged." "A typical solution in this case (applicable to both paths) is to generate a random UUID for each item when it is created" [source: https://web.archive.org/web/2024/https://jamsocket.com/blog/you-might-not-need-a-crdt | You might not need a CRDT | vendor]. (The live URL now redirects to a Modal acquisition post; read from the Wayback copy.)

tldraw: every record has an `id`; the sync storage contract "asserts `id === record.id`" on set; a network diff is "a compact, one-way map of record id to `put`/`patch`/`remove` ops" [source: https://github.com/tldraw/tldraw/blob/main/packages/sync-core/SPEC.md | @tldraw/sync-core behavior specification | vendor].

Excalidraw merges peers' scenes by comparing "all the `ExcalidrawElement.id`s it has locally, and all of the incoming `ExcalidrawElement.id`s" [source: https://plus.excalidraw.com/blog/building-excalidraw-p2p-collaboration-feature | Building Excalidraw's P2P collaboration feature | vendor].

Automerge refs can select a list element by a field instead of an index: `handle.ref("users", { id: "b" })`; "The Refs API is currently experimental" [source: https://github.com/automerge/automerge.github.io/blob/main/content/docs/reference/repositories/refs.md | Automerge Deep References | vendor].

Yjs collections: the Yjs author advises storing collection entries in a Y.Array rather than as Y.Map keys (see SQ1) [source: https://discuss.yjs.dev/t/map-metadata-overhead/492 | Map metadata overhead | practitioner]. The Liveblocks tldraw-on-Yjs example stores each record whole under its id in a YKeyValue over a Y.Array: `yDoc.getArray("tl_records")`, `new YKeyValue(yArr)`, `yStore.set(record.id, record)`, `yStore.delete(record.id)` [source: https://github.com/liveblocks/liveblocks/blob/main/examples/nextjs-tldraw-whiteboard-yjs/src/components/useYjsStore.ts | Liveblocks tldraw Yjs example | vendor]. In that mapping a concurrent write to two properties of one record keeps one whole record, not both properties (record-level, not property-level, last writer wins) [author estimate: follows from YKeyValue storing `val: TLRecord` as one value per key].

### Order: fractional indexing vs ordered arrays vs move operations

Figma: "the order of the children for an element of the tree is determined by sorting all children by their index"; "every index is a fraction between 0 and 1 exclusive"; insert by setting "the index for the new object to the average index of the two objects on either side"; "arbitrary-precision fractions instead of 64-bit doubles"; "Each index is stored as a string ... base 95 instead of base 10." On a tie: the server "can avoid ever having two objects with an identical position" by "generating and assigning a unique position to the second insert operation" [source: https://www.figma.com/blog/realtime-editing-of-ordered-sequences/ | Realtime editing of ordered sequences | vendor].

Figma parent links: "storing a link to the parent as a property on the child"; "the parent link and this position must both be stored as a single property so they update atomically"; servers "reject parent property updates that would cause a cycle" [source: https://www.figma.com/blog/how-figmas-multiplayer-technology-works/ | How Figma's multiplayer technology works | vendor].

Evan Wallace (Figma co-founder): "If two peers both simultaneously insert a run of objects at the same location, the resulting objects may be interleaved"; fix: "add a random offset to the end of the fraction during each insert operation"; limitation: "Index length can become long in pathological scenarios" [source: https://madebyevan.com/algos/crdt-fractional-indexing/ | CRDT: Fractional Indexing | practitioner].

tldraw gives each shape an `index` property that sets z-order with string-based fractional indexing [source: https://tldraw.dev/sdk-features/shape-indexing | tldraw Shape indexing | vendor] (read through search snippet only).

Excalidraw retrofit. PR #7359 "feat: fractional indexing", merged 2024-04-04: "Potentially conflicting indices are ordered by `element.id` and later on deduplicated." Invariant: "after each update is completed, for every element in the elements array, its fractional index is strictly smaller than the next element's." Design: the "Array ... should be used as a cache of elements order"; "it's necessary to always keep the array support for backwards compatibility (restore) - old scenes, old libraries"; "re-order based on fractional indices should be part of (multiplayer) operations such as reconciliation & undo/redo" [source: https://github.com/excalidraw/excalidraw/pull/7359 | Excalidraw PR 7359 | vendor]. Follow-up work on the same feature continued into 2026: PR #11258 "fix: Fractional index validation" merged 2026-05-04, PR #11934 "fix: don't re-version remote elements when healing fractional indices" open [source: https://github.com/excalidraw/excalidraw/pulls?q=fractional | Excalidraw PR search | vendor]. Before the retrofit, the 2020 collaboration post says the element array was kept "in Z-index order" with no fractional index [source: https://plus.excalidraw.com/blog/building-excalidraw-p2p-collaboration-feature | Building Excalidraw's P2P collaboration feature | vendor].

Kleppmann, "Moving Elements in List CRDTs" (PaPoC 2020): if an element is moved by delete and re-insert on two replicas, "the result is that the element is duplicated"; "duplicating the moved element at both destinations is unlikely to be desirable behaviour in most applications"; the proposal: "we can use a long-established CRDT: a last-writer wins (LWW) register. We need one such register for each list element, containing the position of that element. Moving an element to a new location is then merely a matter of updating the value of that element's register." Moving ranges is called "the open problem" [source: https://martin.kleppmann.com/papers/list-move-papoc20.pdf | Moving Elements in List CRDTs | independent].

Juxtaposed spec fact: the decks spec stores block order in the `blocks` array and stacking as integer `z`, and slide order as the `slides` array [source: /home/pascal/Code/.worktrees/decks/research/docs/spec-v0.md | decks v0 spec | practitioner]. Figma, tldraw and Excalidraw (since 2024) store order as a fractional-index string on each object (sources above).

### Last-writer-wins per property

Figma: the server keeps "the latest value that any client has sent for a given property on a given object"; "the document will just end up with the last value that was sent to the server" [source: https://www.figma.com/blog/how-figmas-multiplayer-technology-works/ | How Figma's multiplayer technology works | vendor].

tldraw diffs: "`props` and `meta` are the only nested keys at the top level: changes inside them are expressed as `['patch', ...]` ops. Any other top-level key ... is compared with deep equality and produces a whole-value `['put', next]` on change"; same-length arrays get index patches only if "at most `max(length/5, 1)` items changed", otherwise "the whole array is put"; a string that grows by suffix becomes `['append', ...]` [source: https://github.com/tldraw/tldraw/blob/main/packages/sync-core/SPEC.md | @tldraw/sync-core spec, rules D3, D5, D6 | vendor].

Excalidraw: per element, not per property: a peer "increments the version number of that element before sending"; on merge "we throw out old versions of each element and just keep the latest ones"; ties broken "by favoring the one with the lower `versionNonce`" [source: https://plus.excalidraw.com/blog/building-excalidraw-p2p-collaboration-feature | Building Excalidraw's P2P collaboration feature | vendor].

Automerge, Loro and Yjs maps are LWW per key (sources in SQ1). Automerge also keeps the losing values readable through `getConflicts` [source: https://automerge.org/docs/reference/documents/conflicts/ | Automerge conflicts | vendor].

### Rich text as its own CRDT type

Google Docs (2010) models text as operations: "all edits boil down to three basic types of changes: inserting text, deleting text, and applying styles to a range of text"; "{ApplyStyle bold @10-20} transformed against {InsertText 'ABC' @15} results in {ApplyStyle Bold @10-23}" [source: https://drive.googleblog.com/2010/09/whats-different-about-new-google-docs_22.html | What's different about the new Google Docs: Conflict resolution | vendor].

tldraw stores text inside record props and diffs strings with an `append` op only when "`next` starts with `prev`"; "Other string changes are puts" [source: https://github.com/tldraw/tldraw/blob/main/packages/sync-core/SPEC.md | @tldraw/sync-core spec, rule D5 | vendor].

Yjs binds ProseMirror to a Y.XmlFragment (y-prosemirror 1.x) and offers `prosemirrorJSONToYDoc` for conversion [source: https://github.com/yjs/y-prosemirror/blob/v1.3.7/README.md | y-prosemirror README v1.3.7 | vendor]. Automerge has a ProseMirror cookbook (`cookbook/rich-text-prosemirror-react.md`, `rich-text-prosemirror-vanilla.md`) and a rich text schema page [source: https://github.com/automerge/automerge.github.io/tree/main/content/docs/cookbook | Automerge docs tree | vendor]. Loro lists a "Rich Text CRDT" [source: https://github.com/loro-dev/loro/blob/main/README.md | Loro README | vendor].

Linear: a third-party reverse engineering of Linear's sync engine describes server-side transactions with LWW for models [source: https://github.com/wzhudev/reverse-linear-sync-engine | Reverse Linear Sync Engine | practitioner]. A search summary said Linear uses CRDTs for issue descriptions; that line was not found in the wzhudev README and is not verified here.

Juxtaposed facts: Blattomat stores text as its own JSON (`TextProps` with `Para` and `Run` definitions) [source: /home/pascal/Code/blattwerk/src/blattwerk/sheet.schema.json | Blattomat sheet schema | practitioner]; the decks spec shows `"props": { "text": "Why decks as code" }` and asks for formatting on part of the text and lists with levels [source: /home/pascal/Code/.worktrees/decks/research/docs/spec-v0.md | decks v0 spec | practitioner].

### Derived data, invariants, ephemeral state

Jamsocket on invariants: "two innocent changes made concurrently on two replicas could combine to break the invariant"; with a global order "The authoritative server can apply the change locally to detect if invariants are violated. If they are, instead of broadcasting the change, it can notify the sender that there is a conflict" [source: https://web.archive.org/web/2024/https://jamsocket.com/blog/you-might-not-need-a-crdt | You might not need a CRDT | vendor].

Loro: "CRDTs merge; they do not reject"; poor fits include "Uniqueness constraints", "Authorization decisions that must be enforced at write time", "Global invariants (referential integrity, cross-document totals)" [source: https://github.com/loro-dev/loro-docs/blob/main/pages/docs/concepts/when_not_crdt.mdx | When Not to Use CRDTs | vendor].

tldraw keeps presence out of the document: "Presence records (cursors etc.) live in the room's in-memory `PresenceStore`, never in document storage" [source: https://github.com/tldraw/tldraw/blob/main/packages/sync-core/SPEC.md | @tldraw/sync-core spec | vendor]. Yjs advice for cursor positions: store them in Awareness rather than the Doc (search summary of discuss.yjs.dev; thread not opened) [source: https://discuss.yjs.dev/t/document-growing-unbounded/3729 | discuss.yjs.dev | practitioner].

Excalidraw deletes by tombstone: "a new field ... called `isDeleted`"; "the size of your list grows forever"; "we do remove any element with `isDeleted` set to true when saving to persistent storage" [source: https://plus.excalidraw.com/blog/building-excalidraw-p2p-collaboration-feature | Building Excalidraw's P2P collaboration feature | vendor].

Juxtaposed spec fact: the decks spec says "deleting a block deletes" its animations [source: /home/pascal/Code/.worktrees/decks/research/docs/spec-v0.md | decks v0 spec | practitioner]; that is a cascade across two collections (blocks and the slide's `animations` list), the kind of cross-object rule the Loro and Jamsocket passages above discuss.

### Schema evolution

Edwards, Petříček and van der Storm (2023): "Convergent Replicated Data Types (CRDTs) are used to coordinate data changes peer-to-peer. Unfortunately these techniques so far have not addressed schema change nor code deployment" [source: https://arxiv.org/pdf/2309.11406 | Live & Local Schema Change: Challenge Problems | independent] (quote from search snippet; PDF not opened). Cambria (Litt, van Hardenberg, Henry, PaPoC 2021) translates between schema versions with bidirectional lenses [source: https://www.inkandswitch.com/cambria/ | Project Cambria | independent]. Automerge's own docs point to Cambria and say it is "not yet implemented in Automerge" [source: https://github.com/automerge/automerge.github.io/blob/main/content/docs/cookbook/modeling-data.md | Automerge: Modeling Data | vendor]. tldraw sync "runs migrations to make sure clients of different versions can collaborate", and "We don't guarantee server backwards compatibility forever" [source: https://tldraw.dev/docs/sync | tldraw sync docs | vendor].

---

## SQ3: How tldraw, Excalidraw, Figma, Pitch and Google Slides do multiplayer, and whether a server-authoritative operation log works without a CRDT

### Figma

"Figma isn't using true CRDTs though." "CRDTs are designed for decentralized systems where there is no single central authority"; "Since Figma is centralized (our server is the central authority)"; "the server is the ultimate authority on what the document looks like"; OT was "unnecessarily complex for our problem space" [source: https://www.figma.com/blog/how-figmas-multiplayer-technology-works/ | How Figma's multiplayer technology works | vendor]. Undo: after undo, copy, and redo back to the present, "the document should not change"; "an undo operation modifies redo history at the time of the undo" [source: same | vendor].

Reliability (Darren Tsung, 2022-10-20): one in-memory process per file ("all clients that open a particular file get connected to the multiplayer instance that has the file open"); checkpoints "every 30 to 60 seconds"; a write-ahead journal with the goal "<1s of data loss"; "95% of edits to a Figma file get saved within 600ms"; "clients send updates every 33ms (30 FPS)"; "the journal handles >2.2B received changes per day" [source: https://www.figma.com/blog/making-multiplayer-more-reliable/ | Making multiplayer more reliable | vendor].

### tldraw sync

"tldraw sync is deliberately not a CRDT." "In tldraw sync, the server keeps the authoritative document and reconciles diffs from clients." "We chose that design because general-purpose CRDTs aren't built for canvas data" [source: https://tldraw.dev/blog/20-things-i-wish-ai-chatbots-knew-about-tldraw | 20 things I wish AI chatbots knew about tldraw | vendor]. Announced 2024-08-05; "designed ... to be self-hosted" [source: https://tldraw.dev/blog/announcing-tldraw-sync | Announcing tldraw sync | vendor]. `TLSocketRoom` "stores the authoritative copy of the document, relays changes between sync clients over WebSockets"; one server instance per room, else "users won't see each other and will overwrite others' changes"; auth is left to the app, with `authorizeRecord` for per-record rules [source: https://tldraw.dev/docs/sync | tldraw sync docs | vendor]. "the tldraw SDK works with any real-time data backend", e.g. Liveblocks [source: https://tldraw.dev/docs/collaboration | tldraw collaboration docs | vendor].

Protocol, from the sync-core spec:
- "The **document clock** is a monotonically increasing integer owned by the room's storage. It advances at most once per storage transaction that writes." A tombstone "records the deletion of a document id at a clock value" [source: https://github.com/tldraw/tldraw/blob/main/packages/sync-core/SPEC.md | @tldraw/sync-core spec §1 | vendor].
- Push results: "`commit` (applied verbatim), `discard` (no effect), or `rebaseWithDiff` (applied with modifications)" [source: same §1 | vendor].
- Client rebase: "speculative changes are reverted ..., buffered server events are applied, then any still-pending and unsent local changes are re-applied on top" (CR2) [source: same §21 | vendor].
- Server: "A `patch` for a missing (e.g. concurrently deleted) record is silently ignored" (RP4); records failing schema validation reject the session with `INVALID_RECORD` (RP2); "Applied changes are broadcast as a `patch` to every other connected session, never echoed to the pusher" (RP8) [source: same §24 | vendor].
- Catch-up: `getChangesSince(c)` returns documents with `lastChangedClock > c` and tombstones `> c`; older clients get a full wipe; tombstones are pruned above `MAX_TOMBSTONES` (5000) (SS15, SS18) [source: same §10 | vendor].
- Storage: `InMemorySyncStorage` and `SQLiteSyncStorage`; "Document state is stored as JSON-encoded BLOBs" (SQ6) [source: same §12 | vendor]. SDK 4.3 added SQLite-backed sync storage [source: https://tldraw.dev/blog/tldraw-sdk-4.3 | tldraw SDK 4.3 | vendor] (search snippet).
- License: `@tldraw/sync-core` and `@tldraw/sync` are under the tldraw license (see SQ1 table); `@tldraw/store` is MIT.

### Excalidraw

"a pseudo-P2P model, where a central server relays end-to-end encrypted messages to all the peers in the room, but does no centralized coordination"; "We did not want to store anything server-side." Merge by element `version`, ties by `versionNonce` [source: https://plus.excalidraw.com/blog/building-excalidraw-p2p-collaboration-feature | Building Excalidraw's P2P collaboration feature | vendor]. Fractional indices added in 2024 (PR #7359) and "multiplayer undo / redo" in PR #7348 merged 2024-04-17 [source: https://github.com/excalidraw/excalidraw/pull/7348 | Excalidraw PR 7348 | vendor] (title and merge date only).

### Google Slides

The Google Docs editors use OT through a central server: "Collaboration in Google Docs consists of sending changes from one editor to the server, and then to the other editors. Each editor transforms incoming changes so that they make sense relative to the local version of the document." Documents are saved "as a revision log consisting of a list of these changes"; "To display a document, we replay the revision log from the beginning" (John Day-Richter, 2010-09-22). The post is about Docs text, not Slides [source: https://drive.googleblog.com/2010/09/whats-different-about-new-google-docs_22.html | What's different about the new Google Docs: Conflict resolution | vendor]. No Google source describing the Slides editor's own collaboration model was found.

Slides API writes: `batchUpdate` is atomic: "If any request is not valid, then the entire request will fail and nothing will be applied"; "the updates in your request are guaranteed to be applied together atomically." `writeControl.requiredRevisionId`: if it does not match, "the request is not processed and returns a 400 bad request error." Without it, "your changes may be altered with respect to collaborator changes" [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/batchUpdate | presentations.batchUpdate | vendor].

### Pitch

No usable sources for Pitch's sync architecture. Searches tried: "Pitch.com engineering real-time collaboration sync Clojure", "pitch.com blog engineering multiplayer architecture", "Pitch ... Clojure datascript sync", "Pitch help center collaborate in real time". Found only: Pitch is "written mostly with Clojure" and focuses on "real-time collaboration" [source: https://www.git-tower.com/blog/developing-for-the-desktop-pitch/ | Pitch: Developing a Collaborative Presentation Tool | vendor]; on the 2020 launch thread, non-employees inferred transport only ("it looks like they are using pusher.com", user knes; video "WebRTC using Daily", user fancy_pantser) [source: https://news.ycombinator.com/item?id=24836676 | Pitch launch, Hacker News | practitioner].

### Linear (server-ordered transactions, for comparison)

Per a third-party study (author "not affiliated with the Linear team"): client operations are "encapsulated as transactions" that "execute exclusively on the server" and are "reversible on the client in case of failure"; offline they are "cached in IndexedDB and automatically resent"; the server broadcasts "delta packets", each action with a "sync id"; "Linear's collaboration model aligns more closely with OT, as it relies on a centralized server to establish the order of all transactions" [source: https://github.com/wzhudev/reverse-linear-sync-engine | Reverse Linear Sync Engine | practitioner].

### Blattomat (the code decks ports from)

Blattomat has no operation log. It saves the whole sheet document with optimistic concurrency:
- Server: `PATCH /api/sheets/{sheet_id}` takes `doc` and the `version` it was based on ("a doc needs the version it was based on"); SQL `UPDATE sheets SET ... version = version + (?2 IS NOT NULL) ... WHERE id = ?3 AND user_id = ?4 AND (?2 IS NULL OR version = ?5)`; no row means "it was saved from somewhere else in the meantime" and the server raises `HTTPException(409)` [source: /home/pascal/Code/blattwerk/src/blattwerk/sheets.py | Blattomat sheets.py | practitioner].
- Client: `frontend/src/saves.ts` keeps "The version every save is based on" and on a 409 checks whether the server holds this browser's own unanswered save ("then it holds a document sent from here, and saving goes on from its version"), else marks a clash [source: /home/pascal/Code/blattwerk/frontend/src/saves.ts | Blattomat saves.ts | practitioner].
- Agent rule: "Every write takes the version you have and answers the new one"; fine tools (`add_block`, `update_block`, `move_block`, `delete_block`, ...) "If its version is not the one you passed, they write nothing and say so"; `replace_doc` retries once only when "the version moved, the content did not"; "Never write an old doc back over the teacher's change" [source: /home/pascal/Code/blattwerk/docs/okf/versions.md | Versions and the 409 rule | practitioner].

### Current presentations plugin

The deck editor in `editing-presentations` watches deck files by modification time (`watched = {DECK: DECK.stat().st_mtime_ns}`) and answers 409 when "nothing is selected in the editor" [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/editing-presentations/assets/server.py | deck editor server.py | practitioner]; the overlay reloads "from the file" on "an edit the page cannot mirror" [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/editing-presentations/assets/overlay.js | overlay.js | practitioner]. No multi-user sync.

### Is a server-authoritative log viable without a CRDT? Evidence on both sides

For: Jamsocket: "when we started talking to dozens of companies building ambitious browser-based apps, we found it rare for apps to use true CRDTs to power multiplayer collaboration"; "browsers are inherently not peer-to-peer ... Since the server is centralized anyway, we can have it enforce a global ordering over the events"; "social locking: the tendency of reasonable people not to clobber each other's work-in-progress" [source: https://web.archive.org/web/2024/https://jamsocket.com/blog/you-might-not-need-a-crdt | You might not need a CRDT | vendor]. Figma, tldraw sync and Linear (above) all order changes on a server. The Linear study author: "CRDTs often introduce metadata overhead and become challenging to manage in scenarios involving partial syncing or permission controls" [source: https://github.com/wzhudev/reverse-linear-sync-engine | Reverse Linear Sync Engine | practitioner].

Against or limits: Jamsocket concedes "State machine replication is not a silver bullet; we still need to be thoughtful about how change operations apply when the underlying data has changed" and notes a multi-server deployment "complicates using 'the server' as a source of truth"; Figma routes each document to one process [source: same | vendor]. tldraw requires exactly one server instance per room [source: https://tldraw.dev/docs/sync | tldraw sync docs | vendor]. Figma's checkpoint design could "lose up to 60 seconds of work on the server-side if multiplayer crashes" before the journal [source: https://www.figma.com/blog/making-multiplayer-more-reliable/ | Making multiplayer more reliable | vendor]. Text: Google's OT needs transform rules for every pair of change types ("must handle all of the different ways that InsertText, DeleteText, and ApplyStyle changes can be paired") [source: https://drive.googleblog.com/2010/09/whats-different-about-new-google-docs_22.html | Google Docs conflict resolution | vendor]; tldraw's string diff falls back to a whole put for non-append edits (D5) [source: https://github.com/tldraw/tldraw/blob/main/packages/sync-core/SPEC.md | sync-core spec | vendor].

---

## SQ4: How agent edits (MCP tool calls, branches) fit with CRDTs or op logs

### Patches addressed by id vs by index

JSON Patch (RFC 6902) addresses array elements by JSON Pointer index; for `add` "The specified index MUST NOT be greater than the number of elements in the array"; "-" appends; `test` "tests that a value at the target location is equal to a specified value"; a failed operation means "application of the entire patch document SHALL NOT be deemed successful" [source: https://www.rfc-editor.org/rfc/rfc6902 | RFC 6902 JSON Patch | independent].

tldraw network diffs are keyed by record id, not path index (see SQ2) [source: https://github.com/tldraw/tldraw/blob/main/packages/sync-core/SPEC.md | sync-core spec | vendor]. Google Slides API requests name objects by `objectId` and apply atomically (SQ3) [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/batchUpdate | presentations.batchUpdate | vendor].

### Versioned writes and optimistic concurrency

Blattomat's MCP tools take a `version` and refuse on mismatch (SQ3) [source: /home/pascal/Code/blattwerk/docs/okf/versions.md | Versions and the 409 rule | practitioner]. Google Slides `requiredRevisionId` refuses with 400 on mismatch; without it the write is merged with collaborators' changes [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/batchUpdate | presentations.batchUpdate | vendor]. tldraw answers a push with `commit`, `discard` or `rebaseWithDiff` and ignores patches to deleted records (SQ3) [source: https://github.com/tldraw/tldraw/blob/main/packages/sync-core/SPEC.md | sync-core spec | vendor].

Automerge "write at a past version": `changeAt` "produces an equivalent history, but without having to create a fork of the document"; the documented workflow: "On initialization save the current heads ... Apply all the unreconciled changes to the document using `changeAt(doc, oldHeads, ...)` ... Get the diff from the resulting document to the current document using diff" [source: https://github.com/automerge/automerge/blob/main/javascript/src/implementation.ts | Automerge JS implementation.ts, changeAt doc comment | vendor]. The same file exports `view(doc, heads)` ("an immutable view of an automerge document as at `heads`"), `clone` ("a full writable copy ... It also generates a new actor ID"), `merge(local, remote)`, `diff(doc, before, after)` returning patches, `getHeads`, `getConflicts` [source: same | vendor].

Loro: "Loro supports primitives that allow users to switch between different versions, fork new branches, edit on new branches, and merge branches"; "You can also use `doc.fork()` to create a separate doc at the current version"; editing from a checkout needs `setDetachedEditing(true)` [source: https://github.com/loro-dev/loro-docs/blob/main/pages/blog/v1.0.mdx | Loro 1.0 announcement | vendor]. `doc.checkout(frontiers)` "places the document in a detached state, preventing further edits ... This design is temporary and will be phased out once we have a more refined version control API" [source: https://github.com/loro-dev/loro-docs/blob/main/pages/docs/tutorial/time_travel.mdx | Loro time travel | vendor].

### Branches for AI edits (Ink & Switch Patchwork)

"an AI bot for a style guide makes changes as another collaborator in the document. It puts changes on a branch, which you can choose to partially or completely merge, just like you would suggestions from a human coauthor"; "Once you accept edits from a bot, the history timeline also shows which edits came from the bot"; a variant where "the bot leaves comments with edit rationale" was "helpful for understanding whether to accept a particular edit" (2024-03-19) [source: https://www.inkandswitch.com/patchwork/notebook/2024-version-control/07/ | 07, AI bots in version control | independent].

"Automerge supports cloning documents and then merging them back together in reasonable ways"; "Automerge can compute an exact difference between two points in history of a document. It doesn't need to resort to heuristics because the full edit history is tracked"; "Automerge never deletes anything" [source: https://www.inkandswitch.com/patchwork/notebook/2024-version-control/ | Patchwork lab notebook, entry 08 | independent]. Limits found: "There's no branching from branches" (entry 06); porting to a tldraw-based diagram tool "took minimal effort" for branches and timeline, but diff views and comments needed domain-specific work; "Auto-generated LLM summaries work ok for visual edits, but are less obviously good than text summaries" (entry 10) [source: same | independent]. Ink & Switch also named "tricky challenges ... including some new ways of representing changes in Automerge to make it easier to cherry-pick edits across documents" (search summary of the notebook) [source: https://www.inkandswitch.com/patchwork/notebook/2024-version-control/ | Patchwork lab notebook | independent].

### Server-side agent writes into a live CRDT document

Hocuspocus (MIT) has `openDirectConnection(documentName, context)`, which opens the document server-side with `isAuthenticated: true, readOnly: false`, and a `DirectConnection.transact(transaction)` method [source: https://github.com/ueberdosis/hocuspocus/blob/main/packages/server/src/Hocuspocus.ts | Hocuspocus.ts | vendor] [source: https://github.com/ueberdosis/hocuspocus/blob/main/packages/server/src/DirectConnection.ts | DirectConnection.ts | vendor]. (The docs page tiptap.dev/docs/hocuspocus/server/methods returned 404 [inaccessible]; read from source.)

Tiptap's Server AI Toolkit: "server-side editing so you don't need an open browser anymore. You can build agents that edit documents even when your users are away, and the changes flow back into their editors through our Collaboration service"; restricted release [source: https://tiptap.dev/docs/content-ai/capabilities/server-ai-toolkit/overview | Server AI Toolkit | vendor] (search snippet). Tiptap roadmap: AI changes "can be surfaced as suggestions for human review, streamed in real-time, or applied directly" [source: https://tiptap.dev/roadmap | Tiptap roadmap | vendor] (search snippet).

Python: pycrdt (MIT, Yrs bindings) exposes `Doc`, `Map`, `Array`, `Text`, `XmlFragment` [source: https://github.com/y-crdt/pycrdt/blob/main/python/pycrdt/__init__.py | pycrdt | vendor]; automerge-py (MIT) exists [source: https://github.com/automerge/automerge-py/blob/main/LICENSE | automerge-py LICENSE | vendor]. Not checked: whether either covers the full JS API (for example Automerge `changeAt`).

### Counter-evidence and limits

Convergence is not correctness. Jamsocket: "Both paths will allow us to ensure that each replica converges on the same state, but that alone does not guarantee that this state is the 'correct' state from the point of view of user intent" [source: https://web.archive.org/web/2024/https://jamsocket.com/blog/you-might-not-need-a-crdt | You might not need a CRDT | vendor]. Loro: "Merging is monotonic: remote edits cannot be 'rolled back' on arrival"; "Check-then-set across peers needs coordination, not just convergence" [source: https://github.com/loro-dev/loro-docs/blob/main/pages/docs/concepts/when_not_crdt.mdx | When Not to Use CRDTs | vendor].

Two 2025/2026 papers on LLM agents over CRDTs surfaced in search (AgentRoom, said to be arXiv 2608.23740; CodeCRDT, said to be arXiv 2510.18893, with a claim that 5 to 10% of concurrent LLM code edits merged structurally but conflicted logically). The arXiv export API returned no record for either id on 2026-10-20; neither was read, and neither claim is used here [source: https://pith.science/paper/2608.23740 | AgentRoom (Pith listing) | independent] [inaccessible].

Vendor marketing on CRDTs for agents (Velt, crdt.ms) appeared in search and offers no independent evidence; not used [source: https://velt.dev/blog/best-crdt-libraries-real-time-data-sync | Best CRDT Libraries | vendor].
