# تمرین‌های فصل هشتم: اصول SOLID

> این تمرین‌ها هر پنج اصلِ SOLID را با مثال‌های نرم‌افزاریِ واقعی می‌سنجند. ابتدا خودتان پاسخ دهید، سپس با `08-solid-answer.md` مقایسه کنید.

---

## بخش الف: Single Responsibility و Open/Closed

**۱.** اصلِ Single Responsibility (SRP) چیست؟ در کلاسِ زیر چرا وجودِ متدِ `send_email` نقضِ SRP است؟ دو «دلیلِ مستقلِ تغییر» را که این کلاس دارد، نام ببرید.

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
    def save(self):
        print("saved to database")
    def send_email(self, message):
        print(f"Email: {message}")
```

**۲.** اصلِ Open/Closed (OCP) چیست؟ کدِ زیر چرا OCP را نقض می‌کند و چرا این «خطرناک» است؟

```python
class PaymentProcessor:
    def process(self, method, amount):
        if method == "card":
            print(f"Card: {amount}")
        elif method == "wallet":
            print(f"Wallet: {amount}")
```

**۳. (کدنویسی)** کدِ سوالِ ۲ را طوری بازنویسی کنید که با انتزاع و چندریختی از OCP پیروی کند. سپس نشان دهید افزودنِ یک روشِ پرداختِ جدید (`CryptoPayment`) بدونِ ویرایشِ `PaymentProcessor` ممکن است.

---

## بخش ب: Liskov Substitution و Interface Segregation

**۴.** اصلِ Liskov Substitution (LSP) چیست؟ در کدِ زیر، چرا `ReadOnlyStorage` این اصل را نقض می‌کند؟

```python
class FileStorage:
    def save(self, data):
        print(f"Saved: {data}")

class ReadOnlyStorage(FileStorage):
    def save(self, data):
        raise NotImplementedError("is read-only!")
```

**۵.** چه نشانه‌هایی به ما می‌گویند یک زیرکلاس احتمالاً LSP را نقض کرده است؟ دست‌کم دو نشانه بگویید.

**۶.** اصلِ Interface Segregation (ISP) چیست؟ در کدِ زیر، `RobotWorker` چطور **هم** ISP و **هم** LSP را نقض می‌کند؟

```python
class Worker:
    def work(self): ...
    def eat(self): ...
    def sleep(self): ...

class RobotWorker(Worker):
    def work(self): print("works")
    def eat(self): raise NotImplementedError
    def sleep(self): raise NotImplementedError
```

**۷. (کدنویسی)** اینترفیسِ چاقِ `Worker` در سوالِ ۶ را به چند اینترفیسِ کوچک و متمرکز بشکنید، سپس `HumanWorker` و `RobotWorker` را طوری بنویسید که هرکدام فقط به آنچه واقعاً نیاز دارد وابسته باشد.

---

## بخش ج: Dependency Inversion

**۸.** اصلِ Dependency Inversion (DIP) چیست؟ در کدِ زیر، وابستگیِ `UserService` به `MySQLDatabase` چه مشکلی ایجاد می‌کند؟

```python
class UserService:
    def __init__(self):
        self.db = MySQLDatabase()
    def register(self, user):
        self.db.save(user)
```

**۹. (کدنویسی)** کدِ سوالِ ۸ را با رعایتِ DIP بازنویسی کنید: یک انتزاعِ `Database` تعریف کنید، دو پیاده‌سازی (`MySQLDatabase` و `InMemoryDatabase`) بسازید، و `UserService` را طوری بنویسید که پیاده‌سازی از **بیرون تزریق** شود.

**۱۰.** «Dependency Injection» چیست و چه ارتباطی با اصلِ DIP و تست‌پذیریِ کد دارد؟

---

## بخش د: تشخیص و اصولِ مکمل

**۱۱. (تشخیص)** برای هر توضیحِ زیر بگویید کدام اصلِ SOLID نقض شده است:
- الف) کلاسی که هم منطقِ سبد خرید را دارد و هم فایلِ لاگ را می‌نویسد و هم گزارشِ HTML می‌سازد.
- ب) برای افزودنِ یک نوعِ گزارشِ جدید، باید داخلِ یک متدِ بزرگ یک `elif` تازه اضافه کنید.
- ج) کلاسی که مستقیماً یک شیءِ `SMTPClient` مشخص را داخلِ خودش `new` می‌کند و به آن میخکوب است.
- د) زیرکلاسی که یکی از متدهای والد را با `pass` خالی می‌گذارد چون نمی‌تواند آن را پیاده کند.

**۱۲.** پنج اصلِ مکملِ SOLID که در فصل ذکر شد (Composition over Inheritance، Law of Demeter، DRY، YAGNI، KISS) را نام ببرید و هرکدام را در یک جمله توضیح دهید.

---

## بخش ه: پرسشِ تحلیلی

**۱۳.** این جمله را نقد کنید: «هرچه اصولِ SOLID را بیشتر و کامل‌تر رعایت کنیم بهتر است؛ پس بیایید همه‌چیز را انتزاعی، تزریقی و به کوچک‌ترین کلاس‌های ممکن تقسیم کنیم.»

**۱۴.** «SRP افراطی» چه خطری دارد؟ همچنین توضیح دهید چطور «OCP افراطی» می‌تواند با اصلِ YAGNI در تنش قرار بگیرد.

**۱۵.** نویسنده SOLID را «یک قطب‌نما، نه یک قفس» می‌نامد. منظور از این تشبیه چیست و چه چیزی را درباره‌ی زمانِ درستِ به‌کارگیریِ این اصول به ما می‌گوید؟
