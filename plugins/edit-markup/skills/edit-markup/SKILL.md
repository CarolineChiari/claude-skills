---
name: "edit-markup"
description: "Produce teacher-style markup and clean rewrites of user-authored text such as Quora answers, Substack drafts, essays, and emails. Use this skill WHENEVER the user explicitly asks to rewrite, revise, edit, or produce a corrected version of something they wrote, or says \"give me the version you would write,\" \"apply your edits,\" \"rewrite this,\" \"fix this up,\" \"produce a clean version,\" \"show me the edited version,\" or similar. Output is a ruled-paper HTML artifact with two panels: a marked-up version (red strikethroughs for cuts, green highlights for inserts, numbered carets linking to handwriting-style margin notes explaining the reasoning) and a clean final version at the bottom. DO NOT use for general feedback sessions where the user just wants comments on their draft; only when they explicitly want a rewritten version produced. Preserves as much of the user's original language as possible; edits should feel like targeted teacher markup, not a ground-up rewrite."
---

# Edit Markup Skill

This skill produces teacher-style markup revisions of user-authored text
as a ruled-paper HTML artifact. It has two jobs:

1. **Visually separate** what was cut, what was inserted, and what was
   kept, so the user can see every change at a glance.
2. **Explain the reasoning** for each change in margin notes, so the
   user learns the editorial logic rather than just receiving a new
   version.

The output is a single HTML file saved to `/mnt/user-data/outputs/` and
presented via `present_files`.

---

## When to use this skill

**Use when the user says:**
- "give me the version you would write"
- "apply your edits / apply these changes"
- "rewrite this"
- "produce a clean version"
- "show me the edited version"
- "can you fix this up"
- "incorporate that / add it to the artifact"
- "update the artifact"

**Do NOT use when the user:**
- Just wants critique or feedback ("what do you think of this?")
- Is brainstorming or exploring ideas
- Wants a short inline suggestion
- Explicitly asks for prose-style feedback

If in doubt, ask. The artifact is expensive to produce; don't fire it
off when the user wanted a conversation.

---

## Core editorial philosophy

**Preserve the user's voice.** The marked-up version should make it
obvious that most of the original is intact. If more than ~40% of the
original is struck through, reconsider whether a full rewrite is what
was asked for, or whether the user wanted something less invasive.

**Every change needs a margin note.** The numbered carets exist to
teach. A note that says "made this shorter" is useless. A good note
explains *why*: what was broken about the original, what the change
accomplishes, what rhetorical or factual principle it illustrates.

**Changes must be defensible.** Don't change things for cosmetic
consistency or style preference. Change them because something was
factually off, rhetorically weak, structurally broken, or out of
register with the surrounding piece. If you can't articulate the
reason in a margin note, don't make the change.

**Keep the close.** The final line or paragraph of a user's piece is
usually what they care most about. Unless it's clearly broken, leave
it verbatim. A margin note that says "Kept verbatim. The close does
the resonance work and doesn't need changing" is a legitimate note.

---

## Structure of the artifact

The HTML contains, in order:

1. **Title** (e.g., "✎ Revision — [piece name]")
2. **Legend** explaining the color coding
3. **Marked-up panel** with:
   - Optional question or prompt callout at top (italic, left-rule)
   - Two columns per row: text on the left (~64%), margin notes on the
     right (~32%)
   - Red strikethrough for cuts, green highlight for inserts, numbered
     red circle carets linking to margin notes
   - Ruled paper background (light blue lines, red vertical margin)
4. **"Clean revised version" heading**
5. **Clean panel** with the final text, no markup, on white background
   with a green left border (signaling "approved version")
6. **Copy button** directly above or below the clean panel that copies
   the clean text to the clipboard (see "Copy button" below)

Both panels should include the question callout if there is one.

---

## The ruled-paper CSS alignment formula

**This is the single most fiddly part of the skill. Get it right.**

The visual effect requires text to sit ON the ruled lines, not float
above or cut through them. The math:

### Rules

1. **`line-height` must be a fixed pixel value** equal to the
   background repeat distance. E.g., `line-height: 28px` with a 28px
   background repeat. Do NOT use ratios like `1.5` or `1.55`. They
   compute to fractional pixels that drift off-grid.

