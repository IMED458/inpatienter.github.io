<!DOCTYPE html>
<html lang="ka">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>კლინიკა • საწოლების მართვა (ტესტი)</title>

  <!-- Firebase SDK v10+ -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-app.js";
    import { getFirestore, collection, addDoc, deleteDoc, doc, updateDoc, onSnapshot, serverTimestamp, query, orderBy, getDocs } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-firestore.js";
    import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-auth.js";

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
    const auth = getAuth(app);

    // ანონიმური შესვლა
    signInAnonymously(auth).catch(() => console.log("ანონიმური ავტორიზაცია"));

    window.db = db;
    window.auth = auth;
    window.currentCollection = null;
    window.patients = [];
    window.currentSort = { column: 'timestamp', dir: 'desc' };
    window.editingDocId = null;

    // Toast
    window.showToast = function(msg, error = false) {
      const toast = document.getElementById('toast');
      toast.textContent = msg;
      toast.className = 'toast active' + (error ? ' error' : '');
      setTimeout(() => toast.classList.remove('active'), 4000);
    };

    function escapeHtml(text) {
      if (text === null || text === undefined) return '-';
      const div = document.createElement('div');
      div.textContent = text;
      return div.innerHTML;
    }

    function formatDate(ts) {
      if (!ts?.toDate) return '-';
      return ts.toDate().toLocaleString('ka-GE', {
        day: 'numeric', month: 'long', year: 'numeric',
        hour: '2-digit', minute: '2-digit'
      });
    }

    // დარბაზის გახსნა
    window.openRoom = function(room) {
      onAuthStateChanged(auth, user => {
        if (!user) return showToast('ავტორიზაცია ვერ მოხერხდა', true);

        const collName = room === 'observation' ? 'observation_room' : 'shock_room';
        window.currentCollection = collection(db, collName);

        document.getElementById('login-screen').style.display = 'none';
        document.getElementById('app').style.display = 'block';
        document.getElementById('room-title').textContent = room === 'observation' 
          ? 'ობსერვაციის დარბაზი' : 'შოკის დარბაზი';

        loadPatients();
      });
    };

    // ტაბები
    window.switchTab = function(btn, tab) {
      document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
      document.querySelectorAll('.tab-content').forEach(c => c.style.display = 'none');
      btn.classList.add('active');
      document.getElementById(tab + '-tab').style.display = 'block';
    };

    // ცხრილის რენდერი
    window.renderTables = function() {
      const search = (document.getElementById('search')?.value || '').toLowerCase();
      const active = window.patients.filter(p => !p.archived);
      const archived = window.patients.filter(p => p.archived);

      let list = active.filter(p =>
        (p.patient_name || '').toLowerCase().includes(search) ||
        (p.history_number || '').includes(search) ||
        (p.bed || '').toLowerCase().includes(search) ||
        (p.doctor || '').toLowerCase().includes(search)
      );

      // სორტირება
      list.sort((a, b) => {
        let av = a[window.currentSort.column] ?? '';
        let bv = b[window.currentSort.column] ?? '';
        if (window.currentSort.column === 'timestamp') {
          av = a.timestamp?.seconds || 0;
          bv = b.timestamp?.seconds || 0;
        } else if (typeof av === 'string') {
          av = av.toLowerCase();
          bv = bv.toLowerCase();
        }
        const result = av < bv ? -1 : av > bv ? 1 : 0;
        return result * (window.currentSort.dir === 'asc' ? 1 : -1);
      });

      // აქტიურები
      document.getElementById('active-tbody').innerHTML = list.length === 0
        ? '<tr><td colspan="8" class="empty-state">პაციენტები არ არის</td></tr>'
        : list.map(p => `
          <tr>
            <td><strong>${escapeHtml(p.bed)}</strong></td>
            <td>${escapeHtml(p.patient_name)}</td>
            <td>${escapeHtml(p.history_number)}</td>
            <td>${escapeHtml(p.icd10_code)}</td>
            <td>${escapeHtml(p.doctor)}</td>
            <td>${escapeHtml(p.comment)}</td>
            <td>${formatDate(p.timestamp)}</td>
            <td>
              <button class="btn btn-primary" onclick="openEdit('${p.id}')">რედ.</button>
              <button class="btn btn-danger" onclick="archivePatient('${p.id}')">არქივი</button>
              <button class="btn btn-delete" onclick="deletePatient('${p.id}')">წაშლა</button>
            </td>
          </tr>
        `).join('');

      // არქივი
      document.getElementById('archive-tbody').innerHTML = archived.length === 0
        ? '<tr><td colspan="9" class="empty-state">არქივი ცარიელია</td></tr>'
        : archived.map(p => `
          <tr>
            <td>${escapeHtml(p.bed)}</td>
            <td>${escapeHtml(p.patient_name)}</td>
            <td>${escapeHtml(p.history_number)}</td>
            <td>${escapeHtml(p.icd10_code)}</td>
            <td>${escapeHtml(p.doctor)}</td>
            <td>${escapeHtml(p.comment)}</td>
            <td>${formatDate(p.timestamp)}</td>
            <td>${formatDate(p.archived_at)}</td>
            <td>
              <button class="btn btn-success" onclick="restorePatient('${p.id}')">აღდგენა</button>
              <button class="btn btn-delete" onclick="deletePatient('${p.id}')">წაშლა</button>
            </td>
          </tr>
        `).join('');
    };

    // სტატისტიკა
    window.updateStats = function() {
      const today = new Date().toDateString();
      const activeCount = window.patients.filter(p => !p.archived).length;
      const archivedCount = window.patients.filter(p => p.archived).length;
      const addedToday = window.patients.filter(p => !p.archived && p.timestamp?.toDate?.().toDateString() === today).length;
      const archivedToday = window.patients.filter(p => p.archived && p.archived_at?.toDate?.().toDateString() === today).length;

      document.getElementById('stat-active').textContent = activeCount;
      document.getElementById('stat-today-added').textContent = addedToday;
      document.getElementById('stat-today-deleted').textContent = archivedToday;
      document.getElementById('stat-archived').textContent = archivedCount;
    };

    // მონაცემების ჩატვირთვა
    window.loadPatients = function() {
      if (!window.currentCollection) return;
      const q = query(window.currentCollection, orderBy("timestamp", "desc"));
      onSnapshot(q, snapshot => {
        window.patients = snapshot.docs.map(d => ({ id: d.id, ...d.data() }));
        renderTables();
        updateStats();
      }, err => {
        showToast('მონაცემების ჩატვირთვა ვერ მოხერხდა', true);
        console.error(err);
      });
    };

    // ახალი პაციენტის დამატება
    document.getElementById('patient-form').onsubmit = async function(e) {
      e.preventDefault();
      try {
        await addDoc(window.currentCollection, {
          bed: document.getElementById('bed').value,
          patient_name: document.getElementById('patient-name').value.trim(),
          history_number: document.getElementById('history-number').value.trim() || null,
          icd10_code: document.getElementById('icd10').value.trim() || null,
          doctor: document.getElementById('doctor').value.trim() || null,
          comment: document.getElementById('comment').value.trim() || null,
          archived: false,
          timestamp: serverTimestamp()
        });
        this.reset();
        showToast('პაციენტი დაემატა!');
      } catch (err) {
        showToast('შეცდომა: ' + err.message, true);
      }
    };

    // რედაქტირება
    window.openEdit = function(id) {
      const p = window.patients.find(x => x.id === id);
      if (!p) return;
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
      try {
        await updateDoc(doc(db, window.currentCollection.path, window.editingDocId), {
          bed: document.getElementById('edit-bed').value,
          patient_name: document.getElementById('edit-name').value.trim(),
          history_number: document.getElementById('edit-history').value.trim() || null,
          icd10_code: document.getElementById('edit-icd').value.trim() || null,
          doctor: document.getElementById('edit-doctor').value.trim() || null,
          comment: document.getElementById('edit-comment').value.trim() || null,
        });
        document.getElementById('edit-modal').classList.remove('active');
        showToast('ცვლილებები შენახულია');
      } catch (err) {
        showToast('შენახვა ვერ მოხერხდა', true);
      }
    };

    // ქმედებები
    window.archivePatient = async id => {
      if (!confirm('გადატანა არქივში?')) return;
      try {
        await updateDoc(doc(db, window.currentCollection.path, id), { archived: true, archived_at: serverTimestamp() });
        showToast('არქივში გადავიდა');
      } catch (err) { showToast('შეცდომა', true); }
    };

    window.restorePatient = async id => {
      if (!confirm('აღდგენა აქტიურებში?')) return;
      try {
        await updateDoc(doc(db, window.currentCollection.path, id), { archived: false, archived_at: null });
        showToast('აღდგენილია');
      } catch (err) { showToast('შეცდომა', true); }
    };

    window.deletePatient = async id => {
      if (!confirm('სამუდამოდ წაშლა?')) return;
      if (!confirm('დარწმუნებული ხართ? ეს შეუქცევადია!')) return;
      try {
        await deleteDoc(doc(db, window.currentCollection.path, id));
        showToast('პაციენტი წაიშალა');
      } catch (err) { showToast('წაშლა ვერ მოხერხდა', true); }
    };

    window.clearAllData = async function() {
      if (!confirm("ყველა პაციენტი წაიშლება!")) return;
      if (!confirm("დარწმუნებული ხართ?")) return;
      const pass = prompt("ჩაწერეთ „ტესტი წაშლა“");
      if (pass !== "ტესტი წაშლა") return showToast('გაუქმდა');
      
      try {
        const snapshot = await getDocs(window.currentCollection);
        const deletes = snapshot.docs.map(d => deleteDoc(doc(db, window.currentCollection.path, d.id)));
        await Promise.all(deletes);
        showToast('დარბაზი გასუფთავდა');
      } catch (err) {
        showToast('ვერ მოხერხდა წაშლა', true);
      }
    };

    window.sortTable = col => {
      if (window.currentSort.column === col) {
        window.currentSort.dir = window.currentSort.dir === 'asc' ? 'desc' : 'asc';
      } else {
        window.currentSort.column = col;
        window.currentSort.dir = 'asc';
      }
      renderTables();
    };

    document.getElementById('search')?.addEventListener('input', renderTables);
  </script>

  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: 'Segoe UI', sans-serif; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); min-height: 100vh; padding: 20px; }
    .login-screen { display: flex; gap: 100px; justify-content: center; align-items: center; min-height: 100vh; flex-wrap: wrap; }
    .room-btn { width: 450px; padding: 80px 40px; background: white; border-radius: 32px; text-align: center; box-shadow: 0 30px 80px rgba(0,0,0,0.4); cursor: pointer; transition: all 0.4s ease; border: 5px solid transparent; }
    .room-btn:hover { transform: translateY(-25px) scale(1.08); box-shadow: 0 50px 120px rgba(0,0,0,0.55); border-color: #667eea; }
    .room-btn h2 { font-size: 2.8em; color: #333; margin-bottom: 20px; }
    .room-btn p { font-size: 1.4em; color: #666; margin-bottom: 35px; }
    .enter-arrow { font-size: 4.5em; color: #667eea; transition: 0.4s; }
    .room-btn:hover .enter-arrow { color: #5568d3; transform: scale(1.4); }

    #app { display: none; }
    .back-btn { position: fixed; top: 25px; left: 25px; background: rgba(255,255,255,0.3); backdrop-filter: blur(12px); color: white; border: none; padding: 16px 36px; border-radius: 50px; font-size: 1.2em; font-weight: bold; cursor: pointer; z-index: 999; }
    .back-btn:hover { background: rgba(255,255,255,0.5); transform: scale(1.05); }

    .container { max-width: 1500px; margin: 0 auto; padding: 20px; }
    .header { background: white; padding: 45px; border-radius: 24px; box-shadow: 0 12px 35px rgba(0,0,0,0.18); margin-bottom: 35px; text-align: center; }
    .header h1 { margin: 0 0 12px; color: #667eea; font-size: 3.2em; }
    .header p { color: #888; font-size: 1.3em; }

    .tabs { display: flex; gap: 18px; margin-bottom: 35px; flex-wrap: wrap; justify-content: center; }
    .tab-btn { padding: 18px 36px; background: white; border: none; border-radius: 14px; cursor: pointer; font-size: 19px; font-weight: 600; color: #667eea; transition: all 0.3s; box-shadow: 0 6px 18px rgba(0,0,0,0.12); }
    .tab-btn:hover { transform: translateY(-5px); }
    .tab-btn.active { background: #667eea; color: white; }

    .card { background: white; padding: 45px; border-radius: 24px; box-shadow: 0 12px 35px rgba(0,0,0,0.14); margin-bottom: 35px; }
    .form-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 28px; margin-bottom: 35px; }
    .form-group label { display: block; margin-bottom: 12px; font-weight: 600; color: #333; font-size: 1.1em; }
    .form-group input, .form-group select, .form-group textarea { width: 100%; padding: 18px; border: 2px solid #e0e0e0; border-radius: 14px; font-size: 17px; }
    .form-group input:focus, .form-group select:focus, .form-group textarea:focus { outline: none; border-color: #667eea; box-shadow: 0 0 0 5px rgba(102,126,234,0.15); }

    .btn { padding: 18px 36px; border: none; border-radius: 14px; cursor: pointer; font-size: 18px; font-weight: 600; transition: all 0.3s; margin: 8px; }
    .btn-primary { background: #667eea; color: white; }
    .btn-primary:hover { background: #5568d3; transform: translateY(-4px); }
    .btn-danger { background: #e74c3c; color: white; }
    .btn-success { background: #27ae60; color: white; }
    .btn-delete { background: #c0392b; color: white; font-size: 15px; padding: 10px 18px; }

    .clear-all-btn { background: #e74c3c; color: white; padding: 20px 50px; border: none; border-radius: 14px; font-size: 19px; font-weight: bold; cursor: pointer; margin: 35px auto; display: block; box-shadow: 0 10px 30px rgba(231,76,60,0.45); }
    .clear-all-btn:hover { background: #c0392b; transform: translateY(-5px); }

    table { width: 100%; border-collapse: collapse; background: white; border-radius: 18px; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.12); margin-top: 25px; }
    th, td { padding: 20px; text-align: left; border-bottom: 1px solid #eee; }
    th { background: #f8f9fa; font-weight: 600; color: #333; cursor: pointer; user-select: none; }
    tr:hover { background: #f9faff; }
    .empty-state { text-align: center; padding: 100px 20px; color: #999; font-size: 1.5em; }

    .stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 30px; margin-top: 25px; }
    .stat-card { background: linear-gradient(135deg, #667eea, #764ba2); color: white; padding: 40px; border-radius: 24px; text-align: center; box-shadow: 0 12px 35px rgba(0,0,0,0.35); }
    .stat-card h3 { font-size: 1.5em; margin-bottom: 12px; }
    .stat-card .number { font-size: 4.5em; font-weight: bold; }

    .toast { position: fixed; top: 30px; right: 30px; background: #27ae60; color: white; padding: 22px 40px; border-radius: 16px; z-index: 2000; display: none; box-shadow: 0 15px 40px rgba(0,0,0,0.35); font-size: 1.2em; font-weight: bold; }
    .toast.active { display: block; animation: slideIn 0.5s; }
    .toast.error { background: #e74c3c; }
    @keyframes slideIn { from { transform: translateX(100%); } to { transform: translateX(0); } }

    .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.75); z-index: 1000; justify-content: center; align-items: center; }
    .modal.active { display: flex; }
    .modal-content { background: white; padding: 45px; border-radius: 24px; width: 90%; max-width: 750px; max-height: 90vh; overflow-y: auto; }
  </style>
</head>
<body>

  <div class="login-screen" id="login-screen">
    <div class="room-btn" onclick="openRoom('observation')">
      <h2>ობსერვაციის დარბაზი</h2>
      <p>აქტიური პაციენტები</p>
      <div class="enter-arrow">→</div>
    </div>
    <div class="room-btn" onclick="openRoom('shock')">
      <h2>შოკის დარბაზი</h2>
      <p>კრიტიკული მდგომარეობა</p>
      <div class="enter-arrow">→</div>
    </div>
  </div>

  <div id="app">
    <button class="back-btn" onclick="location.reload()">დარბაზის შეცვლა</button>
    <div class="container">
      <div class="header">
        <h1 id="room-title">დარბაზი</h1>
        <p>სტაციონარი / Inpatient</p>
      </div>

      <div class="tabs">
        <button class="tab-btn active" onclick="switchTab(this, 'active')">აქტიური</button>
        <button class="tab-btn" onclick="switchTab(this, 'archive')">არქივი</button>
        <button class="tab-btn" onclick="switchTab(this, 'statistics')">სტატისტიკა</button>
      </div>

      <div id="active-tab" class="tab-content" style="display: block;">
        <div class="card">
          <h2>ახალი პაციენტის დამატება</h2>
          <form id="patient-form">
            <div class="form-grid">
              <div class="form-group">
                <label>საწოლი</label>
                <select id="bed" required>
                  <option value="">აირჩიეთ</option>
                  <option>1</option><option>2</option><option>3</option><option>4</option><option>5</option>
                  <option>6</option><option>7</option><option>8</option><option>9</option><option>10</option>
                  <option>ლოჯი</option><option>მცირე</option>
                </select>
              </div>
              <div class="form-group"><label>პაციენტი</label><input type="text" id="patient-name" required></div>
              <div class="form-group"><label>ისტორია №</label><input type="text" id="history-number"></div>
              <div class="form-group"><label>ICD-10</label><input type="text" id="icd10"></div>
              <div class="form-group"><label>ექიმი</label><input type="text" id="doctor"></div>
            </div>
            <div class="form-group"><label>კომენტარი</label><textarea id="comment" rows="3"></textarea></div>
            <button type="submit" class="btn btn-primary">დამატება</button>
          </form>
          <button type="button" class="clear-all-btn" onclick="clearAllData()">ყველას წაშლა (ტესტი)</button>
        </div>

        <div class="card">
          <h2>აქტიური პაციენტები</h2>
          <input type="text" id="search" placeholder="ძებნა..." style="padding:18px; width:100%; max-width:650px; border-radius:14px; border:2px solid #ddd; font-size:17px; margin-bottom:25px;">
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

      <div id="archive-tab" class="tab-content" style="display: none;">
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

      <div id="statistics-tab" class="tab-content" style="display: none;">
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

    <div id="toast" class="toast"></div>
  </div>

  <!-- რედაქტირების მოდალი -->
  <div id="edit-modal" class="modal">
    <div class="modal-content">
      <h2>პაციენტის რედაქტირება</h2>
      <form id="edit-form">
        <div class="form-grid">
          <div class="form-group">
            <label>საწოლი</label>
            <select id="edit-bed">
              <option>1</option><option>2</option><option>3</option><option>4</option><option>5</option>
              <option>6</option><option>7</option><option>8</option><option>9</option><option>10</option>
              <option>ლოჯი</option><option>მცირე</option>
            </select>
          </div>
          <div class="form-group"><label>პაციენტი</label><input id="edit-name" required></div>
          <div class="form-group"><label>ისტორია</label><input id="edit-history"></div>
          <div class="form-group"><label>ICD-10</label><input id="edit-icd"></div>
          <div class="form-group"><label>ექიმი</label><input id="edit-doctor"></div>
        </div>
        <div class="form-group"><label>კომენტარი</label><textarea id="edit-comment" rows="3"></textarea></div>
        <div style="display:flex; gap:18px; justify-content:flex-end; margin-top:30px;">
          <button type="button" class="btn" style="background:#f0f0f0; color:#333;" onclick="document.getElementById('edit-modal').classList.remove('active')">გაუქმება</button>
          <button type="submit" class="btn btn-primary">შენახვა</button>
        </div>
      </form>
    </div>
  </div>

</body>
</html>
