<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Hybrid Super Glitch Engine – FULL</title>
<style>
html, body {margin:0; background:#000; overflow:hidden; font-family:system-ui, sans-serif; color:#fff;}
canvas {width:100vw; height:100vh; display:block;}

.controls {
  position:fixed;
  bottom:12px;
  left:12px;
  background:rgba(0,0,0,0.85);
  padding:16px;
  border-radius:14px;
  max-height:90vh;
  overflow:auto;
}

.controls label, .controls button{display:block; font-size:12px; margin-bottom:8px;}
button{cursor:pointer; padding:4px 8px;}
hr{margin:10px 0; border:0; height:1px; background:#444;}

#uiToggle{
  position:fixed;
  top:20px;
  right:20px;
  z-index:999;
  padding:10px 14px;
  font-size:14px;
}
</style>
</head>
<body>

<button id="startCam" style="position:fixed;top:20px;left:20px;z-index:999;padding:12px 18px;font-size:16px;">
Start Camera
</button>

<button id="uiToggle">Toggle UI</button>

<video id="video" autoplay playsinline muted style="display:none"></video>
<canvas id="canvas"></canvas>

<div class="controls" id="controlsPanel">

<label><input id="bframe" type="checkbox" checked> B-Frame Blend</label>
<label>B-Frame Intensity <input id="bIntensity" type="range" min="0" max="10" step="0.1" value="5"></label>

<hr>

<label><input id="psHorizontal" type="checkbox"> Pixel Sort Horizontal</label>
<label><input id="psVertical" type="checkbox"> Pixel Sort Vertical</label>
<label><input id="psDual" type="checkbox"> Dual-Axis Sort</label>
<label><input id="psMotion" type="checkbox"> Motion-Reactive Sort</label>
<label><input id="psDestruct" type="checkbox"> Destructive Chaos</label>
<label>Sort Threshold <input id="psThreshold" type="range" min="0" max="255" value="120"></label>
<label>Motion Sensitivity <input id="psMotionStrength" type="range" min="0" max="100" value="30"></label>
<label>Chaos Intensity <input id="psChaos" type="range" min="0" max="100" value="50"></label>

<hr>

<button id="captureFrame">Capture Overlay</button>
<button id="clearOverlay">Clear Overlays</button>
<label>Overlay Opacity <input id="overlayOpacity" type="range" min="0" max="1" step="0.01" value="0.25"></label>
<label>Overlay Decay <input id="overlayDecay" type="range" min="0" max="0.05" step="0.001" value="0.005"></label>
<label><input id="overlayWarp" type="checkbox"> Warp Overlays</label>
<label><input id="overlayFeedback" type="checkbox"> Overlay → Live Feedback</label>

</div>

<script>

/* ===== UI TOGGLE ===== */

const uiToggle = document.getElementById("uiToggle");
const controlsPanel = document.getElementById("controlsPanel");

uiToggle.onclick = () => {
  if (controlsPanel.style.display === "none") {
    controlsPanel.style.display = "block";
  } else {
    controlsPanel.style.display = "none";
  }
};

/* ===== ORIGINAL SCRIPT ===== */

const video=document.getElementById('video');
const canvas=document.getElementById('canvas');
const ctx=canvas.getContext('2d');
const buffer=document.createElement('canvas');
const bctx=buffer.getContext('2d');

// Controls
const bframeToggle=document.getElementById('bframe');
const bIntensity=document.getElementById('bIntensity');

const psHorizontal=document.getElementById('psHorizontal');
const psVertical=document.getElementById('psVertical');
const psDual=document.getElementById('psDual');
const psMotion=document.getElementById('psMotion');
const psDestruct=document.getElementById('psDestruct');
const psThreshold=document.getElementById('psThreshold');
const psMotionStrength=document.getElementById('psMotionStrength');
const psChaos=document.getElementById('psChaos');

const captureBtn=document.getElementById('captureFrame');
const clearBtn=document.getElementById('clearOverlay');
const overlayOpacitySlider=document.getElementById('overlayOpacity');
const overlayDecaySlider=document.getElementById('overlayDecay');
const overlayWarp=document.getElementById('overlayWarp');
const overlayFeedback=document.getElementById('overlayFeedback');

let w=0,h=0;
let prevFrame=null, prevPrevFrame=null;
let overlays=[];

const startBtn = document.getElementById("startCam");

startBtn.onclick = async () => {
  const stream = await navigator.mediaDevices.getUserMedia({
    video:true,
    audio:false
  });

  video.srcObject = stream;

  video.onloadedmetadata = () => {
    video.play();
    w = canvas.width = buffer.width = video.videoWidth;
    h = canvas.height = buffer.height = video.videoHeight;
    startBtn.remove();
  };
};

/* --- rest of your original script stays unchanged --- */

</script>
</body>
</html>
