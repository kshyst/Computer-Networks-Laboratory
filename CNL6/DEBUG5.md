# DEBUG5 — دیباگ شکست همه pingهای بند ۵

## دامنه

این فایل فقط شکست pingهای بند ۵ آزمایش ششم را بررسی می‌کند:

```text
ping 213.80.11.6
ping 192.168.0.2
```

هدف این برنامه دیباگ، پیدا کردن دقیق محل قطع‌شدن مسیر رفت یا برگشت است. هیچ تنظیمی قبل از اثبات علت اصلی به‌عنوان راه‌حل دائمی تغییر داده نمی‌شود.

## نتیجه محتمل پیش از شروع دیباگ

با baseline فعلی walkthrough، شکست هر دو ping در بند ۵ **به احتمال زیاد رفتار مورد انتظار همین مرحله** است، نه خرابی توپولوژی:

- روی `Internet` نگاشت Static NAT زیر وجود دارد:

```text
192.168.0.2 <-> 213.80.11.6
```

- روی `R5` تنظیمات source NAT آزمایش ۵ عمداً پاک شده‌اند و Dynamic NAT هنوز تا بندهای ۷ تا ۱۲ ساخته نشده است.
- روتر `Internet` طبق شرط آزمایش هیچ route به شبکه‌های داخلی `10.10.*` ندارد.
- درخواست عمومی ممکن است از `PC1` تا `Server0` برسد، اما پاسخ برای مقصد `10.10.6.1` در `Internet` مسیر برگشت ندارد.
- ping مستقیم `192.168.0.2` نیز همین مشکل مسیر برگشت را دارد و علاوه بر آن، پاسخ Server0 در عبور inside-to-outside می‌تواند با source عمومی `213.80.11.6` خارج شود.

پس ابتدا باید با Simulation ثابت شود بسته دقیقاً کجا drop می‌شود. فقط بعد از آن می‌توان بین «رفتار طراحی‌شده بند ۵» و «خطای پیکربندی» تفاوت گذاشت.

## وضعیت مرجع صحیح

| مورد | مقدار صحیح |
|---|---|
| `PC1` | `10.10.6.1/24` |
| gateway مربوط به `PC1` | `10.10.6.2` |
| `R5 Fa1/0` | `213.80.11.4/24` |
| default route روی `R5` | next hop برابر `213.80.11.5` |
| `Internet Fa0/0` | `213.80.11.5/24` و NAT outside |
| `Internet Fa0/1` | `192.168.0.1/24` و NAT inside |
| `Server0` | `192.168.0.2/24` |
| gateway مربوط به `Server0` | `192.168.0.1` |
| Static NAT روی `Internet` | `192.168.0.2` به `213.80.11.6` |
| routeهای خصوصی روی `Internet` | نباید route به `10.10.*` وجود داشته باشد |
| source NAT روی `R5` در بند ۵ | هنوز نباید فعال باشد |

---

# فاز ۱ — بازتولید کنترل‌شده

## ۱. یک نسخه دیباگ جدا ذخیره کنید

فایل فعلی Packet Tracer را با `File > Save As` به نام زیر ذخیره کنید:

```text
KiarashShojaei-6-2-6-DEBUG5.pkt
```

روی فایل اصلی آزمایش تغییر تشخیصی دائمی انجام ندهید.

## ۲. Simulation را آماده کنید

1. از پایین Packet Tracer وارد `Simulation` شوید.
2. در `Edit Filters` فقط `ARP` و `ICMP` را فعال نگه دارید.
3. Event List را پاک کنید.
4. روی `Internet` بلوک زیر را paste کنید:

```text
enable
clear ip nat translation *
show ip nat translations
show ip nat statistics
```

نگاشت static باید بعد از پاک‌کردن translationهای موقت همچنان در configuration و جدول قابل مشاهده باشد.

---

# فاز ۲ — بررسی مسیر به‌ترتیب از مبدأ تا مقصد

## ۳. تنظیمات محلی PC1 را بررسی کنید

در `PC1 > Desktop > Command Prompt` بلوک زیر را paste کنید:

```text
ipconfig
ping 10.10.6.2
```

### تفسیر

- اگر ping به `10.10.6.2` شکست خورد، هنوز وارد بحث NAT نشوید.
- IP، mask، gateway، کابل `PC1-SW1-R1` و وضعیت واسط R1 را اصلاح کنید.
- اگر موفق شد، لایه محلی PC1 سالم است و مرحله بعد را اجرا کنید.

