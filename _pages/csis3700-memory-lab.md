---
permalink: /csis3700-memory-lab/
title: "CSIS 3700 - Memory Lab Visualizer"
author_profile: true
---

# CSIS 3700 Memory Lab Visualizer

This interactive memory visualizer helps you understand C++ memory management concepts, including stack, heap, pointers, and dynamic memory allocation.

## Features

- **Step-by-step execution**: Execute code line by line and observe memory state changes
- **Visual representation**: Intuitive display of stack and heap memory allocations
- **Multiple scenarios**: 8 different scenarios covering from basic variables to complex pointer operations
- **Interactive controls**: Support for forward, backward, and reset operations

## How to Use

1. Select different program scenarios (Q1-Q8)
2. Use the "Step" button to execute code line by line
3. Observe memory changes in stack and heap
4. Use keyboard shortcuts: ←/→ to control steps, 1-8 to switch programs

## Program Scenarios

- **Q1**: Local variables (stack)
- **Q2**: new/delete (heap integer)
- **Q3**: new[] array
- **Q4**: new Point object
- **Q5A**: Ten independent point objects
- **Q5B**: Ten pointers pointing to the same object
- **Q6**: Delete and dangling pointers (aliases)
- **Q7**: Pointer parameters (pass-by-value)
- **Q8**: int*& allocation (change caller pointer)

---

<div id="memory-lab-container"></div>

<!-- Load Tailwind CSS for styling -->
<script src="https://cdn.tailwindcss.com"></script>

<!-- Load React and Babel first -->
<script src="https://unpkg.com/react@18/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

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
// A Program has: id, name, lines, exec(vm, lineIndex), meta: {question, conclusion}
const prog = (id, name, lines, exec, meta={}) => ({ id, name, lines, exec, ...meta });

// Short Q/A texts per requirement
const QA = {
  Q1: {
    question: "Local variable on the stack: declare int i; print &i; then initialize i=0 and print i and &i. What does this show?",
    conclusion: "An uninitialized local has an indeterminate value; it lives on the stack and is safe to read only after initialization."
  },
  Q2: {
    question: "new allocates on the heap and delete frees it — what happens to the pointer variable before/after?",
    conclusion: "new returns a heap address; delete frees the heap block but does not reset the pointer variable — set it to nullptr."
  },
  Q3: {
    question: "Dynamic array with new[]: are a[i] and *(a+i) equivalent, and how should it be freed?",
    conclusion: "A dynamic array is contiguous; a[i] ≡ *(a+i); arrays must be freed with delete[]."
  },
  Q4: {
    question: "Dynamic object: confirm (*p).member ≡ p->member; what must be done after delete?",
    conclusion: "(*p).member and p->member are equivalent; after delete, null the pointer to avoid dangling."
  },
  Q5A: {
    question: "Ten points with the same coordinates as independent objects: do changes propagate? How to free?",
    conclusion: "Same values, different objects → different addresses; changes don't propagate; delete each and then null pointers."
  },
  Q5B: {
    question: "Ten pointers aliasing the same object: do changes propagate? What happens after a single delete?",
    conclusion: "All pointers share one address; any change is seen by all; after delete they all dangle and must be nulled."
  },
  Q6: {
    question: "In an aliasing scenario, what exactly does delete remove, and what remains?",
    conclusion: "Delete frees only the heap block; aliasing pointer variables retain the old address (dangling) until you clear them."
  },
  Q7: {
    question: "Pointer parameter passed by value vs. modifying through the pointer — which affects the caller?",
    conclusion: "Reassigning the parameter doesn't change the caller's binding; dereferencing modifies the caller's object; beware leaks."
  },
  Q8: {
    question: "Pointer-by-reference (int*&): can a function change the caller's pointer to a new allocation?",
    conclusion: "Passing a pointer by reference (T*&) or returning a pointer changes the caller's binding; remember delete[]."
  }
};

// ----- Q1 -----
const Q1 = prog(
  'Q1','Q1 Local variable (stack)',
  [ 'int i;', 'i = 0;' ],
  (vm, i) => {
    if(i===0){ vm.allocStack('i','int','(uninitialized)',{warning:'Do not read before init (UB).'}); vm.snapshot('Q1 • int i;','Local variable on the stack (uninitialized).'); }
    if(i===1){ vm.setStack('i',0); vm.snapshot('Q1 • i = 0;','After initialization we can safely read i.'); }
  }, QA.Q1
);

