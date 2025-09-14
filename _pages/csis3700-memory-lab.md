---
permalink: /csis3700-memory-lab/
title: "CSIS 3700 - Memory Lab Visualizer"
author_profile: true
---

# CSIS 3700 Memory Lab Visualizer

这个交互式内存可视化器帮助您理解 C++ 中的内存管理概念，包括栈、堆、指针和动态内存分配。

## 功能特点

- **逐步执行**：逐行执行代码，观察内存状态变化
- **可视化展示**：直观显示栈和堆的内存分配
- **多种场景**：涵盖从基础变量到复杂指针操作的8个不同场景
- **交互控制**：支持前进、后退、重置等操作

## 使用方法

1. 选择不同的程序场景（Q1-Q8）
2. 使用"Step"按钮逐步执行代码
3. 观察栈和堆中内存的变化
4. 使用键盘快捷键：←/→ 控制步骤，1-8 切换程序

## 程序场景

- **Q1**: 局部变量（栈）
- **Q2**: new/delete（堆整数）
- **Q3**: new[] 数组
- **Q4**: new Point 对象
- **Q5A**: 十个独立点对象
- **Q5B**: 十个指针指向同一对象
- **Q6**: 删除和悬空指针（别名）
- **Q7**: 指针参数（按值传递）
- **Q8**: int*& 分配（改变调用者指针）

---

<div id="memory-lab-container"></div>

<script type="text/babel">
const { useState, useRef, useMemo, useEffect } = React;

// ---------- helpers ----------
const toHex = (n) => (n === null || n === undefined) ? "" : "0x" + n.toString(16).toUpperCase();
const deep = (obj) => JSON.parse(JSON.stringify(obj));

class MemoryModel {
  constructor() {
    this.nextStack = 0x9000;   // stack grows downward (visual only)
    this.nextHeap  = 0x100000; // heap grows upward (visual only)
    this.stack = {};           // name -> {addr, type, value, meta}
    this.heap = [];            // {addr, size, label, freed, content}
    this.snapshots = [];       // {title, desc, notes[], stack, heap}
  }
  // ----- stack ops -----
  allocStack(name, type, value, meta={}){
    const addr = this.nextStack; this.nextStack -= 0x10;
    this.stack[name] = { addr, type, value, meta };
    return addr;
  }
  ensureStack(name, type){ if(!this.stack[name]) this.allocStack(name, type, undefined); }
  setStack(name, value){ if(this.stack[name]) this.stack[name].value = value; }
  setStackMeta(name, meta){ if(this.stack[name]) this.stack[name].meta = { ...(this.stack[name].meta||{}), ...meta }; }

  // ----- heap ops -----
  allocHeap(label, content, size=1){
    const addr = this.nextHeap; this.nextHeap += 0x20;
    this.heap.push({ addr, size, label, freed:false, content });
    return addr;
  }
  findHeap(addr){ return this.heap.find(b=>b.addr===addr); }
  freeHeap(addr){ const b = this.findHeap(addr); if(b) b.freed = true; }

  // ----- snapshot -----
  snapshot(title, desc, notes=[]) {
    const _notes = Array.isArray(notes) ? notes : (notes ? [notes] : []);
    const annotatedStack = Object.fromEntries(Object.entries(this.stack).map(([k,v])=>{
      let extra = {};
      if(v.type === 'pointer' && v.value && v.value !== 'nullptr'){
        const blk = this.findHeap(v.value);
        if(!blk || blk.freed) extra.dangling = true;
      }
      return [k, { ...v, meta:{...(v.meta||{}), ...extra} }];
    }));
    this.snapshots.push({ title, desc, notes:_notes, stack:deep(annotatedStack), heap:deep(this.heap) });
  }
}

// ---------- Program definition helpers ----------
const prog = (name, lines, exec, introNote="") => ({ id:name, name, lines, exec, introNote });

