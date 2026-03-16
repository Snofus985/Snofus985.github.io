<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Hybrid Super Glitch Engine – FULL</title>

<style>
html,body{margin:0;background:#000;overflow:hidden;font-family:system-ui,sans-serif;color:#fff;}
canvas{width:100vw;height:100vh;display:block;}

.controls{
position:fixed;
bottom:12px;
left:12px;
background:rgba(0,0,0,0.85);
padding:16px;
border-radius:14px;
max-height:90vh;
overflow:auto;
}

.controls label,.controls button{display:block;font-size:12px;margin-bottom:8px;}

button{cursor:pointer;padding:4px 8px;}

#uiToggle{
position:fixed;
top:20px;
right:20px;
z-index:999;
padding:10px 14px;
}

#switchCam{
position:fixed;
top:20px;
left:160px;
z-index:999;
padding:12px 18px;
}

#cameraSelect{
position:fixed;
top:70px;
left:20px;
z-index:999;
}
</style>
</head>

<body>

<button id="startCam" style="position:fixed;top:20px;left:20px;z-index:999;padding:12px 18px;font-size:16px;">Start Camera</button>
<button id="switchCam">Switch Camera</button>
<select id="cameraSelect"></select>
<button id="uiToggle">Toggle UI</button>

<video id="video" autoplay playsinline muted style="display:none"></video>
<canvas id="canvas"></canvas>

<div class="controls" id="controlsPanel">
<label><input id="bframe" type="checkbox" checked>B-Frame Blend</label>
<label>B-Frame Intensity <input id="bIntensity" type="range" min="0" max="10" step="0.1" value="5"></label>
</div>

<script>

/* UI TOGGLE */

const uiToggle=document.getElementById("uiToggle");
const controlsPanel=document.getElementById("controlsPanel");

uiToggle.onclick=()=>{
controlsPanel.style.display=
controlsPanel.style.display==="none"?"block":"none";
};

/* CAMERA SYSTEM (FIXED) */

const video=document.getElementById("video");
const startBtn=document.getElementById("startCam");
const switchBtn=document.getElementById("switchCam");
const cameraSelect=document.getElementById("cameraSelect");

let currentStream=null;
let cameras=[];
let currentIndex=0;

function stopStream(){
if(currentStream){
currentStream.getTracks().forEach(track=>track.stop());
}
}

async function startCamera(index){

stopStream();

let constraints;

if(cameras[index]){
constraints={
video:{deviceId:{exact:cameras[index].deviceId}},
audio:false
};
}else{
constraints={video:true,audio:false};
}

try{

const stream=await navigator.mediaDevices.getUserMedia(constraints);

currentStream=stream;
video.srcObject=stream;

}catch(err){

console.warn("Camera start failed, using fallback");

const stream=await navigator.mediaDevices.getUserMedia({video:true});

currentStream=stream;
video.srcObject=stream;

}

}

async function getCameras(){

const devices=await navigator.mediaDevices.enumerateDevices();

cameras=devices.filter(d=>d.kind==="videoinput");

cameraSelect.innerHTML="";

cameras.forEach((cam,i)=>{

const option=document.createElement("option");

option.value=i;
option.text=cam.label || "Camera "+(i+1);

cameraSelect.appendChild(option);

});

}

startBtn.onclick=async()=>{

/* request permission first */

const tempStream=await navigator.mediaDevices.getUserMedia({video:true});
tempStream.getTracks().forEach(t=>t.stop());

/* detect cameras */

await getCameras();

/* start first camera */

await startCamera(0);

startBtn.remove();

};

switchBtn.onclick=async()=>{

if(cameras.length===0) return;

currentIndex++;

if(currentIndex>=cameras.length)
currentIndex=0;

cameraSelect.value=currentIndex;

await startCamera(currentIndex);

};

cameraSelect.onchange=async()=>{

currentIndex=parseInt(cameraSelect.value);

await startCamera(currentIndex);

};

/* CANVAS ENGINE */

const canvas=document.getElementById("canvas");
const ctx=canvas.getContext("2d");

const buffer=document.createElement("canvas");
const bctx=buffer.getContext("2d");

let w=0,h=0;

video.onloadedmetadata=()=>{

video.play();

w=canvas.width=buffer.width=video.videoWidth;
h=canvas.height=buffer.height=video.videoHeight;

};

function getFrame(){
bctx.drawImage(video,0,0,w,h);
return bctx.getImageData(0,0,w,h);
}

function loop(){

if(!w||video.readyState<2){
requestAnimationFrame(loop);
return;
}

let frame=getFrame();

bctx.putImageData(frame,0,0);

ctx.clearRect(0,0,w,h);
ctx.drawImage(buffer,0,0);

requestAnimationFrame(loop);

}

loop();

</script>

</body>
</html>
