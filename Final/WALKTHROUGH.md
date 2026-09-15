# راهنمای کامل آزمون پایان‌ترم آزمایشگاه شبکه

این راهنما برای ساخت سناریوی نهایی در **Cisco Packet Tracer** نوشته شده است. همه فرمان‌ها بدون Prompt هستند و هر بلوک فقط روی دستگاهی اجرا می‌شود که بلافاصله بالای آن نام برده شده است.

## خروجی‌های رسمی آزمون

طبق صفحه نخست صورت سؤال:

- آزمون انفرادی است و پروژه باید مستقل ساخته شود.
- فقط فایل شبیه‌سازی Packet Tracer با پسوند `.pkt` و ویدیوی اجرای پروژه تحویل داده می‌شوند.
- تهیه گزارش متنی Word یا PDF لازم نیست.
- در ابتدای ویدیو ساختار سناریو کوتاه معرفی و سپس تمام تست‌های اجباری به‌ترتیب اجرا شوند.
- مدت پیشنهادی ویدیو حداکثر ۱۰ دقیقه است.
- مهلت درج‌شده در صورت سؤال پایان روز شنبه ۱۳ تیر است.

Checkpointهای PNG این راهنما فقط برای کنترل شخصی و آماده‌سازی ویدیو هستند و جزو فایل‌های رسمی تحویل نیستند. هر Checkpoint که چند دستگاه یا چند پنجره دارد می‌تواند با پسوندهای `-a`، `-b` و `-c` به چند تصویر خوانا تقسیم شود؛ نام پایه و تعداد ۳۲ Checkpoint تغییر نمی‌کند.

---

# ۱. تفسیر عملی دو تناقض آدرس‌دهی صورت سؤال

دو مقدار ماسک در PDF با بقیه الزام‌ها ناسازگارند. راه‌حل زیر یک **تفسیر عملی برای Packet Tracer** است و اجرای لفظ‌به‌لفظ همان دو ماسک نیست؛ زیرا اجرای لفظی، تست‌های اجباری را غیرممکن می‌کند.

صورت سؤال از آدرس‌های زیر استفاده می‌کند:

- لینک `ISP ↔ R-EDGE`: آدرس‌های `203.0.113.1` و `203.0.113.2`
- آدرس عمومی PAT: `203.0.113.10`
- آدرس عمومی وب: `203.0.113.20`
- کاربر اینترنت: `203.0.113.50/24`

اگر لینک ISP با ماسک `/30` ساخته شود، آدرس‌های `.10`، `.20` و `.50` خارج از آن زیرشبکه خواهند بود و آزمایش‌های اجباری NAT در Packet Tracer به‌صورت مستقیم کار نمی‌کنند. برای حفظ تمام IPهای الزامی و اجرای واقعی آزمایش‌ها، بخش خارجی در این راهنما یک شبکه `/24` است:

```text
203.0.113.0/24
```

همچنین استفاده از `10.10.0.1/16` روی لینک `R-EDGE ↔ SW-CORE` با SVIهای `10.10.x.0/28` روی SW-CORE هم‌پوشانی ایجاد می‌کند. لینک مسیریابی با حفظ IP داده‌شده R-EDGE به‌صورت زیر ساخته می‌شود:

```text
R-EDGE Gi0/0/1 = 10.10.0.1/30
SW-CORE Gi0/1 = 10.10.0.2/30
```

این دو اصلاح فقط ماسک لینک‌ها را سازگار می‌کنند؛ همه آدرس‌های VLAN، سرورها، NAT و کاربر اینترنت دقیقاً مطابق صورت سؤال باقی می‌مانند.

---

# ۲. تجهیزات موردنیاز

در یک پروژه خالی این تجهیزات را قرار دهید:

- دو روتر **Cisco ISR 4321** با نام‌های `R-EDGE` و `ISP`
- یک سوئیچ **3560-24PS** با نام `SW-CORE`
- چهار سوئیچ **2960-24TT** با نام‌های `SW-F1`، `SW-F2`، `SW-F3` و `INTERNET-SW`
- هجده PC داخلی، دو PC برای هر تیم
- یک PC با نام `Internet-User`
- پنج Server-PT با نام‌های `WEB1`، `WEB2`، `TEST`، `DNS` و `DNS-ISP`

نام PCها:

| طبقه | VLAN | PC اول | PC دوم |
|---|---:|---|---|
| طبقه ۱ | 10 | `PC-S1` | `PC-S2` |
| طبقه ۱ | 11 | `PC-M1` | `PC-M2` |
| طبقه ۱ | 12 | `PC-U1` | `PC-U2` |
| طبقه ۲ | 20 | `PC-B1` | `PC-B2` |
| طبقه ۲ | 21 | `PC-F1` | `PC-F2` |
| طبقه ۲ | 22 | `PC-Q1` | `PC-Q2` |
| طبقه ۳ | 30 | `PC-FN1` | `PC-FN2` |
| طبقه ۳ | 31 | `PC-H1` | `PC-H2` |
| طبقه ۳ | 32 | `PC-N1` | `PC-N2` |

## اتصال کابل‌ها

از **Automatically Choose Connection Type** استفاده کنید. سوئیچ `INTERNET-SW` فقط نقش بخش چنددسترسی Cloud/ISP شکل را در Packet Tracer بازی می‌کند و جزئی از Campus شرکت نیست.

| مبدأ | پورت | مقصد | پورت |
|---|---|---|---|
| R-EDGE | `Gi0/0/0` | INTERNET-SW | `Gi0/1` |
| ISP | `Gi0/0/0` | INTERNET-SW | `Gi0/2` |
| Internet-User | `Fa0` | INTERNET-SW | `Fa0/1` |
| DNS-ISP | `Fa0` | INTERNET-SW | `Fa0/2` |
| R-EDGE | `Gi0/0/1` | SW-CORE | `Gi0/1` |
| SW-CORE | `Fa0/21` | SW-F1 | `Gi0/1` |
| SW-CORE | `Fa0/22` | SW-F2 | `Gi0/1` |
| SW-CORE | `Fa0/23` | SW-F3 | `Gi0/1` |
| WEB1 | `Fa0` | SW-CORE | `Fa0/1` |
| WEB2 | `Fa0` | SW-CORE | `Fa0/2` |
| TEST | `Fa0` | SW-CORE | `Fa0/3` |
| DNS | `Fa0` | SW-CORE | `Fa0/4` |

روی هر Access Switch، شش PC را به‌ترتیب به `Fa0/1` تا `Fa0/6` متصل کنید.