// ----- Q2 -----
const Q2 = prog(
  'Q2','Q2 new/delete (heap int)',
  [ 'int* p1;', 'p1 = nullptr;', 'p1 = new int;', '*p1 = 20;', 'delete p1;', 'p1 = nullptr;' ],
  (vm, i) => {
    if(i===0){ vm.allocStack('p1','pointer','(indeterminate)'); vm.snapshot('Q2 • int* p1;','Uninitialized pointer (do not dereference).'); }
    if(i===1){ vm.setStack('p1','nullptr'); vm.snapshot('Q2 • p1 = nullptr;','Safe null pointer.'); }
    if(i===2){ const h = vm.allocHeap('int',0); vm.setStack('p1',h); vm.snapshot('Q2 • p1 = new int;','Heap int allocated; p1 holds its address.'); }
    if(i===3){ const h = vm.stack['p1'].value; vm.findHeap(h).content = 20; vm.snapshot('Q2 • *p1 = 20;','Write through pointer.'); }
    if(i===4){ const h = vm.stack['p1'].value; vm.freeHeap(h); vm.snapshot('Q2 • delete p1;','Heap freed; p1 still holds old address → dangling.'); }
    if(i===5){ vm.setStack('p1','nullptr'); vm.snapshot('Q2 • p1 = nullptr;','Now safe.'); }
  }, QA.Q2
);

// ----- Q3 -----
const Q3 = prog(
  'Q3','Q3 new[] array',
  [ 'int* a;', 'a = new int[4]{0,0,0,0};', 'a[2] = 20;', 'delete[] a;', 'a = nullptr;' ],
  (vm, i) => {
    if(i===0){ vm.allocStack('a','pointer','(indeterminate)'); vm.snapshot('Q3 • int* a;','Pointer for dynamic array.'); }
    if(i===1){ const base = vm.allocHeap('int[4]',[0,0,0,0],4); vm.setStack('a',base); vm.snapshot('Q3 • a = new int[4]{0,0,0,0};','a points to first element; block is contiguous.'); }
    if(i===2){ const base = vm.stack['a'].value; vm.findHeap(base).content[2] = 20; vm.snapshot('Q3 • a[2] = 20;','Indexing writes to the 3rd slot.'); }
    if(i===3){ const base = vm.stack['a'].value; vm.freeHeap(base); vm.snapshot('Q3 • delete[] a;','Array freed; a dangling.'); }
    if(i===4){ vm.setStack('a','nullptr'); vm.snapshot('Q3 • a = nullptr;','Now safe.'); }
  }, QA.Q3
);

// ----- Q4 -----
const Q4 = prog(
  'Q4','Q4 new Point / ->',
  [ 'Point* p;', 'p = new Point{1.0, 2.0};', '(*p).x; p->y;', 'delete p;', 'p = nullptr;' ],
  (vm, i) => {
    if(i===0){ vm.allocStack('p','pointer','(indeterminate)'); vm.snapshot('Q4 • Point* p;','Pointer to object.'); }
    if(i===1){ const addr = vm.allocHeap('Point',{x:1.0,y:2.0}); vm.setStack('p',addr); vm.snapshot('Q4 • p = new Point{1.0,2.0};','Heap object with x=1.0, y=2.0.'); }
    if(i===2){ vm.snapshot('Q4 • (*p).x; p->y;','Both syntaxes access the same object members.'); }
    if(i===3){ const addr = vm.stack['p'].value; vm.freeHeap(addr); vm.snapshot('Q4 • delete p;','Object freed; p dangling.'); }
    if(i===4){ vm.setStack('p','nullptr'); vm.snapshot('Q4 • p = nullptr;','Now safe.'); }
  }, QA.Q4
);

