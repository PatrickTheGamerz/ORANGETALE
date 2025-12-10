<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Mini Scratch Clone</title>
<style>
  :root{
    --bg:#0f1724; --panel:#0b1220; --accent:#ffcc00; --muted:#9aa7b2;
    --block:#1f2937; --block-2:#253244; --snap:#0ea5a4;
    --font-sans: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }
  html,body{height:100%;margin:0;font-family:var(--font-sans);background:linear-gradient(180deg,#071021 0%, #071827 100%);color:#e6eef6}
  .app{display:grid;grid-template-columns:260px 1fr 360px;gap:18px;padding:18px;height:100vh;box-sizing:border-box}
  .panel{background:linear-gradient(180deg,var(--panel),#071827);border-radius:12px;padding:14px;box-shadow:0 6px 18px rgba(2,6,23,.6);overflow:hidden}
  .left{display:flex;flex-direction:column;gap:12px}
  h2{margin:0 0 8px 0;font-size:14px;color:var(--muted);letter-spacing:.6px}
  .palette{display:flex;flex-direction:column;gap:10px;overflow:auto;padding-right:6px;height:calc(100% - 60px)}
  .block{user-select:none;cursor:grab;padding:10px 12px;border-radius:8px;background:linear-gradient(180deg,var(--block),var(--block-2));color:#fff;box-shadow:0 4px 8px rgba(2,6,23,.5);display:inline-block}
  .block.move{background:linear-gradient(180deg,#2b6cb0,#1e3a8a)}
  .block.turn{background:linear-gradient(180deg,#f97316,#c2410c)}
  .block.say{background:linear-gradient(180deg,#10b981,#047857)}
  .block.wait{background:linear-gradient(180deg,#7c3aed,#5b21b6)}
  .block.repeat{background:linear-gradient(180deg,#ef4444,#b91c1c)}
  .center{display:flex;flex-direction:column;gap:12px}
  .toolbar{display:flex;gap:8px;align-items:center}
  .btn{background:#0b1220;border:1px solid rgba(255,255,255,.04);color:var(--muted);padding:8px 12px;border-radius:8px;cursor:pointer}
  .btn.primary{background:linear-gradient(90deg,#06b6d4,#0ea5a4);color:#042027;border:none}
  .workspace{flex:1;background:linear-gradient(180deg,#071827,#04101a);border-radius:10px;padding:12px;display:flex;gap:12px;align-items:flex-start}
  .canvas{flex:1;background:linear-gradient(180deg,#071827,#04101a);border-radius:8px;padding:12px;min-height:520px;position:relative;overflow:auto;border:1px dashed rgba(255,255,255,.03)}
  .drop-hint{position:absolute;left:50%;top:8px;transform:translateX(-50%);color:rgba(255,255,255,.06);font-size:12px}
  .script{min-height:40px;padding:8px;border-radius:8px;background:linear-gradient(180deg,#06121a,#04101a);box-shadow:inset 0 1px 0 rgba(255,255,255,.02);display:flex;flex-direction:column;gap:8px}
  .block-instance{display:flex;align-items:center;padding:8px;border-radius:8px;background:linear-gradient(180deg,#1b2b3a,#0f1b26);color:#fff;cursor:grab}
  .block-instance .label{flex:1}
  .block-instance .handle{width:10px;height:10px;border-radius:50%;background:rgba(255,255,255,.06);margin-left:8px}
  .right{display:flex;flex-direction:column;gap:12px}
  .stage{height:360px;background:linear-gradient(180deg,#e6f7ff,#dff6ff);border-radius:8px;position:relative;overflow:hidden;border:6px solid #0b1220}
  .sprite{position:absolute;width:64px;height:64px;background:#ffcc00;border-radius:12px;display:flex;align-items:center;justify-content:center;font-weight:700;color:#071827;box-shadow:0 8px 20px rgba(2,6,23,.4);transform:translate(-50%,-50%);transition:transform .12s linear}
  .controls{display:flex;gap:8px;align-items:center}
  .small{font-size:12px;padding:6px 8px}
  .io{display:flex;gap:8px;align-items:center}
  .json-area{width:100%;height:120px;background:#071827;border-radius:8px;padding:8px;color:var(--muted);font-family:monospace;font-size:12px;border:1px solid rgba(255,255,255,.02)}
  .footer{display:flex;justify-content:space-between;align-items:center;color:var(--muted);font-size:12px}
  /* Snap indicator */
  .snap-line{position:absolute;height:6px;background:linear-gradient(90deg,transparent,#0ea5a4,transparent);left:0;right:0;border-radius:4px;pointer-events:none;opacity:0;transition:opacity .12s}
  /* responsive */
  @media (max-width:1000px){.app{grid-template-columns:1fr;grid-auto-rows:auto;padding:12px}.right{order:3}.left{order:1}.center{order:2}}
</style>
</head>
<body>
<div class="app">
  <div class="panel left">
    <h2>Blocks</h2>
    <div class="palette" id="palette">
      <div class="block move" draggable="true" data-type="move">move 10 steps</div>
      <div class="block turn" draggable="true" data-type="turn">turn 15 degrees</div>
      <div class="block say" draggable="true" data-type="say">say Hello! for 2s</div>
      <div class="block wait" draggable="true" data-type="wait">wait 1 second</div>
      <div class="block repeat" draggable="true" data-type="repeat">repeat 5 times</div>
    </div>
    <div style="display:flex;gap:8px;margin-top:8px">
      <button class="btn" id="exportBtn">Export</button>
      <button class="btn" id="importBtn">Import</button>
    </div>
  </div>

  <div class="panel center">
    <div class="toolbar">
      <button class="btn primary" id="runBtn">Run</button>
      <button class="btn" id="stopBtn">Stop</button>
      <div style="flex:1"></div>
      <div class="controls">
        <div class="io"><strong style="color:var(--muted);margin-right:8px">Speed</strong>
          <input id="speed" type="range" min="0.2" max="2" step="0.1" value="1" />
        </div>
      </div>
    </div>

    <div class="workspace">
      <div class="canvas" id="canvas">
        <div class="drop-hint">Drag blocks here to build a script</div>
        <div class="script" id="scriptArea"></div>
        <div class="snap-line" id="snapLine"></div>
      </div>
    </div>
  </div>

  <div class="panel right">
    <h2>Stage</h2>
    <div class="stage" id="stage">
      <div class="sprite" id="sprite" style="left:50%;top:50%">🙂</div>
    </div>

    <div style="margin-top:12px">
      <h2>Project JSON</h2>
      <textarea id="jsonArea" class="json-area" placeholder="Exported project JSON appears here"></textarea>
    </div>

    <div class="footer" style="margin-top:8px">
      <div>Mini Scratch Clone</div>
      <div style="color:var(--muted)">Built with HTML/CSS/JS</div>
    </div>
  </div>
</div>

<script>
/* Simple block-based editor runtime
   - Blocks are represented as objects with type and params
   - Workspace holds an ordered list of blocks
   - Interpreter runs blocks sequentially, supports repeat
*/

const palette = document.getElementById('palette');
const scriptArea = document.getElementById('scriptArea');
const canvas = document.getElementById('canvas');
const sprite = document.getElementById('sprite');
const runBtn = document.getElementById('runBtn');
const stopBtn = document.getElementById('stopBtn');
const speedInput = document.getElementById('speed');
const exportBtn = document.getElementById('exportBtn');
const importBtn = document.getElementById('importBtn');
const jsonArea = document.getElementById('jsonArea');
const snapLine = document.getElementById('snapLine');

let workspace = []; // array of block instances
let running = false;
let runAbort = false;
let dragData = null;
let idCounter = 1;

// Utility
function uid(){ return 'b'+(idCounter++); }
function clamp(v,a,b){return Math.max(a,Math.min(b,v));}

// Create a visual block instance
function createBlockInstance(type, params = {}) {
  const el = document.createElement('div');
  el.className = 'block-instance';
  el.dataset.type = type;
  el.dataset.id = uid();
  el.style.margin = '6px 0';
  const label = document.createElement('div');
  label.className = 'label';
  label.textContent = renderLabel(type, params);
  el.appendChild(label);
  const handle = document.createElement('div');
  handle.className = 'handle';
  el.appendChild(handle);

  // editable params for some blocks
  if(type === 'move' || type === 'turn' || type === 'say' || type === 'wait' || type === 'repeat'){
    el.addEventListener('dblclick', ()=> editBlockParams(el));
  }

  // drag reorder
  el.draggable = true;
  el.addEventListener('dragstart', (e)=>{
    dragData = {type:'instance', id:el.dataset.id};
    e.dataTransfer.setData('text/plain','instance');
    setTimeout(()=> el.style.opacity = '0.4', 0);
  });
  el.addEventListener('dragend', ()=> { dragData = null; el.style.opacity = '1'; snapLine.style.opacity = 0; });

  // right-click remove
  el.addEventListener('contextmenu', (e)=>{
    e.preventDefault();
    removeBlockById(el.dataset.id);
  });

  return el;
}

function renderLabel(type, params){
  if(type==='move') return `move ${params.steps ?? 10} steps`;
  if(type==='turn') return `turn ${params.deg ?? 15} degrees`;
  if(type==='say') return `say ${params.text ?? 'Hello!'} for ${params.time ?? 2}s`;
  if(type==='wait') return `wait ${params.time ?? 1} second`;
  if(type==='repeat') return `repeat ${params.times ?? 5} times`;
  return type;
}

function editBlockParams(el){
  const type = el.dataset.type;
  const id = el.dataset.id;
  const current = workspace.find(b=>b.id===id);
  if(!current) return;
  if(type==='move'){
    const v = prompt('Steps to move', current.params.steps ?? 10);
    if(v!==null) { current.params.steps = Number(v); el.querySelector('.label').textContent = renderLabel(type,current.params); }
  } else if(type==='turn'){
    const v = prompt('Degrees to turn', current.params.deg ?? 15);
    if(v!==null) { current.params.deg = Number(v); el.querySelector('.label').textContent = renderLabel(type,current.params); }
  } else if(type==='say'){
    const t = prompt('Text to say', current.params.text ?? 'Hello!');
    if(t!==null){
      const s = prompt('Seconds to say', current.params.time ?? 2);
      if(s!==null){ current.params.text = t; current.params.time = Number(s); el.querySelector('.label').textContent = renderLabel(type,current.params); }
    }
  } else if(type==='wait'){
    const s = prompt('Seconds to wait', current.params.time ?? 1);
    if(s!==null){ current.params.time = Number(s); el.querySelector('.label').textContent = renderLabel(type,current.params); }
  } else if(type==='repeat'){
    const t = prompt('Times to repeat', current.params.times ?? 5);
    if(t!==null){ current.params.times = Number(t); el.querySelector('.label').textContent = renderLabel(type,current.params); }
  }
}

// palette drag
palette.querySelectorAll('.block').forEach(b=>{
  b.addEventListener('dragstart', (e)=>{
    dragData = {type:'palette', blockType: b.dataset.type};
    e.dataTransfer.setData('text/plain','palette');
    setTimeout(()=> b.style.opacity = '0.5', 0);
  });
  b.addEventListener('dragend', ()=> { dragData = null; b.style.opacity = '1'; });
});

// canvas drop handling
canvas.addEventListener('dragover', (e)=>{
  e.preventDefault();
  // show snap line where it would be inserted
  const y = e.clientY - canvas.getBoundingClientRect().top + canvas.scrollTop;
  const children = Array.from(scriptArea.children);
  let insertIndex = children.length;
  for(let i=0;i<children.length;i++){
    const r = children[i].getBoundingClientRect();
    const top = r.top - canvas.getBoundingClientRect().top + canvas.scrollTop;
    if(y < top + r.height/2){ insertIndex = i; break; }
  }
  // position snap line
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
    const block = { id: uid(), type: dragData.blockType, params: defaultParams(dragData.blockType) };
    appendBlock(block, null, true);
  } else if(dragData.type === 'instance'){
    // reorder existing
    const id = dragData.id;
    const el = scriptArea.querySelector(`[data-id="${id}"]`);
    if(!el) return;
    // compute insert index
    const y = e.clientY - canvas.getBoundingClientRect().top + canvas.scrollTop;
    const children = Array.from(scriptArea.children).filter(c=>c.dataset.id !== id);
    let insertIndex = children.length;
    for(let i=0;i<children.length;i++){
      const r = children[i].getBoundingClientRect();
      const top = r.top - canvas.getBoundingClientRect().top + canvas.scrollTop;
      if(y < top + r.height/2){ insertIndex = i; break; }
    }
    // remove and reinsert in workspace array and DOM
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

// default params
function defaultParams(type){
  if(type==='move') return {steps:10};
  if(type==='turn') return {deg:15};
  if(type==='say') return {text:'Hello!', time:2};
  if(type==='wait') return {time:1};
  if(type==='repeat') return {times:5, body:[]};
  return {};
}

// append block to workspace and DOM
function appendBlock(block, index=null, scrollIntoView=false){
  const el = createBlockInstance(block.type, block.params);
  el.dataset.id = block.id;
  if(index === null || index >= workspace.length){
    workspace.push(block);
    scriptArea.appendChild(el);
  } else {
    workspace.splice(index,0,block);
    scriptArea.insertBefore(el, scriptArea.children[index]);
  }
  if(scrollIntoView) el.scrollIntoView({behavior:'smooth', block:'center'});
}

// remove block
function removeBlockById(id){
  const idx = workspace.findIndex(b=>b.id===id);
  if(idx===-1) return;
  workspace.splice(idx,1);
  const el = scriptArea.querySelector(`[data-id="${id}"]`);
  if(el) el.remove();
}

// initial demo blocks
appendBlock({id:uid(), type:'move', params:{steps:60}});
appendBlock({id:uid(), type:'turn', params:{deg:45}});
appendBlock({id:uid(), type:'say', params:{text:'Hi!', time:1}});
appendBlock({id:uid(), type:'wait', params:{time:0.6}});

// run/stop logic
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
  }
});

stopBtn.addEventListener('click', ()=> { runAbort = true; });

// interpreter
async function runScript(blocks){
  const speed = Number(speedInput.value);
  // simple sequential interpreter with support for repeat
  async function execList(list){
    for(let i=0;i<list.length;i++){
      if(runAbort) throw new Error('aborted');
      const b = list[i];
      await execBlock(b);
    }
  }
  async function execBlock(b){
    const t = b.type;
    if(t==='move'){
      await animateMove(b.params.steps, speed);
    } else if(t==='turn'){
      await animateTurn(b.params.deg, speed);
    } else if(t==='say'){
      await animateSay(b.params.text, b.params.time, speed);
    } else if(t==='wait'){
      await sleep(b.params.time * 1000 / speed);
    } else if(t==='repeat'){
      const times = Math.max(0, Math.floor(b.params.times || 0));
      for(let i=0;i<times;i++){
        if(runAbort) throw new Error('aborted');
        // if repeat has nested body, run it; otherwise run next blocks as body until end or next repeat (simple)
        // For simplicity, treat repeat as repeating the next immediate block if present
        // Here we support nested body stored in params.body
        if(Array.isArray(b.params.body) && b.params.body.length) await execList(b.params.body);
        else {
          // no nested body: do nothing
        }
      }
    }
  }
  await execList(blocks);
}

// animations
function getSpritePos(){
  const rect = sprite.getBoundingClientRect();
  const stageRect = document.getElementById('stage').getBoundingClientRect();
  const x = (rect.left + rect.width/2) - stageRect.left;
  const y = (rect.top + rect.height/2) - stageRect.top;
  return {x,y};
}
function setSpritePos(x,y){
  const stage = document.getElementById('stage').getBoundingClientRect();
  const cx = clamp(x, 16, stage.width-16);
  const cy = clamp(y, 16, stage.height-16);
  sprite.style.left = (cx) + 'px';
  sprite.style.top = (cy) + 'px';
  sprite.style.transform = 'translate(-50%,-50%)';
}

async function animateMove(steps, speed){
  const stageRect = document.getElementById('stage').getBoundingClientRect();
  // convert steps to pixels (approx)
  const px = steps * 2;
  // move to the right by px with easing
  const start = getSpritePos();
  const targetX = clamp(start.x + px, 16, stageRect.width-16);
  const duration = clamp(Math.abs(px)/200 * 600 / speed, 80, 2000);
  await tween(start.x, targetX, duration, (v)=> setSpritePos(v, start.y));
}

async function animateTurn(deg, speed){
  // rotate sprite visually
  const el = sprite;
  const start = getComputedStyle(el).transform;
  el.style.transition = `transform ${200/speed}ms linear`;
  el.style.transform += ` rotate(${deg}deg)`;
  await sleep(200 / speed);
  // remove rotation transition to keep transform consistent
  el.style.transition = '';
}

async function animateSay(text, seconds, speed){
  const bubble = document.createElement('div');
  bubble.style.position = 'absolute';
  bubble.style.left = sprite.style.left;
  bubble.style.top = (parseFloat(sprite.style.top) - 60) + 'px';
  bubble.style.transform = 'translate(-50%,-50%)';
  bubble.style.padding = '8px 10px';
  bubble.style.background = '#fff';
  bubble.style.color = '#071827';
  bubble.style.borderRadius = '8px';
  bubble.style.boxShadow = '0 6px 18px rgba(2,6,23,.2)';
  bubble.style.fontSize = '14px';
  bubble.textContent = text;
  document.getElementById('stage').appendChild(bubble);
  await sleep((seconds*1000) / speed);
  bubble.remove();
}

function sleep(ms){ return new Promise(res=> setTimeout(res, ms)); }

function tween(a,b,duration, onUpdate){
  return new Promise(res=>{
    const start = performance.now();
    function frame(now){
      const t = clamp((now-start)/duration,0,1);
      const v = a + (b-a) * easeInOutCubic(t);
      onUpdate(v);
      if(t < 1) requestAnimationFrame(frame);
      else res();
    }
    requestAnimationFrame(frame);
  });
}
function easeInOutCubic(t){ return t<.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2,3)/2; }

// export/import
exportBtn.addEventListener('click', ()=>{
  const data = JSON.stringify(workspace, null, 2);
  jsonArea.value = data;
  jsonArea.select();
  document.execCommand('copy');
  exportBtn.textContent = 'Copied';
  setTimeout(()=> exportBtn.textContent = 'Export', 1200);
});

importBtn.addEventListener('click', ()=>{
  try{
    const data = JSON.parse(jsonArea.value);
    // clear
    workspace = [];
    scriptArea.innerHTML = '';
    data.forEach(b=>{
      // ensure id exists
      if(!b.id) b.id = uid();
      appendBlock(b);
    });
    importBtn.textContent = 'Imported';
    setTimeout(()=> importBtn.textContent = 'Import', 1200);
  }catch(e){
    alert('Invalid JSON');
  }
});

// simple keyboard shortcuts
document.addEventListener('keydown', (e)=>{
  if(e.key === 'Delete' || e.key === 'Backspace'){
    // remove selected? not implemented; remove last block
    if(workspace.length) removeBlockById(workspace[workspace.length-1].id);
  }
});

// allow clicking on script area to add block at end
scriptArea.addEventListener('click', (e)=>{
  // nothing for now
});

// small UX: clicking a palette block clones it into workspace
palette.querySelectorAll('.block').forEach(b=>{
  b.addEventListener('click', ()=>{
    const block = { id: uid(), type: b.dataset.type, params: defaultParams(b.dataset.type) };
    appendBlock(block, null, true);
  });
});

// allow nested repeat body by double-clicking repeat to add body
scriptArea.addEventListener('dblclick', (e)=>{
  const el = e.target.closest('.block-instance');
  if(!el) return;
  const id = el.dataset.id;
  const b = workspace.find(x=>x.id===id);
  if(!b) return;
  if(b.type === 'repeat'){
    // open a small modal to add a single block into repeat body
    const choice = prompt('Add a block inside repeat (move/turn/say/wait)', 'move');
    if(choice){
      const t = choice.trim();
      if(['move','turn','say','wait'].includes(t)){
        const child = { id: uid(), type: t, params: defaultParams(t) };
        b.params.body = b.params.body || [];
        b.params.body.push(child);
        alert('Added block to repeat body. Note: nested body executes when running repeat.');
      } else alert('Unsupported block type');
    }
  }
});

// small responsive stage init
(function initStage(){
  // position sprite center
  const st = document.getElementById('stage').getBoundingClientRect();
  setSpritePos(st.width/2, st.height/2);
})();

</script>
</body>
</html>