**Checkpoint `F-01-topology.png`:** از کل توپولوژی عکس بگیرید؛ نام همه دستگاه‌ها و کابل‌های بین Edge، Core و سه Access Switch خوانا باشند.

---

# ۳. جدول نهایی آدرس‌دهی

| VLAN | نام | شبکه | Gateway | روش کلاینت‌ها |
|---:|---|---|---|---|
| 10 | Sales | `10.10.10.0/28` | `10.10.10.1` | DHCP |
| 11 | Marketing | `10.10.11.0/28` | `10.10.11.1` | DHCP |
| 12 | Support | `10.10.12.0/28` | `10.10.12.1` | DHCP |
| 20 | Backend | `10.10.20.0/28` | `10.10.20.1` | DHCP |
| 21 | Frontend | `10.10.21.0/28` | `10.10.21.1` | DHCP |
| 22 | QA | `10.10.22.0/28` | `10.10.22.1` | DHCP |
| 30 | Finance | `10.10.30.0/28` | `10.10.30.1` | DHCP |
| 31 | HR | `10.10.31.0/28` | `10.10.31.1` | DHCP |
| 32 | Network | `10.10.32.0/28` | `10.10.32.1` | Static برای دو عضو تیم شبکه |
| 99 | Management | `10.10.99.0/28` | `10.10.99.1` | Static برای سوئیچ‌ها |
| 100 | Server Farm | `10.10.100.0/28` | `10.10.100.1` | Static |
| 999 | Native | بدون IP | بدون Gateway | فقط Trunk |

آدرس‌های ثابت:

| دستگاه | IP | Mask | Gateway | DNS |
|---|---|---|---|---|
| WEB1 | `10.10.100.10` | `255.255.255.240` | `10.10.100.1` | `10.10.100.13` |
| WEB2 | `10.10.100.11` | `255.255.255.240` | `10.10.100.1` | `10.10.100.13` |
| TEST | `10.10.100.12` | `255.255.255.240` | `10.10.100.1` | `10.10.100.13` |
| DNS | `10.10.100.13` | `255.255.255.240` | `10.10.100.1` | `10.10.100.13` |
| PC-N1 | `10.10.32.10` | `255.255.255.240` | `10.10.32.1` | `10.10.100.13` |
| PC-N2 | `10.10.32.11` | `255.255.255.240` | `10.10.32.1` | `10.10.100.13` |
| Internet-User | `203.0.113.50` | `255.255.255.0` | `203.0.113.1` | `203.0.113.53` |
| DNS-ISP | `203.0.113.53` | `255.255.255.0` | `203.0.113.1` | `203.0.113.53` |

---

# ۴. پیکربندی ISP و R-EDGE

## دستگاه: ISP

```text
enable
configure terminal
hostname ISP
interface GigabitEthernet0/0/0
 ip address 203.0.113.1 255.255.255.0
 no shutdown
exit
ip route 10.10.0.0 255.255.0.0 203.0.113.2
end
write memory
```

مسیر `10.10.0.0/16` فقط برای آن است که تست اجباری دسترسی Internet-User به TEST واقعاً تا ACL خارجی R-EDGE برسد و شمارنده deny افزایش یابد.

## دستگاه: R-EDGE

```text
enable
configure terminal
hostname R-EDGE
interface GigabitEthernet0/0/0
 description OUTSIDE-TO-ISP
 ip address 203.0.113.2 255.255.255.0
 ip nat outside
 no shutdown
exit
interface GigabitEthernet0/0/1
 description INSIDE-TO-SW-CORE
 ip address 10.10.0.1 255.255.255.252
 ip nat inside
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 203.0.113.1
end
write memory
```

## دستگاه: R-EDGE — بررسی اولیه

```text
enable
show ip interface brief
show ip route
```

انتظار می‌رود هر دو رابط `up/up` باشند و مسیر پیش‌فرض به `203.0.113.1` دیده شود.

**Checkpoint `F-02-edge-isp-links.png`:** خروجی `show ip interface brief` و مسیر پیش‌فرض R-EDGE را ثبت کنید.

---

# ۵. ساخت VLANها، SVIها و Trunkها

## دستگاه: SW-CORE

```text
enable
configure terminal
hostname SW-CORE
ip routing
vlan 10
 name Sales
vlan 11
 name Marketing
vlan 12
 name Support
vlan 20
 name Backend
vlan 21
 name Frontend
vlan 22
 name QA
vlan 30
 name Finance
vlan 31
 name HR
vlan 32
 name Network
vlan 99
 name Management
vlan 100
 name Server-Farm
vlan 999
 name Native
interface range FastEthernet0/1 - 4
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast
 no shutdown
exit
interface FastEthernet0/21
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,11,12,99,999
 no shutdown
exit
interface FastEthernet0/22
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,21,22,99,999
 no shutdown
exit
interface FastEthernet0/23
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,31,32,99,999
 no shutdown
exit
interface Vlan10
 ip address 10.10.10.1 255.255.255.240
 no shutdown
exit
interface Vlan11
 ip address 10.10.11.1 255.255.255.240
 no shutdown
exit
interface Vlan12
 ip address 10.10.12.1 255.255.255.240
 no shutdown
exit
interface Vlan20
 ip address 10.10.20.1 255.255.255.240
 no shutdown
exit
interface Vlan21
 ip address 10.10.21.1 255.255.255.240
 no shutdown
exit
interface Vlan22
 ip address 10.10.22.1 255.255.255.240
 no shutdown
exit
interface Vlan30
 ip address 10.10.30.1 255.255.255.240
 no shutdown
exit
interface Vlan31
 ip address 10.10.31.1 255.255.255.240
 no shutdown
exit
interface Vlan32
 ip address 10.10.32.1 255.255.255.240
 no shutdown
exit
interface Vlan99
 ip address 10.10.99.1 255.255.255.240
 no shutdown
exit
interface Vlan100
 ip address 10.10.100.1 255.255.255.240
 no shutdown
exit
interface GigabitEthernet0/1
 no switchport
 ip address 10.10.0.2 255.255.255.252
 no shutdown
exit
end
write memory
```

## دستگاه: SW-F1

```text
enable
configure terminal
hostname SW-F1
vlan 10
 name Sales
vlan 11
 name Marketing
vlan 12
 name Support
vlan 99
 name Management
vlan 999
 name Native
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit
interface range FastEthernet0/3 - 4
 switchport mode access
 switchport access vlan 11
 spanning-tree portfast
exit
interface range FastEthernet0/5 - 6
 switchport mode access
 switchport access vlan 12
 spanning-tree portfast
exit
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,11,12,99,999
 no shutdown
exit
interface Vlan99
 ip address 10.10.99.11 255.255.255.240
 no shutdown
exit
ip default-gateway 10.10.99.1
end
write memory
```