> **Screenshot checkpoint — `D5-01-pc1-local-baseline.png`:** بلافاصله پس از اجرای `ipconfig` و ping gateway تصویر بگیرید. IP، mask، gateway و نتیجه کامل ping باید دیده شوند.

## ۴. مسیر داخلی تا R5 را بررسی کنید

در `PC1 > Desktop > Command Prompt` اجرا کنید:

```text
ping 10.10.11.2
ping 213.80.11.4
```

### تفسیر

- شکست `10.10.11.2` یعنی مسیر داخلی یا OSPF تا R5 مشکل دارد.
- موفقیت `10.10.11.2` و شکست `213.80.11.4` معمولاً به route، interface یا ACL محلی R5 مربوط است.
- موفقیت هر دو یعنی PC1 تا خود R5 مسیر رفت و برگشت سالم دارد.

> **Screenshot checkpoint — `D5-02-pc1-to-r5-path.png`:** پس از تمام‌شدن هر دو ping و پیش از رفتن سراغ CLI روترها تصویر بگیرید. هر دو مقصد و نتایج کامل باید دیده شوند.

## ۵. وضعیت route و interfaceهای R5 را بررسی کنید

روی `R5` بلوک زیر را paste کنید:

```text
enable
show ip interface brief
show ip route 10.10.6.0
show ip route 0.0.0.0
show ip nat statistics
show ip nat translations
```

### وضعیت صحیح

- `Fa0/0`، `Fa0/1` و `Fa1/0` باید `up/up` باشند.
- route شبکه `10.10.6.0/24` باید به سمت شبکه داخلی وجود داشته باشد.
- default route باید به `213.80.11.5` اشاره کند.
- در بند ۵، association مربوط به Dynamic NAT یا PAT روی R5 نباید فعال باشد.

> **Screenshot checkpoint — `D5-03-r5-routing-and-nat-state.png`:** بلافاصله پس از بلوک بالا تصویر بگیرید. route داخلی، default route و وضعیت خالی یا غیرفعال NAT روی R5 باید خوانا باشند؛ در صورت نیاز از فایل‌های `-a` و `-b` استفاده کنید.

## ۶. لینک R5 تا Internet را مستقیم بررسی کنید

روی `R5` اجرا کنید:

```text
enable
ping 213.80.11.5
```

### تفسیر

- اگر این ping شکست خورد، مشکل در NAT نیست؛ لینک `R5-SW4-Internet`، IPهای `.4` و `.5`، mask یا وضعیت interface را اصلاح کنید.
- اگر موفق شد، مرز خارجی بین دو روتر سالم است.

> **Screenshot checkpoint — `D5-04-r5-to-internet-link.png`:** بعد از ping مستقیم R5 به Internet تصویر بگیرید. مقصد و نتیجه کامل باید مشخص باشند.

## ۷. وضعیت Internet و Static NAT را بررسی کنید

روی `Internet` بلوک زیر را paste کنید:

```text
enable
show ip interface brief
show ip route
show ip nat statistics
show ip nat translations
```

### وضعیت صحیح

- `Fa0/0 = 213.80.11.5` و `Fa0/1 = 192.168.0.1` باید `up/up` باشند.
- `FastEthernet0/0` باید در `Outside interfaces` باشد.
- `FastEthernet0/1` باید در `Inside interfaces` باشد.
- نگاشت `192.168.0.2 <-> 213.80.11.6` باید دیده شود.
- routeهای connected برای `213.80.11.0/24` و `192.168.0.0/24` باید وجود داشته باشند.
- نباید route به `10.10.0.0/16` یا `10.10.6.0/24` وجود داشته باشد.

> **Screenshot checkpoint — `D5-05-internet-routes-and-static-nat.png`:** بلافاصله پس از بلوک بالا تصویر بگیرید. interfaceها، نقش inside/outside، نگاشت static و نبود route خصوصی باید مستند شوند؛ در صورت نیاز از `-a` و `-b` استفاده کنید.

## ۸. اتصال محلی Internet تا Server0 را بررسی کنید

روی `Internet` اجرا کنید:

```text
enable
ping 192.168.0.2
```

سپس در `Server0 > Desktop > Command Prompt` اجرا کنید:

```text
ipconfig
ping 192.168.0.1
```

### تفسیر

اگر هرکدام شکست خورد، IP، mask، gateway، کابل و `Fa0/1` را اصلاح کنید. تا زمانی که هر دو جهت محلی موفق نشده‌اند، ping بند ۵ معیار معتبر NAT نیست.

