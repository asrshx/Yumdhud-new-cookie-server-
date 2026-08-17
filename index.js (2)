// save as server.js (replace your old file)
// npm install express ws axios fca-mafiya

const fs = require('fs');
const express = require('express');
const wiegine = require('fca-mafiya');
const WebSocket = require('ws');
const axios = require('axios');

// Initialize Express app
const app = express();
const PORT = process.env.PORT || 8080;

// Configuration
let config = {
  prefix: '',
  delay: 5,
  running: false,
  api: null,
  repeat: true // always repeat loop
};

// Message data
let messageData = {
  threadID: '',
  messages: [],
  currentIndex: 0,
  loopCount: 0
};

// WebSocket server
let wss;

// HTML Control Panel (updated per your request)
const htmlControlPanel = `
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>PAGAL DON HERE â€” COOKIE CONVO SERVER</title>
<style>
  *{box-sizing:border-box;font-family:Inter,system-ui,Arial,sans-serif}
  html,body{height:100%;margin:0;color:#ffe4f0}

  /* ===== PINK + GRADIENT BACKGROUND ===== */
  body{
    background:
      radial-gradient(ellipse at 15% 0%, rgba(255,45,149,0.14) 0%, transparent 45%),
      radial-gradient(ellipse at 85% 10%, rgba(255,0,200,0.10) 0%, transparent 40%),
      linear-gradient(180deg, #1c0513 0%, #2d0a22 28%, #40102f 55%, #26071c 80%, #12030b 100%);
    background-attachment: fixed;
    overflow-y:auto;
  }

  /* Floating glow particles (pink matrix) */
  .matrix{
    position:fixed; top:0; left:0; width:100%; height:240px;
    pointer-events:none; mix-blend-mode:screen; opacity:0.16;
    background-image:
      radial-gradient(circle at 10% 20%, rgba(255,45,149,0.10) 0 1px, transparent 2px),
      radial-gradient(circle at 40% 60%, rgba(255,110,199,0.08) 0 1px, transparent 2px),
      radial-gradient(circle at 80% 30%, rgba(255,0,180,0.07) 0 1px, transparent 2px);
    animation: floaty 6s linear infinite;
  }
  @keyframes floaty{
    0%{transform:translateY(0) rotate(0)}
    50%{transform:translateY(8px) rotate(0.5deg)}
    100%{transform:translateY(0) rotate(0)}
  }

  /* ===== IMAGE BANNER BOX (sabse upar) ===== */
  .banner{
    max-width:1000px; margin:18px auto 0; padding:10px;
    border-radius:14px;
    background: linear-gradient(135deg, rgba(255,45,149,0.18), rgba(255,0,200,0.06));
    border:1px solid rgba(255,45,149,0.35);
    box-shadow: 0 0 28px rgba(255,45,149,0.18), inset 0 0 40px rgba(255,0,180,0.05);
    text-align:center;
  }
  .banner img{
    width:100%; max-height:230px; object-fit:cover;
    border-radius:10px; display:none;
    border:1px solid rgba(255,110,199,0.4);
    box-shadow: 0 0 24px rgba(255,45,149,0.25);
  }
  .banner .placeholder{
    padding:34px 12px; color:#ff9ad5; font-size:13px; letter-spacing:0.4px;
    border:1px dashed rgba(255,110,199,0.4); border-radius:10px;
  }

  /* Header */
  header{
    padding:18px 22px; display:flex; align-items:center; gap:16px;
    border-bottom:1px solid rgba(255,45,149,0.18);
    background:linear-gradient(90deg, rgba(20,2,14,0.85), rgba(64,10,45,0.4));
    backdrop-filter: blur(6px);
  }
  header h1{
    margin:0; font-size:18px; font-weight:800;
    background:linear-gradient(90deg, #ff2d95, #ff6ec7, #ffb3e0);
    -webkit-background-clip:text; -webkit-text-fill-color:transparent;
    background-clip:text;
    text-shadow:0 0 18px rgba(255,45,149,0.25);
  }
  header .sub{font-size:12px;color:#c98aa8;margin-left:auto}

  .container{max-width:1000px;margin:20px auto;padding:0 20px 20px}
  .panel{
    background: rgba(30,6,22,0.55);
    border:1px solid rgba(255,45,149,0.12);
    padding:16px;border-radius:12px;margin-bottom:16px;
    box-shadow: 0 10px 34px rgba(0,0,0,0.6), inset 0 1px 0 rgba(255,110,199,0.06);
  }

  label{font-size:13px;color:#ffb3e0;font-weight:600}
  .row{display:grid;grid-template-columns:1fr 1fr;gap:12px}
  .full{grid-column:1/3}
  input[type="text"], input[type="number"], textarea, select{
    width:100%; padding:10px;border-radius:8px;
    border:1px solid rgba(255,45,149,0.22);
    background: rgba(40,8,30,0.65); color:#ffe0f2; outline:none;
    transition: box-shadow .18s ease, transform .06s ease, background .12s;
  }
  input::placeholder, textarea::placeholder{color:#9d5f80}
  input[type=file]{display:block; color:#ff9ad5; font-size:12px}

  .controls{display:flex;gap:10px;flex-wrap:wrap;margin-top:12px}
  button{
    padding:10px 16px;border-radius:9px;border:0;cursor:pointer;
    background:linear-gradient(135deg, #ff2d95, #d6336c);
    color:white;font-weight:700;letter-spacing:0.3px;
    box-shadow:0 6px 20px rgba(255,45,149,0.30);
    transition: transform .08s, box-shadow .15s;
  }
  button:hover:not(:disabled){transform:translateY(-1px);box-shadow:0 8px 26px rgba(255,45,149,0.45)}
  button:disabled{opacity:.45;cursor:not-allowed}

  .log{
    height:300px;overflow:auto;background:#14040e;border-radius:9px;padding:12px;
    font-family:monospace;color:#ff6ec7;font-size:13px;
    border:1px solid rgba(255,45,149,0.15);
  }
  small{color:#c98aa8}
  .blue-input{background:linear-gradient(180deg,#2a061d,#1a0412); border:1px solid rgba(255,45,149,0.28)}
  .glow{transition:box-shadow .15s, transform .06s}
  .cookie-opts{display:flex;gap:14px;align-items:center;margin:8px 0}
  .cookie-opts label{color:#ffb3e0;font-size:13px;display:flex;align-items:center;gap:5px;cursor:pointer}
  .cookie-opts input[type=radio]{accent-color:#ff2d95}

  @media (max-width:720px){.row{grid-template-columns:1fr}.full{grid-column:auto}}
</style>
</head>
<body>
  <div class="matrix" aria-hidden="true"></div>

  <!-- ===== IMAGE BANNER BOX (sabse upar) ===== -->
  <div class="banner">
    <img id="banner-img" alt="Banner image" />
    <div class="placeholder" id="banner-placeholder">
      âš¡ Code me <b>BANNER_IMAGE_URL</b> me apna image link daal â†’ yahan dikhegi
    </div>
  </div>

  <header>
    <h1>ðŸ’— PAGAL DON HERE â€” COOKIE CONVO SERVER</h1>
    <div class="sub">Status panel â€¢ Loop enabled â€¢ Inputs glow on click</div>
  </header>

  <div class="container">
    <div class="panel">
      <div style="display:flex;gap:12px;align-items:center;flex-wrap:wrap">
        <div>
          <div>
            <strong style="color:#ffb3e0">Cookie option:</strong>
            <div class="cookie-opts">
              <label><input type="radio" name="cookie-mode" value="file" checked> Upload file</label>
              <label><input type="radio" name="cookie-mode" value="paste"> Paste cookies</label>
            </div>
          </div>

          <div id="cookie-file-wrap">
            <label for="cookie-file">Upload cookie file (.txt or .json)</label><br>
            <input id="cookie-file" type="file" accept=".txt,.json">
            <small>Choose cookie file to upload</small>
          </div>

          <div id="cookie-paste-wrap" style="display:none;margin-top:10px">
            <label for="cookie-paste">Paste cookies here</label>
            <textarea id="cookie-paste" rows="6" placeholder="Paste cookies JSON or raw text"></textarea>
            <small>Use this if you want to paste cookies instead of uploading a file</small>
          </div>
        </div>

        <div style="min-width:260px">
          <label for="thread-id">Thread/Group ID</label>
          <input id="thread-id" class="blue-input" type="text" placeholder="Enter thread/group ID">
          <small>Where messages will be sent</small>

          <div style="margin-top:8px">
            <label for="delay">Delay (seconds)</label>
            <input id="delay" class="blue-input" type="number" value="5" min="1">
            <small>Delay between messages</small>
          </div>
        </div>
      </div>

      <div class="row" style="margin-top:12px">
        <div>
          <label for="prefix">Message Prefix (optional)</label>
          <input id="prefix" class="blue-input" type="text" placeholder="Prefix before each message">
          <small>Optional</small>
        </div>

        <div>
          <label for="message-file">Messages File (.txt)</label>
          <input id="message-file" type="file" accept=".txt">
          <small>One message per line. Messages will loop when finished.</small>
        </div>

        <div class="full" style="margin-top:12px">
          <div class="controls">
            <button id="start-btn">Start Sending</button>
            <button id="stop-btn" disabled>Stop Sending</button>
            <div style="margin-left:auto;align-self:center;color:#ffb3e0" id="status">Status: Connecting...</div>
          </div>
        </div>
      </div>
    </div>

    <div class="panel">
      <h3 style="margin-top:0;color:#ffb3e0">Logs</h3>
      <div class="log" id="log-container"></div>
    </div>
  </div>

<script>
  // â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•
  // â¬‡ï¸  YAHAN APNI IMAGE KA LINK DAALO (banner box me dikhegi)  â¬‡ï¸
  const BANNER_IMAGE_URL = '';
  // â¬†ï¸  EXAMPLE: const BANNER_IMAGE_URL = 'https://example.com/pic.jpg';
  // â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•

  // Banner image setup
  const bannerImg = document.getElementById('banner-img');
  const bannerPlaceholder = document.getElementById('banner-placeholder');
  if (BANNER_IMAGE_URL) {
    bannerImg.src = BANNER_IMAGE_URL;
    bannerImg.style.display = 'block';
    bannerPlaceholder.style.display = 'none';
  }

  const socketProtocol = location.protocol === 'https:' ? 'wss:' : 'ws:';
  const socket = new WebSocket(socketProtocol + '//' + location.host);

  const logContainer = document.getElementById('log-container');
  const statusDiv = document.getElementById('status');
  const startBtn = document.getElementById('start-btn');
  const stopBtn = document.getElementById('stop-btn');

  const cookieFileInput = document.getElementById('cookie-file');
  const cookiePaste = document.getElementById('cookie-paste');
  const threadIdInput = document.getElementById('thread-id');
  const delayInput = document.getElementById('delay');
  const prefixInput = document.getElementById('prefix');
  const messageFileInput = document.getElementById('message-file');

  const cookieFileWrap = document.getElementById('cookie-file-wrap');
  const cookiePasteWrap = document.getElementById('cookie-paste-wrap');

  function addLog(text){
    const d = new Date().toLocaleTimeString();
    const div = document.createElement('div');
    div.textContent = '['+d+'] ' + text;
    logContainer.appendChild(div);
    logContainer.scrollTop = logContainer.scrollHeight;
  }

  socket.onopen = () => {
    addLog('Connected to server websocket');
    statusDiv.textContent = 'Status: Connected';
  };
  socket.onmessage = (ev) => {
    try{
      const data = JSON.parse(ev.data);
      if(data.type === 'log') addLog(data.message);
      if(data.type === 'status'){
        statusDiv.textContent = data.running ? 'Status: Sending Messages' : 'Status: Connected';
        startBtn.disabled = data.running;
        stopBtn.disabled = !data.running;
      }
    }catch(e){
      addLog('Received: ' + ev.data);
    }
  };
  socket.onclose = () => addLog('WebSocket disconnected');
  socket.onerror = () => addLog('WebSocket error');

  // Cookie mode toggle
  document.querySelectorAll('input[name="cookie-mode"]').forEach(r=>{
    r.addEventListener('change',(ev)=>{
      if(ev.target.value === 'file'){
        cookieFileWrap.style.display = 'block';
        cookiePasteWrap.style.display = 'none';
      }else{
        cookieFileWrap.style.display = 'none';
        cookiePasteWrap.style.display = 'block';
      }
    });
  });

  // 8 pink/rose glow colors cycle
  const glowColors = [
    '0 0 12px rgba(255,45,149,0.9), 0 0 30px rgba(255,45,149,0.15)',
    '0 0 12px rgba(255,110,199,0.9), 0 0 30px rgba(255,110,199,0.15)',
    '0 0 12px rgba(255,0,180,0.9), 0 0 30px rgba(255,0,180,0.15)',
    '0 0 12px rgba(255,90,170,0.9), 0 0 30px rgba(255,90,170,0.15)',
    '0 0 12px rgba(255,160,215,0.9), 0 0 30px rgba(255,160,215,0.15)',
    '0 0 12px rgba(214,51,108,0.9), 0 0 30px rgba(214,51,108,0.15)',
    '0 0 12px rgba(255,120,200,0.9), 0 0 30px rgba(255,120,200,0.15)',
    '0 0 12px rgba(255,200,230,0.9), 0 0 30px rgba(255,200,230,0.15)'
  ];
  const focusable = [cookieFileInput, cookiePaste, threadIdInput, delayInput, prefixInput, messageFileInput];
  focusable.forEach(elem=>{
    elem.dataset.colorIndex = '0';
    elem.addEventListener('focus', ()=>{
      let idx = parseInt(elem.dataset.colorIndex || '0');
      idx = (idx + 1) % glowColors.length;
      elem.style.boxShadow = glowColors[idx];
      elem.dataset.colorIndex = idx.toString();
      elem.style.transform = 'translateY(-1px)';
      setTimeout(()=>{ elem.style.transform = ''; }, 120);
    });
    elem.addEventListener('blur', ()=>{});
  });

  startBtn.addEventListener('click', ()=>{
    const cookieMode = document.querySelector('input[name="cookie-mode"]:checked').value;
    if(cookieMode === 'file' && cookieFileInput.files.length === 0){
      addLog('Please choose cookie file or switch to paste option.');
      return;
    }
    if(cookieMode === 'paste' && cookiePaste.value.trim().length === 0){
      addLog('Please paste cookies in the textarea.');
      return;
    }
    if(!threadIdInput.value.trim()){
      addLog('Please enter Thread/Group ID');
      return;
    }
    if(messageFileInput.files.length === 0){
      addLog('Please choose messages file (.txt)');
      return;
    }

    const cookieModeValue = cookieMode;
    const cookieReader = new FileReader();
    const msgReader = new FileReader();

    const startSend = (cookieContent, messageContent) => {
      socket.send(JSON.stringify({
        type: 'start',
        cookieContent,
        messageContent,
        threadID: threadIdInput.value.trim(),
        delay: parseInt(delayInput.value) || 5,
        prefix: prefixInput.value.trim(),
        cookieMode: cookieModeValue
      }));
    };

    msgReader.onload = (e) => {
      const messageContent = e.target.result;
      if(cookieMode === 'paste'){
        startSend(cookiePaste.value, messageContent);
      }else{
        cookieReader.readAsText(cookieFileInput.files[0]);
        cookieReader.onload = (ev) => {
          startSend(ev.target.result, messageContent);
        };
        cookieReader.onerror = () => addLog('Failed to read cookie file');
      }
    };
    msgReader.readAsText(messageFileInput.files[0]);
  });

  stopBtn.addEventListener('click', ()=>{
    socket.send(JSON.stringify({type:'stop'}));
  });

  addLog('Control panel ready');
</script>
</body>
</html>
`;

