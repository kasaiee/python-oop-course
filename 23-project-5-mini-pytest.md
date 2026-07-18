# فصل بیست‌وسوم — پروژه پنجم: فریمورک تست خودت را بساز

> در فصل سیزدهم با pytest تست نوشتید و شاید جاهایی به نظرتان «جادویی» آمد: از کجا تست‌ها را *پیدا* می‌کند؟ fixture چطور فقط با هم‌نام‌بودن پارامتر *تزریق* می‌شود؟ در این پروژه جادو را باطل می‌کنیم: یک فریمورک تست کوچک به نام `minitest` می‌سازیم — با کشف خودکار، fixture و parametrize. حدوداً صد خط، فقط کتابخانه‌ی استاندارد.

---

## مقدمه

بعد از ORM، این دومین «ابزارسازی» شماست و از قصد انتخابش کرده‌ایم: pytest پرکاربردترین ابزار روزمره‌ی یک پایتون‌کار است، و ساختن نسخه‌ی کوچکش سه مهارت عمیق را یک‌جا تمرین می‌دهد — **Introspection** (فصل ۱۱: برنامه‌ای که برنامه‌ی دیگر را می‌خوانَد)، **دکوراتورها و context managerها** (فصل ۵) و **طراحی API** (چطور ابزاری بسازیم که استفاده‌اش لذت‌بخش باشد).

و یک انگیزه‌ی صادقانه‌ی دیگر: در مصاحبه، بین «با pytest کار کرده‌ام» و «می‌دانم pytest چطور تست‌ها را کشف و fixtureها را تزریق می‌کند، چون خودم نمونه‌اش را ساخته‌ام» یک دنیا فاصله است.

---

## نیاز کارفرما

مسئول کارآموزی یک مجموعه‌ی آموزشی سراغتان آمده:

> «برای دوره‌ی کارآموزها یک محیط تمرین آفلاین داریم — ماشین‌هایی بدون اینترنت و بدون اجازه‌ی نصب پکیج. می‌خواهم بچه‌ها برای تمرین‌هایشان تست بنویسند و یک گزارش خوانا بگیرند: چی پاس شد، چی نشد، چرا. چیزی شبیه همان pytest که در صنعت هست، ولی تک‌فایل که فقط کپی‌اش کنیم روی سیستم‌ها. لازم نیست غول باشد؛ باید ساده، قابل‌فهم و قابل‌اعتماد باشد — خود بچه‌ها باید بتوانند سورسش را بخوانند.»

---

## تحلیل نیازمندی‌ها

**شما:** «تست‌ها را چطور تعریف کنند؟ کلاس‌محور مثل unittest یا تابع‌محور مثل pytest؟»
**مسئول:** «تابع‌محور — همان سبکی که بعداً در بازار کار می‌بینند: `def test_something(): assert ...`»

**شما:** «وقتی تستی شکست، چه اطلاعاتی نمایش بدهیم؟»
**مسئول:** «اسم تست، دلیل شکست، و خطی که خراب شد. لازم نیست مثل pytest مقادیر را کالبدشکافی کنی.»

**شما:** «تست خطاها چطور؟ بچه‌ها باید بتوانند بگویند "این کد باید ValueError بدهد".»
**مسئول:** «بله، حتماً. همان `raises` که در pytest هست.»

**شما:** «fixture و parametrize لازم دارید یا زیاده‌روی است؟»
**مسئول:** «نسخه‌ی ساده‌شان را بله — این دو مفهوم دقیقاً چیزی است که می‌خواهم یاد بگیرند.»

**شما:** «خروجی برنامه چه باشد؟ فقط چاپ؟»
**مسئول:** «چاپ گزارش + کد خروج درست (صفر/یک) که بتوانم در اسکریپت تصحیح خودکار استفاده‌اش کنم.»

## جمع‌بندی نیازمندی‌ها

| نیاز | جزئیات |
|------|--------|
| کشف خودکار | توابع `test_*` از ماژول داده‌شده |
| گزارش | پاس/شکست به‌تفکیک + خلاصه + کد خروج (۰ موفق، ۱ ناموفق) |
| جزئیات شکست | نام تست، نوع خطا، پیام و خط شکست |
| `raises` | context manager برای تست استثناها |
| fixture | تزریق بر اساس نام پارامتر؛ هر تست نمونه‌ی تازه |
| parametrize | اجرای یک تست با چند مجموعه‌داده |
| بدون وابستگی | تک‌فایل، فقط کتابخانه‌ی استاندارد |

