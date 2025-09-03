<!DOCTYPE html> 
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>MSTMS - Two-Week Timecard</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #ccecf1;
      padding: 20px;
      margin: 0;
    }
    .container {
      max-width: 1000px;
      margin: auto;
      background: #fff;
      padding: 20px;
      border-radius: 10px;
      border: 2px solid #99c;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-bottom: 20px;
    }
    th, td {
      padding: 8px;
      text-align: center;
      border: 1px solid #ccc;
    }
    input, select {
      padding: 5px;
      width: 100%;
      box-sizing: border-box;
    }
    label {
      font-weight: bold;
      display: block;
      margin-top: 10px;
    }
    .output-box {
      margin-top: 20px;
      padding: 15px;
      background-color: #e6f7ff;
      border: 1px solid #99c;
      border-radius: 5px;
    }
    .highlight {
      background-color: yellow;
      font-weight: bold;
    }
    .btn-group {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin: 20px 0;
    }
    button {
      flex: 1 1 48%;
      padding: 10px;
      font-size: 16px;
      cursor: pointer;
    }
  </style>
</head>
<body>

<div class="container">
  <h2>Timecard Entry (Two Weeks)</h2>

  <label for="dateFrom">Dates From:</label>
  <input type="date" id="dateFrom" onchange="calculateDateDiff(); updateWeekTables();" />

  <label for="dateTo">Dates To:</label>
  <input type="date" id="dateTo" onchange="calculateDateDiff()" />

  <label for="daysBetween">Number of Days:</label>
  <input type="text" id="daysBetween" readonly />

  <div id="weeksContainer"></div>

  <label for="leaveFrom">Request Leave From:</label>
  <input type="date" id="leaveFrom" />

  <label for="leaveTo">Request Leave To:</label>
  <input type="date" id="leaveTo" />

  <div class="btn-group">
    <button onclick="tallyTime()">Tally TimeCard</button>
    <button onclick="clearForm()">Clear Form</button>
    <button onclick="saveTimecard()">Save Timecard</button>
    <button onclick="clearSavedTimecards()">Clear Saved Timecards</button>
  </div>

  <label for="savedTimecards">Saved Timecards:</label>
  <select id="savedTimecards" onchange="loadTimecard()">
    <option value="">Select a saved timecard</option>
  </select>

  <div class="output-box" id="result"></div>
</div>

