<!DOCTYPE html>
<html dir="rtl" lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام إدارة تقييم المناقشات</title>
    
    <!-- مكتبات CSS -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- مكتبات JavaScript -->
    <script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.0/papaparse.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js"></script>
    
    <style>
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
        
        .scrollbar-thin {
            scrollbar-width: thin;
            scrollbar-color: #cbd5e1 #f1f5f9;
        }
        
        .scrollbar-thin::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        
        .scrollbar-thin::-webkit-scrollbar-track {
            background: #f1f5f9;
            border-radius: 4px;
        }
        
        .scrollbar-thin::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        
        .hover-lift:hover {
            transform: translateY(-2px);
            transition: transform 0.2s ease;
        }
        
        /* تأثيرات دخول وخروج */
        .modal-enter {
            animation: modalEnter 0.3s ease-out;
        }
        
        @keyframes modalEnter {
            from {
                opacity: 0;
                transform: scale(0.9);
            }
            to {
                opacity: 1;
                transform: scale(1);
            }
        }
        
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            10%, 30%, 50%, 70%, 90% { transform: translateX(-5px); }
            20%, 40%, 60%, 80% { transform: translateX(5px); }
        }
        .animate-shake {
            animation: shake 0.5s ease-in-out;
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen">
    
<!-- طبقة نظام الحماية -->
<div id="securityLayer" class="fixed inset-0 bg-gradient-to-br from-indigo-900 to-purple-900 flex items-center justify-center z-50">
    <div class="bg-white rounded-2xl shadow-2xl p-8 max-w-md w-full mx-4 modal-enter">
        <div class="text-center mb-8">
            <div class="h-20 w-20 rounded-full bg-indigo-100 flex items-center justify-center mx-auto mb-4">
                <i class="fas fa-lock text-3xl text-indigo-600"></i>
            </div>
            <h2 class="text-2xl font-black text-gray-800">نظام الحماية</h2>
            <p class="text-gray-600 mt-2">أدخل كلمة المرور للوصول للنظام</p>
        </div>
        
        <div class="space-y-4">
            <div>
                <label class="block font-bold text-gray-700 mb-2">كلمة المرور:</label>
                <div class="relative">
                    <input type="password" id="passwordInput" class="w-full p-4 border-2 border-gray-300 rounded-lg focus:border-indigo-500 focus:ring-2 focus:ring-indigo-200 outline-none text-center text-lg tracking-widest" placeholder="••••••••">
                    <button onclick="togglePassword()" class="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-500">
                        <i id="passwordEye" class="far fa-eye"></i>
                    </button>
                </div>
                <p class="text-xs text-gray-500 mt-2">كلمة المرور: 1234</p>
            </div>
            
            <div class="mt-6">
                <button onclick="login()" class="w-full py-3 bg-gradient-to-r from-indigo-600 to-indigo-800 hover:from-indigo-700 hover:to-indigo-900 text-white font-black rounded-lg transition shadow-md hover:shadow-lg">
                    <i class="fas fa-sign-in-alt ml-2"></i>دخول للنظام
                </button>
            </div>
            
            <div class="mt-4 text-center">
                <button onclick="forgotPassword()" class="text-sm text-indigo-600 hover:text-indigo-800">
                    <i class="fas fa-key ml-1"></i>نسيت كلمة المرور؟
                </button>
            </div>
        </div>
    </div>
</div>

<!-- نافذة تغيير كلمة المرور -->
<div id="changePasswordModal" class="fixed inset-0 bg-black/50 hidden items-center justify-center z-50 p-4">
    <div class="bg-white rounded-2xl shadow-2xl max-w-md w-full p-6 modal-enter">
        <div class="flex justify-between items-center mb-6">
            <h3 class="text-xl font-black text-gray-800"><i class="fas fa-key ml-2"></i>تغيير كلمة المرور</h3>
            <button onclick="closeChangePassword()" class="text-gray-500 hover:text-gray-800">
                <i class="fas fa-times"></i>
            </button>
        </div>
        
        <div class="space-y-4">
            <div>
                <label class="block font-bold text-gray-700 mb-2">كلمة المرور الحالية:</label>
                <input type="password" id="currentPassword" class="w-full p-3 border-2 border-gray-300 rounded-lg">
            </div>
            
            <div>
                <label class="block font-bold text-gray-700 mb-2">كلمة المرور الجديدة:</label>
                <input type="password" id="newPassword" class="w-full p-3 border-2 border-gray-300 rounded-lg">
            </div>
            
            <div>
                <label class="block font-bold text-gray-700 mb-2">تأكيد كلمة المرور:</label>
                <input type="password" id="confirmPassword" class="w-full p-3 border-2 border-gray-300 rounded-lg">
            </div>
            
            <div class="mt-6 flex gap-4">
                <button onclick="saveNewPassword()" class="flex-1 py-3 bg-gradient-to-r from-green-600 to-green-800 text-white font-black rounded-lg">
                    <i class="fas fa-save ml-2"></i>حفظ
                </button>
                <button onclick="closeChangePassword()" class="px-6 py-3 bg-gray-200 hover:bg-gray-300 rounded-lg">
                    إلغاء
                </button>
            </div>
        </div>
    </div>
</div>

<div id="loading" class="fixed inset-0 bg-white/90 flex items-center justify-center z-40 hidden">
    <div class="text-center">
        <div class="block animate-spin rounded-full h-16 w-16 border-4 border-indigo-600 border-t-transparent"></div>
        <p class="mt-4 font-black text-indigo-900 text-lg">جاري مزامنة البيانات...</p>
    </div>
</div>

<div id="app" class="max-w-7xl mx-auto space-y-6 p-4 hidden">
    <!-- رأس الصفحة -->
    <header class="text-center py-6 bg-gradient-to-r from-indigo-900 to-indigo-700 rounded-2xl shadow-xl text-white relative">
        <div class="absolute top-4 left-4 text-sm opacity-80 flex gap-2">
            <button onclick="showCopyright()" class="px-3 py-1 bg-white/20 hover:bg-white/30 rounded-lg transition">
                <i class="fas fa-copyright ml-1"></i>حقوق النشر
            </button>
            <div id="userBadge" class="px-3 py-1 bg-emerald-500 rounded-lg">
                <i class="fas fa-user-shield ml-1"></i>
                <span id="userRole">مدير النظام</span>
            </div>
        </div>
        <h1 class="text-3xl md:text-4xl font-black mb-2">نظام إدارة تقييم المناقشات</h1>
        <p class="text-lg opacity-90">الإصدار النهائي - نظام متكامل للتقييم الإلكتروني</p>
        <div class="mt-4 flex flex-wrap justify-center gap-4">
            <button onclick="switchSection('home')" class="px-5 py-2 bg-white/20 hover:bg-white/30 rounded-lg transition"><i class="fas fa-home ml-2"></i>الرئيسية</button>
            <button onclick="switchSection('evaluation')" class="px-5 py-2 bg-white/20 hover:bg-white/30 rounded-lg transition"><i class="fas fa-clipboard-check ml-2"></i>التقييم</button>
            <button onclick="switchSection('admin')" class="px-5 py-2 bg-white/20 hover:bg-white/30 rounded-lg transition"><i class="fas fa-chart-bar ml-2"></i>الإحصائيات</button>
            <button onclick="fetchSheetData()" class="px-5 py-2 bg-emerald-500 hover:bg-emerald-600 rounded-lg transition"><i class="fas fa-sync-alt ml-2"></i>مزامنة البيانات</button>
            <button onclick="syncToGoogleSheet()" class="px-5 py-2 bg-orange-500 hover:bg-orange-600 rounded-lg transition"><i class="fas fa-cloud-upload-alt ml-2"></i>حفظ في Google Sheets</button>
            <button onclick="showChangePassword()" class="px-5 py-2 bg-purple-500 hover:bg-purple-600 rounded-lg transition"><i class="fas fa-key ml-2"></i>تغيير كلمة السر</button>
            <button onclick="logout()" class="px-5 py-2 bg-red-500 hover:bg-red-600 rounded-lg transition"><i class="fas fa-sign-out-alt ml-2"></i>تسجيل خروج</button>
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
            <h3 class="text-xl font-black text-gray-800 mb-4"><i class="fas fa-user-edit ml-2"></i>إعدادات الناقش/المشرف</h3>
            <div class="grid md:grid-cols-2 gap-4">
                <div>
                    <label class="block font-bold text-gray-700 mb-2">اسم الناقش/المشرف:</label>
                    <input type="text" id="examinerName" placeholder="اسم الناقش/المشرف ..." class="w-full p-3 border-2 border-gray-300 rounded-lg focus:border-indigo-500 focus:ring-2 focus:ring-indigo-200 outline-none">
                </div>
                <div>
                    <label class="block font-bold text-gray-700 mb-2">التوقيع:</label>
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
                <button onclick="saveAndSync()" class="flex-1 py-3 bg-gradient-to-r from-orange-600 to-orange-800 hover:from-orange-700 hover:to-orange-900 text-white font-black rounded-lg transition shadow-md hover:shadow-lg">
                    <i class="fas fa-cloud-upload-alt ml-2"></i>حفظ ومزامنة
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
            <div class="flex flex-wrap gap-2 mt-4">
                <div class="px-3 py-1 bg-indigo-100 text-indigo-800 rounded-full text-sm">
                    <i class="fas fa-users ml-1"></i> <span id="statsGroups">0</span> مجموعة
                </div>
                <div class="px-3 py-1 bg-green-100 text-green-800 rounded-full text-sm">
                    <i class="fas fa-user-graduate ml-1"></i> <span id="statsStudents">0</span> طالب
                </div>
                <div class="px-3 py-1 bg-amber-100 text-amber-800 rounded-full text-sm">
                    <i class="fas fa-clipboard-check ml-1"></i> <span id="statsEvaluations">0</span> تقييم
                </div>
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

        <div class="overflow-x-auto scrollbar-thin">
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
                        <th class="py-3 px-4 border text-right">حفظ في Sheets</th>
                        <th class="py-3 px-4 border text-right">الإجراءات</th>
                    </tr>
                </thead>
                <tbody id="resultsTable">
                    <!-- سيتم ملء الجدول ديناميكيًا -->
                </tbody>
            </table>
        </div>

        <div class="mt-6 flex gap-4 flex-wrap">
            <button onclick="syncAllToGoogleSheet()" class="px-6 py-3 bg-gradient-to-r from-orange-500 to-orange-700 hover:from-orange-600 hover:to-orange-800 text-white font-bold rounded-lg transition">
                <i class="fas fa-cloud-upload-alt ml-2"></i>حفظ البيانات على Google Sheets
            </button>
            <button onclick="exportDirectFromSheet()" class="px-6 py-3 bg-gradient-to-r from-purple-500 to-purple-700 hover:from-purple-600 hover:to-purple-800 text-white font-bold rounded-lg transition">
                <i class="fas fa-database ml-2"></i>تصدير مباشر من Google Sheets
            </button>
            <button onclick="openExcelInBrowser()" class="px-6 py-3 bg-gradient-to-r from-cyan-500 to-cyan-700 hover:from-cyan-600 hover:to-cyan-800 text-white font-bold rounded-lg transition">
                <i class="fas fa-eye ml-2"></i>عرض البيانات في المتصفح
            </button>
            <button onclick="shareResults()" class="px-6 py-3 bg-green-600 hover:bg-green-700 text-white rounded-lg">
                <i class="fab fa-whatsapp ml-2"></i>مشاركة واتساب
            </button>
            <button onclick="printPDF()" class="px-6 py-3 bg-red-600 hover:bg-red-700 text-white rounded-lg">
                <i class="fas fa-file-pdf ml-2"></i>طباعة PDF
            </button>
            <button onclick="exportToExcel()" class="px-6 py-3 bg-emerald-600 hover:bg-emerald-700 text-white rounded-lg">
                <i class="fas fa-file-excel ml-2"></i>تصدير إكسل
            </button>
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
        <div class="bg-white rounded-2xl shadow-2xl max-w-md w-full p-6 modal-enter">
            <div class="text-center mb-6">
                <i class="fas fa-copyright text-4xl text-indigo-600 mb-4"></i>
                <h3 class="text-2xl font-black text-gray-800">حقوق النشر والتطوير</h3>
            </div>
            <div class="space-y-4">
                <div class="p-4 bg-gradient-to-r from-indigo-50 to-blue-50 rounded-xl">
                    <p class="font-bold text-indigo-800">المهندس:</p>
                    <p class="text-lg font-black text-gray-800">أنس جعبري</p>
                    <p class="text-gray-600 mt-2">مهندس مساحة</p>
                </div>
                <div class="p-4 bg-gradient-to-r from-emerald-50 to-green-50 rounded-xl">
                    <p class="font-bold text-emerald-800">النسخة:</p>
                    <p class="text-lg font-black text-gray-800">2.0 المتكاملة</p>
                    <p class="text-gray-600 mt-2">نظام إدارة تقييم المناقشات</p>
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
        <div class="bg-white rounded-2xl shadow-2xl max-w-2xl w-full p-6 max-h-[90vh] overflow-y-auto modal-enter">
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
        <div class="bg-white rounded-2xl shadow-2xl max-w-md w-full p-6 modal-enter">
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
            <p>نظام إدارة تقييم المناقشات - المهندس: أنس جعبري</p>
        </div>
    </footer>

</div>

<script>
    // روابط Google Sheets
    const SOURCE_SHEET_CSV_URL = "https://docs.google.com/spreadsheets/d/1Ne4jRjMj75t2zk-w9bCoO2jZlkJkOIxtQAlqNG91p3U/export?format=csv";
    
    // رابط Google Apps Script الخاص بك
    const GAS_WEB_APP_URL = "https://script.google.com/macros/s/AKfycbx4YXKWLtmy0Iygf_RvHfsRxTazsoQpNd7i8mvJ1-TXYnjOGFzzPwjMCfKsV1-wE1pIIQ/exec";
    
    let mainDB = [];
    let activeRole = '';
    let resultsDB = JSON.parse(localStorage.getItem('grad_sys_results')) || [];
    let customCriteria = JSON.parse(localStorage.getItem('custom_criteria')) || [];
    let examiners = JSON.parse(localStorage.getItem('examiners_list')) || [];

    // نظام الحماية المبسط
    const DEFAULT_PASSWORD = "1234";
    let systemPassword = localStorage.getItem('system_password') || DEFAULT_PASSWORD;

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
                { name: 'القدرة على النقاش', max: 30 },
                { name: 'الكتاب المظري والعملي', max: 50 }
            ]
        }
    };

    // نظام الحماية المبسط
    function login() {
        const passwordInput = document.getElementById('passwordInput').value;
        
        if (passwordInput === systemPassword) {
            // إخفاء طبقة الحماية
            document.getElementById('securityLayer').classList.add('hidden');
            document.getElementById('app').classList.remove('hidden');
            
            // حفظ حالة الدخول
            localStorage.setItem('user_logged_in', 'true');
            
            alert('🎉 تم الدخول بنجاح! مرحبًا بك في نظام إدارة تقييم المناقشات.');
        } else {
            alert('كلمة المرور غير صحيحة! حاول مرة أخرى.');
            document.getElementById('passwordInput').value = '';
            document.getElementById('passwordInput').focus();
            
            // تأثير اهتزاز عند خطأ
            const input = document.getElementById('passwordInput');
            input.classList.add('border-red-500');
            input.classList.add('animate-shake');
            setTimeout(() => {
                input.classList.remove('border-red-500');
                input.classList.remove('animate-shake');
            }, 500);
        }
    }

    function togglePassword() {
        const passwordInput = document.getElementById('passwordInput');
        const eyeIcon = document.getElementById('passwordEye');
        
        if (passwordInput.type === 'password') {
            passwordInput.type = 'text';
            eyeIcon.classList.remove('fa-eye');
            eyeIcon.classList.add('fa-eye-slash');
        } else {
            passwordInput.type = 'password';
            eyeIcon.classList.remove('fa-eye-slash');
            eyeIcon.classList.add('fa-eye');
        }
    }

    function forgotPassword() {
        alert(`كلمة المرور الافتراضية هي: ${DEFAULT_PASSWORD}\nيمكنك تغييرها من خلال خيار "تغيير كلمة السر" بعد الدخول للنظام.`);
    }

    function showChangePassword() {
        document.getElementById('changePasswordModal').classList.remove('hidden');
        document.getElementById('changePasswordModal').classList.add('flex');
    }

    function closeChangePassword() {
        document.getElementById('changePasswordModal').classList.add('hidden');
        document.getElementById('changePasswordModal').classList.remove('flex');
    }

    function saveNewPassword() {
        const currentPass = document.getElementById('currentPassword').value;
        const newPass = document.getElementById('newPassword').value;
        const confirmPass = document.getElementById('confirmPassword').value;
        
        if (currentPass !== systemPassword) {
            alert('كلمة المرور الحالية غير صحيحة!');
            return;
        }
        
        if (newPass !== confirmPass) {
            alert('كلمة المرور الجديدة غير متطابقة!');
            return;
        }
        
        if (newPass.length < 6) {
            alert('كلمة المرور يجب أن تكون 6 أحرف على الأقل!');
            return;
        }
        
        systemPassword = newPass;
        localStorage.setItem('system_password', newPass);
        
        alert('✅ تم تغيير كلمة المرور بنجاح!');
        closeChangePassword();
    }

    function logout() {
        if (confirm('هل تريد تسجيل الخروج من النظام؟')) {
            localStorage.removeItem('user_logged_in');
            
            document.getElementById('app').classList.add('hidden');
            document.getElementById('securityLayer').classList.remove('hidden');
            document.getElementById('passwordInput').value = '';
            document.getElementById('passwordInput').type = 'password';
            document.getElementById('passwordEye').classList.remove('fa-eye-slash');
            document.getElementById('passwordEye').classList.add('fa-eye');
        }
    }

    // دالة إضافة + و - للتقييم
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
                
                <div class="flex items-center gap-3 mb-3">
                    <button onclick="adjustScore(${index}, -1)" class="w-10 h-10 bg-red-100 text-red-700 rounded-full hover:bg-red-200 transition flex items-center justify-center">
                        <i class="fas fa-minus"></i>
                    </button>
                    
                    <div class="flex-1">
                        <input 
                            type="range" 
                            min="0" 
                            max="${criterion.max}" 
                            value="0" 
                            class="w-full h-3 bg-gray-300 rounded-lg appearance-none cursor-pointer slider-${index}"
                            oninput="updateScoreDisplay(${index}, this.value); calculateTotal()"
                        >
                    </div>
                    
                    <button onclick="adjustScore(${index}, 1)" class="w-10 h-10 bg-green-100 text-green-700 rounded-full hover:bg-green-200 transition flex items-center justify-center">
                        <i class="fas fa-plus"></i>
                    </button>
                </div>
                
                <div class="flex justify-between mt-2">
                    <span>0</span>
                    <div class="flex items-center gap-2">
                        <span class="font-black text-lg ${settings.color.split(' ')[2]}">
                            <span id="score${index}">0</span> / ${criterion.max}
                        </span>
                        <button onclick="setMaxScore(${index}, ${criterion.max})" class="px-2 py-1 text-xs bg-blue-100 text-blue-700 hover:bg-blue-200 rounded">
                            أعلى
                        </button>
                        <button onclick="setMinScore(${index})" class="px-2 py-1 text-xs bg-gray-100 text-gray-700 hover:bg-gray-200 rounded">
                            صفر
                        </button>
                    </div>
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

    function adjustScore(index, change) {
        const slider = document.querySelector(`.slider-${index}`);
        const max = parseInt(slider.max);
        const currentValue = parseInt(slider.value) || 0;
        let newValue = currentValue + change;
        
        // التأكد من أن القيمة ضمن النطاق
        if (newValue < 0) newValue = 0;
        if (newValue > max) newValue = max;
        
        slider.value = newValue;
        updateScoreDisplay(index, newValue);
        calculateTotal();
        
        // تأثير عند التغيير
        const scoreElement = document.getElementById(`score${index}`);
        scoreElement.classList.add('animate-pulse');
        setTimeout(() => {
            scoreElement.classList.remove('animate-pulse');
        }, 300);
    }

    function setMaxScore(index, max) {
        const slider = document.querySelector(`.slider-${index}`);
        slider.value = max;
        updateScoreDisplay(index, max);
        calculateTotal();
    }

    function setMinScore(index) {
        const slider = document.querySelector(`.slider-${index}`);
        slider.value = 0;
        updateScoreDisplay(index, 0);
        calculateTotal();
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
        
        // تغيير لون المجموع حسب القيمة
        const totalElement = document.getElementById('totalScore');
        totalElement.classList.remove('text-indigo-700', 'text-green-600', 'text-yellow-600', 'text-red-600');
        
        if (total >= 90) {
            totalElement.classList.add('text-green-600');
        } else if (total >= 70) {
            totalElement.classList.add('text-indigo-700');
        } else if (total >= 60) {
            totalElement.classList.add('text-yellow-600');
        } else {
            totalElement.classList.add('text-red-600');
        }
    }

    // بقية الدوال بدون تغيير
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
            const res = await fetch(SOURCE_SHEET_CSV_URL);
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
        updateAdminStats();
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
                            ${existingEval.synced ? '<span class="px-2 py-1 bg-blue-100 text-blue-800 text-xs rounded mr-2">✓ محفوظ في السحابة</span>' : ''}
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
            timestamp: new Date().toISOString(),
            synced: false
        };

        resultsDB.push(result);
        localStorage.setItem('grad_sys_results', JSON.stringify(resultsDB));
        
        alert(`✅ تم حفظ تقييم الطالب ${studentName} بنجاح!\nالدرجة: ${total} - التقدير: ${calculateGrade(total)}`);
        resetForm();
        updateStats();
        renderAdminTable();
        renderStudentsCards();
    }

    async function saveAndSync() {
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
            timestamp: new Date().toISOString(),
            synced: false
        };

        resultsDB.push(result);
        localStorage.setItem('grad_sys_results', JSON.stringify(resultsDB));
        
        await syncResultToGoogleSheet(result);
        
        resetForm();
        updateStats();
        renderAdminTable();
        renderStudentsCards();
    }

    async function syncResultToGoogleSheet(result) {
        document.getElementById('loading').classList.remove('hidden');
        
        try {
            const group = mainDB.find(g => g.id === result.group) || {};
            
            const dataToSend = {
                student: result.student,
                group: result.group,
                project: group.project || '',
                roleName: result.roleName,
                examiner: result.examiner,
                signature: result.signature || '',
                total: result.total,
                grade: result.grade,
                date: result.date,
                notes: result.notes || '',
                timestamp: new Date().toISOString()
            };
            
            await fetch(GAS_WEB_APP_URL, {
                method: 'POST',
                mode: 'no-cors',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(dataToSend)
            });
            
            result.synced = true;
            localStorage.setItem('grad_sys_results', JSON.stringify(resultsDB));
            
            document.getElementById('loading').classList.add('hidden');
            alert('✅ تم حفظ التقييم ومزامنته مع Google Sheets بنجاح!');
            
        } catch (error) {
            document.getElementById('loading').classList.add('hidden');
            alert('⚠️ تم حفظ التقييم محلياً، ولكن حدث خطأ في المزامنة مع Google Sheets.');
            console.error('Error syncing to Google Sheet:', error);
        }
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
            updateAdminStats();
        }
    }

    function renderAdminTable() {
        const tableBody = document.getElementById('resultsTable');
        tableBody.innerHTML = '';
        
        if (resultsDB.length === 0) {
            tableBody.innerHTML = '<tr><td colspan="10" class="py-8 text-center text-gray-500">لا توجد نتائج مسجلة بعد</td></tr>';
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
                    ${result.synced ? 
                        '<span class="px-2 py-1 bg-green-100 text-green-700 text-xs rounded">✓ محفوظ</span>' : 
                        `<button onclick="syncSingleResult(${result.id})" class="px-2 py-1 bg-orange-100 text-orange-700 hover:bg-orange-200 text-xs rounded">
                            <i class="fas fa-cloud-upload-alt ml-1"></i>مزامنة
                        </button>`
                    }
                </td>
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

    async function syncSingleResult(resultId) {
        const result = resultsDB.find(r => r.id === resultId);
        if (!result) return;
        
        if (confirm('هل تريد مزامنة هذا التقييم مع Google Sheets؟')) {
            await syncResultToGoogleSheet(result);
            renderAdminTable();
        }
    }

    async function syncAllToGoogleSheet() {
        if (resultsDB.length === 0) {
            alert('لا توجد بيانات للمزامنة!');
            return;
        }
        
        if (confirm(`هل تريد مزامنة جميع التقييمات (${resultsDB.length}) مع Google Sheets؟`)) {
            document.getElementById('loading').classList.remove('hidden');
            
            let syncedCount = 0;
            let failedCount = 0;
            
            for (const result of resultsDB) {
                if (!result.synced) {
                    try {
                        const group = mainDB.find(g => g.id === result.group) || {};
                        
                        const dataToSend = {
                            student: result.student,
                            group: result.group,
                            project: group.project || '',
                            roleName: result.roleName,
                            examiner: result.examiner,
                            signature: result.signature || '',
                            total: result.total,
                            grade: result.grade,
                            date: result.date,
                            notes: result.notes || '',
                            timestamp: new Date().toISOString()
                        };
                        
                        await fetch(GAS_WEB_APP_URL, {
                            method: 'POST',
                            mode: 'no-cors',
                            headers: {
                                'Content-Type': 'application/json',
                            },
                            body: JSON.stringify(dataToSend)
                        });
                        
                        result.synced = true;
                        syncedCount++;
                        
                    } catch (error) {
                        failedCount++;
                        console.error('Failed to sync result:', result.student, error);
                    }
                }
            }
            
            localStorage.setItem('grad_sys_results', JSON.stringify(resultsDB));
            document.getElementById('loading').classList.add('hidden');
            
            alert(`✅ تم مزامنة ${syncedCount} تقييم بنجاح!\n${failedCount > 0 ? `فشل في مزامنة ${failedCount} تقييم` : ''}`);
            renderAdminTable();
        }
    }

    async function syncToGoogleSheet() {
        if (resultsDB.length === 0) {
            alert('لا توجد بيانات للمزامنة!');
            return;
        }
        
        await syncAllToGoogleSheet();
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
        
        if (filtered.length === 0) {
            tableBody.innerHTML = '<tr><td colspan="10" class="py-8 text-center text-gray-500">لا توجد نتائج تطابق البحث</td></tr>';
            return;
        }
        
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
                    ${result.synced ? 
                        '<span class="px-2 py-1 bg-green-100 text-green-700 text-xs rounded">✓ محفوظ</span>' : 
                        `<button onclick="syncSingleResult(${result.id})" class="px-2 py-1 bg-orange-100 text-orange-700 hover:bg-orange-200 text-xs rounded">
                            <i class="fas fa-cloud-upload-alt ml-1"></i>مزامنة
                        </button>`
                    }
                </td>
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

    function updateAdminStats() {
        document.getElementById('statsGroups').textContent = mainDB.length;
        
        const totalStudents = mainDB.reduce((sum, group) => sum + group.students.length, 0);
        document.getElementById('statsStudents').textContent = totalStudents;
        
        document.getElementById('statsEvaluations').textContent = resultsDB.length;
    }

    function deleteResult(id) {
        if (confirm('هل أنت متأكد من حذف هذا التقييم؟')) {
            resultsDB = resultsDB.filter(result => result.id !== id);
            localStorage.setItem('grad_sys_results', JSON.stringify(resultsDB));
            renderAdminTable();
            updateStats();
            updateAdminStats();
        }
    }

    function shareResults() {
        if (resultsDB.length === 0) {
            alert('لا توجد نتائج للمشاركة!');
            return;
        }
        
        let message = "📢 *نتائج تقييم المناقشات*\n\n";
        resultsDB.slice(-10).forEach(result => {
            message += `👤 ${result.student}\n`;
            message += `📝 المجموع: ${result.total}\n`;
            message += `🎓 التقدير: ${result.grade}\n`;
            message += `🖋️ الناقش: ${result.examiner}\n`;
            message += `🕙 ${result.date}\n`;
            message += "───────────────\n";
        });
        
        const encodedMessage = encodeURIComponent(message);
        const whatsappURL = `https://wa.me/?text=${encodedMessage}`;
        window.open(whatsappURL, '_blank');
    }

    function shareResult(resultId) {
        const result = resultsDB.find(r => r.id === resultId);
        if (!result) return;
        
        let message = `📢 *نتيجة التقييم*\n\n`;
        message += `👤 *الطالب:* ${result.student}\n`;
        message += `👥 *المجموعة:* ${result.group}\n`;
        message += `📝 *الدرجة الكلية:* ${result.total}/100\n`;
        message += `🎓 *التقدير:* ${result.grade}\n`;
        message += `🖋️ *الناقش:* ${result.examiner}\n`;
        message += `🕙 *التاريخ:* ${result.date}\n\n`;
        
        message += `*تفاصيل الدرجات:*\n`;
        result.scores.forEach(score => {
            message += `• ${score.criterion}: ${score.score}/${score.max}\n`;
        });
        
        const encodedMessage = encodeURIComponent(message);
        const whatsappURL = `https://wa.me/?text=${encodedMessage}`;
        window.open(whatsappURL, '_blank');
    }

    function printPDF() {
        if (resultsDB.length === 0) {
            alert('لا توجد نتائج لطباعة PDF!');
            return;
        }

        const { jsPDF } = window.jspdf;
        const doc = new jsPDF('p', 'pt', 'a4');
        
        doc.setFontSize(20);
        doc.setTextColor(40);
        doc.text('تقرير نتائج تقييم المناقشات', 300, 40, { align: 'center' });
        
        doc.setFontSize(12);
        doc.text(`تاريخ التقرير: ${new Date().toLocaleDateString('ar-SA')}`, 500, 80, { align: 'right' });
        
        doc.setFontSize(11);
        doc.text(`إجمالي المجموعات: ${mainDB.length}`, 500, 110, { align: 'right' });
        doc.text(`إجمالي الطلاب: ${mainDB.reduce((sum, group) => sum + group.students.length, 0)}`, 500, 130, { align: 'right' });
        doc.text(`عدد التقييمات: ${resultsDB.length}`, 500, 150, { align: 'right' });
        
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
        
        doc.setFontSize(10);
        doc.text('توقيع المدير: ___________________', 100, doc.internal.pageSize.height - 40);
        doc.text('توقيع الناقش: ___________________', 400, doc.internal.pageSize.height - 40);
        
        doc.save(`نتائج_التقييم_${new Date().toISOString().split('T')[0]}.pdf`);
    }

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

    function exportToExcelFull() {
        if (resultsDB.length === 0) {
            alert('لا توجد بيانات للتصدير!');
            return;
        }
        
        const wb = XLSX.utils.book_new();
        
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

    function showCopyright() {
        document.getElementById('copyrightModal').classList.remove('hidden');
        document.getElementById('copyrightModal').classList.add('flex');
    }

    function closeCopyright() {
        document.getElementById('copyrightModal').classList.add('hidden');
        document.getElementById('copyrightModal').classList.remove('flex');
    }

    function exportDirectFromSheet() {
        if (mainDB.length === 0) {
            alert('لا توجد بيانات للتصدير! قم بمزامنة البيانات أولاً.');
            return;
        }

        const wb = XLSX.utils.book_new();
        
        const sheetData = [];
        mainDB.forEach(group => {
            group.students.forEach(student => {
                sheetData.push({
                    'اسم الطالب': student,
                    'رقم المجموعة': group.id,
                    'اسم المشروع': group.project,
                    'رقم القاعة': group.room || 'غير محدد',
                    'اسم المشرف': group.supervisor || 'غير محدد',
                    'عدد الطلاب في المجموعة': group.students.length
                });
            });
        });
        
        const ws1 = XLSX.utils.json_to_sheet(sheetData);
        XLSX.utils.book_append_sheet(wb, ws1, "البيانات_الأصلية");
        
        if (resultsDB.length > 0) {
            const evaluationData = resultsDB.map(result => {
                const group = mainDB.find(g => g.id === result.group) || {};
                return {
                    'اسم الطالب': result.student,
                    'رقم المجموعة': result.group,
                    'اسم المشروع': group.project || 'غير معروف',
                    'نوع التقييم': result.roleName,
                    'الناقش/المشرف': result.examiner || 'غير معروف',
                    'التوقيع': result.signature || '',
                    'الدرجة الكلية': result.total,
                    'التقدير': result.grade,
                    'تاريخ التقييم': result.date,
                    'ملاحظات': result.notes || ''
                };
            });
            
            const ws2 = XLSX.utils.json_to_sheet(evaluationData);
            XLSX.utils.book_append_sheet(wb, ws2, "التقييمات");
            
            const detailedData = [];
            resultsDB.forEach(result => {
                result.scores.forEach(score => {
                    detailedData.push({
                        'اسم الطالب': result.student,
                        'المجموعة': result.group,
                        'نوع التقييم': result.roleName,
                        'المعيار': score.criterion,
                        'الدرجة': score.score,
                        'الحد الأقصى': score.max,
                        'النسبة %': ((score.score / score.max) * 100).toFixed(1)
                    });
                });
            });
            
            const ws3 = XLSX.utils.json_to_sheet(detailedData);
            XLSX.utils.book_append_sheet(wb, ws3, "التقييم_المفصل");
        }
        
        const statsData = [
            {
                'المؤشر': 'المجموعات الكلية',
                'القيمة': mainDB.length
            },
            {
                'المؤشر': 'إجمالي الطلاب',
                'القيمة': mainDB.reduce((sum, group) => sum + group.students.length, 0)
            },
            {
                'المؤشر': 'عدد التقييمات',
                'القيمة': resultsDB.length
            },
            {
                'المؤشر': 'نسبة التقييم %',
                'القيمة': mainDB.length > 0 ? 
                    ((resultsDB.length / mainDB.reduce((sum, group) => sum + group.students.length, 0)) * 100).toFixed(1) + '%' : '0%'
            },
            {
                'المؤشر': 'متوسط الدرجات',
                'القيمة': resultsDB.length > 0 ? 
                    (resultsDB.reduce((sum, result) => sum + result.total, 0) / resultsDB.length).toFixed(1) : '0.0'
            },
            {
                'المؤشر': 'أعلى درجة',
                'القيمة': resultsDB.length > 0 ? Math.max(...resultsDB.map(r => r.total)) : '0'
            },
            {
                'المؤشر': 'أقل درجة',
                'القيمة': resultsDB.length > 0 ? Math.min(...resultsDB.map(r => r.total)) : '0'
            }
        ];
        
        const ws4 = XLSX.utils.json_to_sheet(statsData);
        XLSX.utils.book_append_sheet(wb, ws4, "الإحصائيات");
        
        if (examiners.length > 0 || resultsDB.length > 0) {
            const examinersReport = {};
            
            resultsDB.forEach(result => {
                if (!examinersReport[result.examiner]) {
                    examinersReport[result.examiner] = {
                        'عدد التقييمات': 0,
                        'مجموع الدرجات': 0,
                        'متوسط الدرجات': 0,
                        'أعلى درجة': 0,
                        'أقل درجة': 100
                    };
                }
                
                examinersReport[result.examiner]['عدد التقييمات']++;
                examinersReport[result.examiner]['مجموع الدرجات'] += result.total;
                
                if (result.total > examinersReport[result.examiner]['أعلى درجة']) {
                    examinersReport[result.examiner]['أعلى درجة'] = result.total;
                }
                
                if (result.total < examinersReport[result.examiner]['أقل درجة']) {
                    examinersReport[result.examiner]['أقل درجة'] = result.total;
                }
            });
            
            Object.keys(examinersReport).forEach(examiner => {
                const report = examinersReport[examiner];
                report['متوسط الدرجات'] = (report['مجموع الدرجات'] / report['عدد التقييمات']).toFixed(1);
            });
            
            const examinersData = Object.keys(examinersReport).map(examiner => ({
                'اسم الناقش/المشرف': examiner,
                'عدد التقييمات': examinersReport[examiner]['عدد التقييمات'],
                'متوسط الدرجات': examinersReport[examiner]['متوسط الدرجات'],
                'أعلى درجة': examinersReport[examiner]['أعلى درجة'],
                'أقل درجة': examinersReport[examiner]['أقل درجة']
            }));
            
            const ws5 = XLSX.utils.json_to_sheet(examinersData);
            XLSX.utils.book_append_sheet(wb, ws5, "تقرير_الناقشين");
        }
        
        if (resultsDB.length > 0) {
            const datesReport = {};
            
            resultsDB.forEach(result => {
                if (!datesReport[result.date]) {
                    datesReport[result.date] = {
                        'عدد التقييمات': 0,
                        'متوسط الدرجات': 0,
                        'مجموع الدرجات': 0
                    };
                }
                
                datesReport[result.date]['عدد التقييمات']++;
                datesReport[result.date]['مجموع الدرجات'] += result.total;
            });
            
            Object.keys(datesReport).forEach(date => {
                datesReport[date]['متوسط الدرجات'] = 
                    (datesReport[date]['مجموع الدرجات'] / datesReport[date]['عدد التقييمات']).toFixed(1);
            });
            
            const datesData = Object.keys(datesReport).map(date => ({
                'التاريخ': date,
                'عدد التقييمات': datesReport[date]['عدد التقييمات'],
                'متوسط الدرجات': datesReport[date]['متوسط الدرجات']
            }));
            
            datesData.sort((a, b) => new Date(b['التاريخ']) - new Date(a['التاريخ']));
            
            const ws6 = XLSX.utils.json_to_sheet(datesData);
            XLSX.utils.book_append_sheet(wb, ws6, "التقييمات_حسب_التاريخ");
        }
        
        const fileName = `تقرير_تقييم_مناقشات_${new Date().toISOString().split('T')[0]}.xlsx`;
        XLSX.writeFile(wb, fileName);
        
        alert(`✅ تم تصدير الملف بنجاح!\n\nيحتوي الملف على:\n• البيانات الأصلية من Google Sheets\n• جميع التقييمات\n• إحصائيات مفصلة\n• تقارير الناقشين\n• التقييمات حسب التاريخ`);
    }

    function openExcelInBrowser() {
        if (mainDB.length === 0) {
            alert('لا توجد بيانات للعرض! قم بمزامنة البيانات أولاً.');
            return;
        }

        let html = `
        <!DOCTYPE html>
        <html dir="rtl" lang="ar">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>عرض بيانات التقييم</title>
            <style>
                * { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
                body { padding: 20px; background: #f5f7fa; }
                .container { max-width: 100%; }
                h1 { color: #1e3a8a; border-bottom: 3px solid #3b82f6; padding-bottom: 10px; }
                .tabs { display: flex; gap: 10px; margin-bottom: 20px; flex-wrap: wrap; }
                .tab { padding: 10px 20px; background: #e5e7eb; border-radius: 8px; cursor: pointer; }
                .tab.active { background: #3b82f6; color: white; }
                .sheet { display: none; background: white; border-radius: 10px; padding: 20px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); overflow-x: auto; }
                .sheet.active { display: block; }
                table { width: 100%; border-collapse: collapse; margin-top: 10px; }
                th { background: #1e3a8a; color: white; padding: 12px; text-align: right; }
                td { padding: 10px; border: 1px solid #e5e7eb; text-align: right; }
                tr:nth-child(even) { background: #f8fafc; }
                .badge { padding: 4px 8px; border-radius: 12px; font-size: 12px; }
                .badge-primary { background: #dbeafe; color: #1e40af; }
                .stats-card { background: linear-gradient(135deg, #3b82f6, #1d4ed8); color: white; padding: 20px; border-radius: 10px; margin: 10px; }
                .stats-container { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; margin-bottom: 20px; }
                .print-btn { background: #10b981; color: white; padding: 12px 24px; border: none; border-radius: 8px; cursor: pointer; margin: 10px; }
            </style>
        </head>
        <body>
            <div class="container">
                <h1>📊 بيانات تقييم المناقشات - عرض مباشر</h1>
                
                <div class="tabs" id="tabsContainer">
                </div>
                
                <div id="sheetsContainer">
                </div>
                
                <div style="margin-top: 20px; text-align: center;">
                    <button onclick="window.print()" class="print-btn">🖨️ طباعة التقرير</button>
                    <button onclick="window.close()" class="print-btn" style="background: #ef4444;">❌ إغلاق</button>
                </div>
            </div>
            
            <script>
                let currentTab = 'البيانات_الأصلية';
                
                function showTab(tabName) {
                    document.querySelectorAll('.tab').forEach(tab => {
                        tab.classList.remove('active');
                    });
                    document.querySelector(\`[onclick="showTab(''\${tabName}'')"]\`).classList.add('active');
                    
                    document.querySelectorAll('.sheet').forEach(sheet => {
                        sheet.classList.remove('active');
                    });
                    document.getElementById(\`sheet-\${tabName}\`).classList.add('active');
                    
                    currentTab = tabName;
                }
            <\/script>
        </body>
        </html>
        `;

        const newWindow = window.open('', '_blank');
        newWindow.document.write(html);
        newWindow.document.close();
        
        setTimeout(() => {
            const tabsContainer = newWindow.document.getElementById('tabsContainer');
            const sheetsContainer = newWindow.document.getElementById('sheetsContainer');
            
            tabsContainer.innerHTML += `<div class="tab active" onclick="showTab('البيانات_الأصلية')">📋 البيانات الأصلية</div>`;
            
            let sheet1HTML = `<div id="sheet-البيانات_الأصلية" class="sheet active">
                <h2>📋 بيانات الطلاب من Google Sheets</h2>
                <table>
                    <thead>
                        <tr>
                            <th>اسم الطالب</th>
                            <th>رقم المجموعة</th>
                            <th>اسم المشروع</th>
                            <th>رقم القاعة</th>
                            <th>اسم المشرف</th>
                        </tr>
                    </thead>
                    <tbody>`;
            
            mainDB.forEach(group => {
                group.students.forEach(student => {
                    sheet1HTML += `
                    <tr>
                        <td>${student}</td>
                        <td>${group.id}</td>
                        <td>${group.project}</td>
                        <td>${group.room || 'غير محدد'}</td>
                        <td>${group.supervisor || 'غير محدد'}</td>
                    </tr>`;
                });
            });
            
            sheet1HTML += `</tbody></table></div>`;
            sheetsContainer.innerHTML += sheet1HTML;
            
            if (resultsDB.length > 0) {
                tabsContainer.innerHTML += `<div class="tab" onclick="showTab('التقييمات')">📝 التقييمات</div>`;
                
                let sheet2HTML = `<div id="sheet-التقييمات" class="sheet">
                    <h2>📝 سجل التقييمات</h2>
                    <table>
                        <thead>
                            <tr>
                                <th>اسم الطالب</th>
                                <th>المجموعة</th>
                                <th>نوع التقييم</th>
                                <th>الناقش</th>
                                <th>الدرجة</th>
                                <th>التقدير</th>
                                <th>التاريخ</th>
                            </tr>
                        </thead>
                        <tbody>`;
                
                resultsDB.forEach(result => {
                    const gradeClass = result.total >= 90 ? 'color: green; font-weight: bold;' : 
                                    result.total >= 70 ? 'color: blue;' : 
                                    result.total >= 60 ? 'color: orange;' : 'color: red;';
                    
                    sheet2HTML += `
                    <tr>
                        <td>${result.student}</td>
                        <td>${result.group}</td>
                        <td><span class="badge badge-primary">${result.roleName}</span></td>
                        <td>${result.examiner || 'غير معروف'}</td>
                        <td style="${gradeClass}">${result.total}</td>
                        <td>${result.grade}</td>
                        <td>${result.date}</td>
                    </tr>`;
                });
                
                sheet2HTML += `</tbody></table></div>`;
                sheetsContainer.innerHTML += sheet2HTML;
            }
            
            tabsContainer.innerHTML += `<div class="tab" onclick="showTab('الإحصائيات')">📊 الإحصائيات</div>`;
            
            const totalStudents = mainDB.reduce((sum, group) => sum + group.students.length, 0);
            const avgScore = resultsDB.length > 0 ? 
                (resultsDB.reduce((sum, result) => sum + result.total, 0) / resultsDB.length).toFixed(1) : '0.0';
            
            let sheet3HTML = `<div id="sheet-الإحصائيات" class="sheet">
                <h2>📊 إحصائيات التقييم</h2>
                <div class="stats-container">
                    <div class="stats-card">
                        <h3>المجموعات الكلية</h3>
                        <p style="font-size: 2em; margin: 10px 0;">${mainDB.length}</p>
                    </div>
                    <div class="stats-card" style="background: linear-gradient(135deg, #10b981, #059669);">
                        <h3>إجمالي الطلاب</h3>
                        <p style="font-size: 2em; margin: 10px 0;">${totalStudents}</p>
                    </div>
                    <div class="stats-card" style="background: linear-gradient(135deg, #f59e0b, #d97706);">
                        <h3>عدد التقييمات</h3>
                        <p style="font-size: 2em; margin: 10px 0;">${resultsDB.length}</p>
                    </div>
                    <div class="stats-card" style="background: linear-gradient(135deg, #8b5cf6, #7c3aed);">
                        <h3>متوسط الدرجات</h3>
                        <p style="font-size: 2em; margin: 10px 0;">${avgScore}</p>
                    </div>
                </div>`;
            
            if (resultsDB.length > 0) {
                const gradeDistribution = {
                    'ممتاز': resultsDB.filter(r => r.grade === 'ممتاز').length,
                    'جيد جداً': resultsDB.filter(r => r.grade === 'جيد جداً').length,
                    'جيد': resultsDB.filter(r => r.grade === 'جيد').length,
                    'مقبول': resultsDB.filter(r => r.grade === 'مقبول').length,
                    'راسب': resultsDB.filter(r => r.grade === 'راسب').length
                };
                
                sheet3HTML += `
                <h3 style="margin-top: 30px;">توزيع التقديرات</h3>
                <table>
                    <thead>
                        <tr>
                            <th>التقدير</th>
                            <th>عدد الطلاب</th>
                            <th>النسبة %</th>
                        </tr>
                    </thead>
                    <tbody>`;
                
                Object.keys(gradeDistribution).forEach(grade => {
                    const count = gradeDistribution[grade];
                    const percentage = ((count / resultsDB.length) * 100).toFixed(1);
                    
                    sheet3HTML += `
                    <tr>
                        <td>${grade}</td>
                        <td>${count}</td>
                        <td>${percentage}%</td>
                    </tr>`;
                });
                
                sheet3HTML += `</tbody></table>`;
            }
            
            sheet3HTML += `</div>`;
            sheetsContainer.innerHTML += sheet3HTML;
            
        }, 100);
    }

    function quickEvaluation() {
        const modal = document.getElementById('quickEvalModal');
        modal.classList.remove('hidden');
        modal.classList.add('flex');
        
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
                timestamp: new Date().toISOString(),
                synced: false
            };
            
            resultsDB.push(result);
        });
        
        localStorage.setItem('grad_sys_results', JSON.stringify(resultsDB));
        alert(`✅ تم تطبيق الدرجة ${commonScore} على ${group.students.length} طالب!`);
        closeQuickEval();
        renderAdminTable();
        updateStats();
        updateAdminStats();
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
            timestamp: new Date().toISOString(),
            synced: false
        };
        
        resultsDB.push(result);
        localStorage.setItem('grad_sys_results', JSON.stringify(resultsDB));
        
        alert(`✅ تم حفظ تقييم الطالب ${student} بنجاح!\nالدرجة: ${score}`);
        closeStudentEval();
        renderAdminTable();
        updateStats();
        updateAdminStats();
    }

    function clearSignature() {
        document.getElementById('examinerSignature').value = '';
    }

    function clearAllData() {
        if (confirm('⚠️ هل أنت متأكد من حذف جميع البيانات؟ هذا الإجراء لا يمكن التراجع عنه!')) {
            localStorage.removeItem('grad_sys_results');
            localStorage.removeItem('custom_criteria');
            localStorage.removeItem('examiners_list');
            localStorage.removeItem('grad_sys_db');
            resultsDB = [];
            customCriteria = [];
            examiners = [];
            mainDB = [];
            renderAdminTable();
            updateStats();
            updateAdminStats();
            updateExaminersFilter();
            populateGroupSelect();
            alert('تم حذف جميع البيانات بنجاح.');
        }
    }

    window.onload = () => {
        document.getElementById('evalDate').valueAsDate = new Date();
        
        const storedDB = localStorage.getItem('grad_sys_db');
        const storedResults = localStorage.getItem('grad_sys_results');
        const storedCriteria = localStorage.getItem('custom_criteria');
        const storedExaminers = localStorage.getItem('examiners_list');
        const userLoggedIn = localStorage.getItem('user_logged_in');
        
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
        
        // التحقق من حالة الدخول السابقة
        if (userLoggedIn === 'true') {
            document.getElementById('securityLayer').classList.add('hidden');
            document.getElementById('app').classList.remove('hidden');
        }
        
        updateStats();
        updateAdminStats();
        switchSection('home');
    };

</script>
</body>
</html>