// ----- Q1 -----
const Q1 = prog(
  'Q1 Local variable (stack)',
  [
    'int i;',
    'i = 0;',
  ],
  (vm, i) => {
    if(i===0){
      vm.allocStack('i','int','(uninitialized)',{warning:'Do not read before init (UB).'});
      vm.snapshot('Q1 • int i;','Local variable on the stack (uninitialized).');
    }
    if(i===1){
      vm.setStack('i',0);
      vm.snapshot('Q1 • i = 0;','After initialization we can safely read i.');
    }
  }
);

// ----- Q2 -----
const Q2 = prog(
  'Q2 new/delete (heap int)',
  [
    'int* p1;',
    'p1 = nullptr;',
    'p1 = new int;',
    '*p1 = 20;',
    'delete p1;',
    'p1 = nullptr;',
  ],
  (vm, i) => {
    if(i===0){ vm.allocStack('p1','pointer','(indeterminate)'); vm.snapshot('Q2 • int* p1;','Uninitialized pointer (do not dereference).'); }
    if(i===1){ vm.setStack('p1','nullptr'); vm.snapshot('Q2 • p1 = nullptr;','Safe null pointer.'); }
    if(i===2){ const h = vm.allocHeap('int',0); vm.setStack('p1',h); vm.snapshot('Q2 • p1 = new int;','Heap int allocated; p1 holds its address.'); }
    if(i===3){ const h = vm.stack['p1'].value; vm.findHeap(h).content = 20; vm.snapshot('Q2 • *p1 = 20;','Write through pointer.'); }
    if(i===4){ const h = vm.stack['p1'].value; vm.freeHeap(h); vm.snapshot('Q2 • delete p1;','Heap freed; p1 still holds old address → dangling.'); }
    if(i===5){ vm.setStack('p1','nullptr'); vm.snapshot('Q2 • p1 = nullptr;','Now safe.'); }
  }
);

// ----- Q3 -----
const Q3 = prog(
  'Q3 new[] array',
  [
    'int* a;',
    'a = new int[4]{0,0,0,0};',
    'a[2] = 20;',
    'delete[] a;',
    'a = nullptr;',
  ],
  (vm, i) => {
    if(i===0){ vm.allocStack('a','pointer','(indeterminate)'); vm.snapshot('Q3 • int* a;','Pointer for dynamic array.'); }
    if(i===1){ const base = vm.allocHeap('int[4]',[0,0,0,0],4); vm.setStack('a',base); vm.snapshot('Q3 • a = new int[4]{0,0,0,0};','a points to first element; block is contiguous.'); }
    if(i===2){ const base = vm.stack['a'].value; vm.findHeap(base).content[2] = 20; vm.snapshot('Q3 • a[2] = 20;','Indexing writes to the 3rd slot.'); }
    if(i===3){ const base = vm.stack['a'].value; vm.freeHeap(base); vm.snapshot('Q3 • delete[] a;','Array freed; a dangling.'); }
    if(i===4){ vm.setStack('a','nullptr'); vm.snapshot('Q3 • a = nullptr;','Now safe.'); }
  }
);

// ----- Q4 -----
const Q4 = prog(
  'Q4 new Point / ->',
  [
    'Point* p;',
    'p = new Point{1.0, 2.0};',
    '(*p).x; p->y;',
    'delete p;',
    'p = nullptr;',
  ],
  (vm, i) => {
    if(i===0){ vm.allocStack('p','pointer','(indeterminate)'); vm.snapshot('Q4 • Point* p;','Pointer to object.'); }
    if(i===1){ const addr = vm.allocHeap('Point',{x:1.0,y:2.0}); vm.setStack('p',addr); vm.snapshot('Q4 • p = new Point{1.0,2.0};','Heap object with x=1.0, y=2.0.'); }
    if(i===2){ vm.snapshot('Q4 • (*p).x; p->y;','Both syntaxes access the same object members.'); }
    if(i===3){ const addr = vm.stack['p'].value; vm.freeHeap(addr); vm.snapshot('Q4 • delete p;','Object freed; p dangling.'); }
    if(i===4){ vm.setStack('p','nullptr'); vm.snapshot('Q4 • p = nullptr;','Now safe.'); }
  }
);

