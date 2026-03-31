# Robot Protocols Panel — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a member-editable "Robot Protocols" panel to the Engineering Logbook — a searchable directory of named robot behaviors with expandable diagnostic checklists, backed by a Firestore `protocols` collection.

**Architecture:** All changes are in `logbook.html` (single-file app). Protocols are stored in a Firestore `protocols` collection using the same `onSnapshot` + `addDoc`/`updateDoc`/`deleteDoc` pattern as entries and requests. Step-completion state is a local DOM toggle only — never written to Firestore. All write operations are gated behind `requireMember()`.

**Tech Stack:** Firebase Firestore v10, vanilla JS ES modules, Share Tech Mono / Bebas Neue / Rajdhani fonts, CSS clip-path polygon buttons — all already present in the file.

**Note on testing:** This is a browser-only app with no JS test framework. Each task ends with a manual browser verification step. Open `logbook.html` via a local server (e.g. `npx serve .`) — the Firebase SDK requires HTTP not `file://`.

---

## File Modified

- Modify: `logbook.html` (all changes — 8 targeted insertions/edits)

---

### Task 1: CSS — Protocol Panel Styles

**Files:**
- Modify: `logbook.html` (insert CSS before line 1026 `</style>`)

The anchor for insertion is the last CSS rule in the `<style>` block:
```
.notif-dd::-webkit-scrollbar-thumb{background:rgba(0,200,90,.12);}
```

- [ ] **Step 1: Insert CSS before `</style>`**

Find:
```css
/* ── Notif dropdown scrollbar ── */
.notif-dd::-webkit-scrollbar{width:2px;}
.notif-dd::-webkit-scrollbar-thumb{background:rgba(0,200,90,.12);}

</style>
```

