# فصل چهارم: اصول چهارگانه شیءگرایی — ارث‌بری، انتزاع، چندریختی و کپسول‌سازی

## مقدمه

در فصل‌های قبل بلوک‌های پایه را ساختیم: کلاس، شیء، `self` و سازنده. یاد گرفتیم چطور مثل یک طراح شیءگرا فکر کنیم و کلاس‌های درست را از دل مسئله بیرون بکشیم. حالا وقت آن رسیده که این بلوک‌ها را به یک ساختمان تبدیل کنیم.

چهار اصل بنیادین شیءگرایی — **ارث‌بری (Inheritance)، انتزاع (Abstraction)، چندریختی (Polymorphism) و کپسول‌سازی (Encapsulation)** — ستون‌های هر زبان شیءگرا هستند. اسم‌هایشان پرطمطراق است، اما پشت هرکدام یک ایده‌ی ساده و کاربردی نشسته است. درک درست این چهار اصل، مرز میان کسی است که «کلاس بلد است» و کسی که «شیءگرا طراحی می‌کند» — و دقیقاً همین چهار مفهوم است که در مصاحبه‌های استخدامی، بیش از هر موضوع دیگری از شیءگرایی پرسیده می‌شود.

این فصل مفصل‌ترین فصل نیمه‌ی اول دوره است؛ هر یک از این چهار اصل به‌راحتی می‌توانست یک فصل مستقل باشد. پس با حوصله جلو بروید، کدها را خودتان اجرا کنید و هر بخش را که تمام کردید، چند لحظه مکث کنید و از خودتان بپرسید: «این اصل چه مشکلی را حل کرد؟»

## چرا باید چهار اصل شیءگرایی را یاد بگیریم؟

فرض کنید کدی نوشته‌اید که کار می‌کند. آیا همین کافی است؟ اجازه دهید چند سوال سخت‌تر بپرسیم: آیا این کد در برابر تغییر مقاوم است؟ وقتی یک نوع پرداخت جدید به سیستم اضافه می‌شود، باید ده جای مختلف را دستکاری کنید یا یک جا؟ کسی که فردا کد شما را می‌خواند، می‌فهمد چه خبر است؟ و اگر همکارتان به اشتباه داده‌ی درونی یک شیء را خراب کند، چه چیزی جلویش را می‌گیرد؟

پاسخ هر چهار سوال، به یکی از این چهار اصل برمی‌گردد:

- **ارث‌بری** اجازه می‌دهد رفتار مشترک را یک‌بار بنویسید و در کلاس‌های متعدد بازاستفاده کنید، به‌جای آنکه همان کد را کپی کنید و بعدها در ده نسخه‌ی ناهماهنگ نگهداری‌اش کنید.
- **انتزاع** پیچیدگی را پشت یک رابط ساده پنهان می‌کند تا استفاده‌کننده‌ی کلاس فقط با چیزهایی سروکار داشته باشد که واقعاً به آن‌ها نیاز دارد.
- **چندریختی** کدی می‌سازد که با انواع مختلف، بدون زنجیره‌های بی‌پایان `if/elif`، یکسان کار می‌کند — و افزودن نوع جدید، به تغییر کد قدیمی نیاز ندارد.
- **کپسول‌سازی** جلوی دستکاری ناخواسته‌ی داده را می‌گیرد و تضمین می‌کند شیء هرگز به حالت نامعتبر نرود.

بدون این‌ها، کد شما شاید امروز اجرا شود، اما با هر تغییر کوچک شکننده‌تر می‌شود؛ تا روزی که همه از دست‌زدن به آن بترسند.

### ترتیب یادگیری در این فصل

این چهار اصل را به ترتیبی می‌خوانیم که به‌طور طبیعی روی هم سوار می‌شوند: اول **ارث‌بری**، چون زیربنای ساختن سلسله‌مراتب کلاس‌هاست. بعد **انتزاع**، که به ما یاد می‌دهد این سلسله‌مراتب‌ها را حول رابط‌های ساده طراحی کنیم. سپس **چندریختی**، که میوه‌ی آن دو تای قبلی است: وقتی سلسله‌مراتبی با رابط مشترک دارید، می‌توانید با همه‌ی اعضایش یکسان رفتار کنید. و در پایان **کپسول‌سازی**، که نگهبان همه‌ی این‌هاست و داده‌ی درونی هر کلاس را از دستکاری بی‌ضابطه محافظت می‌کند.

---

## ستون اول — ارث‌بری (Inheritance)

### مشکلی که ارث‌بری حل می‌کند: کد تکراری

بهترین راه فهمیدن یک ابزار، دیدن دنیای بدون آن است. فرض کنید در سیستم منابع انسانی یک شرکت، دو نوع نیرو داریم: کارمند عادی و مدیر. بدون ارث‌بری، مجبوریم دو کلاس جدا بنویسیم:

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def pay(self):
        return f"{self.name}: {self.salary} Toman"

class Manager:
    def __init__(self, name, salary, team_size):
        self.name = name            # duplicated
        self.salary = salary        # duplicated
        self.team_size = team_size

    def pay(self):                  # duplicated, line by line
        return f"{self.name}: {self.salary} Toman"

    def bonus(self):
        return self.salary * 0.1 * self.team_size
```

به `Manager` نگاه کنید: دو خط از سازنده و کل متد `pay` عیناً از `Employee` کپی شده‌اند. این فقط زشت نیست؛ خطرناک است. فردا که قانون پرداخت عوض شود — مثلاً قرار شود مالیات هم در `pay` محاسبه شود — باید یادتان بماند که **هر دو** کلاس را تغییر دهید. با ده نوع نیروی مختلف، این یعنی ده جای فراموش‌شدنی. کد تکراری، بدهی‌ای است که با بهره پس می‌دهید.

### راه‌حل: ارث‌بری

**ارث‌بری** به یک کلاس اجازه می‌دهد داده و رفتار کلاس دیگری را به ارث ببرد و فقط چیزهای خاص خودش را اضافه یا عوض کند. کافی است نام کلاس والد را در پرانتز جلوی نام کلاس فرزند بنویسیم:

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def pay(self):
        return f"{self.name}: {self.salary} Toman"

class Manager(Employee):              # Manager inherits from Employee
    def __init__(self, name, salary, team_size):
        super().__init__(name, salary)    # delegate shared setup to the parent
        self.team_size = team_size

    def bonus(self):
        return self.salary * 0.1 * self.team_size

m = Manager("Ali", 5_000_000, 4)
print(m.pay())      # Ali: 5000000 Toman — inherited from Employee
print(m.bonus())    # 2000000.0 — specific to Manager
```

حالا `pay` فقط یک‌جا تعریف شده و `Manager` آن را رایگان به ارث برده است. اگر منطق پرداخت تغییر کند، یک جا را عوض می‌کنید و همه‌ی زیرکلاس‌ها خودبه‌خود به‌روز می‌شوند.

چند اصطلاح که از این‌جا به بعد زیاد می‌شنوید: به کلاسی که ارث می‌دهد **والد** (Parent، یا کلاس پایه/Base) و به کلاسی که ارث می‌برد **فرزند** (Child، یا زیرکلاس/Subclass) می‌گوییم. در پایتون هر کلاسی که والدی برایش ننویسید، به‌طور خودکار از کلاس `object` ارث می‌برد — به همین دلیل است که همه‌ی اشیاء پایتون از ابتدا رفتارهای مشترکی مثل قابل چاپ بودن دارند.

### رابطه‌ی Is-A: قلب ارث‌بری

ارث‌بری فقط یک ترفند برای صرفه‌جویی در تایپ نیست؛ یک **ادعای معنایی** است. وقتی می‌نویسید `class Manager(Employee)`، دارید به همه اعلام می‌کنید: «مدیر **یک نوع** کارمند است.» به این رابطه، **Is-A** می‌گوییم.

