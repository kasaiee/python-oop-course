# فصل نوزدهم — پروژه چهارم: ORM خودت را بساز

> این پروژه با سه پروژه‌ی قبلی یک فرقِ اساسی دارد: آنجا «استفاده‌کننده‌ی» ابزارها بودید؛ اینجا **ابزارساز** می‌شوید. قرار است نسخه‌ی کوچکِ چیزی را بسازیم که هر روز در Django و SQLAlchemy استفاده می‌شود — و بعد از آن، دیگر هیچ فریمورکی برایتان «جعبه‌ی سیاه» نخواهد بود.

---

## مقدمه

اگر با Django کار کرده باشید، این کد برایتان عادی است:

```python
class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.IntegerField(default=0)
```

تا حالا از خودتان پرسیده‌اید این چند خطِ ساده چطور کار می‌کند؟ نه `__init__`ی نوشته‌اید، نه جدولی ساخته‌اید؛ اما `Product(name="کیبورد")` کار می‌کند، اعتبارسنجی دارد، و `Product.objects.all()` از دیتابیس می‌خوانَد. جوابِ کوتاه: **descriptor + `__set_name__` + ثبتِ خودکارِ کلاس‌ها** — یعنی دقیقاً فصل‌های پانزدهم، ششم و شانزدهم. جوابِ کامل، همین پروژه است: یک ORM کوچک به نامِ `minidb` می‌سازیم، از صفر، فقط با کتابخانه‌ی استاندارد.

این پروژه سنگین‌ترین پروژه‌ی دوره است — و به همان نسبت، «مدرک‌سازترین». کسی که بتواند در مصاحبه بگوید «ORM کوچکِ خودم را نوشته‌ام و می‌توانم توضیح بدهم Django پشتِ پرده چه می‌کند»، در دسته‌ی دیگری از کاندیداها قرار می‌گیرد.

---

## نیاز کارفرما

این‌بار «کارفرما» بیرونِ شرکت نیست؛ سرپرستِ فنیِ تیمِ خودتان است:

> «چند تا ابزارِ داخلیِ کوچک داریم — اسکریپت‌های گزارش، ثبتِ رخداد، چند سرویسِ کمکی — که همه با SQLite کار می‌کنند. کدشان پر از SQLِ دستی و تبدیلِ دیکشنری به شیء و برعکس است؛ هر بار هم یکی جایی یک ستون را اشتباه می‌نویسد و تا production لو نمی‌رود. Django برای این ابزارها زیادی بزرگ است و نمی‌خواهم وابستگیِ بیرونی اضافه کنیم. یک لایه‌ی مدلِ سبک می‌خواهم که بچه‌های تیم مدل‌هایشان را تمیز تعریف کنند و ذخیره/خواندن هم پشتش باشد. ساده باشد، ولی نه اسباب‌بازی.»

---

## تحلیل نیازمندی‌ها

سوال‌هایی که یک برنامه‌نویسِ باتجربه قبل از کد زدن می‌پرسد:

**شما:** «مدل‌ها چه نوع فیلدهایی لازم دارند؟ همین int و str کافی است یا تاریخ و رابطه (ForeignKey) هم می‌خواهید؟»
**سرپرست:** «برای شروع int و str و bool کافی است. رابطه بینِ جدول‌ها فعلاً نه — ابزارهایمان ساده‌اند.»

**شما:** «رفتار در برابرِ داده‌ی غلط چه باشد؟ اگر کسی به فیلدِ عددی رشته داد، همان لحظه خطا بدهیم یا موقعِ ذخیره؟»
**سرپرست:** «همان لحظه. کلِ دردمان همین است که خطاها دیر لو می‌روند.»

**شما:** «ساختِ جدول‌ها با کیست؟ مهاجرت (migration) هم می‌خواهید؟»
**سرپرست:** «مهاجرت نه، بلندپروازی نکن. همین که هر مدل بتواند جدولِ خودش را بسازد کافی است.»

**شما:** «چه عملیاتی از دیتابیس لازم است؟»
**سرپرست:** «ذخیره‌ی شیء، خواندنِ همه، پیداکردن با شرطِ ساده. همین سه تا، ولی درست.»