// ----- Q5A -----
const Q5A = prog(
  'Q5A','Q5A Ten points (independent)',
  [
    'Point* arrA[10];',
    ...Array.from({length:10},(_,i)=>`arrA[${i}] = new Point{1.0, 2.0};`),
    'arrA[0]->x = 100;',
    ...Array.from({length:10},(_,i)=>`delete arrA[${i}];`),
    ...Array.from({length:10},(_,i)=>`arrA[${i}] = nullptr;`),
  ],
  (vm, i) => {
    if(i===0){ vm.snapshot('Q5A • Point* arrA[10];','Ten pointers to independent Points.'); return; }
    if(i>=1 && i<=10){ const k = i-1; const addr = vm.allocHeap('Point',{x:1.0,y:2.0}); vm.allocStack(`arrA[${k}]`,'pointer',addr); vm.snapshot(`Q5A • arrA[${k}] = new Point{1.0,2.0};`,'Different addresses for each element.'); return; }
    if(i===11){ const a0 = vm.stack['arrA[0]'].value; vm.findHeap(a0).content.x = 100.0; vm.snapshot('Q5A • arrA[0]->x = 100;','Only arrA[0] changes.'); return; }
    if(i>=12 && i<=21){ const k=i-12; const addr = vm.stack[`arrA[${k}]`].value; vm.freeHeap(addr); vm.snapshot(`Q5A • delete arrA[${k}];`,'Freed one object.'); return; }
    if(i>=22 && i<=31){ const k=i-22; vm.setStack(`arrA[${k}]`,'nullptr'); vm.snapshot(`Q5A • arrA[${k}] = nullptr;`,'Pointer cleared.'); return; }
  }, QA.Q5A
);

// ----- Q5B -----
const Q5B = prog(
  'Q5B','Q5B Ten pointers (aliasing one object)',
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
  }, QA.Q5B
);

// ----- Q6 -----
const Q6 = prog(
  'Q6','Q6 delete & dangling (aliases)',
  [ 'Point* alias[3];', 'shared = new Point{3.0,3.0};', 'alias[0] = shared;', 'alias[1] = shared;', 'alias[2] = shared;', 'delete shared;', 'alias[0] = alias[1] = alias[2] = nullptr;' ],
  (vm, i) => {
    if(i===0){ vm.snapshot('Q6 • alias[3]','Three alias pointers.'); }
    if(i===1){ const a = vm.allocHeap('Point',{x:3.0,y:3.0}); vm.allocStack('shared','pointer',a); vm.snapshot('Q6 • shared = new Point{3.0,3.0};'); }
    if(i===2){ vm.allocStack('alias[0]','pointer', vm.stack['shared'].value); vm.snapshot('Q6 • alias[0] = shared;'); }
    if(i===3){ vm.allocStack('alias[1]','pointer', vm.stack['shared'].value); vm.snapshot('Q6 • alias[1] = shared;'); }
    if(i===4){ vm.allocStack('alias[2]','pointer', vm.stack['shared'].value); vm.snapshot('Q6 • alias[2] = shared;'); }
    if(i===5){ const addr = vm.stack['shared'].value; vm.freeHeap(addr); vm.snapshot('Q6 • delete shared;','Aliases are now dangling.'); }
    if(i===6){ vm.setStack('alias[0]','nullptr'); vm.setStack('alias[1]','nullptr'); vm.setStack('alias[2]','nullptr'); vm.snapshot('Q6 • null all aliases','Cleared.'); }
  }, QA.Q6
);

// ----- Q7 -----
const Q7 = prog(
  'Q7','Q7 pointer parameter (pass-by-value)',
  [ 'int* p = new int(80);', 'reassign_param(p); // q = new int(42);', 'set_through_pointer(p); // *p = 42;', 'delete p;', 'p = nullptr;' ],
  (vm, i) => {
    if(i===0){ const a = vm.allocHeap('int',80); vm.allocStack('p','pointer',a); vm.snapshot('Q7 • int* p = new int(80);'); }
    if(i===1){ /* reassign_param copies pointer and rebinds the copy */ vm.allocHeap('int',42); vm.snapshot('Q7 • reassign_param(p)','Parameter reassigned internally; caller p unchanged.'); }
    if(i===2){ const a = vm.stack['p'].value; vm.findHeap(a).content = 42; vm.snapshot('Q7 • set_through_pointer(p)','The object pointed by p is modified to 42.'); }
    if(i===3){ const a = vm.stack['p'].value; vm.freeHeap(a); vm.snapshot('Q7 • delete p','Freed; p dangling.'); }
    if(i===4){ vm.setStack('p','nullptr'); vm.snapshot('Q7 • p = nullptr','Safe.'); }
  }, QA.Q7
);