پیش از هر ارث‌بری، این جمله را بلند بگویید: «X یک نوع Y است.» اگر جمله طبیعی بود، ارث‌بری قابل دفاع است؛ اگر دلتان نیامد جمله را بگویید، احتمالاً رابطه از جنس دیگری است:

- «مدیر یک نوع کارمند است.» — طبیعی است؛ ارث‌بری درست است.
- «خودرو یک نوع موتور است.» — نه! خودرو موتور **دارد**. این رابطه‌ی **Has-A** است و ابزارش ترکیب (Composition) است، نه ارث‌بری. به این نکته در پایان همین بخش برمی‌گردیم.

### فرزند با ارثیه‌اش چه می‌تواند بکند؟

یک زیرکلاس سه اختیار دارد و معمولاً ترکیبی از هر سه را به‌کار می‌گیرد: می‌تواند ارثیه را **همان‌طور که هست استفاده کند**، می‌تواند **چیز جدیدی به آن اضافه کند**، و می‌تواند **بخشی از آن را بازنویسی کند**. هر سه را در یک مثال ببینیم:

```python
class Vehicle:
    def __init__(self, brand):
        self.brand = brand

    def start(self):
        return f"{self.brand} engine started"

    def describe(self):
        return f"a vehicle made by {self.brand}"

class ElectricCar(Vehicle):
    def charge(self):                     # 1) add new behavior
        return f"{self.brand} is charging"

    def start(self):                      # 2) override inherited behavior
        return f"{self.brand} started silently"

tesla = ElectricCar("Tesla")
print(tesla.describe())   # a vehicle made by Tesla   ← 3) used as-is
print(tesla.start())      # Tesla started silently    ← overridden
print(tesla.charge())     # Tesla is charging         ← newly added
```

دقت کنید که وقتی `tesla.describe()` را صدا می‌زنیم، پایتون اول در خود `ElectricCar` دنبال `describe` می‌گردد؛ پیدا نمی‌کند، پس سراغ والد یعنی `Vehicle` می‌رود و آن‌جا پیدایش می‌کند. این جست‌وجوی «از فرزند به والد» مکانیزم پایه‌ای ارث‌بری در پایتون است.

### super()‏: پل ارتباط با والد

در مثال `Manager` دیدید که داخل سازنده‌ی فرزند نوشتیم `super().__init__(name, salary)`. تابع `super()` به شما دسترسی به پیاده‌سازی کلاس والد را می‌دهد؛ یعنی می‌گوید «همین متد را از والد اجرا کن».

رایج‌ترین کاربردش دقیقاً همین‌جاست: زیرکلاس اول سازنده‌ی والد را صدا می‌زند تا داده‌های مشترک مقداردهی شوند، بعد داده‌های مخصوص خودش را اضافه می‌کند. اگر این تماس را فراموش کنید، attributeهای والد اصلاً ساخته نمی‌شوند:

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

class BrokenManager(Employee):
    def __init__(self, name, salary, team_size):
        # forgot super().__init__(name, salary)!
        self.team_size = team_size

b = BrokenManager("Sara", 8_000_000, 3)
print(b.team_size)    # 3
# print(b.name)       # AttributeError: 'BrokenManager' object has no attribute 'name'
```

شاید بپرسید: چرا `super().__init__(...)` و نه مستقیماً `Employee.__init__(self, ...)`؟ نوشتن مستقیم نام والد هم کار می‌کند، اما دو عیب دارد: اول اینکه نام والد را در بدنه‌ی فرزند حک می‌کند و اگر روزی والد عوض شود، باید همه‌ی این تماس‌ها را هم پیدا و اصلاح کنید. دوم — و مهم‌تر — اینکه در وراثت چندگانه، `super()` مسیر درست را بر اساس MRO دنبال می‌کند و از اجرای دوباره‌ی یک والد جلوگیری می‌کند؛ چیزی که با صدازدن مستقیم به‌راحتی خراب می‌شود. فعلاً همین‌قدر کافی است؛ مکانیزم دقیق MRO را در فصل ششم کامل باز می‌کنیم.

`super()` فقط مخصوص `__init__` نیست و در ارث‌بری چندسطحی هم زنجیره را به‌درستی دنبال می‌کند:

```python
class A:
    def setup(self):
        print("A")

class B(A):
    def setup(self):
        super().setup()      # goes to A
        print("B")

class C(B):
    def setup(self):
        super().setup()      # goes to B (which itself goes to A)
        print("C")

C().setup()
# Output:
# A
# B
# C
```

به ترتیب خروجی دقت کنید: هر لایه اول کار والدش را انجام می‌دهد، بعد کار خودش را. این الگوی «اول والد، بعد خودم» در عمل بسیار رایج است — مثلاً در مقداردهی اولیه، والد باید زودتر از فرزند آماده شود.

### Override: بازنویسی رفتار والد

**Override** یعنی زیرکلاس، متدی از والد را با پیاده‌سازی خودش جایگزین کند: متدی با **همان نام** در فرزند تعریف می‌کنید و از آن به بعد، روی اشیاء فرزند نسخه‌ی جدید اجرا می‌شود. بسته به اینکه رفتار والد هنوز به‌درد می‌خورد یا نه، دو سبک داریم.

**۱. Override کامل** — رفتار والد را به‌کل کنار می‌گذاریم:

```python
class Employee:
    def role(self):
        return "regular employee"

class Contractor(Employee):
    def role(self):                  # full override
        return "contractor"

print(Contractor().role())   # contractor — parent's version is ignored
```

**۲. Override توسعه‌ای (با super)** — رفتار والد را نگه می‌داریم و چیزی به آن می‌افزاییم:

```python
class Employee:
    def __init__(self, name):
        self.name = name

    def describe(self):
        return f"Name: {self.name}"

class Manager(Employee):
    def describe(self):              # extending override
        base = super().describe()    # take the parent's behavior
        return f"{base} | role: manager"   # and extend it

print(Manager("Ali").describe())   # Name: Ali | role: manager
```

قاعده‌ی انتخاب ساده است: اگر منطق والد هنوز معتبر است و فقط می‌خواهید چیزی به آن اضافه کنید، سبک توسعه‌ای با `super()` را انتخاب کنید تا منطق والد را کپی نکرده باشید. اگر رفتار فرزند از اساس متفاوت است، Override کامل صادقانه‌تر است.

یک هشدار: متد بازنویسی‌شده باید همان **قراردادِ** متد والد را رعایت کند — همان نوع ورودی را بپذیرد و همان جنس خروجی را بدهد. اگر `pay` والد عدد برمی‌گرداند و نسخه‌ی فرزند رشته برگرداند، هر کدی که با «کارمندها» کار می‌کند و به عدد بودن خروجی تکیه دارد، روی مدیرها می‌شکند. این اصل («زیرکلاس باید جانشین بی‌دردسر والد باشد») آن‌قدر مهم است که نام رسمی دارد — اصل جانشینی لیسکوف — و در فصل هشتم مفصل سراغش می‌رویم.

### انواع ارث‌بری

تا این‌جا فقط ساده‌ترین شکل ارث‌بری را دیده‌ایم: یک والد و یک فرزند. اما سلسله‌مراتب‌ها شکل‌های دیگری هم دارند. چهار شکل رایج را بشناسیم:

**۱. ارث‌بری ساده (Single):** یک فرزند از یک والد ارث می‌برد. همان `Manager(Employee)` که دیدیم.

```
Employee
   ▲
   │
Manager
```

**۲. ارث‌بری چندسطحی (Multilevel):** زنجیره‌ای از نسل‌ها؛ فرزندِ فرزند. هر سطح، ارثیه‌ی همه‌ی سطح‌های بالاتر را دارد:

```
Animal ──► Bird ──► Penguin        (پنگوئن هم پرنده است، هم حیوان)
```

```python
class Animal:
    def breathe(self):
        return "breathing"

