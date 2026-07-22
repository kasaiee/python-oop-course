# آزمون خودسنجی فصل صفر: آماده‌ی شروع هستید؟

> این آزمون با تمرین‌های فصل‌های دیگر فرق دارد: هدفش **سنجش آمادگی شما برای شروع دوره** است، نه یادگیری مطلب جدید. بدون کتاب و بدون اجرای کد، روی کاغذ جواب دهید؛ بعد با فایل پاسخ (`00-getting-ready-answer.md`) مقایسه کنید. راهنمای نمره‌گذاری در انتهای فایل پاسخ است — به خودتان دروغ نگویید، این آزمون فقط به خودتان گزارش می‌دهد.

---

## تمرین ۱ - مفاهیم پایه

**۱.** تفاوت `list` و `tuple` چیست؟ برای نگه‌داشتن «طول و عرض جغرافیایی یک فروشگاه» کدام مناسب‌تر است و چرا؟

**۲.** برای هر نیاز، ظرف مناسب (`list`، `dict`، `tuple`، `set`) را انتخاب کنید:

- الف) شناسه‌ی کاربرانی که امروز وارد سایت شده‌اند، بدون تکرار.
- ب) صف سفارش‌ها به ترتیب ثبت.
- ج) مشخصات یک محصول: نام، قیمت، موجودی.
- د) روز و ماه و سال تولد به‌صورت یک مقدار ثابت.

**۳.** تفاوت `print` و `return` در یک تابع چیست؟ اگر تابعی `return` نداشته باشد، خروجی‌اش چیست؟

**۴.** خط `x = y` در پایتون یک «کپی از مقدار» می‌سازد یا «برچسبی جدید روی همان مقدار»؟ (اگر مطمئن نیستید، حدس بزنید — در فصل دوم این موضوع حیاتی می‌شود.)

---

## تمرین ۲ - خواندن کد

بدون اجرا، خروجی هر قطعه را پیش‌بینی کنید.

**۵.**

```python
cart = ["keyboard", "mouse"]
cart.append("monitor")
print(len(cart), cart[1])
```

**۶.**

```python
product = {"name": "keyboard", "stock": 5}
product["stock"] -= 2
print(product.get("price", 0))
print(product["stock"])
```

**۷.**

```python
def apply_discount(amount, percent=10):
    return amount - amount * percent / 100

print(apply_discount(200_000))
print(apply_discount(200_000, 50))
```

**۸.**

```python
total = 0
for n in [100, 250, 400]:
    if n > 300:
        continue
    total += n
print(total)
```

**۹.**

```python
def greet(name):
    print(f"hello {name}")

result = greet("sara")
print(result)
```

**۱۰.**

```python
def shout(text):
    return text.upper()

f = shout
print(f("ok"))
```

---

## تمرین ۳ - نوشتن کد

**۱۱.** تابعی به نام `is_valid_password` بنویسید که یک رشته بگیرد و فقط وقتی `True` برگرداند که طولش دست‌کم ۸ کاراکتر باشد.

**۱۲.** تابعی به نام `count_paid` بنویسید که لیستی از سفارش‌ها (هر سفارش یک دیکشنری با کلید `paid`) بگیرد و تعداد سفارش‌های پرداخت‌شده را برگرداند.

**۱۳.** تابعی به نام `describe_user` بنویسید که پارامتر اجباری `username` و پارامتر اختیاری `role` (پیش‌فرض: `"member"`) بگیرد و رشته‌ای مثل `"ali (admin)"` برگرداند.

---

## تمرین ۴ - خواندن Traceback

**۱۴.** برنامه‌ای این خطا را داده است:

```
Traceback (most recent call last):
  File "app.py", line 12, in <module>
    show_report(sales)
  File "app.py", line 7, in show_report
    average = total / count
              ~~~~~~^~~~~~~
ZeroDivisionError: division by zero
```

- الف) نوع خطا چیست و یعنی چه؟
- ب) خطا دقیقاً در کدام فایل، کدام خط و درون کدام تابع رخ داده؟
- ج) آن خط از کجا صدا زده شده بود؟

**۱۵.** کد زیر هنگام اجرا خطا می‌دهد. بدون اجرا بگویید نوع خطا چیست و چطور رفعش می‌کنید:

```python
user = {"username": "ali"}
print("email: " + user["email"])
```

---

پاسخ‌ها و راهنمای نمره‌گذاری: [`00-getting-ready-answer.md`](00-getting-ready-answer.md)