// ----- Q8 -----
const Q8 = prog(
  'Q8','Q8 int*& allocate (change caller pointer)',
  [ 'int* ages = nullptr;', 'allocate_int_array(ages, 5);', 'delete[] ages;', 'ages = nullptr;' ],
  (vm, i) => {
    if(i===0){ vm.allocStack('ages','pointer','nullptr'); vm.snapshot('Q8 • int* ages = nullptr;'); }
    if(i===1){ const arr = vm.allocHeap('int[n]',[0,0,0,0,0],5); vm.setStack('ages',arr); vm.snapshot('Q8 • allocate_int_array(ages,5)','Pointer-by-reference changes caller binding.'); }
    if(i===2){ const arr = vm.stack['ages'].value; vm.freeHeap(arr); vm.snapshot('Q8 • delete[] ages;','Freed; ages dangling.'); }
    if(i===3){ vm.setStack('ages','nullptr'); vm.snapshot('Q8 • ages = nullptr;','Safe.'); }
  }, QA.Q8
);

const PROGRAMS = [Q1,Q2,Q3,Q4,Q5A,Q5B,Q6,Q7,Q8];

// ---------- Hooks ----------
function useProgram(id){
  const program = useMemo(()=> PROGRAMS.find(p=>p.id===id) || PROGRAMS[0], [id]);
  const vmRef = useRef(new MemoryModel());
  const [idx, setIdx] = useState(-1); // -1 means before first line
  const reset = ()=>{ vmRef.current = new MemoryModel(); setIdx(-1); };
  const step = ()=>{ if(idx+1 < program.lines.length){ const vm = vmRef.current; const next = idx+1; program.exec(vm, next); setIdx(next); } };
  const back = ()=>{ if(idx>=0) setIdx(idx-1); };
  const snapshots = vmRef.current.snapshots;
  const currentSnap = (idx>=0 && idx < snapshots.length) ? snapshots[idx] : null;
  const atFinal = idx+1 === program.lines.length;
  return { program, idx, step, back, reset, currentSnap, atFinal };
}

// ---------- UI components ----------
function CodeView({lines, current}){
  return (
    <div className="rounded-3xl border-2 border-slate-200 bg-white shadow-lg overflow-hidden">
      <div className="px-4 py-3 text-sm font-bold text-slate-700 border-b-2 border-slate-200 bg-slate-50">Code (execute one line at a time)</div>
      <pre className="m-0 p-4 text-base leading-7 font-mono whitespace-pre-wrap">
        {lines.map((ln,i)=> (
          <div key={i} className={`flex gap-4 py-1 px-2 rounded-lg ${i===current? 'bg-yellow-100 border-2 border-yellow-300' : 'hover:bg-slate-50'}`}>
            <span className="w-10 text-right text-slate-500 select-none font-bold">{i+1}</span>
            <span className="flex-1 text-slate-800">{ln}</span>
          </div>
        ))}
      </pre>
    </div>
  );
}

function StackView({stack}){
  const items = Object.entries(stack).sort((a,b)=>b[1].addr - a[1].addr);
  return (
    <div className="space-y-3">
      {items.length===0 && <div className="text-base text-slate-500 text-center py-8 bg-slate-50 rounded-2xl border-2 border-dashed border-slate-300">(empty)</div>}
      {items.map(([name,v])=>{
        const addr = toHex(v.addr);
        const isPtr = v.type==='pointer';
        const dangling = v.meta?.dangling;
        return (
          <div key={name} className={`rounded-2xl p-4 border-2 shadow-md bg-white ${dangling? 'border-red-400 ring-2 ring-red-200 bg-red-50': 'border-slate-200 hover:border-slate-300'}`}>
            <div className="flex justify-between text-base font-mono mb-2">
              <span className="font-bold text-slate-800">{name}</span>
              <span className="text-slate-500 font-medium">{addr}</span>
            </div>
            <div className="text-base font-mono">
              {!isPtr && <span>value: <span className="font-bold text-slate-800">{String(v.value)}</span></span>}
              {isPtr && (
                <span>pointer → <span className={`font-bold ${dangling? 'text-red-600': 'text-blue-700'}`}>
                  {v.value==='nullptr' ? 'nullptr' : toHex(v.value)}
                </span>{dangling && <span className="ml-2 text-red-600 font-medium">(dangling)</span>}</span>
              )}
            </div>
            {v.meta?.warning && <div className="mt-2 text-sm text-amber-700 bg-amber-50 p-2 rounded-lg border border-amber-200">⚠︎ {v.meta.warning}</div>}
          </div>
        )
      })}
    </div>
  );
}

