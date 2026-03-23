<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>FileLock — Cifra i tuoi file</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@400;600;800&display=swap');

  :root {
    --bg: #0a0a0f;
    --surface: #111118;
    --surface2: #1a1a26;
    --border: #2a2a40;
    --accent: #c8ff00;
    --accent2: #ff4d6d;
    --text: #e8e8f0;
    --muted: #6666a0;
    --mono: 'Space Mono', monospace;
    --sans: 'Syne', sans-serif;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: var(--sans);
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    padding: 40px 20px 80px;
    position: relative;
    overflow-x: hidden;
  }

  /* Background grid */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      linear-gradient(var(--border) 1px, transparent 1px),
      linear-gradient(90deg, var(--border) 1px, transparent 1px);
    background-size: 40px 40px;
    opacity: 0.3;
    pointer-events: none;
    z-index: 0;
  }

  body::after {
    content: '';
    position: fixed;
    top: -200px; left: -200px;
    width: 600px; height: 600px;
    background: radial-gradient(circle, rgba(200,255,0,0.04) 0%, transparent 70%);
    pointer-events: none;
    z-index: 0;
    animation: drift 8s ease-in-out infinite alternate;
  }

  @keyframes drift {
    from { transform: translate(0, 0); }
    to   { transform: translate(80px, 60px); }
  }

  .container {
    position: relative;
    z-index: 1;
    width: 100%;
    max-width: 680px;
  }

  /* Header */
  header {
    text-align: center;
    margin-bottom: 48px;
  }

  .logo-wrap {
    display: inline-flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 12px;
  }

  .logo-icon {
    width: 44px; height: 44px;
    background: var(--accent);
    border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
    font-size: 22px;
    flex-shrink: 0;
  }

  h1 {
    font-family: var(--sans);
    font-size: 2.4rem;
    font-weight: 800;
    letter-spacing: -1px;
    color: var(--text);
  }

  h1 span { color: var(--accent); }

  .subtitle {
    font-family: var(--mono);
    font-size: 0.75rem;
    color: var(--muted);
    letter-spacing: 2px;
    text-transform: uppercase;
    margin-top: 6px;
  }

  /* Mode toggle */
  .mode-toggle {
    display: flex;
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 4px;
    margin: 0 auto 40px;
    width: fit-content;
    gap: 4px;
  }

  .mode-btn {
    font-family: var(--mono);
    font-size: 0.8rem;
    letter-spacing: 1px;
    text-transform: uppercase;
    padding: 10px 28px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.2s;
    background: transparent;
    color: var(--muted);
  }

  .mode-btn.active {
    background: var(--accent);
    color: #000;
    font-weight: 700;
  }

  .mode-btn:not(.active):hover {
    color: var(--text);
    background: var(--border);
  }

  /* Card */
  .card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 16px;
    padding: 36px;
    display: flex;
    flex-direction: column;
    gap: 28px;
  }

  /* Drop zone */
  .drop-zone {
    border: 2px dashed var(--border);
    border-radius: 12px;
    padding: 48px 24px;
    text-align: center;
    cursor: pointer;
    transition: all 0.25s;
    position: relative;
    overflow: hidden;
  }

  .drop-zone:hover, .drop-zone.drag-over {
    border-color: var(--accent);
    background: rgba(200,255,0,0.03);
  }

  .drop-zone.has-file {
    border-color: var(--accent);
    border-style: solid;
    background: rgba(200,255,0,0.04);
    padding: 24px;
  }

  .drop-zone input[type="file"] {
    position: absolute;
    inset: 0;
    opacity: 0;
    cursor: pointer;
    width: 100%;
    height: 100%;
  }

  .drop-icon {
    font-size: 2.5rem;
    margin-bottom: 12px;
    display: block;
    transition: transform 0.2s;
  }

  .drop-zone:hover .drop-icon { transform: scale(1.1); }

  .drop-label {
    font-family: var(--mono);
    font-size: 0.78rem;
    color: var(--muted);
    letter-spacing: 1px;
  }

  .drop-label strong {
    display: block;
    font-size: 1rem;
    color: var(--text);
    margin-bottom: 6px;
    font-family: var(--sans);
    font-weight: 600;
    letter-spacing: 0;
  }

  .file-info {
    display: none;
    align-items: center;
    gap: 14px;
    text-align: left;
  }

  .has-file .file-info { display: flex; }
  .has-file .drop-hint { display: none; }

  .file-icon-big {
    font-size: 2.2rem;
    flex-shrink: 0;
  }

  .file-details .file-name {
    font-family: var(--mono);
    font-size: 0.85rem;
    color: var(--text);
    word-break: break-all;
  }

  .file-details .file-size {
    font-family: var(--mono);
    font-size: 0.72rem;
    color: var(--muted);
    margin-top: 4px;
  }

  /* Field */
  .field label {
    display: block;
    font-family: var(--mono);
    font-size: 0.72rem;
    letter-spacing: 2px;
    text-transform: uppercase;
    color: var(--muted);
    margin-bottom: 10px;
  }

  .input-wrap {
    position: relative;
    display: flex;
    align-items: center;
  }

  .input-wrap input {
    width: 100%;
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: 10px;
    padding: 14px 50px 14px 16px;
    font-family: var(--mono);
    font-size: 0.95rem;
    color: var(--text);
    outline: none;
    transition: border-color 0.2s;
    letter-spacing: 1px;
  }

  .input-wrap input:focus {
    border-color: var(--accent);
  }

  .input-wrap input::placeholder { color: var(--muted); letter-spacing: 0; }

  .eye-btn {
    position: absolute;
    right: 14px;
    background: none;
    border: none;
    cursor: pointer;
    font-size: 1.1rem;
    color: var(--muted);
    transition: color 0.2s;
    padding: 4px;
  }

  .eye-btn:hover { color: var(--text); }

  /* Strength meter */
  .strength-bar {
    display: flex;
    gap: 4px;
    margin-top: 8px;
  }

  .strength-seg {
    height: 3px;
    flex: 1;
    border-radius: 2px;
    background: var(--border);
    transition: background 0.3s;
  }

  .strength-seg.active-weak   { background: var(--accent2); }
  .strength-seg.active-medium { background: #ffaa00; }
  .strength-seg.active-strong { background: var(--accent); }

  .strength-label {
    font-family: var(--mono);
    font-size: 0.68rem;
    margin-top: 5px;
    color: var(--muted);
    letter-spacing: 1px;
  }

  /* Action button */
  .action-btn {
    width: 100%;
    padding: 18px;
    border: none;
    border-radius: 12px;
    font-family: var(--sans);
    font-size: 1rem;
    font-weight: 800;
    letter-spacing: 1px;
    cursor: pointer;
    transition: all 0.2s;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    position: relative;
    overflow: hidden;
  }

  .action-btn.encrypt-btn {
    background: var(--accent);
    color: #000;
  }

  .action-btn.decrypt-btn {
    background: var(--accent2);
    color: #fff;
  }

  .action-btn:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 8px 30px rgba(200,255,0,0.25);
  }

  .decrypt-btn:hover:not(:disabled) {
    box-shadow: 0 8px 30px rgba(255,77,109,0.3);
  }

  .action-btn:active:not(:disabled) { transform: translateY(0); }

  .action-btn:disabled {
    opacity: 0.35;
    cursor: not-allowed;
  }

  /* Progress */
  .progress-wrap {
    display: none;
    flex-direction: column;
    gap: 8px;
  }

  .progress-wrap.visible { display: flex; }

  .progress-bar-bg {
    background: var(--surface2);
    border-radius: 6px;
    height: 6px;
    overflow: hidden;
  }

  .progress-bar-fill {
    height: 100%;
    border-radius: 6px;
    background: var(--accent);
    width: 0%;
    transition: width 0.15s linear;
  }

  .progress-text {
    font-family: var(--mono);
    font-size: 0.72rem;
    color: var(--muted);
    letter-spacing: 1px;
  }

  /* Status / result */
  .status-box {
    display: none;
    align-items: flex-start;
    gap: 12px;
    padding: 16px 20px;
    border-radius: 10px;
    font-family: var(--mono);
    font-size: 0.82rem;
    line-height: 1.5;
  }

  .status-box.success {
    display: flex;
    background: rgba(200,255,0,0.07);
    border: 1px solid rgba(200,255,0,0.25);
    color: var(--accent);
  }

  .status-box.error {
    display: flex;
    background: rgba(255,77,109,0.07);
    border: 1px solid rgba(255,77,109,0.25);
    color: var(--accent2);
  }

  .status-icon { font-size: 1.1rem; flex-shrink: 0; margin-top: 1px; }

  /* Info footer */
  .info-strip {
    margin-top: 32px;
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 12px;
  }

  .info-item {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 10px;
    padding: 16px;
    text-align: center;
  }

  .info-item .info-icon { font-size: 1.4rem; margin-bottom: 6px; display: block; }

  .info-item .info-title {
    font-family: var(--sans);
    font-size: 0.8rem;
    font-weight: 600;
    color: var(--text);
    margin-bottom: 4px;
  }

  .info-item .info-desc {
    font-family: var(--mono);
    font-size: 0.65rem;
    color: var(--muted);
    letter-spacing: 0.5px;
    line-height: 1.5;
  }

  /* Spinner animation */
  @keyframes spin { to { transform: rotate(360deg); } }
  .spinning { display: inline-block; animation: spin 0.8s linear infinite; }

  /* Fade in */
  @keyframes fadeUp {
    from { opacity:0; transform: translateY(20px); }
    to   { opacity:1; transform: translateY(0); }
  }
  .container { animation: fadeUp 0.5s ease; }
