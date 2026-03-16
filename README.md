<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=no">
<title>Hybrid Super Glitch Engine – Mobile</title>
<style>
:root{
  --panel-bg: rgba(0,0,0,0.82);
  --panel-border: rgba(255,255,255,0.12);
  --btn-bg: rgba(255,255,255,0.1);
  --btn-bg-active: rgba(255,255,255,0.2);
}
*{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
html,body{
  margin:0;
  width:100%;
  height:100%;
  background:#000;
  overflow:hidden;
  font-family:system-ui,-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif;
  color:#fff;
  touch-action:manipulation;
}
body{position:fixed; inset:0; overscroll-behavior:none;}
canvas{
  width:100vw;
  height:100vh;
  display:block;
  object-fit:cover;
  background:#000;
}
video{display:none}
button,input,select{font:inherit}
button,select{
  min-height:40px;
  border-radius:10px;
  border:1px solid rgba(255,255,255,0.16);
  background:var(--btn-bg);
  color:#fff;
}
button{padding:8px 10px; cursor:pointer}
button:active{transform:scale(0.98); background:var(--btn-bg-active)}
select{padding:8px 10px; width:100%}
input[type="range"]{width:100%}
label,.control-row,.status{display:block; font-size:12px; margin-bottom:10px}
hr{margin:12px 0; border:0; height:1px; background:#444}

.controls-toggle{
  position:fixed;
  top:max(12px, env(safe-area-inset-top));
  right:max(12px, env(safe-area-inset-right));
  z-index:30;
  min-width:52px;
  min-height:52px;
  border-radius:999px;
  backdrop-filter:blur(10px);
}
.controls{
  position:fixed;
  left:max(10px, env(safe-area-inset-left));
  right:max(10px, env(safe-area-inset-right));
  bottom:max(10px, env(safe-area-inset-bottom));
  z-index:20;
  background:var(--panel-bg);
  border:1px solid var(--panel-border);
  border-radius:16px;
  padding:14px;
  max-height:min(72vh, 720px);
  overflow:auto;
  backdrop-filter:blur(10px);
  -webkit-backdrop-filter:blur(10px);
  transition:transform .2s ease, opacity .2s ease;
}
.controls.hidden{
  transform:translateY(calc(100% + 20px));
  opacity:0;
  pointer-events:none;
}
.control-grid{
  display:grid;
  grid-template-columns:1fr 1fr;
  gap:8px;
  margin-bottom:10px;
}
.controls .section-title{
  font-size:11px;
  text-transform:uppercase;
  letter-spacing:.08em;
  opacity:.75;
  margin-bottom:8px;
}
.inline-label{
  display:flex;
  align-items:center;
  justify-content:space-between;
  gap:10px;
}
.inline-label input[type="checkbox"]{transform:scale(1.2)}
.status{
  padding:8px 10px;
  border-radius:10px;
  background:rgba(255,255,255,0.06);
  line-height:1.35;
}
@media (min-width: 900px){
  .controls{
    left:12px;
    right:auto;
    width:min(380px, 42vw);
    max-height:90vh;
  }
}


.capture-bar{
  position:fixed;
  left:max(12px, env(safe-area-inset-left));
  right:max(12px, env(safe-area-inset-right));
  bottom:max(12px, env(safe-area-inset-bottom));
  z-index:45;
  display:flex;
  align-items:center;
  justify-content:center;
  gap:18px;
  pointer-events:none;
}
.capture-bar button{
  pointer-events:auto;
  min-height:72px;
  min-width:72px;
  width:72px;
  height:72px;
  padding:0;
  backdrop-filter:blur(10px);
  background:rgba(0,0,0,.45);
  border:2px solid rgba(255,255,255,.85);
  border-radius:999px;
  box-shadow:0 8px 30px rgba(0,0,0,.35);
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:12px;
  font-weight:700;
}
.capture-bar #takeSnapshot{
  background:rgba(255,255,255,.12);
}
.capture-bar #recordToggle{
  width:88px;
  height:88px;
  border-width:3px;
  position:relative;
}
.capture-bar #recordToggle::before{
  content:'';
  width:26px;
  height:26px;
  border-radius:999px;
  background:#ff3b30;
  display:block;
}
.capture-bar #recordToggle.recording::before{
  width:24px;
  height:24px;
  border-radius:7px;
}
.capture-bar .capture-label{
  position:absolute;
  bottom:-22px;
  left:50%;
  transform:translateX(-50%);
  font-size:11px;
  font-weight:600;
  white-space:nowrap;
  text-shadow:0 1px 3px rgba(0,0,0,.65);
}
.capture-side{
  position:relative;
}
.controls{
  bottom:max(120px, calc(env(safe-area-inset-bottom) + 110px));
}
.top-actions{
  display:grid;
  grid-template-columns:1fr 1fr;
  gap:8px;
  margin-bottom:10px;
}
.perf-badge{
  display:inline-flex;
  align-items:center;
  justify-content:center;
  min-height:40px;
}
.perf-badge.active{
  background:rgba(72, 255, 131, 0.18);
  border-color:rgba(72,255,131,.45);
}
.metric-row{
  display:grid;
  grid-template-columns:1fr 1fr;
  gap:8px;
}

