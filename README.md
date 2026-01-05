<pocket>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Pocket Traker</title>
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">
<style>
body{
    font-family:'Roboto',sans-serif;
    margin:0;
    padding:0;
    background: linear-gradient(135deg,#00c6ff,#0072ff);
    color:#fff;
}
header{
    text-align:center;
    padding:20px;
    font-size:32px;
    font-weight:bold;
    text-shadow: 2px 2px 5px rgba(0,0,0,0.3);
}
#loginScreen,#dashboard{max-width:800px;margin:auto;padding:20px;}
input,button{padding:8px 12px;margin:5px;border-radius:8px;border:none;}
button{cursor:pointer;font-weight:bold;background:#ff9800;color:#fff;}
button:hover{opacity:0.85;}
input{width:180px;outline:none;}
.boxes{display:flex;flex-wrap:wrap;justify-content:center;margin:15px 0;}
.box{
    flex:1;
    min-width:150px;
    margin:10px;
    padding:15px;
    text-align:center;
    border-radius:15px;
    background:rgba(255,255,255,0.1);
    box-shadow:0 6px 15px rgba(0,0,0,0.3);
    font-weight:bold;
}
.expenseBtn{flex:1;min-width:120px;margin:5px;padding:10px;background:#4caf50;color:#fff;border-radius:10px;}
.expenseBtn:hover{opacity:0.85;}
table{width:100%;border-collapse:collapse;margin-top:15px;background:rgba(255,255,255,0.95);color:#333;border-radius:10px;overflow:hidden;}
th,td{border:1px solid #ddd;padding:8px;text-align:center;}
th{background:#0288d1;color:#fff;}
tr:nth-child(even){background:rgba(0,172,193,0.1);}
tr:hover{background:rgba(0,172,193,0.2);}
#datetime{text-align:center;margin:10px;font-weight:bold;color:#ffeb3b;}
</style>
</head>
<body>

<header>📊 Pocket Traker 📊</header>
<div id="datetime"></div>

<!-- Login -->
<div id="loginScreen">
<h2>Login / Register</h2>
<input type="text" id="username" placeholder="Username"><br>
<input type="password" id="password" placeholder="Password"><br>
<button onclick="login()">Login</button>
<button onclick="register()">Register</button>
<p id="loginMsg" style="color:#ffeb3b;"></p>
</div>

<!-- Dashboard -->
<div id="dashboard" style="display:none;">
<div style="text-align:right;margin-bottom:10px;">
<span id="welcomeUser"></span>
<button onclick="logout()">Logout</button>
</div>

<div class="boxes">
<div class="box">Username<br><span id="userBox">-</span></div>
<div class="box">Salary<br><span id="salaryBox">0</span></div>
<div class="box">Current Balance<br><span id="balanceBox">0</span></div>
<div class="box">Saving<br><span id="savingBox">0</span></div>
<div class="box">Loan<br><span id="loanBox">0</span></div>
</div>

<h3>Add Daily Expense</h3>
<div class="boxes">
<button class="expenseBtn" onclick="addExpense('Food')">🍔 Food</button>
<button class="expenseBtn" onclick="addExpense('Fuel')">⛽ Fuel</button>
<button class="expenseBtn" onclick="addExpense('Snacks')">🍿 Snacks</button>
<button class="expenseBtn" onclick="addExpense('Bills')">💡 Bills</button>
<button class="expenseBtn" onclick="addExpense('Fun')">🎮 Fun</button>
<button class="expenseBtn" onclick="clearData()">🗑️ Clear All</button>
</div>

<h3>Daily Expenses Table</h3>
<table id="expenseTable">
<tr><th>Day</th><th>Food</th><th>Fuel</th><th>Snacks</th><th>Bills</th><th>Fun</th><th>Total</th><th>Date</th></tr>
</table>

</div>

<script>
// Timer
function updateDateTime(){
    const now=new Date();
    document.getElementById('datetime').innerText=now.toLocaleDateString() + ' | ' + now.toLocaleTimeString();
}
setInterval(updateDateTime,1000);
updateDateTime();

// Users
let users=JSON.parse(localStorage.getItem('users')||'{}');
let currentUser=null;

// Login/Register
function login(){
    const u=document.getElementById('username').value.trim();
    const p=document.getElementById('password').value;
    if(users[u] && users[u].pass===p){
        currentUser=u;
        localStorage.setItem('currentUser',currentUser);
        loadDashboard();
    } else {document.getElementById('loginMsg').innerText='Invalid login!';}
}
function register(){
    const u=document.getElementById('username').value.trim();
    const p=document.getElementById('password').value;
    if(u && p){
        if(users[u]){document.getElementById('loginMsg').innerText='User exists!'; return;}
        users[u]={pass:p,salary:0,loan:0,daily:[]};
        localStorage.setItem('users',JSON.stringify(users));
        document.getElementById('loginMsg').innerText='Registered! Login now.';
    }
}

// Load Dashboard
function loadDashboard(){
    document.getElementById('loginScreen').style.display='none';
    document.getElementById('dashboard').style.display='block';
    currentUser=currentUser||localStorage.getItem('currentUser');
    const userData=users[currentUser];
    document.getElementById('welcomeUser').innerText=`Welcome, ${currentUser}!`;
    document.getElementById('userBox').innerText=currentUser;
    document.getElementById('salaryBox').innerText=userData.salary;
    updateBalance();
    renderTable();
}

// Update Balance and Saving
function updateBalance(){
    const userData=users[currentUser];
    let totalExpense=0;
    userData.daily.forEach(d=>{totalExpense+=d.total;});
    const balance=userData.salary-totalExpense;
    const saving=Math.max(balance,0);
    document.getElementById('balanceBox').innerText=balance;
    document.getElementById('savingBox').innerText=saving;
    document.getElementById('loanBox').innerText=userData.loan;
}

// Add Expense
function addExpense(type){
    const amt=parseInt(prompt(`Enter ${type} amount:`,"0"))||0;
    const date=new Date().toLocaleDateString();
    let userData=users[currentUser];
    if(!userData.daily.length || userData.daily[userData.daily.length-1].date!==date){
        userData.daily.push({Food:0,Fuel:0,Snacks:0,Bills:0,Fun:0,total:0,date:date});
    }
    const today=userData.daily[userData.daily.length-1];
    today[type]+=amt;
    today.total=today.Food+today.Fuel+today.Snacks+today.Bills+today.Fun;
    users[currentUser]=userData;
    localStorage.setItem('users',JSON.stringify(users));
    updateBalance();
    renderTable();
}

// Render Table
function renderTable(){
    const userData=users[currentUser];
    const table=document.getElementById('expenseTable');
    table.innerHTML="<tr><th>Day</th><th>Food</th><th>Fuel</th><th>Snacks</th><th>Bills</th><th>Fun</th><th>Total</th><th>Date</th></tr>";
    userData.daily.forEach((d,i)=>{
        const row=table.insertRow();
        row.insertCell(0).innerText=i+1;
        row.insertCell(1).innerText=d.Food;
        row.insertCell(2).innerText=d.Fuel;
        row.insertCell(3).innerText=d.Snacks;
        row.insertCell(4).innerText=d.Bills;
        row.insertCell(5).innerText=d.Fun;
        row.insertCell(6).innerText=d.total;
        row.insertCell(7).innerText=d.date;
    });
}

// Clear All
function clearData(){
    if(confirm('Delete all daily data?')){
        users[currentUser].daily=[];
        localStorage.setItem('users',JSON.stringify(users));
        updateBalance();
        renderTable();
    }
}

// Logout
function logout(){
    localStorage.removeItem('currentUser');
    currentUser=null;
    document.getElementById('dashboard').style.display='none';
    document.getElementById('loginScreen').style.display='block';
}

// Auto load
window.onload=function(){
    const savedUser=localStorage.getItem('currentUser');
    if(savedUser && users[savedUser]){
        currentUser=savedUser;
        loadDashboard();
    }
};
</script>

</body>
</html>
