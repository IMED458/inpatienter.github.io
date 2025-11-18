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
      from {
        transform: translateX(400px);
        opacity: 0;
      }
      to {
        transform: translateX(0);
        opacity: 1;
      }
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

    .empty-state svg {
      width: 100px;
      height: 100px;
      margin-bottom: 20px;
      opacity: 0.5;
    }

    @media (max-width: 768px) {
      .header h1 {
        font-size: 1.8em;
      }

      .form-grid {
        grid-template-columns: 1fr;
      }

      .search-filter {
        flex-direction: column;
      }

      .search-filter input,
      .search-filter select {
        width: 100%;
      }

      table {
        font-size: 14px;
      }

      th, td {
        padding: 10px;
      }

      .action-buttons {
        flex-direction: column;
      }
    }
  </style>
  <style>@view-transition { navigation: auto; }</style>
  <script src="https://cdn.tailwindcss.com" type="text/javascript"></script>
 </head>
 <body>
  <div class="container">
   <div class="header">
    <h1 id="hospital-name">საწოლების მართვის სისტემა</h1>
    <p id="department-name">სტაციონარი / Inpatient</p>
   </div>
   <div class="tabs"><button class="tab-btn active" onclick="switchTab('active')">აქტიური პაციენტები</button> <button class="tab-btn" onclick="switchTab('archive')">არქივი</button> <button class="tab-btn" onclick="switchTab('statistics')">სტატისტიკა</button>
   </div>
   <div id="active-tab" class="tab-content active">
    <div class="card">
     <h2 id="add-patient-title">ახალი პაციენტის დამატება</h2>
     <form id="patient-form">
      <div class="form-grid">
       <div class="form-group"><label for="bed">საწოლი *</label> <select id="bed" required> <option value="">აირჩიეთ საწოლი</option> <option value="1">1</option> <option value="2">2</option> <option value="3">3</option> <option value="4">4</option> <option value="5">5</option> <option value="6">6</option> <option value="7">7</option> <option value="8">8</option> <option value="ლოჯი">ლოჯი</option> <option value="მცირე (3 საწოლი)">მცირე (3 საწოლი)</option> </select>
       </div>
       <div class="form-group"><label for="patient-name">პაციენტის სახელი და გვარი *</label> <input type="text" id="patient-name" required placeholder="მაგ: გიორგი გელაშვილი">
       </div>
       <div class="form-group"><label for="history-number">ისტორიის ნომერი *</label> <input type="text" id="history-number" required placeholder="მაგ: 12345">
       </div>
       <div class="form-group"><label for="icd10">ICD-10 კოდი</label> <input type="text" id="icd10" placeholder="მაგ: J18.9">
       </div>
       <div class="form-group"><label for="doctor">ექიმი</label> <input type="text" id="doctor" placeholder="მაგ: დ. ბერიძე">
       </div>
      </div>
      <div class="form-group"><label for="comment">კომენტარი</label> <textarea id="comment" placeholder="დამატებითი ინფორმაცია..."></textarea>
      </div><button type="submit" class="btn btn-primary" id="submit-btn"> <span id="submit-text">დამატება</span> </button>
     </form>
    </div>
    <div class="card">
     <h2>აქტიური პაციენტები</h2>
     <div class="search-filter"><input type="text" id="search" placeholder="🔍 ძებნა (სახელი, გვარი, ისტორია)"> <select id="filter-bed"> <option value="">ყველა საწოლი</option> <option value="1">1</option> <option value="2">2</option> <option value="3">3</option> <option value="4">4</option> <option value="5">5</option> <option value="6">6</option> <option value="7">7</option> <option value="8">8</option> <option value="ლოჯი">ლოჯი</option> <option value="მცირე (3 საწოლი)">მცირე (3 საწოლი)</option> </select> <select id="filter-doctor"> <option value="">ყველა ექიმი</option> </select> <select id="filter-icd"> <option value="">ყველა ICD-10</option> </select>
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
       <tbody id="active-tbody">
       </tbody>
      </table>
     </div>
     <div class="export-buttons"><button class="btn btn-success" onclick="exportToExcel('active')">📊 Excel ექსპორტი</button> <button class="btn btn-success" onclick="exportToPDF('active')">📄 PDF ექსპორტი</button>
     </div>
    </div>
   </div>
   <div id="archive-tab" class="tab-content">
    <div class="card">
     <h2>წაშლილი პაციენტების არქივი</h2>
     <div class="search-filter"><input type="text" id="archive-search" placeholder="🔍 ძებნა არქივში"> <select id="archive-filter-bed"> <option value="">ყველა საწოლი</option> <option value="1">1</option> <option value="2">2</option> <option value="3">3</option> <option value="4">4</option> <option value="5">5</option> <option value="6">6</option> <option value="7">7</option> <option value="8">8</option> <option value="ლოჯი">ლოჯი</option> <option value="მცირე (3 საწოლი)">მცირე (3 საწოლი)</option> </select>
     </div>
     <div class="table-container">
      <table id="archive-table">
       <thead>
        <tr>
         <th>საწოლი</th>
         <th>პაციენტი</th>
         <th>ისტორია</th>
         <th>ICD-10</th>
         <th>ექიმი</th>
         <th>კომენტარი</th>
         <th>ჩარიცხვა</th>
         <th>წაშლა</th>
         <th>მოქმედება</th>
        </tr>
       </thead>
       <tbody id="archive-tbody">
       </tbody>
      </table>
     </div>
     <div class="export-buttons"><button class="btn btn-success" onclick="exportToExcel('archive')">📊 Excel ექსპორტი</button> <button class="btn btn-success" onclick="exportToPDF('archive')">📄 PDF ექსპორტი</button>
     </div>
    </div>
   </div>
   <div id="statistics-tab" class="tab-content">
    <div class="card">
     <h2 id="statistics-title">დღიური სტატისტიკა</h2>
     <div class="stats-grid">
      <div class="stat-card">
       <h3>აქტიური პაციენტები</h3>
       <div class="number" id="stat-active">
        0
       </div>
      </div>
      <div class="stat-card">
       <h3>დღეს დამატებული</h3>
       <div class="number" id="stat-added-today">
        0
       </div>
      </div>
      <div class="stat-card">
       <h3>დღეს წაშლილი</h3>
       <div class="number" id="stat-deleted-today">
        0
       </div>
      </div>
      <div class="stat-card">
       <h3>სულ არქივში</h3>
       <div class="number" id="stat-archived">
        0
       </div>
      </div>
     </div>
    </div>
   </div>
  </div>
  <div id="edit-modal" class="modal">
   <div class="modal-content">
    <div class="modal-header">
     <h2>პაციენტის რედაქტირება</h2><button class="close-btn" onclick="closeEditModal()">×</button>
    </div>
    <form id="edit-form">
     <div class="form-grid">
      <div class="form-group"><label for="edit-bed">საწოლი *</label> <select id="edit-bed" required> <option value="1">1</option> <option value="2">2</option> <option value="3">3</option> <option value="4">4</option> <option value="5">5</option> <option value="6">6</option> <option value="7">7</option> <option value="8">8</option> <option value="ლოჯი">ლოჯი</option> <option value="მცირე (3 საწოლი)">მცირე (3 საწოლი)</option> </select>
      </div>
      <div class="form-group"><label for="edit-patient-name">პაციენტის სახელი და გვარი *</label> <input type="text" id="edit-patient-name" required>
      </div>
      <div class="form-group"><label for="edit-history-number">ისტორიის ნომერი *</label> <input type="text" id="edit-history-number" required>
      </div>
      <div class="form-group"><label for="edit-icd10">ICD-10 კოდი</label> <input type="text" id="edit-icd10">
      </div>
      <div class="form-group"><label for="edit-doctor">ექიმი</label> <input type="text" id="edit-doctor">
      </div>
     </div>
     <div class="form-group"><label for="edit-comment">კომენტარი</label> <textarea id="edit-comment"></textarea>
     </div>
     <div class="modal-actions"><button type="button" class="btn btn-secondary" onclick="closeEditModal()">გაუქმება</button> <button type="submit" class="btn btn-primary" id="edit-submit-btn"> <span id="edit-submit-text">შენახვა</span> </button>
     </div>
    </form>
   </div>
  </div>
  <div id="view-modal" class="modal">
   <div class="modal-content">
    <div class="modal-header">
     <h2>პაციენტის დეტალები</h2><button class="close-btn" onclick="closeViewModal()">×</button>
    </div>
    <div id="view-content"></div>
    <div class="modal-actions"><button type="button" class="btn btn-secondary" onclick="closeViewModal()">დახურვა</button> <button type="button" class="btn btn-success" onclick="exportSinglePatientPDF()">📄 PDF ექსპორტი</button>
    </div>
   </div>
  </div>
  <div id="delete-modal" class="modal">
   <div class="modal-content">
    <div class="modal-header">
     <h2>დადასტურება</h2><button class="close-btn" onclick="closeDeleteModal()">×</button>
    </div>
    <p style="margin: 20px 0;">დარწმუნებული ხართ, რომ გსურთ ამ პაციენტის წაშლა? ჩანაწერი გადავა არქივში.</p>
    <div class="modal-actions"><button type="button" class="btn btn-secondary" onclick="closeDeleteModal()">გაუქმება</button> <button type="button" class="btn btn-danger" id="confirm-delete-btn" onclick="confirmDelete()"> <span id="delete-text">წაშლა</span> </button>
    </div>
   </div>
  </div>
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

    const dataHandler = {
      onDataChanged(data) {
        allPatients = data;
        renderActivePatientsTable();
        renderArchivedPatientsTable();
        updateStatistics();
        updateFilterOptions();
      }
    };

    async function initializeApp() {
      const initResult = await window.dataSdk.init(dataHandler);
      if (!initResult.isOk) {
        showToast('სისტემის ინიციალიზაცია ვერ მოხერხდა', 'error');
      }
    }

    const elementApi = {
      defaultConfig,
      async onConfigChange(config) {
        document.getElementById('hospital-name').textContent = config.hospital_name || defaultConfig.hospital_name;
        document.getElementById('department-name').textContent = config.department_name || defaultConfig.department_name;
        document.getElementById('add-patient-title').textContent = config.add_patient_label || defaultConfig.add_patient_label;
        document.getElementById('statistics-title').textContent = config.statistics_label || defaultConfig.statistics_label;
      },
      mapToCapabilities: (config) => ({
        recolorables: [],
        borderables: [],
        fontEditable: undefined,
        fontSizeable: undefined
      }),
      mapToEditPanelValues: (config) => new Map([
        ['hospital_name', config.hospital_name || defaultConfig.hospital_name],
        ['department_name', config.department_name || defaultConfig.department_name],
        ['add_patient_label', config.add_patient_label || defaultConfig.add_patient_label],
        ['statistics_label', config.statistics_label || defaultConfig.statistics_label]
      ])
    };

    if (window.elementSdk) {
      window.elementSdk.init(elementApi);
    }

    document.getElementById('patient-form').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      if (allPatients.filter(p => p.__collection === 'active').length >= 999) {
        showToast('მიღწეულია პაციენტების მაქსიმალური რაოდენობა (999)', 'error');
        return;
      }

      const submitBtn = document.getElementById('submit-btn');
      const submitText = document.getElementById('submit-text');
      submitBtn.disabled = true;
      submitText.innerHTML = '<span class="loading"></span>';

      const newPatient = {
        __collection: 'active',
        bed: document.getElementById('bed').value,
        patient_name: document.getElementById('patient-name').value,
        history_number: document.getElementById('history-number').value,
        icd10_code: document.getElementById('icd10').value,
        doctor: document.getElementById('doctor').value,
        comment: document.getElementById('comment').value,
        admission_date: new Date().toISOString(),
        deleted_date: ''
      };

      const result = await window.dataSdk.create(newPatient);
      
      submitBtn.disabled = false;
      submitText.textContent = 'დამატება';

      if (result.isOk) {
        showToast('პაციენტი წარმატებით დაემატა');
        document.getElementById('patient-form').reset();
      } else {
        showToast('შეცდომა პაციენტის დამატებისას', 'error');
      }
    });

    document.getElementById('edit-form').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      if (!currentEditPatient) return;

      const editBtn = document.getElementById('edit-submit-btn');
      const editText = document.getElementById('edit-submit-text');
      editBtn.disabled = true;
      editText.innerHTML = '<span class="loading"></span>';

      const updatedPatient = {
        ...currentEditPatient,
        bed: document.getElementById('edit-bed').value,
        patient_name: document.getElementById('edit-patient-name').value,
        history_number: document.getElementById('edit-history-number').value,
        icd10_code: document.getElementById('edit-icd10').value,
        doctor: document.getElementById('edit-doctor').value,
        comment: document.getElementById('edit-comment').value
      };

      const result = await window.dataSdk.update(updatedPatient);
      
      editBtn.disabled = false;
      editText.textContent = 'შენახვა';

      if (result.isOk) {
        showToast('ცვლილებები შენახულია');
        closeEditModal();
      } else {
        showToast('შეცდომა შენახვისას', 'error');
      }
    });

    function openEditModal(patient) {
      currentEditPatient = patient;
      document.getElementById('edit-bed').value = patient.bed;
      document.getElementById('edit-patient-name').value = patient.patient_name;
      document.getElementById('edit-history-number').value = patient.history_number;
      document.getElementById('edit-icd10').value = patient.icd10_code;
      document.getElementById('edit-doctor').value = patient.doctor;
      document.getElementById('edit-comment').value = patient.comment;
      document.getElementById('edit-modal').classList.add('active');
    }

    function closeEditModal() {
      currentEditPatient = null;
      document.getElementById('edit-modal').classList.remove('active');
    }

    function openDeleteModal(patient) {
      currentDeletePatient = patient;
      document.getElementById('delete-modal').classList.add('active');
    }

    function closeDeleteModal() {
      currentDeletePatient = null;
      document.getElementById('delete-modal').classList.remove('active');
    }

    async function confirmDelete() {
      if (!currentDeletePatient) return;

      const deleteBtn = document.getElementById('confirm-delete-btn');
      const deleteText = document.getElementById('delete-text');
      deleteBtn.disabled = true;
      deleteText.innerHTML = '<span class="loading"></span>';

      const archivedPatient = {
        ...currentDeletePatient,
        __collection: 'archived',
        deleted_date: new Date().toISOString()
      };

      const updateResult = await window.dataSdk.update(archivedPatient);
      
      deleteBtn.disabled = false;
      deleteText.textContent = 'წაშლა';

      if (updateResult.isOk) {
        showToast('პაციენტი გადატანილია არქივში');
        closeDeleteModal();
      } else {
        showToast('შეცდომა არქივში გადატანისას', 'error');
      }
    }

    function openViewModal(patient) {
      currentViewPatient = patient;
      const content = document.getElementById('view-content');
      const admissionDate = patient.admission_date ? new Date(patient.admission_date).toLocaleString('ka-GE') : 'N/A';
      const deletedDate = patient.deleted_date ? new Date(patient.deleted_date).toLocaleString('ka-GE') : 'N/A';
      
      content.innerHTML = `
        <div style="line-height: 2;">
          <p><strong>საწოლი:</strong> ${patient.bed}</p>
          <p><strong>პაციენტი:</strong> ${patient.patient_name}</p>
          <p><strong>ისტორიის ნომერი:</strong> ${patient.history_number}</p>
          <p><strong>ICD-10:</strong> ${patient.icd10_code || 'N/A'}</p>
          <p><strong>ექიმი:</strong> ${patient.doctor || 'N/A'}</p>
          <p><strong>კომენტარი:</strong> ${patient.comment || 'N/A'}</p>
          <p><strong>ჩარიცხვის თარიღი:</strong> ${admissionDate}</p>
          ${patient.__collection === 'archived' ? `<p><strong>წაშლის თარიღი:</strong> ${deletedDate}</p>` : ''}
        </div>
      `;
      document.getElementById('view-modal').classList.add('active');
    }

    function closeViewModal() {
      currentViewPatient = null;
      document.getElementById('view-modal').classList.remove('active');
    }

    function renderActivePatientsTable() {
      const tbody = document.getElementById('active-tbody');
      const search = document.getElementById('search').value.toLowerCase();
      const filterBed = document.getElementById('filter-bed').value;
      const filterDoctor = document.getElementById('filter-doctor').value;
      const filterIcd = document.getElementById('filter-icd').value;

      let filtered = allPatients.filter(p => p.__collection === 'active');

      if (search) {
        filtered = filtered.filter(p => 
          p.patient_name.toLowerCase().includes(search) ||
          p.history_number.toLowerCase().includes(search)
        );
      }

      if (filterBed) {
        filtered = filtered.filter(p => p.bed === filterBed);
      }

      if (filterDoctor) {
        filtered = filtered.filter(p => p.doctor === filterDoctor);
      }

      if (filterIcd) {
        filtered = filtered.filter(p => p.icd10_code === filterIcd);
      }

      filtered.sort((a, b) => {
        let aVal = a[currentSortColumn] || '';
        let bVal = b[currentSortColumn] || '';
        
        if (currentSortColumn === 'admission_date') {
          aVal = new Date(aVal).getTime();
          bVal = new Date(bVal).getTime();
        }

        if (aVal < bVal) return currentSortDirection === 'asc' ? -1 : 1;
        if (aVal > bVal) return currentSortDirection === 'asc' ? 1 : -1;
        return 0;
      });

      if (filtered.length === 0) {
        tbody.innerHTML = `
          <tr>
            <td colspan="8" class="empty-state">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor">
                <path d="M9 11l3 3L22 4"></path>
                <path d="M21 12v7a2 2 0 01-2 2H5a2 2 0 01-2-2V5a2 2 0 012-2h11"></path>
              </svg>
              <p>პაციენტები არ არის</p>
            </td>
          </tr>
        `;
        return;
      }

      tbody.innerHTML = filtered.map(patient => {
        const admissionDate = new Date(patient.admission_date).toLocaleDateString('ka-GE');
        return `
          <tr>
            <td>${patient.bed}</td>
            <td>${patient.patient_name}</td>
            <td>${patient.history_number}</td>
            <td>${patient.icd10_code || 'N/A'}</td>
            <td>${patient.doctor || 'N/A'}</td>
            <td>${patient.comment || 'N/A'}</td>
            <td>${admissionDate}</td>
            <td>
              <div class="action-buttons">
                <button class="btn btn-primary" onclick='openViewModal(${JSON.stringify(patient).replace(/'/g, "&apos;")})'>ნახვა</button>
                <button class="btn btn-secondary" onclick='openEditModal(${JSON.stringify(patient).replace(/'/g, "&apos;")})'>რედაქტირება</button>
                <button class="btn btn-danger" onclick='openDeleteModal(${JSON.stringify(patient).replace(/'/g, "&apos;")})'>წაშლა</button>
              </div>
            </td>
          </tr>
        `;
      }).join('');
    }

    function renderArchivedPatientsTable() {
      const tbody = document.getElementById('archive-tbody');
      const search = document.getElementById('archive-search').value.toLowerCase();
      const filterBed = document.getElementById('archive-filter-bed').value;

      let filtered = allPatients.filter(p => p.__collection === 'archived');

      if (search) {
        filtered = filtered.filter(p => 
          p.patient_name.toLowerCase().includes(search) ||
          p.history_number.toLowerCase().includes(search)
        );
      }

      if (filterBed) {
        filtered = filtered.filter(p => p.bed === filterBed);
      }

      if (filtered.length === 0) {
        tbody.innerHTML = `
          <tr>
            <td colspan="9" class="empty-state">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor">
                <path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"></path>
                <circle cx="12" cy="7" r="4"></circle>
              </svg>
              <p>არქივი ცარიელია</p>
            </td>
          </tr>
        `;
        return;
      }

      tbody.innerHTML = filtered.map(patient => {
        const admissionDate = new Date(patient.admission_date).toLocaleDateString('ka-GE');
        const deletedDate = new Date(patient.deleted_date).toLocaleDateString('ka-GE');
        return `
          <tr>
            <td>${patient.bed}</td>
            <td>${patient.patient_name}</td>
            <td>${patient.history_number}</td>
            <td>${patient.icd10_code || 'N/A'}</td>
            <td>${patient.doctor || 'N/A'}</td>
            <td>${patient.comment || 'N/A'}</td>
            <td>${admissionDate}</td>
            <td>${deletedDate}</td>
            <td>
              <button class="btn btn-primary" onclick='openViewModal(${JSON.stringify(patient).replace(/'/g, "&apos;")})'>ნახვა</button>
            </td>
          </tr>
        `;
      }).join('');
    }

    function updateStatistics() {
      const active = allPatients.filter(p => p.__collection === 'active');
      const archived = allPatients.filter(p => p.__collection === 'archived');
      
      const today = new Date().toDateString();
      const addedToday = active.filter(p => new Date(p.admission_date).toDateString() === today).length;
      const deletedToday = archived.filter(p => new Date(p.deleted_date).toDateString() === today).length;

      document.getElementById('stat-active').textContent = active.length;
      document.getElementById('stat-added-today').textContent = addedToday;
      document.getElementById('stat-deleted-today').textContent = deletedToday;
      document.getElementById('stat-archived').textContent = archived.length;
    }

    function updateFilterOptions() {
      const doctors = [...new Set(allPatients.map(p => p.doctor).filter(d => d))];
      const icds = [...new Set(allPatients.map(p => p.icd10_code).filter(i => i))];

      const doctorSelect = document.getElementById('filter-doctor');
      const icdSelect = document.getElementById('filter-icd');

      doctorSelect.innerHTML = '<option value="">ყველა ექიმი</option>' + 
        doctors.map(d => `<option value="${d}">${d}</option>`).join('');

      icdSelect.innerHTML = '<option value="">ყველა ICD-10</option>' + 
        icds.map(i => `<option value="${i}">${i}</option>`).join('');
    }

    function sortTable(column) {
      if (currentSortColumn === column) {
        currentSortDirection = currentSortDirection === 'asc' ? 'desc' : 'asc';
      } else {
        currentSortColumn = column;
        currentSortDirection = 'asc';
      }
      renderActivePatientsTable();
    }

    function switchTab(tab) {
      document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
      document.querySelectorAll('.tab-content').forEach(content => content.classList.remove('active'));
      
      if (tab === 'active') {
        document.querySelector('.tab-btn:nth-child(1)').classList.add('active');
        document.getElementById('active-tab').classList.add('active');
      } else if (tab === 'archive') {
        document.querySelector('.tab-btn:nth-child(2)').classList.add('active');
        document.getElementById('archive-tab').classList.add('active');
      } else if (tab === 'statistics') {
        document.querySelector('.tab-btn:nth-child(3)').classList.add('active');
        document.getElementById('statistics-tab').classList.add('active');
      }
    }

    function showToast(message, type = 'success') {
      const toast = document.getElementById('toast');
      toast.textContent = message;
      toast.className = `toast ${type} active`;
      setTimeout(() => {
        toast.classList.remove('active');
      }, 3000);
    }

    function exportToExcel(type) {
      const data = type === 'active' 
        ? allPatients.filter(p => p.__collection === 'active')
        : allPatients.filter(p => p.__collection === 'archived');

      const exportData = data.map(p => ({
        'საწოლი': p.bed,
        'პაციენტი': p.patient_name,
        'ისტორია': p.history_number,
        'ICD-10': p.icd10_code || '',
        'ექიმი': p.doctor || '',
        'კომენტარი': p.comment || '',
        'ჩარიცხვა': new Date(p.admission_date).toLocaleString('ka-GE'),
        ...(type === 'archive' && { 'წაშლა': new Date(p.deleted_date).toLocaleString('ka-GE') })
      }));

      const ws = XLSX.utils.json_to_sheet(exportData);
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, type === 'active' ? 'აქტიური' : 'არქივი');
      XLSX.writeFile(wb, `${type === 'active' ? 'აქტიური_პაციენტები' : 'არქივი'}_${new Date().toLocaleDateString('ka-GE')}.xlsx`);
      showToast('Excel ფაილი ჩამოიტვირთა');
    }

    function exportToPDF(type) {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      
      const data = type === 'active' 
        ? allPatients.filter(p => p.__collection === 'active')
        : allPatients.filter(p => p.__collection === 'archived');

      doc.setFontSize(16);
      doc.text(type === 'active' ? 'Active Patients' : 'Archived Patients', 14, 15);
      
      let y = 30;
      doc.setFontSize(10);
      
      data.forEach((p, index) => {
        if (y > 270) {
          doc.addPage();
          y = 20;
        }
        
        doc.text(`${index + 1}. ${p.patient_name} - Bed: ${p.bed}`, 14, y);
        y += 7;
        doc.text(`   History: ${p.history_number} | ICD-10: ${p.icd10_code || 'N/A'}`, 14, y);
        y += 7;
        doc.text(`   Doctor: ${p.doctor || 'N/A'}`, 14, y);
        y += 10;
      });
      
      doc.save(`${type === 'active' ? 'active_patients' : 'archive'}_${new Date().toLocaleDateString()}.pdf`);
      showToast('PDF ფაილი ჩამოიტვირთა');
    }

    function exportSinglePatientPDF() {
      if (!currentViewPatient) return;

      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      const p = currentViewPatient;
      
      doc.setFontSize(18);
      doc.text('Patient Record', 14, 20);
      
      doc.setFontSize(12);
      let y = 35;
      const lineHeight = 10;
      
      doc.text(`Bed: ${p.bed}`, 14, y);
      y += lineHeight;
      doc.text(`Patient Name: ${p.patient_name}`, 14, y);
      y += lineHeight;
      doc.text(`History Number: ${p.history_number}`, 14, y);
      y += lineHeight;
      doc.text(`ICD-10: ${p.icd10_code || 'N/A'}`, 14, y);
      y += lineHeight;
      doc.text(`Doctor: ${p.doctor || 'N/A'}`, 14, y);
      y += lineHeight;
      doc.text(`Comment: ${p.comment || 'N/A'}`, 14, y);
      y += lineHeight;
      doc.text(`Admission Date: ${new Date(p.admission_date).toLocaleString('ka-GE')}`, 14, y);
      
      if (p.__collection === 'archived') {
        y += lineHeight;
        doc.text(`Deleted Date: ${new Date(p.deleted_date).toLocaleString('ka-GE')}`, 14, y);
      }
      
      doc.save(`patient_${p.history_number}.pdf`);
      showToast('PDF ფაილი ჩამოიტვირთა');
    }

    document.getElementById('search').addEventListener('input', renderActivePatientsTable);
    document.getElementById('filter-bed').addEventListener('change', renderActivePatientsTable);
    document.getElementById('filter-doctor').addEventListener('change', renderActivePatientsTable);
    document.getElementById('filter-icd').addEventListener('change', renderActivePatientsTable);
    document.getElementById('archive-search').addEventListener('input', renderArchivedPatientsTable);
    document.getElementById('archive-filter-bed').addEventListener('change', renderArchivedPatientsTable);

    initializeApp();
  </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9a05113a408e26fa',t:'MTc2MzQ0MzczNy4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
