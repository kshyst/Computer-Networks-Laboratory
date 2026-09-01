# آزمایش ششم — راهنمای کامل اجرای NAT در Packet Tracer

## دامنه و سند اصلی

این راهنما فقط الزامات «آزمایش ششم» در فایل `آزمایش ششم (1).pdf` را پوشش می‌دهد. ترتیب بخش‌ها، بندهای ۱ تا ۱۸ و «نکات مهم» سند اصلی حفظ شده‌اند. این فایل راهنمای اجرا و ثبت شواهد است؛ خروجی اندازه‌گیری‌شده یا تصویر ساختگی در آن وجود ندارد.

## هدف آزمایش

در پایان آزمایش باید بتوانید:

- تفاوت `Inside Local`، `Inside Global`، `Outside Local` و `Outside Global` را در عمل تشخیص دهید؛
- `Static NAT` را روی مسیریاب `Internet` پیاده‌سازی و جدول ترجمه را بررسی کنید؛
- `Dynamic NAT` را روی `R5` با یک `ACL` و یک `NAT pool` اجرا کنید؛
- `PAT` را یک بار با مجموعه‌ای از آدرس‌ها و یک بار با آدرس واسط خروجی اجرا کنید؛
- تفاوت جدول‌های ترجمه در `Dynamic NAT` و `PAT` را توضیح دهید؛
- با یک نگاشت `Static NAT` روی `R5` دسترسی آغازشده از `Server0` به `Server1` را فراهم کنید؛
- همه خروجی‌ها، تصاویر و فایل Packet Tracer را مطابق قواعد تحویل سند اصلی نگه دارید.

## پیش‌نیازها

- Cisco Packet Tracer 8.2.1 یا نسخه سازگار؛
- فایل پایه یکی از توپولوژی‌های نهایی آزمایش ۵، ترجیحاً `../CNL5/5/final/OSPF.pkt`؛
- شماره گروه: **۶**؛
- شماره کلاس: **۲**؛
- آشنایی با `show ip interface brief`، جدول مسیریابی و حالت `Simulation`؛
- یک نسخه کاری مستقل که فایل آزمایش ۵ را بازنویسی نکند.

## فایل‌ها و شواهدی که باید در پایان موجود باشند

- `KiarashShojaei-6-2-6.pkt`؛
- گزارش نهایی تکمیل‌شده، برای مثال `report.pdf`، همراه با فایل منبع آن در صورت مطالبه استاد؛
- تصویر توپولوژی و جدول آدرس‌دهی؛
- شواهد بندهای ۱ تا ۱۸ با نام‌های پیشنهادی این راهنما؛
- خروجی‌های `show ip nat translations` و `show ip nat statistics` برای هر حالت؛
- فایل ZIP نهایی با الگوی خواسته‌شده در سند اصلی:

```text
KiarashShojaei-6-2-6.zip
```

## قواعد ثبت شواهد

1. در هر تصویر، نام دستگاه یا prompt آن، فرمان کامل و خروجی مرتبط را نگه دارید.
2. برای جدول NAT، ستون‌های `Pro`، `Inside global`، `Inside local`، `Outside local` و `Outside global` باید دیده شوند.
3. در تصاویر `Simulation`، هم `Event List` و هم جزئیات `Inbound PDU` و `Outbound PDU` را تا حد امکان نشان دهید.
4. نتیجه مورد انتظار را به‌جای نتیجه واقعی گزارش نکنید. خروجی واقعی Packet Tracer را ثبت کنید.
5. پیش از هر آزمایش جدید، ترجمه‌های قبلی را پاک کنید تا جدول قدیمی باعث برداشت اشتباه نشود:

```text
enable
clear ip nat translation *
```

6. اگر Packet Tracer فرمانی را با نام کوتاه قبول نکرد، نام کامل واسط را با `show ip interface brief` پیدا کنید.
7. همه بلوک‌های اجرایی این راهنما بدون promptهایی مانند `R1#` یا `R5(config)#` نوشته شده‌اند. نام دستگاه دقیقاً بالای هر بلوک آمده است؛ فقط همان بلوک را در همان دستگاه paste کنید.

## ناسازگاری‌های خود سند اصلی و تصمیم اجرایی این راهنما

سند اصلی چند ناسازگاری دارد. برای اینکه آزمایش قابل اجرا باشد، این راهنما آن‌ها را پنهان نمی‌کند و تصمیم عملی را صریحاً ثبت می‌کند:

1. در بند ۱ عبارت «شکل ۶» آمده، اما سند فقط «شکل ۱» دارد؛ آدرس‌ها از شکل ۱ گرفته می‌شوند.
2. prompt نمونه‌ها `R1` است، ولی بندهای ۲ تا ۴ مربوط به مسیریاب `Internet` و بندهای ۷ تا ۱۸ مربوط به `R5` هستند. نام دستگاه ملاک است، نه متن prompt نمونه.
3. بند ۹ شبکه `10.0.2.0/24` را در ACL نشان می‌دهد، درحالی‌که شبکه‌های این توپولوژی `10.10.z.0/24` هستند. ACL اجرایی باید آدرس‌های واقعی `10.10.*` را پوشش دهد.
4. متن بند ۱۰ بازه `213.80.11.16` تا `213.80.11.31` را می‌خواهد، اما دستور نمونه بازه `203.0.113.4` تا `203.0.113.14` دارد. همچنین در زیرشبکه `/28`، آدرس `.16` آدرس شبکه و `.31` broadcast است. بنابراین بازه معتبر و متصل `213.80.11.17` تا `213.80.11.30` استفاده می‌شود.
5. سند فرمان `show ip nat translation` را مفرد نوشته است؛ فرمان IOS مورد استفاده در Packet Tracer معمولاً `show ip nat translations` است.
6. بند ۱۷ واسط `fa0/0` را برای PAT تک‌آدرسی نشان می‌دهد، ولی طبق شکل و بند ۷، واسط `outside` در `R5` برابر `Fa1/0` است. واسط واقعی outside ملاک است.
7. عبارت نبود اطلاعات مسیرهای داخلی به این صورت اجرا می‌شود: `Internet` هیچ مسیر اختصاصی به شبکه‌های `10.10.*` ندارد؛ `R5` نیز مسیر اختصاصی به `192.168.0.0/24` ندارد و فقط یک default route به `Internet` دارد. `R5` باید برای بازگرداندن بسته‌های NAT، مسیر شبکه‌های داخلی خودش را بداند.

---

# ۱. آشنایی با NAT

## ۱.۱. مقدمه

### نیازمندی

درک هدف NAT و چهار اصطلاحی که سند اصلی معرفی کرده است.

### توضیح

`NAT` در لایه ۳ مدل `OSI` آدرس IP را هنگام عبور بسته از یک مرز شبکه تغییر می‌دهد. در این آزمایش دو مرز NAT داریم:

- `Internet`: آدرس واقعی `Server0` یعنی `192.168.0.2` را با آدرس عمومی `213.80.11.6` نمایش می‌دهد؛
- `R5`: آدرس‌های خصوصی شبکه‌های `10.10.*` را برای خروج به شبکه خارجی ترجمه می‌کند.

کاربردهای ذکرشده در سند اصلی عبارت‌اند از:

- ترجمه آدرس‌های `Private` به `Public` یا برعکس؛
- تغییر سرویس‌دهنده اینترنت بدون تغییر آدرس‌های داخلی؛
- پنهان‌کردن بخشی از ساختار داخلی از شبکه خارجی؛
- تغییر شفاف آدرس یا پورت مقصد بسته‌ها.

### اصطلاحات عیناً مطابق سند اصلی

سند اصلی از ترتیب نام‌گذاری زیر استفاده می‌کند. این نام‌ها و تعریف‌ها برای حفظ انطباق با متن PDF عیناً ثبت می‌شوند:

| برچسب چاپ‌شده در PDF | تعریف چاپ‌شده در PDF |
|---|---|
| `Local Inside` | آدرس‌هایی که روی کلاینت‌های شبکه داخلی تنظیم شده‌اند |
| `Global Inside` | آدرسی که به واسط داخلی روتر متصل به شبکه داخلی داده شده است |
| `Local Outside` | آدرس‌هایی که درون اینترنت یا شبکه `Public` قرار دارند |
| `Global Outside` | آدرس‌هایی که روی واسط خارجی روتر متصل به شبکه `Public` قرار دارند |

