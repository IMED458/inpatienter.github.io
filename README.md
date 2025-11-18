<!DOCTYPE html>
<html lang="ka">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Inpatient Management System</title>
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

  <!-- ====== აქტიური პაციენტები ====== -->
  <div id="active-tab" class="tab-content active">
    <div class="card">
      <h2 id="add-patient-title">ახალი პაციენტის დამატება</h2>
      <form id="patient-form">
        <div class="form-grid">
          <div class="form-group"><label for="bed">საწოლი *</label>
            <select id="bed" required>
              <option value="">აირჩიეთ საწოლი</option>
              <option value="1">1</option><option value="2">2</option><option value="3">3</option>
              <option value="4">4</option><option value="5">5</option><option value="6">6</option>
              <option value="7">7</option><option value="8">8</option>
              <option value="ლოჯი">ლოჯი</option>
              <option value="მცირე (3 საწოლი)">მცირე (3 საწოლი)</option>
            </select>
          </div>
          <div class="form-group"><label for="patient-name">პაციენტის სახელი და გვარი *</label>
            <input type="text" id="patient-name" required placeholder="მაგ: გიორგი გელაშვილი">
          </div>
          <div class="form-group"><label for="history-number">ისტორიის ნომერი *</label>
            <input type="text" id="history-number" required placeholder="მაგ: 12345">
          </div>
          <div class="form-group"><label for="icd10">ICD-10 კოდი</label>
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
        <input type="text" id="search" placeholder="ძებნა (სახელი, გვარი, ისტორია)">
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
        <button class="btn btn-success" onclick="exportToExcel('active')">Excel ექსპორტი</button>
        <button class="btn btn-success" onclick="exportToPDF('active')">PDF ექსპორტი</button>
      </div>
    </div>
  </div>

  <!-- ====== არქივი ====== -->
  <div id="archive-tab" class="tab-content">
    <div class="card">
      <h2>წაშლილი პაციენტების არქივი</h2>
      <div class="search-filter">
        <input type="text" id="archive-search" placeholder="ძებნა არქივში">
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
        <button class="btn btn-success" onclick="exportToExcel('archive')">Excel ექსპორტი</button>
        <button class="btn btn-success" onclick="exportToPDF('archive')">PDF ექსპორტი</button>
      </div>
    </div>
  </div>

  <!-- ====== სტატისტიკა ====== -->
  <div id="statistics-tab" class="tab-content">
    <div class="card">
      <h2 id="statistics-title">დღიური სტატისტიკა</h2>
      <div class="stats-grid">
        <div class="stat-card"><h3>აქტიური პაციენტები</h3><div class="number" id="stat-active">0</div></div>
        <div class="stat-card"><h3>დღეს დამატებული</h3><div class="number" id="stat-added-today">0</div></div>
        <div class="stat-card"><h3>დღეს წაშლილი</h3><div class="number" id="stat-deleted-today">0</div></div>
        <div class="stat-card"><h3>სულ არქივში</h3><div class="number" id="stat-archived">0</div></div>
      </div>
    </div>
  </div>
</div>

<!-- მოდალები -->
<div id="edit-modal" class="modal">
  <div class="modal-content">
    <div class="modal-header"><h2>პაციენტის რედაქტირება</h2><button class="close-btn" onclick="closeEditModal()">x</button></div>
    <form id="edit-form">
      <div class="form-grid">
        <div class="form-group"><label for="edit-bed">საწოლი *</label>
          <select id="edit-bed" required>
            <option value="1">1</option><option value="2">2</option><option value="3">3</option>
            <option value="4">4</option><option value="5">5</option><option value="6">6</option>
            <option value="7">7</option><option value="8">8</option>
            <option value="ლოჯი">ლოჯი</option><option value="მცირე (3 საწოლი)">მცირე (3 საწოლი)</option>
          </select>
        </div>
        <div class="form-group"><label for="edit-patient-name">პაციენტის სახელი და გვარი *</label><input type="text" id="edit-patient-name" required></div>
        <div class="form-group"><label for="edit-history-number">ისტორიის ნომერი *</label><input type="text" id="edit-history-number" required></div>
        <div class="form-group"><label for="edit-icd10">ICD-10 კოდი</label><input type="text" id="edit-icd10"></div>
        <div class="form-group"><label for="edit-doctor">ექიმი</label><input type="text" id="edit-doctor"></div>
      </div>
      <div class="form-group"><label for="edit-comment">კომენტარი</label><textarea id="edit-comment"></textarea></div>
      <div class="modal-actions">
        <button type="button" class="btn btn-secondary" onclick="closeEditModal()">გაუქმება</button>
        <button type="submit" class="btn btn-primary" id="edit-submit-btn"><span id="edit-submit-text">შენახვა</span></button>
      </div>
    </form>
  </div>
