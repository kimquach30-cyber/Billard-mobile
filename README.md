<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="mobile-web-app-capable" content="yes">
<meta name="theme-color" content="#000000">
<title>🎱 Billard 3D Mobile</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
  html, body {
    width: 100%; height: 100%; overflow: hidden;
    background: #000; font-family: -apple-system, 'Segoe UI', Arial, sans-serif;
    color: #fff; user-select: none; -webkit-user-select: none;
    touch-action: none; overscroll-behavior: none;
    position: fixed;
  }
  canvas { display: block; touch-action: none; }

  /* ============================
     HUD COMPACT MOBILE
  ============================ */
  #hud {
    position: absolute;
    top: calc(env(safe-area-inset-top, 0px) + 8px);
    left: 8px;
    background: rgba(0,0,0,0.7);
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    border: 1px solid rgba(0,255,255,0.3);
    border-radius: 10px;
    padding: 6px 12px;
    font-size: 11px;
    line-height: 1.5;
    pointer-events: none;
    letter-spacing: 0.5px;
    z-index: 10;
  }
  #hud .turn { color: #ffcc00; font-weight: bold; }
  #hud b { color: #0ff; }

  /* ============================
     INDICATEUR DE TOUR (gros, en haut à droite)
  ============================ */
  #turnBadge {
    position: absolute;
    top: calc(env(safe-area-inset-top, 0px) + 8px);
    right: 8px;
    background: rgba(0,0,0,0.75);
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    border: 2px solid #0ff;
    border-radius: 12px;
    padding: 6px 14px;
    font-size: 11px;
    font-weight: bold;
    letter-spacing: 1px;
    color: #0ff;
    text-align: center;
    pointer-events: none;
    z-index: 10;
    box-shadow: 0 0 15px rgba(0,255,255,0.4);
    transition: all 0.3s;
  }
  #turnBadge.other {
    border-color: #ff6464;
    color: #ff6464;
    box-shadow: 0 0 15px rgba(255,100,100,0.4);
  }
  #turnBadge .sub {
    font-size: 9px;
    color: #888;
    font-weight: normal;
    display: block;
    letter-spacing: 0.5px;
  }

  /* ============================
     BARRE DE PUISSANCE
  ============================ */
  #power {
    position: absolute;
    bottom: calc(env(safe-area-inset-bottom, 0px) + 100px);
    left: 50%;
    transform: translateX(-50%);
    width: min(70vw, 300px);
    height: 12px;
    background: rgba(0,0,0,0.8);
    border-radius: 6px;
    border: 1px solid rgba(255,255,255,0.3);
    overflow: hidden;
    opacity: 0;
    transition: opacity 0.15s;
    z-index: 20;
    pointer-events: none;
  }
  #power.show { opacity: 1; }
  #powerFill {
    height: 100%; width: 0%;
    background: linear-gradient(90deg, #0f0, #ff0, #f00);
    border-radius: 6px;
  }

  /* ============================
     MESSAGES
  ============================ */
  #msg {
    position: absolute;
    top: 40%; left: 50%;
    transform: translate(-50%, -50%) scale(0.7);
    font-size: clamp(18px, 5vw, 32px);
    font-weight: bold;
    text-shadow: 0 0 30px #0ff, 0 0 60px #0ff;
    opacity: 0;
    transition: all 0.35s;
    letter-spacing: 2px;
    pointer-events: none;
    white-space: nowrap;
    text-align: center;
    z-index: 30;
    padding: 0 20px;
  }
  #msg.show { opacity: 1; transform: translate(-50%, -50%) scale(1); }

  /* ============================
     BOUTONS COMPACTS (bas d'écran)
  ============================ */
  #bottomBar {
    position: absolute;
    bottom: calc(env(safe-area-inset-bottom, 0px) + 8px);
    left: 8px;
    right: 8px;
    display: flex;
    gap: 6px;
    justify-content: space-between;
    align-items: center;
    z-index: 15;
    pointer-events: none;
  }

  #leftBtns, #rightBtns {
    display: flex;
    gap: 6px;
    pointer-events: auto;
  }

  .btn {
    background: rgba(0,0,0,0.75);
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    border: 1px solid rgba(0,255,255,0.3);
    border-radius: 10px;
    color: #0ff;
    font-weight: bold;
    cursor: pointer;
    letter-spacing: 0.5px;
    font-size: 11px;
    padding: 10px 14px;
    transition: 0.15s;
    min-width: 48px;
    text-align: center;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    line-height: 1.1;
  }
  .btn:active {
    background: rgba(0,255,255,0.3);
    transform: scale(0.95);
  }
  .btn.off {
    color: #666;
    border-color: rgba(255,255,255,0.15);
  }
  .btn .icon { font-size: 16px; margin-bottom: 2px; }
  .btn .label { font-size: 9px; opacity: 0.8; }

  /* ============================
     ROUE EFFET (à droite, plus grosse pour le doigt)
  ============================ */
  #spinWheel {
    position: absolute;
    bottom: calc(env(safe-area-inset-bottom, 0px) + 80px);
    right: 12px;
    width: 90px; height: 90px;
    border-radius: 50%;
    background: radial-gradient(circle at 35% 35%, #fff, #ddd 40%, #999);
    box-shadow: 0 8px 24px rgba(0,0,0,0.8), inset 0 0 20px rgba(0,0,0,0.3);
    cursor: pointer;
    border: 2px solid rgba(255,255,255,0.4);
    touch-action: none;
    z-index: 15;
  }
  #spinWheel::before {
    content: '';
    position: absolute;
    inset: 8px;
    border-radius: 50%;
    border: 1px dashed rgba(0,0,0,0.3);
  }
  #spinDot {
    position: absolute;
    width: 14px; height: 14px;
    background: #d41b1b;
    border-radius: 50%;
    left: 50%; top: 50%;
    transform: translate(-50%, -50%);
    box-shadow: 0 0 8px rgba(212,27,27,0.8);
    pointer-events: none;
  }
  #spinWheel .label {
    position: absolute;
    bottom: -16px; left: 0; right: 0;
    text-align: center;
    font-size: 8px;
    color: #aaa;
    letter-spacing: 1px;
    pointer-events: none;
  }

  /* ============================
     CURSEUR DE VISÉE (tactile)
  ============================ */
  #aimHint {
    position: absolute;
    top: 50%; left: 50%;
    transform: translate(-50%, -50%);
    background: rgba(0,255,255,0.15);
    border: 2px dashed rgba(0,255,255,0.6);
    border-radius: 50%;
    width: 60px; height: 60px;
    pointer-events: none;
    opacity: 0;
    transition: opacity 0.2s;
    z-index: 5;
  }
  #aimHint.show { opacity: 1; }

  /* ============================
     MENU DE DÉMARRAGE
  ============================ */
  #menu {
    position: absolute;
    inset: 0;
    background: linear-gradient(135deg, #000 0%, #0a1a0a 100%);
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    padding: 20px;
    z-index: 100;
    text-align: center;
    gap: 15px;
  }
  #menu.hide { display: none; }
  #menu h1 {
    font-size: clamp(28px, 8vw, 44px);
    letter-spacing: 4px;
    text-shadow: 0 0 20px #0ff, 0 0 40px #0ff;
    margin-bottom: 8px;
  }
  #menu .subtitle {
    color: #888;
    font-size: 12px;
    letter-spacing: 3px;
    margin-bottom: 20px;
  }
  #menu .help {
    color: #aaa;
    font-size: 12px;
    line-height: 1.9;
    max-width: 320px;
    margin-bottom: 20px;
    text-align: left;
    padding: 14px 18px;
    background: rgba(0,255,255,0.05);
    border: 1px solid rgba(0,255,255,0.15);
    border-radius: 12px;
  }
  #menu .help b { color: #0ff; }
  #menu button {
    padding: 16px 40px;
    background: linear-gradient(180deg, #0ff, #088);
    border: none;
    border-radius: 12px;
    color: #000;
    font-weight: bold;
    font-size: 18px;
    letter-spacing: 3px;
    cursor: pointer;
    transition: 0.2s;
    box-shadow: 0 8px 30px rgba(0,255,255,0.4);
    min-width: 200px;
  }
  #menu button:active {
    transform: scale(0.96);
  }
  #menu .mode {
    display: flex;
    gap: 10px;
    margin-top: 10px;
    flex-wrap: wrap;
    justify-content: center;
  }
  #menu .mode button {
    padding: 10px 20px;
    font-size: 12px;
    letter-spacing: 2px;
    min-width: auto;
    background: rgba(0,255,255,0.1);
    color: #0ff;
    border: 1px solid rgba(0,255,255,0.3);
    box-shadow: none;
  }
  #menu .mode button.active {
    background: linear-gradient(180deg, #0ff, #088);
    color: #000;
    border: none;
  }

  /* ============================
     HINT ROTATION (portrait)
  ============================ */
  #rotate {
    display: none;
    position: absolute;
    inset: 0;
    background: #000;
    z-index: 300;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    color: #0ff;
    font-size: 16px;
    letter-spacing: 2px;
    text-align: center;
    padding: 30px;
    gap: 20px;
  }
  @media (orientation: portrait) and (max-width: 900px) {
    #rotate { display: flex; }
  }
  #rotate .icon {
    font-size: 60px;
    animation: rotate 2s ease-in-out infinite;
  }
  @keyframes rotate {
    0%, 100% { transform: rotate(0deg); }
    50% { transform: rotate(90deg); }
  }

  /* ============================
     LOADING
  ============================ */
  #loading {
    position: absolute;
    inset: 0;
    background: #000;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    z-index: 200;
    color: #0ff;
    font-size: 14px;
    letter-spacing: 4px;
    gap: 20px;
    transition: opacity 0.5s;
  }
  #loading.hide { opacity: 0; pointer-events: none; }
  #loading .spinner {
    width: 40px; height: 40px;
    border: 3px solid rgba(0,255,255,0.2);
    border-top-color: #0ff;
    border-radius: 50%;
    animation: spin 1s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }

  /* Paysage forcé : ajustements si vraiment petit */
  @media (max-height: 400px) {
    #hud { font-size: 10px; padding: 5px 10px; }
    #turnBadge { font-size: 10px; padding: 5px 10px; }
    .btn { padding: 8px 12px; font-size: 10px; }
    .btn .icon { font-size: 14px; }
    #spinWheel { width: 80px; height: 80px; bottom: 70px; }
    #menu .help { font-size: 11px; padding: 10px 14px; }
  }
