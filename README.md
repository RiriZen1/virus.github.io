<!doctype html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>SARS-CoV-2 — 3D интерактивный портал</title>
<style>
  :root {
    --primary: #0b3d91; --primary-light: #e3eaf6; --bg: #f4f7fb; --card: #fff;
    --text: #1a1a1a; --muted: #5a6a7a; --border: #d8e0ea;
    --danger: #c0392b; --success: #1e8449;
  }
  * { box-sizing: border-box; }
  body { margin: 0; font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
    background: var(--bg); color: var(--text); line-height: 1.6; }
  header { background: var(--primary); color: #fff; padding: 1rem 2rem;
    display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: .5rem; }
  header h1 { margin: 0; font-size: 1.2rem; }
  header h1 a { color: #fff; text-decoration: none; }
  nav { display: flex; gap: .3rem; flex-wrap: wrap; }
  nav button { background: transparent; border: 1px solid rgba(255,255,255,.35); color: #fff;
    padding: .4rem .9rem; border-radius: 6px; cursor: pointer; font-size: .85rem; }
  nav button:hover, nav button.active { background: rgba(255,255,255,.2); }
  main { max-width: 1200px; margin: 0 auto; padding: 1.2rem; }
  section.page { display: none; } section.page.active { display: block; }
  h2 { color: var(--primary); margin-top: 0; } h3 { color: var(--primary); }
  .card { background: var(--card); border-radius: 12px; padding: 1.2rem 1.4rem;
    box-shadow: 0 2px 10px rgba(0,0,0,.06); margin-bottom: 1rem; }
  .grid2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
  @media (max-width: 800px) { .grid2 { grid-template-columns: 1fr; } }
  .badge { display: inline-block; background: var(--primary-light); color: var(--primary);
    padding: .15rem .6rem; border-radius: 20px; font-size: .78rem; margin: .1rem .2rem .1rem 0; }
  table { width: 100%; border-collapse: collapse; background: #fff; margin: .8rem 0; }
  th, td { border: 1px solid var(--border); padding: .45rem .6rem; text-align: left; font-size: .88rem; }
  th { background: var(--primary-light); color: var(--primary); }
  .btn { display: inline-block; padding: .5rem 1.1rem; background: var(--primary); color: #fff;
    border: none; border-radius: 8px; cursor: pointer; font-size: .92rem; margin: .2rem .3rem .2rem 0; }
  .btn:hover { background: #09306f; }
  .btn.secondary { background: var(--primary-light); color: var(--primary); }
  .btn.secondary:hover { background: #d0dcf0; }
  .hidden { display: none !important; }
  .muted { color: var(--muted); font-size: .88rem; }

  /* 3D контейнеры */
  .viewer3d { width: 100%; height: 480px; border-radius: 16px; overflow: hidden;
    background: radial-gradient(circle at 50% 45%, #1a1a2e 0%, #0a0a14 100%); position: relative; }
  .viewer3d canvas { display: block; }
  .overlay-hint { position: absolute; bottom: 10px; left: 50%; transform: translateX(-50%);
    background: rgba(0,0,0,.55); color: #fff; padding: .35rem .9rem; border-radius: 20px;
    font-size: .8rem; pointer-events: none; white-space: nowrap; }

  .controls { display: flex; flex-wrap: wrap; gap: .5rem; margin-top: .8rem; }
  .controls label { display: flex; align-items: center; gap: .35rem; font-size: .88rem;
    background: #fff; padding: .35rem .8rem; border-radius: 8px; border: 1px solid var(--border); cursor: pointer; }
  .controls input { accent-color: var(--primary); }

  /* Карта */
  .map-wrap { position: relative; background: #0b1a2e; border-radius: 16px; overflow: hidden; }
  .map-svg { width: 100%; height: auto; display: block; }
  .map-tooltip { position: absolute; background: #fff; border-radius: 8px; padding: .5rem .8rem;
    box-shadow: 0 4px 16px rgba(0,0,0,.3); font-size: .82rem; pointer-events: none;
    max-width: 230px; z-index: 10; display: none; line-height: 1.4; }
  .map-legend { display: flex; gap: 1rem; flex-wrap: wrap; margin-top: .8rem; font-size: .82rem; }
  .map-legend span { display: flex; align-items: center; gap: .3rem; }
  .map-legend i { width: 12px; height: 12px; border-radius: 50%; display: inline-block; }

  /* Заражение */
  .infection-controls { display: flex; align-items: center; gap: 1rem; flex-wrap: wrap; margin-top: .8rem; }
  .step-info { background: var(--primary-light); border-left: 4px solid var(--primary);
    padding: .8rem 1rem; border-radius: 0 8px 8px 0; margin-top: 1rem; }
  .step-info h4 { margin: 0 0 .3rem; color: var(--primary); }
  .progress-dots { display: flex; gap: .35rem; margin: .8rem 0; flex-wrap: wrap; }
  .progress-dots span { width: 30px; height: 30px; border-radius: 50%; background: #d8e0ea;
    display: flex; align-items: center; justify-content: center; font-size: .75rem;
    cursor: pointer; transition: all .2s; }
  .progress-dots span.active { background: var(--primary); color: #fff; }
  .progress-dots span.done { background: var(--success); color: #fff; }

  /* Тест */
  .level-tabs { display: flex; gap: .5rem; margin-bottom: 1rem; flex-wrap: wrap; }
  .level-tabs button { padding: .5rem 1.2rem; border-radius: 8px; border: 1px solid var(--border);
    background: #fff; cursor: pointer; font-size: .92rem; }
  .level-tabs button.active { background: var(--primary); color: #fff; border-color: var(--primary); }
  .quiz-q { margin-bottom: 1rem; } .quiz-q p { margin: 0 0 .4rem; font-weight: 600; }
  .quiz-q label { display: block; padding: .3rem .5rem; border-radius: 6px; cursor: pointer; }
  .quiz-q label:hover { background: var(--primary-light); }
  .quiz-q input { margin-right: .5rem; }
  .quiz-result { margin-top: 1rem; padding: .8rem; border-radius: 8px; font-weight: 600; }
  .quiz-result.good { background: #d4edda; color: var(--success); }
  .quiz-result.bad { background: #f8d7da; color: var(--danger); }
  footer { text-align: center; padding: 1.2rem; color: var(--muted); font-size: .82rem; }
</style>
</head>
<body>

<header>
  <h1><a href="#" onclick="showPage('home');return false;">🧬 SARS-CoV-2 — 3D портал</a></h1>
  <nav>
    <button data-page="home" class="active" onclick="showPage('home')">Главная</button>
    <button data-page="structure" onclick="showPage('structure')">3D строение</button>
    <button data-page="infection" onclick="showPage('infection')">Заражение</button>
    <button data-page="map" onclick="showPage('map')">Карта вспышек</button>
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
      <b>Геном:</b> одноцепочечная (+)РНК ~29.9 т.п.н. •
      <b>Диаметр вириона:</b> 80–200 нм [citation:10]
    </p>
    <p>
      Оболочечный вирус с характерными шиповидными белками S, формирующими «корону».
      S-белок связывается с рецептором <b>ACE2</b> на клетках дыхательных путей,
      после чего протеаза хозяина <b>TMPRSS2</b> разрезает S и запускает слияние мембран [citation:4].
    </p>
    <p>
      <span class="badge">Betacoronavirus</span> <span class="badge">ssRNA(+)</span>
      <span class="badge">ACE2</span> <span class="badge">TMPRSS2</span>
      <span class="badge">80–200 нм</span> <span class="badge">~29.9 kb</span>
      <span class="badge">R0 2.0–3.8</span> [citation:10]
    </p>
  </div>

  <div class="grid2">
    <div class="card">
      <h3>Структурные белки</h3>
      <table>
        <tr><th>Белок</th><th>Функция</th><th>PDB</th></tr>
        <tr><td><b>S</b> (Spike)</td><td>Связывание ACE2, вход в клетку</td><td><a href="https://www.rcsb.org/structure/6VXX" target="_blank">6VXX</a></td></tr>
        <tr><td><b>N</b> (Nucleocapsid)</td><td>Упаковка РНК</td><td><a href="https://www.rcsb.org/structure/6M3M" target="_blank">6M3M</a></td></tr>
        <tr><td><b>M</b> (Membrane)</td><td>Формирование оболочки</td><td><a href="https://www.rcsb.org/structure/8R7D" target="_blank">8R7D</a></td></tr>
        <tr><td><b>E</b> (Envelope)</td><td>Ионный канал, сборка</td><td><a href="https://www.rcsb.org/structure/5X29" target="_blank">5X29</a></td></tr>
      </table>
    </div>
    <div class="card">
      <h3>Неструктурные белки (nsp)</h3>
      <p>
        ORF1a/ORF1b кодируют полипротеины pp1a/pp1ab, которые нарезаются на <b>16 неструктурных белков</b> (nsp1–16).
        Среди них: <b>3CLpro</b> (главная протеаза), <b>RdRp</b> (РНК-зависимая РНК-полимераза),
        <b>Hel</b> (хеликаза), <b>ExoN</b> (корректура) [citation:10].
      </p>
      <h3 style="margin-top:1rem;">Ключевые факты</h3>
      <ul style="margin:.3rem 0; padding-left:1.2rem;">
        <li>Инкубационный период: 2–14 дней, в среднем 5–7 [citation:4].</li>
        <li>Передача: воздушно-капельная, аэрозольная, контактная [citation:10].</li>
        <li>Мутации Spike: <b>D614G</b> (инфекционность), <b>N501Y</b> (аффинность ACE2), <b>E484K</b> (уход от антител).</li>
        <li>Тяжёлое течение связано с «цитокиновым штормом» (IL-6, TNFα) и поражением эндотелия [citation:4].</li>
        <li>Вирус поражает не только лёгкие: ACE2 есть в кишечнике, почках, сердце, эндотелии [citation:4].</li>
      </ul>
    </div>
  </div>

  <div class="card">
    <h3>Клиническая картина</h3>
    <p>
      Основные симптомы: повышение температуры (>90%), кашель (80%), одышка (30%),
      утомляемость (40%), заложенность грудной клетки (>20%). Также возможны боль в горле,
      насморк, снижение обоняния и вкуса, конъюнктивит [citation:4].
    </p>
    <p>
      Наиболее тяжёлая одышка развивается к 6–8 дню от инфицирования.
      Среди первых симптомов могут быть миалгия (11%), спутанность сознания (9%),
      головные боли (8%), диарея (3%) [citation:4].
    </p>
  </div>
</section>

<!-- 3D СТРОЕНИЕ -->
<section id="page-structure" class="page">
  <h2>🔬 3D строение вириона</h2>
  <p class="muted">Вращайте мышью, приближайте колёсиком. Отключайте слои, чтобы увидеть внутреннее строение.</p>

  <div class="viewer3d" id="viewer-structure">
    <div class="overlay-hint">🖱️ Вращение • Колёсико — зум • Клик по белку — информация</div>
  </div>

  <div class="controls" id="structure-controls">
    <label><input type="checkbox" checked data-layer="spike"> S-белки (шипы)</label>
    <label><input type="checkbox" checked data-layer="envelope"> Липидная оболочка</label>
    <label><input type="checkbox" checked data-layer="membrane"> M-белки</label>
    <label><input type="checkbox" checked data-layer="e"> E-белки</label>
    <label><input type="checkbox" checked data-layer="rna"> РНК + N-белок</label>
  </div>

  <div class="card" style="margin-top:1rem;">
    <h3 id="struct-info-title">Нажмите на элемент</h3>
    <p id="struct-info-text" class="muted">
      S-белок (Spike) — тример, формирующий «корону». Содержит RBD, связывающий ACE2.
      Главная мишень нейтрализующих антител и вакцин.
    </p>
  </div>
</section>

<!-- ЗАРАЖЕНИЕ -->
<section id="page-infection" class="page">
  <h2>🦠 Как SARS-CoV-2 заражает организм</h2>
  <p class="muted">Пошаговая 3D-анимация: от приближения к клетке до выхода новых вирионов.</p>

  <div class="viewer3d" id="viewer-infection" style="height:420px;">
    <div class="overlay-hint">🖱️ Вращение • Шаги управляются кнопками ниже</div>
  </div>

  <div class="progress-dots" id="infection-dots"></div>

  <div class="step-info" id="infection-info">
    <h4>Шаг 1. Приближение</h4>
    <p>Вирус приближается к клетке эпителия дыхательных путей. На поверхности — S-белки.</p>
  </div>

  <div class="infection-controls">
    <button class="btn secondary" onclick="prevStep()">← Назад</button>
    <button class="btn" onclick="nextStep()">Далее →</button>
    <button class="btn secondary" onclick="resetSteps()">Сброс</button>
  </div>
</section>

<!-- КАРТА -->
<section id="page-map" class="page">
  <h2>🌍 Карта вспышек COVID-19 (2020)</h2>
  <p class="muted">Наведите на точку — событие. Упрощённые контуры материков для гарантированной загрузки.</p>

  <div class="map-wrap" id="map-wrap">
    <svg class="map-svg" viewBox="0 0 900 480" xmlns="http://www.w3.org/2000/svg">
      <rect width="900" height="480" fill="#0b1a2e"/>
      <!-- Упрощённые контуры (не претендуют на картографическую точность) -->
      <g fill="#1c3a5c" opacity="0.75">
        <path d="M110 130 Q170 80 250 95 Q300 115 280 170 Q250 220 200 235 Q140 220 110 180 Z"/>
        <path d="M195 260 Q240 245 265 290 Q275 345 240 395 Q205 415 185 380 Q165 330 175 290 Z"/>
        <path d="M395 85 Q440 65 485 78 Q510 100 490 135 Q455 155 420 140 Q390 115 395 85 Z"/>
        <path d="M400 165 Q455 145 505 165 Q530 220 505 295 Q470 345 425 330 Q385 295 375 235 Q375 195 400 165 Z"/>
        <path d="M505 65 Q610 40 730 75 Q810 105 825 170 Q800 225 735 240 Q675 250 615 215 Q555 195 515 145 Q485 105 505 65 Z"/>
        <path d="M700 330 Q755 305 805 328 Q825 360 790 395 Q745 405 705 380 Q685 355 700 330 Z"/>
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
  <h2>📝 Проверь себя</h2>
  <div class="level-tabs" id="level-tabs">
    <button data-level="basic" class="active" onclick="setLevel('basic')">🟢 Базовый</button>
    <button data-level="medium" onclick="setLevel('medium')">🟡 Средний</button>
    <button data-level="hard" onclick="setLevel('hard')">🔴 Продвинутый</button>
  </div>
  <div class="card">
    <div id="quiz-container"></div>
    <button class="btn" onclick="checkQuiz()">Проверить ответы</button>
    <div id="quiz-result" class="quiz-result hidden"></div>
  </div>
</section>

</main>
<footer>Учебно-научный прототип • 3D: Three.js • данные: PDB, литература (демо)</footer>

<!-- Three.js с CDN -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
/* ============================================================
   ДАННЫЕ
   ============================================================ */
const OUTBREAKS = [
  { x: 620, y: 155, city: "Ухань, Китай", date: "31 дек 2019", desc: "ВОЗ уведомлена о кластере пневмонии неизвестного происхождения", color: "#e74c3c" },
  { x: 645, y: 205, city: "Таиланд", date: "13 янв 2020", desc: "Первый подтверждённый случай за пределами Китая", color: "#e74c3c" },
  { x: 565, y: 105, city: "Япония", date: "16 янв 2020", desc: "Первый случай; вспышка на Diamond Princess", color: "#e74c3c" },
  { x: 460, y: 115, city: "Италия", date: "31 янв 2020", desc: "Первые случаи в Европе; к марту — эпицентр", color: "#e67e22" },
  { x: 440, y: 98, city: "Франция", date: "24 янв 2020", desc: "Возможная циркуляция уже в конце декабря 2019", color: "#e67e22" },
  { x: 235, y: 155, city: "США", date: "21 янв 2020", desc: "Первый случай; к апрелю — более 1 млн случаев", color: "#f1c40f" },
  { x: 485, y: 205, city: "Иран", date: "19 фев 2020", desc: "Крупная вспышка; новый эпицентр на Ближнем Востоке", color: "#e67e22" },
  { x: 525, y: 235, city: "Индия", date: "30 янв 2020", desc: "Первый случай; в марте — общенациональный локдаун", color: "#f1c40f" },
  { x: 225, y: 325, city: "Бразилия", date: "26 фев 2020", desc: "Первый случай в Латинской Америке", color: "#2ecc71" },
  { x: 745, y: 365, city: "Австралия", date: "25 янв 2020", desc: "Первый случай; локальные вспышки", color: "#2ecc71" }
];

const EVENTS = [
  "31 декабря 2019 — ВОЗ уведомлена о пневмонии неизвестного происхождения в Ухане",
  "30 января 2020 — ВОЗ объявляет чрезвычайную ситуацию в области общественного здравоохранения",
  "11 февраля 2020 — болезнь получает название COVID-19",
  "11 марта 2020 — ВОЗ объявляет пандемию",
  "Апрель 2020 — число случаев в мире превышает 1 млн",
  "Май 2020 — Южная Америка становится новым эпицентром"
];

/* ============================================================
   НАВИГАЦИЯ
   ============================================================ */
const PAGES = ["home","structure","infection","map","quiz"];
let structureInitialized = false, infectionInitialized = false, mapInitialized = false;

function showPage(name) {
  PAGES.forEach(p => document.getElementById("page-" + p).classList.toggle("active", p === name));
  document.querySelectorAll("nav button").forEach(b => b.classList.toggle("active", b.dataset.page === name));
  window.scrollTo({ top: 0, behavior: "smooth" });
  if (name === "structure" && !structureInitialized) { initStructure(); structureInitialized = true; }
  if (name === "infection" && !infectionInitialized) { initInfection(); infectionInitialized = true; }
  if (name === "map" && !mapInitialized) { initMap(); mapInitialized = true; }
  if (name === "quiz") renderQuiz();
}

/* ============================================================
   3D СТРОЕНИЕ — Three.js
   ============================================================ */
let structScene, structCamera, structRenderer, structLayers = {}, structAnimId;

function initStructure() {
  const container = document.getElementById("viewer-structure");
  const w = container.clientWidth, h = container.clientHeight;

  structScene = new THREE.Scene();
  structCamera = new THREE.PerspectiveCamera(45, w/h, 0.1, 1000);
  structCamera.position.set(0, 1.5, 7);

  structRenderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  structRenderer.setSize(w, h);
  structRenderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  container.appendChild(structRenderer.domElement);

  // Свет
  structScene.add(new THREE.AmbientLight(0xffffff, 0.6));
  const d1 = new THREE.DirectionalLight(0xffffff, 0.9); d1.position.set(5, 5, 5); structScene.add(d1);
  const d2 = new THREE.DirectionalLight(0x88aaff, 0.4); d2.position.set(-5, -3, -5); structScene.add(d2);

  // Группы слоёв
  ["spike","envelope","membrane","e","rna"].forEach(k => {
    const g = new THREE.Group(); structLayers[k] = g; structScene.add(g);
  });

  // ---- Оболочка ----
  const envGeo = new THREE.SphereGeometry(1.8, 48, 48);
  const envMat = new THREE.MeshPhongMaterial({
    color: 0x4a2c6a, transparent: true, opacity: 0.55,
    shininess: 30, side: THREE.DoubleSide
  });
  const envMesh = new THREE.Mesh(envGeo, envMat);
  envMesh.userData = { info: "Липидная оболочка. Двойной слой липидов, украденный у клетки-хозяина. Уязвим для спиртов и детергентов." };
  structLayers.envelope.add(envMesh);

  // Сетка на оболочке
  const wireGeo = new THREE.SphereGeometry(1.805, 24, 24);
  const wireMat = new THREE.MeshBasicMaterial({ color: 0x9b59b6, wireframe: true, transparent: true, opacity: 0.15 });
  structLayers.envelope.add(new THREE.Mesh(wireGeo, wireMat));

  // ---- S-белки (шипы) ----
  const spikePositions = [
    [0, 2.2, 0], [1.6, 1.6, 0], [1.9, 0, 1.1], [1.2, -1.5, 1.1],
    [0, -2.2, 0], [-1.2, -1.5, -1.1], [-1.9, 0, -1.1], [-1.6, 1.6, 0],
    [1.2, 0, -1.7], [-1.2, 0, 1.7], [0, 1.9, -1.1], [0, -1.9, 1.1]
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

  // ---- M-белки ----
  for (let i = 0; i < 20; i++) {
    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    const r = 1.82;
    const x = r * Math.sin(phi) * Math.cos(theta);
    const y = r * Math.sin(phi) * Math.sin(theta);
    const z = r * Math.cos(phi);
    const m = new THREE.Mesh(
      new THREE.SphereGeometry(0.1, 8, 8),
      new THREE.MeshPhongMaterial({ color: 0xf39c12 })
    );
    m.position.set(x, y, z);
    m.userData = { info: "M-белок (Membrane). Самый многочисленный структурный белок. Формирует оболочку, участвует в сборке и почковании." };
    structLayers.membrane.add(m);
  }

  // ---- E-белки ----
  for (let i = 0; i < 12; i++) {
    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    const r = 1.83;
    const x = r * Math.sin(phi) * Math.cos(theta);
    const y = r * Math.sin(phi) * Math.sin(theta);
    const z = r * Math.cos(phi);
    const e = new THREE.Mesh(
      new THREE.SphereGeometry(0.06, 6, 6),
      new THREE.MeshPhongMaterial({ color: 0xb07cc6 })
    );
    e.position.set(x, y, z);
    e.userData = { info: "E-белок (Envelope). Мелкий ионный канал. Участвует в сборке, почковании и высвобождении вириона." };
    structLayers.e.add(e);
  }

  // ---- РНК + N ----
  const rnaPoints = [];
  for (let t = 0; t < Math.PI * 6; t += 0.15) {
    rnaPoints.push(new THREE.Vector3(
      Math.sin(t) * 0.8 * Math.cos(t * 0.7),
      Math.cos(t) * 0.5,
      Math.sin(t) * 0.8 * Math.sin(t * 0.7)
    ));
  }
  const rnaCurve = new THREE.CatmullRomCurve3(rnaPoints);
  const rnaGeo = new THREE.TubeGeometry(rnaCurve, 120, 0.07, 8, false);
  const rnaMat = new THREE.MeshPhongMaterial({ color: 0x27ae60, emissive: 0x0a3a1a });
  const rnaMesh = new THREE.Mesh(rnaGeo, rnaMat);
  rnaMesh.userData = { info: "Генетический материал: (+)РНК ~29.9 т.п.н., упакована N-белком. Содержит 10 ORF; ORF1a/1b кодируют 16 nsp, включая RdRp и протеазы." };
  structLayers.rna.add(rnaMesh);

  // N-белки на РНК
  for (let i = 0; i < 20; i++) {
    const t = i / 20;
    const p = rnaCurve.getPoint(t);
    const n = new THREE.Mesh(
      new THREE.SphereGeometry(0.09, 8, 8),
      new THREE.MeshPhongMaterial({ color: 0x2ecc71 })
    );
    n.position.copy(p);
    n.userData = { info: "N-белок (Nucleocapsid). Упаковывает РНК в нуклеокапсид." };
    structLayers.rna.add(n);
  }

  // ---- Вращение + Raycasting ----
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

  // Клик по объекту
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
      let info = obj.userData.info;
      if (!info && obj.parent) info = obj.parent.userData.info;
      if (info) {
        document.getElementById("struct-info-title").textContent = "Информация";
        document.getElementById("struct-info-text").textContent = info;
      }
    }
  });

  // Чекбоксы слоёв
  document.querySelectorAll("#structure-controls input").forEach(cb => {
    cb.addEventListener("change", () => {
      const layer = cb.dataset.layer;
      if (structLayers[layer]) structLayers[layer].visible = cb.checked;
    });
  });

  // Анимация
  function animate() {
    structAnimId = requestAnimationFrame(animate);
    structScene.rotation.y += 0.002;
    structRenderer.render(structScene, structCamera);
  }
  animate();

  // Ресайз
  window.addEventListener("resize", () => {
    const w2 = container.clientWidth, h2 = container.clientHeight;
    structCamera.aspect = w2/h2; structCamera.updateProjectionMatrix();
    structRenderer.setSize(w2, h2);
  });
}

/* ============================================================
   3D ЗАРАЖЕНИЕ
   ============================================================ */
let infScene, infCamera, infRenderer, infSteps = [], infCurrentStep = 0, infAnimId;

const INF_STEPS = [
  { title: "Шаг 1. Приближение", text: "Вирус SARS-CoV-2 диаметром 80–200 нм приближается к клетке эпителия дыхательных путей." },
  { title: "Шаг 2. Связывание с ACE2", text: "S-белок связывается с рецептором ACE2 на поверхности клетки. Это первый специфичный шаг [citation:4]." },
  { title: "Шаг 3. Прайминг TMPRSS2", text: "Протеаза хозяина TMPRSS2 разрезает S-белок в сайте S2', активируя его для слияния [citation:4]." },
  { title: "Шаг 4. Слияние мембран", text: "S2-субъединица встраивается в мембрану клетки и запускает слияние. Вирусная оболочка сливается с клеточной [citation:4]." },
  { title: "Шаг 5. РНК в цитоплазме", text: "(+)РНК высвобождается в цитоплазму. Начинается трансляция ORF1a/ORF1b → pp1a/pp1ab [citation:10]." },
  { title: "Шаг 6. Неструктурные белки", text: "Из pp1ab нарезаются 16 nsp (включая RdRp и протеазы). Они перестраивают мембраны ER в репликационные органеллы [citation:10]." },
  { title: "Шаг 7. Репликация РНК", text: "RdRp синтезирует (−)РНК как матрицу, затем новые (+)РНК и субгеномные мРНК для S, E, M, N [citation:10]." },
  { title: "Шаг 8. Сборка и выход", text: "Белки и РНК собираются в ERGIC, почкуются и выходят через экзоцитоз, заражая соседние клетки [citation:10]." }
];

function initInfection() {
  const container = document.getElementById("viewer-infection");
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

  // Клетка (большая сфера)
  const cell = new THREE.Mesh(
    new THREE.SphereGeometry(3.5, 48, 48),
    new THREE.MeshPhongMaterial({ color: 0x1a3a5c, transparent: true, opacity: 0.35, side: THREE.DoubleSide })
  );
  cell.position.set(2, 0, 0);
  infScene.add(cell);

  // Рецепторы ACE2 (жёлтые стержни на поверхности)
  for (let i = 0; i < 12; i++) {
    const angle = (i / 12) * Math.PI * 2;
    const r = 3.5;
    const rec = new THREE.Mesh(
      new THREE.CylinderGeometry(0.05, 0.05, 0.4, 6),
      new THREE.MeshPhongMaterial({ color: 0xf1c40f })
    );
    rec.position.set(2 + Math.cos(angle) * r, Math.sin(angle) * r, 0);
    rec.lookAt(2, 0, 0);
    infScene.add(rec);
  }

  // Вирус (группа)
  const virus = new THREE.Group();
  const body = new THREE.Mesh(
    new THREE.SphereGeometry(0.7, 24, 24),
    new THREE.MeshPhongMaterial({ color: 0x2c3e50 })
  );
  virus.add(body);
  for (let i = 0; i < 16; i++) {
    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    const r = 0.72;
    const sx = r * Math.sin(phi) * Math.cos(theta);
    const sy = r * Math.sin(phi) * Math.sin(theta);
    const sz = r * Math.cos(phi);
    const spike = new THREE.Mesh(
      new THREE.CylinderGeometry(0.03, 0.05, 0.3, 6),
      new THREE.MeshPhongMaterial({ color: 0xe74c3c })
    );
    spike.position.set(sx, sy, sz);
    spike.lookAt(0, 0, 0);
    spike.rotateX(Math.PI/2);
    virus.add(spike);
  }
  virus.position.set(-6, 0, 0);
  infScene.add(virus);
  infSteps.push(virus);

  // РНК внутри (появится позже)
  const rnaGroup = new THREE.Group();
  for (let i = 0; i < 6; i++) {
    const s = new THREE.Mesh(
      new THREE.SphereGeometry(0.1, 6, 6),
      new THREE.MeshPhongMaterial({ color: 0x27ae60 })
    );
    s.position.set(
      (Math.random() - 0.5) * 3 + 2,
      (Math.random() - 0.5) * 2,
      0.5
    );
    rnaGroup.add(s);
  }
  rnaGroup.visible = false;
  infScene.add(rnaGroup);
  infSteps.push(rnaGroup);

  // Новые вирионы (появятся в конце)
  const newVirions = new THREE.Group();
  for (let i = 0; i < 5; i++) {
    const v = new THREE.Mesh(
      new THREE.SphereGeometry(0.35, 12, 12),
      new THREE.MeshPhongMaterial({ color: 0x2c3e50 })
    );
    v.position.set(3 + i * 1.2, 2 + Math.random() * 2, 0);
    newVirions.add(v);
  }
  newVirions.visible = false;
  infScene.add(newVirions);
  infSteps.push(newVirions);

  // Прогресс-точки
  const dots = document.getElementById("infection-dots");
  INF_STEPS.forEach((s, i) => {
    const span = document.createElement("span");
    span.textContent = i + 1;
    span.onclick = () => { infCurrentStep = i; updateInfection(); };
    dots.appendChild(span);
  });
  updateInfection();

  // Анимация
  function animate() {
    infAnimId = requestAnimationFrame(animate);
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
  const virus = infSteps[0];
  const rnaGroup = infSteps[1];
  const newVirions = infSteps[2];

  // Позиции вируса по шагам
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
  if (virus && positions[infCurrentStep]) {
    virus.position.copy(positions[infCurrentStep]);
  }

  // РНК появляется на шаге 5
  if (rnaGroup) rnaGroup.visible = infCurrentStep >= 4;
  // Новые вирионы на шаге 8
  if (newVirions) newVirions.visible = infCurrentStep >= 7;

  document.querySelectorAll("#infection-dots span").forEach((s, i) => {
    s.classList.toggle("active", i === infCurrentStep);
    s.classList.toggle("done", i < infCurrentStep);
  });

  const info = INF_STEPS[infCurrentStep];
  document.getElementById("infection-info").innerHTML =
    `<h4>${info.title}</h4><p>${info.text}</p>`;
}

function nextStep() { if (infCurrentStep < INF_STEPS.length - 1) { infCurrentStep++; updateInfection(); } }
function prevStep() { if (infCurrentStep > 0) { infCurrentStep--; updateInfection(); } }
function resetSteps() { infCurrentStep = 0; updateInfection(); }

/* ============================================================
   КАРТА
   ============================================================ */
function initMap() {
  const g = document.getElementById("map-points");
  if (g.children.length) return;
  const tooltip = document.getElementById("map-tooltip");
  const wrap = document.getElementById("map-wrap");

  OUTBREAKS.forEach(o => {
    const c = document.createElementNS("http://www.w3.org/2000/svg", "circle");
    c.setAttribute("cx", o.x); c.setAttribute("cy", o.y); c.setAttribute("r", 9);
    c.setAttribute("fill", o.color); c.setAttribute("stroke", "#fff"); c.setAttribute("stroke-width", 2);
    c.setAttribute("cursor", "pointer");
    c.style.transition = "r .15s";
    c.onmouseenter = e => {
      c.setAttribute("r", 14);
      tooltip.style.display = "block";
      tooltip.innerHTML = `<b>${o.city}</b><br>${o.date}<br><span style="color:#555">${o.desc}</span>`;
    };
    c.onmousemove = e => {
      const rect = wrap.getBoundingClientRect();
      tooltip.style.left = Math.min(rect.width - 240, e.clientX - rect.left + 15) + "px";
      tooltip.style.top = Math.max(0, e.clientY - rect.top - 20) + "px";
    };
    c.onmouseleave = () => { c.setAttribute("r", 9); tooltip.style.display = "none"; };
    g.appendChild(c);
  });

  const list = document.getElementById("event-list");
  if (!list.children.length) {
    EVENTS.forEach(ev => { const li = document.createElement("li"); li.textContent = ev; list.appendChild(li); });
  }
}

/* ============================================================
   ТЕСТ
   ============================================================ */
const QUIZ = {
  basic: [
    { q: "Какой белок SARS-CoV-2 связывается с ACE2?", opts: ["S (Spike)", "N (Nucleocapsid)", "M (Membrane)"], a: 0 },
    { q: "К какому семейству относится SARS-CoV-2?", opts: ["Orthomyxoviridae", "Coronaviridae", "Filoviridae"], a: 1 },
    { q: "Какой тип генома у SARS-CoV-2?", opts: ["ДНК", "(−)РНК", "(+)РНК"], a: 2 },
    { q: "Что такое D614G?", opts: ["Белок", "Мутация в Spike", "Рецептор"], a: 1 }
  ],
  medium: [
    { q: "Какая протеаза хозяина активирует S-белок?", opts: ["TMPRSS2", "ACE2", "Furin (только)"], a: 0 },
    { q: "Что кодируют ORF1a и ORF1b?", opts: ["Структурные белки", "Полипротеин pp1ab → 16 nsp", "Только Spike"], a: 1 },
    { q: "Где происходит сборка новых вирионов?", opts: ["В ядре", "В ERGIC", "В митохондриях"], a: 1 },
    { q: "Какая мутация повышает аффинность к ACE2?", opts: ["D614G", "N501Y", "P681H"], a: 1 }
  ],
  hard: [
    { q: "Какие два события расщепления S-белка необходимы для входа?", opts: ["Только S1/S2", "S1/S2 и S2'", "Только S2'"], a: 1 },
    { q: "Какой белок образует ионный канал и участвует в высвобождении?", opts: ["M", "E", "N"], a: 1 },
    { q: "Что из перечисленного НЕ является подтверждённым ко-рецептором?", opts: ["Нейропилин-1", "BASIGIN/CD147", "Гемоглобин"], a: 2 },
    { q: "Какая особенность S-белка SARS-CoV-2 отличает его от SARS-CoV-1?", opts: ["Отсутствие RBD", "Фурин-подобный сайт на границе S1/S2", "Отсутствие S2-субъединицы"], a: 1 },
    { q: "Где происходит репликация РНК?", opts: ["В ядре", "В репликационных органеллах из мембран ER", "В митохондриях"], a: 1 }
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