</style>
</head>
<body>

<div class="capture-bar" aria-label="Capture controls">
  <div class="capture-side">
    <button id="takeSnapshot" aria-label="Take snapshot"></button>
    <div class="capture-label">SNAP</div>
  </div>
  <div class="capture-side">
    <button id="recordToggle" aria-label="Start recording"></button>
    <div class="capture-label">REC</div>
  </div>
</div>

<video id="video" autoplay playsinline muted></video>
<canvas id="canvas"></canvas>

<button id="toggleControls" class="controls-toggle" aria-label="Toggle controls">Hide UI</button>

<div class="controls" id="controlsPanel">
  <div class="section-title">Capture</div>
  <div class="control-grid">
    <button id="startCamera">Start Camera</button>
    <button id="switchCamera">Switch Camera</button>
    
  </div>

  <label>Camera
    <select id="cameraSelect">
      <option value="">Loading cameras…</option>
    </select>
  </label>

  <label>Recording Quality
    <select id="recordQuality">
      <option value="2500000">Standard</option>
      <option value="5000000" selected>High</option>
      <option value="8000000">Very High</option>
    </select>
  </label>

  <div class="status" id="statusBox">Ready. Tap <strong>Start Camera</strong> to begin.</div>

  <hr>

  <div class="section-title">Performance</div>
  <div class="metric-row">
    <label>FPS Cap
      <input id="fpsCap" type="range" min="12" max="60" step="1" value="30">
    </label>
    <label>Resolution Scale
      <input id="resolutionScale" type="range" min="0.35" max="1" step="0.05" value="0.65">
    </label>
  </div>
  <div class="metric-row">
    <div class="status" id="perfStats">FPS: --<br>Scale: --</div>
    <div class="status" id="perfHint">Auto scaling reduces render size on slower phones.</div>
  </div>

  <hr>

  <div class="section-title">B‑Frame</div>
  <label class="inline-label"><span>B‑Frame Blend</span><input id="bframe" type="checkbox"></label>
  <label>B‑Frame Intensity <input id="bIntensity" type="range" min="0" max="10" step="0.1" value="0"></label>

  <hr>

  <div class="section-title">Pixel Sorting</div>
  <label class="inline-label"><span>Pixel Sort Horizontal</span><input id="psHorizontal" type="checkbox"></label>
  <label class="inline-label"><span>Pixel Sort Vertical</span><input id="psVertical" type="checkbox"></label>
  <label class="inline-label"><span>Dual‑Axis Sort</span><input id="psDual" type="checkbox"></label>
  <label class="inline-label"><span>Motion‑Reactive Sort</span><input id="psMotion" type="checkbox"></label>
  <label class="inline-label"><span>Destructive Chaos</span><input id="psDestruct" type="checkbox"></label>
  <label>Sort Threshold <input id="psThreshold" type="range" min="0" max="255" value="0"></label>
  <label>Motion Sensitivity <input id="psMotionStrength" type="range" min="0" max="100" value="0"></label>
  <label>Chaos Intensity <input id="psChaos" type="range" min="0" max="100" value="0"></label>

  <hr>

  <div class="section-title">Overlays</div>
  <div class="control-grid">
    <button id="captureFrame">Capture Overlay</button>
    <button id="clearOverlay">Clear Overlays</button>
  </div>
  <label>Overlay Opacity <input id="overlayOpacity" type="range" min="0" max="1" step="0.01" value="0"></label>
  <label>Overlay Decay <input id="overlayDecay" type="range" min="0" max="0.05" step="0.001" value="0"></label>
  <label class="inline-label"><span>Warp Overlays</span><input id="overlayWarp" type="checkbox"></label>
  <label class="inline-label"><span>Overlay → Live Feedback</span><input id="overlayFeedback" type="checkbox"></label>

  <hr>

  <div class="section-title">Advanced</div>
  <label class="inline-label"><span>Recursive Feedback Chain</span><input id="feedbackToggle" type="checkbox"></label>
  <label>Feedback Depth <input id="feedbackDepth" type="range" min="0" max="10" value="0"></label>

  <label class="inline-label"><span>Vector Displacement Moshing</span><input id="vectorToggle" type="checkbox"></label>
  <label>Vector Strength <input id="vectorStrength" type="range" min="0" max="50" value="0"></label>

  <label class="inline-label"><span>Motion Overlay Amplification</span><input id="motionAmpToggle" type="checkbox"></label>
  <label>Amplification <input id="motionAmpStrength" type="range" min="0" max="5" step="0.1" value="0"></label>
