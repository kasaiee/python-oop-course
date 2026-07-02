# فصل شانزدهم — پروژه سوم: سیستم رزرو آنلاین

> مقاله‌ی **آموزش با مثال** و پروژه‌ی پایانیِ دوره. این پروژه همه‌چیز را کنار هم می‌آورد: معماریِ لایه‌ای، مدیریتِ حالت، جلوگیری از رزروِ همزمان، و طراحیِ تست‌پذیر.

---

## مقدمه

به آخرین پروژه‌ی دوره رسیدیم. در دو پروژه‌ی قبل، یک سیستمِ داده‌محور و یک سیستمِ انعطاف‌محور ساختیم. حالا سراغِ چالشی می‌رویم که همه‌ی ابعاد را می‌طلبد: یک **سیستم رزرو آنلاین** (برای سالنِ سینما).

این پروژه سخت‌ترینِ سه‌تاست، چون یک مسئله‌ی ظریف دارد که در دو پروژه‌ی قبل نبود: **دو نفر نباید بتوانند یک صندلی را همزمان رزرو کنند.** این ما را به مدیریتِ دقیقِ حالت و طراحیِ معماریِ لایه‌ای می‌کشاند.

---

## نیاز کارفرما

کارفرما صاحبِ یک سینماست:

> «می‌خواهم مردم بتوانند آنلاین صندلیِ سینما رزرو کنند. هر سانس یک سالن دارد با یک‌سری صندلی. مشتری صندلی‌اش را انتخاب می‌کند و رزرو می‌کند. مهم‌ترین چیز این است که یک صندلی دو بار فروخته نشود — این فاجعه است! رزرو باید یک مهلتِ پرداخت داشته باشد؛ اگر تا ۱۰ دقیقه پرداخت نشد، صندلی آزاد شود. و می‌خواهم بتوانم گزارش بگیرم که هر سانس چند صندلی فروخته.»

دو نیازِ کلیدی: **جلوگیری از فروشِ دوباره** («فاجعه!») و **انقضای رزروِ پرداخت‌نشده**. این‌ها مسائلِ مدیریتِ حالت‌اند.

---

## تحلیل نیازمندی‌ها

**درباره‌ی ساختار:**
- «هر سالن چند صندلی دارد و چطور شماره‌گذاری می‌شوند؟» → «ردیف و شماره، مثلِ A1، A2، B1.»
- «آیا انواعِ صندلی دارید (معمولی، VIP)؟» → «بله، با قیمت‌های متفاوت.»

**درباره‌ی رزرو:**
- «وضعیت‌های یک صندلی چیست؟» → «آزاد، رزروِموقت (در انتظارِ پرداخت)، فروخته‌شده.»
- «اگر دو نفر همزمان یک صندلی را بخواهند؟» → «فقط اولی موفق شود، دومی خطا بگیرد.»
- «مهلتِ پرداخت؟» → «۱۰ دقیقه، بعد آزاد شود.»

**درباره‌ی گزارش:**
- «چه گزارشی می‌خواهید؟» → «تعدادِ فروخته‌شده و درآمدِ هر سانس.»

---

## جمع‌بندی نیازمندی‌ها

**معماریِ لایه‌ای (فصل ۷):**

```
┌─────────────────────────────────────┐
│  BookingService (لایه‌ی منطق)         │  رزرو، تأیید، انقضا
├─────────────────────────────────────┤
│  Showtime, Seat, Reservation (مدل)   │  حالت و قوانین
├─────────────────────────────────────┤
│  ReservationRepository (لایه‌ی داده)  │  ذخیره و بازیابی
└─────────────────────────────────────┘
```

**ماشینِ حالتِ صندلی:**

```
  AVAILABLE ──رزرو──► HELD ──پرداخت──► SOLD
      ▲                 │
      └──انقضا/لغو───────┘
```

**قوانینِ کسب‌وکار:**

| قانون | مقدار |
|-------|-------|
| مهلتِ پرداخت | ۱۰ دقیقه |
| رزروِ صندلیِ غیرِآزاد | ممنوع (خطا) |
| انواعِ صندلی | معمولی، VIP (قیمتِ متفاوت) |

---

## پیش‌نیازهای دانشی

- **[فصل ۳: مفاهیم بنیادین](03-fundamentals-class-object-self-constructor.md)** — حالت (State) و رفتار، sازنده.
- **[فصل ۴: اصول چهارگانه](04-four-pillars.md)** — کپسول‌سازی (محافظت از حالتِ صندلی)، استثنا.
- **[فصل ۷: Composition و معماری](07-composition-architecture.md)** — معماریِ لایه‌ای، Repository، تست‌پذیری.
- **[فصل ۸: SOLID](08-solid.md)** — SRP و DIP در لایه‌بندی.
- **[فصل ۱۱: مدل‌های داده مدرن](11-modern-data-models.md)** — `Enum` برای وضعیت، `dataclass`.
- **[فصل ۱۵: Performance](15-performance.md)** — thread-safety و شرایطِ مسابقه.