Replace with:
```css
/* ── Notif dropdown scrollbar ── */
.notif-dd::-webkit-scrollbar{width:2px;}
.notif-dd::-webkit-scrollbar-thumb{background:rgba(0,200,90,.12);}

/* ══════════════════════════════════════════
   PROTOCOLS VIEW
══════════════════════════════════════════ */
.protocols-grid{display:flex;flex-direction:column;gap:2px;}
.protocol-card{background:var(--card-bg);border:1px solid var(--border);border-left:3px solid #ffd700;position:relative;overflow:hidden;transition:border-color .2s,transform .15s;}
.protocol-card:hover{border-color:rgba(255,215,0,.4);transform:translateX(2px);}
.protocol-card.expanded{border-color:rgba(255,215,0,.5);}
.protocol-card.expanded .protocol-body{display:block;}
.protocol-header{display:flex;align-items:center;gap:9px;padding:11px 13px;flex:1;cursor:pointer;}
.protocol-title{font-family:'Rajdhani',sans-serif;font-weight:700;font-size:.95rem;letter-spacing:.06em;color:var(--white);flex:1;}
.protocol-step-count{font-size:.46rem;letter-spacing:.12em;background:rgba(255,215,0,.1);border:1px solid rgba(255,215,0,.25);padding:2px 6px;color:#ffd700;flex-shrink:0;}
.protocol-chevron{font-size:.6rem;color:var(--grey);transition:transform .2s;flex-shrink:0;}
.protocol-card.expanded .protocol-chevron{transform:rotate(90deg);}
.protocol-body{display:none;padding:12px 16px 16px;border-top:1px solid rgba(255,215,0,.08);}
.protocol-desc{font-size:.66rem;line-height:1.7;color:rgba(240,255,244,.45);margin-bottom:10px;}
.protocol-steps-label{font-size:.42rem;letter-spacing:.22em;text-transform:uppercase;color:#ffd700;margin-bottom:8px;}
.protocol-step-list{list-style:none;margin:0;padding:0;display:flex;flex-direction:column;gap:5px;}
.protocol-step{display:flex;align-items:flex-start;gap:9px;padding:6px 8px;background:rgba(255,215,0,.03);border:1px solid rgba(255,215,0,.07);cursor:pointer;user-select:none;}
.protocol-step input[type="checkbox"]{accent-color:var(--green-bright);width:13px;height:13px;flex-shrink:0;margin-top:2px;cursor:pointer;}
.protocol-step-text{font-size:.67rem;line-height:1.65;color:rgba(240,255,244,.7);transition:opacity .15s,text-decoration .15s;}
.protocol-step.done .protocol-step-text{opacity:.35;text-decoration:line-through;}
.proto-empty{text-align:center;padding:40px 20px;font-family:'Share Tech Mono',monospace;font-size:.58rem;letter-spacing:.18em;text-transform:uppercase;color:rgba(0,200,90,.18);}
.proto-add-btn{font-family:'Bebas Neue',sans-serif;font-size:.9rem;letter-spacing:.14em;text-transform:uppercase;color:var(--black);background:var(--green-bright);border:none;cursor:pointer;padding:9px 18px;clip-path:polygon(0 0,calc(100% - 7px) 0,100% 7px,100% 100%,7px 100%,0 calc(100% - 7px));transition:background .15s;}
.proto-add-btn:hover{background:#00ff6a;}
/* Modal step rows */
.protocol-step-row{display:flex;align-items:center;gap:7px;margin-bottom:6px;}
.protocol-step-row .field-input{flex:1;padding:7px 10px;}
.proto-step-remove{background:transparent;border:1px solid rgba(255,85,85,.2);color:rgba(255,85,85,.5);cursor:pointer;padding:5px 9px;font-size:.7rem;flex-shrink:0;transition:all .15s;}
.proto-step-remove:hover{border-color:#ff5555;color:#ff5555;}
.proto-step-add{background:transparent;border:1px solid rgba(0,200,90,.2);color:rgba(0,200,90,.5);cursor:pointer;padding:6px 12px;font-family:'Share Tech Mono',monospace;font-size:.52rem;letter-spacing:.1em;text-transform:uppercase;margin-top:4px;transition:all .15s;width:100%;}
.proto-step-add:hover{border-color:var(--green-bright);color:var(--green-bright);}
.btn-del-proto{background:transparent;border:1px solid rgba(255,85,85,.25);color:rgba(255,85,85,.6);cursor:pointer;padding:8px 15px;font-family:'Share Tech Mono',monospace;font-size:.55rem;letter-spacing:.1em;text-transform:uppercase;transition:all .15s;}
.btn-del-proto:hover{border-color:#ff5555;color:#ff5555;}

</style>
```

- [ ] **Step 2: Verify CSS loaded**

Open `logbook.html` in the browser. Open DevTools → Elements, search for `.proto-add-btn`. Confirm the rule appears with the correct `clip-path` value.

- [ ] **Step 3: Commit**

```bash
git add logbook.html
git commit -m "feat(protocols): add CSS styles for protocols panel"
```

---

### Task 2: HTML — Sidebar Button

**Files:**
- Modify: `logbook.html` (lines ~1131–1137, sidebar `<aside>`)

- [ ] **Step 1: Insert "Resources" section into sidebar**

Find:
```html
    <div class="sb-divider"></div>
    <div class="sb-section">Export</div>
    <button class="sb-btn" onclick="exportPDF()" style="color:rgba(255,107,53,.7);"><span class="sb-dot" style="background:rgba(255,107,53,.6)"></span>Export PDF Report</button>
```

Replace with:
```html
    <div class="sb-section">Resources</div>
    <button class="sb-btn" id="sb-protocols" onclick="setView('protocols')">
      <span class="sb-dot" style="background:#ffd700"></span>Protocols
      <span class="sb-count" id="cnt-protocols">—</span>
    </button>
    <div class="sb-divider"></div>
    <div class="sb-section">Export</div>
    <button class="sb-btn" onclick="exportPDF()" style="color:rgba(255,107,53,.7);"><span class="sb-dot" style="background:rgba(255,107,53,.6)"></span>Export PDF Report</button>
```