> **Screenshot checkpoint — `D5-06-server0-local-baseline.png`:** بعد از تست هر دو جهت تصویر بگیرید. تنظیمات Server0 و پاسخ موفق gateway باید دیده شوند؛ در صورت نیاز یک تصویر دوم با پسوند `-b` بگیرید.

---

# فاز ۳ — دنبال‌کردن دقیق ping عمومی

## ۹. فقط ping عمومی را بازتولید کنید

Event List را پاک کنید. سپس در `PC1 > Desktop > Command Prompt` فقط این فرمان را اجرا کنید:

```text
ping 213.80.11.6
```

در Simulation با `Capture/Forward` بسته را hopبهhop دنبال کنید.

### مسیر رفت مورد انتظار

```text
PC1 -> R1 -> مسیر داخلی -> R5 -> Internet -> Server0
```

روی `Internet`، PDU ورودی و خروجی را باز کنید. مقصد باید به این شکل تغییر کند:

```text
213.80.11.6 -> 192.168.0.2
```

> **Screenshot checkpoint — `D5-07-public-request-static-translation.png`:** دقیقاً وقتی Internet بسته را از outside به inside عبور داده است تصویر بگیرید. Event List و آدرس مقصد قبل و بعد از Static NAT باید دیده شوند.

## ۱۰. مسیر پاسخ Server0 را دنبال کنید

Capture/Forward را ادامه دهید تا echo reply از Server0 به `Internet` برسد.

### نقطه شکست محتمل

مقصد reply برابر `10.10.6.1` است، اما `Internet` هیچ route به شبکه `10.10.*` ندارد. بنابراین بسته روی Internet متوقف یا drop می‌شود. Static NAT آدرس Server0 را ترجمه می‌کند، اما route برگشت برای PC1 ایجاد نمی‌کند.

> **Screenshot checkpoint — `D5-08-public-reply-drop-on-internet.png`:** در لحظه توقف یا drop شدن echo reply تصویر بگیرید. آدرس مقصد `10.10.6.1`، دستگاه Internet، Event List و نبود مسیر خروجی باید تا حد امکان در قاب باشند.

## ۱۱. ping مستقیم private را جدا بررسی کنید

Event List را پاک کنید و در PC1 اجرا کنید:

```text
ping 192.168.0.2
```

این بار request مستقیماً آدرس local سرور را هدف می‌گیرد. در مسیر برگشت، Server0 از داخل به خارج Internet عبور می‌کند و نگاشت static می‌تواند source را به `213.80.11.6` تبدیل کند؛ بااین‌حال مقصد پاسخ همچنان `10.10.6.1` است و Internet route برگشت ندارد.

> **Screenshot checkpoint — `D5-09-private-ping-failure-path.png`:** بعد از کامل‌شدن Simulation تصویر بگیرید. مقصد اولیه `192.168.0.2` و نقطه شکست reply روی Internet باید ثبت شوند.

---

# فاز ۴ — تصمیم‌گیری براساس محل شکست

| آخرین تست موفق | اولین تست ناموفق | علت محتمل | اقدام بعدی |
|---|---|---|---|
| هیچ‌کدام | PC1 به `10.10.6.2` | IP، gateway، کابل یا R1 LAN | تنظیم PC1 و R1 را اصلاح کنید |
| gateway PC1 | PC1 به `10.10.11.2` | route یا OSPF داخلی | جدول route روترهای داخلی را اصلاح کنید |
| R5 inside | PC1 یا R5 به `213.80.11.5` | لینک خارجی، IP یا mask | `Fa1/0` R5 و `Fa0/0` Internet را اصلاح کنید |
| R5 به Internet | Internet به `192.168.0.2` | Server0، gateway یا `Fa0/1` | اتصال و IPهای Server0 را اصلاح کنید |
| Internet به Server0 | request عمومی به Server0 می‌رسد ولی reply drop می‌شود | نبود route برگشت روی Internet و نبود source NAT روی R5 | رفتار مورد انتظار بند ۵؛ شواهد را ثبت و به بندهای ۷ تا ۱۲ بروید |
| request حتی روی Internet ترجمه نمی‌شود | مقصد `.6` به `.2` تغییر نمی‌کند | نقش inside/outside یا static mapping غلط | بندهای ۲ تا ۴ را اصلاح کنید |

## تشخیص اصلی اگر مراحل ۳ تا ۱۰ مطابق انتظار بودند

