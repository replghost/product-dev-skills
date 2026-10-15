---
name: slidedoc
description: Use when writing, building, converting or reviewing a slidedoc, a deck meant to be read rather than presented (pre-read, briefing, "send me your slides", leave-behind, reference doc). Applies Duarte's slidedoc method and William's house style.
---

# Slidedoc

A slidedoc is a document built in slide software where words and visuals together make **one clear point per page**. People read it on their own, without a presenter: before a meeting, in the first ten minutes of one, or forwarded to someone who was never in the room. It sits between a dense document, which is exhaustive and topical, and a cinematic presentation, which is sparse and spoken. It should be explanatory, modular, educational, visually reasoned and quick to understand. (Source: Nancy Duarte, *Slidedocs*, Duarte Inc. 2014.)

Use this skill for the content and the design. The build mechanics come from the output format:
- **Default:** the Slides artifact type. Create from the type, then follow the instructions it returns for files, calls and the slide format. This skill decides what goes on each page and how it looks.
- **PowerPoint:** use only when the user asks for .pptx. Read the pptx skill, then apply everything below.
- **Revising an existing slidedoc**, such as the ECOFIN Opportunity Overview: read `project/deck.json` and the slides first, keep the deck's established system, and change only what was asked.

---

## 0. Decide whether it should be a slidedoc

Ask: *Do people need to hear this from you directly?* If yes, make a presentation. *Does it need a lot of detail to be understood?* If yes, make a slidedoc.

Good fits: proposals, research findings, briefs and memos, strategy and execution plans, status updates, board and committee pre-reads, client collateral, appendices, and anything someone asks you to "send along".

If the person wants to present it to more than about 6 people, or to an external audience, suggest a separate cinematic deck built from the slidedoc's key findings. Never present a slidedoc by reading it aloud.

## 1. Pin down the audience, use and Big Idea before writing

Settle these first. Ask only for what you can't infer from the request.

1. **Reader:** who they are, what they already know, what they worry about, and what's in it for them.
2. **Use and length budget.** These come from Duarte and are hard ceilings:

| Use | Length |
|---|---|
| Pre-read sent before the meeting | Readable in **≤ 20 min** |
| Read at the start of the meeting | **≤ 10 pages** (under 10 min) |
| Emissary ("send me your slides"), for an executive | **≈ 5 pages** |
| Prospect or client collateral | **≤ 10 pages** |
| Handed out during a talk | **≤ 10 pages**, only what the talk covers |
| Reference material, before or after a talk | Up to **50 pages** |
| Follow-up | 10 pages for collateral, up to a book-length slidedoc |

   Put detail beyond the budget in **appendices** after the main body. The main body has to work on its own.
3. **Big Idea:** one complete sentence that states (a) your point of view and (b) what's at stake for the reader if they adopt it or don't. A topic like "Q4 strategy" is not a Big Idea. It goes on the cover subtitle and frames the summary.

## 2. Write the spine: headlines that tell the argument

Draft the **headline of every page first**, as a list, before writing any body copy. This is the outline view. Check it:

- Read in order, the headlines make a complete, persuasive argument by themselves.
- Each headline states the **point**, not the category. Write "Fast network routers speed time to market", not "Network router options".
- Use a topical structure, not a Roman-numeral outline. Each headline should let the reader predict what the page says.
- Each headline fits in **two lines or fewer**. Cut the fluff if it runs longer.
- If an idea doesn't need its own page, fold it into another page or cut it.

Once the body is written, test every page: Does the copy support the headline? Does the headline support the Big Idea? Is it concise? If a page fails the second test, delete it.

Editorial process for high-stakes docs:
1. **Ideate:** sections and page titles.
2. **Create:** dump everything into each page. Aim for quantity.
3. **Refine:** tighten, re-chunk, and check the flow again.

## 3. Write the copy

- **Use full sentences.** No presenter will fill in the gaps, so fragmentary bullets fail. Bullets are fine when they are full sentences in parallel structure.
- **Use active voice.** Watch for forms of "to be". "We developed it in 9 months" beats "It was developed in 9 months".
- **Word budget:** about **100 words per page** is concise, and that is the default. **175** is the usual maximum. **250** is the dense ceiling. Above that, it should be a document, not a slidedoc. Tables and data pages may run denser if they scan well.
- **Make the copy scannable.** Use this hierarchy on a page:
  1. Headline
  2. Subhead or thesis line
  3. Short paragraphs, each one complete thought
  4. Full-sentence bullets
  5. A pull-quote or boxed callout for the key line
- **Persuade, don't just inform:**
  - Make the benefit to the reader explicit.
  - Cite examples and proof.
  - Use an analogy where it helps.
  - Allow some emotional appeal.
- **Purge.** Highlight what the reader must take away and cut the rest. Cutting 60% is normal: "brevity is a by-product of vigor."
- **Never invent figures, quotes or sources.** Use bracketed placeholders such as `[€__]` and list them in your reply.

## 4. Architecture, borrowed from books

Every slidedoc over about 6 pages gets these parts:

| Element | Rules |
|---|---|
| **Cover** | A conceptual, interesting title (not "Q4 Strategy"), a subtitle carrying the Big Idea, the author, and a status date. The logo is small and discreet, the way a publisher's mark sits on a book spine. Optional kicker: "Pre-read · Board and funding committee". |
| **Contents** | Written last, from the headlines. Reading the contents alone should convey the argument. List section names, short page descriptions and page numbers. Add links where the format supports them. |
| **Summary / intro** | Near the front for pre-reads. If you can't write it from the headlines, the structure is broken. |
| **Section heads** | A clearly different layout for each new section (an inverted colour flood, large type), so the reader knows they've changed chapter. |
| **Section signifiers** | A kicker on each page naming the section ("Appendix C · Competitive Analysis"). Tabs, colour bars or a navigation diagram are optional extras. |
| **Footer** | On every page: document title · confidentiality · date, plus the **page number**. The footer travels with any page that gets copied out, and page numbers let people say "turn to page 12" in the meeting. |
| **End matter** | Whichever of these apply: summary, discussion questions, glossary, appendix, sources, next steps or asks. |

Keep the figure and its text on the **same page**. Never write "see figure 4 on page 9".

## 5. Design system

- **One visual language, used throughout:**
  - 3–5 colours plus a neutral and a highlight colour.
  - At least two neutral greys.
  - One illustration style and one photo style.
  - No clip art.
  - Pages "Frankensteined" from different decks erode trust.
- **Grid.** Lay pages on a consistent column grid. Duarte uses 4×3. On 16:9, use 12 columns and pick from a few fixed splits: full width, 1:1, 3:2, 5:2, and a narrow meta column beside a wide body. Never run body text across the full page width.
- **Line length:** **50–70 characters** (12–15 words). If a line runs longer, split it into columns.
- **Text hierarchy.** Define it once, before layout, and never improvise:
  - The headline dominates.
  - The subtitle is smaller than the headline but larger than the body.
  - Subheads are set apart by weight or colour.
  - Body copy is neutral and easy to read.
  - Callouts use italic or a colour.
  - The footer is the least dominant element.
  - Set things apart with weight, case, italics and size before colour. Avoid a "text rainbow".
- **Typesetting:**
  - Left-align text and never justify it.
  - Leave generous line spacing (about 1.4).
  - Add space between paragraphs.
  - Fix widows and ragged lines with manual breaks.
- **White space.** Give key points room. Not every page should be all text. The deck as a whole needs some breathing pages.
- **Light backgrounds** for reading pages, since they are easier to read and print. Dark floods are for section heads only.
- **Static.** No animation or builds: slidedocs get printed and read as PDFs.
- **Speaker notes = the appendix that travels with the page.** Put sources, caveats, verbatim quotes and the reasoning behind ratings in the notes, so the page stays clean and the evidence stays attached.

## 6. Data and diagrams

- **Data:** state the conclusion in the headline and the prose. Never just drop a chart onto the page. A chart has three layers:
  1. Background (axes and grid) in neutral colours.
  2. All the data in a neutral colour.
  3. The thing to notice in **one** highlight colour.
- No 3D, no gradients, and no decoration. Analytical readers distrust anything that looks like "marketing data".
- **Diagrams are content.** Choose the relationship type on purpose:

| Type | Shows | Examples |
|---|---|---|
| **Flow** | Direction | Process, cycle, timeline, sequence |
| **Network** | Hierarchy or connection | Tree, hub-and-spoke, mind-map, ecosystem |
| **Segment** | Parts of a whole | Pie, donut, core |
| **Stack** | Order or magnitude | Bars, concentric circles |
| **Join** | Overlap or shared values | Venn, puzzle, link |

  Proximity, scale and connectors carry meaning, so check that they say what you intend.
- Prefer clear geometry to clever geometry. For example, Duarte replaced swooping arrows with concentric circles to show tsunami arrival times.

## 7. Converting existing material

- **Dense document → slidedoc:** split it into one idea per page, turn the topics into point headlines, turn passages into pictures and tables, and add callouts.
- **Overloaded slides → slidedoc:** merge a topic that's spread across several slides onto one page. Pull essential content up from the speaker notes. Unify the graphic style.
- **Cinematic slides → slidedoc (leave-behind):** put each slide image beside the prose of what was said about it, as in notes-page view. A sparse slide that circulates without its talk gets misread.
- **Teleprompter-length slides:** either cut them down to a true slide or expand them into a stand-alone slidedoc page. Don't leave them in between.

## 8. Review checklist (run before handing over)

Work through these from the outline view first, then page by page:

- [ ] Do the headlines alone tell the argument, and does the contents page convey it?
- [ ] Is the Big Idea stated as a sentence on the cover and in the summary?
- [ ] Is the main body within the length budget, with the rest in appendices?
- [ ] Is there one point per page, and does every sentence serve it?
- [ ] Are pages around 100 words (≤ 175, never > 250), in full sentences and active voice?
- [ ] Does my eye know what to read first, and is the main message easy to pick out?
- [ ] Is the text big enough for the reading medium?
- [ ] Are there too many fonts or colours? Is the grid consistent, and does each repeated element sit in the same place on every page?
- [ ] Are there some white-space pages, rather than a wall of all-text pages?
- [ ] Is each section head distinct, and does each page have its kicker, footer, confidentiality line and page number?
- [ ] Are the **page numbers in the contents and section heads still correct** after the last reorder?
- [ ] Are charts highlighted on the single point they make? Is there no 3D or decoration?
- [ ] Is every number, quote and date backed by a source (in the notes or a Sources page)? Are placeholders listed for the user?