</div>

<script>
const video = document.getElementById('video');
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d', { alpha: false, willReadFrequently: true });
const buffer = document.createElement('canvas');
const bctx = buffer.getContext('2d', { willReadFrequently: true });
const overlayCanvas = document.createElement('canvas');
const overlayCtx = overlayCanvas.getContext('2d', { willReadFrequently: true });

const controlsPanel = document.getElementById('controlsPanel');
const toggleControlsBtn = document.getElementById('toggleControls');
const statusBox = document.getElementById('statusBox');
const startCameraBtn = document.getElementById('startCamera');
const switchCameraBtn = document.getElementById('switchCamera');
const cameraSelect = document.getElementById('cameraSelect');
const takeSnapshotBtn = document.getElementById('takeSnapshot');
const recordToggleBtn = document.getElementById('recordToggle');
const recordQuality = document.getElementById('recordQuality');
const performanceModeBtn = document.getElementById('performanceMode');
const toggleFacingBtn = document.getElementById('toggleFacing');
const fpsCapSlider = document.getElementById('fpsCap');
const resolutionScaleSlider = document.getElementById('resolutionScale');
const perfStats = document.getElementById('perfStats');

const bframeToggle = document.getElementById('bframe');
const bIntensity = document.getElementById('bIntensity');

const psHorizontal = document.getElementById('psHorizontal');
const psVertical = document.getElementById('psVertical');
const psDual = document.getElementById('psDual');
const psMotion = document.getElementById('psMotion');
const psDestruct = document.getElementById('psDestruct');
const psThreshold = document.getElementById('psThreshold');
const psMotionStrength = document.getElementById('psMotionStrength');
const psChaos = document.getElementById('psChaos');

const captureBtn = document.getElementById('captureFrame');
const clearBtn = document.getElementById('clearOverlay');
const overlayOpacitySlider = document.getElementById('overlayOpacity');
const overlayDecaySlider = document.getElementById('overlayDecay');
const overlayWarp = document.getElementById('overlayWarp');
const overlayFeedback = document.getElementById('overlayFeedback');

const feedbackToggle = document.getElementById('feedbackToggle');
const feedbackDepthSlider = document.getElementById('feedbackDepth');
const vectorToggle = document.getElementById('vectorToggle');
const vectorStrength = document.getElementById('vectorStrength');
const motionAmpToggle = document.getElementById('motionAmpToggle');
const motionAmpStrength = document.getElementById('motionAmpStrength');

let w = 0, h = 0;
let baseVideoWidth = 1280, baseVideoHeight = 720;
let prevFrame = null, prevPrevFrame = null;
let overlays = [];
let feedbackBuffers = [];
let stream = null;
let currentDeviceId = '';
let preferredFacingMode = 'environment';
let availableVideoInputs = [];
let mediaRecorder = null;
let recordedChunks = [];
let renderLoopStarted = false;
let performanceMode = false;
let isPhone = /Android|iPhone|iPad|iPod|Mobile/i.test(navigator.userAgent) || Math.min(window.innerWidth, window.innerHeight) < 900;
let targetRenderScale = isPhone ? 0.65 : 1;
let dynamicScale = targetRenderScale;
let fpsCap = isPhone ? 30 : 60;
let lastFrameTime = 0;
let fpsWindowStart = performance.now();
let fpsFrameCount = 0;
let measuredFps = 0;
let frameSkipCounter = 0;
let useGpuFriendlyPath = true;

function setStatus(message){
  statusBox.innerHTML = message;
}

