<Monthly>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>PocketTracker Final</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg,#f0f4f8,#d9e2ec);
  color: #333;
  margin:0; padding:20px;
}
h1,h2 {text-align:center;color:#1a1a1a;}
#datetime{text-align:center;font-weight:bold;margin-bottom:20px;color:#555;}
input, button {padding:8px;margin:5px;border:2px solid #4a90e2;border-radius:8px;outline:none;transition:0.3s;}
input:focus {border-color:#357ABD; box-shadow:0 0 5px #357ABD;}
button{padding:10px 20px;background:#4a90e2;color:#fff;border:none;border-radius:8px;cursor:pointer;}
button:hover{background:#357ABD; transform:scale(1.05);}
table{border-collapse: collapse;width:100%;margin-top:20px;background:#fff;border-radius:8px;overflow:hidden;box-shadow:0 4px 10px rgba(0,0,0,0.1);}
th,td{border:1px solid #e0e0e0;padding:8px;text-align:center;color:#333;}
th{background:#f5f7fa;}
td:hover{background:#f0f4f8;}
.progress-bar {height:25px;background:#e0e0e0;border-radius:12px;overflow:hidden;margin:10px 0;}
.progress {height:100%;background:#4a90e2;width:0%;color:#fff;text-align:center;line-height:25px;font-weight:bold;transition:0.5s;}
</style>
</head>
<body>

<h1>🌟 PocketTracker Final 🌟</h1>
<div id="datetime"></div>

<div style="text-align:center;">
  <label>Salary: <input type="number" id="salary" value="49800"></label>
  <label>Ghar Kharcha: <input type="number" id="house" value="20000"></label>
  <label>Loan: <input type="number" id="loan" value="10000"></label>
  <label>Savings Goal: <input type="number" id="goal" value="10000"></label>
  <br>
  <button onclick="addDay()">Add Day</button>
  <button onclick="calculate()">Recalculate</button>
  <button onclick="exportCSV()">Export CSV</button>
</div>

<h2>Monthly Overview</h2>
<table>
<tr><th>Total Salary</th><th>Total Expense</th><th>Remaining Saving</th><th>Goal %</th></tr>
<tr>
<td id="totalSalary">0</td>
<td id="totalExpense">0</td>
<td id="remaining">0</td>
<td id="goalStatus">0%</td>
</tr>
</table>
<div class="progress-bar"><div id="progress" class="progress">0%</div></div>

<h2>Daily Expenses</h2>
<table id="dailyTable">
<tr><th>Day</th><th>Food</th><th>Fuel</th><th>Snacks</th><th>Bills</th><th>Entertainment</th><th>Total</th></tr>
</table>

<h2>Weekly Summary</h2>
<table>
<tr><th>Week</th><th>Expense</th><th>Saving</th></tr>
<tr><td>Week 1</td><td id="w1Expense">0</td><td id="w1Saving">0</td></tr>
<tr><td>Week 2</td><td id="w2Expense">0</td><td id="w2Saving">0</td></tr>
<tr><td>Week 3</td><td id="w3Expense">0</td><td id="w3Saving">0</td></tr>
<tr><td>Week 4</td><td id="w4Expense">0</td><td id="w4Saving">0</td></tr>
</table>

<h2>Visual Overview</h2>
<canvas id="chart" width="400" height="200"></canvas>

<script>
// Timer
function updateDateTime(){
    const now = new Date();
    const options = {weekday:'long', year:'numeric', month:'long', day:'numeric'};
    document.getElementById('datetime').innerText = now.toLocaleDateString('en-US', options) + ' | ' + now.toLocaleTimeString();
}
setInterval(updateDateTime,1000);
updateDateTime();

// Load from localStorage
let dailyData = [];
if(localStorage.getItem('dailyData')) dailyData = JSON.parse(localStorage.getItem('dailyData'));
calculate();

// Add new day manually
function addDay(){
    const food = parseInt(prompt("Food PKR:", "200")) || 0;
    const fuel = parseInt(prompt("Fuel PKR:", "50")) || 0;
    const snacks = parseInt(prompt("Snacks PKR:", "30")) || 0;
    const bills = parseInt(prompt("Bills PKR:", "50")) || 0;
    const entertainment = parseInt(prompt("Entertainment PKR:", "50")) || 0;
    const total = food+fuel+snacks+bills+entertainment;
    dailyData.push({food,fuel,snacks,bills,entertainment,total});
    localStorage.setItem('dailyData',JSON.stringify(dailyData));
    calculate();
}

// Main calculate
function calculate(){
    let salary = parseInt(document.getElementById('salary').value);
    let house = parseInt(document.getElementById('house').value);
    let loan = parseInt(document.getElementById('loan').value);
    let goal = parseInt(document.getElementById('goal').value);

    // Update daily table
    const table = document.getElementById('dailyTable');
    table.innerHTML = "<tr><th>Day</th><th>Food</th><th>Fuel</th><th>Snacks</th><th>Bills</th><th>Entertainment</th><th>Total</th></tr>";
    let totalExpense = house+loan;
    dailyData.forEach((day,index)=>{
        const row = table.insertRow();
        row.insertCell(0).innerText = index+1;
        row.insertCell(1).innerText = day.food;
        row.insertCell(2).innerText = day.fuel;
        row.insertCell(3).innerText = day.snacks;
        row.insertCell(4).innerText = day.bills;
        row.insertCell(5).innerText = day.entertainment;
        row.insertCell(6).innerText = day.total;
        totalExpense += day.total;
    });

    const remaining = salary-totalExpense;
    document.getElementById('totalSalary').innerText = salary;
    document.getElementById('totalExpense').innerText = totalExpense;
    document.getElementById('remaining').innerText = remaining;

    let goalPercent = Math.min(Math.round((remaining/goal)*100),100);
    document.getElementById('goalStatus').innerText = goalPercent+'%';
    document.getElementById('progress').style.width = goalPercent+'%';
    document.getElementById('progress').innerText = goalPercent+'%';

    // Weekly summary
    const weekCount = 4;
    const weekLen = Math.ceil(dailyData.length/weekCount);
    for(let i=1;i<=weekCount;i++){
        const start=(i-1)*weekLen;
        const end=i*weekLen;
        let weekExpense = house/4 + loan/4;
        dailyData.slice(start,end).forEach(d=>weekExpense+=d.total);
        let weekSaving = salary/4 - weekExpense;
        document.getElementById('w'+i+'Expense').innerText = Math.round(weekExpense);
        document.getElementById('w'+i+'Saving').innerText = Math.round(weekSaving);
    }

    // Chart
    const ctx=document.getElementById('chart').getContext('2d');
    if(window.barChart) window.barChart.destroy();
    window.barChart=new Chart(ctx,{
        type:'bar',
        data:{
            labels:['Salary','Total Expense','Remaining Saving'],
            datasets:[{label:'PKR',data:[salary,totalExpense,remaining],backgroundColor:['#4a90e2','#f39c12','#2ecc71'],borderColor:['#357ABD','#d35400','#27ae60'],borderWidth:2}]
        },
        options:{responsive:true,plugins:{legend:{display:false}},scales:{y:{beginAtZero:true}}}
    });
}

// Export CSV
function exportCSV(){
    let csv='Day,Food,Fuel,Snacks,Bills,Entertainment,Total\n';
    dailyData.forEach((d,index)=>{
        csv+=`${index+1},${d.food},${d.fuel},${d.snacks},${d.bills},${d.entertainment},${d.total}\n`;
    });
    let blob = new Blob([csv], {type:'text/csv'});
    let link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = 'PocketTracker_Month.csv';
    link.click();
}
</script>
</body>
</html>
