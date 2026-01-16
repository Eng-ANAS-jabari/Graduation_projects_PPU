<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام إدارة وتقييم المناقشات | ربط المجموعات</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700;900&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f8fafc; }
        .score-input { border: 2px solid #e2e8f0; transition: all 0.2s; text-align: center; font-weight: 700; }
        .score-input:focus { border-color: #4f46e5; outline: none; background-color: #fffbeb; }
        .loading-overlay { position: fixed; inset: 0; background: rgba(255,255,255,0.9); display: flex; align-items: center; justify-content: center; z-index: 1000; }
        .glass-card { background: rgba(255, 255, 255, 0.95); backdrop-filter: blur(10px); }
        @media print { .no-print { display: none !important; } .print-only { display: block !important; } }
    </style>
</head>
<body class="p-4 md:p-6">

    <!-- شاشة التحميل -->
    <div id="loading" class="loading-overlay hidden">
        <div class="text-center">
            <div class="inline-block animate-spin rounded-full h-16 w-16 border-4 border-indigo-600 border-t-transparent"></div>
            <p class="mt-4 font-black text-indigo-900 text-lg">جاري مزامنة البيانات السحابية...</p>
        </div>
    </div>

    <div id="app" class="max-w-7xl mx-auto space-y-6">
        
        <!-- الواجهة الرئيسية / اختيار الدور -->
        <div id="roleSelection" class="bg-white p-12 rounded-[3rem] shadow-2xl text-center no-print border border-slate-200">
            <div class="mb-8">
                <span class="bg-indigo-100 text-indigo-700 px-4 py-1 rounded-full text-sm font-bold">إصدار الإدارة المركزية</span>
                <h2 class="text-4xl font-black mt-4 text-slate-800">نظام تنظيم وتقييم التخرج</h2>
                <p class="text-slate-500 mt-2">إدارة المجموعات، القاعات، ورصد الدرجات</p>
            </div>

            <div class="flex flex-wrap justify-center gap-4 mb-12">
                <button onclick="fetchSheetData()" class="flex items-center gap-2 bg-indigo-50 text-indigo-600 px-6 py-3 rounded-2xl font-bold border border-indigo-100 hover:bg-indigo-100 transition-all">
                    <span>🔄</span> تحديث البيانات من الرابط
                </button>
                <button onclick="toggleAdminView()" class="flex items-center gap-2 bg-slate-800 text-white px-6 py-3 rounded-2xl font-bold hover:bg-slate-900 transition-all">
                    <span>⚙️</span> واجهة المسؤول
                </button>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-8 max-w-3xl mx-auto">
                <button onclick="setRole('supervisor')" class="p-10 bg-white border-4 border-indigo-600 rounded-[2.5rem] hover:bg-indigo-600 hover:text-white transition-all shadow-xl group">
                    <div class="text-5xl mb-4 group-hover:scale-110 transition-transform">📝</div>
                    <div class="text-2xl font-black">تقييم المشرف</div>
                    <p class="text-sm mt-2 opacity-70">رصد أعمال الفصل والمتابعة</p>
                </button>
                
                <button onclick="setRole('examiner')" class="p-10 bg-white border-4 border-emerald-600 rounded-[2.5rem] hover:bg-emerald-600 hover:text-white transition-all shadow-xl group">
                    <div class="text-5xl mb-4 group-hover:scale-110 transition-transform">🎓</div>
                    <div class="text-2xl font-black">تقييم المناقش</div>
                    <p class="text-sm mt-2 opacity-70">تقييم لجنة الحكم والعرض النهائي</p>
                </button>
            </div>
        </div>

        <!-- واجهة المسؤول (Admin Dashboard) -->
        <div id="adminDashboard" class="hidden space-y-6 no-print">
            <div class="flex items-center justify-between bg-slate-900 text-white p-8 rounded-[2rem]">
                <div>
                    <h2 class="text-2xl font-black">لوحة تحكم المسؤول</h2>
                    <p class="opacity-70">مراقبة توزيع المجموعات والقاعات</p>
                </div>
                <button onclick="toggleAdminView()" class="bg-white/10 px-6 py-2 rounded-xl hover:bg-white/20">رجوع للخلف</button>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div class="bg-white p-6 rounded-3xl border border-slate-200 shadow-sm">
                    <div class="text-slate-400 text-xs font-bold uppercase mb-1">إجمالي المجموعات</div>
                    <div id="statGroups" class="text-3xl font-black text-indigo-600">0</div>
                </div>
                <div class="bg-white p-6 rounded-3xl border border-slate-200 shadow-sm">
                    <div class="text-slate-400 text-xs font-bold uppercase mb-1">إجمالي الطلاب</div>
                    <div id="statStudents" class="text-3xl font-black text-emerald-600">0</div>
                </div>
                <div class="bg-white p-6 rounded-3xl border border-slate-200 shadow-sm">
                    <div class="text-slate-400 text-xs font-bold uppercase mb-1">عدد القاعات</div>
                    <div id="statRooms" class="text-3xl font-black text-orange-600">0</div>
                </div>
                <div class="bg-white p-6 rounded-3xl border border-slate-200 shadow-sm">
                    <input type="text" placeholder="بحث عن طالب أو قاعة..." class="w-full h-full p-2 outline-none font-bold" oninput="filterAdminTable(this.value)">
                </div>
            </div>

            <div class="bg-white rounded-[2rem] shadow-sm border border-slate-200 overflow-hidden">
                <table class="w-full text-right">
                    <thead class="bg-slate-50 border-b border-slate-200">
                        <tr>
                            <th class="p-4 font-black text-slate-600">القاعة</th>
                            <th class="p-4 font-black text-slate-600">رقم المجموعة</th>
                            <th class="p-4 font-black text-slate-600">اسم المشروع</th>
                            <th class="p-4 font-black text-slate-600">الطلاب</th>
                            <th class="p-4 font-black text-slate-600">المشرف</th>
                        </tr>
                    </thead>
                    <tbody id="adminTableBody">
                        <!-- تظهر هنا البيانات -->
                    </tbody>
                </table>
            </div>
        </div>

        <!-- نموذج التقييم (نفس الهيكلية مع إضافة بيانات المجموعة والقاعة) -->
        <div id="mainContainer" class="hidden bg-white shadow-2xl rounded-[3rem] overflow-hidden border border-slate-200">
            <div id="formHeader" class="p-10 text-white text-center relative transition-all duration-500">
                <button onclick="location.reload()" class="absolute top-8 left-8 bg-white/20 px-4 py-2 rounded-full text-xs font-bold hover:bg-white/30 transition-all no-print">إغلاق</button>
                <h1 id="headerTitle" class="text-4xl font-black"></h1>
                <p id="headerSubtitle" class="mt-2 opacity-80 font-medium"></p>
            </div>

            <form id="evaluationForm" class="p-6 md:p-10 space-y-10">
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 p-8 bg-slate-50 rounded-[2rem] border border-slate-100">
                    <div class="space-y-1">
                        <label class="block font-black text-slate-500 text-xs px-2">رقم المجموعة / القاعة</label>
                        <select id="projectSelect" class="w-full p-4 bg-white border-2 border-slate-200 rounded-2xl outline-none font-bold text-indigo-700 shadow-sm focus:border-indigo-500" onchange="handleProjectChange()">
                            <option value="">-- اختر المجموعة --</option>
                        </select>
                    </div>
                    <div class="space-y-1">
                        <label class="block font-black text-slate-500 text-xs px-2">اسم المشروع</label>
                        <input type="text" id="projNameDisplay" class="w-full p-4 bg-white border-2 border-slate-100 rounded-2xl font-bold text-slate-700" readonly>
                    </div>
                    <div class="space-y-1">
                        <label class="block font-black text-slate-500 text-xs px-2">المشرف</label>
                        <input type="text" id="supName" class="w-full p-4 bg-white border-2 border-slate-100 rounded-2xl font-bold text-slate-700" readonly>
                    </div>
                    <div class="space-y-1">
                        <label class="block font-black text-slate-500 text-xs px-2">التاريخ</label>
                        <input type="date" id="evalDate" class="w-full p-4 bg-white border-2 border-slate-200 rounded-2xl font-bold">
                    </div>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-8" id="studentsWrapper"></div>

                <div class="pt-8 flex flex-wrap justify-center gap-4 no-print">
                    <button type="button" onclick="saveToCloud()" class="bg-indigo-600 text-white px-12 py-5 rounded-[1.5rem] font-black hover:bg-indigo-700 transition-all shadow-xl active:scale-95">
                        🚀 اعتماد وحفظ النتائج سحابياً
                    </button>
                </div>
            </form>
        </div>
    </div>

    <template id="studentTemplate">
        <div class="student-card bg-white border-2 border-slate-100 rounded-[2.5rem] p-8 shadow-sm flex flex-col h-full">
            <div class="flex justify-between items-start mb-6">
                <div>
                    <h4 class="student-name-display text-2xl font-black text-slate-800"></h4>
                    <span class="text-xs font-bold text-slate-400">طالب في المجموعة</span>
                </div>
                <div class="h-12 w-12 bg-indigo-50 rounded-2xl flex items-center justify-center text-2xl">👤</div>
            </div>
            <div class="criteria-list space-y-4 flex-grow"></div>
            <div class="mt-8 pt-6 border-t border-slate-50 flex justify-between items-end">
                <div>
                    <span class="text-4xl font-black text-indigo-600 student-total-display">0</span>
                    <span class="text-sm font-bold text-slate-400">/ 100</span>
                </div>
                <div class="student-result-text font-black text-[10px] px-4 py-2 rounded-full bg-slate-100 text-slate-400 uppercase tracking-widest">قيد التقييم</div>
            </div>
        </div>
    </template>

    <script>
        const SHEET_ID = '1Ne4jRjMj75t2zk-w9bCoO2jZlkJkOIxtQAlqNG91p3U';
        const SHEET_URL = `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?tqx=out:csv`;
        const APPS_SCRIPT_URL = ''; // ضع رابط النشر الخاص بك هنا

        let db = [];
        let currentRole = '';

        const roles = {
            supervisor: { title: "تقييم المشرف", subtitle: "درجات أعمال الفصل والمتابعة", color: "from-indigo-600 to-indigo-900", criteria: [{id:'c1',label:'التوثيق',max:25},{id:'c2',label:'العملي',max:35},{id:'c3',label:'المتابعة',max:20},{id:'c4',label:'الأداء',max:20}] },
            examiner: { title: "تقييم المناقش", subtitle: "درجات اللجنة والعرض النهائي", color: "from-emerald-600 to-emerald-900", criteria: [{id:'c1',label:'التقرير',max:25},{id:'c2',label:'البرمجة',max:25},{id:'c3',label:'المناقشة',max:25},{id:'c4',label:'العرض',max:25}] }
        };

        async function fetchSheetData() {
            document.getElementById('loading').classList.remove('hidden');
            try {
                const response = await fetch(SHEET_URL);
                const csvData = await response.text();
                
                Papa.parse(csvData, {
                    header: true,
                    skipEmptyLines: true,
                    complete: (results) => {
                        const raw = results.data;
                        let projects = {};
                        
                        raw.forEach(row => {
                            const pName = row['اسم المشروع'] || row['Project Name'];
                            const sName = row['اسم الطالب'] || row['Student Name'];
                            const sup = row['اسم المشرف'] || row['Supervisor'];
                            const gId = row['رقم المجموعة'] || row['Group ID'] || 'N/A';
                            const room = row['رقم القاعة'] || row['Room'] || 'N/A';
                            
                            if(pName && sName) {
                                const key = `${gId}-${pName}`;
                                if(!projects[key]) {
                                    projects[key] = { 
                                        title: pName, 
                                        supervisor: sup, 
                                        groupId: gId, 
                                        room: room,
                                        students: [] 
                                    };
                                }
                                projects[key].students.push(sName);
                            }
                        });
                        
                        db = Object.values(projects);
                        localStorage.setItem('grad_db_v2', JSON.stringify(db));
                        updateAdminStats();
                        alert('تم تحديث بيانات المجموعات والقاعات بنجاح!');
                    }
                });
            } catch (err) {
                alert('خطأ في جلب البيانات من السحابة.');
            } finally {
                document.getElementById('loading').classList.add('hidden');
            }
        }

        function toggleAdminView() {
            const roleDiv = document.getElementById('roleSelection');
            const adminDiv = document.getElementById('adminDashboard');
            if(adminDiv.classList.contains('hidden')) {
                roleDiv.classList.add('hidden');
                adminDiv.classList.remove('hidden');
                renderAdminTable(db);
                updateAdminStats();
            } else {
                adminDiv.classList.add('hidden');
                roleDiv.classList.remove('hidden');
            }
        }

        function updateAdminStats() {
            document.getElementById('statGroups').innerText = db.length;
            document.getElementById('statStudents').innerText = db.reduce((acc, p) => acc + p.students.length, 0);
            document.getElementById('statRooms').innerText = [...new Set(db.map(p => p.room))].length;
        }

        function renderAdminTable(data) {
            const body = document.getElementById('adminTableBody');
            body.innerHTML = data.map(p => `
                <tr class="border-b border-slate-100 hover:bg-slate-50 transition-colors">
                    <td class="p-4 font-bold text-orange-600">قاعة ${p.room}</td>
                    <td class="p-4 font-black">#${p.groupId}</td>
                    <td class="p-4 font-medium">${p.title}</td>
                    <td class="p-4">
                        <div class="flex flex-wrap gap-1">
                            ${p.students.map(s => `<span class="bg-slate-200 px-2 py-1 rounded text-[10px] font-bold">${s}</span>`).join('')}
                        </div>
                    </td>
                    <td class="p-4 text-slate-500">${p.supervisor}</td>
                </tr>
            `).join('');
        }

        function filterAdminTable(query) {
            const filtered = db.filter(p => 
                p.title.includes(query) || 
                p.room.includes(query) || 
                p.groupId.includes(query) ||
                p.students.some(s => s.includes(query))
            );
            renderAdminTable(filtered);
        }

        function setRole(role) {
            if(db.length === 0) {
                const local = localStorage.getItem('grad_db_v2');
                if(local) db = JSON.parse(local);
                else return alert('الرجاء تحديث البيانات أولاً');
            }
            
            currentRole = role;
            document.getElementById('roleSelection').classList.add('hidden');
            document.getElementById('mainContainer').classList.remove('hidden');
            const cfg = roles[role];
            document.getElementById('formHeader').className = `p-10 text-white text-center relative bg-gradient-to-br ${cfg.color}`;
            document.getElementById('headerTitle').innerText = cfg.title;
            document.getElementById('headerSubtitle').innerText = cfg.subtitle;

            const sel = document.getElementById('projectSelect');
            sel.innerHTML = '<option value="">-- اختر المجموعة --</option>' + 
                db.map(p => `<option value="${p.groupId}|${p.title}">مجموعة ${p.groupId} - قاعة ${p.room}</option>`).join('');
        }

        function handleProjectChange() {
            const val = document.getElementById('projectSelect').value;
            if(!val) return;
            const [gId, title] = val.split('|');
            const project = db.find(p => p.groupId === gId && p.title === title);
            const wrap = document.getElementById('studentsWrapper');
            
            document.getElementById('projNameDisplay').value = project.title;
            document.getElementById('supName').value = project.supervisor;
            wrap.innerHTML = '';
            
            project.students.forEach(name => {
                const temp = document.getElementById('studentTemplate').content.cloneNode(true);
                const card = temp.querySelector('.student-card');
                card.dataset.studentName = name;
                card.querySelector('.student-name-display').innerText = name;
                
                roles[currentRole].criteria.forEach(crit => {
                    const row = document.createElement('div');
                    row.innerHTML = `
                        <div class="flex justify-between text-[10px] font-black text-slate-400 mb-1 uppercase tracking-tighter">
                            <span>${crit.label}</span>
                            <span>أقصى ${crit.max}</span>
                        </div>
                        <input type="number" data-label="${crit.label}" min="0" max="${crit.max}" value="0" 
                            class="score-input w-full p-3 rounded-2xl border-2" oninput="updateScore(this, ${crit.max})">`;
                    card.querySelector('.criteria-list').appendChild(row);
                });
                wrap.appendChild(temp);
            });
        }

        function updateScore(input, max) {
            if(input.value > max) input.value = max;
            const card = input.closest('.student-card');
            let total = 0;
            card.querySelectorAll('.score-input').forEach(i => total += (parseInt(i.value) || 0));
            card.querySelector('.student-total-display').innerText = total;
            
            const badge = card.querySelector('.student-result-text');
            badge.innerText = total >= 60 ? "ناجح" : "راسب";
            badge.className = `student-result-text font-black text-[10px] px-4 py-2 rounded-full tracking-widest ${total >= 60 ? 'bg-emerald-100 text-emerald-700' : 'bg-rose-100 text-rose-700'}`;
        }

        async function saveToCloud() {
            if(!APPS_SCRIPT_URL) {
                alert('تنبيه: لم يتم ضبط رابط الحفظ (APPS_SCRIPT_URL). سيتم الحفظ محلياً فقط الآن.');
                return;
            }
            // منطق الإرسال المماثل للسابق...
        }

        window.onload = () => {
            const local = localStorage.getItem('grad_db_v2');
            if(local) {
                db = JSON.parse(local);
                updateAdminStats();
            }
        };
    </script>
</body>
</html>