---

## پیاده‌سازی

### گام ۱: صندلی و ماشینِ حالت

قلبِ پروژه، صندلی است. وضعیتش را با `Enum` (فصل ۱۱) و حالتش را با **کپسول‌سازی** (فصل ۴) محافظت می‌کنیم:

```python
from enum import Enum

class SeatStatus(Enum):
    AVAILABLE = "available"
    HELD = "held"
    SOLD = "sold"

class SeatType(Enum):
    NORMAL = "normal"
    VIP = "vip"

# استثناها (فصل ۴، ۱۲)
class BookingError(Exception): pass
class SeatNotAvailableError(BookingError): pass

class Seat:
    PRICES = {SeatType.NORMAL: 500_000, SeatType.VIP: 1_200_000}

    def __init__(self, row, number, seat_type=SeatType.NORMAL):
        self.row = row
        self.number = number
        self.seat_type = seat_type
        self._status = SeatStatus.AVAILABLE     # کپسوله‌شده — مستقیم دست نزنید

    @property
    def label(self):
        return f"{self.row}{self.number}"

    @property
    def status(self):                            # فقط‌خواندنی از بیرون (فصل ۵)
        return self._status

    @property
    def price(self):
        return self.PRICES[self.seat_type]

    def hold(self):
        if self._status != SeatStatus.AVAILABLE:
            raise SeatNotAvailableError(f"صندلی {self.label} آزاد نیست")
        self._status = SeatStatus.HELD

    def sell(self):
        if self._status != SeatStatus.HELD:
            raise BookingError(f"صندلی {self.label} ابتدا باید رزرو شود")
        self._status = SeatStatus.SOLD

    def release(self):
        self._status = SeatStatus.AVAILABLE
```

نکته‌ی طراحی: `_status` خصوصی است و فقط از راهِ متدهای `hold`/`sell`/`release` تغییر می‌کند. این **کپسول‌سازی** تضمین می‌کند صندلی هرگز به حالتِ نامعتبر نرود — مثلاً نمی‌توان صندلیِ آزاد را مستقیم «فروخته» کرد. `status` یک property فقط‌خواندنی است. این همان اصلِ «کنترلِ دسترسی به حالت» فصل ۴ است.

### گام ۲: سانس (Showtime)

```python
class Showtime:
    def __init__(self, movie, start_time, hall_name):
        self.movie = movie
        self.start_time = start_time
        self.hall_name = hall_name
        self.seats = {}          # label → Seat

    def add_seat(self, seat):
        self.seats[seat.label] = seat

    def get_seat(self, label):
        seat = self.seats.get(label)
        if seat is None:
            raise BookingError(f"صندلیِ {label} در این سانس وجود ندارد")
        return seat

    def available_seats(self):
        return [s for s in self.seats.values()
                if s.status == SeatStatus.AVAILABLE]

    def build_hall(self, rows, seats_per_row, vip_rows=()):
        for row in rows:
            for n in range(1, seats_per_row + 1):
                stype = SeatType.VIP if row in vip_rows else SeatType.NORMAL
                self.add_seat(Seat(row, n, stype))
```

`build_hall` یک متدِ کمکی است که کلِ سالن را می‌سازد. مثلاً ردیف‌های A و B معمولی، ردیفِ C در VIP.

### گام ۳: رزرو با مهلتِ انقضا

`Reservation` را با `dataclass` (فصل ۱۱) و منطقِ انقضا می‌سازیم:

```python
from dataclasses import dataclass, field
from datetime import datetime, timedelta

@dataclass
class Reservation:
    reservation_id: str
    showtime: Showtime
    seats: list
    created_at: datetime = field(default_factory=datetime.now)
    hold_minutes: int = 10
    is_paid: bool = False

    @property
    def expires_at(self):
        return self.created_at + timedelta(minutes=self.hold_minutes)

    def is_expired(self, now=None):
        now = now or datetime.now()
        return not self.is_paid and now > self.expires_at

    def total_price(self):
        return sum(seat.price for seat in self.seats)
```

`is_expired` بررسی می‌کند که آیا رزروِ پرداخت‌نشده از مهلتش گذشته یا نه. `expires_at` یک property محاسبه‌شونده است.

### گام ۴: لایه‌ی داده — Repository

طبقِ معماریِ لایه‌ای (فصل ۷)، ذخیره‌سازی را از منطق جدا می‌کنیم:

