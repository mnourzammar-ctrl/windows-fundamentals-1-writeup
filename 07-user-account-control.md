# Task 7: User Account Control (UAC)

## السؤال والإجابة
- **السؤال:** *What does UAC mean?*
- **الإجابة:** `User Account Control`

## المفهوم الأمني

### معمارية الرموز المزدوجة (Split Token Architecture)
عندما يسجّل مستخدم من مجموعة Administrators دخوله، لا يحصل مباشرة على صلاحياته الكاملة. بدلاً من ذلك يُنشئ Windows رمزين (Tokens):
- **Filtered/Standard Token**: يُستخدم افتراضياً لكل العمليات العادية (تصفح، فتح ملفات...) بصلاحيات مستخدم قياسي.
- **Elevated Token**: يُفعَّل فقط عند الموافقة على طلب UAC (نافذة "هل تريد السماح لهذا التطبيق...")، ويُستخدم لتلك العملية المحددة فقط.
هذا الفصل يمنع تشغيل كل شيء بصلاحيات كاملة طوال الوقت، فيقلل من الضرر المحتمل لو تم تشغيل برنامج خبيث بالخطأ.

### مستويات التكامل (Mandatory Integrity Control — MIC)
آلية إضافية تمنع العمليات ذات مستوى الثقة الأقل (**Medium Integrity**، وهي المستوى الافتراضي لمعظم البرامج) من التعديل على عمليات ذات مستوى أعلى (**High** أو **System**)، حتى لو كانا يعملان تحت نفس المستخدم. هذا يمنع برنامجاً خبيثاً يعمل بصلاحيات عادية من حقن كود في عملية نظام حساسة مباشرة.

### تقنيات تجاوز UAC (UAC Bypass)
بعض برامج Windows الموثوقة مُعلَّمة بخاصية `autoElevate = true`، بمعنى أنها تُرفَّع صلاحياتها تلقائياً دون إظهار نافذة تأكيد UAC (لأن Microsoft تثق بها مسبقاً). المهاجمون يستغلون هذه البرامج (عبر حقن أو استبدال DLL أو خداع مسار تنفيذ) لتشغيل كود خبيث بصلاحيات مرتفعة دون أن يظهر أي تنبيه للمستخدم — وهي من أشهر أساليب تصعيد الصلاحيات في Windows.

## شرح الأوامر العملية

### `reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v EnableLUA`
تفكيك الأمر:
1. `reg query` — أمر CMD لقراءة قيم من سجل النظام (**Registry**).
2. `"HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System"` — المسار (Key) داخل الـ Registry الذي يحتوي إعدادات سياسة UAC.
3. `/v EnableLUA` — يحدد القيمة (Value) المطلوب قراءتها تحديداً: `EnableLUA`، وهي المفتاح الذي يتحكم بتفعيل/تعطيل UAC بالكامل على مستوى النظام (`1` = مفعّل، `0` = معطّل).

هذا الأمر يُستخدم للتحقق مما إذا كانت حماية UAC مفعّلة على جهاز معين — فتعطيلها يدوياً (قيمة 0) يُعد علامة حمراء أثناء التدقيق الأمني، لأنه يعني أن كل البرامج تعمل بصلاحيات كاملة دون أي طبقة تأكيد.

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v EnableLUA
```

### `[Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent() | Select-Object -ExpandProperty Claims`
أمر PowerShell أكثر تقدماً يتعامل مباشرة مع كائنات .NET، خطوة بخطوة:
1. `[Security.Principal.WindowsIdentity]::GetCurrent()` — يستدعي دالة ثابتة (Static Method) من مكتبة .NET تُرجع هوية المستخدم الحالي في الجلسة الحالية (اسمه، الرمز/Token الخاص به).
2. `[Security.Principal.WindowsPrincipal]` — يُحوِّل (Cast) تلك الهوية إلى كائن "Principal"، وهو تمثيل أوسع يتضمن معلومات الأدوار والصلاحيات المرتبطة بها.
3. `| Select-Object -ExpandProperty Claims` — يمرر الكائن الناتج، ثم يستخرج خاصية `Claims` تحديداً ويعرضها موسّعة (Expanded) بدلاً من عرضها كخاصية متداخلة مضغوطة. الـ Claims هنا هي قائمة "الادعاءات" أو الخصائص الأمنية المرتبطة بهوية المستخدم (مثل عضوية المجموعات، مستوى التكامل، وغيرها من بيانات الرمز الأمني).

هذا الأمر يُستخدم لفحص تفصيلي عميق لهوية المستخدم الحالي وصلاحياته الفعلية في الجلسة، بشكل أدق من أوامر CMD التقليدية.

```powershell
[Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent() | Select-Object -ExpandProperty Claims
```