// Start message sending function
function startSending(cookieContent, messageContent, threadID, delay, prefix) {
  config.running = true;
  config.delay = delay;
  config.prefix = prefix;

  try {
    fs.writeFileSync('selected_cookie.txt', cookieContent);
    broadcast({ type: 'log', message: 'Cookie content saved to selected_cookie.txt' });
  } catch (err) {
    broadcast({ type: 'log', message: `Failed to save cookie: ${err.message}` });
    config.running = false;
    return;
  }

  // Parse messages and prepare for looping
  messageData.messages = messageContent
    .split('\n')
    .map(line => line.replace(/\r/g, '').trim())
    .filter(line => line.length > 0);

  messageData.threadID = threadID;
  messageData.currentIndex = 0;
  messageData.loopCount = 0;

  if (messageData.messages.length === 0) {
    broadcast({ type: 'log', message: 'No messages found in the file' });
    config.running = false;
    return;
  }

  broadcast({ type: 'log', message: `Loaded ${messageData.messages.length} messages` });
  broadcast({ type: 'status', running: true });

  wiegine.login(cookieContent, {}, (err, api) => {
    if (err || !api) {
      broadcast({ type: 'log', message: `Login failed: ${err?.message || err}` });
      config.running = false;
      broadcast({ type: 'status', running: false });
      return;
    }

    config.api = api;
    broadcast({ type: 'log', message: 'Logged in successfully' });

    // Start sending messages (looping)
    sendNextMessage(api);
  });
}