```python
class ReservationRepository:
    def __init__(self):
        self._reservations = {}      # id → Reservation

    def save(self, reservation):
        self._reservations[reservation.reservation_id] = reservation

    def get(self, reservation_id):
        return self._reservations.get(reservation_id)

    def all(self):
        return list(self._reservations.values())

    def delete(self, reservation_id):
        self._reservations.pop(reservation_id, None)
```

این لایه فقط ذخیره و بازیابی می‌کند — هیچ منطقِ کسب‌وکاری ندارد (SRP، فصل ۸). می‌توان بعداً نسخه‌ی دیتابیسی‌اش را جایگزین کرد بدونِ تغییرِ منطق.

### گام ۵: لایه‌ی منطق — BookingService

اینجا قلبِ هماهنگی است. `BookingService` به Repository وابسته است اما از طریقِ تزریق (**DIP**، فصل ۸):

```python
import threading
import uuid

class BookingService:
    def __init__(self, repository: ReservationRepository):
        self.repository = repository        # تزریقِ وابستگی
        self._lock = threading.Lock()        # برای thread-safety (فصل ۱۵)

    def reserve(self, showtime, seat_labels):
        with self._lock:                     # جلوگیری از شرایطِ مسابقه
            seats = [showtime.get_seat(label) for label in seat_labels]
            # اول همه را بررسی کن (یا همه، یا هیچ)
            for seat in seats:
                if seat.status != SeatStatus.AVAILABLE:
                    raise SeatNotAvailableError(
                        f"صندلی {seat.label} قبلاً گرفته شده")
            # حالا همه را رزرو کن
            for seat in seats:
                seat.hold()
            reservation = Reservation(
                reservation_id=str(uuid.uuid4())[:8],
                showtime=showtime,
                seats=seats,
            )
            self.repository.save(reservation)
            return reservation

    def confirm_payment(self, reservation_id):
        reservation = self.repository.get(reservation_id)
        if reservation is None:
            raise BookingError("رزرو یافت نشد")
        if reservation.is_expired():
            self._release(reservation)
            raise BookingError("مهلتِ پرداخت گذشته؛ رزرو منقضی شد")
        for seat in reservation.seats:
            seat.sell()
        reservation.is_paid = True
        return reservation

    def expire_pending(self):
        """رزروهای منقضی را آزاد می‌کند (مثلاً با یک زمان‌بند دوره‌ای صدا زده می‌شود)"""
        for reservation in self.repository.all():
            if reservation.is_expired():
                self._release(reservation)

    def _release(self, reservation):
        for seat in reservation.seats:
            seat.release()
        self.repository.delete(reservation.reservation_id)
```

سه نکته‌ی حیاتی: **اول**، `reserve` با `_lock` محافظت شده تا دو نخ نتوانند همزمان یک صندلی را بگیرند (**شرایطِ مسابقه**، فصل ۱۵) — پاسخِ مستقیم به «فاجعه»ی کارفرما. **دوم**, منطقِ «یا همه یا هیچ»: اول همه‌ی صندلی‌ها را بررسی می‌کنیم، بعد رزرو — تا نصفه‌کاره نماند. **سوم**، `confirm_payment` قبل از فروش، انقضا را چک می‌کند.

### گام ۶: گزارش‌گیری

```python
class BookingService:
    # ... متدهای قبلی ...

    def sales_report(self, showtime):
        sold = [s for s in showtime.seats.values()
                if s.status == SeatStatus.SOLD]
        revenue = sum(s.price for s in sold)
        return {
            "movie": showtime.movie,
            "sold_count": len(sold),
            "available_count": len(showtime.available_seats()),
            "revenue": revenue,
        }
```

### گام ۷: همه‌چیز کنار هم

```python
from datetime import datetime

# راه‌اندازی
repo = ReservationRepository()
service = BookingService(repo)

showtime = Showtime("دون ۲", datetime(2026, 7, 5, 20, 0), "سالن ۱")
showtime.build_hall(rows=["A", "B", "C"], seats_per_row=5, vip_rows=["C"])
print(f"صندلی‌های آزاد: {len(showtime.available_seats())}")   # 15

# رزرو
res = service.reserve(showtime, ["A1", "A2", "C1"])
print(f"رزرو {res.reservation_id} ساخته شد. مبلغ: {res.total_price():,}")
#          (A1+A2 معمولی + C1 وی‌آی‌پی)

# تلاشِ دوم برای همان صندلی — باید شکست بخورد
try:
    service.reserve(showtime, ["A1"])
except SeatNotAvailableError as e:
    print(f"خطا (مطابقِ انتظار): {e}")

# پرداخت
service.confirm_payment(res.reservation_id)
print("پرداخت تأیید شد")

# گزارش
report = service.sales_report(showtime)
print(f"فروخته‌شده: {report['sold_count']}، درآمد: {report['revenue']:,}")
```