ترتیب واژه‌ها و بخشی از تعریف‌های بالا با اصطلاحات استاندارد Cisco سازگار نیست. در خروجی‌های IOS و ادامه این راهنما، اصطلاحات استاندارد `Inside Local`، `Inside Global`، `Outside Local` و `Outside Global` به کار می‌روند. این اصلاح باید در گزارش نیز صریحاً ذکر شود و نباید به‌صورت تغییر بی‌توضیح متن سند ارائه گردد.

### اصطلاحات استاندارد Cisco و انطباق آن‌ها با این توپولوژی

| اصطلاح | معنی | نمونه در آزمایش |
|---|---|---|
| `Inside Local` | آدرس واقعی میزبان داخلی | `Server0 = 192.168.0.2` روی NAT مسیریاب Internet؛ یا `PC1 = 10.10.6.1` روی NAT مسیریاب R5 |
| `Inside Global` | آدرس قابل مشاهده میزبان داخلی در سمت خارج | `Server0 = 213.80.11.6`؛ یا یکی از آدرس‌های pool روی R5 |
| `Outside Local` | آدرس میزبان خارجی آن‌گونه که از داخل دیده می‌شود | آدرس مقصد خارجی در بسته ورودی به NAT؛ معمولاً بدون ترجمه با Outside Global برابر است |
| `Outside Global` | آدرس واقعی و قابل مسیریابی میزبان خارجی | برای NAT روی R5، آدرس عمومی `Server0` یعنی `213.80.11.6` |

### راستی‌آزمایی

بعد از هر پیکربندی، جدول `show ip nat translations` را با این چهار مفهوم تطبیق دهید و مشخص کنید هر ستون به کدام آدرس توپولوژی اشاره دارد.

### نیازمندی برآورده‌شده

بخش `۱. آشنایی با NAT` و `۱.۱. مقدمه` سند اصلی.

---

## ۱.۲. شرح آزمایش

## نقشه آدرس‌دهی شکل ۱ برای گروه ۶

قاعده سند اصلی برابر `z = x + GroupNum` است. با شماره گروه ۶، آدرس‌ها به صورت زیر می‌شوند.

سند اصلی همچنین تصریح می‌کند افرادی که گروه ندارند باید یک عدد تصادفی بین **۳۰ تا ۲۳۰** انتخاب و آن را به‌جای `GroupNum` استفاده کنند. این حالت در این اجرا کاربرد ندارد، زیرا شماره گروه ۶ مشخص است، اما بخشی از الزام بند ۱ محسوب می‌شود.

| دستگاه | واسط پیشنهادی | اتصال | آدرس | Mask / Gateway |
|---|---|---|---|---|
| `PC1` | `FastEthernet0` | `SW1` | `10.10.6.1` | `/24`، gateway: `10.10.6.2` |
| `R1` | `Fa0/0` | شبکه PC1 | `10.10.6.2` | `/24` |
| `Server1` | `FastEthernet0` | `SW2` | `10.10.7.2` | `/24`، gateway: `10.10.7.1` |
| `R1` | `Fa1/1` یا واسط آزاد | شبکه Server1 | `10.10.7.1` | `/24` |
| `R1` | `Fa0/1` | `R2` | `10.10.8.1` | `/24` |
| `R2` | `Fa0/0` | `R1` | `10.10.8.2` | `/24` |
| `R2` | `Fa0/1` | `R3` | `10.10.9.1` | `/24` |
| `R3` | `Fa0/0` | `R2` | `10.10.9.2` | `/24` |
| `R3` | `Fa0/1` | `R4` | `10.10.10.1` | `/24` |
| `R4` | `Fa0/1` | `R3` | `10.10.10.2` | `/24` |
| `R1` | `Fa1/0` | `R5` | `10.10.11.1` | `/24` |
| `R5` | `Fa0/0` | `R1` | `10.10.11.2` | `/24` |
| `R4` | `Fa1/0` | `R5` | `10.10.12.1` | `/24` |
| `R5` | `Fa0/1` | `R4` | `10.10.12.2` | `/24` |
| `PC3` | `FastEthernet0` | `SW3` | `10.10.16.1` | `/24`، gateway: `10.10.16.2` |
| `R4` | `Fa0/0` | شبکه PC3 | `10.10.16.2` | `/24` |
| `R5` | `Fa1/0` | `SW4/Internet` | `213.80.11.4` | `/24` |
| `Internet` | `Fa0/0` | `SW4/R5` | `213.80.11.5` | `/24` |
| `Internet` | `Fa0/1` | `Server0` | `192.168.0.1` | `/24` |
| `Server0` | `FastEthernet0` | `Internet` | `192.168.0.2` | `/24`، gateway: `192.168.0.1` |

اگر شماره واسط‌های فایل شما متفاوت است، آدرس و نقش را به واسطی بدهید که واقعاً به دستگاه ستون «اتصال» وصل است. قبل از هر فرمان NAT این موضوع را با دستور زیر قطعی کنید:

```text
enable
show ip interface brief
```

---

## ۱. ساختار شبکه شکل ۱ را در Packet Tracer پیاده‌سازی کنید

### نیازمندی

پیاده‌سازی کامل شکل ۱، اعمال قاعده آدرس‌دهی گروه و فراهم‌کردن مسیریابی داخلی بدون انتشار شبکه‌های خصوصی به مسیریاب `Internet`.

### هدف

ساخت یک baseline سالم تا تفاوت نتیجه NAT با خرابی ساده آدرس‌دهی یا مسیریابی اشتباه گرفته نشود.

### تبدیل دقیق فایل آزمایش ۵ به توپولوژی آزمایش ۶

#### A. فایل پایه را بدون بازنویسی آزمایش ۵ آماده کنید

1. فایل `../CNL5/5/final/OSPF.pkt` را در Packet Tracer باز کنید.
2. بلافاصله `File > Save As` را بزنید و فایل را با نام `CNL6/KiarashShojaei-6-2-6.pkt` ذخیره کنید.
3. از این لحظه فقط روی نسخه آزمایش ۶ کار کنید.

#### B. دستگاه‌ها و کابل‌های جدید را دقیقاً اضافه کنید

1. یک `2960-24TT` با نام `SW2` اضافه کنید.
2. یک `Server-PT` با نام `Server1` اضافه کنید.
3. با کابل `Copper Straight-Through` این اتصال‌ها را بسازید:
   - `R1 FastEthernet1/1` به `SW2 FastEthernet0/1`؛
   - `Server1 FastEthernet0` به `SW2 FastEthernet0/2`.
4. یک `Server-PT` دیگر با نام `Server0` اضافه کنید.
5. با `Automatically Choose Connection Type` یا کابل `Copper Cross-Over`، `Internet FastEthernet0/1` را مستقیم به `Server0 FastEthernet0` وصل کنید.
6. صبر کنید همه لینک‌های جدید سبز شوند. اگر `R1` واسط `FastEthernet1/1` ندارد، فایل پایه اشتباه است؛ فایل نهایی OSPF آزمایش ۵ باید ماژول `NM-2FE2W` را داشته باشد.

#### C. NAT باقی‌مانده از آزمایش ۵ را روی R5 پاک کنید

فایل نهایی آزمایش ۵ از قبل PAT دارد. برای اینکه مراحل ۷ تا ۱۸ آزمایش ۶ واقعی و قابل مشاهده باشند، ابتدا آن NAT را پاک کنید. بلوک زیر را کامل و یک‌جا در CLI دستگاه `R5` paste کنید:

```text
enable
configure terminal
no ip nat inside source list 1 interface FastEthernet1/0 overload
no access-list 1
interface FastEthernet0/0
 no ip nat inside
exit
interface FastEthernet0/1
 no ip nat inside
exit
interface FastEthernet1/0
 no ip nat outside
 ip address 213.80.11.4 255.255.255.0
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 213.80.11.5
router ospf 1
 default-information originate
end
clear ip nat translation *
write memory
```

اگر یکی از فرمان‌های `no ...` پیام داد که تنظیم مورد نظر وجود ندارد، همان پیام فقط یعنی آن مورد از قبل پاک بوده است؛ ادامه دهید.

#### D. واسط جدید R1 و شبکه Server1 را تنظیم کنید

بلوک زیر را کامل در CLI دستگاه `R1` paste کنید:

```text
enable
configure terminal
interface FastEthernet1/1
 description TO-SW2-SERVER1
 ip address 10.10.7.1 255.255.255.0
 no shutdown
exit
router ospf 1
 network 10.10.7.0 0.0.0.255 area 0
 passive-interface FastEthernet1/1
end
write memory
```