## دستگاه: SW-F2

```text
enable
configure terminal
hostname SW-F2
vlan 20
 name Backend
vlan 21
 name Frontend
vlan 22
 name QA
vlan 99
 name Management
vlan 999
 name Native
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit
interface range FastEthernet0/3 - 4
 switchport mode access
 switchport access vlan 21
 spanning-tree portfast
exit
interface range FastEthernet0/5 - 6
 switchport mode access
 switchport access vlan 22
 spanning-tree portfast
exit
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,21,22,99,999
 no shutdown
exit
interface Vlan99
 ip address 10.10.99.12 255.255.255.240
 no shutdown
exit
ip default-gateway 10.10.99.1
end
write memory
```

## دستگاه: SW-F3

```text
enable
configure terminal
hostname SW-F3
vlan 30
 name Finance
vlan 31
 name HR
vlan 32
 name Network
vlan 99
 name Management
vlan 999
 name Native
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit
interface range FastEthernet0/3 - 4
 switchport mode access
 switchport access vlan 31
 spanning-tree portfast
exit
interface range FastEthernet0/5 - 6
 switchport mode access
 switchport access vlan 32
 spanning-tree portfast
exit
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,31,32,99,999
 no shutdown
exit
interface Vlan99
 ip address 10.10.99.13 255.255.255.240
 no shutdown
exit
ip default-gateway 10.10.99.1
end
write memory
```

## دستگاه: SW-CORE — بررسی VLAN و Trunk

```text
enable
show vlan brief
show interfaces trunk
show ip interface brief
```

**Checkpoint `F-03-core-vlans.png`:** همه VLANهای ۱۰، ۱۱، ۱۲، ۲۰، ۲۱، ۲۲، ۳۰، ۳۱، ۳۲، ۹۹، ۱۰۰ و ۹۹۹ را نشان دهید.

**Checkpoint `F-04-core-trunks.png`:** سه Trunk فعال و Native VLAN برابر 999 را نشان دهید.

**Checkpoint `F-05-core-svis.png`:** همه SVIها و رابط routed متصل به R-EDGE را در خروجی ثبت کنید.

---

# ۶. DHCP روی SW-CORE

## دستگاه: SW-CORE

```text
enable
configure terminal
ip dhcp excluded-address 10.10.10.1
ip dhcp excluded-address 10.10.11.1
ip dhcp excluded-address 10.10.12.1
ip dhcp excluded-address 10.10.20.1
ip dhcp excluded-address 10.10.21.1
ip dhcp excluded-address 10.10.22.1
ip dhcp excluded-address 10.10.30.1
ip dhcp excluded-address 10.10.31.1
ip dhcp excluded-address 10.10.32.1 10.10.32.11
ip dhcp excluded-address 10.10.100.1 10.10.100.13
ip dhcp pool SALES
 network 10.10.10.0 255.255.255.240
 default-router 10.10.10.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool MARKETING
 network 10.10.11.0 255.255.255.240
 default-router 10.10.11.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool SUPPORT
 network 10.10.12.0 255.255.255.240
 default-router 10.10.12.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool BACKEND
 network 10.10.20.0 255.255.255.240
 default-router 10.10.20.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool FRONTEND
 network 10.10.21.0 255.255.255.240
 default-router 10.10.21.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool QA
 network 10.10.22.0 255.255.255.240
 default-router 10.10.22.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool FINANCE
 network 10.10.30.0 255.255.255.240
 default-router 10.10.30.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool HR
 network 10.10.31.0 255.255.255.240
 default-router 10.10.31.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool NETWORK
 network 10.10.32.0 255.255.255.240
 default-router 10.10.32.1
 dns-server 10.10.100.13
 lease 1
exit
ip dhcp pool SERVER-FARM
 network 10.10.100.0 255.255.255.240
 default-router 10.10.100.1
 dns-server 10.10.100.13
 lease 1
exit
end
write memory
```

Pool مربوط به Server Farm برای رعایت عبارت «برای هر VLAN یک Pool» ساخته شده است، اما آدرس‌های `.1` تا `.13` از آن کنار گذاشته شده‌اند و چهار سرور حتماً Static باقی می‌مانند. برای VLAN مدیریت Pool ساخته نشده است، زیرا این VLAN اختیاری است و فقط تجهیزات با آدرس ثابت از آن استفاده می‌کنند.

برای هر VLAN ابتدا PC اول جدول نام‌گذاری را روی DHCP قرار دهید و فقط پس از دریافت آدرس، PC دوم همان VLAN را فعال کنید. این ترتیب در یک پروژه تازه باعث می‌شود PC اول آدرس `.2` و PC دوم آدرس `.3` را بگیرد. برای تمام PCهای VLANهای ۱۰، ۱۱، ۱۲، ۲۰، ۲۱، ۲۲، ۳۰ و ۳۱:

1. `Desktop > IP Configuration`
2. گزینه **DHCP** را انتخاب کنید.
3. تا نمایش IP، Gateway و DNS صبر کنید.

در VLAN 22 حتماً `PC-Q1` را پیش از `PC-Q2` روی DHCP قرار دهید و در Checkpoint همان‌جا تأیید کنید که `PC-Q1` آدرس `10.10.22.2` گرفته است. اگر پروژه از قبل Lease دارد، پیش از ادامه یک پروژه تازه بسازید یا Leaseهای قبلی را پاک کنید؛ تست‌های پایین بر مبنای این ترتیب قطعی نوشته شده‌اند.

برای `PC-N1` و `PC-N2` از جدول آدرس‌های ثابت بخش ۳ استفاده کنید.

## دستگاه: SW-CORE — بررسی DHCP

```text
enable
show ip dhcp binding
show ip dhcp pool
```

**Checkpoint `F-06-dhcp-bindings.png`:** Leaseهای VLANهای مختلف و آدرس‌های MAC آن‌ها را ثبت کنید.

**Checkpoint `F-07-two-client-ipconfig.png`:** روی `PC-S1` و `PC-Q1` دستور `ipconfig` را اجرا و IP، Gateway و DNS را ثبت کنید. برای خوانایی، نام پایه را با پسوندهای `-a` برای PC-S1 و `-b` برای PC-Q1 تقسیم کنید.

---

# ۷. آدرس‌دهی و سرویس سرورها

روی هر سرور به `Desktop > IP Configuration` بروید و مقادیر جدول بخش ۳ را وارد کنید.