class Bird(Animal):
    def lay_eggs(self):
        return "laying eggs"

class Penguin(Bird):
    def swim(self):
        return "swimming"

p = Penguin()
print(p.breathe())    # breathing  — from Animal (two levels up)
print(p.lay_eggs())   # laying eggs — from Bird
print(p.swim())       # swimming    — its own
```

زنجیره‌های چندسطحی کار می‌کنند، اما هرچه عمیق‌تر شوند فهمیدن اینکه «این متد از کجا آمده» سخت‌تر می‌شود. تجربه‌ی رایج صنعت: بیش از دو، سه سطح، علامت هشدار است.

**۳. ارث‌بری سلسله‌مراتبی (Hierarchical):** چند فرزند از یک والد مشترک ارث می‌برند. این پرکاربردترین شکل در پروژه‌های واقعی است:

```
              Employee
                 ▲
     ┌───────────┼───────────┐
  Manager    Developer    Designer
```

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def monthly_pay(self):
        return self.salary

class Manager(Employee):
    def __init__(self, name, salary, team_size):
        super().__init__(name, salary)
        self.team_size = team_size

    def monthly_pay(self):                        # extending override
        return super().monthly_pay() + 500_000 * self.team_size

class Developer(Employee):
    def __init__(self, name, salary, overtime_hours):
        super().__init__(name, salary)
        self.overtime_hours = overtime_hours

    def monthly_pay(self):                        # extending override
        return super().monthly_pay() + 200_000 * self.overtime_hours

class Designer(Employee):
    pass                                          # inherits everything as-is

staff = [
    Manager("Ali", 9_000_000, team_size=4),
    Developer("Sara", 8_000_000, overtime_hours=10),
    Designer("Reza", 7_000_000),
]

for person in staff:
    print(f"{person.name}: {person.monthly_pay():,} Toman")
# Ali: 11,000,000 Toman
# Sara: 10,000,000 Toman
# Reza: 7,000,000 Toman
```

به حلقه‌ی آخر خوب نگاه کنید: با اینکه سه نوع نیروی متفاوت در لیست است، حلقه فقط یک‌بار نوشته شده و هیچ `if`ی ندارد؛ هر شیء، نسخه‌ی خودش از `monthly_pay` را اجرا می‌کند. این دقیقاً همان **چندریختی** است که در ستون سوم این فصل مفصل بازش می‌کنیم — این حلقه را به خاطر بسپارید.

**۴. وراثت چندگانه (Multiple):** یک فرزند از چند والد هم‌زمان ارث می‌برد. پایتون برخلاف جاوا این را مجاز می‌داند:

```python
class Camera:
    def take_photo(self):
        return "photo taken"

class Phone:
    def make_call(self):
        return "calling..."

class Smartphone(Camera, Phone):      # inherits from both
    pass

s = Smartphone()
print(s.take_photo())   # photo taken
print(s.make_call())    # calling...
```

وراثت چندگانه قدرتمند اما پرریسک است: اگر دو والد متدی هم‌نام داشته باشند، کدام برنده می‌شود؟ پاسخ به ترتیب مشخصی به نام MRO بستگی دارد که همراه با «مسئله‌ی الماس» و Mixinها، موضوع اصلی **فصل ششم** است. فعلاً همین‌قدر بدانید که چنین امکانی وجود دارد و باید سنجیده استفاده شود.

### کاربرد واقعی: استثناهای سفارشی

ارث‌بری فقط برای مدل‌کردن آدم‌ها و حیوان‌ها نیست؛ یکی از کاربردی‌ترین جاهایش در هر پروژه‌ی واقعی، ساختن **استثناهای سفارشی** است. با ارث‌بری از `Exception`، خطاهایی معنادار و مخصوص دامنه‌ی پروژه‌تان می‌سازید:

```python
class PaymentError(Exception):
    """base error for anything payment-related"""

class InsufficientFundsError(PaymentError):
    """balance is not enough"""

class CardExpiredError(PaymentError):
    """card has expired"""

def charge(balance, amount):
    if balance < amount:
        raise InsufficientFundsError("insufficient balance")
    return balance - amount

try:
    charge(1000, 5000)
except PaymentError as e:      # catches every payment error at once
    print(f"Payment failed: {e}")
# Payment failed: insufficient balance
```

زیبایی کار در سطربندی خطاهاست: چون `InsufficientFundsError` و `CardExpiredError` هر دو فرزند `PaymentError` هستند، یک `except PaymentError` هر دو را می‌گیرد؛ و هر جا لازم بود، می‌توانید فقط یکی‌شان را جداگانه مدیریت کنید. سلسله‌مراتب استثناها همان سلسله‌مراتب ارث‌بری است — و در فصل یازدهم آن را کامل می‌کنیم.

### چه زمانی از ارث‌بری استفاده نکنیم؟

ارث‌بری وسوسه‌انگیزترین ابزار شیءگرایی است و به همین دلیل، پراشتباه‌ترین هم هست. سه علامت هشدار را جدی بگیرید:

**۱. رابطه Has-A است، نه Is-A.** «خودرو یک موتور دارد»؛ پس `Car` نباید از `Engine` ارث ببرد، بلکه باید یک `Engine` را درون خودش نگه دارد:

```python
class Engine:
    def start(self):
        return "engine on"

class Car:
    def __init__(self):
        self.engine = Engine()     # composition: Car HAS an Engine

    def start(self):
        return self.engine.start()
```

**۲. فقط دنبال بازاستفاده از چند متد هستید.** اینکه «کلاس B چند متد از A لازم دارد» به‌تنهایی ارث‌بری را توجیه نمی‌کند؛ چون با ارث‌بری، B تمام رابط A را هم به دوش می‌کشد — حتی متدهایی که برایش بی‌معنا هستند. برای بازاستفاده‌ی صرف، ترکیب تقریباً همیشه انتخاب سالم‌تری است.

**۳. سلسله‌مراتب دارد عمیق و مبهم می‌شود.** وقتی برای فهمیدن یک متد باید چهار کلاس بالا بروید، ارث‌بری از ابزار به مانع تبدیل شده است.

جمع‌بندی این بحث یک شعار معروف دارد: «ترکیب را به ارث‌بری ترجیح بده» (Composition over Inheritance). این شعار و ابزارهایش موضوع کامل **فصل هفتم** است؛ آن‌جا خواهید دید که بسیاری از سلسله‌مراتب‌های ارث‌بری، با ترکیب تمیزتر درمی‌آیند.

---

## ستون دوم — انتزاع (Abstraction)

### انتزاع چیست؟

هر روز صبح که پشت فرمان می‌نشینید، با یکی از بزرگ‌ترین دستاوردهای مهندسی طرف می‌شوید: می‌چرخانید، ماشین می‌پیچد. پشت این حرکت ساده، جعبه‌فرمان و هیدرولیک و ده‌ها قطعه‌ی دیگر در کارند؛ اما شما لازم نیست هیچ‌کدام را بشناسید. فرمانْ **جزئیات ضروری** را در اختیارتان می‌گذارد (جهت را عوض کن) و **جزئیات غیرضروری** را پنهان می‌کند (چگونگی انتقال نیرو به چرخ‌ها).

**انتزاع** در نرم‌افزار دقیقاً همین است: نمایش‌دادن فقط چیزهایی که استفاده‌کننده لازم دارد، و پنهان‌کردن سازوکار پشت آن. حاصلش رابطی ساده است روی سیستمی پیچیده.

نکته‌ی جالب اینکه شما از اولین روز برنامه‌نویسی، مصرف‌کننده‌ی انتزاع بوده‌اید بی‌آنکه اسمش را بدانید:

```python
numbers = [3, 1, 2]
numbers.sort()          # do you know which algorithm runs behind this?
print(numbers)          # [1, 2, 3]

file = open("data.txt", "w")   # do you know how the OS allocates the file?
file.write("hello")
file.close()
```

