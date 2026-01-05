<MONTHLY>
<html>
<head>
  <title>Sweetie Salary Tracker</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: Arial; padding: 20px; background: #fefefe; color: #333; }
    h1 { color: #4CAF50; }
    input { padding: 5px; width: 120px; margin: 5px; }
    button { padding: 8px 15px; margin: 10px 0; background: #4CAF50; color: white; border: none; cursor: pointer; }
    table { border-collapse: collapse; width: 100%; margin-top: 20px; }
    th, td { border: 1px solid #ccc; padding: 10px; text-align: center; }
    th { background-color: #4CAF50; color: white; }
    .green { background-color: #d4edda; }
    .red { background-color: #f8d7da; }
  </style>
</head>
<body>

<h1>Sweetie Salary Tracker</h1>

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

<h2>Weekly Summary</h2>
<table>
  <tr>
    <th>Week</th>
    <th>Kharcha</th>
    <th>Saving</th>
  </tr>
  <tr>
    <td>Week 1</td><td id="w1Expense">0</td><td id="w1Saving">0</td>
  </tr>
  <tr>
    <td>Week 2</td><td id="w2Expense">0</td><td id="w2Saving">0</td>
  </tr>
  <tr>
    <td>Week 3</td><td id="w3Expense">0</td><td id="w3Saving">0</td>
  </tr>
  <tr>
    <td>Week 4</td><td id="w4Expense">0</td><td id="w4Saving">0</td>
  </tr>
</table>

<h2>Visual Overview</h2>
<canvas id="chart" width="400" height="200"></canvas>

<script>
function calculate() {
  let salary = parseInt(document.getElementById('salary').value);
  let house = parseInt(document.getElementById('house').value);
  let loan = parseInt(document.getElementById('loan').value);
  let daily = parseInt(document.getElementById('daily').value);
  let workdays = 26;

  let totalExpense = house + loan + (daily * workdays);
  let remaining = salary - totalExpense;

  document.getElementById('totalSalary').innerText = salary;
  document.getElementById('totalExpense').innerText = totalExpense;
  document.getElementById('remaining').innerText = remaining;

  // Weekly Summary (6-7 workdays per week)
  let dailyPerWeek = daily * 6.5; // approx
  for(let i=1;i<=4;i++){
    document.getElementById('w'+i+'Expense').innerText = (house/4 + loan/4 + dailyPerWeek).toFixed(0);
    document.getElementById('w'+i+'Saving').innerText = (salary/4 - (house/4 + loan/4 + dailyPerWeek)).toFixed(0);
  }

  // Chart
  const ctx = document.getElementById('chart').getContext('2d');
  if(window.barChart) window.barChart.destroy();
  window.barChart = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: ['Salary','Kharcha','Remaining Saving'],
      datasets: [{
        label: 'PKR',
        data: [salary, totalExpense, remaining],
        backgroundColor: ['#4CAF50','#f44336','#2196F3']
      }]
    },
    options: { responsive: true, plugins: { legend: { display: false } } }
  });
}
</script>

</body>
</html>