</style>
</head>
<body>

<div id="loading">
  <div class="spinner"></div>
  <div>CHARGEMENT...</div>
</div>

<div id="rotate">
  <div class="icon">📱↻</div>
  <div>Tournez votre téléphone</div>
  <div style="font-size:11px;color:#666;">Le billard se joue en paysage</div>
</div>

<!-- MENU PRINCIPAL -->
<div id="menu">
  <h1>🎱 BILLARD 3D</h1>
  <div class="subtitle">VERSION MOBILE</div>

  <div class="help">
    <b>👆 Viser :</b> glisser sur la table<br>
    <b>💪 Charger :</b> glisser vers l'arrière<br>
    <b>🎯 Tirer :</b> relâcher<br>
    <b>🌀 Effet :</b> roue en bas à droite<br>
    <b>📷 Caméra :</b> 2 doigts
  </div>

  <button onclick="startGame()">▶️ JOUER</button>

  <div class="mode">
    <button id="modeAI" class="active" onclick="setMode('ai')">🤖 vs IA</button>
    <button id="mode2P" onclick="setMode('2p')">👥 2 Joueurs</button>
  </div>
</div>

<!-- HUD -->
<div id="hud">
  Score : <b id="score">0 - 0</b><br>
  Empotées : <b id="potted">0</b>/15
</div>

<div id="turnBadge">
  <span id="turnIcon">🎯</span>
  <span id="turnName">JOUEUR 1</span>
  <span class="sub" id="turnSub">À vous de jouer</span>
</div>

<!-- Barre de puissance -->
<div id="power"><div id="powerFill"></div></div>

<!-- Message central -->
<div id="msg"></div>

<!-- Indicateur visuel de toucher -->
<div id="aimHint"></div>

<!-- Roue effet -->
<div id="spinWheel">
  <div id="spinDot"></div>
  <div class="label">EFFET</div>