## WEB1

1. `Services > HTTP`
2. HTTP را **On** کنید.
3. فایل `index.html` را ویرایش کنید تا عبارت `NetLab WEB1 Production` واضح باشد.

## WEB2

1. `Services > HTTP`
2. HTTP را **On** کنید.
3. فایل `index.html` را ویرایش کنید تا عبارت `NetLab WEB2 Backup Portal` واضح باشد.

## TEST

1. `Services > HTTP`
2. HTTP را **On** کنید.
3. فایل `index.html` را ویرایش کنید تا عبارت `NetLab Internal TEST Server` واضح باشد.

**Checkpoint `F-08-server-addresses.png`:** IP ثابت هر چهار سرور داخلی را در دو یا چند نما ثبت کنید؛ `.10` تا `.13` و Gateway `.1` باید خوانا باشند.

---

# ۸. DNS داخلی و خارجی

## دستگاه: DNS

1. به `Services > DNS` بروید.
2. سرویس DNS را **On** کنید.
3. رکوردهای زیر را یکی‌یکی با نوع `A Record` اضافه کنید:

| Name | Address |
|---|---|
| `netlab.ir` | `203.0.113.20` |
| `www.netlab.ir` | `10.10.100.10` |
| `backup.netlab.ir` | `10.10.100.11` |
| `test.netlab.ir` | `10.10.100.12` |
| `ns.netlab.ir` | `10.10.100.13` |

این رکوردها Split DNS داخلی را می‌سازند؛ `www` و `backup` از داخل مستقیماً به IP خصوصی می‌روند و به Hairpin NAT وابسته نیستند.

## دستگاه: DNS-ISP

1. در `Desktop > IP Configuration` آدرس `203.0.113.53/24`، Gateway برابر `203.0.113.1` و DNS برابر خودش را تنظیم کنید.
2. به `Services > DNS` بروید و DNS را **On** کنید.
3. رکوردهای زیر را اضافه کنید:

| Name | Address |
|---|---|
| `netlab.ir` | `203.0.113.20` |
| `www.netlab.ir` | `203.0.113.20` |
| `backup.netlab.ir` | `203.0.113.20` |
| `www.techcorp.ir` | `203.0.113.20` |
| `backup.techcorp.ir` | `203.0.113.20` |

صورت سؤال در جدول DNS از `netlab.ir` استفاده می‌کند، اما در یک بند NAT نام `techcorp.ir` آمده است. تعریف هر دو نام، این تناقض متنی را بدون تغییر مقصد عمومی حل می‌کند. برای TEST هیچ رکورد خارجی ایجاد نکنید.

## دستگاه: Internet-User

در `Desktop > IP Configuration` وارد کنید:

```text
IP Address: 203.0.113.50
Subnet Mask: 255.255.255.0
Default Gateway: 203.0.113.1
DNS Server: 203.0.113.53
```

**Checkpoint `F-09-internal-dns-records.png`:** پنج رکورد DNS داخلی را ثبت کنید.

**Checkpoint `F-10-external-dns-records.png`:** رکوردهای عمومی و نبود رکورد TEST را ثبت کنید.

---

# ۹. OSPF Area 0

## دستگاه: SW-CORE

```text
enable
configure terminal
router ospf 1
 router-id 1.1.1.1
 passive-interface default
 no passive-interface GigabitEthernet0/1
 network 10.10.0.0 0.0.255.255 area 0
exit
end
write memory
```

## دستگاه: R-EDGE

```text
enable
configure terminal
router ospf 1
 router-id 2.2.2.2
 passive-interface default
 no passive-interface GigabitEthernet0/0/1
 network 10.10.0.0 0.0.0.3 area 0
 default-information originate
exit
end
write memory
```

## دستگاه: SW-CORE — بررسی OSPF

```text
enable
show ip ospf neighbor
show ip route
```

باید همسایه `2.2.2.2` در وضعیت `FULL` و مسیر پیش‌فرض OSPF با علامت `O*E2` دیده شود.

## دستگاه: R-EDGE — بررسی مسیرهای داخلی

```text
enable
show ip ospf neighbor
show ip route ospf
```

باید مسیر VLANهای داخلی با حرف `O` دیده شوند.

**Checkpoint `F-11-ospf-neighbor.png`:** همسایگی FULL روی SW-CORE را ثبت کنید.

**Checkpoint `F-12-core-routing-table.png`:** مسیر پیش‌فرض و شبکه‌های متصل را ثبت کنید.

---

# ۱۰. PAT و Port Forwarding روی R-EDGE

## دستگاه: R-EDGE

```text
enable
configure terminal
ip access-list extended PAT-USERS
 permit ip 10.10.10.0 0.0.0.15 any
 permit ip 10.10.11.0 0.0.0.15 any
 permit ip 10.10.12.0 0.0.0.15 any
 permit ip 10.10.20.0 0.0.0.15 any
 permit ip 10.10.21.0 0.0.0.15 any
 permit ip 10.10.22.0 0.0.0.15 any
 permit tcp 10.10.30.0 0.0.0.15 any eq www
 permit udp 10.10.30.0 0.0.0.15 any eq domain
 permit tcp 10.10.30.0 0.0.0.15 any eq domain
 permit ip 10.10.31.0 0.0.0.15 any
 permit ip 10.10.32.0 0.0.0.15 any
exit
ip nat pool EMPLOYEE-PAT 203.0.113.10 203.0.113.10 netmask 255.255.255.0
ip nat inside source list PAT-USERS pool EMPLOYEE-PAT overload
ip nat inside source static tcp 10.10.100.10 80 203.0.113.20 80
ip nat inside source static tcp 10.10.100.11 80 203.0.113.20 8080
end
write memory
```

ACL مربوط به NAT برای همه VLANهای کاربری تعریف شده است، نه برای Server Farm، Management یا لینک Transit. کاربران Finance در پیکربندی پایه فقط ترافیک HTTP و DNS را می‌توانند از طریق PAT به اینترنت بفرستند؛ بنابراین سیاست «دسترسی محدود» حتی بدون بخش امتیازی زمان‌دار اجرا شده است.

هیچ NAT یا Port Forwarding برای `10.10.100.12` تعریف نکنید.

## دستگاه: PC-S1 — تولید PAT

```text
ping 203.0.113.1
```

## دستگاه: R-EDGE — بررسی NAT

```text
enable
show ip nat translations
show ip nat statistics
```

باید ترجمه PAT با Inside Local متعلق به PC-S1 و Inside Global برابر `203.0.113.10` دیده شود. دو نگاشت ثابت TCP نیز باید همیشه وجود داشته باشند.

