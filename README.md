# SSTP-VPN-OpenWrt-21.02

## راهنمای نصب و رفع خطای پروتکل SSTP در OpenWrt
دستورات زیر را کپی کرده و در ترمینال SSH خود (مانند MobaXterm) وارد کنید
این مستندات شامل دستورات لازم برای نصب کلاینت SSTP، اضافه کردن رابط گرافیکی (LuCI) و رفع مشکل عدم شناسایی پروتکل (خطای `Unsupported protocol type`) در نسخه‌های جدید OpenWrt (مانند 21.02) است.

### مرحله ۱: به‌روزرسانی مخازن و نصب پکیج‌های اصلی

ابتدا باید هسته کلاینت SSTP و رابط گرافیکی آن را روی روتر نصب کنید:

```bash
opkg update
opkg install sstp-client luci-proto-sstp

```

### مرحله ۲: نصب پکیج سازگاری (luci-compat)

در نسخه‌های جدید OpenWrt (از جمله فِرم‌ورهای GL.iNet)، افزونه‌های گرافیکی برای شناسایی صحیح در سیستم به پکیج واسطه نیاز دارند. نصب این پکیج برای جلوگیری از خطای `Not present` الزامی است:

```bash
opkg install luci-compat

```

### مرحله ۳: ایجاد اینترفیس از طریق خط فرمان (UCI)

اگر پس از نصب پکیج‌ها، گزینه SSTP در منوی کشویی `Network > Interfaces` ظاهر نشد، می‌توانید ساختار اولیه کانکشن را مستقیماً از طریق دستورات UCI در شبکه ثبت کنید:

```bash
uci set network.SSTP_VPN=interface
uci set network.SSTP_VPN.proto='sstp'
uci commit network

```

### مرحله ۴: پاک کردن کش LuCI و راه‌اندازی مجدد سرویس‌ها

برای اینکه رابط کاربری تحت وب متوجه تغییرات شود و منوها را به‌روز کند، پاک کردن حافظه موقت (Cache) و راه‌اندازی مجدد وب‌سرور و شبکه ضروری است:

```bash
# پاک کردن کش رابط گرافیکی
rm -rf /tmp/luci-indexcache /tmp/luci-modulecache/*

# راه‌اندازی مجدد وب‌سرور و سرویس‌های مرتبط
/etc/init.d/uhttpd restart
/etc/init.d/rpcd restart
/etc/init.d/network reload

```

### مرحله ۵: اعمال نهایی (ریبوت)

در برخی معماری‌ها، سرویس مدیریت شبکه (`netifd`) پروتکل‌های جدید را تا زمان راه‌اندازی مجدد کامل سیستم بارگذاری نمی‌کند. پیشنهاد می‌شود در انتها یک بار روتر را ریبوت کنید:

```bash
reboot

```

> **نکته:** پس از بالا آمدن روتر، با فشردن کلیدهای `Ctrl + F5` کش مرورگر خود را نیز خالی کنید. اکنون می‌توانید با مراجعه به منوی Interfaces، مشخصات سرور، یوزرنیم و پسورد SSTP خود را وارد کنید.

### تنظیمات عمومی (General Settings)

یک سرور از سایت زیر انتخاب کنید
https://www.vpngate.net/en/ 
یا از  لیست تست ‌شده زیر قرار دهید:

```bash
یا
public-vpn-0.opengw.net:443
public-vpn-151.opengw.net:443
public-vpn-184.opengw.net:443
public-vpn-217.opengw.net:443

SSTP Port: مقدار 443 (یا پورت‌های ۹۹۵ و ۱۳۲۱ بسته به سرور انتخابی)

username: vpn
password: vpn
```
اکثر سرور ها SSTP با پورت 443 کار میکنند اگر خواستید گزینه پورت را به LUCI اضافه کنید فایل جیسون sstp.js به این آدرس در روتر منتقل کنید
/www/luci-static/resources/protocol/
_____________________________________________

### تنظیمات پیشرفته و جلوگیری از نشت Leak  DNS
 
✅ ویرایش اینترفیس   SSTP  و تنظیمات DNS آن روی حالت پیش‌فرض (تیک خورده با وزن 0) است.

برای اینکه DNS مودم شما همچنان بر DNS کانکشن SSTP غلبه می‌کند، تنظیمات اینترفیس SSTP نیست.
```bash
✅مهم تنظیمات DNS اینترفیس WAN (همان کابلی که از مودم به روتر آمده) است.
برای حل مشکل Leak  DNS باید در صفحه زیر این کار را انجام دهید:
1.	در منوی اصلی به مسیر Network > Interfaces بروید.
2.	این بار اینترفیس WAN   را پیدا کنید و دکمه Edit آن را بزنید.
3.	به تب Advanced Settings  بروید.
4.	مهم تیک گزینه Use DNS servers advertised by peer  را در آنجا بردارید.
5. یک دی‌ان‌اس دلخواه (مثل 8.8.8.8) در کادر زیرش بنویسید و (+) بزنید و ذخیره کنید.
با این کار، روتر دیگر DNS محلی مودم را جدی نمی‌گیرد و مجبور می‌شود فقط از DNS اختصاصی SSTP شما با اولویت صفر استفاده کند.
```
<img width="1778" height="1346" alt="Screenshot 2026-07-11 232616" src="https://github.com/user-attachments/assets/51288e54-7e74-4101-aa60-24d7e3192545" />

✅✅برای چک کردن نشت دی ان اس Leak  DNSاز سایت فوق العاده زیر استفاده کنید✅✅

https://browserleaks.com/ip

<img width="1197" height="1234" alt="Screenshot 2026-07-11 232202" src="https://github.com/user-attachments/assets/2eb557ec-ca75-43cb-a822-2e1112c875e4" />




راهنمایی ساخت اینترفیس
Network
Interfaces » Add new interface » SSTP_VPN
 
<img width="1786" height="2598" alt="SSTP VPN" src="https://github.com/user-attachments/assets/a1b548a5-f062-4d19-8141-50ff77e1daa1" />

