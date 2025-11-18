<!DOCTYPE html>
<html lang="ka">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>კლინიკა • საწოლების მართვა</title>

  <!-- Firebase SDK -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-app.js";
    import { getFirestore, collection, addDoc, deleteDoc, doc, updateDoc, onSnapshot, serverTimestamp, query,  orderBy, getDocs } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-firestore.js";

    const firebaseConfig = {
      apiKey: "AIzaSyCDze1tz15HdKZVSPOPW_-7t-9ag4AiZYs",
      authDomain: "clinic-inpatient.firebaseapp.com",
      projectId: "clinic-inpatient",
      storageBucket: "clinic-inpatient.firebasestorage.app",
      messagingSenderId: "586729386322",
      appId: "1:586729386322:web:17a92324784c2c988a4a8b"
    };

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);

    // გლობალური ცვლადები
    window.db = db;
    window.currentCollection = null;
    window.patients = [];
    window.currentSort = { column: 'timestamp', dir: 'desc' };
    window.editingDocId = null;

    window.showToast = function(msg, error = false) {
      const toast = document.getElementById('toast');
      toast.textContent = msg;
      toast.className = 'toast active' + (error ? ' error' : '');
      setTimeout(() => toast.classList.remove('active'), 4000);
    };
  </script>

  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      min-height: 100vh;
      padding: 30px;
    }

    /* შესვლის ეკრანი */
    .login-screen {
      display: flex;
      gap: 80px;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      flex-wrap: wrap;
    }

    .room-btn {
      width: 420px;
      padding: 60px 40px;
      background: white;
      border-radius: 28px;
      text-align: center;
      box-shadow: 0 25px 70px rgba(0,0,0,0.35);
      cursor: pointer;
      transition: all 0.4s ease;
      border: 4px solid transparent;
    }

    .room-btn:hover {
      transform: translateY(-20px) scale(1.06);
      box-shadow: 0 40px 100px rgba(0,0,0,0.5);
      border-color: #667eea;
    }

    .room-btn h2 {
      font-size: 2.6em;
      color: #333;
      margin-bottom: 15px;
    }

    .room-btn p {
      font-size: 1.35em;
      color: #666;
      margin-bottom: 30px;
    }

    .enter-arrow {
      font-size: 4em;
      color: #667eea;
      transition: 0.3s;
    }

    .room-btn:hover .enter-arrow {
      color: #5568d3;
      transform: scale(1.3);
    }

    /* აპლიკაცია */
    #app { display: none; min-height: 100vh; }

    .back-btn {
      position: fixed;
      top: 25px; left: 25px;
      background: rgba(255,255,255,0.3);
      backdrop-filter: blur(12px);
      color: white;
      border: none;
      padding: 14px 32px;
      border-radius: 50px;
      font-size: 1.1em;
      font-weight: bold;
      cursor: pointer;
      z-index: 999;
      transition: 0.3s;
    }
    .back-btn:hover { background: rgba(255,255,255,0.5); transform: scale(1.05); }

    .container { max-width: 1450px; margin: 0 auto; padding: 20px; }
    .header { background: white; padding: 40px; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,0.15); margin-bottom: 30px; text-align: center; }
    .header h1 { margin: 0 0 10px; color: #667eea; font-size: 3em; }
    .header p { color: #888; font-size: 1.2em; }

    .tabs { display: flex; gap: 15px; margin-bottom: 30px; flex-wrap: wrap; justify-content: center; }
    .tab-btn { padding: 16px 32px; background: white; border: none; border-radius: 12px; cursor: pointer; font-size: 18px; font-weight: 600; color: #667eea; transition: all 0.3s; box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
    .tab-btn:hover { transform: translateY(-4px); box-shadow: 0 10px 25px rgba(0,0,0,0.2); }
    .tab-btn.active { background: #667eea; color: white; }

    .card { background: white; padding: 40px; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,0.12); margin-bottom: 30px; }
    .form-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 25px; margin-bottom: 30px; }
    .form-group label { display: block; margin-bottom: 10px; font-weight: 600; color: #333; }
    .form-group input, .form-group select, .form-group textarea {
      width: 100%; padding: 16px; border: 2px solid #e0e0e0; border-radius: 12px; font-size: 16px; transition: border 0.3s;
    }
    .form-group input:focus, .form-group select:focus, .form-group textarea:focus {
      outline: none; border-color: #667eea; box-shadow: 0 0 0 4px rgba(102,126,234,0.1);
    }

    .btn { padding: 16px 32px; border: none; border-radius: 12px; cursor: pointer; font-size: 17px; font-weight: 600; transition: all 0.3s; }
    .btn-primary { background: #667eea; color: white; }
    .btn-primary:hover { background: #5568d3; transform: translateY(-3px); }
    .btn-danger { background: #e74c3c; color: white; }
    .btn-success { background: #27ae60; color: white; }
    .btn-delete { background: #c0392b; color: white; font-size: 14px; padding: 8px 16px; }

    .clear-all-btn {
      background: #e74c3c; color: white; padding: 18px 40px; border: none; border-radius: 12px;
      font-size: 18px; font-weight: bold; cursor: pointer; margin: 30px auto; display: block;
      box-shadow: 0 8px 25px rgba(231,76,60,0.4);
    }
    .clear-all-btn:hover { background: #c0392b; transform: translateY(-4px); }

    table { width: 100%; border-collapse: collapse; background: white; border-radius: 16px; overflow: hidden; box-shadow: 0 8px 25px rgba(0,0,0,0.1); margin-top: 20px; }
    th, td { padding: 18px; text-align: left; border-bottom: 1px solid #eee; }
    th { background: #f8f9fa; font-weight: 600; color: #333; cursor: pointer; user-select: none; }
    th:hover { background: #eef0ff; }
    tr:hover { background: #f8f9ff; }
    .action-buttons button { margin: 4px; }

    .stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 25px; margin-top: 20px; }
    .stat-card { background: linear-gradient(135deg, #667eea, #764ba2); color: white; padding: 35px; border-radius: 20px; text-align: center; box-shadow: 0 10px 30px rgba(0,0,0,0.3); }
    .stat-card h3 { font-size: 1.4em; margin-bottom: 10px; }
    .stat-card .number { font-size: 4em; font-weight: bold; }

    .empty-state { text-align: center; padding: 80px 20px; color: #999; font-size: 1.4em; }

    .toast {
      position: fixed; top: 30px; right: 30px; background: #27ae60; color: white;
      padding: 20px 35px; border-radius: 14px; z-index: 2000; display: none;
      box-shadow: 0 15px 40px rgba(0,0,0,0.3); font-size: 1.1em; font-weight: bold;
      animation: slideIn 0.5s ease;
    }
    .toast.active { display: block; }
    .toast.error { background: #e74c3c; }
    @keyframes slideIn { from { transform: translateX(100%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }

    .modal {
      display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.7); z-index: 1000;
      justify-content: center; align-items: center;
    }
    .modal.active { display: flex; }
    .modal-content {
      background: white; padding: 40px; border-radius: 20px; width: 90%; max-width: 700px; max-height: 90vh; overflow-y: auto;
    }

    @media (max-width: 900px) {
      .login-screen { gap: 40px; }
      .room-btn { width: 90%; max-width: 400px; }
    }
  </style>
</head>
<body>

<!-- შესვლის ეკრანი -->
<div class="login-screen">
  <div class="room-btn" onclick="openRoom('observation')">
    <h2>ობსერვაციის დარბაზი</h2>
    <p>აქტიური პაციენტები • მონიტორინგი</p>
    <div class="enter-arrow">→</div>
  </div>
  <div class="room-btn" onclick="openRoom('shock')">
    <h2>შოკის დარბაზი</h2>
    <p>კრიტიკული მდგომარეობა • 24/7</p>
    <div class="enter-arrow">→</div>
  </div>
</div>

<!-- მთავარი აპლიკაცია -->
<div id="app">
  <button class="back-btn" onclick="location.reload()">დარბაზის შეცვლა</button>

  <div class="container">
    <div class="header">
      <h1 id="room-title">დარბაზი</h1>
      <p>სტაციონარი / Inpatient</p>
    </div>

    <div class="tabs">
      <button class="tab-btn active" onclick="switchTab('active')">აქტიური პაციენტები</button>
      <button class="tab-btn" onclick="switchTab('archive')">არქივი</button>
      <button class="tab-btn" onclick="switchTab('statistics')">სტატისტიკა</button>
    </div>

    <!-- აქტიური პაციენტები -->
    <div id="active-tab" class="tab-content active">
      <div class="card">
        <h2>ახალი პაციენტის დამატება</h2>
        <form id="patient-form">
          <div class="form-grid">
            <div class="form-group">
              <label>საწოლი</label>
              <select id="bed" required>
                <option value="">აირჩიეთ</option>
                <option value="1">1</option><option value="2">2</option><option value="3">3</option><option value="4">4</option>
                <option value="5">5</option><option value="6">6</option><option value="7">7</option><option value="8">8</option>
                <option value="9">9</option><option value="10">10</option>
                <option value="ლოჯი">ლოჯი</option><option value="მცირე">მცირე</option>
              </select>
            </div>
            <div class="form-group"><label>პაციენტი</label><input type="text" id="patient-name" required></div>
            <div class="form-group"><label>ისტორია</label><input type="text" id="history-number"></div>
            <div class="form-group"><label>ICD-10</label><input type="text" id="icd10"></div>
            <div class="form-group"><label>ექიმი</label><input type="text" id="doctor"></div>
          </div>
          <div class="form-group"><label>კომენტარი</label><textarea id="comment" rows="3"></textarea></div>
          <button type="submit" class="btn btn-primary">დამატება</button>
        </form>

        <button class="clear-all-btn" onclick="clearAllData()">
          მთლიანი გასუფთავება (ყველა პაციენტი)
        </button>
      </div>

      <div class="card">
        <h2>აქტიური პაციენტები</h2>
        <input type="text" id="search" placeholder="ძებნა..." style="padding:16px; width:100%; max-width:600px; border-radius:12px; border:2px solid #ddd; font-size:16px; margin-bottom:20px;">
        <table>
          <thead>
            <tr>
              <th onclick="sortTable('bed')">საწოლი</th>
              <th onclick="sortTable('patient_name')">პაციენტი</th>
              <th onclick="sortTable('history_number')">ისტორია</th>
              <th onclick="sortTable('icd10_code')">ICD-10</th>
              <th onclick="sortTable('doctor')">ექიმი</th>
              <th>კომენტარი</th>
              <th onclick="sortTable('timestamp')">ჩარიცხვა</th>
              <th>მოქმედება</th>
            </tr>
          </thead>
          <tbody id="active-tbody"></tbody>
        </table>
      </div>
    </div>

    <!-- არქივი -->
    <div id="archive-tab" class="tab-content">
      <div class="card">
        <h2>არქივი</h2>
        <table>
          <thead>
            <tr>
              <th>საწოლი</th><th>პაციენტი</th><th>ისტორია</th><th>ICD-10</th><th>ექიმი</th>
              <th>კომენტარი</th><th>ჩარიცხვა</th><th>არქივში</th><th>მოქმედება</th>
            </tr>
          </thead>
          <tbody id="archive-tbody"></tbody>
        </table>
      </div>
    </div>

    <!-- სტატისტიკა -->
    <div id="statistics-tab" class="tab-content">
      <div class="card">
        <h2>სტატისტიკა</h2>
        <div class="stats-grid">
          <div class="stat-card"><h3>აქტიური</h3><div class="number" id="stat-active">0</div></div>
          <div class="stat-card"><h3>დღეს დამატებული</h3><div class="number" id="stat-today-added">0</div></div>
          <div class="stat-card"><h3>დღეს არქივში</h3><div class="number" id="stat-today-deleted">0</div></div>
          <div class="stat-card"><h3>სულ არქივში</h3><div class="number" id="stat-archived">0</div></div>
        </div>
      </div>
    </div>
  </div>

  <!-- Toast -->
  <div id="toast" class="toast"></div>
</div>

<!-- რედაქტირების მოდალი -->
<div id="edit-modal" class="modal">
  <div class="modal-content">
    <h2 style="margin-bottom:25px;">პაციენტის რედაქტირება</h2>
    <form id="edit-form">
      <div class="form-grid">
        <div class="form-group"><label>საწოლი</label>
          <select id="edit-bed">
            <option value="1">1</option><option value="2">2</option><option value="3">3</option><option value="4">4</option>
            <option value="5">5</option><option value="6">6</option><option value="7">7</option><option value="8">8</option>
            <option value="9">9</option><option value="10">10</option>
            <option value="ლოჯი">ლოჯი</option><option value="მცირე">მცირე</option>
          </select>
        </div>
        <div class="form-group"><label>პაციენტი</label><input id="edit-name" required></div>
        <div class="form-group"><label>ისტორია</label><input id="edit-history"></div>
        <div class="form-group"><label>ICD-10</label><input id="edit-icd"></div>
        <div class="form-group"><label>ექიმი</label><input id="edit-doctor"></div>
      </div>
      <div class="form-group"><label>კომენტარი</label><textarea id="edit-comment" rows="3"></textarea></div>
      <div style="display:flex; gap:15px; justify-content:flex-end; margin-top:25px;">
        <button type="button" class="btn" style="background:#f0f0f0; color:#333;" onclick="document.getElementById('edit-modal').classList.remove('active')">გაუქმება</button>
        <button type="submit" class="btn btn-primary">შენახვა</button>
      </div>
    </form>
  </div>
</div>

<script type="module">
  // ყველა ფუნქცია აქ არის
  function escapeHtml(text) { return text ? String(text).replace(/[&<>"']/g, m => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'})[m]) : ''; }
  function formatDate(ts) {
    if (!ts || !ts.toDate) return '-';
    const d = ts.toDate();
    return d.toLocaleDateString('ka-GE', {day:'numeric', month:'long', year:'numeric'}) + ' ' + d.toLocaleTimeString('ka-GE', {hour:'2-digit', minute:'2-digit'});
  }

  window.openRoom = function(room) {
    window.currentCollection = collection(window.db, room === 'observation' ? 'observation_room' : 'shock_room');
    document.querySelector('.login-screen').style.display = 'none';
    document.getElementById('app').style.display = 'block';
    document.getElementById('room-title').textContent = room === 'observation' ? 'ობსერვაციის დარბაზი' : 'შოკის დარბაზიressa';
    loadPatients();
  };

  window.renderTables = function() {
    const active = window.patients.filter(p => !p.archived);
    const archived = window.patients.filter(p => p.archived);

    let list = active;
    const search = document.getElementById('search')?.value.toLowerCase() || '';
    if (search) list = list.filter(p => (p.patient_name||'').toLowerCase().includes(search) || (p.history_number||'').includes(search));

    list.sort((a,b) => {
      let av = a[window.currentSort.column] ?? '';
      let bv = b[window.currentSort.column] ?? '';
      if (window.currentSort.column === 'timestamp') { av = a.timestamp?.seconds || 0; bv = b.timestamp?.seconds || 0; }
      return (av < bv ? -1 : 1) * (window.currentSort.dir === 'asc' ? 1 : -1);
    });

    document.getElementById('active-tbody').innerHTML = list.length === 0 ?
      '<tr><td colspan="8" class="empty-state">პაციენტები არ არის</td></tr>' :
      list.map(p => `
        <tr>
          <td>${escapeHtml(p.bed || '-')}</td>
          <td>${escapeHtml(p.patient_name || '-')}</td>
          <td>${escapeHtml(p.history_number || '-')}</td>
          <td>${escapeHtml(p.icd10_code || '-')}</td>
          <td>${escapeHtml(p.doctor || '-')}</td>
          <td>${escapeHtml(p.comment || '-')}</td>
          <td>${formatDate(p.timestamp)}</td>
          <td class="action-buttons">
            <button class="btn btn-primary" onclick="openEdit('${p.id}')">რედაქტირება</button>
            <button class="btn btn-danger" onclick="archivePatient('${p.id}')">არქივი</button>
            <button class="btn btn-delete" onclick="deletePatient('${p.id}')">წაშლა</button>
          </td>
        </tr>
      `).join('');

    document.getElementById('archive-tbody').innerHTML = archived.length === 0 ?
      '<tr><td colspan="9" class="empty-state">არქივი ცარიელია</td></tr>' :
      archived.map(p => `
        <tr>
          <td>${escapeHtml(p.bed || '-')}</td>
          <td>${escapeHtml(p.patient_name || '-')}</td>
          <td>${escapeHtml(p.history_number || '-')}</td>
          <td>${escapeHtml(p.icd10_code || '-')}</td>
          <td>${escapeHtml(p.doctor || '-')}</td>
          <td>${escapeHtml(p.comment || '-')}</td>
          <td>${formatDate(p.timestamp)}</td>
          <td>${formatDate(p.archived_at)}</td>
          <td class="action-buttons">
            <button class="btn btn-success" onclick="restorePatient('${p.id}')">დაბრუნება</button>
            <button class="btn btn-delete" onclick="deletePatient('${p.id}')">წაშლა</button>
          </td>
        </tr>
      `).join('');
  };

  window.updateStats = function() {
    const today = new Date().toDateString();
    const active = window.patients.filter(p => !p.archived).length;
    const archived = window.patients.filter(p => p.archived).length;
    const addedToday = window.patients.filter(p => !p.archived && p.timestamp && p.timestamp.toDate().toDateString() === today).length;
    const archivedToday = window.patients.filter(p => p.archived && p.archived_at && p.archived_at.toDate().toDateString() === today).length;

    document.getElementById('stat-active').textContent = active;
    document.getElementById('stat-today-added').textContent = addedToday;
    document.getElementById('stat-today-deleted').textContent = archivedToday;
    document.getElementById('stat-archived').textContent = archived;
  };

  window.clearAllData = async function() {
    if (!confirm("ყველა პაციენტი წაიშლება! გავაგრძელო?")) return;
    if (!confirm("სამუდამოდ წაშლა! დარწმუნებული ხართ?")) return;
    const snapshot = await getDocs(window.currentCollection);
    const deletes = snapshot.docs.map(d => deleteDoc(doc(window.db, window.currentCollection.path, d.id)));
    await Promise.all(deletes);
    showToast('დარბაზი გასუფთავდა');
  };

  document.getElementById('patient-form').onsubmit = async function(e) {
    e.preventDefault();
    await addDoc(window.currentCollection, {
      bed: document.getElementById('bed').value,
      patient_name: document.getElementById('patient-name').value.trim(),
      history_number: document.getElementById('history-number').value.trim(),
      icd10_code: document.getElementById('icd10').value.trim(),
      doctor: document.getElementById('doctor').value.trim(),
      comment: document.getElementById('comment').value.trim(),
      archived: false,
      timestamp: serverTimestamp()
    });
    this.reset();
    showToast('დამატებულია');
  };

  window.openEdit = function(id) {
    const p = window.patients.find(x => x.id === id);
    window.editingDocId = id;
    document.getElementById('edit-bed').value = p.bed || '';
    document.getElementById('edit-name').value = p.patient_name || '';
    document.getElementById('edit-history').value = p.history_number || '';
    document.getElementById('edit-icd').value = p.icd10_code || '';
    document.getElementById('edit-doctor').value = p.doctor || '';
    document.getElementById('edit-comment').value = p.comment || '';
    document.getElementById('edit-modal').classList.add('active');
  };

  document.getElementById('edit-form').onsubmit = async function(e) {
    e.preventDefault();
    await updateDoc(doc(window.db, window.currentCollection.path, window.editingDocId), {
      bed: document.getElementById('edit-bed').value,
      patient_name: document.getElementById('edit-name').value.trim(),
      history_number: document.getElementById('edit-history').value.trim(),
      icd10_code: document.getElementById('edit-icd').value.trim(),
      doctor: document.getElementById('edit-doctor').value.trim(),
      comment: document.getElementById('edit-comment').value.trim()
    });
    document.getElementById('edit-modal').classList.remove('active');
    showToast('შენახულია');
  };

  window.archivePatient = id => confirm('არქივში გადატანა?') && updateDoc(doc(window.db, window.currentCollection.path, id), { archived: true, archived_at: serverTimestamp() }).then(() => showToast('არქივშია'));
  window.restorePatient = id => confirm('დაბრუნება?') && updateDoc(doc(window.db, window.currentCollection.path, id), { archived: false, archived_at: null }).then(() => showToast('დაბრუნდა'));
  window.deletePatient = id => confirm('წაშლა?') && deleteDoc(doc(window.db, window.currentCollection.path, id)).then(() => showToast('წაიშალა'));

  window.sortTable = col => {
    if (window.currentSort.column === col) window.currentSort.dir = window.currentSort.dir === 'asc' ? 'desc' : 'asc';
    else { window.currentSort.column = col; window.currentSort.dir = 'asc'; }
    renderTables();
  };

  window.switchTab = tab => {
    document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
    document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
    event.target.classList.add('active');
    document.getElementById(tab + '-tab').classList.add('active');
  };

  document.getElementById('search').addEventListener('input', renderTables);

  window.loadPatients = function() {
    if (!window.currentCollection) return;
    const q = query(window.currentCollection, orderBy("timestamp", "desc"));
    onSnapshot(q, snapshot => {
      window.patients = [];
      snapshot.forEach(doc => window.patients.push({ id: doc.id, ...doc.data() }));
      renderTables();
      updateStats();
    });
  };
</script>
</body>
</html>