الگوریتم `sort` پایتون (Timsort) صدها خط کد پیچیده است؛ `open` با سیستم‌عامل و دیسک سروکله می‌زند. شما هیچ‌کدام را نمی‌دانید و **نیازی هم ندارید بدانید** — و همین «نیاز نداشتن به دانستن» است که کارکردن با این ابزارها را ممکن می‌کند. انتزاع یعنی ساختن همین تجربه برای استفاده‌کنندگان کلاس‌های خودتان.

### سطوح انتزاع: لایه روی لایه

انتزاع یک اتفاق تک‌مرحله‌ای نیست؛ نرم‌افزار از **لایه‌های** انتزاع ساخته می‌شود که هر لایه، پیچیدگی لایه‌ی زیرین را پنهان می‌کند:

```
لایه‌های انتزاع در یک «ارسال پیامک» ساده
═══════════════════════════════════════════════
  کد شما            notify_user(user, text)
     │  «به این کاربر خبر بده»
     ▼
  کلاس پیامک        sms.send(phone, text)
     │  «این متن را به این شماره بفرست»
     ▼
  کتابخانه HTTP     requests.post(url, data)
     │  «این درخواست را به این آدرس بزن»
     ▼
  سیستم‌عامل        socket, TCP/IP, ...
```

قدرت این ساختار در این است که در هر لحظه فقط باید **در یک سطح** فکر کنید. وقتی `notify_user` را می‌نویسید، ذهن‌تان درگیر سوکت و TCP نیست؛ وقتی کتابخانه‌ی HTTP را دیباگ می‌کنید، کاری به منطق «اطلاع‌رسانی به کاربر» ندارید. طراح خوب کسی است که این لایه‌ها را چنان می‌چیند که هیچ‌کس مجبور نباشد هم‌زمان به دو لایه فکر کند.

### انتزاع در طراحی کلاس: رابط ساده روی پیچیدگی

حالا این ایده را در یک کلاس واقعی پیاده کنیم: کلاس ارسال پیامک. استفاده‌کننده‌ی این کلاس نمی‌خواهد بداند پشت‌صحنه چه اتصال HTTP برقرار می‌شود، احراز هویت چطور انجام می‌شود و بدنه‌ی درخواست چه قالبی دارد. او فقط می‌خواهد یک پیام بفرستد:

```python
class SMSSender:
    def __init__(self, api_key):
        self._api_key = api_key

    def send(self, to, message):                # the simple, public interface
        self._authenticate()
        payload = self._build_payload(to, message)
        return self._post(payload)

    # complex details, hidden from the user:
    def _authenticate(self): ...
    def _build_payload(self, to, message): ...
    def _post(self, payload): ...

sender = SMSSender("secret-key")
sender.send("0912...", "Hello")   # the user only ever sees this
```

کل رابط این کلاس از دید استفاده‌کننده، یک متد `send` است؛ سه متد دیگر جزئیات درونی‌اند. (خط زیرینِ ابتدای نام این متدها یک قرارداد پایتونی است به معنی «جزء داخلی، از بیرون صدا نزن»؛ داستان کاملش را در بخش کپسول‌سازی همین فصل می‌خوانید.)

سود واقعی این طراحی فردا معلوم می‌شود: اگر شرکت پیامکی عوض شود و کل منطق `_authenticate` و `_post` از نو نوشته شود، تا وقتی امضای `send` ثابت بماند، **حتی یک خط** از کدهایی که از این کلاس استفاده می‌کنند تغییر نمی‌کند. انتزاع، نقطه‌ی تغییر را ایزوله می‌کند.

از همین مثال یک قاعده‌ی طراحی هم بیرون می‌آید: **رابط را حول نیاز استفاده‌کننده طراحی کنید، نه حول ساختار درونی.** متد `send(to, message)` زبان مسئله است («بفرست به این شماره، این متن را»)؛ اگر به‌جایش متدهایی مثل `set_auth_header` و `post_json` را عمومی می‌کردیم، زبان پیاده‌سازی را به استفاده‌کننده تحمیل کرده بودیم و او مجبور بود پیچیدگی ما را یاد بگیرد.

### کلاس‌های انتزاعی: وقتی انتزاع قرارداد می‌شود

تا این‌جا انتزاع را با «پنهان‌کردن جزئیات پشت متدهای عمومی» ساختیم. پایتون یک پله بالاتر هم می‌رود: گاهی می‌خواهید خودِ مفهوم را به‌صورت انتزاعی تعریف کنید، بی‌آنکه اصلاً پیاده‌سازی‌ای داشته باشد.

به «شکل هندسی» فکر کنید. هر شکلی مساحت دارد — اما «شکلِ خالی» چیست و مساحتش چطور حساب می‌شود؟ پاسخی ندارد. «شکل» یک مفهوم انتزاعی است: قراردادی که می‌گوید هر شکل واقعی باید مساحت‌پذیر باشد، ولی خودش قابل ساختن نیست. پایتون برای بیان دقیق همین ایده، **کلاس انتزاعی (Abstract Base Class)** دارد:

```python
from abc import ABC, abstractmethod

class Shape(ABC):                     # an abstract class
    @abstractmethod
    def area(self):
        """every real shape must implement this"""

    def describe(self):               # abstract classes CAN have real methods too
        return f"a shape with area {self.area()}"

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):                   # fulfilling the contract
        return 3.14159 * self.radius ** 2

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):                   # fulfilling the contract
        return self.width * self.height

# shape = Shape()      # TypeError: Can't instantiate abstract class Shape
c = Circle(2)
print(c.area())        # 12.56636
print(c.describe())    # a shape with area 12.56636 — inherited concrete method
```

سه چیز در این کد اتفاق افتاد که ارزش مکث دارد:

اول، `Shape` از `ABC` ارث برده و `area` با `@abstractmethod` علامت خورده است؛ نتیجه اینکه پایتون **اجازه‌ی ساختن شیء مستقیم از `Shape` را نمی‌دهد**. مفهوم انتزاعی، نمونه‌ی واقعی ندارد — و حالا زبان هم این را ضمانت می‌کند.

دوم، هر زیرکلاسی که `area` را پیاده نکند، خودش هم انتزاعی می‌ماند و قابل ساختن نیست. یعنی قرارداد «هر شکل باید مساحت داشته باشد» دیگر یک توصیه‌ی شفاهی نیست؛ **اجباری زبانی** است که فراموش‌کاری را همان لحظه‌ی ساخت شیء گزارش می‌کند، نه سه هفته بعد در محیط عملیاتی.

سوم، کلاس انتزاعی می‌تواند در کنار متدهای انتزاعی، متدهای واقعی هم داشته باشد (مثل `describe`) که همه‌ی فرزندان به ارث می‌برند. پس ABC هم‌زمان دو نقش بازی می‌کند: قرارداد رابط، و خانه‌ی رفتار مشترک.

این‌جا فقط درِ این بحث را باز کردیم. ابزارهای رسمی‌ترِ تعریف قرارداد — ABCهای پیشرفته‌تر، Protocolها و تفاوت‌هایشان — موضوع اصلی **فصل نهم** است.

### انتزاع یعنی آزادی تعویض پیاده‌سازی

جمع‌بندی این ستون را با نمونه‌ای ببینیم که ارزش عملی انتزاع را یک‌جا نشان می‌دهد: ذخیره‌سازی فایل. برنامه‌ی شما امروز روی دیسک محلی ذخیره می‌کند و شاید سال بعد روی فضای ابری. اگر از روز اول یک رابط انتزاعی تعریف کنید، این مهاجرت دردناک نخواهد بود:

```python
from abc import ABC, abstractmethod

class Storage(ABC):
    @abstractmethod
    def save(self, name, data): ...

    @abstractmethod
    def load(self, name): ...

class LocalStorage(Storage):
    def __init__(self):
        self._files = {}                  # simulating a local disk

    def save(self, name, data):
        self._files[name] = data
        return f"saved {name} locally"

    def load(self, name):
        return self._files[name]

class CloudStorage(Storage):
    def save(self, name, data):
        return f"uploaded {name} to the cloud"

    def load(self, name):
        return f"downloaded {name} from the cloud"

def backup_report(storage, content):       # written once, against the abstraction
    return storage.save("report.txt", content)

print(backup_report(LocalStorage(), "..."))   # saved report.txt locally
print(backup_report(CloudStorage(), "..."))   # uploaded report.txt to the cloud
```

تابع `backup_report` علیه مفهوم انتزاعی `Storage` نوشته شده، نه علیه یک پیاده‌سازی خاص. به این می‌گویند «برنامه‌نویسی برای رابط، نه پیاده‌سازی» — یکی از قدیمی‌ترین توصیه‌های طراحی نرم‌افزار، و میوه‌ی مستقیم انتزاع. توجه کنید که این تابع با هر دو نوع Storage کار کرد؛ این خودش پیش‌پرده‌ی ستون بعدی است.

---

## ستون سوم — چندریختی (Polymorphism)

### چندریختی چیست؟

واژه‌اش را بشکافیم: poly یعنی «چند» و morph یعنی «شکل». **چندریختی** یعنی یک پیام واحد، بسته به گیرنده، شکل‌های مختلفی از رفتار بگیرد. شما به شیء می‌گویید «کارت را بکن» و هر شیء به شیوه‌ی خودش انجامش می‌دهد؛ بی‌آنکه شما لازم باشد بدانید دقیقاً با چه نوعی طرفید.

پیش از هر تعریف رسمی، ببینید که چندریختی از روز اول زیر دست‌تان بوده است:

```python
print(len("hello"))       # 5  — length of a string
print(len([1, 2, 3]))     # 3  — length of a list
print(len({"a": 1}))      # 1  — length of a dict

print(3 + 4)              # 7            — numeric addition
print("py" + "thon")      # python       — string concatenation
print([1, 2] + [3])       # [1, 2, 3]    — list concatenation
```

یک `len`، سه رفتار؛ یک `+`، سه معنا. هر نوع، پاسخ خودش را به پیام مشترک می‌دهد. چندریختی در طراحی کلاس‌ها یعنی ساختن همین تجربه برای انواعی که خودتان تعریف می‌کنید.

در بخش ارث‌بری از شما خواستم حلقه‌ی حقوق کارکنان را به خاطر بسپارید:

```python
for person in staff:
    print(person.monthly_pay())
```

آن حلقه چندریختی در عمل بود: یک پیام (`monthly_pay`) به سه نوع شیء (`Manager`، `Developer`، `Designer`) و سه محاسبه‌ی متفاوت، بدون حتی یک `if`. حالا وقتش است این پدیده را دقیق بشناسیم. در پایتون چندریختی از چند مسیر به دست می‌آید؛ سه مسیر اصلی را یکی‌یکی ببینیم.

### مسیر اول: چندریختی با ارث‌بری و Override

کلاسیک‌ترین مسیر: چند زیرکلاس، متدِ والد مشترک را هرکدام به سبک خودشان Override می‌کنند، و کدِ استفاده‌کننده بی‌خبر از نوع دقیق، همان متد را صدا می‌زند:

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "..."

class Dog(Animal):
    def speak(self):
        return f"{self.name}: Woof!"

class Cat(Animal):
    def speak(self):
        return f"{self.name}: Meow!"

class Duck(Animal):
    def speak(self):
        return f"{self.name}: Quack!"

zoo = [Dog("Rex"), Cat("Whiskers"), Duck("Donald")]
for animal in zoo:
    print(animal.speak())     # one call site, three behaviors
# Rex: Woof!
# Whiskers: Meow!
# Donald: Quack!
```

این‌جا باید یک ابهام رایج را هم روشن کنیم — سوالی که در مصاحبه‌ها زیاد می‌آید: «چندریختی همان Override است؟» نه؛ نسبت‌شان نسبت ابزار و نتیجه است. **Override مکانیزم است**: زیرکلاس متد والد را بازنویسی می‌کند. **چندریختی نتیجه است**: حالا می‌توانید با همه‌ی زیرکلاس‌ها از یک دریچه‌ی واحد کار کنید. Override یکی از راه‌های رسیدن به چندریختی است — و بلافاصله می‌بینیم که تنها راهش نیست.

### مسیر دوم: Duck Typing — چندریختی بدون خویشاوندی

در زبان‌هایی مثل جاوا، چندریختی فقط از مسیر خویشاوندی می‌گذرد: دو کلاس باید والد یا اینترفیس مشترک داشته باشند تا بشود یکسان با آن‌ها رفتار کرد. پایتون سخت‌گیر نیست؛ برای پایتون مهم نیست شیء **از چه تباری است**، مهم این است که **رفتار لازم را دارد**:

```python
class Circle:
    def __init__(self, r):
        self.r = r

    def area(self):
        return 3.14159 * self.r ** 2

class Rectangle:
    def __init__(self, w, h):
        self.w, self.h = w, h

    def area(self):
        return self.w * self.h

# no common parent! and yet:
def total_area(shapes):
    return sum(shape.area() for shape in shapes)

print(total_area([Circle(2), Rectangle(3, 4)]))   # 24.56636
```

`Circle` و `Rectangle` هیچ نسبت فامیلی‌ای ندارند؛ نه والد مشترک، نه اینترفیس. اما `total_area` با هر دو کار می‌کند، چون تنها چیزی که از عناصرش می‌خواهد یک متد `area` است. به این سبک، **Duck Typing** می‌گویند، از این ضرب‌المثل: «اگر مثل اردک راه می‌رود و مثل اردک صدا می‌کند، اردک است.» هویت را از شناسنامه نمی‌پرسیم؛ از رفتار نتیجه می‌گیریم.

Duck Typing سبک امضادار پایتون است و بیشترِ کد پایتونی که خواهید خواند بر آن تکیه دارد. بهایش هم مشخص است: هیچ‌کس در لحظه‌ی تعریف کلاس چک نمی‌کند که `area` را درست پیاده کرده باشید؛ اگر یادتان برود، خطا را اولین‌بار در زمان اجرا می‌بینید. این بها گاهی می‌ارزد و گاهی نه — که می‌رسیم به مسیر سوم.

### مسیر سوم: چندریختی با قرارداد رسمی (ABC)

اگر بخواهید هم انعطاف چندریختی را داشته باشید و هم ضمانتِ «همه‌ی اعضای خانواده این متد را دارند»، دو ستون این فصل به هم می‌رسند: کلاس انتزاعی تعریف می‌کنید (انتزاع)، زیرکلاس‌ها پیاده‌اش می‌کنند (ارث‌بری)، و کد استفاده‌کننده با همه یکسان کار می‌کند (چندریختی):

```python
from abc import ABC, abstractmethod

class Notifier(ABC):
    @abstractmethod
    def notify(self, user, text): ...

class EmailNotifier(Notifier):
    def notify(self, user, text):
        return f"[email to {user}] {text}"

class SMSNotifier(Notifier):
    def notify(self, user, text):
        return f"[sms to {user}] {text}"

class PushNotifier(Notifier):
    def notify(self, user, text):
        return f"[push to {user}] {text}"

def broadcast(notifiers, user, text):
    for n in notifiers:
        print(n.notify(user, text))

