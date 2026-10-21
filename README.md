<!doctype html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>SARS-CoV-2 — 3D портал (Three.js)</title>
<style>
  :root { --primary:#0b3d91; --primary-light:#e3eaf6; --bg:#f4f7fb; --card:#fff;
    --text:#1a1a1a; --muted:#5a6a7a; --border:#d8e0ea; --danger:#c0392b; --success:#1e8449; }
  * { box-sizing: border-box; }
  body { margin:0; font-family:system-ui,-apple-system,"Segoe UI",Roboto,sans-serif;
    background:var(--bg); color:var(--text); line-height:1.6; }
  header { background:var(--primary); color:#fff; padding:1rem 2rem;
    display:flex; justify-content:space-between; align-items:center; flex-wrap:wrap; gap:.5rem; }
  header h1 { margin:0; font-size:1.2rem; } header h1 a { color:#fff; text-decoration:none; }
  nav { display:flex; gap:.3rem; flex-wrap:wrap; }
  nav button { background:transparent; border:1px solid rgba(255,255,255,.35); color:#fff;
    padding:.4rem .9rem; border-radius:6px; cursor:pointer; font-size:.85rem; }
  nav button:hover, nav button.active { background:rgba(255,255,255,.2); }
  main { max-width:1200px; margin:0 auto; padding:1.2rem; }
  section.page { display:none; } section.page.active { display:block; }
  h2 { color:var(--primary); margin-top:0; } h3 { color:var(--primary); }
  .card { background:var(--card); border-radius:12px; padding:1.2rem 1.4rem;
    box-shadow:0 2px 10px rgba(0,0,0,.06); margin-bottom:1rem; }
  .grid2 { display:grid; grid-template-columns:1fr 1fr; gap:1rem; }
  @media (max-width:800px){ .grid2{grid-template-columns:1fr;} }
  .badge { display:inline-block; background:var(--primary-light); color:var(--primary);
    padding:.15rem .6rem; border-radius:20px; font-size:.78rem; margin:.1rem .2rem .1rem 0; }
  table { width:100%; border-collapse:collapse; background:#fff; margin:.8rem 0; }
  th,td { border:1px solid var(--border); padding:.45rem .6rem; text-align:left; font-size:.88rem; }
  th { background:var(--primary-light); color:var(--primary); }
  .btn { display:inline-block; padding:.5rem 1.1rem; background:var(--primary); color:#fff;
    border:none; border-radius:8px; cursor:pointer; font-size:.92rem; margin:.2rem .3rem .2rem 0; }
  .btn:hover { background:#09306f; }
  .btn.secondary { background:var(--primary-light); color:var(--primary); }
  .btn.secondary:hover { background:#d0dcf0; }
  .hidden { display:none !important; }
  .muted { color:var(--muted); font-size:.88rem; }

  .viewer3d { width:100%; height:480px; border-radius:16px; overflow:hidden;
    background:radial-gradient(circle at 50% 45%, #1a1a2e 0%, #08080f 100%); position:relative; }
  .viewer3d canvas { display:block; }
  .overlay-hint { position:absolute; bottom:10px; left:50%; transform:translateX(-50%);
    background:rgba(0,0,0,.55); color:#fff; padding:.35rem .9rem; border-radius:20px;
    font-size:.8rem; pointer-events:none; white-space:nowrap; }
  .loading-overlay { position:absolute; inset:0; display:flex; align-items:center; justify-content:center;
    background:rgba(10,10,20,.9); color:#fff; font-size:.95rem; z-index:5; transition:opacity .4s; }
  .loading-overlay.hide { opacity:0; pointer-events:none; }

  .layer-toggles { display:flex; flex-wrap:wrap; gap:.5rem; margin-top:1rem; }
  .layer-toggles label { display:flex; align-items:center; gap:.4rem; background:#fff;
    padding:.45rem .9rem; border-radius:8px; border:1px solid var(--border);
    cursor:pointer; font-size:.88rem; user-select:none; }
  .layer-toggles input { accent-color:var(--primary); }
  .info-box { background:var(--primary-light); border-left:4px solid var(--primary);
    padding:.8rem 1rem; border-radius:0 8px 8px 0; margin-top:1rem; }
  .info-box h4 { margin:0 0 .3rem; color:var(--primary); }

  .progress-dots { display:flex; gap:.35rem; margin:.8rem 0; flex-wrap:wrap; }
  .progress-dots span { width:30px; height:30px; border-radius:50%; background:#d8e0ea;
    display:flex; align-items:center; justify-content:center; font-size:.75rem;
    cursor:pointer; transition:all .2s; }
  .progress-dots span.active { background:var(--primary); color:#fff; }
  .progress-dots span.done { background:var(--success); color:#fff; }
  .step-info { background:var(--primary-light); border-left:4px solid var(--primary);
    padding:.8rem 1rem; border-radius:0 8px 8px 0; margin-top:.5rem; }
  .step-info h4 { margin:0 0 .3rem; color:var(--primary); }

  .map-wrap { position:relative; background:#0a1628; border-radius:16px; overflow:hidden; }
  .map-svg { width:100%; height:auto; display:block; }
  .map-tooltip { position:absolute; background:#fff; border-radius:8px; padding:.5rem .8rem;
    box-shadow:0 4px 16px rgba(0,0,0,.3); font-size:.82rem; pointer-events:none;
    max-width:230px; z-index:10; display:none; line-height:1.4; }
  .map-legend { display:flex; gap:1rem; flex-wrap:wrap; margin-top:.8rem; font-size:.82rem; }
  .map-legend span { display:flex; align-items:center; gap:.3rem; }
  .map-legend i { width:12px; height:12px; border-radius:50%; display:inline-block; }

  .level-tabs { display:flex; gap:.5rem; margin-bottom:1rem; flex-wrap:wrap; }
  .level-tabs button { padding:.5rem 1.2rem; border-radius:8px; border:1px solid var(--border);
    background:#fff; cursor:pointer; font-size:.92rem; }
  .level-tabs button.active { background:var(--primary); color:#fff; border-color:var(--primary); }
  .quiz-q { margin-bottom:1rem; } .quiz-q p { margin:0 0 .4rem; font-weight:600; }
  .quiz-q label { display:block; padding:.3rem .5rem; border-radius:6px; cursor:pointer; }
  .quiz-q label:hover { background:var(--primary-light); }
  .quiz-q input { margin-right:.5rem; }
  .quiz-result { margin-top:1rem; padding:.8rem; border-radius:8px; font-weight:600; }
  .quiz-result.good { background:#d4edda; color:var(--success); }
  .quiz-result.bad { background:#f8d7da; color:var(--danger); }
  footer { text-align:center; padding:1.2rem; color:var(--muted); font-size:.82rem; }
</style>
</head>
<body>

<header>
  <h1><a href="#" onclick="showPage('home');return false;">🧬 SARS-CoV-2 — 3D портал</a></h1>
  <nav>
    <button data-page="home" class="active" onclick="showPage('home')">Главная</button>
    <button data-page="structure" onclick="showPage('structure')">3D строение</button>
    <button data-page="infection" onclick="showPage('infection')">Заражение</button>
    <button data-page="map" onclick="showPage('map')">Карта</button>
    <button data-page="quiz" onclick="showPage('quiz')">Тест</button>
  </nav>
</header>

<main>

<!-- ГЛАВНАЯ -->
<section id="page-home" class="page active">
  <div class="card">
    <h2>SARS-CoV-2 — возбудитель COVID-19</h2>
    <p>
      <b>Семейство:</b> Coronaviridae • <b>Род:</b> Betacoronavirus •
      <b>Геном:</b> (+)РНК ~29.9 т.п.н. • <b>Диаметр:</b> 80–200 нм
    </p>
    <p>
      Оболочечный вирус с шиповидными белками S, формирующими «корону».
      S-белок связывается с <b>ACE2</b>, затем протеаза <b>TMPRSS2</b>
      разрезает его и запускает слияние мембран.
    </p>
    <p>
      <span class="badge">Betacoronavirus</span> <span class="badge">ssRNA(+)</span>
      <span class="badge">ACE2</span> <span class="badge">TMPRSS2</span>
      <span class="badge">80–200 нм</span> <span class="badge">R0 2.0–3.8</span>
    </p>
  </div>
  <div class="grid2">
    <div class="card">
      <h3>Структурные белки</h3>
      <table>
        <tr><th>Белок</th><th>Функция</th></tr>
        <tr><td><b>S</b></td><td>Связывание ACE2, вход</td></tr>
        <tr><td><b>N</b></td><td>Упаковка РНК</td></tr>
        <tr><td><b>M</b></td><td>Оболочка, сборка</td></tr>
        <tr><td><b>E</b></td><td>Ионный канал</td></tr>
      </table>
    </div>
    <div class="card">
      <h3>Ключевые факты</h3>
      <ul style="margin:.3rem 0; padding-left:1.2rem;">
        <li>16 неструктурных белков (nsp1–16), включая RdRp и 3CLpro</li>
        <li>Инкубация 2–14 дней, в среднем 5–7</li>
        <li>Передача: капельная, аэрозольная, контактная</li>
        <li>Мутации: D614G, N501Y, E484K</li>
        <li>Поражает лёгкие, кишечник, почки, сердце</li>
      </ul>
    </div>
  </div>
</section>

<!-- 3D СТРОЕНИЕ -->
<section id="page-structure" class="page">
  <h2>🔬 3D строение вириона</h2>
  <p class="muted">Вращайте мышью, приближайте колёсиком. Клик по белку — описание. Чекбоксы скрывают слои.</p>
  <div class="viewer3d" id="viewer-structure">
    <div class="loading-overlay" id="loading-structure">Загрузка 3D…</div>
    <div class="overlay-hint">🖱️ Вращение • Колёсико — зум • Клик по белку — информация</div>
  </div>
  <div class="layer-toggles" id="structure-controls">
    <label><input type="checkbox" checked data-layer="spike"> 🔴 S-белки</label>
    <label><input type="checkbox" checked data-layer="envelope"> 🟣 Оболочка</label>
    <label><input type="checkbox" checked data-layer="membrane"> 🟠 M-белки</label>
    <label><input type="checkbox" checked data-layer="e"> 🟪 E-белки</label>
    <label><input type="checkbox" checked data-layer="rna"> 🟢 РНК + N</label>
  </div>
  <div class="info-box">
    <h4 id="struct-info-title">Нажмите на элемент</h4>
    <p id="struct-info-text" class="muted" style="margin:0;">
      S-белок (Spike) — тример, формирующий «корону». Содержит RBD, связывающий ACE2.
      Главная мишень нейтрализующих антител и вакцин.
    </p>
  </div>
</section>

<!-- ЗАРАЖЕНИЕ -->
<section id="page-infection" class="page">
  <h2>🦠 Заражение клетки (3D)</h2>
  <p class="muted">Нажимайте «Далее» — вирус движется по этапам.</p>
  <div class="viewer3d" id="viewer-infection" style="height:420px;">
    <div class="loading-overlay" id="loading-infection">Загрузка сцены…</div>
    <div class="overlay-hint">🖱️ Вращение • Шаги управляются кнопками ниже</div>
  </div>
  <div class="progress-dots" id="infection-dots"></div>
  <div class="step-info" id="infection-info"></div>
  <div style="margin-top:.6rem;">
    <button class="btn secondary" onclick="prevStep()">← Назад</button>
    <button class="btn" onclick="nextStep()">Далее →</button>
    <button class="btn secondary" onclick="resetSteps()">Сброс</button>
  </div>
</section>

<!-- КАРТА -->
<section id="page-map" class="page">
  <h2>🌍 Карта вспышек COVID-19 (2020)</h2>
  <p class="muted">Реальные очертания материков. Наведите на точку — событие.</p>
  <div class="map-wrap" id="map-wrap">
    <svg class="map-svg" viewBox="0 0 1000 500" xmlns="http://www.w3.org/2000/svg">
      <rect width="1000" height="500" fill="#0a1628"/>
      <g stroke="#1a3a5c" stroke-width="0.5" opacity="0.4">
        <line x1="0" y1="125" x2="1000" y2="125"/>
        <line x1="0" y1="250" x2="1000" y2="250"/>
        <line x1="0" y1="375" x2="1000" y2="375"/>
        <line x1="250" y1="0" x2="250" y2="500"/>
        <line x1="500" y1="0" x2="500" y2="500"/>
        <line x1="750" y1="0" x2="750" y2="500"/>
      </g>
      <g fill="#1e3a5f" stroke="#2e5a8f" stroke-width="0.8" opacity="0.9">
        <path d="M120,110 Q150,80 200,75 Q260,70 290,90 Q320,110 310,140 Q300,170 270,180 Q250,200 230,220 Q200,240 180,230 Q150,220 130,190 Q110,160 120,110Z"/>
        <path d="M280,50 Q320,40 350,55 Q370,75 350,100 Q330,115 300,105 Q280,90 280,50Z"/>
        <path d="M240,260 Q270,250 290,270 Q310,300 300,340 Q290,380 270,410 Q250,430 230,420 Q210,400 210,370 Q200,330 220,290 Q230,270 240,260Z"/>
        <path d="M470,90 Q500,80 520,90 Q540,100 530,120 Q520,140 500,145 Q480,150 460,140 Q440,120 450,100 Q460,95 470,90Z"/>
        <path d="M510,60 Q530,50 545,65 Q550,85 540,105 Q525,110 515,100 Q505,85 510,60Z"/>
        <path d="M460,170 Q510,150 550,160 Q580,180 590,220 Q600,270 580,310 Q560,350 520,360 Q480,350 460,310 Q440,270 445,220 Q448,190 460,170Z"/>
        <path d="M590,300 Q605,290 615,305 Q620,325 610,340 Q595,345 585,330 Q580,315 590,300Z"/>
        <path d="M540,85 Q600,70 680,75 Q760,80 820,110 Q860,140 850,180 Q830,210 780,220 Q740,225 700,210 Q660,220 620,200 Q580,180 560,150 Q545,120 540,85Z"/>
        <path d="M650,220 Q680,210 700,225 Q715,250 700,280 Q680,300 660,290 Q640,270 635,245 Q640,230 650,220Z"/>
        <path d="M720,230 Q740,220 760,235 Q770,260 755,280 Q735,290 720,275 Q710,255 720,230Z"/>
        <path d="M830,130 Q845,120 855,135 Q860,155 850,170 Q835,175 825,160 Q820,145 830,130Z"/>
        <path d="M770,340 Q820,320 860,335 Q880,360 870,390 Q850,410 810,405 Q775,395 760,370 Q755,350 770,340Z"/>
        <path d="M890,390 Q900,380 908,395 Q912,410 900,420 Q888,415 885,400Z"/>
        <path d="M0,460 Q200,440 400,450 Q600,445 800,455 Q900,460 1000,450 L1000,500 L0,500Z" fill="#1a2a3f"/>
      </g>
      <g id="map-points"></g>
    </svg>
    <div class="map-tooltip" id="map-tooltip"></div>
  </div>
  <div class="map-legend">
    <span><i style="background:#e74c3c"></i> Дек 2019 — Янв 2020</span>
    <span><i style="background:#e67e22"></i> Фев 2020</span>
    <span><i style="background:#f1c40f"></i> Мар 2020</span>
    <span><i style="background:#2ecc71"></i> Апр–Май 2020</span>
  </div>
  <div class="card" style="margin-top:1rem;">
    <h3>Хронология</h3>
    <ul id="event-list" style="padding-left:1.2rem;"></ul>
  </div>
</section>

<!-- ТЕСТ -->
<section id="page-quiz" class="page">
  <h2>📝 Тест</h2>
  <div class="level-tabs" id="level-tabs">
    <button data-level="basic" class="active" onclick="setLevel('basic')">🟢 Базовый</button>
    <button data-level="medium" onclick="setLevel('medium')">🟡 Средний</button>
    <button data-level="hard" onclick="setLevel('hard')">🔴 Продвинутый</button>
  </div>
  <div class="card">
    <div id="quiz-container"></div>
    <button class="btn" onclick="checkQuiz()">Проверить</button>
    <div id="quiz-result" class="quiz-result hidden"></div>
  </div>
</section>

</main>
<footer>Учебно-научный прототип • 3D: Three.js (CDN)</footer>

<!-- Three.js с CDN -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
/* ============================================================
   НАВИГАЦИЯ
   ============================================================ */
const PAGES = ["home","structure","infection","map","quiz"];
let structInit = false, infInit = false, mapInit = false;

function showPage(name) {
  PAGES.forEach(p => document.getElementById("page-" + p).classList.toggle("active", p === name));
  document.querySelectorAll("nav button").forEach(b => b.classList.toggle("active", b.dataset.page === name));
  window.scrollTo({ top: 0, behavior: "smooth" });
  if (name === "structure" && !structInit) { initStructure(); structInit = true; }
  if (name === "infection" && !infInit) { initInfection(); infInit = true; }
  if (name === "map" && !mapInit) { initMap(); mapInit = true; }
  if (name === "quiz") renderQuiz();
}

/* Проверка, что Three.js загрузился */
function threeReady() { return typeof THREE !== "undefined"; }

/* ============================================================
   3D СТРОЕНИЕ
   ============================================================ */
let structScene, structCamera, structRenderer, structLayers = {};

function initStructure() {
  const container = document.getElementById("viewer-structure");
  const loading = document.getElementById("loading-structure");
  if (!threeReady()) {
    loading.textContent = "Three.js не загрузился. Проверьте интернет.";
    return;
  }
  loading.classList.add("hide");

  const w = container.clientWidth, h = container.clientHeight;
  structScene = new THREE.Scene();
  structCamera = new THREE.PerspectiveCamera(45, w/h, 0.1, 1000);
  structCamera.position.set(0, 1.5, 7);

  structRenderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  structRenderer.setSize(w, h);
  structRenderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  container.appendChild(structRenderer.domElement);

  structScene.add(new THREE.AmbientLight(0xffffff, 0.6));
  const d1 = new THREE.DirectionalLight(0xffffff, 0.9); d1.position.set(5, 5, 5); structScene.add(d1);
  const d2 = new THREE.DirectionalLight(0x88aaff, 0.4); d2.position.set(-5, -3, -5); structScene.add(d2);

  ["spike","envelope","membrane","e","rna"].forEach(k => {
    structLayers[k] = new THREE.Group();
    structScene.add(structLayers[k]);
  });

  // Оболочка
  const envMesh = new THREE.Mesh(
    new THREE.SphereGeometry(1.8, 48, 48),
    new THREE.MeshPhongMaterial({ color: 0x4a2c6a, transparent: true, opacity: 0.5,
      shininess: 30, side: THREE.DoubleSide })
  );
  envMesh.userData = { info: "Липидная оболочка. Двойной слой липидов, украденный у клетки-хозяина. Уязвим для спиртов и детергентов." };
  structLayers.envelope.add(envMesh);

  const wire = new THREE.Mesh(
    new THREE.SphereGeometry(1.805, 24, 24),
    new THREE.MeshBasicMaterial({ color: 0x9b59b6, wireframe: true, transparent: true, opacity: 0.15 })
  );
  structLayers.envelope.add(wire);

  // S-белки
  const spikePositions = [
    [0,2.2,0],[1.6,1.6,0],[1.9,0,1.1],[1.2,-1.5,1.1],[0,-2.2,0],
    [-1.2,-1.5,-1.1],[-1.9,0,-1.1],[-1.6,1.6,0],[1.2,0,-1.7],[-1.2,0,1.7],
    [0,1.9,-1.1],[0,-1.9,1.1]
  ];
  spikePositions.forEach(pos => {
    const g = new THREE.Group();
    const stalk = new THREE.Mesh(
      new THREE.CylinderGeometry(0.06, 0.08, 0.55, 8),
      new THREE.MeshPhongMaterial({ color: 0xe74c3c })
    );
    stalk.position.y = 0.27;
    g.add(stalk);
    const head = new THREE.Mesh(
      new THREE.SphereGeometry(0.18, 12, 12),
      new THREE.MeshPhongMaterial({ color: 0xff6b6b, emissive: 0x330000 })
    );
    head.position.y = 0.58;
    g.add(head);
    g.position.set(pos[0], pos[1], pos[2]);
    g.lookAt(0, 0, 0);
    g.rotateX(Math.PI/2);
    g.userData = { info: "S-белок (Spike). Тример с RBD, связывающим ACE2. Главная мишень антител и вакцин. Мутации N501Y и E484K критичны для ухода от иммунитета." };
    structLayers.spike.add(g);
  });

  // M-белки
  for (let i = 0; i < 20; i++) {
    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    const r = 1.82;
    const m = new THREE.Mesh(
      new THREE.SphereGeometry(0.1, 8, 8),
      new THREE.MeshPhongMaterial({ color: 0xf39c12 })
    );
    m.position.set(r*Math.sin(phi)*Math.cos(theta), r*Math.sin(phi)*Math.sin(theta), r*Math.cos(phi));
    m.userData = { info: "M-белок (Membrane). Самый многочисленный структурный белок. Формирует оболочку, участвует в сборке и почковании." };
    structLayers.membrane.add(m);
  }

  // E-белки
  for (let i = 0; i < 12; i++) {
    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    const r = 1.83;
    const e = new THREE.Mesh(
      new THREE.SphereGeometry(0.06, 6, 6),
      new THREE.MeshPhongMaterial({ color: 0xb07cc6 })
    );
    e.position.set(r*Math.sin(phi)*Math.cos(theta), r*Math.sin(phi)*Math.sin(theta), r*Math.cos(phi));
    e.userData = { info: "E-белок (Envelope). Мелкий ионный канал. Участвует в сборке, почковании и высвобождении вириона." };
    structLayers.e.add(e);
  }

  // РНК
  const rnaPoints = [];
  for (let t = 0; t < Math.PI * 6; t += 0.15) {
    rnaPoints.push(new THREE.Vector3(
      Math.sin(t) * 0.8 * Math.cos(t * 0.7),
      Math.cos(t) * 0.5,
      Math.sin(t) * 0.8 * Math.sin(t * 0.7)
    ));
  }
  const curve = new THREE.CatmullRomCurve3(rnaPoints);
  const rnaMesh = new THREE.Mesh(
    new THREE.TubeGeometry(curve, 120, 0.07, 8, false),
    new THREE.MeshPhongMaterial({ color: 0x27ae60, emissive: 0x0a3a1a })
  );
  rnaMesh.userData = { info: "Генетический материал: (+)РНК ~29.9 т.п.н., упакована N-белком. Содержит 10 ORF; ORF1a/1b кодируют 16 nsp, включая RdRp и протеазы." };
  structLayers.rna.add(rnaMesh);

  for (let i = 0; i < 20; i++) {
    const p = curve.getPoint(i / 20);
    const n = new THREE.Mesh(
      new THREE.SphereGeometry(0.09, 8, 8),
      new THREE.MeshPhongMaterial({ color: 0x2ecc71 })
    );
    n.position.copy(p);
    n.userData = { info: "N-белок (Nucleocapsid). Упаковывает РНК в нуклеокапсид." };
    structLayers.rna.add(n);
  }

  // Управление мышью + raycasting
  const raycaster = new THREE.Raycaster();
  const mouse = new THREE.Vector2();
  let isDragging = false, prevX = 0, prevY = 0;

  container.addEventListener("mousedown", e => { isDragging = true; prevX = e.clientX; prevY = e.clientY; });
  window.addEventListener("mouseup", () => isDragging = false);
  container.addEventListener("mousemove", e => {
    if (isDragging) {
      const dx = e.clientX - prevX, dy = e.clientY - prevY;
      structScene.rotation.y += dx * 0.01;
      structScene.rotation.x += dy * 0.01;
      prevX = e.clientX; prevY = e.clientY;
    }
  });
  container.addEventListener("wheel", e => {
    e.preventDefault();
    structCamera.position.z = Math.max(3, Math.min(15, structCamera.position.z + e.deltaY * 0.005));
  }, { passive: false });

  container.addEventListener("click", e => {
    const rect = structRenderer.domElement.getBoundingClientRect();
    mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
    raycaster.setFromCamera(mouse, structCamera);
    const all = [];
    Object.values(structLayers).forEach(g => g.traverse(c => { if (c.isMesh) all.push(c); }));
    const hits = raycaster.intersectObjects(all);
    if (hits.length > 0) {
      const obj = hits[0].object;
      let info = obj.userData.info || (obj.parent && obj.parent.userData.info);
      if (info) {
        document.getElementById("struct-info-title").textContent = "Информация";
        document.getElementById("struct-info-text").textContent = info;
      }
    }
  });

  document.querySelectorAll("#structure-controls input").forEach(cb => {
    cb.addEventListener("change", () => {
      if (structLayers[cb.dataset.layer]) structLayers[cb.dataset.layer].visible = cb.checked;
    });
  });

  function animate() {
    requestAnimationFrame(animate);
    structScene.rotation.y += 0.002;
    structRenderer.render(structScene, structCamera);
  }
  animate();

  window.addEventListener("resize", () => {
    const w2 = container.clientWidth, h2 = container.clientHeight;
    structCamera.aspect = w2/h2; structCamera.updateProjectionMatrix();
    structRenderer.setSize(w2, h2);
  });
}

/* ============================================================
   3D ЗАРАЖЕНИЕ
   ============================================================ */
let infScene, infCamera, infRenderer, infVirus, infRnaGroup, infNewGroup;
let currentStep = 0;

const STEPS = [
  { t: "Шаг 1. Приближение", d: "Вирус приближается к клетке эпителия дыхательных путей." },
  { t: "Шаг 2. Связывание с ACE2", d: "S-белок связывается с рецептором ACE2." },
  { t: "Шаг 3. TMPRSS2", d: "Протеаза хозяина разрезает S-белок, активируя слияние." },
  { t: "Шаг 4. Слияние мембран", d: "Оболочка вируса сливается с мембраной клетки." },
  { t: "Шаг 5. РНК внутри", d: "(+)РНК высвобождается в цитоплазму." },
  { t: "Шаг 6. Трансляция", d: "Синтезируются неструктурные белки (RdRp, протеазы)." },
  { t: "Шаг 7. Репликация", d: "RdRp создаёт новые копии РНК и субгеномные мРНК." },
  { t: "Шаг 8. Выход", d: "Собираются новые вирионы и выходят из клетки." }
];

function initInfection() {
  const container = document.getElementById("viewer-infection");
  const loading = document.getElementById("loading-infection");
  if (!threeReady()) {
    loading.textContent = "Three.js не загрузился. Проверьте интернет.";
    return;
  }
  loading.classList.add("hide");

  const w = container.clientWidth, h = container.clientHeight;
  infScene = new THREE.Scene();
  infScene.background = new THREE.Color(0x0f172a);
  infCamera = new THREE.PerspectiveCamera(50, w/h, 0.1, 1000);
  infCamera.position.set(0, 0, 12);

  infRenderer = new THREE.WebGLRenderer({ antialias: true });
  infRenderer.setSize(w, h);
  infRenderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  container.appendChild(infRenderer.domElement);

  infScene.add(new THREE.AmbientLight(0xffffff, 0.7));
  const dl = new THREE.DirectionalLight(0xffffff, 1); dl.position.set(3, 5, 8); infScene.add(dl);

  // Клетка
  const cell = new THREE.Mesh(
    new THREE.SphereGeometry(3.5, 48, 48),
    new THREE.MeshPhongMaterial({ color: 0x1a3a5c, transparent: true, opacity: 0.35, side: THREE.DoubleSide })
  );
  cell.position.set(2, 0, 0);
  infScene.add(cell);

  // ACE2
  for (let i = 0; i < 12; i++) {
    const a = (i / 12) * Math.PI * 2;
    const rec = new THREE.Mesh(
      new THREE.CylinderGeometry(0.05, 0.05, 0.4, 6),
      new THREE.MeshPhongMaterial({ color: 0xf1c40f })
    );
    rec.position.set(2 + Math.cos(a) * 3.5, Math.sin(a) * 3.5, 0);
    rec.lookAt(2, 0, 0);
    infScene.add(rec);
  }

  // Вирус
  infVirus = new THREE.Group();
  infVirus.add(new THREE.Mesh(
    new THREE.SphereGeometry(0.7, 24, 24),
    new THREE.MeshPhongMaterial({ color: 0x2c3e50 })
  ));
  for (let i = 0; i < 16; i++) {
    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    const r = 0.72;
    const spike = new THREE.Mesh(
      new THREE.CylinderGeometry(0.03, 0.05, 0.3, 6),
      new THREE.MeshPhongMaterial({ color: 0xe74c3c })
    );
    spike.position.set(r*Math.sin(phi)*Math.cos(theta), r*Math.sin(phi)*Math.sin(theta), r*Math.cos(phi));
    spike.lookAt(0, 0, 0);
    spike.rotateX(Math.PI/2);
    infVirus.add(spike);
  }
  infVirus.position.set(-6, 0, 0);
  infScene.add(infVirus);

  // РНК внутри
  infRnaGroup = new THREE.Group();
  for (let i = 0; i < 6; i++) {
    const s = new THREE.Mesh(
      new THREE.SphereGeometry(0.1, 6, 6),
      new THREE.MeshPhongMaterial({ color: 0x27ae60 })
    );
    s.position.set((Math.random()-0.5)*3 + 2, (Math.random()-0.5)*2, 0.5);
    infRnaGroup.add(s);
  }
  infRnaGroup.visible = false;
  infScene.add(infRnaGroup);

  // Новые вирионы
  infNewGroup = new THREE.Group();
  for (let i = 0; i < 5; i++) {
    const v = new THREE.Mesh(
      new THREE.SphereGeometry(0.35, 12, 12),
      new THREE.MeshPhongMaterial({ color: 0x2c3e50 })
    );
    v.position.set(3 + i * 1.2, 2 + Math.random() * 2, 0);
    infNewGroup.add(v);
  }
  infNewGroup.visible = false;
  infScene.add(infNewGroup);

  // Прогресс-точки
  const dots = document.getElementById("infection-dots");
  STEPS.forEach((s, i) => {
    const span = document.createElement("span");
    span.textContent = i + 1;
    span.onclick = () => { currentStep = i; updateInfection(); };
    dots.appendChild(span);
  });
  updateInfection();

  function animate() {
    requestAnimationFrame(animate);
    infScene.rotation.y += 0.001;
    infRenderer.render(infScene, infCamera);
  }
  animate();

  window.addEventListener("resize", () => {
    const w2 = container.clientWidth, h2 = container.clientHeight;
    infCamera.aspect = w2/h2; infCamera.updateProjectionMatrix();
    infRenderer.setSize(w2, h2);
  });
}

function updateInfection() {
  const positions = [
    new THREE.Vector3(-6, 0, 0),
    new THREE.Vector3(-3.8, 1.5, 0),
    new THREE.Vector3(-2.8, 1.2, 0),
    new THREE.Vector3(-1.5, 1.0, 0),
    new THREE.Vector3(0.5, 0.5, 0),
    new THREE.Vector3(2, 0, 0),
    new THREE.Vector3(2, 0, 0),
    new THREE.Vector3(3, 1.5, 0)
  ];
  if (infVirus && positions[currentStep]) infVirus.position.copy(positions[currentStep]);
  if (infRnaGroup) infRnaGroup.visible = currentStep >= 4;
  if (infNewGroup) infNewGroup.visible = currentStep >= 7;

  document.querySelectorAll("#infection-dots span").forEach((s, i) => {
    s.classList.toggle("active", i === currentStep);
    s.classList.toggle("done", i < currentStep);
  });
  document.getElementById("infection-info").innerHTML =
    `<h4>${STEPS[currentStep].t}</h4><p>${STEPS[currentStep].d}</p>`;
}

function nextStep() { if (currentStep < STEPS.length - 1) { currentStep++; updateInfection(); } }
function prevStep() { if (currentStep > 0) { currentStep--; updateInfection(); } }
function resetSteps() { currentStep = 0; updateInfection(); }

/* ============================================================
   КАРТА
   ============================================================ */
const OUTBREAKS = [
  { x:790, y:190, city:"Ухань, Китай", date:"31 дек 2019", desc:"Первый кластер пневмонии", color:"#e74c3c" },
  { x:755, y:240, city:"Таиланд", date:"13 янв 2020", desc:"Первый случай вне Китая", color:"#e74c3c" },
  { x:835, y:155, city:"Япония", date:"16 янв 2020", desc:"Diamond Princess", color:"#e74c3c" },
  { x:510, y:115, city:"Италия", date:"31 янв 2020", desc:"Эпицентр в Европе", color:"#e67e22" },
  { x:480, y:110, city:"Франция", date:"24 янв 2020", desc:"Первые случаи", color:"#e67e22" },
  { x:230, y:155, city:"США", date:"21 янв 2020", desc:"Более 1 млн к апрелю", color:"#f1c40f" },
  { x:620, y:195, city:"Иран", date:"19 фев 2020", desc:"Эпицентр на Ближнем Востоке", color:"#e67e22" },
  { x:680, y:250, city:"Индия", date:"30 янв 2020", desc:"Локдаун в марте", color:"#f1c40f" },
  { x:270, y:340, city:"Бразилия", date:"26 фев 2020", desc:"Первый случай в Латинской Америке", color:"#2ecc71" },
  { x:820, y:370, city:"Австралия", date:"25 янв 2020", desc:"Локальные вспышки", color:"#2ecc71" }
];

const EVENTS = [
  "31 дек 2019 — ВОЗ уведомлена о пневмонии в Ухане",
  "30 янв 2020 — ЧС в области общественного здравоохранения",
  "11 фев 2020 — болезнь названа COVID-19",
  "11 мар 2020 — объявлена пандемия",
  "Апр 2020 — более 1 млн случаев в мире",
  "Май 2020 — Южная Америка становится эпицентром"
];

function initMap() {
  const g = document.getElementById("map-points");
  const tooltip = document.getElementById("map-tooltip");
  const wrap = document.getElementById("map-wrap");
  if (!g || g.children.length) return;

  OUTBREAKS.forEach(o => {
    const c = document.createElementNS("http://www.w3.org/2000/svg", "circle");
    c.setAttribute("cx", o.x); c.setAttribute("cy", o.y); c.setAttribute("r", 8);
    c.setAttribute("fill", o.color); c.setAttribute("stroke", "#fff"); c.setAttribute("stroke-width", 2);
    c.setAttribute("cursor", "pointer");
    c.onmouseenter = () => {
      c.setAttribute("r", 13);
      tooltip.style.display = "block";
      tooltip.innerHTML = `<b>${o.city}</b><br>${o.date}<br><span style="color:#555">${o.desc}</span>`;
    };
    c.onmousemove = e => {
      const rect = wrap.getBoundingClientRect();
      tooltip.style.left = Math.min(rect.width - 230, e.clientX - rect.left + 15) + "px";
      tooltip.style.top = Math.max(0, e.clientY - rect.top - 20) + "px";
    };
    c.onmouseleave = () => { c.setAttribute("r", 8); tooltip.style.display = "none"; };
    g.appendChild(c);
  });

  const list = document.getElementById("event-list");
  EVENTS.forEach(ev => { const li = document.createElement("li"); li.textContent = ev; list.appendChild(li); });
}

/* ============================================================
   ТЕСТ
   ============================================================ */
const QUIZ = {
  basic: [
    { q:"Какой белок связывается с ACE2?", opts:["S","N","M"], a:0 },
    { q:"Семейство SARS-CoV-2?", opts:["Orthomyxoviridae","Coronaviridae","Filoviridae"], a:1 },
    { q:"Тип генома?", opts:["ДНК","(−)РНК","(+)РНК"], a:2 },
    { q:"D614G — это…", opts:["Белок","Мутация в Spike","Рецептор"], a:1 }
  ],
  medium: [
    { q:"Протеаза, активирующая S?", opts:["TMPRSS2","ACE2","Furin"], a:0 },
    { q:"ORF1a/1b кодируют…", opts:["Структурные белки","pp1ab → 16 nsp","Только Spike"], a:1 },
    { q:"Сборка вирионов происходит в…", opts:["Ядре","ERGIC","Митохондриях"], a:1 },
    { q:"Мутация, повышающая аффинность к ACE2?", opts:["D614G","N501Y","P681H"], a:1 }
  ],
  hard: [
    { q:"Сколько событий расщепления S нужно для входа?", opts:["1","2 (S1/S2 и S2')","0"], a:1 },
    { q:"Ионный канал, участвующий в высвобождении?", opts:["M","E","N"], a:1 },
    { q:"Что НЕ является ко-рецептором?", opts:["NRP1","CD147","Гемоглобин"], a:2 },
    { q:"Отличие S от SARS-CoV-1?", opts:["Нет RBD","Фурин-подобный сайт S1/S2","Нет S2"], a:1 },
    { q:"Репликация РНК происходит в…", opts:["Ядре","Репликационных органеллах ER","Митохондриях"], a:1 }
  ]
};

let currentLevel = "basic";

function setLevel(level) {
  currentLevel = level;
  document.querySelectorAll("#level-tabs button").forEach(b => b.classList.toggle("active", b.dataset.level === level));
  renderQuiz();
}

function renderQuiz() {
  const c = document.getElementById("quiz-container");
  if (!c) return;
  c.innerHTML = "";
  document.getElementById("quiz-result").classList.add("hidden");
  QUIZ[currentLevel].forEach((item, i) => {
    const div = document.createElement("div");
    div.className = "quiz-q";
    div.innerHTML = `<p>${i+1}. ${item.q}</p>` +
      item.opts.map((o, j) => `<label><input type="radio" name="q${i}" value="${j}"> ${o}</label>`).join("");
    c.appendChild(div);
  });
}

function checkQuiz() {
  const items = QUIZ[currentLevel];
  let score = 0;
  items.forEach((item, i) => {
    const sel = document.querySelector(`input[name="q${i}"]:checked`);
    if (sel && parseInt(sel.value) === item.a) score++;
  });
  const r = document.getElementById("quiz-result");
  r.classList.remove("hidden", "good", "bad");
  r.classList.add(score >= items.length * 0.7 ? "good" : "bad");
  r.textContent = `Результат: ${score} из ${items.length} (${Math.round(score/items.length*100)}%)`;
  items.forEach((item, i) => {
    document.querySelectorAll(`input[name="q${i}"]`).forEach((inp, j) => {
      if (j === item.a) inp.parentElement.style.background = "#d4edda";
      else if (inp.checked && j !== item.a) inp.parentElement.style.background = "#f8d7da";
    });
  });
}

/* ============================================================
   СТАРТ
   ============================================================ */
document.addEventListener("DOMContentLoaded", () => {
  showPage("home");
  renderQuiz();
});
</script>
</body>
</html>
