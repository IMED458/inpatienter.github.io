<!DOCTYPE html>
<html lang="ka">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>საწოლების მართვის სისტემა</title>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-app.js";
    import { getFirestore, collection, addDoc, updateDoc, deleteDoc, doc, onSnapshot, serverTimestamp, query, orderBy, getDocs } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-firestore.js";
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

    // ანონიმური ავტორიზაცია
    signInAnonymously(auth).catch(err => console.error("Auth failed:", err));

    let currentCollection = null;
    let patients = [];
    let currentSort = { column: 'timestamp', dir: 'desc' };
    let editingId = null;

    function showToast(msg, error = false) {
      const toast = document.getElementById('toast');
      toast.textContent = msg;
      toast.className = 'toast active' + (error ? ' error' : '');
      setTimeout(() => toast.classList.remove('active'), 4000);
    }

    function escapeHtml(text) {
      if (!text) return '-';
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

    // მთავარი ფუნქცია — ახლა სრულად მუშაობს!
    window.openRoom = function(room) {
      const collName = room === 'observation' ? 'observation_room' : 'shock_room';
      currentCollection = collection(db, collName);

      document.getElementById('welcome-screen').style.display = 'none';
      document.getElementById('app-container').style.display = 'block';
      document.querySelector('.header h1').textContent = room === 'observation' 
        ? 'ობსერვაციის დარბაზი' : 'შოკის დარბაზი';

      // დავიწყოთ მონაცემების ჩატვირთვა
      loadPatients();
    };

    function loadPatients() {
      if (!currentCollection) {
        showToast('დარბაზი არ არის არჩეული', true);
        return;
      }

      const q = query(currentCollection, orderBy("timestamp", "desc"));
      onSnapshot(q, snapshot => {
        patients = snapshot.docs.map(d => ({ id: d.id, ...d.data() }));
        renderTables();
        updateStats();
      }, err => {
        console.error("Firestore error:", err);
        showToast('მონაცემები ვერ ჩაიტვირთა', true);
      });
    }

    // დანარჩენი ფუნქციები (რენდერი, სორტირება, და ა.შ.) — უცვლელი და სრულად მუშა
    function renderActive() {
      let list = patients.filter(p => !p.archived);
      const search = document.getElementById('search')?.value.toLowerCase() || '';
      if (search) {
        list = list.filter(p =>
          (p.patient_name || '').toLowerCase().includes(search) ||
          (p.history_number || '').includes(search)
        );
      }

      list.sort((a, b) => {
        let av = a[currentSort.column] ?? '';
        let bv = b[currentSort.column] ?? '';
        if (currentSort.column === 'timestamp') {
          av = a.timestamp?.seconds || 0;
          bv = b.timestamp?.seconds || 0;
        }
        const result = av < bv ? -1 : av > bv ? 1 : 0;
        return result * (currentSort.dir === 'asc' ? 1 : -1);
      });

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
            <td class="action-buttons">
              <button class="btn btn-secondary" onclick="openEditModal('${p.id}')">რედაქტირება</button>
              <button class="btn btn-danger" onclick="archivePatient('${p.id}')">არქივი</button>
              <button class="btn btn-delete" onclick="permanentlyDelete('${p.id}')">წაშლა</button>
            </td>
          </tr>
        `).join('');
    }

    function renderArchive() {
      const list = patients.filter(p => p.archived);
      document.getElementById('archive-tbody').innerHTML = list.length === 0
        ? '<tr><td colspan="9" class="empty-state">არქივი ცარიელია</td></tr>'
        : list.map(p => `
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
              <button class="btn btn-success" onclick="restorePatient('${p.id}')">დაბრუნება</button>
              <button class="btn btn-delete" onclick="permanentlyDelete('${p.id}')">წაშლა</button>
            </td>
          </tr>
        `).join('');
    }

    function renderTables() { renderActive(); renderArchive(); }

    function updateStats() {
      const today = new Date().toDateString();
      const active = patients.filter(p => !p.archived).length;
      const archived = patients.filter(p => p.archived).length;
      const addedToday = patients.filter(p => !p.archived && p.timestamp?.toDate?.().toDateString() === today).length;
      const archivedToday = patients.filter(p => p.archived && p.archived_at?.toDate?.().toDateString() === today).length;

      document.getElementById('stat-active').textContent = active;
      document.getElementById('stat-today-added').textContent = addedToday;
      document.getElementById('stat-today-deleted').textContent = archivedToday;
      document.getElementById('stat-archived').textContent = archived;
    }

    // ფორმა
    document.getElementById('patient-form')?.addEventListener('submit', async e => {
      e.preventDefault();
      try {
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
      } catch (err) {
        showToast('დამატება ვერ მოხერხდა', true);
      }
    });

    window.openEditModal = id => {
      const p = patients.find(x => x.id === id);
      if (!p) return;
      editingId = id;
      document.getElementById('edit-bed').value = p.bed || '';
      document.getElementById('edit-name').value = p.patient_name || '';
      document.getElementById('edit-history').value = p.history_number || '';
      document.getElementById('edit-icd').value = p.icd10_code || '';
      document.getElementById('edit-doctor').value = p.doctor || '';
      document.getElementById('edit-comment').value = p.comment || '';
      document.getElementById('edit-modal').classList.add('active');
    };

    window.closeEditModal = () => {
      document.getElementById('edit-modal').classList.remove('active');
      editingId = null;
    };

    document.getElementById('edit-form')?.addEventListener('submit', async e => {
      e.preventDefault();
      if (!editingId) return;
      try {
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
      } catch (err) {
        showToast('შენახვა ვერ მოხერხდა', true);
      }
    });

    window.archivePatient = async id => {
      if (!confirm('არქივში გადატანა?')) return;
      await updateDoc(doc(db, currentCollection.path, id), { archived: true, archived_at: serverTimestamp() });
      showToast('არქივში გადავიდა');
    };

    window.restorePatient = async id => {
      if (!confirm('აღდგენა?')) return;
      await updateDoc(doc(db, currentCollection.path, id), { archived: false, archived_at: null });
      showToast('აღდგენილია');
    };

    window.permanentlyDelete = async id => {
      if (!confirm('სამუდამოდ წაშლა?')) return;
      if (!confirm('დარწმუნებული ხართ?')) return;
      await deleteDoc(doc(db, currentCollection.path, id));
      showToast('წაიშალა');
    };

    window.clearAllData = async () => {
      if (!confirm("ყველა პაციენტი წაიშლება!")) return;
      if (!confirm("დარწმუნებული ხართ?")) return;
      const pass = prompt("პაროლი: გასუფთავება2025");
      if (pass !== "გასუფთავება2025") return showToast("პაროლი არასწორია");
      const snapshot = await getDocs(currentCollection);
      await Promise.all(snapshot.docs.map(d => deleteDoc(d.ref)));
      showToast("დარბაზი გასუფთავდა");
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

  <!-- სრული CSS და HTML — ისევ შენი დიზაინი -->
  <style>
    body{margin:0;padding:0;font-family:'Segoe UI',sans-serif;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);min-height:100vh;color:#333;}
    *{box-sizing:border-box;}
    .welcome{display:flex;flex-direction:column;align-items:center;justify-content:center;min-height:100vh;color:white;text-align:center;}
    .welcome h1{font-size:3.5em;margin-bottom:20px;}
    .room-choice{background:rgba(255,255,255,0.15);padding:30px 60px;border-radius:20px;cursor:pointer;margin:20px;font-size:1.8em;font-weight:bold;transition:.3s;}
    .room-choice:hover{background:rgba(255,255,255,0.25);transform:scale(1.05);}
    .back-to-rooms{position:fixed;top:20px;left:20px;background:rgba(255,255,255,0.2);color:white;border:none;padding:12px 20px;border-radius:50px;cursor:pointer;z-index:999;}
    #app-container{display:none;}
    .container{max-width:1400px;margin:0 auto;padding:20px;}
    .header{background:white;padding:30px;border-radius:12px;box-shadow:0 4px 12px rgba(0,0,0,0.1);margin-bottom:20px;text-align:center;}
    .header h1{margin:0;color:#667eea;font-size:2.5em;}
    .tabs{display:flex;gap:10px;margin-bottom:20px;flex-wrap:wrap;}
    .tab-btn{padding:12px 24px;background:white;border:none;border-radius:8px;cursor:pointer;font-weight:600;color:#667eea;}
    .tab-btn.active{background:#667eea;color:white;}
    .tab-content{display:none;}
    .tab-content.active{display:block;}
    .card{background:white;padding:30px;border-radius:12px;box-shadow:0 4px 12px rgba(0,0,0,0.1);margin-bottom:20px;}
    .form-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:20px;}
    .form-group label{margin-bottom:8px;font-weight:600;}
    .form-group input,.form-group select,.form-group textarea{padding:12px;border:2px solid #e0e0e0;border-radius:8px;}
    .form-group input:focus,.form-group select:focus{outline:none;border-color:#667eea;}
    .btn{padding:12px 24px;border:none;border-radius:8px;cursor:pointer;font-weight:600;}
    .btn-primary{background:#667eea;color:white;}
    .btn-primary:hover{background:#5568d3;}
    .btn-danger{background:#e74c3c;color:white;}
    .btn-success{background:#27ae60;color:white;}
    .btn-delete{background:#c0392b;color:white;font-size:13px;padding:6px 10px;}
    table{width:100%;border-collapse:collapse;}
    th,td{padding:15px;border-bottom:1px solid #eee;text-align:left;}
    th{background:#f8f9fa;cursor:pointer;}
    tr:hover{background:#f9faff;}
    .action-buttons{display:flex;gap:8px;flex-wrap:wrap;}
    .stats-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:20px;}
    .stat-card{background:linear-gradient(135deg,#667eea,#764ba2);color:white;padding:25px;border-radius:12px;text-align:center;}
    .stat-card .number{font-size:3em;font-weight:bold;}
    .modal{display:none;position:fixed;inset:0;background:rgba(0,0,0,0.5);z-index:1000;justify-content:center;align-items:center;}
    .modal.active{display:flex;}
    .modal-content{background:white;padding:30px;border-radius:12px;width:90%;max-width:600px;}
    .toast{position:fixed;top:20px;right:20px;background:#27ae60;color:white;padding:15px 25px;border-radius:8px;z-index:2000;display:none;}
    .toast.active{display:block;animation:slide .3s;}
    .toast.error{background:#e74c3c;}
    @keyframes slide{from{transform:translateX(100%);}to{transform:none;}}
    .clear-all-btn{background:#e74c3c;color:white;padding:15px 30px;border:none;border-radius:8px;margin:20px auto;display:block;font-weight:bold;cursor:pointer;}
  </style>
</head>
<body>

  <div id="welcome-screen" class="welcome">
    <h1>აირჩიეთ დარბაზი</h1>
    <div class="room-choice" onclick="openRoom('observation')">ობსერვაციის დარბაზი</div>
    <div class="room-choice" onclick="openRoom('shock')">შოკის დარბაზი</div>
  </div>

  <div id="app-container">
    <button class="back-to-rooms" onclick="location.reload()">დარბაზის შეცვლა</button>
    <div class="container">
      <div class="header"><h1>დარბაზი</h1><p>სტაციონარი / Inpatient</p></div>
      <div class="tabs">
        <button class="tab-btn active" onclick="switchTab('active')">აქტიური</button>
        <button class="tab-btn" onclick="switchTab('archive')">არქივი</button>
        <button class="tab-btn" onclick="switchTab('statistics')">სტატისტიკა</button>
      </div>

      <div id="active-tab" class="tab-content active">
        <div class="card">
          <h2>ახალი პაციენტი</h2>
          <form id="patient-form">
            <div class="form-grid">
              <div class="form-group"><label>საწოლი</label><select id="bed" required><option value="">აირჩიეთ</option><option>1</option><option>2</option><option>3</option><option>4</option><option>5</option><option>6</option><option>7</option><option>8</option><option>9</option><option>10</option><option>ლოჯი</option><option>მცირე</option></select></div>
              <div class="form-group"><label>პაციენტი</label><input type="text" id="patient-name" required></div>
              <div class="form-group"><label>ისტორია</label><input type="text" id="history-number"></div>
              <div class="form-group"><label>ICD-10</label><input type="text" id="icd10"></div>
              <div class="form-group"><label>ექიმი</label><input type="text" id="doctor"></div>
            </div>
            <div class="form-group"><label>კომენტარი</label><textarea id="comment"></textarea></div>
            <button type="submit" class="btn btn-primary">დამატება</button>
          </form>
          <button type="button" class="clear-all-btn" onclick="clearAllData()">მთლიანი გასუფთავება</button>
        </div>

        <div class="card">
          <h2>აქტიური პაციენტები</h2>
          <input type="text" id="search" placeholder="ძებნა..." style="padding:12px;width:100%;max-width:500px;border-radius:8px;border:2px solid #ddd;margin-bottom:15px;">
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

      <div id="archive-tab" class="tab-content">
        <div class="card">
          <h2>არქივი</h2>
          <table>
            <thead>
              <tr><th>საწოლი</th><th>პაციენტი</th><th>ისტორია</th><th>ICD-10</th><th>ექიმი</th><th>კომენტარი</th><th>ჩარიცხვა</th><th>არქივში</th><th>მოქმედება</th></tr>
            </thead>
            <tbody id="archive-tbody"></tbody>
          </table>
        </div>
      </div>

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

    <div id="toast" class="toast"></div>
  </div>

  <!-- მოდალი -->
  <div id="edit-modal" class="modal">
    <div class="modal-content">
      <h2>რედაქტირება</h2>
      <form id="edit-form">
        <div class="form-grid">
          <div class="form-group"><label>საწოლი</label><select id="edit-bed"><option>1</option><option>2</option><option>3</option><option>4</option><option>5</option><option>6</option><option>7</option><option>8</option><option>9</option><option>10</option><option>ლოჯი</option><option>მცირე</option></select></div>
          <div class="form-group"><label>პაციენტი</label><input id="edit-name" required></div>
          <div class="form-group"><label>ისტორია</label><input id="edit-history"></div>
          <div class="form-group"><label>ICD-10</label><input id="edit-icd"></div>
          <div class="form-group"><label>ექიმი</label><input id="edit-doctor"></div>
        </div>
        <div class="form-group"><label>კომენტარი</label><textarea id="edit-comment"></textarea></div>
        <div style="text-align:right;margin-top:20px;">
          <button type="button" class="btn" style="background:#f0f0f0;margin-right:10px;" onclick="closeEditModal()">გაუქმება</button>
          <button type="submit" class="btn btn-primary">შენახვა</button>
        </div>
      </form>
    </div>
  </div>

</body>
</html>
