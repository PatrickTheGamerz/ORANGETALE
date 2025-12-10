<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Mini Scratch — Improved (Load JSON + Responsive)</title>
<style>
  :root{
    --bg:#04101a; --panel:#071827; --muted:#9fb0c2; --accent:#06b6d4; --text:#e6eef6;
    --card:#0f1724; --glass: rgba(255,255,255,0.03);
  }
  html,body{height:100%;margin:0;background:linear-gradient(180deg,#04101a,#071827);font-family:Inter,system-ui,Segoe UI,Roboto,Arial;color:var(--text);-webkit-font-smoothing:antialiased}
  .app{display:grid;grid-template-columns:300px 1fr 360px;gap:14px;padding:14px;height:100vh;box-sizing:border-box}
  .panel{background:linear-gradient(180deg,var(--card),#06121a);border-radius:12px;padding:12px;box-shadow:0 8px 30px rgba(2,6,23,.6);overflow:hidden;display:flex;flex-direction:column}
  h3{margin:0 0 10px 0;color:var(--muted);font-size:13px;letter-spacing:.6px}
  /* Left palette */
  .palette{flex:1;overflow:auto;padding-right:6px;display:flex;flex-direction:column;gap:8px}
  .palette-search{display:flex;gap:8px;margin-bottom:8px}
  .palette-search input{flex:1;padding:8px;border-radius:8px;border:1px solid var(--glass);background:#071827;color:var(--muted)}
  .block-tpl{display:flex;align-items:center;gap:8px;padding:10px;border-radius:8px;background:linear-gradient(180deg,#122433,#0b1b2a);color:var(--text);cursor:grab;border:1px solid rgba(255,255,255,.02)}
  .block-tpl .kind{width:10px;height:10px;border-radius:50%}
  .kind-move{background:#2b6cb0}.kind-turn{background:#f97316}.kind-look{background:#10b981}.kind-control{background:#ef4444}.kind-sound{background:#7c3aed}.kind-pen{background:#f59e0b}.kind-sense{background:#06b6d4}.kind-operator{background:#06b6d4}.kind-variable{background:#f97316}
  .block-tpl small{color:var(--muted);margin-left:auto}

  /* Center workspace */
  .toolbar{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
  .toolbar .left{display:flex;gap:8px;align-items:center}
  .toolbar .right{display:flex;gap:8px;align-items:center;margin-left:auto}
  .btn{background:#071827;border:1px solid rgba(255,255,255,.03);color:var(--muted);padding:8px 12px;border-radius:8px;cursor:pointer;font-weight:600;white-space:nowrap}
  .btn.primary{background:linear-gradient(90deg,#06b6d4,#0ea5a4);color:#042027;border:none}
  .btn.ghost{background:transparent;border:1px solid var(--glass);color:var(--muted)}
  .workspace{flex:1;background:linear-gradient(180deg,#06121a,#04101a);border-radius:10px;padding:12px;display:flex;gap:12px;align-items:flex-start;min-height:520px}
  .canvas{flex:1;background:linear-gradient(180deg,#071827,#04101a);border-radius:8px;padding:12px;min-height:520px;position:relative;overflow:auto;border:1px dashed rgba(255,255,255,.03)}
  .drop-hint{position:absolute;left:50%;top:8px;transform:translateX(-50%);color:rgba(255,255,255,.04);font-size:12px}
  .script{min-height:40px;padding:8px;border-radius:8px;background:linear-gradient(180deg,#06121a,#04101a);box-shadow:inset 0 1px 0 rgba(255,255,255,.02);display:flex;flex-direction:column;gap:8px}
  .block-instance{display:flex;align-items:center;padding:8px;border-radius:8px;background:linear-gradient(180deg,#122433,#0b1b2a);color:#fff;cursor:grab;gap:8px;min-width:220px}
  .block-instance .label{flex:1;display:flex;gap:6px;align-items:center;flex-wrap:wrap}
  .block-instance input[type="text"], .block-instance input[type="number"], .block-instance select{
    background:rgba(255,255,255,.06);border:none;color:var(--text);padding:4px 6px;border-radius:6px;font-size:13px;min-width:40px
  }

  /* Right stage */
  .stage{height:360px;background:linear-gradient(180deg,#e6f7ff,#dff6ff);border-radius:8px;position:relative;overflow:hidden;border:6px solid #071827}
  #stageCanvas{position:absolute;left:0;top:0;width:100%;height:100%}
  .sprite{position:absolute;width:64px;height:64px;background:#ffcc00;border-radius:12px;display:flex;align-items:center;justify-content:center;font-weight:700;color:#071827;box-shadow:0 8px 20px rgba(2,6,23,.4);transform:translate(-50%,-50%);transition:transform .12s linear}
  .json-area{width:100%;height:120px;background:#071827;border-radius:8px;padding:8px;color:var(--muted);font-family:monospace;font-size:12px;border:1px solid rgba(255,255,255,.02);resize:vertical}
  .footer{display:flex;justify-content:space-between;align-items:center;color:var(--muted);font-size:12px;margin-top:8px}
  .snap-line{position:absolute;height:6px;background:linear-gradient(90deg,transparent,#0ea5a4,transparent);left:0;right:0;border-radius:4px;pointer-events:none;opacity:0;transition:opacity .12s}
  .controls-row{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
  .vars{background:linear-gradient(180deg,#071827,#06121a);border-radius:8px;padding:8px;color:var(--muted);font-family:monospace;font-size:13px;min-height:40px}
  .small{font-size:12px;color:var(--muted)}
  /* responsive */
  @media (max-width:1100px){.app{grid-template-columns:1fr;grid-auto-rows:auto;padding:10px}.right{order:3}.left{order:1}.center{order:2}}
</style>
</head>
<body>
<div class="app">
  <div class="panel left">
    <h3>Blocks (50)</h3>
    <div class="palette-search">
      <input id="search" placeholder="Search blocks..." aria-label="search blocks" />
      <button class="btn ghost" id="resetSearch" title="Reset search">Reset</button>
    </div>
    <div class="palette" id="palette" aria-label="block palette"></div>
    <div style="display:flex;gap:8px;margin-top:8px">
      <button class="btn" id="exportBtn" title="Copy project JSON to textarea">Export</button>
      <button class="btn" id="importBtn" title="Load JSON from textarea">Load JSON</button>
      <label class="btn" style="display:inline-flex;align-items:center;gap:8px;cursor:pointer">
        Load file
        <input id="fileInput" type="file" accept="application/json" style="display:none" />
      </label>
    </div>
  </div>

  <div class="panel center">
    <div class="toolbar" role="toolbar" aria-label="editor toolbar">
      <div class="left">
        <button class="btn primary" id="runBtn" title="Run the script">Run</button>
        <button class="btn" id="stopBtn" title="Stop execution">Stop</button>
        <button class="btn" id="clearBtn" title="Clear workspace">Clear</button>
      </div>
      <div class="right">
        <div class="controls-row">
          <label class="small" style="margin-right:6px">Speed</label>
          <input id="speed" type="range" min="0.2" max="2" step="0.1" value="1" />
        </div>
      </div>
    </div>

    <div class="workspace">
      <div class="canvas" id="canvas">
        <div class="drop-hint">Drag blocks here to build a script</div>
        <div class="script" id="scriptArea" aria-label="workspace"></div>
        <div class="snap-line" id="snapLine"></div>
      </div>
    </div>
  </div>

  <div class="panel right">
    <h3>Stage & Controls</h3>
    <div class="stage" id="stage">
      <canvas id="stageCanvas"></canvas>
      <div class="sprite" id="sprite" style="left:50%;top:50%">🙂</div>
    </div>

    <div style="margin-top:12px;display:flex;gap:8px;flex-direction:column">
      <div style="display:flex;gap:8px;align-items:center">
        <div style="flex:1">
          <h3 style="margin:0 0 6px 0">Project JSON</h3>
          <textarea id="jsonArea" class="json-area" placeholder="Exported project JSON appears here"></textarea>
        </div>
        <div style="width:140px">
          <h3 style="margin:0 0 6px 0">Variables</h3>
          <div class="vars" id="varsUI">score: 0</div>
        </div>
      </div>

      <div class="footer">
        <div>Mini Scratch — Improved</div>
        <div class="small">Drag blocks • Load JSON • Responsive UI</div>
      </div>
    </div>
  </div>
</div>

<script>
/* Improved editor with JSON file load and responsive toolbar
   - Load JSON from file or textarea
   - Export copies JSON to textarea and clipboard
   - Live variables UI
   - Responsive toolbar (wraps) so buttons don't overflow
*/

document.addEventListener('DOMContentLoaded', ()=> {
  // Elements
  const paletteEl = document.getElementById('palette');
  const scriptArea = document.getElementById('scriptArea');
  const canvas = document.getElementById('canvas');
  const sprite = document.getElementById('sprite');
  const runBtn = document.getElementById('runBtn');
  const stopBtn = document.getElementById('stopBtn');
  const clearBtn = document.getElementById('clearBtn');
  const speedInput = document.getElementById('speed');
  const exportBtn = document.getElementById('exportBtn');
  const importBtn = document.getElementById('importBtn');
  const fileInput = document.getElementById('fileInput');
  const jsonArea = document.getElementById('jsonArea');
  const snapLine = document.getElementById('snapLine');
  const searchInput = document.getElementById('search');
  const resetSearch = document.getElementById('resetSearch');
  const varsUI = document.getElementById('varsUI');
  const stageCanvas = document.getElementById('stageCanvas');
  const stage = document.getElementById('stage');
  const stageCtx = stageCanvas.getContext('2d');

  // State
  let workspace = [];
  let running = false;
  let runAbort = false;
  let dragData = null;
  let idCounter = 1;
  let variables = {};
  let broadcasts = {};
  let penState = {down:false,color:'#000000',size:2};
  let timerStart = performance.now();

  // Helpers
  function uid(){ return 'b'+(idCounter++); }
  function clamp(v,a,b){return Math.max(a,Math.min(b,v));}
  function sleep(ms){ return new Promise(r=>setTimeout(r,ms)); }
  function ease(t){ return t<.5?4*t*t*t:1-Math.pow(-2*t+2,3)/2; }

  // Responsive canvas
  function resizeStageCanvas(){
    stageCanvas.width = stage.clientWidth;
    stageCanvas.height = stage.clientHeight;
  }
  window.addEventListener('resize', resizeStageCanvas);
  resizeStageCanvas();

  // Block definitions (50)
  const blocks = [
    {id:'move', title:'move', kind:'move', inputs:[{name:'steps',type:'number',label:'steps',default:10}]},
    {id:'move_by', title:'move by', kind:'move', inputs:[{name:'dx',type:'number',label:'dx',default:10},{name:'dy',type:'number',label:'dy',default:0}]},
    {id:'glide', title:'glide to', kind:'move', inputs:[{name:'x',type:'number',label:'x',default:100},{name:'y',type:'number',label:'y',default:100},{name:'secs',type:'number',label:'secs',default:1}]},
    {id:'set_x', title:'set x to', kind:'move', inputs:[{name:'x',type:'number',label:'x',default:0}]},
    {id:'change_x', title:'change x by', kind:'move', inputs:[{name:'dx',type:'number',label:'dx',default:10}]},
    {id:'set_y', title:'set y to', kind:'move', inputs:[{name:'y',type:'number',label:'y',default:0}]},
    {id:'change_y', title:'change y by', kind:'move', inputs:[{name:'dy',type:'number',label:'dy',default:10}]},
    {id:'go_to', title:'go to', kind:'move', inputs:[{name:'x',type:'number',label:'x',default:0},{name:'y',type:'number',label:'y',default:0}]},
    {id:'point_in_direction', title:'point in direction', kind:'move', inputs:[{name:'dir',type:'number',label:'deg',default:90}]},
    {id:'bounce_if_on_edge', title:'if on edge, bounce', kind:'move', inputs:[]},

    {id:'turn_right', title:'turn right', kind:'turn', inputs:[{name:'deg',type:'number',label:'deg',default:15}]},
    {id:'turn_left', title:'turn left', kind:'turn', inputs:[{name:'deg',type:'number',label:'deg',default:15}]},
    {id:'set_rotation_style', title:'set rotation style', kind:'turn', inputs:[{name:'style',type:'select',label:'style',options:['all around','left-right','don\'t rotate'],default:'all around'}]},
    {id:'show', title:'show', kind:'look', inputs:[]},
    {id:'hide', title:'hide', kind:'look', inputs:[]},
    {id:'switch_costume', title:'next costume', kind:'look', inputs:[]},

    {id:'say', title:'say', kind:'look', inputs:[{name:'text',type:'text',label:'',default:'Hello!'},{name:'secs',type:'number',label:'for',default:2}]},
    {id:'think', title:'think', kind:'look', inputs:[{name:'text',type:'text',label:'',default:'Hmm...'},{name:'secs',type:'number',label:'for',default:2}]},
    {id:'set_size', title:'set size to', kind:'look', inputs:[{name:'size',type:'number',label:'%',default:100}]},
    {id:'change_size', title:'change size by', kind:'look', inputs:[{name:'delta',type:'number',label:'%',default:10}]},
    {id:'set_color', title:'set color', kind:'look', inputs:[{name:'color',type:'text',label:'',default:'#ffcc00'}]},
    {id:'change_color', title:'change color by', kind:'look', inputs:[{name:'delta',type:'number',label:'',default:10}]},

    {id:'play_sound', title:'play sound', kind:'sound', inputs:[{name:'name',type:'text',label:'',default:'pop'}]},
    {id:'play_sound_wait', title:'play sound and wait', kind:'sound', inputs:[{name:'name',type:'text',label:'',default:'pop'}]},
    {id:'stop_sounds', title:'stop all sounds', kind:'sound', inputs:[]},
    {id:'set_volume', title:'set volume to', kind:'sound', inputs:[{name:'vol',type:'number',label:'%',default:100}]},

    {id:'wait', title:'wait', kind:'control', inputs:[{name:'secs',type:'number',label:'seconds',default:1}]},
    {id:'repeat', title:'repeat', kind:'control', inputs:[{name:'times',type:'number',label:'times',default:5}]},
    {id:'forever', title:'forever', kind:'control', inputs:[]},
    {id:'if', title:'if', kind:'control', inputs:[{name:'cond',type:'text',label:'condition',default:'true'}]},
    {id:'if_else', title:'if else', kind:'control', inputs:[{name:'cond',type:'text',label:'condition',default:'true'}]},
    {id:'wait_until', title:'wait until', kind:'control', inputs:[{name:'cond',type:'text',label:'condition',default:'false'}]},
    {id:'stop', title:'stop', kind:'control', inputs:[{name:'what',type:'select',label:'',options:['this script','all','other scripts in sprite'],default:'this script'}]},
    {id:'broadcast', title:'broadcast', kind:'control', inputs:[{name:'msg',type:'text',label:'',default:'message1'}]},

    {id:'ask', title:'ask', kind:'sense', inputs:[{name:'text',type:'text',label:'',default:'What?'}]},
    {id:'answer', title:'answer', kind:'sense', inputs:[]},
    {id:'touching_color', title:'touching color', kind:'sense', inputs:[{name:'color',type:'text',label:'',default:'#000000'}]},
    {id:'touching_sprite', title:'touching sprite', kind:'sense', inputs:[{name:'name',type:'text',label:'',default:'sprite2'}]},
    {id:'distance_to', title:'distance to', kind:'sense', inputs:[{name:'name',type:'text',label:'',default:'mouse'}]},
    {id:'timer_reset', title:'reset timer', kind:'sense', inputs:[]},

    {id:'math', title:'math', kind:'operator', inputs:[{name:'expr',type:'text',label:'',default:'1+1'}]},
    {id:'random', title:'pick random', kind:'operator', inputs:[{name:'min',type:'number',label:'min',default:1},{name:'max',type:'number',label:'max',default:10}]},
    {id:'join', title:'join', kind:'operator', inputs:[{name:'a',type:'text',label:'',default:'hello'},{name:'b',type:'text',label:'',default:'world'}]},
    {id:'length', title:'length of', kind:'operator', inputs:[{name:'s',type:'text',label:'',default:'hello'}]},

    {id:'set_var', title:'set', kind:'variable', inputs:[{name:'var',type:'text',label:'',default:'myVar'},{name:'value',type:'text',label:'to',default:'0'}]},
    {id:'change_var', title:'change', kind:'variable', inputs:[{name:'var',type:'text',label:'',default:'myVar'},{name:'delta',type:'number',label:'by',default:1}]},
    {id:'show_var', title:'show variable', kind:'variable', inputs:[{name:'var',type:'text',label:'',default:'myVar'}]},
    {id:'hide_var', title:'hide variable', kind:'variable', inputs:[{name:'var',type:'text',label:'',default:'myVar'}]},
    {id:'set_list', title:'set list', kind:'variable', inputs:[{name:'list',type:'text',label:'',default:'myList'},{name:'value',type:'text',label:'to',default:''}]},
    {id:'add_to_list', title:'add to list', kind:'variable', inputs:[{name:'list',type:'text',label:'',default:'myList'},{name:'value',type:'text',label:'',default:''}]},

    {id:'pen_down', title:'pen down', kind:'pen', inputs:[]},
    {id:'pen_up', title:'pen up', kind:'pen', inputs:[]},
    {id:'pen_set_color', title:'set pen color', kind:'pen', inputs:[{name:'color',type:'text',label:'',default:'#000000'}]},
    {id:'pen_set_size', title:'set pen size', kind:'pen', inputs:[{name:'size',type:'number',label:'',default:2}]}
  ];

  // Render palette
  function renderPalette(filter=''){
    paletteEl.innerHTML = '';
    const list = blocks.filter(b=>{
      if(!filter) return true;
      const f = filter.toLowerCase();
      if(b.title.toLowerCase().includes(f)) return true;
      return (b.inputs||[]).some(i=> String(i.name).toLowerCase().includes(f) || String(i.default||'').toLowerCase().includes(f));
    });
    if(list.length === 0){
      const empty = document.createElement('div');
      empty.style.color = 'var(--muted)';
      empty.style.padding = '8px';
      empty.textContent = 'No blocks match your search';
      paletteEl.appendChild(empty);
      return;
    }
    list.forEach(b=>{
      const el = document.createElement('div');
      el.className = 'block-tpl';
      el.draggable = true;
      el.dataset.type = b.id;
      el.innerHTML = `<div class="kind kind-${b.kind}"></div><div style="flex:1">${b.title}</div><small>${b.inputs.length? b.inputs.map(i=>i.label||i.name).join(' '):''}</small>`;
      paletteEl.appendChild(el);

      el.addEventListener('dragstart', (e)=>{
        dragData = {type:'palette', blockType: b.id};
        e.dataTransfer.setData('text/plain','palette');
        setTimeout(()=> el.style.opacity = '0.5', 0);
      });
      el.addEventListener('dragend', ()=> { dragData = null; el.style.opacity = '1'; });
      el.addEventListener('click', ()=> {
        const block = { id: uid(), type: b.id, params: cloneDefaults(b) };
        appendBlock(block, null, true);
      });
    });
  }

  // Search handlers
  searchInput.addEventListener('input', ()=> renderPalette(searchInput.value));
  resetSearch.addEventListener('click', ()=> { searchInput.value=''; renderPalette(); });

  // Clone defaults
  function cloneDefaults(def){
    const p = {};
    (def.inputs||[]).forEach(i=> p[i.name] = i.default);
    return p;
  }

  // Create block instance element with inline inputs
  function createBlockInstance(def, instance){
    const el = document.createElement('div');
    el.className = 'block-instance';
    el.dataset.type = def.id;
    el.dataset.id = instance.id;
    el.style.margin = '6px 0';
    const label = document.createElement('div');
    label.className = 'label';
    const titleSpan = document.createElement('span');
    titleSpan.className = 'small';
    titleSpan.textContent = def.title;
    label.appendChild(titleSpan);

    (def.inputs||[]).forEach(inp=>{
      if(inp.type === 'text'){
        const input = document.createElement('input');
        input.type = 'text';
        input.value = instance.params[inp.name] ?? inp.default ?? '';
        input.addEventListener('input', ()=> { instance.params[inp.name] = input.value; updateVarsUI(); });
        label.appendChild(input);
      } else if(inp.type === 'number'){
        const input = document.createElement('input');
        input.type = 'number';
        input.value = instance.params[inp.name] ?? inp.default ?? 0;
        input.style.width = '80px';
        input.addEventListener('input', ()=> { instance.params[inp.name] = Number(input.value); updateVarsUI(); });
        label.appendChild(input);
      } else if(inp.type === 'select'){
        const sel = document.createElement('select');
        (inp.options||[]).forEach(opt=> {
          const o = document.createElement('option'); o.value = opt; o.textContent = opt; sel.appendChild(o);
        });
        sel.value = instance.params[inp.name] ?? inp.default;
        sel.addEventListener('change', ()=> { instance.params[inp.name] = sel.value; updateVarsUI(); });
        label.appendChild(sel);
      }
    });

    el.appendChild(label);
    const handle = document.createElement('div');
    handle.className = 'handle';
    handle.style.width='10px'; handle.style.height='10px'; handle.style.borderRadius='50%'; handle.style.background='rgba(255,255,255,.06)';
    el.appendChild(handle);

    // interactions
    el.draggable = true;
    el.addEventListener('dragstart', (e)=>{
      dragData = {type:'instance', id:el.dataset.id};
      e.dataTransfer.setData('text/plain','instance');
      setTimeout(()=> el.style.opacity = '0.4', 0);
    });
    el.addEventListener('dragend', ()=> { dragData = null; el.style.opacity = '1'; snapLine.style.opacity = 0; });

    el.addEventListener('contextmenu', (e)=>{
      e.preventDefault();
      removeBlockById(el.dataset.id);
    });

    el.addEventListener('dblclick', ()=>{
      const first = el.querySelector('input,select');
      if(first) first.focus();
    });

    return el;
  }

  // Append block to workspace
  function appendBlock(block, index=null, scrollIntoView=false){
    const def = blocks.find(b=>b.id===block.type);
    if(!def) return;
    if(index === null || index >= workspace.length){
      workspace.push(block);
      const el = createBlockInstance(def, block);
      scriptArea.appendChild(el);
    } else {
      workspace.splice(index,0,block);
      const el = createBlockInstance(def, block);
      scriptArea.insertBefore(el, scriptArea.children[index]);
    }
    if(scrollIntoView){
      const el = scriptArea.querySelector(`[data-id="${block.id}"]`);
      if(el) el.scrollIntoView({behavior:'smooth', block:'center'});
    }
    updateVarsUI();
  }

  // Remove block
  function removeBlockById(id){
    const idx = workspace.findIndex(b=>b.id===id);
    if(idx===-1) return;
    workspace.splice(idx,1);
    const el = scriptArea.querySelector(`[data-id="${id}"]`);
    if(el) el.remove();
    updateVarsUI();
  }

  // Initial demo blocks
  appendBlock({id:uid(), type:'move', params:{steps:40}});
  appendBlock({id:uid(), type:'turn_right', params:{deg:45}});
  appendBlock({id:uid(), type:'say', params:{text:'Hello!', secs:2}});
  appendBlock({id:uid(), type:'wait', params:{secs:0.6}});

  // Drag & drop into canvas
  canvas.addEventListener('dragover', (e)=>{
    e.preventDefault();
    const y = e.clientY - canvas.getBoundingClientRect().top + canvas.scrollTop;
    const children = Array.from(scriptArea.children);
    let insertIndex = children.length;
    for(let i=0;i<children.length;i++){
      const r = children[i].getBoundingClientRect();
      const top = r.top - canvas.getBoundingClientRect().top + canvas.scrollTop;
      if(y < top + r.height/2){ insertIndex = i; break; }
    }
    if(insertIndex === children.length){
      snapLine.style.top = (scriptArea.getBoundingClientRect().bottom - canvas.getBoundingClientRect().top + canvas.scrollTop - 8) + 'px';
    } else {
      const target = children[insertIndex];
      snapLine.style.top = (target.getBoundingClientRect().top - canvas.getBoundingClientRect().top + canvas.scrollTop - 6) + 'px';
    }
    snapLine.style.opacity = 1;
  });
  canvas.addEventListener('dragleave', ()=> snapLine.style.opacity = 0);

  canvas.addEventListener('drop', (e)=>{
    e.preventDefault();
    snapLine.style.opacity = 0;
    if(!dragData) return;
    if(dragData.type === 'palette'){
      const def = blocks.find(b=>b.id===dragData.blockType);
      if(!def) return;
      const block = { id: uid(), type: def.id, params: cloneDefaults(def) };
      appendBlock(block, null, true);
    } else if(dragData.type === 'instance'){
      const id = dragData.id;
      const el = scriptArea.querySelector(`[data-id="${id}"]`);
      if(!el) return;
      const y = e.clientY - canvas.getBoundingClientRect().top + canvas.scrollTop;
      const children = Array.from(scriptArea.children).filter(c=>c.dataset.id !== id);
      let insertIndex = children.length;
      for(let i=0;i<children.length;i++){
        const r = children[i].getBoundingClientRect();
        const top = r.top - canvas.getBoundingClientRect().top + canvas.scrollTop;
        if(y < top + r.height/2){ insertIndex = i; break; }
      }
      const idx = workspace.findIndex(b=>b.id===id);
      if(idx === -1) return;
      const [block] = workspace.splice(idx,1);
      workspace.splice(insertIndex,0,block);
      scriptArea.removeChild(el);
      if(insertIndex >= scriptArea.children.length) scriptArea.appendChild(el);
      else scriptArea.insertBefore(el, scriptArea.children[insertIndex]);
    }
    dragData = null;
  });

  // Run/Stop/Clear
  runBtn.addEventListener('click', async ()=>{
    if(running) return;
    running = true; runAbort = false;
    runBtn.textContent = 'Running...';
    runBtn.classList.remove('primary');
    runBtn.classList.add('btn');
    try{
      await runScript(workspace);
    }catch(e){
      console.error(e);
    } finally {
      running = false;
      runBtn.textContent = 'Run';
      runBtn.classList.remove('btn');
      runBtn.classList.add('primary');
      updateVarsUI();
    }
  });
  stopBtn.addEventListener('click', ()=> { runAbort = true; });
  clearBtn.addEventListener('click', ()=> { workspace=[]; scriptArea.innerHTML=''; updateVarsUI(); });

  // Export: copy JSON to textarea and clipboard
  exportBtn.addEventListener('click', ()=>{
    const data = JSON.stringify({workspace,variables}, null, 2);
    jsonArea.value = data;
    jsonArea.select();
    try{ document.execCommand('copy'); }catch(e){}
    exportBtn.textContent = 'Copied';
    setTimeout(()=> exportBtn.textContent = 'Export', 1200);
  });

  // Import from textarea
  importBtn.addEventListener('click', ()=> {
    try{
      const data = JSON.parse(jsonArea.value);
      loadProject(data);
      importBtn.textContent = 'Loaded';
      setTimeout(()=> importBtn.textContent = 'Load JSON', 1200);
    }catch(e){
      alert('Invalid JSON in textarea');
    }
  });

  // Load from file input
  fileInput.addEventListener('change', (ev)=>{
    const f = ev.target.files && ev.target.files[0];
    if(!f) return;
    const reader = new FileReader();
    reader.onload = ()=> {
      try{
        const data = JSON.parse(reader.result);
        loadProject(data);
      }catch(e){
        alert('Invalid JSON file');
      }
    };
    reader.readAsText(f);
    // reset input so same file can be reloaded later
    fileInput.value = '';
  });

  // Load project helper
  function loadProject(data){
    workspace = data.workspace || [];
    variables = data.variables || {};
    scriptArea.innerHTML = '';
    workspace.forEach(b=>{
      const def = blocks.find(x=>x.id===b.type);
      if(!def) return;
      if(!b.id) b.id = uid();
      appendBlock(b);
    });
    updateVarsUI();
  }

  // Update variables UI
  function updateVarsUI(){
    const keys = Object.keys(variables);
    if(keys.length === 0){
      varsUI.textContent = '—';
      return;
    }
    varsUI.innerHTML = keys.map(k=>`${k}: <strong style="color:#fff">${variables[k]}</strong>`).join('<br>');
  }

  // Interpreter (same core as before, with live variable updates)
  async function runScript(blocksList){
    const speed = Number(speedInput.value);
    function evalExpr(expr){
      try{
        const safe = String(expr).replace(/\b([A-Za-z_]\w*)\b/g, (m)=>{
          if(Object.prototype.hasOwnProperty.call(variables,m)) return JSON.stringify(variables[m]);
          return m;
        });
        // eslint-disable-next-line no-eval
        return eval(safe);
      }catch(e){
        return false;
      }
    }

    async function execList(list){
      for(let i=0;i<list.length;i++){
        if(runAbort) throw new Error('aborted');
        const b = list[i];
        await execBlock(b);
      }
    }

    async function execBlock(b){
      const def = blocks.find(x=>x.id===b.type);
      if(!def) return;
      const p = b.params || {};
      // Implemented behaviors (core)
      if(b.type === 'move'){
        await animateMove(p.steps || 10, speed);
      } else if(b.type === 'move_by'){
        const cur = getSpritePos();
        setSpritePos(cur.x + (p.dx||0), cur.y + (p.dy||0));
        await sleep(20);
      } else if(b.type === 'glide'){
        await animateGlide(p.x||0, p.y||0, Math.max(0.01,p.secs||0.5), speed);
      } else if(b.type === 'set_x'){
        const pos = getSpritePos(); setSpritePos(p.x||0, pos.y);
      } else if(b.type === 'change_x'){
        const pos = getSpritePos(); setSpritePos(pos.x + (p.dx||0), pos.y);
      } else if(b.type === 'set_y'){
        const pos = getSpritePos(); setSpritePos(pos.x, p.y||0);
      } else if(b.type === 'change_y'){
        const pos = getSpritePos(); setSpritePos(pos.x, pos.y + (p.dy||0));
      } else if(b.type === 'go_to'){
        setSpritePos(p.x||0, p.y||0);
      } else if(b.type === 'point_in_direction'){
        sprite.dataset.dir = (p.dir||0);
        sprite.style.transform = `translate(-50%,-50%) rotate(${sprite.dataset.dir}deg)`;
      } else if(b.type === 'bounce_if_on_edge'){
        const pos = getSpritePos();
        const st = stage.getBoundingClientRect();
        if(pos.x < 20 || pos.x > st.width-20) sprite.dataset.dir = (Number(sprite.dataset.dir||0) + 180) % 360;
        if(pos.y < 20 || pos.y > st.height-20) sprite.dataset.dir = (Number(sprite.dataset.dir||0) + 180) % 360;
        sprite.style.transform = `translate(-50%,-50%) rotate(${sprite.dataset.dir}deg)`;
      } else if(b.type === 'turn_right'){
        sprite.dataset.dir = (Number(sprite.dataset.dir||0) + (p.deg||15)) % 360;
        sprite.style.transform = `translate(-50%,-50%) rotate(${sprite.dataset.dir}deg)`;
      } else if(b.type === 'turn_left'){
        sprite.dataset.dir = (Number(sprite.dataset.dir||0) - (p.deg||15)) % 360;
        sprite.style.transform = `translate(-50%,-50%) rotate(${sprite.dataset.dir}deg)`;
      } else if(b.type === 'show'){
        sprite.style.display = 'flex';
      } else if(b.type === 'hide'){
        sprite.style.display = 'none';
      } else if(b.type === 'say' || b.type === 'think'){
        await animateSay(p.text||'', p.secs||2, speed, b.type==='think');
      } else if(b.type === 'set_size'){
        sprite.style.width = (64 * ((p.size||100)/100)) + 'px';
        sprite.style.height = (64 * ((p.size||100)/100)) + 'px';
      } else if(b.type === 'change_size'){
        const curW = parseFloat(getComputedStyle(sprite).width);
        sprite.style.width = (curW + (p.delta||10)) + 'px';
        sprite.style.height = sprite.style.width;
      } else if(b.type === 'set_color'){
        sprite.style.background = p.color || '#ffcc00';
      } else if(b.type === 'change_color'){
        sprite.style.filter = `hue-rotate(${(Number(p.delta)||0)}deg)`;
      } else if(b.type === 'play_sound' || b.type === 'play_sound_wait'){
        await sleep(200);
      } else if(b.type === 'stop_sounds'){
        // no-op
      } else if(b.type === 'set_volume'){
        // no-op
      } else if(b.type === 'wait'){
        await sleep((p.secs||1)*1000 / speed);
      } else if(b.type === 'repeat'){
        const times = Math.max(0, Math.floor(p.times||0));
        if(Array.isArray(p.body) && p.body.length){
          for(let i=0;i<times;i++){
            if(runAbort) throw new Error('aborted');
            await execList(p.body);
          }
        } else {
          const idx = workspace.findIndex(x=>x.id===b.id);
          if(idx !== -1 && idx+1 < workspace.length){
            for(let i=0;i<times;i++){
              if(runAbort) throw new Error('aborted');
              await execBlock(workspace[idx+1]);
            }
          }
        }
      } else if(b.type === 'forever'){
        while(!runAbort){
          await sleep(200);
        }
      } else if(b.type === 'if'){
        if(evalExpr(p.cond)) {
          // placeholder
        }
      } else if(b.type === 'if_else'){
        // placeholder
      } else if(b.type === 'wait_until'){
        while(!evalExpr(p.cond)){
          if(runAbort) throw new Error('aborted');
          await sleep(100);
        }
      } else if(b.type === 'stop'){
        if(p.what === 'all') runAbort = true;
      } else if(b.type === 'broadcast'){
        broadcasts[p.msg] = (broadcasts[p.msg]||0) + 1;
      } else if(b.type === 'ask'){
        const ans = prompt(p.text||'');
        variables['answer'] = ans;
        updateVarsUI();
      } else if(b.type === 'answer'){
        return variables['answer'];
      } else if(b.type === 'touching_color'){
        return (sprite.style.background === p.color);
      } else if(b.type === 'distance_to'){
        return 0;
      } else if(b.type === 'timer_reset'){
        timerStart = performance.now();
      } else if(b.type === 'math'){
        return evalExpr(p.expr);
      } else if(b.type === 'random'){
        const a = Number(p.min||0), b2 = Number(p.max||1);
        return Math.floor(Math.random()*(b2-a+1))+a;
      } else if(b.type === 'join'){
        return String(p.a) + String(p.b);
      } else if(b.type === 'length'){
        return String(p.s).length;
      } else if(b.type === 'set_var'){
        variables[p.var] = p.value;
        updateVarsUI();
      } else if(b.type === 'change_var'){
        variables[p.var] = (Number(variables[p.var]||0) + Number(p.delta||0));
        updateVarsUI();
      } else if(b.type === 'show_var' || b.type === 'hide_var'){
        // no-op
      } else if(b.type === 'set_list' || b.type === 'add_to_list'){
        // no-op
      } else if(b.type === 'pen_down'){
        penState.down = true;
      } else if(b.type === 'pen_up'){
        penState.down = false;
      } else if(b.type === 'pen_set_color'){
        penState.color = p.color || '#000';
      } else if(b.type === 'pen_set_size'){
        penState.size = Number(p.size||2);
      }
    }

    await execList(blocksList);
  }

  // Sprite helpers
  function getSpritePos(){
    const rect = sprite.getBoundingClientRect();
    const stageRect = stage.getBoundingClientRect();
    const x = (rect.left + rect.width/2) - stageRect.left;
    const y = (rect.top + rect.height/2) - stageRect.top;
    return {x,y};
  }
  function setSpritePos(x,y){
    const st = stage.getBoundingClientRect();
    const cx = clamp(x, 16, st.width-16);
    const cy = clamp(y, 16, st.height-16);
    sprite.style.left = (cx) + 'px';
    sprite.style.top = (cy) + 'px';
    sprite.style.transform = `translate(-50%,-50%) rotate(${sprite.dataset.dir||0}deg)`;
    if(penState.down){
      stageCtx.fillStyle = penState.color;
      stageCtx.beginPath();
      stageCtx.arc(cx, cy, penState.size, 0, Math.PI*2);
      stageCtx.fill();
    }
  }

  // Animations
  async function animateMove(steps, speed){
    const px = (steps||10) * 2;
    const start = getSpritePos();
    const targetX = clamp(start.x + px, 16, stageCanvas.width-16);
    const duration = clamp(Math.abs(px)/200 * 600 / speed, 80, 1200);
    await tween(start.x, targetX, duration, (v)=> setSpritePos(v, start.y));
  }
  async function animateGlide(x,y,secs,speed){
    const start = getSpritePos();
    const duration = Math.max(20, secs*1000 / speed);
    await tweenMulti(start.x, start.y, x, y, duration, (nx,ny)=> setSpritePos(nx,ny));
  }
  async function animateSay(text, secs, speed, think=false){
    const bubble = document.createElement('div');
    bubble.style.position = 'absolute';
    bubble.style.left = sprite.style.left;
    bubble.style.top = (parseFloat(sprite.style.top) - 60) + 'px';
    bubble.style.transform = 'translate(-50%,-50%)';
    bubble.style.padding = '8px 10px';
    bubble.style.background = think ? '#f3f4f6' : '#fff';
    bubble.style.color = '#071827';
    bubble.style.borderRadius = '8px';
    bubble.style.boxShadow = '0 6px 18px rgba(2,6,23,.2)';
    bubble.style.fontSize = '14px';
    bubble.textContent = text;
    stage.appendChild(bubble);
    await sleep((secs*1000) / speed);
    bubble.remove();
  }
  function tween(a,b,duration,onUpdate){
    return new Promise(res=>{
      const start = performance.now();
      function frame(now){
        const t = clamp((now-start)/duration,0,1);
        const v = a + (b-a) * ease(t);
        onUpdate(v);
        if(t < 1) requestAnimationFrame(frame);
        else res();
      }
      requestAnimationFrame(frame);
    });
  }
  function tweenMulti(x1,y1,x2,y2,duration,onUpdate){
    return new Promise(res=>{
      const start = performance.now();
      function frame(now){
        const t = clamp((now-start)/duration,0,1);
        const v = ease(t);
        const nx = x1 + (x2-x1)*v;
        const ny = y1 + (y2-y1)*v;
        onUpdate(nx,ny);
        if(t < 1) requestAnimationFrame(frame);
        else res();
      }
      requestAnimationFrame(frame);
    });
  }

  // Keyboard shortcuts
  document.addEventListener('keydown', (e)=>{
    if(e.key === 'Delete' || e.key === 'Backspace'){
      if(workspace.length) removeBlockById(workspace[workspace.length-1].id);
    }
  });

  // Initialize palette and stage
  renderPalette();
  (function init(){
    const st = stage.getBoundingClientRect();
    setSpritePos(st.width/2, st.height/2);
    sprite.dataset.dir = 0;
    stageCtx.clearRect(0,0,stageCanvas.width,stageCanvas.height);
    updateVarsUI();
  })();
  
});
</script>
</body>
</html>
