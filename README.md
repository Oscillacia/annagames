function updateHearts(){const el=document.getElementById('hearts');
let s='';for(let i=0;i<MAX_HEARTS;i++){
s+=i<hearts?'❤️':'<span class="heart-lost">❤️</span>';}
el.innerHTML=s;}
function updateMoves(){document.getElementById('movesLabel').textContent=movesLeft+' / '+MAX_MOVES;}
function spendMove(){movesLeft--;updateMoves();}
function usePuzzle(petals){if(movesLeft<=0)return;
petals.forEach(i=>{if(i>=PETAL_COUNT)return;
if(stage[i]===3){stage[i]=GREEN;return;}
stage[i]++;if(stage[i]===3){hearts--;}});
spendMove();drawFlower();updateHearts();checkEnd();}
function useReset(){if(movesLeft<=0)return;
for(let i=0;i<PETAL_COUNT;i++){
if(stage[i]>0&&stage[i]!==3)stage[i]--;
else if(stage[i]===3)stage[i]=GREEN;}
spendMove();drawFlower();updateHearts();checkEnd();}
function checkEnd(){if(hearts<=0){endLose('Сердечки закончились!','Ты пережёг слишком много лепестков.');return;}
let allGreen=true;
for(let i=0;i<PETAL_COUNT;i++)if(stage[i]!==GREEN){allGreen=false;break;}
if(allGreen){gameActive=false;
document.querySelectorAll('.puzzle').forEach(p=>p.classList.add('disabled'));
document.getElementById('status').innerHTML='<span class="win">🎉 Цветок полностью окрашен!</span>';
setTimeout(showWin,400);return;}
if(movesLeft<=0){endLose('Ходы закончились!','Не успел окрасить все лепестки. Попробуй ещё раз.');return;}
let done=0;for(let i=0;i<PETAL_COUNT;i++)if(stage[i]===GREEN)done++;
document.getElementById('status').textContent='Готово лепестков: '+done+' / '+PETAL_COUNT;}
function endLose(title,text){gameActive=false;
document.querySelectorAll('.puzzle').forEach(p=>p.classList.add('disabled'));
document.getElementById('loseTitle').textContent=title;
document.getElementById('loseText').textContent=text;
document.getElementById('status').innerHTML='<span class="lose">'+title+'</span>';
setTimeout(()=>document.getElementById('loseScreen').classList.add('show'),400);}
const startScreen=document.getElementById('startScreen');
const winScreen=document.getElementById('winScreen');
const loseScreen=document.getElementById('loseScreen');
function showWin(){document.getElementById('winText').textContent=
'Уровень '+LEVEL+' пройден: все '+PETAL_COUNT+
' лепестков зелёные. Осталось ❤️: '+hearts+', ходов: '+movesLeft+'.';
winScreen.classList.add('show');}
function startLevel(lvl){LEVEL=lvl;
PETAL_COUNT=petalCountForLevel(LEVEL);
MAX_HEARTS=heartsForLevel(LEVEL);
MAX_MOVES=movesForLevel(LEVEL);
stage=new Array(PETAL_COUNT).fill(0);
hearts=MAX_HEARTS;movesLeft=MAX_MOVES;hoveredPetals=null;gameActive=true;
document.getElementById('lvlLabel').textContent=LEVEL;
updateHearts();updateMoves();
winScreen.classList.remove('show');loseScreen.classList.remove('show');
startScreen.classList.remove('show');
renderPuzzles();drawFlower();drawTargetFlower();
document.getElementById('status').textContent='Готово лепестков: 0 / '+PETAL_COUNT;}
document.getElementById('startBtn').addEventListener('click',()=>startLevel(1));
document.getElementById('nextBtn').addEventListener('click',()=>startLevel(LEVEL+1));
document.getElementById('replayBtn').addEventListener('click',()=>startLevel(LEVEL));
document.getElementById('retryBtn').addEventListener('click',()=>startLevel(LEVEL));
document.getElementById('restart').addEventListener('click',()=>startLevel(LEVEL));
PETAL_COUNT=petalCountForLevel(LEVEL);
stage=new Array(PETAL_COUNT).fill(0);
drawFlower();drawTargetFlower();
<\/script>
</body></html>`;

/* ================= ПОДКЛЮЧЕНИЕ ИГР В IFRAME ================= */
document.getElementById('frame1').srcdoc = FOX_HTML;
document.getElementById('frame2').srcdoc = FLOWER_HTML;
</script>

</body>
</html>
