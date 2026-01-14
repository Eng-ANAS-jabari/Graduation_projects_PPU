<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام تقييم مشاريع التخرج | النسخة المحسنة</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { 
            font-family: 'Tajawal', sans-serif; 
            background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
            min-height: 100vh;
        }
        
        .score-input { 
            border: 2px solid #e2e8f0; 
            transition: all 0.2s; 
            text-align: center; 
            font-weight: 700; 
            font-size: 1.1rem; 
        }
        
        .score-input:focus { 
            border-color: #4f46e5; 
            outline: none; 
            background-color: #fffbeb; 
            box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
        }
        
        .student-card {
            transition: all 0.3s ease;
            border: 2px solid transparent;
        }
        
        .student-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
            border-color: #4f46e5;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .fade-in {
            animation: fadeIn 0.5s ease forwards;
        }
        
        .whatsapp-modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.7);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        
        .whatsapp-modal-content {
            background: white;
            padding: 30px;
            border-radius: 20px;
            width: 90%;
            max-width: 500px;
            max-height: 80vh;
            overflow-y: auto;
        }
        
        /* تخصيص شريط البحث */
        .search-bar {
            position: relative;
        }
        
        .search-bar input {
            padding-right: 40px;
        }
        
        .search-bar i {
            position: absolute;
            left: 12px;
            top: 50%;
            transform: translateY(-50%);
            color: #94a3b8;
        }
        
        /* تخصيص الأقسام العلمية */
        .department-badge {
            display: inline-block;
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: 600;
            margin: 2px;
        }
        
        .department-cs { background: #e0f2fe; color: #0369a1; }
        .department-it { background: #f0f9ff; color: #0c4a6e; }
        .department-is { background: #eff6ff; color: #1d4ed8; }
        .department-se { background: #fef3c7; color: #92400e; }
        .department-ce { background: #dcfce7; color: #166534; }
        
        /* تصميم البطاقات العلمية */
        .academic-card {
            background: linear-gradient(135deg, #ffffff 0%, #f8fafc 100%);
            border: 1px solid #e2e8f0;
            border-radius: 16px;
            transition: all 0.3s ease;
        }
        
        .academic-card:hover {
            border-color: #4f46e5;
            box-shadow: 0 10px 25px rgba(79, 70, 229, 0.1);
        }
        
        /* تصميم الجدول العلمي */
        .academic-table {
            width: 100%;
            border-collapse: separate;
            border-spacing: 0;
        }
        
        .academic-table th {
            background: #f1f5f9;
            padding: 12px;
            font-weight: 700;
            text-align: right;
            border-bottom: 2px solid #e2e8f0;
        }
        
        .academic-table td {
            padding: 12px;
            border-bottom: 1px solid #e2e8f0;
        }
        
        .academic-table tr:hover {
            background: #f8fafc;
        }
        
        /* بطاقات الطلاب في الإدارة */
        .student-tag {
            display: inline-flex;
            align-items: center;
            gap: 8px;
            background: #f1f5f9;
            padding: 8px 16px;
            border-radius: 12px;
            margin: 4px;
            transition: all 0.3s ease;
        }
        
        .student-tag:hover {
            background: #e2e8f0;
            transform: translateY(-2px);
        }
        
        /* تصميم الإشعارات */
        .notification {
            position: fixed;
            top: 20px;
            right: 20px;
            padding: 12px 24px;
            border-radius: 12px;
            color: white;
            font-weight: bold;
            z-index: 9999;
            animation: slideIn 0.3s ease, fadeOut 0.3s ease 2.7s;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
        }
        
        .notification-success { background: linear-gradient(135deg, #10b981 0%, #059669 100%); }
        .notification-error { background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%); }
        .notification-info { background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%); }
        .notification-warning { background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%); }
        
        @keyframes slideIn {
            from { transform: translateX(100%); opacity: 0; }
            to { transform: translateX(0); opacity: 1; }
        }
        
        @keyframes fadeOut {
            from { opacity: 1; }
            to { opacity: 0; }
        }
        
        /* تأثيرات خاصة */
        .pulse {
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0% { box-shadow: 0 0 0 0 rgba(79, 70, 229, 0.4); }
            70% { box-shadow: 0 0 0 10px rgba(79, 70, 229, 0); }
            100% { box-shadow: 0 0 0 0 rgba(79, 70, 229, 0); }
        }
        
        .glow {
            box-shadow: 0 0 20px rgba(79, 70, 229, 0.3);
        }
        
        @media print { 
            .no-print { display: none !important; } 
            body { padding: 0 !important; background: white !important; } 
            .container { box-shadow: none !important; border: none !important; }
            .student-card { border: 1px solid #eee !important; break-inside: avoid; }
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <div id="app" class="max-w-7xl mx-auto space-y-6">
        
        <!-- واجهة اختيار الدور -->
        <div id="roleSelection" class="bg-white p-10 rounded-[2.5rem] shadow-2xl text-center no-print border border-slate-200 fade-in">
            <div class="mb-8">
                <div class="w-24 h-24 mx-auto mb-4 bg-gradient-to-r from-indigo-600 to-emerald-600 rounded-full flex items-center justify-center pulse">
                    <i class="fas fa-graduation-cap text-white text-4xl"></i>
                </div>
                <h2 class="text-3xl font-black mb-2 text-slate-800">نظام إدارة مشاريع التخرج العلمية</h2>
                <p class="text-slate-500">إدارة وتقييم مشاريع التخرج للكليات العلمية</p>
            </div>
            
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-10">
                <button onclick="requestAdminAccess()" class="group p-8 bg-gradient-to-br from-slate-50 to-white border-2 border-slate-200 rounded-[2.rem] hover:from-slate-900 hover:to-slate-800 hover:text-white transition-all duration-300 shadow-lg">
                    <div class="text-4xl mb-4">🔐</div>
                    <div class="text-xl font-black">الإدارة العلمية</div>
                    <p class="text-sm mt-2 text-slate-500 group-hover:text-slate-300">بوابة إدارة المشاريع والطلاب</p>
                </button>

                <button onclick="setRole('supervisor')" class="group p-8 bg-gradient-to-br from-white to-indigo-50 border-2 border-indigo-200 rounded-[2.5rem] hover:from-indigo-600 hover:to-indigo-800 hover:text-white transition-all duration-300 shadow-lg">
                    <div class="text-4xl mb-4">📝</div>
                    <div class="text-xl font-black">تقييم المشرف</div>
                    <p class="text-sm mt-2 text-indigo-600 group-hover:text-indigo-200">التقييم التحضيري والتنفيذي</p>
                </button>
                
                <button onclick="setRole('examiner')" class="group p-8 bg-gradient-to-br from-white to-emerald-50 border-2 border-emerald-200 rounded-[2.5rem] hover:from-emerald-600 hover:to-emerald-800 hover:text-white transition-all duration-300 shadow-lg">
                    <div class="text-4xl mb-4">🎓</div>
                    <div class="text-xl font-black">تقييم المناقش</div>
                    <p class="text-sm mt-2 text-emerald-600 group-hover:text-emerald-200">التقييم النهائي والمناقشة</p>
                </button>
            </div>
            
            <!-- إحصائيات النظام -->
            <div class="mt-8 pt-8 border-t border-slate-200">
                <h3 class="text-lg font-bold text-slate-700 mb-6">📊 الإحصائيات العلمية</h3>
                <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                    <div class="bg-gradient-to-br from-indigo-50 to-white p-4 rounded-2xl border border-indigo-100">
                        <div class="text-xl font-bold text-indigo-700" id="totalProjects">0</div>
                        <div class="text-sm text-indigo-600">مشروع علمي</div>
                    </div>
                    <div class="bg-gradient-to-br from-emerald-50 to-white p-4 rounded-2xl border border-emerald-100">
                        <div class="text-xl font-bold text-emerald-700" id="totalStudents">0</div>
                        <div class="text-sm text-emerald-600">طالب مسجل</div>
                    </div>
                    <div class="bg-gradient-to-br from-amber-50 to-white p-4 rounded-2xl border border-amber-100">
                        <div class="text-xl font-bold text-amber-700" id="totalEvaluations">0</div>
                        <div class="text-sm text-amber-600">تقييم مكتمل</div>
                    </div>
                    <div class="bg-gradient-to-br from-purple-50 to-white p-4 rounded-2xl border border-purple-100">
                        <div class="text-xl font-bold text-purple-700" id="activeDepartments">0</div>
                        <div class="text-sm text-purple-600">قسم علمي</div>
                    </div>
                </div>
            </div>
        </div>

        <!-- لوحة التحكم (المسؤول) -->
        <div id="adminPanel" class="hidden bg-white shadow-2xl rounded-[2.5rem] overflow-hidden border border-slate-200 fade-in">
            <div class="bg-gradient-to-r from-slate-900 to-slate-800 p-6 text-white">
                <div class="flex justify-between items-center mb-4">
                    <div>
                        <h2 class="text-2xl font-bold">وحدة الإدارة العلمية</h2>
                        <p class="text-sm opacity-80 mt-1">إدارة مشاريع التخرج للكليات العلمية</p>
                    </div>
                    <button onclick="goBack()" class="bg-white/20 hover:bg-white/30 px-4 py-2 rounded-lg text-sm transition-all flex items-center">
                        <i class="fas fa-arrow-left ml-2"></i> العودة
                    </button>
                </div>
                
                <!-- شريط البحث -->
                <div class="relative">
                    <input type="text" id="adminSearch" placeholder="ابحث عن مشروع، طالب، أو قسم..." 
                           class="w-full p-3 bg-white/10 border border-white/20 rounded-xl text-white placeholder-white/50 outline-none"
                           oninput="filterAdminData()">
                    <i class="fas fa-search absolute left-3 top-3 text-white/50"></i>
                </div>
            </div>
            
            <!-- توجيهات الإدارة -->
            <div class="px-6 py-3 bg-gradient-to-r from-indigo-50 to-white border-b border-slate-200">
                <div class="flex items-center justify-between">
                    <div class="flex items-center gap-3">
                        <div class="w-8 h-8 bg-indigo-100 rounded-lg flex items-center justify-center">
                            <i class="fas fa-info text-indigo-600"></i>
                        </div>
                        <p class="text-sm text-slate-600">إدارة الطلاب العالمية: أضف الطلاب مرة واحدة ليتمكن جميع المستخدمين من استخدامها</p>
                    </div>
                    <div class="flex gap-2">
                        <button onclick="showGlobalStudentsModal()" 
                                class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg text-sm font-bold transition-all flex items-center gap-2">
                            <i class="fas fa-users"></i> إدارة الطلاب العالميين
                        </button>
                    </div>
                </div>
            </div>
            
            <div class="p-6 space-y-8">
                <!-- أدوات الإدارة -->
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <!-- استيراد البيانات -->
                    <div class="academic-card p-6">
                        <div class="flex items-center gap-4 mb-6">
                            <div class="w-14 h-14 bg-indigo-100 rounded-2xl flex items-center justify-center">
                                <i class="fas fa-file-excel text-2xl text-indigo-600"></i>
                            </div>
                            <div>
                                <h3 class="font-bold text-slate-800 text-lg">استيراد البيانات العلمية</h3>
                                <p class="text-sm text-slate-600">رفع ملف Excel يحتوي على بيانات المشاريع والطلاب</p>
                            </div>
                        </div>
                        <input type="file" id="excelUpload" accept=".xlsx, .xls" class="hidden" onchange="importExcel(event)">
                        <button onclick="document.getElementById('excelUpload').click()" 
                                class="w-full bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-3 rounded-xl font-bold transition-all flex items-center justify-center gap-2">
                            <i class="fas fa-upload"></i> رفع ملف بيانات
                        </button>
                    </div>
                    
                    <!-- إضافة مشروع جديد -->
                    <div class="academic-card p-6">
                        <div class="flex items-center gap-4 mb-6">
                            <div class="w-14 h-14 bg-emerald-100 rounded-2xl flex items-center justify-center">
                                <i class="fas fa-plus-circle text-2xl text-emerald-600"></i>
                            </div>
                            <div>
                                <h3 class="font-bold text-slate-800 text-lg">إضافة مشروع جديد</h3>
                                <p class="text-sm text-slate-600">إضافة مشروع تخرج جديد إلى النظام</p>
                            </div>
                        </div>
                        <button onclick="showAddProjectModal()" 
                                class="w-full bg-emerald-600 hover:bg-emerald-700 text-white px-6 py-3 rounded-xl font-bold transition-all flex items-center justify-center gap-2">
                            <i class="fas fa-plus"></i> إضافة مشروع علمي
                        </button>
                    </div>
                    
                    <!-- إدارة الطلاب -->
                    <div class="academic-card p-6 glow">
                        <div class="flex items-center gap-4 mb-6">
                            <div class="w-14 h-14 bg-purple-100 rounded-2xl flex items-center justify-center">
                                <i class="fas fa-users text-2xl text-purple-600"></i>
                            </div>
                            <div>
                                <h3 class="font-bold text-slate-800 text-lg">الطلاب العالميون</h3>
                                <p class="text-sm text-slate-600">قائمة الطلاب المتاحة لجميع المستخدمين</p>
                            </div>
                        </div>
                        <button onclick="showGlobalStudentsModal()" 
                                class="w-full bg-purple-600 hover:bg-purple-700 text-white px-6 py-3 rounded-xl font-bold transition-all flex items-center justify-center gap-2">
                            <i class="fas fa-cog"></i> إدارة الطلاب
                        </button>
                        <div class="mt-4 text-center">
                            <span class="text-sm font-bold text-purple-600" id="globalStudentsCount">0</span>
                            <span class="text-sm text-slate-500"> طالب متاح</span>
                        </div>
                    </div>
                </div>
                
                <!-- إحصائيات مفصلة -->
                <div class="academic-card p-6">
                    <h3 class="font-bold text-slate-800 text-xl mb-6 flex items-center gap-3">
                        <i class="fas fa-chart-bar text-indigo-600"></i>
                        الإحصائيات التفصيلية
                    </h3>
                    <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                        <div class="text-center">
                            <div class="text-2xl font-bold text-indigo-700" id="projectsByDept">0</div>
                            <div class="text-sm text-slate-600">مشروع/قسم</div>
                        </div>
                        <div class="text-center">
                            <div class="text-2xl font-bold text-emerald-700" id="avgStudents">0</div>
                            <div class="text-sm text-slate-600">طالب/مشروع</div>
                        </div>
                        <div class="text-center">
                            <div class="text-2xl font-bold text-amber-700" id="completionRate">0%</div>
                            <div class="text-sm text-slate-600">معدل الإكمال</div>
                        </div>
                        <div class="text-center">
                            <div class="text-2xl font-bold text-purple-700" id="activeSupervisors">0</div>
                            <div class="text-sm text-slate-600">مشرف نشط</div>
                        </div>
                    </div>
                </div>
                
                <!-- قائمة المشاريع -->
                <div>
                    <div class="flex justify-between items-center mb-6">
                        <h3 class="font-bold text-slate-800 text-xl flex items-center gap-3">
                            <i class="fas fa-project-diagram text-indigo-600"></i>
                            المشاريع العلمية المسجلة
                            <span class="text-sm font-normal text-slate-500 bg-slate-100 px-3 py-1 rounded-full" id="projectsCount">0 مشروع</span>
                        </h3>
                        <div class="flex gap-2">
                            <button onclick="exportAcademicReport()" class="bg-slate-100 hover:bg-slate-200 text-slate-700 px-4 py-2 rounded-lg text-sm font-bold transition-all">
                                <i class="fas fa-download ml-1"></i> تقرير
                            </button>
                            <button onclick="refreshData()" class="bg-indigo-100 hover:bg-indigo-200 text-indigo-700 px-4 py-2 rounded-lg text-sm font-bold transition-all">
                                <i class="fas fa-sync-alt"></i>
                            </button>
                        </div>
                    </div>
                    
                    <div id="adminDataList" class="space-y-6">
                        <!-- سيتم تعبئتها ديناميكياً -->
                    </div>
                </div>
            </div>
        </div>

        <!-- نموذج التقييم الرئيسي -->
        <div id="mainContainer" class="hidden bg-white shadow-2xl rounded-[2.5rem] overflow-hidden border border-slate-200 fade-in">
            <!-- الهيدر -->
            <div id="formHeader" class="p-10 text-white text-center relative">
                <button onclick="goBack()" class="absolute top-8 left-8 bg-white/20 px-4 py-2 rounded-full text-xs font-bold hover:bg-white/30 transition-all no-print">
                    <i class="fas fa-arrow-left ml-2"></i> الرئيسية
                </button>
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
                    <button type="button" onclick="exportToExcel()" class="bg-slate-100 hover:bg-slate-200 text-slate-700 px-8 py-4 rounded-2xl font-black transition-all shadow-sm">
                        <i class="fas fa-file-excel ml-2"></i> تصدير Excel
                    </button>
                    <button type="button" onclick="showWhatsAppModal()" class="bg-green-500 hover:bg-green-600 text-white px-8 py-4 rounded-2xl font-black transition-all shadow-lg">
                        <i class="fab fa-whatsapp ml-2"></i> إرسال عبر WhatsApp
                    </button>
                    <button type="button" onclick="window.print()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-10 py-4 rounded-2xl font-black transition-all shadow-xl">
                        <i class="fas fa-print ml-2"></i> طباعة التقرير
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- قالب بطاقة الطالب -->
    <template id="studentTemplate">
        <div class="student-card bg-white border border-slate-200 rounded-[2.5rem] p-8 shadow-sm hover:shadow-md transition-all flex flex-col h-full fade-in">
            <div class="flex justify-between items-start mb-6">
                <div>
                    <span class="text-[10px] font-black text-slate-400 uppercase tracking-widest">طالب مشروع تخرج</span>
                    <div class="flex items-center gap-3 mt-1">
                        <h4 class="student-name-display text-2xl font-black text-slate-800"></h4>
                        <button type="button" onclick="changeStudentName(this)" class="text-slate-400 hover:text-indigo-600 text-sm no-print">
                            <i class="fas fa-edit"></i>
                        </button>
                    </div>
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

    <!-- نموذج إضافة مشروع -->
    <div id="addProjectModal" class="fixed inset-0 bg-black/50 z-50 items-center justify-center hidden">
        <div class="bg-white rounded-3xl p-8 max-w-2xl w-full mx-4 max-h-[90vh] overflow-y-auto">
            <div class="flex justify-between items-center mb-6">
                <h3 class="text-2xl font-bold text-slate-800">إضافة مشروع تخرج جديد</h3>
                <button onclick="closeAddProjectModal()" class="text-slate-500 hover:text-slate-700">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            
            <div class="space-y-6">
                <!-- المعلومات الأساسية -->
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div class="space-y-2">
                        <label class="block font-bold text-slate-700">عنوان المشروع</label>
                        <input type="text" id="projectTitleInput" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" 
                               placeholder="أدخل عنوان المشروع العلمي">
                    </div>
                    <div class="space-y-2">
                        <label class="block font-bold text-slate-700">المشرف الأكاديمي</label>
                        <input type="text" id="projectSupervisorInput" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" 
                               placeholder="اسم المشرف العلمي">
                    </div>
                </div>
                
                <!-- المعلومات العلمية -->
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <div class="space-y-2">
                        <label class="block font-bold text-slate-700">القسم العلمي</label>
                        <select id="projectDepartment" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none">
                            <option value="">اختر القسم</option>
                            <option value="cs">علوم الحاسب</option>
                            <option value="it">تقنية المعلومات</option>
                            <option value="is">نظم المعلومات</option>
                            <option value="se">هندسة البرمجيات</option>
                            <option value="ce">الهندسة المدنية</option>
                            <option value="ee">الهندسة الكهربائية</option>
                            <option value="me">الهندسة الميكانيكية</option>
                            <option value="other">أخرى</option>
                        </select>
                    </div>
                    <div class="space-y-2">
                        <label class="block font-bold text-slate-700">السنة الأكاديمية</label>
                        <select id="projectYear" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none">
                            <option value="2024">2024</option>
                            <option value="2025">2025</option>
                            <option value="2026">2026</option>
                            <option value="2027">2027</option>
                        </select>
                    </div>
                    <div class="space-y-2">
                        <label class="block font-bold text-slate-700">الفصل الدراسي</label>
                        <select id="projectSemester" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none">
                            <option value="1">الأول</option>
                            <option value="2">الثاني</option>
                            <option value="summer">الصيفي</option>
                        </select>
                    </div>
                </div>
                
                <!-- اختيار الطلاب -->
                <div class="space-y-4">
                    <div class="flex justify-between items-center">
                        <label class="block font-bold text-slate-700">الطلاب المشاركون</label>
                        <div class="flex gap-2">
                            <button type="button" onclick="useGlobalStudents()" class="text-sm text-purple-600 hover:text-purple-800 font-bold">
                                <i class="fas fa-users ml-1"></i> استخدام القائمة العالمية
                            </button>
                            <button type="button" onclick="addStudentField()" class="text-sm text-indigo-600 hover:text-indigo-800 font-bold">
                                <i class="fas fa-plus ml-1"></i> إضافة طالب
                            </button>
                        </div>
                    </div>
                    <div id="studentsContainer" class="space-y-3">
                        <div class="flex gap-3">
                            <input type="text" class="flex-1 p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" 
                                   placeholder="اسم الطالب الأول">
                            <button type="button" onclick="removeStudentField(this)" class="text-rose-500 hover:text-rose-700">
                                <i class="fas fa-times"></i>
                            </button>
                        </div>
                    </div>
                    <div id="globalStudentsHint" class="hidden bg-purple-50 p-3 rounded-lg border border-purple-200">
                        <p class="text-sm text-purple-700">
                            <i class="fas fa-info-circle ml-1"></i>
                            تم استخدام القائمة العالمية للطلاب. يمكنك تعديل الأسماء يدوياً إذا أردت.
                        </p>
                    </div>
                </div>
                
                <!-- وصف المشروع -->
                <div class="space-y-2">
                    <label class="block font-bold text-slate-700">وصف المشروع (اختياري)</label>
                    <textarea id="projectDescription" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none h-32" 
                              placeholder="وصف مختصر للمشروع العلمي"></textarea>
                </div>
                
                <!-- الأزرار -->
                <div class="flex justify-end gap-3 pt-6 border-t border-slate-200">
                    <button onclick="closeAddProjectModal()" class="px-6 py-3 border border-slate-300 rounded-xl hover:bg-slate-50 transition-all">
                        إلغاء
                    </button>
                    <button onclick="saveNewProject()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-8 py-3 rounded-xl font-bold transition-all">
                        <i class="fas fa-save ml-2"></i> حفظ المشروع
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- نافذة إدارة الطلاب العالميين -->
    <div id="globalStudentsModal" class="fixed inset-0 bg-black/50 z-50 items-center justify-center hidden">
        <div class="bg-white rounded-3xl p-8 max-w-4xl w-full mx-4 max-h-[90vh] overflow-y-auto">
            <div class="flex justify-between items-center mb-6">
                <h3 class="text-2xl font-bold text-slate-800">إدارة الطلاب العالميين</h3>
                <button onclick="closeGlobalStudentsModal()" class="text-slate-500 hover:text-slate-700">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            
            <div class="space-y-6">
                <!-- التعليمات -->
                <div class="bg-gradient-to-r from-purple-50 to-indigo-50 p-4 rounded-xl border border-purple-200">
                    <div class="flex items-center gap-3">
                        <div class="w-10 h-10 bg-purple-100 rounded-lg flex items-center justify-center">
                            <i class="fas fa-lightbulb text-purple-600"></i>
                        </div>
                        <div>
                            <p class="text-sm font-bold text-slate-700">ملاحظة هامة:</p>
                            <p class="text-sm text-slate-600">الطلاب المضافون هنا سيظهرون تلقائياً لجميع المستخدمين في النظام عند اختيار المشروع</p>
                        </div>
                    </div>
                </div>
                
                <!-- قائمة الطلاب الحالية -->
                <div>
                    <div class="flex justify-between items-center mb-4">
                        <h4 class="font-bold text-slate-700 text-lg">الطلاب المسجلون حالياً</h4>
                        <span class="text-sm text-slate-500" id="currentStudentsCount">0 طالب</span>
                    </div>
                    <div id="globalStudentsList" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3 mb-6">
                        <!-- سيتم تعبئتها ديناميكياً -->
                    </div>
                </div>
                
                <!-- إضافة طالب جديد -->
                <div class="bg-slate-50 p-6 rounded-xl border border-slate-200">
                    <h4 class="font-bold text-slate-700 text-lg mb-4">إضافة طالب جديد</h4>
                    <div class="flex gap-3">
                        <input type="text" id="newStudentName" 
                               class="flex-1 p-3 bg-white border border-slate-300 rounded-xl outline-none" 
                               placeholder="أدخل اسم الطالب الجديد">
                        <button onclick="addGlobalStudent()" 
                                class="bg-purple-600 hover:bg-purple-700 text-white px-6 py-3 rounded-xl font-bold transition-all">
                            <i class="fas fa-plus ml-1"></i> إضافة
                        </button>
                    </div>
                </div>
                
                <!-- استيراد من ملف Excel -->
                <div class="bg-slate-50 p-6 rounded-xl border border-slate-200">
                    <h4 class="font-bold text-slate-700 text-lg mb-4">استيراد أسماء طلاب من ملف Excel</h4>
                    <div class="space-y-4">
                        <div class="flex items-center gap-3">
                            <div class="w-12 h-12 bg-emerald-100 rounded-xl flex items-center justify-center">
                                <i class="fas fa-file-excel text-emerald-600"></i>
                            </div>
                            <div>
                                <p class="text-sm text-slate-600">يمكنك استيراد أسماء الطلاب من ملف Excel يحتوي على عمود "اسم الطالب"</p>
                            </div>
                        </div>
                        <input type="file" id="studentsExcelUpload" accept=".xlsx, .xls" class="hidden" onchange="importStudentsExcel(event)">
                        <button onclick="document.getElementById('studentsExcelUpload').click()" 
                                class="w-full bg-emerald-600 hover:bg-emerald-700 text-white px-6 py-3 rounded-xl font-bold transition-all flex items-center justify-center gap-2">
                            <i class="fas fa-upload"></i> رفع ملف Excel
                        </button>
                    </div>
                </div>
                
                <!-- تصدير القائمة -->
                <div class="bg-slate-50 p-6 rounded-xl border border-slate-200">
                    <h4 class="font-bold text-slate-700 text-lg mb-4">تصدير قائمة الطلاب</h4>
                    <div class="flex gap-3">
                        <button onclick="exportStudentsList()" 
                                class="flex-1 bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-3 rounded-xl font-bold transition-all flex items-center justify-center gap-2">
                            <i class="fas fa-download"></i> تصدير إلى Excel
                        </button>
                        <button onclick="exportStudentsAsText()" 
                                class="flex-1 bg-slate-600 hover:bg-slate-700 text-white px-6 py-3 rounded-xl font-bold transition-all flex items-center justify-center gap-2">
                            <i class="fas fa-file-text"></i> تصدير كملف نصي
                        </button>
                    </div>
                </div>
                
                <!-- حذف الكل -->
                <div class="bg-rose-50 p-6 rounded-xl border border-rose-200">
                    <h4 class="font-bold text-rose-700 text-lg mb-4">خيارات متقدمة</h4>
                    <p class="text-sm text-rose-600 mb-4">⚠️ تنبيه: هذه العملية لا يمكن التراجع عنها</p>
                    <button onclick="clearAllGlobalStudents()" 
                            class="bg-rose-600 hover:bg-rose-700 text-white px-6 py-3 rounded-xl font-bold transition-all flex items-center justify-center gap-2">
                        <i class="fas fa-trash"></i> حذف جميع الطلاب العالميين
                    </button>
                </div>
                
                <!-- الأزرار -->
                <div class="flex justify-end gap-3 pt-6 border-t border-slate-200">
                    <button onclick="closeGlobalStudentsModal()" class="px-6 py-3 border border-slate-300 rounded-xl hover:bg-slate-50 transition-all">
                        إغلاق
                    </button>
                    <button onclick="refreshGlobalStudents()" class="bg-purple-600 hover:bg-purple-700 text-white px-8 py-3 rounded-xl font-bold transition-all">
                        <i class="fas fa-sync-alt ml-2"></i> تحديث القائمة
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- نافذة واتساب -->
    <div id="whatsappModal" class="whatsapp-modal">
        <div class="whatsapp-modal-content">
            <div class="flex justify-between items-center mb-6">
                <h3 class="text-xl font-bold text-slate-800">إرسال النتائج عبر واتساب</h3>
                <button onclick="closeWhatsAppModal()" class="text-slate-500 hover:text-slate-700">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            
            <div class="space-y-4 mb-6">
                <div class="bg-slate-50 p-4 rounded-xl">
                    <label class="block text-sm font-bold text-slate-700 mb-2">👨‍🏫 اسم المقيم</label>
                    <input type="text" id="evaluatorName" class="w-full p-3 bg-white border border-slate-200 rounded-lg outline-none" placeholder="أدخل اسمك">
                </div>
                
                <div class="bg-slate-50 p-4 rounded-xl">
                    <label class="block text-sm font-bold text-slate-700 mb-2">📱 رقم الهاتف (اختياري)</label>
                    <input type="text" id="phoneNumber" class="w-full p-3 bg-white border border-slate-200 rounded-lg outline-none" placeholder="مثال: 966500000000">
                </div>
                
                <div class="bg-slate-50 p-4 rounded-xl">
                    <label class="block text-sm font-bold text-slate-700 mb-2">💬 تخصيص الرسالة</label>
                    <textarea id="customMessage" class="w-full p-3 bg-white border border-slate-200 rounded-lg outline-none h-32" placeholder="أضف ملاحظات إضافية هنا..."></textarea>
                </div>
            </div>
            
            <div class="mb-6">
                <h4 class="font-bold text-slate-700 mb-3">معاينة الرسالة:</h4>
                <div id="messagePreview" class="bg-green-50 border border-green-200 rounded-xl p-4 text-sm text-slate-700 whitespace-pre-line max-h-40 overflow-y-auto">
                    سيظهر محتوى الرسالة هنا...
                </div>
            </div>
            
            <div class="flex gap-3">
                <button onclick="closeWhatsAppModal()" class="flex-1 bg-slate-100 hover:bg-slate-200 text-slate-700 px-6 py-3 rounded-xl font-bold transition-all">
                    إلغاء
                </button>
                <button onclick="sendWhatsAppMessage()" class="flex-1 bg-green-500 hover:bg-green-600 text-white px-6 py-3 rounded-xl font-bold transition-all flex items-center justify-center gap-2">
                    <i class="fab fa-whatsapp"></i>
                    إرسال الآن
                </button>
            </div>
        </div>
    </div>

    <script>
        // قاعدة البيانات المحلية مع دعم الطلاب العالميين
        let db = JSON.parse(localStorage.getItem('grad_db_academic')) || {
            projects: [],
            globalStudents: ["أحمد محمد", "سارة محمود", "خالد عبد الله", "فاطمة علي", "محمد حسن"],
            evaluations: {},
            departments: ["علوم الحاسب", "تقنية المعلومات", "نظم المعلومات", "هندسة البرمجيات"],
            lastModified: new Date().toISOString()
        };
        
        let currentRole = '';
        let currentProject = null;
        let currentStudentElement = null;

        const roles = {
            supervisor: { 
                title: "تقييم المشرف الأكاديمي", 
                subtitle: "التقييم التحضيري والتنفيذي", 
                color: "bg-gradient-to-r from-indigo-600 to-indigo-800",
                roleName: "المشرف",
                criteria: [
                    {id:'book',label:'توثيق البحث العلمي',max:25},
                    {id:'practical',label:'التنفيذ العملي',max:35},
                    {id:'meetings',label:'الحضور والمتابعة',max:20},
                    {id:'ethics',label:'أخلاقيات العمل الجماعي',max:20}
                ] 
            },
            examiner: { 
                title: "تقييم لجنة المناقشة", 
                subtitle: "التقييم النهائي والمناقشة العلمية", 
                color: "bg-gradient-to-r from-emerald-600 to-emerald-800",
                roleName: "المناقش",
                criteria: [
                    {id:'report',label:'جودة التقرير النهائي',max:25},
                    {id:'logic',label:'المنطق البرمجي والتصميم',max:25},
                    {id:'defense',label:'قوة المناقشة والحوار',max:25},
                    {id:'presentation',label:'العرض المرئي والتقديم',max:25}
                ] 
            }
        };

        // تحميل الإحصائيات
        function updateStats() {
            // إحصائيات المشاريع
            document.getElementById('totalProjects').textContent = db.projects.length;
            document.getElementById('totalStudents').textContent = db.globalStudents.length;
            
            // تحديث عداد الطلاب العالميين
            document.getElementById('globalStudentsCount').textContent = db.globalStudents.length;
            
            // حساب التقييمات
            let evalCount = 0;
            for (let key in db.evaluations) {
                if (db.evaluations[key]) evalCount++;
            }
            document.getElementById('totalEvaluations').textContent = evalCount;
            
            // حساب الأقسام النشطة
            const uniqueDepts = [...new Set(db.projects.map(p => p.department).filter(d => d))];
            document.getElementById('activeDepartments').textContent = uniqueDepts.length;
            
            // إحصائيات مفصلة
            document.getElementById('projectsByDept').textContent = db.projects.length > 0 ? 
                Math.round(db.projects.length / Math.max(uniqueDepts.length, 1)) : 0;
            
            // متوسط الطلاب لكل مشروع
            let totalStudentsInProjects = 0;
            db.projects.forEach(p => {
                totalStudentsInProjects += p.students ? p.students.length : 0;
            });
            document.getElementById('avgStudents').textContent = db.projects.length > 0 ? 
                Math.round(totalStudentsInProjects / db.projects.length) : 0;
            
            // معدل الإكمال
            const completionRate = evalCount > 0 ? Math.min(100, Math.round((evalCount / (db.projects.length * 3)) * 100)) : 0;
            document.getElementById('completionRate').textContent = `${completionRate}%`;
            
            // المشرفين النشطين
            const uniqueSupervisors = [...new Set(db.projects.map(p => p.supervisor).filter(s => s))];
            document.getElementById('activeSupervisors').textContent = uniqueSupervisors.length;
            
            // تحديث العداد
            document.getElementById('projectsCount').textContent = `${db.projects.length} مشروع`;
        }

        // وظائف اختيار الدور
        function setRole(role) {
            currentRole = role;
            const cfg = roles[role];
            
            document.getElementById('roleSelection').classList.add('hidden');
            document.getElementById('mainContainer').classList.remove('hidden');
            document.getElementById('formHeader').className = `p-10 text-white text-center relative ${cfg.color}`;
            document.getElementById('headerTitle').innerText = cfg.title;
            document.getElementById('headerSubtitle').innerText = cfg.subtitle;
            
            // تعبئة قائمة المشاريع
            const sel = document.getElementById('projectSelect');
            sel.innerHTML = '<option value="">-- اختر المشروع --</option>' + 
                db.projects.map(p => `<option value="${p.id}">${p.title}</option>`).join('');
            
            // إعداد الحقول الديناميكية
            const dyn = document.getElementById('dynamicFields');
            dyn.innerHTML = `
                <div class="space-y-2">
                    <label class="block font-black text-slate-700 text-sm">المشرف المسؤول</label>
                    <input type="text" id="supervisorName" class="w-full p-3 bg-white border border-slate-200 rounded-xl outline-none font-bold" readonly>
                </div>
                <div class="space-y-2">
                    <label class="block font-black text-slate-700 text-sm">التاريخ</label>
                    <input type="date" id="evalDate" class="w-full p-3 bg-white border border-slate-200 rounded-xl outline-none font-bold" value="${new Date().toISOString().split('T')[0]}">
                </div>`;
            
            showNotification(`مرحباً بك في ${cfg.title}`, 'success');
        }

        // التعامل مع تغيير المشروع
        function handleProjectChange() {
            const projectId = document.getElementById('projectSelect').value;
            const project = db.projects.find(p => p.id === projectId);
            const wrap = document.getElementById('studentsWrapper');
            
            if(!project) { 
                wrap.innerHTML = `
                    <div class="col-span-full py-20 text-center opacity-40">
                        <div class="text-5xl mb-4">🔍</div>
                        <p class="font-bold">يرجى اختيار المشروع</p>
                    </div>`; 
                return; 
            }
            
            currentProject = project;
            document.getElementById('supervisorName').value = project.supervisor;
            wrap.innerHTML = '';
            const criteria = roles[currentRole].criteria;

            // استخدام الطلاب المحليين إذا وجدوا، وإلا استخدم القائمة العالمية
            const students = project.students && project.students.length > 0 ? project.students : db.globalStudents;
            
            students.forEach((name, index) => {
                const temp = document.getElementById('studentTemplate').content.cloneNode(true);
                const card = temp.querySelector('.student-card');
                card.dataset.studentIndex = index;
                
                // تعيين اسم الطالب
                const nameDisplay = card.querySelector('.student-name-display');
                nameDisplay.textContent = name;
                
                // تحميل التقييمات السابقة إن وجدت
                const studentKey = `${projectId}_${index}`;
                const previousScores = db.evaluations[studentKey]?.[currentRole] || {};
                
                // إضافة معايير التقييم
                criteria.forEach(crit => {
                    const row = document.createElement('div');
                    row.innerHTML = `
                        <div class="flex justify-between text-[10px] font-black text-slate-400 mb-2 uppercase tracking-tighter">
                            <span>${crit.label}</span>
                            <span>الحد الأقصى: ${crit.max}</span>
                        </div>
                        <input type="number" min="0" max="${crit.max}" value="${previousScores[crit.id] || 0}" 
                               class="score-input w-full p-2 rounded-2xl border" 
                               oninput="updateCardScore(this, ${crit.max}, '${studentKey}', '${crit.id}')">`;
                    card.querySelector('.criteria-list').appendChild(row);
                });
                
                wrap.appendChild(temp);
                updateCardTotal(card, studentKey);
            });
            
            showNotification(`تم تحميل ${students.length} طالب`, 'info');
        }

        // تحديث درجات الطالب
        function updateCardScore(input, max, studentKey, criteriaId) {
            let val = parseInt(input.value) || 0;
            if(val > max) input.value = max;
            if(val < 0) input.value = 0;
            
            // حفظ في قاعدة البيانات
            if (!db.evaluations[studentKey]) {
                db.evaluations[studentKey] = {};
            }
            if (!db.evaluations[studentKey][currentRole]) {
                db.evaluations[studentKey][currentRole] = {};
            }
            db.evaluations[studentKey][currentRole][criteriaId] = val;
            saveDatabase();
            
            // تحديث البطاقة
            const card = input.closest('.student-card');
            updateCardTotal(card, studentKey);
        }

        function updateCardTotal(card, studentKey) {
            const inputs = card.querySelectorAll('.score-input');
            let total = 0;
            inputs.forEach(i => total += (parseInt(i.value) || 0));
            
            // تحديث المجموع
            card.querySelector('.student-total-display').innerText = total;
            
            // تحديث التقدير
            const badge = card.querySelector('.student-result-text');
            if(total >= 90) { 
                badge.innerText = "ممتاز"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-indigo-100 text-indigo-700 uppercase tracking-wide"; 
            }
            else if(total >= 80) { 
                badge.innerText = "جيد جداً"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-emerald-100 text-emerald-700 uppercase tracking-wide"; 
            }
            else if(total >= 70) { 
                badge.innerText = "جيد"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-blue-100 text-blue-700 uppercase tracking-wide"; 
            }
            else if(total >= 60) { 
                badge.innerText = "مقبول"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-amber-100 text-amber-700 uppercase tracking-wide"; 
            }
            else if(total >= 50) { 
                badge.innerText = "ناجح"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-green-100 text-green-700 uppercase tracking-wide"; 
            }
            else { 
                badge.innerText = "راسب"; 
                badge.className = "student-result-text font-black text-xs px-5 py-2 rounded-full bg-rose-100 text-rose-700 uppercase tracking-wide"; 
            }
        }

        // الإدارة والبيانات
        function requestAdminAccess() {
            const password = prompt("أدخل كلمة مرور الإدارة:");
            if (password === "admin") {
                showSection('admin');
                updateStats();
                renderGlobalStudentsList();
            } else {
                showNotification("عذراً، كلمة المرور خاطئة", "error");
            }
        }

        function showSection(id) {
            document.getElementById('roleSelection').classList.add('hidden');
            document.getElementById('adminPanel').classList.toggle('hidden', id !== 'admin');
            document.getElementById('mainContainer').classList.toggle('hidden', id !== 'main');
            
            if (id === 'admin') {
                renderAdminData();
                renderGlobalStudentsList();
            }
        }

        function goBack() {
            document.getElementById('adminPanel').classList.add('hidden');
            document.getElementById('mainContainer').classList.add('hidden');
            document.getElementById('roleSelection').classList.remove('hidden');
            updateStats();
        }

        function renderAdminData() {
            const list = document.getElementById('adminDataList');
            
            if (db.projects.length === 0) {
                list.innerHTML = `
                    <div class="text-center py-12">
                        <div class="text-5xl mb-4 text-slate-300">📚</div>
                        <h4 class="text-xl font-bold text-slate-600 mb-2">لا توجد مشاريع مسجلة</h4>
                        <p class="text-slate-500 mb-6">ابدأ بإضافة مشروعك الأول باستخدام زر "إضافة مشروع علمي"</p>
                        <button onclick="showAddProjectModal()" class="bg-indigo-600 text-white px-6 py-3 rounded-xl font-bold">
                            <i class="fas fa-plus ml-2"></i> إضافة مشروع أول
                        </button>
                    </div>
                `;
                return;
            }
            
            list.innerHTML = db.projects.map(p => {
                const deptClass = getDepartmentClass(p.department);
                const deptName = getDepartmentName(p.department);
                
                return `
                <div class="academic-card p-6">
                    <div class="flex justify-between items-start mb-4">
                        <div>
                            <div class="flex items-center gap-3 mb-2">
                                <h5 class="font-black text-slate-800 text-lg">${p.title}</h5>
                                <span class="department-badge ${deptClass}">${deptName}</span>
                            </div>
                            <p class="text-sm text-slate-500">
                                <i class="fas fa-user-tie ml-1"></i> إشراف: ${p.supervisor}
                            </p>
                            <div class="flex items-center gap-4 mt-3">
                                <span class="text-xs font-bold text-slate-600">
                                    <i class="fas fa-calendar ml-1"></i> ${p.year || '2024'}
                                </span>
                                <span class="text-xs font-bold text-slate-600">
                                    <i class="fas fa-graduation-cap ml-1"></i> الفصل ${p.semester === '1' ? 'الأول' : p.semester === '2' ? 'الثاني' : 'الصيفي'}
                                </span>
                            </div>
                        </div>
                        <div class="flex gap-2">
                            <button onclick="editProject('${p.id}')" class="text-indigo-600 hover:text-indigo-800">
                                <i class="fas fa-edit"></i>
                            </button>
                            <button onclick="deleteProject('${p.id}')" class="text-rose-500 hover:text-rose-700">
                                <i class="fas fa-trash"></i>
                            </button>
                        </div>
                    </div>
                    
                    <div class="mt-4">
                        <div class="flex justify-between items-center mb-3">
                            <div class="text-xs font-bold text-slate-400">الطلاب المشاركون:</div>
                            <span class="text-xs text-slate-500">${p.students ? p.students.length : db.globalStudents.length} طالب</span>
                        </div>
                        <div class="flex flex-wrap gap-2">
                            ${(p.students && p.students.length > 0 ? p.students : db.globalStudents.slice(0, 4)).map(s => `
                                <span class="bg-slate-100 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-lg">
                                    <i class="fas fa-user-graduate ml-1"></i> ${s}
                                </span>
                            `).join('')}
                            ${p.students && p.students.length > 4 ? `<span class="bg-slate-200 text-slate-600 text-xs font-bold px-3 py-1.5 rounded-lg">+${p.students.length - 4}</span>` : ''}
                        </div>
                    </div>
                    
                    <div class="mt-6 pt-4 border-t border-slate-200 flex justify-between items-center">
                        <div class="text-xs text-slate-500">
                            <i class="fas fa-clock ml-1"></i> ${formatDate(p.createdAt)}
                        </div>
                        <div class="flex gap-2">
                            <button onclick="viewProjectDetails('${p.id}')" class="text-sm text-indigo-600 hover:text-indigo-800 font-bold">
                                <i class="fas fa-eye ml-1"></i> تفاصيل
                            </button>
                            <button onclick="manageProjectStudents('${p.id}')" class="text-sm text-purple-600 hover:text-purple-800 font-bold">
                                <i class="fas fa-users ml-1"></i> الطلاب
                            </button>
                        </div>
                    </div>
                </div>
                `;
            }).join('');
        }

        // إدارة الطلاب العالميين
        function showGlobalStudentsModal() {
            renderGlobalStudentsList();
            document.getElementById('globalStudentsModal').classList.remove('hidden');
        }

        function closeGlobalStudentsModal() {
            document.getElementById('globalStudentsModal').classList.add('hidden');
        }

        function renderGlobalStudentsList() {
            const list = document.getElementById('globalStudentsList');
            const count = document.getElementById('currentStudentsCount');
            
            if (db.globalStudents.length === 0) {
                list.innerHTML = `
                    <div class="col-span-full text-center py-8">
                        <div class="text-4xl mb-4 text-slate-300">👥</div>
                        <p class="text-slate-500">لا يوجد طلاب مسجلين في القائمة العالمية</p>
                    </div>
                `;
                count.textContent = '0 طالب';
                return;
            }
            
            list.innerHTML = db.globalStudents.map((student, index) => `
                <div class="student-tag">
                    <div class="flex-1">
                        <div class="font-bold text-slate-700">${student}</div>
                        <div class="text-xs text-slate-400">طالب ${index + 1}</div>
                    </div>
                    <button onclick="removeGlobalStudent(${index})" class="text-rose-500 hover:text-rose-700">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
            `).join('');
            
            count.textContent = `${db.globalStudents.length} طالب`;
        }

        function addGlobalStudent() {
            const input = document.getElementById('newStudentName');
            const name = input.value.trim();
            
            if (!name) {
                showNotification('يرجى إدخال اسم الطالب', 'error');
                return;
            }
            
            if (db.globalStudents.includes(name)) {
                showNotification('هذا الطالب مسجل بالفعل', 'warning');
                return;
            }
            
            db.globalStudents.push(name);
            saveDatabase();
            renderGlobalStudentsList();
            updateStats();
            
            input.value = '';
            showNotification('تم إضافة الطالب بنجاح', 'success');
        }

        function removeGlobalStudent(index) {
            if (confirm('هل أنت متأكد من حذف هذا الطالب من القائمة العالمية؟')) {
                db.globalStudents.splice(index, 1);
                saveDatabase();
                renderGlobalStudentsList();
                updateStats();
                showNotification('تم حذف الطالب بنجاح', 'warning');
            }
        }

        function importStudentsExcel(e) {
            const file = e.target.files[0];
            if (!file) return;
            
            const reader = new FileReader();
            reader.onload = (event) => {
                try {
                    const data = new Uint8Array(event.target.result);
                    const workbook = XLSX.read(data, { type: 'array' });
                    const sheet = workbook.Sheets[workbook.SheetNames[0]];
                    const json = XLSX.utils.sheet_to_json(sheet);
                    
                    let importedCount = 0;
                    
                    json.forEach(row => {
                        const studentName = row['اسم الطالب'] || row['student'] || row['Student'] || row['اسم'] || row['الطالب'];
                        
                        if (studentName && !db.globalStudents.includes(studentName)) {
                            db.globalStudents.push(studentName);
                            importedCount++;
                        }
                    });
                    
                    saveDatabase();
                    renderGlobalStudentsList();
                    updateStats();
                    showNotification(`تم استيراد ${importedCount} طالب جديد`, 'success');
                    
                } catch (error) {
                    console.error('Import error:', error);
                    showNotification('خطأ في استيراد الملف', 'error');
                }
            };
            
            reader.readAsArrayBuffer(file);
        }

        function exportStudentsList() {
            const data = [
                ["قائمة الطلاب العالميين - نظام إدارة مشاريع التخرج"],
                ["تاريخ التصدير", new Date().toLocaleDateString('ar-SA')],
                ["عدد الطلاب", db.globalStudents.length],
                [],
                ["م", "اسم الطالب"]
            ];
            
            db.globalStudents.forEach((student, index) => {
                data.push([index + 1, student]);
            });
            
            const ws = XLSX.utils.aoa_to_sheet(data);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, "الطلاب");
            XLSX.writeFile(wb, `قائمة_الطلاب_العالميين_${new Date().toISOString().split('T')[0]}.xlsx`);
            
            showNotification('تم تصدير قائمة الطلاب', 'success');
        }

        function exportStudentsAsText() {
            let content = "قائمة الطلاب العالميين - نظام إدارة مشاريع التخرج\n";
            content += `تاريخ التصدير: ${new Date().toLocaleDateString('ar-SA')}\n`;
            content += `عدد الطلاب: ${db.globalStudents.length}\n\n`;
            
            db.globalStudents.forEach((student, index) => {
                content += `${index + 1}. ${student}\n`;
            });
            
            const blob = new Blob([content], { type: 'text/plain;charset=utf-8' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `قائمة_الطلاب_${new Date().toISOString().split('T')[0]}.txt`;
            a.click();
            URL.revokeObjectURL(url);
            
            showNotification('تم تصدير القائمة كملف نصي', 'success');
        }

        function clearAllGlobalStudents() {
            if (confirm('⚠️ هل أنت متأكد من حذف جميع الطلاب العالميين؟ هذه العملية لا يمكن التراجع عنها.')) {
                db.globalStudents = [];
                saveDatabase();
                renderGlobalStudentsList();
                updateStats();
                showNotification('تم حذف جميع الطلاب العالميين', 'warning');
            }
        }

        function refreshGlobalStudents() {
            renderGlobalStudentsList();
            showNotification('تم تحديث القائمة', 'info');
        }

        // استخدام القائمة العالمية في إضافة المشروع
        function useGlobalStudents() {
            const container = document.getElementById('studentsContainer');
            const hint = document.getElementById('globalStudentsHint');
            
            // تفريغ الحقول الحالية
            container.innerHTML = '';
            
            // إضافة الطلاب العالميين
            db.globalStudents.slice(0, 6).forEach((student, index) => {
                const div = document.createElement('div');
                div.className = 'flex gap-3';
                div.innerHTML = `
                    <input type="text" value="${student}" class="flex-1 p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none">
                    ${index > 0 ? `<button type="button" onclick="removeStudentField(this)" class="text-rose-500 hover:text-rose-700">
                        <i class="fas fa-times"></i>
                    </button>` : ''}
                `;
                container.appendChild(div);
            });
            
            // إضافة حقل فارغ لإمكانية إضافة المزيد
            const addDiv = document.createElement('div');
            addDiv.className = 'flex gap-3';
            addDiv.innerHTML = `
                <input type="text" placeholder="إضافة طالب إضافي..." class="flex-1 p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none">
                <button type="button" onclick="removeStudentField(this)" class="text-rose-500 hover:text-rose-700">
                    <i class="fas fa-times"></i>
                </button>
            `;
            container.appendChild(addDiv);
            
            // إظهار التلميح
            hint.classList.remove('hidden');
            
            showNotification('تم استخدام القائمة العالمية للطلاب', 'info');
        }

        // بقية الوظائف تبقى كما هي مع تعديلات طفيفة
        // [يتبع باقي الكود بدون تغيير كبير...]

        function getDepartmentClass(dept) {
            const deptMap = {
                'cs': 'department-cs',
                'it': 'department-it',
                'is': 'department-is',
                'se': 'department-se',
                'ce': 'department-ce',
                'ee': 'department-cs',
                'me': 'department-it',
                'other': 'department-is'
            };
            return deptMap[dept] || 'department-cs';
        }

        function getDepartmentName(dept) {
            const deptMap = {
                'cs': 'علوم الحاسب',
                'it': 'تقنية المعلومات',
                'is': 'نظم المعلومات',
                'se': 'هندسة البرمجيات',
                'ce': 'الهندسة المدنية',
                'ee': 'الهندسة الكهربائية',
                'me': 'الهندسة الميكانيكية',
                'other': 'أخرى'
            };
            return deptMap[dept] || 'غير محدد';
        }

        function formatDate(dateString) {
            if (!dateString) return 'غير محدد';
            const date = new Date(dateString);
            return date.toLocaleDateString('ar-SA');
        }

        function filterAdminData() {
            const searchTerm = document.getElementById('adminSearch').value.toLowerCase();
            const projects = document.querySelectorAll('.academic-card');
            
            projects.forEach(card => {
                const text = card.textContent.toLowerCase();
                card.style.display = text.includes(searchTerm) ? 'block' : 'none';
            });
        }

        function showAddProjectModal() {
            document.getElementById('addProjectModal').classList.remove('hidden');
        }

        function closeAddProjectModal() {
            document.getElementById('addProjectModal').classList.add('hidden');
            // إعادة تعليم الحقل التلميحي
            document.getElementById('globalStudentsHint').classList.add('hidden');
        }

        function addStudentField() {
            const container = document.getElementById('studentsContainer');
            const div = document.createElement('div');
            div.className = 'flex gap-3';
            div.innerHTML = `
                <input type="text" class="flex-1 p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" 
                       placeholder="اسم الطالب">
                <button type="button" onclick="removeStudentField(this)" class="text-rose-500 hover:text-rose-700">
                    <i class="fas fa-times"></i>
                </button>
            `;
            container.appendChild(div);
        }

        function removeStudentField(button) {
            const container = document.getElementById('studentsContainer');
            if (container.children.length > 1) {
                button.closest('.flex').remove();
            }
        }

        function saveNewProject() {
            const title = document.getElementById('projectTitleInput').value.trim();
            const supervisor = document.getElementById('projectSupervisorInput').value.trim();
            const department = document.getElementById('projectDepartment').value;
            const year = document.getElementById('projectYear').value;
            const semester = document.getElementById('projectSemester').value;
            const description = document.getElementById('projectDescription').value.trim();
            
            // جمع أسماء الطلاب
            const studentInputs = document.querySelectorAll('#studentsContainer input');
            const students = Array.from(studentInputs)
                .map(input => input.value.trim())
                .filter(name => name.length > 0);
            
            // التحقق من المدخلات
            if (!title || !supervisor || !department) {
                showNotification('يرجى تعبئة جميع الحقول الإلزامية (العنوان، المشرف، القسم)', 'error');
                return;
            }
            
            if (students.length === 0) {
                showNotification('يرجى إدخال اسم طالب واحد على الأقل', 'error');
                return;
            }
            
            // إنشاء مشروع جديد
            const newProject = {
                id: 'proj_' + Date.now(),
                title,
                supervisor,
                department,
                year,
                semester,
                students,
                description,
                createdAt: new Date().toISOString(),
                status: 'نشط'
            };
            
            db.projects.push(newProject);
            saveDatabase();
            renderAdminData();
            updateStats();
            closeAddProjectModal();
            
            // تفريغ الحقول
            document.getElementById('projectTitleInput').value = '';
            document.getElementById('projectSupervisorInput').value = '';
            document.getElementById('projectDescription').value = '';
            
            showNotification('تم إضافة المشروع العلمي بنجاح', 'success');
        }

        function editProject(projectId) {
            const project = db.projects.find(p => p.id === projectId);
            if (!project) return;
            
            const newTitle = prompt('تعديل عنوان المشروع العلمي:', project.title);
            if (newTitle && newTitle !== project.title) {
                project.title = newTitle;
                saveDatabase();
                renderAdminData();
                showNotification('تم تعديل المشروع العلمي', 'info');
            }
        }

        function manageProjectStudents(projectId) {
            const project = db.projects.find(p => p.id === projectId);
            if (!project) return;
            
            let studentsList = project.students && project.students.length > 0 ? 
                project.students.join('\n') : db.globalStudents.join('\n');
            
            const newStudents = prompt('إدارة طلاب المشروع (سطر واحد لكل طالب):', studentsList);
            if (newStudents !== null) {
                const studentsArray = newStudents.split('\n')
                    .map(s => s.trim())
                    .filter(s => s.length > 0);
                
                if (studentsArray.length > 0) {
                    project.students = studentsArray;
                    saveDatabase();
                    renderAdminData();
                    showNotification('تم تحديث قائمة طلاب المشروع', 'success');
                }
            }
        }

        function viewProjectDetails(projectId) {
            const project = db.projects.find(p => p.id === projectId);
            if (!project) return;
            
            const deptName = getDepartmentName(project.department);
            const semesterName = project.semester === '1' ? 'الأول' : project.semester === '2' ? 'الثاني' : 'الصيفي';
            
            let details = `📋 *تفاصيل المشروع العلمي*\n\n`;
            details += `🏷️ *العنوان:* ${project.title}\n`;
            details += `👨‍🏫 *المشرف:* ${project.supervisor}\n`;
            details += `🎓 *القسم:* ${deptName}\n`;
            details += `📅 *السنة:* ${project.year}\n`;
            details += `📚 *الفصل:* ${semesterName}\n`;
            details += `👥 *عدد الطلاب:* ${project.students ? project.students.length : db.globalStudents.length}\n`;
            details += `📝 *الوصف:* ${project.description || 'لا يوجد وصف'}\n\n`;
            details += `📊 *الطلاب المشاركون:*\n`;
            
            const students = project.students && project.students.length > 0 ? project.students : db.globalStudents;
            students.forEach((student, index) => {
                details += `${index + 1}. ${student}\n`;
            });
            
            alert(details);
        }

        function deleteProject(projectId) {
            if (confirm('هل أنت متأكد من حذف هذا المشروع العلمي؟')) {
                db.projects = db.projects.filter(p => p.id !== projectId);
                saveDatabase();
                renderAdminData();
                updateStats();
                showNotification('تم حذف المشروع العلمي', 'warning');
            }
        }

        function importExcel(e) {
            const file = e.target.files[0];
            if (!file) return;
            
            const reader = new FileReader();
            reader.onload = (event) => {
                try {
                    const data = new Uint8Array(event.target.result);
                    const workbook = XLSX.read(data, { type: 'array' });
                    const sheet = workbook.Sheets[workbook.SheetNames[0]];
                    const json = XLSX.utils.sheet_to_json(sheet);
                    
                    let importedCount = 0;
                    
                    json.forEach(row => {
                        const title = row['عنوان المشروع'] || row['project'] || row['Project'];
                        const student = row['اسم الطالب'] || row['student'] || row['Student'];
                        const supervisor = row['المشرف'] || row['supervisor'] || row['Supervisor'];
                        const department = row['القسم'] || row['department'] || 'other';
                        const year = row['السنة'] || row['year'] || '2024';
                        
                        if (title && student && supervisor) {
                            // البحث عن المشروع
                            let project = db.projects.find(p => p.title === title && p.supervisor === supervisor);
                            
                            if (project) {
                                // إضافة الطالب إذا لم يكن موجوداً
                                if (!project.students.includes(student)) {
                                    project.students.push(student);
                                    importedCount++;
                                }
                            } else {
                                // إنشاء مشروع جديد
                                db.projects.push({
                                    id: 'proj_' + Date.now() + Math.random(),
                                    title,
                                    supervisor,
                                    department,
                                    year,
                                    semester: '1',
                                    students: [student],
                                    createdAt: new Date().toISOString(),
                                    status: 'نشط'
                                });
                                importedCount++;
                            }
                            
                            // إضافة الطالب إلى القائمة العالمية إذا لم يكن موجوداً
                            if (!db.globalStudents.includes(student)) {
                                db.globalStudents.push(student);
                            }
                        }
                    });
                    
                    saveDatabase();
                    renderAdminData();
                    renderGlobalStudentsList();
                    updateStats();
                    showNotification(`تم استيراد ${importedCount} سجل بنجاح`, 'success');
                    
                } catch (error) {
                    console.error('Import error:', error);
                    showNotification('خطأ في استيراد الملف', 'error');
                }
            };
            
            reader.readAsArrayBuffer(file);
        }

        function exportAcademicReport() {
            const data = [
                ["التقرير العلمي الشامل - مشاريع التخرج"],
                ["تاريخ التصدير", new Date().toLocaleDateString('ar-SA')],
                ["عدد المشاريع", db.projects.length],
                ["عدد الطلاب العالميين", db.globalStudents.length],
                [],
                ["م", "العنوان", "المشرف", "القسم", "السنة", "عدد الطلاب"]
            ];
            
            db.projects.forEach((p, i) => {
                data.push([
                    i + 1,
                    p.title,
                    p.supervisor,
                    getDepartmentName(p.department),
                    p.year || '2024',
                    p.students ? p.students.length : db.globalStudents.length
                ]);
            });
            
            const ws = XLSX.utils.aoa_to_sheet(data);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, "المشاريع");
            XLSX.writeFile(wb, `التقرير_العلمي_${new Date().toISOString().split('T')[0]}.xlsx`);
            
            showNotification('تم تصدير التقرير العلمي', 'success');
        }

        function refreshData() {
            renderAdminData();
            renderGlobalStudentsList();
            updateStats();
            showNotification('تم تحديث البيانات', 'info');
        }

        function saveDatabase() {
            db.lastModified = new Date().toISOString();
            localStorage.setItem('grad_db_academic', JSON.stringify(db));
        }

        function showNotification(message, type = 'info') {
            // إزالة أي إشعارات سابقة
            const existing = document.querySelectorAll('.notification');
            existing.forEach(el => el.remove());
            
            const notification = document.createElement('div');
            notification.className = `notification notification-${type}`;
            notification.innerHTML = `
                <div class="flex items-center gap-3">
                    <i class="fas fa-${type === 'success' ? 'check-circle' : type === 'error' ? 'exclamation-circle' : type === 'warning' ? 'exclamation-triangle' : 'info-circle'}"></i>
                    <span>${message}</span>
                </div>
            `;
            
            document.body.appendChild(notification);
            
            setTimeout(() => {
                if (notification.parentNode) {
                    notification.parentNode.removeChild(notification);
                }
            }, 3000);
        }

        // التصدير
        function exportToExcel() {
            if (!currentProject) {
                showNotification('يرجى اختيار مشروع أولاً', 'error');
                return;
            }
            
            const project = currentProject;
            const roleName = roles[currentRole].roleName;
            const date = document.getElementById('evalDate').value;
            
            const data = [
                ["تقرير نتائج التقييم النهائي"],
                ["نوع التقييم", roleName],
                ["المشروع", project.title],
                ["المشرف الأكاديمي", project.supervisor],
                ["تاريخ التقييم", date],
                ["القسم العلمي", getDepartmentName(project.department)],
                [],
                ["م", "اسم الطالب", "الدرجة النهائية (من 100)", "التقدير"]
            ];
            
            document.querySelectorAll('.student-card').forEach((card, index) => {
                data.push([
                    index + 1,
                    card.querySelector('.student-name-display').innerText,
                    card.querySelector('.student-total-display').innerText,
                    card.querySelector('.student-result-text').innerText
                ]);
            });
            
            const ws = XLSX.utils.aoa_to_sheet(data);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, "النتائج");
            XLSX.writeFile(wb, `تقرير_تقييم_${project.title.replace(/[^\w\s]/gi, '')}_${new Date().toISOString().split('T')[0]}.xlsx`);
            
            showNotification('تم تصدير التقرير إلى Excel', 'success');
        }

        // واتساب
        function showWhatsAppModal() {
            if (!currentProject) {
                showNotification('يرجى اختيار مشروع أولاً', 'error');
                return;
            }
            
            const modal = document.getElementById('whatsappModal');
            modal.style.display = 'flex';
            updateMessagePreview();
        }

        function closeWhatsAppModal() {
            document.getElementById('whatsappModal').style.display = 'none';
        }

        function updateMessagePreview() {
            const evaluatorName = document.getElementById('evaluatorName').value || 'المقيم';
            const customMessage = document.getElementById('customMessage').value;
            const project = currentProject;
            const roleName = roles[currentRole].roleName;
            const date = document.getElementById('evalDate').value;
            
            let message = `📋 *تقرير التقييم النهائي*\n\n`;
            message += `🎓 *نوع التقييم:* ${roleName}\n`;
            message += `👨‍🏫 *اسم المقيم:* ${evaluatorName}\n`;
            message += `📁 *اسم المشروع:* ${project.title}\n`;
            message += `👨‍🏫 *المشرف الأكاديمي:* ${project.supervisor}\n`;
            message += `🎓 *القسم العلمي:* ${getDepartmentName(project.department)}\n`;
            message += `📅 *تاريخ التقييم:* ${date}\n\n`;
            message += `📊 *نتائج تقييم الطلاب:*\n`;
            message += `══════════════════\n\n`;
            
            document.querySelectorAll('.student-card').forEach((card, index) => {
                const studentName = card.querySelector('.student-name-display').textContent;
                const totalScore = card.querySelector('.student-total-display').textContent;
                const grade = card.querySelector('.student-result-text').textContent;
                
                message += `👤 *الطالب ${index + 1}:* ${studentName}\n`;
                message += `⭐ *الدرجة النهائية:* ${totalScore}/100\n`;
                message += `🏆 *التقدير:* ${grade}\n`;
                message += `────────────────────\n`;
            });
            
            message += `\n${customMessage}\n\n`;
            message += `🔸 *ملاحظة:* هذا التقرير تم إنشاؤه تلقائياً بواسطة نظام إدارة مشاريع التخرج العلمية`;
            
            document.getElementById('messagePreview').textContent = message;
        }

        function sendWhatsAppMessage() {
            const evaluatorName = document.getElementById('evaluatorName').value || 'المقيم';
            const phoneNumber = document.getElementById('phoneNumber').value;
            const project = currentProject;
            const roleName = roles[currentRole].roleName;
            const date = document.getElementById('evalDate').value;
            
            let message = `📋 *تقرير التقييم النهائي*%0A%0A`;
            message += `🎓 *نوع التقييم:* ${roleName}%0A`;
            message += `👨‍🏫 *اسم المقيم:* ${evaluatorName}%0A`;
            message += `📁 *اسم المشروع:* ${project.title}%0A`;
            message += `👨‍🏫 *المشرف الأكاديمي:* ${project.supervisor}%0A`;
            message += `🎓 *القسم العلمي:* ${getDepartmentName(project.department)}%0A`;
            message += `📅 *تاريخ التقييم:* ${date}%0A%0A`;
            message += `📊 *نتائج تقييم الطلاب:*%0A`;
            message += `══════════════════%0A%0A`;
            
            document.querySelectorAll('.student-card').forEach((card, index) => {
                const studentName = card.querySelector('.student-name-display').textContent;
                const totalScore = card.querySelector('.student-total-display').textContent;
                const grade = card.querySelector('.student-result-text').textContent;
                
                message += `👤 *الطالب ${index + 1}:* ${studentName}%0A`;
                message += `⭐ *الدرجة النهائية:* ${totalScore}/100%0A`;
                message += `🏆 *التقدير:* ${grade}%0A`;
                message += `────────────────────%0A`;
            });
            
            const customMessage = document.getElementById('customMessage').value;
            if (customMessage.trim()) {
                message += `%0A${customMessage}%0A%0A`;
            }
            
            message += `🔸 *ملاحظة:* هذا التقرير تم إنشاؤه تلقائياً بواسطة نظام إدارة مشاريع التخرج العلمية`;
            
            let whatsappURL = `https://wa.me/?text=${message}`;
            
            if (phoneNumber.trim()) {
                const cleanNumber = phoneNumber.replace(/\D/g, '');
                whatsappURL = `https://wa.me/${cleanNumber}?text=${message}`;
            }
            
            window.open(whatsappURL, '_blank');
            closeWhatsAppModal();
        }

        // تهيئة النظام
        document.addEventListener('DOMContentLoaded', function() {
            // تهيئة البيانات الأولية إذا لم تكن موجودة
            if (db.projects.length === 0) {
                db.projects.push({
                    id: 'proj_1',
                    title: "نظام ذكي لإدارة المكتبات الجامعية",
                    supervisor: "د. أحمد محمود",
                    department: "cs",
                    year: "2024",
                    semester: "1",
                    students: ["محمد خالد", "سارة أحمد", "عمر حسن"],
                    description: "نظام متكامل لإدارة عمليات المكتبات الجامعية باستخدام تقنيات الذكاء الاصطناعي",
                    createdAt: new Date().toISOString(),
                    status: "نشط"
                });
                
                db.projects.push({
                    id: 'proj_2',
                    title: "منصة التعليم الإلكتروني المتقدمة",
                    supervisor: "د. فاطمة علي",
                    department: "se",
                    year: "2024",
                    semester: "2",
                    students: ["خالد عبد الله", "نورة محمد"],
                    description: "منصة تعليمية تفاعلية تدعم التعلم الذاتي والتقييم الآلي",
                    createdAt: new Date().toISOString(),
                    status: "نشط"
                });
                
                saveDatabase();
            }
            
            updateStats();
            console.log('نظام إدارة مشاريع التخرج العلمية - النسخة المحسنة جاهزة للاستخدام');
        });
    </script>
</body>
</html>