channels = [EmailNotifier(), SMSNotifier(), PushNotifier()]
broadcast(channels, "ali", "your order has shipped")
# [email to ali] your order has shipped
# [sms to ali] your order has shipped
# [push to ali] your order has shipped
```

فرق این با Duck Typing در ضمانت است: اگر کسی `TelegramNotifier` بسازد و `notify` را جا بیندازد، در همان لحظه‌ی ساختن شیء `TypeError` می‌گیرد، نه وسط اجرای `broadcast` در محیط واقعی.

انتخاب بین این سه مسیر یک قاعده‌ی سرانگشتی دارد:

| مسیر | کی مناسب است |
|------|--------------|
| ارث‌بری + Override | زیرکلاس‌ها واقعاً Is-A هستند و رفتار مشترک هم دارند |
| Duck Typing | کدهای کوچک و اسکریپت‌ها؛ وقتی انعطاف مهم‌تر از ضمانت است |
| ABC (قرارداد رسمی) | تیم‌های بزرگ و مرزهای مهم سیستم؛ وقتی ضمانت مهم‌تر است |

### چندریختی چه چیزی را حذف می‌کند؟ زنجیره‌ی if/elif

ارزش چندریختی وقتی ملموس می‌شود که دنیای بدونش را ببینید. اگر چندریختی نبود، تابع «مساحت کل» را باید این‌طور می‌نوشتیم:

```python
# without polymorphism — fragile and ever-growing
def total_area_bad(shapes):
    total = 0
    for shape in shapes:
        if shape.kind == "circle":
            total += 3.14159 * shape.r ** 2
        elif shape.kind == "rectangle":
            total += shape.w * shape.h
        # every new shape forces yet another elif here!
    return total
```

این تابع دو بیماری دارد. اول اینکه منطق مساحتِ همه‌ی شکل‌ها را از دل کلاس‌هایشان بیرون کشیده و یک‌جا تلنبار کرده؛ دانشی که باید کنار داده‌اش می‌ماند، آواره شده است. دوم — بدتر — اینکه این تابع هرگز تمام نمی‌شود: با هر شکل جدید باید بازش کنید و یک `elif` دیگر بچسبانید، و هر بازکردنِ کدِ کارکرده یعنی ریسک شکستنش.

نسخه‌ی چندریختانه هر دو بیماری را درمان می‌کند: منطق هر شکل داخل کلاس خودش می‌ماند و `total_area` برای همیشه بسته می‌شود. افزودن شکل جدید یعنی فقط نوشتن یک کلاس تازه با متد `area` — بدون دست‌زدن به هیچ کد موجودی. این خاصیت («باز برای توسعه، بسته برای تغییر») همان اصل Open/Closed است که در فصل هشتم به‌عنوان یکی از اصول SOLID می‌بینید؛ چندریختی موتور محرک آن اصل است.

هر وقت در کدتان زنجیره‌ی `if/elif` روی «نوع چیزها» دیدید، بایستید و بپرسید: آیا این زنجیره در واقع یک متد چندریخت نیست که آواره شده؟

### یک نکته‌ی تکمیلی: Overriding در برابر Overloading

چون در مصاحبه‌ها مدام کنار هم می‌آیند، این دو واژه‌ی شبیه‌به‌هم را از هم جدا کنیم. **Overriding** همان است که دیدیم: زیرکلاس متد والد را بازنویسی می‌کند. **Overloading** چیز دیگری است: در زبان‌هایی مثل جاوا می‌توانید چند نسخه از یک متد با تعداد یا نوع پارامترهای متفاوت در **همان کلاس** تعریف کنید و کامپایلر بر اساس آرگومان‌ها یکی را انتخاب کند.

پایتون Overloading به آن معنا ندارد — تعریف دوباره‌ی متد هم‌نام، نسخه‌ی قبلی را به‌سادگی جایگزین می‌کند. جای خالی‌اش را ابزارهای انعطاف‌پذیرتر خود پایتون پر می‌کنند: مقدار پیش‌فرض و آرگومان‌های کلیدواژه‌ای:

```python
class Report:
    def export(self, path, fmt="pdf", compress=False):
        return f"exporting {path} as {fmt} (compress={compress})"

r = Report()
print(r.export("a.txt"))                      # one method,
print(r.export("a.txt", fmt="csv"))           # many calling shapes —
print(r.export("a.txt", compress=True))       # no overloading needed
```

اگر مصاحبه‌گر پرسید «پایتون Overloading دارد؟»، پاسخ دقیق این است: به سبک جاوا نه؛ اما با آرگومان‌های پیش‌فرض و کلیدواژه‌ای به همان نتیجه می‌رسد.

---

## ستون چهارم — کپسول‌سازی (Encapsulation)

### کپسول‌سازی چیست و چرا مهم است؟

به کپسول دارو فکر کنید: محتوای فعال، درون پوسته‌ای بسته‌بندی شده که هم اجزای مرتبط را کنار هم نگه می‌دارد و هم از تماس مستقیم محافظت می‌کند. **کپسول‌سازی** در شیءگرایی همین دو کار را با هم انجام می‌دهد: **بسته‌بندی** داده و رفتار مربوط به آن در یک واحد (کلاس)، و **کنترل دسترسی** به آن داده.

ایده‌ی مرکزی این است: داده‌ی درونی یک شیء نباید بی‌واسطه از بیرون دستکاری شود؛ باید از طریق متدهایی که خود کلاس تعریف کرده — و با قوانین خود کلاس — تغییر کند. چرا؟ چون بدون این کنترل، هر گوشه‌ای از برنامه می‌تواند شیء را به حالت نامعتبر ببرد:

```python
class BankAccount:
    def __init__(self, balance):
        self.balance = balance

acc = BankAccount(1000)
acc.balance = -5000     # disaster! nobody stopped this
```

هیچ بانکی موجودی منفی ۵۰۰۰ تومانی ندارد، اما این کد بدون هیچ اعتراضی اجرا شد. مشکل این نیست که «کسی عمداً خرابکاری می‌کند»؛ مشکل این است که شش ماه بعد، یک همکار خسته در یک گوشه‌ی دیگر برنامه، بی‌خبر از قوانین حساب بانکی، همین یک خط را می‌نویسد و باگی می‌کارد که پیداکردنش روزها طول می‌کشد.

با کپسول‌سازی، داده را درونی علامت می‌زنیم و تغییرش را فقط از مسیر متدهای کنترل‌شده مجاز می‌کنیم:

```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance          # marked as internal

    def withdraw(self, amount):
        if amount > self._balance:
            raise ValueError("insufficient balance")
        self._balance -= amount

    def get_balance(self):
        return self._balance
```

حالا تنها راهِ کم‌کردن موجودی، عبور از گیت `withdraw` است و آن گیت، قانون «برداشت بیش از موجودی ممنوع» را اجرا می‌کند. به چنین قانونی که باید همیشه درباره‌ی یک شیء برقرار بماند، **ناوردا (Invariant)** می‌گویند — مثلاً «موجودی هرگز منفی نمی‌شود». کپسول‌سازی یعنی جمع‌کردن همه‌ی راه‌های نقض ناورداها پشت گیت‌های بازرسی.

### سطوح دسترسی در پایتون: Public، Protected و Private

پایتون برخلاف جاوا کلیدواژه‌های `public` و `private` ندارد و این مفاهیم را با **قراردادهای نام‌گذاری** بیان می‌کند:

- به‌طور پیش‌فرض، **همه‌چیز در پایتون عمومی (Public)** است.
- افزودن **یک زیرخط** (`_`) به ابتدای نام، آن را **محافظت‌شده (Protected)** علامت می‌زند.
- افزودن **دو زیرخط** (`__`) به ابتدای نام، آن را **خصوصی (Private)** می‌کند.

| نام | سطح | معنا | به ارث می‌رسد؟ |
|-----|------|------|----------------|
| `name` | Public | آزادانه قابل استفاده در همه‌جا | بله |
| `_name` | Protected | برای استفاده‌ی داخلی کلاس و زیرکلاس‌هایش؛ از بیرون دست نزنید | بله |
| `__name` | Private | مخصوص خود کلاس؛ با Name Mangling از تداخل نام در ارث‌بری جلوگیری می‌کند | مستقیم نه (نامش تغییر می‌کند) |

حالا هر سطح را دقیق ببینیم.

**Public (`name`)** — حالت پیش‌فرض. هیچ محدودیتی در کار نیست و هر کدی از هر جا می‌تواند بخواند و بنویسد. attributeهایی را Public بگذارید که واقعاً جزء رابط کلاس‌اند و تغییر آزادشان بی‌خطر است.

**Protected (`_name`)** — یک زیرخط، یک **قرارداد** است، نه یک قفل. به خواننده‌ی کد می‌گوید: «این عضو برای استفاده‌ی داخلی کلاس طراحی شده؛ لطفاً از بیرون به آن دست نزن.» پایتون هیچ مانع فنی‌ای نمی‌گذارد و زیرکلاس‌ها هم به‌راحتی آن را به ارث می‌برند و استفاده می‌کنند:

```python
class Config:
    def __init__(self):
        self.public_value = 1
        self._internal_value = 2   # protected by convention