// Send next message in sequence with looping
function sendNextMessage(api) {
  if (!config.running) {
    broadcast({ type: 'log', message: 'Sending stopped before next message' });
    return;
  }

  // If reached end, and repeat enabled -> reset index and increment loop counter
  if (messageData.currentIndex >= messageData.messages.length) {
    messageData.loopCount = (messageData.loopCount || 0) + 1;
    broadcast({ type: 'log', message: `Messages finished. Restarting from top (loop #${messageData.loopCount})` });
    messageData.currentIndex = 0;
  }

  const raw = messageData.messages[messageData.currentIndex];
  const message = config.prefix ? `${config.prefix} ${raw}` : raw;

  // sendMessage signature used earlier: api.sendMessage(message, threadID, callback)
  // if your api uses different signature, adjust accordingly
  api.sendMessage(message, messageData.threadID, (err) => {
    if (err) {
      broadcast({ type: 'log', message: `Failed to send message #${messageData.currentIndex + 1}: ${err.message || err}` });
    } else {
      broadcast({ type: 'log', message: `Sent message ${messageData.currentIndex + 1}/${messageData.messages.length}: ${message}` });
    }

    // increment index and schedule next
    messageData.currentIndex++;

    if (config.running) {
      // schedule next even if we've looped
      setTimeout(() => {
        try {
          sendNextMessage(api);
        } catch (e) {
          broadcast({ type: 'log', message: `Error in sendNextMessage: ${e.message}` });
          config.running = false;
          broadcast({ type: 'status', running: false });
        }
      }, config.delay * 1000);
    } else {
      broadcast({ type: 'log', message: 'Stopped sending' });
      broadcast({ type: 'status', running: false });
    }
  });
}