<script>
  const days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'];

  function updateWeekTables(savedData = null) {
    const startDateInput = document.getElementById("dateFrom").value;
    if (!startDateInput) {
      createWeekTablesDefault();
      return;
    }

    const startDate = new Date(startDateInput);
    let html = '';
    for (let w = 0; w < 2; w++) {
      html += `<h3>Week ${w + 1}</h3><table><tr><th>Day</th><th>Date</th><th>Hours</th><th>Type</th><th>Task Type</th><th>Task Details</th></tr>`;
      for (let d = 0; d < 7; d++) {
        const dayOffset = w * 7 + d;
        const currentDate = new Date(startDate);
        currentDate.setDate(startDate.getDate() + dayOffset);
        const actualDayOfWeek = currentDate.getDay(); // 0 (Sunday) to 6 (Saturday)
        const dayName = days[(actualDayOfWeek === 0 ? 6 : actualDayOfWeek - 1)]; // Map to Monday=0, Sunday=6
        const dateStr = currentDate.toISOString().split('T')[0];

        // Default values
        let defaultHours = (dayName === 'Saturday' || dayName === 'Sunday') ? 0 : 8;
        let defaultType = (dayName === 'Saturday' || dayName === 'Sunday') ? 'Off' : 'Regular';
        let defaultTaskType = (dayName === 'Saturday' || dayName === 'Sunday') ? 'Off' : 'Analytic Assistance';
        let defaultTaskDetails = '';

        // Override with screenshot defaults for Week 1 if no saved data
        if (!savedData && w === 0) {
        const screenshotDates = [
 	  { date: '2025-03-07', day: 'Thursday', hours: 8, type: 'Regular', taskType: 'Analytic Assistance', taskDetails: '' },
 	  { date: '2025-03-08', day: 'Friday', hours: 8, type: 'Regular', taskType: 'Analytic Assistance', taskDetails: '' },
	  { date: '2025-03-09', day: 'Saturday', hours: 0, type: 'Off', taskType: 'Off', taskDetails: '' },
	  { date: '2025-03-10', day: 'Sunday', hours: 0, type: 'Off', taskType: 'Off', taskDetails: '' },
 	  { date: '2025-03-11', day: 'Monday', hours: 8, type: 'Regular', taskType: 'Analytic Assistance', taskDetails: '' },
	  { date: '2025-03-12', day: 'Tuesday', hours: 8, type: 'Regular', taskType: 'Analytic Assistance', taskDetails: '' },
	  { date: '2025-03-13', day: 'Wednesday', hours: 8, type: 'Regular', taskType: 'Analytic Assistance', taskDetails: '' }
	];

          const matchingEntry = screenshotDates.find(entry => entry.date === dateStr);
          if (matchingEntry) {
            defaultHours = matchingEntry.hours;
            defaultType = matchingEntry.type;
            defaultTaskType = matchingEntry.taskType;
            defaultTaskDetails = matchingEntry.taskDetails;
          }
        }

        // Override with saved data if available
        if (savedData && savedData.entries) {
          const entryKey = `${dayName}${w + 1}`;
          const savedEntry = savedData.entries[entryKey];
          if (savedEntry) {
            defaultHours = savedEntry.hours;
            defaultType = savedEntry.type;
            defaultTaskType = savedEntry.taskType;
            defaultTaskDetails = savedEntry.taskDetails || '';
          }
        }

        html += `
          <tr>
            <td><input type="text" value="${dayName}" readonly></td>
            <td><input type="text" value="${dateStr}" readonly></td>
            <td><input type="number" id="${dayName}${w + 1}Hours" value="${defaultHours}"></td>
            <td>
              <select id="${dayName}${w + 1}Type">
                <option value="Sick" ${defaultType === 'Sick' ? 'selected' : ''}>Sick</option>
                <option value="Regular" ${defaultType === 'Regular' ? 'selected' : ''}>Regular</option>
                <option value="Leave" ${defaultType === 'Leave' ? 'selected' : ''}>Leave</option>
                <option value="Off" ${defaultType === 'Off' ? 'selected' : ''}>Off</option>
              </select>
            </td>
            <td>
              <select id="${dayName}${w + 1}TaskType">
                <option value="Analytic Assistance" ${defaultTaskType === 'Analytic Assistance' ? 'selected' : ''}>Analytic Assistance</option>
                <option value="On Call" ${defaultTaskType === 'On Call' ? 'selected' : ''}>On Call</option>
                <option value="Emergency Response Support" ${defaultTaskType === 'Emergency Response Support' ? 'selected' : ''}>Emergency Response Support</option>
                <option value="Off" ${defaultTaskType === 'Off' ? 'selected' : ''}>Off</option>
              </select>
            </td>
            <td><input type="text" id="${dayName}${w + 1}TaskDetails" value="${defaultTaskDetails}" /></td>
          </tr>`;
      }
      html += '</table>';
    }
    document.getElementById('weeksContainer').innerHTML = html;
  }

  function createWeekTablesDefault() {
    let html = '';
    for (let w = 1; w <= 2; w++) {
      html += `<h3>Week ${w}</h3><table><tr><th>Day</th><th>Date</th><th>Hours</th><th>Type</th><th>Task Type</th><th>Task Details</th></tr>`;
      days.forEach(day => {
        const defaultHours = (day === 'Saturday' || day === 'Sunday') ? 0 : 8;
        const defaultType = (day === 'Saturday' || day === 'Sunday') ? 'Off' : 'Regular';
        const defaultTaskType = (day === 'Saturday' || day === 'Sunday') ? 'Off' : 'Analytic Assistance';
        html += `
          <tr>
            <td><input type="text" value="${day}" readonly></td>
            <td><input type="text" value="" readonly></td>
            <td><input type="number" id="${day}${w}Hours" value="${defaultHours}"></td>
            <td>
              <select id="${day}${w}Type">
                <option value="Sick" ${defaultType === 'Sick' ? 'selected' : ''}>Sick</option>
                <option value="Regular" ${defaultType === 'Regular' ? 'selected' : ''}>Regular</option>
                <option value="Leave" ${defaultType === 'Leave' ? 'selected' : ''}>Leave</option>
                <option value="Off" ${defaultType === 'Off' ? 'selected' : ''}>Off</option>
              </select>
            </td>
            <td>
              <select id="${day}${w}TaskType">
                <option value="Analytic Assistance" ${defaultTaskType === 'Analytic Assistance' ? 'selected' : ''}>Analytic Assistance</option>
                <option value="On Call" ${defaultTaskType === 'On Call' ? 'selected' : ''}>On Call</option>
                <option value="Emergency Response Support" ${defaultTaskType === 'Emergency Response Support' ? 'selected' : ''}>Emergency Response Support</option>
                <option value="Off" ${defaultTaskType === 'Off' ? 'selected' : ''}>Off</option>
              </select>
            </td>
            <td><input type="text" id="${day}${w}TaskDetails" /></td>
          </tr>`;
      });
      html += '</table>';
    }
    document.getElementById('weeksContainer').innerHTML = html;
  }

  function calculateDateDiff() {
    const startDate = document.getElementById("dateFrom").value;
    const endDate = document.getElementById("dateTo").value;

    if (startDate && endDate) {
      const start = new Date(startDate);
      const end = new Date(endDate);
      if (start <= end) {
        const difference = Math.ceil((end - start) / (1000 * 60 * 60 * 24)) + 1;
        document.getElementById("daysBetween").value = `${difference} days`;
        if (difference !== 14) {
          document.getElementById("daysBetween").value += " (Note: Two-week timecard requires 14 days)";
        }
      } else {
        document.getElementById("daysBetween").value = "Invalid range";
      }
    } else {
      document.getElementById("daysBetween").value = "";
    }
  }

  function tallyTime() {
    let sickHours = 0, regularHours = 0, leaveHours = 0, totalHours = 0;
    let taskEntries = [];

    const dateFrom = document.getElementById("dateFrom").value;
    const dateTo = document.getElementById("dateTo").value;
    if (!dateFrom || !dateTo) {
      alert("Please select both 'Dates From' and 'Dates To' before tallying.");
      return;
    }

    const startDate = new Date(dateFrom);
    const endDate = new Date(dateTo);
    if (startDate > endDate) {
      alert("Invalid date range: 'Dates From' must be before 'Dates To'.");
      return;
    }

    const diffDays = Math.ceil((endDate - startDate) / (1000 * 60 * 60 * 24)) + 1;
    if (diffDays !== 14) {
      alert("Please ensure the date range is exactly 14 days for a two-week timecard.");
      return;
    }

    for (let w = 0; w < 2; w++) {
      for (let d = 0; d < 7; d++) {
        const dayOffset = w * 7 + d;
        const currentDate = new Date(startDate);
        currentDate.setDate(startDate.getDate() + dayOffset);
        const actualDayOfWeek = currentDate.getDay();
        const dayName = days[(actualDayOfWeek === 0 ? 6 : actualDayOfWeek - 1)];

        const hrs = parseFloat(document.getElementById(`${dayName}${w + 1}Hours`).value) || 0;
        const type = document.getElementById(`${dayName}${w + 1}Type`).value;
        const taskType = document.getElementById(`${dayName}${w + 1}TaskType`).value;
        const taskDetail = document.getElementById(`${dayName}${w + 1}TaskDetails`).value.trim();

        if (type === 'Sick') sickHours += hrs;
        else if (type === 'Regular') regularHours += hrs;
        else if (type === 'Leave') leaveHours += hrs;

        totalHours += hrs;

        const formattedDate = currentDate.toISOString().split('T')[0];
        taskEntries.push({
          type: type,
          taskType: taskType,
          day: dayName,
          week: w + 1,
          date: formattedDate,
          detail: taskDetail || "None",
          timestamp: currentDate.getTime()
        });
      }
    }

    const taskSummaryByType = {};
    taskEntries.forEach(entry => {
      if (!taskSummaryByType[entry.taskType]) {
        taskSummaryByType[entry.taskType] = [];
      }
      taskSummaryByType[entry.taskType].push(entry);
    });

    for (const taskType in taskSummaryByType) {
      taskSummaryByType[taskType].sort((a, b) => a.timestamp - b.timestamp);
    }

    let taskSummaryHTML = '<strong>Task Summary:</strong><br>';
    for (const taskType in taskSummaryByType) {
      taskSummaryHTML += `<strong>${taskType}:</strong><br>`;
      taskSummaryByType[taskType].forEach(entry => {
        taskSummaryHTML += `- ${entry.day} (${entry.date}) Week ${entry.week}: Type: ${entry.type}, Task Type: ${entry.taskType} - ${entry.detail}<br>`;
      });
    }

    document.getElementById("result").innerHTML = `
      <strong>Time Card Information for Payroll Department:</strong><br>
      Dates From: ${dateFrom} to ${dateTo}<br><br>
      Sick Hours: ${sickHours} hours<br>
      Regular Hours: ${regularHours} hours<br>
      Leave Hours: ${leaveHours} hours<br>
      <span class="highlight">Total Tally Time Card: ${totalHours} hours</span><br><br>
      ${taskSummaryHTML}
    `;
  }

  function clearForm() {
    document.querySelectorAll("input, select").forEach(el => {
      if (el.type === 'text' || el.type === 'number' || el.type === 'date') el.value = '';
      if (el.tagName === 'SELECT') el.selectedIndex = 0;
    });
    document.getElementById("result").innerHTML = '';
    calculateDateDiff();
    createWeekTablesDefault();
  }

  function saveTimecard() {
    const timecards = JSON.parse(localStorage.getItem("timecards")) || [];

    if (timecards.length >= 5) {
      alert("You can only save up to 5 timecards.");
      return;
    }

    // Collect entries for each day
    const entries = {};
    for (let w = 0; w < 2; w++) {
      for (let d = 0; d < 7; d++) {
        const dayName = days[d];
        const entryKey = `${dayName}${w + 1}`;
        entries[entryKey] = {
          hours: document.getElementById(`${dayName}${w + 1}Hours`)?.value || '',
          type: document.getElementById(`${dayName}${w + 1}Type`)?.value || '',
          taskType: document.getElementById(`${dayName}${w + 1}TaskType`)?.value || '',
          taskDetails: document.getElementById(`${dayName}${w + 1}TaskDetails`)?.value || ''
        };
      }
    }

    const timecardData = {
      dateFrom: document.getElementById("dateFrom").value,
      dateTo: document.getElementById("dateTo").value,
      daysBetween: document.getElementById("daysBetween").value,
      result: document.getElementById("result").innerHTML,
      entries: entries,
      leaveFrom: document.getElementById("leaveFrom").value,
      leaveTo: document.getElementById("leaveTo").value
    };

    timecards.push(timecardData);
    localStorage.setItem("timecards", JSON.stringify(timecards));
    loadSavedTimecards();
  }

  function loadSavedTimecards() {
    const timecards = JSON.parse(localStorage.getItem("timecards")) || [];
    const select = document.getElementById("savedTimecards");

    select.innerHTML = '<option value="">Select a saved timecard</option>';

    timecards.forEach((tc, index) => {
      const option = document.createElement("option");
      option.value = index;
      option.textContent = `Timecard ${index + 1} (${tc.dateFrom} - ${tc.dateTo})`;
      select.appendChild(option);
    });
  }

  function loadTimecard() {
    const timecards = JSON.parse(localStorage.getItem("timecards")) || [];
    const index = document.getElementById("savedTimecards").value;

    if (index !== "") {
      const tc = timecards[index];
      document.getElementById("dateFrom").value = tc.dateFrom;
      document.getElementById("dateTo").value = tc.dateTo;
      document.getElementById("daysBetween").value = tc.daysBetween;
      document.getElementById("leaveFrom").value = tc.leaveFrom || '';
      document.getElementById("leaveTo").value = tc.leaveTo || '';
      document.getElementById("result").innerHTML = tc.result;
      updateWeekTables(tc); // Pass the saved data to pre-fill the table
    }
  }

  function clearSavedTimecards() {
    localStorage.removeItem("timecards");
    loadSavedTimecards();
    alert("All saved timecards have been cleared.");
  }

  window.onload = function () {
    createWeekTablesDefault();
    calculateDateDiff();
    loadSavedTimecards();
  };
</script>

</body>
</html>
