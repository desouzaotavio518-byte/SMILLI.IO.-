
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>SMILLI.IO</title>

<style>
*{box-sizing:border-box}
body{
 margin:0;background:#10151b;color:white;
 font-family:Arial;text-align:center;
}
header{
 padding:12px;background:#080d12;
 border-bottom:2px solid #00ed91;
}
h1{color:#00ed91;margin:5px;font-size:38px}
button,select{
 border:0;border-radius:9px;padding:12px;
 font-weight:bold;margin:4px;cursor:pointer;
}
button{background:#00ed91;color:#07130e}
.menu{padding:12px}
#game{
 width:100%;height:65vh;display:block;
 background:#17252b;touch-action:none;
}
.panel{
 background:#080d12;padding:10px;
 display:flex;justify-content:center;
 flex-wrap:wrap;gap:5px;
}
.info{color:#00ed91;font-weight:bold}
#shop,#admin{display:none;padding:15px}
.skin{
 display:inline-block;background:#26343d;
 padding:12px;margin:5px;border-radius:12px;
}
.skin button{display:block;width:100%}
#message{color:#ffdc65}
</style>
</head>

<body>

<header>
<h1>🐍 SMILLI.IO</h1>
<div>O mundo das cobras!</div>
</header>

<div class="menu">
<input id="nickname" placeholder="Seu nome"
maxlength="15">
<br>
<button onclick="startGame()">JOGAR</button>
<button onclick="openShop()">SKINS</button>
<button onclick="openAdmin()">ADMIN</button>
</div>

<div class="panel">
<span class="info">⭐ Pontos: <span id="score">0</span></span>
<span class="info">🪙 Moedas: <span id="coins">0</span></span>
<span class="info">🏆 Nível: <span id="level">1</span></span>
</div>

<canvas id="game"></canvas>

<div id="message">Escolha uma skin e comece a jogar!</div>

<div id="shop">
<h2>🛒 LOJA DE SKINS</h2>
<div id="skins"></div>
<button onclick="closePanels()">VOLTAR</button>
</div>

<div id="admin">
<h2>👑 PAINEL DO ADMINISTRADOR</h2>
<p>Área do administrador do SMILLI.IO</p>
<button onclick="gift()">🎁 ENVIAR PRESENTE</button>
<button onclick="adminCoins()">🪙 DAR MOEDAS</button>
<button onclick="adminFood()">🍎 DAR COMIDA</button>
<button onclick="closePanels()">VOLTAR</button>
</div>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

let running=false;
let score=0,coins=0,level=1;
let selected="🐍";
let snakes=[];
let foods=[];
let camera={x:0,y:0};
let mouse={x:0,y:0};
let player;
let speed=3;
let world=2500;

const skins=[
 {name:"Cobra",head:"🐍",price:0},
 {name:"Dragão",head:"🐉",price:100},
 {name:"Dinossauro",head:"🦖",price:120},
 {name:"Vaca",head:"🐮",price:150},
 {name:"Tigre",head:"🐯",price:180},
 {name:"Leão",head:"🦁",price:200},
 {name:"Lobo",head:"🐺",price:220},
 {name:"Raposa",head:"🦊",price:240},
 {name:"Coelho",head:"🐰",price:250},
 {name:"Porco",head:"🐷",price:270},
 {name:"Gato",head:"🐱",price:300},
 {name:"Robô",head:"🤖",price:350},
 {name:"Alien",head:"👽",price:400},
 {name:"Tubarão",head:"🦈",price:450},
 {name:"Polvo",head:"🐙",price:500}
];

function resize(){
 canvas.width=window.innerWidth;
 canvas.height=window.innerHeight*.65;
}
window.addEventListener("resize",resize);
resize();

function createSnake(x,y,head,name,bot=false){
 return {
  x,y,head,name,bot,
  angle:Math.random()*6.28,
  length:12,
  body:[],
  alive:true
 };
}

function startGame(){
 if(running)return;
 selected=selected||"🐍";
 player=createSnake(1250,1250,selected,
 document.getElementById("nickname").value||"Jogador");
 snakes=[player];

 for(let i=0;i<9;i++){
  let bot=createSnake(
   Math.random()*world,
   Math.random()*world,
   ["🐍","🐉","🦖","🐮","🦊"][i%5],
   "Jogador"+(i+1),true
  );
  snakes.push(bot);
 }

 foods=[];
 for(let i=0;i<250;i++)createFood();

 running=true;
 closePanels();
 document.getElementById("message").innerText=
 "Boa sorte! Coma para crescer!";
 requestAnimationFrame(loop);
}

function createFood(x,y){
 foods.push({
  x:x??Math.random()*world,
  y:y??Math.random()*world,
  color:["#ff4b4b","#ffe14b","#40eaff","#a65aff"][Math.floor(Math.random()*4)],
  value:1
 });
}

function update(){
 if(!running)return;

 let target=Math.atan2(mouse.y-canvas.height/2,
 mouse.x-canvas.width/2);

 if(mouse.x||mouse.y){
  let diff=Math.atan2(Math.sin(target-player.angle),
  Math.cos(target-player.angle));
  player.angle+=diff*.08;
 }

 snakes.forEach(s=>{
  if(!s.alive)return;

  if(s.bot){
   s.angle+= (Math.random()-.5)*.12;
   let nearest=foods[0];
   if(nearest){
    let a=Math.atan2(nearest.y-s.y,nearest.x-s.x);
    let d=Math.atan2(Math.sin(a-s.angle),Math.cos(a-s.angle));
    s.angle+=d*.025;
   }
  }

  s.body.unshift({x:s.x,y:s.y});
  while(s.body.length>s.length)s.body.pop();

  let velocity=s===player?speed:2.1;
  s.x+=Math.cos(s.angle)*velocity;
  s.y+=Math.sin(s.angle)*velocity;

  s.x=Math.max(10,Math.min(world-10,s.x));
  s.y=Math.max(10,Math.min(world-10,s.y));

  for(let i=0;i<foods.length;i++){
   let f=foods[i];
   if(Math.hypot(s.x-f.x,s.y-f.y)<23){
    foods.splice(i,1);
    s.length+=2;
    if(s===player){
     score+=10;
     coins+=2;
     updateUI();
    }
    createFood();
    break;
   }
  }
 });

 // Colisão entre cobras
 snakes.forEach(s=>{
  if(!s.alive)return;
  snakes.forEach(other=>{
   if(s===other||!other.alive)return;
   for(let i=5;i<other.body.length;i++){
    if(Math.hypot(s.x-other.body[i].x,
    s.y-other.body[i].y)<12){
     if(s===player){
      score+=50;
      coins+=20;
      player.length+=8;
      document.getElementById("message").innerText=
      "Você derrotou uma cobra! +50 pontos!";
      updateUI();
     }
     other.alive=false;
     break;
    }
   }
  });
 });

 snakes=snakes.filter(s=>s.alive||s===player);

 if(player&&player.alive===false){
  running=false;
  document.getElementById("message").innerText=
  "Você foi derrotado! Clique em JOGAR para recomeçar.";
 }

 if(player){
  level=Math.floor(score/200)+1;
  speed=3+Math.min(level*.12,2);
  camera.x=player.x-canvas.width/2;
  camera.y=player.y-canvas.height/2;
  updateUI();
 }
}

function draw(){
 ctx.clearRect(0,0,canvas.width,canvas.height);

 ctx.fillStyle="#17252b";
 ctx.fillRect(0,0,canvas.width,canvas.height);

 ctx.save();
 ctx.translate(-camera.x,-camera.y);

 // Mapa
 ctx.strokeStyle="#293a40";
 for(let x=0;x<world;x+=80){
  ctx.beginPath();ctx.moveTo(x,0);
  ctx.lineTo(x,world);ctx.stroke();
 }
 for(let y=0;y<world;y+=80){
  ctx.beginPath();ctx.moveTo(0,y);
  ctx.lineTo(world,y);ctx.stroke();
 }

 // Comidas
 foods.forEach(f=>{
  ctx.fillStyle=f.color;
  ctx.beginPath();
  ctx.arc(f.x,f.y,6,0,Math.PI*2);
  ctx.fill();
 });

 // Cobras
 snakes.forEach(s=>{
  if(!s.alive)return;

  s.body.forEach((p,i)=>{
   ctx.fillStyle=s.head==="🐮"?
   (i%2===0?"#fff":"#111"):"#00bd83";

   if(s.head==="🐉")ctx.fillStyle="#e64c35";
   if(s.head==="🦖")ctx.fillStyle="#55ad42";
   if(s.head==="🦊")ctx.fillStyle="#f08c34";

   ctx.beginPath();
   ctx.arc(p.x,p.y,12-i*.015,0,Math.PI*2);
   ctx.fill();
  });

  ctx.font="25px Arial";
  ctx.textAlign="center";
  ctx.fillText(s.head,s.x,s.y+9);

  ctx.font="12px Arial";
  ctx.fillStyle="white";
  ctx.fillText(s.name,s.x,s.y-25);
 });

 ctx.restore();
}

function loop(){
 if(!running)return;
 update();
 draw();
 requestAnimationFrame(loop);
}

function updateUI(){
 document.getElementById("score").innerText=score;
 document.getElementById("coins").innerText=coins;
 document.getElementById("level").innerText=level;
}

canvas.addEventListener("mousemove",e=>{
 let r=canvas.getBoundingClientRect();
 mouse.x=e.clientX-r.left;
 mouse.y=e.clientY-r.top;
});

canvas.addEventListener("touchmove",e=>{
 e.preventDefault();
 let r=canvas.getBoundingClientRect();
 mouse.x=e.touches[0].clientX-r.left;
 mouse.y=e.touches[0].clientY-r.top;
},{passive:false});

function openShop(){
 running=false;
 document.getElementById("shop").style.display="block";
 document.getElementById("admin").style.display="none";
 renderSkins();
}

function renderSkins(){
 let html="";
 skins.forEach(s=>{
 html+=`
 <div class="skin">
 <h2>${s.head}</h2>
 <b>${s.name}</b>
 <p>${s.price===0?"GRÁTIS":s.price+" moedas"}</p>
 <button onclick="buySkin('${s.head}',${s.price})">
 ESCOLHER
 </button>
 </div>`;
 });
 document.getElementById("skins").innerHTML=html;
}

function buySkin(head,price){
 if(price>coins){
  alert("Você precisa de mais moedas!");
  return;
 }
 coins-=price;
 selected=head;
 updateUI();
 alert("Skin selecionada: "+head);
}

function openAdmin(){
 running=false;
 document.getElementById("admin").style.display="block";
 document.getElementById("shop").style.display="none";
}

function gift(){
 let name=prompt("Nome do jogador que receberá o presente:");
 if(!name)return;
 let present=prompt("Qual presente? Exemplo: 500 moedas, skin ou comida:");
 if(present){
  alert("Presente preparado para "+name+": "+present);
 }
}

function adminCoins(){
 let amount=Number(prompt("Quantas moedas deseja adicionar?"));
 if(amount>0&&amount<=100000){
  coins+=amount;
  updateUI();
 }
}

function adminFood(){
 if(player){
  for(let i=0;i<50;i++){
   createFood(player.x+Math.random()*200-100,
   player.y+Math.random()*200-100);
  }
  alert("50 comidas criadas!");
 }
}

function closePanels(){
 document.getElementById("shop").style.display="none";
 document.getElementById("admin").style.display="none";
}
</script>
</body>
</html>