- [ ] **Step 2: Verify sidebar button appears**

Refresh the browser. The sidebar should show a "Resources" section heading with a "Protocols" button bearing an amber dot. Clicking it does nothing yet (no view exists) — that's expected.

- [ ] **Step 3: Commit**

```bash
git add logbook.html
git commit -m "feat(protocols): add Protocols sidebar button"
```

---

### Task 3: HTML — Protocols View & Modal

**Files:**
- Modify: `logbook.html` (insert view before `</main>`, insert modal before `<div class="lb-toast"`)

- [ ] **Step 1: Insert `#view-protocols` before `</main>`**

Find:
```html
      <div id="tt-session-list"></div>
      </div>
    </div>
  </main>
</div>
```

Replace with:
```html
      <div id="tt-session-list"></div>
      </div>
    </div>

    <!-- PROTOCOLS VIEW -->
    <div id="view-protocols" style="display:none;">
      <div class="lb-header-bar">
        <div class="lb-view-title glitch-title" data-text="ROBOT PROTOCOLS">ROBOT <span>PROTOCOLS</span></div>
        <div style="display:flex;align-items:center;gap:10px;">
          <div class="lb-search">
            <span class="lb-search-icon">⌕</span>
            <input type="text" placeholder="Search protocols…" id="proto-search" oninput="protocolSearch=this.value;renderProtocols()"/>
          </div>
          <button class="proto-add-btn" id="btn-add-proto" onclick="openProtocolModal()" style="display:none;">+ Add Protocol</button>
        </div>
      </div>
      <div class="protocols-grid" id="protocols-grid"></div>
    </div>
  </main>
</div>
```

- [ ] **Step 2: Insert protocol modal before `<div class="lb-toast"`**

Find:
```html
<div class="lb-toast" id="lb-toast"></div>
```

Replace with:
```html
<!-- PROTOCOL MODAL -->
<div class="modal-bg" id="protocol-modal-bg" onclick="bgClick(event,'protocol-modal-bg')">
  <div class="modal">
    <div class="modal-hdr">
      <div class="modal-title" id="protocol-modal-title">NEW PROTOCOL</div>
      <button class="modal-close" onclick="closeModal('protocol-modal-bg')">✕</button>
    </div>
    <div class="modal-body">
      <div class="field-group">
        <label class="field-label">Behavior Name <span class="field-req">*</span></label>
        <input type="text" class="field-input" id="proto-title" placeholder="e.g. Drive Train Jitter" maxlength="120"/>
      </div>
      <div class="field-group">
        <label class="field-label">Description <span style="font-size:.42rem;color:var(--grey);">(optional)</span></label>
        <textarea class="field-textarea" id="proto-desc" style="min-height:60px;" placeholder="Brief summary of when this protocol applies…"></textarea>
      </div>
      <div class="field-group">
        <label class="field-label">Diagnostic Steps</label>
        <div id="proto-steps-list"></div>
        <button class="proto-step-add" onclick="addProtocolStep()">+ Add Step</button>
      </div>
    </div>
    <div class="modal-footer">
      <button class="btn-del-proto" id="btn-del-proto" style="display:none;" onclick="deleteProtocolFromModal()">Delete Protocol</button>
      <div style="display:flex;gap:9px;margin-left:auto;">
        <button class="btn-cancel" onclick="closeModal('protocol-modal-bg')">Cancel</button>
        <button class="btn-save" id="btn-save-proto" onclick="saveProtocol()">Save Protocol</button>
      </div>
    </div>
  </div>
</div>

<div class="lb-toast" id="lb-toast"></div>
```

- [ ] **Step 3: Verify HTML structure**

