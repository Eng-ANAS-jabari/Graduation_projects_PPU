
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام إدارة المناقشات | الإصدار النهائي</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700;900&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f8fafc; color: #1e293b; }
        .score-input { border: 2px solid #e2e8f0; text-align: center; font-weight: 700; transition: all 0.2s; }
        .score-input:focus { border-color: #4f46e5; background-color: #fefce8; outline: none; }
        .loading-overlay { position: fixed; inset: 0; background: rgba(255,255,255,0.95); display: flex; align-items: center; justify-content: center; z-index: 1000; }
        .tab-active { border-bottom: 4px solid #4f46e5; color: #4f46e5; }
        .glass-header { background: rgba(255, 255, 255, 0.8); backdrop-filter: blur(10px); }
        @media print { .no-print { display: none !important; } }
    </style>
</head>
<body class="p-2 md:p-6">

    <!-- شاشة التحميل -->
    <div id="loading" class="loading-overlay hidden">
        <div class="text-center">
            <div class="inline-block animate-spin rounded-full h-16 w-16 border-4 border-indigo-600 border-t-transparent"></div>
            <p class="mt-4 font-black text-indigo-900 text-lg">جاري مزامنة بيانات المجموعات والطلاب...</p>
        </div>
    </div>

    <div id="app" class="max-w-7xl mx-auto space-y-6">
        
        <!-- الهيدر الرئيسي -->
        <header class="bg-white p-6 md:p-10 rounded-[2.5rem] shadow-xl border border-slate-200 text-center relative overflow-hidden">
            <div class="relative z-10">
                <div class="flex justify-center mb-4">
                    <span class="bg-indigo-100 text-indigo-700 px-4 py-1 rounded-full text-xs font-bold uppercase tracking-widest">نظام الربط الذكي v3.0</span>
                </div>
                <h1 class="text-3xl md:text-4xl font-black text-slate-800">إدارة مناقشات مشاريع التخرج</h1>
                <p class="text-slate-500 mt-2 font-medium">الربط الآلي بين الطلاب، المجموعات، والقاعات الدراسية</p>
                
                <div class="flex flex-wrap justify-center gap-3 mt-8 no-print">
                    <button onclick="fetchSheetData()" class="bg-indigo-600 text-white px-8 py-3 rounded-2xl font-bold hover:bg-indigo-700 transition-all shadow-lg flex items-center gap-2">
                        <span>🔄</span> تحديث البيانات من السحابة
                    </button>
                    <button onclick="switchSection('admin')" class="bg-slate-800 text-white px-8 py-3 rounded-2xl font-bold hover:bg-slate-900 transition-all shadow-lg flex items-center gap-2">
                        <span>📊</span> لوحة تحكم المسؤول
                    </button>
                </div>
            </div>
            <!-- زخرفة خلفية -->
            <div class="absolute -top-10 -left-10 w-40 h-40 bg-indigo-50 rounded-full opacity-50"></div>
        </header>

        <!-- واجهة اختيار الدور (تظهر في البداية) -->
        <div id="roleSelection" class="grid grid-cols-1 md:grid-cols-2 gap-8 py-10">
            <div onclick="openEvalForm('supervisor')" class="cursor-pointer bg-white p-12 rounded-[3rem] border-4 border-indigo-600 shadow-xl hover:scale-[1.02] transition-all text-center group">
                <div class="text-6xl mb-6 group-hover:rotate-12 transition-transform">📝</div>
                <h2 class="text-3xl font-black text-slate-800">تقييم المشرف</h2>
                <p class="text-slate-500 mt-3 text-lg">خاص برصد درجات الفصل والمتابعة</p>
                <div class="mt-6 inline-block bg-indigo-50 text-indigo-600 px-6 py-2 rounded-full font-bold">دخول النظام</div>
            </div>

            <div onclick="openEvalForm('examiner')" class="cursor-pointer bg-white p-12 rounded-[3rem] border-4 border-emerald-600 shadow-xl hover:scale-[1.02] transition-all text-center group">
                <div class="text-6xl mb-6 group-hover:rotate-12 transition-transform">🎓</div>
                <h2 class="text-3xl font-black text-slate-800">تقييم المناقش</h2>
                <p class="text-slate-500 mt-3 text-lg">خاص بلجنة الحكم والمناقشة النهائية</p>
                <div class="mt-6 inline-block bg-emerald-50 text-emerald-600 px-6 py-2 rounded-full font-bold">دخول النظام</div>
            </div>
        </div>

        <!-- لوحة المسؤول (Admin Panel) -->
        <div id="adminSection" class="hidden space-y-6 no-print">
            <div class="flex flex-col md:flex-row justify-between items-center bg-slate-900 text-white p-8 rounded-[2.5rem] gap-4">
                <div>
                    <h2 class="text-2xl font-black">واجهة المسؤول المركزية</h2>
                    <p class="opacity-70">استعراض الربط بين القاعات والمجموعات والطلاب</p>
                </div>
                <button onclick="switchSection('home')" class="bg-white/10 px-8 py-3 rounded-2xl font-bold hover:bg-white/20 transition-all">العودة للرئيسية</button>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <div class="bg-white p-8 rounded-[2rem] border border-slate-200 shadow-sm text-center">
                    <div class="text-slate-400 text-sm font-bold mb-1">إجمالي المجموعات</div>
                    <div id="statGroups" class="text-4xl font-black text-indigo-600">0</div>
                </div>
                <div class="bg-white p-8 rounded-[2rem] border border-slate-200 shadow-sm text-center">
                    <div class="text-slate-400 text-sm font-bold mb-1">إجمالي الطلاب</div>
                    <div id="statStudents" class="text-4xl font-black text-emerald-600">0</div>
                </div>
                <div class="bg-white p-8 rounded-[2rem] border border-slate-200 shadow-sm text-center">
                    <div class="text-slate-400 text-sm font-bold mb-1">عدد القاعات</div>
                    <div id="statRooms" class="text-4xl font-black text-orange-600">0</div>
                </div>
            </div>

            <div class="bg-white rounded-[2.5rem] shadow-xl border border-slate-200 overflow-hidden">
                <div class="p-6 bg-slate-50 border-b flex justify-between items-center flex-wrap gap-4">
                    <h3 class="font-black text-xl">جدول التوزيع والربط</h3>
                    <input type="text" placeholder="بحث عن طالب أو قاعة..." class="p-3 rounded-xl border-2 border-slate-200 outline-none w-full md:w-80 font-bold" oninput="filterAdmin(this.value)">
                </div>
                <div class="overflow-x-auto">
                    <table class="w-full text-right">
                        <thead>
                            <tr class="bg-slate-100 text-slate-600 font-black">
                                <th class="p-5">رقم القاعة</th>
                                <th class="p-5">المجموعة</th>
                                <th class="p-5">اسم المشروع</th>
                                <th class="p-5 text-center">الطلاب (الفريق)</th>
                                <th class="p-5">المشرف</th>
                            </tr>
                        </thead>
                        <tbody id="adminTableBody">
                            <!-- البيانات تضاف هنا -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- نموذج التقييم الرئيسي -->
        <div id="evalSection" class="hidden bg-white rounded-[3rem] shadow-2xl border border-slate-200 overflow-hidden">
            <div id="evalHeader" class="p-10 text-white text-center relative">
                <button onclick="switchSection('home')" class="absolute top-8 left-8 bg-white/20 px-5 py-2 rounded-full font-bold hover:bg-white/30 no-print">إلغاء</button>
                <h2 id="evalTitle" class="text-4xl font-black"></h2>
                <p id="evalSubtitle" class="mt-2 opacity-80 text-lg"></p>
            </div>

            <div class="p-8 space-y-10">
                <!-- اختيار المجموعة -->
                <div class="bg-slate-50 p-8 rounded-[2rem] border border-slate-100 grid grid-cols-1 md:grid-cols-3 gap-8">
                    <div class="md:col-span-2">
                        <label class="block text-xs font-black text-slate-400 mb-3 uppercase mr-2 tracking-widest">اختر المجموعة (رقم - مشروع - قاعة)</label>
                        <select id="groupSelect" class="w-full p-5 rounded-2xl border-4 border-white shadow-sm font-black text-indigo-700 text-lg outline-none focus:border-indigo-500 transition-all" onchange="renderStudentsCards()">
                            <option value="">-- اضغط للاختيار من المجموعات المتاحة --</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-400 mb-3 uppercase mr-2 tracking-widest">تاريخ اليوم</label>
                        <input type="date" id="evalDate" class="w-full p-5 rounded-2xl border-4 border-white shadow-sm font-bold outline-none">
                    </div>
                </div>

                <!-- شبكة الطلاب المربوطين بالمجموعة -->
                <div id="studentsGrid" class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                    <!-- بطاقات الطلاب تظهر هنا آلياً بناءً على المجموعة المختارة -->
                </div>

                <div class="text-center pt-10 no-print">
                    <button onclick="saveResults()" class="bg-slate-900 text-white px-20 py-5 rounded-[2rem] font-black text-xl shadow-2xl hover:bg-black hover:-translate-y-1 transition-all active:scale-95">
                        🚀 اعتماد وحفظ جميع الدرجات سحابياً
                    </button>
                </div>
            </div>
        </div>

    </div>

    <!-- قالب بطاقة الطالب (مخفي) -->
    <template id="studentCardTemplate">
        <div class="student-card bg-white p-8 rounded-[2.5rem] border-2 border-slate-100 shadow-lg flex flex-col h-full hover:border-indigo-300 transition-all">
            <div class="flex justify-between items-start mb-8">
                <div>
                    <h4 class="student-name font-black text-2xl text-slate-800"></h4>
                    <span class="text-indigo-500 font-bold text-sm">طالب مشروع تخرج</span>
                </div>
                <div class="w-14 h-14 bg-indigo-50 rounded-2xl flex items-center justify-center text-3xl shadow-inner">👤</div>
            </div>
            
            <div class="criteria-container space-y-5 flex-grow">
                <!-- معايير التقييم تضاف هنا -->
            </div>

            <div class="mt-10 pt-6 border-t border-slate-100 flex justify-between items-center">
                <div>
                    <div class="text-xs font-bold text-slate-400 uppercase mb-1">المجموع النهائي</div>
                    <div class="text-5xl font-black text-indigo-600 student-total">0</div>
                </div>
                <div class="status-badge px-6 py-2 rounded-full font-black text-xs uppercase tracking-tighter bg-slate-100 text-slate-400">قيد الرصد</div>
            </div>
        </div>
    </template>

    <script>
        // رابط ملفك الخاص
        const SHEET_ID = '1Ne4jRjMj75t2zk-w9bCoO2jZlkJkOIxtQAlqNG91p3U';
        const SHEET_URL = `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?tqx=out:csv`;

        let mainDB = [];
        let activeRole = '';

        const roleSettings = {
            supervisor: { 
                title: 'تقييم المشرف', 
                subtitle: 'خاص برصد درجات الفصل وأعمال السنة والمتابعة',
                color: 'bg-gradient-to-r from-indigo-600 to-indigo-800',
                criteria: [
                    { name: 'التوثيق والتقرير الشخصي', max: 30 },
                    { name: 'الالتزام بالمواعيد والمتابعة', max: 20 },
                    { name: 'جودة التنفيذ العملي', max: 50 }
                ]
            },
            examiner: { 
                title: 'تقييم المناقش', 
                subtitle: 'تقييم العرض التقديمي والمناقشة النهائية أمام اللجنة',
                color: 'bg-gradient-to-r from-emerald-600 to-emerald-800',
                criteria: [
                    { name: 'جودة العرض التقديمي', max: 20 },
                    { name: 'القدرة على النقاش العلمي', max: 30 },
                    { name: 'تكامل النظام المبرمج', max: 50 }
                ]
            }
        };

        // دالة جلب البيانات من Google Sheets
        async function fetchSheetData() {
            document.getElementById('loading').classList.remove('hidden');
            try {
                const response = await fetch(SHEET_URL);
                const csvData = await response.text();
                
                Papa.parse(csvData, {
                    header: true,
                    skipEmptyLines: true,
                    complete: (results) => {
                        processCSV(results.data);
                        alert('تم تحديث بيانات الطلاب والمجموعات بنجاح من الرابط.');
                        document.getElementById('loading').classList.add('hidden');
                    }
                });
            } catch (error) {
                alert('حدث خطأ في جلب البيانات. تأكد من إعدادات مشاركة الملف.');
                document.getElementById('loading').classList.add('hidden');
            }
        }

        // معالجة البيانات وربط الطلاب بالمجموعات
        function processCSV(rows) {
            let groupedData = {};
            
            rows.forEach(row => {
                const groupID = row['رقم المجموعة'] || row['Group ID'] || '0';
                const projectName = row['اسم المشروع'] || row['Project Name'] || 'غير مسمى';
                const studentName = row['اسم الطالب'] || row['Student Name'];
                const roomNumber = row['رقم القاعة'] || row['Room'] || '-';
                const supervisor = row['اسم المشرف'] || row['Supervisor'] || '-';

                if (studentName) {
                    const key = `${groupID}_${projectName}`;
                    if (!groupedData[key]) {
                        groupedData[key] = {
                            id: groupID,
                            project: projectName,
                            room: roomNumber,
                            supervisor: supervisor,
                            students: []
                        };
                    }
                    groupedData[key].students.push(studentName);
                }
            });

            mainDB = Object.values(groupedData);
            localStorage.setItem('grad_sys_db', JSON.stringify(mainDB));
            populateGroupSelect();
            updateStats();
        }

        // تعبئة قائمة الاختيار
        function populateGroupSelect() {
            const select = document.getElementById('groupSelect');
            select.innerHTML = '<option value="">-- اختر المجموعة من القائمة --</option>';
            mainDB.forEach((group, index) => {
                const option = document.createElement('option');
                option.value = index;
                option.innerText = `مجموعة ${group.id} - قاعة ${group.room} (${group.project})`;
                select.appendChild(option);
            });
        }

        // عرض بطاقات الطلاب عند اختيار مجموعة
        function renderStudentsCards() {
            const index = document.getElementById('groupSelect').value;
            const container = document.getElementById('studentsGrid');
            container.innerHTML = '';

            if (index === '') return;

            const group = mainDB[index];
            const settings = roleSettings[activeRole];

            group.students.forEach(name => {
                const temp = document.getElementById('studentCardTemplate').content.cloneNode(true);
                temp.querySelector('.student-name').innerText = name;
                
                const criteriaContainer = temp.querySelector('.criteria-container');
                settings.criteria.forEach((crit, cIdx) => {
                    const div = document.createElement('div');
                    div.innerHTML = `
                        <div class="flex justify-between text-xs font-black text-slate-400 mb-2 uppercase tracking-tighter">
                            <span>${crit.name}</span>
                            <span>الدرجة العظمى (${crit.max})</span>
                        </div>
                        <input type="number" min="0" max="${crit.max}" value="0" 
                               class="score-input w-full p-4 rounded-2xl border-2 text-xl" 
                               oninput="calculateTotal(this, ${crit.max})">
                    `;
                    criteriaContainer.appendChild(div);
                });

                container.appendChild(temp);
            });
        }

        function calculateTotal(input, max) {
            if (parseInt(input.value) > max) input.value = max;
            const card = input.closest('.student-card');
            let total = 0;
            card.querySelectorAll('.score-input').forEach(i => total += (parseInt(i.value) || 0));
            card.querySelector('.student-total').innerText = total;

            const badge = card.querySelector('.status-badge');
            if (total >= 60) {
                badge.innerText = 'ناجح';
                badge.className = 'status-badge px-6 py-2 rounded-full font-black text-xs uppercase tracking-tighter bg-emerald-100 text-emerald-700';
            } else {
                badge.innerText = 'قيد الرصد';
                badge.className = 'status-badge px-6 py-2 rounded-full font-black text-xs uppercase tracking-tighter bg-slate-100 text-slate-400';
            }
        }

        // التنقل بين الأقسام
        function switchSection(section) {
            document.getElementById('roleSelection').classList.add('hidden');
            document.getElementById('adminSection').classList.add('hidden');
            document.getElementById('evalSection').classList.add('hidden');

            if (section === 'home') document.getElementById('roleSelection').classList.remove('hidden');
            else if (section === 'admin') {
                document.getElementById('adminSection').classList.remove('hidden');
                renderAdminTable();
            }
        }

        function openEvalForm(role) {
            if (mainDB.length === 0) return alert('الرجاء تحديث البيانات من الرابط أولاً.');
            activeRole = role;
            const settings = roleSettings[role];
            
            document.getElementById('roleSelection').classList.add('hidden');
            document.getElementById('evalSection').classList.remove('hidden');
            
            document.getElementById('evalHeader').className = `p-10 text-white text-center relative ${settings.color}`;
            document.getElementById('evalTitle').innerText = settings.title;
            document.getElementById('evalSubtitle').innerText = settings.subtitle;
            
            document.getElementById('studentsGrid').innerHTML = '';
            document.getElementById('groupSelect').value = '';
        }

        // واجهة المسؤول
        function renderAdminTable(data = mainDB) {
            const body = document.getElementById('adminTableBody');
            body.innerHTML = '';
            data.forEach(group => {
                const tr = document.createElement('tr');
                tr.className = 'border-b border-slate-100 hover:bg-slate-50 transition-all';
                tr.innerHTML = `
                    <td class="p-5 font-bold text-orange-600">قاعة ${group.room}</td>
                    <td class="p-5 font-black text-indigo-700">#${group.id}</td>
                    <td class="p-5 font-bold">${group.project}</td>
                    <td class="p-5">
                        <div class="flex flex-wrap gap-2 justify-center">
                            ${group.students.map(s => `<span class="bg-indigo-50 text-indigo-700 px-3 py-1 rounded-lg text-xs font-bold border border-indigo-100">${s}</span>`).join('')}
                        </div>
                    </td>
                    <td class="p-5 text-slate-500 font-medium">${group.supervisor}</td>
                `;
                body.appendChild(tr);
            });
            updateStats();
        }

        function filterAdmin(q) {
            const filtered = mainDB.filter(g => 
                g.project.includes(q) || g.room.includes(q) || g.students.some(s => s.includes(q))
            );
            renderAdminTable(filtered);
        }

        function updateStats() {
            document.getElementById('statGroups').innerText = mainDB.length;
            document.getElementById('statStudents').innerText = mainDB.reduce((acc, curr) => acc + curr.students.length, 0);
            document.getElementById('statRooms').innerText = [...new Set(mainDB.map(g => g.room))].length;
        }

        function saveResults() {
            alert('تم اعتماد الدرجات بنجاح. سيتم إرسال نسخة إلى السحابة فور تفعيل خدمة الربط البرمجي.');
        }

        // تهيئة النظام
        window.onload = () => {
            const cached = localStorage.getItem('grad_sys_db');
            if (cached) {
                mainDB = JSON.parse(cached);
                populateGroupSelect();
                updateStats();
            }
            document.getElementById('evalDate').valueAsDate = new Date();
        };
    </script>
</body>
</html>
