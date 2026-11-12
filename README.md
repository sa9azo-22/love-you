<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Particle Heart</title>
<style>
:root{--bg1:#4a0a22;--bg2:#12001f;--text:#ffd3df;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
html,body{height:100%;margin:0}
body{background:radial-gradient(circle at 50% 44%,var(--bg1),var(--bg2) 70%);color:var(--text);font:500 1rem/1.4 system-ui,sans-serif;overflow:hidden}
canvas{position:fixed;inset:0;width:100%;height:100%;display:block;touch-action:none}
p{position:fixed;left:0;right:0;bottom:calc(env(safe-area-inset-bottom,0px) + 1.6rem);text-align:center;margin:0;opacity:.6;letter-spacing:.02em}
</style>
</head>
<body>
<canvas id="c"></canvas>
<p>Touch the heart and the particles scatter, then find their way back.</p>
<script>
const cv=document.getElementById('c'),ctx=cv.getContext('2d');
const calm=matchMedia('(prefers-reduced-motion:reduce)').matches;
let W,H,DPR,R,CX,CY,P=[];
const mouse={x:-999,y:-999,on:false};

// 1. The heart curve: t goes 0 → 2π, returns a point on a heart outline (roughly -1…1)
function heart(t){
  const x=16*Math.pow(Math.sin(t),3);
  const y=13*Math.cos(t)-5*Math.cos(2*t)-2*Math.cos(3*t)-Math.cos(4*t);
  return {x:x/17,y:-y/17};
}

// 2. Three nested hearts: scale, band thickness, share of particles
const layers=[{s:1,th:.24,share:.56},{s:.62,th:.32,share:.28},{s:.22,th:.5,share:.08}];

function build(){
  const N=innerWidth<600?2600:4200;
  P=[];
  layers.forEach(L=>{
    for(let i=0;i<N*L.share;i++){
      const h=heart(Math.random()*Math.PI*2);
      // push the point inward by a random amount, so most dots hug the outline
      const k=L.s*(1-L.th*Math.pow(Math.random(),1.6));
      P.push(mk(h.x*k,h.y*k));
    }
  });
  // stray dots floating around the heart
  for(let i=0;i<N*.08;i++) P.push(mk((Math.random()*2-1)*1.6,(Math.random()*2-1)*1.4,true));
}
function mk(bx,by,stray){
  return{bx,by,stray,
    x:Math.random()*W,y:Math.random()*H,vx:0,vy:0,   // start scattered; the spring pulls them in
    f:.6+Math.random()*1.4,ph:Math.random()*6.28,a:1+Math.random()*2.5,
    sz:Math.random()<.12?2:1.2,c:Math.random()*3|0};
}

function resize(){
  DPR=Math.min(devicePixelRatio||1,2);
  W=innerWidth;H=innerHeight;
  cv.width=W*DPR;cv.height=H*DPR;ctx.setTransform(DPR,0,0,DPR,0,0);
  R=Math.min(W*.42,H*.27);CX=W/2;CY=H*.44;
}
resize();build();
addEventListener('resize',()=>{resize();build()});

// 3. Pointer pushes dots away
const move=e=>{const t=e.touches?e.touches[0]:e;mouse.x=t.clientX;mouse.y=t.clientY;mouse.on=true};
addEventListener('pointermove',move);addEventListener('pointerdown',move);
addEventListener('pointerup',()=>mouse.on=false);addEventListener('pointerleave',()=>mouse.on=false);

const colors=['rgba(255,120,160,.85)','rgba(255,170,195,.8)','rgba(255,70,110,.8)'];
let t0=performance.now();

function frame(now){
  const t=(now-t0)/1000;
  // 4. Heartbeat: a quick double pulse that repeats
  const beat=calm?1:1+.07*Math.pow(Math.max(0,Math.sin(t*5.2)),8)+.03*Math.pow(Math.max(0,Math.sin(t*5.2-.9)),8);
  ctx.clearRect(0,0,W,H);
  ctx.globalCompositeOperation='lighter';   // overlapping dots glow brighter
  for(const p of P){
    const wob=calm?0:Math.sin(t*p.f+p.ph)*p.a;
    const tx=CX+p.bx*R*beat+wob*(p.stray?4:1);
    const ty=CY+p.by*R*beat+(calm?0:Math.cos(t*p.f+p.ph)*p.a*(p.stray?4:1));
    // spring toward the target spot
    p.vx+=(tx-p.x)*.035;p.vy+=(ty-p.y)*.035;
    if(mouse.on){
      const dx=p.x-mouse.x,dy=p.y-mouse.y,d2=dx*dx+dy*dy;
      if(d2<14000){const f=(1-d2/14000)*9,d=Math.sqrt(d2)||1;p.vx+=dx/d*f;p.vy+=dy/d*f}
    }
    p.vx*=.86;p.vy*=.86;p.x+=p.vx;p.y+=p.vy;
  }
  for(let c=0;c<3;c++){
    ctx.fillStyle=colors[c];
    for(const p of P) if(p.c===c) ctx.fillRect(p.x,p.y,p.sz,p.sz);
  }
  requestAnimationFrame(frame);
}
requestAnimationFrame(frame);
</script>
</body>
</html>