Refresh the browser. Navigate to the Protocols sidebar button — the view should appear (empty, no protocols yet). The modal won't open yet (JS not wired). Check DevTools Elements for `#view-protocols` and `#protocol-modal-bg`.

- [ ] **Step 4: Commit**

```bash
git add logbook.html
git commit -m "feat(protocols): add protocols view and modal HTML"
```

---

### Task 4: JS — State, Collection Reference, and Firestore Listener

**Files:**
- Modify: `logbook.html` (three JS edits)

- [ ] **Step 1: Add state variables (after line ~1580)**

Find:
```js
let sessionResultVal='', milestoneStatusVal='';
```

Replace with:
```js
let sessionResultVal='', milestoneStatusVal='';
let protocols=[];
let protocolSearch='';
let editingProtocolId=null;
```

- [ ] **Step 2: Add Firestore collection reference (after line ~1586)**

Find:
```js
const colEntries  = collection(db,'entries');
const colRequests = collection(db,'requests');
const colSessions = collection(db,'timeSessions');
const colNotifs   = collection(db,'notifications');
const metaRef     = doc(db,'_meta','config');
```

Replace with:
```js
const colEntries   = collection(db,'entries');
const colRequests  = collection(db,'requests');
const colSessions  = collection(db,'timeSessions');
const colNotifs    = collection(db,'notifications');
const colProtocols = collection(db,'protocols');
const metaRef      = doc(db,'_meta','config');
```

- [ ] **Step 3: Add `onSnapshot` listener inside `startFirestoreListeners()` (after line ~1864)**

Find:
```js
  unsubSessions = onSnapshot(query(colSessions,orderBy('clockIn','desc')),snap=>{
    sessions=snap.docs.map(d=>({id:d.id,...d.data()}));
    updateStats();buildSidebar();
    if(currentView==='timetracker')renderTT();
    if(currentView==='dashboard')renderDashboard();
  },err=>console.error('sessions',err));
}
```

Replace with:
```js
  unsubSessions = onSnapshot(query(colSessions,orderBy('clockIn','desc')),snap=>{
    sessions=snap.docs.map(d=>({id:d.id,...d.data()}));
    updateStats();buildSidebar();
    if(currentView==='timetracker')renderTT();
    if(currentView==='dashboard')renderDashboard();
  },err=>console.error('sessions',err));

  onSnapshot(query(colProtocols,orderBy('createdAt','asc')),snap=>{
    protocols=snap.docs.map(d=>({id:d.id,...d.data()}));
    updateStats();
    if(currentView==='protocols')renderProtocols();
  },err=>console.error('protocols',err));
}
```

- [ ] **Step 4: Verify listener wires up**

Refresh the browser, sign in as a guest. Open DevTools Console — there should be no `protocols` error (the collection is empty so a `[]` result is expected). If you see a Firestore permission error, see the note below.

> **Firestore Security Rules note:** If you see `FirebaseError: Missing or insufficient permissions`, you need to add a `protocols` rule in the Firebase console → Firestore → Rules. The existing rule pattern for `entries` (allow read for any auth, allow write for member emails) should be replicated. Contact Sam to update the rules file if needed.

- [ ] **Step 5: Commit**

```bash
git add logbook.html
git commit -m "feat(protocols): add state, collection ref, and Firestore listener"
```

---

### Task 5: JS — `setView` and `updateStats` Integration

**Files:**
- Modify: `logbook.html` (two edits in the `setView` function and one in `updateStats`)

- [ ] **Step 1: Update the view-ID reset array in `setView`**

Find:
```js
  ['view-dashboard','view-entries','view-chronicles','view-requests','view-tt'].forEach(id=>document.getElementById(id).style.display='none');
```

Replace with:
```js
  ['view-dashboard','view-entries','view-chronicles','view-requests','view-tt','view-protocols'].forEach(id=>document.getElementById(id).style.display='none');
```

