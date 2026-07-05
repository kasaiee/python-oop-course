# فصل سیزدهم: متاکلاس‌ها و برنامه‌نویسی فراکلاس

---

## مقدمه

به یکی از عمیق‌ترین و مرموزترین مباحثِ پایتون رسیدیم: **متاکلاس‌ها**.

جمله‌ای معروف هست: «متاکلاس‌ها جادوی عمیق‌تری هستند که ۹۹٪ کاربران هرگز لازمشان ندارند. اگر شک دارید که به آن‌ها نیاز دارید، نیاز ندارید.» — این جمله را تیم پیترز (از توسعه‌دهندگانِ اصلیِ پایتون) گفته. پس چرا یادشان بگیریم؟ چون همان ۱٪ موارد، در قلبِ فریمورک‌هایی مثل جنگو، SQLAlchemy و Pydantic‌اند. برای فهمِ عمیقِ این ابزارها — و پایتون — باید بدانید متاکلاس چیست.

اگر متاکلاس اولین بار گیج‌کننده به‌نظر رسید، طبیعی است. با آرامش پیش بروید. هدفِ این فصل «درک» است، نه اینکه فردا در پروژه‌تان متاکلاس بنویسید.

---

## چرا متاکلاس‌ها را یاد بگیریم؟

بیشترِ برنامه‌نویسان هرگز متاکلاس **نمی‌نویسند**، و این کاملاً درست است. اما درکشان کمک می‌کند:

- فریمورک‌های بزرگ را عمیق بفهمید (بدانید پشتِ `class Meta`ی جنگو چه می‌گذرد)
- کتابخانه‌های پیچیده و منعطف بنویسید (اگر روزی لازم شد)
- کدِ تکراری را در سطحِ **کلاس‌ها** (نه فقط اشیاء) حذف کنید
- به عمقِ زبانِ پایتون پی ببرید

اما به‌یاد داشته باشید: در فصل ششم دیدیم که `__init_subclass__` و دکوراتورهای کلاسی، اغلب جایگزینِ ساده‌ترِ متاکلاس‌اند. متاکلاس آخرین ابزارِ جعبه‌ابزار است، نه اولین.

---

## چگونه متاکلاس‌ها را یاد بگیریم؟

### همه‌چیز شیء است — حتی کلاس‌ها

در فصل اول گفتیم «در پایتون همه‌چیز شیء است». این شاملِ **خودِ کلاس‌ها** هم می‌شود. یک کلاس، خودش یک شیء است — شیئی که می‌تواند اشیای دیگر بسازد.

اگر کلاس یک شیء است، پس از روی چه چیزی ساخته شده؟ پاسخ: از روی یک **متاکلاس**. همان‌طور که اشیای معمولی نمونه‌ی یک کلاس‌اند، کلاس‌ها نمونه‌ی یک متاکلاس‌اند.

```python
class User:
    pass

print(type(User))          # <class 'type'> — the User class is an instance of type!
print(type(User()))        # <class '__main__.User'> — the object is an instance of User

# that is:
#   an ordinary object   →  instance of  →  class (User)
#   class (User)   →  instance of  →  metaclass (type)
```

**متاکلاسِ پیش‌فرضِ پایتون `type` است.** هر کلاسی که تعریف می‌کنید، در پشت‌صحنه توسطِ `type` ساخته می‌شود.

### type چگونه کلاس‌ها را می‌سازد

`type` دو کاربرد دارد. یکی را می‌شناسید (گرفتنِ نوعِ یک شیء). اما `type` با سه آرگومان، **یک کلاسِ تازه می‌سازد** — دقیقاً همان کاری که `class` می‌کند:

```python
# these two are completely equivalent:

class Dog:
    legs = 4
    def bark(self):
        return "Woof"

# the manual equivalent with type:
def bark(self):
    return "Woof"

Dog2 = type("Dog2", (), {"legs": 4, "bark": bark})
#           name     parents  namespace (attributes and methods)

d = Dog2()
print(d.legs)      # 4
print(d.bark())    # Woof
```

این نکته‌ی کلیدیِ فصل است: عبارتِ `class` فقط یک **میان‌بُرِ نحوی** برای فراخوانیِ `type(name, bases, namespace)` است. وقتی می‌نویسید `class Dog:`, پایتون در پشت‌صحنه `type` را با نام، والدها و بدنه صدا می‌زند.

### نوشتن یک متاکلاس سفارشی

یک متاکلاس، کلاسی است که از `type` ارث می‌برد و رفتارِ **ساختِ کلاس** را سفارشی می‌کند. مثال: متاکلاسی که هنگامِ ساختِ هر کلاس، نامِ متدهایش را لاگ می‌کند:

```python
class LoggingMeta(type):
    def __new__(mcs, name, bases, namespace):
        # here we intervene before the class is created
        methods = [k for k, v in namespace.items() if callable(v)]
        print(f"class {name} is being created with methods {methods}")
        return super().__new__(mcs, name, bases, namespace)

class Service(metaclass=LoggingMeta):     # use the custom metaclass
    def start(self): ...
    def stop(self): ...

# output when the class is defined (not when an object is created):
# class Service is being created with methods ['start', 'stop']
```

با `metaclass=LoggingMeta`, هنگامِ تعریفِ `Service`, متدِ `__new__`ِ متاکلاس صدا زده می‌شود و می‌تواند در فرایندِ ساخت مداخله کند — مثلاً بررسی، تغییر، یا ثبتِ کلاس.

### متاکلاس در برابر دکوراتور کلاسی

هم متاکلاس و هم دکوراتورِ کلاسی می‌توانند کلاس را تغییر دهند. تفاوت:

```python
# class decorator — simpler, acts after the class is created
def add_repr(cls):
    cls.__repr__ = lambda self: f"<{cls.__name__}>"
    return cls

@add_repr
class Point:
    pass

print(repr(Point()))     # <Point>
```

| | دکوراتور کلاسی | متاکلاس |
|---|----------------|---------|
| زمان عمل | **بعد از** ساختِ کلاس | **حین** ساختِ کلاس |
| پیچیدگی | ساده | پیچیده |
| ارث‌بری | به زیرکلاس‌ها منتقل نمی‌شود | به زیرکلاس‌ها منتقل می‌شود |
| کاربرد | تغییرِ یک کلاسِ مشخص | کنترلِ خانواده‌ای از کلاس‌ها |

**قاعده:** اگر فقط می‌خواهید یک کلاسِ مشخص را تغییر دهید، دکوراتورِ کلاسی؛ اگر می‌خواهید همه‌ی زیرکلاس‌های یک سلسله‌مراتب را کنترل کنید، متاکلاس. و اگر فقط می‌خواهید هنگامِ تعریفِ زیرکلاس کاری کنید، `__init_subclass__` (فصل ششم) از هر دو ساده‌تر است.

### __prepare__

`__prepare__` (پیشرفته) اجازه می‌دهد **نوعِ فضای‌نامِ** کلاس را پیش از اجرای بدنه‌اش کنترل کنید. پیش‌فرض یک `dict` معمولی است، اما مثلاً می‌توانید `OrderedDict` بدهید تا ترتیبِ تعریفِ اعضا حفظ شود (که پیش از پایتون ۳.۷ که dictها ترتیب‌دار شدند، مهم بود):

```python
class OrderedMeta(type):
    @classmethod
    def __prepare__(mcs, name, bases):
        print("namespace is being prepared")
        return dict()        # you can return a custom type

    def __new__(mcs, name, bases, namespace):
        return super().__new__(mcs, name, bases, namespace)

class Form(metaclass=OrderedMeta):
    field1 = "a"
    field2 = "b"
```

`__prepare__` بسیار تخصصی است و به‌ندرت لازم می‌شود.

### type در برابر metaclass در تعریف کلاس

جمع‌بندیِ رابطه‌ها:

```
شیء ← نمونه‌ی ← کلاس ← نمونه‌ی ← متاکلاس ← نمونه‌ی ← type
u    ←         User  ←          (type یا سفارشی) ←   type
```

`type` هم متاکلاسِ پیش‌فرض است و هم متاکلاسِ خودش (`type(type)` برابرِ `type` است). این نقطه، انتهای زنجیره است.

---

## ابزارهای مرتبط

چند قابلیتِ مرتبط با متاکلاس که کاربردی‌ترند:

### __set_name__ در Descriptorها

در فصل پنجم Descriptor را دیدیم. `__set_name__` (از پایتون ۳.۶) به Descriptor اجازه می‌دهد **نامِ attributeی که به آن نسبت داده شده** را بداند — بدونِ نیاز به متاکلاس:

```python
class Field:
    def __set_name__(self, owner, name):
        self.name = name                # the field name is provided automatically
        print(f"field '{name}' defined in class {owner.__name__}")

    def __get__(self, obj, objtype=None):
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value):
        obj.__dict__[self.name] = value

class User:
    username = Field()
    email = Field()

# output when the class is defined:
# field 'username' defined in class User
# field 'email' defined in class User
```

