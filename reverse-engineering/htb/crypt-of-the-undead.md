[← Назад к оглавлению](../../README.md)

# Crypt of the Undead — Декомпиляция в IDA Pro

| Параметр | Значение |
| :--- | :--- |
| **Платформа** | Hack The Box |
| **Категория** | Reverse Engineering |
| **Инструменты** | IDA Pro (Hex-Rays Decompiler `F5`) |

## Условие задачи

> *Все мои важные файлы были зомбированы печально известной бандой киберпреступников-зомби. Можете ли вы вернуть их и успокоить?*

В задаче даны два файла: бинарник `crypt` и зашифрованный `flag.txt.undead`.

<img width="754" height="63" alt="Условие задачи" src="https://github.com/user-attachments/assets/d0724b1a-77a5-4bcd-8a35-f8acc92abe50" />

---

## Решение

### 1. Первичный запуск бинарника
Запускаем `crypt` на виртуальной машине, чтобы посмотреть ожидаемые аргументы:

<img width="294" height="57" alt="Справка по запуску crypt" src="https://github.com/user-attachments/assets/6aa1c258-a0ab-48d8-8176-1c5a849084a1" />

Пытаемся передать программе `flag.txt.undead`, но получаем ошибку:

<img width="477" height="70" alt="Ошибка .undead" src="https://github.com/user-attachments/assets/19c425b5-a008-441d-badb-c5bce50ed578" />

*(«То, что уже нежить, не может быть зашифровано»).*

---

### 2. Декомпиляция в IDA Pro
Открываем `crypt` в IDA Pro и жмём `F5` для просмотра псевдокода:

<img width="873" height="650" alt="Псевдокод main в IDA Pro (1)" src="https://github.com/user-attachments/assets/2120f881-77c2-4189-93e8-a88dc71d08cb" />

<img width="1061" height="697" alt="Псевдокод main в IDA Pro (2)" src="https://github.com/user-attachments/assets/a8b29e2b-41f5-4cff-8c52-76de563bfed0" />

Находим ветку с текстом ошибки:

<img width="569" height="134" alt="Проверка расширения .undead" src="https://github.com/user-attachments/assets/dec6e29e-7b68-4ca9-9986-b08c676a6575" />

Логика простая: если строка `old_filename` заканчивается на `.undead`, программа отказывается обрабатывать файл. По алгоритму же внутри используется обратимая операция (XOR/потоковое шифрование), поэтому повторный прогон через `crypt` расшифрует файл обратно.

---

### 3. Обход проверки и получение флага
Переименовываем `flag.txt.undead` в `flag.txt` и снова передаём его бинарнику `crypt`:

<img width="754" height="63" alt="Успешная обработка файла" src="https://github.com/user-attachments/assets/809f1e6d-33bf-4477-9e8a-e404c4a19386" />

Программа заново создаёт `flag.txt.undead` с расшифрованным содержимым. Открываем его и забираем флаг!
