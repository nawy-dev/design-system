# Pattern: Android via Capacitor Shell

القرار: [ADR 0005](../DECISIONS/0005-android-capacitor-shell.md). نُفّذ أولًا في `nawy-dev/Taqweem` ثم `nawy-dev/nawy-do` (الأكثر تفصيلًا: `docs/ANDROID.md` هناك). هذا الملف الخلاصة القابلة لإعادة الاستخدام في أي تطبيق ناوي.
حالة التحقق: الخطوات مجرَّبة في CI وعلى محاكي أندرويد (API 34/35) في «ناوي دو». **لم تُجرَّب على هاتف حقيقي**، ولم يُرفع شيء إلى Play بعد.

## 1. المبدأ

مصدر ويب واحد ← `vite build` ← Capacitor يغلّفه. مشروع `android/` يُولَّد بـ`cap add android` في كل تشغيل CI ولا يُرفع. كل ما يخص الهاتف في `src/app/native/*` خلف واجهة وكشف `isNativeApp()`؛ المتصفح لا يتأثر.

## 2. ما يُنقل إلى تطبيق جديد

1. **تبعيات** (إصدارات مثبّتة): `@capacitor/core` و`@capacitor/local-notifications` و`@capacitor/filesystem` و`@capacitor/share`؛ و`@capacitor/cli` و`@capacitor/android` (dev). ADR في المنتج.
2. `capacitor.config.json` (`appId`، `appName`، `webDir: "dist"`، `plugins.LocalNotifications.smallIcon`)، و`/android/` في `.gitignore`.
3. `tools/android/`: `patch_manifest.py` (الأذونات)، `make_icons.py` (أيقونات من علامة المنتج)، `patch_gradle.py` (الإصدار والتوقيع من متغيرات البيئة)، `debug.keystore` ثابت.
4. Workflows: APK تجريبي، اختبار محاكي (يدوي)، إصدار AAB موقّع (يدوي).
5. `src/app/native/{env,plan,scheduler,useNativeAlerts,share}.ts` + ربط في `src/app` + اختبارات بمحاكاة الإضافات.
6. صفحة `public/privacy.html` (استثنِها من الخدمة العاملة) ونصوص المتجر.
نسخ الملفات من `nawy-do` أسرع من الكتابة من الصفر؛ عدّل `appId` والأسماء والأيقونة وخطوات الواجهة في سكربت المحاكي.

## 3. قواعد لا تُكسر

- لا ترفع `android/` ولا مفاتيح توقيع (إلا `debug.keystore` التجريبي). أسرار الإصدار: `ANDROID_KEYSTORE_BASE64` و`ANDROID_KEYSTORE_PASSWORD` و`ANDROID_KEY_ALIAS`.
- `appId` ثابت بعد أول رفع على Play، ولا يكون كلمة محجوزة في Java (`do`, `new`, `class`…).
- فُكّ أسرار التوقيع **بعد** `checkout` (يمحو مجلد العمل).
- لا تُرجع كائن إضافة Capacitor من `Promise`/`async` (وكيل يردّ على `then`)؛ لفّه في كائن عادي.
- الإضافات بـ`import()` ومن `src/app` فقط؛ لا من الواجهة أو النواة.
- لا تسجّل خدمة عاملة داخل الغلاف.
- الصدق: لا تعِد في الواجهة أو المتجر بـ«تنبيه والتطبيق مغلق» خارج الغلاف.
- `USE_EXACT_ALARM` للمنبّهات والتقاويم فقط؛ لغيرها `SCHEDULE_EXACT_ALARM` (يمنحه المستخدم) مع قبول تأخر دقائق.
- التصدير داخل WebView عبر `Filesystem` (مخبأ) + `Share`؛ لا `a[download]` ولا `navigator.share`.

## 4. التنبيهات (النمط)

دالة نقية تحسب الخطة (مهام مفتوحة بوقت، خلال 14 يومًا، حد 60، معرّف إشعار ثابت = تجزئة (معرّف + وقت))، ومحوّل `replaceAll` يلغي ما في الهاتف ويجدول الخطة عند كل تغيير. إذن `POST_NOTIFICATIONS` يُطلب بفعل من المستخدم. قناة بأهمية عالية.

## 5. التحقق على المحاكي (النمط)

`reactivecircus/android-emulator-runner` (API 34 و35، x86_64، KVM). السكربت: `adb install`، `pm grant POST_NOTIFICATIONS`، تشغيل، `adb forward tcp:9222 localabstract:webview_devtools_remote_<pid>`، ثم CDP بـWebSocket خام لقيادة الواجهة (ضبط الحقول عبر `Object.getPrototypeOf(el)` وأحداث `input`+`change`؛ اختر العناصر **الظاهرة**). أضف مهمة بعد دقيقتين، صدّر نسخة (تحقق من `run-as <pkg> ls cache` و`dumpsys window`)، `HOME` + `am kill`، انتظر، واقرأ `dumpsys notification`. النجاح = الهاتف عرض الإشعار والتطبيق بلا عملية.

## 6. مشاكل متكررة وحلولها

| المشكلة | الحل |
| --- | --- |
| `namespace`/`package` غير صالح في Gradle | `appId` بلا كلمة محجوزة (`todo.nawy.app` لا `do.nawy.app`) |
| تبعية جديدة تُفشل التثبيت (قفل مُولَّد في CI) | اسحب القفل من فرع `ci-lockfile` بعد انتهاء `ci` للـSHA نفسه وتأكد أنه يحوي التبعية |
| ميزانية الحجم | الأجزاء الاحتياطية للويب من إضافات Capacitor؛ ارفع `js`/`total` بسبب مكتوب لا `critical`، والأفضل إخراجها من precache |
| الخدمة العاملة تبتلع صفحة ثابتة (`privacy.html`) | `navigateFallbackDenylist` + اختبار |
| Lighthouse SEO على صفحة ثابتة | `meta description` |
| `workflow_dispatch` لملف جديد لا يعمل | يلزم وجوده في الفرع الافتراضي، أو شغّله بـ`push` على نمط فرع |
| خطأ lint لـ`new Date` أو ملف > حد الأسطر | مساعد الساعة المشترك؛ انقل الربط إلى hook |
| فشل عشوائي في تثبيت متصفح Playwright | أعد تشغيل الوظائف الفاشلة |
| سكربت المحاكي: ضغط على عنصر مخفي أو نافذة لم تُفتح | اختر الظاهر؛ سجّل حالة الصفحة وخذ لقطة عند كل خطوة |

## 7. على المالك

مفتاح الرفع وأسراره، حساب مطوّر Play (الحساب الشخصي الجديد: اختبار مغلق 12 مختبرًا × 14 يومًا)، سياسة الخصوصية وبطاقة المتجر وإجابات «أمان البيانات»، وتجربة على هاتف حقيقي (سامسونج: قد يحظر **Auto Blocker** أو Play Protect تثبيت APK التجريبي؛ يُعطَّلان مؤقتًا).