// ----- Q5A -----
const Q5A = prog(
  'Q5A Ten points (independent)',
  [
    'Point* arrA[10];',
    ...Array.from({length:10},(_,i)=>`arrA[${i}] = new Point{1.0, 2.0};`),
    'arrA[0]->x = 100;',
    ...Array.from({length:10},(_,i)=>`delete arrA[${i}];`),
    ...Array.from({length:10},(_,i)=>`arrA[${i}] = nullptr;`),
  ],
  (vm, i) => {
    if(i===0){ vm.snapshot('Q5A • Point* arrA[10];','Ten pointers to independent Points.'); return; }
    if(i>=1 && i<=10){
      const k = i-1; const addr = vm.allocHeap('Point',{x:1.0,y:2.0}); vm.allocStack(`arrA[${k}]`,'pointer',addr); vm.snapshot(`Q5A • arrA[${k}] = new Point{1.0,2.0};`,'Different addresses for each element.'); return;
    }
    if(i===11){ const a0 = vm.stack['arrA[0]'].value; vm.findHeap(a0).content.x = 100.0; vm.snapshot('Q5A • arrA[0]->x = 100;','Only arrA[0] changes.'); return; }
    if(i>=12 && i<=21){ const k=i-12; const addr = vm.stack[`arrA[${k}]`].value; vm.freeHeap(addr); vm.snapshot(`Q5A • delete arrA[${k}];`,'Freed one object.'); return; }
    if(i>=22 && i<=31){ const k=i-22; vm.setStack(`arrA[${k}]`,'nullptr'); vm.snapshot(`Q5A • arrA[${k}] = nullptr;`,'Pointer cleared.'); return; }
  }
);

// ----- Q5B -----
const Q5B = prog(
  'Q5B Ten pointers (aliasing one object)',
  [
    'Point* shared = new Point{1.0, 2.0};',
    ...Array.from({length:10},(_,i)=>`arrB[${i}] = shared;`),
    'arrB[3]->x = 200;',
    'delete shared;',
    ...Array.from({length:10},(_,i)=>`arrB[${i}] = nullptr;`),
  ],
  (vm, i) => {
    if(i===0){ const addr = vm.allocHeap('Point',{x:1.0,y:2.0}); vm.allocStack('shared','pointer',addr); vm.snapshot('Q5B • shared = new Point{1.0,2.0};','One object to be shared by many pointers.'); return; }
    if(i>=1 && i<=10){ const k=i-1; vm.allocStack(`arrB[${k}]`,'pointer', vm.stack['shared'].value); vm.snapshot(`Q5B • arrB[${k}] = shared;`,'All addresses equal.'); return; }
    if(i===11){ const addr = vm.stack['shared'].value; vm.findHeap(addr).content.x = 200.0; vm.snapshot('Q5B • arrB[3]->x = 200;','All aliases observe x=200.'); return; }
    if(i===12){ const addr = vm.stack['shared'].value; vm.freeHeap(addr); vm.snapshot('Q5B • delete shared;','Object freed; all arrB pointers now dangling.'); return; }
    if(i>=13 && i<=22){ const k=i-13; vm.setStack(`arrB[${k}]`,'nullptr'); vm.snapshot(`Q5B • arrB[${k}] = nullptr;`,'Pointer cleared.'); return; }
  }
);

