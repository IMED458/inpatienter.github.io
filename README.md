<!DOCTYPE html>
<html lang="ka">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>საწოლების მართვა</title>
  <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;600;700&display=swap" rel="stylesheet">

  <!-- GitHub Pages-ის ყველა ზედმეტი ზოლი და ჩარჩო სრულად ქრება -->
  <meta name="theme-color" content="#2563eb">
  <style>
    html, body {
      height: 100%;
      margin: 0;
      padding: 0;
      overflow: hidden;
      font-family: 'Roboto', sans-serif;
    }
    body { display: flex; flex-direction: column; background: linear-gradient(135deg, #f0f7ff 0%, #e0f2fe 100%); }

    /* GitHub Pages-ის ყველა ზედა ელემენტი იმალება */
    iframe, [aria-label="Site"], header, [data-testid], .github-corner {
      display: none !important;
    }
  </style>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-app.js";
    import { getFirestore, collection, addDoc, updateDoc, deleteDoc, doc, onSnapshot, serverTimestamp, query, orderBy, getDocs } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-firestore.js";

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

    let currentCollection = null;
    let patients = [];
    let currentSort = { column: 'timestamp', dir: 'desc' };
    let editingId = null;

    function showToast(msg, error = false) {
      const t = document.getElementById('toast');
      t.textContent = msg;
      t.className = 'toast active' + (error ? ' error' : '');
      setTimeout(() => t.classList.remove('active'), 4000);
    }

    function escapeHtml(text) { return text ? String(text).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'})[c]) : '-'; }
    function formatDate(ts) { return ts?.toDate?.() ? ts.toDate().toLocaleString('ka-GE', {day:'numeric', month:'long', year:'numeric', hour:'2-digit', minute:'2-digit'}) : '-'; }

    window.openRoom = function(room) {
      currentCollection = collection(db, room === 'observation' ? 'observation_room' : 'shock_room');
      document.getElementById('welcome-screen').style.display = 'none';
      document.getElementById('app-container').style.display = 'block';
      document.querySelector('.header h1').textContent = room === 'observation' ? 'ობსერვაციის დარბაზი' : 'შოკის დარბაზი';
      loadPatients();
    };

    function loadPatients() {
      if (!currentCollection) return;
      const q = query(currentCollection, orderBy("timestamp", "desc"));
      onSnapshot(q, snap => {
        patients = snap.docs.map(d => ({ id: d.id, ...d.data() }));
        renderActive();
        renderArchive();
        updateStats();
      });
    }

    function renderActive() {
      let list = patients.filter(p => !p.archived);
      const search = document.getElementById('search')?.value.toLowerCase() || '';
      if (search) list = list.filter(p => (p.patient_name||'').toLowerCase().includes(search) || (p.history_number||'').includes(search) || (p.bed||'').toLowerCase().includes(search));

      list.sort((a,b) => {
        let av = a[currentSort.column] ?? '';
        let bv = b[currentSort.column] ?? '';
        if (currentSort.column === 'timestamp') { av = a.timestamp?.seconds || 0; bv = b.timestamp?.seconds || 0; }
        return (av < bv ? -1 : 1) * (currentSort.dir === 'asc' ? 1 : -1);
      });

      const tbody = document.getElementById('active-tbody');
      tbody.innerHTML = list.length === 0 ? '<tr><td colspan="8" class="empty-state">პაციენტები არ არის</td></tr>' : list.map(p => `
        <tr>
          <td><strong>${escapeHtml(p.bed)}</strong></td>
          <td>${escapeHtml(p.patient_name)}</td>
          <td>${escapeHtml(p.history_number)}</td>
          <td>${escapeHtml(p.icd10_code)}</td>
          <td>${escapeHtml(p.doctor)}</td>
          <td>${escapeHtml(p.comment)}</td>
          <td>${formatDate(p.timestamp)}</td>
          <td class="action-buttons">
            <button class="btn btn-edit" onclick="openEditModal('${p.id}')">რედაქტირება</button>
            <button class="btn btn-archive" onclick="archivePatient('${p.id}')">არქივი</button>
            <button class="btn btn-delete" onclick="permanentlyDelete('${p.id}')">წაშლა</button>
          </td>
        </tr>
      `).join('');
    }

    function renderArchive() {
      const list = patients.filter(p => p.archived);
      const tbody = document.getElementById('archive-tbody');
      tbody.innerHTML = list.length === 0 ? '<tr><td colspan="9" class="empty-state">არქივი ცარიელია</td></tr>' : list.map(p => `
        <tr>
          <td>${escapeHtml(p.bed)}</td>
          <td>${escapeHtml(p.patient_name)}</td>
          <td>${escapeHtml(p.history_number)}</td>
          <td>${escapeHtml(p.icd10_code)}</td>
          <td>${escapeHtml(p.doctor)}</td>
          <td>${escapeHtml(p.comment)}</td>
          <td>${formatDate(p.timestamp)}</td>
          <td>${formatDate(p.archived_at)}</td>
          <td class="action-buttons">
            <button class="btn btn-restore" onclick="restorePatient('${p.id}')">აღდგენა</button>
            <button class="btn btn-delete" onclick="permanentlyDelete('${p.id}')">წაშლა</button>
          </td>
        </tr>
      `).join('');
    }

    function updateStats() {
      const today = new Date().toDateString();
      const active = patients.filter(p => !p.archived).length;
      const archived = patients.filter(p => p.archived).length;
      const addedToday = patients.filter(p => !p.archived && p.timestamp?.toDate?.().toDateString() === today).length;
      const dischargedToday = patients.filter(p => p.archived && p.archived_at?.toDate?.().toDateString() === today).length;
      document.getElementById('stat-active').textContent = active;
      document.getElementById('stat-today-added').textContent = addedToday;
      document.getElementById('stat-today-deleted').textContent = dischargedToday;
      document.getElementById('stat-archived').textContent = archived;
    }

    document.getElementById('patient-form').addEventListener('submit', async e => {
      e.preventDefault();
      await addDoc(currentCollection, {
        bed: document.getElementById('bed').value,
        patient_name: document.getElementById('patient-name').value.trim(),
        history_number: document.getElementById('history-number').value.trim() || null,
        icd10_code: document.getElementById('icd10').value.trim() || null,
        doctor: document.getElementById('doctor').value.trim() || null,
        comment: document.getElementById('comment').value.trim() || null,
        archived: false,
        timestamp: serverTimestamp()
      });
      e.target.reset();
      showToast('პაციენტი დამატებულია');
    });

    window.openEditModal = id => {
      const p = patients.find(x => x.id === id);
      editingId = id;
      document.getElementById('edit-bed').value = p.bed || '';
      document.getElementById('edit-name').value = p.patient_name || '';
      document.getElementById('edit-history').value = p.history_number || '';
      document.getElementById('edit-icd').value = p.icd10_code || '';
      document.getElementById('edit-doctor').value = p.doctor || '';
      document.getElementById('edit-comment').value = p.comment || '';
      document.getElementById('edit-modal').classList.add('active');
    };

    window.closeEditModal = () => document.getElementById('edit-modal').classList.remove('active');

    document.getElementById('edit-form').addEventListener('submit', async e => {
      e.preventDefault();
      await updateDoc(doc(db, currentCollection.path, editingId), {
        bed: document.getElementById('edit-bed').value,
        patient_name: document.getElementById('edit-name').value.trim(),
        history_number: document.getElementById('edit-history').value.trim() || null,
        icd10_code: document.getElementById('edit-icd').value.trim() || null,
        doctor: document.getElementById('edit-doctor').value.trim() || null,
        comment: document.getElementById('edit-comment').value.trim() || null,
      });
      closeEditModal();
      showToast('ცვლილებები შენახულია');
    });

    window.archivePatient = id => confirm('არქივში გადატანა?') && updateDoc(doc(db, currentCollection.path, id), {archived: true, archived_at: serverTimestamp()}).then(() => showToast('არქივში გადავიდა'));
    window.restorePatient = id => confirm('აღდგენა?') && updateDoc(doc(db, currentCollection.path, id), {archived: false, archived_at: null}).then(() => showToast('აღდგენილია'));
    window.permanentlyDelete = id => confirm('სამუდამოდ წაშლა?') && confirm('დარწმუნებული ხართ?') && deleteDoc(doc(db, currentCollection.path, id)).then(() => showToast('წაიშალა'));

    window.clearAllData = async () => {
      if (!confirm('ყველა წაიშლება სამუდამოდ!') || !confirm('დარწმუნებული ხართ?')) return;
      const snap = await getDocs(currentCollection);
      await Promise.all(snap.docs.map(d => deleteDoc(d.ref)));
      showToast('დარბაზი გასუფთავდა');
    };

    window.sortTable = col => {
      if (currentSort.column === col) currentSort.dir = currentSort.dir === 'asc' ? 'desc' : 'asc';
      else { currentSort.column = col; currentSort.dir = 'asc'; }
      renderActive();
    };

    window.switchTab = tab => {
      document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
      document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
      document.querySelector(`.tab-btn[onclick="switchTab('${tab}')"]`).classList.add('active');
      document.getElementById(tab + '-tab').classList.add('active');
    };

    document.getElementById('search')?.addEventListener('input', renderActive);
  </script>

  <style>
    :root { --primary:#2563eb; --primary-light:#3b82f6; --success:#10b981; --danger:#ef4444; --warning:#f59e0b; }
    #welcome-screen, #app-container { flex:1; min-height:100vh; display:flex; flex-direction:column; }
    .welcome { flex:1; display:flex; flex-direction:column; align-items:center; justify-content:center; background:linear-gradient(135deg,#3b82f6,#1d4ed8); color:white; text-align:center; padding:2rem; }
    .welcome h1 { font-size:3.5rem; margin-bottom:1rem; }
    .room-choice { background:white; color:#1d4ed8; padding:2rem 4rem; border-radius:20px; font-size:1.8rem; font-weight:600; cursor:pointer; margin:1.5rem; min-width:380px; box-shadow:0 20px 40px rgba(0,0,0,0.2); transition:0.3s; }
    .room-choice:hover { transform:translateY(-10px); }

    .back-to-rooms { position:fixed; top:1rem; left:1rem; background:white; color:#2563eb; border:none; padding:0.8rem 1.5rem; border-radius:50px; font-weight:600; cursor:pointer; z-index:999; box-shadow:0 8px 20px rgba(0,0,0,0.15); }

    .container { max-width:1500px; margin:0 auto; padding:0 1.5rem; flex:1; display:flex; flex-direction:column; }
    .header { background:white; padding:2rem; border-radius:20px; text-align:center; margin:6rem 0 2rem; box-shadow:0 10px 30px rgba(0,0,0,0.1); }
    .header h1 { font-size:2.8rem; color:#1d4ed8; }

    .tabs { display:flex; gap:1rem; margin-bottom:2rem; background:white; padding:1rem; border-radius:16px; box-shadow:0 10px 25px rgba(0,0,0,0.08); justify-content:center; flex-wrap:wrap; }
    .tab-btn { padding:0.9rem 2rem; background:transparent; border:none; border-radius:12px; color:#475569; font-weight:600; cursor:pointer; }
    .tab-btn:hover { background:#f1f5f9; }
    .tab-btn.active { background:var(--primary); color:white; }

    .tab-content { display:none; flex:1; }
    .tab-content.active { display:block; }

    .card { background:white; border-radius:20px; padding:2rem; margin-bottom:2rem; box-shadow:0 10px 30px rgba(0,0,0,0.08); }
    .card h2 { font-size:1.8rem; margin-bottom:1.5rem; color:#1e293b; padding-bottom:0.8rem; border-bottom:3px solid var(--primary-light); display:inline-block; }

    .form-grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(280px,1fr)); gap:1.5rem; margin-bottom:1.5rem; }
    .form-group label { display:block; margin-bottom:0.6rem; font-weight:600; }
    .form-group input, .form-group select, .form-group textarea { width:100%; padding:1rem; border-radius:12px; border:2px solid #e2e8f0; font-size:1rem; }
    .form-group input:focus, .form-group select:focus, .form-group textarea:focus { outline:none; border-color:var(--primary-light); box-shadow:0 0 0 4px rgba(59,130,246,0.15); }

    .btn { padding:0.9rem 1.8rem; border:none; border-radius:12px; cursor:pointer; font-weight:600; transition:0.3s; }
    .btn-primary { background:var(--primary); color:white; }
    .btn-primary:hover { background:#1d4ed8; transform:translateY(-2px); }
    .btn-edit { background:#6366f1; color:white; }
    .btn-archive { background:var(--warning); color:white; }
    .btn-restore { background:var(--success); color:white; }
    .btn-delete { background:var(--danger); color:white; }

    table { width:100%; border-collapse:collapse; border-radius:16px; overflow:hidden; box-shadow:0 10px 25px rgba(0,0,0,0.08); }
    th { background:var(--primary); color:white; padding:1.3rem 1rem; text-align:left; cursor:pointer; }
    td { padding:1.2rem 1rem; border-bottom:1px solid #f1f5f9; }
    tr:hover { background:#f8fafc; }
    .action-buttons { display:flex; gap:0.8rem; flex-wrap:wrap; }
    .empty-state { text-align:center; padding:4rem; color:#94a3b8; font-size:1.3rem; }

    .stats-grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(260px,1fr)); gap:1.5rem; }
    .stat-card { background:linear-gradient(135deg,var(--primary-light),var(--primary)); color:white; padding:2rem; border-radius:20px; text-align:center; box-shadow:0 15px 35px rgba(59,130,246,0.25); }
    .stat-card .number { font-size:3.8rem; font-weight:700; }

    .modal { display:none; position:fixed; inset:0; background:rgba(0,0,0,0.5); z-index:1000; align-items:center; justify-content:center; }
    .modal.active { display:flex; }
    .modal-content { background:white; padding:2.5rem; border-radius:20px; max-width:650px; width:90%; box-shadow:0 25px 50px rgba(0,0,0,0.2); }

    .toast { position:fixed; top:2rem; right:2rem; padding:1.2rem 2rem; border-radius:14px; color:white; font-weight:600; z-index:2000; opacity:0; transform:translateX(100%); transition:0.4s; background:var(--success); }
    .toast.active { opacity:1; transform:translateX(0); }
    .toast.error { background:var(--danger); }

    .clear-all-btn { background:var(--danger); color:white; padding:1.2rem 2.5rem; border:none; border-radius:14px; font-weight:600; cursor:pointer; margin-top:1.5rem; }
    .clear-all-btn:hover { background:#b91c1c; }
  </style>
</head>
<body>

  <div id="welcome-screen" class="welcome">
    <h1>საწოლების მართვა</h1>
    <p>აირჩიეთ დარბაზი სამუშაოდ</p>
    <div class="room-choice" onclick="openRoom('observation')">ობსერვაციის დარბაზი</div>
    <div class="room-choice" onclick="openRoom('shock')">შოკის დარბაზი</div>
  </div>

  <div id="app-container" style="display:none">
    <button class="back-to-rooms" onclick="location.reload()">დარბაზის შეცვლა</button>
    <div class="container">
      <div class="header"><h1>დარბაზი</h1><p>სტაციონარული პაციენტების მართვა</p></div>

      <div class="tabs">
        <button class="tab-btn active" onclick="switchTab('active')">აქტიური პაციენტები</button>
        <button class="tab-btn" onclick="switchTab('archive')">არქივი</button>
        <button class="tab-btn" onclick="switchTab('statistics')">სტატისტიკა</button>
      </div>

      <div id="active-tab" class="tab-content active">
        <div class="card">
          <h2>ახალი პაციენტის დამატება</h2>
          <form id="patient-form">
            <div class="form-grid">
              <div class="form-group"><label>საწოლი</label><select id="bed" required><option value="">აირჩიეთ</option><option>1</option><option>2</option><option>3</option><option>4</option><option>5</option><option>6</option><option>7</option><option>8</option><option>9</option><option>10</option><option>ლოჯი</option><option>მცირე</option></select></div>
              <div class="form-group"><label>პაციენტი</label><input id="patient-name" required></div>
              <div class="form-group"><label>ისტორია</label><input id="history-number"></div>
              <div class="form-group"><label>ICD-10</label><input id="icd10"></div>
              <div class="form-group"><label>ექიმი</label><input id="doctor"></div>
            </div>
            <div class="form-group"><label>კომენტარი</label><textarea id="comment" rows="3"></textarea></div>
            <button type="submit" class="btn btn-primary">დამატება</button>
          </form>
          <button type="button" class="clear-all-btn" onclick="clearAllData()">ყველას წაშლა</button>
        </div>

        <div class="card">
          <h2>აქტიური პაციენტები</h2>
          <div style="margin-bottom:1rem;"><input type="text" id="search" placeholder="ძებნა..." style="width:100%; max-width:500px; padding:1rem; border-radius:14px; border:2px solid #e2e8f0;"></div>
          <table><thead><tr>
            <th onclick="sortTable('bed')">საწოლი</th>
            <th onclick="sortTable('patient_name')">პაციენტი</th>
            <th onclick="sortTable('history_number')">ისტორია</th>
            <th onclick="sortTable('icd10_code')">ICD-10</th>
            <th onclick="sortTable('doctor')">ექიმი</th>
            <th>კომენტარი</th>
            <th onclick="sortTable('timestamp')">ჩარიცხვა</th>
            <th>მოქმედება</th>
          </tr></thead><tbody id="active-tbody"></tbody></table>
        </div>
      </div>

      <div id="archive-tab" class="tab-content">
        <div class="card"><h2>არქივი</h2>
          <table><thead><tr><th>საწოლი</th><th>პაციენტი</th><th>ისტორია</th><th>ICD-10</th><th>ექიმი</th><th>კომენტარი</th><th>ჩარიცხვა</th><th>გაწერა</th><th>მოქმედება</th></tr></thead><tbody id="archive-tbody"></tbody></table>
        </div>
      </div>

      <div id="statistics-tab" class="tab-content">
        <div class="card"><h2>დღიური სტატისტიკა</h2>
          <div class="stats-grid">
            <div class="stat-card"><h3>აქტიური</h3><div class="number" id="stat-active">0</div></div>
            <div class="stat-card"><h3>დღეს დამატებული</h3><div class="number" id="stat-today-added">0</div></div>
            <div class="stat-card"><h3>დღეს გაწერილი</h3><div class="number" id="stat-today-deleted">0</div></div>
            <div class="stat-card"><h3>სულ არქივში</h3><div class="number" id="stat-archived">0</div></div>
          </div>
        </div>
      </div>
    </div>

    <div id="edit-modal" class="modal">
      <div class="modal-content">
        <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:1.5rem;">
          <h2 style="margin:0; color:#1d4ed8;">რედაქტირება</h2>
          <button onclick="closeEditModal()" style="background:none; border:none; font-size:2.5rem; cursor:pointer;">×</button>
        </div>
        <form id="edit-form">
          <div class="form-grid">
            <div class="form-group"><label>საწოლი</label><select id="edit-bed"><option>1</option><option>2</option><option>3</option><option>4</option><option>5</option><option>6</option><option>7</option><option>8</option><option>9</option><option>10</option><option>ლოჯი</option><option>მცირე</option></select></div>
            <div class="form-group"><label>პაციენტი</label><input id="edit-name" required></div>
            <div class="form-group"><label>ისტორია</label><input id="edit-history"></div>
            <div class="form-group"><label>ICD-10</label><input id="edit-icd"></div>
            <div class="form-group"><label>ექიმი</label><input id="edit-doctor"></div>
          </div>
          <div class="form-group"><label>კომენტარი</label><textarea id="edit-comment" rows="3"></textarea></div>
          <div style="display:flex; gap:1rem; justify-content:flex-end; margin-top:2rem;">
            <button type="button" class="btn" style="background:#94a3b8; color:white;" onclick="closeEditModal()">გაუქმება</button>
            <button type="submit" class="btn btn-primary">შენახვა</button>
          </div>
        </form>
      </div>
    </div>

    <div id="toast" class="toast"></div>
  </div>
</body>
</html>
