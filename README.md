<html lang="ka">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>საწოლების მართვის სისტემა</title>
  <style>
    body{box-sizing:border-box;margin:0;padding:0;font-family:'Segoe UI',Tahoma,Geneva,Verdana,sans-serif;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);min-height:100vh;}
    *{box-sizing:border-box;}
    .welcome{display:flex;flex-direction:column;align-items:center;justify-content:center;min-height:100vh;color:white;text-align:center;}
    .welcome h1{font-size:3.5em;margin-bottom:20px;}
    .welcome p{font-size:1.4em;margin-bottom:50px;opacity:0.9;}
    .room-choice{background:rgba(255,255,255,0.15);padding:30px 60px;border-radius:20px;cursor:pointer;margin:20px;font-size:1.8em;font-weight:bold;transition:all .3s;box-shadow:0 10px 30px rgba(0,0,0,0.3);}
    .room-choice:hover{background:rgba(255,255,255,0.25);transform:scale(1.05);}
    .back-to-rooms{position:fixed;top:20px;left:20px;background:rgba(255,255,255,0.2);color:white;border:none;padding:12px 20px;border-radius:50px;font-size:16px;cursor:pointer;z-index:999;}
    .back-to-rooms:hover{background:rgba(255,255,255,0.4);}
    #app-container{display:none;}
  </style>
</head>
<body>

<!-- არჩევანი -->
<div id="welcome-screen" class="welcome">
  <h1>აირჩიეთ დარბაზი</h1>
  <p>სად გსურთ მუშაობა?</p>
  <div class="room-choice" onclick="openRoom('observation')">ობსერვაციის დარბაზი</div>
  <div class="room-choice" onclick="openRoom('shock')">შოკის დარბაზი</div>
</div>

<!-- მთავარი აპლიკაცია -->
<div id="app-container">
  <button class="back-to-rooms" onclick="location.reload()">დარბაზის შეცვლა</button>
  <div class="container" style="max-width:1400px;margin:0 auto;padding:20px;" id="main-app">
    <!-- აქ ჩაიტვირთება შენი ძველი ინტერფეისი -->
  </div>
</div>

<script>
// ორი განცალკევებული მონაცემთა ბაზა
const STORAGE_KEYS = {
  observation: 'inpatient_observation_data',
  shock: 'inpatient_shock_data'
};

function openRoom(room) {
  const key = STORAGE_KEYS[room];
  document.getElementById('welcome-screen').style.display = 'none';
  document.getElementById('app-container').style.display = 'block';

  // ჩავსვათ შენი ზუსტი ძველი კოდი (HTML + CSS + JS)
  document.getElementById('main-app').innerHTML = getFullOriginalApp();

  // გავუშვათ სკრიპტი შესაბამისი localStorage-ით
  initOriginalApp(key, room === 'observation' ? 'ობსერვაციის დარბაზი' : 'შოკის დარბაზი');
}