### گام ۸: تست‌پذیری — نشان‌دادنِ قدرتِ معماری

چون معماری لایه‌ای است و وابستگی‌ها تزریق می‌شوند (فصل ۷)، تست‌کردن آسان است. مثلاً تستِ انقضا بدونِ انتظارِ واقعیِ ۱۰ دقیقه:

```python
from datetime import datetime, timedelta

def test_expired_reservation_cannot_be_paid():
    repo = ReservationRepository()
    service = BookingService(repo)
    showtime = Showtime("فیلم", datetime.now(), "سالن")
    showtime.build_hall(["A"], 2)

    res = service.reserve(showtime, ["A1"])
    # به‌جای انتظارِ واقعی، created_at را دستکاری می‌کنیم
    res.created_at = datetime.now() - timedelta(minutes=11)

    try:
        service.confirm_payment(res.reservation_id)
        assert False, "باید خطا می‌داد"
    except BookingError:
        pass  # درست: منقضی شده
    # و صندلی باید دوباره آزاد شده باشد
    assert showtime.get_seat("A1").status == SeatStatus.AVAILABLE
    print("✓ تستِ انقضا موفق")

test_expired_reservation_cannot_be_paid()
```

چون منطق در `BookingService` جدا و وابستگی‌هایش تزریق‌شده‌اند، توانستیم بدونِ دیتابیسِ واقعی و بدونِ انتظارِ واقعی، رفتار را تست کنیم. این ثمره‌ی مستقیمِ طراحیِ لایه‌ای و تست‌پذیرِ فصل ۷ است.

---

## جمع‌بندی پروژه

| بخش | مفهوم | فصل |
|-----|-------|-----|
| `_status` محافظت‌شده | کپسول‌سازی، ماشینِ حالت | ۳، ۴ |
| `SeatStatus`, `SeatType` | `Enum` | ۱۱ |
| `Reservation` | `dataclass`, انقضا | ۱۱ |
| سه لایه (Service/Model/Repo) | معماریِ لایه‌ای، SRP | ۷، ۸ |
| تزریقِ `repository` | Dependency Inversion | ۸ |
| `_lock` در `reserve` | thread-safety، شرایطِ مسابقه | ۱۵ |
| تستِ انقضا | طراحیِ تست‌پذیر | ۷ |

**درسِ اصلی:** این پروژه نشان داد که چطور مفاهیمِ پراکنده‌ی پانزده فصل، در یک سیستمِ واقعی به هم گره می‌خورند. کپسول‌سازی حالتِ صندلی را امن نگه داشت، معماریِ لایه‌ای مسئولیت‌ها را جدا کرد، تزریقِ وابستگی تست را ممکن کرد، و قفل، «فاجعه»ی فروشِ دوباره را حل کرد. هیچ‌کدام از این‌ها به‌تنهایی کافی نبود؛ **ترکیبشان** یک سیستمِ حرفه‌ای ساخت.

**تمرینِ پیشنهادی:** سیستم را گسترش دهید تا `expire_pending` را به‌صورتِ خودکار و دوره‌ای (مثلاً هر دقیقه در یک نخِ جدا) اجرا کند. Tradeoffهای thread-safety (فصل ۱۵) را در نظر بگیرید.

---

## پایان دوره

تبریک — به انتهای دوره رسیدید.

از فلسفه‌ی شیءگرایی شروع کردیم، تفکرِ شیءگرا را تمرین کردیم، مفاهیمِ بنیادین و اصولِ چهارگانه را ساختیم، ابزارهای پیشرفته‌ی پایتون را آموختیم، اصولِ طراحی (SOLID، Composition، دیمتر) را فهمیدیم، به عمقِ زبان (متاکلاس، حافظه، Performance) رفتیم، و در سه پروژه‌ی واقعی همه را کنار هم گذاشتیم.

اما مهم‌ترین درسِ کلِ دوره، یک جمله بود که بارها تکرار شد: **هیچ راه‌حلِ مطلقاً درستی وجود ندارد.** ارث‌بری یا Composition؟ `__slots__` یا انعطاف؟ SOLID کامل یا سادگی؟ پاسخ همیشه «بستگی دارد» است. مهارتِ واقعیِ یک طراحِ شیءگرا، نه حفظِ قواعد، بلکه توانایی سنجیدنِ **Tradeoff**ها و انتخابِ بهترینِ موقعیت است.

حالا شما نه‌فقط سینتکسِ شیءگراییِ پایتون، بلکه **طرزِ فکرِ** پشتِ آن را می‌دانید. بروید و بسازید.
