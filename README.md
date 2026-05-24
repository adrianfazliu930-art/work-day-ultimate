<!DOCTYPE html>
<html lang="sq">
<head>
<meta charset="UTF-8">
<title>WorkDay ULTIMATE | Smart Work Calendar</title>
<meta name="viewport" content="width=device-width, initial-scale=1">

<style>
:root{
 --bg:#f4f6f8;--card:#fff;--text:#1f2937;
 --work:#22c55e;--off:#ef4444;--holiday:#3b82f6;
}
body.dark{
 --bg:#020617;--card:#0f172a;--text:#e5e7eb;
}
*{box-sizing:border-box;font-family:Segoe UI,Tahoma}
body{margin:0;background:var(--bg);color:var(--text);transition:.3s}

header{
 background:linear-gradient(135deg,#14b8a6,#0f766e);
 color:#fff;padding:18px;
 display:flex;justify-content:space-between;align-items:center
}
header h1{margin:0;font-size:22px}
header .actions button{margin-left:8px}

button,select{
 padding:8px 14px;border-radius:8px;border:none;cursor:pointer
}
button{background:#14b8a6;color:#fff}

main{max-width:1200px;margin:auto;padding:20px}
.card{
 background:var(--card);padding:16px;border-radius:14px;
 box-shadow:0 10px 30px rgba(0,0,0,.07);margin-bottom:20px
}

.controls{display:flex;gap:10px;flex-wrap:wrap}
.calendar{margin-top:10px}
.weekdays,.days{display:grid;grid-template-columns:repeat(7,1fr)}
.weekdays div{text-align:center;font-weight:600;padding:8px}
.days{gap:10px}

.day{
 height:90px;border-radius:12px;padding:8px;
 cursor:pointer;transition:.2s;display:flex;flex-direction:column
}
.day:hover{transform:scale(1.04)}
.work{border:2px solid var(--work);background:rgba(34,197,94,.15)}
.off{border:2px solid var(--off);background:rgba(239,68,68,.15)}
.holiday{border:2px solid var(--holiday);background:rgba(59,130,246,.15)}

.panel{display:none}
.panel.active{display:block}

.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:15px}
.stat{padding:14px;border-radius:12px;background:rgba(0,0,0,.05)}

footer{text-align:center;padding:20px;font-size:13px;opacity:.6}
</style>
</head>

<body>
<header>
 <h1>WorkDay ULTIMATE</h1>
 <div class="actions">
   <button onclick="toggleTheme()">🌙</button>
   <button onclick="toggleLang()">🌍</button>
 </div>
</header>

<main>

<div class="card controls">
 <select id="month"></select>
 <select id="year"></select>
 <button onclick="render()">🔄</button>
</div>

<div class="card calendar">
 <div class="weekdays" id="week"></div>
 <div class="days" id="days"></div>
</div>

<div class="card panel" id="info"></div>

<div class="card">
 <h3 id="statsTitle">Statistika mujore</h3>
 <div class="stats" id="stats"></div>
</div>

</main>

<footer>© 2026 WorkDay – Public, Secure, Professional</footer>

<script>
/* ===== DATA ===== */
const data={
 sq:{
  days:["Hën","Mar","Mër","Enj","Pre","Sht","Die"],
  months:["Janar","Shkurt","Mars","Prill","Maj","Qershor","Korrik","Gusht","Shtator","Tetor","Nëntor","Dhjetor"],
  work:"Ditë pune",off:"Pushim",holiday:"Festë",stats:"Statistika mujore"
 },
 en:{
  days:["Mon","Tue","Wed","Thu","Fri","Sat","Sun"],
  months:["January","February","March","April","May","June","July","August","September","October","November","December"],
  work:"Workday",off:"Off day",holiday:"Holiday",stats:"Monthly statistics"
 }
};

let lang=localStorage.lang||"sq";
const holidays=["1-1","17-2","28-11","25-12"];

/* ===== ELEMENTS ===== */
const monthSel=month=document.getElementById("month");
const yearSel=document.getElementById("year");
const daysEl=document.getElementById("days");
const weekEl=document.getElementById("week");
const info=document.getElementById("info");
const statsEl=document.getElementById("stats");
const statsTitle=document.getElementById("statsTitle");

/* ===== INIT ===== */
(function(){
 if(localStorage.theme==="dark")document.body.classList.add("dark");
 data[lang].months.forEach((m,i)=>monthSel.add(new Option(m,i)));
 const y=new Date().getFullYear();
 for(let i=y-3;i<=y+3;i++)yearSel.add(new Option(i,i));
 monthSel.value=new Date().getMonth();yearSel.value=y;
 renderWeek();render();
})();

/* ===== FUNCTIONS ===== */
function toggleTheme(){
 document.body.classList.toggle("dark");
 localStorage.theme=document.body.classList.contains("dark")?"dark":"light";
}
function toggleLang(){
 lang=lang==="sq"?"en":"sq";
 localStorage.lang=lang;
 location.reload();
}
function renderWeek(){
 weekEl.innerHTML="";
 data[lang].days.forEach(d=>weekEl.innerHTML+=`<div>${d}</div>`);
 statsTitle.textContent=data[lang].stats;
}
function isHoliday(d,m){return holidays.includes(`${d}-${m+1}`)}

function render(){
 daysEl.innerHTML="";info.classList.remove("active");
 statsEl.innerHTML="";
 let work=0,off=0,hol=0;

 const m=+monthSel.value,y=+yearSel.value;
 const first=new Date(y,m,1).getDay()||7;
 const total=new Date(y,m+1,0).getDate();

 for(let i=1;i<first;i++)daysEl.appendChild(document.createElement("div"));

 for(let d=1;d<=total;d++){
  const date=new Date(y,m,d);
  let type="work";
  if(date.getDay()==6||date.getDay()==0)type="off";
  if(isHoliday(d,m))type="holiday";

  if(type==="work")work++; if(type==="off")off++; if(type==="holiday")hol++;

  const el=document.createElement("div");
  el.className=`day ${type}`;
  el.innerHTML=`<strong>${d}</strong>`;
  el.onclick=()=>showInfo(d,m,y,type);
  daysEl.appendChild(el);
 }

 statsEl.innerHTML=`
  <div class="stat">🟢 ${data[lang].work}: <b>${work}</b></div>
  <div class="stat">🔴 ${data[lang].off}: <b>${off}</b></div>
  <div class="stat">🔵 ${data[lang].holiday}: <b>${hol}</b></div>
 `;
}

function showInfo(d,m,y,type){
 info.className="card panel active";
 info.innerHTML=`
  <h3>${d} ${data[lang].months[m]} ${y}</h3>
  <p><b>${type==="work"?data[lang].work:type==="off"?data[lang].off:data[lang].holiday}</b></p>
 `;
}
</script>
</body>
</html>