**Checkpoint `F-13-pat-translation.png`:** بلافاصله پس از Ping، رکورد PAT را ثبت کنید.

**Checkpoint `F-14-static-port-forwarding.png`:** دو نگاشت WEB1 روی پورت ۸۰ و WEB2 روی پورت ۸۰۸۰ را ثبت کنید.

---

# ۱۱. ACL داخلی سرورها

در صورت سؤال عبارت «ACL-SERVERS-IN روی VLAN 100 به‌صورت Inbound» آمده است. روی SVI، جهت `in` ترافیکی را بررسی می‌کند که **از سرورها وارد SW-CORE** می‌شود؛ در حالی که قوانین خواسته‌شده باید درخواست‌های کاربران را **به مقصد سرورها** کنترل کنند. برای اجرای واقعی سیاست، ACL روی `Vlan100` در جهت `out` اعمال می‌شود. این انتخاب با عبارت جهت در PDF یکسان نیست، اما استفاده از `in` امکان اجرای آزمون‌های QA، غیر QA و مدیریت را با ACL مقصدگرا از بین می‌برد. این تناقض را در توضیح ویدیو کوتاه و صریح بیان کنید.

## دستگاه: SW-CORE

```text
enable
configure terminal
ip access-list extended ACL-SERVERS-IN
 permit udp any host 10.10.100.13 eq domain
 permit tcp any host 10.10.100.13 eq domain
 permit tcp any host 10.10.100.10 eq www
 permit tcp any host 10.10.100.11 eq www
 permit icmp 10.10.10.0 0.0.0.15 host 10.10.100.10 echo
 permit tcp 10.10.22.0 0.0.0.15 host 10.10.100.12 eq www
 permit ip 10.10.32.0 0.0.0.15 10.10.100.0 0.0.0.15
 deny tcp any 10.10.100.0 0.0.0.15 eq 22
 deny tcp any 10.10.100.0 0.0.0.15 eq telnet
 deny ip 10.10.30.0 0.0.0.15 10.10.100.0 0.0.0.15
 deny ip any host 10.10.100.12
 deny ip any 10.10.100.0 0.0.0.15
 permit ip any any
exit
interface Vlan100
 ip access-group ACL-SERVERS-IN out
exit
end
write memory
```

قاعده ICMP از VLAN 10 به WEB1 فقط برای سازگارشدن با تست اجباری Ping صفحه ۱۲ صورت سؤال اضافه شده است. سایر کاربران عادی فقط HTTP و DNS مجاز را دریافت می‌کنند.

**Checkpoint `F-15-server-acl-config.png`:** ترتیب ACL و اتصال آن به `Vlan100 out` را ثبت کنید.

---

# ۱۲. ACL مرزی اینترنت

## دستگاه: R-EDGE

```text
enable
configure terminal
ip access-list extended INTERNET-IN
 permit tcp any host 203.0.113.20 eq www
 permit tcp any host 203.0.113.20 eq 8080
 permit tcp any host 203.0.113.10 established
 permit udp any host 203.0.113.10 gt 1023
 permit icmp any host 203.0.113.10 echo-reply
 deny ip any 10.10.0.0 0.0.255.255
 deny ip any host 203.0.113.20
 deny ip any host 203.0.113.10
 deny ip any any
exit
interface GigabitEthernet0/0/0
 ip access-group INTERNET-IN in
exit
end
write memory
```

دو Permit نخست فقط وب عمومی را باز می‌کنند. سه Permit بعدی پاسخ‌های لازم برای ارتباط‌های PAT آغازشده از داخل را عبور می‌دهند. TEST هیچ نگاشت عمومی ندارد و ترافیک مستقیم به شبکه خصوصی نیز رد می‌شود.

**Checkpoint `F-16-internet-acl-config.png`:** ACL خارجی و اتصال inbound آن به `Gi0/0/0` را ثبت کنید.

---

# ۱۳. مدیریت SSH/Telnet فقط برای Network Team

Server-PT استاندارد در بسیاری از نسخه‌های Packet Tracer سرویس SSH/Telnet قابل فعال‌سازی ندارد، در حالی که صورت سؤال اتصال به «همان سرور» را می‌خواهد و شکل WEB1 را با HTTP+SSH نشان می‌دهد. این یک محدودیت حل‌ناشدنی مدل Server-PT است و نباید در ویدیو به‌عنوان پیاده‌سازی کامل سرور معرفی شود.

اگر در `WEB1 > Services` گزینه SSH موجود است، آن را On کنید، کاربر `netadmin` با رمز `NetLab32Pass` بسازید و تست‌های عادی و Network Team را مستقیماً با `10.10.100.10` انجام دهید. اگر این گزینه وجود ندارد، اثبات را در دو بخش انجام دهید:

1. در Simulation Mode با **Add Complex PDU** یک بسته TCP با Destination Port برابر 22 از PC-S1 به WEB1 بسازید؛ بسته باید روی SW-CORE توسط ACL حذف شود.
2. همان بسته را از PC-N1 به WEB1 بسازید؛ ACL باید آن را تا WEB1 عبور دهد، هرچند خود Server-PT به علت نبود سرویس SSH اتصال کاربردی را کامل نمی‌کند.
3. برای نشان‌دادن یک نشست مدیریتی واقعی، اتصال SSH عادی و Network Team را به مقصد یکسان R-EDGE مقایسه کنید.

این روش هم تصمیم ACL به مقصد سرور را نشان می‌دهد و هم محدودیت شبیه‌ساز را صادقانه از موفقیت نشست SSH جدا می‌کند. سیاست دسترسی مدیریتی تجهیزات شبکه با بلوک‌های زیر اعمال می‌شود.

## دستگاه: R-EDGE

```text
enable
configure terminal
ip domain-name netlab.ir
username netadmin privilege 15 secret NetLab32Pass
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
ip access-list standard MGMT-VTY
 permit 10.10.32.0 0.0.0.15
 deny any
exit
line vty 0 4
 login local
 transport input ssh telnet
 access-class MGMT-VTY in
exit
end
write memory
```

## دستگاه: SW-CORE

```text
enable
configure terminal
ip domain-name netlab.ir
username netadmin privilege 15 secret NetLab32Pass
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
ip access-list standard MGMT-VTY
 permit 10.10.32.0 0.0.0.15
 deny any
exit
line vty 0 15
 login local
 transport input ssh telnet
 access-class MGMT-VTY in
exit
end
write memory
```

## دستگاه: SW-F1

