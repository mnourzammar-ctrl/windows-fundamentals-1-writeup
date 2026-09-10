# Task 6: User Accounts, Profiles, and Permissions

## الأسئلة والإجابات
1. **What is the name of the other user account?** → `tryhackmebilly`
2. **What groups is this user a member of?** → `Remote Desktop Users, Users`
3. **What built-in account is for guest access to the computer?** → `Guest`
4. **What is the account description?** → `Built-in account for guest access to the computer/domain`

## المفهوم الأمني

### مبدأ أقل الصلاحيات (Principle of Least Privilege — PoLP)
الفصل بين حسابات المسؤولين (**Administrator**) والمستخدمين القياسيين (**Standard**) يقلل مساحة الهجوم (Attack Surface): إذا تم اختراق حساب مستخدم قياسي، فإن الضرر المحتمل محدود بصلاحياته، بعكس اختراق حساب مسؤول الذي يمنح المهاجم تحكماً كاملاً بالجهاز فوراً.

### قاعدة بيانات SAM (Security Account Manager)
تُخزَّن هاشات (Hashes) كلمات مرور الحسابات المحلية في الملف `C:\Windows\System32\config\SAM`. هذا الملف محمي أثناء عمل النظام (لا يمكن نسخه مباشرة وهو مقفل)، لكنه هدف رئيسي لأدوات مثل **Mimikatz** التي تستخرج الهاشات من ذاكرة عملية `lsass.exe` أو من نسخ الظل (Shadow Copies) لاستخدامها لاحقاً في هجمات Pass-the-Hash أو كسر كلمات المرور Offline.

### الحسابات المدمجة (Built-in Accounts)
فهم دور الحسابات الخاصة المدمجة في Windows مهم لتمييز السلوك الطبيعي عن المشبوه:
- **Guest**: حساب ضيف محدود الصلاحيات، مُعطَّل افتراضياً في النسخ الحديثة.
- **WDAGUtilityAccount**: حساب خاص يُستخدم مع ميزة **Windows Defender Application Guard** (المعزل الأمني) لتشغيل المتصفح داخل بيئة معزولة عند فتح مواقع غير موثوقة.

## شرح الأوامر العملية

### `net user`
يعرض قائمة بكل حسابات المستخدمين المحليين المعرّفة على الجهاز. خطوة استكشافية أساسية (Enumeration) سواء للمسؤول (لمراجعة الحسابات) أو للمهاجم بعد الوصول الأولي (لمعرفة الحسابات المتاحة والبحث عن أهداف لتصعيد الصلاحيات).

```cmd
net user
```

### `net user tryhackmebilly`
نفس الأمر السابق لكن موجَّه لحساب محدد (`tryhackmebilly`)، فيعرض تفاصيله الكاملة: تاريخ الإنشاء، آخر دخول، المجموعات التي ينتمي إليها، وحالة الحساب (مفعّل/معطّل).

```cmd
net user tryhackmebilly
```

### `net localgroup Administrators`
يعرض كل أعضاء مجموعة **Administrators** المحلية. من أهم الأوامر أمنياً — سواء للتدقيق (التأكد أن لا أحد غير مصرَّح له عضو في هذه المجموعة) أو للمهاجم (لمعرفة من يملك صلاحيات كاملة على الجهاز كهدف للانتحال أو التصعيد).

```cmd
net localgroup Administrators
```

### `whoami /priv`
يعرض كل الامتيازات (**Privileges**) المرتبطة بالمستخدم الحالي في جلسته الحالية (مثل `SeDebugPrivilege` أو `SeImpersonatePrivilege`). هذه الامتيازات غالباً ما تكون طريق تصعيد الصلاحيات — فامتياز واحد مفعَّل بالخطأ لمستخدم عادي قد يُستغل بالكامل للوصول إلى صلاحيات SYSTEM.

```cmd
whoami /priv
```

### `Get-LocalUser | Select-Object Name, Enabled, LastLogon`
معادل PowerShell لـ `net user` لكن بمخرجات أكثر قابلية للمعالجة البرمجية:
1. `Get-LocalUser` — يجلب كل الحسابات المحلية ككائنات (Objects) وليس نصاً مسطحاً.
2. `Select-Object Name, Enabled, LastLogon` — يُصفّي الأعمدة المعروضة إلى ثلاثة فقط: الاسم، حالة التفعيل، وآخر دخول — وهي أهم ثلاث معلومات لتدقيق سريع للحسابات المشبوهة (حساب مفعّل لم يُستخدم منذ فترة طويلة مثلاً).

```powershell
Get-LocalUser | Select-Object Name, Enabled, LastLogon
```

### `Get-LocalGroupMember -Group "Remote Desktop Users"`
يعرض أعضاء مجموعة محلية محددة — هنا **Remote Desktop Users**، وهي مجموعة حساسة لأن عضويتها تعني إمكانية تسجيل الدخول عن بعد عبر RDP. مراجعة هذه القائمة مهمة لأن أي حساب مُضاف إليها بدون مبرر (خصوصاً بعد اختراق) يمنح المهاجم وصولاً دائماً عن بعد (Persistence).

```powershell
Get-LocalGroupMember -Group "Remote Desktop Users"
```
