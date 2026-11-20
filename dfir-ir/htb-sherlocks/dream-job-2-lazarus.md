[← Назад к оглавлению](../../README.md)

# Operation Dream Job-2 — Анализ APT Lazarus

| Параметр | Значение |
| :--- | :--- |
| **Платформа** | Hack The Box (Sherlocks) |
| **Категория** | Threat Intelligence / Malware Triage / DFIR |
| **Инструменты** | MITRE ATT&CK, `isoinfo`, `exiftool`, VirusTotal, `pescan`, `olevba`, `pcode2code` |

## Сценарий

Вы — аналитик по киберугрозам (Threat Intelligence), расследующий операцию «Работа мечты» (Operation Dream Job). Известно, что группировка **Lazarus** использовала кастомное вредоносное ПО и набор инструментов для проведения атак. Задача — проанализировать предоставленные артефакты и восстановить технический профиль атаки.

Предоставленные файлы:

<img width="368" height="181" alt="Предоставленные файлы" src="https://github.com/user-attachments/assets/b420ee0c-d3f4-4ce4-a0d9-9ed8c96e7260" />

---

## Ход расследования и ответы

### Вопрос 1
**Согласно MITRE ATT&CK, с каким ранее известным вредоносным ПО имеет сходство DRATzarus?**

Находим описание `DRATzarus` в базе знаний MITRE ATT&CK:

<img width="909" height="166" alt="DRATzarus в MITRE ATT&CK" src="https://github.com/user-attachments/assets/2bcb1e3d-4a12-43d5-bca5-cce1f9b396f5" />

**Ответ:** `Bankshot`

---

### Вопрос 2
**Какую функцию Windows API использует DRATzarus для определения наличия отладчика?**

В разделе обхода отладки (Debugger Evasion) указан вызов системной функции:

<img width="1258" height="49" alt="Функция проверки отладчика" src="https://github.com/user-attachments/assets/74a4e614-6197-47c1-94f4-3e137f27581a" />

**Ответ:** `IsDebuggerPresent`

---

### Вопрос 3
**Torisma — ещё один инструмент Lazarus. По данным MITRE, она шифрует командные каналы с помощью XOR и какого ещё метода?**

Открываем профиль `Torisma` в MITRE ATT&CK и смотрим блок шифрования C2-каналов:

<img width="1160" height="58" alt="Шифрование Torisma" src="https://github.com/user-attachments/assets/f9ee5cd5-fb6e-4203-89fa-bd2af918b0c6" />

**Ответ:** `VEST-32`

---

### Вопрос 4
**Какой метод сжатия/упаковки использовался для маскировки Torisma?**

В разделе обфускации файлов находим алгоритм сжатия:

<img width="946" height="52" alt="Упаковка Torisma" src="https://github.com/user-attachments/assets/052fc8c0-af24-484b-b093-3c85a1f00dc8" />

**Ответ:** `LZ4 compression`

---

### Вопрос 5
**Проанализируйте предоставленный ISO-образ. Какой исполняемый файл в нём содержится?**

Изучаем содержимое образа через `isoinfo` (в WSL / Linux) и извлекаем файлы:

<img width="750" height="117" alt="Анализ ISO через isoinfo" src="https://github.com/user-attachments/assets/6c7eecea-cb4b-4cc8-82f0-06d9ad788bae" />

<img width="181" height="32" alt="Содержимое ISO" src="https://github.com/user-attachments/assets/50cd1bae-c28d-4bfc-9bbe-28428b6b8d14" />

**Ответ:** `InternalViewer.exe`

---

### Вопрос 6
**Исполняемый файл из предыдущего вопроса был переименован. Каково его исходное имя?**

Смотрим метаданные PE-файла с помощью `exiftool`:

<img width="543" height="27" alt="Метаданные exiftool" src="https://github.com/user-attachments/assets/cb4579d7-ab04-4ab1-9d10-c15f290ecae1" />

**Ответ:** `SumatraPDF.exe`

---

