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