- [ ] **Step 2: Add `protocols` branch to the `setView` if/else chain**

Find:
```js
  } else {
    if(v==='all') document.getElementById('sb-all')?.classList.add('active');
    else document.getElementById('sb-sub-'+slug(v.slice(4)))?.classList.add('active');
    document.getElementById('view-entries').style.display='block';
```

Replace with:
```js
  } else if(v==='protocols'){
    document.getElementById('sb-protocols')?.classList.add('active');
    document.getElementById('view-protocols').style.display='block';
    renderProtocols();
  } else {
    if(v==='all') document.getElementById('sb-all')?.classList.add('active');
    else document.getElementById('sb-sub-'+slug(v.slice(4)))?.classList.add('active');
    document.getElementById('view-entries').style.display='block';
```

- [ ] **Step 3: Add protocol count to `updateStats`**

Find:
```js
  document.getElementById('hdr-ci').innerHTML=`<strong>${ci}</strong>Clocked In`;
}
```

Replace with:
```js
  document.getElementById('hdr-ci').innerHTML=`<strong>${ci}</strong>Clocked In`;
  const cntProto=document.getElementById('cnt-protocols');
  if(cntProto)cntProto.textContent=protocols.length||'—';
}
```

- [ ] **Step 4: Verify navigation works**

Refresh the browser, sign in as guest. Click "Protocols" in the sidebar — the view should appear with the `[ NO PROTOCOLS DEFINED ]` empty state, the sidebar button should highlight with the green left-bar indicator, and all other views should hide correctly.

- [ ] **Step 5: Commit**

```bash
git add logbook.html
git commit -m "feat(protocols): wire setView and updateStats for protocols"
```

---

### Task 6: JS — `renderProtocols`, `toggleProtocol`, `toggleStep`

**Files:**
- Modify: `logbook.html` (insert JS before closing `</script>` tag at line ~4061)

- [ ] **Step 1: Insert render and toggle functions before `</script>`**

Find:
```js
  toast('PDF report generated ✓');
};

</script>
```

Replace with:
```js
  toast('PDF report generated ✓');
};

/* ══════════════════════════════════════════
   PROTOCOLS
══════════════════════════════════════════ */
window.renderProtocols=()=>{
  const grid=document.getElementById('protocols-grid');if(!grid)return;
  const addBtn=document.getElementById('btn-add-proto');
  if(addBtn)addBtn.style.display=isMember?'':'none';
  const q=protocolSearch.trim().toLowerCase();
  const filtered=protocols.filter(p=>!q||p.title.toLowerCase().includes(q)||(p.description||'').toLowerCase().includes(q));
  if(!filtered.length){
    grid.innerHTML=`<div class="proto-empty">${q?'[ NO MATCHES ]':'[ NO PROTOCOLS DEFINED ]'}</div>`;
    return;
  }
  grid.innerHTML=filtered.map(p=>{
    const steps=p.steps||[];
    const memberActions=isMember?`<div class="entry-actions">
      <button class="entry-act-btn" title="Edit" onclick="editProtocol('${p.id}',event)">✎</button>
      <button class="entry-act-btn del" title="Delete" onclick="deleteProtocol('${p.id}',event)">✕</button>
    </div>`:'';
    const stepsList=steps.length
      ?`<div class="protocol-steps-label">Checklist</div>
        <ul class="protocol-step-list">
          ${steps.map((s,i)=>`<li class="protocol-step" id="ps-${p.id}-${i}" onclick="toggleStep('${p.id}',${i},event)">
            <input type="checkbox" id="psc-${p.id}-${i}" onclick="event.stopPropagation();toggleStep('${p.id}',${i},event)"/>
            <span class="protocol-step-text">${esc(s.text)}</span>
          </li>`).join('')}
        </ul>`
      :'<div class="proto-empty" style="padding:14px 0;font-size:.52rem;">No steps defined</div>';
    return `<div class="entry-card protocol-card stat-card-glitch" id="pc-${p.id}">
      <div class="entry-top">
        <div class="entry-stripe" style="background:#ffd700"></div>
        <div class="protocol-header" onclick="toggleProtocol('${p.id}')">
          <span class="protocol-title">${esc(p.title)}</span>
          <span class="protocol-step-count">${steps.length} step${steps.length!==1?'s':''}</span>
          <span class="protocol-chevron" id="pc-chev-${p.id}">▶</span>
        </div>
        ${memberActions}
      </div>
      <div class="protocol-body" id="pb-${p.id}">
        ${p.description?`<div class="protocol-desc">${esc(p.description)}</div>`:''}
        ${stepsList}
      </div>
    </div>`;
  }).join('');
};

window.toggleProtocol=id=>{
  const card=document.getElementById('pc-'+id);if(!card)return;
  const open=card.classList.toggle('expanded');
  const chev=document.getElementById('pc-chev-'+id);
  if(chev)chev.textContent=open?'▼':'▶';
};

window.toggleStep=(protocolId,stepIdx,ev)=>{
  if(ev)ev.stopPropagation();
  const item=document.getElementById('ps-'+protocolId+'-'+stepIdx);
  const cb=document.getElementById('psc-'+protocolId+'-'+stepIdx);
  if(!item)return;
  const done=item.classList.toggle('done');
  if(cb)cb.checked=done;
};

</script>
```