function HeapView({heap}){
  return (
    <div className="grid grid-cols-1 gap-3">
      {heap.length===0 && <div className="text-base text-slate-500 text-center py-8 bg-slate-50 rounded-2xl border-2 border-dashed border-slate-300">(no allocations)</div>}
      {heap.map(b=>{
        const addr = toHex(b.addr);
        const freed = b.freed;
        return (
          <div key={addr} className={`rounded-2xl p-4 border-2 shadow-md ${freed? 'bg-slate-100 border-slate-300 opacity-75' : 'bg-emerald-50 border-emerald-300'}`}>
            <div className="flex justify-between text-base font-mono mb-2">
              <span className="font-bold text-slate-800">{b.label}</span>
              <span className="text-slate-600 font-medium">{addr}</span>
            </div>
            <div className="mt-2 text-base font-mono">
              {!Array.isArray(b.content) && typeof b.content === 'object' ? (
                <div className="flex gap-3 flex-wrap">{Object.entries(b.content).map(([k,v])=> (
                  <div key={k} className="px-3 py-1 rounded-lg bg-white border-2 border-slate-200 text-slate-800 font-medium">{k}: {String(v)}</div>
                ))}</div>
              ) : Array.isArray(b.content) ? (
                <div className="flex flex-wrap gap-2">{b.content.map((v,i)=> (
                  <div key={i} className="px-3 py-1 rounded-lg bg-white border-2 border-slate-200 text-slate-800 font-medium">[{i}] {String(v)}</div>
                ))}</div>
              ) : (
                <div className="px-3 py-1 rounded-lg bg-white border-2 border-slate-200 inline-block text-slate-800 font-medium">{String(b.content)}</div>
              )}
            </div>
            {freed && <div className="mt-2 text-sm text-slate-600 font-medium bg-slate-200 px-3 py-1 rounded-lg inline-block">(freed)</div>}
          </div>
        )
      })}
    </div>
  );
}

function Controls({onBack, onStep, onReset, idx, max}){
  return (
    <div className="flex items-center gap-3">
      <button onClick={onBack} disabled={idx<0} className="px-4 py-2 rounded-xl border-2 border-slate-300 bg-white text-slate-700 font-medium hover:bg-slate-50 disabled:opacity-50 disabled:cursor-not-allowed transition-colors">← Back</button>
      <button onClick={onStep} disabled={idx+1>=max} className="px-4 py-2 rounded-xl bg-blue-600 text-white font-medium hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed transition-colors">Step →</button>
      <button onClick={onReset} className="px-4 py-2 rounded-xl border-2 border-slate-300 bg-white text-slate-700 font-medium hover:bg-slate-50 transition-colors">Reset</button>
      <div className="ml-3 text-base font-medium text-slate-700">Line <span className="font-bold text-blue-600">{Math.max(0,idx+1)}</span>/<span className="font-bold">{max}</span></div>
    </div>
  );
}

