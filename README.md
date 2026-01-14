<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>نظام تقييم مشاريع التخرج</title>

<script src="https://cdn.tailwindcss.com"></script>
<link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

<style>
body { font-family: 'Tajawal', sans-serif; background:#f1f5f9 }
.score-input{border:2px solid #e2e8f0;text-align:center;font-weight:700}
.score-input:focus{border-color:#4f46e5;background:#fffbeb;outline:none}
.admin-card{background:#fff;border:1px solid #e5e7eb;padding:1.5rem;border-radius:1.5rem}
</style>
</head>

<body class="p-6">

<div id="app" class="max-w-6xl mx-auto space-y-6">

<!-- ================= ROLE SELECTION ================= -->
<div id="roleSelection" class="bg-white p-10 rounded-3xl shadow-xl text-center">
<h2 class="text-3xl font-black mb-2">بوابة التقييم الرقمية</h2>
<p class="text-gray-500 mb-10">اختر نوع الدخول</p>

<div class="grid md:grid-cols-3 gap-6">
<button onclick="requestAdminAccess()" class="p-8 bg-gray-100 rounded-2xl hover:bg-gray-800 hover:text-white">🔐<br>لوحة المسؤول</button>
<button onclick="setRole('supervisor')" class="p-8 border-4 border-indigo-600 rounded-2xl">📋<br>المشرف</button>
<button onclick="setRole('examiner')" class="p-8 border-4 border-emerald-600 rounded-2xl">🎓<br>المناقش</button>
</div>
</div>

<!-- ================= ADMIN PANEL ================= -->
<div id="adminPanel" class="hidden bg-white rounded-3xl shadow-xl overflow-hidden">

<div class="bg-gray-800 text-white p-6 flex justify-between">
<h2 class="text-xl font-bold">لوحة المسؤول</h2>
<button onclick="goBack()">خروج</button>
</div>

<div class="p-6 space-y-8">

<div class="bg-indigo-50 p-6 rounded-xl text-center">
<input type="file" id="excelUpload" accept=".xlsx,.xls" hidden onchange="importExcel(event)">
<button onclick="excelUpload.click()" class="bg-indigo-600 text-white px-6 py-2 rounded-xl">استيراد Excel</button>
</div>

<div class="grid md:grid-cols-2 gap-6">

<div class="admin-card">
<h3 class="font-bold text-indigo-600 mb-2">المشاريع</h3>
<input id="newProject" class="w-full p-2 border rounded mb-2" placeholder="اسم المشروع">
<input id="newSupervisor" class="w-full p-2 border rounded mb-2" placeholder="اسم المشرف">
<button onclick="addProject()" class="w-full bg-indigo-600 text-white rounded py-2">إضافة</button>
<ul id="adminProjectsList" class="mt-4 space-y-2"></ul>
</div>

<div class="admin-card">
<h3 class="font-bold text-emerald-600 mb-2">الطلاب</h3>
<input id="newStudent" class="w-full p-2 border rounded mb-2" placeholder="اسم الطالب">
<button onclick="addStudent()" class="w-full bg-emerald-600 text-white rounded py-2">إضافة</button>
<ul id="adminStudentsList" class="mt-4 space-y-2"></ul>
</div>

</div>
</div>
</div>

<!-- ================= MAIN FORM ================= -->
<div id="mainContainer" class="hidden bg-white rounded-3xl shadow-xl">

<div id="formHeader" class="p-8 text-white text-center">
<button onclick="goBack()" class="absolute left-6 top-6">الرئيسية</button>
<h1 id="headerTitle" class="text-3xl font-black"></h1>
</div>

<form class="p-8 space-y-8">

<div class="grid md:grid-cols-3 gap-6">
<div>
<label class="font-bold">المشروع</label>
<select id="projectSelect" onchange="loadProjectData()" class="w-full p-2 border rounded"></select>
<input id="projectTitle" class="w-full p-2 border rounded mt-2 hidden">
</div>

<div id="dynamicFields" class="contents"></div>
</div>

<div id="syncSection" class="hidden bg-amber-50 p-4 rounded-xl flex justify-between">
<span>دمج علامات الكتاب والعملي</span>
<button type="button" onclick="toggleSync()" id="syncBtn" class="bg-amber-500 text-white px-4 py-1 rounded">تفعيل</button>
</div>

<div id="studentsWrapper" class="grid lg:grid-cols-3 gap-6"></div>

<div class="flex justify-center gap-4 pt-6">
<button type="button" onclick="exportExcel()" class="bg-emerald-600 text-white px-6 py-2 rounded-xl">Excel</button>
<button type="button" onclick="shareWhatsApp()" class="bg-green-600 text-white px-6 py-2 rounded-xl">WhatsApp</button>
<button type="button" onclick="window.print()" class="bg-gray-800 text-white px-6 py-2 rounded-xl">PDF</button>
</div>

</form>
</div>
</div>

<!-- ================= TEMPLATE ================= -->
<template id="studentTemplate">
<div class="bg-gray-50 p-6 rounded-2xl border">
<select class="student-select w-full p-2 border rounded mb-2"></select>
<div class="criteria space-y-3"></div>
<div class="flex justify-between mt-4">
<span class="total font-black text-2xl">0</span>
<span class="result text-sm"></span>
</div>
</div>
</template>

<script>
// ================= STATE =================
let currentRole="", sync=false;
const ADMIN_PASS="1234";

// ================= DATA =================
let db = JSON.parse(localStorage.getItem("grad_db")) || {
projects:[{title:"نظام إدارة",supervisor:"د. محمد"}],
students:["أحمد","سارة"]
};

const config={
supervisor:{
title:"نموذج المشرف",
color:"bg-indigo-700",
criteria:[
{id:"book",label:"الكتاب",max:25,shared:true},
{id:"practical",label:"العملي",max:35,shared:true},
{id:"team",label:"التعاون",max:40}
],
fields:["اسم المشرف"]
},
examiner:{
title:"نموذج المناقش",
color:"bg-emerald-700",
criteria:[
{id:"report",label:"التقرير",max:30},
{id:"demo",label:"العرض",max:30},
{id:"skills",label:"المهارات",max:40}
],
fields:["اسم المشرف","اسم المناقش"]
}
};

// ================= NAV =================
function requestAdminAccess(){
const p=prompt("كلمة المرور:");
if(p===ADMIN_PASS){roleSelection.classList.add("hidden");adminPanel.classList.remove("hidden");renderAdmin();}
else alert("خطأ");
}

function goBack(){
adminPanel.classList.add("hidden");
mainContainer.classList.add("hidden");
roleSelection.classList.remove("hidden");
}

function setRole(r){
currentRole=r;
roleSelection.classList.add("hidden");
mainContainer.classList.remove("hidden");

formHeader.className=`p-8 text-white ${config[r].color}`;
headerTitle.innerText=config[r].title;

syncSection.classList.toggle("hidden",r!=="supervisor");

projectSelect.innerHTML=`<option value="">اختر</option>`+
db.projects.map(p=>`<option data-sup="${p.supervisor}" value="${p.title}">${p.title}</option>`).join("")+
`<option value="custom">يدوي</option>`;

dynamicFields.innerHTML=config[r].fields.map(f=>`
<div><label class="font-bold">${f}</label><input class="w-full p-2 border rounded"></div>`).join("");

renderStudents();
}

// ================= STUDENTS =================
function renderStudents(){
studentsWrapper.innerHTML="";
for(let i=0;i<3;i++){
const t=studentTemplate.content.cloneNode(true);
const sel=t.querySelector(".student-select");
sel.innerHTML=`<option value="">طالب</option>`+
db.students.map(s=>`<option>${s}</option>`).join("");

config[currentRole].criteria.forEach(c=>{
const d=document.createElement("div");
d.innerHTML=`<label>${c.label} (${c.max})</label>
<input type="number" min="0" max="${c.max}" value="0" class="score-input w-full" data-id="${c.id}" data-max="${c.max}" data-shared="${c.shared||false}">`;
d.querySelector("input").oninput=e=>handleScore(e,t);
t.querySelector(".criteria").appendChild(d);
});
studentsWrapper.appendChild(t);
}
}

function handleScore(e,card){
let max=+e.target.dataset.max;
e.target.value=Math.max(0,Math.min(+e.target.value||0,max));
if(sync && e.target.dataset.shared==="true")syncShared(e.target.dataset.id,e.target.value);
updateTotal(card);
}

function updateTotal(card){
let total=0;
card.querySelectorAll(".score-input").forEach(i=>total+=+i.value);
card.querySelector(".total").innerText=total;
card.querySelector(".result").innerText=total>=50?"ناجح":"راسب";
}

// ================= ADMIN =================
function renderAdmin(){
adminProjectsList.innerHTML=db.projects.map((p,i)=>`
<li>${p.title} (${p.supervisor}) <button onclick="delProject(${i})">🗑</button></li>`).join("");
adminStudentsList.innerHTML=db.students.map((s,i)=>`
<li>${s} <button onclick="delStudent(${i})">🗑</button></li>`).join("");
}

function addProject(){
if(newProject.value){
db.projects.push({title:newProject.value,supervisor:newSupervisor.value});
saveDB();renderAdmin();newProject.value=newSupervisor.value="";
}
}
function addStudent(){if(newStudent.value){db.students.push(newStudent.value);saveDB();renderAdmin();newStudent.value="";}}
function delProject(i){db.projects.splice(i,1);saveDB();renderAdmin();}
function delStudent(i){db.students.splice(i,1);saveDB();renderAdmin();}
function saveDB(){localStorage.setItem("grad_db",JSON.stringify(db));}

// ================= EXTRAS =================
function toggleSync(){sync=!sync;syncBtn.innerText=sync?"إيقاف":"تفعيل";}
function syncShared(id,val){
document.querySelectorAll(`[data-id="${id}"]`).forEach(i=>i.value=val);
}
function loadProjectData(){
if(projectSelect.value==="custom")projectTitle.classList.remove("hidden");
else{projectTitle.classList.add("hidden");projectTitle.value=projectSelect.value;}
}

// ================= EXPORT =================
function exportExcel(){
const data=[["الاسم","المجموع"]];
document.querySelectorAll(".student-select").forEach((s,i)=>{
if(s.value){
const t=studentsWrapper.children[i].querySelector(".total").innerText;
data.push([s.value,t]);
}
});
const ws=XLSX.utils.aoa_to_sheet(data);
const wb=XLSX.utils.book_new();
XLSX.utils.book_append_sheet(wb,ws,"Evaluation");
XLSX.writeFile(wb,"evaluation.xlsx");
}

function shareWhatsApp(){
let msg="تقرير التقييم:%0A";
document.querySelectorAll(".student-select").forEach((s,i)=>{
if(s.value){
const t=studentsWrapper.children[i].querySelector(".total").innerText;
msg+=`${s.value}: ${t}%0A`;
}
});
window.open(`https://wa.me/?text=${msg}`);
}
</script>

</body>
</html>