## پیش‌نیازهای دانشی

- **[فصل ۱۱: Introspection](11-exceptions-serialization-introspection.md)** — قلب کشف خودکار: `vars`، `callable`، `inspect.signature`.
- **[فصل ۵: دکوراتورها و متدهای جادویی](05-property-decorators-magic-methods.md)** — `@fixture`، `@parametrize`، و `__enter__`/`__exit__` برای `raises`.
- **[فصل ۱۳: تست‌نویسی](13-testing-oop.md)** — می‌دانید قرار است چه چیزی بسازیم، چون استفاده‌اش کرده‌اید.
- **[فصل ۱۲: الگوهای طراحی](12-design-patterns.md)** — Registry برای fixtureها.

---

## پیاده‌سازی

### گام ۱: هسته — پیداکردن و اجرای تست‌ها

اولین قابلیت: به یک ماژول نگاه کن، هر تابع `test_*` را پیدا کن، اجرا کن، نتیجه را جمع کن. این همان Introspection فصل ۱۱ است — کدی که کد دیگر را می‌خوانَد:

```python
# minitest.py — step 1
import traceback

class TestResult:
    """The outcome of one test: name + status + failure details."""
    def __init__(self, name, passed, error=None):
        self.name = name
        self.passed = passed
        self.error = error


def discover(module):
    """Find all test functions in a module, in definition order."""
    tests = []
    for name, obj in vars(module).items():     # ch 11: a module's namespace
        if name.startswith("test_") and callable(obj):
            tests.append(obj)
    return tests


def run_one(test_func):
    try:
        test_func()
        return TestResult(test_func.__name__, passed=True)
    except AssertionError as e:
        line = traceback.extract_tb(e.__traceback__)[-1].line   # the failing line
        detail = f"assert failed: {line}" + (f"  ({e})" if str(e) else "")
        return TestResult(test_func.__name__, passed=False, error=detail)
    except Exception as e:
        return TestResult(test_func.__name__, passed=False,
                          error=f"{type(e).__name__}: {e}")
```

سه تصمیم طراحی در همین گام: **شکست دو نوع است** — `AssertionError` یعنی «ادعای تست غلط بود» و بقیه‌ی استثناها یعنی «تست خودش خراب شد»؛ گزارششان را جدا کردیم چون معنایشان فرق دارد. با ماژول `traceback` **خط دقیق assert شکست‌خورده** را بیرون می‌کشیم — همان جادوی کوچکی که وقت دیباگ طلاست. و `run_one` هرگز استثنا را بیرون نمی‌دهد؛ **شکست یک تست نباید اجرای بقیه را متوقف کند**.

### گام ۲: گزارش‌گیری و کد خروج

```python
# minitest.py — step 2
def run(module):
    """Run all tests of a module and print a report. Returns the exit code."""
    results = [run_one(t) for t in discover(module)]

    for r in results:
        mark = "PASS" if r.passed else "FAIL"
        print(f"[{mark}] {r.name}")
        if r.error:
            print(f"       {r.error}")

    passed = sum(1 for r in results if r.passed)
    failed = len(results) - passed
    print(f"\n{passed} passed, {failed} failed")
    return 0 if failed == 0 else 1
```

امتحانش کنیم — یک ماژول تست واقعی، بدون ذره‌ای ثبت‌نام دستی:

```python
# test_cart.py — the intern's code
class Cart:
    def __init__(self):
        self.items = []
    def add(self, name, price):
        if price < 0:
            raise ValueError("price cannot be negative")
        self.items.append((name, price))
    def total(self):
        return sum(price for _, price in self.items)

def test_empty_cart():
    assert Cart().total() == 0

def test_total():
    cart = Cart()
    cart.add("keyboard", 500)
    cart.add("mouse", 300)
    assert cart.total() == 800

def test_this_one_fails():                    # intentionally wrong, to see the report
    cart = Cart()
    cart.add("pad", 100)
    assert cart.total() == 999

import minitest
raise SystemExit(minitest.run(__import__(__name__)))
```

```
[PASS] test_empty_cart
[PASS] test_total
[FAIL] test_this_one_fails
       assert failed: assert cart.total() == 999

2 passed, 1 failed
```

فریمورک زنده است. کارآموز فقط تابع می‌نویسد؛ کشف و اجرا و گزارش با ماست.

### گام ۳: raises — تست استثناها

