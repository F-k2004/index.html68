<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<title>🛰️ Sun–Earth–Spacecraft Geometry</title>
<style>
html,body{margin:0;overflow:hidden;background:#000012;font-family:system-ui}
canvas{display:block}
.hud{
 position:absolute;left:16px;top:16px;
 padding:14px 18px;border-radius:14px;
 background:rgba(255,255,255,0.06);
 backdrop-filter:blur(10px);
 color:#d9f3ff;font-size:13px;min-width:540px;
}
.good{color:#9ff0ff}
.arn{color:#ffcc66}
.bad{color:#ff6666}
</style>
</head>
<body>

<canvas id="c"></canvas>
<div class="hud" id="hud"></div>

<script>
const c=document.getElementById("c");
const ctx=c.getContext("2d");
let w,h;
function resize(){w=c.width=innerWidth;h=c.height=innerHeight}
resize();addEventListener("resize",resize);

const AU_PIX=120;
const sun={x:w/2,y:h/2};

const earth={a:1*AU_PIX,angle:0};
const ship={a:4*AU_PIX,angle:1.5};

function pos(a,ang){
 return {
   x:sun.x+Math.cos(ang)*a,
   y:sun.y+Math.sin(ang)*a
 };
}

function angleBetween(a,b){
 return Math.acos(
   (a.x*b.x + a.y*b.y) /
   (Math.hypot(a.x,a.y)*Math.hypot(b.x,b.y))
 );
}

function plasmaNoise(sepRad){
 const minAngle=0.01;
 const effective=Math.max(sepRad,minAngle);
 return 1e-21 + 5e-20*(1/Math.sin(effective));
}

function draw(){
 ctx.fillStyle="rgba(0,0,20,0.35)";
 ctx.fillRect(0,0,w,h);

 earth.angle+=0.002;
 ship.angle+=0.0007;

 const pe=pos(earth.a,earth.angle);
 const ps=pos(ship.a,ship.angle);

 // Draw Sun
 ctx.fillStyle="#ffcc66";
 ctx.beginPath();ctx.arc(sun.x,sun.y,16,0,Math.PI*2);ctx.fill();

 // Earth
 ctx.fillStyle="#2a7fff";
 ctx.beginPath();ctx.arc(pe.x,pe.y,7,0,Math.PI*2);ctx.fill();

 // Spacecraft
 ctx.fillStyle="#fff";
 ctx.beginPath();ctx.arc(ps.x,ps.y,5,0,Math.PI*2);ctx.fill();

 // Line of sight
 ctx.strokeStyle="rgba(200,200,255,0.5)";
 ctx.beginPath();
 ctx.moveTo(pe.x,pe.y);
 ctx.lineTo(ps.x,ps.y);
 ctx.stroke();

 // Compute SEP
 const vecES={x:sun.x-pe.x,y:sun.y-pe.y};
 const vecEP={x:ps.x-pe.x,y:ps.y-pe.y};
 const sep=angleBetween(vecES,vecEP);
 const sepDeg=sep*180/Math.PI;

 const noise=plasmaNoise(sep);
 const snr=1/noise;

 let cls=sepDeg>5?"good":sepDeg>2?"warn":"bad";

 document.getElementById("hud").innerHTML=`
🛰️ Sun–Earth–Probe Geometry<br>
📐 SEP Angle: <span class="${cls}">${sepDeg.toFixed(2)}°</span><br>
🌞 Conjunction Risk: ${sepDeg<3?"HIGH":"LOW"}<br>
📡 Plasma Noise: ${noise.toExponential(2)}<br>
📶 SNR: ${snr.toExponential(2)}
`;

 requestAnimationFrame(draw);
}

draw();
</script>
</body>
</html>
