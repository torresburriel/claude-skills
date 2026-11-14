---
name: reader-annotations
description: Turns a text or document into an HTML page you can annotate, with yellow highlight, an important highlight and notes, in a Google Reader look, saved locally. Use when asked to annotate, highlight or take notes on a text.
---

# Highlighting and annotation system for HTML pages, Google Reader skin

A self-contained module (CSS, HTML and JavaScript, no dependencies) that lets the reader select text, highlight it, attach a note and review everything in a list. It is stored in localStorage. It works for any text or document worth reading slowly: articles, reports, converted PDFs, notes.

The skin is the Google Reader of 2011 to 2013, the same visual family as reader.torresburriel.es. There are two kinds of mark, highlight and important, plus notes.

## When to use it

- The user asks to highlight, underline, annotate or comment on a text or document: an article, a report, a PDF whose text is extracted, an HTML page. It is the default option for annotating.
- It does not provide annotations shared between people or synced across devices: it is personal and local. If that is requested, say so and suggest another route.

## What it does

- Selecting text shows a floating bar with three actions: "Highlight" (yellow), "Important" (a higher-priority highlight) and "+ Note" (highlights in yellow and opens the note).
- Tapping a mark opens an editor: change type, write a note, delete.
- A list gathers every annotation, with filters by type and a "Go to text" button.
- A free-form notepad, optional.
- A "Copy and transfer" dialog: exports the data as text to copy and paste on another device, imports it, shows it as readable text and deletes everything with a double press.
- A status indicator: number of annotations and whether the browser allows saving.

## Interface language

The interface strings are in English by default. When the document is in another language, translate the labels into the document's language: the buttons in the HTML and the strings in the JavaScript (status, list, dialog messages and `CFG.types`). Do not translate the internal keys (`highlight`, `important`), class names or ids.

## The Reader skin: criteria