```text
enable
configure terminal
ip domain-name netlab.ir
username netadmin privilege 15 secret NetLab32Pass
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
ip access-list standard MGMT-VTY
 permit 10.10.32.0 0.0.0.15
 deny any
exit
line vty 0 15
 login local
 transport input ssh telnet
 access-class MGMT-VTY in
exit
end
write memory
```

## دستگاه: SW-F2

```text
enable
configure terminal
ip domain-name netlab.ir
username netadmin privilege 15 secret NetLab32Pass
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
ip access-list standard MGMT-VTY
 permit 10.10.32.0 0.0.0.15
 deny any
exit
line vty 0 15
 login local
 transport input ssh telnet
 access-class MGMT-VTY in
exit
end
write memory
```

## دستگاه: SW-F3

```text
enable
configure terminal
ip domain-name netlab.ir
username netadmin privilege 15 secret NetLab32Pass
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
ip access-list standard MGMT-VTY
 permit 10.10.32.0 0.0.0.15
 deny any
exit
line vty 0 15
 login local
 transport input ssh telnet
 access-class MGMT-VTY in
exit
end
write memory
```

**Checkpoint `F-17-management-vty-acl.png`:** ACL مدیریت و خطوط VTY روی R-EDGE را ثبت کنید.

---

# ۱۴. بخش امتیازی محدودیت زمانی Finance

صورت سؤال ساعت دقیقی تعیین نکرده است. برای نمونه، دسترسی Finance در روزهای کاری از 08:00 تا 18:00 مجاز می‌شود. قبل از اجرا، ساعت Packet Tracer را با سناریوی تست هماهنگ کنید.

## دستگاه: R-EDGE

```text
enable
configure terminal
time-range FINANCE-HOURS
 periodic weekdays 08:00 to 18:00
exit
ip access-list extended PAT-USERS
 no permit tcp 10.10.30.0 0.0.0.15 any eq www
 no permit udp 10.10.30.0 0.0.0.15 any eq domain
 no permit tcp 10.10.30.0 0.0.0.15 any eq domain
 permit tcp 10.10.30.0 0.0.0.15 any eq www time-range FINANCE-HOURS
 permit udp 10.10.30.0 0.0.0.15 any eq domain time-range FINANCE-HOURS
 permit tcp 10.10.30.0 0.0.0.15 any eq domain time-range FINANCE-HOURS
exit
end
write memory
```

اگر نسخه Packet Tracer دستور `time-range` را پشتیبانی نمی‌کند، این بخش امتیازی را اجرا نکنید و پیکربندی اصلی PAT را نگه دارید.

---

# ۱۵. تست‌های اجباری به‌ترتیب ویدیو

## تست ۱ — VLAN، Trunk و SVI

### دستگاه: SW-CORE

```text
enable
show vlan brief
show interfaces trunk
show ip interface brief
```

مواردی که باید در ویدیو خوانا باشند:

- همه VLANها ایجاد شده‌اند.
- `Fa0/21`، `Fa0/22` و `Fa0/23` Trunk هستند.
- Native VLAN هر سه لینک `999` است.
- SVIهای کاربری و Server Farm آدرس صحیح دارند.

**Checkpoint `F-18-video-vlan-trunk-svi.png`**

## تست ۲ — DHCP روی دو VLAN متفاوت

### دستگاه: PC-S1

```text
ipconfig
```

انتظار در پروژه تازه:

```text
IP: 10.10.10.2/28
Gateway: 10.10.10.1
DNS: 10.10.100.13
```

### دستگاه: PC-Q1

```text
ipconfig
```

انتظار در پروژه تازه:

```text
IP: 10.10.22.2/28
Gateway: 10.10.22.1
DNS: 10.10.100.13
```

**Checkpoint `F-19-video-dhcp-two-vlans.png`:** دو خروجی را به‌صورت `-a` برای PC-S1 و `-b` برای PC-Q1 ثبت کنید؛ IP، Mask، Gateway و DNS باید کامل خوانا باشند.

## تست ۳ — Routing و OSPF

### دستگاه: SW-CORE

```text
enable
show ip route
show ip ospf neighbor
```

### دستگاه: PC-S1

```text
ping 10.10.22.2
ping 10.10.100.10
```

هر دو Ping باید موفق باشند. Ping دوم توسط استثنای مشخص ACL برای تست اجباری مجاز شده است.

**Checkpoint `F-20-video-ospf-and-pings.png`:** از `-a` برای Route/Neighbor روی SW-CORE و `-b` برای دو Ping موفق PC-S1 استفاده کنید.

## تست ۴ — DNS و صفحات داخلی

### دستگاه: PC-S1

```text
nslookup www.netlab.ir
nslookup backup.netlab.ir
```

نتایج مورد انتظار:

```text
www.netlab.ir -> 10.10.100.10
backup.netlab.ir -> 10.10.100.11
```

سپس در `Desktop > Web Browser` همین PC باز کنید:

```text
http://www.netlab.ir
http://backup.netlab.ir
```

صفحه WEB1 و WEB2 باید بدون استفاده از IP باز شوند.

**Checkpoint `F-21-video-internal-dns.png`**

**Checkpoint `F-22-video-internal-web-pages.png`**

## تست ۵ — PAT

### دستگاه: PC-S1

```text
ping 203.0.113.1
```

### دستگاه: R-EDGE

```text
enable
show ip nat translations
show ip nat statistics
```

رکورد PAT باید Inside Global برابر `203.0.113.10` داشته باشد. چون ترجمه ICMP سریع منقضی می‌شود، جدول NAT را بلافاصله نمایش دهید.

**Checkpoint `F-23-video-pat.png`**

## تست ۶ — Static NAT از Internet-User

در Web Browser دستگاه `Internet-User` ابتدا دو URL الزام‌شده در صورت سؤال را به‌ترتیب باز کنید:

```text
http://www.techcorp.ir
http://backup.techcorp.ir:8080
```

نتیجه مورد انتظار:

- URL اول صفحه `NetLab WEB1 Production` را باز کند.
- URL دوم با پورت ۸۰۸۰ صفحه `NetLab WEB2 Backup Portal` را باز کند.

برای نشان‌دادن سازگاری با نام دامنه اصلی شرکت، این دو نام نیز باید همان نتایج را بدهند:

```text
http://www.netlab.ir
http://backup.netlab.ir:8080
```

### دستگاه: R-EDGE

```text
enable
show ip nat translations
```

**Checkpoint `F-24-video-public-web1.png`:** صفحه WEB1 و نوار آدرس شامل `http://www.techcorp.ir` باید هم‌زمان خوانا باشند.