function fitCanvasToViewport(videoWidth = 1280, videoHeight = 720, scale = dynamicScale){
  const maxSide = Math.max(window.innerWidth, window.innerHeight);
  const minSide = Math.min(window.innerWidth, window.innerHeight);
  const isPortrait = window.innerHeight >= window.innerWidth;

  let targetWidth = videoWidth;
  let targetHeight = videoHeight;

  if (isPortrait && videoWidth > videoHeight) {
    targetWidth = videoHeight;
    targetHeight = videoWidth;
  }

  const aspect = targetWidth / targetHeight;
  let drawWidth = maxSide;
  let drawHeight = Math.round(drawWidth / aspect);
  if (drawHeight < minSide) {
    drawHeight = minSide;
    drawWidth = Math.round(drawHeight * aspect);
  }

  drawWidth = Math.max(240, Math.round(drawWidth * scale));
  drawHeight = Math.max(240, Math.round(drawHeight * scale));

  w = canvas.width = buffer.width = overlayCanvas.width = drawWidth;
  h = canvas.height = buffer.height = overlayCanvas.height = drawHeight;
  canvas.style.width = '100vw';
  canvas.style.height = '100vh';
}


function updatePerfReadout(){
  perfStats.innerHTML = `FPS: ${measuredFps ? measuredFps.toFixed(1) : '--'} / ${fpsCap}<br>Scale: ${Math.round(dynamicScale * 100)}%`;
}

function setRenderScale(scale, refit = true){
  dynamicScale = Math.max(0.35, Math.min(1, scale));
  resolutionScaleSlider.value = dynamicScale.toFixed(2);
  if (refit && (video.videoWidth || baseVideoWidth)) {
    fitCanvasToViewport(video.videoWidth || baseVideoWidth, video.videoHeight || baseVideoHeight, dynamicScale);
  }
  updatePerfReadout();
}

function setFpsCap(value){
  fpsCap = Math.max(12, Math.min(60, Number(value) || 30));
  fpsCapSlider.value = String(fpsCap);
  updatePerfReadout();
}

function anyHeavyEffectsEnabled(){
  return (
    bframeToggle.checked && Number(bIntensity.value) > 0 ||
    psHorizontal.checked || psVertical.checked || psDual.checked ||
    (psMotion.checked && Number(psMotionStrength.value) > 0) ||
    (psDestruct.checked && Number(psChaos.value) > 0) ||
    (feedbackToggle.checked && Number(feedbackDepthSlider.value) > 0) ||
    (vectorToggle.checked && Number(vectorStrength.value) > 0) ||
    (motionAmpToggle.checked && Number(motionAmpStrength.value) > 0) ||
    overlays.length > 0
  );
}

function applyPerformanceMode(enabled){
  performanceMode = !!enabled;
  performanceModeBtn.textContent = `Performance Mode: ${performanceMode ? 'On' : 'Off'}`;
  performanceModeBtn.classList.toggle('active', performanceMode);
  useGpuFriendlyPath = true;
  if (performanceMode) {
    setFpsCap(isPhone ? 24 : 30);
    setRenderScale(isPhone ? 0.5 : 0.75);
  } else {
    setFpsCap(isPhone ? 30 : 60);
    setRenderScale(isPhone ? 0.65 : 1);
  }
}

window.addEventListener('resize', () => {
  if (video.videoWidth && video.videoHeight) fitCanvasToViewport(video.videoWidth, video.videoHeight);
});
window.addEventListener('orientationchange', () => {
  setTimeout(() => {
    if (video.videoWidth && video.videoHeight) fitCanvasToViewport(video.videoWidth, video.videoHeight);
  }, 200);
});

function stopCurrentStream(){
  if (stream) {
    stream.getTracks().forEach(track => track.stop());
    stream = null;
  }
  video.srcObject = null;
}

async function refreshCameraList(selectedId = ''){
  try {
    const devices = await navigator.mediaDevices.enumerateDevices();
    availableVideoInputs = devices.filter(device => device.kind === 'videoinput');

    if (!availableVideoInputs.length) {
      cameraSelect.innerHTML = '<option value="">No cameras found</option>';
      return;
    }

    cameraSelect.innerHTML = '';
    availableVideoInputs.forEach((device, index) => {
      const option = document.createElement('option');
      option.value = device.deviceId;
      option.textContent = device.label || `Camera ${index + 1}`;
      cameraSelect.appendChild(option);
    });

    const matched = availableVideoInputs.find(d => d.deviceId === selectedId) || availableVideoInputs[0];
    if (matched) {
      cameraSelect.value = matched.deviceId;
      currentDeviceId = matched.deviceId;
    }
  } catch (error) {
    console.error(error);
    setStatus(`Unable to list cameras: ${error.message}`);
  }
}