این دقیقاً همان مکانیزمی است که به `dataclass` و ORMهای SQLAlchemy اجازه می‌دهد نامِ فیلدها را خودکار بفهمند. پیش از `__set_name__`, این کار نیازمندِ متاکلاس بود؛ حالا ساده‌تر شده.

### __class_getitem__ و Generic

`__class_getitem__` (از پایتون ۳.۷) تعیین می‌کند وقتی از سینتکسِ `ClassName[X]` استفاده می‌کنید چه شود. همین است که به `Repository[User]` (فصل نهم) معنا می‌دهد:

```python
class Container:
    def __class_getitem__(cls, item):
        print(f"Container with parameter {item}")
        return cls

Container[int]      # Container with parameter <class 'int'>
```

به‌لطفِ این متد، `list[int]`، `dict[str, int]` و Genericها بدونِ متاکلاس کار می‌کنند.

### __instancecheck__ و __subclasscheck__

این دو متدِ متاکلاس، رفتارِ `isinstance` و `issubclass` را سفارشی می‌کنند:

```python
class LenMeta(type):
    def __instancecheck__(cls, instance):
        # count anything that has len as an "instance"
        return hasattr(instance, "__len__")

class Sized(metaclass=LenMeta):
    pass

print(isinstance([1, 2, 3], Sized))     # True — a list has len
print(isinstance("Hello", Sized))         # True — a string has len
print(isinstance(42, Sized))            # False — a number has no len
```

همین مکانیزم است که پشتِ `isinstance(x, Iterable)` در ماژولِ `collections.abc` کار می‌کند: بررسیِ **قابلیت** به‌جای ارث‌بریِ واقعی. این پلی است به Protocolهای فصل نهم.

---

## چه زمانی متاکلاس بنویسیم؟

خلاصه‌ی صادقانه: **تقریباً هرگز، مگر کتابخانه/فریمورک بنویسید.** برای بیشترِ کارها:

- می‌خواهید یک کلاسِ مشخص را تغییر دهید → **دکوراتور کلاسی**
- می‌خواهید هنگامِ تعریفِ زیرکلاس کاری کنید → **`__init_subclass__`** (فصل ششم)
- می‌خواهید attribute را کنترل کنید → **Descriptor** یا **property** (فصل پنجم)
- می‌خواهید `ClassName[X]` کار کند → **`__class_getitem__`**

متاکلاس را فقط وقتی به‌کار ببرید که واقعاً باید در **قلبِ فرایندِ ساختِ کلاس** مداخله کنید و هیچ ابزارِ ساده‌تری کفایت نمی‌کند. این، تعریفِ عملیِ «۹۹٪ لازمش ندارند» است.

---

## خلاصه

```
سلسله‌مراتبِ نوع در پایتون
════════════════════════════════════════════════
  u = User()

  u      →  نمونه‌ی  →  User   (کلاس)
  User   →  نمونه‌ی  →  type   (متاکلاس)
  type   →  نمونه‌ی  →  type   (خودش)

  class Dog: ...
      ≡  Dog = type("Dog", (bases), {namespace})
```

| مفهوم | نکته‌ی کلیدی |
|-------|--------------|
| متاکلاس | کلاسی که کلاس‌ها را می‌سازد؛ پیش‌فرض `type` |
| `type(name, bases, ns)` | ساختِ دستیِ کلاس |
| متاکلاسِ سفارشی | ارث از `type`، سفارشیِ `__new__` |
| متاکلاس در برابر دکوراتور | حین‌ساخت در برابر بعد‌از‌ساخت |
| `__set_name__` | Descriptor نامش را می‌فهمد (بی‌نیاز از متاکلاس) |
| `__class_getitem__` | معنا به `Class[X]` (Generics) |
| `__instancecheck__` | سفارشیِ `isinstance` |
| `__init_subclass__` | جایگزینِ ساده‌ترِ متاکلاس (فصل ۶) |

**نکته‌ی طلایی:** متاکلاس‌ها قدرتِ خام‌اند، و قدرتِ خام مسئولیت می‌آورد. زیباییِ فهمِ متاکلاس در این نیست که بنویسیدشان، بلکه در این است که وقتی کدِ جنگو یا SQLAlchemy را می‌خوانید، دیگر جادو به‌نظر نرسند. برای کدِ خودتان، تقریباً همیشه ابزارِ ساده‌تری هست — و انتخابِ ساده‌ترین ابزارِ کافی، خودش نشانه‌ی مهندسِ باتجربه است.

---

**در فصل بعد:** از انتزاعِ عمیق به موضوعی عمل‌گرایانه‌تر برمی‌گردیم: **سازمان‌دهیِ کد در ماژول‌ها و پکیج‌ها** و رابطه‌اش با شیءگرایی.