For a high-stakes doc, have a separate agent that hasn't seen the drafting critique it against this checklist. It should check titles, flow, what to cut, flaws in reasoning, and the examples.

---

## House style (William / Parity): the default unless the user or a design system says otherwise

This style is taken from the *ECOFIN Opportunity Overview* slidedoc, a Slides artifact. Reuse it for internal Parity slidedocs so that his documents look like one family.

**Canvas and margins:** 1920×1080.
- Content pages: `padding: 96px 112px 136px`.
- Cover: `padding: 112px`.
- Footer band pinned at `bottom: 56px`.

**Typefaces:**
- *DM Serif Display* 400 for the cover, divider and page headlines.
- *DM Sans* for everything else.
- Load both as Google Fonts faces in `deck.json`.

**Palette** (monochrome; black is the only emphasis colour):

| Token | Hex | Use |
|---|---|---|
| paper | `#ffffff` | Page background |
| ink | `#000000` | Headlines; highlight fills (callout panel, stat chips, highlighted table row); section-head background |
| body | `#333333` | Body copy |
| meta | `#6b6b6b` | Kickers, captions, footer, page numbers |
| rule | `#b3b3b3` | Hairlines, table borders |
| tint | `#f0f0f0` | Zebra rows, soft side panels |
| on-ink body / meta | `#d9d9d9` / `#a6a6a6` | Text on black section heads |

**Type scale (px):**

| Element | Font | Size / line height | Weight / notes |
|---|---|---|---|
| Cover title | Serif | 104 / 114 | — |
| Section-head title | Serif | 112 | — |
| Page headline | Serif | 46 / 53 | Letter-spacing 0.2px |
| Summary headline | Serif | 52 | — |
| Statement / pull-quote | Sans | 36 / 45 | 500 |
| Block title (h3) | Sans | 28 / 34 | 600 |
| Body | Sans | 24 / 33.6 | Never smaller |
| Kicker | Sans | 24 | 500, meta colour |

**Page anatomy:**
- **Top:** a kicker (`Section` or `Appendix C · Competitive Analysis · Comparison`), then a gap of about 28px, then the headline. The headline is a full-sentence assertion of up to two lines.
- **Body:** columns in a 3:2 or 1:1 split with a gap of 52–72px.
- **Footer**, in three parts:
  - Left (`left: 112px`, width 1200, meta colour): `<Doc title> · Internal · <d Mon yyyy>`
  - Page number: right-aligned at `right: 290px`.
  - Parity logo: `right: 112px; bottom: 54px; 146×40`. Use the dark logo on white pages and the light logo on black pages, both reused from the existing deck's blobs.
- **Notes (`<aside>`):** sources with dates, paraphrase flags, caveats and "what's not decided".

**Recurring components:**
- **Quote / requirement:** a left rule `border-left: 2px solid #000`, the statement at 36px, and an attribution line in meta.
- **Before → after:** two 1px-bordered boxes joined by an `arrow-right` shape.
- **Callout panel:** black fill with white h3 and list. Use one per page at most.
- **Stat chips:** a row of black boxes with white text, such as **7 calls** or **~12 months**.
- **Labelled rows (summary):** the h3 in a 200px left column and the paragraph on the right, separated by `border-top` rules. The first rule is 2px black, the rest 1px `#b3b3b3`.
- **Tables:** 1px `#b3b3b3` border and zebra `#f0f0f0` rows. Set the row you are arguing *for* in black with white text.
- **Timeline:** a 9-column grid with `border-top: 4px solid #000` on each step. Fill the current step black. Use a dashed meta-grey border for future steps.
- **Meeting-log page:** a 340px left meta column (log position, date, attendees, status) and a body with "What was discussed" on the left and a tinted panel for "Outcome / Actions" on the right.
- **Section head (divider):** a black page with a 112px serif title. Below it, lettered or numbered rows, each holding a title, a one-line description and a page number, separated by `border-top` rules.

**Standard order for an opportunity or proposal pre-read:**
1. Cover
2. Contents
3. One-page summary (what it is / where we are / what happens next, plus stat chips)
4. The case, in this order:
   1. Market
   2. Problem
   3. Opportunity
   4. Partner
   5. Relevance to Polkadot
   6. First step
   7. Why us/here
   8. Path
   9. Structure
   10. Unknowns and kill criteria
   11. Status
   12. Asks / next steps
5. Section head: Appendices
6. Technical design
7. Deep dives
8. Competitive analysis
9. Meeting log
10. Sources

**Page numbers are typed by hand into each slide, the contents page and the section heads.** After adding, removing or reordering pages, renumber every footer and re-check the page references on the contents and section heads in the same edit.