</style>
</head>
<body>
<div class="container">

  <header>
    <div class="logo-wrap">
      <div class="logo-icon">🔒</div>
      <h1>File<span>Lock</span></h1>
    </div>
    <p class="subtitle">AES-256-GCM · Portable · Zero dipendenze</p>
  </header>

  <div class="mode-toggle">
    <button class="mode-btn active" id="btn-encrypt" onclick="setMode('encrypt')">🔐 Cifra</button>
    <button class="mode-btn" id="btn-decrypt" onclick="setMode('decrypt')">🔓 Decifra</button>
  </div>

  <div class="card">
    <!-- Drop zone -->
    <div class="drop-zone" id="dropZone">
      <input type="file" id="fileInput" onchange="onFileSelected(this)">
      <div class="drop-hint">
        <span class="drop-icon">📂</span>
        <div class="drop-label">
          <strong>Trascina qui il file</strong>
          oppure clicca per sfogliare
        </div>
      </div>
      <div class="file-info">
        <span class="file-icon-big" id="fileIconBig">📄</span>
        <div class="file-details">
          <div class="file-name" id="fileName">—</div>
          <div class="file-size" id="fileSize">—</div>
        </div>
      </div>
    </div>

    <!-- Key field -->
    <div class="field">
      <label>Chiave di cifratura</label>
      <div class="input-wrap">
        <input type="password" id="keyInput" placeholder="Inserisci la tua chiave segreta…" oninput="updateStrength()" autocomplete="off">
        <button class="eye-btn" onclick="toggleEye()" title="Mostra/Nascondi">👁</button>
      </div>
      <div class="strength-bar">
        <div class="strength-seg" id="s1"></div>
        <div class="strength-seg" id="s2"></div>
        <div class="strength-seg" id="s3"></div>
        <div class="strength-seg" id="s4"></div>
      </div>
      <div class="strength-label" id="strengthLabel">— inserisci una chiave</div>
    </div>

    <!-- Progress -->
    <div class="progress-wrap" id="progressWrap">
      <div class="progress-bar-bg">
        <div class="progress-bar-fill" id="progressFill"></div>
      </div>
      <div class="progress-text" id="progressText">Elaborazione…</div>
    </div>

    <!-- Status -->
    <div class="status-box" id="statusBox">
      <span class="status-icon" id="statusIcon"></span>
      <span id="statusMsg"></span>
    </div>

    <!-- Action button -->
    <button class="action-btn encrypt-btn" id="actionBtn" onclick="runAction()">
      <span id="btnIcon">🔐</span>
      <span id="btnLabel">CIFRA FILE</span>
    </button>
  </div>

  <div class="info-strip">
    <div class="info-item">
      <span class="info-icon">🛡️</span>
      <div class="info-title">AES-256-GCM</div>
      <div class="info-desc">Cifratura autenticata<br>standard militare</div>
    </div>
    <div class="info-item">
      <span class="info-icon">🌐</span>
      <div class="info-title">100% Locale</div>
      <div class="info-desc">Nessun server.<br>Gira nel tuo browser.</div>
    </div>
    <div class="info-item">
      <span class="info-icon">📦</span>
      <div class="info-title">Portabile</div>
      <div class="info-desc">Un solo file .html.<br>Nessuna installazione.</div>
    </div>
  </div>

