<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>IT Inventory Manager</title>
<style>
  :root {
    --bg: #0f1117; --panel: #181b24; --panel-2: #1f2330; --border: #2a2f3d;
    --text: #e6e9ef; --muted: #8b92a5; --accent: #5b8cff; --accent-2: #3d6ae0;
    --danger: #ff5b6e; --ok: #3ddc84;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    background: var(--bg); color: var(--text); line-height: 1.5; padding: 24px; }
  .wrap { max-width: 1000px; margin: 0 auto; }
  header { margin-bottom: 24px; }
  h1 { font-size: 24px; font-weight: 650; letter-spacing: -0.02em; }
  .sub { color: var(--muted); font-size: 14px; margin-top: 4px; }
  .toolbar { display: flex; gap: 12px; flex-wrap: wrap; align-items: center; margin-bottom: 16px; }
  .search { flex: 1; min-width: 220px; position: relative; }
  .search input { width: 100%; background: var(--panel); border: 1px solid var(--border);
    color: var(--text); padding: 10px 14px 10px 38px; border-radius: 10px; font-size: 14px; outline: none; }
  .search input:focus { border-color: var(--accent); }
  .search svg { position: absolute; left: 12px; top: 11px; width: 16px; height: 16px; fill: var(--muted); }
  button { font-family: inherit; font-size: 14px; cursor: pointer; border: none; border-radius: 10px;
    padding: 10px 16px; transition: background .15s, opacity .15s; }
  .btn-primary { background: var(--accent); color: #fff; font-weight: 550; }
  .btn-primary:hover { background: var(--accent-2); }
  .btn-ghost { background: var(--panel); color: var(--text); border: 1px solid var(--border); }
  .btn-ghost:hover { background: var(--panel-2); }
  .stats { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 12px; margin-bottom: 20px; }
  .stat { background: var(--panel); border: 1px solid var(--border); border-radius: 12px; padding: 14px 16px; }
  .stat .n { font-size: 24px; font-weight: 650; }
  .stat .l { color: var(--muted); font-size: 12px; text-transform: uppercase; letter-spacing: .04em; margin-top: 2px; }
  table { width: 100%; border-collapse: collapse; background: var(--panel); border-radius: 12px; overflow: hidden; }
  th, td { text-align: left; padding: 12px 16px; font-size: 14px; }
  thead th { background: var(--panel-2); color: var(--muted); font-weight: 550; font-size: 12px;
    text-transform: uppercase; letter-spacing: .04em; cursor: pointer; user-select: none; white-space: nowrap; }
  thead th:hover { color: var(--text); }
  tbody tr { border-top: 1px solid var(--border); }
  tbody tr:hover { background: var(--panel-2); }
  td.qty { font-variant-numeric: tabular-nums; font-weight: 600; }
  td.actions { text-align: right; white-space: nowrap; }
  .pill { font-size: 12px; padding: 2px 9px; border-radius: 99px; background: rgba(91,140,255,.15); color: var(--accent); }
  .pill.zero { background: rgba(255,91,110,.15); color: var(--danger); }
  .pill.muted { background: var(--panel-2); color: var(--muted); }
  .pill.assign { background: rgba(61,220,132,.15); color: var(--ok); cursor: default; }
  .icon-btn { background: transparent; padding: 6px 10px; color: var(--muted); border-radius: 8px; font-size: 13px; }
  .icon-btn:hover { background: var(--bg); color: var(--text); }
  .icon-btn.del:hover { color: var(--danger); }
  .icon-btn.manage { color: var(--accent); }
  .empty { text-align: center; color: var(--muted); padding: 40px; }
  .overlay { position: fixed; inset: 0; background: rgba(0,0,0,.6); display: none;
    align-items: center; justify-content: center; padding: 20px; z-index: 10; }
  .overlay.show { display: flex; }
  .modal { background: var(--panel); border: 1px solid var(--border); border-radius: 16px;
    width: 100%; max-width: 460px; padding: 24px; max-height: 85vh; overflow-y: auto; }
  .modal h2 { font-size: 18px; margin-bottom: 4px; }
  .modal .msub { color: var(--muted); font-size: 13px; margin-bottom: 18px; }
  .field { margin-bottom: 14px; }
  .field label { display: block; font-size: 13px; color: var(--muted); margin-bottom: 6px; }
  .field input { width: 100%; background: var(--bg); border: 1px solid var(--border); color: var(--text);
    padding: 10px 12px; border-radius: 9px; font-size: 14px; outline: none; }
  .field input:focus { border-color: var(--accent); }
  .modal-actions { display: flex; gap: 10px; justify-content: flex-end; margin-top: 20px; }
  .footer-note { color: var(--muted); font-size: 12px; margin-top: 18px; text-align: center; }
  /* assignment panel */
  .assign-add { display: flex; gap: 8px; margin-bottom: 16px; }
  .assign-add input { flex: 1; background: var(--bg); border: 1px solid var(--border); color: var(--text);
    padding: 10px 12px; border-radius: 9px; font-size: 14px; outline: none; }
  .assign-add input:focus { border-color: var(--accent); }
  .assign-list { list-style: none; }
  .assign-list li { display: flex; align-items: center; justify-content: space-between;
    padding: 10px 12px; background: var(--bg); border: 1px solid var(--border); border-radius: 9px; margin-bottom: 8px; }
  .assign-list .who { font-size: 14px; }
  .assign-list .when { font-size: 11px; color: var(--muted); margin-left: 8px; }
  .assign-empty { color: var(--muted); font-size: 13px; padding: 12px 0; text-align: center; }
  .summary { font-size: 13px; color: var(--muted); margin-bottom: 14px; }
  .summary b { color: var(--text); }
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>IT Inventory Manager</h1>
    <div class="sub">Track equipment and assign laptops &amp; monitors to people. Export to CSV to sync back to your sheet.</div>
  </header>

  <div class="stats" id="stats"></div>

  <div class="toolbar">
    <div class="search">
      <svg viewBox="0 0 24 24"><path d="M15.5 14h-.79l-.28-.27a6.5 6.5 0 1 0-.7.7l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0A4.5 4.5 0 1 1 14 9.5 4.5 4.5 0 0 1 9.5 14z"/></svg>
      <input id="search" placeholder="Search items or assignees..." />
    </div>
    <button class="btn-ghost" id="exportBtn">Export CSV</button>
    <button class="btn-primary" id="addBtn">+ Add Item</button>
  </div>

  <table>
    <thead>
      <tr>
        <th data-sort="name">Item</th>
        <th data-sort="qty">Total Qty</th>
        <th data-sort="inUse">In Use</th>
        <th data-sort="available">Available</th>
        <th></th>
      </tr>
    </thead>
    <tbody id="rows"></tbody>
  </table>
  <div class="empty" id="empty" style="display:none">No matching items.</div>

  <div class="footer-note">Data lives in this page for your session. Use <b>Export CSV</b> to save (assignments included) and import back into Google Sheets.</div>
</div>

<!-- Add/Edit Modal -->
<div class="overlay" id="overlay">
  <div class="modal">
    <h2 id="modalTitle">Add Item</h2>
    <div class="msub" id="modalSub" style="display:none">Assignable items track "in use" automatically.</div>
    <div class="field"><label>Item name</label><input id="f-name" placeholder="e.g. Training Laptops" /></div>
    <div class="field"><label>Total quantity</label><input id="f-qty" type="number" min="0" placeholder="0" /></div>
    <div class="field" id="inuse-field"><label>Number in use</label><input id="f-inuse" type="number" min="0" placeholder="0" /></div>
    <div class="modal-actions">
      <button class="btn-ghost" id="cancelBtn">Cancel</button>
      <button class="btn-primary" id="saveBtn">Save</button>
    </div>
  </div>
</div>

<!-- Assignment Modal -->
<div class="overlay" id="assignOverlay">
  <div class="modal">
    <h2 id="assignTitle">Assign</h2>
    <div class="summary" id="assignSummary"></div>
    <div class="assign-add">
      <input id="assignName" placeholder="Person's name..." />
      <button class="btn-primary" id="assignAddBtn">Assign</button>
    </div>
    <ul class="assign-list" id="assignList"></ul>
    <div class="assign-empty" id="assignEmpty" style="display:none">No units assigned yet.</div>
    <div class="modal-actions">
      <button class="btn-ghost" id="assignCloseBtn">Done</button>
    </div>
  </div>
</div>

<script>
  // Items whose name contains one of these (case-insensitive) are assignable
  const ASSIGNABLE = ["laptop", "monitor"];
  const isAssignable = name => ASSIGNABLE.some(k => name.toLowerCase().includes(k));

  // Seeded from the Google Sheet "IT Inventory". Assignable items use an `assignments` array.
  let items = [
    { name: "Training Laptops",      qty: 10, assignments: [] },
    { name: "Desktops",              qty: 8,  inUse: 1 },
    { name: "Keyboards",             qty: 42, inUse: 3 },
    { name: "Mouses",                qty: 32, inUse: 3 },
    { name: "Hdmi Cords",            qty: 0,  inUse: 0 },
    { name: "Display Cords",         qty: 0,  inUse: 0 },
    { name: "Dual Monitor Adapters", qty: 4,  inUse: 3 },
    { name: "Power Cords",           qty: 10, inUse: 0 },
    { name: "Wifi Adapters",         qty: 1,  inUse: 0 },
    { name: "Wired Headsets",        qty: 5,  inUse: 0 },
    { name: "Wireless Headsets",     qty: 5,  inUse: 0 },
    { name: "Laptops For Any Use",   qty: 6,  assignments: [] },
    { name: "Monitors",              qty: 4,  assignments: [] },
  ];

  let sortKey = "name", sortDir = 1, editIndex = null, assignIndex = null;

  const $ = id => document.getElementById(id);
  const rows = $("rows"), empty = $("empty"), searchEl = $("search");
  const overlay = $("overlay"), assignOverlay = $("assignOverlay");

  // For assignable items, in-use = number of assignments
  function inUseOf(i) { return isAssignable(i.name) ? (i.assignments ? i.assignments.length : 0) : (i.inUse || 0); }
  function available(i) { return Math.max(0, i.qty - inUseOf(i)); }

  function render() {
    const q = searchEl.value.trim().toLowerCase();
    let list = items.map((it, idx) => ({ ...it, _idx: idx })).filter(it => {
      if (it.name.toLowerCase().includes(q)) return true;
      // also match by assignee name
      return isAssignable(it.name) && (it.assignments || []).some(a => a.name.toLowerCase().includes(q));
    });

    list.sort((a, b) => {
      let av, bv;
      if (sortKey === "name") { av = a.name.toLowerCase(); bv = b.name.toLowerCase(); }
      else if (sortKey === "available") { av = available(a); bv = available(b); }
      else if (sortKey === "inUse") { av = inUseOf(a); bv = inUseOf(b); }
      else { av = a[sortKey]; bv = b[sortKey]; }
      if (av < bv) return -1 * sortDir;
      if (av > bv) return 1 * sortDir;
      return 0;
    });

    rows.innerHTML = "";
    empty.style.display = list.length ? "none" : "block";

    for (const it of list) {
      const avail = available(it), used = inUseOf(it), assignable = isAssignable(it.name);
      const tr = document.createElement("tr");
      tr.innerHTML = `
        <td>${escapeHtml(it.name)}${assignable ? ` <span class="pill assign">assignable</span>` : ``}</td>
        <td class="qty">${it.qty}</td>
        <td>${used > 0 ? `<span class="pill">${used} in use</span>` : `<span class="pill muted">0</span>`}</td>
        <td><span class="pill ${avail === 0 ? "zero" : ""}">${avail}</span></td>
        <td class="actions">
          ${assignable ? `<button class="icon-btn manage" data-manage="${it._idx}">Manage</button>` : ``}
          <button class="icon-btn" data-edit="${it._idx}">Edit</button>
          <button class="icon-btn del" data-del="${it._idx}">Delete</button>
        </td>`;
      rows.appendChild(tr);
    }
    renderStats();
  }

  function renderStats() {
    const types = items.length;
    const totalUnits = items.reduce((s, i) => s + i.qty, 0);
    const inUse = items.reduce((s, i) => s + inUseOf(i), 0);
    const out = items.filter(i => available(i) === 0).length;
    $("stats").innerHTML = `
      <div class="stat"><div class="n">${types}</div><div class="l">Item types</div></div>
      <div class="stat"><div class="n">${totalUnits}</div><div class="l">Total units</div></div>
      <div class="stat"><div class="n">${inUse}</div><div class="l">In use</div></div>
      <div class="stat"><div class="n">${out}</div><div class="l">Out of stock</div></div>`;
  }

  function escapeHtml(s) {
    return String(s).replace(/[&<>"']/g, c => ({ "&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;" }[c]));
  }

  document.querySelectorAll("th[data-sort]").forEach(th => {
    th.addEventListener("click", () => {
      const k = th.dataset.sort;
      if (sortKey === k) sortDir *= -1; else { sortKey = k; sortDir = 1; }
      render();
    });
  });
  searchEl.addEventListener("input", render);

  // ---- Add / Edit modal ----
  function openModal(idx) {
    editIndex = idx;
    const it = idx === null ? { name:"", qty:"" } : items[idx];
    const assignable = idx !== null && isAssignable(it.name);
    $("modalTitle").textContent = idx === null ? "Add Item" : "Edit Item";
    $("modalSub").style.display = assignable ? "block" : "none";
    $("inuse-field").style.display = assignable ? "none" : "block"; // auto for assignable
    $("f-name").value = it.name;
    $("f-qty").value = it.qty;
    $("f-inuse").value = it.inUse || 0;
    overlay.classList.add("show");
    $("f-name").focus();
  }
  function closeModal() { overlay.classList.remove("show"); editIndex = null; }

  $("addBtn").addEventListener("click", () => openModal(null));
  $("cancelBtn").addEventListener("click", closeModal);
  overlay.addEventListener("click", e => { if (e.target === overlay) closeModal(); });

  $("saveBtn").addEventListener("click", () => {
    const name = $("f-name").value.trim();
    if (!name) { $("f-name").focus(); return; }
    const qty = Math.max(0, parseInt($("f-qty").value) || 0);
    const assignable = isAssignable(name);

    if (editIndex === null) {
      items.push(assignable ? { name, qty, assignments: [] } : { name, qty, inUse: 0 });
    } else {
      const existing = items[editIndex];
      if (assignable) {
        // keep assignments; if it wasn't assignable before, start fresh
        const assignments = existing.assignments || [];
        items[editIndex] = { name, qty, assignments };
      } else {
        items[editIndex] = { name, qty, inUse: Math.max(0, parseInt($("f-inuse").value) || 0) };
      }
    }
    closeModal();
    render();
  });

  rows.addEventListener("click", e => {
    const editId = e.target.dataset.edit;
    const delId = e.target.dataset.del;
    const manageId = e.target.dataset.manage;
    if (editId !== undefined) openModal(Number(editId));
    if (manageId !== undefined) openAssign(Number(manageId));
    if (delId !== undefined) {
      const i = Number(delId);
      if (confirm(`Delete "${items[i].name}"?`)) { items.splice(i, 1); render(); }
    }
  });

  // ---- Assignment modal ----
  function openAssign(idx) {
    assignIndex = idx;
    assignOverlay.classList.add("show");
    $("assignName").value = "";
    renderAssign();
    $("assignName").focus();
  }
  function closeAssign() { assignOverlay.classList.remove("show"); assignIndex = null; render(); }

  function renderAssign() {
    const it = items[assignIndex];
    if (!it.assignments) it.assignments = [];
    const avail = available(it);
    $("assignTitle").textContent = "Assign — " + it.name;
    $("assignSummary").innerHTML =
      `<b>${it.assignments.length}</b> of <b>${it.qty}</b> assigned · <b>${avail}</b> available`;

    const list = $("assignList");
    list.innerHTML = "";
    $("assignEmpty").style.display = it.assignments.length ? "none" : "block";
    it.assignments.forEach((a, i) => {
      const li = document.createElement("li");
      li.innerHTML = `
        <span><span class="who">${escapeHtml(a.name)}</span><span class="when">${a.date}</span></span>
        <button class="icon-btn del" data-unassign="${i}">Return</button>`;
      list.appendChild(li);
    });

    // disable adding if none available
    $("assignAddBtn").disabled = avail <= 0;
    $("assignAddBtn").style.opacity = avail <= 0 ? .5 : 1;
    $("assignName").placeholder = avail <= 0 ? "No units available" : "Person's name...";
  }

  function addAssignment() {
    const it = items[assignIndex];
    const name = $("assignName").value.trim();
    if (!name) { $("assignName").focus(); return; }
    if (available(it) <= 0) return;
    const date = new Date().toLocaleDateString(undefined, { year:"numeric", month:"short", day:"numeric" });
    it.assignments.push({ name, date });
    $("assignName").value = "";
    renderAssign();
    $("assignName").focus();
  }

  $("assignAddBtn").addEventListener("click", addAssignment);
  $("assignName").addEventListener("keydown", e => { if (e.key === "Enter") addAssignment(); });
  $("assignCloseBtn").addEventListener("click", closeAssign);
  assignOverlay.addEventListener("click", e => { if (e.target === assignOverlay) closeAssign(); });
  $("assignList").addEventListener("click", e => {
    const u = e.target.dataset.unassign;
    if (u !== undefined) { items[assignIndex].assignments.splice(Number(u), 1); renderAssign(); }
  });

  // ---- CSV export (includes assignees) ----
  $("exportBtn").addEventListener("click", () => {
    const header = "Item,Total Qty,In Use,Available,Assigned To\n";
    const body = items.map(i => {
      const assignedTo = isAssignable(i.name) && i.assignments && i.assignments.length
        ? i.assignments.map(a => `${a.name} (${a.date})`).join("; ") : "";
      return `"${i.name.replace(/"/g,'""')}",${i.qty},${inUseOf(i)},${available(i)},"${assignedTo.replace(/"/g,'""')}"`;
    }).join("\n");
    const blob = new Blob([header + body], { type: "text/csv" });
    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "it-inventory.csv";
    a.click();
    URL.revokeObjectURL(a.href);
  });

  document.addEventListener("keydown", e => {
    if (e.key === "Escape") { closeModal(); closeAssign(); }
  });

  render();
</script>
</body>
</html>