// ----- Q6 -----
const Q6 = prog(
  'Q6 delete & dangling (aliases)',
  [
    'Point* alias[3];',
    'shared = new Point{3.0,3.0};',
    'alias[0] = shared;',
    'alias[1] = shared;',
    'alias[2] = shared;',
    'delete shared;',
    'alias[0] = alias[1] = alias[2] = nullptr;',
  ],
  (vm, i) => {
    if(i===0){ vm.snapshot('Q6 • alias[3]','Three alias pointers.'); }
    if(i===1){ const a = vm.allocHeap('Point',{x:3.0,y:3.0}); vm.allocStack('shared','pointer',a); vm.snapshot('Q6 • shared = new Point{3.0,3.0};'); }
    if(i===2){ vm.allocStack('alias[0]','pointer', vm.stack['shared'].value); vm.snapshot('Q6 • alias[0] = shared;'); }
    if(i===3){ vm.allocStack('alias[1]','pointer', vm.stack['shared'].value); vm.snapshot('Q6 • alias[1] = shared;'); }
    if(i===4){ vm.allocStack('alias[2]','pointer', vm.stack['shared'].value); vm.snapshot('Q6 • alias[2] = shared;'); }
    if(i===5){ const addr = vm.stack['shared'].value; vm.freeHeap(addr); vm.snapshot('Q6 • delete shared;','Aliases are now dangling.'); }
    if(i===6){ vm.setStack('alias[0]','nullptr'); vm.setStack('alias[1]','nullptr'); vm.setStack('alias[2]','nullptr'); vm.snapshot('Q6 • null all aliases','Cleared.'); }
  }
);

// ----- Q7 -----
const Q7 = prog(
  'Q7 pointer parameter (pass-by-value)',
  [
    'int* p = new int(80);',
    'reassign_param(p); // q = new int(42);',
    'set_through_pointer(p); // *p = 42;',
    'delete p;',
    'p = nullptr;',
  ],
  (vm, i) => {
    if(i===0){ const a = vm.allocHeap('int',80); vm.allocStack('p','pointer',a); vm.snapshot('Q7 • int* p = new int(80);'); }
    if(i===1){ /* reassign_param copies pointer and rebinds the copy */ vm.allocHeap('int',42); vm.snapshot('Q7 • reassign_param(p)','Parameter reassigned internally; caller p unchanged.'); }
    if(i===2){ const a = vm.stack['p'].value; vm.findHeap(a).content = 42; vm.snapshot('Q7 • set_through_pointer(p)','The object pointed by p is modified to 42.'); }
    if(i===3){ const a = vm.stack['p'].value; vm.freeHeap(a); vm.snapshot('Q7 • delete p','Freed; p dangling.'); }
    if(i===4){ vm.setStack('p','nullptr'); vm.snapshot('Q7 • p = nullptr','Safe.'); }
  }
);

// ----- Q8 -----
const Q8 = prog(
  'Q8 int*& allocate (change caller pointer)',
  [
    'int* ages = nullptr;',
    'allocate_int_array(ages, 5);',
    'delete[] ages;',
    'ages = nullptr;',
  ],
  (vm, i) => {
    if(i===0){ vm.allocStack('ages','pointer','nullptr'); vm.snapshot('Q8 • int* ages = nullptr;'); }
    if(i===1){ const arr = vm.allocHeap('int[n]',[0,0,0,0,0],5); vm.setStack('ages',arr); vm.snapshot('Q8 • allocate_int_array(ages,5)','Pointer-by-reference changes caller binding.'); }
    if(i===2){ const arr = vm.stack['ages'].value; vm.freeHeap(arr); vm.snapshot('Q8 • delete[] ages;','Freed; ages dangling.'); }
    if(i===3){ vm.setStack('ages','nullptr'); vm.snapshot('Q8 • ages = nullptr;','Safe.'); }
  }
);

const PROGRAMS = [Q1,Q2,Q3,Q4,Q5A,Q5B,Q6,Q7,Q8];

// ---------- Hooks ----------
function useProgram(id){
  const program = useMemo(()=> PROGRAMS.find(p=>p.id===id) || PROGRAMS[0], [id]);
  const vmRef = useRef(new MemoryModel());
  const [idx, setIdx] = useState(-1); // -1 means before first line
  const [snapCount, setSnapCount] = useState(0);

  const reset = ()=>{ vmRef.current = new MemoryModel(); setIdx(-1); setSnapCount(0); };
  const step = ()=>{
    if(idx+1 >= program.lines.length) return;
    const vm = vmRef.current;
    const next = idx+1;
    program.exec(vm, next);
    setIdx(next);
    setSnapCount(vm.snapshots.length);
  };
  const back = ()=>{ if(idx>=0) setIdx(idx-1); };

  const snapshots = vmRef.current.snapshots;
  const currentSnap = (idx>=0 && idx < snapshots.length) ? snapshots[idx] : null;
  return { program, idx, step, back, reset, currentSnap, snapshots };
}