حالا قول `raises`. در فصل سیزدهم استفاده‌اش کردید؛ حالا با پروتکل context manager فصل ۵ می‌سازیمش. منطقش ظریف است: موفقیت این بلاک، **رخ‌دادن** استثناست:

```python
# minitest.py — step 3
class raises:
    def __init__(self, expected_exception):
        self.expected = expected_exception

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, tb):
        if exc_type is None:                   # nothing was raised -> the test lied
            raise AssertionError(
                f"expected {self.expected.__name__}, but nothing was raised")
        if issubclass(exc_type, self.expected):
            self.value = exc_value             # keep it for inspection
            return True                        # swallow: this failure was the goal
        return False                           # a DIFFERENT error -> let it escape
```

سه شاخه‌ی `__exit__` را با دقت بخوانید؛ کل هوش کلاس همین‌جاست. «استثنایی نیامد» یعنی شکست تست — پس خودمان `AssertionError` پرت می‌کنیم. «استثنای موردانتظار آمد» یعنی موفقیت — با `return True` قورتش می‌دهیم (فصل ۵: مقدار برگشتی `__exit__` یعنی «هضم شد؟»). و «استثنای دیگری آمد» یعنی باگ واقعی — با `return False` می‌گذاریم بالا برود و تست FAIL شود. مقایسه با `issubclass` هم یعنی زیرکلاس‌ها را می‌پذیریم — هماهنگ با سلسله‌مراتب استثناهای فصل ۱۱.

```python
def test_negative_price_is_rejected():
    cart = Cart()
    with minitest.raises(ValueError):
        cart.add("hacked", -1)
```

### گام ۴: fixture — تزریق با نام پارامتر

جادوی محبوب pytest. اول ثبت (Registry فصل ۱۲)، بعد تزریق با Introspection:

```python
# minitest.py — step 4
import inspect

_fixtures = {}                                 # name -> factory function

def fixture(func):
    """Decorator: register a factory under its own name."""
    _fixtures[func.__name__] = func
    return func

def _resolve_arguments(test_func):
    """For each parameter of the test, build the same-named fixture."""
    kwargs = {}
    for param in inspect.signature(test_func).parameters:   # ch 11
        if param in _fixtures:
            kwargs[param] = _fixtures[param]()  # a FRESH instance for every test
    return kwargs
```

و `run_one` فقط یک خط تغییر می‌کند:

```python
def run_one(test_func, prebuilt_kwargs=None):
    kwargs = prebuilt_kwargs if prebuilt_kwargs is not None else _resolve_arguments(test_func)
    try:
        test_func(**kwargs)
        return TestResult(test_func.__name__, passed=True)
    except AssertionError as e:
        line = traceback.extract_tb(e.__traceback__)[-1].line
        detail = f"assert failed: {line}" + (f"  ({e})" if str(e) else "")
        return TestResult(test_func.__name__, passed=False, error=detail)
    except Exception as e:
        return TestResult(test_func.__name__, passed=False,
                          error=f"{type(e).__name__}: {e}")
```

استفاده‌اش عین pytest است:

```python
@minitest.fixture
def loaded_cart():
    cart = Cart()
    cart.add("keyboard", 500)
    cart.add("mouse", 300)
    return cart

def test_loaded_total(loaded_cart):            # injected by parameter name
    assert loaded_cart.total() == 800

def test_add_to_loaded(loaded_cart):           # a fresh cart — not the one above!
    loaded_cart.add("pad", 100)
    assert loaded_cart.total() == 900
```

راز از بین رفت: pytest نه ذهن می‌خوانَد نه جادو می‌کند؛ `inspect.signature` امضای تابع شما را می‌خوانَد و نام‌ها را در registry می‌جوید. به `_fixtures[param]()` هم دقت کنید — پرانتز یعنی **هر تست، نمونه‌ی تازه**؛ استقلال تست‌ها (فصل ۱۳) در همین یک جفت پرانتز زندگی می‌کند.

### گام ۵: parametrize — یک تست، چند داده

دکوراتور `parametrize` چیزی اجرا نمی‌کند؛ فقط داده‌ها را به خود تابع **سنجاق** می‌کند (توابع شیءاند — فصل صفر تا همین‌جا آمد!):

```python
# minitest.py — step 5
def parametrize(param_names, cases):
    def decorator(func):
        func._parametrize = (param_names, cases)
        return func                            # unchanged, just tagged
    return decorator
```

و `run` تابع‌های سنجاق‌دار را چندبار اجرا می‌کند — برای هر مجموعه‌داده یک نتیجه‌ی مستقل با نام گویا:

```python
def _expand(test_func):
    """One test -> one or many (name, kwargs) runs."""
    if not hasattr(test_func, "_parametrize"):
        return [(test_func.__name__, None)]
    names, cases = test_func._parametrize
    runs = []
    for case in cases:
        case = case if isinstance(case, tuple) else (case,)
        kwargs = dict(zip(names.split(","), case))
        label = f"{test_func.__name__}[{'-'.join(map(str, case))}]"
        runs.append((label, kwargs))
    return runs

def run(module):
    results = []
    for test in discover(module):
        for label, kwargs in _expand(test):
            result = run_one(test, prebuilt_kwargs=kwargs)
            result.name = label
            results.append(result)

    for r in results:
        mark = "PASS" if r.passed else "FAIL"
        print(f"[{mark}] {r.name}")
        if r.error:
            print(f"       {r.error}")

    passed = sum(1 for r in results if r.passed)
    failed = len(results) - passed
    print(f"\n{passed} passed, {failed} failed")
    return 0 if failed == 0 else 1
```

### گام ۶: همه‌چیز کنار هم — جلسه‌ی تحویل

فایل تست نهایی یک کارآموز، با هر چهار قابلیت:

```python
# test_cart_full.py
import minitest
from cart import Cart

@minitest.fixture
def cart():
    return Cart()

def test_empty(cart):
    assert cart.total() == 0

def test_negative_price(cart):
    with minitest.raises(ValueError):
        cart.add("bad", -5)

@minitest.parametrize("price,expected", [
    (100, 100),
    (0, 0),
    (250, 250),
])
def test_single_item_total(price, expected):
    c = Cart()
    c.add("item", price)
    assert c.total() == expected

def test_shows_failure_nicely(cart):
    cart.add("pen", 10)
    # intentional bug, to see the failure report:
    assert cart.total() == 11

raise SystemExit(minitest.run(__import__(__name__)))
```

```
[PASS] test_empty
[PASS] test_negative_price
[PASS] test_single_item_total[100-100]
[PASS] test_single_item_total[0-0]
[PASS] test_single_item_total[250-250]
[FAIL] test_shows_failure_nicely
       assert failed: assert cart.total() == 11

5 passed, 1 failed
```

مسئول کارآموزی چیزی را می‌گیرد که خواسته بود: تک‌فایل، خوانا، آفلاین، با کد خروج قابل‌اسکریپت. و کارآموزها چیزی مهم‌تر می‌گیرند: ابزاری که وقتی سورسش را باز کنند، **تا تهش را می‌فهمند.**

---

## جمع‌بندی پروژه

| مفهوم | کجا استفاده شد |
|-------|----------------|
| Introspection (فصل ۱۱) | `vars(module)` برای کشف؛ `inspect.signature` برای تزریق؛ `traceback` برای خط شکست |
| Context Manager (فصل ۵) | `raises` — سه شاخه‌ی `__exit__` و معنای `return True` |
| دکوراتورها (فصل ۵) | `@fixture` (ثبت) و `@parametrize` (سنجاق‌کردن داده به تابع) |
| Registry (فصل ۱۲) | دیکشنری `_fixtures` |
| سلسله‌مراتب استثنا (فصل ۱۱) | تفکیک AssertionError از خطاهای دیگر؛ `issubclass` در raises |
| استقلال تست (فصل ۱۳) | fixtureی تازه برای هر تست؛ ادامه‌ی اجرا بعد از هر شکست |

**درس اصلی:** pytest هشت سال است هر روز زیر دستتان بود و «جادو» به نظر می‌رسید؛ امروز دیدید که سه‌چهار مفهوم خوب‌فهمیده — introspection، دکوراتور، context manager، registry — کافی بود تا خودتان بسازیدش. دفعه‌ی بعد که ابزاری «جادویی» دیدید، سوال درست این نیست که «چطور ممکن است؟»؛ این است که «کدام پروتکل‌ها زیرش است؟»

**تمرین پیشنهادی:** سه گسترش: (۱) fixtureهای جنریتوری با `yield` برای پاک‌سازی بعد از تست (راهنمایی: `next()` قبل از تست، `next()` بعدش)؛ (۲) دکوراتور `@skip(reason)` با نتیجه‌ی سوم SKIP در گزارش؛ (۳) پشتیبانی از fixture در تست‌های parametrize (ترکیب `_expand` و `_resolve_arguments`).
