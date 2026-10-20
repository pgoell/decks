# Masters, layouts and placeholders in PowerPoint, Google Slides and Pitch (open question 2)

Scope: open question 2 of `docs/spec-v0.md` ("How do Google Slides, Pitch and PowerPoint model masters, layouts and placeholders, and what of that must the JSON hold for the pptx export to give real layouts?"). Evidence only, no conclusions.

Local experiments referenced below (all in the session scratchpad, `S=/tmp/claude-1000/-home-pascal-Code--worktrees-decks-research/c7daf069-e30d-4d28-a991-3a72a1d85e6d/scratchpad`):

- python-pptx 1.0.2: script `S/scripts/mk.py`, output `S/out/pp.pptx`, unzipped XML in `S/out/ppx/`.
- PptxGenJS 4.0.1: script `S/scripts/pg.js`, output `S/out/pg.pptx`, unzipped XML in `S/out/pgx/`.
- No render check was done (no LibreOffice image on the host); the experiments read XML only.

## SQ1: How does OOXML (ECMA-376 PresentationML) model slide masters, slide layouts, placeholders, inheritance and themes, and what must a pptx contain for PowerPoint to show real layouts and let users switch layout?

### The three tiers and the theme

The slide master holds "all elements that describe the objects and their corresponding formatting for within a presentation slide"; its two main elements are `cSld` (shapes and their text bodies) and `txStyles` (text formatting), plus colour information, headers and footers, timing and transitions [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/sldMaster.html | ECMA-376 Part 1 reference mirror (docx4java), sldMaster | independent].

The same definition, quoted from ISO/IEC 29500, appears in Microsoft's Open XML SDK docs, with child elements `clrMap`, `cSld`, `extLst`, `hf`, `sldLayoutIdLst`, `timing`, `transition`, `txStyles` [source: https://learn.microsoft.com/en-us/office/open-xml/presentation/working-with-slide-masters | Working with slide masters (Open XML SDK) | vendor].

A slide layout "contains in essence a template slide design that can be applied to any existing slide. When applied to an existing slide all corresponding content should be mapped to the new slide layout" [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/sldLayout.html | ECMA-376 reference mirror, sldLayout | independent].

`sldLayout` attributes in the schema: `matchingName` ("used for layout matching in response to layout changes and template applications"), `type` (ST_SlideLayoutType, default `cust`), `preserve` (default false: layout is deleted when no slide uses it), `userDrawn`, plus `showMasterSp` ("if shapes on the master slide should be shown on slides or not") and `showMasterPhAnim` [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/sldLayout.html | ECMA-376 reference mirror, sldLayout | independent].

ST_SlideLayoutType is "not tied to an exact positioning of placeholders, but rather provides a higher-level description of the type of content and positioning of placeholders. This information may be used by the application to aid in mapping between different layouts." Values include `blank`, `title`, `titleOnly`, `obj`, `tx`, `twoObj`, `twoColTx`, `secHead`, `picTx`, `objTx`, `tbl`, `chart`, `cust` and about twenty more [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/ST_SlideLayoutType.html | ECMA-376 reference mirror, ST_SlideLayoutType | independent].

The master's `sldLayoutIdLst` "is used to determine which layouts are being used within the slide master file. Each layout within the list of slide layouts has its own identification number and relationship identifier" [source: https://learn.microsoft.com/en-us/office/open-xml/presentation/working-with-slide-masters | Working with slide masters | vendor].

`sldLayoutId@id` is ST_SlideLayoutId and `sldMasterId@id` is ST_SlideMasterId; both are unsignedInt with "a minimum value of greater than or equal to 2147483648" [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/ST_SlideLayoutId.html | ST_SlideLayoutId | independent] [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/ST_SlideMasterId.html | ST_SlideMasterId | independent].

The package chain in a python-pptx file: `presentation.xml` has `<p:sldMasterIdLst><p:sldMasterId id="2147483648" r:id="rId1"/>`; `slideMaster1.xml` lists eleven `<p:sldLayoutId id="2147483649".."2147483659">`; each `slideLayoutN.xml.rels` points to `../slideMasters/slideMaster1.xml`; `slide1.xml.rels` points to `../slideLayouts/slideLayout2.xml` [source: S/out/ppx/ppt/presentation.xml, S/out/ppx/ppt/slideMasters/slideMaster1.xml, S/out/ppx/ppt/slides/_rels/slide1.xml.rels | local experiment, python-pptx 1.0.2 default template | practitioner].

A slide has no explicit "master" field; it reaches its master only through its layout's relationship (local experiment above: `slide1.xml.rels` holds only a slideLayout relationship) [source: S/out/ppx/ppt/slides/_rels/slide1.xml.rels | local experiment | practitioner].

### Placeholders: `p:ph` type and idx

`ph` "specifies that the corresponding shape should be represented by the generating application as a placeholder". Attributes and defaults from CT_Placeholder: `type` (ST_PlaceholderType, default `obj`), `orient` (default `horz`), `sz` (default `full`), `idx` (unsignedInt, default 0), `hasCustomPrompt` (default false). On `idx`: "Specifies the placeholder index. This is used when applying templates or changing layouts to match a placeholder on one template/master to another." [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/ph.html | ECMA-376 reference mirror, ph | independent].

`sz` values `full`, `half`, `quarter` are "described relative to the body placeholder on the master" [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/ST_PlaceholderSize.html | ST_PlaceholderSize | independent].