</div>

<!-- Barre inférieure -->
<div id="bottomBar">
  <div id="leftBtns">
    <button class="btn" onclick="toggleSound()" id="btnSound">
      <span class="icon">🔊</span>
      <span class="label">SON</span>
    </button>
    <button class="btn" onclick="resetGame()">
      <span class="icon">🔄</span>
      <span class="label">RESET</span>
    </button>
  </div>
  <div id="rightBtns">
    <button class="btn" onclick="toggleView()" id="btnView">
      <span class="icon">📷</span>
      <span class="label">VUE</span>
    </button>
    <button class="btn" onclick="toggleSpinMode()" id="btnSpin">
      <span class="icon">🎯</span>
      <span class="label">EFFET</span>
    </button>
  </div>
</div>

<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/",
    "cannon-es": "https://unpkg.com/cannon-es@0.20.0/dist/cannon-es.js"
  }
}
</script>

<script type="module">
import * as THREE from 'three';
import * as CANNON from 'cannon-es';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

/* =========================================================
   CONFIGURATION
========================================================= */
const TABLE_W = 2.24;
const TABLE_L = 4.48;
const BALL_R = 0.0286;
const POCKET_R = 0.06;
const CUSHION_H = 0.045;
const CUSHION_W = 0.04;
const BALL_MASS = 0.17;
const BALL_COLORS = [
  0xf5d000, 0x1a4fc4, 0xd41b1b, 0x5a1a8c, 0xf07d00, 0x1e7a2e, 0x8b1a1a,
  0x111111,
  0xf5d000, 0x1a4fc4, 0xd41b1b, 0x5a1a8c, 0xf07d00, 0x1e7a2e, 0x8b1a1a
];

// Détection mobile + qualité adaptative
const IS_MOBILE = /Android|iPhone|iPad|iPod|Mobile/i.test(navigator.userAgent);
const IS_LOW_END = IS_MOBILE && (navigator.hardwareConcurrency || 4) <= 4;

/* =========================================================
   SCÈNE
========================================================= */
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x050505);
scene.fog = new THREE.FogExp2(0x000000, 0.06);

const camera = new THREE.PerspectiveCamera(50, innerWidth / innerHeight, 0.1, 100);
camera.position.set(0, 3.5, 4.5);

const renderer = new THREE.WebGLRenderer({
  antialias: !IS_MOBILE,
  powerPreference: 'high-performance',
  stencil: false,
  depth: true
});
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, IS_MOBILE ? 1.25 : 2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.15;
renderer.outputColorSpace = THREE.SRGBColorSpace;
document.body.appendChild(renderer.domElement);

// Contrôles caméra (2 doigts)
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.1;
controls.minDistance = 2;
controls.maxDistance = 9;
controls.maxPolarAngle = Math.PI / 2.2;
controls.minPolarAngle = Math.PI / 6;
controls.target.set(0, 0.75, 0);
controls.enablePan = true;
controls.panSpeed = 0.5;
controls.touches = { ONE: null, TWO: THREE.TOUCH.DOLLY_ROTATE };
controls.mouseButtons = { LEFT: null, MIDDLE: THREE.MOUSE.DOLLY, RIGHT: THREE.MOUSE.ROTATE };

/* =========================================================
   LUMIÈRES (optimisées mobile)
========================================================= */
scene.add(new THREE.AmbientLight(0xffffff, 0.35));

const mainLight = new THREE.SpotLight(0xffffff, 220, 12, Math.PI / 5, 0.5, 1.5);
mainLight.position.set(0, 3.5, 0);
mainLight.castShadow = true;
mainLight.shadow.mapSize.set(1024, 1024);
mainLight.shadow.camera.near = 0.5;
mainLight.shadow.camera.far = 10;
mainLight.shadow.bias = -0.0005;
scene.add(mainLight, mainLight.target);

// 2 spots au lieu de 3 sur mobile
const spots = IS_LOW_END ? [[-1, 3, 0], [1, 3, 0]] : [[-1.2, 3.2, 0], [1.2, 3.2, 0], [0, 3.2, -1.5]];
spots.forEach(([x, y, z]) => {
  const s = new THREE.SpotLight(0xfff4d6, 50, 8, Math.PI / 6, 0.5, 1.5);
  s.position.set(x, y, z);
  s.castShadow = false; // pas d'ombre secondaire sur mobile
  scene.add(s);
});

const fillLight = new THREE.PointLight(0xffaa66, 12, 8);
fillLight.position.set(0, 1.5, 2);
scene.add(fillLight);

/* =========================================================
   TABLE
========================================================= */
const tableTopY = 0.75;
const tableGroup = new THREE.Group();
scene.add(tableGroup);

// Cadre
const woodMat = new THREE.MeshStandardMaterial({
  color: 0x3a1c08, roughness: 0.5, metalness: 0.1
});
const frameW = TABLE_W + CUSHION_W * 2 + 0.15;
const frameL = TABLE_L + CUSHION_W * 2 + 0.15;
const frameH = 0.06;

const frame = new THREE.Mesh(new THREE.BoxGeometry(frameW, frameH, frameL), woodMat);
frame.position.y = tableTopY - frameH / 2;
frame.castShadow = true; frame.receiveShadow = true;
tableGroup.add(frame);

// Bande dorée
const goldMat = new THREE.MeshStandardMaterial({ color: 0xb8860b, metalness: 0.9, roughness: 0.3 });
const trim = new THREE.Mesh(new THREE.BoxGeometry(frameW + 0.005, 0.005, frameL + 0.005), goldMat);
trim.position.y = tableTopY + 0.001;
tableGroup.add(trim);

// Tapis
const feltMat = new THREE.MeshStandardMaterial({ color: 0x0a5a2a, roughness: 0.95 });
const felt = new THREE.Mesh(new THREE.BoxGeometry(TABLE_W, 0.01, TABLE_L), feltMat);
felt.position.y = tableTopY - 0.005;
felt.receiveShadow = true;
tableGroup.add(felt);

