<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام إدارة المناقشات | الإصدار النهائي</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700;900&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
</head>
<body class="p-2 md:p-6 bg-gray-50 min-h-screen" style="font-family: 'Tajawal', sans-serif;">

    <div id="loading" class="fixed inset-0 bg-white/90 flex items-center justify-center z-50 hidden">
        <div class="text-center">
            <div class="block animate-spin rounded-full h-16 w-16 border-4 border-indigo-600 border-t-transparent"></div>
            <p class="mt-4 font-black text-indigo-900 text-lg">جاري مزامنة البيانات...</p>
        </div>
    </div>

    <div id="app" class="max-w-7xl mx-auto space-y-6">
        <!-- رأس الصفحة -->
        <header class="text-center py-6 bg-gradient-to-r from-indigo-900 to-indigo-700 rounded-2xl shadow-xl text-white relative">
            <div class="absolute top-4 left-4 text-sm opacity-80">
                <button onclick="showCopyright()" class="hover:text-yellow-300 transition">
                    <i class="fas fa-copyright"></i> حقوق النشر
                </button>
            </div>
            <h1 class="text-3xl md:text-4xl font-black mb-2">نظام إدارة تقييم المناقشات</h1>
            <p class="text-lg opacity-90">الإصدار النهائي - نظام متكامل للتقييم الإلكتروني</p>
            <div class="mt-4 flex flex-wrap justify-center gap-4">
                <button onclick="switchSection('home')" class="px-5 py-2 bg-white/20 hover:bg-white/30 rounded-lg transition"><i class="fas fa-home ml-2"></i>الرئيسية</button>
                <button onclick="switchSection('evaluation')" class="px-5 py-2 bg-white/20 hover:bg-white/30 rounded-lg transition"><i class="fas fa-clipboard-check ml-2"></i>التقييم</button>
                <button onclick="switchSection('admin')" class="px-5 py-2 bg-white/20 hover:bg-white/30 rounded-lg transition"><i class="fas fa-chart-bar ml-2"></i>الإحصائيات</button>
                <button onclick="fetchSheetData()" class="px-5 py-2 bg-emerald-500 hover:bg-emerald-600 rounded-lg transition"><i class="fas fa-sync-alt ml-2"></i>مزامنة البيانات</button>
            </div>
        </header>

        <!-- قسم اختيار الدور -->
        <section id="roleSection" class="bg-white p-6 rounded-2xl shadow-lg">
            <h2 class="text-2xl font-black text-gray-800 mb-4"><i class="fas fa-user-tie ml-2"></i>اختر دور التقييم</h2>
            <div class="grid md:grid-cols-2 gap-6">
                <div class="text-center p-6 border-2 border-indigo-200 rounded-xl hover:border-indigo-500 transition cursor-pointer" onclick="setActiveRole('supervisor')">
                    <div class="h-16 w-16 rounded-full bg-indigo-100 flex items-center justify-center mx-auto mb-4">
                        <i class="fas fa-user-graduate text-3xl text-indigo-600"></i>
                    </div>
                    <h3 class="text-xl font-black text-indigo-700">تقييم المشرف</h3>
                    <p class="text-gray-600 mt-2">خاص برصد درجات الفصل والمتابعة المستمرة</p>
                </div>
                <div class="text-center p-6 border-2 border-emerald-200 rounded-xl hover:border-emerald-500 transition cursor-pointer" onclick="setActiveRole('examiner')">
                    <div class="h-16 w-16 rounded-full bg-emerald-100 flex items-center justify-center mx-auto mb-4">
                        <i class="fas fa-chalkboard-teacher text-3xl text-emerald-600"></i>
                    </div>
                    <h3 class="text-xl font-black text-emerald-700">تقييم المناقش</h3>
                    <p class="text-gray-600 mt-2">تقييم العرض والمناقشة العلمية</p>
                </div>
            </div>
            
            <!-- إضافة: إدخال اسم الناقش -->
            <div class="mt-8 pt-6 border-t border-gray-200">
                <h3 class="text-xl font-black text-gray-800 mb-4"><i class="fas fa-user-edit ml-2"></i>إعدادات الناقش المشرف </h3>
                <div class="grid md:grid-cols-2 gap-4">
                    <div>
                        <label class="block font-bold text-gray-700 mb-2">اسم الناقش/المشرف :</label>
                        <input type="text" id="examinerName" placeholder="اسم الناقش/المشرف ..." class="w-full p-3 border-2 border-gray-300 rounded-lg focus:border-indigo-500 focus:ring-2 focus:ring-indigo-200 outline-none">
                    </div>
                    <div>
                        <label class="block font-bold text-gray-700 mb-2">التوقيع :</label>
                        <div class="flex gap-2">
                            <input type="text" id="examinerSignature" placeholder="التوقيع..." class="flex-1 p-3 border-2 border-gray-300 rounded-lg">
                            <button onclick="clearSignature()" class="px-4 py-3 bg-gray-200 hover:bg-gray-300 rounded-lg"><i class="fas fa-eraser"></i></button>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- قسم التقييم -->
        <section id="evaluationSection" class="hidden bg-white p-6 rounded-2xl shadow-lg">
            <div class="flex flex-wrap items-center justify-between mb-6">
                <div>
                    <h2 id="evalTitle" class="text-2xl font-black"></h2>
                    <p id="evalSubtitle" class="text-gray-600"></p>
                    <p id="examinerInfo" class="text-sm text-indigo-600 mt-1"></p>
                </div>
                <div class="flex items-center gap-4">
                    <div id="roleBadge" class="px-4 py-2 rounded-full text-white font-bold"></div>
                    <button onclick="switchSection('home')" class="px-4 py-2 text-gray-600 hover:text-gray-900"><i class="fas fa-times"></i></button>
                </div>
            </div>

            <div class="grid md:grid-cols-3 gap-6 mb-6">
                <div class="md:col-span-2">
                    <label class="block font-bold text-gray-700 mb-2">اختر مجموعة الطلاب:</label>
                    <select id="groupSelect" onchange="renderStudentsCards()" class="w-full p-3 border-2 border-gray-300 rounded-lg focus:border-indigo-500 focus:ring-2 focus:ring-indigo-200 outline-none">
                        <option value="">-- اختر مجموعة --</option>
                    </select>
                </div>
                <div>
                    <label class="block font-bold text-gray-700 mb-2">تاريخ التقييم:</label>
                    <input type="date" id="evalDate" class="w-full p-3 border-2 border-gray-300 rounded-lg">
                </div>
            </div>

            <div id="studentsContainer" class="space-y-4">
                <!-- هنا سيتم عرض بطاقات الطلاب -->
            </div>

            <div id="criteriaSection" class="mt-8 hidden">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-xl font-black text-gray-800">معايير التقييم</h3>
                    <button onclick="addCustomCriterion()" class="px-4 py-2 bg-blue-100 text-blue-700 hover:bg-blue-200 rounded-lg">
                        <i class="fas fa-plus ml-2"></i>إضافة معيار
                    </button>
                </div>
                <div id="criteriaList" class="space-y-4">
                    <!-- سيتم إضافة المعايير ديناميكيًا -->
                </div>
                <div class="mt-6 p-4 bg-gradient-to-r from-gray-50 to-gray-100 rounded-xl border border-gray-200">
                    <div class="flex justify-between items-center">
                        <span class="font-black text-lg text-gray-800">المجموع الكلي:</span>
                        <span id="totalScore" class="text-3xl font-black text-indigo-700">0</span>
                    </div>
                    <div class="mt-2 text-sm text-gray-600 text-center" id="maxScoreInfo"></div>
                </div>
                <div class="mt-6 flex gap-4">
                    <button onclick="saveResults()" class="flex-1 py-3 bg-gradient-to-r from-indigo-600 to-indigo-800 hover:from-indigo-700 hover:to-indigo-900 text-white font-black rounded-lg transition shadow-md hover:shadow-lg">
                        <i class="fas fa-save ml-2"></i>حفظ التقييم
                    </button>
                    <button onclick="resetForm()" class="px-6 py-3 bg-gray-200 hover:bg-gray-300 text-gray-800 font-bold rounded-lg transition">
                        <i class="fas fa-redo ml-2"></i>إعادة تعيين
                    </button>
                </div>
            </div>
        </section>

        <!-- قسم الإحصائيات -->
        <section id="adminSection" class="hidden bg-white p-6 rounded-2xl shadow-lg">
            <div class="flex flex-wrap justify-between items-center mb-6">
                <h2 class="text-2xl font-black text-gray-800"><i class="fas fa-chart-bar ml-2"></i>لوحة الإحصائيات والنتائج</h2>
                <div class="flex gap-2 mt-2">
                    <button onclick="shareResults()" class="px-4 py-2 bg-green-600 hover:bg-green-700 text-white rounded-lg">
                        <i class="fab fa-whatsapp ml-2"></i>مشاركة واتساب
                    </button>
                    <button onclick="printPDF()" class="px-4 py-2 bg-red-600 hover:bg-red-700 text-white rounded-lg">
                        <i class="fas fa-file-pdf ml-2"></i>طباعة PDF
                    </button>
                    <button onclick="exportToExcel()" class="px-4 py-2 bg-emerald-600 hover:bg-emerald-700 text-white rounded-lg">
                        <i class="fas fa-file-excel ml-2"></i>تصدير إكسل
                    </button>
                </div>
            </div>
            
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4 mb-6">
                <div class="bg-gradient-to-r from-indigo-500 to-indigo-600 text-white p-4 rounded-xl">
                    <div class="text-3xl font-black" id="totalGroups">0</div>
                    <div class="text-sm opacity-90">المجموعات الكلية</div>
                </div>
                <div class="bg-gradient-to-r from-emerald-500 to-emerald-600 text-white p-4 rounded-xl">
                    <div class="text-3xl font-black" id="totalStudents">0</div>
                    <div class="text-sm opacity-90">إجمالي الطلاب</div>
                </div>
                <div class="bg-gradient-to-r from-amber-500 to-amber-600 text-white p-4 rounded-xl">
                    <div class="text-3xl font-black" id="evaluatedCount">0</div>
                    <div class="text-sm opacity-90">تم تقييمهم</div>
                </div>
                <div class="bg-gradient-to-r from-purple-500 to-purple-600 text-white p-4 rounded-xl">
                    <div class="text-3xl font-black" id="avgScore">0</div>
                    <div class="text-sm opacity-90">متوسط الدرجات</div>
                </div>
            </div>

            <div class="mb-6">
                <label class="block font-bold text-gray-700 mb-2">فلترة النتائج:</label>
                <div class="flex flex-wrap gap-4">
                    <select id="filterRole" onchange="filterAdmin()" class="p-2 border rounded-lg">
                        <option value="">جميع الأدوار</option>
                        <option value="supervisor">تقييم المشرف</option>
                        <option value="examiner">تقييم المناقش</option>
                    </select>
                    <select id="filterGroup" onchange="filterAdmin()" class="p-2 border rounded-lg">
                        <option value="">جميع المجموعات</option>
                    </select>
                    <input type="text" id="searchStudent" onkeyup="filterAdmin()" placeholder="بحث عن طالب..." class="p-2 border rounded-lg flex-grow">
                    <select id="filterExaminer" onchange="filterAdmin()" class="p-2 border rounded-lg">
                        <option value="">جميع الناقشين</option>
                    </select>
                </div>
            </div>

            <div class="overflow-x-auto">
                <table class="min-w-full bg-white border border-gray-300">
                    <thead class="bg-gray-100">
                        <tr>
                            <th class="py-3 px-4 border text-right">اسم الطالب</th>
                            <th class="py-3 px-4 border text-right">المجموعة</th>
                            <th class="py-3 px-4 border text-right">المشروع</th>
                            <th class="py-3 px-4 border text-right">نوع التقييم</th>
                            <th class="py-3 px-4 border text-right">الناقش</th>
                            <th class="py-3 px-4 border text-right">الدرجة</th>
                            <th class="py-3 px-4 border text-right">التقييم</th>
                            <th class="py-3 px-4 border text-right">التاريخ</th>
                            <th class="py-3 px-4 border text-right">الإجراءات</th>
                        </tr>
                    </thead>
                    <tbody id="resultsTable">
                        <!-- سيتم ملء الجدول ديناميكيًا -->
                    </tbody>
                </table>
            </div>

            <div class="mt-6 flex gap-4 flex-wrap">
                <button onclick="exportToCSV()" class="px-6 py-3 bg-gradient-to-r from-green-500 to-green-700 hover:from-green-600 hover:to-green-800 text-white font-bold rounded-lg transition">
                    <i class="fas fa-file-export ml-2"></i>تصدير لـ CSV
                </button>
                <button onclick="exportToExcelFull()" class="px-6 py-3 bg-gradient-to-r from-blue-500 to-blue-700 hover:from-blue-600 hover:to-blue-800 text-white font-bold rounded-lg transition">
                    <i class="fas fa-file-excel ml-2"></i>تصدير إكسل كامل
                </button>
                <button onclick="clearAllData()" class="px-6 py-3 bg-gradient-to-r from-red-500 to-red-700 hover:from-red-600 hover:to-red-800 text-white font-bold rounded-lg transition">
                    <i class="fas fa-trash-alt ml-2"></i>حذف جميع البيانات
                </button>
                <button onclick="quickEvaluation()" class="px-6 py-3 bg-gradient-to-r from-purple-500 to-purple-700 hover:from-purple-600 hover:to-purple-800 text-white font-bold rounded-lg transition">
                    <i class="fas fa-bolt ml-2"></i>تقييم سريع
                </button>
            </div>
        </section>

        <!-- قسم حقوق النشر -->
        <div id="copyrightModal" class="fixed inset-0 bg-black/50 hidden items-center justify-center z-50 p-4">
            <div class="bg-white rounded-2xl shadow-2xl max-w-md w-full p-6">
                <div class="text-center mb-6">
                    <i class="fas fa-copyright text-4xl text-indigo-600 mb-4"></i>
                    <h3 class="text-2xl font-black text-gray-800">حقوق النشر والتطوير</h3>
                </div>
                <div class="space-y-4">
                    <div class="p-4 bg-gradient-to-r from-indigo-50 to-blue-50 rounded-xl">
                        <p class="font-bold text-indigo-800">المهندس:</p>
                        <p class="text-lg font-black text-gray-800">أنس جعبري</p>
                        <p class="text-gray-600 mt-2">مهندس مساحة </p>
                    </div>
                    <div class="p-4 bg-gradient-to-r from-emerald-50 to-green-50 rounded-xl">
                        <p class="font-bold text-emerald-800">النسخة:</p>
                    </div>   
                    </div>
                </div>
                <div class="mt-6 text-center">
                    <button onclick="closeCopyright()" class="px-6 py-3 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-lg">
                        فهمت، شكرًا
                    </button>
                </div>
            </div>
        </div>

        <!-- قسم التقييم السريع -->
        <div id="quickEvalModal" class="fixed inset-0 bg-black/50 hidden items-center justify-center z-50 p-4">
            <div class="bg-white rounded-2xl shadow-2xl max-w-2xl w-full p-6 max-h-[90vh] overflow-y-auto">
                <div class="flex justify-between items-center mb-6">
                    <h3 class="text-2xl font-black text-gray-800"><i class="fas fa-bolt ml-2"></i>التقييم السريع للمجموعة</h3>
                    <button onclick="closeQuickEval()" class="text-gray-500 hover:text-gray-800"><i class="fas fa-times"></i></button>
                </div>
                
                <div class="grid md:grid-cols-3 gap-4 mb-6">
                    <div>
                        <label class="block font-bold text-gray-700 mb-2">المجموعة:</label>
                        <select id="quickGroupSelect" class="w-full p-3 border-2 border-gray-300 rounded-lg">
                            <option value="">اختر المجموعة</option>
                        </select>
                    </div>
                    <div>
                        <label class="block font-bold text-gray-700 mb-2">الدرجة المشتركة:</label>
                        <input type="number" id="commonScore" min="0" max="100" value="70" class="w-full p-3 border-2 border-gray-300 rounded-lg">
                    </div>
                    <div>
                        <label class="block font-bold text-gray-700 mb-2">نوع التقييم:</label>
                        <select id="quickRole" class="w-full p-3 border-2 border-gray-300 rounded-lg">
                            <option value="supervisor">تقييم المشرف</option>
                            <option value="examiner">تقييم المناقش</option>
                        </select>
                    </div>
                </div>

                <div id="quickStudentsList" class="space-y-3 mb-6">
                    <!-- قائمة الطلاب -->
                </div>

                <div class="flex gap-4">
                    <button onclick="applyCommonScore()" class="flex-1 py-3 bg-gradient-to-r from-purple-600 to-purple-800 text-white font-black rounded-lg">
                        <i class="fas fa-check-circle ml-2"></i>تطبيق الدرجة المشتركة
                    </button>
                    <button onclick="closeQuickEval()" class="px-6 py-3 bg-gray-200 hover:bg-gray-300 rounded-lg">
                        إلغاء
                    </button>
                </div>
            </div>
        </div>

        <!-- قسم تقييم فردي -->
        <div id="studentEvalModal" class="fixed inset-0 bg-black/50 hidden items-center justify-center z-50 p-4">
            <div class="bg-white rounded-2xl shadow-2xl max-w-md w-full p-6">
                <div class="text-center mb-6">
                    <h3 id="studentNameTitle" class="text-2xl font-black text-gray-800"></h3>
                    <p id="studentGroupInfo" class="text-gray-600 mt-2"></p>
                </div>
                
                <div class="space-y-4 mb-6">
                    <div>
                        <label class="block font-bold text-gray-700 mb-2">الدرجة:</label>
                        <input type="number" id="individualScore" min="0" max="100" class="w-full p-3 border-2 border-gray-300 rounded-lg">
                    </div>
                    <div>
                        <label class="block font-bold text-gray-700 mb-2">ملاحظات:</label>
                        <textarea id="studentNotes" class="w-full p-3 border-2 border-gray-300 rounded-lg" rows="3" placeholder="أدخل ملاحظات إضافية..."></textarea>
                    </div>
                </div>

                <div class="flex gap-4">
                    <button onclick="saveIndividualScore()" class="flex-1 py-3 bg-gradient-to-r from-indigo-600 to-indigo-800 text-white font-black rounded-lg">
                        حفظ التقييم
                    </button>
                    <button onclick="closeStudentEval()" class="px-6 py-3 bg-gray-200 hover:bg-gray-300 rounded-lg">
                        إلغاء
                    </button>
                </div>
            </div>
        </div>

        <!-- قسم التعليمات -->
        <footer class="bg-gradient-to-r from-gray-800 to-gray-900 text-white p-6 rounded-2xl shadow-lg">
            <h3 class="text-xl font-black mb-4"><i class="fas fa-info-circle ml-2"></i>تعليمات الاستخدام</h3>
            <div class="grid md:grid-cols-3 gap-6">
                <div class="space-y-2">
                    <p class="font-bold"><i class="fas fa-sync-alt ml-2"></i>مزامنة البيانات</p>
                    <p class="text-sm opacity-90">انقر على زر "مزامنة البيانات" لاستيراد أحدث البيانات من Google Sheets.</p>
                </div>
                <div class="space-y-2">
                    <p class="font-bold"><i class="fas fa-share-alt ml-2"></i>المشاركة والطباعة</p>
                    <p class="text-sm opacity-90">يمكنك مشاركة النتائج عبر واتساب، طباعة PDF، أو تصدير لإكسل.</p>
                </div>
                <div class="space-y-2">
                    <p class="font-bold"><i class="fas fa-bolt ml-2"></i>التقييم السريع</p>
                    <p class="text-sm opacity-90">استخدم خاصية التقييم السريع لإعطاء درجة مشتركة لمجموعة كاملة.</p>
                </div>
            </div>
            <div class="mt-6 pt-4 border-t border-white/20 text-center text-sm opacity-80">
                <p>نظام إدارة تقييم المناقشات -  تطوير: أنس جعبري</p>
            </div>
        </footer>
    </div>

    <script>
        // -------------------
        // ✅ الرابط الجديد: CSV مباشر من Google Sheets
        const SHEET_CSV_URL = "https://docs.google.com/spreadsheets/d/1Ne4jRjMj75t2zk-w9bCoO2jZlkJkOIxtQAlqNG91p3U/export?format=csv";
        let mainDB = [];
        let activeRole = '';
        let resultsDB = JSON.parse(localStorage.getItem('grad_sys_results')) || [];
        let customCriteria = JSON.parse(localStorage.getItem('custom_criteria')) || [];
        let examiners = JSON.parse(localStorage.getItem('examiners_list')) || [];

        const roleSettings = {
            supervisor: { 
                title: 'تقييم المشرف', 
                subtitle: 'خاص برصد درجات الفصل والمتابعة', 
                color: 'bg-gradient-to-r from-indigo-600 to-indigo-800',
                badge: 'bg-indigo-600',
                criteria: [
                    { name: 'التوثيق والتقرير الشخصي', max: 30 },
                    { name: 'الالتزام بالمواعيد والمتابعة', max: 20 },
                    { name: 'جودة التنفيذ العملي', max: 50 }
                ]
            },
            examiner: { 
                title: 'تقييم المناقش', 
                subtitle: 'تقييم العرض والمناقشة', 
                color: 'bg-gradient-to-r from-emerald-600 to-emerald-800',
                badge: 'bg-emerald-600',
                criteria: [
                    { name: 'جودة العرض التقديمي', max: 20 },
                    { name: 'القدرة على النقاش العلمي', max: 30 },
                    { name: 'تكامل النظام المبرمج', max: 50 }
                ]
            }
        };

        // إضافة معايير مخصصة إلى الأدوار
        function addCustomCriterion() {
            const criterionName = prompt('أدخل اسم المعيار الجديد:', 'معيار إضافي');
            if (!criterionName) return;
            
            const criterionMax = prompt('أدخل القيمة القصوى للمعيار:', '10');
            const maxValue = parseInt(criterionMax) || 10;
            
            customCriteria.push({
                name: criterionName,
                max: maxValue,
                role: activeRole
            });
            
            localStorage.setItem('custom_criteria', JSON.stringify(customCriteria));
            alert(`تم إضافة المعيار: ${criterionName} (${maxValue} نقطة)`);
        }

        async function fetchSheetData() {
            document.getElementById('loading').classList.remove('hidden');
            try {
                const res = await fetch(SHEET_CSV_URL);
                const csvText = await res.text();

                Papa.parse(csvText, {
                    header: true,
                    skipEmptyLines: true,
                    complete: (results) => {
                        processCSV(results.data);
                        alert('تم تحديث البيانات بنجاح من Google Sheets.');
                        document.getElementById('loading').classList.add('hidden');
                    },
                    error: (err) => {
                        alert('خطأ في تحليل البيانات: ' + err.message);
                        document.getElementById('loading').classList.add('hidden');
                    }
                });
            } catch (err) {
                alert('خطأ في جلب البيانات! تأكد من أن الملف عام للجميع.');
                console.error(err);
                document.getElementById('loading').classList.add('hidden');
            }
        }

        function processCSV(rows) {
            let groupedData = {};

            rows.forEach(row => {
                const studentName = row['اسم الطالب'];
                if (!studentName) return;

                const groupID = row['رقم المجموعة'] || 'غير معروف';
                const projectName = row['اسم المشروع'] || '';
                const roomNumber = row['رقم القاعة'] || '';
                const supervisor = row['اسم المشرف'] || '';

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
            });

            mainDB = Object.values(groupedData);
            localStorage.setItem('grad_sys_db', JSON.stringify(mainDB));
            populateGroupSelect();
            updateStats();
            updateExaminersFilter();
        }

        function setActiveRole(role) {
            activeRole = role;
            const settings = roleSettings[role];
            
            document.getElementById('roleSection').classList.add('hidden');
            document.getElementById('evaluationSection').classList.remove('hidden');
            
            document.getElementById('evalTitle').textContent = settings.title;
            document.getElementById('evalSubtitle').textContent = settings.subtitle;
            document.getElementById('roleBadge').textContent = settings.title;
            document.getElementById('roleBadge').className = `px-4 py-2 rounded-full text-white font-bold ${settings.badge}`;
            
            // عرض اسم الناقش إن وجد
            const examinerName = document.getElementById('examinerName').value;
            document.getElementById('examinerInfo').textContent = examinerName ? `الناقش: ${examinerName}` : '';
            
            document.getElementById('evalDate').valueAsDate = new Date();
        }

        function populateGroupSelect() {
            const select = document.getElementById('groupSelect');
            const filterSelect = document.getElementById('filterGroup');
            const quickSelect = document.getElementById('quickGroupSelect');
            
            select.innerHTML = '<option value="">-- اختر مجموعة --</option>';
            filterSelect.innerHTML = '<option value="">جميع المجموعات</option>';
            quickSelect.innerHTML = '<option value="">اختر المجموعة</option>';
            
            mainDB.forEach(group => {
                const option = document.createElement('option');
                option.value = group.id;
                option.textContent = `المجموعة ${group.id}: ${group.project} (${group.students.length} طلاب)`;
                select.appendChild(option);
                
                const filterOption = document.createElement('option');
                filterOption.value = group.id;
                filterOption.textContent = `المجموعة ${group.id}`;
                filterSelect.appendChild(filterOption);
                
                const quickOption = document.createElement('option');
                quickOption.value = group.id;
                quickOption.textContent = `المجموعة ${group.id} - ${group.project}`;
                quickSelect.appendChild(quickOption);
            });
        }

        function renderStudentsCards() {
            const groupId = document.getElementById('groupSelect').value;
            if (!groupId) {
                document.getElementById('studentsContainer').innerHTML = '';
                document.getElementById('criteriaSection').classList.add('hidden');
                return;
            }

            const group = mainDB.find(g => g.id === groupId);
            if (!group) return;

            let studentsHTML = '<div class="grid md:grid-cols-2 gap-4">';
            
            group.students.forEach(student => {
                // البحث إذا كان الطالب قد تم تقييمه مسبقاً
                const existingEval = resultsDB.find(r => 
                    r.student === student && r.group === groupId && r.role === activeRole
                );
                
                studentsHTML += `
                <div class="border border-gray-300 rounded-xl p-4 hover:shadow-md transition ${existingEval ? 'bg-gradient-to-r from-green-50 to-emerald-50' : ''}">
                    <div class="flex items-center justify-between">
                        <div>
                            <h4 class="font-bold text-lg text-gray-800">${student}</h4>
                            <p class="text-sm text-gray-600">المجموعة: ${group.id} | المشروع: ${group.project}</p>
                            ${existingEval ? `
                            <div class="mt-2 flex items-center">
                                <span class="px-2 py-1 bg-green-100 text-green-800 text-xs rounded">تم التقييم</span>
                                <span class="mr-2 text-sm font-bold">${existingEval.total} درجة</span>
                            </div>
                            ` : ''}
                        </div>
                        <div class="flex gap-2">
                            <button onclick="openEvalForm('${student}')" class="px-4 py-2 ${roleSettings[activeRole].color} text-white font-bold rounded-lg hover:opacity-90 transition">
                                <i class="fas fa-edit ml-2"></i>تقييم
                            </button>
                            <button onclick="quickStudentEval('${student}', '${groupId}')" class="px-3 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700">
                                <i class="fas fa-bolt"></i>
                            </button>
                        </div>
                    </div>
                </div>`;
            });
            
            studentsHTML += '</div>';
            document.getElementById('studentsContainer').innerHTML = studentsHTML;
        }

        function openEvalForm(studentName) {
            const criteriaList = document.getElementById('criteriaList');
            criteriaList.innerHTML = '';
            
            const settings = roleSettings[activeRole];
            const customForRole = customCriteria.filter(c => c.role === activeRole);
            const allCriteria = [...settings.criteria, ...customForRole];
            
            let maxTotal = 0;
            
            allCriteria.forEach((criterion, index) => {
                maxTotal += criterion.max;
                
                const criterionHTML = `
                <div class="bg-gray-50 p-4 rounded-xl border border-gray-200">
                    <div class="flex justify-between items-center mb-3">
                        <div class="flex items-center">
                            <label class="font-bold text-gray-800">${criterion.name}</label>
                            ${criterion.role ? '<span class="mr-2 text-xs bg-blue-100 text-blue-800 px-2 py-1 rounded">مخصص</span>' : ''}
                        </div>
                        <span class="text-sm text-gray-600">الحد الأقصى: ${criterion.max}</span>
                    </div>
                    <input 
                        type="range" 
                        min="0" 
                        max="${criterion.max}" 
                        value="0" 
                        class="w-full h-3 bg-gray-300 rounded-lg appearance-none cursor-pointer slider-${index}"
                        oninput="updateScoreDisplay(${index}, this.value); calculateTotal()"
                    >
                    <div class="flex justify-between mt-2">
                        <span>0</span>
                        <span class="font-black text-lg ${settings.color.split(' ')[2]}">
                            <span id="score${index}">0</span> / ${criterion.max}
                        </span>
                        <span>${criterion.max}</span>
                    </div>
                </div>`;
                
                criteriaList.innerHTML += criterionHTML;
            });
            
            document.getElementById('maxScoreInfo').textContent = `الدرجة القصوى: ${maxTotal}`;
            document.getElementById('criteriaSection').classList.remove('hidden');
            document.getElementById('criteriaSection').dataset.student = studentName;
            
            window.scrollTo({ top: document.getElementById('criteriaSection').offsetTop, behavior: 'smooth' });
        }

        function updateScoreDisplay(index, value) {
            const scoreElement = document.getElementById(`score${index}`);
            if (scoreElement) {
                scoreElement.textContent = value;
            }
        }

        function calculateTotal() {
            const settings = roleSettings[activeRole];
            const customForRole = customCriteria.filter(c => c.role === activeRole);
            const allCriteria = [...settings.criteria, ...customForRole];
            let total = 0;
            
            allCriteria.forEach((criterion, index) => {
                const slider = document.querySelector(`.slider-${index}`);
                if (slider) total += parseInt(slider.value) || 0;
            });
            
            document.getElementById('totalScore').textContent = total;
        }

        function saveResults() {
            const studentName = document.getElementById('criteriaSection').dataset.student;
            const groupId = document.getElementById('groupSelect').value;
            const evalDate = document.getElementById('evalDate').value;
            const examinerName = document.getElementById('examinerName').value;
            const examinerSignature = document.getElementById('examinerSignature').value;
            
            if (!studentName || !groupId || !activeRole) {
                alert('يرجى تعبئة جميع الحقول!');
                return;
            }

            if (!examinerName) {
                alert('يرجى إدخال اسم الناقش أولاً!');
                return;
            }

            // حفظ اسم الناقش في القائمة
            if (!examiners.includes(examinerName)) {
                examiners.push(examinerName);
                localStorage.setItem('examiners_list', JSON.stringify(examiners));
                updateExaminersFilter();
            }

            const settings = roleSettings[activeRole];
            const customForRole = customCriteria.filter(c => c.role === activeRole);
            const allCriteria = [...settings.criteria, ...customForRole];
            
            let scores = [];
            let total = 0;
            
            allCriteria.forEach((criterion, index) => {
                const slider = document.querySelector(`.slider-${index}`);
                const score = slider ? parseInt(slider.value) : 0;
                scores.push({
                    criterion: criterion.name,
                    score: score,
                    max: criterion.max,
                    isCustom: !!criterion.role
                });
                total += score;
            });

            const result = {
                id: Date.now(),
                student: studentName,
                group: groupId,
                role: activeRole,
                roleName: settings.title,
                examiner: examinerName,
                signature: examinerSignature,
                scores: scores,
                total: total,
                grade: calculateGrade(total),
                date: evalDate,
                timestamp: new Date().toISOString()
            };

            resultsDB.push(result);
            localStorage.setItem('grad_sys_results', JSON.stringify(resultsDB));
            
            alert(`✅ تم حفظ تقييم الطالب ${studentName} بنجاح!\nالدرجة: ${total} - التقدير: ${calculateGrade(total)}`);
            resetForm();
            updateStats();
            renderAdminTable();
            renderStudentsCards();
        }

        function calculateGrade(score) {
            if (score >= 90) return 'ممتاز';
            if (score >= 80) return 'جيد جداً';
            if (score >= 70) return 'جيد';
            if (score >= 60) return 'مقبول';
            return 'راسب';
        }

        function resetForm() {
            const settings = roleSettings[activeRole];
            const customForRole = customCriteria.filter(c => c.role === activeRole);
            const allCriteria = [...settings.criteria, ...customForRole];
            
            allCriteria.forEach((criterion, index) => {
                const slider = document.querySelector(`.slider-${index}`);
                if (slider) {
                    slider.value = 0;
                    updateScoreDisplay(index, 0);
                }
            });
            calculateTotal();
        }

        function switchSection(section) {
            document.querySelectorAll('section').forEach(sec => sec.classList.add('hidden'));
            
            if (section === 'home') {
                document.getElementById('roleSection').classList.remove('hidden');
                document.getElementById('evaluationSection').classList.add('hidden');
                document.getElementById('adminSection').classList.add('hidden');
            } else if (section === 'evaluation') {
                document.getElementById('roleSection').classList.remove('hidden');
            } else if (section === 'admin') {
                document.getElementById('adminSection').classList.remove('hidden');
                renderAdminTable();
                updateStats();
            }
        }

        function renderAdminTable() {
            const tableBody = document.getElementById('resultsTable');
            tableBody.innerHTML = '';
            
            if (resultsDB.length === 0) {
                tableBody.innerHTML = '<tr><td colspan="9" class="py-8 text-center text-gray-500">لا توجد نتائج مسجلة بعد</td></tr>';
                return;
            }
            
            resultsDB.forEach(result => {
                const group = mainDB.find(g => g.id === result.group) || {};
                const gradeClass = result.total >= 90 ? 'text-green-600' : 
                                 result.total >= 70 ? 'text-blue-600' : 
                                 result.total >= 60 ? 'text-amber-600' : 'text-red-600';
                
                const row = document.createElement('tr');
                row.className = 'hover:bg-gray-50';
                row.innerHTML = `
                    <td class="py-3 px-4 border font-bold">${result.student}</td>
                    <td class="py-3 px-4 border">${result.group}</td>
                    <td class="py-3 px-4 border">${group.project || 'غير معروف'}</td>
                    <td class="py-3 px-4 border">
                        <span class="px-3 py-1 rounded-full text-white ${result.role === 'supervisor' ? 'bg-indigo-600' : 'bg-emerald-600'} text-sm">
                            ${result.roleName}
                        </span>
                    </td>
                    <td class="py-3 px-4 border">${result.examiner || 'غير معروف'}</td>
                    <td class="py-3 px-4 border font-black ${gradeClass}">
                        ${result.total}
                    </td>
                    <td class="py-3 px-4 border">
                        <span class="px-2 py-1 bg-gray-100 text-gray-800 text-sm rounded">${result.grade}</span>
                    </td>
                    <td class="py-3 px-4 border">${result.date}</td>
                    <td class="py-3 px-4 border">
                        <button onclick="shareResult(${result.id})" class="px-3 py-1 bg-green-100 text-green-700 hover:bg-green-200 rounded-lg text-sm ml-2">
                            <i class="fab fa-whatsapp"></i>
                        </button>
                        <button onclick="deleteResult(${result.id})" class="px-3 py-1 bg-red-100 text-red-700 hover:bg-red-200 rounded-lg text-sm">
                            <i class="fas fa-trash ml-1"></i>حذف
                        </button>
                    </td>
                `;
                tableBody.appendChild(row);
            });
        }

        function updateExaminersFilter() {
            const filter = document.getElementById('filterExaminer');
            filter.innerHTML = '<option value="">جميع الناقشين</option>';
            
            examiners.forEach(examiner => {
                const option = document.createElement('option');
                option.value = examiner;
                option.textContent = examiner;
                filter.appendChild(option);
            });
        }

        function filterAdmin() {
            const roleFilter = document.getElementById('filterRole').value;
            const groupFilter = document.getElementById('filterGroup').value;
            const searchTerm = document.getElementById('searchStudent').value.toLowerCase();
            const examinerFilter = document.getElementById('filterExaminer').value;
            
            const filtered = resultsDB.filter(result => {
                const matchesRole = !roleFilter || result.role === roleFilter;
                const matchesGroup = !groupFilter || result.group === groupFilter;
                const matchesSearch = !searchTerm || result.student.toLowerCase().includes(searchTerm);
                const matchesExaminer = !examinerFilter || result.examiner === examinerFilter;
                return matchesRole && matchesGroup && matchesSearch && matchesExaminer;
            });
            
            const tableBody = document.getElementById('resultsTable');
            tableBody.innerHTML = '';
            
            filtered.forEach(result => {
                const group = mainDB.find(g => g.id === result.group) || {};
                const gradeClass = result.total >= 90 ? 'text-green-600' : 
                                 result.total >= 70 ? 'text-blue-600' : 
                                 result.total >= 60 ? 'text-amber-600' : 'text-red-600';
                
                const row = document.createElement('tr');
                row.className = 'hover:bg-gray-50';
                row.innerHTML = `
                    <td class="py-3 px-4 border font-bold">${result.student}</td>
                    <td class="py-3 px-4 border">${result.group}</td>
                    <td class="py-3 px-4 border">${group.project || 'غير معروف'}</td>
                    <td class="py-3 px-4 border">
                        <span class="px-3 py-1 rounded-full text-white ${result.role === 'supervisor' ? 'bg-indigo-600' : 'bg-emerald-600'} text-sm">
                            ${result.roleName}
                        </span>
                    </td>
                    <td class="py-3 px-4 border">${result.examiner || 'غير معروف'}</td>
                    <td class="py-3 px-4 border font-black ${gradeClass}">
                        ${result.total}
                    </td>
                    <td class="py-3 px-4 border">
                        <span class="px-2 py-1 bg-gray-100 text-gray-800 text-sm rounded">${result.grade}</span>
                    </td>
                    <td class="py-3 px-4 border">${result.date}</td>
                    <td class="py-3 px-4 border">
                        <button onclick="shareResult(${result.id})" class="px-3 py-1 bg-green-100 text-green-700 hover:bg-green-200 rounded-lg text-sm ml-2">
                            <i class="fab fa-whatsapp"></i>
                        </button>
                        <button onclick="deleteResult(${result.id})" class="px-3 py-1 bg-red-100 text-red-700 hover:bg-red-200 rounded-lg text-sm">
                            <i class="fas fa-trash ml-1"></i>حذف
                        </button>
                    </td>
                `;
                tableBody.appendChild(row);
            });
        }

        function updateStats() {
            document.getElementById('totalGroups').textContent = mainDB.length;
            
            const totalStudents = mainDB.reduce((sum, group) => sum + group.students.length, 0);
            document.getElementById('totalStudents').textContent = totalStudents;
            
            document.getElementById('evaluatedCount').textContent = resultsDB.length;
            
            const avg = resultsDB.length > 0 
                ? (resultsDB.reduce((sum, result) => sum + result.total, 0) / resultsDB.length).toFixed(1)
                : '0.0';
            document.getElementById('avgScore').textContent = avg;
        }

        function deleteResult(id) {
            if (confirm('هل أنت متأكد من حذف هذا التقييم؟')) {
                resultsDB = resultsDB.filter(result => result.id !== id);
                localStorage.setItem('grad_sys_results', JSON.stringify(resultsDB));
                renderAdminTable();
                updateStats();
            }
        }

        // ✅ 1. مشاركة واتساب
        function shareResults() {
            if (resultsDB.length === 0) {
                alert('لا توجد نتائج للمشاركة!');
                return;
            }
            
            let message = "📊 *نتائج تقييم المناقشات*\n\n";
            resultsDB.slice(-10).forEach(result => {
                message += `👨‍🎓 ${result.student}\n`;
                message += `🏆 المجموع: ${result.total}\n`;
                message += `📋 التقدير: ${result.grade}\n`;
                message += `👨‍🏫 الناقش: ${result.examiner}\n`;
                message += `📅 ${result.date}\n`;
                message += "───────────────\n";
            });
            
            const encodedMessage = encodeURIComponent(message);
            const whatsappURL = `https://wa.me/?text=${encodedMessage}`;
            window.open(whatsappURL, '_blank');
        }

        function shareResult(resultId) {
            const result = resultsDB.find(r => r.id === resultId);
            if (!result) return;
            
            let message = `📋 *نتيجة التقييم*\n\n`;
            message += `👨‍🎓 *الطالب:* ${result.student}\n`;
            message += `🏷️ *المجموعة:* ${result.group}\n`;
            message += `📊 *الدرجة الكلية:* ${result.total}/100\n`;
            message += `🏆 *التقدير:* ${result.grade}\n`;
            message += `👨‍🏫 *الناقش:* ${result.examiner}\n`;
            message += `📅 *التاريخ:* ${result.date}\n\n`;
            
            // تفاصيل الدرجات
            message += `*تفاصيل الدرجات:*\n`;
            result.scores.forEach(score => {
                message += `• ${score.criterion}: ${score.score}/${score.max}\n`;
            });
            
            const encodedMessage = encodeURIComponent(message);
            const whatsappURL = `https://wa.me/?text=${encodedMessage}`;
            window.open(whatsappURL, '_blank');
        }

        // ✅ 2. طباعة PDF
        function printPDF() {
            if (resultsDB.length === 0) {
                alert('لا توجد نتائج لطباعة PDF!');
                return;
            }

            const { jsPDF } = window.jspdf;
            const doc = new jsPDF('p', 'pt', 'a4');
            
            // عنوان التقرير
            doc.setFontSize(20);
            doc.setTextColor(40);
            doc.text('تقرير نتائج تقييم المناقشات', 300, 40, { align: 'center' });
            
            doc.setFontSize(12);
            doc.text(`تاريخ التقرير: ${new Date().toLocaleDateString('ar-SA')}`, 500, 80, { align: 'right' });
            
            // بيانات الإحصائيات
            doc.setFontSize(11);
            doc.text(`إجمالي المجموعات: ${mainDB.length}`, 500, 110, { align: 'right' });
            doc.text(`إجمالي الطلاب: ${mainDB.reduce((sum, group) => sum + group.students.length, 0)}`, 500, 130, { align: 'right' });
            doc.text(`عدد التقييمات: ${resultsDB.length}`, 500, 150, { align: 'right' });
            
            // جدول النتائج
            const headers = [['اسم الطالب', 'المجموعة', 'الدرجة', 'التقدير', 'الناقش', 'التاريخ']];
            const data = resultsDB.map(result => [
                result.student,
                result.group,
                result.total.toString(),
                result.grade,
                result.examiner || 'غير معروف',
                result.date
            ]);
            
            doc.autoTable({
                head: headers,
                body: data,
                startY: 180,
                theme: 'grid',
                headStyles: { fillColor: [59, 130, 246], textColor: 255, halign: 'center' },
                columnStyles: {
                    0: { halign: 'right' },
                    1: { halign: 'center' },
                    2: { halign: 'center' },
                    3: { halign: 'center' },
                    4: { halign: 'right' },
                    5: { halign: 'center' }
                },
                margin: { right: 40 }
            });
            
            // توقيع
            doc.setFontSize(10);
            doc.text('توقيع المدير: ___________________', 100, doc.internal.pageSize.height - 40);
            doc.text('توقيع الناقش: ___________________', 400, doc.internal.pageSize.height - 40);
            
            doc.save(`نتائج_التقييم_${new Date().toISOString().split('T')[0]}.pdf`);
        }

        // ✅ 3. تصدير لإكسل
        function exportToExcel() {
            if (resultsDB.length === 0) {
                alert('لا توجد بيانات للتصدير!');
                return;
            }
            
            const data = resultsDB.map(result => ({
                'اسم الطالب': result.student,
                'رقم المجموعة': result.group,
                'نوع التقييم': result.roleName,
                'الناقش': result.examiner,
                'الدرجة الكلية': result.total,
                'التقدير': result.grade,
                'التاريخ': result.date
            }));
            
            const ws = XLSX.utils.json_to_sheet(data);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, "النتائج");
            
            // إضافة شيت البيانات الأصلية من Google Sheets إن وجدت
            if (mainDB.length > 0) {
                const originalData = [];
                mainDB.forEach(group => {
                    group.students.forEach(student => {
                        originalData.push({
                            'اسم الطالب': student,
                            'رقم المجموعة': group.id,
                            'اسم المشروع': group.project,
                            'رقم القاعة': group.room,
                            'اسم المشرف': group.supervisor
                        });
                    });
                });
                
                const ws2 = XLSX.utils.json_to_sheet(originalData);
                XLSX.utils.book_append_sheet(wb, ws2, "البيانات الأصلية");
            }
            
            XLSX.writeFile(wb, `نتائج_التقييم_${new Date().toISOString().split('T')[0]}.xlsx`);
        }

        // ✅ 4. تصدير إكسل كامل مع جميع البيانات
        function exportToExcelFull() {
            if (resultsDB.length === 0) {
                alert('لا توجد بيانات للتصدير!');
                return;
            }
            
            const wb = XLSX.utils.book_new();
            
            // شيت النتائج المفصلة
            const detailedData = [];
            resultsDB.forEach(result => {
                result.scores.forEach(score => {
                    detailedData.push({
                        'اسم الطالب': result.student,
                        'المجموعة': result.group,
                        'نوع التقييم': result.roleName,
                        'الناقش': result.examiner,
                        'المعيار': score.criterion,
                        'الدرجة': score.score,
                        'الحد الأقصى': score.max,
                        'المجموع': result.total,
                        'التقدير': result.grade,
                        'التاريخ': result.date
                    });
                });
            });
            
            const ws1 = XLSX.utils.json_to_sheet(detailedData);
            XLSX.utils.book_append_sheet(wb, ws1, "النتائج المفصلة");
            
            // شيت الملخص
            const summaryData = resultsDB.map(result => ({
                'اسم الطالب': result.student,
                'المجموعة': result.group,
                'نوع التقييم': result.roleName,
                'الناقش': result.examiner,
                'الدرجة الكلية': result.total,
                'التقدير': result.grade,
                'التاريخ': result.date
            }));
            
            const ws2 = XLSX.utils.json_to_sheet(summaryData);
            XLSX.utils.book_append_sheet(wb, ws2, "ملخص النتائج");
            
            // شيت الإحصائيات
            const statsData = [{
                'المجموعات الكلية': mainDB.length,
                'إجمالي الطلاب': mainDB.reduce((sum, group) => sum + group.students.length, 0),
                'عدد التقييمات': resultsDB.length,
                'متوسط الدرجات': resultsDB.length > 0 ? 
                    (resultsDB.reduce((sum, result) => sum + result.total, 0) / resultsDB.length).toFixed(1) : 0,
                'أعلى درجة': resultsDB.length > 0 ? Math.max(...resultsDB.map(r => r.total)) : 0,
                'أقل درجة': resultsDB.length > 0 ? Math.min(...resultsDB.map(r => r.total)) : 0
            }];
            
            const ws3 = XLSX.utils.json_to_sheet(statsData);
            XLSX.utils.book_append_sheet(wb, ws3, "الإحصائيات");
            
            // شيت بيانات الطلاب الأصلية
            if (mainDB.length > 0) {
                const originalData = [];
                mainDB.forEach(group => {
                    group.students.forEach(student => {
                        originalData.push({
                            'اسم الطالب': student,
                            'رقم المجموعة': group.id,
                            'اسم المشروع': group.project,
                            'رقم القاعة': group.room,
                            'اسم المشرف': group.supervisor
                        });
                    });
                });
                
                const ws4 = XLSX.utils.json_to_sheet(originalData);
                XLSX.utils.book_append_sheet(wb, ws4, "بيانات الطلاب");
            }
            
            XLSX.writeFile(wb, `تقرير_مفصل_${new Date().toISOString().split('T')[0]}.xlsx`);
        }

        function exportToCSV() {
            if (resultsDB.length === 0) {
                alert('لا توجد بيانات للتصدير!');
                return;
            }
            
            let csv = 'اسم الطالب,المجموعة,نوع التقييم,الناقش,الدرجة الكلية,التقدير,التاريخ\n';
            
            resultsDB.forEach(result => {
                csv += `"${result.student}","${result.group}","${result.roleName}","${result.examiner || ''}",${result.total},"${result.grade}","${result.date}"\n`;
            });
            
            const blob = new Blob(['\uFEFF' + csv], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(blob);
            link.download = `نتائج_التقييم_${new Date().toISOString().split('T')[0]}.csv`;
            link.click();
        }

        // ✅ 5. حقوق النشر
        function showCopyright() {
            document.getElementById('copyrightModal').classList.remove('hidden');
            document.getElementById('copyrightModal').classList.add('flex');
        }

        function closeCopyright() {
            document.getElementById('copyrightModal').classList.add('hidden');
            document.getElementById('copyrightModal').classList.remove('flex');
        }

        // ✅ 6. التقييم السريع
        function quickEvaluation() {
            const modal = document.getElementById('quickEvalModal');
            modal.classList.remove('hidden');
            modal.classList.add('flex');
            
            // تحديث قائمة الطلاب للمجموعة المحددة
            const groupSelect = document.getElementById('quickGroupSelect');
            groupSelect.onchange = function() {
                const groupId = this.value;
                if (!groupId) return;
                
                const group = mainDB.find(g => g.id === groupId);
                if (!group) return;
                
                let studentsHTML = '';
                group.students.forEach(student => {
                    studentsHTML += `
                    <div class="flex items-center justify-between p-3 bg-gray-50 rounded-lg">
                        <span class="font-medium">${student}</span>
                        <button onclick="editIndividualScore('${student}', '${groupId}')" class="px-3 py-1 bg-blue-100 text-blue-700 hover:bg-blue-200 rounded text-sm">
                            تعديل
                        </button>
                    </div>`;
                });
                
                document.getElementById('quickStudentsList').innerHTML = studentsHTML;
            };
            
            groupSelect.onchange();
        }

        function closeQuickEval() {
            document.getElementById('quickEvalModal').classList.add('hidden');
            document.getElementById('quickEvalModal').classList.remove('flex');
        }

        function applyCommonScore() {
            const groupId = document.getElementById('quickGroupSelect').value;
            const commonScore = parseInt(document.getElementById('commonScore').value);
            const role = document.getElementById('quickRole').value;
            const examinerName = document.getElementById('examinerName').value;
            const evalDate = new Date().toISOString().split('T')[0];
            
            if (!groupId) {
                alert('يرجى اختيار مجموعة!');
                return;
            }
            
            if (!examinerName) {
                alert('يرجى إدخال اسم الناقش أولاً!');
                return;
            }
            
            const group = mainDB.find(g => g.id === groupId);
            if (!group) return;
            
            const settings = roleSettings[role];
            
            group.students.forEach(student => {
                const scores = settings.criteria.map(criterion => ({
                    criterion: criterion.name,
                    score: Math.round((commonScore / 100) * criterion.max),
                    max: criterion.max
                }));
                
                const total = scores.reduce((sum, s) => sum + s.score, 0);
                
                const result = {
                    id: Date.now() + Math.random(),
                    student: student,
                    group: groupId,
                    role: role,
                    roleName: settings.title,
                    examiner: examinerName,
                    signature: document.getElementById('examinerSignature').value,
                    scores: scores,
                    total: total,
                    grade: calculateGrade(total),
                    date: evalDate,
                    timestamp: new Date().toISOString()
                };
                
                resultsDB.push(result);
            });
            
            localStorage.setItem('grad_sys_results', JSON.stringify(resultsDB));
            alert(`✅ تم تطبيق الدرجة ${commonScore} على ${group.students.length} طالب!`);
            closeQuickEval();
            renderAdminTable();
            updateStats();
        }

        function editIndividualScore(student, groupId) {
            document.getElementById('studentNameTitle').textContent = `تقييم الطالب: ${student}`;
            document.getElementById('studentGroupInfo').textContent = `المجموعة: ${groupId}`;
            document.getElementById('individualScore').value = '';
            document.getElementById('studentNotes').value = '';
            
            const modal = document.getElementById('studentEvalModal');
            modal.dataset.student = student;
            modal.dataset.group = groupId;
            
            modal.classList.remove('hidden');
            modal.classList.add('flex');
        }

        function quickStudentEval(student, groupId) {
            editIndividualScore(student, groupId);
        }

        function closeStudentEval() {
            document.getElementById('studentEvalModal').classList.add('hidden');
            document.getElementById('studentEvalModal').classList.remove('flex');
        }

        function saveIndividualScore() {
            const student = document.getElementById('studentEvalModal').dataset.student;
            const groupId = document.getElementById('studentEvalModal').dataset.group;
            const score = parseInt(document.getElementById('individualScore').value);
            const notes = document.getElementById('studentNotes').value;
            const role = document.getElementById('quickRole').value;
            const examinerName = document.getElementById('examinerName').value;
            const evalDate = new Date().toISOString().split('T')[0];
            
            if (!score || score < 0 || score > 100) {
                alert('يرجى إدخال درجة صحيحة بين 0 و 100!');
                return;
            }
            
            if (!examinerName) {
                alert('يرجى إدخال اسم الناقش أولاً!');
                return;
            }
            
            const settings = roleSettings[role];
            const scores = settings.criteria.map(criterion => ({
                criterion: criterion.name,
                score: Math.round((score / 100) * criterion.max),
                max: criterion.max
            }));
            
            const result = {
                id: Date.now() + Math.random(),
                student: student,
                group: groupId,
                role: role,
                roleName: settings.title,
                examiner: examinerName,
                signature: document.getElementById('examinerSignature').value,
                scores: scores,
                total: score,
                grade: calculateGrade(score),
                notes: notes,
                date: evalDate,
                timestamp: new Date().toISOString()
            };
            
            resultsDB.push(result);
            localStorage.setItem('grad_sys_results', JSON.stringify(resultsDB));
            
            alert(`✅ تم حفظ تقييم الطالب ${student} بنجاح!\nالدرجة: ${score}`);
            closeStudentEval();
            renderAdminTable();
            updateStats();
        }

        function clearSignature() {
            document.getElementById('examinerSignature').value = '';
        }

        function clearAllData() {
            if (confirm('⚠️ هل أنت متأكد من حذف جميع البيانات؟ هذا الإجراء لا يمكن التراجع عنه!')) {
                localStorage.removeItem('grad_sys_results');
                localStorage.removeItem('custom_criteria');
                localStorage.removeItem('examiners_list');
                resultsDB = [];
                customCriteria = [];
                examiners = [];
                renderAdminTable();
                updateStats();
                updateExaminersFilter();
                alert('تم حذف جميع البيانات بنجاح.');
            }
        }

        window.onload = () => {
            document.getElementById('evalDate').valueAsDate = new Date();
            
            const storedDB = localStorage.getItem('grad_sys_db');
            const storedResults = localStorage.getItem('grad_sys_results');
            const storedCriteria = localStorage.getItem('custom_criteria');
            const storedExaminers = localStorage.getItem('examiners_list');
            
            if (storedDB) {
                mainDB = JSON.parse(storedDB);
                populateGroupSelect();
            }
            
            if (storedResults) {
                resultsDB = JSON.parse(storedResults);
            }
            
            if (storedCriteria) {
                customCriteria = JSON.parse(storedCriteria);
            }
            
            if (storedExaminers) {
                examiners = JSON.parse(storedExaminers);
                updateExaminersFilter();
            }
            
            updateStats();
            switchSection('home');
        };
    </script>

    <style>
        /* تخصيص الـ Slider */
        input[type="range"] {
            -webkit-appearance: none;
            height: 8px;
            background: #d1d5db;
            border-radius: 5px;
        }
        
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 22px;
            height: 22px;
            border-radius: 50%;
            background: #4f46e5;
            cursor: pointer;
            border: 3px solid white;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
        }
        
        /* تحسينات للجداول */
        table {
            border-collapse: separate;
            border-spacing: 0;
        }
        
        th, td {
            border-right: 1px solid #e5e7eb;
            border-bottom: 1px solid #e5e7eb;
        }
        
        th:first-child, td:first-child {
            border-right: none;
        }
        
        /* تأثيرات للبطاقات */
        .hover-lift:hover {
            transform: translateY(-2px);
            transition: transform 0.2s ease;
        }
        
        /* تدرج ألوان للنتائج */
        .grade-excellent { background: linear-gradient(135deg, #10b981, #059669); }
        .grade-very-good { background: linear-gradient(135deg, #3b82f6, #1d4ed8); }
        .grade-good { background: linear-gradient(135deg, #f59e0b, #d97706); }
        .grade-acceptable { background: linear-gradient(135deg, #f97316, #ea580c); }
        .grade-fail { background: linear-gradient(135deg, #ef4444, #dc2626); }
    </style>
</body>
</html>