python-pptx lists 18 placeholder types: Title, Center Title, Subtitle, Body, Content (obj), Picture, Clip Art, Chart, Table, Smart Art, Media Clip, Date, Footer, Slide Number, Header (notes and handout only), and three vertical variants [source: https://python-pptx.readthedocs.io/en/latest/user/placeholders-understanding.html | Understanding placeholders (python-pptx) | practitioner].

"Placeholders are an orthogonal category of shape": auto shapes, pictures and graphic frames can be placeholders; group shapes, connectors and content parts cannot [source: https://python-pptx.readthedocs.io/en/latest/user/placeholders-understanding.html | Understanding placeholders | practitioner].

### Inheritance rules

"A layout placeholder inherits from the master placeholder sharing the same type." "A slide placeholder inherits from the layout placeholder having the same idx value." Master placeholders are inheritees only, slide placeholders inheritors only, layout placeholders both [source: https://python-pptx.readthedocs.io/en/latest/user/placeholders-understanding.html | Understanding placeholders | practitioner].

Inherited: "all formatting is inherited from the parent placeholder, including position, size, fill, line, and font"; "Directly applied formatting overrides the inherited value"; reapplying the layout removes direct formatting [source: https://python-pptx.readthedocs.io/en/latest/user/placeholders-understanding.html | Understanding placeholders | practitioner].

From python-pptx's reverse-engineering notes: "Layout inherits properties from master based only on type (e.g. body, dt), with no bearing on idx value." "Slide inherits from layout strictly on idx value." In the default template, layout `dt`, `ftr`, `sldNum` carry idx 10, 11, 12 while the master's carry 2, 3, 4; Two Content's second `obj` is idx 2 on the layout but inherits from master body idx 1 [source: https://python-pptx.readthedocs.io/en/latest/dev/analysis/placeholders/layout-placeholders.html | Layout Placeholders (dev analysis) | practitioner].

Experiment reported there: with two body placeholders on a layout, no matching idx and no position, "both placeholders are present, directly superimposed upon each other" and "both inherit position and fill" [source: https://python-pptx.readthedocs.io/en/latest/dev/analysis/placeholders/layout-placeholders.html | Layout Placeholders | practitioner].

"The placeholder with idx="0" is the title placeholder"; "'0' is the default for the idx attribute on <p:ph>, so one with no idx attribute is the title placeholder." "A placeholder shape on a slide is initially little more than a reference to its "parent" placeholder"; "Position, size, and even geometry are inherited from the layout placeholder" [source: https://python-pptx.readthedocs.io/en/latest/dev/analysis/placeholders/slide-placeholders/index.html | Slide Placeholders (dev analysis) | practitioner].

Local confirmation: the slide written by python-pptx from layout "Title and Content" has `<p:ph type="title"/>` and `<p:ph idx="1"/>`, each with an empty `<p:spPr/>` and only text in `<p:txBody>`; the master's title placeholder holds the geometry `<a:off x="457200" y="274638"/><a:ext cx="8229600" cy="1143000"/>`; the layout's title placeholder has an empty `<p:spPr/>` too [source: S/out/ppx/ppt/slides/slide1.xml, S/out/ppx/ppt/slideLayouts/slideLayout2.xml, S/out/ppx/ppt/slideMasters/slideMaster1.xml | local experiment | practitioner].

What PowerPoint (Mac 2011) does on new slide: every layout placeholder is cloned "excepting those of footer, date, and slide number type"; the slide `<p:ph>` "exactly matched the corresponding element in the slide layout" except `hasCustomPrompt`, which "did not appear in the slide"; "All placeholder shapes had an empty `<p:spPr>` element"; only title, body, ctrTitle, subTitle and obj get a minimal `<p:txBody>`; each gets `<a:spLocks noGrp="1"/>` [source: https://python-pptx.readthedocs.io/en/stable/dev/analysis/placeholders/slide-placeholders/placeholders-in-new-slide.html | Placeholders in new slide | practitioner].

Replacing a placeholder with a table converts it to `<p:graphicFrame>` but keeps the `<p:ph>`; reapplying the layout moves the table back to the placeholder position [source: https://python-pptx.readthedocs.io/en/stable/dev/analysis/placeholders/slide-placeholders/placeholders-in-new-slide.html | Placeholders in new slide | practitioner].

### Text styles: txStyles and list levels

`txStyles` holds "the styling information for title text, the body text and other slide text"; "only for use within the Slide Master". Children: `titleStyle`, `bodyStyle`, `otherStyle`, each CT_TextListStyle [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/txStyles.html | txStyles | independent].

CT_TextListStyle contains `defPPr` and `lvl1pPr` through `lvl9pPr` (nine list levels) [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/bodyStyle.html | bodyStyle | independent].

Spec example title style: `<a:lvl1pPr algn="ctr">` with `<a:defRPr sz="4400">`, `<a:schemeClr val="tx1"/>` and `<a:latin typeface="+mj-lt"/>` [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/txStyles.html | txStyles | independent].

Local: python-pptx default master `bodyStyle` lvl1 is `marL="342900" indent="-342900"`, bullet `•` in Arial, `sz="3200"`, `schemeClr tx1`, `+mn-lt`; lvl2 is `marL="742950"`, bullet character U+2013, `sz="2800"`. `presentation.xml` carries a further `<p:defaultTextStyle>` (lvl1 `sz="1800"`) [source: S/out/ppx/ppt/slideMasters/slideMaster1.xml, S/out/ppx/ppt/presentation.xml | local experiment | practitioner].

Places styling can live (theme `objectDefaults`, presentation `defaultTextStyle`, master `txStyles`, `lstStyle` on master/layout/slide shapes, run `rPr`): an Okapi Framework issue reportedly lists these and states the lookup order is undocumented and that PowerPoint ignores `lstStyle/defPPr` in favour of `lvl1pPr`. That page returned 404 and was not read; the claim comes from a search summary only [source: https://bitbucket.org/okapiframework/okapi/issues/999 | Okapi issue 999 | practitioner] [inaccessible].

No usable source found that states the full run-to-defaultTextStyle lookup order for PresentationML.

### Theme: colour scheme, font scheme, colour map

`clrScheme` defines twelve colours: "six accent colors, two dark colors, two light colors and a color for each of a hyperlink and followed hyperlink" (`dk1 lt1 dk2 lt2 accent1..6 hlink folHlink`, all required) [source: https://webapp.docx4java.org/OnlineDemo/ecma376/DrawingML/clrScheme.html | clrScheme | independent].

The master's `clrMap` "specifies the mapping layer that transforms one color scheme definition to another"; attributes `bg1 tx1 bg2 tx2 accent1..6 hlink folHlink` are required; spec example of an inverted map: `bg1="dk1" tx1="lt1"` [source: https://webapp.docx4java.org/OnlineDemo/ecma376/PresentationML/clrMap.html | clrMap | independent].

Layouts and slides carry `clrMapOvr`: `masterClrMapping` keeps the master's map, `overrideClrMapping` defines a new one [source: https://learn.microsoft.com/en-us/office/open-xml/presentation/working-with-slide-layouts | Working with slide layouts | vendor].

Font scheme: the docx4java page for `fontScheme` returned 404 [inaccessible]. Local: python-pptx's `theme1.xml` has `<a:fontScheme name="Office"><a:majorFont><a:latin typeface="Calibri"/>...` with per-script fallbacks (Jpan, Hang, Hans, Hant, Arab, Hebr, Thai); text references it as `+mj-lt` (major, headings) and `+mn-lt` (minor, body) in `txStyles` [source: S/out/ppx/ppt/theme/theme1.xml, S/out/ppx/ppt/slideMasters/slideMaster1.xml | local experiment | practitioner].

The `extracting-presets` skill already reads these: "A slide resolves through its layout (slide rels into `ppt/slideLayouts/`), the layout through its master (layout rels), the master through the theme (master rels), and the master's `clrMap` remaps scheme slots on the way" [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/extracting-presets/references/slide-masters.md | slide-masters.md | practitioner].

### Switching layout and resetting, as PowerPoint behaves

"When you edit the slide master, all slides that are based on that master will contain those changes." But "if there are existing slides in your presentation that are based on the prior version of the layout ... you'll need to reapply the layout to those slides." [source: https://support.microsoft.com/en-us/office/what-is-a-slide-master-b9abb2a0-7aef-4257-a14e-4329c904da54 | What is a slide master? | vendor].

Reset "resets the position, size, and any other formatting of the text placeholders to their default settings"; "the Reset button does not work with text boxes" [source: https://www.indezine.com/products/powerpoint/learn/interface/2013/reset-slide.html | Reset Slides in PowerPoint 2013 | practitioner].

python-pptx observations on layout change: "Content of a placeholder shape is retained and displayed, even when the slide layout is changed to one without a matching layout placeholder"; switching a title slide with text to blank kept the shapes and "Position and dimension values were added that preserve the height, width, top position but set the left position to zero"; "Restoring the original layout caused those position and dimension values to be removed (and "re-inherited")"; "Size and position are both reset. Background color and font, at least, are not reset" on reapply [source: https://python-pptx.readthedocs.io/en/latest/dev/analysis/placeholders/slide-placeholders/index.html | Slide Placeholders (dev analysis) | practitioner].

Counter-evidence (matching in practice is fragile): a user whose layouts share title, subtitle and section placeholders reports content lands in the wrong placeholder when switching layout. Microsoft moderator: "the placeholder number can't be kept. I'm afraid that there is no out-of-the-box way doing that." Volunteer moderator John Korchok: "The names of placeholders do not affect the order in which content is placed in them"; "Create placeholders in the order in which they are to be filled: top to bottom, left to right"; when the new layout has fewer placeholders, "The content will be orphaned and often end up at the right or left side of the slide." [source: https://learn.microsoft.com/en-us/answers/a/11388916 | Microsoft Q&A: placeholder order across layouts | practitioner].

The O'Reilly Office 2007 Bible section "Changing a Slide's Layout" returned 403 and was not read [source: https://oreilly.com/library/view/office-2007-bible/9780470046913/9780470046913_ch22lev1sec2.html | Office 2007 Bible | practitioner] [inaccessible].

### Other renderers disagree on matching

LibreOffice bug 104202 (filed 2016, resolved as duplicate of 162571): a PowerPoint file where the master subtitle is `<p:ph type="subTitle" sz="quarter" idx="1"/>` but the slide's is `idx="4294967295"`. Comment (2022-02-11): "The problem is that the slide should not be inheriting the textbox from the master ... If I set the slide's idx to 1, then MS 2010 opens like LO - i.e. the master's textbox settings apply to the slide's textbox." A 2021-12 comment: "PP overrides the master slide setting Impress does not, however should." [source: https://bugs.documentfoundation.org/rest/bug/104202/comment | LibreOffice bug 104202 comments (REST API; the HTML page is behind an Anubis block) | independent]. The bug it duplicates, 162571 ("EDITING PPTX: Text box expands/shrinks on both sides instead of just on the right"), is VERIFIED FIXED [source: https://bugs.documentfoundation.org/rest/bug/162571 | LibreOffice bug 162571 | independent].

LibreOffice bug 128547 (2019): a PPTX exported from Keynote for iCloud opens in Impress with "The title position is wrong. The subtitle imported as a left aligned bullet item." (resolved duplicate) [source: https://bugs.documentfoundation.org/rest/bug/128547/comment | LibreOffice bug 128547 | independent].

Relevance to the current toolchain: the `exporting-presentations-to-pptx` skill verifies every export by rendering through LibreOffice in a podman container [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/exporting-presentations-to-pptx/SKILL.md | exporting-presentations-to-pptx SKILL.md, "Render-verify loop" | practitioner].

### Why placeholders matter beyond layout switching

"please refrain from manually adding text boxes. Instead, enter text in the placeholders in the slide"; a screen reader does not read hand-added text boxes; "Screen reader users often use outline view to access the text", and outline view shows only placeholder text [source: https://elgin.edu/accessibility/accessibility-in-presentations/slide-layouts/ | Elgin Community College: Slide Layouts | independent].

Counter-evidence on how binding this is: a WebAIM list contributor calls placeholder use "not a requirement or standard; it is a best practice" (search summary of the thread; thread not fetched) [source: https://webaim.org/discussion/mail_message?id=41715 | WebAIM discussion list | practitioner].

## SQ2: How does the Google Slides API model masters, layouts and placeholders?

### Page types and properties

`PageType` enum: `SLIDE`, `MASTER`, `LAYOUT`, `NOTES`, `NOTES_MASTER` [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages | REST Resource: presentations.pages | vendor].

"Slide masters define the default text styles, background, and page elements"; "Layouts serve as a template for how page elements will be arranged by default on slides using a layout"; notes masters "are read-only in the Slides API" [source: https://developers.google.com/workspace/slides/api/guides/overview | Slides API overview | vendor].

`SlideProperties`: `layoutObjectId` ("The object ID of the layout that this slide is based on. This property is read-only."), `masterObjectId` ("This property is read-only."), `notesPage`, `isSkipped`. `LayoutProperties`: `masterObjectId`, `name`, `displayName`. `MasterProperties`: `displayName` [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages | presentations.pages | vendor].

Contrast with OOXML: Google's slide names both its layout and its master directly; OOXML slides reach the master only through the layout relationship (see SQ1 local experiment).

### Placeholders and parentObjectId

`Placeholder` JSON: `{ "type": enum, "index": integer, "parentObjectId": string }`. `index`: "If the same placeholder types are present in the same page, they would have different index values." `parentObjectId`: "The object ID of this shape's parent placeholder. If unset, the parent placeholder shape does not exist, so the shape does not inherit properties from any other shape." Type enum "correspond[s] to placeholder IDs from the ECMA-376 standard ... ST_PlaceholderType": `NONE, BODY, CHART, CLIP_ART, CENTERED_TITLE, DIAGRAM, DATE_AND_TIME, FOOTER, HEADER, MEDIA, OBJECT, PICTURE, SLIDE_NUMBER, SUBTITLE, TABLE, TITLE, SLIDE_IMAGE` [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages/other | presentations.pages, other types | vendor].

Shape.placeholder: "Placeholders are page elements that inherit from corresponding placeholders on layouts and masters. If set, the shape is a placeholder shape and any inherited properties can be resolved by looking at the parent placeholder identified by the Placeholder.parent_object_id field." [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages/shapes | presentations.pages, shapes | vendor].

"You can create placeholders using the Slides editor, but not using the Slides API." "Shapes are the only type of page element that can have parents"; images, tables and charts cannot be placeholders [source: https://developers.google.com/workspace/slides/api/concepts/page-elements | Pages, page elements and properties | vendor].

Inheritance: "To inherit a property, the child page simply doesn't set a value for that property in its PageProperties message"; the same applies to shapes via `ShapeProperties`; PropertyState `NOT_RENDERED`, `RENDERED`, `INHERIT` (shapes without a parent cannot use `INHERIT`) [source: https://developers.google.com/workspace/slides/api/concepts/page-elements | Pages, page elements and properties | vendor].

### Creating slides from layouts

`CreateSlideRequest`: `objectId`, `insertionIndex`, `slideLayoutReference`, `placeholderIdMappings[]`. The layout reference is resolved "based on the current master" (the previous slide's master, the first slide's, or the first master); "If the LayoutReference is not found in the current master, a 400 bad request error is returned. If you don't specify a layout reference, the slide uses the predefined BLANK layout." [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/request | batchUpdate requests | vendor].

`LayoutReference` is either `predefinedLayout` (BLANK, CAPTION_ONLY, TITLE, TITLE_AND_BODY, TITLE_AND_TWO_COLUMNS, TITLE_ONLY, SECTION_HEADER, SECTION_TITLE_AND_DESCRIPTION, ONE_COLUMN_TEXT, MAIN_POINT, BIG_NUMBER) or `layoutId`. Predefined layouts carry "no guarantee that these layouts are present in the current master as they could have been deleted or not part of the used theme. Additionally, the placeholders on each layout may have been changed." [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/request | batchUpdate requests | vendor].

`LayoutPlaceholderIdMapping`: `objectId` (user-chosen id for the slide placeholder) plus either `layoutPlaceholder` ("Only type and index are needed. For example, a predefined TITLE_AND_BODY layout may usually have a TITLE placeholder with index 0 and a BODY placeholder with index 0.") or `layoutPlaceholderObjectId` [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/request | batchUpdate requests | vendor].

"Layouts normally have placeholder shapes that are copied to the slide when it's created"; `placeholderIdMappings` "lets you modify the copied placeholders in the same batchUpdate request, which improves performance and saves quota" [source: https://developers.google.com/workspace/slides/api/guides/create-slide | Create a slide | vendor].

Object ids must be 5 to 50 characters, start with `[a-zA-Z0-9_]`, then `[a-zA-Z0-9_-:]` [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/request | batchUpdate requests | vendor].

### What the API cannot do

The batchUpdate request list (51 request types extracted, from `AddCommentReplyRequest` to `UpdateVideoPropertiesRequest`) contains no request that changes a slide's layout or creates a layout or master; `UpdateSlidePropertiesRequest`'s documented example field is `isSkipped`, and `layoutObjectId` is read-only [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations/request | batchUpdate requests | vendor] [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages | presentations.pages | vendor].

Apps Script's `Slide` class has `getLayout()` but no `setLayout` or `applyLayout` (0 matches in the class reference) [source: https://developers.google.com/apps-script/reference/slides/slide | Apps Script Slide class | vendor].

Copying across presentations: `appendSlide`/`insertSlide` with a slide from another presentation: "the parent master and layout pages are copied as well if they do not already exist in this presentation"; `SlideLinkingMode.LINKED` lets `Slide.refreshSlide()` update the copy from its source [source: https://developers.google.com/apps-script/reference/slides/presentation | Apps Script Presentation class | vendor]. Placeholder copies: within one presentation, "properties that inherit from master or layout pages also inherit on the inserted element"; from another presentation, those properties "are copied onto the element from the source presentation" [source: https://developers.google.com/apps-script/reference/slides/slide | Apps Script Slide class, insertGroup | vendor].

### Theme colours

`ThemeColorType`: DARK1, LIGHT1, DARK2, LIGHT2, ACCENT1..6, HYPERLINK, FOLLOWED_HYPERLINK, then TEXT1, BACKGROUND1, TEXT2, BACKGROUND2 [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages/other | presentations.pages, other types | vendor].

`PageProperties.colorScheme`: "If unset, the color scheme is inherited from a parent page ... Only the concrete colors of the first 12 ThemeColorTypes are editable. In addition, only the color scheme on Master pages can be updated. To update the field, a color scheme containing mappings from all the first 12 ThemeColorTypes to their concrete colors must be provided." [source: https://developers.google.com/workspace/slides/api/reference/rest/v1/presentations.pages | presentations.pages | vendor].

The first twelve map one to one onto OOXML's `clrScheme` slots (dk1, lt1, dk2, lt2, accent1..6, hlink, folHlink); TEXT1/BACKGROUND1 etc. mirror OOXML's `clrMap` names tx1/bg1 (comparison of the two enum lists above with the ECMA `clrScheme` and `clrMap` pages cited in SQ1).

### The Google Slides editor

A theme is "A preset group of colors, fonts, background, and layouts"; Slide > Change theme, with Import theme from another Slides or PowerPoint file; Slide > Edit theme > Colors; a slide's layout is changed via the Layout button [source: https://support.google.com/docs/answer/1705254 | Google Docs Editors Help: themes, layouts, templates | vendor]. The help page does not say what happens to existing slides on theme change.

Theme Builder: "The Theme (formerly Master) controls the style of every slide"; layouts sit below it; to apply one, right-click a slide thumbnail, "Apply layout"; theme-level images "can only be edited from the Theme Builder" [source: https://www.brightcarbon.com/blog/editing-themes-and-layouts-in-google-slides/ | BrightCarbon: Editing themes and layouts in Google Slides | practitioner].

pptx round trip: a python-pptx user reports that after editing a PowerPoint-made template's master layout in Google Slides and exporting, "PowerPoint can read these" but python-pptx falls back to defaults (open issue, 2022) [source: https://github.com/scanny/python-pptx/issues/864 | python-pptx #864 | practitioner]. No primary source found on whether Google's pptx download keeps masters and layouts as real `sldLayout` parts.

## SQ3: How do Pitch, Keynote, Gamma and Figma Slides handle templates, layouts and masters; what happens on change after the slides exist; how do they export to pptx?

### Pitch

Slide styles: the style should "set the default options for any blocks you add to your slides, like text, shapes, and charts"; it covers style colours, font families, slide background (solid or gradient), hyperlink and bullet colours, text styles, and a chart palette. Replacing a style colour: "Any elements that previously used that color will instantly be updated." Changing a text block's formatting shows a brush icon with Update (push to all text blocks using that style) or Reset [source: https://help.pitch.com/en/articles/4059534-create-your-own-slide-style | Pitch: Create your own slide style | vendor].

Applying styles: select slides, right-click, Apply style; or Apply to all slides; the options menu shows how many slides use a style; deleting down to one style "automatically applies the remaining style to your entire presentation"; styles can be published to the workspace [source: https://help.pitch.com/en/articles/4063049-work-with-slide-styles | Pitch: Work with slide styles | vendor].

Layouts: "Layouts are additional slide designs (timelines, image grids, diagrams, quotes, and more) that work with any template"; their design "dynamically matches the style of your currently selected slide"; "the Layouts collection is pre-defined for everyone in the workspace" and users cannot choose which appear [source: https://help.pitch.com/en/articles/7003075-add-slides-using-layouts | Pitch: Add slides using layouts | vendor]. The article does not say whether an inserted slide stays linked to its layout, and mentions no placeholders.

Templates: "Each template comes with a different slide style"; slides from any template can be added to a presentation; workspace owners can hide gallery templates (premium) [source: https://help.pitch.com/en/articles/4056913-find-and-use-templates | Pitch: Find and use templates | vendor]. The article does not say whether template changes reach existing presentations.

pptx export, supported: slides, images (JPG, PNG, SVG), text ("Bullet colors take the text color of the list item"), shapes, tables, speaker notes, embedded Vimeo/YouTube, charts ("may look different in the export based on PowerPoint's formatting"), fonts ("we cannot always embed the font", fallback to Calibri). Unsupported: uploaded videos, recordings, "Animations (enter/exit effects, transitions, continuity)" [source: https://help.pitch.com/en/articles/6713988-export-a-presentation-to-power-point | Pitch: Export a presentation to PowerPoint | vendor]. The article says nothing about slide layouts or masters in the exported file.

pptx import: "Review your imported presentation to ensure everything looks as expected and replace any unsupported content"; nothing on masters or layouts [source: https://www.pitch.com/integrations/ppt-import | Pitch: PowerPoint import | vendor].

No usable sources found on the internal structure of Pitch's exported pptx (layouts vs blank slides); two searches returned only third-party tools.

### Keynote

Keynote's current guide calls them "slide layouts": "The slide templates used in each Keynote theme are based on slide layouts." Placeholders: text (from a text box or shape), media (image or video), and one object placeholder per layout ("A placeholder you can add a table, chart, or image gallery to is called an object placeholder"). Objects on a layout not defined as placeholders "become part of the slide background and aren't editable in your presentation." "Placeholders are automatically tagged according to type"; tags decide where content goes when a different layout is applied, and can be edited in the Style tab. Layout edits: "The changes appear on every slide in the presentation that's based on that layout." Deleting a used layout prompts for a replacement [source: https://support.apple.com/guide/keynote/add-and-edit-master-slides-tan7a2b69972/mac | Keynote User Guide: Add and edit slide layouts | vendor].

Reapplying: "Reapplying the slide layout won't delete your content"; "text and media placeholders return to their default style and position" [source: https://support.apple.com/guide/keynote/change-the-slide-layout-or-theme-tan584189747/mac | Keynote User Guide: Apply a slide layout | vendor]. An older guide version adds: "If you're sharing a presentation with others, you can't reapply master slides" (search summary of Keynote 9.0 guide) [source: https://help.apple.com/keynote/mac/9.0/en.lproj/tan584189747.html | Keynote 9.0 guide | vendor].

pptx export: Apple's export page covers File > Export To and passwords, nothing on layouts or placeholders [source: https://support.apple.com/guide/keynote/tana0d19882a/mac | Keynote: Export to PowerPoint or another format | vendor]. No usable sources for how Keynote maps its layouts to `sldLayout` parts. Indirect: LibreOffice bug 128547 shows a Keynote-for-iCloud pptx whose subtitle Impress reads "as a left aligned bullet item" (SQ1).

### Gamma

Theme switch: "changing the theme only updates the visual styling-your content remains intact"; themes cover fonts, colours, text styles, backgrounds [source: https://help.gamma.app/en/articles/10262646-how-do-i-change-my-gamma-theme | Gamma: How do I change my theme | vendor]. Gamma can build a theme from an uploaded PowerPoint or Google Drive file's colours, fonts and logo (search summary of Gamma help 11029150 and 7852253; not fetched) [source: https://help.gamma.app/en/articles/11029150-can-i-add-my-own-colors-and-fonts-to-gamma | Gamma: own colors and fonts | vendor].

Export: exports "reflect Present Mode, not Edit Mode, so some visual differences are expected"; "Zoom to fit" applies "only ... to fixed-format exports" [source: https://help.gamma.app/en/articles/15939201-why-doesnt-my-exported-pdf-or-powerpoint-match-what-i-see-in-gamma | Gamma: Why doesn't my export match | vendor].

User reports on pptx export: a request for "Fully editable PowerPoint export (title, subtitle, all text, graphics, speaker notes ...)" after every export produced "a locked single image per slide"; a Gamma designer said she would not expect editable export to fail and asked for the file; the user later traced it to one slide [source: https://ideas.gamma.app/ideas/p/fully-editable-powerpoint-export-title-subtitle-all-text-graphics-speaker-notes | Gamma ideas board | practitioner]. Other ideas-board titles: "Text exports to powerpoint as line by line instead of one big text box" and "Export tables as native PowerPoint tables (editable)" (titles from search results; threads not fetched) [source: https://ideas.gamma.app/ideas/p/text-exports-to-powerpoint-as-line-by-line-instead-of-one-big-text-box | Gamma ideas board | practitioner].

A vendor blog says a Gamma pptx may open with "a blank Slide Master" (search summary, not fetched; WPS sells a competing editor) [source: https://www.wps.com/blog/gamma-ppt-fix-exported-decks-in-wps-presentation/ | WPS blog | vendor]. No primary source on whether Gamma's pptx uses placeholders or real layouts.

### Figma Slides

Changing layout: "It is not possible to change a Slide's layout after it's been created"; workaround is "adding a New slide from template" (Figma Community Support, 2025-11-05); "choosing a new template style will only changes the current template's text styles and color fills on the current slide" (2026-02-24) [source: https://forum.figma.com/ask-the-community-7/how-do-i-change-the-layout-of-an-existing-slide-in-figma-slides-46971 | Figma forum: change layout of existing slide | vendor].

Template updates: "Right now, published template changes will only apply to new slide decks that are made with the template"; "The Figma Slides team has not yet the capability to automatically push template changes to existing Figma Slides" (Community Support, 2024-10-16). Users still report it unresolved on 2025-12-30 and 2026-01-05 [source: https://forum.figma.com/suggest-a-feature-11/figma-slides-publishing-template-changes-should-auto-apply-to-existing-slide-decks-23115 | Figma forum feature request | practitioner]. A user (2025-12-12): republishing a changed layout "seems to have no effect, and i can't find any UI that links my existing layouts with the new published changes" [source: https://forum.figma.com/ask-the-community-7/slides-template-creating-editing-slide-layouts-template-styles-40621 | Figma forum: template styles | practitioner].

pptx export limits: unavailable fonts replaced by the PowerPoint default; "Live interactions and code blocks will be exported as static images"; "Gradient fills will be changed to solid color fills"; a Structure option to "convert all objects to editable PPTX equivalent or to flatten shapes and images into bitmap" [source: https://help.figma.com/hc/en-us/articles/24848334599447 | Figma: Export from Figma Slides | vendor]. User report: "fonts change, alignement get mess up, rotations reset" (2025-06-26), closed without a fix [source: https://forum.figma.com/report-a-problem-6/issues-exporting-slides-to-pptx-42186 | Figma forum | practitioner]. Nothing found on layouts in Figma's pptx.

Third-party prior art: the Pitchdeck plugin (Hypermatic) offers "Create PowerPoint master slides from components" on export; text stays a placeholder; "locking a layer in Figma makes that element export as a static image instead of an editable placeholder" [source: https://docs.hypermatic.com/pitchdeck/tutorials/add-master-slides-to-powerpoint-exports-from-figma.md | Pitchdeck: master slides from Figma | vendor].

### Summary table of change behaviour (facts as reported above)

| Product | Unit of design | Edit master/layout after slides exist | Change a slide's layout |
|---|---|---|---|
| PowerPoint | master, layouts, placeholders | master edits reach slides; layout edits need reapply per Microsoft support | yes; matching by idx/type, orphans possible |
| Google Slides | theme (master) + layouts | editor: yes; API: master colour scheme only | editor: yes; API: no |
| Pitch | slide style (tokens) + fixed layout gallery | style colour edits update all elements | apply another style; layout link not documented |
| Keynote | slide layouts with tagged placeholders | layout edits appear on all slides using it | yes; tags route content; reapply resets |
| Figma Slides | published template, template styles | no propagation to existing decks | no |
| Gamma | theme | theme switch restyles, content kept | not documented |

## SQ4: What minimum must a JSON deck model hold for a pptx export to produce slides attached to real layouts with working placeholders?

### What the current exporter does

The `exporting-presentations-to-pptx` skill builds each deck with a fresh python-pptx script and helpers `tb()` (rich text boxes) and `rect()` (autoshapes); a grep for `slide_layouts`, `Presentation(` and `blank` in the skill returns nothing, and no recipe mentions layouts or placeholders [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/exporting-presentations-to-pptx/SKILL.md and references/python-pptx-recipes.md | exporter skill | practitioner].

The presets hold CSS tokens (`colors.css`, `typography.css`), HTML example slides (27 in `default/slides/`) and a layout grammar; no preset stores a pptx or potx [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/presets/README.md and presets/default/ | presentations presets | practitioner]. `extracting-presets` reads a client pptx's masters as an extraction source ("Placeholder frames on the master and its layouts ... carry positions and sizes in EMU (6350 EMU per px at 1920 wide)") [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/skills/extracting-presets/references/slide-masters.md | slide-masters.md | practitioner].

### python-pptx

python-pptx cannot add a layout: "The short answer is it's not supported yet"; the route is to open a starting-point deck, `Presentation('arbitrary-deck.pptx')`, and use its masters and layouts (Steve Canny, 2018-07-28, issue still open) [source: https://github.com/scanny/python-pptx/issues/413 | python-pptx #413 | practitioner]. Open issues also ask for changing a slide's layout to a second master's layout (2026-02-10) and adding layouts from another presentation [source: https://github.com/scanny/python-pptx/issues/1109 | python-pptx #1109 | practitioner] [source: https://github.com/scanny/python-pptx/issues/175 | python-pptx #175 | practitioner].

`Presentation()` without a file loads a 4:3 default: `<p:sldSz cx="9144000" cy="6858000" type="screen4x3"/>` with eleven layouts (Title Slide, Title and Content, Section Header, Two Content, Comparison, Title Only, Blank, Content with Caption, Picture with Caption, two vertical) whose placeholders carry idx 0, 1, 2..4 and 10, 11, 12 for date, footer, slide number [source: S/out/ppx/ppt/presentation.xml and output of S/scripts/mk.py | local experiment, python-pptx 1.0.2 | practitioner]. Spec v0 assumes 12192000 by 6858000 EMU [source: /home/pascal/Code/.worktrees/decks/research/docs/spec-v0.md | decks spec v0, Units | practitioner].

With a layout present, python-pptx writes slide placeholders that inherit fully: `<p:ph type="title"/>` and `<p:ph idx="1"/>` with empty `<p:spPr/>` (SQ1 local experiment).

License: python-pptx LICENSE is MIT: "Copyright (c) 2013 Steve Canny ... Permission is hereby granted, free of charge, to any person obtaining a copy ... to deal in the Software without restriction" [source: https://github.com/scanny/python-pptx/blob/master/LICENSE (read via `gh api repos/scanny/python-pptx/license`) | python-pptx LICENSE | independent].

### PptxGenJS

Docs: "The defined Masters become first-class Layouts in the exported PowerPoint presentation"; placeholder options `name`, `type` (`title`, `body`, `image`, `chart`, `table`, `media`), `x, y, w, h`, default `text`; slides fill them with `slide.addText(..., { placeholder: "body" })` [source: https://gitbrent.github.io/PptxGenJS/docs/masters/ | PptxGenJS: Slide Masters | vendor].

Theme options documented: `headFontFace`, `bodyFontFace`, `lang` only [source: https://gitbrent.github.io/PptxGenJS/docs/usage-pres-options/ | PptxGenJS: Presentation options | vendor]. Scheme colours `tx1, tx2, bg1, bg2, accent1..6` can be referenced; the docs do not say they can be redefined [source: https://gitbrent.github.io/PptxGenJS/docs/shapes-and-schemes/ | PptxGenJS: Shapes and schemes | vendor]. Request to override scheme colours, open since 2019-10-05; maintainer: "I think this would be a very useful feature" [source: https://github.com/gitbrent/PptxGenJS/issues/604 | PptxGenJS #604 | practitioner].

Local experiment (LAYOUT_WIDE, `theme: {headFontFace: "Georgia", bodyFontFace: "Arial"}`, one master `TITLE_BODY` with a rect, a text, title and body placeholders and a slide number) [source: S/scripts/pg.js, S/out/pgx/ | local experiment, PptxGenJS 4.0.1 | practitioner]:

- Package: `sldSz cx="12192000" cy="6858000"`; one `slideMaster1.xml`; two layouts, `DEFAULT` and `TITLE_BODY`. Each `defineSlideMaster` call becomes a layout under one shared master.
- Layout `TITLE_BODY`: `<p:sldLayout ... preserve="1">` with no `type` attribute (schema default `cust`) and no `matchingName`; placeholders `<p:ph idx="102" type="title" hasCustomPrompt="1"/>`, `<p:ph idx="103" type="body" hasCustomPrompt="1"/>`, `<p:ph type="sldNum" sz="quarter" idx="4294967295"/>`.
- Master: no title or body placeholder, only `<p:ph type="sldNum" sz="quarter" idx="4294967295"/>`; `txStyles` is the stock Office one (`titleStyle` lvl1 `algn="ctr"`, `sz="4400"`, `+mj-lt`).
- Slide: placeholders repeat `idx="102"`/`"103"` and keep `hasCustomPrompt="1"` (PowerPoint drops it on slides, per SQ1); each slide placeholder writes its own `<a:xfrm>` (`off x="457200" y="365760"`, `ext cx="11247120" cy="914400"`), `prstGeom`, `noFill` and run `sz="3600"`/`"2000"`, so position, size and font size are set on the slide, not inherited.
- Theme: `majorFont latin="Georgia"` set from `headFontFace`; `clrScheme name="Office"` stays the stock palette (`dk2 44546A`, `lt2 E7E6E6`).

The `idx="4294967295"` pattern on the sldNum placeholder is the same value LibreOffice bug 104202 shows PowerPoint treating as "no inheritance from the master" (SQ1).

Open PptxGenJS bugs on masters: empty placeholders created as TEXT instead of PLACEHOLDER, "causing duplicate content to appear: both the user's intended content AND an empty "Click to add text" placeholder box", root cause `addPlaceholdersToSlideLayouts()` in `src/gen-objects.ts` (2026-04-15) [source: https://github.com/gitbrent/PptxGenJS/issues/1453 | PptxGenJS #1453 | practitioner]; master placeholder `valign: "middle"` ignored when a user adds a slide via PowerPoint's New Slide (2023-01-05) [source: https://github.com/gitbrent/PptxGenJS/issues/1208 | PptxGenJS #1208 | practitioner]; "Adding a table to a placeholder does not work" (2020) [source: https://github.com/gitbrent/PptxGenJS/issues/856 | PptxGenJS #856 | practitioner]; `shape: "roundRect"` on a placeholder has no effect (2025-09-23) [source: https://github.com/gitbrent/PptxGenJS/issues/1422 | PptxGenJS #1422 | practitioner]; notes master placeholders removed by PowerPoint repair [source: https://github.com/gitbrent/PptxGenJS/issues/1443 | PptxGenJS #1443 | practitioner].

Contradiction: the docs call masters "first-class Layouts" [vendor], while #1453 and #1208 and the local XML show slide placeholders that override geometry and size and layouts without `type`.

License: PptxGenJS LICENSE is MIT: "Copyright (c) 2015-2022 Brent Ely ... Permission is hereby granted, free of charge ... to deal in the Software without restriction" [source: https://github.com/gitbrent/PptxGenJS/blob/master/LICENSE (read via `gh api repos/gitbrent/PptxGenJS/license`) | PptxGenJS LICENSE | independent].

### Data points the evidence ties to each model field

Fields named in spec v0's deck document: slide `master`, block `slot`, block `x, y, w, h`, deck `theme`; "A block with `slot` fills that placeholder of the slide's master and takes its box from the master unless it sets its own" [source: /home/pascal/Code/.worktrees/decks/research/docs/spec-v0.md | decks spec v0, Deck document | practitioner]. Evidence per OOXML element an exporter would have to emit:

- Layout identity: every slide needs a relationship to one `sldLayout` part, which in turn belongs to one master listed in `presentation.xml` (SQ1). Spec v0 calls its layout a "master" ("A master is a layout"); OOXML and Google both separate master (shared styles, fixed shapes) from layout (placeholder arrangement) (SQ1, SQ2).
- Layout kind: `sldLayout@type` from ST_SlideLayoutType "may be used by the application to aid in mapping between different layouts"; `matchingName` serves "layout matching in response to layout changes and template applications" (SQ1).
- Placeholder role: `ph@type` (title, body, pic, dt, ftr, sldNum ...) links layout to master; `ph@idx` links slide to layout and is "used when applying templates or changing layouts" (SQ1). Google needs the same pair (`type`, `index`) to address a layout placeholder in `placeholderIdMappings` (SQ2).
- Placeholder order: PowerPoint fills content into placeholders by creation order, not name, when switching layouts (Korchok, SQ1).
- Inherited vs overridden geometry: an empty `<p:spPr/>` means inherit; any `<a:xfrm>` on the slide pins it, and reapplying the layout removes it (SQ1; python-pptx vs PptxGenJS local XML).
- Footer trio: PowerPoint does not clone `dt`, `ftr`, `sldNum` onto new slides (SQ1); Korchok recommends the Footer placeholder for per-section text, filled via Insert > Header & Footer (SQ1).
- Text styles: master `titleStyle`/`bodyStyle`/`otherStyle`, nine levels each (SQ1); the spec's Text block has "bulleted and numbered lists with levels" [source: /home/pascal/Code/.worktrees/decks/research/docs/spec-v0.md | decks spec v0, Blocks | practitioner].
- Theme colours: OOXML and Google both fix twelve named slots (dk1/lt1/dk2/lt2, accent1..6, hlink, folHlink); text refers to `schemeClr` (tx1, bg1, accent1) through `clrMap`. The default preset's tokens are named by role (`--brand-green`, `--brand-ink`, `--brand-ember`, ...) with a semantic layer on top, not by OOXML slot [source: /home/pascal/Code/pgoell-claude-tools/plugins/presentations/presets/default/colors.css | default preset colors.css | practitioner].
- Theme fonts: major (headings) and minor (body) faces, referenced as `+mj-lt`/`+mn-lt` (SQ1 local experiment); PptxGenJS exposes the same two as `headFontFace`/`bodyFontFace`.
- Non-placeholder master content (logo, rules): lives in master or layout `cSld`; `showMasterSp` controls whether master shapes show on a layout's slides (SQ1). Keynote turns such objects into uneditable background (SQ3).
- Content types allowed in placeholders differ: OOXML allows auto shape, picture and graphic frame (table, chart) placeholders; Google allows only shapes; Keynote allows one object placeholder per layout (SQ1, SQ2, SQ3).
- Raw HTML/SVG/Script blocks: spec v0 exports them as images [source: /home/pascal/Code/.worktrees/decks/research/docs/spec-v0.md | decks spec v0, Export | practitioner]; a picture can be a `pic` placeholder in OOXML (SQ1), not in Google (SQ2).

### Counter-evidence and limits

Applying edits to masters after the fact does not reach existing slides in Figma Slides at all, and in PowerPoint layout edits need a reapply, which resets geometry (SQ1, SQ3).

Reapply and layout switch in PowerPoint can put content into the wrong placeholder or orphan it at the slide edge (SQ1, Microsoft Q&A).

Renderers resolve inheritance differently: LibreOffice applied master text box settings that PowerPoint did not, and vice versa, depending on `idx` (bug 104202, SQ1). The plugin's verify loop runs on LibreOffice (SQ1).

No source found that tests PowerPoint's Change Layout on a deck made by PptxGenJS or by python-pptx from a generated template; no render or PowerPoint test was run here.