</div>

<script>
// ──────────────────────────────────────────
//  STATE
// ──────────────────────────────────────────
let currentMode = 'encrypt';
let selectedFile = null;

// ──────────────────────────────────────────
//  MODE
// ──────────────────────────────────────────
function setMode(mode) {
  currentMode = mode;
  resetStatus();

  const btn = document.getElementById('actionBtn');
  const btnIcon = document.getElementById('btnIcon');
  const btnLabel = document.getElementById('btnLabel');
  const encBtn = document.getElementById('btn-encrypt');
  const decBtn = document.getElementById('btn-decrypt');

  if (mode === 'encrypt') {
    encBtn.classList.add('active');
    decBtn.classList.remove('active');
    btn.className = 'action-btn encrypt-btn';
    btnIcon.textContent = '🔐';
    btnLabel.textContent = 'CIFRA FILE';
  } else {
    decBtn.classList.add('active');
    encBtn.classList.remove('active');
    btn.className = 'action-btn decrypt-btn';
    btnIcon.textContent = '🔓';
    btnLabel.textContent = 'DECIFRA FILE';
  }
}

// ──────────────────────────────────────────
//  FILE SELECTION
// ──────────────────────────────────────────
function onFileSelected(input) {
  if (input.files && input.files[0]) {
    setFile(input.files[0]);
  }
}

