<Monthly>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>PocketTracker - Premium Salary Dashboard</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #f0f4f8, #d9e2ec);
    color: #333;
    margin: 0;
    padding: 20px;
}

h1, h2 {
    text-align: center;
    color: #1a1a1a;
}

p.message {
    text-align: center;
    font-size: 18px;
    color: #1a1a1a;
    margin-bottom: 20px;
}

input {
    padding: 8px;
    width: 130px;
    margin: 5px;
    border: 2px solid #4a90e2;
    border-radius: 8px;
    outline: none;
    transition: 0.3s;
}
input:focus {
    border-color: #357ABD;
    box-shadow: 0 0 5px #357ABD;
}

button {
    padding: 12px 24px;
    margin: 10px 0;
    background: #4a90e2;
    border: none;
    color: #fff;
    font-weight: bold;
    cursor: pointer;
    border-radius: 8px;
    transition: 0.3s;
}
button:hover {
    background: #357ABD;
    transform: scale(1.05);
}

table {
    border-collapse: collapse;
    width: 100%;
    margin-top: 20px;
    background: #fff;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 4px 10px rgba(0,0,0,0.1);
}

th, td {
    border: 1px solid #e0e0e0;
    padding: 12px;
    text-align: center;
    color: #333;
}

th {
    background: #f5f7fa;
    font-weight: bold;
}

td:hover {
    background: #f0f4f8;
}

canvas {
    margin-top: 20px;
    background: #fff;
    border-radius: 12px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.1);
}

label {
    display: inline-block;
    margin: 5px;
    font-weight: bold;
}
</style>
</head>
<body>

<h1>🌟 PocketTracker - Premium Salary Dashboard 🌟</h1>
<p class="message">Hey Sweetie! 😇 Track your salary, daily & monthly expenses, and savings easily with this clean, modern dashboard!</p>

<div style="text-align:center;">
  <label>Salary: <input type="number" id="salary" value="49800"></label>
  <label>Ghar Kharcha: <input type="number" id="house" value="20000"></label>
  <label>Loan: <input type="number" id="loan" value="10000"></label>
  <label>Daily Kharcha: <input type="number" id="daily" value="500"></label>
  <br>
  <button onclick="calculate()">Calculate</button>
</div>

<h2>Monthly Overview</h2>
<table>
  <tr>
    <th>Total Salary</th>
    <th>Total Kharcha</th>
    <th>Remaining Saving</th>
  </tr>
  <tr>
    <td id="totalSalary">0</td>
    <td id="totalExpense">0</td>
    <td id="remaining">0</td>
  </tr>
</table>

<h2>Daily Expenses (26 workdays)</h2>
<table id="dailyTable">
  <tr><th>Day</th><th>Kharcha (PKR)</th></tr>
</table>

<h2>Weekly Summary</h2>
<table>
  <tr>
    <th>Week</th>
    <th>Kharcha</th>
    <th>Saving</th>
  </tr>
  <tr><td>Week 1</td><td id="w1Expense">0</td><td id="w1Saving">0</td></tr>
  <tr><td>Week 2</td><td id="w2Expense">0</td><td id="w2Saving">0</td></tr>
  <tr><td>Week 3</td><td id="w3Expense">0</td><td id="w3Saving">0</td></tr>
  <tr><td>Week 4</td><td id="w4Expense">0</td><td id="w4Saving">0</td></tr>
</table>

<h2>Visual Overview</h2>
<canvas id="chart" width="400" height="200"></canvas>

<script>
function calculate(){
  let salary = parseInt(document.getElementById('salary').value);
  let house = parseInt(document.getElementById('house').value);
  let loan = parseInt(document.getElementById('loan').value);
  let daily = parseInt(document.getElementById('daily').value);
  let workdays = 26;

  // Total
  let totalExpense = house + loan + daily*workdays;
  let remaining = salary - totalExpense;
  document.getElementById('totalSalary').innerText = salary;
  document.getElementById('totalExpense').innerText = totalExpense;
  document.getElementById('remaining').innerText = remaining;

  // Daily table
  let dailyTable = document.getElementById('dailyTable');
  dailyTable.innerHTML = "<tr><th>Day</th><th>Kharcha (PKR)</th></tr>";
  for(let i=1;i<=workdays;i++){
    let row = dailyTable.insertRow();
    row.insertCell(0).innerText = "Day "+i;
    row.insertCell(1).innerText = daily;
  }

  // Weekly summary
  let dailyPerWeek = daily*6.5;
  for(let i=1;i<=4;i++){
    document.getElementById('w'+i+'Expense').innerText = (house/4 + loan/4 + dailyPerWeek).toFixed(0);
    document.getElementById('w'+i+'Saving').innerText = (salary/4 - (house/4 + loan/4 + dailyPerWeek)).toFixed(0);
  }

  // Chart
  const ctx = document.getElementById('chart').getContext('2d');
  if(window.barChart) window.barChart.destroy();
  window.barChart = new Chart(ctx,{
    type:'bar',
    data:{
      labels:['Salary','Kharcha','Remaining Saving'],
      datasets:[{
        label:'PKR',
        data:[salary,totalExpense,remaining],
        backgroundColor:['#4a90e2','#f39c12','#2ecc71'],
        borderColor:['#357ABD','#d35400','#27ae60'],
        borderWidth:2
      }]
    },
    options:{responsive:true, plugins:{legend:{display:false}},
      scales:{y:{beginAtZero:true}, x:{}}
    }
  });
}
</script>

</body>
</html>
