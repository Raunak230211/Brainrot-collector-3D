<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0">
<title>Brainrot 3D: Ultimate Meme Collector</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-user-select: none; }
  body, html { width: 100%; height: 100%; overflow: hidden; background: #0b0f19; font-family: 'Impact', 'Segoe UI', system-ui, sans-serif; }
  #canvas-container { width: 100%; height: 100%; position: absolute; left: 0; top: 0; z-index: 1; }

  /* Top HUD */
  #top-hud {
    position: absolute; top: 10px; left: 10px; right: 10px;
    display: flex; justify-content: space-between; align-items: center;
    z-index: 10; pointer-events: none;
  }
  .hud-card {
    background: rgba(15, 23, 42, 0.92); border: 2px solid #ec4899;
    border-radius: 12px; padding: 6px 14px; color: #fff; pointer-events: auto;
    display: flex; align-items: center; gap: 10px; font-size: 13px; font-weight: 900;
    box-shadow: 0 4px 18px rgba(236,72,153,0.35);
  }
  .aura-txt { color: #c084fc; text-shadow: 0 0 8px #c084fc; }
  .sigma-txt { color: #facc15; text-shadow: 0 0 8px #facc15; }

  /* Side Action Column */
  #side-menu {
    position: absolute; right: 10px; top: 65px; display: flex; flex-direction: column; gap: 8px; z-index: 10;
  }
  .action-btn {
    border: 2px solid #fff; border-radius: 10px; color: #fff;
    padding: 8px 12px; font-weight: 900; font-size: 11px; cursor: pointer;
    box-shadow: 0 4px 0 rgba(0,0,0,0.4); text-align: center;
  }
  .btn-ad { background: linear-gradient(135deg, #10b981, #059669); border-color: #6ee7b7; }
  .btn-dex { background: linear-gradient(135deg, #8b5cf6, #6d28d9); border-color: #c4b5fd; }
  .btn-egg { background: linear-gradient(135deg, #ec4899, #be185d); border-color: #fbcfe8; }
  .btn-spd { background: linear-gradient(135deg, #3b82f6, #1d4ed8); border-color: #93c5fd; }

  /* Center Floating Alert */
  #alert-banner {
    position: absolute; top: 75px; left: 50%; transform: translateX(-50%);
    background: linear-gradient(90deg, #ec4899, #8b5cf6); color: #fff; font-size: 18px; font-weight: 900;
    padding: 10px 24px; border-radius: 30px; border: 2px solid #fff; z-index: 30; display: none;
    box-shadow: 0 0 24px rgba(236, 72, 153, 0.8); text-align: center; pointer-events: none;
  }

  /* Bottom Joystick & Controls */
  #touch-controls {
    position: absolute; bottom: 16px; left: 16px; right: 16px;
    display: flex; justify-content: space-between; align-items: flex-end;
    z-index: 10; pointer-events: none;
  }
  #joystick-base {
    width: 120px; height: 120px; background: rgba(255, 255, 255, 0.2);
    border: 3px solid rgba(236, 72, 153, 0.7); border-radius: 50%;
    position: relative; pointer-events: auto; touch-action: none;
  }
  #joystick-stick {
    width: 48px; height: 48px; background: #ec4899;
    border: 3px solid #fff; border-radius: 50%;
    position: absolute; top: 36px; left: 36px; pointer-events: none;
  }
  .jump-btn {
    width: 76px; height: 76px; border-radius: 50%; border: 3px solid #fff;
    background: radial-gradient(circle, #facc15, #eab308); color: #1e293b;
    font-size: 14px; font-weight: 900; cursor: pointer; pointer-events: auto;
    box-shadow: 0 6px 20px rgba(250, 204, 21, 0.5); display: flex; align-items: center; justify-content: center;
  }

  /* Modals */
  .modal {
    position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
    background: rgba(15, 23, 42, 0.98); border: 3px solid #ec4899; border-radius: 18px;
    padding: 20px; color: #fff; width: 90%; max-width: 360px; max-height: 80vh; overflow-y: auto;
    z-index: 50; display: none; text-align: center; box-shadow: 0 10px 40px rgba(0,0,0,0.85);
  }
  .modal h3 { color: #facc15; font-size: 20px; margin-bottom: 12px; }
  .dex-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-top: 10px; }
  .dex-card {
    background: rgba(255,255,255,0.06); border: 1px solid rgba(255,255,255,0.15);
    border-radius: 10px; padding: 8px; text-align: left; font-size: 11px;
  }
  .dex-card.locked { opacity: 0.35; filter: grayscale(1); }
  .dex-card h4 { font-size: 12px; margin-bottom: 4px; }
  .modal-close { background: #ef4444; margin-top: 16px; border-radius: 8px; border: none; color:#fff; font-weight: 800; padding: 6px 18px; cursor: pointer; }
</style>

<!-- CrazyGames SDK v3 -->
<script src="https://sdk.crazygames.com/crazygames-sdk-v3.js"></script>
<!-- Three.js -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

<div id="canvas-container"></div>

<div id="top-hud">
  <div class="hud-card">
    <span class="aura-txt">🔮 AURA: <span id="aura-val">0</span></span>
    <span class="sigma-txt">🪙 SIGMA: <span id="coins-val">0</span></span>
  </div>
  <div class="hud-card">
    <span>LVL: <span id="lvl-val">1</span></span>
  </div>
</div>

<div id="side-menu">
  <button class="action-btn btn-ad" id="ad-btn">📺 +2500 AURA (FREE)</button>
  <button class="action-btn btn-dex" id="dex-btn">📖 BRAINROT DEX</button>
  <button class="action-btn btn-egg" id="egg-btn">🥚 HATCH PET (100 🪙)</button>
  <button class="action-btn btn-spd" id="spd-btn">⚡ SPEED (50 🪙)</button>
</div>

<div id="alert-banner">SKIBIDI RIZZ!</div>

<div id="touch-controls">
  <div id="joystick-base">
    <div id="joystick-stick"></div>
  </div>
  <button class="jump-btn" id="btn-jump">JUMP</button>
</div>

<!-- Brainrot Dex Modal -->
<div class="modal" id="dex-modal">
  <h3>📖 BRAINROT BESTIARY</h3>
  <p style="font-size:11px; color:#cbd5e1;">Collect all 6 mythical brainrots scattered in the arena!</p>
  <div class="dex-grid" id="dex-list"></div>
  <button class="modal-close" onclick="document.getElementById('dex-modal').style.display='none'">CLOSE</button>
</div>

<!-- Egg Hatching Modal -->
<div class="modal" id="egg-modal">
  <h3>EGG SHOP 🥚</h3>
  <p style="font-size:12px; margin-bottom: 12px; color:#cbd5e1;">Hatch companions like Livvy Dunne & Mewing Chad to multiply your Aura gains!</p>
  <button class="action-btn btn-egg" id="buy-egg-btn" style="width:100%;">HATCH EGG (100 🪙)</button>
  <button class="modal-close" onclick="document.getElementById('egg-modal').style.display='none'">CLOSE</button>
</div>

<script>
/* ==================== CRAZYGAMES REVENUE SDK ==================== */
let csdk = null;
async function initSDK() {
  try {
    if (window.CrazyGames && window.CrazyGames.SDK) {
      csdk = window.CrazyGames.SDK;
      await csdk.init();
      csdk.game.gameplayStart();
    }
  } catch(e) { console.log('Offline/GitHub mode'); }
}
initSDK();

function triggerRewardedAd(onSuccess) {
  if (csdk && csdk.ad) {
    csdk.ad.requestAd('rewarded', {
      adStarted: () => {},
      adFinished: () => onSuccess(),
      adError: () => onSuccess()
    });
  } else {
    onSuccess();
  }
}

function triggerMidgameAd(onDone) {
  if (csdk && csdk.ad) {
    csdk.ad.requestAd('midgame', {
      adStarted: () => {},
      adFinished: () => onDone(),
      adError: () => onDone()
    });
  } else {
    onDone();
  }
}

/* ==================== SOUND SYNTHESIS ==================== */
let actx = null;
function playFx(freq, type='sine', d=0.15) {
  try {
    if (!actx) actx = new (window.AudioContext || window.webkitAudioContext)();
    const osc = actx.createOscillator();
    const g = actx.createGain();
    osc.type = type;
    osc.frequency.setValueAtTime(freq, actx.currentTime);
    g.gain.setValueAtTime(0.15, actx.currentTime);
    g.gain.exponentialRampToValueAtTime(0.01, actx.currentTime + d);
    osc.connect(g); g.connect(actx.destination);
    osc.start(); osc.stop(actx.currentTime + d);
  } catch(e){}
}

/* ==================== BRAINROT BESTIARY DEFINITIONS ==================== */
const BRAINROT_TYPES = [
  {
    id: 'skibidi',
    name: 'Skibidi Toilet',
    tier: 'Common',
    tierCol: '#94a3b8',
    pts: 100,
    desc: 'Porcelain patrol singing in the depths.'
  },
  {
    id: 'grimace',
    name: 'Grimace Shake',
    tier: 'Uncommon',
    tierCol: '#a855f7',
    pts: 250,
    desc: 'One sip and you wake up in Ohio.'
  },
  {
    id: 'mewing',
    name: 'Gigachad Moai',
    tier: 'Rare',
    tierCol: '#38bdf8',
    pts: 500,
    desc: 'Locks in with maximum jawline definition.'
  },
  {
    id: 'fanum',
    name: 'Fanum Tax Pizza',
    tier: 'Epic',
    tierCol: '#f97316',
    pts: 1000,
    desc: 'Takes 10% of everyone\'s lunch instantly.'
  },
  {
    id: 'gronk',
    name: 'Baby Gronk Gold',
    tier: 'Legendary',
    tierCol: '#eab308',
    pts: 2500,
    desc: 'The chosen rizzler of the new generation.'
  },
  {
    id: 'tungsten',
    name: 'Tungsten Rizz Cube',
    tier: 'Mythic',
    tierCol: '#ec4899',
    pts: 10000,
    desc: 'Hyper-dense artifact containing pure sigma energy.'
  }
];

// Dex tracking in localStorage
let dexUnlocked = JSON.parse(localStorage.getItem('br_dex') || '{}');

/* ==================== STATS & UI ==================== */
let aura = parseInt(localStorage.getItem('br_aura') || '0');
let coins = parseInt(localStorage.getItem('br_coins') || '0');
let level = parseInt(localStorage.getItem('br_lvl') || '1');
let speedLvl = parseInt(localStorage.getItem('br_spd') || '1');
let petBonus = parseFloat(localStorage.getItem('br_pet') || '1.0');

function updateUI() {
  document.getElementById('aura-val').textContent = aura;
  document.getElementById('coins-val').textContent = coins;
  document.getElementById('lvl-val').textContent = level;
  localStorage.setItem('br_aura', aura);
  localStorage.setItem('br_coins', coins);
  localStorage.setItem('br_lvl', level);
  localStorage.setItem('br_spd', speedLvl);
  localStorage.setItem('br_pet', petBonus);
  localStorage.setItem('br_dex', JSON.stringify(dexUnlocked));
}

function popBanner(text) {
  const b = document.getElementById('alert-banner');
  b.textContent = text;
  b.style.display = 'block';
  setTimeout(() => { b.style.display = 'none'; }, 1600);
}

// Rewarded Ad
document.getElementById('ad-btn').onclick = () => {
  triggerRewardedAd(() => {
    aura += 2500; coins += 100;
    updateUI(); playFx(600, 'triangle', 0.3);
    popBanner("🔥 +2500 AURA & +100 COINS! (SIGMA)");
  });
};

// Speed upgrade
document.getElementById('spd-btn').onclick = () => {
  const cost = speedLvl * 50;
  if (coins >= cost) {
    coins -= cost; speedLvl++; updateUI();
    playFx(500, 'sine', 0.2); popBanner(`⚡ SPEED LVL ${speedLvl}!`);
  } else {
    alert(`Need ${cost} Sigma Coins!`);
  }
};

// Pet Shop
document.getElementById('egg-btn').onclick = () => { document.getElementById('egg-modal').style.display = 'block'; };
document.getElementById('buy-egg-btn').onclick = () => {
  if (coins >= 100) {
    coins -= 100; petBonus += 1.0; updateUI();
    playFx(850, 'triangle', 0.35); popBanner("🥚 PET HATCHED! (+1x Multiplier)");
    document.getElementById('egg-modal').style.display = 'none';
  } else { alert("Need 100 Sigma Coins!"); }
};

// Open Brainrot Dex
document.getElementById('dex-btn').onclick = () => {
  const list = document.getElementById('dex-list');
  list.innerHTML = '';
  BRAINROT_TYPES.forEach(t => {
    const isFound = dexUnlocked[t.id];
    const card = document.createElement('div');
    card.className = `dex-card ${isFound ? '' : 'locked'}`;
    card.innerHTML = `
      <h4 style="color:${t.tierCol}">${t.name}</h4>
      <div style="font-size:10px; color:#94a3b8;">${t.tier} &bull; ${t.pts} Aura</div>
      <div style="font-size:9px; color:#cbd5e1; margin-top:2px;">${isFound ? t.desc : '??? (Not found yet)'}</div>
      <div style="font-size:10px; color:#facc15; margin-top:4px;">Collected: ${dexUnlocked[t.id] || 0}</div>
    `;
    list.appendChild(card);
  });
  document.getElementById('dex-modal').style.display = 'block';
};

/* ==================== THREE.JS 3D SCENE ==================== */
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x0b0f19);
scene.fog = new THREE.Fog(0x0b0f19, 25, 130);

const camera = new THREE.PerspectiveCamera(65, window.innerWidth / window.innerHeight, 0.1, 300);
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio || 1, 2));
document.getElementById('canvas-container').appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff, 0.85));
const sun = new THREE.DirectionalLight(0xf472b6, 0.8);
sun.position.set(20, 35, 20);
scene.add(sun);

// Arena Floor
const floor = new THREE.Mesh(
  new THREE.PlaneGeometry(64, 64),
  new THREE.MeshLambertMaterial({ color: 0x111827 })
);
floor.rotation.x = -Math.PI / 2;
scene.add(floor);

// Neon Perimeter
const bMat = new THREE.MeshLambertMaterial({ color: 0x4f46e5 });
const bL = new THREE.Mesh(new THREE.BoxGeometry(1, 10, 64), bMat); bL.position.set(-32, 5, 0); scene.add(bL);
const bR = new THREE.Mesh(new THREE.BoxGeometry(1, 10, 64), bMat); bR.position.set(32, 5, 0); scene.add(bR);
const bF = new THREE.Mesh(new THREE.BoxGeometry(64, 10, 1), bMat); bF.position.set(0, 5, 32); scene.add(bF);
const bB = new THREE.Mesh(new THREE.BoxGeometry(64, 10, 1), bMat); bB.position.set(0, 5, -32); scene.add(bB);

// Player Avatar
const player = new THREE.Group();
const pBody = new THREE.Mesh(new THREE.BoxGeometry(1.2, 1.2, 0.6), new THREE.MeshLambertMaterial({ color: 0xec4899 }));
pBody.position.y = 1.2; player.add(pBody);
const pHead = new THREE.Mesh(new THREE.BoxGeometry(0.8, 0.8, 0.8), new THREE.MeshLambertMaterial({ color: 0xfacc15 }));
pHead.position.y = 2.2; player.add(pHead);
const pVisor = new THREE.Mesh(new THREE.BoxGeometry(0.7, 0.2, 0.2), new THREE.MeshLambertMaterial({ color: 0x000000 }));
pVisor.position.set(0, 2.3, 0.42); player.add(pVisor);

const petOrb = new THREE.Mesh(new THREE.SphereGeometry(0.35, 16, 16), new THREE.MeshLambertMaterial({ color: 0x8b5cf6 }));
petOrb.position.set(1.4, 2.2, 0); player.add(petOrb);

scene.add(player);

/* ==================== CUSTOM 3D BRAINROT GENERATOR ==================== */
function createBrainrotMesh(typeId) {
  const g = new THREE.Group();

  if (typeId === 'skibidi') {
    // 3D Toilet Bowl + Smiling Head
    const bowl = new THREE.Mesh(new THREE.CylinderGeometry(0.6, 0.4, 0.6, 16), new THREE.MeshLambertMaterial({ color: 0xffffff }));
    bowl.position.y = 0.3; g.add(bowl);
    const tank = new THREE.Mesh(new THREE.BoxGeometry(0.9, 0.8, 0.4), new THREE.MeshLambertMaterial({ color: 0xffffff }));
    tank.position.set(0, 0.7, -0.4); g.add(tank);
    const head = new THREE.Mesh(new THREE.BoxGeometry(0.45, 0.45, 0.45), new THREE.MeshLambertMaterial({ color: 0xfacc15 }));
    head.position.set(0, 0.85, 0); g.add(head);
  } else if (typeId === 'grimace') {
    // Purple Smoothie Cup + Whipped Cream + Straw
    const cup = new THREE.Mesh(new THREE.CylinderGeometry(0.45, 0.3, 0.9, 16), new THREE.MeshLambertMaterial({ color: 0x7e22ce }));
    cup.position.y = 0.45; g.add(cup);
    const whip = new THREE.Mesh(new THREE.SphereGeometry(0.38, 16, 16), new THREE.MeshLambertMaterial({ color: 0xfdf4ff }));
    whip.position.y = 0.95; g.add(whip);
    const straw = new THREE.Mesh(new THREE.CylinderGeometry(0.04, 0.04, 0.6), new THREE.MeshLambertMaterial({ color: 0xef4444 }));
    straw.rotation.z = 0.25; straw.position.set(0.1, 1.2, 0); g.add(straw);
  } else if (typeId === 'mewing') {
    // Moai Statue Head (Chiseled Jaw)
    const head = new THREE.Mesh(new THREE.BoxGeometry(0.7, 1.3, 0.6), new THREE.MeshLambertMaterial({ color: 0x64748b }));
    head.position.y = 0.65; g.add(head);
    const nose = new THREE.Mesh(new THREE.BoxGeometry(0.2, 0.5, 0.25), new THREE.MeshLambertMaterial({ color: 0x475569 }));
    nose.position.set(0, 0.75, 0.35); g.add(nose);
    const jaw = new THREE.Mesh(new THREE.BoxGeometry(0.75, 0.35, 0.4), new THREE.MeshLambertMaterial({ color: 0x334155 }));
    jaw.position.set(0, 0.2, 0.25); g.add(jaw);
  } else if (typeId === 'fanum') {
    // Pizza Slice (Wedge geometry)
    const pizza = new THREE.Mesh(new THREE.ConeGeometry(0.65, 0.2, 3), new THREE.MeshLambertMaterial({ color: 0xf59e0b }));
    pizza.rotation.x = Math.PI / 2; pizza.position.y = 0.5; g.add(pizza);
    const pep = new THREE.Mesh(new THREE.CylinderGeometry(0.12, 0.12, 0.05), new THREE.MeshLambertMaterial({ color: 0xdc2626 }));
    pep.position.set(0, 0.62, 0.1); g.add(pep);
  } else if (typeId === 'gronk') {
    // Golden Trophy
    const base = new THREE.Mesh(new THREE.CylinderGeometry(0.5, 0.5, 0.3, 16), new THREE.MeshLambertMaterial({ color: 0x78350f }));
    base.position.y = 0.15; g.add(base);
    const cup = new THREE.Mesh(new THREE.SphereGeometry(0.48, 16, 16), new THREE.MeshLambertMaterial({ color: 0xfacc15 }));
    cup.position.y = 0.8; g.add(cup);
  } else {
    // Tungsten Rizz Cube (Prismatic Multi-layer)
    const cube = new THREE.Mesh(new THREE.BoxGeometry(0.8, 0.8, 0.8), new THREE.MeshLambertMaterial({ color: 0xec4899 }));
    cube.position.y = 0.5; g.add(cube);
  }

  // Floating Rarity Glow Ring at Base
  const ring = new THREE.Mesh(
    new THREE.RingGeometry(0.65, 0.85, 24),
    new THREE.MeshBasicMaterial({ color: 0xffffff, side: THREE.DoubleSide })
  );
  ring.rotation.x = -Math.PI / 2;
  ring.position.y = 0.05;
  g.add(ring);

  return g;
}

let items = [];

function spawnBrainrots() {
  items.forEach(it => scene.remove(it.mesh));
  items = [];

  // Spawn 16 randomized Brainrots based on rarity weights
  for (let i = 0; i < 16; i++) {
    const r = Math.random();
    let type;
    if (r < 0.38) type = BRAINROT_TYPES[0]; // Skibidi (38%)
    else if (r < 0.65) type = BRAINROT_TYPES[1]; // Grimace (27%)
    else if (r < 0.82) type = BRAINROT_TYPES[2]; // Moai (17%)
    else if (r < 0.93) type = BRAINROT_TYPES[3]; // Fanum (11%)
    else if (r < 0.98) type = BRAINROT_TYPES[4]; // Baby Gronk (5%)
    else type = BRAINROT_TYPES[5]; // Tungsten Mythic (2%)

    const m = createBrainrotMesh(type.id);
    const rx = (Math.random() - 0.5) * 52;
    const rz = (Math.random() - 0.5) * 52;
    m.position.set(rx, 0, rz);
    scene.add(m);
    items.push({ mesh: m, data: type, isCoin: false });
  }

  // Spawn 8 Sigma Coins
  for (let j = 0; j < 8; j++) {
    const c = new THREE.Mesh(
      new THREE.CylinderGeometry(0.4, 0.4, 0.15, 16),
      new THREE.MeshLambertMaterial({ color: 0xfacc15 })
    );
    c.rotation.x = Math.PI / 2;
    const rx = (Math.random() - 0.5) * 52;
    const rz = (Math.random() - 0.5) * 52;
    c.position.set(rx, 0.6, rz);
    scene.add(c);
    items.push({ mesh: c, data: { name: 'SIGMA COIN', pts: 10 }, isCoin: true });
  }
}

/* ==================== CONTROLS ==================== */
const keys = {};
window.addEventListener('keydown', (e) => { keys[e.code] = true; if (e.code === 'Space') doJump(); });
window.addEventListener('keyup', (e) => { keys[e.code] = false; });

const joyBase = document.getElementById('joystick-base');
const joyStick = document.getElementById('joystick-stick');
let jId = null, jCenter = { x: 0, y: 0 };
let moveDir = new THREE.Vector2();

joyBase.addEventListener('touchstart', (e) => {
  const t = e.changedTouches[0]; jId = t.identifier;
  const r = joyBase.getBoundingClientRect();
  jCenter = { x: r.left + r.width / 2, y: r.top + r.height / 2 };
  updateTouchJoy(t.clientX, t.clientY);
}, { passive: false });

window.addEventListener('touchmove', (e) => {
  for (let i = 0; i < e.changedTouches.length; i++) {
    if (e.changedTouches[i].identifier === jId) updateTouchJoy(e.changedTouches[i].clientX, e.changedTouches[i].clientY);
  }
}, { passive: false });

function resetJoy() { jId = null; joyStick.style.transform = 'translate(0, 0)'; moveDir.set(0, 0); }
window.addEventListener('touchend', (e) => { for (let i = 0; i < e.changedTouches.length; i++) if (e.changedTouches[i].identifier === jId) resetJoy(); });
window.addEventListener('touchcancel', resetJoy);

function updateTouchJoy(cx, cy) {
  const dx = cx - jCenter.x, dy = cy - jCenter.y;
  const dist = Math.hypot(dx, d