function setFile(file) {
  selectedFile = file;
  resetStatus();

  const zone = document.getElementById('dropZone');
  zone.classList.add('has-file');

  const ext = file.name.split('.').pop().toLowerCase();
  const iconMap = {
    pdf:'📄', jpg:'🖼️', jpeg:'🖼️', png:'🖼️', gif:'🖼️', webp:'🖼️',
    mp4:'🎬', mov:'🎬', avi:'🎬', mkv:'🎬',
    mp3:'🎵', wav:'🎵', flac:'🎵', aac:'🎵',
    zip:'🗜️', rar:'🗜️', '7z':'🗜️',
    ai:'✏️', psd:'✏️', svg:'✏️', eps:'✏️',
    doc:'📝', docx:'📝', txt:'📝', xls:'📊', xlsx:'📊',
    html:'💻', js:'💻', ts:'💻', py:'💻', json:'💻',
    lock:'🔒'
  };

  document.getElementById('fileIconBig').textContent = iconMap[ext] || '📦';
  document.getElementById('fileName').textContent = file.name;
  document.getElementById('fileSize').textContent = formatSize(file.size);
}

// ──────────────────────────────────────────
//  DRAG & DROP
// ──────────────────────────────────────────
const zone = document.getElementById('dropZone');

zone.addEventListener('dragover', e => { e.preventDefault(); zone.classList.add('drag-over'); });
zone.addEventListener('dragleave', () => zone.classList.remove('drag-over'));
zone.addEventListener('drop', e => {
  e.preventDefault();
  zone.classList.remove('drag-over');
  const file = e.dataTransfer.files[0];
  if (file) {
    document.getElementById('fileInput').value = '';
    setFile(file);
  }
});