// Coussins
const cushionMat = new THREE.MeshStandardMaterial({ color: 0x064018, roughness: 0.9 });
function addCushion(w, h, d, x, y, z) {
  const c = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), cushionMat);
  c.position.set(x, y, z);
  c.castShadow = true; c.receiveShadow = true;
  tableGroup.add(c);
}
const cy = tableTopY + CUSHION_H / 2;
addCushion(TABLE_W, CUSHION_H, CUSHION_W, 0, cy, -TABLE_L / 2 - CUSHION_W / 2);
addCushion(TABLE_W, CUSHION_H, CUSHION_W, 0, cy, TABLE_L / 2 + CUSHION_W / 2);
addCushion(CUSHION_W, CUSHION_H, TABLE_L, -TABLE_W / 2 - CUSHION_W / 2, cy, 0);
addCushion(CUSHION_W, CUSHION_H, TABLE_L, TABLE_W / 2 + CUSHION_W / 2, cy, 0);

// Poches
const pocketMat = new THREE.MeshStandardMaterial({ color: 0x000000, roughness: 1.0 });
const pocketPositions = [
  [-TABLE_W / 2, -TABLE_L / 2], [0, -TABLE_L / 2 - 0.01], [TABLE_W / 2, -TABLE_L / 2],
  [-TABLE_W / 2, TABLE_L / 2], [0, TABLE_L / 2 + 0.01], [TABLE_W / 2, TABLE_L / 2]
];
const pocketMeshes = [];
pocketPositions.forEach(([x, z]) => {
  const p = new THREE.Mesh(new THREE.CylinderGeometry(POCKET_R, POCKET_R * 0.8, 0.05, 16), pocketMat);
  p.position.set(x, tableTopY - 0.02, z);
  tableGroup.add(p);
  pocketMeshes.push({ x, z, r: POCKET_R * 0.95 });
});

// Diamants dorés
const diamondGeo = new THREE.BoxGeometry(0.012, 0.002, 0.012);
for (let i = 1; i < 8; i++) {
  const z = -TABLE_L / 2 + (TABLE_L / 8) * i;
  const d1 = new THREE.Mesh(diamondGeo, goldMat);
  d1.position.set(-TABLE_W / 2 - CUSHION_W - 0.02, tableTopY + 0.002, z);
  const d2 = d1.clone();
  d2.position.x = TABLE_W / 2 + CUSHION_W + 0.02;
  tableGroup.add(d1, d2);
}
for (let i = 1; i < 4; i++) {
  const x = -TABLE_W / 2 + (TABLE_W / 4) * i;
  const d1 = new THREE.Mesh(diamondGeo, goldMat);
  d1.position.set(x, tableTopY + 0.002, -TABLE_L / 2 - CUSHION_W - 0.02);
  const d2 = d1.clone();
  d2.position.z = TABLE_L / 2 + CUSHION_W + 0.02;
  tableGroup.add(d1, d2);
}

// Sol
const floor = new THREE.Mesh(
  new THREE.PlaneGeometry(20, 20),
  new THREE.MeshStandardMaterial({ color: 0x0a0a0a, roughness: 0.9 })
);
floor.rotation.x = -Math.PI / 2;
floor.receiveShadow = true;
scene.add(floor);

/* =========================================================
   PHYSIQUE
========================================================= */
const world = new CANNON.World({ gravity: new CANNON.Vec3(0, -9.82, 0) });
world.broadphase = new CANNON.SAPBroadphase(world);
world.allowSleep = true;
world.defaultContactMaterial.friction = 0.02;
world.defaultContactMaterial.restitution = 0.94;

const matBall = new CANNON.Material('ball');
const matFelt = new CANNON.Material('felt');
const matCushion = new CANNON.Material('cushion');

world.addContactMaterial(new CANNON.ContactMaterial(matBall, matBall, { friction: 0.05, restitution: 0.95 }));
world.addContactMaterial(new CANNON.ContactMaterial(matBall, matFelt, { friction: 0.15, restitution: 0.4 }));
world.addContactMaterial(new CANNON.ContactMaterial(matBall, matCushion, { friction: 0.1, restitution: 0.75 }));

const groundBody = new CANNON.Body({
  type: CANNON.Body.STATIC,
  shape: new CANNON.Box(new CANNON.Vec3(TABLE_W / 2, 0.005, TABLE_L / 2)),
  material: matFelt,
  position: new CANNON.Vec3(0, tableTopY - 0.005, 0)
});
world.addBody(groundBody);

const wallH = CUSHION_H * 2;
const wallY = tableTopY + wallH / 2;
function addWall(w, h, d, x, y, z) {
  const body = new CANNON.Body({
    type: CANNON.Body.STATIC,
    shape: new CANNON.Box(new CANNON.Vec3(w / 2, h / 2, d / 2)),
    material: matCushion,
    position: new CANNON.Vec3(x, y, z)
  });
  world.addBody(body);
}
addWall(TABLE_W + CUSHION_W * 2, wallH, CUSHION_W, 0, wallY, -TABLE_L / 2 - CUSHION_W / 2);
addWall(TABLE_W + CUSHION_W * 2, wallH, CUSHION_W, 0, wallY, TABLE_L / 2 + CUSHION_W / 2);
addWall(CUSHION_W, wallH, TABLE_L + CUSHION_W * 2, -TABLE_W / 2 - CUSHION_W / 2, wallY, 0);
addWall(CUSHION_W, wallH, TABLE_L + CUSHION_W * 2, TABLE_W / 2 + CUSHION_W / 2, wallY, 0);

/* =========================================================
   BILLES
========================================================= */
const ballMeshes = [];
let balls = [];
let cueBall = null;

function createBallNumberTexture(number, color) {
  const c = document.createElement('canvas');
  c.width = 128; c.height = 128;
  const g = c.getContext('2d');
  g.fillStyle = '#' + color.toString(16).padStart(6, '0');
  g.fillRect(0, 0, 128, 128);
  if (number > 0) {
    g.beginPath();
    g.arc(64, 64, 35, 0, Math.PI * 2);
    g.fillStyle = '#fff';
    g.fill();
    g.fillStyle = '#000';
    g.font = 'bold 46px Arial';
    g.textAlign = 'center';
    g.textBaseline = 'middle';
    g.fillText(number, 64, 68);
  }
  return new THREE.CanvasTexture(c);
}