**شما:** «تست‌پذیری چقدر مهم است؟»
**سرپرست:** «خیلی. و SQLite حافظه‌ای (in-memory) برای تست‌ها عالی می‌شود اگر لایه‌ات اجازه بدهد.»

## جمع‌بندی نیازمندی‌ها

| نیاز | جزئیات |
|------|--------|
| تعریفِ اعلانیِ مدل | `class Product(Model):` با فیلدهای `IntegerField`، `TextField`، `BooleanField` |
| اعتبارسنجیِ فوری | تایپِ غلط یا `None`ِ غیرمجاز → خطا در همان لحظه‌ی مقداردهی |
| ساختِ جدول | هر مدل، `CREATE TABLE` خودش را تولید کند |
| عملیات | `save`، `all`، `find`(شرطِ ساده‌ی برابری) |
| بدونِ وابستگی | فقط کتابخانه‌ی استاندارد (`sqlite3`) |
| تست‌پذیر | دیتابیسِ تزریق‌شدنی؛ پشتیبانی از `:memory:` |

## پیش‌نیازهای دانشی

- **[فصل ۱۵: Descriptorها](15-descriptors.md)** — قلبِ پروژه؛ فیلدها descriptorاند.
- **[فصل ۶: ارث‌بری پیشرفته](06-advanced-inheritance-mro.md)** — `__init_subclass__` برای ثبتِ خودکارِ مدل‌ها.
- **[فصل ۱۶: متاکلاس‌ها](16-metaclasses.md)** — مقایسه‌ی راهِ ما با راهِ Django.
- **[فصل ۱۱: استثناها](11-exceptions-serialization-introspection.md)** — سلسله‌مراتبِ خطاهای ORM.
- **[فصل ۵: Property و دکوراتورها](05-property-decorators-magic-methods.md)** — `classmethod` به‌عنوانِ factory و `__repr__`.

---

## پیاده‌سازی

### گام ۱: فیلدها — descriptorهایی که تایپ می‌فهمند

از فصل پانزدهم شروع می‌کنیم. هر فیلد یک descriptor است که تایپ را تحمیل می‌کند و معادلِ SQLاش را می‌داند:

```python
# minidb.py — step 1
class MiniDbError(Exception):
    """base of all minidb errors"""

class ValidationError(MiniDbError):
    """invalid value for a field"""

class Field:
    python_type = object          # overridden by subclasses
    sql_type = "TEXT"

    def __init__(self, default=None, nullable=True):
        self.default = default
        self.nullable = nullable

    def __set_name__(self, owner, name):      # ch 15: the field learns its own name
        self.name = name
        self.storage = "_" + name

    def __get__(self, instance, owner):
        if instance is None:
            return self                        # Product.price -> the Field object
        return getattr(instance, self.storage, self.default)

    def __set__(self, instance, value):
        if value is None:
            if not self.nullable:
                raise ValidationError(f"{self.name} cannot be null")
        elif not isinstance(value, self.python_type):
            raise ValidationError(
                f"{self.name} expects {self.python_type.__name__}, "
                f"got {type(value).__name__}"
            )
        setattr(instance, self.storage, value)

class IntegerField(Field):
    python_type = int
    sql_type = "INTEGER"

class TextField(Field):
    python_type = str
    sql_type = "TEXT"

class BooleanField(Field):
    python_type = bool
    sql_type = "INTEGER"          # SQLite has no BOOL; we store 0/1
```

نکته‌ی طراحی: به‌جای «برای هر تایپ یک descriptor از صفر»، یک `Field` پایه نوشتیم و زیرکلاس‌ها فقط دو attribute را عوض می‌کنند — ارث‌بریِ درست همین است: تفاوت‌ها کوچک و data-like، رفتارِ مشترک یک‌جا.

### گام ۲: کلاس Model — جایی که مدل‌ها «خودآگاه» می‌شوند

حالا مغزِ ماجرا. کلاسِ پایه‌ی `Model` باید هنگامِ تعریفِ هر زیرکلاس، دو کار کند: فیلدهایش را جمع کند و خودش را در registry ثبت کند — هر دو با `__init_subclass__` فصل ششم:

```python
# minidb.py — step 2
class Model:
    registry = {}                              # model name -> class

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        # collect fields from this class AND parents (walk the MRO, ch 6):
        cls._fields = {}
        for klass in reversed(cls.__mro__):
            for name, attr in vars(klass).items():
                if isinstance(attr, Field):
                    cls._fields[name] = attr
        cls._table = cls.__name__.lower()
        Model.registry[cls._table] = cls       # ch 12: the Registry pattern

    def __init__(self, **kwargs):
        for name, field in self._fields.items():
            value = kwargs.pop(name, field.default)
            setattr(self, name, value)         # goes through Field.__set__ -> validated
        if kwargs:                             # anything left is a typo
            unknown = ", ".join(kwargs)
            raise ValidationError(f"{type(self).__name__} has no field(s): {unknown}")

    def __repr__(self):
        values = ", ".join(f"{n}={getattr(self, n)!r}" for n in self._fields)
        return f"{type(self).__name__}({values})"

    def __eq__(self, other):
        if not isinstance(other, type(self)):
            return NotImplemented
        return all(getattr(self, n) == getattr(other, n) for n in self._fields)
```

همین حالا تستش کنیم — هنوز خبری از دیتابیس نیست، اما مدل‌ها زنده‌اند:

```python
class Product(Model):
    name = TextField(nullable=False)
    price = IntegerField(default=0)
    active = BooleanField(default=True)

p = Product(name="keyboard", price=500_000)
print(p)                        # Product(name='keyboard', price=500000, active=True)
print(Model.registry)           # {'product': <class 'Product'>}
print(Product._fields.keys())   # dict_keys(['name', 'price', 'active'])

# validation is instant — exactly what the team lead asked for:
# Product(name="mouse", price="cheap")   # ValidationError: price expects int, got str
# Product(price=10)                      # ValidationError: name cannot be null
# Product(name="x", pricee=5)            # ValidationError: Product has no field(s): pricee
```

به خطِ آخر دقت کنید: غلطِ تایپیِ `pricee` — همان دردی که سرپرست گفت «تا production لو نمی‌رود» — حالا در لحظه‌ی ساخت می‌میرد.

### گام ۳: تولید SQL — مدل‌ها جدولِ خودشان را می‌سازند

هر مدل همه‌چیزِ لازم را می‌داند: نامِ جدول و فیلدها با تایپ‌هایشان. پس تولیدِ SQL فقط یک متدِ کلاسی است:

```python
# add inside Model — step 3
    @classmethod
    def create_table_sql(cls):
        columns = ["id INTEGER PRIMARY KEY AUTOINCREMENT"]
        for name, field in cls._fields.items():
            null = "" if field.nullable else " NOT NULL"
            columns.append(f"{name} {field.sql_type}{null}")
        return f"CREATE TABLE IF NOT EXISTS {cls._table} ({', '.join(columns)})"

    def insert_sql(self):
        names = list(self._fields)
        placeholders = ", ".join("?" for _ in names)
        values = tuple(getattr(self, n) for n in names)
        sql = f"INSERT INTO {self._table} ({', '.join(names)}) VALUES ({placeholders})"
        return sql, values
```

```python
print(Product.create_table_sql())
# CREATE TABLE IF NOT EXISTS product
#   (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL,
#    price INTEGER, active INTEGER)

print(Product(name="keyboard").insert_sql())
# ('INSERT INTO product (name, price, active) VALUES (?, ?, ?)',
#  ('keyboard', 0, True))
```

دو تصمیمِ امنیتی/طراحی که ارزشِ گفتن دارند: مقدارها با **placeholder** (`?`) جدا از SQL می‌روند — نه با چسباندنِ رشته؛ این سدِ اصلی در برابرِ SQL Injection است. و SQLسازی را از اجرایش جدا کردیم: این متدها فقط **رشته می‌سازند** و به هیچ دیتابیسی دست نمی‌زنند — یعنی بدونِ هیچ اتصالی تست می‌شوند (فصل سیزدهم: جداسازیِ منطقِ خالص از I/O).