- [ ] **Step 2: Verify render works**

Refresh. Navigate to Protocols. The empty state `[ NO PROTOCOLS DEFINED ]` should show. Sign in as a member — the "+ Add Protocol" button should appear. Sign back in as guest — the button should hide.

- [ ] **Step 3: Commit**

```bash
git add logbook.html
git commit -m "feat(protocols): add renderProtocols, toggleProtocol, toggleStep"
```

---

### Task 7: JS — Modal Open/Edit + Step Management

**Files:**
- Modify: `logbook.html` (append to the protocols JS block)

- [ ] **Step 1: Insert modal open/edit functions before `</script>`**

Find:
```js
window.toggleStep=(protocolId,stepIdx,ev)=>{
  if(ev)ev.stopPropagation();
  const item=document.getElementById('ps-'+protocolId+'-'+stepIdx);
  const cb=document.getElementById('psc-'+protocolId+'-'+stepIdx);
  if(!item)return;
  const done=item.classList.toggle('done');
  if(cb)cb.checked=done;
};

</script>
```

Replace with:
```js
window.toggleStep=(protocolId,stepIdx,ev)=>{
  if(ev)ev.stopPropagation();
  const item=document.getElementById('ps-'+protocolId+'-'+stepIdx);
  const cb=document.getElementById('psc-'+protocolId+'-'+stepIdx);
  if(!item)return;
  const done=item.classList.toggle('done');
  if(cb)cb.checked=done;
};

function addProtocolStep(text=''){
  const list=document.getElementById('proto-steps-list');
  const idx=list.children.length;
  const row=document.createElement('div');
  row.className='protocol-step-row';
  row.innerHTML=`<input type="text" class="field-input" placeholder="Step ${idx+1}…" value="${esc(text)}"/>` +
    `<button class="proto-step-remove" onclick="removeProtocolStep(this)">✕</button>`;
  list.appendChild(row);
  _updateRemoveBtns();
  row.querySelector('input').focus();
}
window.addProtocolStep=addProtocolStep;

window.removeProtocolStep=btn=>{
  btn.closest('.protocol-step-row').remove();
  [...document.querySelectorAll('#proto-steps-list .protocol-step-row')].forEach((r,i)=>{
    r.querySelector('input').placeholder=`Step ${i+1}…`;
  });
  _updateRemoveBtns();
};

function _updateRemoveBtns(){
  const rows=document.querySelectorAll('#proto-steps-list .protocol-step-row');
  rows.forEach(r=>r.querySelector('.proto-step-remove').style.display=rows.length<=1?'none':'');
}

window.openProtocolModal=()=>{
  if(!requireMember('manage protocols'))return;
  editingProtocolId=null;
  document.getElementById('protocol-modal-title').textContent='NEW PROTOCOL';
  document.getElementById('proto-title').value='';
  document.getElementById('proto-desc').value='';
  document.getElementById('proto-steps-list').innerHTML='';
  document.getElementById('btn-del-proto').style.display='none';
  addProtocolStep();
  document.getElementById('protocol-modal-bg').classList.add('open');
  setTimeout(()=>document.getElementById('proto-title').focus(),150);
};

window.editProtocol=(id,ev)=>{
  if(!requireMember('edit protocols'))return;
  ev.stopPropagation();
  const p=protocols.find(x=>x.id===id);if(!p)return;
  editingProtocolId=id;
  document.getElementById('protocol-modal-title').textContent='EDIT PROTOCOL';
  document.getElementById('proto-title').value=p.title||'';
  document.getElementById('proto-desc').value=p.description||'';
  document.getElementById('proto-steps-list').innerHTML='';
  (p.steps||[]).forEach(s=>addProtocolStep(s.text));
  if(!(p.steps||[]).length)addProtocolStep();
  document.getElementById('btn-del-proto').style.display='';
  document.getElementById('protocol-modal-bg').classList.add('open');
  setTimeout(()=>document.getElementById('proto-title').focus(),150);
};

</script>
```

