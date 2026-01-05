<pocket tracker>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Just Pocket Tracker</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
/* ===== Base ===== */
body{font-family:'Arial',sans-serif;margin:0;padding:0;background:#f0f2f5;color:#333;}
h2{text-align:center;margin:10px;font-weight:700;}
input,button{padding:8px;margin:5px;border-radius:8px;border:1px solid #ccc;}
button{cursor:pointer;font-weight:700;}
#dashboard,#licenseScreen{display:none;}

/* ===== Header ===== */
header{
    background:linear-gradient(90deg,#4caf50,#2196f3);
    padding:20px;
    text-align:center;
    color:#fff;
    font-size:32px;
    font-weight:bold;
    letter-spacing:1px;
    box-shadow:0 4px 10px rgba(0,0,0,0.2);
}

/* ===== Info Boxes ===== */
.infoBox{
    display:flex;
    flex-wrap:wrap;
    justify-content:center;
    margin:20px 0;
}
.infoBox div{
    background:linear-gradient(135deg,#ff9800,#ff5722);
    color:#fff;
    margin:10px;
    padding:20px;
    border-radius:15px;
    min-width:150px;
    text-align:center;
    box-shadow:0 6px 15px rgba(0,0,0,0.2);
    font-weight:bold;
    transition:0.3s;
}
.infoBox div:hover{transform:scale(1.05);}

/* ===== Buttons ===== */
button.addBtn{background:#4caf50;color:#fff;border:none;}
button.clearBtn{background:#f44336;color:#fff;border:none;}

/* ===== Table ===== */
table{
    width:90%;
    margin:10px auto;
    border-collapse:collapse;
    border-radius:8px;
    overflow:hidden;
    box-shadow:0 4px 10px rgba(0,0,0,0.1);
}
th,td{border:1px solid #ccc;padding:8px;text-align:center;}
th{background:#2196f3;color:#fff;}
tr:nth-child(even){background:#f9f9f9;}
</style>
</head>
<body>

<header>💼 Just Pocket Tracker 💼</header>
<div id="datetime" style="text-align:center;margin-bottom:10px;"></div>

<!-- Login/Register -->
<div id="loginScreen">
    <h2>Login / Register</h2>
    Username: <input type="text" id="loginUser"><br>
    Password: <input type="password" id="loginPass"><br>
    <button onclick="login()">Login</button>
    <button onclick="register()">Register</button>
    <p id="loginMsg" style="color:#ff9800;"></p>
</div>

<!-- License -->
<div id="licenseScreen">
    <h2>Enter License Key</h2>
    <input type="text" id="licenseInput" placeholder="XXXX-XXXX"><br>
    <button onclick="checkLicense()">Unlock Dashboard</button>
    <p id="licenseMsg" style="color:#ff9800;"></p>
</div>

<!-- Dashboard -->
<div id="dashboard">
    <h2 id="welcomeUser"></h2>
    <div style="text-align:center;margin-bottom:10px;">
        Salary: <input type="number" id="salaryInput" value="0" onchange="updateSalary()">
        Goal Saving: <input type="number" id="goalInput" value="10000" onchange="updateGoal()">
        Loan: <input type="number" id="loanInput" value="0" onchange="updateLoan()"><br>
        <button onclick="logout()">Logout</button>
    </div>

    <h3>💰 Dashboard Summary</h3>
    <div class="infoBox">
        <div>Total Expense<br><span id="totalExpense">0</span></div>
        <div>Remaining Balance<br><span id="remaining">0</span></div>
        <div>Current Saving<br><span id="currentSaving">0</span></div>
        <div>Goal Achieved<br><span id="goalStatus">0%</span></div>
    </div>

    <h3>Daily Expenses</h3>
    <div style="text-align:center;">
        <button class="addBtn" onclick="addDaily('food')">Food 🍔</button>
        <button class="addBtn" onclick="addDaily('fuel')">Fuel ⛽</button>
        <button class="addBtn" onclick="addDaily('snacks')">Snacks 🍿</button>
        <button class="addBtn" onclick="addDaily('bills')">Bills 💡</button>
        <button class="addBtn" onclick="addDaily('fun')">Fun 🎮</button>
        <button class="clearBtn" onclick="clearAll()">Clear All 🗑️</button>
    </div>

    <table id="dailyTable">
        <tr><th>Day</th><th>Food</th><th>Fuel</th><th>Snacks</th><th>Bills</th><th>Fun</th><th>Total</th><th>Date</th></tr>
    </table>

    <canvas id="chart" width="400" height="200" style="display:block;margin:20px auto;"></canvas>
</div>

<script>
// ===== Timer =====
function updateDateTime(){
    const now=new Date();
    document.getElementById('datetime').innerText=now.toLocaleDateString()+' | '+now.toLocaleTimeString();
}
setInterval(updateDateTime,1000);
updateDateTime();

// ===== Users =====
let users=JSON.parse(localStorage.getItem('users')||"{}");
let currentUser=null;
let dailyData=[],salary=0,goal=10000,loan=0;

// ===== Login/Register =====
function login(){
    const u=document.getElementById('loginUser').value.trim();
    const p=document.getElementById('loginPass').value;
    if(users[u] && users[u].pass===p){currentUser=u;localStorage.setItem('currentUser',u);loadUserData();}
    else{document.getElementById('loginMsg').innerText='Invalid login!';}
}
function register(){
    const u=document.getElementById('loginUser').value.trim();
    const p=document.getElementById('loginPass').value;
    if(!u || !p){document.getElementById('loginMsg').innerText='Enter username & password!'; return;}
    if(users[u]){document.getElementById('loginMsg').innerText='User exists!'; return;}
    let key=Math.random().toString(36).substring(2,10).toUpperCase();
    users[u]={pass:p,license:false,licenseKey:key,dailyData:[],salary:0,goal:10000,loan:0};
    localStorage.setItem('users',JSON.stringify(users));
    document.getElementById('loginMsg').innerText=`Registered! License Key: ${key}`;
}

// ===== Load User =====
window.onload=function(){
    const savedUser=localStorage.getItem('currentUser');
    if(savedUser && users[savedUser]){currentUser=savedUser;loadUserData();}
}
function loadUserData(){
    document.getElementById('loginScreen').style.display='none';
    if(!users[currentUser].license){document.getElementById('licenseScreen').style.display='block';}
    else{showDashboard();}
}

// ===== License =====
function checkLicense(){
    const key=document.getElementById('licenseInput').value.trim();
    if(key===users[currentUser].licenseKey){users[currentUser].license=true;localStorage.setItem('users',JSON.stringify(users));showDashboard();}
    else{document.getElementById('licenseMsg').innerText='Invalid License!';}
}

// ===== Dashboard =====
function showDashboard(){
    document.getElementById('licenseScreen').style.display='none';
    document.getElementById('dashboard').style.display='block';
    document.getElementById('welcomeUser').innerText=`Welcome, ${currentUser}!`;
    const u=users[currentUser];
    dailyData=u.dailyData||[];
    salary=u.salary||0;
    goal=u.goal||10000;
    loan=u.loan||0;
    document.getElementById('salaryInput').value=salary;
    document.getElementById('goalInput').value=goal;
    document.getElementById('loanInput').value=loan;
    calculate();
}

// ===== Logout =====
function logout(){localStorage.removeItem('currentUser');currentUser=null;document.getElementById('dashboard').style.display='none';document.getElementById('loginScreen').style.display='block';}

// ===== Expenses =====
function addDaily(type){
    const val=parseInt(prompt(`Enter ${type} amount:`,"0"))||0;
    const today={food:0,fuel:0,snacks:0,bills:0,fun:0,total:0,date:new Date().toLocaleDateString()};
    if(dailyData.length>0 && dailyData[dailyData.length-1].date===today.date){
        dailyData[dailyData.length-1][type]+=val;
        let d=dailyData[dailyData.length-1];
        d.total=d.food+d.fuel+d.snacks+d.bills+d.fun;
    } else {today[type]=val;today.total=today.food+today.fuel+today.snacks+today.bills+today.fun;dailyData.push(today);}
    saveUser(); calculate();
}
function updateSalary(){salary=parseInt(document.getElementById('salaryInput').value)||0;saveUser(); calculate();}
function updateGoal(){goal=parseInt(document.getElementById('goalInput').value)||10000;saveUser(); calculate();}
function updateLoan(){loan=parseInt(document.getElementById('loanInput').value)||0;saveUser(); calculate();}
function clearAll(){if(confirm("Delete all data?")){dailyData=[];saveUser();calculate();}}
function saveUser(){users[currentUser].dailyData=dailyData;users[currentUser].salary=salary;users[currentUser].goal=goal;users[currentUser].loan=loan;localStorage.setItem('users',JSON.stringify(users));}

// ===== Calculate & Table =====
function calculate(){
    let totalExpense=loan;
    const table=document.getElementById('dailyTable');
    table.innerHTML="<tr><th>Day</th><th>Food</th><th>Fuel</th><th>Snacks</th><th>Bills</th><th>Fun</th><th>Total</th><th>Date</th></tr>";
    dailyData.forEach((day,index)=>{
        const row=table.insertRow();
        row.insertCell(0).innerText=index+1;
        row.insertCell(1).innerText=day.food;
        row.insertCell(2).innerText=day.fuel;
        row.insertCell(3).innerText=day.snacks;
        row.insertCell(4).innerText=day.bills;
        row.insertCell(5).innerText=day.fun;
        row.insertCell(6).innerText=day.total;
        row.insertCell(7).innerText=day.date;
        totalExpense+=day.total;
    });
    const remaining=salary-totalExpense;
    const saving=Math.max(0,remaining);
    document.getElementById('totalExpense').innerText=totalExpense;
    document.getElementById('remaining').innerText=remaining;
    document.getElementById('currentSaving').innerText=saving;
    document.getElementById('goalStatus').innerText=Math.min(Math.round((saving/goal)*100),100)+'%';

    // Chart
    const ctx=document.getElementById('chart').getContext('2d');
    if(window.barChart) window.barChart.destroy();
    window.barChart=new Chart(ctx,{type:'bar',data:{labels:['Salary','Expense','Saving'],datasets:[{label:'PKR',data:[salary,totalExpense,saving],backgroundColor:['#4caf50','#f44336','#2196f3']}]},options:{responsive:true,plugins:{legend:{display:true},title:{display:true,text:'PocketTracker Overview'}},scales:{y:{beginAtZero:true}}}});
}
</script>

</body>
</html>
