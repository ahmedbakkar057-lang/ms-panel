# MS VPS — Railway و Docker

## الملفات

- `vps_panel.py`: تطبيق Flask الرئيسي.
- `requirements.txt`: الاعتماديات المطلوبة.
- `Procfile` و`railway.json`: إعدادات البناء والتشغيل على Railway.
- `Dockerfile`: صورة Python جاهزة للتشغيل بالحاويات.
- `docker-compose.yml`: تشغيل محلي مع Volume دائم لبيانات اللوحة.
- `.dockerignore`: يستبعد البيانات المؤقتة والملفات السرية من صورة Docker.

## التشغيل باستخدام Docker Compose

```bash
docker compose up -d --build
```

بعدها افتح:

```text
http://localhost:20276
```

لإيقاف الخدمة:

```bash
docker compose down
```

يتم حفظ بيانات اللوحة في Volume باسم `panel_data`، لذلك لا تضيع عند إعادة إنشاء الحاوية. لوحة المالك مقيدة بإنشاء حسابات المستخدمين فقط؛ تغيير كلمة مرور المالك وبقية وظائف الإدارة مغلقة من جهة الخادم. بوت تيليجرام المدمج لا يقبل إلا رسائل خاصة من `TELEGRAM_ADMIN_ID`، ويتيح `/create USERNAME` فقط. عيّن `TELEGRAM_BOT_TOKEN` كمتغير بيئة، ولا تضع التوكن في الملف أو Git أو الصورة. استخدم نسخة جديدة من التوكن بعد تدويره، وشغّل نسخة واحدة فقط من التطبيق حتى لا تتعارض عملية polling.

## التشغيل باستخدام Docker مباشرة

```bash
docker build -t ms-vps-panel .
docker run -d --name ms-vps-panel \
  -p 20276:20276 \
  -e RUN_BOT=1 \
  -e TELEGRAM_BOT_TOKEN="$TELEGRAM_BOT_TOKEN" \
  -e TELEGRAM_ADMIN_ID="${TELEGRAM_ADMIN_ID:-8122328795}" \
  -v ms-vps-panel-data:/app/panel_data \
  ms-vps-panel
```

يمكن تغيير المنفذ الخارجي، مع إبقاء `PORT` مساويًا للمنفذ الداخلي أو تمريره صراحةً:

```bash
docker run -d --name ms-vps-panel \
  -p 8080:20276 \
  -e PORT=20276 \
  -v ms-vps-panel-data:/app/panel_data \
  ms-vps-panel
```

## النشر على Railway

1. ارفع **محتويات المجلد** إلى المستودع، وليس ملف ZIP داخل المستودع. يجب أن يكون `Dockerfile` في جذر المستودع وبنفس الأحرف الكبيرة والصغيرة.
2. اترك `railway.json` في الجذر؛ فهو يحدد `DOCKERFILE` و`Dockerfile` صراحةً.
3. في Railway اختر المستودع ثم نفّذ Redeploy. يجب أن يظهر في سجل البناء ما يشبه `Using detected Dockerfile!`.
4. أضف المتغيرات عند الحاجة:
   - `RUN_BOT=1` لتشغيل البوت المدمج (يمكن ضبطه إلى `0` لإيقافه).
   - `TELEGRAM_BOT_TOKEN` كتخزين سري لمتغير البيئة، ويجب أن يكون التوكن الجديد بعد تدوير القديم.
   - `TELEGRAM_ADMIN_ID=8122328795` أو معرّف حساب تيليجرام الإداري المصرح به.
   - `BASE_PATH=/app/panel_data` اختياري؛ تخزين Railway المحلي مؤقت ما لم تستخدم Volume أو قاعدة بيانات.
5. Railway يضبط `PORT` تلقائيًا، والتطبيق يستمع على `0.0.0.0`.

## ملاحظات مهمة

- لا تضع Bot Token أو كلمات المرور داخل GitHub أو داخل ملفات عامة.
- البوت مدمج داخل `vps_panel.py` ويحتاج إلى متغيرات البيئة المذكورة أعلاه لتشغيله.
- بيانات المستخدمين وملفات اللوحة تُنشأ داخل `panel_data/`، وقد تضيع عند إعادة النشر على Railway بدون Volume.

## الأوامر والصلاحيات بعد التعديل
- حساب المالك في الويب يعرض إنشاء المستخدم فقط؛ جميع واجهات تغيير إعدادات المالك/كلمة مروره/التحكم الأخرى مرفوضة من الخادم.
- بوت تيليجرام يقبل من المعرّف المحدد وفي المحادثة الخاصة فقط: `/start` و`/help` و`/create USERNAME`. كلمة مرور الحساب تُولّد عشوائياً وتُرسل مرة واحدة للمحادثة الخاصة.
- لا تستخدم التوكن الذي سبق إرساله في محادثة؛ ألغِه عبر BotFather وأنشئ توكناً جديداً، ثم خزّنه في متغير `TELEGRAM_BOT_TOKEN` لدى Railway أو ملف `.env` محلي غير مرفوع.
- لملفات Docker Compose، صدّر `TELEGRAM_BOT_TOKEN` و`TELEGRAM_ADMIN_ID` في بيئة التشغيل قبل `docker compose up -d --build`.