- [ ] **Step 2: Verify modal opens**

Refresh. Sign in as a member, navigate to Protocols, click "+ Add Protocol". The modal should open with a single empty step input field, the title "NEW PROTOCOL", and no delete button. Click the pencil icon on a protocol card (once one exists) — the modal should open with "EDIT PROTOCOL" title, the delete button visible, and all existing fields pre-populated.

- [ ] **Step 3: Verify step management**

Inside the modal: click "+ Add Step" multiple times — new rows appear. Click ✕ on any step row — it removes. When exactly one step remains, its ✕ button hides. Re-number placeholders update correctly.

- [ ] **Step 4: Commit**

```bash
git add logbook.html
git commit -m "feat(protocols): add protocol modal open/edit and step management"
```

---

### Task 8: JS — `saveProtocol` and `deleteProtocol`

**Files:**
- Modify: `logbook.html` (append to the protocols JS block)

- [ ] **Step 1: Insert save and delete functions before `</script>`**

Find:
```js
  document.getElementById('protocol-modal-bg').classList.add('open');
  setTimeout(()=>document.getElementById('proto-title').focus(),150);
};

</script>
```

Replace with:
```js
  document.getElementById('protocol-modal-bg').classList.add('open');
  setTimeout(()=>document.getElementById('proto-title').focus(),150);
};

window.saveProtocol=async()=>{
  if(!requireMember('save protocols'))return;
  const title=document.getElementById('proto-title').value.trim();
  if(!title){toast('Behavior name is required',true);return;}
  const description=document.getElementById('proto-desc').value.trim();
  const steps=[...document.querySelectorAll('#proto-steps-list .protocol-step-row input')]
    .map(i=>i.value.trim()).filter(Boolean).map(text=>({text,done:false}));
  const btn=document.getElementById('btn-save-proto');
  btn.disabled=true;
  try{
    if(editingProtocolId){
      await updateDoc(doc(db,'protocols',editingProtocolId),{title,description,steps,updatedAt:serverTimestamp()});
      toast('Protocol updated ✓');
    }else{
      await addDoc(colProtocols,{title,description,steps,createdAt:serverTimestamp(),updatedAt:serverTimestamp(),createdBy:currentUser});
      toast('Protocol saved ✓');
    }
    closeModal('protocol-modal-bg');
  }catch(err){toast('Save failed: '+err.message,true);}
  finally{btn.disabled=false;}
};

window.deleteProtocol=async(id,ev)=>{
  ev.stopPropagation();if(!requireMember('delete protocols'))return;
  if(!confirm('Delete this protocol? Cannot be undone.'))return;
  try{await deleteDoc(doc(db,'protocols',id));toast('Protocol deleted');}
  catch(err){toast('Delete failed: '+err.message,true);}
};

window.deleteProtocolFromModal=async()=>{
  if(!requireMember('delete protocols'))return;
  if(!editingProtocolId)return;
  if(!confirm('Delete this protocol? Cannot be undone.'))return;
  try{
    await deleteDoc(doc(db,'protocols',editingProtocolId));
    toast('Protocol deleted');
    closeModal('protocol-modal-bg');
  }catch(err){toast('Delete failed: '+err.message,true);}
};

</script>
```