فایل OSPF آزمایش ۵ از قبل `network 10.10.0.0 0.0.255.255 area 0` و `passive-interface default` دارد؛ دو خط OSPF بالا صریحاً شبکه جدید را ثبت می‌کنند و روی LAN سرور همسایگی OSPF نمی‌سازند.

#### E. دو واسط مسیریاب Internet را برای توپولوژی جدید آماده کنید

بلوک زیر را کامل در CLI دستگاه `Internet` paste کنید. نقش‌های NAT عمداً پاک می‌شوند تا در بندهای ۲ و ۳ خود آزمایش اضافه شوند:

```text
enable
configure terminal
interface FastEthernet0/0
 description TO-SW4-R5
 ip address 213.80.11.5 255.255.255.0
 no ip nat outside
 no shutdown
exit
interface FastEthernet0/1
 description TO-SERVER0
 ip address 192.168.0.1 255.255.255.0
 no ip nat inside
 no shutdown
exit
no ip nat inside source static 192.168.0.2 213.80.11.6
end
clear ip nat translation *
write memory
```

روی `Internet` هیچ OSPF، EIGRP، RIP یا مسیر static به شبکه‌های `10.10.*` اضافه نکنید.

#### F. IP دو سرور را از رابط گرافیکی تنظیم کنید

روی `Server1` به `Desktop > IP Configuration` بروید و دقیقاً وارد کنید:

```text
IP Address:      10.10.7.2
Subnet Mask:    255.255.255.0
Default Gateway: 10.10.7.1
```

روی `Server0` به `Desktop > IP Configuration` بروید و دقیقاً وارد کنید:

```text
IP Address:      192.168.0.2
Subnet Mask:    255.255.255.0
Default Gateway: 192.168.0.1
```

> **Screenshot checkpoint — `01-topology-and-addresses.png`:** همین حالا که هر دو سرور اضافه و IPها تنظیم شده‌اند، پنجره Command Prompt را ببندید، کل توپولوژی را در یک نما قرار دهید و تصویر بگیرید. نام همه دستگاه‌ها، اتصال‌های جدید `R1-SW2-Server1` و `Internet-Server0` و سبزبودن لینک‌ها باید دیده شوند. اگر IP labelها روی workspace نمایش داده می‌شوند، آن‌ها را هم داخل تصویر نگه دارید.

#### G. تنظیمات انتقال‌یافته را دستگاه‌به‌دستگاه بررسی کنید

روی `R1` paste کنید:

```text
enable
show ip interface brief
show ip route 10.10.7.0
show ip route 0.0.0.0
show ip ospf interface brief
```

روی `R5` paste کنید:

```text
enable
show ip interface brief
show ip route 10.10.7.0
show ip route 0.0.0.0
show ip nat translations
show ip nat statistics
```

در این نقطه، R5 باید route شبکه `10.10.7.0/24` و default route به `213.80.11.5` را داشته باشد، ولی جدول NAT و فهرست inside/outside باید خالی باشند.

> **Screenshot checkpoint — `01-r5-routes-before-nat.png`:** بلافاصله بعد از اجرای بلوک بررسی R5 و پیش از رفتن سراغ Internet تصویر بگیرید. خروجی route شبکه `10.10.7.0/24`، default route به `213.80.11.5` و خالی‌بودن NAT باید در تصویر قابل خواندن باشد؛ در صورت نیاز خروجی‌ها را جدا اجرا کنید و آخرین بخش‌های مرتبط را در یک قاب نگه دارید.

روی `Internet` paste کنید:

```text
enable
show ip interface brief
show ip route
show ip nat translations
show ip nat statistics
```

روی `Internet` فقط شبکه‌های متصل `213.80.11.0/24` و `192.168.0.0/24` باید دیده شوند؛ هیچ route به `10.10.*` و هیچ NAT فعالی نباید وجود داشته باشد.

> **Screenshot checkpoint — `01-internet-routes-before-nat.png`:** همین‌جا از خروجی `show ip route` و خالی‌بودن `show ip nat translations` روی Internet تصویر بگیرید. شبکه‌های connected باید دیده شوند و هیچ مسیر `10.10.*` نباید در قاب باشد.

#### H. اتصال‌های baseline را از هر دستگاه جداگانه آزمایش کنید

در `PC1 > Desktop > Command Prompt` paste کنید:

```text
ping 10.10.16.1
ping 10.10.7.2
```

در `Server1 > Desktop > Command Prompt` paste کنید:

```text
ping 10.10.7.1
ping 10.10.6.1
```

در `Server0 > Desktop > Command Prompt` paste کنید:

```text
ping 192.168.0.1
```

سه بلوک بالا باید موفق باشند. دسترسی `PC1` به `Server0` هنوز معیار baseline نیست، چون NAT مراحل بعدی عمداً پاک شده است.

> **Screenshot checkpoint — `01-internal-connectivity.png`:** بعد از موفق‌شدن pingهای baseline و قبل از شروع بند ۲ تصویر بگیرید. ترجیحاً Command Prompt مربوط به PC1 را با هر دو پاسخ موفق `10.10.16.1` و `10.10.7.2` باز نگه دارید؛ اگر همه خروجی‌ها در یک قاب جا نمی‌شوند، تصاویر اضافی با پسوند `-a` و `-b` بگیرید.

### نتیجه مورد انتظار

- همه لینک‌ها سبز و واسط‌ها `up/up` هستند؛
- `PC1`، `PC3` و `Server1` در شبکه داخلی به یکدیگر دسترسی دارند؛
- `R5` یک default route به `213.80.11.5` دارد؛
- `Internet` هیچ route به `10.10.*` ندارد؛
- هنوز هیچ ترجمه NAT فعالی وجود ندارد.

### راستی‌آزمایی

روی `R5` paste کنید:

```text
enable
show ip route
show ip nat translations
show ip nat statistics
```

روی `Internet` paste کنید:

```text
enable
show ip route
show ip nat translations
show ip nat statistics
```

### شواهد لازم

- `01-topology-and-addresses.png`؛
- `01-r5-routes-before-nat.png`؛
- `01-internet-routes-before-nat.png`؛
- `01-internal-connectivity.png`.

### نیازمندی برآورده‌شده

بند ۱ و همه سه زیربند آن در سند اصلی.

---

## ۲. واسط سمت سوئیچ در مسیریاب Internet را outside کنید

### نیازمندی و هدف

واسط متصل به `SW4/R5` باید سمت خارجی NAT مسیریاب `Internet` باشد.

### روش اجرا

```text
enable
configure terminal
interface FastEthernet0/0
 ip nat outside
 no shutdown
end
show ip interface brief
show ip nat statistics
```

### نتیجه مورد انتظار و راستی‌آزمایی

در `show ip interface brief`، واسط `Fa0/0` باید آدرس `213.80.11.5` و وضعیت `up/up` داشته باشد. در `show ip nat statistics` نیز `FastEthernet0/0` باید زیر `Outside interfaces` دیده شود.

> **Screenshot checkpoint — `02-internet-outside-interface.png`:** بلافاصله پس از اجرای بلوک بند ۲ تصویر بگیرید. خروجی `show ip interface brief` باید IP و وضعیت `Fa0/0` را نشان دهد و خروجی `show ip nat statistics` باید `FastEthernet0/0` را در فهرست outside نشان دهد.

### شاهد لازم

`02-internet-outside-interface.png`.

### نیازمندی برآورده‌شده

بند ۲ سند اصلی.

---

## ۳. واسط سمت Server0 در مسیریاب Internet را inside کنید

### روش اجرا

```text
enable
configure terminal
interface FastEthernet0/1
 ip nat inside
 no shutdown
end
show ip interface brief
show ip nat statistics
```

### نتیجه مورد انتظار

در `show ip interface brief`، واسط `Fa0/1` با آدرس `192.168.0.1` باید `up/up` باشد. در `show ip nat statistics` نیز `FastEthernet0/1` باید زیر `Inside interfaces` دیده شود.

### راستی‌آزمایی

از `Internet` خود `Server0` را ping کنید:

```text
ping 192.168.0.2
```

این ping باید پیش از NAT هم موفق باشد، چون شبکه مستقیماً متصل است.

> **Screenshot checkpoint — `03-internet-inside-interface.png`:** پس از موفق‌شدن ping `192.168.0.2` و پیش از بند ۴ تصویر بگیرید. خروجی `show ip interface brief` باید `Fa0/1 = 192.168.0.1` و `up/up` را نشان دهد، خروجی `show ip nat statistics` باید آن را در فهرست inside نشان دهد و نتیجه ping نیز باید ثبت شود.

### شاهد لازم

