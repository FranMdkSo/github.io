<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Conway’s Game of Life — Learning is the smartest way of life - L&D Rockets 🚀</title>
  <style>
    :root{
      /* Palette inspired by the balloon image (keep orange for title) */
      --bg-1:#0d5bff; /* vivid blue */
      --bg-2:#ffffff; /* white */
      --bg-3:#f1f5f9; /* light grey end */
      --card:#ffffff;
      --text:#0f172a; /* slate-900 */
      --muted:#334155; /* slate-700 for better contrast */
      --line:#e5e7eb; /* gray-200 */
      --brand-blue:#0d5bff; /* primary blue */
      --brand-navy:#102a83; /* deep navy */
      --brand-orange:#f97316; /* keep orange for headline */
      --focus:#38bdf8; /* sky-400 */
      --pill-bg:#e6efff; /* light blue */
      --pill-text:#0b3aa3; /* blue text */
      --grid:#d7e5ff; /* grid lines */
      --grid-major:#d7e5ff; /* same thickness & colour as minor for uniformity */
      --cell:#102a83; /* navy cells */
      --control-border:#bcd2ff;
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{margin:0; color:var(--text);
      font:16px/1.5 system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Cantarell,"Noto Sans",Arial,"Apple Color Emoji","Segoe UI Emoji";
      background: linear-gradient(180deg, var(--bg-2) 0 360px, var(--bg-1) 360px 62vh, var(--bg-3) 62vh 100%);
    }
    .container{max-width:1200px;margin:0 auto;padding:16px;display:grid;gap:16px}

    /* Hero */
    .hero{position:relative;margin-top:8px;display:grid;gap:10px;padding:16px 0}
    .hero::before{content:"";position:absolute;left:50%;transform:translateX(-50%);top:0;width:100vw;height:100%;background:#ffffff;z-index:0;border-bottom:1px solid #eaf0ff}
    .hero > *{position:relative;z-index:1}
    .hero h1{margin:0;font-weight:800;font-size:clamp(20px,2.6vw,30px);line-height:1.15;color:var(--text)}
    .hero .title-grad{background:linear-gradient(90deg,#0b3aa3, var(--brand-blue));-webkit-background-clip:text;background-clip:text;color:transparent}
    .hero .emoji{background:none;-webkit-background-clip:initial;background-clip:initial;color:inherit;-webkit-text-fill-color:initial}
    .hero h2{margin:0;font-weight:900;font-size:clamp(24px,3.2vw,42px);line-height:1.15;color:var(--brand-orange);text-align:center}
    .hero .intro{background:var(--card);border:1px solid var(--control-border);padding:16px;border-radius:14px;box-shadow:0 8px 28px rgba(2,6,23,.08);font-size:15px}

    /* Layout */
    .layout{display:grid;grid-template-columns:360px 1fr;gap:16px;margin-top:6px}
    @media (max-width: 900px){ .layout{grid-template-columns:1fr} .board-card{order:-1} }

    /* Cards */
    .card{background:var(--card);border:1px solid var(--control-border);border-radius:18px;box-shadow:0 10px 30px rgba(2,6,23,.08)}
    .card .section{padding:14px 16px;border-bottom:1px solid #eaf0ff}
    .card .section:last-child{border-bottom:0}

    /* Controls */
    .row{display:flex;align-items:center;gap:10px;flex-wrap:wrap}
    .label{font-weight:700;color:var(--muted)}
    select, input[type=range]{font:inherit}
    select{border:1px solid var(--control-border);border-radius:12px;padding:8px 12px;background:#f8fbff}

    /* Visual-only reordering inside the Controls card */
    .card[aria-label="Controls"]{display:flex;flex-direction:column}
    .card[aria-label="Controls"] .section{order:0}
    .card[aria-label="Controls"] .section:has(#langSel){order:10}
    .card[aria-label="Controls"] .section:has(#speed){order:20}
    .card[aria-label="Controls"] .section:has([data-i18n="rulesTitle"]){order:30}
    .card[aria-label="Controls"] .section:has(#wrap){order:40}
    .card[aria-label="Controls"] .section:has(#preset){order:50}
    .card[aria-label="Controls"] .section:has(#play){order:60}

    .btn{border:1px solid var(--brand-navy);background:var(--brand-navy);color:#fff;padding:10px 14px;border-radius:12px;cursor:pointer;user-select:none;font-weight:800;display:inline-flex;align-items:center;gap:8px}
    .btn[aria-pressed="true"]{background:#0b3aa3}
    .btn.secondary{background:#fff;color:var(--brand-navy);border-color:var(--control-border)}
    .btn.ghost{background:#fff;color:var(--brand-navy);border-color:#e5e7eb}
    .btn.warn{background:var(--brand-orange);border-color:var(--brand-orange)}
    .btn:focus-visible, select:focus-visible, input:focus-visible{outline:3px solid var(--focus);outline-offset:1px}

    .slider-wrap{display:flex;align-items:center;gap:10px}
    #speed{width:180px}

    /* Board */
    .board-card{position:relative;}
    .board-header{position:absolute;left:12px;top:12px;display:flex;gap:8px;z-index:2}
    .pill{background:var(--pill-bg);color:var(--pill-text);border:1px solid var(--control-border);border-radius:999px;padding:6px 10px;font-size:12px;font-weight:800}

    .board-inner{padding:0;display:flex;justify-content:center}
    .board{position:relative;border-radius:16px;border:1px solid var(--control-border);overflow:hidden;display:inline-block}
    canvas{display:block;width:100%;height:100%;background:#ffffff;touch-action:none}

    /* Tooltip for Wrap */
    .has-tip{position:relative; display:inline-flex; align-items:center; gap:10px}
    .has-tip::after{content:attr(data-tip);position:absolute;left:0;top:calc(100% + 6px);background:#111827;color:#fff;border:1px solid #0003;border-radius:8px;padding:6px 8px;font-size:12px;white-space:nowrap;opacity:0;transform:translateY(-4px);transition:opacity .12s ease,transform .12s ease;pointer-events:none;z-index:10}
    .has-tip:hover::after,.has-tip:focus-within::after{opacity:1;transform:translateY(0)}

    /* Patterns modal */
    .modal{position:fixed; inset:0; display:flex; align-items:center; justify-content:center; padding:16px; background:rgba(2,6,23,.46); z-index:1000}
    .modal[hidden]{display:none}
    .modal-card{background:#fff;border:1px solid var(--control-border);border-radius:16px;box-shadow:0 10px 30px rgba(2,6,23,.18);width:min(900px,96vw);max-height:90vh;overflow:auto}
    .modal-header{display:flex;justify-content:space-between;align-items:center;padding:12px 14px;border-bottom:1px solid #eaf0ff}
    .modal-header h3{margin:0;font-size:18px;color:var(--brand-navy)}
    .modal-close{background:#fff;border:1px solid var(--control-border);border-radius:10px;padding:6px 10px;font-weight:800;cursor:pointer}
    .patterns{display:grid;grid-template-columns:repeat(2,minmax(220px,1fr));gap:14px;padding:14px}
    .pattern{border:1px solid var(--control-border);border-radius:12px;padding:10px}
    .pattern figcaption{margin-top:8px;font-weight:700;color:var(--muted)}
    .pattern svg{width:100%;height:auto;display:block;background-image:linear-gradient(to right,#e9f2ff 1px,transparent 1px),linear-gradient(to bottom,#e9f2ff 1px,transparent 1px);background-size:15px 15px;background-color:#fff}
    .cell{fill:var(--cell)}
    @media (max-width:640px){ .patterns{grid-template-columns:1fr} }

    /* Links */
    .links{display:grid;gap:8px}
    .links h3{margin:0;font-size:16px;color:var(--brand-navy)}
    .links a{color:var(--brand-blue);font-weight:700;text-decoration:none;border-bottom:1px dotted var(--brand-blue)}
    .links a:focus-visible,.links a:hover{outline:0;text-decoration:underline}

    /* Utils */
    .accent{color:var(--brand-orange);font-weight:800}
    .sr-only{ position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0, 0, 0, 0); white-space: nowrap; border: 0; }

    /* Mobile-first tweaks */
    @media (max-width: 700px){
      .container{padding:12px}
      .row{gap:8px}
      .slider-wrap{flex-wrap:wrap}
      #speed{width:100%}
      .btn{flex:1;justify-content:center;min-width:calc(50% - 8px)}
      .board-inner{padding:8px}
      .board{border-radius:12px}
      .card{border-radius:14px}
    }
    .sr-only{ position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0, 0, 0, 0); white-space: nowrap; border: 0; }
  </style>
</head>
<body>
  <div class="container">
    <section class="hero" aria-label="Intro">
      <h1><span class="title-grad" data-i18n="heroTitle">Learning is the smartest way of life - L&D Rockets</span> <span class="emoji" aria-hidden="true">🚀</span></h1>
      <h2 data-i18n="heroSubtitle">Can complexity, evolution and life emerge from the simplest rules? Can we create life?</h2>
      <p class="intro" data-i18n="heroIntro" data-i18n-html="true"><strong>Why do I do this?</strong> Because philosophy and learning live in the logic of mathematics; because code and AI spark my human curiosity; and because emotions guide me. I’m driven by the unique power of trying what I’ve never attempted before. I invite you to use your imagination and this metaphor to reflect. This is just an artefact of intellectual provocation. <strong>Why do I do this?</strong></p>
    </section>

    <section class="layout" aria-label="App layout">
      <!-- LEFT: controls -->
      <div class="card" role="group" aria-label="Controls">
        <div class="section">
          <div class="row">
            <span class="label">🌐 <span data-i18n="language">Language</span></span>
            <label class="sr-only" for="langSel" data-i18n="language">Language</label>
            <select id="langSel" aria-label="Language">
              <option value="en" selected>English (UK)</option>
              <option value="ca">Català</option>
              <option value="es">Castellano</option>
              <option value="fr">Français</option>
            </select>
          </div>
        </div>
        <div class="section">
          <div class="row slider-wrap">
            <label class="label" for="speed" data-i18n="speed">Speed</label>
            <input id="speed" type="range" min="1" max="60" value="10" aria-label="Speed" />
            <span id="speedVal">10 gen/s</span>
          </div>
        </div>
        <div class="section">
          <div class="row">
            <div class="has-tip" id="wrapGroup" data-tip="Wrap: connects edges (toroidal). Right↔left, top↔bottom. Press W to toggle.">
              <label class="label" for="wrap" data-i18n="wrap">Wrap</label>
              <input id="wrap" type="checkbox" aria-label="Wrap at edges (W)" />
            </div>
            <button id="patternsBtn" class="btn secondary" type="button" data-i18n="patternsBtn" aria-haspopup="dialog">Patterns</button>
          </div>
        </div>
        <div class="section">
          <div class="row">
            <label class="label" for="preset" data-i18n="grid">Grid</label>
            <select id="preset" aria-label="Grid size preset">
              <option value="32x24">32×24</option>
              <option value="48x36">48×36</option>
              <option value="64x48" selected>64×48</option>
              <option value="96x72">96×72</option>
              <option value="128x96">128×96</option>
            </select>
          </div>
        </div>
        <!-- Rules section inserted here -->
        <div class="section">
          <div class="row" style="align-items:flex-start">
            <div>
              <div class="label" data-i18n="rulesTitle">DOS SIMPLES REGLAS DEL JUEGO</div>
              <p data-i18n="rule1" data-i18n-html="true" style="margin:6px 0 6px 0">1. For a live cell to <strong class="accent">SURVIVE</strong>, it must have 2 or 3 live neighbors. with fewer than 2 or more than 3, it <strong class="accent">DIES</strong>.</p>
              <p data-i18n="rule2" data-i18n-html="true" style="margin:0">2. if a dead cell has exactly 3 live neighbors, it is <strong class="accent">BORN</strong> (comes live).</p>
            </div>
          </div>
        </div>
        <!-- Buttons moved to the end of the toolbar -->
        <div class="section">
          <div class="row">
            <button id="play" class="btn" aria-pressed="false"><span>▶</span><span data-i18n="start">Start</span></button>
            <button id="step" class="btn secondary"><span>⏭</span><span data-i18n="step">Step</span></button>
            <button id="clear" class="btn ghost"><span>🧹</span><span data-i18n="clear">Clear</span></button>
            <button id="random" class="btn warn"><span>🎲</span><span data-i18n="random">Random</span></button>
          </div>
        </div>
      </div>

      <!-- RIGHT: board -->
      <div class="card board-card">
        <div class="board-header">
          <span class="pill"><span data-i18n="gen">Gen</span> <span id="genVal">0</span></span>
          <span class="pill"><span data-i18n="alive">Alive</span> <span id="aliveVal">0</span></span>
        </div>
        <div class="board-inner">
          <div class="board">
            <canvas id="board" aria-label="Game board (click to toggle cells)"></canvas>
            <div class="sr-only" id="liveRegion" aria-live="polite"></div>
          </div>
        </div>
      </div>
    </section>

    <!-- Simple links only, as requested -->
    <section class="card" style="padding:14px 16px" aria-label="Resources">
      <div class="links">
        <h3 data-i18n="linksTitle">Learn more</h3>
        <a data-i18n="linkWiki" href="https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life" target="_blank" rel="noopener">What is the Conway's Game of Life</a>
        <a data-i18n="linkInvent" href="https://www.youtube.com/watch?v=R9Plq-D1gEk" target="_blank" rel="noopener">Inventing the Game of Life</a>
        <a data-i18n="linkNice" href="https://www.youtube.com/watch?v=OWXD_wJxCKQ" target="_blank" rel="noopener">Bonita explicación del juego de Conway</a>
        <a data-i18n="linkLinkedIn" href="https://www.linkedin.com/in/franmondaca/" target="_blank" rel="noopener">Fran Mondaca Linkedin</a>
      </div>
    </section>
  </div>

  <script>
  (function(){
    const $ = s => document.querySelector(s);
    const html = document.documentElement;

    // Elements
    const canvas = $('#board');
    const ctx = canvas.getContext('2d');
    const playBtn = $('#play');
    const stepBtn = $('#step');
    const clearBtn = $('#clear');
    const randBtn = $('#random');
    const wrapChk = $('#wrap');
    const speed = $('#speed');
    const speedVal = $('#speedVal');
    const preset = $('#preset');
    const liveRegion = $('#liveRegion');
    const genVal = $('#genVal');
    const aliveVal = $('#aliveVal');
    const langSel = $('#langSel');
    const wrapGroup = $('#wrapGroup');

    // i18n
    const dict = {
      en:{
        heroTitle:"Learning is the smartest way of life - L&D Rockets",
        heroSubtitle:"Can complexity, evolution and life emerge from the simplest rules? Can we create life?",
        heroIntro:"<strong>Why do I do this?</strong> Because philosophy and learning live in the logic of mathematics; because code and AI spark my human curiosity; and because emotions guide me. I’m driven by the unique power of trying what I’ve never attempted before. I invite you to use your imagination and this metaphor to reflect. This is just an artefact of intellectual provocation.",
        language:"Language", start:"Start", pause:"Pause", step:"Step", clear:"Clear", random:"Random",
        speed:"Speed", wrap:"Wrap (Border)", grid:"Grid", gen:"Gen", alive:"Alive", linksTitle:"Learn more",
        linkWiki:"What is the Conway's Game of Life",
        linkLinkedIn:"Fran Mondaca Linkedin",
        linkInvent:"Inventing the Game of Life",
        linkNice:"Beautiful explanation of Conway's Game",
        wrapTip:"Wrap: connects edges (toroidal). Right↔left, top↔bottom. Press W to toggle.",
        patternsBtn:"Patterns", patternsTitle:"Visual guide — patterns", pGlider:"Glider", pLWSS:"Lightweight spaceship", pPulsar:"Pulsar", pGun:"Gosper glider gun",
        rulesTitle:"TWO SIMPLE RULES OF THE GAME",
        rule1:"1. For a live cell to <strong class=\"accent\">SURVIVE</strong>, it must have 2 or 3 live neighbors. with fewer than 2 or more than 3, it <strong class=\"accent\">DIES</strong>.",
        rule2:"2. if a dead cell has exactly 3 live neighbors, it is <strong class=\"accent\">BORN</strong> (comes live).",
        speedUnit:"gen/s",
        paused:"Paused", running:"Running",
        title:"Conway’s Game of Life — Learning is the smartest way of life - L&D Rockets 🚀"
      },
      es:{
        heroTitle:"Aprender es la forma más inteligente de vivir - L&D Rockets",
        heroSubtitle:"¿Pueden la complejidad, la evolución y la vida surgir de las reglas más simples? ¿Podemos crear vida?",
        heroIntro:"<strong>¿Por qué hago esto?</strong> Porque la filosofía y el aprendizaje viven en la lógica de la matemática, porque el código y la IA encienden mi curiosidad humana, y porque las emociones me guían. Estoy impulsado por el inigualable poder de probar aquello que nunca antes había intentado. Te invito a usar tu imaginación y esta metáfora para reflexionar. Esto es solo un artefacto de provocación intelectual.",
        language:"Idioma", start:"Iniciar", pause:"Pausa", step:"Paso", clear:"Limpiar", random:"Azar",
        speed:"Velocidad", wrap:"Envolver (Borde/Contorno)", grid:"Cuadrícula", gen:"Gen", alive:"Vivas", linksTitle:"Más información",
        linkWiki:"¿Qué es el Juego de la Vida?",
        linkLinkedIn:"LinkedIn de Fran Mondaca",
        linkInvent:"Inventando el Juego de la Vida",
        linkNice:"Bonita explicación del juego de Conway",
        wrapTip:"Envolver: conecta los bordes (toroide). Derecha↔izquierda, arriba↔abajo. Pulsa W para alternar.",
        patternsBtn:"Patrones", patternsTitle:"Guía visual — patrones", pGlider:"Planeador", pLWSS:"Nave ligera (LWSS)", pPulsar:"Púlsar", pGun:"Cañón de planeadores",
        rulesTitle:"DOS SIMPLES REGLAS DEL JUEGO",
        rule1:"1. Para que una célula viva <strong class=\"accent\">SOBREVIVA</strong>, debe tener 2 o 3 vecinas vivas. Con menos de 2 o más de 3, <strong class=\"accent\">MUERE</strong>.",
        rule2:"2. Si una célula muerta tiene exactamente 3 vecinas vivas, <strong class=\"accent\">NACE</strong> (cobra vida).",
        speedUnit:"gen/s",
        paused:"Pausado", running:"En marcha",
        title:"Juego de la Vida de Conway — Aprender es la forma más inteligente de vivir - L&D Rockets 🚀"
      },
      ca:{
        heroTitle:"Aprendre és la manera més intel·ligent de viure - L&D Rockets",
        heroSubtitle:"Pot la complexitat, l’evolució i la vida emergir de les regles més simples? Podem crear vida?",
        heroIntro:"<strong>Per què ho faig?</strong> Perquè la filosofia i l’aprenentatge viuen en la lògica de les matemàtiques; perquè el codi i l’IA encenen la meva curiositat humana; i perquè les emocions em guien. M’empeny el poder únic de provar allò que mai no havia intentat. Et convido a usar la teva imaginació i aquesta metàfora per reflexionar. Això és només un artefacte de provocació intel·lectual.",
        language:"Idioma", start:"Inicia", pause:"Pausa", step:"Pas", clear:"Neteja", random:"Aleatori",
        speed:"Velocitat", wrap:"Contínua (Vora/Contorn)", grid:"Graella", gen:"Gen", alive:"Vives", linksTitle:"Més informació",
        linkWiki:"Què és el Joc de la Vida?",
        linkLinkedIn:"LinkedIn de Fran Mondaca",
        linkInvent:"Inventant el Joc de la Vida",
        linkNice:"Bona explicació del joc de Conway",
        wrapTip:"Contínua: connecta les vores (torus). Dreta↔esquerra, dalt↔baix. Prem W per alternar.",
        patternsBtn:"Patrons", patternsTitle:"Guia visual — patrons", pGlider:"Planejador (Glider)", pLWSS:"Nau lleugera (LWSS)", pPulsar:"Púlsar", pGun:"Canó de planejadors",
        rulesTitle:"DUES REGLES SIMPLES DEL JOC",
        rule1:"1. Perquè una cèl·lula viva <strong class=\"accent\">SOBREVIVI</strong>, ha de tenir 2 o 3 veïnes vives. Amb menys de 2 o més de 3, <strong class=\"accent\">MOR</strong>.",
        rule2:"2. Si una cèl·lula morta té exactament 3 veïnes vives, <strong class=\"accent\">NEIX</strong> (pren vida).",
        speedUnit:"gen/s",
        paused:"En repòs", running:"En marxa",
        title:"Joc de la Vida de Conway — Aprendre és la manera més intel·ligent de viure - L&D Rockets 🚀"
      },
      fr:{
        heroTitle:"Apprendre est la manière la plus intelligente de vivre - L&D Rockets",
        heroSubtitle:"La complexité, l’évolution et la vie peuvent-elles émerger des règles les plus simples ? Pouvons-nous créer la vie ?",
        heroIntro:"<strong>Pourquoi je fais cela ?</strong> Parce que la philosophie et l’apprentissage vivent dans la logique des mathématiques ; parce que le code et l’IA éveillent ma curiosité humaine ; et parce que les émotions me guident. Je suis porté par la puissance unique d’essayer ce que je n’avais jamais tenté. Je vous invite à utiliser votre imagination et cette métaphore pour réfléchir. Ce n’est qu’un artefact de provocation intellectuelle.",
        language:"Langue", start:"Démarrer", pause:"Pause", step:"Pas", clear:"Effacer", random:"Aléatoire",
        speed:"Vitesse", wrap:"Boucler (Bord/Contour)", grid:"Grille", gen:"Gén", alive:"Vivantes", linksTitle:"En savoir plus",
        linkWiki:"Qu’est-ce que le Jeu de la vie ?",
        linkLinkedIn:"LinkedIn de Fran Mondaca",
        linkInvent:"Inventer le Jeu de la vie",
        linkNice:"Belle explication du jeu de Conway",
        wrapTip:"Boucler : relie les bords (tore). Droite↔gauche, haut↔bas. Appuyez sur W pour basculer.",
        patternsBtn:"Motifs", patternsTitle:"Guide visuel — motifs", pGlider:"Planeur", pLWSS:"Vaisseau léger (LWSS)", pPulsar:"Pulsar", pGun:"Canon à planeurs",
        rulesTitle:"DEUX RÈGLES SIMPLES DU JEU",
        rule1:"1. Pour qu’une cellule vivante <strong class=\"accent\">SURVIVE</strong>, elle doit avoir 2 ou 3 voisines vivantes. Avec moins de 2 ou plus de 3, elle <strong class=\"accent\">MEURT</strong>.",
        rule2:"2. Si une cellule morte a exactement 3 voisines vivantes, elle <strong class=\"accent\">NAÎT</strong> (prend vie).",
        speedUnit:"gén/s",
        paused:"En pause", running:"En cours",
        title:"Jeu de la vie de Conway — Apprendre est la manière la plus intelligente de vivre - L&D Rockets 🚀"
      }
    };

    function applyI18n(lang){
      const d = dict[lang] || dict.en;
      html.lang = lang;
      document.title = d.title;
      document.querySelectorAll('[data-i18n]').forEach(el=>{
        const key = el.getAttribute('data-i18n');
        if(d[key]){
          if(el.getAttribute('data-i18n-html') === 'true') el.innerHTML = d[key];
          else el.textContent = d[key];
        }
      });
      // Buttons (Start/Pause state)
      if(running){ playBtn.querySelector('span:last-child').textContent = d.pause; } else { playBtn.querySelector('span:last-child').textContent = d.start; }
      // Speed label
      speedVal.textContent = `${speed.value} ${d.speedUnit}`;
      // Tooltip & aria
      wrapGroup.setAttribute('data-tip', d.wrapTip);
      wrapChk.setAttribute('aria-label', d.wrap + ' (W)');
    }

    document.querySelector('#langSel').addEventListener('change', ()=>{ applyI18n(langSel.value); announce('Language'); draw(); });

    // Simulation state
    let cols = 64, rows = 48;
    let cellsA = new Uint8Array(cols*rows);
    let cellsB = new Uint8Array(cols*rows);
    let front = cellsA, back = cellsB;
    let running = false;
    let generation = 0;
    let lastTime = 0, acc = 0;
    let aliveCount = 0;

    // Layout state
    let dpr = window.devicePixelRatio || 1;
    let cellSize = 12; // CSS px, computed
    let gridW = 0, gridH = 0; // CSS px

    function idx(x,y){ return y*cols + x; }
    function swap(){ const t = front; front = back; back = t; }
    function announce(msg){ liveRegion.textContent = msg; }

    function countAlive(){
      let n=0; for(let i=0;i<front.length;i++) n += front[i];
      aliveCount = n; aliveVal.textContent = String(n);
    }

    function updateGen(){ genVal.textContent = String(generation); }

    function resizeCanvas(){
      const holder = canvas.parentElement; // .board
      const availW = holder.parentElement ? holder.parentElement.clientWidth : holder.clientWidth;
      const safeW = Math.max(280, Math.min(window.innerWidth - 32, availW));
      const safeH = Math.max(240, Math.min(720, window.innerHeight - 240));
      const sizeX = Math.max(6, Math.floor(safeW / cols));
      const sizeY = Math.max(6, Math.floor(safeH / rows));
      cellSize = Math.max(6, Math.min(sizeX, sizeY));
      gridW = cellSize * cols; gridH = cellSize * rows;
      canvas.style.width = gridW + 'px';
      canvas.style.height = gridH + 'px';
      dpr = window.devicePixelRatio || 1;
      canvas.width = Math.round(gridW * dpr);
      canvas.height = Math.round(gridH * dpr);
      ctx.setTransform(dpr,0,0,dpr,0,0);
      draw();
    }

    function clear(){ front.fill(0); generation = 0; updateGen(); countAlive(); draw(); }

    function randomise(){
      for(let i=0;i<front.length;i++) front[i] = Math.random() < 0.33 ? 1 : 0;
      generation = 0; updateGen(); countAlive(); draw();
    }

    function get(x,y){
      if (wrapChk.checked){
        x = (x + cols) % cols; y = (y + rows) % rows;
        return front[idx(x,y)];
      } else {
        if (x < 0 || y < 0 || x >= cols || y >= rows) return 0;
        return front[idx(x,y)];
      }
    }

    function step(){
      for(let y=0;y<rows;y++){
        for(let x=0;x<cols;x++){
          let n = 0;
          n += get(x-1,y-1); n += get(x, y-1); n += get(x+1, y-1);
          n += get(x-1,y  );              n += get(x+1, y  );
          n += get(x-1,y+1); n += get(x, y+1); n += get(x+1, y+1);
          const i = idx(x,y);
          const alive = front[i] === 1;
          back[i] = (alive && (n===2||n===3)) || (!alive && n===3) ? 1 : 0;
        }
      }
      swap();
      generation++; updateGen(); countAlive();
    }

    function draw(){
      // background
      ctx.fillStyle = '#ffffff';
      ctx.fillRect(0,0,gridW,gridH);

      // grid lines (uniform thickness and crisp 1px)
      ctx.beginPath();
      ctx.lineWidth = 1;
      const minor = getComputedStyle(document.documentElement).getPropertyValue('--grid').trim() || '#d7e5ff';
      ctx.strokeStyle = minor;
      for(let x=0;x<=cols;x++){
        const px = Math.round(x*cellSize)+0.5; ctx.moveTo(px,0); ctx.lineTo(px,gridH);
      }
      for(let y=0;y<=rows;y++){
        const py = Math.round(y*cellSize)+0.5; ctx.moveTo(0,py); ctx.lineTo(gridW,py);
      }
      ctx.stroke();

      // Optionally emphasise every 5th line with same thickness/colour (keeps uniform look)
      ctx.beginPath();
      ctx.lineWidth = 1; // same as above
      const major = getComputedStyle(document.documentElement).getPropertyValue('--grid-major').trim() || minor;
      ctx.strokeStyle = major;
      for(let x=0;x<=cols;x+=5){ const px=Math.round(x*cellSize)+0.5; ctx.moveTo(px,0); ctx.lineTo(px,gridH);} 
      for(let y=0;y<=rows;y+=5){ const py=Math.round(y*cellSize)+0.5; ctx.moveTo(0,py); ctx.lineTo(gridW,py);} 
      ctx.stroke();

      // cells
      ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--cell').trim() || '#102a83';
      for(let y=0;y<rows;y++){
        for(let x=0;x<cols;x++){
          if(front[idx(x,y)]){
            ctx.fillRect(x*cellSize+1, y*cellSize+1, cellSize-1, cellSize-1);
          }
        }
      }
    }

    // Interaction
    let isPointerDown = false; let paintValue = 1;
    function cellFromEvent(ev){
      const rect = canvas.getBoundingClientRect();
      const x = Math.floor((ev.clientX - rect.left) / cellSize);
      const y = Math.floor((ev.clientY - rect.top) / cellSize);
      if (x>=0 && y>=0 && x<cols && y<rows) return {x,y};
      return null;
    }
    function setCell(x,y,val){ front[idx(x,y)] = val; }

    canvas.addEventListener('pointerdown', (ev)=>{
      canvas.setPointerCapture(ev.pointerId);
      const c = cellFromEvent(ev); if(!c) return;
      const current = front[idx(c.x,c.y)];
      paintValue = current ? 0 : 1;
      setCell(c.x,c.y, paintValue);
      isPointerDown = true;
      countAlive(); draw();
    });
    canvas.addEventListener('pointermove', (ev)=>{
      if(!isPointerDown) return;
      const c = cellFromEvent(ev); if(!c) return;
      setCell(c.x,c.y, paintValue);
      countAlive(); draw();
    });
    window.addEventListener('pointerup', ()=>{ isPointerDown = false; });
    window.addEventListener('pointercancel', ()=>{ isPointerDown = false; });

    // Controls
    function play(){ running = true; playBtn.setAttribute('aria-pressed','true'); playBtn.querySelector('span:last-child').textContent = (dict[langSel.value]||dict.en).pause; announce('Play'); }
    function pause(){ running = false; playBtn.setAttribute('aria-pressed','false'); playBtn.querySelector('span:last-child').textContent = (dict[langSel.value]||dict.en).start; announce('Pause'); }
    function togglePlay(){ running ? pause() : play(); }

    playBtn.addEventListener('click', togglePlay);
    stepBtn.addEventListener('click', ()=>{ if(!running){ step(); draw(); }});
    clearBtn.addEventListener('click', ()=>{ pause(); clear(); announce('Cleared'); });
    randBtn.addEventListener('click', ()=>{ randomise(); announce('Randomised'); });

    speed.addEventListener('input', ()=>{ const d=dict[langSel.value]||dict.en; speedVal.textContent = `${speed.value} ${d.speedUnit}`; });

    wrapChk.addEventListener('change', ()=>{ const d=dict[langSel.value]||dict.en; announce(`${d.wrap} ${wrapChk.checked?'on':'off'}`); });

    preset.addEventListener('change', ()=>{
      const [c,r] = preset.value.split('x').map(Number);
      cols = c; rows = r;
      front = new Uint8Array(cols*rows);
      back = new Uint8Array(cols*rows);
      pause(); generation = 0; updateGen(); countAlive();
      resizeCanvas();
    });

    // Keyboard (kept working but footer hints removed)
    window.addEventListener('keydown', (e)=>{
      if (e.target && ['INPUT','SELECT','TEXTAREA','BUTTON'].includes(e.target.tagName)){}
      if (e.code === 'Space') { e.preventDefault(); togglePlay(); }
      else if (/^n$/i.test(e.key)) { e.preventDefault(); if(!running){ step(); draw(); }}
      else if (/^r$/i.test(e.key)) { e.preventDefault(); randomise(); }
      else if (/^c$/i.test(e.key)) { e.preventDefault(); pause(); clear(); }
      else if (/^w$/i.test(e.key)) { e.preventDefault(); wrapChk.checked = !wrapChk.checked; const d=dict[langSel.value]||dict.en; announce(`${d.wrap} ${wrapChk.checked?'on':'off'}`); }
    });

    // RAF
    function loop(ts){
      if(!lastTime) lastTime = ts;
      const dt = Math.min(0.25, (ts - lastTime) / 1000);
      lastTime = ts;
      if(running){
        acc += dt;
        const target = 1 / Number(speed.value);
        while (acc >= target){ step(); acc -= target; }
        draw();
      }
      requestAnimationFrame(loop);
    // Patterns modal (deferred binding so it works on GitHub Pages DOM order)
    window.addEventListener('load', ()=>{
      const btn = document.getElementById('patternsBtn');
      const modal = document.getElementById('patternsModal');
      const close = document.getElementById('patternsClose');
      if(!btn || !modal) return;
      const hide = ()=>{ modal.setAttribute('hidden',''); document.body.style.overflow=''; };
      const show = ()=>{ modal.removeAttribute('hidden'); document.body.style.overflow='hidden'; };
      btn.addEventListener('click', show);
      if(close) close.addEventListener('click', hide);
      modal.addEventListener('click', (e)=>{ if(e.target===modal) hide(); });
      window.addEventListener('keydown', (e)=>{ if(e.key==='Escape' && !modal.hasAttribute('hidden')) hide(); });
      // re-apply i18n now that modal elements exist
      applyI18n(document.documentElement.lang || 'en');
    });
    }

    // Init
    applyI18n('en');
    resizeCanvas();
    countAlive();
    updateGen();
    speed.dispatchEvent(new Event('input'));
    window.addEventListener('resize', resizeCanvas);
    requestAnimationFrame(loop);
  })();
  </script>
  <!-- Patterns Modal -->
  <div id="patternsModal" class="modal" hidden role="dialog" aria-modal="true" aria-labelledby="patternsTitle">
    <div class="modal-card">
      <div class="modal-header">
        <h3 id="patternsTitle" data-i18n="patternsTitle">Visual guide — patterns</h3>
        <button class="modal-close" type="button" id="patternsClose" aria-label="Close">✕</button>
      </div>
      <div class="patterns">
        <figure class="pattern">
          <svg viewBox="0 0 75 75" aria-labelledby="p1t" role="img" shape-rendering="crispEdges">
            <!-- Glider 5x5 -->
            <rect class="cell" x="45" y="0" width="15" height="15"/>
            <rect class="cell" x="0" y="15" width="15" height="15"/>
            <rect class="cell" x="45" y="15" width="15" height="15"/>
            <rect class="cell" x="15" y="30" width="15" height="15"/>
            <rect class="cell" x="30" y="30" width="15" height="15"/>
            <rect class="cell" x="45" y="30" width="15" height="15"/>
          </svg>
          <figcaption id="p1t" data-i18n="pGlider">Planeador (Glider)</figcaption>
        </figure>
        <figure class="pattern">
          <svg viewBox="0 0 90 75" aria-labelledby="p2t" role="img" shape-rendering="crispEdges">
            <!-- Lightweight spaceship (simplified 6x5) -->
            <!-- top row -->
            <rect class="cell" x="15" y="0" width="15" height="15"/>
            <rect class="cell" x="30" y="0" width="15" height="15"/>
            <rect class="cell" x="45" y="0" width="15" height="15"/>
            <rect class="cell" x="60" y="0" width="15" height="15"/>
            <!-- row 2 -->
            <rect class="cell" x="0" y="15" width="15" height="15"/>
            <rect class="cell" x="60" y="15" width="15" height="15"/>
            <!-- row 3 -->
            <rect class="cell" x="60" y="30" width="15" height="15"/>
            <!-- row 4 -->
            <rect class="cell" x="0" y="45" width="15" height="15"/>
            <rect class="cell" x="45" y="45" width="15" height="15"/>
          </svg>
          <figcaption id="p2t" data-i18n="pLWSS">Nave ligera (LWSS)</figcaption>
        </figure>
        <figure class="pattern">
          <svg viewBox="0 0 150 150" aria-labelledby="p3t" role="img" shape-rendering="crispEdges">
            <!-- Pulsar 13x13 (indicative arms of 3) -->
            <!-- horizontal arms -->
            <g>
              <rect class="cell" x="30" y="15" width="15" height="15"/>
              <rect class="cell" x="45" y="15" width="15" height="15"/>
              <rect class="cell" x="60" y="15" width="15" height="15"/>

              <rect class="cell" x="90" y="15" width="15" height="15"/>
              <rect class="cell" x="105" y="15" width="15" height="15"/>
              <rect class="cell" x="120" y="15" width="15" height="15"/>
            </g>
            <g>
              <rect class="cell" x="30" y="120" width="15" height="15"/>
              <rect class="cell" x="45" y="120" width="15" height="15"/>
              <rect class="cell" x="60" y="120" width="15" height="15"/>

              <rect class="cell" x="90" y="120" width="15" height="15"/>
              <rect class="cell" x="105" y="120" width="15" height="15"/>
              <rect class="cell" x="120" y="120" width="15" height="15"/>
            </g>
            <!-- vertical arms -->
            <g>
              <rect class="cell" x="15" y="30" width="15" height="15"/>
              <rect class="cell" x="15" y="45" width="15" height="15"/>
              <rect class="cell" x="15" y="60" width="15" height="15"/>

              <rect class="cell" x="15" y="90" width="15" height="15"/>
              <rect class="cell" x="15" y="105" width="15" height="15"/>
              <rect class="cell" x="15" y="120" width="15" height="15"/>
            </g>
            <g>
              <rect class="cell" x="135" y="30" width="15" height="15"/>
              <rect class="cell" x="135" y="45" width="15" height="15"/>
              <rect class="cell" x="135" y="60" width="15" height="15"/>

              <rect class="cell" x="135" y="90" width="15" height="15"/>
              <rect class="cell" x="135" y="105" width="15" height="15"/>
              <rect class="cell" x="135" y="120" width="15" height="15"/>
            </g>
          </svg>
          <figcaption id="p3t" data-i18n="pPulsar">Púlsar (oscillator)</figcaption>
        </figure>
        <figure class="pattern">
          <svg viewBox="0 0 180 120" aria-labelledby="p4t" role="img" shape-rendering="crispEdges">
            <!-- Gosper glider gun (very simplified suggestive layout) -->
            <g>
              <!-- left block -->
              <rect class="cell" x="15" y="45" width="15" height="15"/>
              <rect class="cell" x="30" y="45" width="15" height="15"/>
              <rect class="cell" x="15" y="60" width="15" height="15"/>
              <rect class="cell" x="30" y="60" width="15" height="15"/>
              <!-- right block -->
              <rect class="cell" x="135" y="45" width="15" height="15"/>
              <rect class="cell" x="150" y="45" width="15" height="15"/>
              <rect class="cell" x="135" y="60" width="15" height="15"/>
              <rect class="cell" x="150" y="60" width="15" height="15"/>
              <!-- central nozzle suggestion -->
              <rect class="cell" x="75" y="45" width="15" height="15"/>
              <rect class="cell" x="90" y="45" width="15" height="15"/>
              <rect class="cell" x="90" y="60" width="15" height="15"/>
              <rect class="cell" x="105" y="52.5" width="15" height="15"/>
            </g>
          </svg>
          <figcaption id="p4t" data-i18n="pGun">Cañón de planeadores</figcaption>
        </figure>
      </div>
    </div>
  </div>

</body>
</html>