function getFullOriginalApp() {
  return `
<div class="header">
  <h1>საწოლების მართვის სისტემა</h1>
  <p>სტაციონარი / Inpatient</p>
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
        <div class="form-group"><label for="bed">საწოლი</label>
          <select id="bed" required>
            <option value="">აირჩიეთ საწოლი</option>
            <option value="1">1</option><option value="2">2</option><option value="3">3</option>
            <option value="4">4</option><option value="5">5</option><option value="6">6</option>
            <option value="7">7</option><option value="8">8</option><option value="9">9</option><option value="10">10</option>
            <option value="ლოჯი">ლოჯი</option><option value="მცირე">მცირე</option>
          </select>
        </div>
        <div class="form-group"><label for="patient-name">პაციენტის სახელი და გვარი</label><input type="text" id="patient-name" required></div>
        <div class="form-group"><label for="history-number">ისტორიის ნომერი</label><input type="text" id="history-number"></div>
        <div class="form-group"><label for="icd10">ICD-10 კოდი</label><input type="text" id="icd10"></div>
        <div class="form-group"><label for="doctor">ექიმი</label><input type="text" id="doctor"></div>
      </div>
      <div class="form-group"><label for="comment">კომენტარი</label><textarea id="comment"></textarea></div>
      <button type="submit" class="btn btn-primary" id="submit-btn"><span id="submit-text">დამატება</span></button>
    </form>
  </div>

  <div class="card">
    <h2>აქტიური პაციენტები</h2>
    <div class="search-filter"><input type="text" id="search" placeholder="ძებნა (სახელი, ისტორია)"></div>
    <div class="table-container">
      <table>
        <thead>
          <tr>
            <th onclick="sortTable('bed')">საწოლი</th>
            <th onclick="sortTable('patient_name')">პაციენტი</th>
            <th onclick="sortTable('history_number')">ისტორია</th>
            <th onclick="sortTable('icd10_code')">ICD-10</th>
            <th onclick="sortTable('doctor')">ექიმი</th>
            <th>კომენტარი</th>
            <th onclick="sortTable('admission_date')">ჩარიცხვა</th>
            <th>მოქმედება</th>
          </tr>
        </thead>
        <tbody id="active-tbody"></tbody>
      </table>
    </div>
  </div>
</div>

<div id="archive-tab" class="tab-content">
  <div class="card">
    <h2>არქივი</h2>
    <div class="table-container">
      <table>
        <thead>
          <tr>
            <th>საწოლი</th><th>პაციენტი</th><th>ისტორია</th><th>ICD-10</th><th>ექიმი</th>
            <th>კომენტარი</th><th>ჩარიცხვა</th><th>არქივში გადატანა</th><th>მოქმედება</th>
          </tr>
        </thead>
        <tbody id="archive-tbody"></tbody>
      </table>
    </div>
  </div>
</div>

<div id="statistics-tab" class="tab-content">
  <div class="card">
    <h2>სტატისტიკა</h2>
    <div class="stats-grid">
      <div class="stat-card"><h3>აქტიური პაციენტები</h3><div class="number" id="stat-active">0</div></div>
      <div class="stat-card"><h3>დღეს დამატებული</h3><div class="number" id="stat-today-added">0</div></div>
      <div class="stat-card"><h3>დღეს წაშლილი</h3><div class="number" id="stat-today-deleted">0</div></div>
      <div class="stat-card"><h3>სულ არქივში</h3><div class="number" id="stat-archived">0</div></div>
    </div>
  </div>
</div>

<div id="edit-modal" class="modal">
  <div class="modal-content">
    <div class="modal-header"><h2>პაციენტის რედაქტირება</h2><button class="close-btn" onclick="closeEditModal()">X</button></div>
    <form id="edit-form">
      <div class="form-grid">
        <div class="form-group"><label>საწოლი</label><select id="edit-bed">
          <option value="1">1</option><option value="2">2</option><option value="3">3</option><option value="4">4</option><option value="5">5</option>
          <option value="6">6</option><option value="7">7</option><option value="8">8</option><option value="9">9</option><option value="10">10</option>
          <option value="ლოჯი">ლოჯი</option><option value="მცირე">მცირე</option>
        </select></div>
        <div class="form-group"><label>პაციენტი</label><input id="edit-name" required></div>
        <div class="form-group"><label>ისტორია</label><input id="edit-history"></div>
        <div class="form-group"><label>ICD-10</label><input id="edit-icd"></div>
        <div class="form-group"><label>ექიმი</label><input id="edit-doctor"></div>
      </div>
      <div class="form-group"><label>კომენტარი</label><textarea id="edit-comment"></textarea></div>
      <div class="modal-actions">
        <button type="button" class="btn btn-secondary" onclick="closeEditModal()">გაუქმება</button>
        <button type="submit" class="btn btn-primary">შენახვა</button>
      </div>
    </form>
  </div>
</div>

<div id="toast" class="toast"></div>
  `;
}