function createBall(x, z, color, number = 0) {
  const geo = new THREE.SphereGeometry(BALL_R, IS_LOW_END ? 24 : 32, IS_LOW_END ? 24 : 32);
  const tex = number > 0 ? createBallNumberTexture(number, color) : null;
  const mat = new THREE.MeshStandardMaterial({
    color: number > 0 ? 0xffffff : color,
    map: tex,
    roughness: 0.1,
    metalness: 0.3,
    envMapIntensity: 1.5
  });
  const mesh = new THREE.Mesh(geo, mat);
  mesh.castShadow = true;
  mesh.receiveShadow = true;
  mesh.position.set(x, tableTopY + BALL_R, z);
  scene.add(mesh);

  const body = new CANNON.Body({
    mass: BALL_MASS,
    shape: new CANNON.Sphere(BALL_R),
    material: matBall,
    position: new CANNON.Vec3(x, tableTopY + BALL_R, z),
    linearDamping: 0.35,
    angularDamping: 0.4
  });
  body.allowSleep = true;
  body.sleepSpeedLimit = 0.05;
  body.sleepTimeLimit = 0.5;
  world.addBody(body);

  const ball = { mesh, body, number, color, potted: false, isCue: number === 0 };
  ballMeshes.push(ball);
  return ball;
}

/* =========================================================
   QUEUE 3D
========================================================= */
const cueGroup = new THREE.Group();
scene.add(cueGroup);
const cueLength = 1.45;
const cueRadius = 0.007;
const stickMat = new THREE.MeshStandardMaterial({ color: 0x8b4513, roughness: 0.4 });
const stickMesh = new THREE.Mesh(new THREE.CylinderGeometry(cueRadius * 0.6, cueRadius, cueLength, 12), stickMat);
stickMesh.rotation.z = Math.PI / 2;
stickMesh.castShadow = true;
const tipMat = new THREE.MeshStandardMaterial({ color: 0x1a4fc4, roughness: 0.6 });
const tipMesh = new THREE.Mesh(new THREE.CylinderGeometry(cueRadius * 0.6, cueRadius * 0.6, 0.012, 12), tipMat);
tipMesh.rotation.z = Math.PI / 2;
tipMesh.position.x = cueLength / 2 + 0.006;
cueGroup.add(stickMesh, tipMesh);
cueGroup.visible = false;

/* =========================================================
   ÉTAT DU JEU
========================================================= */
let currentPlayer = 1;
let scores = { 1: 0, 2: 0 };
let pottedCount = 0;
let aiEnabled = true;
let soundEnabled = true;
let aiThinking = false;
let gameOver = false;
let gameStarted = false;
let aimAngle = 0;
let power = 0;
let charging = false;
let chargeStartTime = 0;
let spinY = 0;
let spinMode = false; // quand true, la roue contrôle l'effet
let viewMode = 'top';

/* =========================================================
   AUDIO
========================================================= */
let audioCtx = null;
function initAudio() {
  if (!audioCtx) {
    try { audioCtx = new (window.AudioContext || window.webkitAudioContext)(); } catch (e) { return; }
  }
  if (audioCtx.state === 'suspended') audioCtx.resume();
}

function playCollisionSound(intensity) {
  if (!soundEnabled || !audioCtx) return;
  const t = audioCtx.currentTime;
  const vol = Math.min(0.35, intensity * 0.04);
  const bufSize = 512;
  const buffer = audioCtx.createBuffer(1, bufSize, audioCtx.sampleRate);
  const data = buffer.getChannelData(0);
  for (let i = 0; i < bufSize; i++) data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / bufSize, 4);
  const noise = audioCtx.createBufferSource();
  noise.buffer = buffer;
  const filter = audioCtx.createBiquadFilter();
  filter.type = 'bandpass';
  filter.frequency.value = 2500;
  filter.Q.value = 4;
  const gain = audioCtx.createGain();
  gain.gain.setValueAtTime(vol, t);
  gain.gain.exponentialRampToValueAtTime(0.001, t + 0.07);
  noise.connect(filter).connect(gain).connect(audioCtx.destination);
  noise.start(t); noise.stop(t + 0.09);
}

function playCueHitSound(p) {
  if (!soundEnabled || !audioCtx) return;
  const t = audioCtx.currentTime;
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.type = 'triangle';
  osc.frequency.setValueAtTime(1000, t);
  osc.frequency.exponentialRampToValueAtTime(200, t + 0.06);
  gain.gain.setValueAtTime(0.2 + p * 0.003, t);
  gain.gain.exponentialRampToValueAtTime(0.001, t + 0.08);
  osc.connect(gain).connect(audioCtx.destination);
  osc.start(t); osc.stop(t + 0.09);
}

function playPocketSound() {
  if (!soundEnabled || !audioCtx) return;
  const t = audioCtx.currentTime;
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.type = 'sine';
  osc.frequency.setValueAtTime(200, t);
  osc.frequency.exponentialRampToValueAtTime(60, t + 0.3);
  gain.gain.setValueAtTime(0.3, t);
  gain.gain.exponentialRampToValueAtTime(0.001, t + 0.3);
  osc.connect(gain).connect(audioCtx.destination);
  osc.start(t); osc.stop(t + 0.35);
}

function vibrate(ms = 30) {
  if (navigator.vibrate) navigator.vibrate(ms);
}

/* =========================================================
   INITIALISATION
========================================================= */
function clearBalls() {
  ballMeshes.forEach(b => {
    scene.remove(b.mesh);
    world.removeBody(b.body);
    b.mesh.geometry.dispose();
    b.mesh.material.dispose();
    if (b.mesh.material.map) b.mesh.material.map.dispose();
  });
  ballMeshes.length = 0;
  balls = [];
}

