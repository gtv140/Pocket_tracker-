<Monthly>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>PocketTracker Ultimate Premium</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
/* ===== Base Styles ===== */
body{
    font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    margin:0;padding:0;
    background: linear-gradient(120deg,#1e3c72,#2a5298);
    color:#fff;
}
h1,h2{text-align:center;margin:10px;}
#datetime{text-align:center;font-weight:bold;margin-bottom:15px;color:#ffeb3b;}
button{cursor:pointer;transition:0.3s;}
button:hover{opacity:0.8;}
input{padding:5px;border-radius:6px;border:2px solid #03a9f4;outline:none;margin:3px;}
input:focus{border-color:#0288d1;box-shadow:0 0 5px #0288d1;}

/* ===== Dashboard Styles ===== */
.dashboard{display:flex;flex-wrap:wrap;justify-content:center;margin:20px 0;}
.card{
    background:linear-gradient(135deg,#29b6f6,#00acc1);
    box-shadow:0 4px 12px rgba(0,0,0,0.25);
    border-radius:12px;
    margin:10px;
    padding:15px;
    text-align:center;
    width:120px;
    cursor:pointer;
    transition:0.3s;
    font-weight:bold;
}
.card:hover{transform:scale(1.05);box-shadow:0 6px 16px rgba(0,0,0,0.35);}
.card span{display:block;font-size:30px;margin-bottom:5px;}

.infoBox{
    display:flex;
    flex-wrap:wrap;
    justify-content:space-around;
    margin:20px;
    background:rgba(255,255,255,0.1);
    padding:15px;
    border-radius:12px;
    box-shadow:0 4px 12px rgba(0,0,0,0.25);
}
.infoBox div{
    text-align:center;
    background:linear-gradient(135deg,#00acc1,#29b6f6);
    color:#fff;
    padding:15px;
    margin:5px;
    border-radius:12px;
    flex:1;
    min-width:140px;
    box-shadow:0 4px 12px rgba(0,0,0,0.25);
    transition:0.3s;font-weight:bold;
}
.infoBox div:hover{transform:scale(1.05);}

/* ===== Table ===== */
table{
    border-collapse: collapse;
    width:100%;
    margin-top:20px;
    background:#fff;
    color:#333;
    border-radius:8px;
    overflow:hidden;
    box-shadow:0 4px 10px rgba(0,0,0,0.1);
}
th,td{border:1px solid #e0e0e0;padding:8px;text-align:center;}
th{background:#0288d1;color:#fff;}

/* ===== Theme Toggle ===== */
#themeToggle{position:fixed;top:10px;right:10px;padding:6px 10px;background:#0288d1;color:#fff;border-radius:6px;}
</style>
</head>
<body>

<h1>💼 PocketTracker Ultimate Premium 💼</h1>
<div id="datetime"></div>
<button id="themeToggle" onclick="toggleTheme()">Toggle Theme</button>

<!-- Login / Register -->
<div id="loginScreen">
    <h2>Login / Register</h2>
    Username: <input type="text" id="loginUser"><br>
    Password: <input type="password" id="loginPass"><br>
    <button onclick="login()">Login</button>
    <button onclick="register()">Register</button>
    <p id="loginMsg" style="color:#ffeb3b;"></p>
</div>

<!-- License Screen -->
<div id="licenseScreen">
    <h2>Enter License Key</h2>
    <input type="text" id="licenseInput" placeholder="XXXX-XXXX"><br>
    <button onclick="checkLicense()">Unlock Dashboard</button>
    <p id="licenseMsg" style="color:#ffeb3b;"></p>
</div>

<!-- Dashboard -->
<div id="dashboard" style="display:none;">
    <div style="text-align:right;margin-bottom:10px;">
        <span id="welcomeUser" style="font-weight:bold;font-size:18px;"></span>
        <button onclick="logout()">Logout</button>
    </div>

    <h2>💰 User Info & Goals</h2>
    <div class="infoBox">
        <div>Username<br><span id="infoUser">-</span></div>
        <div>Salary<br><input type="number" id="salaryInput" value="0" onchange="updateSalary()"></div>
        <div>Goal Saving<br><input type="number" id="goalInput" value="10000" onchange="updateGoal()"></div>
        <div>Loan Amount<br><input type="number" id="loanInput" value="0" onchange="updateLoan()"></div>
    </div>

    <h2>Dashboard Summary</h2>
    <div class="infoBox">
        <div>Total Expense<br><span id="totalExpense">0</span></div>
        <div>Remaining Balance<br><span id="remaining">0</span></div>
        <div>Current Saving<br><span id="currentSaving">0</span></div>
        <div>Goal Achieved<br><span id="goalStatus">0%</span></div>
    </div>

    <h2>Daily Expenses</h2>
    <div class="dashboard" id="expenseCards">
        <div class="card" title="Add Food" onclick="addDaily('food')"><span>🍔</span>Food</div>
        <div class="card" title="Add Fuel" onclick="addDaily('fuel')"><span>⛽</span>Fuel</div>
        <div class="card" title="Add Snacks" onclick="addDaily('snacks')"><span>🍿</span>Snacks</div>
        <div class="card" title="Add Bills" onclick="addDaily('bills')"><span>💡</span>Bills</div>
        <div class="card" title="Add Entertainment" onclick="addDaily('entertainment')"><span>🎮</span>Fun</div>
        <div class="card" title="Clear All Data" onclick="clearAll()"><span>🗑️</span>Clear</div>
        <div class="card" title="Export CSV" onclick="exportCSV()"><span>📊</span>Export</div>
        <div class="card" title="Backup JSON" onclick="backupJSON()"><span>💾</span>Backup</div>
        <div class="card" title="Restore JSON" onclick="restoreJSON()"><span>📂</span>Restore</div>
    </div>

    <h2>Daily Expenses Table</h2>
    <table id="dailyTable">
        <tr><th>Day</th><th>Food</th><th>Fuel</th><th>Snacks</th><th>Bills</th><th>Entertainment</th><th>Daily Total</th><th>Total incl. Loan</th><th>Date</th></tr>
    </table>

    <h2>Visual Overview</h2>
    <canvas id="chart" width="400" height="200"></canvas>
</div>

<script>
// ===== Timer =====
function updateDateTime(){
    const now=new Date();
    const options={weekday:'long',year:'numeric',month:'long',day:'numeric'};
    document.getElementById('datetime').innerText=now.toLocaleDateString('en-US',options)+' | '+now.toLocaleTimeString();
}
setInterval(updateDateTime,1000);
updateDateTime();

// ===== Theme Toggle =====
let darkTheme=true;
function toggleTheme(){
    if(darkTheme){
        document.body.style.background="#fff";
        document.body.style.color="#333";
        darkTheme=false;
    }else{
        document.body.style.background="linear-gradient(120deg,#1e3c72,#2a5298)";
        document.body.style.color="#fff";
        darkTheme=true;
    }
}

// ===== Multi-User & License =====
let currentUser=null;
let users=JSON.parse(localStorage.getItem('users')||"{}");
let dailyData=[],loanAmount=0,salary=0,goal=10000;

// Generate License Key
function generateLicenseKey(){
    const chars='ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    let key='';
    for(let i=0;i<4;i++) key+=chars[Math.floor(Math.random()*chars.length)];
    key+='-';
    for(let i=0;i<4;i++) key+=chars[Math.floor(Math.random()*chars.length)];
    return key;
}

// Persistent login
window.onload=function(){
    const savedUser=localStorage.getItem('currentUser');
    if(savedUser && users[savedUser]){
        currentUser=savedUser;
        loadUserData();
    } else {document.getElementById('loginScreen').style.display='block';}
}

// Login/Register
function login(){
    const user=document.getElementById('loginUser').value.trim();
    const pass=document.getElementById('loginPass').value;
    if(users[user] && users[user].pass===pass){
        currentUser=user;
        localStorage.setItem('currentUser',currentUser);
        loadUserData();
    } else {document.getElementById('loginMsg').innerText='Invalid login!';}
}
function register(){
    const user=document.getElementById('loginUser').value.trim();
    const pass=document.getElementById('loginPass').value;
    if(user && pass){
        if(users[user]){document.getElementById('loginMsg').innerText='User exists!'; return;}
        const licenseKey=generateLicenseKey();
        users[user]={pass:pass,license:false,licenseKey:licenseKey,dailyData:[],salary:0,goal:10000,loan:0};
        localStorage.setItem('users',JSON.stringify(users));
        document.getElementById('loginMsg').innerText=`Registered! Your License Key: ${licenseKey}`;
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
    if(key===users[currentUser].licenseKey){
        users[currentUser].license=true;
        localStorage.setItem('users',JSON.stringify(users));
        showDashboard();
    } else {document.getElementById('licenseMsg').innerText='Invalid License!';}
}

// Show Dashboard
function showDashboard(){
    document.getElementById('licenseScreen').style.display='none';
    document.getElementById('dashboard').style.display='block';
    document.getElementById('welcomeUser').innerText=`Welcome, ${currentUser}!`;
    const u=users[currentUser];
    dailyData=u.dailyData||[];
    salary=u.salary||0;
    goal=u.goal||10000;
    loanAmount=u.loan||0;
    document.getElementById('salaryInput').value=salary;
    document.getElementById('goalInput').value=goal;
    document.getElementById('loanInput').value=loanAmount;
    document.getElementById('infoUser').innerText=currentUser;
    calculate();
}

// Logout
function logout(){
    localStorage.removeItem('currentUser');
    currentUser=null;
    document.getElementById('dashboard').style.display='none';
    document.getElementById('loginScreen').style.display='block';
}

// Add Daily Expense
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

// Update Salary, Goal, Loan
function updateSalary(){salary=parseInt(document.getElementById('salaryInput').value)||0;saveUserData();calculate();}
function updateGoal(){goal=parseInt(document.getElementById('goalInput').value)||10000;saveUserData();calculate();}
function updateLoan(){loanAmount=parseInt(document.getElementById('loanInput').value)||0;saveUserData();calculate();}

// Save User Data
function saveUserData(){
    users[currentUser].dailyData=dailyData;
    users[currentUser].salary=salary;
    users[currentUser].goal=goal;
    users[currentUser].loan=loanAmount;
    localStorage.setItem('users',JSON.stringify(users));
}

// Clear All
function clearAll(){if(confirm("Delete all data?")){dailyData=[];saveUserData();calculate();}}

// Backup & Restore
function backupJSON(){
    const blob=new Blob([JSON.stringify(users)],{type:'application/json'});
    const link=document.createElement('a');
    link.href=URL.createObjectURL(blob);
    link.download='PocketTrackerBackup.json';
    link.click();
}
function restoreJSON(){
    const fileInput=document.createElement('input');
    fileInput.type='file';
    fileInput.accept='.json';
    fileInput.onchange=e=>{
        const file=e.target.files[0];
        const reader=new FileReader();
        reader.onload=ev=>{
            users=JSON.parse(ev.target.result);
            localStorage.setItem('users',JSON.stringify(users));
            alert('Backup restored! Refresh page to login.');
        };
        reader.readAsText(file);
    };
    fileInput.click();
}

// Calculate & Update Dashboard
function calculate(){
    const table=document.getElementById('dailyTable');
    table.innerHTML="<tr><th>Day</th><th>Food</th><th>Fuel</th><th>Snacks</th><th>Bills</th><th>Entertainment</th><th>Daily Total</th><th>Total incl. Loan</th><th>Date</th></tr>";
    let totalExpense=loanAmount;
    dailyData.forEach((day,index)=>{
        const row=table.insertRow();
        row.insertCell(0).innerText=index+1;
        row.insertCell(1).innerText=day.food;
        row.insertCell(2).innerText=day.fuel;
        row.insertCell(3).innerText=day.snacks;
        row.insertCell(4).innerText=day.bills;
        row.insertCell(5).innerText=day.entertainment;
        row.insertCell(6).innerText=day.total;
        const totalWithLoan=day.total + Math.round(loanAmount/dailyData.length||0);
        row.insertCell(7).innerText=totalWithLoan;
        row.insertCell(8).innerText=day.date;
        totalExpense+=day.total;
    });
    const remaining=salary-totalExpense;
    const currentSaving=Math.max(0,remaining);
    document.getElementById('totalExpense').innerText=totalExpense;
    document.getElementById('remaining').innerText=remaining;
    document.getElementById('currentSaving').innerText=currentSaving;
    const goalPercent=Math.min(Math.round((currentSaving/goal)*100),100);
    document.getElementById('goalStatus').innerText=goalPercent+'%';
    
    // Chart
    const ctx=document.getElementById('chart').getContext('2d');
    if(window.barChart) window.barChart.destroy();
    window.barChart=new Chart(ctx,{type:'bar',data:{labels:['Salary','Total Expense','Current Saving'],datasets:[{label:'PKR',data:[salary,totalExpense,currentSaving],backgroundColor:['#00acc1','#f39c12','#2ecc71'],borderColor:['#00796b','#d35400','#27ae60'],borderWidth:2}]},options:{responsive:true,plugins:{legend:{display:false}},scales:{y:{beginAtZero:true}}}});
}

// Export CSV
function exportCSV(){
    let csv='Day,Food,Fuel,Snacks,Bills,Entertainment,Daily Total,Total incl. Loan,Date\n';
    dailyData.forEach((d,index)=>{
        const totalWithLoan=d.total + Math.round(loanAmount/dailyData.length||0);
        csv+=`${index+1},${d.food},${d.fuel},${d.snacks},${d.bills},${d.entertainment},${d.total},${totalWithLoan},${d.date}\n`;
    });
    let blob=new Blob([csv],{type:'text/csv'});
    let link=document.createElement('a');
    link.href=URL.createObjectURL(blob);
    link.download='PocketTracker_Pro.csv';
    link.click();
}
</script>
</body>
</html>
