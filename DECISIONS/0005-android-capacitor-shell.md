# ADR 0005: Android via a Capacitor Shell Around the Web App

- **Status:** Accepted for Nawy web apps that need an Android release (first used by Nawy To Do, copied from Nawy Today)
- **Date:** 2026-11-22

## Context

منتجات ناوي ويب (Vite + React، محلية أولًا). المتصفح يقيّد أمورًا يحتاجها المستخدم على الهاتف: تنبيهات والتطبيق مغلق، تنزيل/مشاركة ملفات، حضور في متجر Play. الإصدار الأول لهذا النمط كان في «تقويم» (`nawy-dev/Taqweem`) ثم نُقل إلى «ناوي دو» (`nawy-dev/nawy-do`، ADR 0045 و0046 هناك).

## Decision

- يبقى **مصدر الويب واحدًا**. تغليف أندرويد بـ**Capacitor** (`webDir` = مخرجات `vite build`)، ومشروع `android/` **يُولَّد في CI ولا يُرفع**.
- كل كود الهاتف في `src/app/native/*` خلف واجهة وكشف `isNativeApp()`؛ الإضافات تُحمَّل بـ`import()` وتُغلَّف بكائن عادي (فخ `then` في وكيل الإضافات).
- التبعيات الأصلية الافتراضية: `@capacitor/core|cli|android` و`@capacitor/local-notifications` و`@capacitor/filesystem` و`@capacitor/share` (MIT، إصدارات مثبّتة). غيرها بقرار جديد.
- التوقيع: مفتاح تجريبي **ثابت** مرفوع للـAPK التجريبي، ومفتاح رفع Play **أسرار مستودع** لا يُرفع.
- التحقق بمحاكي أندرويد في CI (workflow يدوي) لا ادعاء من الكود وحده.
- التفاصيل والمشاكل المجرَّبة وحلولها: [`patterns/android-capacitor-shell.md`](../patterns/android-capacitor-shell.md).

## Alternatives

- Trusted Web Activity (PWA في Play بلا غلاف): لا تنبيهات أصلية والتطبيق مغلق، وقيود المشاركة نفسها.
- Web Push بخادم: يناقض المحلي أولًا (ADR 0003).
- تطبيق أصلي منفصل (Kotlin/Flutter): مصدران لكل منتج.
- رفع مجلد `android/` للمستودع: ضجيج وتعارضات؛ التوليد في CI يكفي.

## Consequences

- **منافع:** مصدر واحد، تنبيهات حقيقية، حضور في المتجر، كل شيء قابل للإعادة من CI.
- **تكاليف:** تبعيات أصلية وأثر على حجم الحزمة (أجزاء ويب احتياطية)، ومسار توقيع ومتجر على المالك.
- **مخاطر:** أصل مختلف (`https://localhost`) فبيانات PWA المثبّت لا تنتقل إلا بتصدير/استيراد؛ سياسات Play للأذونات (`SCHEDULE_EXACT_ALARM`)؛ لم يُتحقق على أجهزة حقيقية ولا من توفير الطاقة عند الصانعين.
- هذا القرار لا يثبت أن تطبيقًا بعينه جاهز للنشر؛ كل تطبيق يوثّق حالته في `docs/ANDROID.md` عنده.

## Evidence

- `nawy-dev/nawy-do`: `docs/ANDROID.md`، `docs/decisions/0045-android-capacitor.md`، `0046-android-files-share.md`، workflows `android-apk|android-emulator|android-release`.
- `nawy-dev/Taqweem`: `docs/ANDROID_PLAYBOOK.md`.
