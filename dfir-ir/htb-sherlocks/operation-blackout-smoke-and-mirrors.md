[← Назад к оглавлению](../../README.md)

# Operation Blackout 2025: Smoke & Mirrors — Обход средств защиты

| Параметр | Значение |
| :--- | :--- |
| **Платформа** | Hack The Box (Sherlocks) |
| **Категория** | Incident Response / Windows Event Logs (`.evtx`) |
| **Ключевые темы** | Defense Evasion, LSA Protection, Windows Defender, AMSI Bypass, Safe Mode |

## Сценарий

Специалист по расследованию инцидентов Рейес изучает скрытую постэксплуатационную активность, при которой часть ожидаемых журналов безопасности и алертов Windows Defender отсутствует. По имеющимся экспортированным журналам событий необходимо определить, как именно злоумышленник отключил или обошёл механизмы защиты системы.

---

## Ход расследования и ответы

### Вопрос 1
**Злоумышленник отключил защиту LSA (LSA Protection), изменив ключ реестра. Укажите полный путь к этому ключу.**

Ищем в логах команды модификации реестра (`reg add`):

<img width="532" height="105" alt="Поиск reg add для LSA" src="https://github.com/user-attachments/assets/8e93b02d-0e7c-426d-a157-ad2bc6fe08f2" />

**Ответ:** `HKLM\SYSTEM\CurrentControlSet\Control\LSA`

---

### Вопрос 2
**Какую команду PowerShell злоумышленник выполнил первой для отключения Windows Defender?**

Отслеживаем вызовы командлета `Set-MpPreference` в PowerShell-логах:

<img width="708" height="553" alt="Команда Set-MpPreference" src="https://github.com/user-attachments/assets/2e127220-395f-4661-93d6-a95d9fcfc813" />

**Ответ:** `Set-MpPreference -DisableIOAVProtection $true -DisableEmailScanning $true -DisableBlockAtFirstSeen $true`

---

### Вопрос 3
**Злоумышленник загрузил PowerShell-патч AMSI. Какую функцию в DLL-библиотеке модифицирует скрипт для отключения AMSI?**

Фильтруем события по `.dll` и находим вызов `GetProcAddress(...)`, где имя функции собрано посимвольно:

<img width="728" height="483" alt="Патч AmsiScanBuffer" src="https://github.com/user-attachments/assets/ecc022ee-fa0b-49df-93b1-1fadc69740ef" />

**Ответ:** `AmsiScanBuffer`

---

### Вопрос 4
**Какую команду использовал злоумышленник для перезагрузки хоста в безопасном режиме с поддержкой сети?**

Фильтруем по параметру `safeboot` и находим вызов утилиты `bcdedit.exe`:

<img width="535" height="181" alt="Команда bcdedit safeboot" src="https://github.com/user-attachments/assets/0574b4cc-a60d-4c90-a5fd-457cac248212" />

**Ответ:** `bcdedit.exe /set safeboot network`

---

### Вопрос 5
**Какую команду PowerShell злоумышленник применил для отключения записи истории команд?**

Модуль `PSReadLine` отвечает за сохранение истории консоли; отфильтровываем `Set-PSReadlineOption`:

<img width="644" height="375" alt="Отключение истории PowerShell" src="https://github.com/user-attachments/assets/b2ffb8b1-e9f9-41a0-aaa6-0c20864b6a8c" />

**Ответ:** `Set-PSReadlineOption -HistorySaveStyle SaveNothing`