// ──────────────────────────────────────────
//  KEY STRENGTH
// ──────────────────────────────────────────
function updateStrength() {
  const val = document.getElementById('keyInput').value;
  const segs = [document.getElementById('s1'), document.getElementById('s2'),
                document.getElementById('s3'), document.getElementById('s4')];
  const lbl = document.getElementById('strengthLabel');

  segs.forEach(s => { s.className = 'strength-seg'; });

  if (!val) { lbl.textContent = '— inserisci una chiave'; return; }

  let score = 0;
  if (val.length >= 8)  score++;
  if (val.length >= 16) score++;
  if (/[A-Z]/.test(val) && /[0-9]/.test(val)) score++;
  if (/[^A-Za-z0-9]/.test(val)) score++;

  const cls = score <= 1 ? 'active-weak' : score <= 2 ? 'active-medium' : 'active-strong';
  const labels = ['', 'Debole', 'Debole', 'Media', 'Forte'];
  const colors = ['', '#ff4d6d', '#ffaa00', '#ffaa00', '#c8ff00'];

  for (let i = 0; i < score; i++) segs[i].classList.add(cls);
  lbl.textContent = labels[score] || 'Debole';
  lbl.style.color = colors[score] || '#6666a0';
}

function toggleEye() {
  const inp = document.getElementById('keyInput');
  inp.type = inp.type === 'password' ? 'text' : 'password';
}

// ──────────────────────────────────────────
//  CRYPTO — Web Crypto API (AES-256-GCM)
// ──────────────────────────────────────────

async function deriveKey(password, salt) {
  const enc = new TextEncoder();
  const keyMaterial = await crypto.subtle.importKey(
    'raw', enc.encode(password), 'PBKDF2', false, ['deriveKey']
  );
  return crypto.subtle.deriveKey(
    { name: 'PBKDF2', salt, iterations: 250000, hash: 'SHA-256' },
    keyMaterial,
    { name: 'AES-GCM', length: 256 },
    false,
    ['encrypt', 'decrypt']
  );
}

async function encryptBuffer(buffer, password) {
  const salt = crypto.getRandomValues(new Uint8Array(16));
  const iv   = crypto.getRandomValues(new Uint8Array(12));
  const key  = await deriveKey(password, salt);

  const encrypted = await crypto.subtle.encrypt({ name: 'AES-GCM', iv }, key, buffer);

  // Layout: magic(8) + salt(16) + iv(12) + ciphertext
  const magic = new TextEncoder().encode('FLOCK_01'); // 8 bytes version marker
  const out = new Uint8Array(8 + 16 + 12 + encrypted.byteLength);
  out.set(magic, 0);
  out.set(salt, 8);
  out.set(iv, 24);
  out.set(new Uint8Array(encrypted), 36);
  return out.buffer;
}

async function decryptBuffer(buffer, password) {
  const data = new Uint8Array(buffer);

  // Check magic
  const magic = new TextDecoder().decode(data.slice(0, 8));
  if (magic !== 'FLOCK_01') throw new Error('File non valido o non cifrato con FileLock.');

  const salt      = data.slice(8, 24);
  const iv        = data.slice(24, 36);
  const encrypted = data.slice(36);

  const key = await deriveKey(password, salt);

  try {
    return await crypto.subtle.decrypt({ name: 'AES-GCM', iv }, key, encrypted);
  } catch {
    throw new Error('Chiave errata o file corrotto.');
  }
}

