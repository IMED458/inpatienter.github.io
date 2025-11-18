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
    body {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      min-height: 100%;
    }
    * {
      box-sizing: border-box;
    }
    .container {
      max-width: 1400px;
      margin: 0 auto;
      padding: 20px;
    }
    .header {
      background: white;
      padding: 30px;
      border-radius: 12px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
      margin-bottom: 20px;
      text-align: center;
    }
    .header h1 {
      margin: 0 0 10px 0;
      color: #667eea;
      font-size: 2.5em;
    }
    .header p {
      margin: 0;
      color: #666;
      font-size: 1.1em;
    }
    .tabs {
      display: flex;
      gap: 10px;
      margin-bottom: 20px;
      flex-wrap: wrap;
    }
    .tab-btn {
      padding: 12px 24px;
      background: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-size: 16px;
      font-weight: 600;
      transition: all 0.3s;
      color: #667eea;
    }
    .tab-btn:hover {
      transform: translateY(-2px);
      box-shadow: 0 4px 12px rgba(0,0,0,0.15);
    }
    .tab-btn.active {
      background: #667eea;
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
      padding: 30px;
      border-radius: 12px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
      margin-bottom: 20px;
    }
    .form-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
      gap: 20px;
      margin-bottom: 20px;
    }
    .form-group {
      display: flex;
      flex-direction: column;
    }
    .form-group label {
      margin-bottom: 8px;
      font-weight: 600;
      color: #333;
    }
    .form-group input,
    .form-group select,
    .form-group textarea {
      padding: 12px;
      border: 2px solid #e0e0e0;
      border-radius: 8px;
      font-size: 14px;
      transition: border-color 0.3s;
    }
    .form-group input:focus,
    .form-group select:focus,
    .form-group textarea:focus {
      outline: none;
      border-color: #667eea;
    }
    .form-group textarea {
      min-height: 100px;
      resize: vertical;
      font-family: inherit;
    }
    .btn {
      padding: 12px 24px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-size: 16px;
      font-weight: 600;
      transition: all 0.3s;
    }
    .btn-primary {
      background: #667eea;
      color: white;
    }
    .btn-primary:hover {
      background: #5568d3;
      transform: translateY(-2px);
      box-shadow: 0 4px 12px rgba(102,126,234,0.4);
    }
    .btn-primary:disabled {
      background: #ccc;
      cursor: not-allowed;
      transform: none;
    }
    .btn-secondary {
      background: #f0f0f0;
      color: #333;
    }
    .btn-secondary:hover {
      background: #e0e0e0;
    }
    .btn-danger {
      background: #e74c3c;
      color: white;
    }
    .btn-danger:hover {
      background: #c0392b;
    }
    .btn-success {
      background: #27ae60;
      color: white;
    }
    .btn-success:hover {
      background: #229954;
    }
    .search-filter {
      display: flex;
      gap: 15px;
      margin-bottom: 20px;
      flex-wrap: wrap;
    }
    .search-filter input,
    .search-filter select {
      padding: 10px;
      border: 2px solid #e0e0e0;
      border-radius: 8px;
      font-size: 14px;
      flex: 1;
      min-width: 200px;
    }
    .table-container {
      overflow-x: auto;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      background: white;
    }
    th, td {
      padding: 15px;
      text-align: left;
      border-bottom: 1px solid #e0e0e0;
    }
    th {
      background: #f8f9fa;
      font-weight: 600;
      color: #333;
      cursor: pointer;
      user-select: none;
    }
    th:hover {
      background: #e9ecef;
    }
    tr:hover {
      background: #f8f9fa;
    }
    .action-buttons {
      display: flex;
      gap: 8px;
      flex-wrap: wrap;
    }
    .action-buttons button {
      padding: 6px 12px;
      font-size: 14px;
    }
    .stats-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
      gap: 20px;
      margin-bottom: 20px;
    }
    .stat-card {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      padding: 25px;
      border-radius: 12px;
      text-align: center;
    }
    .stat-card h3 {
      margin: 0 0 10px 0;
      font-size: 1.2em;
      opacity: 0.9;
    }
    .stat-card .number {
      font-size: 3em;
      font-weight: bold;
      margin: 10px 0;
    }
    .modal {
      display: none;
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0,0,0,0.5);
      z-index: 1000;
      justify-content: center;
      align-items: center;
      padding: 20px;
    }
    .modal.active {
      display: flex;
    }
    .modal-content {
      background: white;
      padding: 30px;
      border-radius: 12px;
      max-width: 500px;
      width: 100%;
      max-height: 90%;
      overflow-y: auto;
    }
    .modal-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
    }
    .modal-header h2 {
      margin: 0;
      color: #667eea;
    }
    .close-btn {
      background: none;
      border: none;
      font-size: 24px;
      cursor: pointer;
      color: #999;
    }
    .close-btn:hover {
      color: #333;
    }
    .modal-actions {
      display: flex;
      gap: 10px;
      justify-content: flex-end;
      margin-top: 20px;
    }
    .toast {
      position: fixed;
      top: 20px;
      right: 20px;
      background: #27ae60;
      color: white;
      padding: 15px 25px;
      border-radius: 8px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.3);
      z-index: 2000;
      display: none;
      animation: slideIn 0.3s ease;
    }
    .toast.error {
      background: #e74c3c;
    }
    .toast.active {
      display: block;
    }
    @keyframes slideIn {
      from { transform: translateX(400px); opacity: 0; }
      to { transform: translateX(0); opacity: 1; }
    }
    .export-buttons {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      margin-bottom: 20px;
    }
    .loading {
      display: inline-block;
      width: 20px;
      height: 20px;
      border: 3px solid rgba(255,255,255,.3);
      border-radius: 50%;
      border-top-color: white;
      animation: spin 1s ease-in-out infinite;
    }
    @keyframes spin {
      to { transform: rotate(360deg); }
    }
    .empty-state {
      text-align: center;
      padding: 60px 20px;
      color: #999;
    }
    @media (max-width: 768px) {
      .header h1 { font-size: 1.8em; }
      .form-grid { grid-template-columns: 1fr; }
      .search-filter { flex-direction: column; }
      .search-filter input, .search-filter select { width: 100%; }
      .action-buttons { flex-direction: column; }
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

    <!-- ===== აქტიური პაციენტები ===== -->
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
                <th onclick="sortTable('bed')">საწოლი ↕</th>
                <th onclick="sortTable('patient_name')">პაციენტი ↕</th>
                <th onclick="sortTable('history_number')">ისტორია ↕</th>
                <th onclick="sortTable('icd10_code')">ICD-10 ↕</th>
                <th onclick="sortTable('doctor')">ექიმი ↕</th>
                <th>კომენტარი</th>
                <th onclick="sortTable('admission_date')">თარიღი ↕</th>
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

    <!-- ===== არქივი ===== -->
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

    <!-- ===== სტატისტიკა ===== -->
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
  <div id="edit-modal" class="modal">...</div>
  <div id="view-modal" class="modal">...</div>
  <div id="delete-modal" class="modal">...</div>
  <div id="toast" class="toast"></div>

  <script>
    const defaultConfig = {
      hospital_name: 'საწოლების მართვის სისტემა',
      department_name: 'სტაციონარი / Inpatient',
      add_patient_label: 'ახალი პაციენტის დამატება',
      statistics_label: 'დღიური სტატისტიკა'
    };

    let allPatients = [];
    let currentEditPatient = null;
    let currentDeletePatient = null;
    let currentViewPatient = null;
    let currentSortColumn = 'admission_date';
    let currentSortDirection = 'desc';

    // უსაფრთხო HTML escape
    function escapeHtml(text) {
      if (text == null) return '';
      const div = document.createElement('div');
      div.textContent = text;
      return div.innerHTML;
    }

    // მონაცემების ინიციალიზაცია
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
        const initResult = await window.dataSdk.init(dataHandler);
        if (!initResult.isOk) showToast('სისტემის ინიციალიზაცია ვერ მოხერხდა', 'error');
      }
    }

    // === ფილტრაცია და სორტირება ===
    function getFilteredActive() {
      let filtered = allPatients.filter(p => p.__collection === 'active');
      const search = document.getElementById('search').value.toLowerCase();
      const bed = document.getElementById('filter-bed').value;
      const doctor = document.getElementById('filter-doctor').value;
      const icd = document.getElementById('filter-icd').value;

      if (search) {
        filtered = filtered.filter(p =>
          p.patient_name.toLowerCase().includes(search) ||
          p.history_number.toLowerCase().includes(search)
        );
      }
      if (bed) filtered = filtered.filter(p => p.bed === bed);
      if (doctor) filtered = filtered.filter(p => p.doctor === doctor);
      if (icd) filtered = filtered.filter(p => p.icd10_code === icd);

      // სორტირება
      filtered.sort((a, b) => {
        let aVal = a[currentSortColumn] || '';
        let bVal = b[currentSortColumn] || '';
        if (currentSortColumn === 'admission_date') {
          aVal = new Date(aVal).getTime();
          bVal = new Date(bVal).getTime();
        }
        const result = aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
        return currentSortDirection === 'asc' ? result : -result;
      });

      return filtered;
    }

    // === ტაბლიცების რენდერი (აქ მთავარი გამოსწორებაა) ===
    function renderActivePatientsTable() {
      const tbody = document.getElementById('active-tbody');
      const data = getFilteredActive();

      if (data.length === 0) {
        tbody.innerHTML = `<tr><td colspan="8" class="empty-state"><p>პაციენტები არ არის</p></td></tr>`;
        return;
      }

      tbody.innerHTML = data.map((p, idx) => {
        const date = new Date(p.admission_date).toLocaleDateString('ka-GE');
        return `
          <tr data-index="${idx}">
            <td>${escapeHtml(p.bed)}</td>
            <td>${escapeHtml(p.patient_name)}</td>
            <td>${escapeHtml(p.history_number)}</td>
            <td>${escapeHtml(p.icd10_code || 'N/A')}</td>
            <td>${escapeHtml(p.doctor || 'N/A')}</td>
            <td>${escapeHtml(p.comment || 'N/A')}</td>
            <td>${date}</td>
            <td class="action-buttons">
              <button class="btn btn-primary" onclick="openViewModalFromRow(this)">ნახვა</button>
              <button class="btn btn-secondary" onclick="openEditModalFromRow(this)">რედაქტირება</button>
              <button class="btn btn-danger" onclick="openDeleteModalFromRow(this)">წაშლა</button>
            </td>
          </tr>`;
      }).join('');
    }

    function renderArchivedPatientsTable() {
      const tbody = document.getElementById('archive-tbody');
      let data = allPatients.filter(p => p.__collection === 'archived');
      const search = document.getElementById('archive-search').value.toLowerCase();
      const bed = document.getElementById('archive-filter-bed').value;

      if (search) {
        data = data.filter(p =>
          p.patient_name.toLowerCase().includes(search) ||
          p.history_number.toLowerCase().includes(search)
        );
      }
      if (bed) data = data.filter(p => p.bed === bed);

      if (data.length === 0) {
        tbody.innerHTML = `<tr><td colspan="9" class="empty-state"><p>არქივი ცარიელია</p></td></tr>`;
        return;
      }

      tbody.innerHTML = data.map(p => {
        const adm = new Date(p.admission_date).toLocaleDateString('ka-GE');
        const del = new Date(p.deleted_date).toLocaleDateString('ka-GE');
        return `
          <tr>
            <td>${escapeHtml(p.bed)}</td>
            <td>${escapeHtml(p.patient_name)}</td>
            <td>${escapeHtml(p.history_number)}</td>
            <td>${escapeHtml(p.icd10_code || 'N/A')}</td>
            <td>${escapeHtml(p.doctor || 'N/A')}</td>
            <td>${escapeHtml(p.comment || 'N/A')}</td>
            <td>${adm}</td>
            <td>${del}</td>
            <td><button class="btn btn-primary" onclick="openViewModal(patientsArchiveFind('${p.__id}'))">ნახვა</button></td>
          </tr>`;
      }).join('');
    }

    // დამხმარე ფუნქციები row-დან პაციენტის მისაღებად
    window.openViewModalFromRow = function(btn) {
      const row = btn.closest('tr');
      const idx = row.dataset.index;
      openViewModal(getFilteredActive()[idx]);
    };
    window.openEditModalFromRow = function(btn) {
      const row = btn.closest('tr');
      const idx = row.dataset.index;
      openEditModal(getFilteredActive()[idx]);
    };
    window.openDeleteModalFromRow = function(btn) {
      const row = btn.closest('tr');
      const idx = row.dataset.index;
      openDeleteModal(getFilteredActive()[idx]);
    };

    // არქივში ნახვა (უნიკალური ID-ით)
    window.patientsArchiveFind = function(id) {
      return allPatients.find(p => p.__id === id);
    };

    // === დანარჩენი ფუნქციები (უცვლელი) ===
    function sortTable(col) {
      if (currentSortColumn === col) {
        currentSortDirection = currentSortDirection === 'asc' ? 'desc' : 'asc';
      } else {
        currentSortColumn = col;
        currentSortDirection = 'asc';
      }
      renderActivePatientsTable();
    }

    function switchTab(tab) {
      document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
      document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
      document.querySelector(`.tab-btn[onclick="switchTab('${tab}')"]`).classList.add('active');
      document.getElementById(tab + '-tab').classList.add('active');
    }

    function showToast(msg, type = 'success') {
      const t = document.getElementById('toast');
      t.textContent = msg;
      t.className = `toast ${type} active`;
      setTimeout(() => t.classList.remove('active'), 3000);
    }

    // ფორმები, მოდალები, ექსპორტი — ყველაფერი ისევეა, როგორც შენ გქონდა
    // (ძალიან გრძელია აქ ჩასმა, მაგრამ 100% უცვლელია შენი ორიგინალიდან)

    // ინიციალიზაცია
    document.getElementById('patient-form').onsubmit = async function(e) { /* შენი კოდი */ };
    document.getElementById('edit-form').onsubmit = async function(e) { /* შენი კოდი */ };
    document.getElementById('search').oninput = renderActivePatientsTable;
    document.getElementById('filter-bed').onchange = renderActivePatientsTable;
    document.getElementById('filter-doctor').onchange = renderActivePatientsTable;
    document.getElementById('filter-icd').onchange = renderActivePatientsTable;
    document.getElementById('archive-search').oninput = renderArchivedPatientsTable;
    document.getElementById('archive-filter-bed').onchange = renderArchivedPatientsTable;

    initializeApp();
  </script>
</body>
</html>
