<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام تقييم مشاريع التخرج | النسخة المتصلة</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700;900&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f1f5f9; }
        .score-input { border: 2px solid #e2e8f0; transition: all 0.2s; text-align: center; font-weight: 700; font-size: 1.1rem; }
        .score-input:focus { border-color: #4f46e5; outline: none; background-color: #fffbeb; }
        .student-card { transition: transform 0.2s; }
        .student-card:hover { transform: translateY(-5px); }
        .loading-overlay { position: fixed; inset: 0; background: rgba(255,255,255,0.8); display: flex; align-items: center; justify-content: center; z-index: 100; }
        @media print { .no-print { display: none !important; } body { padding: 0; background: white; } }
    </style>
</head>
<body class="p-4 md:p-8">

    <div id="loading" class="loading-overlay hidden">
        <div class="text-center">
            <div class="inline-block animate-spin rounded-full h-12 w-12 border-4 border-indigo-600 border-t-transparent"></div>
            <p class="mt-4 font-bold text-indigo-800">جاري معالجة البيانات...</p>
        </div>
    </div>

    <div id="app" class="max-w-6xl mx-auto space-y-6">
        
        <!-- واجهة اختيار الدور -->
        <div id="roleSelection" class="bg-white p-10 rounded-[2.5rem] shadow-2xl text-center no-print border border-slate-200">
            <h2 class="text-3xl font-black mb-2 text-slate-800">نظام التقييم المتصل بالسحابة</h2>
            <p class="text-slate-500 mb-6">يتم جلب الأسماء حالياً من Google Sheets</p>
            
            <div class="flex justify-center mb-10">
                <button onclick="fetchSheetData()" class="bg-blue-50 text-blue-600 px-6 py-2 rounded-full text-sm font-bold border border-blue-200 hover:bg-blue-100 transition-all">
                    🔄 تحديث الأسماء من الرابط
                </button>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6 max-w-2xl mx-auto">
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

        <!-- نموذج التقييم الرئيسي -->
        <div id="mainContainer" class="hidden bg-white shadow-2xl rounded-[2.5rem] overflow-hidden border border-slate-200">
            <div id="formHeader" class="p-10 text-white text-center relative">
                <button onclick="location.reload()" class="absolute top-8 left-8 bg-white/20 px-4 py-2 rounded-full text-xs font-bold hover:bg-white/30 transition-all no-print">إغلاق</button>
                <h1 id="headerTitle" class="text-4xl font-black"></h1>
                <p id="headerSubtitle" class="mt-2 opacity-80 font-medium"></p>
            </div>

            <form id="evaluationForm" class="p-8 md:p-12 space-y-12">
                <div class="grid grid-cols-1 md:grid-cols-3 gap-8 p-8 bg-slate-50 rounded-3xl border border-slate-100">
                    <div class="space-y-2">
                        <label class="block font-black text-slate-700 text-sm">اسم المشروع</label>
                        <select id="projectSelect" class="w-full p-3 bg-white border border-slate-200 rounded-xl outline-none font-bold text-indigo-600 shadow-sm" onchange="handleProjectChange()">
                            <option value="">-- اختر المشروع --</option>
                        </select>
                    </div>
                    <div class="space-y-2">
                        <label class="block font-black text-slate-700 text-sm">المشرف</label>
                        <input type="text" id="supName" class="w-full p-3 bg-white border border-slate-200 rounded-xl font-bold" readonly>
                    </div>
                    <div class="space-y-2">
                        <label class="block font-black text-slate-700 text-sm">تاريخ التقييم</label>
                        <input type="date" id="evalDate" class="w-full p-3 bg-white border border-slate-200 rounded-xl font-bold" value="${new Date().toISOString().split('T')[0]}">
                    </div>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-2 xl:grid-cols-3 gap-8" id="studentsWrapper"></div>

                <div class="pt-10 flex flex-wrap justify-center gap-4 border-t border-slate-100 no-print">
                    <button type="button" onclick="saveToCloud()" class="bg-indigo-600 text-white px-10 py-4 rounded-2xl font-black hover:bg-indigo-700 transition-all shadow-xl">
                        🚀 حفظ النتائج في Google Sheets
                    </button>
                    <button type="button" onclick="window.print()" class="bg-slate-100 text-slate-700 px-8 py-4 rounded-2xl font-black hover:bg-slate-200 transition-all">
                        📄 طباعة التقرير
                    </button>
                </div>
            </form>
        </div>
    </div>

    <template id="studentTemplate">
        <div class="student-card bg-white border border-slate-200 rounded-[2.5rem] p-8 shadow-sm hover:shadow-md transition-all flex flex-col h-full">
            <div class="flex justify-between items-start mb-6">
                <h4 class="student-name-display text-2xl font-black text-slate-800"></h4>
                <div class="text-3xl">👤</div>
            </div>
            <div class="criteria-list space-y-5 flex-grow"></div>
            <div class="mt-10 pt-6 border-t border-slate-100 flex justify-between items-end">
                <div>
                    <span class="text-4xl font-black text-indigo-600 student-total-display">0</span>
                    <span class="text-sm font-bold text-slate-400">/ 100</span>
                </div>
                <div class="student-result-text font-black text-xs px-5 py-2 rounded-full bg-slate-100 text-slate-500 uppercase tracking-wide">قيد التقييم</div>
            </div>
        </div>
    </template>

    <script>
        const SHEET_ID = '1Ne4jRjMj75t2zk-w9bCoO2jZlkJkOIxtQAlqNG91p3U';
        const SHEET_URL = `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?tqx=out:csv`;
        
        // رابط الـ Web App الخاص بك (سيتم شرحه أدناه)
        const APPS_SCRIPT_URL = ''; 

        let db = [];
        let currentRole = '';

        const roles = {
            supervisor: { title: "تقييم المشرف", subtitle: "رصد درجات الفصل", color: "from-indigo-600 to-indigo-800", criteria: [{id:'c1',label:'التوثيق',max:25},{id:'c2',label:'العملي',max:35},{id:'c3',label:'المتابعة',max:20},{id:'c4',label:'الأداء',max:20}] },
            examiner: { title: "تقييم المناقش", subtitle: "رصد درجات لجنة الحكم", color: "from-emerald-600 to-emerald-800", criteria: [{id:'c1',label:'التقرير',max:25},{id:'c2',label:'البرمجة',max:25},{id:'c3',label:'المناقشة',max:25},{id:'c4',label:'العرض',max:25}] }
        };

        async function fetchSheetData() {
            document.getElementById('loading').classList.remove('hidden');
            try {
                const response = await fetch(SHEET_URL);
                const csvData = await response.text();
                
                Papa.parse(csvData, {
                    header: true,
                    complete: (results) => {
                        const raw = results.data;
                        let projects = {};
                        
                        raw.forEach(row => {
                            const pName = row['اسم المشروع'] || row['Project Name'];
                            const sName = row['اسم الطالب'] || row['Student Name'];
                            const sup = row['اسم المشرف'] || row['Supervisor'];
                            
                            if(pName && sName) {
                                if(!projects[pName]) projects[pName] = { title: pName, supervisor: sup, students: [] };
                                projects[pName].students.push(sName);
                            }
                        });
                        
                        db = Object.values(projects);
                        localStorage.setItem('grad_db_cloud', JSON.stringify(db));
                        alert('تم تحديث البيانات بنجاح من Google Sheets!');
                    }
                });
            } catch (err) {
                alert('فشل جلب البيانات. تأكد أن الملف "عام" (Anyone with the link can view)');
            } finally {
                document.getElementById('loading').classList.add('hidden');
            }
        }

        function setRole(role) {
            if(db.length === 0) {
                const local = localStorage.getItem('grad_db_cloud');
                if(local) db = JSON.parse(local);
                else return alert('الرجاء الضغط على "تحديث الأسماء" أولاً');
            }
            
            currentRole = role;
            document.getElementById('roleSelection').classList.add('hidden');
            document.getElementById('mainContainer').classList.remove('hidden');
            const cfg = roles[role];
            document.getElementById('formHeader').className = `p-10 text-white text-center relative bg-gradient-to-r ${cfg.color}`;
            document.getElementById('headerTitle').innerText = cfg.title;
            document.getElementById('headerSubtitle').innerText = cfg.subtitle;

            const sel = document.getElementById('projectSelect');
            sel.innerHTML = '<option value="">-- اختر المشروع --</option>' + 
                db.map(p => `<option value="${p.title}">${p.title}</option>`).join('');
        }

        function handleProjectChange() {
            const title = document.getElementById('projectSelect').value;
            const project = db.find(p => p.title === title);
            const wrap = document.getElementById('studentsWrapper');
            if(!project) return wrap.innerHTML = '';

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
                        <div class="flex justify-between text-[10px] font-bold text-slate-400 mb-1 uppercase">
                            <span>${crit.label} (أقصى ${crit.max})</span>
                        </div>
                        <input type="number" data-label="${crit.label}" min="0" max="${crit.max}" value="0" 
                            class="score-input w-full p-2 rounded-xl border" oninput="updateScore(this, ${crit.max})">`;
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
            badge.className = `student-result-text font-black text-xs px-5 py-2 rounded-full ${total >= 60 ? 'bg-emerald-100 text-emerald-700' : 'bg-rose-100 text-rose-700'}`;
        }

        async function saveToCloud() {
            if(!APPS_SCRIPT_URL) {
                alert('يجب ربط البرنامج بـ Apps Script أولاً ليتمكن من التسجيل التلقائي في الملف.');
                return;
            }

            document.getElementById('loading').classList.remove('hidden');
            const results = [];
            document.querySelectorAll('.student-card').forEach(card => {
                results.push({
                    project: document.getElementById('projectSelect').value,
                    student: card.dataset.studentName,
                    role: roles[currentRole].title,
                    total: card.querySelector('.student-total-display').innerText,
                    date: document.getElementById('evalDate').value
                });
            });

            try {
                // إرسال البيانات للسكربت
                await fetch(APPS_SCRIPT_URL, {
                    method: 'POST',
                    mode: 'no-cors',
                    cache: 'no-cache',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(results)
                });
                alert('تم إرسال البيانات بنجاح إلى الملف!');
            } catch (err) {
                alert('حدث خطأ أثناء الاتصال بالسحابة.');
            } finally {
                document.getElementById('loading').classList.add('hidden');
            }
        }

        // محاولة جلب أولية عند الفتح
        window.onload = () => {
            const local = localStorage.getItem('grad_db_cloud');
            if(local) db = JSON.parse(local);
            else fetchSheetData();
        };
    </script>
</body>
</html>