`03-internet-inside-interface.png`.

### نیازمندی برآورده‌شده

بند ۳ سند اصلی.

---

## ۴. Static NAT را روی Internet تنظیم کنید

### نیازمندی

نگاشت ثابت `192.168.0.2 <-> 213.80.11.6`.

### روش اجرا

```text
enable
configure terminal
ip nat inside source static 192.168.0.2 213.80.11.6
end
show ip nat translations
show ip nat statistics
```

### توضیح

- `192.168.0.2` برابر `Inside Local` است؛
- `213.80.11.6` برابر `Inside Global` است؛
- چون نگاشت static است، حتی بدون تولید ترافیک باید در جدول دیده شود.

### نتیجه مورد انتظار

یک ردیف دائمی با `Inside global = 213.80.11.6` و `Inside local = 192.168.0.2` دیده می‌شود.

> **Screenshot checkpoint — `04-internet-static-nat-config-and-table.png`:** بلافاصله بعد از اجرای بلوک بند ۴ و پیش از ایجاد هر ترافیک تصویر بگیرید. جدول باید نگاشت دائمی `192.168.0.2 <-> 213.80.11.6` را نشان دهد و prompt دستگاه Internet نیز در قاب مشخص باشد.

### شاهد لازم

`04-internet-static-nat-config-and-table.png`.

### نیازمندی برآورده‌شده

بند ۴ سند اصلی.

---

## ۵. از یک host، Server0 را با دو آدرس ping کنید و نتایج را شرح دهید

### نیازمندی

از `PC1` یا `PC3` ابتدا `213.80.11.6` و سپس `192.168.0.2` را ping کنید و رفتار را توضیح دهید.

### روش اجرا

1. در `Simulation` فقط `ARP` و `ICMP` را فعال کنید.
2. روی `Internet` بلوک زیر را paste کنید تا ترجمه‌های موقت پاک شوند؛ نگاشت static در configuration باقی می‌ماند:

```text
enable
clear ip nat translation *
```

3. در `PC1 > Desktop > Command Prompt` ابتدا فقط ping عمومی را اجرا کنید:

```text
ping 213.80.11.6
```

> **Screenshot checkpoint — `05-ping-server0-public.png`:** به‌محض تمام‌شدن همین ping و پیش از اجرای ping بعدی، از Command Prompt تصویر بگیرید. فرمان، هر چهار پاسخ یا timeout و آدرس مقصد `213.80.11.6` باید کامل دیده شوند.

حالا ping آدرس private را جدا اجرا کنید:

```text
ping 192.168.0.2
```

> **Screenshot checkpoint — `05-ping-server0-private.png`:** بلافاصله بعد از تمام‌شدن ping دوم تصویر بگیرید. فرمان، نتیجه کامل و مقصد `192.168.0.2` باید دیده شوند تا با تصویر قبلی قابل مقایسه باشد.

4. بسته عمومی را در `Internet` باز کنید و مقایسه کنید:
   - قبل از NAT مقصد `213.80.11.6` است؛
   - بعد از NAT مقصد `192.168.0.2` می‌شود.

> **Screenshot checkpoint — `05-static-nat-pdu-before-after.png`:** در Simulation دقیقاً وقتی بسته روی Internet انتخاب شده است تصویر بگیرید. پنجره PDU باید آدرس مقصد پیش از NAT یعنی `213.80.11.6` و پس از NAT یعنی `192.168.0.2` را نشان دهد؛ Event List را نیز در قاب نگه دارید.

5. بسته مستقیم به `192.168.0.2` را بررسی کنید؛ این بسته از نگاشت static مقصد استفاده نمی‌کند، چون از ابتدا مقصد local را دارد.
6. بلافاصله روی `Internet` بلوک زیر را paste و جدول را ثبت کنید:

```text
enable
show ip nat translations
```

> **Screenshot checkpoint — `05-internet-nat-table-after-ping.png`:** بلافاصله پس از اجرای این فرمان و قبل از پاک‌کردن یا تولید ترافیک جدید تصویر بگیرید. نگاشت static و هر ورودی ICMP مرتبط باید خوانا باشند.

### نتیجه مورد انتظار

با baseline این راهنما، `Internet` مسیر `10.10.*` ندارد. بنابراین درخواست ممکن است تا `Server0` برسد، اما پاسخ به مبدأ خصوصی راه بازگشت ندارد و ping کامل نشود. نکته اصلی بند ۵ مشاهده ترجمه مقصد برای آدرس عمومی و نبود آن برای آدرس مستقیم داخلی است. نتیجه واقعی Packet Tracer را ثبت کنید و در گزارش توضیح دهید که شکست احتمالی ناشی از نبود مسیر بازگشت به مبدأ خصوصی است، نه نبود نگاشت static.

اگر ping عمومی در فایل شما موفق شد، جدول route مسیریاب `Internet` را بررسی کنید؛ احتمالاً یک مسیر قدیمی به `10.10.*` باقی مانده است. آن را به‌عنوان تفاوت تنظیمات ثبت و برای ادامه baseline را اصلاح کنید.

### راستی‌آزمایی

- برای ping عمومی، یک رخداد ترجمه `213.80.11.6 -> 192.168.0.2` دیده شود؛
- جدول static صحیح باقی بماند؛
- نتیجه هر دو ping با علت مسیر رفت و برگشت توضیح داده شود.

### شواهد لازم

- `05-ping-server0-public.png`؛
- `05-ping-server0-private.png`؛
- `05-static-nat-pdu-before-after.png`؛
- `05-internet-nat-table-after-ping.png`.

### نیازمندی برآورده‌شده

بند ۵ و زیربند «نتایج خود را شرح دهید».

---

## ۶. جدول NAT را بررسی و از Server0، Server1 را ping کنید

### نیازمندی

جدول NAT هنگام خروج بسته به سمت `Server0` بررسی شود؛ سپس `Server0`، `Server1` را ping کند و علت نتیجه توضیح داده شود.

### روش اجرا

1. در `Simulation` یک ping از `PC1` به `213.80.11.6` بسازید.
2. درست پس از عبور بسته از `Internet` به سمت `Server0` اجرای شبیه‌سازی را متوقف کنید.
3. روی `Internet` بلوک زیر را paste کنید:

```text
enable
show ip nat translations
show ip nat statistics
```

> **Screenshot checkpoint — `06-internet-nat-table-during-static-test.png`:** درست در همین توقف Simulation و پیش از ادامه‌دادن بسته تصویر بگیرید. جدول NAT، counters و زمان قرارگرفتن بسته پس از خروج از Internet به سمت Server0 باید مستند شوند.

4. در `Server0 > Desktop > Command Prompt` بلوک زیر را paste کنید:

```text
ping 10.10.7.2
```

> **Screenshot checkpoint — `06-server0-to-server1-before-r5-nat.png`:** پس از تمام‌شدن ping ناموفق Server0 و پیش از تغییر تنظیمات R5 تصویر بگیرید. آدرس مقصد و timeoutها باید کامل دیده شوند.

برای ثبت علت شکست، بلوک زیر را روی `Internet` paste کنید:

```text
enable
show ip route 10.10.7.0
show ip route
```

> **Screenshot checkpoint — `06-internet-route-explaining-failure.png`:** بلافاصله پس از اجرای بلوک route تصویر بگیرید. نبود route برای `10.10.7.0/24` و وجود فقط شبکه‌های connected باید مشخص باشد.

### نتیجه مورد انتظار

ping آغازشده از `Server0` به `Server1` در این مرحله برقرار نمی‌شود، زیرا:

- `Internet` route اختصاصی به شبکه `10.10.7.0/24` ندارد؛
- هنوز روی `R5` ترجمه‌ای برای نمایش `Server1` با یک آدرس global ساخته نشده است؛
- `Static NAT` روی `Internet` فقط آدرس `Server0` را نمایش می‌دهد و مشکل دسترسی آغازشده به شبکه پشت `R5` را حل نمی‌کند.

### شواهد لازم

- `06-internet-nat-table-during-static-test.png`؛
- `06-server0-to-server1-before-r5-nat.png`؛
- `06-internet-route-explaining-failure.png`.

### نیازمندی برآورده‌شده

ادامه بند ۵ در ابتدای صفحه ۴ و بند ۶ سند اصلی.

---

## ۷. واسط خارجی R5 را outside کنید

### روش اجرا

ابتدا مطمئن شوید واسط متصل به `SW4/Internet` واقعاً `Fa1/0` است:

```text
enable
show ip interface brief
configure terminal
interface FastEthernet1/0
 ip nat outside
 no shutdown
end
show ip interface brief
show ip nat statistics
```

### نتیجه مورد انتظار

در `show ip interface brief`، واسط `Fa1/0` باید آدرس `213.80.11.4` و وضعیت `up/up` داشته باشد. در `show ip nat statistics` نیز `FastEthernet1/0` باید زیر `Outside interfaces` دیده شود.

> **Screenshot checkpoint — `07-r5-outside-interface.png`:** بلافاصله پس از اجرای بلوک بند ۷ تصویر بگیرید. خروجی interface brief باید IP و وضعیت `Fa1/0` و خروجی NAT statistics باید نقش outside آن را نشان دهد.

### شاهد لازم

`07-r5-outside-interface.png`.

### نیازمندی برآورده‌شده

بند ۷ سند اصلی.

---

## ۸. واسط‌های داخلی R5 را inside کنید

### روش اجرا

```text
enable
configure terminal
interface FastEthernet0/0
 ip nat inside
 no shutdown
exit
interface FastEthernet0/1
 ip nat inside
 no shutdown
end
show ip interface brief
show ip nat statistics
```

### توضیح

- `Fa0/0` به `R1` وصل است؛
- `Fa0/1` به `R4` وصل است؛
- هر دو مسیر می‌توانند ترافیک شبکه خصوصی را به R5 برسانند، پس هر دو باید `inside` باشند.

### راستی‌آزمایی

آدرس‌های آن‌ها باید به‌ترتیب `10.10.11.2/24` و `10.10.12.2/24` باشند.

> **Screenshot checkpoint — `08-r5-inside-interfaces.png`:** بعد از اجرای بلوک بند ۸ و قبل از ساخت ACL تصویر بگیرید. `show ip interface brief` باید IP و وضعیت `Fa0/0` و `Fa0/1` را نشان دهد و `show ip nat statistics` باید هر دو را زیر `Inside interfaces` فهرست کند.

### شاهد لازم

`08-r5-inside-interfaces.png`.

### نیازمندی برآورده‌شده

بند ۸ سند اصلی.

---

## ۹. ACL ترافیک داخلی را ایجاد کنید

### نیازمندی

ساخت `access-list 1` برای انتخاب آدرس‌هایی که باید ترجمه شوند.

### روش اجرا

فرمان نمونه سند (`10.0.2.0 0.0.0.255`) با آدرس‌دهی واقعی شکل ۱ منطبق نیست. برای پوشش همه شبکه‌های `10.10.z.0/24` این آزمایش از خلاصه زیر استفاده کنید:

```text
enable
configure terminal
no access-list 1
access-list 1 permit 10.10.0.0 0.0.255.255
end
show access-lists 1
```

اگر استاد پوشش دقیق هر subnet را بخواهد، به‌جای خلاصه، هر LAN مبدأ مورد استفاده را جداگانه وارد کنید؛ حداقل شبکه‌های `10.10.6.0/24`، `10.10.7.0/24` و `10.10.16.0/24`.

### نتیجه مورد انتظار

`Standard IP access list 1` شبکه‌های واقعی میزبان‌های داخلی را permit می‌کند.

> **Screenshot checkpoint — `09-r5-nat-acl.png`:** بلافاصله بعد از `show access-lists 1` تصویر بگیرید. شماره ACL، عبارت permit برای `10.10.0.0 0.0.255.255` و در مراحل بعد counterهای match باید خوانا باشند.

### شاهد لازم

`09-r5-nat-acl.png`.

### نیازمندی برآورده‌شده

بند ۹ سند اصلی، با رفع ناسازگاری آدرس نمونه.

---

## ۱۰. pool آدرس‌های عمومی را ایجاد کنید

### نیازمندی

ساخت pool با نام `NetLab` از محدوده عمومی متصل به لینک `213.80.11.0/24`.

### روش اجرا

فرمان نمونه PDF، پس از حذف prompt غیرقابل‌کپی `R1(config)#`، این است: `ip nat pool NetLab 203.0.113.4 203.0.113.14 netmask 255.255.255.240`. **این فرمان را اجرا نکنید**؛ فقط برای ثبت ناسازگاری سند آورده شده است.

این فرمان با متن همان بند که بازه `213.80.11.16` تا `213.80.11.31` را می‌خواهد و با شبکه خارجی شکل ۱ ناسازگار است. علاوه بر آن، آدرس‌های `.16` و `.31` در بلوک `/28` به‌ترتیب network و broadcast هستند و قابل واگذاری به میزبان نیستند. بنابراین جایگزین اجرایی امن و متصل، بازه `.17` تا `.30` است:

```text
enable
configure terminal
ip nat pool NetLab 213.80.11.17 213.80.11.30 netmask 255.255.255.240
end
show ip nat statistics
```

### نتیجه مورد انتظار

pool `NetLab` شامل ۱۴ آدرس قابل استفاده است و با شبکه خارجی توپولوژی سازگار است.

> **Screenshot checkpoint — `10-r5-netlab-pool.png`:** بلافاصله پس از اجرای بلوک بند ۱۰ تصویر بگیرید. خروجی `show ip nat statistics` باید pool با نام `NetLab`، بازه `213.80.11.17` تا `213.80.11.30` و netmask مرتبط را نشان دهد.

### شاهد لازم

`10-r5-netlab-pool.png`.

### نیازمندی برآورده‌شده

بند ۱۰ سند اصلی، با استفاده از host range معتبر متن همان بند.

---

## ۱۱. Dynamic NAT را روی R5 فعال کنید

### روش اجرا

```text
enable
configure terminal
ip nat inside source list 1 pool NetLab
end
clear ip nat translation *
show ip nat statistics
```

### توضیح

وقتی یک میزبان مجاز در ACL از `inside` به `outside` بسته بفرستد، R5 یک آدرس آزاد از `NetLab` به آن اختصاص می‌دهد. این ترجمه تا پایان timeout یا پاک‌شدن جدول باقی می‌ماند.

### راستی‌آزمایی اولیه

قبل از تولید ترافیک، جدول ممکن است خالی باشد. پس از ping بند ۱۲ باید نگاشت dynamic ظاهر شود.

> **Screenshot checkpoint — `11-r5-dynamic-nat-config.png`:** همین حالا، پیش از اجرای ping بند ۱۲، تصویر بگیرید. خروجی `show ip nat statistics` باید association مربوط به `list 1 pool NetLab` و نقش‌های inside/outside را نشان دهد؛ خالی‌بودن translation table در این لحظه طبیعی است.

### شاهد لازم

`11-r5-dynamic-nat-config.png`.

### نیازمندی برآورده‌شده

بند ۱۱ سند اصلی.

---

## ۱۲. pingها و جدول Dynamic NAT را بررسی کنید

### نیازمندی

از یک host، `Server0` را با آدرس‌های `213.80.11.6` و `192.168.0.2` ping کنید، نتیجه را شرح دهید و جدول NAT را هنگام عبور بسته بررسی کنید.

### روش اجرا

1. روی `R5` بلوک زیر را paste کنید تا ترجمه‌های قبلی پاک شوند:

```text
enable
clear ip nat translation *
```

2. در `Simulation` فیلتر `ICMP` و `ARP` را فعال کنید.
3. در `PC1 > Desktop > Command Prompt` بلوک زیر را paste کنید:

```text
ping 213.80.11.6
```

> **Screenshot checkpoint — `12-dynamic-public-ping.png`:** به‌محض تمام‌شدن ping عمومی و پیش از اجرای ping private تصویر بگیرید. مقصد `213.80.11.6` و نتیجه کامل باید دیده شوند.

4. پس از عبور بسته از `R5` بلوک زیر را روی `R5` paste و جدول را فوراً ثبت کنید:

```text
enable
show ip nat translations
show ip nat statistics
```

> **Screenshot checkpoint — `12-r5-dynamic-nat-table.png`:** بلافاصله بعد از اولین ping و قبل از timeout شدن entry تصویر بگیرید. `Inside local = 10.10.6.1`، آدرس اختصاص‌یافته از pool و counters باید خوانا باشند.

5. پس از عبور از `Internet`، بلوک زیر را روی `Internet` paste کنید:

```text
enable
show ip nat translations
```

> **Screenshot checkpoint — `12-internet-static-nat-table.png`:** همین‌جا از جدول Internet تصویر بگیرید. نگاشت static `192.168.0.2 <-> 213.80.11.6` و ورودی مرتبط با جریان جاری باید مشخص باشد.