- Arial type, 13 px in the interface, 11 px bold for buttons.
- White background, #222 text, links in blue #1155CC, thin #E5E5E5 rules. No black blocks or thick borders.
- Flat grey buttons (#F5F5F5, #DCDCDC border, 2 px radius), like the ones in the Reader toolbar. The primary action is blue and the destructive one is red text on grey.
- Floating menus and editor in white, with a 1 px grey border and a short shadow, like Google's dropdowns of that era.
- The annotation list imitates Reader's list view: rows separated by a rule, no cards, the quote in link blue and the metadata in small grey.
- The dialog uses a translucent white veil, not a black one.
- Marks: highlight in yellow and important in orange with a red rule underneath. Important is told apart by shape, not only by colour. There are no other types: if another is requested, add it following the pattern of these two.
- Two deliberate departures from the original, for contrast: the primary button blue is #1155CC instead of #4D90FE, and the danger red is #C5221F instead of #D14836. With the originals, 11 px text does not reach 4.5:1. #4D90FE is kept for focus.
- On touch screens the buttons grow to 40 px high. The original Reader was not designed for fingers.

If the user provides tokens from their own design system, those win: replace the values of the --an-* variables and touch nothing else.

## Host page requirements

1. A root container with the annotatable content (main by default).
2. Inside it, section elements with a stable id and a data-title attribute holding the name that will appear in the list.
3. Text lives in leaf blocks: p, li, td, or whatever is added to CFG.blockSel. A block that contains another block does not count; the inner one does.
4. Anything that should not be annotated carries data-noannot (the list itself, sources, captions).
5. Containers with a dark background carry data-an-dark so the highlight stays legible. A page with the Reader skin should not have any, but the rule stays.

## Piece 0, optional: Reader base for the host page

Only when the page is built from scratch and the whole thing should share the same look. If the page already has a design system, skip it.

```css
body{margin:0;background:#FFFFFF;color:#222222;font:13px/1.5 Arial,Helvetica,sans-serif}
a{color:#1155CC;text-decoration:none}
a:hover{text-decoration:underline}
main{max-width:650px;padding:16px 20px}
main p,main li,main td{font-size:14px;line-height:1.6}
h1{font-size:20px;font-weight:400;margin:0 0 12px}
h2{font-size:16px;font-weight:700;margin:20px 0 8px}
h2 a,h1 a{color:#1155CC}
header.rd-bar{position:sticky;top:0;z-index:30;display:flex;gap:12px;align-items:center;flex-wrap:wrap;padding:8px 20px;background:#F5F5F5;border-bottom:1px solid #E5E5E5;font-size:11px;color:#666666}
nav.rd-side{background:#FFFFFF;border-right:1px solid #E5E5E5;font-size:13px}
nav.rd-side a{display:block;padding:4px 16px;color:#222222}
nav.rd-side a.sel{font-weight:700;color:#DD4B39}
section{border-bottom:1px solid #E5E5E5;padding:12px 0}
```

The 650 px reading width is that of Reader entries. The status and the "Copy and transfer" button go in header.rd-bar.

## Piece 1: CSS

It goes inside the page's single style block.

```css
:root{--an-ink:#222222;--an-paper:#FFFFFF;--an-accent:#4D90FE;--an-primary:#1155CC;--an-danger:#C5221F;--an-rule:#E5E5E5;--an-border:#DCDCDC;--an-soft:#F5F5F5;--an-muted:#666666;--an-highlight:#FFF3A8;--an-important:#FFC773;--an-important-rule:#DD4B39;--an-font:Arial,Helvetica,sans-serif}
mark{background:none;color:inherit;cursor:pointer}
mark.h-highlight{background:var(--an-highlight);color:var(--an-ink)}
mark.h-important{background:var(--an-important);color:var(--an-ink);box-shadow:inset 0 -2px 0 var(--an-important-rule)}
mark.has-note{outline:1px dotted var(--an-muted);outline-offset:1px}
mark.flash{animation:anfl 1.2s}
@keyframes anfl{0%,60%{outline:2px solid var(--an-accent)}100%{outline:2px solid transparent}}
[data-an-dark] mark.h-highlight,[data-an-dark] mark.h-important{color:var(--an-ink)}
[data-an-dark] mark.has-note{outline-color:var(--an-paper)}
.selbar,.pop{position:absolute;z-index:40;background:var(--an-paper);color:var(--an-ink);border:1px solid #CCCCCC;box-shadow:0 2px 4px rgba(0,0,0,.2);padding:6px;display:none;gap:6px;align-items:center;flex-wrap:wrap;font:13px/1.4 var(--an-font)}
.selbar.on{display:flex}
.pop{flex-direction:column;align-items:stretch;width:290px;padding:10px}
.pop.on{display:flex}
.pop label{color:var(--an-muted);font-size:11px;font-weight:700;margin-top:8px}
.pop__row{display:flex;gap:6px;flex-wrap:wrap}
.selbar button,.pop button,.an-modal button,.an-filters button,.annlist button,#btnData{font:700 11px/25px var(--an-font);cursor:pointer;border:1px solid var(--an-border);border-radius:2px;background:var(--an-soft);color:#444444;padding:0 8px;min-height:27px}
.selbar button:hover,.pop button:hover,.an-modal button:hover,.an-filters button:hover,.annlist button:hover,#btnData:hover{border-color:#C6C6C6;color:var(--an-ink);box-shadow:0 1px 1px rgba(0,0,0,.1)}
.selbar button:focus-visible,.pop button:focus-visible,.an-modal button:focus-visible,.an-filters button:focus-visible,.annlist button:focus-visible,#btnData:focus-visible{outline:2px solid var(--an-accent);outline-offset:1px}
.an-primary{background:var(--an-primary)!important;border-color:var(--an-primary)!important;color:var(--an-paper)!important}
.pop textarea,.an-modal textarea,#pad{width:100%;box-sizing:border-box;font:13px/1.45 var(--an-font);padding:6px 8px;border:1px solid #D9D9D9;border-top-color:#C0C0C0;background:var(--an-paper);color:var(--an-ink);resize:vertical}
.pop textarea:focus,.an-modal textarea:focus,#pad:focus{outline:none;border-color:var(--an-accent);box-shadow:inset 0 1px 2px rgba(0,0,0,.2)}
.chip--highlight{background:var(--an-highlight)!important;color:var(--an-ink)!important}
.chip--important{background:var(--an-important)!important;color:var(--an-ink)!important;box-shadow:inset 0 -3px 0 var(--an-important-rule)!important}
.chip.sel{outline:2px solid var(--an-accent);outline-offset:1px}
.annlist{list-style:none;padding:0;margin:8px 0;border-top:1px solid var(--an-rule);font:13px/1.45 var(--an-font)}
.annlist li{border-bottom:1px solid var(--an-rule);border-left:3px solid transparent;background:var(--an-paper);padding:8px 10px}
.annlist li:hover{background:#FAFAFA}
.annlist li.t-highlight{border-left-color:#E8C400}.annlist li.t-important{border-left-color:var(--an-important-rule)}
.annlist .quote{color:var(--an-primary);font-weight:700}
.annlist .n{margin-top:4px;color:var(--an-ink)}
.annlist .meta{font-size:11px;color:var(--an-muted);margin-top:6px;display:flex;gap:10px;flex-wrap:wrap;align-items:center}
.an-filters{display:flex;flex-wrap:wrap;gap:6px;margin:8px 0}
.an-filters button.sel{background:#EEEEEE;border-color:#CCCCCC;color:var(--an-ink);box-shadow:inset 0 1px 2px rgba(0,0,0,.1)}
#status{font:11px var(--an-font);color:var(--an-muted)}
.an-modal{position:fixed;inset:0;z-index:60;background:rgba(255,255,255,.75);display:none;align-items:center;justify-content:center;padding:16px;font:13px/1.5 var(--an-font)}
.an-modal.on{display:flex}
.an-modal__box{background:var(--an-paper);color:var(--an-ink);border:1px solid #ACACAC;box-shadow:0 4px 16px rgba(0,0,0,.2);padding:24px;width:min(680px,100%);max-height:90vh;overflow:auto;box-sizing:border-box}
.an-modal__box h2{font-size:16px;font-weight:400;margin:0 0 12px}
.an-modal__row{display:flex;gap:8px;flex-wrap:wrap;margin:10px 0}
.an-danger{color:var(--an-danger)!important}
.an-msg{font-size:11px;color:var(--an-muted);min-height:1.2em}
@media (pointer:coarse){.selbar button,.pop button,.an-modal button,.an-filters button,.annlist button,#btnData{min-height:40px;line-height:38px;padding:0 12px;font-size:13px}}
@media print{.selbar,.pop,.an-modal,.an-filters{display:none!important}mark{-webkit-print-color-adjust:exact;print-color-adjust:exact}}
```

## Piece 2: HTML

Two elements go wherever suits the page: the status with the transfer button (usually in the top bar) and the list panel with the notepad. The rest goes right before the script, outside the root container.

```html
<!-- in a bar or header -->
<span id="status"></span>
<button type="button" id="btnData">Copy and transfer</button>

<!-- review panel: inside a section with data-noannot -->
<div class="an-filters" id="filters">
  <button type="button" data-f="all" class="sel">All</button>
  <button type="button" data-f="highlight">Highlighted</button>
  <button type="button" data-f="important">Important</button>
  <button type="button" data-f="note">With note</button>
</div>
<ul class="annlist" id="annlist"></ul>
<textarea id="pad" rows="14" aria-label="Notepad"></textarea>

<!-- before the script -->
<div class="selbar" id="selbar" role="toolbar" aria-label="Highlight selection">
  <button type="button" class="chip chip--highlight" data-t="highlight">Highlight</button>
  <button type="button" class="chip chip--important" data-t="important">Important</button>
  <button type="button" data-t="highlight" data-note="1">+ Note</button>
</div>
<div class="pop" id="pop" role="dialog" aria-label="Edit highlight">
  <div class="pop__row" id="popTypes">
    <button type="button" class="chip chip--highlight" data-t="highlight">Highlighted</button>
    <button type="button" class="chip chip--important" data-t="important">Important</button>
  </div>
  <label for="popNote">Note</label>
  <textarea id="popNote" rows="4" placeholder="Write your note"></textarea>
  <div class="pop__row" style="margin-top:8px;justify-content:space-between">
    <button type="button" id="popDel" class="an-danger">Delete</button>
    <button type="button" id="popClose" class="an-primary">Done</button>
  </div>
</div>
<div class="an-modal" id="modal" role="dialog" aria-modal="true" aria-labelledby="modalT">
  <div class="an-modal__box">
    <h2 id="modalT">Copy and transfer your annotations</h2>
    <p>Your annotations live in this browser's local storage. To take them to another device, copy the text below and paste it right here on the other one.</p>
    <textarea id="dataArea" rows="9" aria-label="Annotation data"></textarea>
    <div class="an-modal__row">
      <button type="button" id="mCopy" class="an-primary">Copy data</button>
      <button type="button" id="mText">View as readable text</button>
      <button type="button" id="mImport">Import pasted data</button>
    </div>
    <p class="an-msg" id="mMsg"></p>
    <div class="an-modal__row" style="justify-content:space-between">
      <button type="button" id="mWipe" class="an-danger">Delete all</button>
      <button type="button" id="mClose">Close</button>
    </div>
  </div>
</div>
```

## Piece 3: JavaScript

It goes at the end of the body. Only CFG needs changing.

```html
<script>
(function(){
  'use strict';
  var CFG={
    key:'CHANGE-unique-key-v1',
    root:'main',
    blockSel:'p, li, td',
    types:{highlight:'Highlighted',important:'Important'},
    title:'Annotations'
  };
  var KEY=CFG.key, SEL=CFG.blockSel, TYPES=CFG.types, NL=String.fromCharCode(10);
  var state={hl:[],pad:''}, storageOK=true, filter='all', current=null, wipeArmed=false;
  var $=function(id){return document.getElementById(id)};
  var each=function(list,fn){Array.prototype.forEach.call(list,fn)};
  var main=document.querySelector(CFG.root), selbar=$('selbar'), pop=$('pop'), modal=$('modal'), pad=$('pad');

  function load(){
    try{
      localStorage.setItem(KEY+'-t','1'); localStorage.removeItem(KEY+'-t');
      var s=localStorage.getItem(KEY);
      if(s){var d=JSON.parse(s); if(d&&Array.isArray(d.hl)){state.hl=d.hl; state.pad=d.pad||'';}}
    }catch(e){storageOK=false}
  }
  function save(){
    try{localStorage.setItem(KEY,JSON.stringify(state))}catch(e){storageOK=false}
    status();
  }
  function status(){
    var el=$('status'); if(!el) return;
    var n=state.hl.length;
    el.textContent=(n===1?'1 annotation':n+' annotations')+' · '+(storageOK?'saved on this device':'this browser does not allow saving');
  }

  var blocks={};
  each(main.querySelectorAll('section[id]'),function(sec){
    if(sec.hasAttribute('data-noannot')) return;
    var i=0;
    each(sec.querySelectorAll(SEL),function(el){
      if(el.querySelector(SEL)||el.closest('[data-noannot]')) return;
      var id=sec.id+':'+(i++);
      el.setAttribute('data-b',id);
      blocks[id]={el:el,html:el.innerHTML,title:sec.getAttribute('data-title')||sec.id};
    });
  });

  function textNodes(el){
    var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT), out=[], pos=0, n;
    while((n=w.nextNode())){out.push([n,pos]); pos+=n.nodeValue.length;}
    return out;
  }
  function wrap(el,h){
    textNodes(el).forEach(function(p){
      var node=p[0], ns=p[1], ne=ns+node.nodeValue.length;
      var a=Math.max(h.s,ns), b=Math.min(h.e,ne);
      if(b<=a) return;
      var r=document.createRange(); r.setStart(node,a-ns); r.setEnd(node,b-ns);
      var m=document.createElement('mark');
      m.className='h-'+h.t+(h.note?' has-note':'');
      m.setAttribute('data-h',h.id);
      if(h.note) m.title=h.note;
      r.surroundContents(m);
    });
  }
  function anchor(h){
    var b=blocks[h.b]; if(!b) return false;
    var tmp=document.createElement('div'); tmp.innerHTML=b.html;
    var txt=tmp.textContent;
    if(txt.slice(h.s,h.e)===h.text) return true;
    var i=txt.indexOf(h.text);
    if(i<0) return false;
    h.s=i; h.e=i+h.text.length; return true;
  }
  function renderBlock(id){
    var b=blocks[id]; if(!b) return;
    b.el.innerHTML=b.html;
    state.hl.filter(function(h){return h.b===id}).sort(function(x,y){return x.s-y.s}).forEach(function(h){
      h.orphan=!anchor(h);
      if(!h.orphan) wrap(b.el,h);
    });
  }
  function renderAll(){Object.keys(blocks).forEach(renderBlock); state.hl.forEach(function(h){if(!blocks[h.b]) h.orphan=true;});}

  function offsetIn(el,node,o){var r=document.createRange(); r.selectNodeContents(el); r.setEnd(node,o); return r.toString().length;}
  function isSpace(c){return c.trim()===''}
  function selectionRange(){
    var s=window.getSelection();
    if(!s||s.rangeCount===0||s.isCollapsed) return null;
    var r=s.getRangeAt(0);
    if(!r.intersectsNode(main)) return null;
    if(!String(s).trim()) return null;
    return r;
  }
  function blocksIn(r){return Object.keys(blocks).filter(function(id){return r.intersectsNode(blocks[id].el)});}
  function create(type,withNote){
    var r=selectionRange(); if(!r) return;
    var made=[];
    blocksIn(r).forEach(function(id){
      var el=blocks[id].el, full=el.textContent;
      var s=el.contains(r.startContainer)?offsetIn(el,r.startContainer,r.startOffset):0;
      var e=el.contains(r.endContainer)?offsetIn(el,r.endContainer,r.endOffset):full.length;
      while(s<e&&isSpace(full.charAt(s))) s++;
      while(e>s&&isSpace(full.charAt(e-1))) e--;
      if(e<=s) return;
      var h={id:'h'+Date.now().toString(36)+Math.random().toString(36).slice(2,6),b:id,s:s,e:e,t:type,text:full.slice(s,e),note:'',ts:Date.now()};
      state.hl.push(h); made.push(h);
    });
    window.getSelection().removeAllRanges();
    hide(selbar);
    made.forEach(function(h){renderBlock(h.b)});
    save(); list();
    if(withNote&&made.length){var m=main.querySelector('mark[data-h="'+made[0].id+'"]'); if(m) openPop(made[0].id,m,true);}
  }

  function place(box,rect){
    box.classList.add('on');
    var w=box.offsetWidth, sx=window.pageXOffset, sy=window.pageYOffset;
    var x=rect.left+sx+rect.width/2-w/2;
    x=Math.max(sx+8,Math.min(x,sx+document.documentElement.clientWidth-w-8));
    box.style.left=x+'px'; box.style.top=(rect.bottom+sy+10)+'px';
  }
  function hide(box){box.classList.remove('on')}
  function inUI(t){return selbar.contains(t)||pop.contains(t)||modal.contains(t)}
  function onSelect(){
    setTimeout(function(){
      var r=selectionRange();
      if(!r||!blocksIn(r).length){hide(selbar);return;}
      var rects=r.getClientRects(); if(!rects.length){hide(selbar);return;}
      place(selbar,rects[rects.length-1]);
    },10);
  }
  document.addEventListener('mouseup',function(e){if(!inUI(e.target)) onSelect();});
  document.addEventListener('touchend',function(e){if(!inUI(e.target)) setTimeout(onSelect,250);});
  document.addEventListener('keyup',function(e){if(e.shiftKey||e.key==='Shift') onSelect(); if(e.key==='Escape'){hide(pop);hide(selbar);closeModal();}});
  document.addEventListener('selectionchange',function(){var s=window.getSelection(); if(!s||s.isCollapsed) hide(selbar);});
  selbar.addEventListener('mousedown',function(e){e.preventDefault()});
  selbar.addEventListener('click',function(e){var b=e.target.closest('button'); if(b) create(b.getAttribute('data-t'),!!b.getAttribute('data-note'));});

  function find(id){for(var i=0;i<state.hl.length;i++) if(state.hl[i].id===id) return state.hl[i]; return null;}
  function openPop(id,markEl,focus){
    var h=find(id); if(!h) return;
    current=id;
    $('popNote').value=h.note||'';
    each($('popTypes').children,function(b){b.classList.toggle('sel',b.getAttribute('data-t')===h.t)});
    place(pop,markEl.getBoundingClientRect());
    if(focus) $('popNote').focus();
  }
  main.addEventListener('click',function(e){
    var m=e.target.closest('mark[data-h]');
    if(!m){hide(pop);return;}
    var s=window.getSelection(); if(s&&!s.isCollapsed) return;
    openPop(m.getAttribute('data-h'),m,false);
  });
  $('popTypes').addEventListener('click',function(e){
    var b=e.target.closest('button'), h=find(current); if(!b||!h) return;
    h.t=b.getAttribute('data-t'); renderBlock(h.b); save(); list();
    each($('popTypes').children,function(x){x.classList.toggle('sel',x===b)});
  });
  $('popNote').addEventListener('input',function(){
    var h=find(current); if(!h) return;
    h.note=this.value;
    each(main.querySelectorAll('mark[data-h="'+h.id+'"]'),function(m){m.classList.toggle('has-note',!!h.note); m.title=h.note;});
    save(); list();
  });
  $('popDel').addEventListener('click',function(){
    var h=find(current); if(!h) return;
    state.hl=state.hl.filter(function(x){return x.id!==h.id});
    hide(pop); renderBlock(h.b); save(); list();
  });
  $('popClose').addEventListener('click',function(){hide(pop)});

  function esc(s){return String(s).replace(/[&<>"]/g,function(c){return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]})}
  function sorted(){var order=Object.keys(blocks); return state.hl.slice().sort(function(a,b){return (order.indexOf(a.b)-order.indexOf(b.b))||(a.s-b.s)});}
  function list(){
    var ul=$('annlist'); if(!ul) return;
    var items=sorted().filter(function(h){return filter==='all'||(filter==='note'?!!h.note:h.t===filter)});
    if(!items.length){ul.innerHTML='<li>'+(state.hl.length?'No annotations of this type.':'Nothing highlighted yet. Select some text to start.')+'</li>';return;}
    ul.innerHTML=items.map(function(h){
      var b=blocks[h.b], q=h.text.length>180?h.text.slice(0,180)+'…':h.text;
      return '<li class="t-'+h.t+'"><div class="quote">“'+esc(q)+'”</div>'+(h.note?'<div class="n">'+esc(h.note).split(NL).join('<br>')+'</div>':'')+
        '<div class="meta"><span>'+TYPES[h.t]+' · '+esc(b?b.title:'section removed')+(h.orphan?' · not found in the current text':'')+'</span>'+
        (h.orphan?'':'<button type="button" data-go="'+h.id+'">Go to text</button>')+'<button type="button" data-del="'+h.id+'" class="an-danger">Delete</button></div></li>';
    }).join('');
  }
  if($('annlist')) $('annlist').addEventListener('click',function(e){
    var go=e.target.getAttribute('data-go'), del=e.target.getAttribute('data-del');
    if(go){var m=main.querySelector('mark[data-h="'+go+'"]'); if(m){m.scrollIntoView({block:'center'}); m.classList.add('flash'); setTimeout(function(){m.classList.remove('flash')},1300);}}
    if(del){var h=find(del); state.hl=state.hl.filter(function(x){return x.id!==del}); if(h) renderBlock(h.b); save(); list();}
  });
  if($('filters')) $('filters').addEventListener('click',function(e){
    var b=e.target.closest('button'); if(!b) return;
    filter=b.getAttribute('data-f');
    each(this.children,function(x){x.classList.toggle('sel',x===b)});
    list();
  });

  var padT;
  if(pad) pad.addEventListener('input',function(){state.pad=pad.value; clearTimeout(padT); padT=setTimeout(save,300);});

  function asText(){
    var out=[CFG.title,''];
    sorted().forEach(function(h){
      out.push('['+TYPES[h.t]+'] '+(blocks[h.b]?blocks[h.b].title:'')+': “'+h.text+'”'+(h.note?NL+'   Note: '+h.note:''));
    });
    if(state.pad){out.push('','Notepad:',state.pad);}
    return out.join(NL);
  }
  function openModal(){
    $('dataArea').value=JSON.stringify({v:1,hl:state.hl.map(function(h){return {id:h.id,b:h.b,s:h.s,e:h.e,t:h.t,text:h.text,note:h.note,ts:h.ts}}),pad:state.pad});
    $('mMsg').textContent=''; wipeArmed=false; $('mWipe').textContent='Delete all'; modal.classList.add('on');
  }
  function closeModal(){modal.classList.remove('on')}
  if($('btnData')) $('btnData').addEventListener('click',openModal);
  $('mClose').addEventListener('click',closeModal);
  modal.addEventListener('click',function(e){if(e.target===modal) closeModal()});
  $('mCopy').addEventListener('click',function(){
    var ta=$('dataArea'); ta.focus(); ta.select();
    var done=function(ok){$('mMsg').textContent=ok?'Copied to clipboard.':'Could not copy. The text is selected: copy it manually.'};
    try{navigator.clipboard.writeText(ta.value).then(function(){done(true)},function(){done(false)})}catch(e){done(false)}
  });
  $('mText').addEventListener('click',function(){$('dataArea').value=asText(); $('mMsg').textContent='Text view. This view cannot be imported.';});
  $('mImport').addEventListener('click',function(){
    try{
      var d=JSON.parse($('dataArea').value);
      if(!d||!Array.isArray(d.hl)) throw 0;
      var ok=d.hl.filter(function(h){return h&&typeof h.b==='string'&&typeof h.s==='number'&&typeof h.e==='number'&&TYPES[h.t]&&typeof h.text==='string'});
      var have={}; state.hl.forEach(function(h){have[h.id]=1});
      var add=ok.filter(function(h){return !have[h.id]}).map(function(h){return {id:String(h.id),b:h.b,s:h.s,e:h.e,t:h.t,text:h.text,note:String(h.note||''),ts:h.ts||Date.now()}});
      state.hl=state.hl.concat(add);
      if(typeof d.pad==='string'&&d.pad&&!state.pad){state.pad=d.pad; if(pad) pad.value=d.pad;}
      renderAll(); save(); list();
      $('mMsg').textContent='Imported '+add.length+' new annotations.';
    }catch(e){$('mMsg').textContent='The pasted text is not valid annotation data.';}
  });
  $('mWipe').addEventListener('click',function(){
    if(!wipeArmed){wipeArmed=true; this.textContent='Press again to confirm'; return;}
    state={hl:[],pad:''}; if(pad) pad.value=''; renderAll(); save(); list(); closeModal();
  });

  load(); if(pad) pad.value=state.pad; renderAll(); list(); status();
})();
</script>
```

## What to adapt in each project

- CFG.key: a unique key per document (for example, topic-document-v1). Two pages with the same key on the same origin would share annotations.
- CFG.root and CFG.blockSel: the container and the block types. If the page uses other text blocks that are not p, li or td, add their classes to the selector.
- CFG.types and the HTML buttons: the labels of the two types. If they are renamed, change them in both places and in the filters. The internal keys (highlight, important) are tied to the CSS classes; changing them means changing the CSS.
- --an-* variables: only touch them if the user provides their own tokens or asks for a different shade. The skin is the whole point of this skill.
- Status and transfer button: in the top bar if the page is long. The notepad, the status and the button are optional; the script ignores them if they do not exist.

## How it works inside

- Each leaf block gets a stable identifier: the section id plus its order number within it. Editing one section does not shift the annotations of the others.
- A highlight is stored as block, start and end position within the block's text, type, quoted text and note. No HTML is stored.
- When painting, the block is restored to its original HTML and the ranges are wrapped in mark elements. A highlight that crosses bold text or links is split into several marks with the same identifier.
- A selection that spans several blocks creates one highlight per block.
- If the block's text has changed, the quote is searched for inside the block and re-anchored. If it is not found, the annotation is not lost: it stays in the list, flagged as not found.

## Limits to tell the user about

- Annotations live in one specific browser and origin. They do not move on their own to another device or browser.
- A published artifact and the same HTML downloaded are separate stores.
- Inside the artifact viewer the page cannot start file downloads: that is why transfer works by copy and paste.
- Clearing browsing data deletes the annotations.
- In previews inside the chat, storage may not persist; the status indicator warns when the browser does not allow saving.
- Do not use alert, confirm or prompt: dialogs live inside the page.

## Verification before delivering

Test with Playwright and Chromium, serving the file over local HTTP, not just opening it:

1. Programmatically select a range that crosses bold text, dispatch mouseup and check that the bar appears; create the highlight.
2. Create another with "+ Note", write the note and close.
3. Create one that spans two blocks and, if the page has a dark container, another inside it.
4. Reload and check that the annotation count, the marks and the notepad are still there.
5. Change a mark from highlight to important in its editor and check that the two are distinguishable at a glance.
6. Export, delete everything with a double press, import and check that everything comes back.
7. No console errors and no horizontal overflow at 390 px.
8. Take a screenshot and review the skin: Arial, grey buttons with a 2 px radius, white floating panels with a short shadow, list in ruled rows, white veil on the dialog. Check the contrast of button text (primary and danger) and of the grey metadata.

Tested in desktop Chromium, in mobile emulation at 390 px, and on a real iPhone and iPad. The verification above is mandatory for every new page.

## Author

Skill created by Daniel Torres Burriel (Torresburriel Estudio, torresburriel.com). The look takes his reader, reader.torresburriel.es, as its reference. GPL-3.0 licence. Code and example at github.com/torresburriel/claude-skills.
