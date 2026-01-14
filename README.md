<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام تقييم مشاريع التخرج | النسخة القياسية</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f8fafc; }
        .score-input { border: 2px solid #e2e8f0; transition: all 0.2s; text-align: center; font-weight: 700; font-size: 1.1rem; }
        .score-input:focus { border-color: #4f46e5; outline: none; background-color: #fffbeb; }
        @media print { .no-print { display: none; } body { padding: 0; background: white; } }
    </style>
</head>
<body class="p-4 md:p-8">

    <div id="app" class="max-w-6xl mx-auto space-y-6">
        
        <!-- واجهة اختيار الدور -->
        <div id="roleSelection" class="bg-white p-10 rounded-[2.5rem] shadow-2xl text-center no-print border border-slate-200">
            <h2 class="text-3xl font-black mb-2 text-slate-800">نظام تقييم مشاريع التخرج</h2>
            <p class="text-slate-500 mb-10">إدارة التقييمات، توزيع الدرجات، واستخراج التقارير النهائية</p>
            
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <button onclick="requestAdminAccess()" class="group p-8 bg-slate-50 border-4 border-slate-200 rounded-[2.5rem] hover:bg-slate-900 hover:text-white transition-all duration-300">
                    <div class="text-4xl mb-4">🔐</div>
                    <div class="text-xl font-black">الإدارة والبيانات</div>
                </button>

                <button onclick="setRole('supervisor')" class="group p-8 bg-white border-4 border-indigo-600 rounded-[2.5rem] hover:bg-indigo-600 hover:text-white transition-all duration-300 shadow-xl">
                    <div class="text-4xl mb-4">📝</div>
                    <div class="text-xl font-black">تقييم المشرف</div>
                </button>
                
                <button onclick="setRole('examiner')" class="group p-8 bg-white border-4 border-emerald-600 rounded-[2.5rem] hover:bg-emerald-600 hover:text-white transition-all duration-300 shadow-xl">
                    <div class="text-4xl mb-4">🎓</div>
                    <div class="text-xl font-black">تقييم المناقش</div>
                </button>
            </div>
        </div>

        <!-- لوحة التحكم (المسؤول) -->
        <div id="adminPanel" class="hidden bg-white shadow-2xl rounded-[2.5rem] overflow-hidden border border-slate-200">
            <div class="bg-slate-900 p-6 text-white flex justify-between items-center">
                <h2 class="text-2xl font-bold">لوحة التحكم المركزية</h2>
                <button onclick="goBack()" class="bg-white/20 px-4 py-2 rounded-lg text-sm">رجوع</button>
            </div>
            <div class="p-8 space-y-8">
                <div class="bg-indigo-50 p-8 rounded-3xl border-2 border-dashed border-indigo-200 text-center">
                    <h3 class="font-bold text-indigo-800 mb-2 text-lg">📁 استيراد بيانات المشاريع</h3>
                    <p class="text-sm text-indigo-600 mb-4">ارفع ملف Excel لتوزيع الطلاب على المشاريع تلقائياً</p>
                    <input type="file" id="excelUpload" accept=".xlsx, .xls" class="hidden" onchange="importExcel(event)">
                    <button onclick="document.getElementById('excelUpload').click()" class="bg-indigo-600 text-white px-10 py-3 rounded-2xl font-bold shadow-lg hover:bg-indigo-700">رفع الملف</button>
                </div>
                <div id="adminDataList" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
            </div>
        </div>

        <!-- نموذج التقييم الرئيسي -->
        <div id="mainContainer" class="hidden bg-white shadow-2xl rounded-[2.5rem] overflow-hidden border border-slate-200">
            <div id="formHeader" class="p-10 text-white text-center relative">
                <button onclick="goBack()" class="absolute top-8 left-8 bg-white/20 px-4 py-2 rounded-full text-xs font-bold hover:bg-white/30 transition-all">الرئيسية</button>
                <h1 id="headerTitle" class="text-4xl font-black"></h1>
                <p id="headerSubtitle" class="mt-2 opacity-80 font-medium"></p>
            </div>

            <form id="evaluationForm" class="p-8 md:p-12 space-y-12">
                <!-- اختيار المشروع والبيانات العامة -->
                <div class="grid grid-cols-1 md:grid-cols-3 gap-8 p-8 bg-slate-50 rounded-3xl border border-slate-100">
                    <div class="space-y-2">
                        <label class="block font-black text-slate-700 text-sm">اسم المشروع المختار</label>
                        <select id="projectSelect" class="w-full p-3 bg-white border border-slate-200 rounded-xl outline-none font-bold text-indigo-600 shadow-sm" onchange="handleProjectChange()">
                            <option value="">-- اختر من القائمة --</option>
                        </select>
                    </div>
                    <div id="dynamicFields" class="contents"></div>
                </div>

                <!-- شبكة تقييم الطلاب -->
                <div class="grid grid-cols-1 lg:grid-cols-2 xl:grid-cols-3 gap-8" id="studentsWrapper">
                    <div class="col-span-full py-20 text-center opacity-40">
                        <div class="text-5xl mb-4">🔍</div>
                        <p class="font-bold">يرجى اختيار المشروع لعرض الطلاب وبدء عملية التقييم</p>
                    </div>
                </div>

                <!-- إجراءات الحفظ والتصدير -->
                <div class="pt-10 flex flex-wrap justify-center gap-4 border-t border-slate-100 no-print">
                    <button type="button" onclick="exportToExcel()" class="bg-slate-100 text-slate-700 px-8 py-4 rounded-2xl font-black hover:bg-slate-200 transition-all">تصدير Excel</button>
                    <button type="button" onclick="shareWhatsApp()" class="bg-green-500 text-white px-8 py-4 rounded-2xl font-black hover:bg-green-600 transition-all shadow-lg">إرسال عبر WhatsApp</button>
                    <button type="button" onclick="window.print()" class="bg-indigo-600 text-white px-10 py-4 rounded-2xl font-black hover:bg-indigo-700 transition-all shadow-xl">طباعة التقرير / PDF</button>
                </div>
            </form>
        </div>
    </div>

    <!-- قالب بطاقة الطالب -->
    <template id="studentTemplate">
        <div class="student-card bg-white border border-slate-200 rounded-[2.5rem] p-8 shadow-sm hover:shadow-md transition-all flex flex-col h-full">
            <div class="flex justify-between items-start mb-6">
                <div>
                    <span class="text-[10px] font-black text-slate-400 uppercase tracking-widest">طالب مشروع تخرج</span>
                    <h4 class="student-name-display text-2xl font-black text-slate-800"></h4>
                </div>
                <div class="text-3xl">👤</div>
            </div>
            <div class="criteria-list space-y-5 flex-grow"></div>
            <div class="mt-10 pt-6 border-t border-slate-100 flex justify-between items-end">
                <div>
                    <span class="text-[10px] font-black text-slate-400 block mb-1">الدرجة النهائية</span>
                    <div class="flex items-baseline gap-1">
                        <span class="text-4xl font-black text-indigo-600 student-total-display">0</span>
                        <span class="text-sm font-bold text-slate-400">/ 100</span>
                    </div>
                </div>
                <div class="student-result-text font-black text-xs px-5 py-2 rounded-full bg-slate-100 text-slate-500 uppercase tracking-wide">قيد التقييم</div>
            </div>
        </div>
    </template>

    <script>
        // قاعدة البيانات المحلية
        let db = JSON.parse(localStorage.getItem('grad_db_linked')) || [
            { title: "نظام المستودعات الذكي", supervisor: "د. أحمد خليل", students: ["سامي ناصر", "رنا يوسف"] }
        ];
        let currentRole = '';

        const roles = {
            supervisor: { title: "بوابة المشرف", subtitle: "تقييم المرحلة التحضيرية", color: "from-indigo-600 to-indigo-800", criteria: [{id:'book',label:'توثيق البحث',max:25},{id:'practical',label:'التنفيذ العملي',max:35},{id:'meetings',label:'الحضور والمتابعة',max:20},{id:'ethics',label:'أخلاقيات العمل',max:20}] },
            examiner: { title: "لجنة المناقشة", subtitle: "التقييم النهائي للمشروع", color: "from-emerald-600 to-emerald-800", criteria: [{id:'report',label:'جودة التقرير',max:25},{id:'logic',label:'المنطق البرمجي',max:25},{id:'defense',label:'قوة المناقشة',max:25},{id:'presentation',label:'العرض المرئي',max:25}] }
        };

        // وظائف اختيار الدور
        function setRole(role) {
            currentRole = role;
            const cfg = roles[role];
            document.getElementById('roleSelection').classList.add('hidden');
            document.getElementById('mainContainer').classList.remove('hidden');
            document.getElementById('formHeader').className = `p-10 text-white text-center relative bg-gradient-to-r ${cfg.color}`;
            document.getElementById('headerTitle').innerText = cfg.title;
            document.getElementById('headerSubtitle').innerText = cfg.subtitle;
            
            const sel = document.getElementById('projectSelect');
            sel.innerHTML = '<option value="">-- اختر المشروع --</option>' + db.map(p => `<option value="${p.title}">${p.title}</option>`).join('');
            
            const dyn = document.getElementById('dynamicFields');
            dyn.innerHTML = `
                <div class="space-y-2">
                    <label class="block font-black text-slate-700 text-sm">المشرف المسؤول</label>
                    <input type="text" id="supName" class="w-full p-3 bg-white border border-slate-200 rounded-xl outline-none font-bold" readonly>
                </div>
                <div class="space-y-2">
                    <label class="block font-black text-slate-700 text-sm">التاريخ</label>
                    <input type="date" id="evalDate" class="w-full p-3 bg-white border border-slate-200 rounded-xl outline-none font-bold" value="${new Date().toISOString().split('T')[0]}">
                </div>`;
        }

        // التعامل مع تغيير المشروع
        function handleProjectChange() {
            const title = document.getElementById('projectSelect').value;
            const project = db.find(p => p.title === title);
            const wrap = document.getElementById('studentsWrapper');
            if(!project) { wrap.innerHTML = '<div class="col-span-full py-20 text-center opacity-40"><p class="font-bold">يرجى اختيار المشروع</p></div>'; return; }
            
            document.getElementById('supName').value = project.supervisor;
            wrap.innerHTML = '';
            const criteria = roles[currentRole].criteria;

            project.students.forEach(name => {
                const temp = document.getElementById('studentTemplate').content.cloneNode(true);
                const card = temp.querySelector('.student-card');
                card.querySelector('.student-name-display').innerText = name;
                
                criteria.forEach(crit => {
                    const row = document.createElement('div');
                    row.innerHTML = `
                        <div class="flex justify-between text-[10px] font-black text-slate-400 mb-2 uppercase tracking-tighter">
                            <span>${crit.label}</span>
                            <span>الحد الأقصى: ${crit.max}</span>
                        </div>
                        <input type="number" min="0" max="${crit.max}" value="0" class="score-input w-full p-2 rounded-2xl border" oninput="updateCardScore(this, ${crit.max})">`;
                    card.querySelector('.criteria-list').appendChild(row);
                });
                wrap.appendChild(temp);
            });
        }

        // تحديث درجات الطالب
        function updateCardScore(input, max) {
            let val = parseInt(input.value) || 0;
            if(val > max) input.value = max;
            if(val < 0) input.value = 0;
            
            const card = input.closest('.student-card');
            let total = 0;
            card.querySelectorAll('.score-input').forEach(i => total += (parseInt(i.value) || 0));
            card.querySelector('.student-total-display').innerText = total;
            
            const badge = card.querySelector('.student-result-text');
            if(total >= 90) { badge.innerText = "ممتاز"; badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-indigo-100 text-indigo-700"; }
            else if(total >= 60) { badge.innerText = "ناجح"; badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-emerald-100 text-emerald-700"; }
            else { badge.innerText = "راسب"; badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-rose-100 text-rose-700"; }
        }

        // الإدارة والبيانات
        function requestAdminAccess() {
            if(prompt("كلمة المرور:") === "1234") showSection('admin');
            else alert("عذراً، كلمة المرور خاطئة");
        }

        function showSection(id) {
            document.getElementById('roleSelection').classList.add('hidden');
            document.getElementById('adminPanel').classList.toggle('hidden', id !== 'admin');
            if(id === 'admin') renderAdminData();
        }

        function goBack() {
            document.getElementById('adminPanel').classList.add('hidden');
            document.getElementById('mainContainer').classList.add('hidden');
            document.getElementById('roleSelection').classList.remove('hidden');
        }

        function renderAdminData() {
            const list = document.getElementById('adminDataList');
            list.innerHTML = db.map(p => `
                <div class="bg-slate-50 border p-5 rounded-3xl shadow-sm">
                    <h5 class="font-black text-indigo-700 mb-1">${p.title}</h5>
                    <p class="text-xs text-slate-500 mb-3">إشراف: ${p.supervisor}</p>
                    <div class="flex flex-wrap gap-2">
                        ${p.students.map(s => `<span class="bg-white border text-[10px] font-bold px-3 py-1 rounded-full text-slate-600">${s}</span>`).join('')}
                    </div>
                </div>
            `).join('');
        }

        function importExcel(e) {
            const file = e.target.files[0];
            const reader = new FileReader();
            reader.onload = (event) => {
                const workbook = XLSX.read(new Uint8Array(event.target.result), { type: 'array' });
                const json = XLSX.utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]]);
                db = [];
                json.forEach(r => {
                    const p = r['اسم المشروع'], s = r['اسم الطالب'], sup = r['اسم المشرف'];
                    if(!p || !s) return;
                    let project = db.find(item => item.title === p);
                    if(project) { if(!project.students.includes(s)) project.students.push(s); }
                    else db.push({ title: p, supervisor: sup || "غير معروف", students: [s] });
                });
                localStorage.setItem('grad_db_linked', JSON.stringify(db));
                renderAdminData();
                alert("تم استيراد البيانات بنجاح!");
            };
            reader.readAsArrayBuffer(file);
        }

        function exportToExcel() {
            const project = document.getElementById('projectSelect').value;
            const data = [["تقرير نتائج التقييم"], ["المشروع", project], ["المشرف", document.getElementById('supName').value], [], ["اسم الطالب", "الدرجة النهائية", "التقدير"]];
            document.querySelectorAll('.student-card').forEach(c => {
                data.push([c.querySelector('.student-name-display').innerText, c.querySelector('.student-total-display').innerText, c.querySelector('.student-result-text').innerText]);
            });
            const ws = XLSX.utils.aoa_to_sheet(data);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, "النتائج");
            XLSX.writeFile(wb, `Graduation_Report_${project}.xlsx`);
        }

        function shareWhatsApp() {
            const project = document.getElementById('projectSelect').value;
            let msg = `*تقرير تقييم مشروع تخرج*%0A*المشروع:* ${project}%0A%0A`;
            document.querySelectorAll('.student-card').forEach(c => {
                msg += `• ${c.querySelector('.student-name-display').innerText}: ${c.querySelector('.student-total-display').innerText}/100 (${c.querySelector('.student-result-text').innerText})%0A`;
            });
            window.open(`https://wa.me/?text=${msg}`, '_blank');
        }
    </script>
</body>
</html>
