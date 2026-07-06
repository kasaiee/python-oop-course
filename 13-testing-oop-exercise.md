# تمرین‌های فصل سیزدهم: تست‌نویسی برای کد شیءگرا

> این تمرین‌ها کلِ مطالبِ فصل سیزدهم را پوشش می‌دهند. ابتدا خودتان پاسخ دهید، سپس با فایلِ پاسخ (`13-testing-oop-answer.md`) مقایسه کنید. تمرین‌های کدنویسی را واقعاً اجرا کنید — تستی که اجرا نشده، تست نیست.

---

## بخش الف: پرسش‌های مفهومی

**۱.** «فلجِ ریفکتور» چیست و تست‌ها چطور درمانش می‌کنند؟ استعاره‌ی تورِ ایمنی را توضیح دهید.

**۲.** الگوی AAA در ساختارِ یک تست یعنی چه؟ هر مرحله چه کاری می‌کند؟

**۳.** چرا استقلالِ تست‌ها «مقدس» است؟ حالتِ سراسری (مثل Singleton فصل قبل) چطور این استقلال را می‌شکند؟

**۴.** تفاوتِ Fake و Mock چیست؟ هرکدام چه زمانی انتخابِ بهتری است؟

**۵.** جمله‌ی «هر بار که مجبور به patch می‌شوید، طراحی دارد به شما طعنه می‌زند» یعنی چه؟ قاعده‌ی سرانگشتیِ فصل («اول تزریق، بعد Fake، و patch فقط برای...») را کامل کنید و توضیح دهید.

**۶.** «رفتار را تست کنید، نه پیاده‌سازی را» — این دو را با یک مثال مقایسه کنید و بگویید تستِ چسبیده به پیاده‌سازی چرا «شکننده» نامیده می‌شود.

---

## بخش ب: خواندن و تحلیل تست

**۷.** بدونِ اجرا بگویید هر یک از این تست‌ها پاس می‌شود یا شکست می‌خورد، و چرا:

```python
import pytest

class Wallet:
    def __init__(self, balance=0):
        self.balance = balance
    def spend(self, amount):
        if amount > self.balance:
            raise ValueError("insufficient")
        self.balance -= amount

def test_a():
    w = Wallet(100)
    w.spend(40)
    assert w.balance == 60

def test_b():
    w = Wallet(100)
    with pytest.raises(ValueError):
        w.spend(100)

def test_c():
    w = Wallet()
    with pytest.raises(ValueError):
        w.spend(1)
```

**۸.** این تست چه مشکلی دارد؟ (راهنمایی: به رابطه‌ی دو assert آخر با پیاده‌سازیِ داخلی فکر کنید.)

```python
from unittest.mock import Mock

def test_report_generation():
    db = Mock()
    db.fetch.return_value = [1, 2, 3]
    report = ReportService(db)

    result = report.generate()

    assert result == "total: 6"
    db.fetch.assert_called_once()
    assert db.fetch.call_args is not None
    db.connect.assert_not_called()
```

**۹.** دو تستِ زیر به هم وابسته‌اند و با تغییرِ ترتیبِ اجرا می‌شکنند. مشکل را پیدا کنید و راهِ اصلاح را بگویید:

```python
inventory = {"keyboard": 5}

def test_reserve_decreases_stock():
    inventory["keyboard"] -= 2
    assert inventory["keyboard"] == 3

def test_stock_is_positive():
    assert inventory["keyboard"] == 5
```

---

## بخش ج: کدنویسی

**۱۰.** برای کلاسِ زیر چهار تست بنویسید: افزودنِ آیتم، جمعِ قیمت، خطای آیتمِ تکراری، و سبدِ خالی:

```python
class DuplicateItemError(Exception):
    pass

class Cart:
    def __init__(self):
        self._items = {}          # name -> price

    def add(self, name, price):
        if name in self._items:
            raise DuplicateItemError(name)
        self._items[name] = price

    def total(self):
        return sum(self._items.values())
```

**۱۱.** تابعِ `grade(score)` نمره‌ی ۰ تا ۱۰۰ می‌گیرد و برمی‌گرداند: `"A"` برای ۹۰ به بالا، `"B"` برای ۷۵ به بالا، `"C"` برای ۵۰ به بالا، و `"F"` برای بقیه. با `@pytest.mark.parametrize` تستی بنویسید که دست‌کم شش سناریو — شاملِ **مقادیرِ مرزی** — را بسنجد. (خودِ تابع را هم بنویسید.)

**۱۲.** یک fixture به نامِ `loaded_cart` بنویسید که سبدی با دو آیتم (جمعاً ۸۰٬۰۰۰) برگرداند، و دو تست که از آن استفاده کنند.

**۱۳.** کلاسِ `SmsSender` را در نظر بگیرید که واقعاً پیامک می‌فرستد (و در تست نباید بفرستد):

```python
class CampaignService:
    def __init__(self, sender):
        self.sender = sender

    def notify_all(self, phones, text):
        sent = 0
        for phone in phones:
            try:
                self.sender.send(phone, text)
                sent += 1
            except ConnectionError:
                continue          # skip failed numbers
        return sent
```

- الف) با یک **Fake دست‌ساز** تست کنید که `notify_all` به همه‌ی شماره‌ها پیام می‌دهد و تعدادِ درست را برمی‌گرداند.
- ب) با **Mock** و `side_effect` تست کنید که اگر ارسال به یکی از سه شماره با `ConnectionError` شکست بخورد، خروجی ۲ است.

---

## بخش د: طراحی برای تست‌پذیری

**۱۴.** کلاسِ زیر تست‌نویسی را زجرآور کرده. سه مشکلِ تست‌پذیری‌اش را نام ببرید و نسخه‌ی اصلاح‌شده‌اش را بنویسید (فقط اسکلت — بدنه‌ی متدها مهم نیست):

```python
import smtplib
from datetime import datetime

class InvoiceService:
    def send_overdue_notices(self):
        db = MySQLConnection("prod-server", "root", "secret123")
        invoices = db.query("SELECT * FROM invoices")
        for inv in invoices:
            if (datetime.now() - inv.due_date).days > 30:
                smtp = smtplib.SMTP("mail.company.com")
                smtp.send(inv.email, "your invoice is overdue!")
```

---

## بخش ه: پرسشِ تحلیلی (تفکر انتقادی)

**۱۵.** مدیرِ فنیِ تیمی اعلام کرده: «از این به بعد پوششِ تستِ همه‌ی پروژه‌ها باید ۱۰۰٪ باشد؛ هر خطی که تست ندارد، merge نمی‌شود.» این سیاست را نقد کنید: چه رفتارهای ناسالمی را تشویق می‌کند؟ چه جایگزینی پیشنهاد می‌دهید؟ (به تفاوتِ «پوشش» و «اطمینان»، هزینه‌ی تستِ کم‌ارزش، و تست‌های happy-path-only اشاره کنید.)