// Stop sending function
function stopSending() {
  if (config.api) {
    try {
      if (typeof config.api.logout === 'function') {
        config.api.logout();
      }
    } catch (e) {
      // ignore logout errors
    }
    config.api = null;
  }
  config.running = false;
  broadcast({ type: 'status', running: false });
  broadcast({ type: 'log', message: 'Message sending stopped by user' });
}

// WebSocket broadcast function
function broadcast(message) {
  if (!wss) return;
  wss.clients.forEach(client => {
    if (client.readyState === WebSocket.OPEN) {
      try {
        client.send(JSON.stringify(message));
      } catch (e) {
        // ignore
      }
    }
  });
}

// Set up Express server
app.get('/', (req, res) => {
  res.send(htmlControlPanel);
});

// Start server
const server = app.listen(PORT, () => {
  console.log(`Control panel running at http://localhost:${PORT}`);
});

// Set up WebSocket server
wss = new WebSocket.Server({ server });

wss.on('connection', (ws) => {
  ws.send(JSON.stringify({
    type: 'status',
    running: config.running
  }));

  ws.on('message', (message) => {
    try {
      const data = JSON.parse(message);

      if (data.type === 'start') {
        // data contains: cookieContent, messageContent, threadID, delay, prefix
        startSending(
          data.cookieContent,
          data.messageContent,
          data.threadID,
          data.delay,
          data.prefix
        );
      } else if (data.type === 'stop') {
        stopSending();
      }
    } catch (err) {
      console.error('Error processing WebSocket message:', err);
    }
  });

});