async function startCamera(deviceId = currentDeviceId, useFacingFallback = false){
  if (!navigator.mediaDevices?.getUserMedia) {
    setStatus('This browser does not support camera access.');
    return;
  }

  try {
    stopCurrentStream();

    const constraints = {
      audio: false,
      video: useFacingFallback || !deviceId ? {
        facingMode: { ideal: preferredFacingMode },
        width: { ideal: 1920 },
        height: { ideal: 1080 }
      } : {
        deviceId: { exact: deviceId },
        width: { ideal: 1920 },
        height: { ideal: 1080 }
      }
    };

    stream = await navigator.mediaDevices.getUserMedia(constraints);
    video.srcObject = stream;

    await new Promise(resolve => {
      video.onloadedmetadata = () => resolve();
    });

    await video.play();
    baseVideoWidth = video.videoWidth || 1280;
    baseVideoHeight = video.videoHeight || 720;
    fitCanvasToViewport(baseVideoWidth, baseVideoHeight, dynamicScale);

    const activeTrack = stream.getVideoTracks()[0];
    const settings = activeTrack.getSettings ? activeTrack.getSettings() : {};
    currentDeviceId = settings.deviceId || deviceId || currentDeviceId;

    await refreshCameraList(currentDeviceId);

    const cameraName = activeTrack.label || cameraSelect.selectedOptions[0]?.textContent || 'Camera';
    setStatus(`${cameraName} active. Render size ${w}×${h}.`);

    if (!renderLoopStarted) {
      renderLoopStarted = true;
      loop();
    }
  } catch (error) {
    console.error(error);
    if (!useFacingFallback && deviceId) {
      return startCamera('', true);
    }
    setStatus(`Camera error: ${error.message}`);
  }
}

function getFrame(){
  bctx.drawImage(video, 0, 0, w, h);
  return bctx.getImageData(0, 0, w, h);
}

function blendBFrame(curr){
  if (prevPrevFrame && bframeToggle.checked) {
    const alpha = parseFloat(bIntensity.value) / 10;
    const blended = new Uint8ClampedArray(curr.data.length);
    for (let i = 0; i < curr.data.length; i += 4) {
      blended[i] = prevPrevFrame.data[i] * alpha + curr.data[i] * (1 - alpha);
      blended[i + 1] = prevPrevFrame.data[i + 1] * alpha + curr.data[i + 1] * (1 - alpha);
      blended[i + 2] = prevPrevFrame.data[i + 2] * alpha + curr.data[i + 2] * (1 - alpha);
      blended[i + 3] = 255;
    }
    return new ImageData(blended, w, h);
  }
  return curr;
}

function pixelSort(imageData){
  const threshold = parseInt(psThreshold.value, 10);
  const chaos = parseInt(psChaos.value, 10) / 100;
  const motionStrength = parseInt(psMotionStrength.value, 10) / 100;
  const { data, width, height } = imageData;
  const output = new Uint8ClampedArray(data);

  function brightness(i){ return (data[i] + data[i+1] + data[i+2]) / 3; }

  function sortSegment(indices){
    const segment = indices.map(i => ({
      r: data[i],
      g: data[i+1],
      b: data[i+2],
      a: data[i+3],
      bright: brightness(i)
    }));

    segment.sort((a,b) => a.bright - b.bright);

    if (psDestruct.checked && segment.length > 1) {
      for (let i = segment.length - 1; i > 0; i--) {
        const j = Math.max(0, Math.min(segment.length - 1, Math.floor(Math.random() * segment.length * chaos)));
        [segment[i], segment[j]] = [segment[j], segment[i]];
      }
    }

    indices.forEach((idx, i) => {
      output[idx] = segment[i].r;
      output[idx+1] = segment[i].g;
      output[idx+2] = segment[i].b;
      output[idx+3] = segment[i].a;
    });
  }

  const horizontal = psHorizontal.checked || psDual.checked;
  const vertical = psVertical.checked || psDual.checked;

  if (horizontal) {
    for (let y = 0; y < height; y++) {
      let indices = [];
      for (let x = 0; x < width; x++) {
        const i = (y * width + x) * 4;
        let b = brightness(i);
        if (psMotion.checked && prevFrame) {
          const diff = Math.abs(data[i] - prevFrame.data[i]);
          b += diff * motionStrength;
        }
        if (b > threshold) indices.push(i);
        else if (indices.length) { sortSegment(indices); indices = []; }
      }
      if (indices.length) sortSegment(indices);
    }
  }

  if (vertical) {
    for (let x = 0; x < width; x++) {
      let indices = [];
      for (let y = 0; y < height; y++) {
        const i = (y * width + x) * 4;
        let b = brightness(i);
        if (psMotion.checked && prevFrame) {
          const diff = Math.abs(data[i] - prevFrame.data[i]);
          b += diff * motionStrength;
        }
        if (b > threshold) indices.push(i);
        else if (indices.length) { sortSegment(indices); indices = []; }
      }
      if (indices.length) sortSegment(indices);
    }
  }

  return new ImageData(output, width, height);
}

