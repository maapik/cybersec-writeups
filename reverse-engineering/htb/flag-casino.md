[← Назад к оглавлению](../../README.md)

# FlagCasino — Предсказуемый `rand()` и скрипт на `pwntools`

| Параметр | Значение |
| :--- | :--- |
| **Платформа** | Hack The Box |
| **Категория** | Reverse Engineering |
| **Инструменты** | IDA Pro (`F5`), Python (`ctypes`, `pwntools`), `libc` |

## Решение

### 1. Разведка бинарника
При запуске `./casino` программа выводит баннер и ждёт ввод по одному символу:

```text
└─$ ./casino
[ ** WELCOME TO ROBO CASINO **]
     ,     ,
    (\____/)
     (_oo_)
       (O)
     __||__    \)
  []/______\[] /
  / \______/ \/
 /    /__\
(\   /____\
---------------------
[*** PLEASE PLACE YOUR BETS ***]
> 11
[ * INCORRECT * ]
[ *** ACTIVATING SECURITY SYSTEM - PLEASE VACATE *** ]
```

---

### 2. Статический анализ в IDA Pro
Декомпилируем функцию `main`:

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char v4; // [rsp+Bh] [rbp-5h] BYREF
  unsigned int i; // [rsp+Ch] [rbp-4h]

  puts("[ ** WELCOME TO ROBO CASINO **]");
  puts(
    "     ,     ,\n"
    "    (\\____/)\n"
    "     (_oo_)\n"
    "       (O)\n"
    "     __||__    \\)\n"
    "  []/______\\[] /\n"
    "  / \\______/ \\/\n"
    " /    /__\\\n"
    "(\\   /____\\\n"
    "---------------------");
  puts("[*** PLEASE PLACE YOUR BETS ***]");
  for ( i = 0; i <= 0x1D; ++i )
  {
    printf("> ");
    if ( (unsigned int)__isoc99_scanf(" %c", &v4) != 1 )
      exit(-1);
    srand(v4);
    if ( rand() != check[i] )
    {
      puts("[ * INCORRECT * ]");
      puts("[ *** ACTIVATING SECURITY SYSTEM - PLEASE VACATE *** ]");
      exit(-2);
    }
    puts("[ * CORRECT *]");
  }
  puts("[ ** HOUSE BALANCE $0 - PLEASE COME BACK LATER ** ]");
  return 0;
}
```

**Суть механизма:**
- Цикл выполняется от `0` до `0x1D` (`30` символов).
- На каждом шаге введённый символ `v4` передаётся напрямую как seed в `srand(v4)`.
- Затем вызывается `rand()`, и результат сравнивается с эталонным массивом `check[i]`.

Поскольку `srand(x)` + `rand()` из стандартной `libc` полностью детерминированы, мы можем заранее построить таблицу соответствия для всех ASCII-символов.

---

### 3. Эксплойт на Python (`ctypes` + `pwntools`)

```python
import ctypes
from pwn import *

libc = ctypes.CDLL("libc.so.6")
mapping = {}
for i in range(256):
    libc.srand(i)
    mapping[libc.rand()] = chr(i)

flag = ""
e = ELF("./casino", checksec=False)
for j in range(30):
    val = e.u32(e.sym["check"] + j * 4)
    flag += mapping[val]

print(flag)
```