در Simulation روی همان ICMP، PDU را هنگام عبور از R5 و سپس Internet باز کنید.

> **Screenshot checkpoint — `12-pdu-double-nat-before-after.png`:** پیش از اجرای ping private تصویر بگیرید. در یک قاب یا تصاویر `-a` و `-b` نشان دهید که R5 source را از `10.10.6.1` به آدرس pool و Internet مقصد را از `213.80.11.6` به `192.168.0.2` تغییر داده است؛ Event List را نیز نگه دارید.

6. سپس در `PC1 > Desktop > Command Prompt` آزمایش را برای آدرس واقعی تکرار کنید:

```text
ping 192.168.0.2
```

> **Screenshot checkpoint — `12-dynamic-private-ping.png`:** بلافاصله بعد از پایان ping private و پیش از پاک‌کردن translationها تصویر بگیرید. مقصد `192.168.0.2` و پاسخ یا timeout کامل باید دیده شوند.

### نتیجه مورد انتظار

- R5 آدرس مبدأ `PC1 = 10.10.6.1` را به یکی از آدرس‌های `213.80.11.17` تا `.30` تبدیل می‌کند؛
- برای ping آدرس عمومی، `Internet` مقصد `213.80.11.6` را به `192.168.0.2` ترجمه می‌کند؛
- چون پاسخ Server0 اکنون به یک آدرس عمومی متصل در سمت R5 برمی‌گردد، مسیر برگشت کامل می‌شود؛
- ping مستقیم `192.168.0.2` معمولاً کامل نمی‌شود. درخواست می‌تواند با source NAT روی R5 به Server0 برسد، اما پاسخ Server0 هنگام عبور inside-to-outside از `Internet` به‌علت نگاشت static با source برابر `213.80.11.6` خارج می‌شود، نه `192.168.0.2`. این عدم تقارن آدرس با echo request اولیه و state ترجمه R5 سازگار نیست؛ رفتار واقعی Packet Tracer باید ثبت شود؛
- نتیجه واقعی هر دو ping و تفاوت دو جدول باید ثبت شود.

### جدول تحلیلی مورد انتظار

| آزمایش | NAT روی R5 | NAT روی Internet |
|---|---|---|
| `PC1 -> 213.80.11.6` | ترجمه source از `10.10.6.1` به pool | ترجمه destination از `.6` به `192.168.0.2` و source پاسخ در جهت برگشت |
| `PC1 -> 192.168.0.2` | ترجمه source از `10.10.6.1` به pool؛ پاسخ نامتقارن ممکن است با state اولیه تطبیق نکند | درخواست با مقصد local وارد می‌شود؛ پاسخ inside-to-outside با source از `.2` به `213.80.11.6` ترجمه می‌شود و معمولاً ping شکست می‌خورد |

### شواهد لازم

- `12-dynamic-public-ping.png`؛
- `12-dynamic-private-ping.png`؛
- `12-r5-dynamic-nat-table.png`؛
- `12-internet-static-nat-table.png`؛
- `12-pdu-double-nat-before-after.png`.

### نیازمندی برآورده‌شده

بند ۱۲ و تمام زیربندهای آن.

---

## ۱۳. از Server0، Server1 را هنگام Dynamic NAT ping کنید

### روش اجرا

در `Server0 > Desktop > Command Prompt` بلوک زیر را paste کنید:

```text
ping 10.10.7.2
```

> **Screenshot checkpoint — `13-server0-to-server1-dynamic-failure.png`:** به‌محض پایان ping و پیش از ساخت static NAT بند ۱۸ تصویر بگیرید. فرمان، مقصد private و همه timeoutها باید دیده شوند.

برای نشان‌دادن مرز NAT، یک آزمایش دوم نیز با آدرس عمومی‌ای انجام دهید که هنوز برای Server1 تعریف نشده است؛ در این مرحله نباید نگاشت static برای Server1 وجود داشته باشد.

### نتیجه مورد انتظار

اتصال آغازشده از بیرون به `Server1` برقرار نمی‌شود. `Dynamic NAT` فقط هنگامی یک نگاشت می‌سازد که ترافیک از inside آغاز شود. بسته unsolicited از `Server0` هیچ ترجمه از پیش موجودی برای `Server1` ندارد و `Internet` نیز route به `10.10.7.0/24` ندارد.

### راستی‌آزمایی

در `R5` بررسی کنید که برای `Server1 = 10.10.7.2` نگاشت ورودی دائمی وجود ندارد. بلوک زیر را روی `R5` paste کنید:

```text
enable
show ip nat translations
```

> **Screenshot checkpoint — `13-r5-table-no-static-server1.png`:** بلافاصله پس از اجرای این فرمان تصویر بگیرید. جدول باید نشان دهد هیچ نگاشت دائمی برای `10.10.7.2` وجود ندارد؛ اگر entryهای دیگر هستند، ستون‌ها را کامل نگه دارید.

### شواهد لازم

- `13-server0-to-server1-dynamic-failure.png`؛
- `13-r5-table-no-static-server1.png`.

### نیازمندی برآورده‌شده

بند ۱۳ و سؤال «آیا اتصال برقرار است؟ چرا؟».

---

## ۱۴. Dynamic NAT را غیرفعال کنید

### روش اجرا

```text
enable
configure terminal
no ip nat inside source list 1 pool NetLab
end
clear ip nat translation *
show ip nat statistics
```

### نتیجه مورد انتظار

association مربوط به `list 1 pool NetLab` حذف و جدول ترجمه موقت خالی است. خود ACL و pool هنوز وجود دارند تا در بند ۱۵ استفاده شوند.

> **Screenshot checkpoint — `14-dynamic-nat-disabled.png`:** بلافاصله پس از اجرای بلوک بند ۱۴ و پیش از فعال‌کردن PAT تصویر بگیرید. `show ip nat statistics` باید نبود association فعال Dynamic NAT و خالی‌بودن translationها را نشان دهد.

### شاهد لازم

`14-dynamic-nat-disabled.png`.

### نیازمندی برآورده‌شده

بند ۱۴ سند اصلی.

---

## ۱۵. PAT با یک مجموعه IP را ایجاد کنید

### روش اجرا

```text
enable
configure terminal
ip nat inside source list 1 pool NetLab overload
end
clear ip nat translation *
```

> **Screenshot checkpoint — `15-pat-pool-config.png`:** بلافاصله پس از اجرای بلوک و پیش از تولید ترافیک تصویر بگیرید. بخش NAT در running configuration یا `show ip nat statistics` باید عبارت `pool NetLab overload` را نشان دهد.

از دو میزبان داخلی تقریباً هم‌زمان ترافیک ایجاد کنید.

در `PC1 > Desktop > Command Prompt` paste کنید:

```text
ping 213.80.11.6
```

در `PC3 > Desktop > Command Prompt` paste کنید:

```text
ping 213.80.11.6
```

> **Screenshot checkpoint — `15-pat-two-hosts.png`:** پس از تمام‌شدن هر دو ping و قبل از timeout شدن NAT entryها تصویر بگیرید. اگر دو Command Prompt در یک قاب جا می‌شوند هر دو را نشان دهید؛ در غیر این صورت فایل‌های `15-pat-two-hosts-a.png` و `15-pat-two-hosts-b.png` بسازید.

سپس روی `R5` paste کنید:

```text
enable
show ip nat translations
show ip nat statistics
```

> **Screenshot checkpoint — `15-pat-pool-table.png`:** بلافاصله پس از pingهای دو میزبان تصویر بگیرید. جدول باید دو جریان، آدرس‌های inside local متفاوت و استفاده اشتراکی از آدرس global به‌همراه شناسه‌های ICMP را نشان دهد.

### نتیجه مورد انتظار

چند جریان می‌توانند با کمک شناسه ICMP یا شماره پورت از یک آدرس global مشترک استفاده کنند. در جدول، ورودی‌های `icmp` همراه شناسه‌های متفاوت دیده می‌شوند.

### شواهد لازم

- `15-pat-pool-config.png`؛
- `15-pat-two-hosts.png`؛
- `15-pat-pool-table.png`.

### نیازمندی برآورده‌شده

بند ۱۵ سند اصلی.

---

## ۱۶. جدول NAT در PAT و Dynamic NAT را مقایسه کنید

### مقایسه موردنیاز

