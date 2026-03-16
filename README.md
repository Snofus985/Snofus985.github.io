<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Hybrid Super Glitch Engine – Restored</title>

<style>
html,body{margin:0;background:#000;overflow:hidden;font-family:system-ui;color:#fff}
canvas{width:100vw;height:100vh;display:block}

.controls{
position:fixed;
bottom:12px;
left:12px;
background:rgba(0,0,0,.85);
padding:16px;
border-radius:14px;
max-height:90vh;
overflow:auto;
}

.controls label,.controls button{display:block;font-size:12px;margin-bottom:8px}
button{cursor:pointer;padding:4px 8px}
hr{margin:10px 0;border:0;height:1px;background:#444}

#uiToggle{position:fixed;top:20px;right:20px;z-index:999}
#switchCam{position:fixed;top:20px;left:160px;z-index:999}
#cameraSelect{position:fixed;top:70px;left:20px;z-index:999}
</style>
</head>

<body>

<button id="startCam" style="position:fixed;top:20px;left:20px;z-index:999">Start Camera</button>
<button id="switchCam">Switch Camera</button>
<select id="cameraSelect"></select>
<button id="uiToggle">Toggle UI</button>

<video id="video" autoplay playsinline muted style="display:none"></video>
<canvas id="canvas"></canvas>

<div class="controls" id="controlsPanel">

<label><input id="bframe" type="checkbox" checked>B-Frame Blend</label>
<label>B-Frame Intensity <input id="bIntensity" type="range" min="0" max="10" step="0.1" value="5"></label>

<hr>

<label><input id="psHorizontal" type="checkbox">Pixel Sort Horizontal</label>
<label><input id="psVertical" type="checkbox">Pixel Sort Vertical</label>
<label><input id="psDual" type="checkbox">Dual-Axis Sort</label>
<label><input id="psMotion" type="checkbox">Motion-Reactive Sort</label>
<label><input id="psDestruct" type="checkbox">Destructive Chaos</label>

<label>Sort Threshold <input id="psThreshold" type="range" min="0" max="255" value="120"></label>
<label>Motion Sensitivity <input id="psMotionStrength" type="range" min="0" max="100" value="30"></label>
<label>Chaos Intensity <input id="psChaos" type="range" min="0" max="100" value="50"></label>

<hr>

<button id="captureFrame">Capture Overlay</button>
<button id="clearOverlay">Clear Overlays</button>

<label>Overlay Opacity <input id="overlayOpacity" type="range" min="0" max="1" step="0.01" value="0.25"></label>
<label>Overlay Decay <input id="overlayDecay" type="range" min="0" max="0.05" step="0.001" value="0.005"></label>

<label><input id="overlayWarp" type="checkbox">Warp Overlays</label>
<label><input id="overlayFeedback" type="checkbox">Overlay → Live Feedback</label>

</div>

<script>

/* UI toggle */

const controlsPanel=document.getElementById("controlsPanel")
document.getElementById("uiToggle").onclick=()=>{
controlsPanel.style.display=
controlsPanel.style.display==="none"?"block":"none"
}

/* camera system */

const video=document.getElementById("video")
const startBtn=document.getElementById("startCam")
const switchBtn=document.getElementById("switchCam")
const cameraSelect=document.getElementById("cameraSelect")

let currentStream=null
let cameras=[]
let currentIndex=0

function stopStream(){
if(currentStream)currentStream.getTracks().forEach(t=>t.stop())
}

async function startCamera(index){

stopStream()

let constraints

if(cameras[index]){
constraints={video:{deviceId:{exact:cameras[index].deviceId}}}
}else{
constraints={video:true}
}

const stream=await navigator.mediaDevices.getUserMedia(constraints)

currentStream=stream
video.srcObject=stream

}

async function getCameras(){

const devices=await navigator.mediaDevices.enumerateDevices()
cameras=devices.filter(d=>d.kind==="videoinput")

cameraSelect.innerHTML=""

cameras.forEach((cam,i)=>{
const option=document.createElement("option")
option.value=i
option.text=cam.label||"Camera "+(i+1)
cameraSelect.appendChild(option)
})

}

startBtn.onclick=async()=>{

const temp=await navigator.mediaDevices.getUserMedia({video:true})
temp.getTracks().forEach(t=>t.stop())

await getCameras()
await startCamera(0)

startBtn.remove()

}

switchBtn.onclick=async()=>{

currentIndex++
if(currentIndex>=cameras.length)currentIndex=0

cameraSelect.value=currentIndex
await startCamera(currentIndex)

}

cameraSelect.onchange=async()=>{
currentIndex=parseInt(cameraSelect.value)
await startCamera(currentIndex)
}

/* canvas + glitch engine */

const canvas=document.getElementById("canvas")
const ctx=canvas.getContext("2d")

const buffer=document.createElement("canvas")
const bctx=buffer.getContext("2d")

let w=0,h=0
let prevFrame=null
let prevPrevFrame=null
let overlays=[]

video.onloadedmetadata=()=>{
video.play()
w=canvas.width=buffer.width=video.videoWidth
h=canvas.height=buffer.height=video.videoHeight
}

function getFrame(){
bctx.drawImage(video,0,0,w,h)
return bctx.getImageData(0,0,w,h)
}

/* B-frame */

function blendBFrame(curr){

if(prevPrevFrame && document.getElementById("bframe").checked){

const alpha=document.getElementById("bIntensity").value/10
const out=new Uint8ClampedArray(curr.data.length)

for(let i=0;i<curr.data.length;i+=4){

out[i]=prevPrevFrame.data[i]*alpha+curr.data[i]*(1-alpha)
out[i+1]=prevPrevFrame.data[i+1]*alpha+curr.data[i+1]*(1-alpha)
out[i+2]=prevPrevFrame.data[i+2]*alpha+curr.data[i+2]*(1-alpha)
out[i+3]=255

}

return new ImageData(out,w,h)

}

return curr
}

/* pixel sorting */

function pixelSort(imageData){

const threshold=parseInt(psThreshold.value)
const chaos=parseInt(psChaos.value)/100
const motionStrength=parseInt(psMotionStrength.value)/100

const {data,width,height}=imageData
const output=new Uint8ClampedArray(data)

function brightness(i){return(data[i]+data[i+1]+data[i+2])/3}

function sortSegment(indices){

const seg=indices.map(i=>({
r:data[i],
g:data[i+1],
b:data[i+2],
a:data[i+3],
bright:brightness(i)
}))

seg.sort((a,b)=>a.bright-b.bright)

if(psDestruct.checked){

for(let i=seg.length-1;i>0;i--){
const j=Math.floor(Math.random()*seg.length*chaos)
;[seg[i],seg[j]]=[seg[j],seg[i]]
}

}

indices.forEach((idx,i)=>{
output[idx]=seg[i].r
output[idx+1]=seg[i].g
output[idx+2]=seg[i].b
output[idx+3]=seg[i].a
})

}

const horizontal=psHorizontal.checked||psDual.checked
const vertical=psVertical.checked||psDual.checked

if(horizontal){

for(let y=0;y<height;y++){

let indices=[]

for(let x=0;x<width;x++){

const i=(y*width+x)*4

let b=brightness(i)

if(psMotion.checked && prevFrame){

const diff=Math.abs(data[i]-prevFrame.data[i])
b+=diff*motionStrength

}

if(b>threshold)indices.push(i)
else{
if(indices.length){sortSegment(indices);indices=[]}
}

}

if(indices.length)sortSegment(indices)

}

}

if(vertical){

for(let x=0;x<width;x++){

let indices=[]

for(let y=0;y<height;y++){

const i=(y*width+x)*4

let b=brightness(i)

if(psMotion.checked && prevFrame){

const diff=Math.abs(data[i]-prevFrame.data[i])
b+=diff*motionStrength

}

if(b>threshold)indices.push(i)
else{
if(indices.length){sortSegment(indices);indices=[]}
}

}

if(indices.length)sortSegment(indices)

}

}

return new ImageData(output,width,height)

}

/* main loop */

function loop(){

if(!w||video.readyState<2){
requestAnimationFrame(loop)
return
}

let curr=getFrame()

curr=blendBFrame(curr)

if(psHorizontal.checked||psVertical.checked||psDual.checked)
curr=pixelSort(curr)

bctx.putImageData(curr,0,0)

ctx.clearRect(0,0,w,h)
ctx.drawImage(buffer,0,0)

requestAnimationFrame(loop)

}

loop()

</script>
</body>
</html>
