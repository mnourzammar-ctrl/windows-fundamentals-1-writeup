# Windows Fundamentals 1 — TryHackMe Documentation

توثيق تفصيلي لغرفة **Windows Fundamentals 1** على منصة TryHackMe، يغطي المفاهيم الأمنية والأوامر العملية لكل مهمة (Task)، مع شرح موسّع لكل خطوة وكل أمر/سكريبت مستخدم.

## البنية (Structure)

```
windows-fundamentals-1-repo/
├── README.md                  ← هذا الملف
└── tasks/
    ├── 01-introduction.md
    ├── 02-windows-editions.md
    ├── 03-desktop-gui.md
    ├── 04-file-system.md
    ├── 05-system32-folder.md
    ├── 06-user-accounts.md
    ├── 07-user-account-control.md
    ├── 08-settings-control-panel.md
    └── 09-task-manager.md
```

## نظرة عامة على الغرفة

غرفة **Windows Fundamentals 1** هي مدخل تعريفي بنظام التشغيل Windows من منظور أمني: بيئة العمل الرسومية، نظام الملفات NTFS، الحسابات والصلاحيات، آلية حماية UAC، أدوات التحكم (Control Panel/Settings)، ومراقبة العمليات عبر Task Manager. كل مهمة تربط المفهوم الأساسي (كيف يعمل النظام) بالزاوية الأمنية (كيف يُستغل أو يُراقَب من منظور مهاجم/مدافع).

## جدول المهام

| # | المهمة | المحور الأمني الأساسي |
|---|--------|------------------------|
| 1 | Introduction | بيئة الاختبار المعزولة |
| 2 | Windows Editions | فروقات الإصدارات وتأثيرها الأمني (BitLocker/GPO/AD) |
| 3 | Desktop (GUI) | الوصول السريع للأدوات أثناء الاستجابة للحوادث |
| 4 | The File System | صلاحيات NTFS وACLs |
| 5 | Windows\System32 | LOLBINs وDLL Hijacking |
| 6 | User Accounts | PoLP وقاعدة بيانات SAM |
| 7 | User Account Control | Split Token وBypass UAC |
| 8 | Settings & Control Panel | الجدار الناري ومنع Lateral Movement |
| 9 | Task Manager | Threat Hunting وProcess Masquerading |

كل ملف داخل `tasks/` يحتوي على: شرح المفهوم، الأسئلة والإجابات (إن وجدت)، وشرح تفصيلي سطراً بسطر لكل أمر/سكريبت مستخدم — ماذا يفعل، ولماذا هو مهم أمنياً.
