<Monthly>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>PocketTracker Multi-User Dashboard</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body {font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;background: linear-gradient(135deg,#e0f7fa,#80deea);margin:0;padding:20px;color:#333;}
h1{text-align:center;color:#00796b;}
#datetime{text-align:center;font-weight:bold;margin-bottom:20px;color:#004d40;}
.dashboard{display:flex;flex-wrap:wrap;justify-content:center;margin-bottom:20px;}
.card{background:#fff;box-shadow:0 4px 8px rgba(0,0,0,0.1);border-radius:12px;margin:10px;padding:15px;text-align:center;width:120px;cursor:pointer;transition:0.3s;}
.card:hover{transform:scale(1.05);box-shadow:0 6px 12px rgba(0,0,0,0.2);}
.card span{display:block;font-size:24px;margin-bottom:5px;}
table{border-collapse: collapse;width:100%;margin-top:20px;background:#fff;border-radius:8px;overflow:hidden;box-shadow:0 4px 10px rgba(0,0,0,0.1);}
th,td{border:1px solid #e0e0e0;padding:8px;text-align:center;color:#333;}
th{background:#b2dfdb;}
.progress-bar{height:25px;background:#c8e6c9;border-radius:12px;overflow:hidden;margin:10px 0;}
.progress{height:100%;background:#00796b;width:0%;color:#fff;text-align:center;line-height:25px;font-weight:bold;transition:0.5s;}
input.salaryInput,input.loginInput,input.licenseInput{width:80px;text-align:center;border-radius:8px;border:2px solid #4a90e2;padding:5px;outline:none;transition:0.3s;margin-top:5px;}
input.salaryInput:focus,input.loginInput:focus,input.licenseInput:focus{border-color:#357ABD; box-shadow:0 0 5px #357ABD;}
#loginScreen,#licenseScreen,#dashboard{display:none;}
button{padding:5px 10px;margin-top:5px;border-radius:8px;border:none;background:#00796b;color:#fff;cursor:pointer;transition:0.3s;}
button:hover{background:#004d40;}
</style>
</head>
<body>

<h1>🌟 PocketTracker Multi-User 🌟</h1>

<div id="datetime"></div>

<!-- Login Screen -->
<div id="loginScreen">
    <h2>Login</h2>
    Username: <input type="text" id="loginUser" class="loginInput"><br>
    Password: <input type="password" id="loginPass" class="loginInput"><br>
    <button onclick="login()">Login</button>
    <button onclick="register()">Register</button>
    <p id="loginMsg" style="color:red;"></p>
</div>

<!-- License Screen -->
<div id="licenseScreen">
    <h2>Enter License Key</h2>
    <input type="text" id="licenseInput" class="licenseInput" placeholder="XXXX-XXXX"><br>
    <button onclick="checkLicense()">Unlock Dashboard</button>
    <p id="licenseMsg" style="color:red;"></p>
</div>

<!-- Dashboard -->
<div id="dashboard">
    <div class="dashboard">
        <div class="card" title="Add Food" onclick="addDaily('food')"><span>🍔</span>Food</div>
        <div class="card" title="Add Fuel" onclick="addDaily('fuel')"><span>⛽</span>Fuel</div>
        <div class="card" title="Add Snacks" onclick="addDaily('snacks')"><span>🍿</span>Snacks</div>
        <div class="card" title="Add Bills" onclick="addDaily('bills')"><span>💡</span>Bills</div>
        <div class="card" title="Add Entertainment" onclick="addDaily('entertainment')"><span>🎮</span>Fun</div>
        <div class="card" title="Loan Paid Toggle" onclick="toggleLoan()"><span id="loanIcon">💰</span>Loan</div>
        <div class="card" title="Clear All Data" onclick="clearAll()"><span>🗑️</span>Clear</div>
        <div class="card" title="Export CSV" onclick="exportCSV()"><span>📊</span>Export</div>
        <div class="card" title="Update Salary"><span>💰</span>Salary<br><input type="number" class="salaryInput" id="salaryInput" value="49800" onchange="updateSalary()"></div>
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
    <th>Daily Total</th><th>Total incl. Ghar+Loan</th><th>Date</th>
    </tr>
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
</div>

<script>
// Timer
function updateDateTime(){
    const now=new Date();
    const options={weekday:'long',year:'numeric',month:'long',day:'numeric'};
    document.getElementById('datetime').innerText=now.toLocaleDateString('en-US',options)+' | '+now.toLocaleTimeString();
}
setInterval(updateDateTime,1000);
updateDateTime();

// Multi-User Data
let currentUser=null;
let users = JSON.parse(localStorage.getItem('users')||"{}");
let dailyData=[],loanPaidAmount=0,salary=49800,house=20000,loanDefault=10000,goal=10000;

// Show login screen
document.getElementById('loginScreen').style.display='block';

// Login/Register
function login(){
    const user=document.getElementById('loginUser').value.trim();
    const pass=document.getElementById('loginPass').value;
    if(users[user] && users[user].pass===pass){
        currentUser=user;
        loadUserData();
    } else {document.getElementById('loginMsg').innerText='Invalid login!';}
}
function register(){
    const user=document.getElementById('loginUser').value.trim();
    const pass=document.getElementById('loginPass').value;
    if(user && pass){
        if(users[user]){document.getElementById('loginMsg').innerText='User exists!'; return;}
        users[user]={pass:pass,license:false,dailyData:[],salary:49800,loanPaid:0};
        localStorage.setItem('users',JSON.stringify(users));
        document.getElementById('loginMsg').innerText='Registered! Login now.';
    }
}

// Load User Data
function loadUserData(){
    document.getElementById('loginScreen').style.display='none';
    if(!users[currentUser].license){
        document.getElementById('licenseScreen').style.display='block';
    } else {showDashboard();}
}

// License Check
function checkLicense(){
    const key=document.getElementById('licenseInput').value.trim();
    if(key==='ABCD-1234'){ // example license
        users[currentUser].license=true;
        localStorage.setItem('users',JSON.stringify(users));
        showDashboard();
    } else {document.getElementById('licenseMsg').innerText='Invalid License!';}
}

// Show Dashboard
function showDashboard(){
    document.getElementById('licenseScreen').style.display='none';
    document.getElementById('dashboard').style.display='block';
    loadDashboardData();
}

function loadDashboardData(){
    const u=users[currentUser];
    dailyData=u.dailyData||[];
    loanPaidAmount=u.loanPaid||0;
    salary=u.salary||49800;
    document.getElementById('salaryInput').value=salary;
    updateLoanIcon();
    calculate();
}

// Save User Data
function saveUserData(){
    users[currentUser].dailyData=dailyData;
    users[currentUser].loanPaid=loanPaidAmount;
    users[currentUser].salary=salary;
    localStorage.setItem('users',JSON.stringify(users));
}

// Add daily expense
function addDaily(type){
    const value=parseInt(prompt(`Enter ${type} amount in PKR:`,"0"))||0;
    const today={food:0,fuel:0,snacks:0,bills:0,entertainment:0,total:0,date:new Date().toLocaleDateString()};
    if(dailyData.length>0 && dailyData[dailyData.length-1].date===today.date){
        let last=dailyData[dailyData.length-1];
        last[type]+=value;
        last.total=last.food+last.fuel+last.snacks+last.bills+last.entertainment;
    } else {today[type]=value;today.total=today.food+today.fuel+today.snacks+today.bills+today.entertainment;dailyData.push(today);}
    saveUserData();
    calculate();
}

// Loan toggle
function toggleLoan(){
    loanPaidAmount=loanPaidAmount===0?loanDefault:0;
    saveUserData();
    updateLoanIcon();
    calculate();
}
function updateLoanIcon(){document.getElementById('loanIcon').innerText=loanPaidAmount>0?'✅':'💰';}

// Update salary
function updateSalary(){salary=parseInt(document.getElementById('salaryInput').value);saveUserData();calculate();}

// Clear all
function clearAll(){if(confirm("Delete all data?")){dailyData=[];loanPaidAmount=0;users[currentUser].license=users[currentUser].license;saveUserData();calculate();}}

// Calculate
function calculate(){
    const table=document.getElementById('dailyTable');table.innerHTML="<tr><th>Day</th><th>Food</th><th>Fuel</th><th>Snacks</th><th>Bills</th><th>Entertainment</th><th>Daily Total</th><th>Total incl. Ghar+Loan</th><th>Date</th></tr>";
    let totalExpense=house+loanPaidAmount;
    dailyData.forEach((day,index)=>{
        const row=table.insertRow();
        row.insertCell(0).innerText=index+1;
        row.insertCell(1).innerText=day.food;
        row.insertCell(2).innerText=day.fuel;
        row.insertCell(3).innerText=day.snacks;
        row.insertCell(4).innerText=day.bills;
        row.insertCell(5).innerText=day.entertainment;
        row.insertCell(6).innerText=day.total;
        const totalWithLoan=day.total+Math.round(house/dailyData.length)+Math.round(loanPaidAmount/dailyData.length||0);
        row.insertCell(7).innerText=totalWithLoan;
        row.insertCell(8).innerText=day.date;
        totalExpense+=day.total;
    });
    const remaining=salary-totalExpense;
    document.getElementById('totalSalary').innerText=salary;
    document.getElementById('totalExpense').innerText=totalExpense;
    document.getElementById('remaining').innerText=remaining;
    const goalPercent=Math.min(Math.round((remaining/goal)*100),100);
    document.getElementById('goalStatus').innerText=goalPercent+'%';
    document.getElementById('progress').style.width=goalPercent+'%';
    document.getElementById('progress').innerText=goalPercent+'%';
    
    // Chart
    const ctx=document.getElementById('chart').getContext('2d');
    if(window.barChart) window.barChart.destroy();
    window.barChart=new Chart(ctx,{type:'bar',data:{labels:['Salary','Total Expense','Remaining Saving'],datasets:[{label:'PKR',data:[salary,totalExpense,remaining],backgroundColor:['#00796b','#f39c12','#2ecc71'],borderColor:['#004d40','#d35400','#27ae60'],borderWidth:2}]},options:{responsive:true,plugins:{legend:{display:false}},scales:{y:{beginAtZero:true}}}});
}

// Export CSV
function exportCSV(){
    let csv='Day,Food,Fuel,Snacks,Bills,Entertainment,Daily Total,Total incl. Ghar+Loan,Date\n';
    dailyData.forEach((d,index)=>{
        const totalWithLoan=d.total+Math.round(house/dailyData.length)+Math.round(loanPaidAmount/dailyData.length||0);
        csv+=`${index+1},${d.food},${d.fuel},${d.snacks},${d.bills},${d.entertainment},${d.total},${totalWithLoan},${d.date}\n`;
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