2. **Every margin and padding must be 0 or a multiple of the grid.**
   Paragraph margins, row gaps, notes gaps, page padding. If you need
   breathing room between paragraphs, use one full grid unit (e.g.,
   `margin-bottom: 28px`), not 10px or 14px.

3. **The ruled line goes at the baseline position within each cell,
   not the bottom.** Text baselines sit at roughly 70-75% down from
   the top of a line-box, depending on font. For Crimson Pro at
   17px/28px, the baseline is at y=20. So the gradient should place
   the horizontal line at y=20-21, NOT y=27-28.

4. **Both the text column and notes column need the same fixed
   line-height**, even if they use different fonts. Otherwise the
   margin notes drift off the grid even if the body stays aligned.

### Template gradient

```css
background:
  linear-gradient(to right, transparent 0, transparent 58px,
    #e8b0b0 58px, #e8b0b0 60px, transparent 60px),
  repeating-linear-gradient(to bottom,
    transparent 0,
    transparent 20px,   /* space above ruled line */
    #cfe0ea 20px,
    #cfe0ea 21px,       /* the ruled line itself, 1px thick */
    transparent 21px,
    transparent 28px);  /* remainder of cell */
```

### Tuning

If after rendering, text floats ABOVE the line → raise the y value
(e.g., 20 → 22). If text cuts THROUGH the line → lower it (20 → 18).
Tune in 1-2px increments. Different fonts need different values; the
70-75% rule is a starting point, not a rule.

### Fonts

- Body text: **Crimson Pro** (serif, reads as typed/written copy)
- Headings and margin notes: **Caveat** (handwriting, signals the
  "teacher" register)

Both loaded from Google Fonts.

---

## Copy button

The clean version is what the user pastes into Quora, Substack or an
email, often from her phone, so it always gets a Copy button. This
pattern is tested and works in the Claude mobile app.

- Give the clean panel `id="clean"`. Copy its `innerText` so paragraph
  breaks survive; leave the question callout out of the copied text
  (give it its own element outside `#clean`, or skip it in script).
- Call the clipboard only from the click handler, try the Clipboard API
  first, fall back to a hidden-textarea `execCommand('copy')`, and show
  the result in a status line (never `alert()`; it is blocked).
- Style the button to match the page: full width on mobile, green to
  echo the clean panel's border, 16px+ text, at least 44px tall.

```html
<button id="copyBtn" type="button">Copy clean version</button>
<div id="copyStatus" aria-live="polite"></div>
<script>
  function fallbackCopy(text) {
    const ta = document.createElement('textarea');
    ta.value = text; ta.setAttribute('readonly', '');
    ta.style.position = 'fixed'; ta.style.opacity = '0';
    document.body.appendChild(ta);
    ta.select(); ta.setSelectionRange(0, text.length);
    let ok = false;
    try { ok = document.execCommand('copy'); } catch (e) {}
    document.body.removeChild(ta);
    return ok;
  }
  document.getElementById('copyBtn').addEventListener('click', async () => {
    const text = document.getElementById('clean').innerText.trim();
    const status = document.getElementById('copyStatus');
    try {
      await navigator.clipboard.writeText(text);
      status.textContent = '✓ Copied';
    } catch (e) {
      status.textContent = fallbackCopy(text) ? '✓ Copied' : '✗ Copy blocked; select the text and copy manually';
    }
  });
</script>
```

---

## Markup conventions

### Cuts

```html
<span class="cut">text that was removed</span>
```

Renders as red strikethrough. Keep cuts in context — show the reader
what was there, so they can see what the change is against.

### Inserts

```html
<span class="insert">newly added text</span>
```

Renders with a pale green highlight and a green underline. The
underline is important: highlight alone reads as "highlighted for
emphasis," underline signals "insertion."

### Margin note carets

```html
<span class="caret">3</span>
```

A small filled red circle with a white number. Place at the END of
the clause or sentence it refers to. If a single edit has multiple
concerns, use multiple carets in sequence: `<span class="caret">6</span><span class="caret">7</span>`.