// ──────────────────────────────────────────
//  MAIN ACTION
// ──────────────────────────────────────────
async function runAction() {
  resetStatus();

  if (!selectedFile) { showStatus('error', '⚠️ Nessun file selezionato.'); return; }

  const key = document.getElementById('keyInput').value.trim();
  if (!key) { showStatus('error', '⚠️ Inserisci una chiave di cifratura.'); return; }

  const btn = document.getElementById('actionBtn');
  const btnLabel = document.getElementById('btnLabel');
  const btnIcon = document.getElementById('btnIcon');

  btn.disabled = true;
  btnIcon.className = 'spinning';
  btnIcon.textContent = '⏳';
  btnLabel.textContent = 'Elaborazione…';

  showProgress(0, 'Lettura file…');

  try {
    const buffer = await readFileAsBuffer(selectedFile);
    showProgress(35, currentMode === 'encrypt' ? 'Cifratura in corso…' : 'Decifratura in corso…');

    let resultBuffer, outName;

    if (currentMode === 'encrypt') {
      resultBuffer = await encryptBuffer(buffer, key);
      outName = selectedFile.name + '.lock';
    } else {
      resultBuffer = await decryptBuffer(buffer, key);
      // Remove .lock extension if present
      outName = selectedFile.name.endsWith('.lock')
        ? selectedFile.name.slice(0, -5)
        : 'decrypted_' + selectedFile.name;
    }

    showProgress(85, 'Preparazione download…');
    await sleep(200);
    showProgress(100, 'Completato!');

    downloadBuffer(resultBuffer, outName);

    await sleep(300);
    hideProgress();

    if (currentMode === 'encrypt') {
      showStatus('success',
        `✅ File cifrato con successo!\n→ Scaricato come: ${outName}\n\nCondividi il file .lock al destinatario e comunicagli la chiave separatamente.`);
    } else {
      showStatus('success',
        `✅ File decifrato con successo!\n→ Scaricato come: ${outName}`);
    }

  } catch (err) {
    hideProgress();
    showStatus('error', `❌ ${err.message}`);
  }

  btn.disabled = false;
  btnIcon.className = '';
  btnIcon.textContent = currentMode === 'encrypt' ? '🔐' : '🔓';
  btnLabel.textContent = currentMode === 'encrypt' ? 'CIFRA FILE' : 'DECIFRA FILE';
}

// ──────────────────────────────────────────
//  UTILS
// ──────────────────────────────────────────
function readFileAsBuffer(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = e => resolve(e.target.result);
    reader.onerror = () => reject(new Error('Errore lettura file.'));
    reader.readAsArrayBuffer(file);
  });
}

function downloadBuffer(buffer, filename) {
  const blob = new Blob([buffer], { type: 'application/octet-stream' });
  const url  = URL.createObjectURL(blob);
  const a    = document.createElement('a');
  a.href = url; a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}

function formatSize(bytes) {
  if (bytes < 1024) return bytes + ' B';
  if (bytes < 1048576) return (bytes / 1024).toFixed(1) + ' KB';
  if (bytes < 1073741824) return (bytes / 1048576).toFixed(1) + ' MB';
  return (bytes / 1073741824).toFixed(2) + ' GB';
}

function sleep(ms) { return new Promise(r => setTimeout(r, ms)); }

function showProgress(pct, text) {
  const wrap = document.getElementById('progressWrap');
  wrap.classList.add('visible');
  document.getElementById('progressFill').style.width = pct + '%';
  document.getElementById('progressText').textContent = text;
}

function hideProgress() {
  document.getElementById('progressWrap').classList.remove('visible');
}

function showStatus(type, msg) {
  const box = document.getElementById('statusBox');
  box.className = 'status-box ' + type;
  document.getElementById('statusMsg').textContent = msg;
}

function resetStatus() {
  const box = document.getElementById('statusBox');
  box.className = 'status-box';
}
</script>
</body>
</html>
