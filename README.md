<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Mas Tolong — Layanan Pelanggan</title>

<!-- Leaflet & Routing CSS -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<link rel="stylesheet" href="https://unpkg.com/leaflet-routing-machine@latest/dist/leaflet-routing-machine.css" />

<!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>

<style>
:root { 
  --blue-primary: #0264c8; 
  --blue-dark: #014389; 
  --bg: #0f172a; 
  --card-bg: #1e293b;
  --text: #f8fafc; 
  --text-dim: #94a3b8;
  --green: #10b981; 
  --red: #ef4444; 
  --yellow: #f59e0b; 
}
* { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; -webkit-tap-highlight-color: transparent; }
body { background: var(--bg); color: var(--text); padding-bottom: 40px; }

header { background: var(--blue-primary); color: white; padding: 12px 16px; display: flex; justify-content: space-between; align-items: center; box-shadow: 0 4px 12px rgba(0,0,0,0.3); position: relative; z-index: 100; }
.header-title { font-size: 15px; font-weight: bold; letter-spacing: 0.5px; }
.header-sub { font-size: 10px; opacity: 0.9; }

main { max-width: 480px; margin: auto; padding: 12px; position: relative; z-index: 10; }
.card { background: var(--card-bg); border-radius: 16px; padding: 16px; margin-bottom: 12px; border: 1px solid #334155; box-shadow: 0 4px 12px rgba(0,0,0,0.2); position: relative; z-index: 20; }
#map { height: 240px; border-radius: 12px; margin-bottom: 12px; z-index: 1; border: 1px solid #334155; }

.hero-box {
  background: linear-gradient(135deg, #0264c8 0%, #003e82 100%);
  color: white; border-radius: 16px; padding: 16px; text-align: center; margin-bottom: 12px;
  box-shadow: 0 4px 12px rgba(2, 100, 200, 0.3);
}
.hero-logo-title { font-size: 20px; font-weight: 900; letter-spacing: 0.5px; text-transform: uppercase; color: #ffffff; }
.hero-subtitle { font-size: 12px; opacity: 0.95; margin-top: 2px; }

.info-box { background: #0f172a; border: 1px solid #334155; border-radius: 12px; padding: 10px; margin-bottom: 12px; font-size: 11px; color: var(--text-dim); line-height: 1.4; }
.info-box b { color: #38bdf8; }

.menu-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; margin-bottom: 12px; text-align: center; }
.menu-item { background: #0f172a; padding: 10px 4px; border-radius: 12px; border: 1px solid #334155; cursor: pointer; font-size: 11px; font-weight: bold; color: var(--text); }
.menu-item.active { background: #0264c8; border-color: #38bdf8; color: white; }
.menu-icon { font-size: 20px; display: block; margin-bottom: 2px; }

.btn { width: 100%; padding: 12px; border: none; border-radius: 12px; font-weight: bold; font-size: 14px; cursor: pointer; text-decoration: none; display: inline-block; text-align: center; margin-bottom: 6px; position: relative; z-index: 50; }
.btn-primary { background: var(--blue-primary); color: white; }
.btn-green { background: var(--green); color: white; }
.btn-red { background: var(--red); color: white; }
.btn-outline { background: transparent; border: 1.5px solid var(--blue-primary); color: #38bdf8; }

.input-container { position: relative; margin-bottom: 8px; z-index: 40; }
.input-box { width: 100%; padding: 10px; border: 1px solid #334155; border-radius: 10px; font-size: 13px; margin-bottom: 8px; position: relative; z-index: 35; background: #0f172a; color: white; }

.search-results {
  position: absolute; top: 100%; left: 0; right: 0;
  background: #1e293b; border: 1px solid #334155; border-radius: 8px;
  max-height: 150px; overflow-y: auto; z-index: 999;
  box-shadow: 0 4px 12px rgba(0,0,0,0.4); display: none;
}
.search-item { padding: 8px 12px; font-size: 12px; cursor: pointer; border-bottom: 1px solid #334155; color: white; }
.search-item:hover { background: #0264c8; }

.order-card { border: 1.5px solid #0264c8; border-radius: 12px; padding: 12px; margin-bottom: 10px; background: #0f172a; }
.badge { display: inline-block; padding: 3px 8px; border-radius: 10px; font-size: 10px; font-weight: bold; }
.badge-yellow { background: #78350f; color: #fef3c7; }
.badge-green { background: #064e3b; color: #dcfce7; }

.floating-wa {
  position: fixed; bottom: 20px; right: 20px; background: #25d366; color: white;
  width: 50px; height: 50px; border-radius: 50%; display: flex; align-items: center; justify-content: center;
  font-size: 24px; box-shadow: 0 4px 12px rgba(0,0,0,0.4); z-index: 1500; text-decoration: none;
}
.leaflet-routing-container { display: none !important; }
</style>
</head>
<body>

<header>
  <div>
    <div class="header-title">🛵 MAS TOLONG</div>
    <div class="header-sub">Aplikasi Layanan Pelanggan</div>
  </div>
  <div id="headerLogout" style="display:none;">
    <button class="btn btn-red" style="padding:4px 10px; font-size:11px; margin:0; width:auto;" onclick="processLogout()">Log Out 🚪</button>
  </div>
</header>

<main>
  <!-- AUTH / LOGIN & DAFTAR PELANGGAN -->
  <div id="viewAuth" class="card">
    <div class="hero-box">
      <div class="hero-logo-title">🛵 MAS TOLONG</div>
      <div class="hero-subtitle">Portal Layanan Pelanggan Kertosono-Jombang</div>
    </div>
    
    <h3 id="authTitle" style="margin-bottom:10px; text-align:center;">Masuk Pelanggan</h3>

    <!-- BOX INFO FITUR PELANGGAN -->
    <div class="info-box">
      💡 <b>Keuntungan Akun Pelanggan Mas Tolong:</b><br>
      • Pesan Ojek Motor, Kurir Paket, & Jastip Makanan Cepat.<br>
      • Dilengkapi Fitur Saldo Non-Tunai <b>Dompetku</b>.<br>
      • Transparan: Tarif dihitung otomatis berdasarkan jarak GPS.
    </div>

    <input type="text" id="loginUsernameInput" class="input-box" placeholder="👤 Masukkan Username Anda" value="Citra">
    <input type="password" id="loginPasswordInput" class="input-box" placeholder="🔒 Password (SHA-256 Protected)" value="123456">
    
    <button class="btn btn-primary" id="btnAuthAction" onclick="processLoginSubmit()">Masuk Ke Sistem</button>
    <button class="btn btn-outline" id="btnRegisterToggle" style="margin-top:4px;" onclick="toggleRegisterMode()">Belum Punya Akun? Daftar Sekarang</button>
  </div>

  <!-- DASBOR PELANGGAN -->
  <div id="viewCustomer" style="display:none;">
    <div class="card" style="display:flex; align-items:center; justify-content:space-between; padding:12px;">
      <div>
        <h4 id="custDisplayName" style="font-size:14px;">Nama Pelanggan</h4>
        <p style="font-size:10px; color:var(--text-dim);">Saldo Dompetku: <b id="custWalletBalanceDisplay" style="color:var(--green);">Rp 0</b></p>
      </div>
    </div>

    <!-- FORM TOP UP STRUK AUTO-COMPRESS -->
    <div class="card" style="border: 1.5px solid var(--green); background: #064e3b;">
      <h4 style="color: #dcfce7; margin-bottom: 6px;">💳 Top Up Saldo Dompetku</h4>
      <p style="font-size: 11px; color: #a7f3d0; margin-bottom: 8px;">Transfer ke Rekening Admin / QRIS, lalu unggah bukti struk transfer di bawah ini:</p>
      <input type="number" id="topupAmountInput" class="input-box" placeholder="Nominal Top Up" value="50000">
      <label style="font-size:11px; font-weight:bold; color:#a7f3d0; display:block; margin-bottom:2px;">Unggah Struk Transfer:</label>
      <input type="file" id="topupStrukInput" class="input-box" accept="image/*" style="padding:6px;">
      <button class="btn btn-green" onclick="submitTopUpRequest()">📤 Kirim Bukti Struk & Verifikasi</button>
    </div>

    <div class="card">
      <h4 style="margin-bottom:10px;">Pilih Layanan Kami</h4>
      <div class="menu-grid">
        <div class="menu-item active" id="mRide" onclick="selectService('RIDE', '🛵 Beli/Antar Penumpang', 0)"><span class="menu-icon">🛵</span>Antar Orang</div>
        <div class="menu-item" id="mExpress" onclick="selectService('EXPRESS', '📦 Kirim Paket & Dokumen', 0)"><span class="menu-icon">📦</span>Kirim Paket</div>
        <div class="menu-item" id="mFood" onclick="selectService('FOOD', '🍔 Titip Beli Makanan (Jastip)', 3000)"><span class="menu-icon">🍔</span>Titip Makanan</div>
        <div class="menu-item" id="mShop" onclick="selectService('SHOP', '🛒 Belanja Sembako / Pasar', 3000)"><span class="menu-icon">🛒</span>Belanja Pasar</div>
      </div>

      <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 6px;">
        <span style="font-size: 11px; font-weight: bold; color: #38bdf8;">Peta Lokasi & Rute</span>
        <button class="btn btn-outline" style="width: auto; padding: 3px 8px; font-size: 10px; margin: 0;" onclick="useCurrentLocationGPS()">📍 GPS Saya</button>
      </div>
      <div id="map"></div>
      
      <div class="input-container">
        <label style="font-size:11px; font-weight:bold; color:var(--text-dim); display:block; margin-bottom:2px;">Lokasi Jemput / Beli</label>
        <input type="text" id="pickupInput" class="input-box" placeholder="Ketik lokasi jemput..." value="Alun-Alun Jombang" oninput="handleSearch(this.value, 'pickup')" autocomplete="off">
        <div id="pickupResults" class="search-results"></div>
      </div>

      <div class="input-container">
        <label style="font-size:11px; font-weight:bold; color:var(--text-dim); display:block; margin-bottom:2px;">Lokasi Tujuan</label>
        <input type="text" id="destInput" class="input-box" placeholder="Ketik lokasi tujuan..." value="Stasiun Jombang" oninput="handleSearch(this.value, 'dest')" autocomplete="off">
        <div id="destResults" class="search-results"></div>
      </div>

      <div class="input-container">
        <label style="font-size:11px; font-weight:bold; color:var(--text-dim); display:block; margin-bottom:2px;">Metode Pembayaran</label>
        <select id="paymentMethodSelect" class="input-box" style="font-weight:bold;">
          <option value="CASH">💵 Cash (Tunai ke Driver)</option>
          <option value="DOMPETKU">💳 Saldo Dompetku (Non-Tunai)</option>
        </select>
      </div>

      <div class="input-container">
        <label style="font-size:11px; font-weight:bold; color:var(--text-dim); display:block; margin-bottom:2px;">Estimasi Jarak & Waktu Tempuh</label>
        <input type="text" id="kmInputDisplay" class="input-box" readonly placeholder="Otomatis dihitung dari peta" style="background:#0f172a; font-weight:bold;">
      </div>

      <textarea id="notesInput" class="input-box" placeholder="Catatan Khusus untuk Driver..." style="height:60px; resize:none;"></textarea>
      
      <div style="display:flex; justify-content:space-between; align-items:center; margin:10px 0; background:#0f172a; padding:10px; border-radius:10px; border:1px solid #334155;">
        <div><span style="font-size:11px; color:var(--text-dim); display:block;">Rincian Jarak & Waktu</span><span style="font-size:12px; font-weight:bold;" id="kmDisplay">Menghitung...</span></div>
        <div style="text-align:right;"><span style="font-size:11px; color:var(--text-dim); display:block;">Tarif Penuh Pelanggan</span><span style="font-size:16px; font-weight:bold; color:#38bdf8;" id="fareDisplay">Rp 0</span></div>
      </div>

      <button class="btn btn-primary" style="padding:12px; font-size:14px;" onclick="payOrderAndMatchNearestDriver()">🚀 Cari Driver Terdekat Otomatis</button>
    </div>

    <div class="card">
      <h4 style="margin-bottom:8px;">Status Pesanan Active</h4>
      <div id="custStatusBox" style="font-size:13px; color:var(--text-dim);">Belum ada pesanan aktif.</div>
    </div>
  </div>
</main>

<a href="https://wa.me/6285141145797?text=Halo%20Admin" target="_blank" class="floating-wa">💬</a>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet-routing-machine@latest/dist/leaflet-routing-machine.js"></script>

<script>
const firebaseConfig = { databaseURL: "https://mas-tolong-default-rtdb.asia-southeast1.firebasedatabase.app" };
if (!firebase.apps.length) { firebase.initializeApp(firebaseConfig); }
const db = firebase.database();

const GAS_BACKEND_URL = "https://script.google.com/macros/s/AKfycbx.../exec"; 
let currentUserSession = null, isRegisterMode = false;
let startLat = -7.5463, startLng = 112.2338, endLat = -7.5550, endLng = 112.2280;
let calculatedKm = 0, calculatedMins = 0, calculatedFare = 0;
let currentService = 'RIDE', serviceLabel = '🛵 Beli/Antar Penumpang';
let jastipFee = 0, baseRate = 6000, kmRate = 2500, currentActiveOrderId = null;
let map = null, routingControl = null, markerStart = null, markerEnd = null;

async function hashPassword(str) {
  const buf = await crypto.subtle.digest("SHA-256", new TextEncoder().encode(str));
  return Array.from(new Uint8Array(buf)).map(b => b.toString(16).padStart(2, "0")).join("");
}

window.addEventListener('DOMContentLoaded', () => {
  const saved = localStorage.getItem('masTolongCustSession');
  if (saved) {
    currentUserSession = JSON.parse(saved);
    openDashboard();
  }
  listenRealtimeOrdersAndWallets();
});

function toggleRegisterMode() {
  isRegisterMode = !isRegisterMode;
  document.getElementById('authTitle').innerText = isRegisterMode ? 'Pendaftaran Pelanggan Baru' : 'Masuk Pelanggan';
  document.getElementById('btnAuthAction').innerText = isRegisterMode ? 'Buat Akun Baru' : 'Masuk Ke Sistem';
  document.getElementById('btnRegisterToggle').innerText = isRegisterMode ? 'Sudah Punya Akun? Masuk Disini' : 'Belum Punya Akun? Daftar Sekarang';
}

async function processLoginSubmit() {
  const username = document.getElementById('loginUsernameInput').value.trim();
  const rawPass = document.getElementById('loginPasswordInput').value.trim();
  if (!username || !rawPass) { alert("Lengkapi Username dan Password!"); return; }

  const passHash = await hashPassword(rawPass);

  if (isRegisterMode) {
    db.ref('users/' + username).once('value', (snap) => {
      if (snap.exists()) { alert("⚠️ Username ini sudah terdaftar!"); } 
      else {
        db.ref('users').once('value', (allSnap) => {
          let count = allSnap.val() ? Object.keys(allSnap.val()).length + 1 : 1;
          let generatedNoUrut = "#" + count.toString().padStart(3, '0');
          const newUser = { username, passwordHash: passHash, noUrutAdmin: generatedNoUrut, role: 'customer', registeredAt: new Date().toLocaleString("id-ID") };
          db.ref('users/' + username).set(newUser, () => {
            alert(`✅ Registrasi Pelanggan Berhasil!\nUsername: ${username}\nSilakan masuk dengan akun baru Anda.`);
            toggleRegisterMode();
          });
        });
      }
    });
    return;
  }

  db.ref('users/' + username).once('value', (snap) => {
    let uData = snap.val();
    if (!uData) { alert("❌ AKUN BELUM TERDAFTAR!"); return; }
    if (uData.passwordHash === passHash || uData.password === rawPass) {
      currentUserSession = { id: username, name: username, noUrut: uData.noUrutAdmin || "#001", role: 'customer' };
      localStorage.setItem('masTolongCustSession', JSON.stringify(currentUserSession));
      openDashboard();
    } else { alert("❌ PASSWORD SALAH!"); }
  });
}

function openDashboard() {
  document.getElementById('viewAuth').style.display = 'none';
  document.getElementById('viewCustomer').style.display = 'block';
  document.getElementById('headerLogout').style.display = 'block';
  document.getElementById('custDisplayName').innerText = currentUserSession.name;
  setTimeout(() => { initMap(); }, 300);
}

function processLogout() {
  localStorage.removeItem('masTolongCustSession');
  location.reload();
}

function submitTopUpRequest() {
  const amount = parseInt(document.getElementById('topupAmountInput').value);
  const fileInput = document.getElementById('topupStrukInput');
  if (!amount || amount <= 0) return alert("Masukkan nominal top up!");
  if (!fileInput.files || fileInput.files.length === 0) return alert("Pilih foto bukti struk!");

  const file = fileInput.files[0];
  const img = new Image();
  const reader = new FileReader();

  reader.onload = function(e) {
    img.src = e.target.result;
    img.onload = function() {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      const maxDim = 320;
      let width = img.width, height = img.height;
      if (width > height) { if (width > maxDim) { height *= maxDim / width; width = maxDim; } } 
      else { if (height > maxDim) { width *= maxDim / height; height = maxDim; } }
      canvas.width = width; canvas.height = height;
      ctx.drawImage(img, 0, 0, width, height);
      const compressedBase64 = canvas.toDataURL('image/jpeg', 0.3);

      const topupId = "TP-" + Date.now().toString().slice(-6);
      const topupData = {
        id: topupId, customerId: currentUserSession.id, customerName: currentUserSession.name,
        customerUrut: currentUserSession.noUrut || "#001", amount: amount, strukImg: compressedBase64,
        status: 'PENDING', createdAt: new Date().toLocaleString("id-ID")
      };

      db.ref('topupRequests/' + topupId).set(topupData, () => {
        let pesanWA = `*📩 LAPORAN TOP UP SALDO DOMPETKU*\n\n👤 *Pelanggan:* ${topupData.customerName}\n💰 *Nominal:* Rp ${topupData.amount.toLocaleString('id-ID')}\n🆔 *ID:* ${topupData.id}`;
        if (GAS_BACKEND_URL.includes("AKfycbx")) {
          fetch(GAS_BACKEND_URL, { method: 'POST', mode: 'no-cors', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ action: "sendWA", message: pesanWA }) });
        } else {
          window.open(`https://wa.me/6285141145797?text=${encodeURIComponent(pesanWA)}`, '_blank');
        }
        alert("Bukti Struk Terkirim! Menunggu Verifikasi Admin.");
        fileInput.value = "";
      });
    };
  };
  reader.readAsDataURL(file);
}

function listenRealtimeOrdersAndWallets() {
  if (currentUserSession) {
    db.ref('wallets/' + currentUserSession.id).on('value', (snap) => {
      document.getElementById('custWalletBalanceDisplay').innerText = "Rp " + (snap.val() || 0).toLocaleString('id-ID');
    });
  }

  db.ref('orders').on('value', (snap) => {
    const data = snap.val();
    if (currentUserSession && currentActiveOrderId && data && data[currentActiveOrderId]) {
      let ord = data[currentActiveOrderId];
      let statusBadge = ord.status === 'SEARCHING' ? 'badge-yellow' : 'badge-green';
      let statusDesc = ord.status === 'SEARCHING' ? `🔍 Menghubungkan ke driver: <b>${ord.matchedDriverName}</b>...` : `✅ Diterima oleh Driver: <b>${ord.driverName}</b>`;
      document.getElementById('custStatusBox').innerHTML = `
        <div class="order-card">
          <b>[${ord.service}] ID: ${ord.id}</b> <span class="badge ${statusBadge}">${ord.status}</span><br>
          <p style="margin: 6px 0;">${statusDesc}</p>
          📍 Jemput: ${ord.pickup}<br>🏁 Tujuan: ${ord.dest}<br>💳 Metode: ${ord.paymentMethod}<br>
          💰 Total Tarif: <b>Rp ${ord.totalOngkir.toLocaleString('id-ID')}</b>
        </div>
      `;
    }
  });
}

function initMap() {
  if (map) return;
  map = L.map('map').setView([startLat, startLng], 13);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

  markerStart = L.marker([startLat, startLng], { draggable: true }).addTo(map).bindPopup("Titik Jemput").openPopup();
  markerEnd = L.marker([endLat, endLng], { draggable: true }).addTo(map).bindPopup("Titik Tujuan").addTo(map);

  routingControl = L.Routing.control({
    waypoints: [L.latLng(startLat, startLng), L.latLng(endLat, endLng)],
    routeWhileDragging: true, createMarker: () => null
  }).addTo(map);

  routingControl.on('routesfound', function(e) {
    const summary = e.routes[0].summary;
    calculatedKm = (summary.totalDistance / 1000).toFixed(2);
    calculatedMins = Math.round(summary.totalTime / 60);
    document.getElementById('kmInputDisplay').value = `${calculatedKm} km (~${calculatedMins} menit)`;
    let ongkirDasar = calculatedKm <= 3 ? baseRate : baseRate + Math.round((calculatedKm - 3) * kmRate);
    calculatedFare = ongkirDasar + jastipFee;
    document.getElementById('kmDisplay').innerText = `${calculatedKm} km (~${calculatedMins} mnt)`;
    document.getElementById('fareDisplay').innerText = 'Rp ' + calculatedFare.toLocaleString('id-ID');
  });

  markerStart.on('dragend', (e) => { startLat = e.target.getLatLng().lat; startLng = e.target.getLatLng().lng; updateRouteWaypoints(); });
  markerEnd.on('dragend', (e) => { endLat = e.target.getLatLng().lat; endLng = e.target.getLatLng().lng; updateRouteWaypoints(); });
}

function updateRouteWaypoints() {
  if (routingControl) routingControl.setWaypoints([L.latLng(startLat, startLng), L.latLng(endLat, endLng)]);
}

function handleSearch(query, type) {
  const resultBox = document.getElementById(type === 'pickup' ? 'pickupResults' : 'destResults');
  if (query.length < 3) { resultBox.style.display = 'none'; return; }
  fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(query + ', Jombang, Jawa Timur')}&limit=5`)
    .then(res => res.json()).then(data => {
      if (!data || data.length === 0) { resultBox.style.display = 'none'; return; }
      resultBox.style.display = 'block';
      resultBox.innerHTML = data.map(item => `
        <div class="search-item" onclick="selectLocationResult(${item.lat}, ${item.lon}, '${item.display_name.replace(/'/g, "\\'")}', '${type}')">
          📍 <b>${item.display_name}</b>
        </div>
      `).join('');
    });
}

function selectLocationResult(lat, lon, displayName, type) {
  document.getElementById(type === 'pickup' ? 'pickupInput' : 'destInput').value = displayName;
  document.getElementById(type === 'pickup' ? 'pickupResults' : 'destResults').style.display = 'none';
  if (type === 'pickup') { startLat = parseFloat(lat); startLng = parseFloat(lon); if (markerStart) markerStart.setLatLng([startLat, startLng]); } 
  else { endLat = parseFloat(lat); endLng = parseFloat(lon); if (markerEnd) markerEnd.setLatLng([endLat, endLng]); }
  updateRouteWaypoints();
}

function useCurrentLocationGPS() {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition((pos) => {
      startLat = pos.coords.latitude; startLng = pos.coords.longitude;
      if (markerStart) markerStart.setLatLng([startLat, startLng]);
      if (map) map.setView([startLat, startLng], 15);
      document.getElementById('pickupInput').value = "Lokasi GPS Saya Saat Ini";
      updateRouteWaypoints();
    });
  }
}

function selectService(type, label, fee) {
  currentService = type; serviceLabel = label; jastipFee = fee;
  document.querySelectorAll('.menu-item').forEach(el => el.classList.remove('active'));
  if(type==='RIDE') document.getElementById('mRide').classList.add('active');
  if(type==='EXPRESS') document.getElementById('mExpress').classList.add('active');
  if(type==='FOOD') document.getElementById('mFood').classList.add('active');
  if(type==='SHOP') document.getElementById('mShop').classList.add('active');
  updateRouteWaypoints();
}

function calculateDistanceGPS(lat1, lon1, lat2, lon2) {
  let R = 6371;
  let dLat = (lat2 - lat1) * Math.PI / 180; let dLon = (lon2 - lon1) * Math.PI / 180;
  let a = Math.sin(dLat/2) * Math.sin(dLat/2) + Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) * Math.sin(dLon/2) * Math.sin(dLon/2);
  return R * (2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a)));
}

function payOrderAndMatchNearestDriver() {
  let pickupVal = document.getElementById('pickupInput').value;
  let destVal = document.getElementById('destInput').value;
  let methodVal = document.getElementById('paymentMethodSelect').value;
  if(!pickupVal || !destVal) return alert("Tentukan Titik Jemput dan Tujuan!");

  if (methodVal === 'DOMPETKU') {
    db.ref('wallets/' + currentUserSession.id).once('value', (snap) => {
      let currentBalance = snap.val() || 0;
      if (currentBalance < calculatedFare) return alert(`Saldo Dompetku Kurang!\nSaldo: Rp ${currentBalance.toLocaleString('id-ID')}\nTarif: Rp ${calculatedFare.toLocaleString('id-ID')}`);
      processCreateOrder(pickupVal, destVal, methodVal);
    });
  } else { processCreateOrder(pickupVal, destVal, methodVal); }
}

function processCreateOrder(pickupVal, destVal, methodVal) {
  db.ref('activeDrivers').once('value', (snapshot) => {
    const driversData = snapshot.val();
    let assignedDriverId = "ahmadadipMST0000";
    let assignedDriverName = "Ahmad Adip (ahmadadipMST0000)";

    if (driversData) {
      let driversList = Object.values(driversData);
      let shortestDistance = Infinity;
      driversList.forEach(drv => {
        let dist = calculateDistanceGPS(startLat, startLng, drv.lat, drv.lng);
        if (dist < shortestDistance) {
          shortestDistance = dist; assignedDriverId = drv.driverId; assignedDriverName = drv.driverName + " (" + drv.driverId + ")";
        }
      });
    }

    let komisiPlatform = Math.round(calculatedFare * 0.1); 
    let tarifBersihDriver = calculatedFare - komisiPlatform;
    const orderId = "MT-" + Date.now().toString().slice(-6);

    const newOrder = {
      id: orderId, service: serviceLabel, serviceType: currentService,
      customerId: currentUserSession.id, customerName: currentUserSession.name,
      customerUrut: currentUserSession.noUrut || "#001", pickup: pickupVal, dest: destVal,
      pickupLat: startLat, pickupLng: startLng, destLat: endLat, destLng: endLng,
      notes: document.getElementById('notesInput').value.trim() || "-",
      distance: `${calculatedKm} km (~${calculatedMins} mnt)`, totalOngkir: calculatedFare,
      driverNetEarnings: tarifBersihDriver, platformFee: komisiPlatform, paymentMethod: methodVal, 
      status: 'SEARCHING', matchedDriverId: assignedDriverId, matchedDriverName: assignedDriverName,
      createdAt: new Date().toLocaleString("id-ID")
    };

    db.ref('orders/' + orderId).set(newOrder, () => {
      currentActiveOrderId = orderId;
      if (methodVal === 'DOMPETKU') {
        db.ref('wallets/' + currentUserSession.id).once('value', (s) => {
          db.ref('wallets/' + currentUserSession.id).set((s.val() || 0) - calculatedFare);
        });
      }
      alert(`Pesanan Berhasil Dibuat!\nTarif: Rp ${calculatedFare.toLocaleString('id-ID')}`);
    });
  });
}
</script>
</body>
</html>
