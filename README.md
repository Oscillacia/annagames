<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Моё портфолио</title>
<style>
  * { box-sizing: border-box; }
  body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #f5f5f7;
    color: #222;
  }
  /* Меню вкладок */
  nav {
    display: flex;
    background: #1a1a2e;
    padding: 0;
    position: sticky;
    top: 0;
    z-index: 100;
  }
  nav button {
    flex: 1;
    background: none;
    border: none;
    color: #fff;
    padding: 18px 10px;
    font-size: 16px;
    cursor: pointer;
    transition: background 0.2s;
  }
  nav button:hover { background: #2a2a4e; }
  nav button.active { background: #4a9eff; }

  /* Содержимое вкладок */
  .tab-content {
    display: none;
    padding: 30px;
    max-width: 1100px;
    margin: 0 auto;
  }
  .tab-content.active { display: block; }
</style>
</head>
<body>

<nav>
  <button class="tab-btn active" data-tab="games">🎮 Игры</button>
  <button class="tab-btn" data-tab="bots">🤖 ТГ-боты</button>
  <button class="tab-btn" data-tab="realty">🏠 Недвижимость</button>
  <button class="tab-btn" data-tab="books">📚 Книги в подарок</button>
  <button class="tab-btn" data-tab="contacts">👤 Обо мне</button>
</nav>

<!-- ================= ВКЛАДКА 1: ИГРЫ ================= -->
<section id="games" class="tab-content active">
  <h1>Игры</h1>

  <!-- Подменю для игр -->
  <div style="margin-bottom: 20px;">
    <button class="game-btn active" data-game="game1">Игра 1: Лисёнок</button>
    <button class="game-btn" data-game="game2">Игра 2 (скоро)</button>
  </div>

  <!-- Игра 1 -->
  <div id="game1" class="game-panel">
 <!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Лисёнок: приключение</title>
<style>
  body {
    margin: 0;
    background: #1a1a2e;
    display: flex;
    flex-direction: column;
    align-items: center;
    font-family: Arial, sans-serif;
    color: #fff;
    user-select: none;
  }
  canvas {
    background: linear-gradient(#87ceeb, #b8e0f0 70%);
    border: 3px solid #fff;
    margin-top: 15px;
    display: block;
    border-radius: 8px;
  }
  #hud {
    margin-top: 10px;
    font-size: 20px;
    display: flex;
    gap: 26px;
    align-items: center;
    flex-wrap: wrap;
    justify-content: center;
  }
  #hammerHud { color: #ffb74d; font-weight: bold; }
  #levelHud { color: #a5d6ff; font-weight: bold; }
  #hint {
    margin-top: 6px;
    font-size: 14px;
    opacity: 0.75;
    text-align: center;
    max-width: 820px;
    line-height: 1.5;
  }
</style>
</head>
<body>
<canvas id="game" width="800" height="500"></canvas>
<p style="margin-top: 20px; font-size: 14px; color: #888;">
    Contact me: @oscilliacia
</p>
<div id="hud">
  <div>Уровень: <span id="levelHud">1 / 3</span></div>
  <div>Очки: <span id="score">0</span></div>
  <div>Жизни: <span id="lives">❤❤❤</span></div>
  <div>Молоток: <span id="hammerHud">нет</span></div>
</div>
<div id="hint">
  ← → / A D — бежать &nbsp;|&nbsp; ↑ / W — прыжок &nbsp;|&nbsp; F — удар &nbsp;|&nbsp; R — заново<br>
  Собери <span style="color:#e74c3c">ягодки</span>, убей <span style="color:#95a5a6">волков</span> молотком и добеги до <span style="color:#2ecc71">флага</span>. Берегись оврагов!
</div>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const scoreEl = document.getElementById('score');
const livesEl = document.getElementById('lives');
const hammerEl = document.getElementById('hammerHud');
const levelEl = document.getElementById('levelHud');

const W = canvas.width;
const H = canvas.height;
const GROUND_Y = H - 40;

// ---------- Игрок ----------
const player = {
  x: 80, y: GROUND_Y,
  w: 40, h: 40,
  vx: 0, vy: 0,
  speed: 4.5,
  jumpPower: -12,
  onGround: true,
  dir: 1,
  attackTimer: 0,
  lives: 3,
  invuln: 0,
  walkPhase: 0,
  hasHammer: false
};

const GRAVITY = 0.6;

// ---------- Камера ----------
const camera = { x: 0 };

// ---------- Клавиши ----------
const keys = {};
window.addEventListener('keydown', e => {
  const k = e.key.toLowerCase();
  if (e.key === 'ArrowUp' || k === 'w' || k === 'ц') {
    e.preventDefault();
    jump();
    return;
  }
  keys[k] = true;
  if (k === 'f' || k === 'а') attack();
  if (k === 'r') restartGame();
});
window.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

function jump() {
  if (player.onGround && player.lives > 0 && !gameWon && !gameOver) {
    player.vy = player.jumpPower;
    player.onGround = false;
  }
}

// ============================================================
//                    УРОВНИ
// ============================================================
const LEVELS = [
  // ------------------- УРОВЕНЬ 1 -------------------
  {
    width: 2600,
    ground: [
      { x: 0,    w: 700 },
      { x: 820,  w: 600 },
      { x: 1540, w: 1060 }
    ],
    platforms: [
      { x: 340,  y: GROUND_Y - 110, w: 120, h: 20 },
      { x: 560,  y: GROUND_Y - 170, w: 110, h: 20 },
      { x: 940,  y: GROUND_Y - 130, w: 120, h: 20 },
      { x: 1180, y: GROUND_Y - 100, w: 120, h: 20 },
      { x: 1640, y: GROUND_Y - 140, w: 130, h: 20 },
      { x: 1900, y: GROUND_Y - 180, w: 120, h: 20 },
      { x: 2150, y: GROUND_Y - 100, w: 140, h: 20 }
    ],
    crumblePlatforms: [],
    berries: [
      { x: 400,  y: GROUND_Y - 140, taken: false },
      { x: 610,  y: GROUND_Y - 200, taken: false },
      { x: 1000, y: GROUND_Y - 160, taken: false },
      { x: 1230, y: GROUND_Y - 130, taken: false },
      { x: 1700, y: GROUND_Y - 170, taken: false },
      { x: 1960, y: GROUND_Y - 210, taken: false },
      { x: 2220, y: GROUND_Y - 130, taken: false }
    ],
    wolves: [
      { x: 1100, y: GROUND_Y, leftBound: 900, rightBound: 1350, vx: 1.6 }
    ],
    hammer: { x: 610, y: GROUND_Y - 210, taken: false },
    flag:   { x: 2520, y: GROUND_Y - 160 }
  },

  // ------------------- УРОВЕНЬ 2 -------------------
  {
    width: 3000,
    ground: [
      { x: 0,    w: 500 },
      { x: 700,  w: 400 },
      { x: 1260, w: 380 },
      { x: 1820, w: 1180 }
    ],
    platforms: [
      { x: 260,  y: GROUND_Y - 120, w: 110, h: 20 },
      { x: 800,  y: GROUND_Y - 150, w: 120, h: 20 },
      { x: 1000, y: GROUND_Y - 100, w: 110, h: 20 },
      { x: 1400, y: GROUND_Y - 160, w: 120, h: 20 },
      { x: 2000, y: GROUND_Y - 120, w: 130, h: 20 },
      { x: 2280, y: GROUND_Y - 180, w: 120, h: 20 }
    ],
    crumblePlatforms: [
      { x: 540, y: GROUND_Y - 80, w: 80, h: 16 },
      { x: 640, y: GROUND_Y - 80, w: 80, h: 16 },
      { x: 1130, y: GROUND_Y - 70, w: 80, h: 16 },
      { x: 1670, y: GROUND_Y - 90, w: 80, h: 16 },
      { x: 1760, y: GROUND_Y - 90, w: 80, h: 16 }
    ],
    berries: [
      { x: 320,  y: GROUND_Y - 150, taken: false },
      { x: 860,  y: GROUND_Y - 180, taken: false },
      { x: 1050, y: GROUND_Y - 130, taken: false },
      { x: 1460, y: GROUND_Y - 190, taken: false },
      { x: 2060, y: GROUND_Y - 150, taken: false },
      { x: 2340, y: GROUND_Y - 210, taken: false }
    ],
    wolves: [
      { x: 900,  y: GROUND_Y, leftBound: 720,  rightBound: 1080, vx: 1.8 },
      { x: 2000, y: GROUND_Y, leftBound: 1850, rightBound: 2300, vx: 2.0 }
    ],
    hammer: { x: 300, y: GROUND_Y - 150, taken: false },
    flag:   { x: 2900, y: GROUND_Y - 160 }
  },

  // ------------------- УРОВЕНЬ 3 -------------------
  {
    width: 3400,
    ground: [
      { x: 0,    w: 400 },
      { x: 600,  w: 300 },
      { x: 1050, w: 250 },
      { x: 1450, w: 250 },
      { x: 1850, w: 1550 }
    ],
    platforms: [
      { x: 200,  y: GROUND_Y - 130, w: 110, h: 20 },
      { x: 700,  y: GROUND_Y - 160, w: 120, h: 20 },
      { x: 1150, y: GROUND_Y - 120, w: 110, h: 20 },
      { x: 1550, y: GROUND_Y - 150, w: 120, h: 20 },
      { x: 2100, y: GROUND_Y - 180, w: 130, h: 20 },
      { x: 2400, y: GROUND_Y - 130, w: 120, h: 20 },
      { x: 2700, y: GROUND_Y - 100, w: 120, h: 20 }
    ],
    crumblePlatforms: [
      { x: 440,  y: GROUND_Y - 80, w: 80, h: 16 },
      { x: 940,  y: GROUND_Y - 70, w: 80, h: 16 },
      { x: 1340, y: GROUND_Y - 90, w: 80, h: 16 },
      { x: 1740, y: GROUND_Y - 80, w: 80, h: 16 },
      { x: 1970, y: GROUND_Y - 80, w: 80, h: 16 }
    ],
    berries: [
      { x: 250,  y: GROUND_Y - 160, taken: false },
      { x: 760,  y: GROUND_Y - 190, taken: false },
      { x: 1200, y: GROUND_Y - 150, taken: false },
      { x: 1600, y: GROUND_Y - 180, taken: false },
      { x: 2160, y: GROUND_Y - 210, taken: false },
      { x: 2460, y: GROUND_Y - 160, taken: false },
      { x: 2760, y: GROUND_Y - 130, taken: false }
    ],
    wolves: [
      { x: 700,  y: GROUND_Y, leftBound: 620,  rightBound: 880,  vx: 1.8 },
      { x: 1500, y: GROUND_Y, leftBound: 1470, rightBound: 1680, vx: 2.0 },
      { x: 2300, y: GROUND_Y, leftBound: 1870, rightBound: 3100, vx: 2.2 }
    ],
    hammer: { x: 250, y: GROUND_Y - 170, taken: false },
    flag:   { x: 3300, y: GROUND_Y - 160 }
  }
];
// ---------- Активное состояние ----------
let currentLevelIndex = 0;
let ground, platforms, crumblePlatforms, berries, wolves, hammer, flag, worldWidth;
const CRUMBLE_CRUMBLE_TIME = 40;    // 0.66 сек до исчезновения
const CRUMBLE_RESPAWN_TIME = 180;   // 3 сек до восстановления

function loadLevel(idx) {
  currentLevelIndex = idx;
  const src = LEVELS[idx];

  worldWidth = src.width;
  ground = src.ground.map(s => ({ ...s }));
  platforms = src.platforms.map(p => ({ ...p }));
  crumblePlatforms = src.crumblePlatforms.map(p => ({ ...p, state: 'idle', stateTimer: 0 }));
  berries = src.berries.map(b => ({ ...b }));
  wolves = src.wolves.map(w => ({ ...w, alive: true, flash: 0, walkPhase: 0 }));
  hammer = { ...src.hammer };
  flag = { ...src.flag, reached: false };

  player.x = 80;
  player.y = GROUND_Y;
  player.vy = 0;
  player.onGround = true;
  player.invuln = 60;
  player.hasHammer = false;
  player.attackTimer = 0;
  camera.x = 0;

  levelEl.textContent = (idx + 1) + ' / ' + LEVELS.length;
  updateHUD();
}

// ---------- Игровое состояние ----------
let score = 0;
let gameWon = false;
let gameOver = false;
let levelTransitionTimer = 0;

function updateHUD() {
  scoreEl.textContent = score;
  livesEl.textContent = '❤'.repeat(Math.max(0, player.lives)) || '—';
  hammerEl.textContent = player.hasHammer ? 'есть' : 'нет';
}

// ---------- Удар ----------
function attack() {
  if (player.lives <= 0 || gameWon || gameOver) return;
  player.attackTimer = 10;

  const reach = 55;
  const ax = player.dir === 1 ? player.x : player.x - reach;
  const aw = reach;
  const ay = player.y - player.h;
  const ah = player.h;

  for (const w of wolves) {
    if (!w.alive) continue;
    const wx = w.x - 30, wy = w.y - 44;
    const hit = ax < wx + 60 && ax + aw > wx &&
                ay < wy + 44 && ay + ah > wy;
    if (hit) {
      if (player.hasHammer) {
        w.alive = false;
        score += 25;
        updateHUD();
      } else {
        w.flash = 8;
      }
    }
  }
}

// ---------- Коллизии ----------
function isGroundAt(x) {
  for (const seg of ground) {
    if (x >= seg.x && x <= seg.x + seg.w) return true;
  }
  return false;
}

function tryLandOnPlatforms(prevY) {
  // Обычные платформы
  for (const p of platforms) {
    if (player.x + player.w / 2 > p.x &&
        player.x - player.w / 2 < p.x + p.w &&
        prevY <= p.y && player.y >= p.y) {
      player.y = p.y;
      player.vy = 0;
      player.onGround = true;
      return true;
    }
  }
  // Исчезающие платформы — только если они не «gone»
  for (const p of crumblePlatforms) {
    if (p.state === 'gone') continue;
    if (player.x + player.w / 2 > p.x &&
        player.x - player.w / 2 < p.x + p.w &&
        prevY <= p.y && player.y >= p.y) {
      player.y = p.y;
      player.vy = 0;
      player.onGround = true;
      if (p.state === 'idle') {
        p.state = 'crumbling';
        p.stateTimer = CRUMBLE_CRUMBLE_TIME;
      }
      return true;
    }
  }
  return false;
}

function loseLife() {
  player.lives--;
  updateHUD();
  if (player.lives <= 0) {
    gameOver = true;
    return;
  }
  // Возвращаем в начало уровня
  player.x = 80;
  player.y = GROUND_Y;
  player.vy = 0;
  player.invuln = 90;
  camera.x = 0;
}

// ---------- Обновление ----------
function update() {
  if (gameWon || gameOver) return;
  if (levelTransitionTimer > 0) {
    levelTransitionTimer--;
    return;
  }

  // Горизонтальное движение
  let dx = 0;
  if (keys['arrowleft'] || keys['a'] || keys['ф']) dx -= 1;
  if (keys['arrowright'] || keys['d'] || keys['в']) dx += 1;
  if (dx !== 0) {
    player.dir = dx;
    player.x += dx * player.speed;
    player.walkPhase += 0.3;
  }
  player.x = Math.max(player.w / 2, Math.min(worldWidth - player.w / 2, player.x));

  // Гравитация
  const prevY = player.y;
  player.vy += GRAVITY;
  player.y += player.vy;

  // Приземление на платформы
  let landed = false;
  if (player.vy > 0) {
    landed = tryLandOnPlatforms(prevY);
  }

  // Земля
  if (!landed && isGroundAt(player.x)) {
    if (player.y >= GROUND_Y) {
      player.y = GROUND_Y;
      player.vy = 0;
      player.onGround = true;
      landed = true;
    }
  }

  // Если не приземлились — либо в воздухе, либо упали в овраг
  if (!landed) {
    if (player.y >= GROUND_Y && !isGroundAt(player.x)) {
      loseLife();
      return;
    } else if (player.y < GROUND_Y) {
      player.onGround = false;
    }
  }

  if (player.attackTimer > 0) player.attackTimer--;
  if (player.invuln > 0) player.invuln--;

  // Молоток — подбор
  if (!hammer.taken) {
    const hx = hammer.x - 15, hy = hammer.y - 15;
    const px = player.x - player.w / 2, py = player.y - player.h;
    if (px < hx + 30 && px + player.w > hx &&
        py < hy + 30 && py + player.h > hy) {
      hammer.taken = true;
      player.hasHammer = true;
      updateHUD();
    }
  }

  // Ягодки — сбор
  for (const b of berries) {
    if (b.taken) continue;
    const bx = b.x - 12, by = b.y - 12;
    const px = player.x - player.w / 2, py = player.y - player.h;
    if (px < bx + 24 && px + player.w > bx &&
        py < by + 24 && py + player.h > by) {
      b.taken = true;
      score += 5;
      updateHUD();
    }
  }

  // Волки — патруль и урон
  for (const w of wolves) {
    if (!w.alive) continue;
    if (w.flash > 0) w.flash--;
    w.x += w.vx;
    w.walkPhase += 0.2;
    if (w.x < w.leftBound) { w.x = w.leftBound; w.vx *= -1; }
    if (w.x > w.rightBound) { w.x = w.rightBound; w.vx *= -1; }

    const wx = w.x - 30, wy = w.y - 44;
    const px = player.x - player.w / 2, py = player.y - player.h;
    const overlapX = px < wx + 60 && px + player.w > wx;
    const overlapY = py < wy + 44 && py + player.h > wy;
    if (overlapX && overlapY && player.invuln <= 0) {
      player.invuln = 90;
      loseLife();
      return;
    }
  }

  // Исчезающие платформы — таймеры состояния
  for (const p of crumblePlatforms) {
    if (p.state === 'crumbling') {
      p.stateTimer--;
      if (p.stateTimer <= 0) {
        p.state = 'gone';
        p.stateTimer = CRUMBLE_RESPAWN_TIME;
      }
    } else if (p.state === 'gone') {
      p.stateTimer--;
      if (p.stateTimer <= 0) {
        p.state = 'idle';
        p.stateTimer = 0;
      }
    }
  }

  // Флаг — переход на следующий уровень
  if (!flag.reached) {
    const fx = flag.x;
    const px = player.x - player.w / 2;
    if (px < fx + 30 && px + player.w > fx - 30 &&
        player.y <= GROUND_Y + 5 && isGroundAt(player.x)) {
      flag.reached = true;
      score += 100;
      updateHUD();

      if (currentLevelIndex < LEVELS.length - 1) {
        levelTransitionTimer = 60;
        setTimeout(() => {
          loadLevel(currentLevelIndex + 1);
        }, 900);
      } else {
        gameWon = true;
      }
    }
  }

  // Камера — плавно следует за игроком
  const targetCamX = player.x - W / 2;
  camera.x += (targetCamX - camera.x) * 0.12;
  camera.x = Math.max(0, Math.min(worldWidth - W, camera.x));

  updateHUD();
}
// ============================================================
//                    РИСОВАНИЕ
// ============================================================
function drawPlayer() {
  if (player.lives <= 0) return;
  // Мигание при неуязвимости
  if (player.invuln > 0 && Math.floor(player.invuln / 5) % 2 === 0) return;

  const cx = player.x;
  const groundY = player.y;
  const facing = player.dir;
  const bob = player.onGround ? Math.sin(player.walkPhase) * 1.5 : 0;

  ctx.save();
  ctx.translate(cx, groundY + bob);
  ctx.scale(facing, 1);

  // Хвост
  const tailWag = Math.sin(player.walkPhase * 0.8) * 6;
  ctx.fillStyle = '#e67e22';
  ctx.beginPath();
  ctx.moveTo(-14, -14);
  ctx.quadraticCurveTo(-34, -18 + tailWag, -30, -2 + tailWag);
  ctx.quadraticCurveTo(-24, -10, -14, -8);
  ctx.fill();
  ctx.fillStyle = '#fff3e0';
  ctx.beginPath();
  ctx.arc(-30, -6 + tailWag, 5, 0, Math.PI * 2);
  ctx.fill();

  // Тело
  ctx.fillStyle = '#f39c12';
  ctx.beginPath();
  ctx.ellipse(0, -16, 16, 14, 0, 0, Math.PI * 2);
  ctx.fill();

  // Живот
  ctx.fillStyle = '#ffe0b2';
  ctx.beginPath();
  ctx.ellipse(3, -12, 9, 9, 0, 0, Math.PI * 2);
  ctx.fill();

  // Лапки
  ctx.fillStyle = '#d35400';
  const legOff = player.onGround ? Math.sin(player.walkPhase) * 3 : 2;
  ctx.beginPath();
  ctx.ellipse(-6, -2 + legOff, 4, 3, 0, 0, Math.PI * 2);
  ctx.ellipse(6, -2 - legOff, 4, 3, 0, 0, Math.PI * 2);
  ctx.fill();

  // Голова
  ctx.fillStyle = '#f39c12';
  ctx.beginPath();
  ctx.arc(8, -28, 13, 0, Math.PI * 2);
  ctx.fill();

  // Ушки
  ctx.beginPath();
  ctx.moveTo(0, -38); ctx.lineTo(-4, -52); ctx.lineTo(8, -42);
  ctx.closePath(); ctx.fill();
  ctx.beginPath();
  ctx.moveTo(14, -38); ctx.lineTo(20, -52); ctx.lineTo(22, -38);
  ctx.closePath(); ctx.fill();
  // Внутренняя часть ушей
  ctx.fillStyle = '#c0392b';
  ctx.beginPath();
  ctx.moveTo(2, -40); ctx.lineTo(-1, -48); ctx.lineTo(6, -42);
  ctx.closePath(); ctx.fill();
  ctx.beginPath();
  ctx.moveTo(16, -40); ctx.lineTo(19, -48); ctx.lineTo(20, -40);
  ctx.closePath(); ctx.fill();

  // Мордочка
  ctx.fillStyle = '#ffe0b2';
  ctx.beginPath();
  ctx.ellipse(15, -24, 7, 6, 0, 0, Math.PI * 2);
  ctx.fill();

  // Носик
  ctx.fillStyle = '#222';
  ctx.beginPath();
  ctx.arc(20, -24, 2, 0, Math.PI * 2);
  ctx.fill();

  // Глаз
  ctx.fillStyle = '#222';
  ctx.beginPath();
  ctx.arc(12, -30, 2.2, 0, Math.PI * 2);
  ctx.fill();
  ctx.fillStyle = '#fff';
  ctx.beginPath();
  ctx.arc(12.7, -30.7, 0.9, 0, Math.PI * 2);
  ctx.fill();

  // Молоток в лапах
  if (player.hasHammer) {
    ctx.save();
    ctx.translate(14, -12);
    ctx.rotate(-0.3);
    ctx.fillStyle = '#8d6e63';
    ctx.fillRect(-2, -12, 4, 22);
    ctx.fillStyle = '#9e9e9e';
    ctx.fillRect(-9, -18, 18, 10);
    ctx.fillStyle = '#616161';
    ctx.fillRect(-9, -18, 18, 3);
    ctx.restore();
  }

  ctx.restore();

  // Дуга удара
  if (player.attackTimer > 0) {
    ctx.save();
    ctx.strokeStyle = player.hasHammer
      ? 'rgba(255,183,77,0.95)'
      : 'rgba(255, 80, 80, 0.85)';
    ctx.lineWidth = 5;
    ctx.beginPath();
    const a = player.attackTimer / 10;
    const radius = 40 + (1 - a) * 20;
    const startAng = player.dir === 1 ? -0.8 : Math.PI + 0.8;
    const endAng = player.dir === 1 ? 0.8 : Math.PI - 0.8;
    ctx.arc(player.x, player.y - player.h / 2, radius, startAng, endAng);
    ctx.stroke();
    ctx.restore();
  }
}

function drawPlatform(p) {
  // Трава сверху
  ctx.fillStyle = '#7ec850';
  ctx.fillRect(p.x, p.y, p.w, 6);
  // Земля
  ctx.fillStyle = '#8d6e63';
  ctx.fillRect(p.x, p.y + 6, p.w, p.h - 6);
}

function drawCrumblePlatform(p) {
  if (p.state === 'gone') {
    // Призрак — еле видный контур
    ctx.strokeStyle = 'rgba(255,255,255,0.2)';
    ctx.setLineDash([4, 4]);
    ctx.strokeRect(p.x, p.y, p.w, p.h);
    ctx.setLineDash([]);
    return;
  }
  if (p.state === 'crumbling') {
    // Трясётся и мигает
    const shake = (Math.random() - 0.5) * 4;
    const alpha = 0.5 + 0.5 * Math.abs(Math.sin(p.stateTimer * 0.5));
    ctx.globalAlpha = alpha;
    ctx.fillStyle = '#c0392b';
    ctx.fillRect(p.x + shake, p.y, p.w, p.h);
    ctx.globalAlpha = 1;
    // Трещинки
    ctx.strokeStyle = '#7b241c';
    ctx.lineWidth = 1.5;
    ctx.beginPath();
    ctx.moveTo(p.x + p.w * 0.3 + shake, p.y);
    ctx.lineTo(p.x + p.w * 0.4 + shake, p.y + p.h);
    ctx.moveTo(p.x + p.w * 0.7 + shake, p.y);
    ctx.lineTo(p.x + p.w * 0.6 + shake, p.y + p.h);
    ctx.stroke();
  } else {
    // Обычный вид — «ненадёжная» с пунктиром
    ctx.fillStyle = '#e67e22';
    ctx.fillRect(p.x, p.y, p.w, p.h);
    ctx.fillStyle = '#a04000';
    ctx.fillRect(p.x, p.y, p.w, 4);
    ctx.strokeStyle = 'rgba(255,255,255,0.5)';
    ctx.setLineDash([6, 4]);
    ctx.beginPath();
    ctx.moveTo(p.x + 4, p.y + p.h / 2);
    ctx.lineTo(p.x + p.w - 4, p.y + p.h / 2);
    ctx.stroke();
    ctx.setLineDash([]);
  }
}

function drawBerry(b) {
  if (b.taken) return;
  const x = b.x, y = b.y;

  // Свечение
  ctx.save();
  ctx.shadowColor = '#ff5252';
  ctx.shadowBlur = 15;
  ctx.fillStyle = '#e74c3c';
  ctx.beginPath();
  ctx.arc(x, y, 9, 0, Math.PI * 2);
  ctx.fill();
  ctx.restore();

  // Блик
  ctx.fillStyle = 'rgba(255,255,255,0.6)';
  ctx.beginPath();
  ctx.arc(x - 3, y - 3, 3, 0, Math.PI * 2);
  ctx.fill();

  // Листик
  ctx.fillStyle = '#27ae60';
  ctx.beginPath();
  ctx.ellipse(x, y - 10, 5, 3, -0.4, 0, Math.PI * 2);
  ctx.fill();
}

function drawHammerPickup() {
  if (hammer.taken) return;
  const x = hammer.x;
  const y = hammer.y + Math.sin(Date.now() / 250) * 4;

  // Свечение
  ctx.save();
  ctx.shadowColor = '#ffb74d';
  ctx.shadowBlur = 20;
  ctx.fillStyle = '#ffb74d';
  ctx.beginPath();
  ctx.arc(x, y, 22, 0, Math.PI * 2);
  ctx.fill();
  ctx.restore();

  // Сам молоток
  ctx.save();
  ctx.translate(x, y);
  ctx.rotate(Math.sin(Date.now() / 400) * 0.15);
  ctx.fillStyle = '#8d6e63';
  ctx.fillRect(-2, -8, 4, 26);
  ctx.fillStyle = '#9e9e9e';
  ctx.fillRect(-10, -16, 20, 10);
  ctx.fillStyle = '#616161';
  ctx.fillRect(-10, -16, 20, 3);
  ctx.restore();
}
function drawWolf(w) {
  if (!w.alive) return;
  const x = w.x, y = w.y;
  const flip = w.vx < 0 ? -1 : 1;
  const legSwing = Math.sin(w.walkPhase) * 3;

  ctx.save();
  ctx.translate(x, y);
  ctx.scale(flip, 1);

  // Тело
  ctx.fillStyle = w.flash > 0 ? '#fff' : '#7f8c8d';
  ctx.beginPath();
  ctx.ellipse(0, -22, 26, 14, 0, 0, Math.PI * 2);
  ctx.fill();

  // Живот
  ctx.fillStyle = '#bdc3c7';
  ctx.beginPath();
  ctx.ellipse(2, -18, 14, 8, 0, 0, Math.PI * 2);
  ctx.fill();

  // Лапы
  ctx.fillStyle = '#5d6d70';
  ctx.beginPath();
  ctx.ellipse(-14, -4 + legSwing, 4, 5, 0, 0, Math.PI * 2);
  ctx.ellipse(-4, -4 - legSwing, 4, 5, 0, 0, Math.PI * 2);
  ctx.ellipse(8, -4 + legSwing, 4, 5, 0, 0, Math.PI * 2);
  ctx.ellipse(18, -4 - legSwing, 4, 5, 0, 0, Math.PI * 2);
  ctx.fill();

  // Хвост
  ctx.fillStyle = '#7f8c8d';
  ctx.beginPath();
  ctx.moveTo(-22, -24);
  ctx.quadraticCurveTo(-38, -30, -34, -14);
  ctx.quadraticCurveTo(-28, -20, -22, -22);
  ctx.fill();

  // Голова
  ctx.fillStyle = '#7f8c8d';
  ctx.beginPath();
  ctx.arc(20, -30, 13, 0, Math.PI * 2);
  ctx.fill();

  // Ушки
  ctx.beginPath();
  ctx.moveTo(12, -40); ctx.lineTo(10, -52); ctx.lineTo(20, -42);
  ctx.closePath(); ctx.fill();
  ctx.beginPath();
  ctx.moveTo(24, -42); ctx.lineTo(28, -54); ctx.lineTo(30, -40);
  ctx.closePath(); ctx.fill();

  // Морда
  ctx.fillStyle = '#95a5a6';
  ctx.beginPath();
  ctx.ellipse(30, -26, 9, 6, 0, 0, Math.PI * 2);
  ctx.fill();

  // Нос
  ctx.fillStyle = '#222';
  ctx.beginPath();
  ctx.arc(37, -26, 2, 0, Math.PI * 2);
  ctx.fill();

  // Глаз
  ctx.fillStyle = '#f1c40f';
  ctx.beginPath();
  ctx.arc(24, -32, 2.5, 0, Math.PI * 2);
  ctx.fill();
  ctx.fillStyle = '#000';
  ctx.beginPath();
  ctx.arc(24.5, -32, 1.4, 0, Math.PI * 2);
  ctx.fill();

  // Зубы (оскал)
  ctx.fillStyle = '#fff';
  ctx.beginPath();
  ctx.moveTo(32, -22);
  ctx.lineTo(34, -19);
  ctx.lineTo(35, -22);
  ctx.lineTo(37, -19);
  ctx.lineTo(38, -22);
  ctx.closePath();
  ctx.fill();

  ctx.restore();
}

function drawFlag() {
  const x = flag.x, y = flag.y;
  // Шест
  ctx.fillStyle = '#8d6e63';
  ctx.fillRect(x, y, 8, flag.h || 160);
  // Флаг
  if (!flag.reached) {
    const wave = Math.sin(Date.now() / 200) * 4;
    ctx.fillStyle = '#2ecc71';
    ctx.beginPath();
    ctx.moveTo(x + 8, y + 10);
    ctx.lineTo(x + 70, y + 25 + wave);
    ctx.lineTo(x + 8, y + 45);
    ctx.closePath();
    ctx.fill();
    // Звёздочка
    ctx.fillStyle = '#fff';
    ctx.font = 'bold 18px Arial';
    ctx.fillText('★', x + 24, y + 33);
  } else {
    ctx.fillStyle = '#f39c12';
    ctx.fillRect(x + 8, y + 10, 60, 30);
  }
}

function drawBackground() {
  // Небо
  ctx.fillStyle = '#87ceeb';
  ctx.fillRect(0, 0, W, H);

  // Солнце (статично)
  ctx.fillStyle = '#ffe066';
  ctx.beginPath();
  ctx.arc(W - 90, 70, 38, 0, Math.PI * 2);
  ctx.fill();

  // Облака с параллаксом
  ctx.fillStyle = 'rgba(255,255,255,0.85)';
  const cloudOffset = -camera.x * 0.3;
  const clouds = [
    [150, 80], [500, 110], [900, 70], [1300, 100], [1700, 90]
  ];
  for (const [cx0, cy0] of clouds) {
    const cx = ((cx0 + cloudOffset) % (W + 300) + (W + 300)) % (W + 300) - 150;
    ctx.beginPath();
    ctx.arc(cx, cy0, 22, 0, Math.PI * 2);
    ctx.arc(cx + 26, cy0, 28, 0, Math.PI * 2);
    ctx.arc(cx + 55, cy0, 20, 0, Math.PI * 2);
    ctx.fill();
  }

  // Далёкие горы (параллакс 0.5)
  ctx.fillStyle = '#a8d5a2';
  const hillOffset = -camera.x * 0.5;
  for (let i = 0; i < 8; i++) {
    const hx = i * 500 + (hillOffset % 500) - 200;
    ctx.beginPath();
    ctx.moveTo(hx, GROUND_Y);
    ctx.lineTo(hx + 150, GROUND_Y - 130);
    ctx.lineTo(hx + 300, GROUND_Y);
    ctx.closePath();
    ctx.fill();
  }
}

function drawWorldGround() {
  // Земля по сегментам (с оврагами)
  for (const seg of ground) {
    // Трава
    ctx.fillStyle = '#7ec850';
    ctx.fillRect(seg.x, GROUND_Y, seg.w, H - GROUND_Y + 40);
    // Верхняя полоска
    ctx.fillStyle = '#5da838';
    ctx.fillRect(seg.x, GROUND_Y, seg.w, 6);
  }
}

function drawProgressBar() {
  const barW = W - 40, barH = 6;
  const bx = 20, by = 12;
  ctx.fillStyle = 'rgba(0,0,0,0.25)';
  ctx.fillRect(bx, by, barW, barH);
  ctx.fillStyle = '#f39c12';
  ctx.fillRect(bx, by, barW * (player.x / worldWidth), barH);
  // Метка флага
  ctx.fillStyle = '#2ecc71';
  const flagPos = bx + barW * (flag.x / worldWidth);
  ctx.fillRect(flagPos - 1, by - 3, 3, barH + 6);
}

function drawLevelBanner() {
  // Показываем название уровня на старте (первые ~2 сек)
  if (levelTransitionTimer > 0) {
    ctx.fillStyle = 'rgba(0,0,0,0.5)';
    ctx.fillRect(0, H / 2 - 60, W, 120);
    ctx.fillStyle = '#fff';
    ctx.textAlign = 'center';
    ctx.font = 'bold 44px Arial';
    ctx.fillText('Уровень ' + (currentLevelIndex + 1), W / 2, H / 2 + 15);
    ctx.textAlign = 'left';
  }
}

function drawEndScreens() {
  if (gameWon) {
    ctx.fillStyle = 'rgba(46, 204, 113, 0.78)';
    ctx.fillRect(0, 0, W, H);
    ctx.fillStyle = '#fff';
    ctx.textAlign = 'center';
    ctx.font = 'bold 52px Arial';
    ctx.fillText('🏁 Победа!', W / 2, H / 2 - 30);
    ctx.font = '24px Arial';
    ctx.fillText('Все уровни пройдены!', W / 2, H / 2 + 15);
    ctx.font = '22px Arial';
    ctx.fillText('Очки: ' + score, W / 2, H / 2 + 55);
    ctx.font = '18px Arial';
    ctx.fillText('Нажми R, чтобы сыграть заново', W / 2, H / 2 + 95);
    ctx.textAlign = 'left';
  }
  if (gameOver) {
    ctx.fillStyle = 'rgba(0,0,0,0.65)';
    ctx.fillRect(0, 0, W, H);
    ctx.fillStyle = '#fff';
    ctx.textAlign = 'center';
    ctx.font = 'bold 52px Arial';
    ctx.fillText('Игра окончена', W / 2, H / 2 - 30);
    ctx.font = '24px Arial';
    ctx.fillText('Очки: ' + score, W / 2, H / 2 + 20);
    ctx.font = '18px Arial';
    ctx.fillText('Нажми R, чтобы начать заново', W / 2, H / 2 + 60);
    ctx.textAlign = 'left';
  }
}

function restartGame() {
  player.lives = 3;
  score = 0;
  gameWon = false;
  gameOver = false;
  levelTransitionTimer = 0;
  loadLevel(0);
}

// ---------- Главный цикл ----------
function loop() {
  update();

  // Фон (небо, солнце, облака, горы) — не зависит от камеры
  drawBackground();

  // Мир — со сдвигом камеры
  ctx.save();
  ctx.translate(-Math.floor(camera.x), 0);

  drawWorldGround();
  platforms.forEach(drawPlatform);
  crumblePlatforms.forEach(drawCrumblePlatform);
  drawHammerPickup();
  berries.forEach(drawBerry);
  drawFlag();
  wolves.forEach(drawWolf);
  drawPlayer();

  ctx.restore();

  // Поверх мира — прогресс-бар и баннеры
  drawProgressBar();
  drawLevelBanner();

  // Финальные экраны поверх всего
  drawEndScreens();

  requestAnimationFrame(loop);
}

// ---------- Старт ----------
loadLevel(0);
updateHUD();
loop();
</script>
</body>
</html>
  </div>

  <!-- Игра 2 -->
  <div id="game2" class="game-panel" style="display:none;">
    <p>Здесь будет вторая игра. Пока в разработке 🚧</p>
    <<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Цветочные пазлы</title>
<style>
  * { box-sizing: border-box; }
  body {
    margin: 0; font-family: 'Segoe UI', sans-serif;
    background: radial-gradient(circle at 50% 30%, #2a2a40, #12121f);
    color: #fff; height: 100vh; display: flex; flex-direction: column;
    user-select: none; overflow: hidden;
  }
  .topbar {
    display: flex; align-items: center; justify-content: center; gap: 18px;
    padding: 10px; background: #16162a; border-bottom: 2px solid #2a2a40;
    flex-wrap: wrap; font-size: 14px;
  }
  .legend { display: flex; align-items: center; gap: 8px; }
  .swatch { width: 24px; height: 24px; border-radius: 50%; border: 2px solid #555; }
  .arrow { color: #888; font-size: 16px; }
  .hearts { font-size: 18px; letter-spacing: 2px; }
  .heart-lost { opacity: .25; }
  .moves { color: #9ad4ff; }

  .game { flex: 1; display: flex; min-height: 0; }
  .left { flex: 1; display: flex; align-items: center; justify-content: center; }
  #flower { max-width: 100%; max-height: 100%; }

  .right {
    width: 340px; background: #16162a; padding: 16px; overflow-y: auto;
    border-left: 2px solid #2a2a40;
  }
  .right h2 { margin: 0 0 6px; font-size: 17px; }
  .target-block {
    background: #1c1c30; border: 2px solid #33334f; border-radius: 14px;
    padding: 10px; margin-bottom: 14px;
  }
  .target-row { display: flex; align-items: center; gap: 12px; }
  #targetFlower { flex-shrink: 0; }
  .target-text { font-size: 12px; color: #b6bbd4; line-height: 1.5; }
  .hint { font-size: 12px; color: #9aa0b5; margin-bottom: 12px; line-height: 1.4; }
  .puzzle-grid { display: flex; flex-direction: column; gap: 10px; }
  .puzzle {
    background: #222238; border: 2px solid #33334f; border-radius: 12px;
    padding: 8px; cursor: pointer; transition: all .15s;
    display: flex; align-items: center; gap: 10px;
  }
  .puzzle:hover { border-color: #4cafef; background: #28284a; transform: translateX(-3px); }
  .puzzle:active { transform: scale(.97); }
  .puzzle.disabled { opacity: .35; pointer-events: none; }
  .puzzle.reset-puzzle { border-color: #b8860b; }
  .puzzle.reset-puzzle:hover { border-color: #ffd166; }
  .puzzle .label { font-size: 13px; color: #cfd3e6; }

  .status { margin-top: 14px; padding-top: 12px; border-top: 1px solid #2a2a40; font-size: 14px; }
  .status .win { color: #6ee7a0; font-weight: bold; }
  .status .lose { color: #ff7b7b; font-weight: bold; }

  button {
    margin-top: 10px; width: 100%; padding: 10px; cursor: pointer;
    border: none; border-radius: 8px; background: #4cafef; color: #fff;
    font-size: 15px; font-weight: 600;
  }
  button:hover { background: #3a9bd8; }

  .screen {
    position: fixed; inset: 0; display: none;
    align-items: center; justify-content: center;
    background: rgba(10, 10, 20, 0.85); backdrop-filter: blur(6px);
    z-index: 100;
  }
  .screen.show { display: flex; }
  .panel {
    background: #1c1c30; border: 2px solid #33334f; border-radius: 20px;
    padding: 36px 44px; text-align: center; max-width: 500px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.6);
    animation: pop .25s ease-out;
  }
  @keyframes pop { from { transform: scale(.85); opacity: 0; } to { transform: scale(1); opacity: 1; } }
  .panel h1 { margin: 0 0 10px; font-size: 28px; }
  .panel p { margin: 0 0 18px; color: #b6bbd4; line-height: 1.5; font-size: 14px; }
  .panel .big-emoji { font-size: 50px; margin-bottom: 8px; }
  .panel button { margin-top: 6px; padding: 12px; font-size: 16px; }
  .panel .secondary { background: #2a2a40; color: #cfd3e6; margin-top: 8px; }
  .panel .secondary:hover { background: #34345a; }
</style>
</head>
<body>
<!-- ВЕРХ: легенда + сердечки + ходы -->
<div class="topbar">
  <div class="legend"><b>Цепочка цвета:</b></div>
  <div class="legend">
    <div class="swatch" style="background:#ffffff"></div>белый
    <span class="arrow">→</span>
    <div class="swatch" style="background:#e74c3c"></div>красный
    <span class="arrow">→</span>
    <div class="swatch" style="background:#2ecc71"></div>зелёный ✅
    <span class="arrow">→</span>
    <div class="swatch" style="background:#f39c12"></div>оранжевый ⚠️
  </div>
  <div class="legend">
    <span class="hearts" id="hearts">❤️❤️❤️</span>
  </div>
  <div class="legend" style="color:#9aa0b5">
    Уровень: <span id="lvlLabel">1</span> &nbsp;|&nbsp;
    Ходы: <span class="moves" id="movesLabel">0 / 14</span>
  </div>
</div>

<!-- ОСНОВНОЕ -->
<div class="game">
  <!-- ЛЕВО: цветок -->
  <div class="left">
    <canvas id="flower" width="620" height="620"></canvas>
  </div>

  <!-- ПРАВО: цель + пазлы -->
  <div class="right">
    <div class="target-block">
      <h2>Цель</h2>
      <div class="target-row">
        <canvas id="targetFlower" width="120" height="120"></canvas>
        <div class="target-text">
          Доведи <b>все лепестки</b> до зелёного,<br>
          не потеряв все сердечки и уложившись в ходы.<br>
          Лишний клик по зелёному = оранжевый (ошибка).
        </div>
      </div>
    </div>

    <h2>Выбери пазл</h2>
    <p class="hint">Наведи на пазл — лепестки на цветке подсветятся. Один клик = один шаг по цвету. Пазл «Сброс» откатывает все лепестки на шаг назад.</p>
    <div class="puzzle-grid" id="puzzles"></div>

    <div class="status" id="status"></div>
    <button id="restart">Заново</button>
  </div>
</div>

<!-- СТАРТОВЫЙ ЭКРАН -->
<div class="screen show" id="startScreen">
  <div class="panel">
    <div class="big-emoji">🌸</div>
    <h1>Цветочные пазлы</h1>
    <p>Выбирай фигуры-пазлы справа — они нажимают лепестки цветка слева.<br>
       Каждое нажатие двигает лепесток по цепочке:<br>
       <b>белый → красный → зелёный</b>.<br>
       Нажатие по уже зелёному делает его <b>оранжевым</b> — это ошибка (−1 ❤️).<br>
       Оранжевый можно починить ещё одним нажатием (вернётся в зелёный).<br>
       Пазл <b>«Сброс»</b> откатывает все лепестки на шаг назад — ошибку не тратит, но тратит ход.<br>
       Докрась все лепестки до зелёного, сохранив сердечки и уложившись в ходы!</p>
    <button id="startBtn">Играть</button>
  </div>
</div>

<!-- ЭКРАН ПОБЕДЫ -->
<div class="screen" id="winScreen">
  <div class="panel">
    <div class="big-emoji">🎉</div>
    <h1>Уровень пройден!</h1>
    <p id="winText">Цветок полностью окрашен.</p>
    <button id="nextBtn">Следующий уровень →</button>
    <button id="replayBtn" class="secondary">Пройти заново</button>
  </div>
</div>

<!-- ЭКРАН ПРОИГРЫША -->
<div class="screen" id="loseScreen">
  <div class="panel">
    <div class="big-emoji">🥀</div>
    <h1 id="loseTitle">Не получилось!</h1>
    <p id="loseText">Попробуй ещё раз.</p>
    <button id="retryBtn">Повторить уровень</button>
  </div>
</div>
<script>
/* ================= НАСТРОЙКИ УРОВНЕЙ ================= */
// Лепестки по уровням (индекс = уровень)
const PETALS_BY_LEVEL   = [0, 5, 5, 6, 6, 7, 7, 7];
// Сердечки (жизни) по уровням
const HEARTS_BY_LEVEL   = [0, 3, 3, 3, 2, 2, 2, 2];
// Лимит ходов по уровням
const MOVES_BY_LEVEL    = [0, 14, 16, 20, 24, 28, 32, 36];

const COLORS = ['#ffffff', '#e74c3c', '#2ecc71', '#f39c12'];
const GREEN = 2;

function petalCountForLevel(lvl) { return PETALS_BY_LEVEL[Math.min(lvl, PETALS_BY_LEVEL.length - 1)]; }
function heartsForLevel(lvl)     { return HEARTS_BY_LEVEL[Math.min(lvl, HEARTS_BY_LEVEL.length - 1)]; }
function movesForLevel(lvl)      { return MOVES_BY_LEVEL[Math.min(lvl, MOVES_BY_LEVEL.length - 1)]; }

/* ================= СБАЛАНСИРОВАННЫЕ НАБОРЫ ПАЗЛОВ ================= */
// Для 5 лепестков: каждый лепесток встречается 2–3 раза
const PUZZLES_5 = [
  { petals: [0, 1],    name: '1-2' },
  { petals: [1, 2],    name: '2-3' },
  { petals: [3, 4],    name: '4-5' },
  { petals: [2, 4],    name: '3-5' },
  { petals: [0, 2, 4], name: '1-3-5' },
  { petals: [1, 3],    name: '2-4' },
];

// Для 6 лепестков
const PUZZLES_6 = [
  { petals: [0, 1],    name: '1-2' },
  { petals: [1, 2],    name: '2-3' },
  { petals: [2, 3],    name: '3-4' },
  { petals: [3, 4],    name: '4-5' },
  { petals: [4, 5],    name: '5-6' },
  { petals: [0, 5],    name: '1-6' },
  { petals: [0, 2, 4], name: '1-3-5' },
  { petals: [1, 3, 5], name: '2-4-6' },
];

// Для 7 лепестков
const PUZZLES_7 = [
  { petals: [0, 1],    name: '1-2' },
  { petals: [1, 2],    name: '2-3' },
  { petals: [2, 3],    name: '3-4' },
  { petals: [3, 4],    name: '4-5' },
  { petals: [4, 5],    name: '5-6' },
  { petals: [5, 6],    name: '6-7' },
  { petals: [0, 6],    name: '1-7' },
  { petals: [0, 3, 6], name: '1-4-7' },
  { petals: [1, 3, 5], name: '2-4-6' },
  { petals: [0, 2, 4], name: '1-3-5' },
];

function puzzlesForLevel(lvl) {
  const n = petalCountForLevel(lvl);
  if (n <= 5) return PUZZLES_5;
  if (n === 6) return PUZZLES_6;
  return PUZZLES_7;
}

/* ================= СОСТОЯНИЕ ================= */
let LEVEL = 1;
let PETAL_COUNT = 5;
let MAX_HEARTS = 3;
let MAX_MOVES = 14;
let stage = [];
let hearts = 3;
let movesLeft = 14;
let gameActive = false;
let hoveredPetals = null; // для подсветки

/* ================= ЦВЕТОК ================= */
const canvas = document.getElementById('flower');
const ctx = canvas.getContext('2d');
const CX = canvas.width / 2, CY = canvas.height / 2;
const R_PETAL = 110, PETAL_R = 78;

function petalPos(i, count, cx, cy, rp) {
  const angle = (Math.PI * 2 * i) / count - Math.PI / 2;
  return { x: cx + Math.cos(angle) * rp, y: cy + Math.sin(angle) * rp };
}

function drawFlower() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  for (let i = 0; i < PETAL_COUNT; i++) {
    const p = petalPos(i, PETAL_COUNT, CX, CY, R_PETAL);
    const c = COLORS[stage[i]];
    const highlighted = hoveredPetals && hoveredPetals.includes(i);

    ctx.beginPath();
    ctx.arc(p.x, p.y, PETAL_R + 3, 0, Math.PI * 2);
    ctx.fillStyle = 'rgba(0,0,0,0.35)';
    ctx.fill();

    ctx.beginPath();
    ctx.arc(p.x, p.y, PETAL_R, 0, Math.PI * 2);
    ctx.fillStyle = c;
    ctx.fill();
    ctx.lineWidth = highlighted ? 6 : 3;
    ctx.strokeStyle = highlighted ? '#ffd166'
                    : (stage[i] === GREEN ? '#6ee7a0' : 'rgba(255,255,255,0.25)');
    ctx.stroke();

    ctx.fillStyle = 'rgba(0,0,0,0.55)';
    ctx.font = 'bold 22px sans-serif';
    ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
    ctx.fillText(i + 1, p.x, p.y);
  }
  ctx.beginPath();
  ctx.arc(CX, CY, 52, 0, Math.PI * 2);
  ctx.fillStyle = '#ffd166'; ctx.fill();
  ctx.lineWidth = 4; ctx.strokeStyle = '#b8860b'; ctx.stroke();
}

/* ================= ОБРАЗЕЦ ЦЕЛИ ================= */
const targetCanvas = document.getElementById('targetFlower');
const tctx = targetCanvas.getContext('2d');

function drawTargetFlower() {
  const W = targetCanvas.width, H = targetCanvas.height;
  tctx.clearRect(0, 0, W, H);
  const cx = W / 2, cy = H / 2;
  const rp = 34, pr = 22;

  tctx.beginPath();
  tctx.arc(cx, cy, W / 2 - 2, 0, Math.PI * 2);
  tctx.fillStyle = '#0f0f1a';
  tctx.fill();

  for (let i = 0; i < PETAL_COUNT; i++) {
    const p = petalPos(i, PETAL_COUNT, cx, cy, rp);
    tctx.beginPath();
    tctx.arc(p.x, p.y, pr, 0, Math.PI * 2);
    tctx.fillStyle = COLORS[GREEN];
    tctx.fill();
    tctx.lineWidth = 1.5;
    tctx.strokeStyle = 'rgba(255,255,255,0.25)';
    tctx.stroke();
  }
  tctx.beginPath();
  tctx.arc(cx, cy, 16, 0, Math.PI * 2);
  tctx.fillStyle = '#ffd166'; tctx.fill();
  tctx.lineWidth = 2; tctx.strokeStyle = '#b8860b'; tctx.stroke();
}

/* ================= МИНИ-ИКОНКИ ПАЗЛОВ ================= */
function makeMini(petals) {
  const size = 56;
  const c = document.createElement('canvas');
  c.width = size; c.height = size;
  const g = c.getContext('2d');
  const cx = size / 2, cy = size / 2, rp = 19, pr = 11;
  for (let i = 0; i < PETAL_COUNT; i++) {
    const p = petalPos(i, PETAL_COUNT, cx, cy, rp);
    g.beginPath();
    g.arc(p.x, p.y, pr, 0, Math.PI * 2);
    g.fillStyle = '#3a3a55'; g.fill();
  }
  petals.forEach(i => {
    if (i >= PETAL_COUNT) return;
    const p = petalPos(i, PETAL_COUNT, cx, cy, rp);
    g.beginPath();
    g.arc(p.x, p.y, pr, 0, Math.PI * 2);
    g.fillStyle = '#4cafef'; g.fill();
  });
  return c;
}

function makeResetMini() {
  const size = 56;
  const c = document.createElement('canvas');
  c.width = size; c.height = size;
  const g = c.getContext('2d');
  g.clearRect(0, 0, size, size);
  g.font = 'bold 30px sans-serif';
  g.textAlign = 'center'; g.textBaseline = 'middle';
  g.fillStyle = '#ffd166';
  g.fillText('↺', size / 2, size / 2 + 2);
  return c;
}

/* ================= ПАНЕЛЬ ПАЗЛОВ ================= */
const puzzlesEl = document.getElementById('puzzles');

function renderPuzzles() {
  puzzlesEl.innerHTML = '';
  const list = puzzlesForLevel(LEVEL);

  list.forEach(p => {
    const div = document.createElement('div');
    div.className = 'puzzle';
    div.appendChild(makeMini(p.petals));
    const label = document.createElement('div');
    label.className = 'label';
    label.innerHTML = '<b>' + p.petals.map(i => i + 1).join(', ') +
      '</b><br><span style="color:#9aa0b5">' + p.name + '</span>';
    div.appendChild(label);

    div.addEventListener('mouseenter', () => {
      hoveredPetals = p.petals.filter(i => i < PETAL_COUNT);
      drawFlower();
    });
    div.addEventListener('mouseleave', () => {
      hoveredPetals = null;
      drawFlower();
    });
    div.addEventListener('click', () => { if (gameActive) usePuzzle(p.petals); });
    puzzlesEl.appendChild(div);
  });

  // Пазл «Сброс»
  const resetDiv = document.createElement('div');
  resetDiv.className = 'puzzle reset-puzzle';
  resetDiv.appendChild(makeResetMini());
  const rlabel = document.createElement('div');
  rlabel.className = 'label';
  rlabel.innerHTML = '<b>Сброс</b><br><span style="color:#9aa0b5">−1 шаг всем</span>';
  resetDiv.appendChild(rlabel);

  resetDiv.addEventListener('mouseenter', () => {
    hoveredPetals = [];
    for (let i = 0; i < PETAL_COUNT; i++) hoveredPetals.push(i);
    drawFlower();
  });
  resetDiv.addEventListener('mouseleave', () => {
    hoveredPetals = null;
    drawFlower();
  });
  resetDiv.addEventListener('click', () => { if (gameActive) useReset(); });
  puzzlesEl.appendChild(resetDiv);
}

/* ================= ИНТЕРФЕЙС ================= */
function updateHearts() {
  const el = document.getElementById('hearts');
  let s = '';
  for (let i = 0; i < MAX_HEARTS; i++) {
    s += i < hearts ? '❤️' : '<span class="heart-lost">❤️</span>';
  }
  el.innerHTML = s;
}

function updateMoves() {
  document.getElementById('movesLabel').textContent = movesLeft + ' / ' + MAX_MOVES;
}

/* ================= ХОДЫ ================= */
function spendMove() {
  movesLeft--;
  updateMoves();
}

function usePuzzle(petals) {
  if (movesLeft <= 0) return;

  petals.forEach(i => {
    if (i >= PETAL_COUNT) return;
    if (stage[i] === 3) {
      // был оранжевый — починка в зелёный, ошибка уже засчитана
      stage[i] = GREEN;
      return;
    }
    stage[i]++;
    if (stage[i] === 3) {
      // пережгли — теряем сердечко
      hearts--;
    }
  });

  spendMove();
  drawFlower();
  updateHearts();
  checkEnd();
}

function useReset() {
  if (movesLeft <= 0) return;

  for (let i = 0; i < PETAL_COUNT; i++) {
    if (stage[i] > 0 && stage[i] !== 3) stage[i]--;
    else if (stage[i] === 3) stage[i] = GREEN; // починка оранжевого
  }

  spendMove();
  drawFlower();
  updateHearts();
  checkEnd();
}

/* ================= ПРОВЕРКА КОНЦА ================= */
function checkEnd() {
  if (hearts <= 0) {
    endLose('Сердечки закончились!', 'Ты пережёг слишком много лепестков.');
    return;
  }

  let allGreen = true;
  for (let i = 0; i < PETAL_COUNT; i++) if (stage[i] !== GREEN) { allGreen = false; break; }

  if (allGreen) {
    gameActive = false;
    document.querySelectorAll('.puzzle').forEach(p => p.classList.add('disabled'));
    document.getElementById('status').innerHTML =
      '<span class="win">🎉 Цветок полностью окрашен!</span>';
    setTimeout(showWin, 400);
    return;
  }

  if (movesLeft <= 0) {
    endLose('Ходы закончились!', 'Не успел окрасить все лепестки. Попробуй ещё раз.');
    return;
  }

  let done = 0;
  for (let i = 0; i < PETAL_COUNT; i++) if (stage[i] === GREEN) done++;
  document.getElementById('status').textContent =
    'Готово лепестков: ' + done + ' / ' + PETAL_COUNT;
}

function endLose(title, text) {
  gameActive = false;
  document.querySelectorAll('.puzzle').forEach(p => p.classList.add('disabled'));
  document.getElementById('loseTitle').textContent = title;
  document.getElementById('loseText').textContent = text;
  document.getElementById('status').innerHTML =
    '<span class="lose">' + title + '</span>';
  setTimeout(() => document.getElementById('loseScreen').classList.add('show'), 400);
}

/* ================= ЭКРАНЫ ================= */
const startScreen = document.getElementById('startScreen');
const winScreen = document.getElementById('winScreen');
const loseScreen = document.getElementById('loseScreen');

function showWin() {
  document.getElementById('winText').textContent =
    'Уровень ' + LEVEL + ' пройден: все ' + PETAL_COUNT +
    ' лепестков зелёные. Осталось ❤️: ' + hearts + ', ходов: ' + movesLeft + '.';
  winScreen.classList.add('show');
}

function startLevel(lvl) {
  LEVEL = lvl;
  PETAL_COUNT = petalCountForLevel(LEVEL);
  MAX_HEARTS = heartsForLevel(LEVEL);
  MAX_MOVES = movesForLevel(LEVEL);
  stage = new Array(PETAL_COUNT).fill(0);
  hearts = MAX_HEARTS;
  movesLeft = MAX_MOVES;
  hoveredPetals = null;
  gameActive = true;

  document.getElementById('lvlLabel').textContent = LEVEL;
  updateHearts();
  updateMoves();

  winScreen.classList.remove('show');
  loseScreen.classList.remove('show');
  startScreen.classList.remove('show');

  renderPuzzles();
  drawFlower();
  drawTargetFlower();
  document.getElementById('status').textContent =
    'Готово лепестков: 0 / ' + PETAL_COUNT;
}

/* ================= КНОПКИ ================= */
document.getElementById('startBtn').addEventListener('click', () => startLevel(1));
document.getElementById('nextBtn').addEventListener('click', () => startLevel(LEVEL + 1));
document.getElementById('replayBtn').addEventListener('click', () => startLevel(LEVEL));
document.getElementById('retryBtn').addEventListener('click', () => startLevel(LEVEL));
document.getElementById('restart').addEventListener('click', () => startLevel(LEVEL));

/* ================= ИНИЦИАЛИЗАЦИЯ ================= */
PETAL_COUNT = petalCountForLevel(LEVEL);
stage = new Array(PETAL_COUNT).fill(0);
drawFlower();
drawTargetFlower();
</script>
</body>
</html>
  </div>
</section>

<!-- ================= ВКЛАДКА 2: ТГ-БОТЫ ================= -->
<section id="bots" class="tab-content">
  <h1>Мои Telegram-боты</h1>

  <div style="display: flex; gap: 20px; flex-wrap: wrap;">
    <div style="background:#fff; padding:20px; border-radius:12px; width:300px;">
      <h3>Название бота</h3>
      <p>Короткое описание, что он делает.</p>
      <a href="https://t.me/имя_вашего_бота" target="_blank"
         style="display:inline-block; background:#4a9eff; color:#fff; padding:10px 20px; border-radius:8px; text-decoration:none;">
         Открыть в Telegram
      </a>
    </div>
  </div>

  <h2 style="margin-top:40px;">QR-код на бота</h2>
  <p>Можно сгенерировать QR прямо тут — вставьте картинку:</p>
  <!-- 👇 QR-код вставляется как обычная картинка -->
  <img src="qr-bot.png" alt="QR-код бота" style="width:200px;">
  <!-- Чтобы получить QR: зайдите на qr-code-generator.com, введите ссылку t.me/ваш_бот, скачайте PNG, загрузите в репозиторий -->
</section>

<!-- ================= ВКЛАДКА 3: НЕДВИЖИМОСТЬ ================= -->
<section id="realty" class="tab-content">
  <h1>Недвижимость</h1>

  <div style="margin-bottom:20px;">
    <button class="realty-btn active" data-realty="city">🏙 Городская</button>
    <button class="realty-btn" data-realty="country">🌳 Загородная</button>
  </div>

  <!-- Городская -->
  <div id="city" class="realty-panel">
    <!-- 👇 ОБЪЯВЛЕНИЯ. Копируйте блок ниже для каждого объекта -->
    <div style="background:#fff; border-radius:12px; overflow:hidden; width:320px; display:inline-block; margin:10px; vertical-align:top;">
      <img src="photo1.jpg" alt="Квартира" style="width:100%; height:200px; object-fit:cover;">
      <div style="padding:15px;">
        <h3 style="margin:0 0 8px;">2-комн. квартира, 65 м²</h3>
        <p style="margin:0 0 8px; color:#666; font-size:14px;">Москва, Хамовники</p>
        <p style="margin:0 0 12px; font-size:20px; font-weight:bold; color:#4a9eff;">25 000 000 ₽</p>
        <a href="https://www.miel.ru/ссылка_на_ваше_объявление" target="_blank"
           style="display:inline-block; background:#4a9eff; color:#fff; padding:8px 16px; border-radius:8px; text-decoration:none; font-size:14px;">
           Смотреть на Миэль
        </a>
      </div>
    </div>
    <!-- Конец блока объявления -->
  </div>

  <!-- Загородная -->
  <div id="country" class="realty-panel" style="display:none;">
    <!-- 👇 Сюда точно такие же блоки, только с загородными объектами -->
  </div>
</section>

<!-- ================= ВКЛАДКА 4: КНИГИ ================= -->
<section id="books" class="tab-content">
  <h1>Книги в подарок</h1>

  <!-- 👇 Копируйте этот блок для каждой книги -->
  <div style="background:#fff; border-radius:12px; overflow:hidden; width:260px; display:inline-block; margin:10px; vertical-align:top;">
    <img src="book1.jpg" alt="Книга" style="width:100%; height:320px; object-fit:cover;">
    <div style="padding:15px;">
      <h3 style="margin:0 0 8px;">Название книги</h3>
      <p style="margin:0 0 8px; color:#666; font-size:14px;">Короткое описание</p>
      <p style="margin:0; font-size:18px; font-weight:bold; color:#4a9eff;">Бесплатно / 500 ₽</p>
    </div>
  </div>
  <!-- Конец блока книги -->
</section>

<!-- ================= ВКЛАДКА 5: ОБО МНЕ ================= -->
<section id="contacts" class="tab-content">
  <h1>Обо мне</h1>
  <p>Здесь напишите пару предложений о себе: чем занимаетесь, что делаете, чем можете помочь.</p>

  <h2>Контакты</h2>
  <ul style="list-style:none; padding:0; font-size:16px;">
    <li>📱 Telegram: <a href="https://t.me/oscilliacia" target="_blank">@oscilliacia</a></li>
    <li>✉️ Email: <a href="mailto:ваш@email.com">ваш@email.com</a></li>
    <li>💬 WhatsApp: <a href="https://wa.me/79990000000" target="_blank">+7 999 000-00-00</a></li>
  </ul>
</section>

<script>
  // Переключение верхних вкладок
  document.querySelectorAll('.tab-btn').forEach(btn => {
    btn.addEventListener('click', () => {
      document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
      document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
      btn.classList.add('active');
      document.getElementById(btn.dataset.tab).classList.add('active');
    });
  });

  // Переключение игр внутри вкладки "Игры"
  document.querySelectorAll('.game-btn').forEach(btn => {
    btn.addEventListener('click', () => {
      document.querySelectorAll('.game-btn').forEach(b => b.classList.remove('active'));
      document.querySelectorAll('.game-panel').forEach(p => p.style.display = 'none');
      btn.classList.add('active');
      document.getElementById(btn.dataset.game).style.display = 'block';
    });
  });

  // Переключение городская/загородная
  document.querySelectorAll('.realty-btn').forEach(btn => {
    btn.addEventListener('click', () => {
      document.querySelectorAll('.realty-btn').forEach(b => b.classList.remove('active'));
      document.querySelectorAll('.realty-panel').forEach(p => p.style.display = 'none');
      btn.classList.add('active');
      document.getElementById(btn.dataset.realty).style.display = 'block';
    });
  });
</script>

</body>
</html>
