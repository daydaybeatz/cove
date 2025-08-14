<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1" />
<style>
  :root{
    --bg:#0f1220;--card:#171a2a;--text:#e7e9f4;--muted:#9aa0b5;--ok:#34d399;--err:#f87171;
    --accent:#6ee7ff;--accent2:#a78bfa;--border:#262b45;--hover:#2f365b;
    /* darker preview defaults (will be overridden by JS to match theme) */
    --preview-bg:#0c1120;
    --preview-text:#9fb0d4;
  }
  *{box-sizing:border-box}
  html,body{height:100%}
  body{
    margin:0;
    padding:env(safe-area-inset-top) 12px 12px 12px;
    background: radial-gradient(1200px 800px at 10% 10%, #11162b, var(--bg));
    color:var(--text);
    font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, "Helvetica Neue";
    display:grid;
    grid-template-rows: auto auto auto auto auto 1fr auto; /* title, toolbar, linkbar, settings, panels, grid, pager */
    row-gap:12px;
  }
  #bg{position:fixed; inset:0; z-index:-2; background-position:center; background-size:cover; background-repeat:no-repeat; opacity:.25}
  /* Duhg overlay */
  #duhgCanvas{
    position:fixed; inset:0; z-index:5; pointer-events:none; opacity:.42;
    mix-blend-mode:screen; will-change:transform;
  }

  .wrap{width:min(1100px,100%); margin-inline:auto}

  /* Title */
  .titlebar{display:flex; align-items:baseline; justify-content:space-between; gap:10px; padding-top:4px}
  h1{margin:0; font-size:20px; letter-spacing:.2px}
  .sub{color:var(--muted); font-size:12px}

  /* Sticky toolbar (top) */
  .sticky{position:sticky; top:0; z-index:20}
  .sticky .toolbar{
    background: linear-gradient(180deg, rgba(17,22,43,.92), rgba(17,22,43,.86));
    backdrop-filter: blur(6px);
    border:1px solid var(--border);
    border-radius:12px;
    padding:8px;
  }
  .toolbar{display:flex; flex-wrap:wrap; gap:8px; align-items:center}
  .tbtn{
    border:1px solid var(--border);
    background:linear-gradient(180deg,#151a2b,#11162b);
    color:var(--text);
    font-size:14px;
    padding:10px 12px; min-height:44px;
    border-radius:10px; cursor:pointer;
  }
  .tbtn:hover{border-color:var(--hover)}
  .tbtn.primary{
    background:
      radial-gradient(180px 70px at 12% 0%, rgba(255,255,255,.2), transparent 40%),
      linear-gradient(90deg,var(--accent),var(--accent2));
    color:#0a0d1c; font-weight:700; border-color:transparent;
  }

  /* Internet links toolbar */
  .linkbar{display:flex; flex-wrap:wrap; gap:8px; width:min(1100px,100%)}
  .linkbtn{
    border:1px solid var(--border);
    background:linear-gradient(180deg,#151a2b,#11162b);
    color:var(--text); font-size:14px;
    padding:10px 12px; min-height:44px;
    border-radius:10px; cursor:pointer;
    min-width:160px; white-space:nowrap;
  }
  .linkbtn:hover{border-color:var(--hover)}
  .linkbtn.editing{cursor:move}
  .linkdel, .linkmove{
    border:1px solid var(--border); background:#0f1430; color:var(--text);
    font-size:12px; padding:6px 8px; min-height:34px; border-radius:8px; cursor:pointer;
  }
  .linkdel:hover,.linkmove:hover{border-color:var(--hover)}
  .linkwrap{display:flex; align-items:center; gap:6px}

  /* Settings bar + panels */
  .settingsbar{display:none; gap:8px; flex-wrap:wrap; align-items:center}
  .settingsbar.show{display:flex}
  .sbtn{
    border:1px solid var(--border);
    background:#0f1430; color:var(--text);
    font-size:14px; padding:10px 12px; min-height:44px; border-radius:10px; cursor:pointer;
  }
  .sbtn:hover{border-color:var(--hover)}
  .panel{
    display:none; width:min(1100px,100%);
    border:1px solid var(--border); border-radius:12px; padding:12px; background:#12162a
  }
  .panel.show{display:grid; gap:10px}
  .panel label{font-size:12px; color:var(--muted)}
  .row{display:flex; gap:8px; flex-wrap:wrap}
  .panel input,.panel textarea,.panel select{
    width:100%; border-radius:8px; border:1px solid var(--border); background:#0f1430; color:var(--text);
    padding:10px; font-family: ui-monospace, Menlo, Consolas, monospace; font-size:13px;
  }
  .panel textarea{min-height:180px; resize:vertical}

  .bubbles{display:flex; gap:12px; align-items:center; user-select:none; flex-wrap:wrap}
  .bubble{
    width:36px; height:36px; border-radius:50%; border:2px solid rgba(255,255,255,.35);
    cursor:grab; position:relative; touch-action:manipulation;
  }
  .bubble small{
    position:absolute; bottom:-18px; left:50%; transform:translateX(-50%);
    font-size:10px; color:var(--muted); white-space:nowrap; pointer-events:none;
  }

  /* Grid */
  .grid{
    display:grid; gap:12px;
    grid-template-columns: repeat(auto-fit, minmax(520px, 1fr));
  }
  .card{
    background:linear-gradient(180deg,#1a1f35,var(--card));
    border:1px solid var(--border); border-radius:14px; padding:14px;
    box-shadow:0 6px 20px rgba(0,0,0,.25);
    display:flex; flex-direction:column; gap:10px;
  }
  .ctitle{display:flex; align-items:center; gap:10px}
  .ctitle .name{font-weight:700; font-size:16px; color:#ecf2ff; flex:1}
  .ordbox{display:flex; align-items:center; gap:6px; font-size:12px; color:var(--muted)}
  .ordbox input{
    width:64px; padding:6px 8px; border-radius:8px; border:1px solid var(--border);
    background:#0f1430; color:#e7e9f4; font-size:13px;
  }
  .hint{font-size:12px; color:var(--muted)}

  details.preview{
    background:var(--preview-bg);
    border:1px solid var(--border); border-radius:10px; padding:8px 10px
  }
  details.preview[open] summary{margin-bottom:6px}
  details.preview summary{cursor:pointer; list-style:none; font-size:12px; color:var(--muted)}
  details.preview summary::-webkit-details-marker{display:none}
  pre.snip{
    margin:0; font-size:13px; color:var(--preview-text);
    white-space:pre; overflow:auto; line-height:1.35;
    max-height:420px; min-height:300px;
  }
  .actions{display:flex; gap:8px; align-items:center; flex-wrap:wrap}
  .small{font-size:13px; padding:8px 10px; min-height:38px; border-radius:8px; cursor:pointer; border:1px solid var(--border); background:#0f1430; color:#e7e9f4}
  .copy{
    cursor:pointer; border:0; border-radius:10px; padding:10px 12px; min-height:44px;
    font-weight:600; font-size:14px; color:#0a0d1c;
    background:
      radial-gradient(220px 90px at 10% 0%, rgba(255,255,255,.25), transparent 40%),
      linear-gradient(90deg,var(--accent),var(--accent2));
  }
  .status{font-size:12px; color:var(--muted); min-height:1em}
  .status.ok{color:var(--ok)} .status.err{color:var(--err)}

  /* Pager */
  .pager{display:flex; gap:8px; flex-wrap:wrap; align-items:center; justify-content:center; padding:6px 0}
  .pgbtn{border:1px solid var(--border); background:#0f1430; color:#e7e9f4; padding:8px 10px; min-height:36px; border-radius:8px; cursor:pointer; font-size:13px}
  .pgbtn[disabled]{opacity:.45; cursor:not-allowed}
  .pgbtn.active{background:linear-gradient(90deg,var(--accent),var(--accent2)); color:#0a0d1c; border-color:transparent; font-weight:700}

  /* Toast & popup */
  .toast{
    position:fixed; left:50%; transform:translateX(-50%);
    bottom:calc(16px + env(safe-area-inset-bottom));
    background:#10152a; color:#e9f2ff; border:1px solid #273055; padding:10px 14px;
    border-radius:12px; box-shadow:0 8px 30px rgba(0,0,0,.45); display:none; z-index:9999; font-size:14px;
  }
  .toast.show{display:block}
  .pop{
    position:fixed; z-index:10000; display:none; gap:6px; align-items:center;
    background:#0e1430; border:1px solid var(--border); color:#e7e9f4;
    padding:8px 10px; border-radius:10px; font-size:12px;
  }
  .pop.show{display:flex}
  .yn{border:1px solid var(--border); background:#162046; color:#e7e9f4; padding:6px 10px; min-height:36px; border-radius:8px; cursor:pointer}
  .empty{color:var(--muted); text-align:center; padding:20px}
  .list{display:flex; flex-wrap:wrap; gap:8px}
  .pill{padding:8px 12px; min-height:36px; border-radius:999px; border:1px solid var(--border); background:#0f1430; cursor:pointer}
  .pill:hover{border-color:var(--hover)}

  /* Mobile */
  @media (max-width: 768px){
    .wrap{width:100%}
    .grid{grid-template-columns: 1fr}
    pre.snip{max-height:50vh; min-height:240px; font-size:12px}
    .ctitle .name{font-size:15px}
    .titlebar{padding-left:4px; padding-right:4px}
  }
</style>
</head>
<body>
  <div id="bg"></div>
  <canvas id="duhgCanvas"></canvas>

  <!-- Title -->
  <div class="wrap titlebar">
    <h1 id="projTitle">Cove - Project</h1>
    <div class="sub" id="subtitle"></div>
  </div>

  <!-- Sticky top toolbar -->
  <div class="wrap sticky">
    <div class="toolbar">
      <button class="tbtn" id="settingsBtn">Settings</button>
      <button class="tbtn" id="newProject">New Cove</button>
      <button class="tbtn" id="loadProject">Load Cove</button>
      <button class="tbtn" id="backup">Save Backup</button>
      <button class="tbtn" id="toggleAdder">Add Booty</button>
      <button class="tbtn primary" id="copyAll">Copy ALL</button>
      <button class="tbtn" id="deleteAll">Delete All Booty</button>
      <span class="sub">Showing <span id="countSpan">0</span> <span id="countLabel">Booty</span></span>
    </div>
  </div>

  <!-- Internet links -->
  <div class="wrap">
    <div id="linkbar" class="linkbar"></div>
    <div id="linkPanel" class="panel" aria-label="Add Web Link">
      <div class="row">
        <div style="flex:1"><label for="linkTitle">Link title</label><input id="linkTitle" placeholder="Example" /></div>
        <div style="flex:2"><label for="linkURL">URL</label><input id="linkURL" placeholder="https://example.com" /></div>
      </div>
      <div class="row">
        <button class="tbtn" id="saveLink">Save Link</button>
        <button class="tbtn" id="cancelLink">Close</button>
      </div>
    </div>
  </div>

  <!-- Settings toolbar -->
  <div class="wrap">
    <div id="settingsbar" class="settingsbar">
      <button class="sbtn" id="openTheme">Theme</button>
      <button class="sbtn" id="duhgToggle">Duhg Mode: Off</button>
      <button class="sbtn" id="setBg">Set Background Image</button>
      <button class="sbtn" id="removeBg">Remove Background</button>
      <button class="sbtn" id="openLabels">Rename Terms</button>
      <button class="sbtn" id="closeSettings">Close Settings</button>
    </div>
  </div>

  <!-- Theme panel -->
  <div class="wrap">
    <div id="themePanel" class="panel" aria-label="Theme">
      <div class="sub">Tap a bubble to pick a color. Drag to swap (desktop) or tap one bubble and then another to swap (mobile).</div>
      <div class="bubbles" id="bubbleRow"></div>
      <div class="row">
        <button class="tbtn" id="saveThemeBtn">Save Theme</button>
      </div>
    </div>
  </div>

  <!-- Rename Terms -->
  <div class="wrap">
    <div id="labelsPanel" class="panel" aria-label="Rename Terms">
      <div class="row">
        <div style="flex:1">
          <label for="labelCove">Name for Cove</label>
          <input id="labelCove" />
        </div>
        <div style="flex:1">
          <label for="labelBooty">Name for Booty</label>
          <input id="labelBooty" />
        </div>
      </div>
      <div class="row">
        <button class="tbtn" id="saveLabels">Save Names</button>
      </div>
    </div>
  </div>

  <!-- Add Booty -->
  <div class="wrap">
    <div class="panel" id="adder" aria-label="Add Booty">
      <div><label for="sampleTitle">Title</label><input id="sampleTitle" placeholder="Title" /></div>
      <div><label for="sampleCode">Code</label><textarea id="sampleCode" placeholder="Paste code here"></textarea></div>
      <div class="row">
        <button class="tbtn" id="saveSample">Save</button>
        <button class="tbtn" id="cancelSample">Close</button>
      </div>
    </div>
  </div>

  <!-- Load Cove -->
  <div class="wrap">
    <div class="panel" id="loader" aria-label="Load Cove">
      <div><label id="availableLabel">Available Coves</label></div>
      <div class="list" id="projList"></div>
      <div class="row"><button class="tbtn" id="closeLoader">Close</button></div>
    </div>
  </div>

  <!-- Pager (top) -->
  <div class="wrap"><div id="pagerTop" class="pager"></div></div>

  <!-- Grid -->
  <div class="wrap">
    <div id="grid" class="grid" role="list"></div>
    <div id="empty" class="empty" style="display:none">No booty yet. Use Add Booty.</div>
  </div>

  <!-- Pager (bottom) -->
  <div class="wrap"><div id="pagerBottom" class="pager"></div></div>

  <!-- Toast & pop -->
  <div id="toast" class="toast" role="status" aria-live="polite">Saved</div>
  <div id="pop" class="pop">
    <span id="popText">Confirm</span>
    <button class="yn" id="popYes">Yes</button>
    <button class="yn" id="popNo">No</button>
  </div>

<script>
(() => {
  const PAGE_SIZE = 6;
  const INDEX_KEY = "coves::index";
  const LABELS_KEY = "coves::labels";
  const THEME_KEY = "coves::theme";
  const BGIMG_KEY = "coves::bgImage";
  const DUHG_KEY  = "coves::duhgOn";
  const isTouch = matchMedia("(pointer: coarse)").matches;

  const $ = s => document.querySelector(s);
  function toast(msg){ const t=$("#toast"); t.textContent=msg; t.classList.add("show"); clearTimeout(toast._t); toast._t=setTimeout(()=>t.classList.remove("show"), 1400); }
  async function copyText(text){
    try{ await navigator.clipboard.writeText(text); return true; }
    catch{ try{ const ta=document.createElement("textarea"); ta.style.position="fixed"; ta.style.top="-2000px"; ta.style.left="-2000px"; ta.value=text; document.body.appendChild(ta); ta.focus(); ta.select(); const ok=document.execCommand("copy"); ta.remove(); return ok; }catch{return false;} }
  }
  function showPop(x,y,msg,onYes){
    const pop=$("#pop"); $("#popText").textContent=msg;
    pop.style.left=Math.max(8,Math.min(window.innerWidth-260,x))+"px";
    pop.style.top =Math.max(8,Math.min(window.innerHeight-80,y))+"px";
    pop.classList.add("show");
    const cleanup=()=>pop.classList.remove("show");
    $("#popYes").onclick=()=>{cleanup(); onYes&&onYes();};
    $("#popNo").onclick=cleanup;
  }

  /* Labels */
  function loadLabels(){ try{ const j=JSON.parse(localStorage.getItem(LABELS_KEY)||"{}"); return {cove:j.cove||"Cove", booty:j.booty||"Booty"}; }catch{ return {cove:"Cove", booty:"Booty"}; } }
  function saveLabels(obj){ try{ localStorage.setItem(LABELS_KEY, JSON.stringify(obj)); }catch{} }
  let labels = loadLabels();
  function updateTitle(){ $("#projTitle").textContent = labels.cove + " - " + currentCove; $("#subtitle").textContent = labels.cove + ": " + currentCove; document.title = "coves - " + currentCove; }
  function applyLabels(){
    $("#newProject").textContent = "New " + labels.cove;
    $("#loadProject").textContent = "Load " + labels.cove;
    $("#toggleAdder").textContent = "Add " + labels.booty;
    $("#deleteAll").textContent = "Delete All " + labels.booty;
    $("#countLabel").textContent = labels.booty;
    $("#availableLabel").textContent = "Available " + labels.cove + "s";
    $("#empty").textContent = "No " + labels.booty.toLowerCase() + " yet. Use Add " + labels.booty + ".";
    $("#labelCove").value = labels.cove;
    $("#labelBooty").value = labels.booty;
    updateTitle();
  }

  /* Theme */
  const hexToRgb = (hex)=>{ let h=hex.trim(); if(h.startsWith("#"))h=h.slice(1); if(h.length===3)h=h.split("").map(c=>c+c).join(""); const n=parseInt(h,16); return {r:(n>>16)&255,g:(n>>8)&255,b:n&255}; };
  function mix(h1,h2,p){ // both hex strings
    const a=hexToRgb(h1), b=hexToRgb(h2);
    const r=Math.round(a.r+(b.r-a.r)*p), g=Math.round(a.g+(b.g-a.g)*p), bl=Math.round(a.b+(b.b-a.b)*p);
    return "rgb("+r+","+g+","+bl+")";
  }
  const rgbaFromHex = (h,a)=>{ const {r,g,b}=hexToRgb(h); return "rgba("+r+","+g+","+b+","+a+")"; };
  function toHex(c){ if(/^#/.test(c)) return c; const m=c.match(/rgba?\((\d+),\s*(\d+),\s*(\d+)/i); if(!m) return "#ffffff"; const h=n=>("0"+Number(n).toString(16)).slice(-2); return "#"+h(m[1])+h(m[2])+h(m[3]); }

  function loadTheme(){
    try{
      const j=JSON.parse(localStorage.getItem(THEME_KEY)||"{}");
      const cs=getComputedStyle(document.documentElement);
      return {
        accent: j.accent || cs.getPropertyValue("--accent").trim() || "#6ee7ff",
        accent2:j.accent2|| cs.getPropertyValue("--accent2").trim()|| "#a78bfa",
        text:   j.text   || cs.getPropertyValue("--text").trim()   || "#e7e9f4",
        bg:     j.bg     || cs.getPropertyValue("--bg").trim()     || "#0f1220",
        card:   j.card   || cs.getPropertyValue("--card").trim()   || "#171a2a",
        border: j.border || cs.getPropertyValue("--border").trim() || "#262b45"
      };
    }catch{ return {accent:"#6ee7ff", accent2:"#a78bfa", text:"#e7e9f4", bg:"#0f1220", card:"#171a2a", border:"#262b45"}; }
  }
  function saveTheme(t){ try{ localStorage.setItem(THEME_KEY, JSON.stringify(t)); }catch{} }
  function applyTheme(t){
    const r=document.documentElement.style;
    r.setProperty("--accent",t.accent);
    r.setProperty("--accent2",t.accent2);
    r.setProperty("--text",t.text);
    r.setProperty("--bg",t.bg);
    r.setProperty("--card",t.card);
    r.setProperty("--border",t.border);
    // derived hover + preview colors
    r.setProperty("--hover", mix(toHex(t.border), toHex(t.accent), 0.35));
    r.setProperty("--preview-bg", mix(toHex(t.card), "#000000", 0.35)); // darker than card
    r.setProperty("--preview-text", mix(toHex(t.text), toHex(t.bg), 0.6)); // dimmed toward bg
  }

  /* Background */
  function loadBg(){ try{ return localStorage.getItem(BGIMG_KEY) || ""; }catch{ return ""; } }
  function saveBg(data){ try{ data?localStorage.setItem(BGIMG_KEY,data):localStorage.removeItem(BGIMG_KEY); }catch{} }
  function applyBg(data){ $("#bg").style.backgroundImage = data?("url("+data+")"):"none"; }

  /* Storage per cove */
  const keyOf=(proj,suffix)=>"coves::"+proj+"::"+suffix;
  const BOOTY=p=>keyOf(p,"booty");
  const LINKS=p=>keyOf(p,"links");

  function loadIndex(){ try{ const raw=localStorage.getItem(INDEX_KEY); const arr=raw?JSON.parse(raw):[]; return Array.isArray(arr)?[...new Set(arr)].filter(Boolean):[]; }catch{ return []; } }
  function saveIndex(list){ try{ localStorage.setItem(INDEX_KEY, JSON.stringify(list)); }catch{} }
  function loadBooty(p){ try{ const a=JSON.parse(localStorage.getItem(BOOTY(p))||"[]"); return Array.isArray(a)?a:[]; }catch{ return []; } }
  function saveBooty(p,arr){ try{ localStorage.setItem(BOOTY(p), JSON.stringify(arr)); }catch{} $("#countSpan").textContent=arr.length; }
  function loadLinks(p){ try{ const a=JSON.parse(localStorage.getItem(LINKS(p))||"[]"); return Array.isArray(a)?a:[]; }catch{ return []; } }
  function saveLinks(p,arr){ try{ localStorage.setItem(LINKS(p), JSON.stringify(arr)); }catch{} }

  let projectIndex=loadIndex(); if(projectIndex.length===0){ projectIndex=["default"]; saveIndex(projectIndex); }
  let currentCove=projectIndex[0];
  let booty=loadBooty(currentCove).map((s,i)=>({...s, order: typeof s.order==="number"?s.order:(i+1)}));
  let links=loadLinks(currentCove);
  let currentPage=1;

  function setCove(name){
    currentCove=name;
    booty=loadBooty(name).map((s,i)=>({...s, order: typeof s.order==="number"?s.order:(i+1)}));
    normalizeOrders();
    links=loadLinks(name);
    currentPage=1;
    duhg.setSourceFromBooty();
    applyLabels(); renderAll();
  }
  function normalizeOrders(){ booty.sort((a,b)=>(a.order-b.order)||((a.id||0)-(b.id||0))); booty.forEach((s,i)=> s.order=i+1); }

  /* ===== Duhg mode ===== */
  const duhg={
    on:false, raf:null, fps:isTouch?30:60, last:0, fontSize:16, cols:[], idx:0, source:"01",
    canvas:$("#duhgCanvas"), ctx:null,
    charColor:"rgba(120,255,180,.85)", headColor:"rgba(220,255,240,.9)", fadeColor:"rgba(0,0,0,.06)",
    ensureCtx(){ if(!this.ctx) this.ctx=this.canvas.getContext("2d"); },
    setSourceFromBooty(){
      const pool = booty.map(b=>b.code||"").join("\n").replace(/\s+/g," ");
      this.source = pool && pool.length>50 ? pool : "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ{}[]()<>+-=*/\\|;:,._";
    },
    setColorsFromTheme(){
      this.charColor = mix(toHex(theme.accent), "#ffffff", 0.25);
      this.headColor = mix(toHex(theme.accent2), "#ffffff", 0.35);
      this.fadeColor = rgbaFromHex(theme.bg, 0.08);
    },
    sizeToViewport(){
      const dpr = Math.max(1, Math.min(3, window.devicePixelRatio || 1));
      const vw = window.innerWidth, vh = window.innerHeight;
      this.canvas.style.width = vw + "px";
      this.canvas.style.height = vh + "px";
      this.canvas.width  = Math.round(vw * dpr);
      this.canvas.height = Math.round(vh * dpr);
      this.ensureCtx();
      this.ctx.setTransform(dpr,0,0,dpr,0,0);
      const cols = Math.ceil(vw / this.fontSize);
      this.cols = new Array(cols).fill(0).map(()=> Math.random()*vh);
      this.ctx.font = this.fontSize + "px ui-monospace, Menlo, Consolas, monospace";
      this.ctx.textBaseline = "top";
    },
    start(){
      if(this.on) return;
      this.on = true;
      this.ensureCtx();
      this.setSourceFromBooty();
      this.setColorsFromTheme();
      this.sizeToViewport();
      this.last = 0;
      const loop = (t=0)=>{
        if(!this.on) return;
        const minDelta = 1000/this.fps;
        if(t - this.last < minDelta){ this.raf = requestAnimationFrame(loop); return; }
        this.last = t;
        const ctx=this.ctx, w=window.innerWidth, h=window.innerHeight, fs=this.fontSize;
        ctx.fillStyle=this.fadeColor; ctx.fillRect(0,0,w,h);
        const src=this.source; const slen=src.length||1;
        for(let i=0;i<this.cols.length;i++){
          const x=i*fs, y=this.cols[i];
          const ch=src.charAt((this.idx+i)%slen)||"0";
          ctx.fillStyle=this.charColor; ctx.fillText(ch,x,y);
          if((this.idx+i)%23===0){ ctx.fillStyle=this.headColor; ctx.fillText(ch,x,y); }
          this.cols[i] = (y>h+fs*2) ? (-Math.random()*200) : (y+fs);
        }
        this.idx = (this.idx+1) % slen;
        this.raf = requestAnimationFrame(loop);
      };
      this.raf = requestAnimationFrame(loop);
    },
    stop(){
      this.on=false;
      if(this.raf){ cancelAnimationFrame(this.raf); this.raf=null; }
      if(this.ctx){ this.ctx.clearRect(0,0,this.canvas.width,this.canvas.height); }
    }
  };
  const resizeDuhg = ()=>{ if(duhg.on){ duhg.sizeToViewport(); } };
  window.addEventListener("resize", resizeDuhg, {passive:true});
  window.addEventListener("orientationchange", ()=> setTimeout(resizeDuhg, 250), {passive:true});
  document.addEventListener("visibilitychange", ()=>{ if(document.hidden){ duhg.stop(); } else { if(localStorage.getItem(DUHG_KEY)==="1") duhg.start(); } });

  /* Theme init + bubbles */
  let theme = loadTheme();
  applyTheme(theme);

  // Mobile-friendly swapping: tap one bubble, then another to swap roles
  let pendingSwapRole = null;

  function renderBubbles(){
    const row=$("#bubbleRow"); row.innerHTML="";
    const roles = [
      {role:"accent", label:"accent 1"},
      {role:"accent2",label:"accent 2"},
      {role:"text",   label:"text"},
      {role:"bg",     label:"background"},
      {role:"card",   label:"card"},
      {role:"border", label:"border"}
    ];
    roles.forEach(({role,label})=>{
      const b=document.createElement("div"); b.className="bubble"; b.draggable=!isTouch; b.dataset.role=role; b.style.background=theme[role];
      const cap=document.createElement("small"); cap.textContent=label; b.appendChild(cap);

      // Color picker (works on desktop + mobile)
      const openPicker = () => {
        const inp=document.createElement("input");
        inp.type="color";
        inp.value=toHex(theme[role]);
        inp.style.position="fixed"; inp.style.left="-9999px";
        document.body.appendChild(inp);
        inp.oninput=()=>{ theme[role]=inp.value; b.style.background=inp.value; applyTheme(theme); duhg.setColorsFromTheme(); if(duhg.on) duhg.sizeToViewport(); };
        inp.onchange=()=>document.body.removeChild(inp);
        inp.click();
      };
      b.addEventListener("click",(e)=>{
        // if we already have a pending swap, do swap on click instead of picker
        if(isTouch && pendingSwapRole && pendingSwapRole!==role){
          const from=pendingSwapRole, to=role; pendingSwapRole=null;
          const tmp=theme[from]; theme[from]=theme[to]; theme[to]=tmp;
          applyTheme(theme); duhg.setColorsFromTheme(); renderBubbles(); if(duhg.on) duhg.sizeToViewport();
          toast("Swapped "+from+" with "+to);
        } else {
          openPicker();
        }
      });

      // Touch swap: first tap selects source, second tap on a different bubble swaps
      b.addEventListener("touchstart",(e)=>{
        if(!isTouch) return;
        if(pendingSwapRole===role){ pendingSwapRole=null; toast("Swap canceled"); return; }
        if(pendingSwapRole && pendingSwapRole!==role){
          const from=pendingSwapRole, to=role; pendingSwapRole=null;
          const tmp=theme[from]; theme[from]=theme[to]; theme[to]=tmp;
          applyTheme(theme); duhg.setColorsFromTheme(); renderBubbles(); if(duhg.on) duhg.sizeToViewport();
          toast("Swapped "+from+" with "+to);
        } else {
          pendingSwapRole=role;
          toast("Tap another bubble to swap with "+role);
        }
        e.preventDefault();
      }, {passive:false});

      // Desktop drag-and-drop swap
      b.addEventListener("dragstart",e=>{ if(isTouch) return; e.dataTransfer.setData("text/role",role); });
      b.addEventListener("dragover",e=>{ if(isTouch) return; e.preventDefault(); });
      b.addEventListener("drop",e=>{
        if(isTouch) return;
        e.preventDefault();
        const from=e.dataTransfer.getData("text/role"); const to=role;
        if(!from||from===to) return;
        const tmp=theme[from]; theme[from]=theme[to]; theme[to]=tmp;
        applyTheme(theme); duhg.setColorsFromTheme(); renderBubbles(); if(duhg.on) duhg.sizeToViewport();
      });

      row.appendChild(b);
    });
  }
  renderBubbles();

  /* Internet links (desktop drag; mobile up/down) */
  let editLinks=false;
  function arrayMove(arr, from, to){ if(from===to) return arr; const item=arr.splice(from,1)[0]; arr.splice(to,0,item); return arr; }
  function renderLinks(){
    const bar=$("#linkbar"); bar.innerHTML="";
    const add=document.createElement("button"); add.className="linkbtn"; add.textContent="Add Web Link";
    add.onclick=()=>{ $("#linkPanel").classList.toggle("show"); $("#linkTitle").focus(); };
    const edit=document.createElement("button"); edit.className="linkbtn"; edit.textContent = editLinks ? "Done Editing Links" : "Edit Links";
    edit.onclick=()=>{ editLinks=!editLinks; renderLinks(); };
    bar.appendChild(add); bar.appendChild(edit);

    links.forEach((lnk,idx)=>{
      const container=document.createElement("div"); container.className="linkwrap";

      if(!isTouch){
        container.draggable = editLinks;
        if(editLinks){
          container.addEventListener("dragstart",e=>{ e.dataTransfer.setData("text/index", String(idx)); });
          container.addEventListener("dragover",e=>e.preventDefault());
          container.addEventListener("drop",e=>{ e.preventDefault(); const from=Number(e.dataTransfer.getData("text/index")); const to=idx; arrayMove(links, from, to); saveLinks(currentCove,links); renderLinks(); toast("Links reordered"); });
        }
      }

      const b=document.createElement("button"); b.className="linkbtn"+(editLinks&&!isTouch?" editing":""); b.textContent=lnk.title||lnk.url||"link";
      if(!editLinks){ b.onclick=()=> window.open(lnk.url,"_blank"); }
      container.appendChild(b);

      if(editLinks){
        if(isTouch){
          const up=document.createElement("button"); up.className="linkmove"; up.textContent="Up";
          const dn=document.createElement("button"); dn.className="linkmove"; dn.textContent="Down";
          up.onclick=()=>{ if(idx>0){ arrayMove(links, idx, idx-1); saveLinks(currentCove,links); renderLinks(); } };
          dn.onclick=()=>{ if(idx<links.length-1){ arrayMove(links, idx, idx+1); saveLinks(currentCove,links); renderLinks(); } };
          container.appendChild(up); container.appendChild(dn);
        }
        const del=document.createElement("button"); del.className="linkdel"; del.textContent="Delete";
        del.onclick=()=>{ links.splice(idx,1); saveLinks(currentCove,links); renderLinks(); };
        container.appendChild(del);
      }

      bar.appendChild(container);
    });
  }

  /* Grid + pager */
  function makePager(total, node){
    node.innerHTML="";
    const pages=Math.max(1, Math.ceil(total / PAGE_SIZE));
    if(currentPage>pages) currentPage=pages;
    const mk=(label,on,disabled=false,active=false)=>{ const b=document.createElement("button"); b.className="pgbtn"+(active?" active":""); b.textContent=label; b.disabled=!!disabled; b.onclick=on; node.appendChild(b); };
    mk("First",()=>{currentPage=1; renderGrid();}, currentPage===1);
    mk("Prev", ()=>{currentPage=Math.max(1,currentPage-1); renderGrid();}, currentPage===1);
    const pagesToShow=5; let start=Math.max(1,currentPage-Math.floor(pagesToShow/2)); let end=Math.min(pages,start+pagesToShow-1); start=Math.max(1,Math.min(start,end-pagesToShow+1));
    for(let p=start;p<=end;p++){ mk(String(p), ()=>{currentPage=p; renderGrid();}, false, p===currentPage); }
    mk("Next",()=>{currentPage=Math.min(pages,currentPage+1); renderGrid();}, currentPage===pages);
    mk("Last", ()=>{currentPage=pages; renderGrid();}, currentPage===pages);
  }

  // simple history helper for contenteditable code blocks
  function makeHistory(el, initialText){
    const hist = { stack:[initialText], idx:0, timer:null, limit:100 };
    const set = (txt) => { el.innerText = txt; };
    const current = () => hist.stack[hist.idx];
    const push = (txt) => {
      if (txt === current()) return;
      hist.stack = hist.stack.slice(0, hist.idx+1);
      hist.stack.push(txt);
      if(hist.stack.length>hist.limit){ hist.stack.shift(); } else { hist.idx++; }
    };
    const debouncedPush = () => {
      clearTimeout(hist.timer);
      hist.timer = setTimeout(()=> push(el.innerText), 300);
    };
    const undo = () => { if(hist.idx>0){ hist.idx--; set(current()); } };
    const redo = () => { if(hist.idx<hist.stack.length-1){ hist.idx++; set(current()); } };
    const canUndo = () => hist.idx>0;
    const canRedo = () => hist.idx<hist.stack.length-1;
    return { push, debouncedPush, undo, redo, canUndo, canRedo };
  }

  function makeCard(item){
    const card=document.createElement("div"); card.className="card";
    const title=document.createElement("div"); title.className="ctitle";
    const name=document.createElement("div"); name.className="name"; name.textContent=item.title||"(untitled)";
    const ordbox=document.createElement("div"); ordbox.className="ordbox";
    const lab=document.createElement("span"); lab.textContent="Order:";
    const input=document.createElement("input"); input.type="number"; input.value=item.order; input.min=1; input.step=1;
    input.addEventListener("change",()=>{ const v=Math.max(1,Math.floor(Number(input.value)||item.order)); normalizeOrders(); const from=booty.findIndex(s=>s.id===item.id); const to=Math.min(booty.length,v)-1; const [sp]=booty.splice(from,1); booty.splice(to,0,sp); normalizeOrders(); saveBooty(currentCove,booty); renderAll(); duhg.setSourceFromBooty(); });
    ordbox.append(lab,input); title.append(name,ordbox);

    const hint=document.createElement("div"); hint.className="hint"; hint.textContent="Tap preview to edit. Use Undo/Redo, then Save.";
    const details=document.createElement("details"); details.className="preview"; details.open=true;
    const sum=document.createElement("summary"); sum.textContent="Preview";
    const pre=document.createElement("pre"); pre.className="snip"; pre.contentEditable="true"; pre.spellcheck=false; pre.textContent=item.code||"";
    details.append(sum,pre);

    const status=document.createElement("div"); status.className="status"; status.textContent=" ";
    const btnCopy=document.createElement("button"); btnCopy.className="copy"; btnCopy.textContent="Copy to clipboard";
    btnCopy.onclick=async()=>{ const ok=await copyText(pre.innerText); status.className="status "+(ok?"ok":"err"); status.textContent=ok?"Copied":"Copy failed"; toast(ok?"Copied":"Copy failed"); };

    const saveBtn=document.createElement("button"); saveBtn.className="small"; saveBtn.textContent="Save changes"; saveBtn.style.display="none";
    const undoBtn=document.createElement("button"); undoBtn.className="small"; undoBtn.textContent="Undo";
    const redoBtn=document.createElement("button"); redoBtn.className="small"; redoBtn.textContent="Redo";

    const hist = makeHistory(pre, pre.innerText);
    function refreshUndoRedo(){
      undoBtn.disabled = !hist.canUndo();
      redoBtn.disabled = !hist.canRedo();
    }
    refreshUndoRedo();

    function markDirty(){ saveBtn.style.display = pre.innerText !== (item.code||"") ? "inline-flex":"none"; }

    pre.addEventListener("input", ()=>{ hist.debouncedPush(); markDirty(); refreshUndoRedo(); });

    undoBtn.onclick=()=>{ hist.undo(); markDirty(); refreshUndoRedo(); };
    redoBtn.onclick=()=>{ hist.redo(); markDirty(); refreshUndoRedo(); };

    saveBtn.onclick=()=>{ item.code=pre.innerText; const idx=booty.findIndex(s=>s.id===item.id); if(idx>=0){ booty[idx]=item; saveBooty(currentCove,booty); toast("Saved"); duhg.setSourceFromBooty(); } markDirty(); };

    const delBtn=document.createElement("button"); delBtn.className="small"; delBtn.textContent="Delete";
    const confirmWrap=document.createElement("span"); confirmWrap.style.display="none"; confirmWrap.className="row";
    const ynTxt=document.createElement("span"); ynTxt.textContent="Are you sure";
    const yes=document.createElement("button"); yes.className="small"; yes.textContent="Yes";
    const no=document.createElement("button"); no.className="small"; no.textContent="No";
    confirmWrap.append(ynTxt,yes,no);
    delBtn.onclick=()=>{ confirmWrap.style.display="flex"; };
    yes.onclick=()=>{ const idx=booty.findIndex(s=>s.id===item.id); if(idx>=0){ booty.splice(idx,1); normalizeOrders(); saveBooty(currentCove,booty); renderAll(); toast("Deleted"); duhg.setSourceFromBooty(); }};
    no.onclick=()=>{ confirmWrap.style.display="none"; };

    const actions=document.createElement("div"); actions.className="actions";
    actions.append(btnCopy, undoBtn, redoBtn, saveBtn, delBtn, confirmWrap);
    card.append(title,hint,details,actions,status);
    return card;
  }

  function renderGrid(){
    $("#grid").innerHTML="";
    booty.sort((a,b)=>a.order-b.order);
    const start=(currentPage-1)*PAGE_SIZE;
    const slice=booty.slice(start,start+PAGE_SIZE);
    if(!slice.length){ $("#empty").style.display="block"; } else { $("#empty").style.display="none"; slice.forEach(s=>$("#grid").appendChild(makeCard(s))); }
  }
  function renderPagers(){ makePager(booty.length, $("#pagerTop")); makePager(booty.length, $("#pagerBottom")); }
  function renderAll(){ renderGrid(); renderPagers(); renderLinks(); $("#countSpan").textContent=booty.length; }

  /* Settings actions */
  $("#settingsBtn").onclick=()=>{ $("#settingsbar").classList.toggle("show"); };
  $("#closeSettings").onclick=()=>{ $("#settingsbar").classList.remove("show"); $("#themePanel").classList.remove("show"); $("#labelsPanel").classList.remove("show"); };

  $("#openTheme").onclick=()=>{ $("#themePanel").classList.toggle("show"); $("#labelsPanel").classList.remove("show"); };
  $("#saveThemeBtn").onclick=()=>{ saveTheme(theme); duhg.setColorsFromTheme(); if(duhg.on) duhg.sizeToViewport(); toast("Theme saved"); };

  $("#openLabels").onclick=()=>{ $("#labelsPanel").classList.toggle("show"); $("#themePanel").classList.remove("show"); };
  $("#saveLabels").onclick=()=>{ labels={ cove: ($("#labelCove").value||"Cove").trim() || "Cove", booty: ($("#labelBooty").value||"Booty").trim() || "Booty" }; saveLabels(labels); applyLabels(); renderAll(); toast("Names saved"); };

  $("#setBg").onclick=()=>{ const inp=document.createElement("input"); inp.type="file"; inp.accept="image/*"; inp.onchange=()=>{ const f=inp.files&&inp.files[0]; if(!f) return; const r=new FileReader(); r.onload=()=>{ const data=r.result; applyBg(data); saveBg(data); toast("Background set"); }; r.readAsDataURL(f); }; inp.click(); };
  $("#removeBg").onclick=(ev)=>{ showPop(ev.pageX||innerWidth/2, ev.pageY||100, "Remove background image", ()=>{ applyBg(""); saveBg(""); toast("Background removed"); }); };

  /* Link add/save panel */
  $("#cancelLink").onclick=()=>{ $("#linkPanel").classList.remove("show"); $("#linkTitle").value=""; $("#linkURL").value=""; };
  $("#saveLink").onclick=()=>{ const title=($("#linkTitle").value||"").trim(); const url=($("#linkURL").value||"").trim(); if(!url){ toast("Enter a URL"); return; } links.push({title:title||url,url}); saveLinks(currentCove,links); $("#linkTitle").value=""; $("#linkURL").value=""; $("#linkPanel").classList.remove("show"); renderLinks(); toast("Link added"); };

  /* Top toolbar actions */
  $("#toggleAdder").onclick=()=>{ $("#adder").classList.toggle("show"); $("#sampleTitle").focus(); };
  $("#cancelSample").onclick=()=>{ $("#adder").classList.remove("show"); $("#sampleTitle").value=""; $("#sampleCode").value=""; };
  $("#saveSample").onclick=()=>{ const title=($("#sampleTitle").value||"").trim(); const code=($("#sampleCode").value||"").replace(/\r\n/g,"\n"); if(!title||!code){ toast("Enter title and code"); return; } const maxOrder=booty.reduce((m,s)=>Math.max(m,s.order||0),0); booty.push({id:Date.now()+Math.floor(Math.random()*1e6),title,code,order:maxOrder+1}); saveBooty(currentCove,booty); $("#sampleTitle").value=""; $("#sampleCode").value=""; $("#adder").classList.remove("show"); currentPage=Math.ceil(booty.length/PAGE_SIZE); renderAll(); duhg.setSourceFromBooty(); toast(labels.booty+" added"); };

  $("#copyAll").onclick=async()=>{ const all=booty.sort((a,b)=>a.order-b.order).map(s=>s.code).join("\n\n\n-- ================================================ --\n\n"); const ok=await copyText(all); toast(ok?("All "+labels.booty+" copied"):"Copy failed"); };

  $("#deleteAll").onclick=(ev)=>{ showPop(ev.pageX||innerWidth/2, ev.pageY||90, "Delete all "+labels.booty+" in "+currentCove, ()=>{ booty=[]; saveBooty(currentCove,booty); currentPage=1; renderAll(); duhg.setSourceFromBooty(); toast("All deleted"); }); };

  $("#newProject").onclick=()=>{ const name=prompt("New "+labels.cove+" name:"); if(!name) return; if(projectIndex.includes(name)){ alert("That name already exists."); return; } projectIndex.push(name); saveIndex(projectIndex); saveBooty(name,[]); saveLinks(name,[]); setCove(name); toast("Created "+labels.cove+" "+name); };
  function renderProjectList(){ const list=$("#projList"); list.innerHTML=""; projectIndex=loadIndex(); projectIndex.forEach(p=>{ const pill=document.createElement("button"); pill.className="pill"; pill.textContent=p; pill.onclick=()=>{ setCove(p); $("#loader").classList.remove("show"); toast("Loaded "+p); }; list.appendChild(pill); }); }
  $("#loadProject").onclick=()=>{ renderProjectList(); $("#loader").classList.add("show"); };
  $("#closeLoader").onclick=()=>$("#loader").classList.remove("show");

  $("#backup").onclick=async()=>{
    const everything={}; const idx=loadIndex();
    idx.forEach(p=>{ everything[p]={ booty: loadBooty(p), links: loadLinks(p) }; });
    if (window.showDirectoryPicker){
      try{
        const dir=await window.showDirectoryPicker({id:"coves-backup"});
        const stamp=new Date().toISOString().replace(/[:.]/g,"-");
        const backupDir=await dir.getDirectoryHandle("backup_"+stamp,{create:true});
        const idxFile=await backupDir.getFileHandle("projects.json",{create:true});
        const idxWritable=await idxFile.createWritable(); await idxWritable.write(JSON.stringify(idx,null,2)); await idxWritable.close();
        const lblFile=await backupDir.getFileHandle("labels.json",{create:true});
        const lwlbl=await lblFile.createWritable(); await lwlbl.write(JSON.stringify(labels,null,2)); await lwlbl.close();
        const themeFile=await backupDir.getFileHandle("theme.json",{create:true});
        const tw=await themeFile.createWritable(); await tw.write(JSON.stringify(theme,null,2)); await tw.close();
        const bgFile=await backupDir.getFileHandle("background.txt",{create:true});
        const bw=await bgFile.createWritable(); await bw.write(loadBg()||""); await bw.close();
        for (const p of idx){
          const pdir=await backupDir.getDirectoryHandle(p,{create:true});
          const sfile=await pdir.getFileHandle("booty.json",{create:true});
          const sw=await sfile.createWritable(); await sw.write(JSON.stringify(loadBooty(p),null,2)); await sw.close();
          const lfile=await pdir.getFileHandle("links.json",{create:true});
          const lw=await lfile.createWritable(); await lw.write(JSON.stringify(loadLinks(p),null,2)); await lw.close();
        }
        toast("Backup saved");
        return;
      }catch(e){ console.warn("Directory picker canceled/failed:", e); }
    }
    const blob=new Blob([JSON.stringify({projects: idx, data: everything, labels, theme, bg: loadBg()||""}, null, 2)],{type:"application/json"});
    const a=document.createElement("a"); a.href=URL.createObjectURL(blob); a.download="coves_backup_"+Date.now()+".json"; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(a.href);
    toast("Backup downloaded");
  };

  /* Duhg toggle */
  const duhgBtn=$("#duhgToggle");
  function setDuhgButton(on){ duhgBtn.textContent = "Duhg Mode: " + (on?"On":"Off"); }
  const duhgSaved = localStorage.getItem(DUHG_KEY) === "1";
  setDuhgButton(duhgSaved);
  if (duhgSaved){ duhg.start(); }
  duhgBtn.onclick=()=>{ const on=!duhg.on; if(on){ duhg.start(); } else { duhg.stop(); } localStorage.setItem(DUHG_KEY, on?"1":"0"); setDuhgButton(on); };

  /* Init UI */
  applyLabels();
  renderBubbles();
  renderLinks();
  renderAll();

})();
</script>
</body>
</html>