c = Config()
print(c._internal_value)   # 2 — it works, but you should not do this
```

**Private (`__name`)** — دو زیرخط دیگر صرفاً قرارداد نیست؛ مفسر پایتون واقعاً وارد عمل می‌شود و نام عضو را تغییر می‌دهد. این سازوکار آن‌قدر مهم است که بخش خودش را می‌خواهد.

### Name Mangling: سازوکار پشت دو زیرخط

وقتی نامی در بدنه‌ی کلاس با `__` شروع شود (و با `__` تمام نشود)، پایتون آن را در پشت‌صحنه به `_ClassName__name` تغییر می‌دهد. به این کار **Name Mangling** (دستکاری نام) می‌گویند. نتیجه این است که دسترسی از بیرون با نام اصلی شکست می‌خورد:

```python
class Account:
    def __init__(self):
        self.__pin = "1234"      # Python stores it as _Account__pin

a = Account()
# print(a.__pin)        # AttributeError!
print(a._Account__pin)  # 1234 — still reachable, just harder and uglier
```

پس این یک قفل واقعی هم نیست — با دانستن قاعده، هنوز می‌شود به آن رسید. اگر هدف پنهان‌سازی امنیتی نیست، این سازوکار به چه دردی می‌خورد؟ پاسخ: **جلوگیری از تصادم نام در ارث‌بری**. ببینید بدون آن چه می‌شد و با آن چه می‌شود:

```python
class Logger:
    def __init__(self):
        self.__count = 0            # becomes _Logger__count

    def log(self, msg):
        self.__count += 1
        return f"[{self.__count}] {msg}"

class TimedLogger(Logger):
    def __init__(self):
        super().__init__()
        self.__count = 100          # becomes _TimedLogger__count — no clash!

t = TimedLogger()
print(t.log("first"))    # [1] first — Logger's counter is safe
print(t.log("second"))   # [2] second
print(t._TimedLogger__count)   # 100 — the child's own, separate attribute
```

نویسنده‌ی `TimedLogger` نمی‌دانست (و نباید لازم باشد بداند) که والدش درون خودش چیزی به نام `__count` دارد. اگر Name Mangling نبود، مقداردهی او شمارنده‌ی والد را له می‌کرد و `log` از کار می‌افتاد. با Name Mangling هر کلاس فضای خصوصی خودش را دارد: `_Logger__count` و `_TimedLogger__count` دو attribute جدا هستند. این است کاربرد واقعی دو زیرخط: نه دیوار امنیتی، بلکه **بیمه‌ی تصادم** برای attributeهایی که به هیچ زیرکلاسی مربوط نیستند.

### فلسفه‌ی پایتون: «همه این‌جا بزرگسالیم»

شاید هنوز این سوال در ذهن‌تان باشد: چرا پایتون مثل جاوا یک `private` واقعی و قفل‌دار ندارد؟ پاسخ به فلسفه‌ی زبان برمی‌گردد که در جامعه‌ی پایتون با این جمله معروف است: *«We are all consenting adults here»* — «این‌جا همه‌ی ما بزرگسالیم.»

پایتون به‌جای آنکه با قفل جلوی شما را بگیرد، فرض می‌کند حرفه‌ای هستید و مسئولیت رفتارتان را می‌پذیرید. زیرخط‌ها علامت‌اند، نه دیوار: `_` می‌گوید «دست نزن»، `__` دست‌درازی اتفاقی را سخت‌تر هم می‌کند؛ اما اگر آگاهانه تصمیم بگیرید عبور کنید (مثلاً در یک تست یا دیباگ اضطراری)، زبان مانع‌تان نمی‌شود — فقط ردپای تصمیم‌تان (`_Account__pin`) آن‌قدر زشت است که در Code Review دیده شود.

**Tradeoff:** رویکرد جاوا (private اجباری) خطای کمتر و انعطاف کمتر می‌دهد؛ رویکرد پایتون آزادی بیشتر و مسئولیت بیشتر. هیچ‌کدام مطلقاً بهتر نیستند؛ دو پاسخ متفاوت به یک سوال‌اند: «به برنامه‌نویس چقدر اعتماد کنیم؟»

### کپسول‌سازی فقط برای داده نیست

قراردادهای دسترسی برای متدها هم به‌کار می‌روند و اتفاقاً یکی از نشانه‌های طراحی پخته همین است: کلاس، متدهای کمکی درونی‌اش را با `_` علامت می‌زند تا رابط عمومی‌اش کوچک و روشن بماند:

```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance          # protected data

    def _validate_amount(self, amount):  # internal helper — not part of the API
        if amount <= 0:
            raise ValueError("amount must be positive")

    def withdraw(self, amount):          # the public interface
        self._validate_amount(amount)
        if amount > self._balance:
            raise ValueError("insufficient balance")
        self._balance -= amount

    def get_balance(self):
        return self._balance
```

از دید استفاده‌کننده، این کلاس فقط دو متد دارد: `withdraw` و `get_balance`. متد `_validate_amount` جزء آشپزخانه‌ی کلاس است؛ می‌توانیم فردا اسمش را عوض کنیم، دو تکه‌اش کنیم یا حذفش کنیم، بی‌آنکه هیچ کد بیرونی بشکند. هرچه سطح عمومی یک کلاس کوچک‌تر باشد، آزادی نویسنده‌اش برای تغییر درونیات بیشتر است.

### مثال جامع: حساب بانکی با ناورداهای واقعی

همه‌ی قطعات این ستون را در یک کلاس کامل جمع کنیم — با دو ناوردا: «موجودی هرگز منفی نمی‌شود» و «تاریخچه‌ی تراکنش‌ها دقیقاً همه‌ی تغییرات را ثبت می‌کند»:

```python
class BankAccount:
    def __init__(self, owner, initial_balance=0):
        self._owner = owner
        self._balance = 0
        self._transactions = []          # invariant: records every change
        if initial_balance > 0:
            self.deposit(initial_balance)

    # --- internal helpers ---
    def _validate_amount(self, amount):
        if amount <= 0:
            raise ValueError("amount must be a positive number")

    def _record(self, kind, amount):
        self._transactions.append((kind, amount))

    # --- public interface ---
    def deposit(self, amount):
        self._validate_amount(amount)
        self._balance += amount
        self._record("deposit", amount)

    def withdraw(self, amount):
        self._validate_amount(amount)
        if amount > self._balance:       # invariant: balance never goes negative
            raise ValueError("insufficient balance")
        self._balance -= amount
        self._record("withdraw", amount)

    def get_balance(self):
        return self._balance

    def get_history(self):
        return list(self._transactions)  # a copy — the original stays protected

acc = BankAccount("Ali", 1000)
acc.deposit(500)
acc.withdraw(300)
print(acc.get_balance())    # 1200
print(acc.get_history())    # [('deposit', 1000), ('deposit', 500), ('withdraw', 300)]