function initOriginalApp(storageKey, roomName) {
  document.querySelector('.header h1').textContent = roomName;

  let patients = JSON.parse(localStorage.getItem(storageKey) || '[]');
  let currentSort = { column: 'admission_date', dir: 'desc' };
  let editingIndex = -1;

  function saveData() {
    localStorage.setItem(storageKey, JSON.stringify(patients));
    renderTables();
    updateStats();
  }

  function escapeHtml(t) { if(!t) return ''; const div = document.createElement('div'); div.textContent = t; return div.innerHTML; }
  function formatDateTime(d) { const date = new Date(d); return date.toLocaleDateString('ka-GE', {day:'numeric', month:'long', year:'numeric'}) + ' – ' + date.toLocaleTimeString('ka-GE', {hour:'2-digit', minute:'2-digit'}); }

  function renderActive() {
    let list = patients.filter(p => !p.archived);
    const s = document.getElementById('search').value.toLowerCase();
    if(s) list = list.filter(p => (p.patient_name||'').toLowerCase().includes(s) || (p.history_number||'').includes(s));

    list.sort((a,b) => {
      let av = a[currentSort.column] ?? '', bv = b[currentSort.column] ?? '';
      if(currentSort.column === 'admission_date') { av = new Date(av).getTime(); bv = new Date(bv).getTime(); }
      return (av < bv ? -1 : 1) * (currentSort.dir === 'asc' ? 1 : -1);
    });

    const tbody = document.getElementById('active-tbody');
    tbody.innerHTML = list.length === 0 ? '<tr><td colspan="8" class="empty-state">პაციენტები არ არის</td></tr>' : list.map((p,i) => `
      <tr>
        <td>${escapeHtml(p.bed||'-')}</td>
        <td>${escapeHtml(p.patient_name||'-')}</td>
        <td>${escapeHtml(p.history_number||'-')}</td>
        <td>${escapeHtml(p.icd10_code||'-')}</td>
        <td>${escapeHtml(p.doctor||'-')}</td>
        <td>${escapeHtml(p.comment||'-')}</td>
        <td>${new Date(p.admission_date).toLocaleDateString('ka-GE')}</td>
        <td class="action-buttons">
          <button class="btn btn-secondary" onclick="openEditModal(${patients.indexOf(p)})">რედაქტირება</button>
          <button class="btn btn-danger" onclick="archivePatient(${patients.indexOf(p)})">არქივი</button>
          <button class="btn btn-delete" onclick="permanentlyDelete(${patients.indexOf(p)})">წაშლა</button>
        </td>
      </tr>
    `).join('');
  }

  function renderArchive() {
    const list = patients.filter(p => p.archived);
    const tbody = document.getElementById('archive-tbody');
    tbody.innerHTML = list.length === 0 ? '<tr><td colspan="9" class="empty-state">არქივი ცარიელია</td></tr>' : list.map(p => `
      <tr>
        <td>${escapeHtml(p.bed||'-')}</td>
        <td>${escapeHtml(p.patient_name||'-')}</td>
        <td>${escapeHtml(p.history_number||'-')}</td>
        <td>${escapeHtml(p.icd10_code||'-')}</td>
        <td>${escapeHtml(p.doctor||'-')}</td>
        <td>${escapeHtml(p.comment||'-')}</td>
        <td>${new Date(p.admission_date).toLocaleDateString('ka-GE')}</td>
        <td>${formatDateTime(p.deleted_date)}</td>
        <td class="action-buttons">
          <button class="btn btn-success" onclick="restorePatient(${patients.indexOf(p)})">დაბრუნება</button>
          <button class="btn btn-delete" onclick="permanentlyDelete(${patients.indexOf(p)})">წაშლა</button>
        </td>
      </tr>
    `).join('');
  }

  function renderTables() { renderActive(); renderArchive(); }
  function updateStats() {
    const today = new Date().toDateString();
    const active = patients.filter(p => !p.archived).length;
    const archived = patients.filter(p => p.archived).length;
    const added = patients.filter(p => !p.archived && new Date(p.admission_date).toDateString() === today).length;
    const deleted = patients.filter(p => p.archived && p.deleted_date && new Date(p.deleted_date).toDateString() === today).length;
    document.getElementById('stat-active').textContent = active;
    document.getElementById('stat-today-added').textContent = added;
    document.getElementById('stat-today-deleted').textContent = deleted;
    document.getElementById('stat-archived').textContent = archived;
  }

  document.getElementById('patient-form').onsubmit = function(e) {
    e.preventDefault();
    patients.push({
      bed: document.getElementById('bed').value,
      patient_name: document.getElementById('patient-name').value.trim(),
      history_number: document.getElementById('history-number').value.trim(),
      icd10_code: document.getElementById('icd10').value.trim(),
      doctor: document.getElementById('doctor').value.trim(),
      comment: document.getElementById('comment').value.trim(),
      admission_date: new Date().toISOString(),
      archived: false
    });
    saveData();
    this.reset();
    showToast('პაციენტი დამატებულია');
  };

  window.openEditModal = function(i) {
    editingIndex = i;
    const p = patients[i];
    document.getElementById('edit-bed').value = p.bed || '';
    document.getElementById('edit-name').value = p.patient_name || '';
    document.getElementById('edit-history').value = p.history_number || '';
    document.getElementById('edit-icd').value = p.icd10_code || '';
    document.getElementById('edit-doctor').value = p.doctor || '';
    document.getElementById('edit-comment').value = p.comment || '';
    document.getElementById('edit-modal').classList.add('active');
  };

  window.closeEditModal = function() {
    document.getElementById('edit-modal').classList.remove('active');
    editingIndex = -1;
  };

  document.getElementById('edit-form').onsubmit = function(e) {
    e.preventDefault();
    if(editingIndex === -1) return;
    patients[editingIndex] = {...patients[editingIndex],
      bed: document.getElementById('edit-bed').value,
      patient_name: document.getElementById('edit-name').value.trim(),
      history_number: document.getElementById('edit-history').value.trim(),
      icd10_code: document.getElementById('edit-icd').value.trim(),
      doctor: document.getElementById('edit-doctor').value.trim(),
      comment: document.getElementById('edit-comment').value.trim()
    };
    saveData();
    closeEditModal();
    showToast('ცვლილებები შენახულია');
  };

  window.archivePatient = function(i) {
    if(confirm('გსურთ არქივში გადატანა?')) {
      patients[i].archived = true;
      patients[i].deleted_date = new Date().toISOString();
      saveData();
      showToast('პაციენტი გადავიდა არქივში');
    }
  };

  window.restorePatient = function(i) {
    if(confirm('დაბრუნება აქტიურ პაციენტებში?')) {
      patients[i].archived = false;
      delete patients[i].deleted_date;
      saveData();
      showToast('პაციენტი დაბრუნდა');
    }
  };

  window.permanentlyDelete = function(i) {
    if(confirm('სამუდამოდ წაშლა? (შეუქცევადია!)')) {
      patients.splice(i, 1);
      saveData();
      showToast('პაციენტი წაიშალა', true);
    }
  };

  window.sortTable = function(col) {
    if(currentSort.column === col) currentSort.dir = currentSort.dir === 'asc' ? 'desc' : 'asc';
    else { currentSort.column = col; currentSort.dir = 'asc'; }
    renderActive();
  };

  window.switchTab = function(tab) {
    document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
    document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
    document.querySelector(`.tab-btn[onclick="switchTab('${tab}')"]`).classList.add('active');
    document.getElementById(tab + '-tab').classList.add('active');
  };

  function showToast(msg, err = false) {
    const t = document.getElementById('toast');
    t.textContent = msg;
    t.className = 'toast active' + (err ? ' error' : '');
    setTimeout(() => t.classList.remove('active'), 4000);
  }

  document.getElementById('search').addEventListener('input', renderActive);

  // სრული სტილები (შენი ძველი CSS)
  const style = document.createElement('style');
  style.textContent = `
    /* შენი ზუსტი ძველი სტილები — არაფერი შეცვლილია */
    .container{max-width:1400px;margin:0 auto;padding:20px;}
    .header{background:white;padding:30px;border-radius:12px;box-shadow:0 4px 6px rgba(0,0,0,0.1);margin-bottom:20px;text-align:center;}
    .header h1{margin:0 0 10px 0;color:#667eea;font-size:2.5em;}
    .header p{margin:0;color:#666;font-size:1.1em;}
    .tabs{display:flex;gap:10px;margin-bottom:20px;flex-wrap:wrap;}
    .tab-btn{padding:12px 24px;background:white;border:none;border-radius:8px;cursor:pointer;font-size:16px;font-weight:600;transition:all .3s;color:#667eea;}
    .tab-btn:hover{transform:translateY(-2px);box-shadow:0 4px 12px rgba(0,0,0,0.15);}
    .tab-btn.active{background:#667eea;color:white;}
    .tab-content{display:none;}
    .tab-content.active{display:block;}
    .card{background:white;padding:30px;border-radius:12px;box-shadow:0 4px 6px rgba(0,0,0,0.1);margin-bottom:20px;}
    .form-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:20px;margin-bottom:20px;}
    .form-group{display:flex;flex-direction:column;}
    .form-group label{margin-bottom:8px;font-weight:600;color:#333;}
    .form-group input,.form-group select,.form-group textarea{padding:12px;border:2px solid #e0e0e0;border-radius:8px;font-size:14px;transition:border-color .3s;}
    .form-group input:focus,.form-group select:focus,.form-group textarea:focus{outline:none;border-color:#667eea;}
    .form-group textarea{min-height:100px;resize:vertical;}
    .btn{padding:12px 24px;border:none;border-radius:8px;cursor:pointer;font-size:16px;font-weight:600;transition:all .3s;}
    .btn-primary{background:#667eea;color:white;}
    .btn-primary:hover{background:#5568d3;transform:translateY(-2px);box-shadow:0 4px 12px rgba(102,126,234,0.4);}
    .btn-secondary{background:#f0f0f0;color:#333;}
    .btn-secondary:hover{background:#e0e0e0;}
    .btn-success{background:#27ae60;color:white;}
    .btn-success:hover{background:#229954;}
    .btn-danger{background:#e74c3c;color:white;}
    .btn-danger:hover{background:#c0392b;}
    .btn-delete{background:#c0392b;color:white;}
    .btn-delete:hover{background:#a93226;}
    .search-filter input{padding:10px;border:2px solid #e0e0e0;border-radius:8px;font-size:14px;flex:1;min-width:200px;}
    table{width:100%;border-collapse:collapse;background:white;}
    th,td{padding:15px;text-align:left;border-bottom:1px solid #e0e0e0;}
    th{background:#f8f9fa;font-weight:600;color:#333;cursor:pointer;}
    tr:hover{background:#f8f9fa;}
    .action-buttons{display:flex;gap:8px;flex-wrap:wrap;}
    .action-buttons button{padding:6px 12px;font-size:14px;}
    .stats-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:20px;margin-bottom:20px;}
    .stat-card{background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);color:white;padding:25px;border-radius:12px;text-align:center;}
    .stat-card h3{margin:0 0 10px 0;font-size:1.2em;opacity:0.9;}
    .stat-card .number{font-size:3em;font-weight:bold;margin:10px 0;}
    .modal{display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.5);z-index:1000;justify-content:center;align-items:center;}
    .modal.active{display:flex;}
    .modal-content{background:white;padding:30px;border-radius:12px;max-width:600px;width:100%;max-height:90vh;overflow-y:auto;}
    .modal-header{display:flex;justify-content:space-between;align-items:center;margin-bottom:20px;}
    .modal-header h2{margin:0;color:#667eea;}
    .close-btn{background:none;border:none;font-size:28px;cursor:pointer;color:#999;}
    .close-btn:hover{color:#333;}
    .modal-actions{display:flex;gap:10px;justify-content:flex-end;margin-top:20px;}
    .toast{position:fixed;top:20px;right:20px;background:#27ae60;color:white;padding:15 ressent25px;border-radius:8px;box-shadow:0 4px 12px rgba(0,0,0,0.3);z-index:2000;display:none;animation:slideIn .3s ease;}
    .toast.error{background:#e74c3c;}
    .toast.active{display:block;}
    @keyframes slideIn{from{transform:translateX(400px);opacity:0;}to{transform:translateX(0);opacity:1;}}
    .empty-state{text-align:center;padding:60px 20px;color:#999;}
  `;
  document.head.appendChild(style);

  renderTables();
  updateStats();
}
</script>
</body>
</html>
