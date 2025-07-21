<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <title>Расписание занятий</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f9f9f9; margin: 0; padding: 20px; }
    h1 { text-align: center; margin-bottom: 30px; }
    table { width: 100%; max-width: 800px; margin: 0 auto; border-collapse: collapse; }
    th, td { border: 1px solid #ddd; padding: 12px; text-align: center; }
    th { background: #f0f0f0; }
    td.available { background: #e0f7fa; cursor: pointer; }
    td.booked { background: #ffcdd2; color: #777; cursor: default; }
    td.available:hover { background: #b2ebf2; }
    #bookingForm, #overlay {
      display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
    }
    #overlay { background: rgba(0,0,0,0.3); z-index: 999; }
    #bookingForm {
      top: 50%; left: 50%; transform: translate(-50%, -50%);
      background: #fff; padding: 20px; border-radius: 10px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.2); width: 300px; z-index: 1000;
    }
    input, button { width: 100%; margin: 10px 0; padding: 10px; font-size: 16px; }
    button { background: #00796b; color: #fff; border: none; border-radius: 4px; }
    button:hover { background: #00695c; }
  </style>
</head>
<body>

<h1>Запись на занятия по математике</h1>

<table id="schedule">
  <thead>
    <tr>
      <th>Время</th>
      <th>Пн</th><th>Вт</th><th>Ср</th><th>Чт</th><th>Пт</th><th>Сб</th>
    </tr>
  </thead>
  <tbody id="tableBody"></tbody>
</table>

<div id="overlay"></div>
<div id="bookingForm">
  <h3>Запись на:</h3>
  <p id="slotInfo"></p>
  <input type="text" id="nameInput" placeholder="Имя">
  <input type="tel" id="phoneInput" placeholder="Телефон">
  <button id="submitBtn">Записаться</button>
</div>

<script>
const apiUrl = "https://script.google.com/macros/s/AKfycbzUK3TSGRwjGFgAZZJXrQHp5fj3v9Bgm2RsE6HOQabmfYLkwtHTvBBJnz4ycqVIGjLS/exec";

const tableBody = document.getElementById("tableBody");
const overlay = document.getElementById("overlay");
const form = document.getElementById("bookingForm");
const slotInfo = document.getElementById("slotInfo");
let selectedCell = null;

// Время и дни
const times = ["15:00", "16:00", "17:00", "18:00", "19:00", "20:00"];
const days = ["Пн", "Вт", "Ср", "Чт", "Пт", "Сб"];

// Рендер таблицы
function renderTable(bookings = []) {
  tableBody.innerHTML = "";
  times.forEach(time => {
    const row = document.createElement("tr");
    const timeCell = document.createElement("td");
    timeCell.textContent = time;
    row.appendChild(timeCell);

    for (let d = 0; d < days.length; d++) {
      const day = days[d];
      const cell = document.createElement("td");
      const isBooked = bookings.some(b => b.day == day && b.time == time);
      cell.textContent = isBooked ? "Занято" : "Свободно";
      cell.className = isBooked ? "booked" : "available";
      if (!isBooked) {
        cell.onclick = () => openForm(cell, day, time);
      }
      cell.dataset.day = day;
      cell.dataset.time = time;
      row.appendChild(cell);
    }
    tableBody.appendChild(row);
  });
}

// Открыть форму
function openForm(cell, day, time) {
  selectedCell = cell;
  slotInfo.textContent = `${day}, ${time}`;
  overlay.style.display = form.style.display = "block";
}

// Закрыть форму
overlay.onclick = () => {
  overlay.style.display = form.style.display = "none";
  document.getElementById("nameInput").value = "";
  document.getElementById("phoneInput").value = "";
};

// Отправка формы
document.getElementById("submitBtn").onclick = () => {
  const name = document.getElementById("nameInput").value.trim();
  const phone = document.getElementById("phoneInput").value.trim();
  const day = selectedCell.dataset.day;
  const time = selectedCell.dataset.time;
  if (!name || !phone) return alert("Заполните все поля");

  fetch(apiUrl, {
    method: "POST",
    body: JSON.stringify({ name, phone, day, time }),
    headers: { "Content-Type": "application/json" }
  }).then(res => res.json()).then(() => {
    selectedCell.textContent = "Занято";
    selectedCell.className = "booked";
    selectedCell.onclick = null;
    overlay.style.display = form.style.display = "none";
    document.getElementById("nameInput").value = "";
    document.getElementById("phoneInput").value = "";
  });
};

// Загрузка данных
fetch(apiUrl)
  .then(res => res.json())
  .then(data => renderTable(data));
</script>

</body>
</html>
