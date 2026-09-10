# Task 2: Windows Editions

## المحتوى
شرح الإصدارات المختلفة لنظام Windows (Home, Pro, Enterprise, Windows Server) والفروق الوظيفية والأمنية بينها.

## المفهوم الأمني

### الفروقات الأمنية بين الإصدارات
- إصدار **Home** لا يدعم ميزات حيوية للمؤسسات والأمان مثل:
  - **BitLocker**: تشفير القرص الكامل.
  - **Group Policy Objects (GPO)**: إدارة السياسات المركزية (كلمات مرور، صلاحيات، تحديثات...).
  - **Active Directory Domain Join**: الانضمام لنطاق مركزي تديره الشركة.
- إصدارا **Pro** و **Enterprise** مصممان لبيئات الأعمال، ويوفران هذه الأدوات للتحكم المركزي بالحسابات والأجهزة عبر الشبكة.

### إدارة بيئات المؤسسات
الشركات تعتمد على Pro/Enterprise لأن الأمان في بيئة متعددة الأجهزة يحتاج إدارة مركزية: تطبيق سياسة كلمات مرور موحدة، فرض تحديثات، تقييد صلاحيات المستخدمين — وكل هذا غير متاح في Home.

## شرح الأوامر العملية

### `systeminfo`
أمر CMD يعرض تقريراً كاملاً عن النظام: اسم الإصدار (OS Name)، رقم البناء (Build)، التاريخ، المعالج، الذاكرة، الـ Hotfixes المثبتة، وغيرها. من الناحية الأمنية، هذا الأمر من أول ما ينفذه المهاجم بعد الوصول لجهاز (Post-Exploitation Recon) لمعرفة نسخة النظام والثغرات المحتملة غير المرقّعة.

```cmd
systeminfo
```

### `Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsHardwareAbstractionLayer`
أمر PowerShell يقوم بما يلي خطوة بخطوة:
1. `Get-ComputerInfo` — يجلب كائن (Object) ضخم يحتوي على عشرات الخصائص عن النظام (نسخة الويندوز، BIOS، الذاكرة، الشبكة...).
2. الرمز `|` (Pipe) — يمرر ناتج الأمر الأول كمدخل للأمر التالي بدلاً من طباعته كاملاً.
3. `Select-Object WindowsProductName, WindowsVersion, OsHardwareAbstractionLayer` — يُصفّي الناتج ليعرض فقط ثلاث خصائص محددة: اسم الإصدار (مثلاً Windows 10 Pro)، رقم الإصدار، وطبقة تجريد العتاد (HAL).

هذا النمط (Cmdlet → Pipe → Select-Object) هو الأسلوب القياسي في PowerShell لاستخلاص بيانات محددة من ناتج ضخم، بدلاً من قراءة كل شيء يدوياً.

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsHardwareAbstractionLayer
```