اگر این موارد همگی درست بودند:

- PC1 به gateway و R5 دسترسی دارد؛
- R5 به Internet دسترسی دارد؛
- Internet به Server0 دسترسی دارد؛
- Static NAT روی request عمومی انجام می‌شود؛
- reply روی Internet به علت نبود route `10.10.*` متوقف می‌شود؛

آنگاه شکست ping بند ۵ **ریشه‌یابی شده و رفتار طراحی‌شده همین مرحله است**. چیزی را برای موفق‌کردن دائمی ping در بند ۵ تغییر ندهید. تصاویر شکست و توضیح مسیر برگشت را نگه دارید و مراحل ۷ تا ۱۲ را ادامه دهید. Dynamic NAT روی R5 در آن مراحل، source خصوصی PC1 را به یک آدرس عمومی تبدیل می‌کند تا Internet برای پاسخ به route خصوصی نیاز نداشته باشد.

---

# تست تشخیصی اختیاری برای اثبات قطعی مسیر برگشت

این تست فقط برای اثبات فرضیه است و نباید در فایل نهایی باقی بماند.

## ۱۲. یک route برگشت موقت روی Internet اضافه کنید

روی `Internet` paste کنید:

```text
enable
configure terminal
ip route 10.10.0.0 255.255.0.0 213.80.11.4
end
show ip route 10.10.6.0
```

حالا در `PC1 > Desktop > Command Prompt` فقط ping عمومی را تکرار کنید:

```text
ping 213.80.11.6
```

### تفسیر

- اگر ping با route موقت موفق شد، علت اصلی قطعاً نبود مسیر برگشت روی Internet بوده است.
- اگر همچنان شکست خورد، به Simulation برگردید؛ مشکل دیگری پیش از مسیر برگشت وجود دارد.

> **Screenshot checkpoint — `D5-10-temporary-return-route-proof.png`:** بلافاصله پس از تست تصویر بگیرید. route موقت و نتیجه ping عمومی باید ثبت شوند. این تصویر فقط مدرک تشخیصی است و نباید به‌عنوان configuration نهایی معرفی شود.

## ۱۳. route موقت را حتماً حذف کنید

روی `Internet` paste کنید:

```text
enable
configure terminal
no ip route 10.10.0.0 255.255.0.0 213.80.11.4
end
show ip route
write memory
```

`show ip route` باید دوباره فقط شبکه‌های connected خارجی و Server0 را نشان دهد و هیچ route خصوصی `10.10.*` باقی نماند.

> **Screenshot checkpoint — `D5-11-return-route-removed.png`:** بعد از حذف route و پیش از بستن فایل دیباگ تصویر بگیرید. نبود route `10.10.*` باید مشخص باشد.

---

# معیار خروج از دیباگ

دیباگ بند ۵ زمانی کامل است که یکی از دو نتیجه زیر با شواهد ثابت شده باشد:

1. یک خطای واقعی پیش از NAT پیدا و اصلاح شده و بسته اکنون تا نقطه مورد انتظار می‌رسد؛ یا
2. request عمومی به Server0 می‌رسد و reply فقط به‌علت نبود route برگشت روی Internet drop می‌شود؛ در این حالت شکست ping بند ۵ مورد انتظار است و باید مراحل Dynamic NAT ادامه یابند.

تا قبل از یکی از این دو نتیجه، هیچ route دائمی، PAT یا Dynamic NAT خارج از ترتیب سند اصلی اضافه نکنید.

# فهرست نهایی شواهد DEBUG5

- [ ] `D5-01-pc1-local-baseline.png`
- [ ] `D5-02-pc1-to-r5-path.png`
- [ ] `D5-03-r5-routing-and-nat-state.png`
- [ ] `D5-04-r5-to-internet-link.png`
- [ ] `D5-05-internet-routes-and-static-nat.png`
- [ ] `D5-06-server0-local-baseline.png`
- [ ] `D5-07-public-request-static-translation.png`
- [ ] `D5-08-public-reply-drop-on-internet.png`
- [ ] `D5-09-private-ping-failure-path.png`
- [ ] `D5-10-temporary-return-route-proof.png` — اختیاری، فقط برای تست فرضیه
- [ ] `D5-11-return-route-removed.png` — در صورت اجرای تست اختیاری الزامی است

**تعداد پایه دیباگ بدون تست اختیاری: ۹ تصویر.** در صورت اجرای تست route موقت، تعداد پایه به ۱۱ تصویر می‌رسد.