### گام ۴: لایه‌ی Database — یک Facade با context manager

حالا اجراکننده. یک کلاسِ `Database` که اتصال را مدیریت می‌کند (context manager، فصل ۵)، جدول‌ها را از روی registry می‌سازد و عملیاتِ سه‌گانه را ارائه می‌دهد:

```python
# minidb.py — step 4
import sqlite3

class Database:
    def __init__(self, path=":memory:"):
        self.path = path
        self.conn = None

    def __enter__(self):
        self.conn = sqlite3.connect(self.path)
        for model in Model.registry.values():          # the registry pays off
            self.conn.execute(model.create_table_sql())
        return self

    def __exit__(self, exc_type, exc, tb):
        if exc_type is None:
            self.conn.commit()                         # success -> commit
        else:
            self.conn.rollback()                       # error -> roll back
        self.conn.close()
        return False                                   # never swallow exceptions

    def save(self, obj):
        sql, values = obj.insert_sql()
        cursor = self.conn.execute(sql, values)
        obj.id = cursor.lastrowid                      # the object learns its id
        return obj

    def all(self, model):
        rows = self.conn.execute(f"SELECT * FROM {model._table}").fetchall()
        return [model.from_row(row) for row in rows]

    def find(self, model, **conditions):
        where = " AND ".join(f"{name} = ?" for name in conditions)
        sql = f"SELECT * FROM {model._table} WHERE {where}"
        rows = self.conn.execute(sql, tuple(conditions.values())).fetchall()
        return [model.from_row(row) for row in rows]
```

و حلقه‌ی گمشده — تبدیلِ سطرِ دیتابیس به شیء — یک factoryِ کلاسی (فصل ۵) در `Model`:

```python
# add inside Model — step 4
    @classmethod
    def from_row(cls, row):
        # row = (id, field1, field2, ...) in the order of _fields
        obj = cls.__new__(cls)                 # skip __init__ (and re-validation)
        obj.id = row[0]
        for (name, field), value in zip(cls._fields.items(), row[1:]):
            if field.python_type is bool and value is not None:
                value = bool(value)            # SQLite gave us 0/1
            setattr(obj, field.storage, value) # write directly to storage
        return obj
```

چرا `cls.__new__(cls)` به‌جای سازنده؟ (فصل ۲ بالاخره میوه داد!) داده‌ای که از دیتابیس می‌آید قبلاً اعتبارسنجی شده؛ ردکردنش دوباره از `__set__`، هم هزینه دارد و هم اگر قوانین سخت‌گیرتر شده باشند، داده‌ی قدیمیِ سالم را می‌شکند. این تمایزِ «مسیرِ کاربر» و «مسیرِ بازیابی» تصمیمی است که ORMهای واقعی هم می‌گیرند.

### گام ۵: همه‌چیز کنار هم

```python
class User(Model):
    username = TextField(nullable=False)
    age = IntegerField(default=0)

with Database(":memory:") as db:
    db.save(Product(name="keyboard", price=500_000))
    db.save(Product(name="mouse", price=300_000, active=False))
    db.save(User(username="ali", age=30))

    print(db.all(Product))
    # [Product(name='keyboard', price=500000, active=True),
    #  Product(name='mouse', price=300000, active=False)]

    print(db.find(Product, active=True))
    # [Product(name='keyboard', price=500000, active=True)]

    print(db.find(User, username="ali"))
    # [User(username='ali', age=30)]
```

قدم به عقب بگذارید و نگاه کنید: تیمِ شما حالا مدل را در سه خط تعریف می‌کند، اعتبارسنجی فوری دارد، جدول خودکار ساخته می‌شود و ذخیره/خواندن پشتِ یک Facade تمیز است. کلِ `minidb` حدودِ صد خط است — و **تمامِ** آن از مفاهیمِ همین دوره ساخته شده.

### گام ۶: و متاکلاس کجاست؟ مقایسه با Django

قولِ فصل شانزدهم را ادا کنیم. Django همین کار را با **متاکلاس** می‌کند (`ModelBase`). ما `__init_subclass__` را انتخاب کردیم. تفاوت و دلیل:

```python
# the metaclass road (what Django does, simplified):
class ModelMeta(type):
    def __new__(mcls, name, bases, namespace):
        cls = super().__new__(mcls, name, bases, namespace)
        cls._fields = {k: v for k, v in namespace.items() if isinstance(v, Field)}
        return cls

class MetaModel(metaclass=ModelMeta):
    pass
```

برای نیازِ ما هر دو جواب می‌دهند، و طبقِ قاعده‌ی فصل ششم («اگر فقط هنگامِ تعریفِ زیرکلاس کاری داری، `__init_subclass__` کافی است») راهِ ساده‌تر را رفتیم. Django متاکلاس لازم دارد چون کارهای سنگین‌تری می‌کند: دست‌کاریِ خودِ کلاس قبل از ساخت، ترتیبِ دقیقِ فیلدها با `__prepare__` (در پایتون‌های قدیمی)، ساختنِ `Manager`ها و `DoesNotExist`های اختصاصی برای هر مدل. درسِ نهایی: **قدرتمندترین ابزار را انتخاب نکنید؛ کافی‌ترین را انتخاب کنید** — و حالا در موقعیتی هستید که این انتخاب را آگاهانه می‌کنید، نه از سرِ ناچاری.

### گام ۷: تست — قولی که به سرپرست دادیم

`:memory:` یعنی هر تست، دیتابیسِ تازه و ایزوله‌ی خودش (فصل سیزدهم — استقلالِ تست‌ها):

```python
def test_save_assigns_id():
    with Database(":memory:") as db:
        p = db.save(Product(name="pad", price=90_000))
        assert p.id == 1

def test_validation_is_instant():
    import pytest
    with pytest.raises(ValidationError):
        Product(name="x", price="cheap")

def test_find_filters_correctly():
    with Database(":memory:") as db:
        db.save(Product(name="a", active=True))
        db.save(Product(name="b", active=False))
        assert len(db.find(Product, active=True)) == 1
```

---

## جمع‌بندی پروژه

| مفهوم | کجا استفاده شد |
|-------|----------------|
| Descriptor + `__set_name__` (فصل ۱۵) | فیلدها: اعتبارسنجیِ فوری، نامِ خودکار |
| `__init_subclass__` و MRO (فصل ۶) | جمع‌کردنِ فیلدها و ثبتِ خودکارِ مدل‌ها |
| الگوی Registry (فصل ۱۲) | `Model.registry` → ساختِ خودکارِ همه‌ی جدول‌ها |
| `classmethod` factory و `__new__` (فصل‌های ۵ و ۲) | `from_row` — مسیرِ بازیابیِ بدونِ اعتبارسنجیِ مجدد |
| Context Manager (فصل ۵) | `Database`: commit/rollback خودکار |
| استثناهای سفارشی (فصل ۱۱) | `MiniDbError` → `ValidationError` |
| جداسازیِ منطق از I/O (فصل‌های ۷ و ۱۳) | SQLسازی جدا از اجرا؛ تست با `:memory:` |
| متاکلاس (فصل ۱۶) | مقایسه‌ی آگاهانه: چرا لازم نشد |

**درسِ اصلی:** فریمورک‌ها جادو نیستند؛ ترکیبِ منظمِ همین سازوکارهای زبان‌اند. شما امروز با صد خط، هسته‌ی چیزی را ساختید که میلیون‌ها برنامه‌نویس هر روز استفاده می‌کنند — و مهم‌تر: هر تصمیمِ طراحی‌اش را می‌توانید **دفاع** کنید.

**تمرینِ پیشنهادی:** سه گسترش به انتخابِ خودتان: (۱) متدِ `update` با ردیابیِ فیلدهای تغییرکرده (descriptorِ `_dirty` از فصل ۱۵)؛ (۲) `FloatField` و `DateField` (ذخیره به‌صورت ISO رشته)؛ (۳) `delete` و یک متدِ `first` که به‌جای لیست، یک شیء یا `None` برگرداند. هر سه با همان الگوهای موجود قابلِ‌ساخت‌اند.