| ویژگی | Dynamic NAT | PAT با overload |
|---|---|---|
| تخصیص آدرس global | معمولاً یک آدرس global مستقل برای هر inside local فعال | چند جریان می‌توانند یک آدرس global را مشترکاً استفاده کنند |
| عامل تمایز | جفت آدرس local/global | آدرس به‌همراه شماره پورت یا شناسه ICMP |
| ظرفیت | محدود به تعداد آدرس‌های pool | بسیار بیشتر از تعداد آدرس‌های pool |
| شکل جدول | نگاشت آدرس‌محور | چند ردیف با global مشترک و شناسه/پورت متفاوت |
| پایان ظرفیت | پس از مصرف همه آدرس‌های آزاد، میزبان جدید ترجمه نمی‌شود | تا مصرف ظرفیت پورت‌ها/شناسه‌ها ادامه می‌یابد |

### روش مقایسه عملی

1. تصویر `12-r5-dynamic-nat-table.png` را کنار `15-pat-pool-table.png` قرار دهید.
2. برای هر میزبان ستون‌های `Inside local` و `Inside global` را علامت‌گذاری کنید.
3. بررسی کنید در PAT آیا یک `Inside global` برای چند جریان تکرار شده است.
4. پروتکل و شناسه بعد از علامت `:` را نیز مقایسه کنید.
5. تعداد `Total active translations` را از `show ip nat statistics` ثبت کنید.

> **Screenshot checkpoint — `16-dynamic-vs-pat-table-comparison.png`:** پس از قرار دادن تصاویر Dynamic NAT و PAT کنار هم و علامت‌گذاری ستون‌های مورد مقایسه، از نمای نهایی مقایسه تصویر بگیرید. هر دو جدول و تفاوت آدرس/identifier باید خوانا باشند.

### نتیجه مورد انتظار

در Dynamic NAT تفاوت اصلی در آدرس‌های global است؛ در PAT تمایز جریان‌ها علاوه بر آدرس با port/identifier انجام می‌شود.

### شاهد لازم

`16-dynamic-vs-pat-table-comparison.png`.

### نیازمندی برآورده‌شده

بند ۱۶ سند اصلی.

---

## ۱۷. PAT با یک IP را روی واسط خروجی اجرا کنید

### نیازمندی

PAT pool غیرفعال شود و همه میزبان‌ها از آدرس خود واسط outside استفاده کنند.

### روش اجرا

```text
enable
configure terminal
no ip nat inside source list 1 pool NetLab overload
ip nat inside source list 1 interface FastEthernet1/0 overload
end
clear ip nat translation *
```

> **Screenshot checkpoint — `17-pat-interface-config.png`:** بلافاصله پس از اجرای بلوک و پیش از pingها تصویر بگیرید. configuration یا statistics باید `interface FastEthernet1/0 overload` را نشان دهد و association قبلی pool نباید فعال باشد.

از سه میزبان داخلی ترافیک ایجاد کنید.

در `PC1 > Desktop > Command Prompt` paste کنید:

```text
ping 213.80.11.6
```

در `PC3 > Desktop > Command Prompt` paste کنید:

```text
ping 213.80.11.6
```

در `Server1 > Desktop > Command Prompt` paste کنید:

```text
ping 213.80.11.6
```

> **Screenshot checkpoint — `17-pat-interface-three-hosts.png`:** بعد از تمام‌شدن ping هر سه میزبان و پیش از timeout entryها تصویر بگیرید. اگر سه پنجره در یک قاب خوانا نیستند از پسوندهای `-a`، `-b` و `-c` استفاده کنید.

سپس روی `R5` paste کنید:

```text
enable
show ip nat translations
show ip nat statistics
```

> **Screenshot checkpoint — `17-pat-interface-table.png`:** بلافاصله پس از سه ping تصویر بگیرید. همه entryها باید `Inside global = 213.80.11.4` داشته باشند و با identifierهای متفاوت جدا شده باشند.

### نتیجه مورد انتظار

همه ترجمه‌ها از `Inside global = 213.80.11.4`، یعنی آدرس `Fa1/0`، استفاده می‌کنند و با شناسه‌های ICMP یا portهای متفاوت از هم جدا می‌شوند.

### راستی‌آزمایی

در `show ip nat statistics` بخش outside interface باید `FastEthernet1/0` را نشان دهد. اگر سند نمونه `Fa0/0` را نوشته، آن را به‌عنوان خطای نمونه توضیح دهید؛ استفاده از inside interface برای overload درست نیست.

### شواهد لازم

- `17-pat-interface-config.png`؛
- `17-pat-interface-three-hosts.png`؛
- `17-pat-interface-table.png`.

### نیازمندی برآورده‌شده

بند ۱۷ سند اصلی.

---

## ۱۸. با Static NAT روی R5 امکان ping از Server0 به Server1 را فراهم کنید

### نیازمندی

یک نگاشت دائمی روی `R5` بسازید تا `Server0` بتواند ارتباط را به سمت `Server1` آغاز کند.

### طراحی آدرس

- `Server1 Inside Local`: `10.10.7.2`؛
- یک `Inside Global` رزروشده و خارج از pool: `213.80.11.32`؛
- `.32` در subnet خارجی `/24` متصل است و در pool `.17-.30` استفاده نشده است.

### روش اجرا

1. association مربوط به PAT تک‌آدرسی را حذف کنید تا آزمایش مستقل باشد:

```text
enable
configure terminal
no ip nat inside source list 1 interface FastEthernet1/0 overload
ip nat inside source static 10.10.7.2 213.80.11.32
end
clear ip nat translation *
```

2. نگاشت static را با paste کردن بلوک زیر روی `R5` بررسی کنید:

```text
enable
show ip nat translations
show ip nat statistics
```

> **Screenshot checkpoint — `18-r5-server1-static-nat-table.png`:** پیش از اجرای ping Server0 تصویر بگیرید. نگاشت دائمی `10.10.7.2 <-> 213.80.11.32` باید در جدول R5 دیده شود.

3. در `Server0 > Desktop > Command Prompt` آدرس global را ping کنید، نه آدرس خصوصی Server1:

```text
ping 213.80.11.32
```

> **Screenshot checkpoint — `18-server0-ping-server1-global-success.png`:** بلافاصله بعد از ping موفق و پیش از بستن Command Prompt تصویر بگیرید. مقصد global و چهار reply موفق باید کامل دیده شوند.

4. در `Simulation` مسیر را دنبال کنید:
   - `Server0` بسته را به gateway یعنی `192.168.0.1` می‌دهد؛
   - `Internet` در جهت inside-to-outside، source را از `192.168.0.2` به `213.80.11.6` ترجمه می‌کند و مقصد `213.80.11.32` را روی شبکه متصل خارجی می‌بیند؛
   - `R5` برای global static پاسخ ARP می‌دهد و مقصد را از `213.80.11.32` به `10.10.7.2` تبدیل می‌کند؛
   - پاسخ Server1 در R5، source را از `10.10.7.2` به `213.80.11.32` تبدیل می‌کند؛
   - پاسخ در `Internet` در جهت outside-to-inside، destination را از `213.80.11.6` به `192.168.0.2` برمی‌گرداند.

> **Screenshot checkpoint — `18-static-nat-pdu-before-after.png`:** وقتی رفت‌وبرگشت ICMP در Simulation کامل شد تصویر بگیرید. تبدیل source روی Internet، تبدیل destination روی R5 و تبدیل‌های معکوس پاسخ را در یک قاب یا تصاویر `-a` و `-b` ثبت کنید و Event List را نگه دارید.

پس از کامل‌شدن ping و Simulation، بلوک زیر را روی `R5` paste کنید:

```text
enable
show ip nat translations
show ip nat statistics
```

> **Screenshot checkpoint — `18-r5-final-nat-statistics.png`:** این آخرین تصویر بند ۱۸ است. آن را بلافاصله بعد از بلوک بالا بگیرید تا نگاشت static، counters نهایی و نقش‌های inside/outside هم‌زمان ثبت شوند.

در نتیجه، چهار تبدیل اصلی مسیر کامل عبارت‌اند از:

```text
Internet request source: 192.168.0.2  -> 213.80.11.6
R5 request destination:   213.80.11.32 -> 10.10.7.2
R5 reply source:          10.10.7.2     -> 213.80.11.32
Internet reply destination: 213.80.11.6 -> 192.168.0.2
```

### نتیجه مورد انتظار

ping از `Server0` به `213.80.11.32` موفق می‌شود و جدول R5 نگاشت دائمی زیر را نشان می‌دهد:

```text
Inside global: 213.80.11.32
Inside local:  10.10.7.2
```

اگر ping شکست خورد، به‌ترتیب بررسی کنید:

1. gateway سرورها؛
2. route داخلی R5 به `10.10.7.0/24`؛
3. نقش‌های `ip nat inside` و `ip nat outside`؛
4. نبود conflict با pool؛
5. ARP و Event List در Simulation؛
6. ACL یا association قدیمی باقی‌مانده.

### شواهد لازم

- `18-r5-server1-static-nat-table.png`؛
- `18-server0-ping-server1-global-success.png`؛
- `18-static-nat-pdu-before-after.png`؛
- `18-r5-final-nat-statistics.png`.

### نیازمندی برآورده‌شده

بند ۱۸ سند اصلی.

---

# چک‌لیست نهایی پوشش بندهای ۱ تا ۱۸

- [ ] ۱. توپولوژی شکل ۱، آدرس‌دهی گروه ۶ و baseline مسیریابی ساخته شد.
- [ ] ۲. واسط خارجی `Internet` برابر outside است.
- [ ] ۳. واسط سمت Server0 برابر inside است.
- [ ] ۴. static mapping برای Server0 ساخته شد.
- [ ] ۵. هر دو ping عمومی و خصوصی اجرا و نتیجه توضیح داده شد.
- [ ] ۶. جدول NAT در Simulation ثبت و شکست/موفقیت Server0 به Server1 تحلیل شد.
- [ ] ۷. واسط خارجی R5 برابر outside است.
- [ ] ۸. هر دو واسط داخلی R5 برابر inside هستند.
- [ ] ۹. ACL با آدرس واقعی شبکه‌های `10.10.*` ساخته شد.
- [ ] ۱۰. pool عمومی معتبر و متصل ساخته شد.
- [ ] ۱۱. Dynamic NAT فعال شد.
- [ ] ۱۲. pingها و جدول‌های NAT هر دو روتر ثبت شدند.
- [ ] ۱۳. دسترسی آغازشده از بیرون در Dynamic NAT آزمایش و علت نتیجه بیان شد.
- [ ] ۱۴. Dynamic NAT غیرفعال و ترجمه‌های موقت پاک شد.
- [ ] ۱۵. PAT با pool اجرا شد.
- [ ] ۱۶. جدول PAT و Dynamic NAT مقایسه شد.
- [ ] ۱۷. PAT با آدرس `Fa1/0` اجرا شد.
- [ ] ۱۸. static mapping برای Server1 ساخته و ping از Server0 با آدرس global آزمایش شد.

# فهرست نهایی اسکرین‌شات‌های لازم

این فهرست را در پایان کار با پوشه `screenshots/` تطبیق دهید. هر فایل باید هم در checkpoint همان مرحله گرفته شده باشد و هم اینجا تیک بخورد. فایل‌های دارای پسوند `-a`، `-b` یا `-c` فقط وقتی لازم‌اند که اطلاعات خواسته‌شده در یک تصویر خوانا جا نشود.

## بند ۱ — آماده‌سازی توپولوژی

- [ ] `01-topology-and-addresses.png`
- [ ] `01-r5-routes-before-nat.png`
- [ ] `01-internet-routes-before-nat.png`
- [ ] `01-internal-connectivity.png`

## بندهای ۲ تا ۴ — Static NAT روی Internet

- [ ] `02-internet-outside-interface.png`
- [ ] `03-internet-inside-interface.png`
- [ ] `04-internet-static-nat-config-and-table.png`

## بند ۵ — آزمایش دو آدرس Server0

- [ ] `05-ping-server0-public.png`
- [ ] `05-ping-server0-private.png`
- [ ] `05-static-nat-pdu-before-after.png`
- [ ] `05-internet-nat-table-after-ping.png`

## بند ۶ — جدول NAT و شکست دسترسی اولیه Server0 به Server1

- [ ] `06-internet-nat-table-during-static-test.png`
- [ ] `06-server0-to-server1-before-r5-nat.png`
- [ ] `06-internet-route-explaining-failure.png`

## بندهای ۷ تا ۱۱ — آماده‌سازی Dynamic NAT روی R5

- [ ] `07-r5-outside-interface.png`
- [ ] `08-r5-inside-interfaces.png`
- [ ] `09-r5-nat-acl.png`
- [ ] `10-r5-netlab-pool.png`
- [ ] `11-r5-dynamic-nat-config.png`

## بند ۱۲ — اجرای Dynamic NAT

- [ ] `12-dynamic-public-ping.png`
- [ ] `12-dynamic-private-ping.png`
- [ ] `12-r5-dynamic-nat-table.png`
- [ ] `12-internet-static-nat-table.png`
- [ ] `12-pdu-double-nat-before-after.png`

## بندهای ۱۳ و ۱۴ — بررسی و غیرفعال‌کردن Dynamic NAT

- [ ] `13-server0-to-server1-dynamic-failure.png`
- [ ] `13-r5-table-no-static-server1.png`
- [ ] `14-dynamic-nat-disabled.png`

## بندهای ۱۵ و ۱۶ — PAT با pool و مقایسه

- [ ] `15-pat-pool-config.png`
- [ ] `15-pat-two-hosts.png`
- [ ] `15-pat-pool-table.png`
- [ ] `16-dynamic-vs-pat-table-comparison.png`

## بند ۱۷ — PAT با IP واسط

- [ ] `17-pat-interface-config.png`
- [ ] `17-pat-interface-three-hosts.png`
- [ ] `17-pat-interface-table.png`

## بند ۱۸ — Static NAT برای Server1

- [ ] `18-r5-server1-static-nat-table.png`
- [ ] `18-server0-ping-server1-global-success.png`
- [ ] `18-static-nat-pdu-before-after.png`
- [ ] `18-r5-final-nat-statistics.png`

**تعداد پایه مورد انتظار: ۳۸ اسکرین‌شات.** تصاویر اضافه با پسوندهای `-a`، `-b` و `-c` در این تعداد محاسبه نشده‌اند.

# نکات مهم

## ارجاع به منابع

طبق سند اصلی، اگر برای پاسخ به سؤال‌ها از هر منبعی استفاده شود، ارجاع به آن الزامی است. در گزارش نهایی، فرمان‌ها یا توضیح‌هایی که از منبع بیرونی گرفته شده‌اند باید منبع داشته باشند.

## بسته تحویل

1. عبارت صفحه ۶ عیناً چنین است: «تمام فایل‌های مربوط به آزمایش‌ها را در یک فایل zip قرار داده و ارسال کنید.» برای تحویل این تکلیف، این قاعده به‌صورت گردآوری همه فایل‌های مربوط به آزمایش ششم در یک ZIP اجرا می‌شود؛ این محدودسازی یک تفسیر اجرایی است، نه جایگزینی متن اصلی.
2. فایل Packet Tracer نهایی را قبل از ZIP باز کنید و از سالم‌بودن آن مطمئن شوید.
3. گزارش نهایی تکمیل‌شده را نیز داخل بسته قرار دهید. وجود فقط فایل `.pkt` و تصاویر، الزام گزارش را کامل نمی‌کند.
4. ساختار زیر پیشنهادی و اختیاری است؛ اگر استاد ساختار دیگری اعلام کرد، دستور او اولویت دارد:

```text
KiarashShojaei-6-2-6/
├── KiarashShojaei-6-2-6.pkt
├── report.pdf
├── report.tex
└── screenshots/
    ├── 01-...
    ├── 02-...
    └── 18-...
```

5. پوشه را با نام زیر ZIP کنید:

```text
KiarashShojaei-6-2-6.zip
```

6. نام رسمی کلی سند `FullName-GroupNum-ClassNum-ExperimentNum.zip` است.

## راه تماس درج‌شده در سند

```text
alireza.mahmoudi18@sharif.edu
```

# ممیزی نهایی قبل از تحویل

1. تمام ۱۸ بند تیک خورده باشند.
2. هیچ route داخلی `10.10.*` روی `Internet` باقی نمانده باشد.
3. همه roleهای inside/outside با اتصال فیزیکی واقعی منطبق باشند.
4. برای هر حالت NAT، association قبلی حذف و translationهای قدیمی پاک شده باشند.
5. آدرس pool شامل network یا broadcast نباشد.
6. در PAT تک‌آدرسی، interface واقعی outside یعنی `Fa1/0` استفاده شده باشد.
7. نتیجه‌های ثبت‌شده، خروجی واقعی Packet Tracer باشند.
8. تصاویر لازم خوانا و نام‌گذاری‌شده باشند.
9. فایل `.pkt` نهایی باز شود و تمام لینک‌ها و تنظیمات ذخیره شده باشند.
10. ZIP نهایی از الگوی نام‌گذاری سند اصلی پیروی کند.