function initGame() {
  clearBalls();
  scores = { 1: 0, 2: 0 };
  currentPlayer = 1;
  pottedCount = 0;
  gameOver = false;
  power = 0;
  spinY = 0;

  cueBall = createBall(-TABLE_W * 0.25, 0, 0xffffff, 0);
  balls.push(cueBall);

  const startX = TABLE_W * 0.25;
  const spacing = BALL_R * 2 + 0.0002;
  const positions = [];
  for (let row = 0; row < 5; row++) {
    for (let col = 0; col <= row; col++) {
      positions.push({
        x: startX + row * spacing * 0.866,
        z: (col - row / 2) * spacing,
        row, col
      });
    }
  }
  positions.sort((a, b) => {
    const da = Math.abs(a.row - 2) + Math.abs(a.col - 1);
    const db = Math.abs(b.row - 2) + Math.abs(b.col - 1);
    return da - db;
  });
  positions[0].number = 8;

  let order = [...Array(15).keys()].map(i => i + 1).filter(n => n !== 8);
  for (let i = order.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [order[i], order[j]] = [order[j], order[i]];
  }
  let idx = 0;
  for (let i = 1; i < positions.length; i++) positions[i].number = order[idx++];

  positions.forEach(p => {
    balls.push(createBall(p.x, p.z, BALL_COLORS[p.number - 1], p.number));
  });

  updateUI();
}

/* =========================================================
   UI
========================================================= */
function updateUI() {
  document.getElementById('score').textContent = scores[1] + ' - ' + scores[2];
  document.getElementById('potted').textContent = pottedCount;

  const badge = document.getElementById('turnBadge');
  const name = document.getElementById('turnName');
  const sub = document.getElementById('turnSub');
  const icon = document.getElementById('turnIcon');

  const isMyTurn = !aiEnabled || currentPlayer === 1;
  if (isMyTurn) {
    badge.classList.remove('other');
    name.textContent = 'JOUEUR ' + currentPlayer;
    sub.textContent = 'À vous de jouer';
    icon.textContent = '🎯';
  } else {
    badge.classList.add('other');
    name.textContent = 'IA';
    sub.textContent = 'Réfléchit...';
    icon.textContent = '🤖';
  }
}

function showMsg(txt, duration = 2000) {
  const el = document.getElementById('msg');
  el.textContent = txt;
  el.classList.add('show');
  clearTimeout(el._t);
  el._t = setTimeout(() => el.classList.remove('show'), duration);
}

/* =========================================================
   BOUCLE PRINCIPALE
========================================================= */
const clock = new THREE.Clock();
let lastVel = new Map();

function animate() {
  requestAnimationFrame(animate);
  const dt = Math.min(clock.getDelta(), 0.05);

  if (gameStarted) {
    world.step(1 / 120, dt, 4);

    balls.forEach(b => {
      if (b.potted) {
        b.mesh.visible = false;
        b.body.sleep();
        return;
      }
      if (b.body.position.y < tableTopY - BALL_R * 0.3) {
        handlePocket(b);
      }
      b.mesh.position.copy(b.body.position);
      b.mesh.quaternion.copy(b.body.quaternion);
    });

    // Sécurité billes tombées
    balls.forEach(b => {
      if (b.potted) return;
      if (b.body.position.y < 0.3) {
        let closest = null, minD = Infinity;
        pocketPositions.forEach(([px, pz]) => {
          const d = Math.hypot(b.body.position.x - px, b.body.position.z - pz);
          if (d < minD) { minD = d; closest = [px, pz]; }
        });
        b.body.position.set(closest[0], tableTopY - 0.1, closest[1]);
      }
    });

    // Sons
    balls.forEach(b => {
      if (b.potted) return;
      const prev = lastVel.get(b) || { x: 0, y: 0, z: 0 };
      const cur = b.body.velocity;
      const dv = Math.hypot(cur.x - prev.x, cur.y - prev.y, cur.z - prev.z);
      if (dv > 1.8) playCollisionSound(dv);
      lastVel.set(b, { x: cur.x, y: cur.y, z: cur.z });
    });

    updateCueStick();
  }

  // Power bar UI
  const pb = document.getElementById('power');
  const pf = document.getElementById('powerFill');
  if (charging) {
    pb.classList.add('show');
    pf.style.width = power + '%';
  } else {
    pb.classList.remove('show');
  }

  controls.update();
  renderer.render(scene, camera);
}

/* =========================================================
   EMPOCHE
========================================================= */
function handlePocket(ball) {
  if (ball.potted) return;
  ball.potted = true;
  ball.mesh.visible = false;
  ball.body.sleep();
  playPocketSound();
  vibrate(30);

  if (ball.isCue) {
    setTimeout(() => {
      ball.potted = false;
      ball.body.wakeUp();
      ball.body.position.set(-TABLE_W * 0.25, tableTopY + BALL_R, 0);
      ball.body.velocity.set(0, 0, 0);
      ball.body.angularVelocity.set(0, 0, 0);
      ball.body.quaternion.set(0, 0, 0, 1);
      ball.mesh.visible = true;
    }, 700);
  } else {
    pottedCount++;
    if (ball.number === 8) {
      scores[currentPlayer] += 50;
      gameOver = true;
      showMsg(`🏆 JOUEUR ${currentPlayer} GAGNE !`, 4000);
      vibrate([100, 50, 100, 50, 100]);
    } else {
      scores[currentPlayer] += 10;
      vibrate(50);
    }
    updateUI();
  }
}

function allStopped() {
  return balls.every(b => b.potted ||
    Math.hypot(b.body.velocity.x, b.body.velocity.y, b.body.velocity.z) < 0.03);
}

/* =========================================================
   VISÉE
========================================================= */
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();
const aimPlane = new THREE.Plane(new THREE.Vector3(0, 1, 0), -tableTopY);

function getAimPoint(clientX, clientY) {
  const rect = renderer.domElement.getBoundingClientRect();
  pointer.x = ((clientX - rect.left) / rect.width) * 2 - 1;
  pointer.y = -((clientY - rect.top) / rect.height) * 2 + 1;
  raycaster.setFromCamera(pointer, camera);
  const target = new THREE.Vector3();
  raycaster.ray.intersectPlane(aimPlane, target);
  return target;
}

function updateAim(clientX, clientY) {
  if (!cueBall || cueBall.potted) return;
  const target = getAimPoint(clientX, clientY);
  if (!target) return;
  const dx = target.x - cueBall.body.position.x;
  const dz = target.z - cueBall.body.position.z;
  aimAngle = Math.atan2(dz, dx);
}