</div>

<div id="view-modal" class="modal">
  <div class="modal-content">
    <div class="modal-header"><h2>პაციენტის დეტალები</h2><button class="close-btn" onclick="closeViewModal()">x</button></div>
    <div id="view-content"></div>
    <div class="modal-actions">
      <button type="button" class="btn btn-secondary" onclick="closeViewModal()">დახურვა</button>
      <button type="button" class="btn btn-success" onclick="exportSinglePatientPDF()">PDF ექსპორტი</button>
    </div>
  </div>
</div>

<div id="delete-modal" class="modal">
  <div class="modal-content">
    <div class="modal-header"><h2>დადასტურება</h2><button class="close-btn" onclick="closeDeleteModal()">x</button></div>
    <p style="margin:20px 0;">დარწმუნებული ხართ, რომ გსურთ ამ პაციენტის წაშლა? ჩანაწერი გადავა არქივში.</p>
    <div class="modal-actions">
      <button type="button" class="btn btn-secondary" onclick="closeDeleteModal()">გაუქმება</button>
      <button type="button" class="btn btn-danger" id="confirm-delete-btn" onclick="confirmDelete()"><span id="delete-text">წაშლა</span></button>
    </div>
  </div>
</div>

<div id="toast" class="toast"></div>

<script>
  // ---------- გლობალური ცვლადები ----------
  let allPatients = [];
  let currentEditPatient = null;
  let currentDeletePatient = null;
  let currentViewPatient = null;
  let currentSortColumn = 'admission_date';
  let currentSortDirection = 'desc';

  // ---------- უსაფრთხო HTML escape ----------
  function escapeHtml(text) {
    if (text == null) return '';
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
  }

  // ---------- მონაცემების ინიციალიზაცია ----------
  const dataHandler = {
    onDataChanged(data) {
      allPatients = data || [];
      renderActivePatientsTable();
      renderArchivedPatientsTable();
      updateStatistics();
      updateFilterOptions();
    }
  };

  async function initializeApp() {
    if (window.dataSdk) {
      const r = await window.dataSdk.init(dataHandler);
      if (!r.isOk) showToast('სისტემის ინიციალიზაცია ვერ მოხერხდა', 'error');
    }
  }

  // ---------- ფილტრაცია & სორტირება ----------
  function getFilteredActive() {
    let list = allPatients.filter(p => p.__collection === 'active');

    const s = document.getElementById('search').value.toLowerCase();
    const bed = document.getElementById('filter-bed').value;
    const doctor = document.getElementById('filter-doctor').value;
    const icd = document.getElementById('filter-icd').value;

    if (s) list = list.filter(p => p.patient_name.toLowerCase().includes(s) || p.history_number.includes(s));
    if (bed) list = list.filter(p => p.bed === bed);
    if (doctor) list = list.filter(p => p.doctor === doctor);
    if (icd) list = list.filter(p => p.icd10_code === icd);

    list.sort((a, b) => {
      let av = a[currentSortColumn] ?? '';
      let bv = b[currentSortColumn] ?? '';
      if (currentSortColumn === 'admission_date') { av = new Date(av).getTime(); bv = new Date(bv).getTime(); }
      const res = av < bv ? -1 : av > bv ? 1 : 0;
      return currentSortDirection === 'asc' ? res : -res;
    });
    return list;
  }

  // ---------- ტაბლიცების რენდერი ----------
  function renderActivePatientsTable() {
    const tbody = document.getElementById('active-tbody');
    const data = getFilteredActive();

    if (data.length === 0) {
      tbody.innerHTML = '<tr><td colspan="8" class="empty-state"><p>პაციენტები არ არის</p></td></tr>';
      return;
    }

    tbody.innerHTML = data.map((p, i) => `
      <tr data-idx="${i}">
        <td>${escapeHtml(p.bed)}</td>
        <td>${escapeHtml(p.patient_name)}</td>
        <td>${escapeHtml(p.history_number)}</td>
        <td>${escapeHtml(p.icd10_code || 'N/A')}</td>
        <td>${escapeHtml(p.doctor || 'N/A')}</td>
        <td>${escapeHtml(p.comment || 'N/A')}</td>
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
    if (s) data = data.filter(p => p.patient_name.toLowerCase().includes(s) || p.history_number.includes(s));
    if (bed) data = data.filter(p => p.bed === bed);

    if (data.length === 0) {
      tbody.innerHTML = '<tr><td colspan="9" class="empty-state"><p>არქივი ცარიელია</p></td></tr>';
      return;
    }

    tbody.innerHTML = data.map(p => `
      <tr>
        <td>${escapeHtml(p.bed)}</td>
        <td>${escapeHtml(p.patient_name)}</td>
        <td>${escapeHtml(p.history_number)}</td>
        <td>${escapeHtml(p.icd10_code || 'N/A')}</td>
        <td>${escapeHtml(p.doctor || 'N/A')}</td>
        <td>${escapeHtml(p.comment || 'N/A')}</td>
        <td>${new Date(p.admission_date).toLocaleDateString('ka-GE')}</td>
        <td>${new Date(p.deleted_date).toLocaleDateString('ka-GE')}</td>
        <td><button class="btn btn-primary" onclick="openViewModal(findPatientById('${p.__id}'))">ნახვა</button></td>
      </tr>`).join('');
  }

  function findPatientById(id) { return allPatients.find(p => p.__id === id); }

  // ---------- დამატება (აქ იყო მთავარი პრობლემა!) ----------
  document.getElementById('patient-form').addEventListener('submit', async function (e) {
    e.preventDefault();

    const btn = document.getElementById('submit-btn');
    const txt = document.getElementById('submit-text');
    btn.disabled = true;
    txt.innerHTML = '<span class="loading"></span>';

    const newPatient = {
      __collection: 'active',
      bed: document.getElementById('bed').value,
      patient_name: document.getElementById('patient-name').value.trim(),
      history_number: document.getElementById('history-number').value.trim(),
      icd10_code: document.getElementById('icd10').value.trim(),
      doctor: document.getElementById('doctor').value.trim(),
      comment: document.getElementById('comment').value.trim(),
      admission_date: new Date().toISOString(),
      deleted_date: ''
    };

    const result = await window.dataSdk.create(newPatient);

    btn.disabled = false;
    txt.textContent = 'დამატება';

    if (result.isOk) {
      showToast('პაციენტი წარმატებით დაემატა');
      document.getElementById('patient-form').reset();
    } else {
      showToast('შეცდომა დამატებისას', 'error');
    }
  });

  // ---------- რედაქტირება ----------
  document.getElementById('edit-form').addEventListener('submit', async function (e) {
    e.preventDefault();
    if (!currentEditPatient) return;

    const btn = document.getElementById('edit-submit-btn');
    const txt = document.getElementById('edit-submit-text');
    btn.disabled = true;
    txt.innerHTML = '<span class="loading"></span>';

    const updated = {
      ...currentEditPatient,
      bed: document.getElementById('edit-bed').value,
      patient_name: document.getElementById('edit-patient-name').value.trim(),
      history_number: document.getElementById('edit-history-number').value.trim(),
      icd10_code: document.getElementById('edit-icd10').value.trim(),
      doctor: document.getElementById('edit-doctor').value.trim(),
      comment: document.getElementById('edit-comment').value.trim()
    };

    const r = await window.dataSdk.update(updated);
    btn.disabled = false;
    txt.textContent = 'შენახვა';

    if (r.isOk) {
      showToast('ცვლილებები შენახულია');
      closeEditModal();
    } else {
      showToast('შეცდომა შენახვისას', 'error');
    }
  });

  // ---------- წაშლა (არქივში გადატანა) ----------
  async function confirmDelete() {
    if (!currentDeletePatient) return;
    const btn = document.getElementById('confirm-delete-btn');
    const txt = document.getElementById('delete-text');
    btn.disabled = true;
    txt.innerHTML = '<span class="loading"></span>';

    const archived = {
      ...currentDeletePatient,
      __collection: 'archived',
      deleted_date: new Date().toISOString()
    };

    const r = await window.dataSdk.update(archived);
    btn.disabled = false;
    txt.textContent = 'წაშლა';

    if (r.isOk) {
      showToast('პაციენტი გადატანილია არქივში');
      closeDeleteModal();
    } else {
      showToast('შეცდომა', 'error');
    }
  }

  // ---------- მოდალები ----------
  function openEditModal(p) {
    currentEditPatient = p;
    document.getElementById('edit-bed').value = p.bed;
    document.getElementById('edit-patient-name').value = p.patient_name;
    document.getElementById('edit-history-number').value = p.history_number;
    document.getElementById('edit-icd10').value = p.icd10_code || '';
    document.getElementById('edit-doctor').value = p.doctor || '';
    document.getElementById('edit-comment').value = p.comment || '';
    document.getElementById('edit-modal').classList.add('active');
  }
  function closeEditModal() { currentEditPatient = null; document.getElementById('edit-modal').classList.remove('active'); }

  function openViewModal(p) {
    currentViewPatient = p;
    const adm = new Date(p.admission_date).toLocaleString('ka-GE');
    const del = p.deleted_date ? new Date(p.deleted_date).toLocaleString('ka-GE') : '';
    document.getElementById('view-content').innerHTML = `
      <div style="line-height:2;">
        <p><strong>საწოლი:</strong> ${escapeHtml(p.bed)}</p>
        <p><strong>პაციენტი:</strong> ${escapeHtml(p.patient_name)}</p>
        <p><strong>ისტორია:</strong> ${escapeHtml(p.history_number)}</p>
        <p><strong>ICD-10:</strong> ${escapeHtml(p.icd10_code || 'N/A')}</p>
        <p><strong>ექიმი:</strong> ${escapeHtml(p.doctor || 'N/A')}</p>
        <p><strong>კომენტარი:</strong> ${escapeHtml(p.comment || 'N/A')}</p>
        <p><strong>ჩარიცხვა:</strong> ${adm}</p>
        ${del ? `<p><strong>წაშლა:</strong> ${del}</p>` : ''}
      </div>`;
    document.getElementById('view-modal').classList.add('active');
  }
  function closeViewModal() { document.getElementById('view-modal').classList.remove('active'); }

  function openDeleteModal(p) { currentDeletePatient = p; document.getElementById('delete-modal').classList.add('active'); }
  function closeDeleteModal() { currentDeletePatient = null; document.getElementById('delete-modal').classList.remove('active'); }

  // ---------- სორტირება ----------
  function sortTable(col) {
    if (currentSortColumn === col) currentSortDirection = currentSortDirection === 'asc' ? 'desc' : 'asc';
    else { currentSortColumn = col; currentSortDirection = 'asc'; }
    renderActivePatientsTable();
  }

  // ---------- ტაბები ----------
  function switchTab(tab) {
    document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
    document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
    document.querySelector(`.tab-btn[onclick="switchTab('${tab}')"]`).classList.add('active');
    document.getElementById(tab + '-tab').classList.add('active');
  }

  // ---------- სტატისტიკა ----------
  function updateStatistics() {
    const active = allPatients.filter(p => p.__collection === 'active').length;
    const archived = allPatients.filter(p => p.__collection === 'archived').length;
    const today = new Date().toDateString();
    const addedToday = allPatients.filter(p => p.__collection === 'active' && new Date(p.admission_date).toDateString() === today).length;
    const deletedToday = allPatients.filter(p => p.__collection === 'archived' && new Date(p.deleted_date).toDateString() === today).length;

    document.getElementById('stat-active').textContent = active;
    document.getElementById('stat-added-today').textContent = addedToday;
    document.getElementById('stat-deleted-today').textContent = deletedToday;
    document.getElementById('stat-archived').textContent = archived;
  }

  // ---------- ფილტრების განახლება ----------
  function updateFilterOptions() {
    const doctors = [...new Set(allPatients.map(p => p.doctor).filter(Boolean))];
    const icds = [...new Set(allPatients.map(p => p.icd10_code).filter(Boolean))];
    const selDoc = document.getElementById('filter-doctor');
    const selIcd = document.getElementById('filter-icd');
    selDoc.innerHTML = '<option value="">ყველა ექიმი</option>' + doctors.map(d => `<option value="${d}">${d}</option>`).join('');
    selIcd.innerHTML = '<option value="">ყველა ICD-10</option>' + icds.map(i => `<option value="${i}">${i}</option>`).join('');
  }

  // ---------- Toast ----------
  function showToast(msg, type = 'success') {
    const t = document.getElementById('toast');
    t.textContent = msg;
    t.className = 'toast active ' + type;
    setTimeout(() => t.classList.remove('active'), 3000);
  }

  // ---------- Excel & PDF ექსპორტი (უცვლელი) ----------
  function exportToExcel(type) {
    const data = type === 'active'
      ? allPatients.filter(p => p.__collection === 'active')
      : allPatients.filter(p => p.__collection === 'archived');

    const rows = data.map(p => ({
      საწოლი: p.bed,
      პაციენტი: p.patient_name,
      ისტორია: p.history_number,
      'ICD-10': p.icd10_code || '',
      ექიმი: p.doctor || '',
      კომენტარი: p.comment || '',
      ჩარიცხვა: new Date(p.admission_date).toLocaleString('ka-GE'),
      ...(type === 'archive' && { წაშლა: new Date(p.deleted_date).toLocaleString('ka-GE') })
    }));

    const ws = XLSX.utils.json_to_sheet(rows);
    const wb = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, type === 'active' ? 'აქტიური' : 'არქივი');
    XLSX.writeFile(wb, `${type === 'active' ? 'აქტიური' : 'არქივი'}_${new Date().toLocaleDateString('ka-GE')}.xlsx`);
    showToast('Excel ჩამოიტვირთა');
  }

  function exportToPDF(type) {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    const data = type === 'active' ? allPatients.filter(p => p.__collection === 'active') : allPatients.filter(p => p.__collection === 'archived');
    doc.setFontSize(16);
    doc.text(type === 'active' ? 'აქტიური პაციენტები' : 'არქივი', 14, 15);
    let y = 30;
    doc.setFontSize(10);
    data.forEach((p, i) => {
      if (y > 270) { doc.addPage(); y = 20; }
      doc.text(`${i + 1}. ${p.patient_name} - საწოლი: ${p.bed}`, 14, y); y += 7;
      doc.text(`ისტორია: ${p.history_number} | ICD-10: ${p.icd10_code || 'N/A'}`, 14, y); y += 7;
      doc.text(`ექიმი: ${p.doctor || 'N/A'}`, 14, y); y += 10;
    });
    doc.save(`${type === 'active' ? 'active' : 'archive'}_${new Date().toLocaleDateString()}.pdf`);
    showToast('PDF ჩამოიტვირთა');
  }

  function exportSinglePatientPDF() {
    if (!currentViewPatient) return;
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    const p = currentViewPatient;
    doc.setFontSize(18); doc.text('პაციენტის ბარათი', 14, 20);
    let y = 35;
    const line = 10;
    doc.setFontSize(12);
    doc.text(`საწოლი: ${p.bed}`, 14, y); y += line;
    doc.text(`სახელი: ${p.patient_name}`, 14, y); y += line;
    doc.text(`ისტორია: ${p.history_number}`, 14, y); y += line;
    doc.text(`ICD-10: ${p.icd10_code || 'N/A'}`, 14, y); y += line;
    doc.text(`ექიმი: ${p.doctor || 'N/A'}`, 14, y); y += line;
    doc.text(`კომენტარი: ${p.comment || 'N/A'}`, 14, y); y += line;
    doc.text(`ჩარიცხვა: ${new Date(p.admission_date).toLocaleString('ka-GE')}`, 14, y);
    if (p.deleted_date) { y += line; doc.text(`წაშლა: ${new Date(p.deleted_date).toLocaleString('ka-GE')}`, 14, y); }
    doc.save(`patient_${p.history_number}.pdf`);
    showToast('PDF ჩამოიტვირთა');
  }

  // ---------- ფილტრები ----------
  document.getElementById('search').addEventListener('input', renderActivePatientsTable);
  document.getElementById('filter-bed').addEventListener('change', renderActivePatientsTable);
  document.getElementById('filter-doctor').addEventListener('change', renderActivePatientsTable);
  document.getElementById('filter-icd').addEventListener('change', renderActivePatientsTable);
  document.getElementById('archive-search').addEventListener('input', renderArchivedPatientsTable);
  document.getElementById('archive-filter-bed').addEventListener('change', renderArchivedPatientsTable);

  // ---------- ინიციალიზაცია ----------
  initializeApp();
</script>
</body>
</html>