# every illegal path is closed:
# acc.withdraw(999999)   # ValueError: insufficient balance
# acc.deposit(-50)       # ValueError: amount must be a positive number
```

به دو ظرافت این کد دقت کنید. اول، حتی سازنده هم برای واریز اولیه از `deposit` عبور می‌کند؛ یعنی مسیر ورود پول به حساب **دقیقاً یکی** است و قانون و ثبت تراکنش هیچ‌وقت دور زده نمی‌شوند. دوم، `get_history` یک **کپی** از لیست را برمی‌گرداند، نه خودش را؛ وگرنه استفاده‌کننده می‌توانست با `acc.get_history().clear()`ِ نسخه‌ی اصلی، تاریخچه را پاک کند و کپسول از همان سوراخ می‌شکست. کپسول‌سازی واقعی یعنی به همین جزئیات هم فکرکردن.

یک خبر خوب برای پایان این ستون: الگوی `get_balance()` که این‌جا دیدید، در پایتون شکل زیباتری هم دارد. پایتون با `@property` اجازه می‌دهد دسترسی کنترل‌شده را پشت ظاهرِ یک attribute ساده پنهان کنید — یعنی `acc.balance` بنویسید اما زیرش متد اجرا شود. این ابزار، ستاره‌ی **فصل پنجم** است.

---

## چهار ستون در یک قاب

برای جمع‌بندی، برنامه‌ی کوچکی بنویسیم که هر چهار اصل در آن هم‌زمان کار می‌کنند — و کنار هر قطعه، نام اصلش را بگذاریم:

```python
from abc import ABC, abstractmethod

class Sensor(ABC):                        # Abstraction: an abstract contract
    def __init__(self, location):
        self._location = location         # Encapsulation: protected state
        self._readings = []

    @abstractmethod
    def read(self):
        """each sensor type must know how to read its value"""

    def record(self):                     # shared behavior for all children
        value = self.read()
        self._readings.append(value)
        return f"{self._location}: {value}"

    def history(self):
        return list(self._readings)       # Encapsulation: return a copy

class TemperatureSensor(Sensor):          # Inheritance: Is-A Sensor
    def read(self):
        return "23°C"

class HumiditySensor(Sensor):             # Inheritance: Is-A Sensor
    def read(self):
        return "40%"

sensors = [TemperatureSensor("room"), HumiditySensor("basement")]
for s in sensors:                         # Polymorphism: one loop, many types
    print(s.record())
# room: 23°C
# basement: 40%
```

`Sensor` قرارداد انتزاعی و رفتار مشترک را تعریف می‌کند؛ زیرکلاس‌ها با ارث‌بری صاحب آن می‌شوند و فقط `read` مخصوص خودشان را می‌نویسند؛ حلقه‌ی پایانی بدون هیچ پرس‌وجویی از نوع، با همه یکسان کار می‌کند؛ و داده‌ی درونی هر حسگر پشت قراردادهای دسترسی محفوظ است. چهار اصل، چهار ابزار جدا نیستند — چهار وجه یک طراحی خوب‌اند.

## خلاصه

| اصل | ایده‌ی اصلی | کلیدواژه‌ها |
|-----|-------------|--------------|
| ارث‌بری | بازاستفاده از رفتار مشترک (رابطه‌ی Is-A) | `super()`, Override, انواع ارث‌بری |
| انتزاع | پنهان‌کردن پیچیدگی پشت رابط ساده | لایه‌های انتزاع, `ABC`, `@abstractmethod` |
| چندریختی | یک پیام، رفتارهای مختلف | Override, Duck Typing, حذف if/elif |
| کپسول‌سازی | کنترل دسترسی به داده و محافظت از ناورداها | `_`, `__`, Name Mangling |

```
چهار ستون شیءگرایی
════════════════════════════════════
  ارث‌بری      ──►  «رفتار مشترک را یک‌بار می‌نویسم»
  انتزاع       ──►  «فقط چیزی که لازم داری را می‌بینی»
  چندریختی     ──►  «با همه یکسان رفتار می‌کنم»
  کپسول‌سازی   ──►  «داده‌ام را خودم کنترل می‌کنم»
```

**نکته‌ی طلایی:** این چهار اصل ابزارند، نه فرمان. کپسول‌سازی افراطی کد را دست‌وپاگیر می‌کند؛ ارث‌بری بی‌جا آن را شکننده می‌کند؛ انتزاعِ زودهنگام لایه‌هایی می‌سازد که هیچ‌چیز را ساده نمی‌کنند. همیشه بپرسید: «این اصل، این‌جا، چه مشکلی را حل می‌کند؟» اگر پاسخ روشنی نداشتید، شاید لازمش ندارید. تفکر مبتنی بر Tradeoff همیشه بالاتر از پیروی کورکورانه از اصول است.

## پرسش‌های مصاحبه

چهار ستون، چهار سوال همیشگی:

**۱. «کی از ارث‌بری استفاده می‌کنی و کی نه؟»** — پاسخ ممتاز حول آزمون Is-A می‌چرخد: ارث‌بری فقط وقتی که جمله‌ی «X یک نوع Y است» طبیعی باشد و فرزند بتواند جانشین بی‌دردسر والد شود. برای رابطه‌ی Has-A یا بازاستفاده‌ی صرف از چند متد، ترکیب (Composition) درست است. اگر شعار «Composition over Inheritance» را با یک مثال (مثل Car و Engine) بگویید، امتیاز کامل گرفته‌اید.

**۲. «فرق انتزاع و کپسول‌سازی؟»** — پرتکرارترین سوال تئوری مصاحبه‌های فارسی. بهترین راه پاسخ، تعریف هرکدام در زمین خودش است: انتزاع درباره‌ی **طراحی رابط** است — تصمیم می‌گیرد استفاده‌کننده چه چیزی را ببیند و به چه چیزی اصلاً فکر نکند (فرمان خودرو). کپسول‌سازی درباره‌ی **محافظت از حالت درونی** است — تضمین می‌کند داده فقط از مسیرهای کنترل‌شده تغییر کند (موجودی منفی نشود). این دو معمولاً با هم ظاهر می‌شوند اما به دو سوال متفاوت جواب می‌دهند: «چه نشان بدهم؟» و «چطور محافظت کنم؟»

**۳. «چندریختی را با یک مثال کد واقعی توضیح بده.»** — تابعی که با انواع مختلف از طریق یک متد مشترک کار می‌کند، بی‌آنکه نوعشان را بپرسد؛ و حذف زنجیره‌ی if/elif با واگذاری تصمیم به خود اشیاء. اگر اضافه کنید «چندریختی همان چیزی است که اصل Open/Closed را ممکن می‌کند»، پیوند فصل‌ها را هم نشان داده‌اید. و اگر Duck Typing را هم گفتید، یادتان باشد بهایش را هم بگویید: انعطاف در ازای ازدست‌دادن بررسی زودهنگام.

**۴. «پایتون که private واقعی ندارد؛ پس کپسول‌سازی‌اش شوخی است؟»** — نه؛ فلسفه‌اش فرق دارد: قرارداد به‌جای قفل («همه این‌جا بزرگسالیم»). `_` یعنی «دست نزن»، و `__` با Name Mangling دست‌درازی اتفاقی و تصادم نام در ارث‌بری را سخت می‌کند. نکته‌ی طلایی برای جواب: هدف کپسول‌سازی جلوگیری از «دشمن» نیست؛ جلوگیری از «اشتباه صادقانه» است.

**در فصل بعد:** سراغ ابزارهایی می‌رویم که پایتون را پایتون می‌کنند — `@property` که دسترسی کنترل‌شده را پشت ظاهر یک attribute ساده می‌برد، دکوراتورهای `@classmethod` و `@staticmethod`، و خانواده‌ی متدهای جادویی (از `__str__` و `__repr__` برای نمایش اشیاء تا `__call__` و عملگرها) که کلاس‌های شما را با خود زبان یکپارچه می‌کنند.