**Checkpoint `F-25-video-public-web2.png`:** صفحه WEB2 و نوار آدرس شامل `http://backup.techcorp.ir:8080` باید هم‌زمان خوانا باشند.

**Checkpoint `F-26-video-static-nat-table.png`**

## تست ۷ — QA به TEST باید مجاز باشد

روی Web Browser دستگاه `PC-Q1` باز کنید:

```text
http://test.netlab.ir
```

صفحه `NetLab Internal TEST Server` باید باز شود.

**Checkpoint `F-27-video-qa-test-allowed.png`**

## تست ۸ — کاربر غیر QA به TEST باید مسدود شود

روی Web Browser دستگاه `PC-S1` باز کنید:

```text
http://test.netlab.ir
```

DNS باید نام را به `10.10.100.12` تبدیل کند، اما صفحه نباید باز شود. این تفاوت نشان می‌دهد مشکل از DNS نیست و ACL دسترسی را مسدود کرده است.

**Checkpoint `F-28-video-nonqa-test-blocked.png`**

## تست ۹ — مسیر SSH کاربر عادی به WEB1 رد شود

در **Simulation Mode** یک **Add Complex PDU** بسازید:

```text
Source Device: PC-S1
Destination Device: WEB1
Protocol: TCP
Source Port: 1025
Destination Port: 22
One Shot
```

بسته باید روی SW-CORE توسط ACL حذف شود و شمارنده قاعده منع SSH افزایش یابد. سپس برای اثبات ردشدن نشست واقعی مدیریت تجهیزات اجرا کنید:

### دستگاه: PC-S1

```text
ssh -l netadmin 10.10.0.1
```

اتصال به R-EDGE باید به دلیل `MGMT-VTY` رد یا Timeout شود.

**Checkpoint `F-29-video-ordinary-ssh-denied.png`:** در صورت نیاز نام پایه را با پسوند `-a` برای Drop بسته به WEB1 و `-b` برای رد نشست R-EDGE تقسیم کنید.

## تست ۱۰ — مسیر SSH تیم Network به همان WEB1 مجاز باشد

در **Simulation Mode** همان Complex PDU را این بار با مبدأ PC-N1 بسازید:

```text
Source Device: PC-N1
Destination Device: WEB1
Protocol: TCP
Source Port: 1025
Destination Port: 22
One Shot
```

بسته باید از ACL عبور کند و به WEB1 برسد. اگر سرویس SSH روی WEB1 در نسخه شما موجود است، تست واقعی را نیز مستقیماً اجرا کنید:

### دستگاه: PC-N1

```text
ssh -l netadmin 10.10.100.10
```

اگر Server-PT سرویس SSH ندارد، برای اثبات نشست مدیریتی واقعی روی تجهیزات از همان PC اجرا کنید:

### دستگاه: PC-N1

```text
ssh -l netadmin 10.10.0.1
```

رمز آزمایش:

```text
NetLab32Pass
```

اتصال R-EDGE باید برقرار شود. پس از مشاهده Prompt دستگاه، با `exit` خارج شوید. در ویدیو صریحاً بگویید رسیدن PDU تیم Network به WEB1 مجازبودن ACL سرور را ثابت می‌کند، اما کامل‌شدن نشست کاربردی WEB1 به وجود سرویس SSH در مدل Server-PT وابسته است.

**Checkpoint `F-30-video-network-ssh-allowed.png`:** در صورت نیاز نام پایه را با پسوند `-a` برای رسیدن بسته به WEB1 و `-b` برای نشست موفق R-EDGE تقسیم کنید.

## تست ۱۱ — Internet-User به TEST باید مسدود شود

### دستگاه: Internet-User

```text
nslookup test.netlab.ir
ping 10.10.100.12
```

نتیجه مورد انتظار:

- `test.netlab.ir` در DNS خارجی رکورد ندارد.
- Ping مستقیم به IP خصوصی TEST نیز توسط ACL خارجی R-EDGE رد می‌شود.
- هیچ URL یا Port Forwarding عمومی برای TEST وجود ندارد.

**Checkpoint `F-31-video-internet-test-blocked.png`**

## تست ۱۲ — شمارنده‌های ACL

### دستگاه: SW-CORE

```text
enable
show access-lists ACL-SERVERS-IN
```

### دستگاه: R-EDGE

```text
enable
show access-lists INTERNET-IN
show access-lists MGMT-VTY
```

Permit و Denyهایی که در تست‌های قبلی استفاده شدند باید شمارنده داشته باشند.

**Checkpoint `F-32-video-acl-counters.png`:** خروجی SW-CORE را در `-a` و دو ACL دستگاه R-EDGE را در `-b` ثبت کنید؛ شماره Matchهای Permit و Deny باید خوانا باشند.

---

# ۱۶. عیب‌یابی کوتاه

## اگر SVI پایین است

- حداقل یک پورت Access یا Trunk حامل آن VLAN باید `up` باشد.
- VLAN باید روی SW-CORE و Access Switch مربوطه وجود داشته باشد.
- Native VLAN هر دو سمت Trunk باید `999` باشد.

## اگر DHCP کار نمی‌کند

### دستگاه: SW-CORE

```text
enable
show ip dhcp pool
show ip dhcp binding
show ip interface brief
```

بررسی کنید SVI VLAN موردنظر `up/up` و Pool دارای آدرس آزاد باشد.

## اگر OSPF همسایه ندارد

### دستگاه: SW-CORE

```text
enable
show ip ospf interface GigabitEthernet0/1
show ip ospf neighbor
```

### دستگاه: R-EDGE

```text
enable
show ip ospf interface GigabitEthernet0/0/1
show ip ospf neighbor
```

هر دو سمت باید در `10.10.0.0/30` و Area 0 باشند.

## اگر WEB2 از اینترنت باز نمی‌شود

- URL باید حتماً پورت `8080` داشته باشد.
- نگاشت داخلی WEB2 همچنان به پورت `80` است.
- ACL خارجی باید پورت مقصد `8080` روی `203.0.113.20` را مجاز کند.

## اگر NAT Table خالی است

ابتدا از یک PC داخلی `203.0.113.1` را Ping کنید و بلافاصله روی R-EDGE اجرا کنید:

```text
enable
show ip nat translations
```

---

# ۱۷. ترتیب پیشنهادی ویدیوی حداکثر ۱۰ دقیقه