// ---------- UI components ----------
function CodeView({lines, current}){
  return (
    <div className="rounded-2xl border bg-white/90 shadow-sm overflow-hidden">
      <div className="px-3 py-2 text-xs text-slate-600 border-b bg-slate-50">Code (executing one line at a time)</div>
      <pre className="m-0 p-3 text-sm leading-6 font-mono whitespace-pre-wrap">
        {lines.map((ln,i)=> (
          <div key={i} className={`flex gap-3 ${i===current? 'bg-yellow-100' : ''}`}>
            <span className="w-8 text-right text-slate-400 select-none">{i+1}</span>
            <span className="flex-1">{ln}</span>
          </div>
        ))}
      </pre>
    </div>
  );
}

function StackView({stack}){
  const items = Object.entries(stack).sort((a,b)=>b[1].addr - a[1].addr);
  return (
    <div className="space-y-2">
      {items.length===0 && <div className="text-sm text-gray-500">(empty)</div>}
      {items.map(([name,v])=>{
        const addr = toHex(v.addr);
        const isPtr = v.type==='pointer';
        const dangling = v.meta?.dangling;
        return (
          <div key={name} className={`rounded-xl p-3 border shadow-sm bg-white/70 ${dangling? 'border-red-500 ring-1 ring-red-400': 'border-gray-200'}`}>
            <div className="flex justify-between text-sm font-mono">
              <span className="font-semibold">{name}</span>
              <span className="text-gray-500">{addr}</span>
            </div>
            <div className="mt-1 text-sm font-mono">
              {!isPtr && <span>value: <span className="font-semibold">{String(v.value)}</span></span>}
              {isPtr && (
                <span>pointer → <span className={`font-semibold ${dangling? 'text-red-600': 'text-blue-700'}`}>
                  {v.value==='nullptr' ? 'nullptr' : toHex(v.value)}
                </span>{dangling && <span className="ml-2 text-red-600">(dangling)</span>}</span>
              )}
            </div>
            {v.meta?.warning && <div className="mt-1 text-xs text-amber-700">⚠︎ {v.meta.warning}</div>}
          </div>
        )
      })}
    </div>
  );
}

function HeapView({heap}){
  return (
    <div className="grid grid-cols-1 gap-2">
      {heap.length===0 && <div className="text-sm text-gray-500">(no allocations)</div>}
      {heap.map(b=>{
        const addr = toHex(b.addr);
        const freed = b.freed;
        return (
          <div key={addr} className={`rounded-xl p-3 border shadow-sm ${freed? 'bg-gray-100 border-gray-300 opacity-75' : 'bg-emerald-50 border-emerald-200'}`}>
            <div className="flex justify-between text-sm font-mono">
              <span className="font-semibold">{b.label}</span>
              <span className="text-gray-600">{addr}</span>
            </div>
            <div className="mt-1 text-sm font-mono">
              {!Array.isArray(b.content) && typeof b.content === 'object' ? (
                <div className="flex gap-3 flex-wrap">{Object.entries(b.content).map(([k,v])=> (
                  <div key={k} className="px-2 py-0.5 rounded bg-white border text-gray-800">{k}: {String(v)}</div>
                ))}</div>
              ) : Array.isArray(b.content) ? (
                <div className="flex flex-wrap gap-1">{b.content.map((v,i)=> (
                  <div key={i} className="px-2 py-0.5 rounded bg-white border text-gray-800">[{i}] {String(v)}</div>
                ))}</div>
              ) : (
                <div className="px-2 py-0.5 rounded bg-white border inline-block">{String(b.content)}</div>
              )}
            </div>
            {freed && <div className="mt-1 text-xs text-gray-600">(freed)</div>}
          </div>
        )
      })}
    </div>
  );
}

