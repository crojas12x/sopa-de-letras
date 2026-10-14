# sopa-de-letras
<!DOCTYPE html>
<html lang="es"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Sopa de letras Mecatrónica</title>
<style>
:root{--bg:#f4f9f6;--card:#fff;--tx:#12332b;--g:#1b7a4e;--y:#f5b800;--b:#d5e6dc;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#0e1f1a;--card:#17302a;--tx:#e8f3ee;--g:#3fcf8e;--y:#f5c542;--b:#2a4a40}}
*{box-sizing:border-box}body{margin:0 auto;background:var(--bg);color:var(--tx);font-family:system-ui,Arial,sans-serif;padding:14px;max-width:560px}
h1{font-size:21px;margin:4px 0}small{opacity:.75}
#grid{display:grid;grid-template-columns:repeat(14,1fr);gap:2px;margin:12px 0;user-select:none;-webkit-user-select:none}
.c{aspect-ratio:1;display:flex;align-items:center;justify-content:center;background:var(--card);border:1px solid var(--b);border-radius:4px;font-weight:700;font-size:clamp(11px,3.6vw,18px);cursor:pointer}
.c.s{background:var(--y);color:#222}.c.f{background:var(--g);color:#fff}
#words{display:flex;flex-wrap:wrap;gap:6px}
.w{background:var(--card);border:1px solid var(--b);border-radius:10px;padding:4px 9px;font-size:13px;font-weight:600}
.w.f{text-decoration:line-through;opacity:.5}
.top{display:flex;justify-content:space-between;font-weight:700;align-items:center}
button{font:inherit;padding:8px 14px;border-radius:10px;border:0;background:var(--g);color:#fff;font-weight:700;cursor:pointer}
#msg{margin:8px 0;font-weight:700;min-height:22px}
</style></head><body>
<h1>🔎 Sopa de letras Mecatrónica</h1>
<small>Toca la primera y la última letra de cada palabra (horizontal, vertical o diagonal)</small>
<div class="top" style="margin-top:10px"><span id="cnt"></span><button onclick="init()">Nueva sopa</button></div>
<div id="msg"></div>
<div id="grid"></div>
<div id="words"></div>
<script>
const WL=["MECATRONICA","EMPLEABILIDAD","PORTAFOLIOS","DIAGNOSTICO","AUTOMATIZACION","DIGITALIZACION","COMPETENCIAS","SABANA","CHIA","TALLERES","MENTORIAS","EGRESADOS"];
const N=14,D=[[0,1],[1,0],[1,1],[-1,1]];
let G,found,sel;
function build(){
 for(let t=0;t<50;t++){
  const g=Array.from({length:N},()=>Array(N).fill(''));let ok=true;
  for(const w of [...WL].sort((a,b)=>b.length-a.length)){
   let placed=false;
   for(let k=0;k<300&&!placed;k++){
    const [dr,dc]=D[Math.random()*4|0],r=Math.random()*N|0,c=Math.random()*N|0;
    const er=r+dr*(w.length-1),ec=c+dc*(w.length-1);
    if(er<0||er>=N||ec<0||ec>=N)continue;
    let f=true;for(let i=0;i<w.length;i++){const x=g[r+dr*i][c+dc*i];if(x&&x!=w[i]){f=false;break}}
    if(!f)continue;
    for(let i=0;i<w.length;i++)g[r+dr*i][c+dc*i]=w[i];placed=true}
   if(!placed){ok=false;break}}
  if(ok){const A="ABCDEFGHIJKLMNOPQRSTUVWXYZ";
   for(let r=0;r<N;r++)for(let c=0;c<N;c++)if(!g[r][c])g[r][c]=A[Math.random()*26|0];
   return g}}
}
function init(){G=build();found=new Set();sel=null;
 const gr=document.getElementById('grid');gr.innerHTML='';
 for(let r=0;r<N;r++)for(let c=0;c<N;c++){const d=document.createElement('div');d.className='c';d.id='c'+r+'_'+c;d.textContent=G[r][c];d.onclick=()=>tap(r,c);gr.appendChild(d)}
 document.getElementById('words').innerHTML=WL.map(w=>`<span class="w" id="w${w}">${w}</span>`).join('');
 document.getElementById('msg').textContent='';upd()}
function el(r,c){return document.getElementById('c'+r+'_'+c)}
function upd(){document.getElementById('cnt').textContent=`Encontradas: ${found.size}/${WL.length}`}
function tap(r,c){
 if(!sel){sel=[r,c];el(r,c).classList.add('s');return}
 const [r0,c0]=sel;el(r0,c0).classList.remove('s');sel=null;
 const dr=r-r0,dc=c-c0,L=Math.max(Math.abs(dr),Math.abs(dc));
 if(L==0||!(dr==0||dc==0||Math.abs(dr)==Math.abs(dc)))return;
 const sr=Math.sign(dr),sc=Math.sign(dc);let s='',cells=[];
 for(let i=0;i<=L;i++){s+=G[r0+sr*i][c0+sc*i];cells.push([r0+sr*i,c0+sc*i])}
 const rev=[...s].reverse().join('');
 const w=WL.find(x=>x==s||x==rev);
 if(w&&!found.has(w)){found.add(w);cells.forEach(([a,b])=>el(a,b).classList.add('f'));
  document.getElementById('w'+w).classList.add('f');
  document.getElementById('msg').textContent='✅ '+w;upd();
  if(found.size==WL.length)document.getElementById('msg').textContent='🏆 ¡Circuito completo! Encontraste todas las palabras.'}
 else document.getElementById('msg').textContent='❌ Intenta de nuevo'}
init();
</script></body></html>
