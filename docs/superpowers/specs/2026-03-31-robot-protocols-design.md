# Robot Protocols Panel — Design Spec
**Date:** 2026-03-31
**Project:** Team Hyperion Engineering Logbook (`logbook.html`)

---

## Overview

Add a "Robot Protocols" panel to the Engineering Logbook sidebar. This panel acts as a living troubleshooting manual: each entry represents a named robot behavior (e.g., "Drive Train Jitter") paired with an ordered checklist of diagnostic/fix steps. Step completion is a local session toggle (scratchpad use), not persisted. All write operations are member-only.

---

## Architecture

### Firestore Collection: `protocols`

Each document:
```js
{
  id: "<auto>",
  title: "Drive Train Jitter",       // required, behavior name
  description: "...",                // optional short summary
  steps: [
    { text: "Check motor wiring", done: false },
    { text: "Inspect encoder cables", done: false }
  ],
  createdAt: serverTimestamp(),
  updatedAt: serverTimestamp(),
  createdBy: "Sam"                   // display name of creating member
}
```

- `steps.done` is initialised `false` on save and reset to `false` on every `onSnapshot` update (local-only toggle).
- `onSnapshot` on `collection(db,'protocols')` ordered by `createdAt asc` keeps the view live.
- All writes gated behind `requireMember()`.

### State additions
```js
let protocols = [];           // live array from onSnapshot
let protocolSearch = '';      // current search filter string
let editingProtocolId = null; // null = new, string = editing existing
```

---

## Sidebar Integration

Insert after the Chronicles section divider, before the Categories section:

```html
<div class="sb-divider"></div>
<div class="sb-section">Resources</div>
<button class="sb-btn" id="sb-protocols" onclick="setView('protocols')">
  <span class="sb-dot" style="background:#ffd700"></span>Protocols
  <span class="sb-count" id="cnt-protocols">—</span>
</button>
```

Dot colour: `#ffd700` (amber) — visually distinct from green (entries) and blue (requests).

---

## View: `#view-protocols`

### Header bar
- Glitch title: `ROBOT <span>PROTOCOLS</span>`
- `.lb-search` input wired to `protocolSearch` — instant client-side filter on `title` + `description`
- "Add Protocol" button: `clip-path` polygon, green, hidden (`display:none`) for guests — calls `openProtocolModal()`

### Protocol card list
- Vertical stack, same gap as `.entries-grid`
- Each card uses `.entry-card` + `.stat-card-glitch` for corner brackets
- Card header row: behavior title (Rajdhani bold), step-count badge (e.g. `3 steps`), chevron icon (▶/▼)
- Clicking the header row toggles `.expanded` — reveals the checklist body
- Member-only action buttons (pencil / trash) in the card's `.entry-actions` column, hidden for guests

### Checklist (expanded body)
- Each step is a `<label>` with a styled `<input type="checkbox">` + step text
- Checking/unchecking updates local DOM state only (no Firestore write)
- Steps rendered in array order

### Empty state
- If no protocols exist: centered message `[ NO PROTOCOLS DEFINED ]` in muted green mono text
- If search yields no results: `[ NO MATCHES ]`

---

## Modals

### Add / Edit Protocol Modal (`#protocol-modal-bg`)

Reuses `.modal-bg` / `.modal` / `.modal-hdr` / `.modal-body` / `.modal-footer` pattern exactly.

**Fields:**
1. **Behavior Name** — text input, required
2. **Description** — textarea, optional, 2 rows
3. **Steps** — dynamic list:
   - Each step row: text input + remove button (`✕`)
   - "Add Step" button appends a new empty row
   - Steps can be removed individually (if only one step remains, remove button is hidden)

**Footer:**
- Left: Delete button (red, member-only, only rendered when `editingProtocolId !== null`) — calls `deleteProtocol(id)`, closes modal, shows toast. This is a secondary delete path; the primary delete is the trash icon on the card itself.
- Right: Cancel + Save buttons

> **Clarification:** The card's `.entry-actions` column contains the pencil (edit) and trash (delete) buttons, visible in both collapsed and expanded state for members. The modal footer delete button is a convenience duplicate shown only in edit mode — not on new protocol creation.

**Submit handler (`saveProtocol`):**
- Validates title is non-empty
- Builds `steps` array from current input values (filters out blank entries), each with `done: false`
- On new: `addDoc(collection(db,'protocols'), payload)`
- On edit: `updateDoc(doc(db,'protocols', editingProtocolId), {...payload, updatedAt: serverTimestamp()})`
- Closes modal, shows toast

### Delete
- `deleteProtocol(id)`: calls `deleteDoc`, shows toast `'Protocol deleted'`
- No confirmation modal — matches existing `deleteEntry` pattern (just a toast)

---

## `setView` Integration

Update the hardcoded view-ID array and add a `protocols` branch:

```js
// In the reset block:
['view-dashboard','view-entries','view-chronicles','view-requests','view-tt','view-protocols']
  .forEach(id => document.getElementById(id).style.display = 'none');

// New branch:
} else if (v === 'protocols') {
  document.getElementById('sb-protocols')?.classList.add('active');
  document.getElementById('view-protocols').style.display = 'block';
  renderProtocols();
}
```

`updateStats()` should also update `#cnt-protocols` with `protocols.length`.

---

## CSS

New styles scoped to protocols — no changes to existing rules:

- `.protocols-grid` — flex column, `gap: 2px` (matches `.entries-grid`)
- `.protocol-card` — extends `.entry-card` base look; left stripe `#ffd700`
- `.protocol-step-list` — `list-style: none`, padding reset
- `.protocol-step` — flex row, gap, checkbox styled to match cyberpunk aesthetic (custom `accent-color: var(--green-bright)`)
- `.protocol-step.done` — text has `opacity: 0.4`, `text-decoration: line-through`
- `.proto-add-btn` — matches `btn-lb-new` clip-path polygon style, but full-width in view header
- `.protocol-step-row` — modal step row: flex, gap, input + remove button
- Step-count badge reuses `.sb-count` class

All fonts: `Share Tech Mono` for metadata/steps, `Rajdhani` bold for titles, `Bebas Neue` for section headers.

---

## Access Control Summary

| Action | Guest | Member |
|---|---|---|
| View protocols | ✓ | ✓ |
| Expand checklist | ✓ | ✓ |
| Toggle step done (local) | ✓ | ✓ |
| Add protocol | ✗ | ✓ |
| Edit protocol | ✗ | ✓ |
| Delete protocol | ✗ | ✓ |

Enforcement: `requireMember()` called at the start of `openProtocolModal()`, `saveProtocol()`, and `deleteProtocol()`. Edit/delete buttons hidden via `display:none` when `!isMember`.

---

## Out of Scope

- Step reordering (drag-and-drop) — not required
- Per-step persistence of `done` state — local toggle only
- Protocol categories/tags — can be added later if needed
- PDF export inclusion — not required in this iteration
