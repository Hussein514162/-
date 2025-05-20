# -
مشروع
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>حساب المصاريف الشهرية المتقدم</title>
<style>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    max-width: 700px;
    margin: 20px auto;
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: #fff;
    padding: 20px;
    border-radius: 12px;
    box-shadow: 0 10px 25px rgba(0,0,0,0.2);
  }

  h2, h3 {
    text-align: center;
    text-shadow: 1px 1px 4px rgba(0,0,0,0.3);
  }

  label, button {
    display: block;
    margin-top: 10px;
  }

  select, input[type="text"], input[type="number"], input[type="date"] {
    padding: 8px;
    width: 100%;
    box-sizing: border-box;
    border-radius: 6px;
    border: none;
    font-size: 14px;
  }

  button.btn {
    cursor: pointer;
    background-color: #9f7aea;
    color: white;
    font-weight: bold;
    border: none;
    padding: 10px;
    border-radius: 8px;
    transition: background-color 0.3s ease;
  }

  button.btn:hover {
    background-color: #7c3aed;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
    background-color: rgba(255, 255, 255, 0.15);
    border-radius: 10px;
    overflow: hidden;
  }

  th, td {
    border: none;
    padding: 10px;
    text-align: center;
    color: #fff;
  }

  th {
    background-color: rgba(0,0,0,0.25);
  }

  button.action-btn {
    background: none;
    border: none;
    color: #fff;
    font-size: 18px;
    margin: 0 5px;
  }

  button.action-btn:hover {
    color: #d1c4e9;
  }

  #filters {
    margin-top: 20px;
    display: flex;
    gap: 10px;
  }

  #filters > * {
    flex: 1;
  }

  #chart-container {
    margin-top: 30px;
    background-color: rgba(255, 255, 255, 0.15);
    padding: 15px;
    border-radius: 10px;
  }

  #export-btn {
    margin-top: 15px;
  }
</style>
</head>
<body>

<h2>حساب المصاريف الشهرية المتقدم</h2>

<form id="form-مصروفات">
  <label>النوع:
    <select id="input-نوع" required>
      <option value="مصروف">مصروف</option>
      <option value="دخل">دخل</option>
    </select>
  </label>

  <label>التصنيف:
    <select id="input-تصنيف" required>
      <option value="طعام">طعام</option>
      <option value="مواصلات">مواصلات</option>
      <option value="إيجار">إيجار</option>
      <option value="راتب">راتب</option>
      <option value="ترفيه">ترفيه</option>
      <option value="أخرى">أخرى</option>
    </select>
    <input type="text" id="input-تصنيف-جديد" placeholder="أضف تصنيف جديد..." style="margin-top:5px; display:none;" />
    <button type="button" id="btn-إضافة-تصنيف" style="margin-top:5px; display:none;">➕ إضافة تصنيف</button>
  </label>

  <label>القيمة:
    <input id="input-قيمة" type="number" step="0.01" min="0" required />
  </label>

  <label>التاريخ:
    <input id="input-تاريخ" type="date" required />
  </label>

  <button type="submit" class="btn">➕ إضافة</button>
</form>

<div id="filters">
  <select id="فلتر-نوع">
    <option value="الكل">الكل</option>
    <option value="مصروف">مصروف</option>
    <option value="دخل">دخل</option>
  </select>

  <select id="فلتر-تصنيف">
    <option value="الكل">الكل</option>
  </select>

  <input type="date" id="فلتر-تاريخ-من" />
  <input type="date" id="فلتر-تاريخ-إلى" />
</div>

<button id="export-btn" class="btn">⬇️ تصدير CSV</button>

<h3>قائمة المصاريف</h3>
<table id="جدول-مصروفات">
  <thead>
    <tr>
      <th>النوع</th>
      <th>التصنيف</th>
      <th>القيمة</th>
      <th>التاريخ</th>
      <th>الإجراءات</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<h3>الإحصائيات</h3>
<div>
  <p id="إجمالي-الدخل">💰 الدخل: 0</p>
  <p id="إجمالي-المصاريف">🧾 المصاريف: 0</p>
  <p id="الصافي">📊 الصافي: 0</p>
</div>

<div id="chart-container">
  <canvas id="chart"></canvas>
</div>

<!-- مكتبة Chart.js -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
  const STORAGE_KEY = "مصروفات_شهرية_متقدم";
  let بيانات = [];
  let تصنيفات = ["طعام", "مواصلات", "إيجار", "راتب", "ترفيه", "أخرى"];
  let تعديلIndex = null; // لتتبع العنصر عند التعديل
  let chart = null;

  // تحميل البيانات من التخزين
  function جلب_البيانات() {
    const data = localStorage.getItem(STORAGE_KEY);
    return data ? JSON.parse(data) : [];
  }

  // حفظ البيانات في التخزين
  function حفظ_البيانات() {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(بيانات));
  }

  // تحديث قائمة التصنيفات في الاختيار مع إضافة خيار "أخرى"
  function تحديث_التصنيفات() {
    const select = document.getElementById("input-تصنيف");
    const filterSelect = document.getElementById("فلتر-تصنيف");
    select.innerHTML = "";
    filterSelect.innerHTML = `<option value="الكل">الكل</option>`;

    تصنيفات.forEach(cat => {
      const option1 = document.createElement("option");
      option1.value = cat;
      option1.textContent = cat;
      select.appendChild(option1);

      const option2 = document.createElement("option");
      option2.value = cat;
      option2.textContent = cat;
      filterSelect.appendChild(option2);
    });

    // أضف خيار "أخرى"
    const otherOption = document.createElement("option");
    otherOption.value = "أخرى";
    otherOption.textContent = "أخرى";
    select.appendChild(otherOption);
  }

  // تحديث الجدول بناءً على الفلاتر
  function تحديث_الجدول() {
    const tbody = document.querySelector("#جدول-مصروفات tbody");
    tbody.innerHTML = "";

    const فلترنوع = document.getElementById("فلتر-نوع").value;
    const فلترتصنيف = document.getElementById("فلتر-تصنيف").value;
    const فلترتاريخمن = document.getElementById("فلتر-تاريخ-من").value;
    const فلترتاريخإلى = document.getElementById("فلتر-تاريخ-إلى").value;

    // فلترة البيانات حسب الفلاتر
    let بيانات_مفلترة = بيانات.filter(عنصر => {
      if (فلترنوع !== "الكل" && عنصر.النوع !== فلترنوع) return false;
      if (فلترتصنيف !== "الكل" && عنصر.التصنيف !== فلترتصنيف) return false;
      if (فلترتاريخمن && عنصر.التاريخ < فلترتاريخ