| زمان تقریبی | نمایش |
|---|---|
| ۰:۰۰ تا ۰:۴۵ | نمای کامل توپولوژی و معماری Edge/Core/Access |
| ۰:۴۵ تا ۱:۴۵ | VLAN، Trunk و SVI روی SW-CORE |
| ۱:۴۵ تا ۲:۳۰ | DHCP روی دو کلاینت از دو VLAN |
| ۲:۳۰ تا ۳:۳۰ | OSPF، Route و دو Ping اجباری |
| ۳:۳۰ تا ۴:۳۰ | nslookup و صفحات داخلی با نام دامنه |
| ۴:۳۰ تا ۵:۳۰ | PAT و جدول ترجمه‌ها |
| ۵:۳۰ تا ۶:۳۰ | WEB1 و WEB2 از Internet-User |
| ۶:۳۰ تا ۸:۱۵ | QA مجاز، غیر QA مسدود، TEST خارجی مسدود |
| ۸:۱۵ تا ۹:۱۵ | SSH کاربر عادی و Network Team |
| ۹:۱۵ تا ۱۰:۰۰ | شمارنده‌های ACL و جمع‌بندی |

---

# ۱۸. چک‌لیست کامل نهایی

## توپولوژی و لایه ۲

- [ ] R-EDGE و ISP از مدل ISR 4321 هستند.
- [ ] SW-CORE از مدل 3560-24PS است.
- [ ] هر طبقه یک 2960 مستقل دارد.
- [ ] هجده PC داخلی، چهار سرور داخلی و یک Internet-User وجود دارند.
- [ ] همه VLANهای ۱۰، ۱۱، ۱۲، ۲۰، ۲۱، ۲۲، ۳۰، ۳۱، ۳۲، ۹۹، ۱۰۰ و ۹۹۹ ساخته شده‌اند.
- [ ] پورت‌های کاربران Access و در VLAN درست هستند.
- [ ] سه Trunk فعال‌اند.
- [ ] Native VLAN در هر دو سمت همه Trunkها ۹۹۹ است.

## لایه ۳ و سرویس‌ها

- [ ] همه SVIها Gateway دقیق جدول را دارند.
- [ ] لینک CORE به EDGE برابر `10.10.0.0/30` است.
- [ ] OSPF Process 1 و Area 0 روی هر دو دستگاه فعال است.
- [ ] همسایگی OSPF در وضعیت FULL است.
- [ ] Default Route از R-EDGE در OSPF منتشر شده است.
- [ ] برای هر VLAN کاربری یک DHCP Pool وجود دارد.
- [ ] Lease برابر ۲۴ ساعت و DNS برابر `10.10.100.13` است.
- [ ] اعضای Network Team و سرورها Static هستند.
- [ ] پنج رکورد داخلی DNS ساخته شده‌اند.
- [ ] TEST در DNS عمومی رکورد ندارد.

## NAT و امنیت

- [ ] PAT همه VLANهای کاربری را به `203.0.113.10` ترجمه می‌کند.
- [ ] `203.0.113.20:80` به `WEB1:80` می‌رود.
- [ ] `203.0.113.20:8080` به `WEB2:80` می‌رود.
- [ ] هیچ NAT برای TEST وجود ندارد.
- [ ] QA از طریق HTTP به TEST دسترسی دارد.
- [ ] غیر QA به TEST دسترسی ندارد.
- [ ] Finance در شبکه داخلی فقط HTTP و DNS مجاز به سمت Server Farm دارد، به TEST دسترسی ندارد و در اینترنت نیز فقط HTTP و DNS آن از PAT عبور می‌کند.
- [ ] فقط Network Team اجازه مدیریت SSH/Telnet تجهیزات را دارد.
- [ ] ACL خارجی فقط وب عمومی و پاسخ‌های لازم PAT را عبور می‌دهد.

## تست‌ها و تحویل

- [ ] `show vlan brief` ثبت شده است.
- [ ] `show interfaces trunk` ثبت شده است.
- [ ] `show ip interface brief` ثبت شده است.
- [ ] `ipconfig` دو VLAN ثبت شده است.
- [ ] `show ip route` و `show ip ospf neighbor` ثبت شده‌اند.
- [ ] Ping از VLAN 10 به VLAN 22 موفق است.
- [ ] Ping از VLAN 10 به WEB1 موفق است.
- [ ] هر دو nslookup موفق‌اند.
- [ ] هر دو صفحه داخلی با نام دامنه باز می‌شوند.
- [ ] رکورد PAT نمایش داده شده است.
- [ ] WEB1 و WEB2 از Internet-User باز می‌شوند.
- [ ] QA به TEST مجاز و غیر QA مسدود است.
- [ ] SSH کاربر عادی رد و SSH Network Team برقرار است.
- [ ] TEST از اینترنت قابل دسترسی نیست.
- [ ] شمارنده‌های ACL نمایش داده شده‌اند.
- [ ] فایل نهایی `.pkt` ذخیره شده است.
- [ ] ویدیو حداکثر ۱۰ دقیقه است و تست‌ها را به‌ترتیب نشان می‌دهد.

## فهرست ۳۲ Checkpoint

- [ ] `F-01-topology.png`
- [ ] `F-02-edge-isp-links.png`
- [ ] `F-03-core-vlans.png`
- [ ] `F-04-core-trunks.png`
- [ ] `F-05-core-svis.png`
- [ ] `F-06-dhcp-bindings.png`
- [ ] `F-07-two-client-ipconfig.png`
- [ ] `F-08-server-addresses.png`
- [ ] `F-09-internal-dns-records.png`
- [ ] `F-10-external-dns-records.png`
- [ ] `F-11-ospf-neighbor.png`
- [ ] `F-12-core-routing-table.png`
- [ ] `F-13-pat-translation.png`
- [ ] `F-14-static-port-forwarding.png`
- [ ] `F-15-server-acl-config.png`
- [ ] `F-16-internet-acl-config.png`
- [ ] `F-17-management-vty-acl.png`
- [ ] `F-18-video-vlan-trunk-svi.png`
- [ ] `F-19-video-dhcp-two-vlans.png`
- [ ] `F-20-video-ospf-and-pings.png`
- [ ] `F-21-video-internal-dns.png`
- [ ] `F-22-video-internal-web-pages.png`
- [ ] `F-23-video-pat.png`
- [ ] `F-24-video-public-web1.png`
- [ ] `F-25-video-public-web2.png`
- [ ] `F-26-video-static-nat-table.png`
- [ ] `F-27-video-qa-test-allowed.png`
- [ ] `F-28-video-nonqa-test-blocked.png`
- [ ] `F-29-video-ordinary-ssh-denied.png`
- [ ] `F-30-video-network-ssh-allowed.png`
- [ ] `F-31-video-internet-test-blocked.png`
- [ ] `F-32-video-acl-counters.png`
