# typing-moduli-bilan-aniq-tiplashtirish
Python-da `typing` moduli yordamida murakkab tiplarni ifodalash, generic konteyner yaratish hamda `Protocol` orqali strukturaviy tiplashtirish (duck typing) ko'rsatilgan tayyor tiplashtirilgan modul:

```python
"""
typing moduli bilan aniq tiplashtirish namunasi.
Ushbu modul `mypy` tip tekshiruvchisidan toza (xatosiz) o'tadi.
"""

from typing import Optional, Union, TypeVar, Generic, Protocol, List


# ==============================================================================
# 1. Optional va Union misoli
# ==============================================================================

# Izoh: Optional[X] bu aynan Union[X, None] ning qisqartmasidir.
# Ya'ni, Optional[str] va Union[str, None] bir xil ma'noni anglatadi.

def find_user(user_id: int) -> Optional[str]:
    """Foydalanuvchi ismini qaytaradi yoki topilmasa None beradi."""
    database = {1: "Ali", 2: "Vali"}
    return database.get(user_id)  # str yoki None qaytaradi


def process_id(identifier: Union[int, str]) -> str:
    """Parametr sifatida int yoki str qabul qiluvchi funksiya."""
    if isinstance(identifier, int):
        return f"ID raqami: {identifier}"
    return f"ID kodi: {identifier.upper()}"


# Union funksiyasini ikkala tip bilan chaqirish
result_int = process_id(101)
result_str = process_id("usr_101")


# ==============================================================================
# 2. TypeVar va Generic bilan tiplashtirilgan Stack klassi
# ==============================================================================

T = TypeVar('T')


class Stack(Generic[T]):
    """TypeVar('T') yordamida yaratilgan generic Stack konteyneri."""

    def __init__(self) -> None:
        self._items: List[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        if not self._items:
            raise IndexError("Stack bo'sh!")
        return self._items.pop()

    def is_empty(self) -> bool:
        return len(self._items) == 0


# Generic'ni aniq tip (int) bilan instansiyalash
int_stack: Stack[int] = Stack[int]()
int_stack.push(10)
int_stack.push(20)

# TIP TEKSHIRUVCHI RAD ETADIGAN CHAQRUV:
# mypy ushbu qatorda xatolik beradi, chunki `Stack[int]` faqat int qabul qiladi:
# int_stack.push("hello")  # Error: Argument 1 to "push" of "Stack" has incompatible type "str"; expected "int"


# ==============================================================================
# 3. Protocol va Strukturaviy Tiplashtirish (Duck Typing)
# ==============================================================================

class Renderable(Protocol):
    """Renderable protokoli: Har qanday `render` metodiga ega ob'ektni belgilaydi."""

    def render(self) -> str:
        ...


# Renderable protokolidan meros OLMAGAN birinchi klass
class HTMLButton:
    def render(self) -> str:
        return "<button>Bosing</button>"


# Renderable protokolidan meros OLMAGAN ikkinchi klass
class TextWidget:
    def render(self) -> str:
        return "[Matnli vidjet]"


def display_component(component: Renderable) -> None:
    """Protocol'ga mos har qanday ob'ektni (meros olmagan bo'lsa ham) qabul qiladi."""
    print(component.render())


# Ikkala klass ham Renderable protokoli shakliga to'g'ri kelgani uchun tip tekshiruvi o'tadi
button = HTMLButton()
widget = TextWidget()

display_component(button)  # Xatosiz ishlaydi
display_component(widget)  # Xatosiz ishlaydi

```

### Kod va tushunchalar izohi:

1. **`Optional[X]` vs `Union[X, None]**`: `Optional[str]` ko'rinishida yozilgan annotatsiya kapot ostida `Union[str, None]` bilan mutlaqo bir xil. U funksiya natijasi ma'lumot bo'lishi yoki umuman qiymat bo'lmasligi (`None`) mumkinligini bildiradi.
2. **`Union[int, str]`**: Funksiya bir nechta aniq tiplardan birini qabul qila olishini ta'minlaydi.
3. **`TypeVar` va `Generic**`: `Stack[T]` klassi ma'lumot turidan qat'i nazar qayta ishlatilishi mumkin. Birom `Stack[int]` qilib e'lon qilingandan so'ng `mypy` unga `str` qo'shish harakatini darhol rad etadi.
4. **`Protocol` (Duck Typing)**: `HTMLButton` va `TextWidget` klasslari `Renderable` protokolini ochiqchasiga `class HTMLButton(Renderable):` ko'rinishida meros olmagan. Shunga qaramay, ularning tarkibida `render() -> str` metodi borligi uchun `display_component` funksiyasi ularni xatosiz qabul qiladi.

Ushbu kodni `.py` faylga saqlab `mypy fayl_nomi.py` buyrug'i orqali tekshirsangiz, `Success: no issues found in 1 source file` natijasini olasiz.
