<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover" />
  <title>Sopa de letras — Mecatrónica</title>
  <style>
    :root{--bg:#f4f9f6;--card:#fff;--tx:#12332b;--g:#1b7a4e;--y:#f5b800;--b:#d5e6dc}
    @media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#0e1f1a;--card:#17302a;--tx:#e8f3ee;--g:#3fcf8e;--y:#f5c542;--b:#2a4a40}}
    *{box-sizing:border-box}
    body{margin:0 auto;background:var(--bg);color:var(--tx);font-family:system-ui,Arial,sans-serif;padding:14px;max-width:720px}
    h1{font-size:22px;margin:6px 0}
    small{opacity:.8}
    #grid{display:grid;grid-template-columns:repeat(var(--N),1fr);gap:6px;margin:14px 0;user-select:none}
    .c{aspect-ratio:1;display:flex;align-items:center;justify-content:center;background:var(--card);border:1px solid var(--b);border-radius:6px;font-weight:800;font-size:clamp(14px,3.4vw,20px);cursor:pointer}
    .c.s{outline:3px solid rgba(27,122,78,.15)}
    .c.f{background:var(--g);color:#fff}
    .c.sel{background:var(--y);color:#222}
    #words{display:flex;flex-wrap:wrap;gap:8px;margin-top:8px}
    .w{background:var(--card);border:1px solid var(--b);border-radius:10px;padding:6px 10px;font-size:14px;font-weight:700}
    .w.f{opacity:.6;text-decoration:line-through}
    .top{display:flex;justify-content:space-between;align-items:center;gap:12px}
    button{font:inherit;padding:8px 14px;border-radius:10px;border:0;background:var(--g);color:#fff;font-weight:700;cursor:pointer}
    #msg{margin:8px 0;font-weight:700;min-height:26px}
    footer{margin-top:12px;color:rgba(0,0,0,.6);font-size:13px}
  </style>
</head>
<body>
  <h1>🔎 Sopa de letras — Mecatrónica</h1>
  <small>Toca (o haz clic) la primera y la última letra de cada palabra para seleccionarla. Puedes seleccionar en horizontal, vertical o diagonal.</small>

  <div class="top" style="margin-top:12px">
    <div id="cnt"></div>
    <div style="display:flex;gap:8px">
      <button id="new">Nueva sopa</button>
      <button id="shuffle">Reubicar palabras</button>
    </div>
  </div>

  <div id="msg"></div>
  <div id="grid" style="--N:14"></div>
  <div id="words"></div>

  <script>
    const WL = [
      "MECATRONICA",
      "EMPLEABILIDAD",
      "PORTAFOLIOS",
      "DIAGNOSTICO",
      "AUTOMATIZACION",
      "DIGITALIZACION",
      "COMPETENCIAS",
      "SABANA",
      "CHIA",
      "TALLERES",
      "MENTORIAS",
      "EGRESADOS"
    ];

    const N = 14;
    const D = [[0,1],[1,0],[1,1],[-1,1],[0,-1],[-1,0],[-1,-1],[1,-1]];

    let G, found, sel;

    function build(){
      for(let attempts=0; attempts<100; attempts++){
        const g = Array.from({length:N}, ()=>Array(N).fill(''));
        let ok=true;
        for(const w of [...WL].sort((a,b)=>b.length-a.length)){
          let placed=false;
          for(let k=0;k<1000 && !placed;k++){
            const dir = D[Math.floor(Math.random()*D.length)];
            const dr = dir[0], dc = dir[1];
            const r = Math.floor(Math.random()*N), c = Math.floor(Math.random()*N);
            const er = r + dr*(w.length-1), ec = c + dc*(w.length-1);
            if(er<0||er>=N||ec<0||ec>=N) continue;
            let clash=false;
            for(let i=0;i<w.length;i++){
              const ch = g[r+dr*i][c+dc*i];
              if(ch && ch !== w[i]){ clash = true; break; }
            }
            if(clash) continue;
            for(let i=0;i<w.length;i++) g[r+dr*i][c+dc*i] = w[i];
            placed = true;
          }
          if(!placed){ ok=false; break; }
        }
        if(ok){
          const A = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
          for(let r=0;r<N;r++) for(let c=0;c<N;c++) if(!g[r][c]) g[r][c] = A[Math.floor(Math.random()*A.length)];
          return g;
        }
      }
      throw new Error('No se pudo generar la sopa con las palabras dadas.');
    }

    function init(){
      G = build(); found = new Set(); sel = null;
      const gr = document.getElementById('grid'); gr.innerHTML = '';
      gr.style.setProperty('--N', N);
      for(let r=0;r<N;r++){
        for(let c=0;c<N;c++){
          const d = document.createElement('div');
          d.className = 'c'; d.id = 'c'+r+'_'+c; d.textContent = G[r][c];
          d.addEventListener('click', ()=> tap(r,c));
          gr.appendChild(d);
        }
      }
      document.getElementById('words').innerHTML = WL.map(w=>`<span class=\"w\" id=\"w${w}\">${w}</span>`).join('');
      document.getElementById('msg').textContent = '';
      upd();
    }

    function el(r,c){ return document.getElementById('c'+r+'_'+c); }
    function upd(){ document.getElementById('cnt').textContent = `Encontradas: ${found.size}/${WL.length}` }

    function tap(r,c){
      if(!sel){ sel = [r,c]; el(r,c).classList.add('s'); return; }
      const [r0,c0] = sel; el(r0,c0).classList.remove('s'); sel = null;
      const dr = r - r0, dc = c - c0, L = Math.max(Math.abs(dr), Math.abs(dc));
      if(L === 0) return;
      if(!(dr === 0 || dc === 0 || Math.abs(dr) === Math.abs(dc))) return;
      const sr = Math.sign(dr), sc = Math.sign(dc);
      let s = '', cells = [];
      for(let i=0;i<=L;i++){ s += G[r0+sr*i][c0+sc*i]; cells.push([r0+sr*i,c0+sc*i]); }
      const rev = [...s].reverse().join('');
      const w = WL.find(x=> x === s || x === rev);
      if(w && !found.has(w)){
        found.add(w);
        cells.forEach(([a,b])=> el(a,b).classList.add('f'));
        const we = document.getElementById('w'+w); if(we) we.classList.add('f');
        document.getElementById('msg').textContent = '✅ ' + w;
        upd();
        if(found.size === WL.length) document.getElementById('msg').textContent = '🏆 ¡Circuito completo! Encontraste todas las palabras.';
      } else {
        document.getElementById('msg').textContent = '❌ Intenta de nuevo';
      }
    }

    document.getElementById('new').addEventListener('click', ()=> init());
    document.getElementById('shuffle').addEventListener('click', ()=> {
      G = build();
      for(let r=0;r<N;r++) for(let c=0;c<N;c++) el(r,c).textContent = G[r][c];
      document.getElementById('msg').textContent = '';
      found = new Set();
      Array.from(document.querySelectorAll('.c')).forEach(x=>x.classList.remove('f','s'));
      document.querySelectorAll('.w').forEach(x=>x.classList.remove('f'));
      upd();
    });

    init();
  </script>

  <footer>
    Generado automáticamente con la lista de palabras proporcionada.
  </footer>
</body>
</html>