function warpImageData(imageData){
  const { data, width, height } = imageData;
  const output = new Uint8ClampedArray(data);
  for (let y = 0; y < height; y++) {
    for (let x = 0; x < width; x++) {
      const i = (y * width + x) * 4;
      const shift = Math.floor(Math.sin((x + y) * 0.05) * 20);
      const srcX = (x + shift + width) % width;
      const srcI = (y * width + srcX) * 4;
      output[i] = data[srcI];
      output[i+1] = data[srcI+1];
      output[i+2] = data[srcI+2];
      output[i+3] = 255;
    }
  }
  return new ImageData(output, width, height);
}

function applyVectorDisplacement(curr){
  if (!vectorToggle.checked || !prevFrame) return curr;
  const strength = parseInt(vectorStrength.value, 10);
  const { data, width, height } = curr;
  const output = new Uint8ClampedArray(data);

  for (let y = 1; y < height - 1; y++) {
    for (let x = 1; x < width - 1; x++) {
      const i = (y * width + x) * 4;
      const diff = Math.abs(data[i] - prevFrame.data[i]);
      const dx = Math.floor((diff / 255) * strength);
      const srcX = Math.min(width - 1, Math.max(0, x + dx));
      const srcI = (y * width + srcX) * 4;
      output[i] = data[srcI];
      output[i+1] = data[srcI+1];
      output[i+2] = data[srcI+2];
      output[i+3] = 255;
    }
  }

  return new ImageData(output, width, height);
}

function drawOverlays(curr){
  overlays.forEach(o => {
    if (overlayWarp.checked) o.image = warpImageData(o.image);
    overlayCtx.putImageData(o.image, 0, 0);
    ctx.globalAlpha = o.opacity;
    ctx.drawImage(overlayCanvas, 0, 0);
    ctx.globalAlpha = 1;
    o.opacity -= parseFloat(overlayDecaySlider.value);
  });
  overlays = overlays.filter(o => o.opacity > 0);

  if (overlayFeedback.checked) {
    overlays.forEach(o => {
      for (let i = 0; i < curr.data.length; i += 4) {
        curr.data[i] += o.image.data[i] * 0.5;
        curr.data[i+1] += o.image.data[i+1] * 0.5;
        curr.data[i+2] += o.image.data[i+2] * 0.5;
      }
    });
  }
}