function fireCue() {
  if (!cueBall || cueBall.potted) return;
  const dirX = Math.cos(aimAngle);
  const dirZ = Math.sin(aimAngle);
  const speed = (power / 100) * 9;

  cueBall.body.wakeUp();
  cueBall.body.velocity.set(dirX * speed, 0.05 * speed * 0.3, dirZ * speed);
  cueBall.body.velocity.y += spinY * 0.3;

  playCueHitSound(power);
  vibrate(60);
  spinY = 0;
  resetSpinDot();
}

/* =========================================================
   QUEUE 3D
========================================================= */
function updateCueStick() {
  const ready = cueBall && !cueBall.potted && allStopped() && !gameOver;
  cueGroup.visible = ready && !(aiEnabled && currentPlayer === 2 && aiThinking);
  if (!ready) return;

  const dirX = Math.cos(aimAngle);
  const dirZ = Math.sin(aimAngle);
  const pullback = 0.08 + (charging ? power / 100 * 0.45 : 0);
  const startX = cueBall.body.position.x - dirX * (BALL_R + pullback);
  const startZ = cueBall.body.position.z - dirZ * (BALL_R + pullback);
  const startY = tableTopY + BALL_R + 0.01;

  cueGroup.position.set(startX - dirX * cueLength / 2, startY, startZ - dirZ * cueLength / 2);
  cueGroup.rotation.y = -aimAngle;
}

/* =========================================================
   IA
========================================================= */
function aiPlay() {
  if (gameOver || currentPlayer !== 2) return;
  aiThinking = true;
  updateUI();

  setTimeout(() => {
    const targets = balls.filter(b => !b.potted && !b.isCue && b.number !== 8);
    const eight = balls.find(b => b.number === 8 && !b.potted);
    const list = targets.length > 0 ? targets : (eight ? [eight] : []);

    let best = null;
    for (const target of list) {
      for (const pocket of pocketMeshes) {
        const tpX = pocket.x - target.body.position.x;
        const tpZ = pocket.z - target.body.position.z;
        const tpD = Math.hypot(tpX, tpZ);
        if (tpD < 0.01) continue;
        const tpNX = tpX / tpD, tpNZ = tpZ / tpD;

        const cx = target.body.position.x - tpNX * (BALL_R * 2);
        const cz = target.body.position.z - tpNZ * (BALL_R * 2);
        const cpx = cx - cueBall.body.position.x;
        const cpz = cz - cueBall.body.position.z;
        const cpD = Math.hypot(cpx, cpz);
        if (cpD < 0.01) continue;
        const cpNX = cpx / cpD, cpNZ = cpz / cpD;

        const dot = cpNX * tpNX + cpNZ * tpNZ;
        if (dot < 0.5) continue;

        let blocked = false;
        for (const other of balls) {
          if (other === cueBall || other === target || other.potted) continue;
          const vx = other.body.position.x - cueBall.body.position.x;
          const vz = other.body.position.z - cueBall.body.position.z;
          const proj = vx * cpNX + vz * cpNZ;
          if (proj < 0 || proj > cpD) continue;
          const perpX = vx - proj * cpNX;
          const perpZ = vz - proj * cpNZ;
          if (Math.hypot(perpX, perpZ) < BALL_R * 2) { blocked = true; break; }
        }
        if (blocked) continue;

        const score = dot * 100 - cpD * 30 - tpD * 20;
        if (!best || score > best.score) {
          best = { angle: Math.atan2(cpz, cpx), dist: cpD, score };
        }
      }
    }

    if (best) {
      aimAngle = best.angle;
      const desiredPower = Math.min(80, 35 + best.dist * 40);
      power = desiredPower;
      charging = true;
      chargeStartTime = performance.now() - (desiredPower / 100) * 1500;
      setTimeout(() => {
        charging = false;
        fireCue();
        aiThinking = false;
      }, 700);
    } else {
      aimAngle = Math.random() * Math.PI * 2;
      setTimeout(() => {
        power = 50;
        fireCue();
        aiThinking = false;
      }, 500);
    }
  }, 400);
}

/* =========================================================
   INPUT TACTILE + SOURIS
========================================================= */
let activePointer = null;
let pointerStart = null;
let startAimAngle = 0;

renderer.domElement.addEventListener('pointerdown', e => {
  initAudio();
  if (!gameStarted || gameOver) return;
  if (!allStopped() || cueBall.potted) return;
  if (aiEnabled && currentPlayer === 2) return;

  // Ignorer si touch multi-doigts (2 doigts = caméra)
  if (e.pointerType === 'touch' && e.isPrimary === false) return;

  // Bouton gauche uniquement (souris)
  if (e.pointerType === 'mouse' && e.button !== 0) return;

  activePointer = e.pointerId;
  pointerStart = { x: e.clientX, y: e.clientY };
  updateAim(e.clientX, e.clientY);
  startAimAngle = aimAngle;

  charging = true;
  chargeStartTime = performance.now();
  power = 0;

  // Indicateur visuel
  const hint = document.getElementById('aimHint');
  hint.style.left = e.clientX + 'px';
  hint.style.top = e.clientY + 'px';
  hint.classList.add('show');

  vibrate(10);
}, { passive: false });

renderer.domElement.addEventListener('pointermove', e => {
  if (e.pointerId !== activePointer) {
    // Aperçu visée avec souris
    if (e.pointerType === 'mouse' && gameStarted && allStopped() && !aiThinking && (!aiEnabled || currentPlayer === 1)) {
      updateAim(e.clientX, e.clientY);
    }
    return;
  }

  // Toujours viser pendant le glissement
  updateAim(e.clientX, e.clientY);

  // Sur mobile : tirer vers l'arrière = charger la puissance
  if (e.pointerType === 'touch') {
    const dx = e.clientX - pointerStart.x;
    const dy = e.clientY - pointerStart.y;
    const dist = Math.hypot(dx, dy);

    // Puissance max à 150px de glissement
    const autoPower = Math.min(100, (dist / 150) * 100);
    if (autoPower > power) power = autoPower;
  }

  // Suivre l'indicateur
  const hint = document.getElementById('aimHint');
  hint.style.left = e.clientX + 'px';
  hint.style.top = e.clientY + 'px';
}, { passive: false });

renderer.domElement.addEventListener('pointerup', e => {
  if (e.pointerId !== activePointer) return;
  activePointer = null;
  document.getElementById('aimHint').classList.remove('show');

  if (!charging) return;
  charging = false;

  if (power < 5) { power = 0; return; }
  fireCue();
});