### Вопрос 7
**По данным VirusTotal, когда этот файл впервые был замечен в дикой природе (UTC)?**

Проверяем SHA-256 хэш файла на VirusTotal:

<img width="543" height="42" alt="VirusTotal First Submission" src="https://github.com/user-attachments/assets/554fc9a0-37d6-4fd0-b36e-96f5ed0e8b80" />

**Ответ:** `2020-08-13 08:44:50`

---

### Вопрос 8
**Какой упаковщик использовался для упаковки этого исполняемого файла (полное название)?**

Анализируем секции бинарника утилитой `pescan` и определяем сигнатуру упаковщика (`UPX`):

<img width="874" height="311" alt="Вывод pescan" src="https://github.com/user-attachments/assets/880fb0c3-657f-42da-8876-df3dbe9d4e71" />

<img width="606" height="97" alt="Расшифровка UPX" src="https://github.com/user-attachments/assets/09300643-9358-4e52-8609-0670a6acd841" />

**Ответ:** `Ultimate Packer for eXecutables`

---

### Вопрос 9
**Какой полный URL указан в макросе документа `Salary_Lockheed_Martin_job_opportunities_confidential.doc`?**

Извлекаем и анализируем встроенный VBA-макрос командой `olevba Salary_Lockheed_Martin_job_opportunities_confidential.doc`:

<img width="726" height="509" alt="Вывод olevba" src="https://github.com/user-attachments/assets/a7ea93d6-604b-4ab1-90a6-ab8e1766adb7" />

**Ответ:** `https://markettrendingcenter.com/lk_job_oppor.docx`

---

### Вопросы 10 и 11
**Кто является автором документа `Salary_Lockheed_Martin_job_opportunities_confidential.doc` и кем он был изменён (Last Modified By)?**

Проверяем свойства документа в Windows:

<img width="261" height="60" alt="Свойства документа" src="https://github.com/user-attachments/assets/f4a90ed3-32e3-4cba-8e1b-6d7f3ae31b6c" />

- **Ответ на вопрос 10 (Author):** `Mickey`
- **Ответ на вопрос 11 (Last Modified By):** `Challenger`

---

### Вопрос 12
**Проанализируйте шаблон `17.dotm`. В каком каталоге создавалась подозрительная папка?**

`vipermonkey` не справился с файлом, поэтому проверяем его через `olevba`:

<img width="832" height="428" alt="VBA Stomping в 17.dotm" src="https://github.com/user-attachments/assets/1253a686-8b2a-454a-a702-00204e349add" />

Здесь применён **VBA Stomping** — исходный читаемый текст макроса удалён, остался только скомпилированный p-code, чтобы обойти сигнатуры антивирусов. Восстанавливаем логику утилитой `pcode2code`:

<img width="799" height="408" alt="Декомпиляция p-code через pcode2code" src="https://github.com/user-attachments/assets/23c653ef-f67a-4471-91b8-b32091391305" />

**Ответ:** `\AppData\Local\Microsoft\Notice`

---

### Вопрос 13
**Какой подозрительный файл проверялся на наличие в этом каталоге?**

На том же скриншоте декомпилированного p-code видна проверка файла `wsuser.db`:

**Ответ:** `wsuser.db`

---

## Цепочка атаки (Cyber Kill Chain)

1. **Reconnaissance (Разведка):** поиск целевых email-адресов сотрудников.
2. **Weaponization (Вооружение):** внедрение вредоносного макроса (и p-code stomping) в документ Word.
3. **Delivery (Доставка):** упаковка документа и бинарника-приманки в ISO-образ для обхода Mark-of-the-Web.
4. **Exploitation (Эксплуатация):** открытие документа пользователем и запуск макроса.
5. **Installation (Установка):** создание каталога `\AppData\Local\Microsoft\Notice` и дроп `wsuser.db`.
6. **Command & Control (C2):** обращение за полезной нагрузкой по URL `markettrendingcenter.com`.
7. **Actions on Objectives (Действия):** закрепление и кража данных.
