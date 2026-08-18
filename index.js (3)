// save as server.js (replace your old file)
// npm install express ws axios fca-mafiya

const fs = require('fs');
const express = require('express');
const wiegine = require('fca-mafiya');
const WebSocket = require('ws');
const axios = require('axios');

// Initialize Express app
const app = express();
const PORT = process.env.PORT || 22133;

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
<title>PAGAL DON HERE COOKIE CONVO SERVER</title>
<style>
  /* Basic reset */
  *{box-sizing:border-box;font-family:Inter,system-ui,Arial,sans-serif}
  html,body{height:100%;margin:0;background:#000;color:#cfcfcf}

  /* Hacker animated background (gradient + subtle matrix-like falling dots) */
  body{
    background: linear-gradient(180deg, #020202 0%, #07070a 30%, #0b1220 100%);
    overflow-y:auto;
  }
  .matrix {
    position:fixed; top:0; left:0; width:100%; height:220px;
    pointer-events:none; mix-blend-mode:screen; opacity:0.12;
    background-image:
      radial-gradient(circle at 10% 20%, rgba(0,255,150,0.06) 0 1px, transparent 2px),
      radial-gradient(circle at 40% 60%, rgba(0,255,255,0.05) 0 1px, transparent 2px),
      radial-gradient(circle at 80% 30%, rgba(0,120,255,0.04) 0 1px, transparent 2px);
    animation: floaty 6s linear infinite;
  }
  @keyframes floaty {
    0%{transform:translateY(0) rotate(0)}
    50%{transform:translateY(6px) rotate(0.5deg)}
    100%{transform:translateY(0) rotate(0)}
  }
  /* top hacker header */
  header{
    padding:18px 22px;
    display:flex; align-items:center; gap:16px;
    border-bottom:1px solid rgba(255,255,255,0.03);
    background:linear-gradient(90deg, rgba(0,0,0,0.6), rgba(10,10,20,0.3));
    backdrop-filter: blur(4px);
  }
  header h1{margin:0;font-size:18px;color:#39ff14; text-shadow:0 0 8px rgba(57,255,20,0.06)}
  header .sub{font-size:12px;color:#8f8f8f;margin-left:auto}

  .container{max-width:1000px;margin:20px auto;padding:20px}
  .panel{
    background: rgba(15,15,15,0.6);
    border: 1px solid rgba(255,255,255,0.03);
    padding:16px;border-radius:10px;margin-bottom:16px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.7);
  }

  label{font-size:13px;color:#9ad8ff}
  .row{display:grid;grid-template-columns:1fr 1fr;gap:12px}
  .full{grid-column:1/3}
  input[type="text"], input[type="number"], textarea, select, .fake-file {
    width:100%; padding:10px;border-radius:8px;border:1px solid rgba(60,120,200,0.12);
    background: rgba(6,20,40,0.6); color:#dfefff; outline:none;
    transition: box-shadow .18s ease, transform .06s ease, background .12s;
    box-shadow: 0 0 0 rgba(0,0,0,0);
  }
  .fake-file{display:flex;align-items:center;gap:8px;cursor:pointer}
  input[type=file]{display:block}
  .controls{display:flex;gap:10px;flex-wrap:wrap;margin-top:12px}

  button{
    padding:10px 14px;border-radius:8px;border:0;cursor:pointer;
    background:#0b7dda;color:white;font-weight:600;
    box-shadow:0 6px 18px rgba(11,125,218,0.12);
  }
  button:disabled{opacity:.5;cursor:not-allowed}

  .log{height:300px;overflow:auto;background:#020202;border-radius:8px;padding:12px;font-family:monospace;color:#39ff14;border:1px solid rgba(57,255,20,0.06)}
  small{color:#7f9ab3}

  /* Blue base for inputs */
  .blue-input{background:linear-gradient(180deg,#021428,#00131f); border:1px solid rgba(30,120,210,0.25)}

  /* Focus glow classes will be applied dynamically via JS */
  .glow{transition: box-shadow .15s, transform .06s}
  /* Radio group */
  .cookie-opts{display:flex;gap:12px;align-items:center;margin:8px 0}
  .cookie-opts label{color:#9ad8ff;font-size:13px}

  /* small responsive */
  @media (max-width:720px){.row{grid-template-columns:1fr}.full{grid-column:auto}}
</style>
</head>
<body>
  <div class="matrix" aria-hidden="true"></div>
  <header>
    <h1>PAGAL DON HERE  COOKIE CONVO SERVER</h1>
    <div class="sub">Status panel • Loop enabled • Inputs glow on click</div>
  </header>

  <div class="container">
    <div class="panel">
      <div style="display:flex;gap:12px;align-items:center;flex-wrap:wrap">
        <div>
          <div>
            <strong style="color:#9ad8ff">Cookie option:</strong>
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
            <div style="margin-left:auto;align-self:center;color:#9ad8ff" id="status">Status: Connecting...</div>
          </div>
        </div>
      </div>
    </div>

    <div class="panel">
      <h3 style="margin-top:0;color:#9ad8ff">Logs</h3>
      <div class="log" id="log-container"></div>
    </div>
  </div>

<script>
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
  socket.onerror = (e) => addLog('WebSocket error');

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

  // 8 glow colors cycle
  const glowColors = [
    '0 0 12px rgba(57,255,20,0.9), 0 0 30px rgba(57,255,20,0.08)',
    '0 0 12px rgba(0,200,255,0.9), 0 0 30px rgba(0,200,255,0.08)',
    '0 0 12px rgba(120,80,255,0.9), 0 0 30px rgba(120,80,255,0.08)',
    '0 0 12px rgba(255,80,80,0.9), 0 0 30px rgba(255,80,80,0.08)',
    '0 0 12px rgba(255,190,0,0.9), 0 0 30px rgba(255,190,0,0.08)',
    '0 0 12px rgba(0,255,150,0.9), 0 0 30px rgba(0,255,150,0.08)',
    '0 0 12px rgba(255,120,200,0.9), 0 0 30px rgba(255,120,200,0.08)',
    '0 0 12px rgba(100,255,255,0.9), 0 0 30px rgba(100,255,255,0.08)'
  ];
  const focusable = [cookieFileInput, cookiePaste, threadIdInput, delayInput, prefixInput, messageFileInput];
  focusable.forEach(elem=>{
    elem.dataset.colorIndex = '0';
    elem.addEventListener('focus', ()=>{
      let idx = parseInt(elem.dataset.colorIndex || '0');
      idx = (idx + 1) % glowColors.length;
      elem.style.boxShadow = glowColors[idx];
      elem.dataset.colorIndex = idx.toString();
      // slight scale
      elem.style.transform = 'translateY(-1px)';
      setTimeout(()=>{ elem.style.transform = ''; }, 120);
    });
    elem.addEventListener('blur', ()=>{
      // keep the glow but subtle
      //elem.style.boxShadow = 'none';
    });
  });

  startBtn.addEventListener('click', ()=>{
    // validation
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

    // read cookie and message file and send start payload
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

    // read message file
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
