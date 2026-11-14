# مِرصد / Terra Bytes — تشغيل الموقع في VS Code

هذا هو المشروع المحلي المستقل داخل المجلد الحالي. لا يحتاج خدمات Manus أو حساب AI، ولا يرفع كودك أو بياناتك. توجد صفحة رئيسية `/`، لوحة القياسات `/dashboard`، فهرس المصادر `/resources`، ودليل الملفات `/guide`. PostgreSQL وFastAPI مستقلان عن خادم الويب؛ البيانات تمر من الواجهة عبر `/api/agro` إلى FastAPI ثم PostgreSQL.

## افتح المشروع

من VS Code اختر File → Open Workspace from File وافتح `Terra-Bytes.code-workspace` الموجود في جذر المشروع. أو اختر Open Folder وافتح نفس المجلد. يوجد `.vscode/tasks.json` للتشغيل والفحص وحالة Docker؛ لا يحتوي أسرارًا. ملفات الواجهة في `client/src`، وخادم الموقع في `server`، وFastAPI ومخططات PostgreSQL في `services/agro`.

## التشغيل الكامل بـDocker

افتح Docker Desktop، ثم Terminal في جذر المشروع. احتفظ بملف `.env` الموجود عندك؛ لا تستبدله لأن بيانات volume الحالية مرتبطة بكلمة المرور الحالية. على نسخة جديدة لا تحتوي `.env` يمكنك إنشاء إعداد عشوائي آمن بـ:

```powershell
node scripts/setup-env.mjs
```

هذا الأمر يعمل مع Node الموجود لإنشاء الإعداد فقط، ولا يغيّر ملفًا موجودًا ولا يكشف كلمة المرور. أو أنشئ `.env` يدويًا من `.env.example` واستبدل placeholders بقيم خاصة قوية. لا تشارك الملف.

شغّل:

```powershell
docker compose -f docker-compose.integration.yml up --build -d
```

إذا لم يتعرف حساب Windows على `docker compose` فاستخدم النسخة المثبتة مباشرة:

```powershell
& 'C:\Program Files\Docker\Docker\resources\bin\docker-compose.exe' -f docker-compose.integration.yml up --build -d
```

اسم المشروع مثبت على `terrabytes` ليستخدم volume الموجود `terrabytes_agro_postgres`. لا تستخدم `down -v`؛ فهذا يحذف بيانات القاعدة. قد ينشئ Docker قاعدة جديدة فارغة على جهاز مختلف لا يملك هذا volume، وهو أمر طبيعي وليس نسخة من بيانات القرص القديمة.

بعد نجاح بناء وتشغيل الخدمات: الويب على `http://localhost:3000`، FastAPI docs على `http://localhost:8000/docs`، وPostgreSQL على `localhost:5432`. `/api/health` يفحص عملية الويب فقط، و`/api/agro/health` يفحص خدمة البيانات وPostgreSQL فعلًا. إذا كان البناء فاشلًا فلا توجد واجهة عاملة حتى لو كانت القاعدة شغالة.

على جهازك الحالي تم تحميل 409 سجلات بالفعل، لذلك لا تحتاج إعادة الاستيراد. على جهاز جديد، بعد نجاح تشغيل Backend:

```powershell
node scripts/import-bundle.mjs
```

الأداة تتحقق من بصمات JSON وتستورد POWER وSentinel-2 فقط إن كان جدول المصدر فارغًا. إذا كانت هناك قياسات تحافظ عليها وتتخطى الاستيراد. بقية ملفات JSON فارغة حين لا تتوفر قياسات؛ لا نملأها ببيانات مصطنعة.

## تطوير الواجهة دون حاوية Node

المشروع يحتاج Node **22.12 أو أحدث**؛ يوصى بسلسلة Node 22 LTS. النسخة المرصودة على جهازك `20.3.1` لا تكفي لـVite 7، ولم نغيرها أو نثبت بديلًا عالميًا. شغّل Node المناسب وpnpm 10.18.0، ثم:

```powershell
pnpm install --frozen-lockfile
pnpm dev
```

إذا لم يكن pnpm متاحًا، يمكن استخدام npm لجلب نفس إصدار مدير الحزم دون تثبيت عالمي:

```powershell
npm exec --yes --package=pnpm@10.18.0 -- pnpm install --frozen-lockfile
npm run dev
```

يلزم اتصال يسمح بتنزيل الاعتماديات. تشغيل dev يفتح Express/Vite على `127.0.0.1:3000`؛ لا يُشغّل PostgreSQL أو FastAPI تلقائيًا. شغّلهما فقط بـ:

```powershell
docker compose -f docker-compose.integration.yml up --build -d db agro
```

إذا كان Docker Dashboard يستخدم 3000 بالفعل، أوقف خدمة `dashboard` فقط قبل dev؛ لا توقف أو تحذف القاعدة. إعداد `.env.example` يربط Express بـFastAPI المحلي `http://127.0.0.1:8000`، بينما Compose يربطهما داخل الشبكة بـ`http://agro:8000` و`db:5432`.

## الفحص والتشخيص

```powershell
npm run doctor
npm run test:logic
npm run check
npm test
npm run build
npm run test:integration
```

اختبارات `test:logic` لا تحتاج node_modules وتعمل باستخدام Node الموجود. فحوص TypeScript وVitest والبناء تحتاج Node والاعتماديات. فحص `test:integration` للقراءة فقط ولا يزرع صفوفًا تجريبية؛ يحتاج الخدمات فعليًا. الاختبار الذي يفشل بسبب غياب خدمة أو اعتماديات ليس شهادة نجاح.

## استخدام الموقع

افتح Dashboard ثم اختر المصدر والسلسلة المكانية/المنتج والفترة. التاريخ الابتدائي يأتي من metadata للمصدر بدل سنة ثابتة. الصفحة تعرض القيم والوحدات والفترة وجودة الصفوف وraw JSON. لا تخلط المنتجات أو تقارن NDVI بكميات المطر، ولا تعتبر شهرًا ناقصًا مجموع هطول صالحًا. اختر المصدر من القائمة داخل Dashboard؛ روابط الفهرس الحالية تفتح اللوحة ولا تغيّر المصدر تلقائيًا.

تنزيل Dashboard يصدّر الصفوف المحمّلة مع بيان العدد الإجمالي والحدود، وليس كل القاعدة إذا كان extract محدودًا. صفحة `/guide` تعرض 34 ملفًا عامًا محددًا: 16 JSON موارد، 14 SQL، كتالوج وmanifest، وNotebook وHTML. ملفاتها لقطة الحزمة، وليست تصديرًا آنيًا لأي قياسات جديدة. لا يوجد endpoint لتنزيل `.env` أو مسار arbitrary.

الواجهة للقراءة فقط. جلب NASA POWER أو Sentinel-2 أو استيراد صفوف جديدة يستخدم API كتابة محميًا، ومفتاحه `AGRO_WRITE_TOKEN` من ملفك الخاص على الخادم. لا تضعه في React ولا في `VITE_*`. توثيق المصادر الأخرى يبين أن معظمها يحتاج معالجة وحسابات ومنتجات، وليس API لحظيًا جاهزًا.

## حالة التحقق لهذه النسخة

PostgreSQL الحالي يعمل و409 صفوف كانت قد طابقت JSON الأصلي؛ الأدلة محفوظة في `outputs/local_docker_setup` على جهازك. ثمانية اختبارات منطق/فهرس ملفات نجحت محليًا للنسخة الجديدة. أضيفت اختبارات proxy وDashboard وFastAPI ومراجعة مستقلة للكود؛ فحص Python syntax نجح في مهمة المراجعة، لكن unittest للـBackend تخطّى التنفيذ بسبب اعتماد psycopg غير المتاح.

لا تزال تنزيلات npm وNode وصور التطبيق تفشل بانقطاع اتصال، وnode_modules غير موجود. لذلك **لم يتأكد بناء React/TypeScript الكامل أو العرض البصري أو تشغيل الصفحات الأربع وproxy المعدل على Windows**. لا توجد صورة Dashboard أو نتيجة browser اختبار نزعم نجاحها. تفاصيل الفحوص والقيود في `outputs/website_release/VERIFICATION_AR.md`. أصلح الوصول إلى السجلات واستخدم الأوامر أعلاه قبل اعتماد هذه النسخة للإنتاج. لا نشر عام ولا خدمة مجدولة ولا AI متصل في هذه الحزمة.
