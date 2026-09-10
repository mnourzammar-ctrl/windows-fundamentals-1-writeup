# Task 8: Settings and the Control Panel

## السؤال والإجابة
- **السؤال:** *In the Control Panel, change the view to Small icons. What is the last setting in the Control Panel view?*
- **الإجابة:** `Windows Defender Firewall`

## المفهوم الأمني

### التحكم بالشبكة المحلية
يميّز Windows الحديث بين واجهتين لإدارة الإعدادات:
- **Settings** (الحديثة، أبسط وأكثر ودّية للمستخدم العادي).
- **Control Panel** (التقليدية، وتحتوي إعدادات متقدمة لا تزال غير متوفرة بالكامل في Settings، مثل بعض إعدادات الجدار الناري المتقدمة والشبكة).
معرفة كلتا الواجهتين مهمة لأن بعض الإعدادات الأمنية الحرجة (كقواعد الجدار الناري المتقدمة) لا يمكن الوصول إليها إلا عبر Control Panel أو أدوات مخصصة مثل `wf.msc`.

### الحد من التحرك الجانبي (Lateral Movement)
استخدام الجدار الناري (Windows Defender Firewall) لحظر منافذ حرجة معروفة باستغلالها في الانتشار داخل الشبكة:
- **445 (SMB)**: البروتوكول الذي استُغل في هجمات ضخمة مثل WannaCry وNotPetya للانتشار التلقائي بين الأجهزة.
- **3389 (RDP)**: بروتوكول سطح المكتب البعيد، هدف شائع لهجمات القوة الغاشمة (Brute Force) والوصول غير المصرَّح به.
حظر هذه المنافذ على مستوى الشبكة الداخلية (وليس فقط من الإنترنت) يقلل بشكل كبير من قدرة برمجية خبيثة على الانتشار من جهاز مصاب إلى بقية الشبكة (Lateral Movement).

## شرح الأوامر العملية

### `control firewall.cpl`
يفتح مباشرة لوحة تحكم **Windows Defender Firewall** التقليدية (ملف `.cpl` هو ملف تنفيذي خاص بعناصر Control Panel). طريقة سريعة للوصول إلى إعدادات الجدار الناري دون التنقل يدوياً بين القوائم.

```cmd
control firewall.cpl
```

### `control /name Microsoft.ControlPanel`
يفتح Control Panel بأكمله (الصفحة الرئيسية) باستخدام الاسم الكانوني (Canonical Name) الخاص به بدلاً من فتح عنصر فرعي محدد. مفيد كنقطة بداية عامة للتنقل بين كل الإعدادات.

```cmd
control /name Microsoft.ControlPanel
```

### `Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction`
أمر PowerShell لفحص حالة الجدار الناري برمجياً، خطوة بخطوة:
1. `Get-NetFirewallProfile` — يجلب إعدادات كل ملف تعريف شبكة (Firewall Profile) موجود: **Domain**، **Private**، **Public**.
2. `Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction` — يعرض فقط: اسم الملف الشخصي، هل الجدار الناري مفعّل فيه، والسلوك الافتراضي للاتصالات الواردة والصادرة (سماح/حظر).

هذا الأمر أساسي في التدقيق الأمني السريع: التأكد أن الجدار الناري مفعّل في كل الملفات الثلاثة، وأن السلوك الافتراضي للاتصالات الواردة هو الحظر (وليس السماح).

```powershell
Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

### `New-NetFirewallRule -DisplayName "Block SMB Inbound" -Direction Inbound -LocalPort 445 -Protocol TCP -Action Block`
أمر ينشئ قاعدة جدار ناري جديدة برمجياً، تفكيك المعاملات (Parameters):
- `-DisplayName "Block SMB Inbound"` — اسم وصفي للقاعدة يظهر لاحقاً في قائمة القواعد.
- `-Direction Inbound` — تنطبق القاعدة على الاتصالات **الواردة** إلى الجهاز فقط (وليس الصادرة منه).
- `-LocalPort 445` — تستهدف تحديداً المنفذ 445 (بروتوكول SMB) على هذا الجهاز.
- `-Protocol TCP` — تنطبق على بروتوكول TCP تحديداً.
- `-Action Block` — الإجراء عند تطابق القاعدة هو **الحظر الكامل** لتلك الاتصالات.

نتيجة الأمر: أي محاولة اتصال واردة على المنفذ 445 عبر TCP سيتم حظرها فوراً على مستوى الجهاز — إجراء دفاعي مباشر لمنع استغلال ثغرات SMB (مثل EternalBlue) أو منع دودة تنتشر عبر هذا البروتوكول من الوصول لهذا الجهاز.

```powershell
New-NetFirewallRule -DisplayName "Block SMB Inbound" -Direction Inbound -LocalPort 445 -Protocol TCP -Action Block
```