function MemoryLabVisualizer(){
  const [programId, setProgramId] = useState('Q2'); // default Q2
  const { program, idx, step, back, reset, currentSnap, atFinal } = useProgram(programId);

  // keyboard shortcuts
  useEffect(()=>{
    const onKey = (e)=>{
      if(e.key==='ArrowRight') step();
      if(e.key==='ArrowLeft') back();
      const order = ['Q1','Q2','Q3','Q4','Q5A','Q5B','Q6','Q7','Q8'];
      if(/^[1-9]$/.test(e.key)){
        const k = parseInt(e.key,10)-1; const id = order[k]; if(id){ setProgramId(id); reset(); }
      }
    };
    window.addEventListener('keydown', onKey);
    return ()=> window.removeEventListener('keydown', onKey);
  }, [step, back, reset]);

  const max = program.lines.length;
  const stack = currentSnap ? currentSnap.stack : {};
  const heap  = currentSnap ? currentSnap.heap  : [];

  return (
    <div className="min-h-full w-full p-6 bg-gradient-to-br from-slate-50 via-blue-50 to-indigo-50 text-slate-900">
      <div className="max-w-7xl mx-auto">
        <header className="mb-8 p-6 bg-white rounded-3xl shadow-lg border-2 border-slate-200">
          <h1 className="text-3xl font-bold text-slate-800 mb-2">CSIS3700 Memory Lab — Program Mode</h1>
          <p className="text-base text-slate-600">Execute one line at a time; see stack/heap side by side. Final step reveals the one‑sentence conclusion.</p>
        </header>

        <div className="flex flex-wrap items-center gap-4 mb-8 p-4 bg-white rounded-2xl shadow-md border-2 border-slate-200">
          <select value={programId} onChange={(e)=>{ setProgramId(e.target.value); reset(); }} className="px-4 py-2 rounded-xl border-2 border-slate-300 bg-white font-medium text-slate-700 focus:border-blue-500 focus:ring-2 focus:ring-blue-200">
            {PROGRAMS.map(p=> <option key={p.id} value={p.id}>{p.name}</option>)}
          </select>
          <Controls onBack={back} onStep={step} onReset={reset} idx={idx} max={max} />
          <div className="ml-auto text-sm text-slate-600 font-medium">Shortcuts: 1..9 to switch, ←/→ to step</div>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-8">
          <CodeView lines={program.lines} current={idx} />

          <section className="rounded-3xl bg-white shadow-lg border-2 border-slate-200 p-6">
            <div className="flex items-baseline justify-between mb-4 pb-2 border-b-2 border-slate-100">
              <h2 className="text-xl font-bold text-slate-800">Stack (simplified)</h2>
              <div className="text-sm text-slate-500 font-medium">highest → lowest addresses</div>
            </div>
            <StackView stack={stack} />
          </section>
        </div>

        <section className="mb-8 rounded-3xl bg-white shadow-lg border-2 border-slate-200 p-6">
          <div className="flex items-baseline justify-between mb-4 pb-2 border-b-2 border-slate-100">
            <h2 className="text-xl font-bold text-slate-800">Heap (allocations)</h2>
            <div className="text-sm text-slate-500 font-medium">green = live, gray = freed</div>
          </div>
          <HeapView heap={heap} />
        </section>

        <section className="mb-8 grid grid-cols-1 lg:grid-cols-2 gap-6">
          <div className="rounded-3xl bg-white border-2 border-slate-200 p-6 shadow-lg">
            <div className="text-base font-bold mb-3 text-slate-800 border-b border-slate-200 pb-2">Question</div>
            <div className="text-base text-slate-700 leading-relaxed">{program.question}</div>
          </div>
          <div className={`rounded-3xl border-2 p-6 shadow-lg ${atFinal? 'bg-emerald-50 border-emerald-300' : 'bg-slate-50 border-slate-200'}`}>
            <div className="text-base font-bold mb-3 text-slate-800 border-b border-slate-200 pb-2">One‑sentence conclusion {atFinal? '' : '(revealed at final step)'}</div>
            <div className={`text-base leading-relaxed ${atFinal? 'text-emerald-900' : 'text-slate-500'}`}>{atFinal? program.conclusion : '—'}</div>
          </div>
        </section>

        {currentSnap && (
          <section className="mb-8 rounded-3xl bg-white border-2 border-slate-200 p-6 shadow-lg">
            <div className="text-base font-bold mb-3 text-slate-800 border-b border-slate-200 pb-2">{currentSnap.title}</div>
            <div className="text-base text-slate-700 leading-relaxed">{currentSnap.desc}</div>
          </section>
        )}
      </div>
    </div>
  );
}

// Render the component
ReactDOM.render(<MemoryLabVisualizer />, document.getElementById('memory-lab-container'));
</script>
