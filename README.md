<!DOCTYPE html>
<html lang="ka">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>საწოლების მართვის სისტემა</title>
  <script src="/_sdk/data_sdk.js"></script>
  <script src="/_sdk/element_sdk.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body {box-sizing:border-box;margin:0;padding:0;font-family:'Segoe UI',Tahoma,Geneva,Verdana,sans-serif;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);min-height:100vh;}
    *{box-sizing:border-box;}
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
    .btn-primary:disabled{background:#ccc;cursor:not-allowed;transform:none;}
    .btn-secondary{background:#f0f0f0;color:#333;}
    .btn-secondary:hover{background:#e0e0e0;}
    .btn-danger{background:#e74c3c;color:white;}
    .btn-danger:hover{background:#c0392b;}
    .btn-success{background:#27ae60;color:white;}
    .btn-success:hover{background:#229954;}
    .search-filter{display:flex;gap:15px;margin-bottom:20px;flex-wrap:wrap;}
    .search-filter input,.search-filter select{padding:10px;border:2px solid #e0e0e0;border-radius:8px;font-size:14px;flex:1;min-width:200px;}
    .table-container{overflow-x:auto;}
    table{width:100%;border-collapse:collapse;background:white;}
    th,td{padding:15px;text-align:left;border-bottom:1px solid #e0e0e0;}
    th{background:#f8f9fa;font-weight:600;color:#333;cursor:pointer;user-select:none;}
    th:hover{background:#e9ecef;}
    tr:hover{background:#f8f9fa;}
    .action-buttons{display:flex;gap:8px;flex-wrap:wrap;}
    .action-buttons button{padding:6px 12px;font-size:14px;}
    .stats-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:20px;margin-bottom:20px;}
    .stat-card{background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);color:white;padding:25px;border-radius:12px;text-align:center;}
    .stat-card h3{margin:0 0 10px 0;font-size:1.2em;opacity:0.9;}
    .stat-card .number{font-size:3em;font-weight:bold;margin:10px 0;}
    .modal{display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.5);z-index:1000;justify-content:center;align-items:center;padding:20px;}
    .modal.active{display:flex;}
    .modal-content{background:white;padding:30px;border-radius:12px;max-width:500px;width:100%;max-height:90%;overflow-y:auto;}
    .modal-header{display:flex;justify-content:space-between;align-items:center;margin-bottom:20px;}
    .modal-header h2{margin:0;color:#667eea;}
    .close-btn{background:none;border:none;font-size:24px;cursor:pointer;color:#999;}
    .close-btn:hover{color:#333;}
    .modal-actions{display:flex;gap:10px;justify-content:flex-end;margin-top:20px;}
    .toast{position:fixed;top:20px;right:20px;background:#27ae60;color:white;padding:15px 25px;border-radius:8px;box-shadow:0 4px 12px rgba(0,0,0,0.3);z-index:2000;display:none;animation:slideIn .3s ease;}
    .toast.error{background:#e74c3c;}
    .toast.active{display:block;}
    @keyframes slideIn{from{transform:translateX(400px);opacity:0;}to{transform:translateX(0);opacity:1;}}
    .export-buttons{display:flex;gap:10px;flex-wrap:wrap;margin-bottom:20px;}
    .loading{display:inline-block;width:20px;height:20px;border:3px solid rgba(255,255,255,.3);border-radius:50%;border-top-color:white;animation:spin 1s ease-in-out infinite;}
    @keyframes spin{to{transform:rotate(360deg);}}
    .empty-state{text-align:center;padding:60px 20px;color:#999;}
    @media(max-width:768px){
      .header h1{font-size:1.8em;}
      .form-grid{grid-template-columns:1fr;}
      .search-filter{flex-direction:column;}
      .search-filter input,.search-filter select{width:100%;}
      .action-buttons{flex-direction:column;}
    }
  </style>
</head>
<body>
<div class="container">
  <div class="header">
    <h1 id="hospital-name">საწოლების მართვის სისტემა</h1>
    <p id="department-name">სტაციონარი / Inpatient</p>
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
          <div class="form-group"><label for="bed">საწოლი</label>
            <select id="bed">
              <option value="">— აირჩიეთ —</option>
              <option value="1">1</option><option value="2">2</option><option value="3">3</option>
              <option value="4">4</option><option value="5">5</option><option value="6">6</option>
              <option value="7">7</option><option value="8">8</option>
              <option value="ლოჯი">ლოჯი</option>
              <option value="მცირე (3 საწოლი)">მცირე (3 საწოლი)</option>
            </select>
          </div>
          <div class="form-group"><label for="patient-name">პაციენტი</label>
            <input type="text" id="patient-name" placeholder="მაგ: გიორგი გელაშვილი">
          </div>
          <div class="form-group"><label for="history-number">ისტორია</label>
            <input type="text" id="history-number" placeholder="მაგ: 12345">
          </div>
          <div class="form-group"><label for="icd10">ICD-10</label>
            <input type="text" id="icd10" placeholder="მაგ: J18.9">
          </div>
          <div class="form-group"><label for="doctor">ექიმი</label>
            <input type="text" id="doctor" placeholder="მაგ: დ. ბერიძე">
          </div>
        </div>
        <div class="form-group"><label for="comment">კომენტარი</label>
          <textarea id="comment" placeholder="დამატებითი ინფორმაცია..."></textarea>
        </div>
        <button type="submit" class="btn btn-primary" id="submit-btn">
          <span id="submit-text">დამატება</span>
        </button>
      </form>
    </div>

    <div class="card">
      <h2>აქტიური პაციენტები</h2>
      <div class="search-filter">
        <input type="text" id="search" placeholder="ძებნა">
        <select id="filter-bed">
          <option value="">ყველა საწოლი</option>
          <option value="1">1</option><option value="2">2</option><option value="3">3</option>
          <option value="4">4</option><option value="5">5</option><option value="6">6</option>
          <option value="7">7</option><option value="8">8</option>
          <option value="ლოჯი">ლოჯი</option>
          <option value="მცირე (3 საწოლი)">მცირე (3 საწოლი)</option>
        </select>
        <select id="filter-doctor"><option value="">ყველა ექიმი</option></select>
        <select id="filter-icd"><option value="">ყველა ICD-10</option></select>
      </div>

      <div class="table-container">
        <table id="active-table">
          <thead>
            <tr>
              <th onclick="sortTable('bed')">საწოლი</th>
              <th onclick="sortTable('patient_name')">პაციენტი</th>
              <th onclick="sortTable('history_number')">ისტორია</th>
              <th onclick="sortTable('icd10_code')">ICD-10</th>
              <th onclick="sortTable('doctor')">ექიმი</th>
              <th>კომენტარი</th>
              <th onclick="sortTable('admission_date')">თარიღი</th>
              <th>მოქმედება</th>
            </tr>
          </thead>
          <tbody id="active-tbody"></tbody>
        </table>
      </div>

      <div class="export-buttons">
        <button class="btn btn-success" onclick="exportToExcel('active')">Excel</button>
        <button class="btn btn-success" onclick="exportToPDF('active')">PDF</button>
      </div>
    </div>
  </div>

  <!-- არქივი -->
  <div id="archive-tab" class="tab-content">
    <div class="card">
      <h2>არქივი</h2>
      <div class="search-filter">
        <input type="text" id="archive-search" placeholder="ძებნა">
        <select id="archive-filter-bed">
          <option value="">ყველა საწოლი</option>
          <option value="1">1</option><option value="2">2</option><option value="3">3</option>
          <option value="4">4</option><option value="5">5</option><option value="6">6</option>
          <option value="7">7</option><option value="8">8</option>
          <option value="ლოჯი">ლოჯი</option>
          <option value="მცირე (3 საწოლი)">მცირე (3 საწოლი)</option>
        </select>
      </div>
      <div class="table-container">
        <table id="archive-table">
          <thead>
            <tr>
              <th>საწოლი</th><th>პაციენტი</th><th>ისტორია</th><th>ICD-10</th>
              <th>ექიმი</th><th>კომენტარი</th><th>ჩარიცხვა</th><th>წაშლა</th><th>მოქმედება</th>
            </tr>
          </thead>
          <tbody id="archive-tbody"></tbody>
        </table>
      </div>
      <div class="export-buttons">
        <button class="btn btn-success" onclick="exportToExcel('archive')">Excel</button>
        <button class="btn btn-success" onclick="exportToPDF('archive')">PDF</button>
      </div>
    </div>
  </div>

  <!-- სტატისტიკა -->
  <div id="statistics-tab" class="tab-content">
    <div class="card">
      <h2>სტატისტიკა</h2>
      <div class="stats-grid">
        <div class="stat-card"><h3>აქტიური</h3><div class="number" id="stat-active">0</div></div>
        <div class="stat-card"><h3>დღეს დამატებული</h3><div class="number" id="stat-added-today">0</div></div>
        <div class="stat-card"><h3>დღეს წაშლილი</h3><div class="number" id="stat-deleted-today">0</div></div>
        <div class="stat-card"><h3>არქივში</h3><div class="number" id="stat-archived">0</div></div>
      </div>
    </div>
  </div>
</div>

<!-- მოდალები -->
<div id="edit-modal" class="modal"><div class="modal-content">...</div></div>
<div id="view-modal" class="modal"><div class="modal-content">...</div></div>
<div id="delete-modal" class="modal"><div class="modal-content">...</div></div>
<div id="toast" class="toast"></div>

<script>
  let allPatients = [];
  let currentSortColumn = 'admission_date';
  let currentSortDirection = 'desc';

  function escapeHtml(t) { return t == null ? '' : String(t).replace(/[&<>"']/g, m => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'})[m]); }

  const dataHandler = { onDataChanged: d => { allPatients = d || []; renderTables(); updateStats(); updateFilters(); } };
  async function init() { if (window.dataSdk) await window.dataSdk.init(dataHandler); }
  init();

  // მთავარი ფუნქცია — ყველა ცხრილის განახლება
  function renderTables() {
    renderActivePatientsTable();
    renderArchivedPatientsTable();
  }

  function getActiveFiltered() {
    let list = allPatients.filter(p => p.__collection === 'active');
    const s = document.getElementById('search').value.toLowerCase();
    const bed = document.getElementById('filter-bed').value;
    const doc = document.getElementById('filter-doctor').value;
    const icd = document.getElementById('filter-icd').value;

    if (s) list = list.filter(p => (p.patient_name||'').toLowerCase().includes(s) || (p.history_number||'').includes(s));
    if (bed) list = list.filter(p => p.bed === bed);
    if (doc) list = list.filter(p => p.doctor === doc);
    if (icd) list = list.filter(p => p.icd10_code === icd);

    list.sort((a,b) => {
      let av = a[currentSortColumn] ?? '';
      let bv = b[currentSortColumn] ?? '';
      if (currentSortColumn === 'admission_date') { av = new Date(av).getTime(); bv = new Date(bv).getTime(); }
      return (av < bv ? -1 : 1) * (currentSortDirection === 'asc' ? 1 : -1);
    });
    return list;
  }

  function renderActivePatientsTable() {
    const tbody = document.getElementById('active-tbody');
    const data = getActiveFiltered();
    if (!data.length) {
      tbody.innerHTML = '<tr><td colspan="8" class="empty-state"><p>პაციენტები არ არის</p></td></tr>';
      return;
    }
    tbody.innerHTML = data.map((p,i) => `
      <tr data-idx="${i}">
        <td>${escapeHtml(p.bed || '-')}</td>
        <td>${escapeHtml(p.patient_name || '-')}</td>
        <td>${escapeHtml(p.history_number || '-')}</td>
        <td>${escapeHtml(p.icd10_code || '-')}</td>
        <td>${escapeHtml(p.doctor || '-')}</td>
        <td>${escapeHtml(p.comment || '-')}</td>
        <td>${new Date(p.admission_date).toLocaleDateString('ka-GE')}</td>
        <td class="action-buttons">
          <button class="btn btn-primary"   onclick="openViewModal(data[${i}])">ნახვა</button>
          <button class="btn btn-secondary" onclick="openEditModal(data[${i}])">რედაქტირება</button>
          <button class="btn btn-danger"    onclick="openDeleteModal(data[${i}])">წაშლა</button>
        </td>
      </tr>`).join('');
  }

  function renderArchivedPatientsTable() {
    const tbody = document.getElementById('archive-tbody');
    let data = allPatients.filter(p => p.__collection === 'archived');
    const s = document.getElementById('archive-search').value.toLowerCase();
    const bed = document.getElementById('archive-filter-bed').value;
    if (s) data = data.filter(p => (p.patient_name||'').toLowerCase().includes(s) || (p.history_number||'').includes(s));
    if (bed) data = data.filter(p => p.bed === bed);

    if (!data.length) {
      tbody.innerHTML = '<tr><td colspan="9" class="empty-state"><p>არქივი ცარიელია</p></td></tr>';
      return;
    }
    tbody.innerHTML = data.map(p => `
      <tr>
        <td>${escapeHtml(p.bed || '-')}</td>
        <td>${escapeHtml(p.patient_name || '-')}</td>
        <td>${escapeHtml(p.history_number || '-')}</td>
        <td>${escapeHtml(p.icd10_code || '-')}</td>
        <td>${escapeHtml(p.doctor || '-')}</td>
        <td>${escapeHtml(p.comment || '-')}</td>
        <td>${new Date(p.admission_date).toLocaleDateString('ka-GE')}</td>
        <td>${new Date(p.deleted_date).toLocaleDateString('ka-GE')}</td>
        <td><button class="btn btn-primary" onclick="openViewModal(findById('${p.__id}'))">ნახვა</button></td>
      </tr>`).join('');
  }

  function findById(id) { return allPatients.find(p => p.__id === id); }

  // ---------- დამატება (მთავარი გამოსწორება) ----------
  document.getElementById('patient-form').addEventListener('submit', async e => {
    e.preventDefault();
    const btn = document.getElementById('submit-btn');
    const txt = document.getElementById('submit-text');
    btn.disabled = true;
    txt.innerHTML = '<span class="loading"></span>';

    const patient = {
      __collection: 'active',
      bed: document.getElementById('bed').value || '',
      patient_name: document.getElementById('patient-name').value.trim() || '',
      history_number: document.getElementById('history-number').value.trim() || '',
      icd10_code: document.getElementById('icd10').value.trim() || '',
      doctor: document.getElementById('doctor').value.trim() || '',
      comment: document.getElementById('comment').value.trim() || '',
      admission_date: new Date().toISOString(),
      deleted_date: ''
    };

    const result = await window.dataSdk.create(patient);

    btn.disabled = false;
    txt.textContent = 'დამატება';

    if (result.isOk) {
      showToast('პაციენტი დაემატა');
      document.getElementById('patient-form').reset();
    } else {
      showToast('შეცდომა', 'error');
    }
  });

  // ---------- სხვა ფუნქციები (მოდალები, სორტირება, სტატისტიკა და ა.შ.) ----------
  function sortTable(col) {
    if (currentSortColumn === col) currentSortDirection = currentSortDirection === 'asc' ? 'desc' : 'asc';
    else { currentSortColumn = col; currentSortDirection = 'asc'; }
    renderActivePatientsTable();
  }

  function switchTab(t) {
    document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
    document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
    document.querySelector(`.tab-btn[onclick="switchTab('${t}')"]`).classList.add('active');
    document.getElementById(t + '-tab').classList.add('active');
  }

  function showToast(msg, type = 'success') {
    const t = document.getElementById('toast');
    t.textContent = msg;
    t.className = 'toast active ' + type;
    setTimeout(() => t.classList.remove('active'), 3000);
  }

  function updateStats() {
    const active = allPatients.filter(p => p.__collection === 'active').length;
    const archived = allPatients.filter(p => p.__collection === 'archived').length;
    const today = new Date().toDateString();
    const added = allPatients.filter(p => p.__collection === 'active' && new Date(p.admission_date).toDateString() === today).length;
    const deleted = allPatients.filter(p => p.__collection === 'archived' && new Date(p.deleted_date).toDateString() === today).length;
    document.getElementById('stat-active').textContent = active;
    document.getElementById('stat-added-today').textContent = added;
    document.getElementById('stat-deleted-today').textContent = deleted;
    document.getElementById('stat-archived').textContent = archived;
  }

  function updateFilters() {
    const docs = [...new Set(allPatients.map(p => p.doctor).filter(Boolean))];
    const icds = [...new Set(allPatients.map(p => p.icd10_code).filter(Boolean))];
    document.getElementById('filter-doctor').innerHTML = '<option value="">ყველა ექიმი</option>' + docs.map(d => `<option>${escapeHtml(d)}</option>`).join('');
    document.getElementById('filter-icd').innerHTML = '<option value="">ყველა ICD-10</option>' + icds.map(i => `<option>${escapeHtml(i)}</option>`).join('');
  }

  // მოდალების ფუნქციები (შენ შეგიძლია დაამატო შენი ვერსია, ან დატოვო ცარიელი)
  function openViewModal(p) { console.log('ნახვა:', p); }
  function openEditModal(p) { console.log('რედაქტირება:', p); }
  function openDeleteModal(p) { console.log('წაშლა:', p); }

  // ფილტრების მოსმენა
  ['search','filter-bed','filter-doctor','filter-icd'].forEach(id => document.getElementById(id).addEventListener('input', renderActivePatientsTable));
  document.getElementById('archive-search').addEventListener('input', renderArchivedPatientsTable);
  document.getElementById('archive-filter-bed').addEventListener('change', renderArchivedPatientsTable);
</script>
</body>
</html>
