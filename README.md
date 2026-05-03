<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mini Dash 😈</title>
<style>
body{margin:0;overflow:hidden;background:#000;}
canvas{display:block;touch-action:none;}
</style>
</head>
<body>
<canvas id="game"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

// ===== FULL SCREEN =====
function resize(){
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}
resize();
window.addEventListener("resize", resize);

// ===== GAME VAR =====
let ground = 100;
let score = 0;

let player = {
  x:120,y:0,size:40,
  dy:0,gravity:1,jump:-18,
  grounded:false
};

let spikes = [];
let speed = 8;

let gameOver=false;
let mode="menu";

// ===== INPUT =====
canvas.addEventListener("touchstart",(e)=>{
  e.preventDefault();

  if(mode==="menu"){
    mode="play";
    restart();
    return;
  }

  if(gameOver){
    restart();
    return;
  }

  if(player.grounded){
    player.dy = player.jump;
    player.grounded=false;
  }
});

// ===== SPAWN SPIKE =====
function spawnSpike(){
  if(mode==="menu" || gameOver) return;

  let count;

  if(score >= 9995){
    count = 4;
  } else {
    count = Math.floor(Math.random()*3)+1; // 1-3
  }

  let gap = 50;

  for(let i=0;i<count;i++){
    spikes.push({
      x:canvas.width + i*gap,
      y:canvas.height-ground,
      size:40,
      passed:false
    });
  }
}
setInterval(spawnSpike,1200);

// ===== RESTART =====
function restart(){
  player.x=120;
  player.y=canvas.height-ground-player.size;
  player.dy=0;

  spikes=[];
  score=0;
  gameOver=false;
}

// ===== UPDATE =====
function update(){

  // sky
  ctx.fillStyle="#4ecbff";
  ctx.fillRect(0,0,canvas.width,canvas.height);

  // ground
  ctx.fillStyle="green";
  ctx.fillRect(0,canvas.height-ground,canvas.width,ground);

  if(mode==="menu"){
    ctx.fillStyle="black";
    ctx.font="30px Arial";
    ctx.fillText("TAP TO START",canvas.width/2-100,canvas.height/2);
    requestAnimationFrame(update);
    return;
  }

  // physics
  if(!gameOver){
    player.dy += player.gravity;
    player.y += player.dy;

    if(player.y + player.size >= canvas.height-ground){
      player.y = canvas.height-ground-player.size;
      player.dy = 0;
      player.grounded = true;
    }
  }

  // player
  ctx.fillStyle="cyan";
  ctx.fillRect(player.x,player.y,player.size,player.size);

  // SPIKES
  ctx.fillStyle="red";

  spikes.forEach((s,i)=>{
    if(!gameOver){
      s.x -= speed;
    }

    // draw spike
    ctx.beginPath();
    ctx.moveTo(s.x, s.y);
    ctx.lineTo(s.x + s.size/2, s.y - s.size);
    ctx.lineTo(s.x + s.size, s.y);
    ctx.fill();

    // collision = chết ngay
    if(
      !gameOver &&
      player.x < s.x + s.size &&
      player.x + player.size > s.x &&
      player.y < s.y &&
      player.y + player.size > s.y - s.size
    ){
      gameOver = true;
    }

    // tính điểm khi vượt qua
    if(!s.passed && s.x + s.size < player.x){
      s.passed = true;

      if(score < 9999){
        // xác định số spike trong cụm
        let clusterSize = spikes.filter(sp =>
          Math.abs(sp.x - s.x) < 120
        ).length;

        let add = 1;
        if(clusterSize === 2) add = 3;
        if(clusterSize === 3) add = 7;
        if(clusterSize === 4) add = 4;

        score = Math.min(9999, score + add);
      }
    }

    if(s.x < -50){
      spikes.splice(i,1);
    }
  });

  // GAME OVER
  if(gameOver){
    ctx.fillStyle="black";
    ctx.font="30px Arial";
    ctx.fillText("GAME OVER",canvas.width/2-90,canvas.height/2);
    ctx.font="18px Arial";
    ctx.fillText("Tap to Restart",canvas.width/2-80,canvas.height/2+40);
  }

  // SCORE
  ctx.fillStyle="black";
  ctx.fillText("Score: "+score,20,40);

  requestAnimationFrame(update);
}

restart();
update();
</script>
</body>
</html>