### Margin notes

Handwritten-style (Caveat font), red ink. Numbered to match carets.
Length: 2-5 sentences. Each note should teach something, not just
describe the change.

**Good notes:**

> "Your original developmental-history framing was right; my earlier
> rewrite was wrong (that's the old policy, not the current one).
> Quoting the policy back at itself exposes the justification-policy
> mismatch."

> "Reframed 'zero studies' so it's unfalsifiable. Preempts the hostile
> reader who'll Google and find Hilton & Lundberg; now those don't
> count against you because they studied cis men, which is your point."

**Bad notes:**

> "Changed wording for clarity." *(tells the user nothing)*
> "Made this better." *(tells the user nothing)*
> "Shorter is better." *(not a reason; it's a preference)*

---

## Process

### Step 1: Confirm scope

Before producing the artifact, make sure you know:

- What piece is being rewritten (paste or reference)
- What the user wants changed vs. preserved
- Any non-negotiables (specific phrases to keep, tone to maintain)
- Whether there's a prompt/question the piece is answering (if so,
  include it as a callout at the top of both panels)

### Step 2: Build the marked-up version

Work paragraph by paragraph. For each unit of text:
- Decide what to keep (most of it, in almost all cases)
- Wrap cuts in `<span class="cut">`
- Wrap inserts in `<span class="insert">`
- Add caret(s) at the end of the sentence containing the change
- Write the matching margin note

If a paragraph is unchanged, you can either include it plain (no
row/notes pair needed) or flag it with a "kept verbatim" note.

### Step 3: Build the clean version

Take the marked-up text and resolve it: remove cuts entirely, accept
inserts as plain text, drop the carets. This is what the user will
actually copy. Make sure no markup classes leak in. Add the Copy
button wired to the clean panel, and check that the copied text has
no carets, cut text or margin notes in it.

### Step 4: Save, present, summarize

Save to `/mnt/user-data/outputs/[descriptive-name]-revision.html`.
Call `present_files` on it. Then in chat, give a brief summary of
substantive changes (3-7 bullet points), each phrased as what
changed and why — NOT "I updated step 6," but "Step 6 now uses the
IOC's actual policy language instead of the prior framework's, so a
hostile reader can't wave it off as misquoting."

---

## Template file

A ready-to-use template with the correct CSS, font loading, and
structure is bundled at `references/template.html`. Start from this
and fill in the `{{TITLE}}`, `{{QUESTION}}`, `{{MARKED_UP}}`, and
`{{CLEAN}}` placeholders.

---

## Common pitfalls

- **Don't change the user's closing line unless it's broken.** The
  close is often load-bearing emotionally; an "improved" close that
  loses the user's voice fails the whole piece.
- **Don't pile carets on a single sentence.** If a sentence needs
  four margin notes, it probably needs to be a discussion in chat,
  not a markup edit. Carets should feel like targeted observations,
  not a swarm.
- **Don't rewrite the user's whole thesis in the margin notes.** If
  there's a fundamental disagreement about what the piece should
  argue, raise it in chat BEFORE producing the artifact, not through
  ten margin notes that collectively say "I would have written
  something different."
- **Don't use em dashes in the output.** Per user preference, use
  semicolons or restructure the sentence.
- **Don't forget the question callout** if the piece is answering a
  specific prompt (Quora answer, interview question, prompt-response).
  The question anchors the critique; without it, cuts and inserts
  float free of context.
- **Don't hardcode line-height ratios.** Fixed pixel line-heights
  only. See the CSS alignment formula above.
- **Don't forget the Copy button** on the clean version, and keep it
  working after edits: if you restructure the clean panel, keep
  `id="clean"` on it.

---

## After producing the artifact

The user may ask for edits *to the artifact itself* — add something,
remove a cut, rewrite a margin note, apply ™ to three spots, etc.
Use `str_replace` on the HTML file rather than regenerating from
scratch. Preserve everything the user didn't ask to change.

When making changes to both the marked-up and clean versions,
remember they're separate sections of the same file. A change to one
without the other creates inconsistency; always update both unless
the user specifically wants the marked-up panel to show the before
state of an edit the clean panel reflects.