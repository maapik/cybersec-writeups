[← Назад к оглавлению](../../../README.md)

# Duckerz: Скрытая документация

| Параметр | Значение |
| :--- | :--- |
| **Платформа** | Duckerz CTF |
| **Категория** | Web Enumeration (Dirbusting) |
| **Уровень** | Easy |

## Описание

> *"Хорошая документация — залог понятного и безопасного кода"*

Необходимо найти скрытую документацию на веб-сервере.

## Решение

```bash
# 1. Перебор директорий с помощью gobuster
gobuster dir -u http://duckerz.local -w <wordlist>

# 2. Найден эндпоинт:
/docs (200 OK)
```

3. Переходим на `http://duckerz.local/docs` — внизу страницы находим подсказку:  
   `"Документация разработчика: /administrator_secret_folder/creds.txt"`
4. Открываем `http://duckerz.local/administrator_secret_folder/creds.txt` и забираем данные/флаг.
