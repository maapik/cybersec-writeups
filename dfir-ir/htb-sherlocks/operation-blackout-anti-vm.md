[← Назад к оглавлению](../../README.md)

# Operation Blackout 2025 — Обнаружение Anti-VM проверок в PowerShell-логах

| Параметр | Значение |
| :--- | :--- |
| **Платформа** | Hack The Box (Sherlocks) |
| **Категория** | DFIR / PowerShell Log Analysis (`.evtx`) |
| **Артефакты** | `Windows-Powershell-Operational.evtx` (Event ID `4104`), `Microsoft-Windows-Powershell.evtx` (Event ID `800`) |

## Сценарий

Инцидент-респондеры подозревают, что атакующий выполнял Anti-VM / Sandbox-проверки, чтобы избежать анализа в изолированной среде. Нужно изучить журналы PowerShell и зафиксировать конкретные техники проверки реестра, WMI и процессов.

## Исходные артефакты

- **`Windows-Powershell-Operational.evtx` (Event ID `4104` — Script Block Logging):** сохраняет полный исходный код запускаемых блоков скриптов и команд.
- **`Microsoft-Windows-Powershell.evtx` (Event ID `800` — Pipeline Execution Details):** фиксирует факты выполнения команд конвейера и их результаты.

<img width="632" height="111" alt="Event ID 4104 в журнале" src="https://github.com/user-attachments/assets/f4719f61-b823-4384-b0db-cead3ed4edea" />

<img width="601" height="213" alt="Детали события PowerShell" src="https://github.com/user-attachments/assets/a813aac5-e6aa-4336-b3c9-f55b2bc65b1a" />

---

## Хронология действий злоумышленника

### Шаг 1. Проверка модели и производителя через WMI
Злоумышленник запрашивает базовую информацию о железе командой:
```powershell
Get-WmiObject -Class Win32_ComputerSystem
```

<img width="838" height="277" alt="Запрос Win32_ComputerSystem" src="https://github.com/user-attachments/assets/31850cc9-0b44-4095-b75e-d0e34bc8f7c0" />

На физическом ПК в `Manufacturer` / `Model` будут условные Dell, ASUS или Lenovo, тогда как на ВМ часто фигурируют `VMware`, `VirtualBox` или `QEMU`.

---

### Шаг 2. Проверка аппаратных датчиков температуры
Выполняется WMI-запрос к термальным зонам материнской платы:
```sql
SELECT * FROM MSAcpi_ThermalZoneTemperature
```

<img width="703" height="376" alt="Запрос MSAcpi_ThermalZoneTemperature" src="https://github.com/user-attachments/assets/97cdbcb2-9bb6-4666-9b2e-58c3b9e0fb68" />

Большинство виртуальных машин не эмулируют физические термодатчики — пустой вывод или ошибка сразу выдаёт изолированную среду.

---

### Шаг 3. Загрузка скрипта `Check-VM` из фреймворка Nishang
В памяти запускается комплексный модуль `Check-VM` (из известного offensive-фреймворка **Nishang**), содержащий сигнатуры Hyper-V, VMware, VirtualBox и Virtual PC:

<img width="733" height="172" alt="Функция Check-VM" src="https://github.com/user-attachments/assets/b804ce54-5307-45c2-ab4a-3c4cf207e857" />

<img width="539" height="85" alt="Следы Nishang" src="https://github.com/user-attachments/assets/38a8b6b8-9f79-45f5-a252-076099f1c4e7" />

<img width="730" height="404" alt="Проверки каталогов в скрипте" src="https://github.com/user-attachments/assets/5e804953-759b-4c46-80af-8ec34b99cd04" />

---

### Шаг 4. Глубокая проверка реестра и процессов
Скрипт проверяет:
- **Ветку реестра** `HKLM:\SYSTEM\ControlSet001\Services` — поиск служб интеграции гостевых ОС (например, `vmicheartbeat`, `vmicvss` для Hyper-V).
- **Запущенные процессы** через `Get-Process` — поиск гостевых агентов и утилит (например, `vboxservice.exe`, `vboxtray.exe`).

<img width="585" height="392" alt="Проверка служб и процессов в логах" src="https://github.com/user-attachments/assets/a3d40b87-108e-4d43-bdff-a6d3ef93defa" />

---

### Шаг 5. Итог выполнения скрипта (Event ID `800`)
В журнале `Microsoft-Windows-Powershell.evtx` (Event ID `800`) зафиксирован результат работы `Check-VM` — строки `"This is a Hyper-V machine."` и `"This is a VMWare machine."`:

<img width="555" height="361" alt="Вывод скрипта Check-VM в Event ID 800" src="https://github.com/user-attachments/assets/71c84aef-9e24-43ec-a035-b2d311cd1bbb" />
