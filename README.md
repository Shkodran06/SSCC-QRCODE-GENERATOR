<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SSCC QR-CODE GENERATOR</title>

<style>
body {
  font-family: Arial, sans-serif;
  margin:0; padding:0;
  display:flex; flex-direction:column; align-items:center; justify-content:flex-start;
  min-height:100vh;
  background:#fff; color:#ff6600;
  transition: all 0.3s;
}
.night { background:#000; color:#00f; }
.wrapper { display:flex; flex-wrap:wrap; gap:20px; max-width:900px; width:90%; background:#f8f8f8; padding:20px; border-radius:12px; box-shadow:0 3px 15px rgba(0,0,0,0.4);}
.night .wrapper { background:#111; color:#00f; }
.controls { flex:1 1 280px; display:flex; flex-direction:column; gap:10px; }
.input-row { display:flex; gap:8px; align-items:center; }
.input-row input { flex:1; padding:10px; border-radius:6px; border:1px solid #ccc; font-size:1rem; }
.input-row button { padding:10px; font-size:1.2rem; border:none; border-radius:6px; cursor:pointer; background:#ff6600; color:#fff; }
.night .input-row button { background:#00f; color:#000; }
.controls button { padding:8px 12px; border:none; border-radius:6px; background:#0b5cff; color:#fff; cursor:pointer; font-size:0.95rem; }
.night .controls button { background:#0736d1; color:#fff; }
.history-title { margin-top:15px; font-size:0.95rem; font-weight:bold; }
.history { max-height:150px; overflow-y:auto; font-size:0.9rem; }
.history-item { padding:4px 0; border-bottom:1px solid #ccc; cursor:pointer; user-select:text; }
.history-item:hover { text-decoration:underline; }
.preview-card { flex:1 1 260px; background:#fff; color:#000; border-radius:10px; padding:10px; display:flex; flex-direction:column; align-items:center; }
.night .preview-card { background:#222; color:#00f; }
#qrContainer { width:50mm; height:30mm; background:#fff; border:1px solid #ccc; border-radius:6px; display:flex; justify-content:center; align-items:center; margin-bottom:5px; }
.qr-placeholder { font-size:0.85rem; color:#888; text-align:center; }
#last4 { font-size:1.1rem; font-weight:bold; margin-top:2px; }
footer { width:100%; text-align:center; padding:10px 0; position:fixed; bottom:0; left:0; font-size:0.95rem; color:#f0f0f0; background:#111; }
footer span { cursor:pointer; }
</style>

<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-auth-compat.js"></script>

</head>
<body>

<div class="wrapper">
  <!-- Sito principale -->
  <div class="controls">
    <label for="ssccInput">SSCC Numero</label>
    <div class="input-row">
      <input id="ssccInput" type="text" placeholder="Inserisci o scannerizza SSCC...">
    </div>
    <div style="display:flex; gap:8px;">
      <button class="qtyBtn" data-qty="5">5</button>
      <button class="qtyBtn" data-qty="10">10</button>
      <button class="qtyBtn" data-qty="15">15</button>
      <button class="qtyBtn" data-qty="20">20</button>
    </div>
    <div class="history-title">Ultimi SSCC</div>
    <div class="history" id="historyList"></div>
    <div style="margin-top:10px;"><span id="toggleTheme">🌓 Cambia Tema</span></div>
    <div style="margin-top:10px;"><span id="showDashboard">🔒 Dashboard</span></div>
  </div>

  <div class="preview-card">
    <div id="qrContainer">
      <div class="qr-placeholder">📱 QR verrà qui</div>
    </div>
    <div id="last4">----</div>
  </div>
</div>

<!-- Sezione dashboard (nascosta) -->
<div id="dashboardContainer" style="display:none; padding:20px;">
  <h2>Dashboard Riservata</h2>
  <div id="dashboardLogin">
    <input type="email" id="loginEmail" placeholder="Email">
    <input type="password" id="loginPassword" placeholder="Password">
    <button id="loginBtn">Accedi</button>
  </div>
  <div id="dashboardContent" style="display:none;">
    <button id="logoutBtn">Esci</button>
    <h3>Etichette stampate</h3>
    <div id="stats"></div>
    <div id="chartContainer" style="width:100%; max-width:600px;"></div>
  </div>
</div>

<footer>
  Made with ❤️ by Shko
</footer>

<script>
/* ------------------ Firebase ------------------ */
const firebaseConfig = {
  apiKey: "AIzaSyA5kXm_3HMWO0YY4h_FzGr9bOpHZ5J6oxg",
  authDomain: "server-06-0201.firebaseapp.com",
  projectId: "server-06-0201",
  storageBucket: "server-06-0201.firebasestorage.app",
  messagingSenderId: "436450955649",
  appId: "1:436450955649:web:44645c7d39942b44875880"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();
const auth = firebase.auth();

/* ------------------ Variabili ------------------ */
const ssccInput = document.getElementById('ssccInput');
const qrContainer = document.getElementById('qrContainer');
const last4El = document.getElementById('last4');
const historyList = document.getElementById('historyList');
let qrCode = null;
let ssccHistory = [];
const pcId = "PC_" + Math.floor(Math.random()*10000); // id PC casuale

/* ------------------ Tema ------------------ */
document.getElementById('toggleTheme').addEventListener('click', ()=>{
  document.body.classList.toggle('night');
});

/* ------------------ Funzioni QR ------------------ */
function generateQR(value){
  qrContainer.innerHTML='';
  last4El.textContent='----';
  if(!value){ qrContainer.innerHTML='<div class="qr-placeholder">📱 QR verrà qui</div>'; return; }
  qrCode=new QRCode(qrContainer,{text:value,width:180,height:108}); // 50x30mm approx
  last4El.textContent=value.slice(-4);
  addToHistory(value);
}

/* ------------------ Cronologia ------------------ */
function addToHistory(sscc){
  if(!sscc) return;
  const now = new Date();
  const time = now.getHours().toString().padStart(2,'0') + ':' + now.getMinutes().toString().padStart(2,'0');
  ssccHistory = ssccHistory.filter(item => item.value !== sscc);
  ssccHistory.unshift({value:sscc, time:time});
  if(ssccHistory.length>6) ssccHistory.pop();
  renderHistory();
}

function renderHistory(){
  historyList.innerHTML='';
  ssccHistory.forEach(item=>{
    const div=document.createElement('div');
    div.className='history-item';
    div.textContent=`${item.value} (${item.time})`;
    div.addEventListener('click', ()=>{
      ssccInput.value=item.value;
      generateQR(item.value);
    });
    historyList.appendChild(div);
  });
}

/* ------------------ Bottoni quantità ------------------ */
document.querySelectorAll('.qtyBtn').forEach(btn=>{
  btn.addEventListener('click', ()=>{
    const qty = parseInt(btn.dataset.qty);
    const val = ssccInput.value.trim();
    if(!val) return alert('Inserisci SSCC');
    generateQR(val);
    savePrintLog(val, qty);
    alert(`Stampate ${qty} etichette`);
  });
});

/* ------------------ Salvataggio Firestore ------------------ */
function savePrintLog(sscc, qty){
  db.collection("printLogs").add({
    pcId: pcId,
    sscc: sscc,
    quantity: qty,
    timestamp: firebase.firestore.FieldValue.serverTimestamp()
  }).then(()=>console.log("Stampata salvata"))
    .catch(err=>console.error(err));
}

/* ------------------ Scanner input automatico ------------------ */
ssccInput.addEventListener('input', e=>{
  const val = ssccInput.value.trim();
  if(val.length>0){
    generateQR(val);
  } else {
    qrContainer.innerHTML='<div class="qr-placeholder">📱 QR verrà qui</div>';
    last4El.textContent='----';
  }
});

/* ------------------ Dashboard ------------------ */
const dashboardContainer = document.getElementById('dashboardContainer');
const showDashboard = document.getElementById('showDashboard');
const dashboardLogin = document.getElementById('dashboardLogin');
const dashboardContent = document.getElementById('dashboardContent');
const loginBtn = document.getElementById('loginBtn');
const logoutBtn = document.getElementById('logoutBtn');
const statsDiv = document.getElementById('stats');

showDashboard.addEventListener('click', ()=>{
  dashboardContainer.style.display='block';
  document.querySelector('.wrapper').style.display='none';
});

loginBtn.addEventListener('click', ()=>{
  const email = document.getElementById('loginEmail').value;
  const pass = document.getElementById('loginPassword').value;
  auth.signInWithEmailAndPassword(email, pass)
    .then(()=>{
      dashboardLogin.style.display='none';
      dashboardContent.style.display='block';
      loadStats();
    })
    .catch(err=>alert("Errore login: "+err.message));
});

logoutBtn.addEventListener('click', ()=>{
  auth.signOut().then(()=>{
    dashboardLogin.style.display='block';
    dashboardContent.style.display='none';
  });
});

/* ------------------ Statistiche ------------------ */
function loadStats(){
  db.collection("printLogs").orderBy("timestamp","desc").limit(50)
    .onSnapshot(snapshot=>{
      statsDiv.innerHTML='';
      snapshot.forEach(doc=>{
        const data = doc.data();
        const div=document.createElement('div');
        div.textContent=`PC: ${data.pcId}, SSCC: ${data.sscc}, Quantità: ${data.quantity}`;
        statsDiv.appendChild(div);
      });
    });
}
</script>

</body>
</html>

