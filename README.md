<html lang="ka">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>საწოლების მართვის სისტემა</title>
  <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;600;700&display=swap" rel="stylesheet">

  <!-- FAVICON – inpatient.png უნდა იდოს ამ ფაილთან იმავე საქაღალდეში -->
  <link rel="icon" type="image/png" href="inpatient.png">
  <link rel="shortcut icon" type="image/png" href="inpatient.png">
  <!-- სურვილის შემთხვევაში:
  <link rel="apple-touch-icon" href="inpatient.png">
  -->

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-app.js";
    import {
      getFirestore,
      collection,
      addDoc,
      updateDoc,
      deleteDoc,
      doc,
      onSnapshot,
      serverTimestamp,
      query,
      orderBy,
      getDocs
    } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-firestore.js";

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
    let editingId = null;

    // საწოლების თანმიმდევრული სორტირება
    function getBedOrder(bed) {
      if (!bed) return 999;
      const num = parseInt(bed);
      if (!isNaN(num) && num >= 1 && num <= 10) return num;
      if (bed === 'ლოჯი') return 11;
      if (bed === 'მცირე') return 12;
      return 999;
    }

    function showToast(msg, error = false) {
      const t = document.getElementById('toast');
      t.textContent = msg;
      t.className = 'toast active' + (error ? ' error' : '');
      setTimeout(() => t.classList.remove('active'), 4000);
    }

    function escapeHtml(text) {
      if (!text) return '-';
      const div = document.createElement('div');
      div.textContent = text;
      return div.innerHTML;
    }

    function formatDate(ts) {
      if (!ts || !ts.toDate) return '-';
      return ts.toDate().toLocaleString('ka-GE', {
        day: 'numeric',
        month: 'long',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      });
    }

    // დარბაზის არჩევა
    window.openRoom = function (room) {
      currentCollection = collection(
        db,
        room === 'observation' ? 'observation_room' : 'shock_room'
      );
      document.getElementById('welcome-screen').style.display = 'none';
      document.getElementById('app-container').style.display = 'block';
      document.querySelector('.header h1').textContent =
        room === 'observation' ? 'ობსერვაციის დარბაზი' : 'შოკის დარბაზი';
      loadPatients();
    };

    function loadPatients() {
      if (!currentCollection) return;
      const q = query(currentCollection, orderBy('timestamp', 'desc'));
      onSnapshot(q, snap => {
        patients = snap.docs.map(d => ({ id: d.id, ...d.data() }));
        renderActive();
        renderArchive();
        updateStats();
      });
    }

    // აქტიური პაციენტების რენდერი (ჩარიცხვა ამოღებულია + სახელის გაზრდა)
    function renderActive() {
      let list = patients.filter(p => !p.archived);
      const search = document.getElementById('search')?.value.toLowerCase() || '';
      if (search) {
        list = list.filter(p =>
          (p.patient_name || '').toLowerCase().includes(search) ||
          (p.history_number || '').includes(search) ||
          (p.bed || '').toLowerCase().includes(search) ||
          (p.doctor || '').toLowerCase().includes(search)
        );
      }

      list.sort((a, b) => getBedOrder(a.bed) - getBedOrder(b.bed));

      const tbody = document.getElementById('active-tbody');
      tbody.innerHTML =
        list.length === 0
          ? '<tr><td colspan="7" class="empty-state">პაციენტები არ არის</td></tr>'
          : list
              .map(
                p => `
          <tr>
            <td><strong style="color:#1d4ed8;">${escapeHtml(p.bed)}</strong></td>
            <td><strong style="font-size: 1.6em;">${escapeHtml(
              p.patient_name || '—'
            )}</strong></td>
            <td>${escapeHtml(p.history_number || '—')}</td>
            <td>${escapeHtml(p.icd10_code || '—')}</td>
            <td>${escapeHtml(p.doctor || '—')}</td>
            <td>${escapeHtml(p.comment || '—')}</td>
            <td class="action-buttons">
              <button class="btn btn-edit" onclick="openEditModal('${
                p.id
              }')">✏️</button>
              <button class="btn btn-archive" onclick="archivePatient('${
                p.id
              }')">📦</button>
              <button class="btn btn-delete" onclick="permanentlyDelete('${
                p.id
              }')">🗑️</button>
            </td>
          </tr>
        `
              )
              .join('');
    }

    // არქივის რენდერი (აქ მხოლოდ სახელის ზომა გავზარდე)
    function renderArchive() {
      let list = patients.filter(p => p.archived);
      list.sort((a, b) => getBedOrder(a.bed) - getBedOrder(b.bed));

      const tbody = document.getElementById('archive-tbody');
      tbody.innerHTML =
        list.length === 0
          ? '<tr><td colspan="9" class="empty-state">არქივი ცარიელია</td></tr>'
          : list
              .map(
                p => `
          <tr>
            <td>${escapeHtml(p.bed)}</td>
            <td><strong style="font-size: 1.6em;">${escapeHtml(
              p.patient_name
            )}</strong></td>
            <td>${escapeHtml(p.history_number)}</td>
            <td>${escapeHtml(p.icd10_code)}</td>
            <td>${escapeHtml(p.doctor)}</td>
            <td>${escapeHtml(p.comment)}</td>
            <td>${formatDate(p.timestamp)}</td>
            <td>${formatDate(p.archived_at)}</td>
            <td class="action-buttons">
              <button class="btn btn-restore" onclick="restorePatient('${
                p.id
              }')">🔄</button>
              <button class="btn btn-delete" onclick="permanentlyDelete('${
                p.id
              }')">🗑️</button>
            </td>
          </tr>
        `
              )
              .join('');
    }

    function updateStats() {
      const today = new Date().toDateString();
      const active = patients.filter(p => !p.archived).length;
      const archived = patients.filter(p => p.archived).length;
      const addedToday = patients.filter(
        p =>
          !p.archived &&
          p.timestamp?.toDate?.().toDateString() === today
      ).length;
      const dischargedToday = patients.filter(
        p =>
          p.archived &&
          p.archived_at?.toDate?.().toDateString() === today
      ).length;
      document.getElementById('stat-active').textContent = active;
      document.getElementById('stat-today-added').textContent = addedToday;
      document.getElementById('stat-today-deleted').textContent = dischargedToday;
      document.getElementById('stat-archived').textContent = archived;
    }

    document
      .getElementById('patient-form')
      .addEventListener('submit', async e => {
        e.preventDefault();
        try {
          await addDoc(currentCollection, {
            bed: document.getElementById('bed').value,
            patient_name:
              document.getElementById('patient-name').value.trim() || null,
            history_number:
              document.getElementById('history-number').value.trim() || null,
            icd10_code: document.getElementById('icd10').value.trim() || null,
            doctor: document.getElementById('doctor').value.trim() || null,
            comment: document.getElementById('comment').value.trim() || null,
            archived: false,
            timestamp: serverTimestamp()
          });
          e.target.reset();
          showToast('პაციენტი დამატებულია');
        } catch (err) {
          showToast('შეცდომა: ' + err.message, true);
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

    window.closeEditModal = () =>
      document.getElementById('edit-modal').classList.remove('active');

    document
      .getElementById('edit-form')
      .addEventListener('submit', async e => {
        e.preventDefault();
        try {
          await updateDoc(doc(db, currentCollection.path, editingId), {
            bed: document.getElementById('edit-bed').value,
            patient_name:
              document.getElementById('edit-name').value.trim() || null,
            history_number:
              document
                .getElementById('edit-history')
                .value.trim() || null,
            icd10_code:
              document.getElementById('edit-icd').value.trim() || null,
            doctor: document.getElementById('edit-doctor').value.trim() || null,
            comment:
              document.getElementById('edit-comment').value.trim() || null
          });
          closeEditModal();
          showToast('ცვლილებები შენახულია');
        } catch (err) {
          showToast('შეცდომა: ' + err.message, true);
        }
      });

    window.archivePatient = id => {
      if (confirm('არქივში გადატანა?')) {
        updateDoc(doc(db, currentCollection.path, id), {
          archived: true,
          archived_at: serverTimestamp()
        }).then(() => showToast('არქივში გადავიდა'));
      }
    };

    window.restorePatient = id => {
      if (confirm('აღდგენა აქტიურ პაციენტებში?')) {
        updateDoc(doc(db, currentCollection.path, id), {
          archived: false,
          archived_at: null
        }).then(() => showToast('პაციენტი აღდგენილია'));
      }
    };

    window.permanentlyDelete = id => {
      if (
        confirm('სამუდამოდ წაშლა? (შეუქცევადია!)') &&
        confirm('დარწმუნებული ხართ?')
      ) {
        deleteDoc(doc(db, currentCollection.path, id)).then(() =>
          showToast('პაციენტი წაიშალა')
        );
      }
    };

    window.clearAllData = async () => {
      if (
        !confirm('ყველა პაციენტი წაიშლება სამუდამოდ!') ||
        !confirm('დარწმუნებული ხართ?')
      )
        return;
      const snap = await getDocs(currentCollection);
      await Promise.all(snap.docs.map(d => deleteDoc(d.ref)));
      showToast('დარბაზი გასუფთავდა');
    };

    window.switchTab = tab => {
      document
        .querySelectorAll('.tab-btn')
        .forEach(b => b.classList.remove('active'));
      document
        .querySelectorAll('.tab-content')
        .forEach(c => c.classList.remove('active'));
      document
        .querySelector(`.tab-btn[onclick="switchTab('${tab}')"]`)
        .classList.add('active');
      document.getElementById(tab + '-tab').classList.add('active');
    };

    document.getElementById('search')?.addEventListener('input', renderActive);
  </script>

  <style>
    :root {
      --primary: #2563eb;
      --primary-light: #3b82f6;
      --gray-100: #f8fafc;
      --gray-200: #e2e8f0;
      --gray-600: #475569;
      --success: #10b981;
      --danger: #ef4444;
      --warning: #f59e0b;
    }
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    body {
      font-family: 'Roboto', sans-serif;
      background: linear-gradient(135deg, #f0f7ff 0%, #e0f2fe 100%);
      color: #1e293b;
      min-height: 100vh;
    }
    .welcome {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      text-align: center;
      padding: 2rem;
      background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%);
      color: white;
    }
    .welcome h1 {
      font-size: 3.8rem;
      margin-bottom: 1rem;
    }
    .room-choice {
      background: white;
      color: #1d4ed8;
      padding: 2.5rem 5rem;
      border-radius: 20px;
      font-size: 2rem;
      font-weight: 600;
      cursor: pointer;
      margin: 1.5rem;
      min-width: 420px;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
      transition: all 0.3s;
    }
    .room-choice:hover {
      transform: translateY(-10px);
      box-shadow: 0 25px 50px rgba(59, 130, 246, 0.3);
    }
    .back-to-rooms {
      position: fixed;
      top: 1.5rem;
      left: 1.5rem;
      background: white;
      color: #2563eb;
      border: none;
      padding: 0.8rem 1.8rem;
      border-radius: 50px;
      font-weight: 600;
      cursor: pointer;
      z-index: 999;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
    }
    #app-container {
      display: none;
      padding-top: 6rem;
    }
    .container {
      max-width: 1500px;
      margin: 0 auto;
      padding: 0 1.5rem;
    }
    .header {
      background: white;
      padding: 2.5rem;
      border-radius: 20px;
      text-align: center;
      margin-bottom: 2rem;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
    }
    .header h1 {
      font-size: 3rem;
      color: #1d4ed8;
      margin-bottom: 0.5rem;
    }
    .tabs {
      display: flex;
      gap: 1rem;
      margin-bottom: 2rem;
      background: white;
      padding: 1rem;
      border-radius: 16px;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.08);
      justify-content: center;
      flex-wrap: wrap;
    }
    .tab-btn {
      padding: 0.9rem 2rem;
      background: transparent;
      border: none;
      border-radius: 12px;
      color: #475569;
      font-weight: 600;
      cursor: pointer;
      transition: all 0.3s;
    }
    .tab-btn:hover {
      background: #f1f5f9;
    }
    .tab-btn.active {
      background: var(--primary);
      color: white;
    }
    .tab-content {
      display: none;
    }
    .tab-content.active {
      display: block;
    }
    .card {
      background: white;
      border-radius: 20px;
      padding: 2rem;
      margin-bottom: 2rem;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
    }
    .card h2 {
      font-size: 1.8rem;
      margin-bottom: 1.5rem;
      color: #1e293b;
      padding-bottom: 0.8rem;
      border-bottom: 3px solid var(--primary-light);
      display: inline-block;
    }
    .form-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
      gap: 1.5rem;
      margin-bottom: 1.5rem;
    }
    .form-group label {
      display: block;
      margin-bottom: 0.6rem;
      font-weight: 600;
      color: #374151;
    }
    .form-group input,
    .form-group select,
    .form-group textarea {
      width: 100%;
      padding: 1rem;
      border-radius: 12px;
      border: 2px solid #e2e8f0;
      font-size: 1rem;
      transition: all 0.3s;
    }
    .form-group input:focus,
    .form-group select:focus,
    .form-group textarea:focus {
      outline: none;
      border-color: var(--primary-light);
      box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.15);
    }
    .btn {
      padding: 0.5rem;
      border: none;
      border-radius: 12px;
      cursor: pointer;
      font-weight: 600;
      font-size: 1rem;
      transition: all 0.3s;
      min-width: 40px;
    }
    .btn-primary {
      background: var(--primary);
      color: white;
    }
    .btn-primary:hover {
      background: #1d4ed8;
      transform: translateY(-2px);
    }
    .btn-edit {
      background: #6366f1;
      color: white;
    }
    .btn-archive {
      background: var(--warning);
      color: white;
    }
    .btn-restore {
      background: var(--success);
      color: white;
    }
    .btn-delete {
      background: var(--danger);
      color: white;
    }
    .search-filter input {
      width: 100%;
      max-width: 500px;
      padding: 1rem 1.2rem;
      border-radius: 14px;
      border: 2px solid #e2e8f0;
      font-size: 1.1rem;
      margin-bottom: 1rem;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      background: white;
      border-radius: 16px;
      overflow: hidden;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.08);
    }
    th {
      background: var(--primary);
      color: white;
      padding: 1.3rem 1rem;
      text-align: left;
      font-weight: 600;
      cursor: default;
    }
    td {
      padding: 1.2rem 1rem;
    }

    /* მხოლოდ გამყოფი ხაზები მუქი ლურჯი აქტიურ ტაბში */
    #active-tab td,
    #active-tab th {
      border-bottom: 2px solid #1d4ed8 !important;
    }
    #active-tab tr:hover {
      background: #eff6ff;
    }

    tr:hover {
      background: #f8fafc;
    }
    .action-buttons {
      display: flex;
      gap: 0.8rem;
      flex-wrap: wrap;
    }
    .stats-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
      gap: 1.5rem;
    }
    .stat-card {
      background: linear-gradient(135deg, var(--primary-light) 0%, var(--primary) 100%);
      color: white;
      padding: 2rem;
      border-radius: 20px;
      text-align: center;
      box-shadow: 0 15px 35px rgba(59, 130, 246, 0.25);
    }
    .stat-card .number {
      font-size: 3.8rem;
      font-weight: 700;
      margin: 0.8rem 0;
    }
    .toast {
      position: fixed;
      top: 2rem;
      right: 2rem;
      padding: 1.2rem 2rem;
      border-radius: 14px;
      color: white;
      font-weight: 600;
      z-index: 2000;
      opacity: 0;
      transform: translateX(100%);
      transition: all 0.4s ease;
    }
    .toast.active {
      opacity: 1;
      transform: translateX(0);
      background: var(--success);
    }
    .toast.error {
      background: var(--danger);
    }
    .clear-all-btn {
      background: var(--danger);
      color: white;
      padding: 1.2rem 2.5rem;
      border: none;
      border-radius: 14px;
      font-size: 1.1rem;
      font-weight: 600;
      cursor: pointer;
      margin-top: 2rem;
    }
    .clear-all-btn:hover {
      background: #b91c1c;
    }
    .empty-state {
      text-align: center;
      padding: 5rem 1rem;
      color: #94a3b8;
      font-size: 1.3rem;
    }

    .modal {
      display: none;
      position: fixed;
      inset: 0;
      background: rgba(0, 0, 0, 0.6);
      z-index: 1000;
      align-items: center;
      justify-content: center;
      padding: 1rem;
    }
    .modal.active {
      display: flex;
    }
    .modal-content {
      background: white;
      border-radius: 20px;
      width: 100%;
      max-width: 620px;
      max-height: 90vh;
      display: flex;
      flex-direction: column;
      overflow: hidden;
      box-shadow: 0 25px 60px rgba(0, 0, 0, 0.3);
    }
    .modal-header {
      padding: 1.5rem 2rem;
      border-bottom: 1px solid #e2e8f0;
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-shrink: 0;
    }
    .modal-header h2 {
      margin: 0;
      font-size: 1.7rem;
      color: #1d4ed8;
    }
    .modal-close {
      background: none;
      border: none;
      font-size: 2.2rem;
      color: #94a3b8;
      cursor: pointer;
    }
    .modal-body {
      padding: 1.5rem 2rem;
      overflow-y: auto;
      flex: 1;
    }
    .modal-footer {
      padding: 1.5rem 2rem;
      background: #f8fafc;
      border-top: 1px solid #e2e8f0;
      display: flex;
      gap: 1rem;
      justify-content: flex-end;
      flex-shrink: 0;
    }
    @media (max-width: 768px) {
      .form-grid {
        grid-template-columns: 1fr;
      }
      .action-buttons {
        flex-direction: column;
      }
      .room-choice {
        min-width: 300px;
        padding: 2rem 3rem;
        font-size: 1.8rem;
      }
    }
  </style>
