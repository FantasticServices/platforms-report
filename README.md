<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Weekly Performance Analytics Dashboard</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 20px;
      background-color: #f0f2f5;
      color: #333;
    }
    .container {
      max-width: 900px;
      margin: 0 auto;
      background: #fff;
      border-radius: 10px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      padding: 20px;
    }
    h1 {
      text-align: center;
      color: #2c3e50;
      margin-bottom: 20px;
    }
    .header {
      background-color: #3498db;
      color: white;
      padding: 10px;
      border-radius: 5px;
      text-align: center;
      margin-bottom: 20px;
    }
    .controls {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
      flex-wrap: wrap;
    }
    .controls select, .controls input {
      padding: 5px;
      margin-right: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    .controls button {
      background-color: #2ecc71;
      color: white;
      border: none;
      padding: 8px 15px;
      border-radius: 4px;
      cursor: pointer;
    }
    .controls button:hover {
      background-color: #27ae60;
    }
    .metrics {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
      gap: 15px;
      margin-bottom: 20px;
    }
    .card {
      background: #ecf0f1;
      padding: 15px;
      border-radius: 5px;
      text-align: center;
      box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
    }
    .card h3 {
      margin: 0;
      font-size: 14px;
      color: #7f8c8d;
    }
    .card p {
      margin: 5px 0;
      font-size: 18px;
      font-weight: bold;
      color: #2c3e50;
    }
    .positive {
      color: #27ae60;
    }
    .negative {
      color: #e74c3c;
    }
    .table-container {
      overflow-x: auto;
      margin-top: 20px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 8px;
      text-align: center;
    }
    th {
      background-color: #3498db;
      color: white;
    }
    .loading {
      text-align: center;
      padding: 20px;
      color: #7f8c8d;
    }
    .error {
      background-color: #e74c3c;
      color: white;
      padding: 10px;
      border-radius: 5px;
      margin-bottom: 20px;
    }
    @media (max-width: 768px) {
      .controls {
        flex-direction: column;
        align-items: stretch;
      }
      .controls select, .controls input, .controls button {
        margin-bottom: 10px;
        width: 100%;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Weekly Performance Analytics Dashboard</h1>
    
    <div id="errorMessage" class="error" style="display: none;"></div>
    
    <div class="header">
      <h2 id="weekHeader">Select a week to view performance data</h2>
    </div>
    
    <div class="controls">
      <div>
        <select id="weekSelect">
          <option value="">Select Week...</option>
        </select>
        <input type="date" id="customDate" placeholder="Custom Date">
        <button onclick="refreshData()">Refresh</button>
      </div>
    </div>
    
    <div id="loadingIndicator" class="loading">Loading...</div>
    
    <div id="metricsContainer" style="display: none;">
      <div class="metrics">
        <div class="card">
          <h3>Total Leads</h3>
          <p id="totalLeads">0</p>
        </div>
        <div class="card">
          <h3>Total Bookings</h3>
          <p id="totalBookings">0</p>
        </div>
        <div class="card">
          <h3>Conversion Rate</h3>
          <p id="conversionRate">0%</p>
        </div>
        <div class="card">
          <h3>Total Revenue</h3>
          <p id="totalRevenue">£0</p>
        </div>
        <div class="card">
          <h3>Total Commission</h3>
          <p id="totalCommission">£0</p>
        </div>
        <div class="card">
          <h3>Total Cost</h3>
          <p id="totalCost">£0</p>
        </div>
        <div class="card">
          <h3>Operating Result</h3>
          <p id="operatingResult">£0</p>
        </div>
      </div>
      
      <div class="table-container">
        <table id="platformTable">
          <thead>
            <tr>
              <th>Platform</th>
              <th>Leads</th>
              <th>Bookings</th>
              <th>CR</th>
              <th>Add-ons</th>
              <th>Booked Revenue</th>
              <th>Commission</th>
              <th>Cost</th>
              <th>Operating Result</th>
            </tr>
          </thead>
          <tbody id="platformTableBody">
          </tbody>
        </table>
      </div>
    </div>
  </div>

  <script>
    // Replace this with your actual Google Apps Script web app URL
    const SCRIPT_URL = 'YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL_HERE';



    // Initialize the dashboard
    document.addEventListener('DOMContentLoaded', function() {
      loadWeeks();
    });

    // Load weeks from Google Apps Script
    async function loadWeeks() {
      try {
        const response = await fetch(`${SCRIPT_URL}?action=getAvailableWeeks`);
        const result = await response.json();
        
        if (result.success) {
          const weekSelect = document.getElementById('weekSelect');
          weekSelect.innerHTML = '<option value="">Select Week...</option>';
          
          result.weeks.forEach(week => {
            const option = document.createElement('option');
            option.value = week.value;
            option.textContent = `Week ${week.number} (${week.dateRange})`;
            weekSelect.appendChild(option);
          });
        } else {
          showError('Failed to load weeks: ' + result.message);
        }
      } catch (error) {
        console.error('Error loading weeks:', error);
        showError('Failed to connect to data source');
      }
    }

    // Fetch data from Google Apps Script
    async function refreshData() {
      const weekSelect = document.getElementById('weekSelect');
      const customDate = document.getElementById('customDate');
      const selectedWeek = weekSelect.value;
      const selectedDate = customDate.value;
      
      if (!selectedWeek && !selectedDate) {
        showError('Please select a week or custom date');
        return;
      }
      
      showLoading();
      
      try {
        let url = `${SCRIPT_URL}?action=getWeeklySummaryData`;
        if (selectedWeek) {
          url += `&selectedWeek=${selectedWeek}`;
        }
        if (selectedDate) {
          url += `&customDate=${selectedDate}`;
        }
        
        const response = await fetch(url);
        const result = await response.json();
        
        if (result.success) {
          displayWeeklyData(result.data);
        } else {
          showError('Failed to load data: ' + result.message);
        }
      } catch (error) {
        console.error('Error fetching data:', error);
        showError('Failed to connect to data source');
      }
    }

    function displayWeeklyData(data) {
      hideLoading();
      hideError();
      
      // Update header
      document.getElementById('weekHeader').textContent = `${data.week} (${data.dateRange})`;
      
      // Update metrics
      document.getElementById('totalLeads').textContent = data.totals.leads;
      document.getElementById('totalBookings').textContent = data.totals.bookings;
      document.getElementById('conversionRate').textContent = data.totals.cr;
      document.getElementById('totalRevenue').textContent = data.totals.bookedRevenue;
      document.getElementById('totalCommission').textContent = data.totals.commission;
      document.getElementById('totalCost').textContent = data.totals.cost;
      
      const operatingElement = document.getElementById('operatingResult');
      operatingElement.textContent = data.totals.operatingResult;
      operatingElement.className = data.totals.isPositive ? 'positive' : 'negative';
      
      // Update table
      const tableBody = document.getElementById('platformTableBody');
      tableBody.innerHTML = '';
      
      data.platforms.forEach(platform => {
        const row = document.createElement('tr');
        row.innerHTML = `
          <td>${platform.name}</td>
          <td>${platform.leads}</td>
          <td>${platform.bookings}</td>
          <td>${platform.cr}</td>
          <td>${platform.addOns}</td>
          <td>${platform.bookedRevenue}</td>
          <td>${platform.commission}</td>
          <td>${platform.cost}</td>
          <td class="${platform.isPositive ? 'positive' : 'negative'}">${platform.operatingResult}</td>
        `;
        tableBody.appendChild(row);
      });
      
      document.getElementById('metricsContainer').style.display = 'block';
    }

    function showLoading() {
      document.getElementById('loadingIndicator').style.display = 'block';
      document.getElementById('metricsContainer').style.display = 'none';
    }

    function hideLoading() {
      document.getElementById('loadingIndicator').style.display = 'none';
    }

    function showError(message) {
      const errorDiv = document.getElementById('errorMessage');
      errorDiv.textContent = message;
      errorDiv.style.display = 'block';
      hideLoading();
    }

    function hideError() {
      document.getElementById('errorMessage').style.display = 'none';
    }

    // Week selection change handler
    document.getElementById('weekSelect').addEventListener('change', function() {
      if (this.value) {
        refreshData();
      }
    });
  </script>
</body>
</html>