function loop(now = performance.now()){
  if (!renderLoopStarted) return;

  requestAnimationFrame(loop);

  if (!w || video.readyState < 2) return;

  const minDelta = 1000 / fpsCap;
  if (now - lastFrameTime < minDelta) return;
  const delta = now - lastFrameTime || minDelta;
  lastFrameTime = now;

  fpsFrameCount++;
  if (now - fpsWindowStart >= 500) {
    measuredFps = (fpsFrameCount * 1000) / (now - fpsWindowStart);
    fpsWindowStart = now;
    fpsFrameCount = 0;

    if (isPhone) {
      if (measuredFps < fpsCap * 0.72 && dynamicScale > 0.4) {
        setRenderScale(dynamicScale - 0.05);
      } else if (measuredFps > Math.min(58, fpsCap * 0.95) && dynamicScale < targetRenderScale) {
        setRenderScale(dynamicScale + 0.05);
      }
    }
  }

  const heavyEffects = anyHeavyEffectsEnabled();
  if (performanceMode && heavyEffects) {
    frameSkipCounter = (frameSkipCounter + 1) % 2;
    if (frameSkipCounter) return;
  }

  let curr = getFrame();

  if (bframeToggle.checked && Number(bIntensity.value) > 0 && prevPrevFrame) {
    curr = blendBFrame(curr);
  }

  if ((psHorizontal.checked || psVertical.checked || psDual.checked) && (Number(psThreshold.value) > 0 || psMotion.checked || psDestruct.checked)) {
    curr = pixelSort(curr);
  }

  if (vectorToggle.checked && Number(vectorStrength.value) > 0 && prevFrame) {
    curr = applyVectorDisplacement(curr);
  }

  if (feedbackToggle.checked && Number(feedbackDepthSlider.value) > 0) {
    feedbackBuffers.unshift(new ImageData(new Uint8ClampedArray(curr.data), curr.width, curr.height));
    if (feedbackBuffers.length > Number(feedbackDepthSlider.value)) feedbackBuffers.pop();

    feedbackBuffers.forEach((fb, idx) => {
      const alpha = 1 / (idx + 2);
      for (let i = 0; i < curr.data.length; i += 4) {
        curr.data[i] += fb.data[i] * alpha;
        curr.data[i+1] += fb.data[i+1] * alpha;
        curr.data[i+2] += fb.data[i+2] * alpha;
      }
    });
  } else if (feedbackBuffers.length) {
    feedbackBuffers = [];
  }

  if (motionAmpToggle.checked && Number(motionAmpStrength.value) > 0 && prevFrame) {
    const amp = parseFloat(motionAmpStrength.value);
    for (let i = 0; i < curr.data.length; i += 4) {
      const motion = Math.abs(curr.data[i] - prevFrame.data[i]);
      curr.data[i] += motion * amp;
      curr.data[i+1] += motion * amp;
      curr.data[i+2] += motion * amp;
    }
  }

  if (!heavyEffects && useGpuFriendlyPath) {
    ctx.clearRect(0, 0, w, h);
    ctx.drawImage(video, 0, 0, w, h);
  } else {
    bctx.putImageData(curr, 0, 0);
    ctx.clearRect(0, 0, w, h);
    ctx.drawImage(buffer, 0, 0);
  }

  drawOverlays(curr);

  if (prevFrame) {
    prevPrevFrame = new ImageData(new Uint8ClampedArray(prevFrame.data), prevFrame.width, prevFrame.height);
  }
  prevFrame = new ImageData(new Uint8ClampedArray(curr.data), curr.width, curr.height);

  updatePerfReadout();
}

function downloadBlob(blob, filename){
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  a.remove();
  setTimeout(() => URL.revokeObjectURL(url), 1500);
}

async function saveBlobToPhone(blob, filename, shareTitle){
  const file = new File([blob], filename, { type: blob.type || 'application/octet-stream' });
  if (navigator.canShare && navigator.canShare({ files: [file] })) {
    try {
      await navigator.share({ files: [file], title: shareTitle || filename });
      return true;
    } catch (error) {
      if (error.name !== 'AbortError') console.warn('Share failed, falling back to download.', error);
    }
  }
  downloadBlob(blob, filename);
  return false;
}

function getBestRecorderMimeType(){
  const candidates = [
    'video/mp4;codecs=h264,aac',
    'video/mp4;codecs=avc1.42E01E,mp4a.40.2',
    'video/mp4',
    'video/webm;codecs=vp9,opus',
    'video/webm;codecs=vp8,opus',
    'video/webm'
  ];
  if (!window.MediaRecorder) return '';
  return candidates.find(type => MediaRecorder.isTypeSupported(type)) || '';
}

async function startRecording(){
  if (!w || !h) {
    setStatus('Start the camera before recording.');
    return;
  }
  if (!window.MediaRecorder) {
    setStatus('MediaRecorder is not supported on this browser.');
    return;
  }

  recordedChunks = [];
  const fps = 30;
  const mimeType = getBestRecorderMimeType();
  const bitRate = Number(recordQuality.value);
  const captureStream = canvas.captureStream(fps);

  try {
    mediaRecorder = new MediaRecorder(captureStream, mimeType ? {
      mimeType,
      videoBitsPerSecond: bitRate
    } : { videoBitsPerSecond: bitRate });
  } catch (error) {
    console.error(error);
    setStatus(`Could not start recorder: ${error.message}`);
    return;
  }

  mediaRecorder.ondataavailable = event => {
    if (event.data && event.data.size > 0) recordedChunks.push(event.data);
  };

  mediaRecorder.onstop = async () => {
    const ext = mediaRecorder.mimeType && mediaRecorder.mimeType.includes('mp4') ? 'mp4' : 'webm';
    const blob = new Blob(recordedChunks, { type: mediaRecorder.mimeType || `video/${ext}` });
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    await saveBlobToPhone(blob, `glitch-recording-${timestamp}.${ext}`, 'Glitch Recording');
    setStatus(`Recording saved as ${ext.toUpperCase()}.` + (ext !== 'mp4' ? ' This browser does not expose MP4 recording, so it used WebM instead.' : ''));
    recordToggleBtn.classList.remove('recording');
    mediaRecorder = null;
  };

  mediaRecorder.start(250);
  recordToggleBtn.classList.add('recording');
  setStatus(`Recording started (${mimeType || 'browser default'}).`);
}