renderer.domElement.addEventListener('pointercancel', e => {
  if (e.pointerId === activePointer) {
    activePointer = null;
    charging = false;
    power = 0;
    document.getElementById('aimHint').classList.remove('show');
  }
});

// Empêcher le comportement par défaut
renderer.domElement.addEventListener('contextmenu', e => e.preventDefault());
renderer.domElement.addEventListener('touchstart', e => {
  if (e.touches.length > 1) {
    // 2 doigts = annuler la visée
    charging = false;
    power = 0;
    activePointer = null;
    document.getElementById('aimHint').classList.remove('show');
  }
}, { passive: true });

/* =========================================================
   EFFET (molette souris + roue tactile)
========================================================= */
renderer.domElement.addEventListener('wheel', e => {
  if (!gameStarted) return;
  e.preventDefault();
  spinY = Math.max(-5, Math.min(5, spinY - e.deltaY * 0.01));
  updateSpinDotFromValues();
}, { passive: false });

const spinWheel = document.getElementById('spinWheel');
const spinDot = document.getElementById('spinDot');
let draggingSpin = false;

function updateSpinFromEvent(clientX, clientY) {
  const r = spinWheel.getBoundingClientRect();
  let x = clientX - r.left - r.width / 2;
  let y = clientY - r.top - r.height / 2;
  const maxR = r.width / 2 - 8;
  const d = Math.hypot(x, y);
  if (d > maxR) { x = x / d * maxR; y = y / d * maxR; }
  spinDot.style.left = `${50 + (x / maxR) * 50}%`;
  spinDot.style.top = `${50 + (y / maxR) * 50}%`;
  spinY = -(y / maxR) * 5;
}

function updateSpinDotFromValues() {
  const v = spinY / 5;
  spinDot.style.left = '50%';
  spinDot.style.top = `${50 - v * 40}%`;
}

function resetSpinDot() {
  spinDot.style.left = '50%';
  spinDot.style.top = '50%';
}

spinWheel.addEventListener('pointerdown', e => {
  e.stopPropagation();
  e.preventDefault();
  draggingSpin = true;
  spinWheel.setPointerCapture(e.pointerId);
  updateSpinFromEvent(e.clientX, e.clientY);
  vibrate(15);
});

spinWheel.addEventListener('pointermove', e => {
  if (draggingSpin) {
    e.stopPropagation();
    updateSpinFromEvent(e.clientX, e.clientY);
  }
});

spinWheel.addEventListener('pointerup', e => {
  draggingSpin = false;
});

spinWheel.addEventListener('pointercancel', () => {
  draggingSpin = false;
});

/* =========================================================
   CHARGE LOOP
========================================================= */
function chargeLoop() {
  if (charging && power < 100) {
    const elapsed = performance.now() - chargeStartTime;
    const computed = Math.min(100, (elapsed / 1200) * 100);
    if (computed > power) power = computed;
  }
  requestAnimationFrame(chargeLoop);
}

/* =========================================================
   DÉTECTION FIN DE TOUR
========================================================= */
setInterval(() => {
  if (!gameStarted || gameOver || aiThinking) return;
  if (!allStopped()) return;
  if (window._turnPending) return;

  window._turnPending = true;
  setTimeout(() => {
    if (allStopped() && !gameOver) {
      currentPlayer = currentPlayer === 1 ? 2 : 1;
      updateUI();
      if (aiEnabled && currentPlayer === 2) setTimeout(aiPlay, 500);
    }
    window._turnPending = false;
  }, 600);
}, 1500);

/* =========================================================
   BOUTONS UI
========================================================= */
let gameMode = 'ai';

window.setMode = function(mode) {
  gameMode = mode;
  aiEnabled = (mode === 'ai');
  document.getElementById('modeAI').classList.toggle('active', mode === 'ai');
  document.getElementById('mode2P').classList.toggle('active', mode === '2p');
};

window.startGame = function() {
  initAudio();
  vibrate(30);
  document.getElementById('menu').classList.add('hide');
  gameStarted = true;
  initGame();
  showMsg(gameMode === 'ai' ? 'Vous contre l\'IA !' : 'Joueur 1 vs Joueur 2 !');
};

window.toggleSound = function() {
  soundEnabled = !soundEnabled;
  const btn = document.getElementById('btnSound');
  btn.classList.toggle('off', !soundEnabled);
  btn.querySelector('.icon').textContent = soundEnabled ? '🔊' : '🔇';
  if (soundEnabled) initAudio();
  vibrate(20);
};

window.resetGame = function() {
  initGame();
  showMsg('Nouvelle partie !');
  vibrate(30);
};

window.toggleView = function() {
  viewMode = viewMode === 'top' ? 'angle' : 'top';
  const btn = document.getElementById('btnView');
  if (viewMode === 'top') {
    camera.position.set(0, 3.5, 4.5);
    controls.target.set(0, 0.75, 0);
    btn.querySelector('.label').textContent = 'VUE';
  } else {
    camera.position.set(2.5, 1.5, 2.5);
    controls.target.set(0, 0.75, 0);
    btn.querySelector('.label').textContent = 'ANGLE';
  }
  vibrate(20);
};

window.toggleSpinMode = function() {
  spinMode = !spinMode;
  const btn = document.getElementById('btnSpin');
  btn.classList.toggle('off', !spinMode);
  vibrate(20);
};

/* =========================================================
   RESIZE / ORIENTATION
========================================================= */
function onResize() {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
}

window.addEventListener('resize', onResize);
window.addEventListener('orientationchange', () => {
  setTimeout(onResize, 300);
});

// Empêcher le scroll/zoom
document.addEventListener('gesturestart', e => e.preventDefault());
document.addEventListener('touchmove', e => {
  if (e.target === renderer.domElement) e.preventDefault();
}, { passive: false });

/* =========================================================
   DÉMARRAGE
========================================================= */
initGame();
animate();
chargeLoop();

setTimeout(() => {
  document.getElementById('loading').classList.add('hide');
  setTimeout(() => document.getElementById('loading').remove(), 600);
}, 1000);
</script>
</body>
</html>