- [ ] **Step 2: Verify create flow (as member)**

Sign in as a member. Navigate to Protocols. Click "+ Add Protocol". Enter:
- Behavior Name: `Drive Train Jitter`
- Description: `Robot veers or stutters during straight-line movement`
- Steps: `Check motor wiring`, `Inspect encoder cables`, `Test on flat surface`

Click "Save Protocol". Toast `Protocol saved ✓` appears. The card appears immediately in the grid (via `onSnapshot`).

- [ ] **Step 3: Verify read flow (as guest)**

Sign out → sign in as guest. Navigate to Protocols. The "Drive Train Jitter" card should be visible. The "+ Add Protocol" button and the edit/delete action buttons should be hidden. Click the card header — it expands showing the checklist. Check items — they toggle locally, the card count badge does not change, and refreshing the page resets all checkboxes.

- [ ] **Step 4: Verify search**

As any user, type `jitter` in the search box — only matching cards show. Clear search — all cards return.

- [ ] **Step 5: Verify edit flow (as member)**

Sign in as member. Click the pencil icon on "Drive Train Jitter". Modal opens with all fields pre-populated. Modify the title to `Drive Train Jitter (Updated)`. Click Save. Toast `Protocol updated ✓`. Card updates immediately via `onSnapshot`.

- [ ] **Step 6: Verify delete flow (as member)**

Click ✕ on a protocol card. Confirm dialog appears. Confirm → toast `Protocol deleted`, card disappears from grid. Alternatively, open edit modal → click "Delete Protocol" — same behavior.

- [ ] **Step 7: Verify member guard**

Sign in as guest. In DevTools console, call `openProtocolModal()`. Toast should fire: `Sign in as a team member to manage protocols.` — no modal opens.

- [ ] **Step 8: Commit**

```bash
git add logbook.html
git commit -m "feat(protocols): add saveProtocol and deleteProtocol — feature complete"
```

---

## Spec Coverage Check

| Spec Requirement | Task |
|---|---|
| Sidebar "Protocols" button under Resources section | Task 2 |
| `#view-protocols` view with glitch title | Task 3 |
| Search filtering by title + description | Task 6 |
| Protocol cards with expandable checklist | Task 6 |
| Local-only step toggle (not persisted) | Task 6 |
| Member-only "+ Add Protocol" button | Task 6 |
| Edit/delete icon buttons hidden for guests | Task 6 |
| Add Protocol modal with dynamic step list | Task 7 |
| Add/remove steps in modal | Task 7 |
| Save new protocol to Firestore | Task 8 |
| Update existing protocol in Firestore | Task 8 |
| Delete from card | Task 8 |
| Delete from edit modal | Task 8 |
| `requireMember()` guard on all write operations | Tasks 7–8 |
| `onSnapshot` live sync | Task 4 |
| `#cnt-protocols` badge in sidebar | Task 5 |
| `setView('protocols')` integration | Task 5 |
| Cyberpunk aesthetic (amber stripe, clip-path btns, mono font) | Task 1 |
