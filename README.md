<monthly>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>PocketTracker Icon Dashboard</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg,#e0f7fa,#80deea);
    color:#333; margin:0; padding:20px;
}
h1 {text-align:center; color:#00796b;}
#datetime {text-align:center;font-weight:bold;margin-bottom:20px;color:#004d40;}
.dashboard {display:flex;flex-wrap:wrap;justify-content:center;margin-bottom:20px;}
.card {background:#fff;box-shadow:0 4px 8px rgba(0,0,0,0.1);border-radius:12px;margin:10px;padding:15px;text-align:center;width:120px;cursor:pointer;transition:0.3s;}
.card:hover{transform:scale(1.05);box-shadow:0 6px 12px rgba(0,0,0,0.2);}
.card span{display:block;font-size:24px;margin-bottom:5px;}
table{border-collapse: collapse;width:100%;margin-top:20px;background:#fff;border-radius:8px;overflow:hidden;box-shadow:0 4px 10px rgba(0,0,0,0.1);}
th,td{border:1px solid #e0e0e0;padding:8px;text-align:center;color:#333;}
th{background:#b2dfdb;}
.progress-bar {height:25px;background:#c8e6c9;border-radius:12px;overflow:hidden;margin:10px 0;}
.progress {height:100%;background:#00796b;width:0%;color:#fff;text-align:center;line-height:25px;font-weight:bold;transition:0.5s;}
</style>
</head>
<body>

<h1>🌟 PocketTracker Dashboard 🌟</h1>
<div id="datetime"></div>

<div class="dashboard">
    <div class="card" title="Add Food" onclick="addDaily('food')"><span>🍔</span>Food</div>
    <div class="card" title="Add Fuel" onclick="addDaily('fuel')"><span>⛽</span>Fuel</div>
    <div class="card" title="Add Snacks" onclick="addDaily('snacks')"><span>🍿</span>Snacks</div>
    <div class="card" title="Add Bills" onclick="addDaily('bills')"><span>💡</span>Bills</div>
    <div class="card" title="Add Entertainment" onclick="addDaily('entertainment')"><span>🎮</span>Fun</div>
    <div class="card" title="Loan Paid Toggle" onclick="toggleLoan()"><span id="loanIcon">💰</span>Loan</div>
    <div class="card" title="Clear All Data" onclick="clearAll()"><span>🗑️</span>Clear</div>
    <div class="card" title="Export CSV" onclick="exportCSV()"><span>📊</span>Export</div>
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
<tr>
<th>Day</th><th>Food</th><th>Fuel</th><th>Snacks</th><th>Bills</th><th>Entertainment</th>
<th>Daily Total</th><th>Total incl. Ghar+Loan</th>
</tr>
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

// Data
let dailyData = [];
let loanPaidAmount = 0;
const salary = 49800;
const house = 20000;
const loanDefault = 10000;
const goal = 10000;

if(localStorage.getItem('dailyData')) dailyData = JSON.parse(localStorage.getItem('dailyData'));
if(localStorage.getItem('loanPaid')) loanPaidAmount = parseInt(localStorage.getItem('loanPaid')) || 0;
updateLoanIcon();
calculate();

// Add daily expense
function addDaily(type){
    const value = parseInt(prompt(`Enter ${type} amount in PKR:`,"0")) || 0;
    if(dailyData.length===0) dailyData.push({food:0,fuel:0,snacks:0,bills:0,entertainment:0,total:0});
    let today = dailyData[dailyData.length-1];
    today[type] += value;
    today.total = today.food+today.fuel+today.snacks+today.bills+today.entertainment;
    localStorage.setItem('dailyData',JSON.stringify(dailyData));
    calculate();
}

// Loan toggle
function toggleLoan(){
    if(loanPaidAmount===0){
        loanPaidAmount=loanDefault;
    } else {
        loanPaidAmount=0;
    }
    localStorage.setItem('loanPaid',loanPaidAmount);
    updateLoanIcon();
    calculate();
}
function updateLoanIcon(){
    document.getElementById('loanIcon').innerText = loanPaidAmount>0 ? '✅' : '💰';
}

// Clear all
function clearAll(){
    if(confirm("Delete all data?")){
        dailyData=[];
        loanPaidAmount=0;
        localStorage.removeItem('dailyData');
        localStorage.removeItem('loanPaid');
        updateLoanIcon();
        calculate();
    }
}

// Calculate
function calculate(){
    const table=document.getElementById('dailyTable');
    table.innerHTML="<tr><th>Day</th><th>Food</th><th>Fuel</th><th>Snacks</th><th>Bills</th><th>Entertainment</th><th>Daily Total</th><th>Total incl. Ghar+Loan</th></tr>";
    let totalExpense = house + loanPaidAmount;
    dailyData.forEach((day,index)=>{
        const row = table.insertRow();
        row.insertCell(0).innerText = index+1;
        row.insertCell(1).innerText = day.food;
        row.insertCell(2).innerText = day.fuel;
        row.insertCell(3).innerText = day.snacks;
        row.insertCell(4).innerText = day.bills;
        row.insertCell(5).innerText = day.entertainment;
        row.insertCell(6).innerText = day.total;
        const totalWithLoan = day.total + Math.round(house/dailyData.length) + Math.round(loanPaidAmount/dailyData.length || 0);
        row.insertCell(7).innerText = totalWithLoan;
        totalExpense += day.total;
    });
    const remaining = salary - totalExpense;
    document.getElementById('totalSalary').innerText = salary;
    document.getElementById('totalExpense').innerText = totalExpense;
    document.getElementById('remaining').innerText = remaining;
    const goalPercent = Math.min(Math.round((remaining/goal)*100),100);
    document.getElementById('goalStatus').innerText = goalPercent+'%';
    document.getElementById('progress').style.width = goalPercent+'%';
    document.getElementById('progress').innerText = goalPercent+'%';

    // Chart
    const ctx=document.getElementById('chart').getContext('2d');
    if(window.barChart) window.barChart.destroy();
    window.barChart=new Chart(ctx,{
        type:'bar',
        data:{
            labels:['Salary','Total Expense','Remaining Saving'],
            datasets:[{label:'PKR',data:[salary,totalExpense,remaining],backgroundColor:['#00796b','#f39c12','#2ecc71'],borderColor:['#004d40','#d35400','#27ae60'],borderWidth:2}]
        },
        options:{responsive:true,plugins:{legend:{display:false}},scales:{y:{beginAtZero:true}}}
    });
}

// Export CSV
function exportCSV(){
    let csv='Day,Food,Fuel,Snacks,Bills,Entertainment,Daily Total,Total incl. Ghar+Loan\n';
    dailyData.forEach((d,index)=>{
        const totalWithLoan = d.total + Math.round(house/dailyData.length) + Math.round(loanPaidAmount/dailyData.length || 0);
        csv+=`${index+1},${d.food},${d.fuel},${d.snacks},${d.bills},${d.entertainment},${d.total},${totalWithLoan}\n`;
    });
    let blob=new Blob([csv],{type:'text/csv'});
    let link=document.createElement('a');
    link.href=URL.createObjectURL(blob);
    link.download='PocketTracker_Month.csv';
    link.click();
}
</script>
</body>
</html>