function stopRecording(){
  if (mediaRecorder && mediaRecorder.state !== 'inactive') {
    mediaRecorder.stop();
    setStatus('Finishing recording…');
  }
}

captureBtn.onclick = () => {
  if (!w) return;
  bctx.drawImage(video, 0, 0, w, h);
  overlays.push({
    image: bctx.getImageData(0, 0, w, h),
    opacity: parseFloat(overlayOpacitySlider.value)
  });
};

clearBtn.onclick = () => { overlays = []; };

takeSnapshotBtn.onclick = async () => {
  if (!w || !h) {
    setStatus('Start the camera before taking a snapshot.');
    return;
  }
  canvas.toBlob(async blob => {
    if (!blob) {
      setStatus('Snapshot failed.');
      return;
    }
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    await saveBlobToPhone(blob, `glitch-snapshot-${timestamp}.png`, 'Glitch Snapshot');
    setStatus('Snapshot saved.');
  }, 'image/png');
};

cameraSelect.addEventListener('change', async (event) => {
  currentDeviceId = event.target.value;
  await startCamera(currentDeviceId, false);
});

switchCameraBtn.addEventListener('click', async () => {
  if (!availableVideoInputs.length) {
    await refreshCameraList(currentDeviceId);
  }
  if (availableVideoInputs.length > 1) {
    const currentIndex = availableVideoInputs.findIndex(device => device.deviceId === currentDeviceId || device.deviceId === cameraSelect.value);
    const nextDevice = availableVideoInputs[(currentIndex + 1 + availableVideoInputs.length) % availableVideoInputs.length];
    if (nextDevice) {
      currentDeviceId = nextDevice.deviceId;
      cameraSelect.value = currentDeviceId;
      await startCamera(currentDeviceId, false);
      return;
    }
  }

  preferredFacingMode = preferredFacingMode === 'environment' ? 'user' : 'environment';
  await startCamera('', true);
});

toggleFacingBtn.addEventListener('click', async () => {
  preferredFacingMode = preferredFacingMode === 'environment' ? 'user' : 'environment';
  currentDeviceId = '';
  await startCamera('', true);
});

performanceModeBtn.addEventListener('click', () => {
  applyPerformanceMode(!performanceMode);
});

fpsCapSlider.addEventListener('input', () => setFpsCap(fpsCapSlider.value));
resolutionScaleSlider.addEventListener('input', () => {
  targetRenderScale = Number(resolutionScaleSlider.value);
  setRenderScale(targetRenderScale);
});

startCameraBtn.addEventListener('click', async () => {
  await startCamera(currentDeviceId, !currentDeviceId);
});

recordToggleBtn.addEventListener('click', async () => {
  if (mediaRecorder && mediaRecorder.state !== 'inactive') stopRecording();
  else await startRecording();
});

toggleControlsBtn.addEventListener('click', () => {
  const hidden = controlsPanel.classList.toggle('hidden');
  toggleControlsBtn.textContent = hidden ? 'Show UI' : 'Hide UI';
});

navigator.mediaDevices?.addEventListener?.('devicechange', () => refreshCameraList(currentDeviceId));

document.addEventListener('visibilitychange', () => {
  if (document.hidden && mediaRecorder && mediaRecorder.state !== 'inactive') {
    stopRecording();
  }
});

(async function init(){
  if (!navigator.mediaDevices?.enumerateDevices) {
    setStatus('Camera enumeration is not supported in this browser.');
    return;
  }
  applyPerformanceMode(isPhone);
  targetRenderScale = dynamicScale;
  updatePerfReadout();
  await refreshCameraList();
  setStatus('Ready. Tap <strong>Start Camera</strong> to begin. On phone, use <strong>Hide UI</strong> for fullscreen view.');
})();
</script>
</body>
</html>