function Controls({onBack, onStep, onReset, idx, max}){
  return (
    <div className="flex items-center gap-2">
      <button onClick={onBack} disabled={idx<0} className="px-3 py-1.5 rounded-xl border bg-white disabled:opacity-50">← Back</button>
      <button onClick={onStep} disabled={idx+1>=max} className="px-3 py-1.5 rounded-xl bg-gray-900 text-white disabled:opacity-50">Step →</button>
      <button onClick={onReset} className="px-3 py-1.5 rounded-xl border bg-white">Reset</button>
      <div className="ml-2 text-sm">Line <span className="font-semibold">{Math.max(0,idx+1)}</span>/<span>{max}</span></div>
    </div>
  );
}

function MemoryLabVisualizer(){
  const [programId, setProgramId] = useState(PROGRAMS[1].id); // default Q2
  const { program, idx, step, back, reset, currentSnap } = useProgram(programId);

  // keyboard shortcuts
  useEffect(()=>{
    const onKey = (e)=>{
      if(e.key==='ArrowRight') step();
      if(e.key==='ArrowLeft') back();
      const map = ['Q1','Q2','Q3','Q4','Q5A','Q5B','Q6','Q7','Q8'];
      if(/^[1-9]$/.test(e.key)){
        const k = parseInt(e.key,10)-1; const id = PROGRAMS[k]?.id; if(id){ setProgramId(id); reset(); }
      }
    };
    window.addEventListener('keydown', onKey);
    return ()=> window.removeEventListener('keydown', onKey);
  }, [step, back, reset]);

  const max = program.lines.length;
  const stack = currentSnap ? currentSnap.stack : {};
  const heap  = currentSnap ? currentSnap.heap  : [];

  return (
    <div className="min-h-full w-full p-6 bg-gradient-to-b from-slate-50 to-slate-100 text-slate-900">
      <div className="max-w-6xl mx-auto">
        <header className="mb-4">
          <h1 className="text-2xl font-bold">CSIS3700 Memory Lab — Program Mode</h1>
          <p className="text-sm text-slate-600">View the entire code and execute one line at a time.</p>
        </header>

        <div className="flex flex-wrap items-center gap-3 mb-4">
          <select value={programId} onChange={(e)=>{ setProgramId(e.target.value); reset(); }} className="px-3 py-2 rounded-xl border bg-white">
            {PROGRAMS.map(p=> <option key={p.id} value={p.id}>{p.name}</option>)}
          </select>
          <Controls onBack={back} onStep={step} onReset={reset} idx={idx} max={max} />
          <div className="ml-auto text-xs text-slate-600">Shortcuts: 1..8 to switch, ←/→ to step</div>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
          <CodeView lines={program.lines} current={idx} />

          <section className="rounded-2xl bg-white/80 backdrop-blur border p-4 shadow-sm">
            <div className="flex items-baseline justify-between mb-3">
              <h2 className="text-lg font-semibold">Stack (simplified)</h2>
              <div className="text-sm text-slate-500">highest → lowest addresses</div>
            </div>
            <StackView stack={stack} />
          </section>
        </div>

        <section className="mt-6 rounded-2xl bg-white/80 backdrop-blur border p-4 shadow-sm">
          <div className="flex items-baseline justify-between mb-3">
            <h2 className="text-lg font-semibold">Heap (allocations)</h2>
            <div className="text-sm text-slate-500">green = live, gray = freed</div>
          </div>
          <HeapView heap={heap} />
        </section>

        {currentSnap && (
          <section className="mt-6 rounded-2xl bg-white/80 border p-4 shadow-sm">
            <div className="text-sm font-semibold mb-1">{currentSnap.title}</div>
            <div className="text-sm text-slate-700">{currentSnap.desc}</div>
          </section>
        )}
      </div>
    </div>
  );
}

// Render the component
ReactDOM.render(<MemoryLabVisualizer />, document.getElementById('memory-lab-container'));
</script>

<script src="https://unpkg.com/react@18/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
