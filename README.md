<MONTHLY>
<html>
<head>
  <title>PocketTracker - Neon Salary Dashboard</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap');

    body { font-family:'Orbitron', sans-serif; background:#000; color:#0ff; padding:20px; }
    h1, h2 { color:#0ff; text-shadow:0 0 10px #0ff,0 0 20px #0ff; }
    p.message { color:#0ff; text-shadow:0 0 5px #0ff,0 0 10px #00f; font-size:18px; }
    input { padding:8px; width:120px; margin:5px; background:#111; border:2px solid #0ff; border-radius:5px; color:#0ff; }
    input:focus { box-shadow:0 0 10px #0ff,0 0 20px #0ff; border-color:#0ff; }
    button { padding:10px 20px; margin:10px 0; background:#00f; border:none; color:#0ff; font-weight:bold; cursor:pointer; border-radius:5px; box-shadow:0 0 5px #0ff,0 0 15px #0ff; transition:0.3s; }
    button:hover { box-shadow:0 0 20px #0ff,0 0 40px #0ff; transform:scale(1.05); }
    table { border-collapse:collapse; width:100%; margin-top:20px; background:#111; border:2px solid #0ff; box-shadow:0 0 10px #0ff; }
    th, td { border:1px solid #0ff; padding:10px; text-align:center; color:#0ff; transition:0.3s; }
    th { background:#001f3f; text-shadow:0 0 5px #0ff; }
    td:hover { color:#00f; text-shadow:0 0 10px #0ff,0 0 20px #00f; }
    canvas { margin-top:20px; background:#000; box-shadow:0 0 20px #0ff; border-radius:10px; }
    label { display:inline-block; margin:5px; color:#0ff; text-shadow:0 0 5px #0ff; }
  </style>
</head>
<body>

<h1>🌌 PocketTracker - Neon Salary Dashboard 🌌</h1>
<p class="message">Hey Sweetie! 😇 Ab tu apni salary, daily & monthly kharcha aur saving easy aur neon style me track kar sakta hai! 💸✨</p>

<div>
  <label>Salary: <input type="number" id="salary" value="49800"></label>
  <label>Ghar Kharcha: <input type="number" id="house" value="20000"></label>
  <label>Loan: <input type="number" id="loan" value="10000"></label>
  <label>Daily Kharcha: <input type="number" id="daily" value="500"></label>
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

  // Weekly summary (6-7 workdays per week)
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
        backgroundColor:['#0ff','#00f','#0f0'],
        borderColor:['#0ff','#00f','#0f0'],
        borderWidth:2
      }]
    },
    options:{responsive:true, plugins:{legend:{display:false}},
      scales:{y:{beginAtZero:true,ticks:{color:'#0ff'}}, x:{ticks:{color:'#0ff'}}}
    }
  });
}
</script>

</body>
</html>