</head>
<body>
  <div id="welcome-screen" class="welcome">
    <h1>საწოლების მართვა</h1>
    <p>აირჩიეთ დარბაზი სამუშაოდ</p>
    <div class="room-choice" onclick="openRoom('observation')">ობსერვაციის დარბაზი</div>
    <div class="room-choice" onclick="openRoom('shock')">შოკის დარბაზი</div>
  </div>

  <div id="app-container">
    <button class="back-to-rooms" onclick="location.reload()">დარბაზის შეცვლა</button>
    <div class="container">
      <div class="header">
        <h1>დარბაზი</h1>
        <p>სტაციონარული პაციენტების მართვა</p>
      </div>

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
              <div class="form-group">
                <label for="bed">საწოლი *</label>
                <select id="bed" required>
                  <option value="">აირჩიეთ საწოლი</option>
                  <option value="1">1</option>
                  <option value="2">2</option>
                  <option value="3">3</option>
                  <option value="4">4</option>
                  <option value="5">5</option>
                  <option value="6">6</option>
                  <option value="7">7</option>
                  <option value="8">8</option>
                  <option value="9">9</option>
                  <option value="10">10</option>
                  <option value="ლოჯი">ლოჯი</option>
                  <option value="მცირე">მცირე</option>
                </select>
              </div>
              <div class="form-group">
                <label for="patient-name">პაციენტის სახელი და გვარი</label>
                <input type="text" id="patient-name">
              </div>
              <div class="form-group">
                <label for="history-number">ისტორიის ნომერი</label>
                <input type="text" id="history-number">
              </div>
              <div class="form-group">
                <label for="icd10">ICD-10 კოდი</label>
                <input type="text" id="icd10">
              </div>
              <div class="form-group">
                <label for="doctor">ექიმი</label>
                <input type="text" id="doctor">
              </div>
            </div>
            <div class="form-group">
              <label for="comment">კომენტარი</label>
              <textarea id="comment" rows="3"></textarea>
            </div>
            <button type="submit" class="btn btn-primary">დამატება</button>
          </form>
          <button type="button" class="clear-all-btn" onclick="clearAllData()">ყველა პაციენტის წაშლა</button>
        </div>

        <div class="card">
          <h2>აქტიური პაციენტები (დალაგებულია საწოლით)</h2>
          <div class="search-filter">
            <input type="text" id="search" placeholder="ძებნა...">
          </div>
          <table>
            <thead>
              <tr>
                <th>საწოლი</th>
                <th>პაციენტი</th>
                <th>ისტორია</th>
                <th>ICD-10</th>
                <th>ექიმი</th>
                <th>კომენტარი</th>
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
              <tr>
                <th>საწოლი</th>
                <th>პაციენტი</th>
                <th>ისტორია</th>
                <th>ICD-10</th>
                <th>ექიმი</th>
                <th>კომენტარი</th>
                <th>ჩარიცხვა</th>
                <th>არქივში</th>
                <th>მოქმედება</th>
              </tr>
            </thead>
            <tbody id="archive-tbody"></tbody>
          </table>
        </div>
      </div>

      <div id="statistics-tab" class="tab-content">
        <div class="card">
          <h2>დღიური სტატისტიკა</h2>
          <div class="stats-grid">
            <div class="stat-card">
              <h3>აქტიური</h3>
              <div class="number" id="stat-active">0</div>
            </div>
            <div class="stat-card">
              <h3>დღეს დამატებული</h3>
              <div class="number" id="stat-today-added">0</div>
            </div>
            <div class="stat-card">
              <h3>დღეს გაწერილი</h3>
              <div class="number" id="stat-today-deleted">0</div>
            </div>
            <div class="stat-card">
              <h3>სულ არქივში</h3>
              <div class="number" id="stat-archived">0</div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div id="edit-modal" class="modal">
      <div class="modal-content">
        <div class="modal-header">
          <h2>პაციენტის რედაქტირება</h2>
          <button class="modal-close" onclick="closeEditModal()">×</button>
        </div>
        <div class="modal-body">
          <form id="edit-form">
            <div class="form-grid">
              <div class="form-group">
                <label>საწოლი</label>
                <select id="edit-bed">
                  <option value="1">1</option>
                  <option value="2">2</option>
                  <option value="3">3</option>
                  <option value="4">4</option>
                  <option value="5">5</option>
                  <option value="6">6</option>
                  <option value="7">7</option>
                  <option value="8">8</option>
                  <option value="9">9</option>
                  <option value="10">10</option>
                  <option value="ლოჯი">ლოჯი</option>
                  <option value="მცირე">მცირე</option>
                </select>
              </div>
              <div class="form-group">
                <label>პაციენტი</label>
                <input id="edit-name">
              </div>
              <div class="form-group">
                <label>ისტორია</label>
                <input id="edit-history">
              </div>
              <div class="form-group">
                <label>ICD-10</label>
                <input id="edit-icd">
              </div>
              <div class="form-group">
                <label>ექიმი</label>
                <input id="edit-doctor">
              </div>
            </div>
            <div class="form-group">
              <label>კომენტარი</label>
              <textarea id="edit-comment" rows="3"></textarea>
            </div>
          </form>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn"
            style="background:#94a3b8; color:white;"
            onclick="closeEditModal()"
          >
            გაუქმება
          </button>
          <button type="submit" form="edit-form" class="btn btn-primary">
            შენახვა
          </button>
        </div>
      </div>
    </div>

    <div id="toast" class="toast"></div>
  </div>
</body>
</html>
